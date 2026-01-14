# Advanced Temporal Patterns
## The "Gaps and Islands" Problem
### Problem Statement
You have a table of user logins and want to find "streaks"—consecutive days a user logged in. If a user logs in on Jan 1, 2, 3, and 5, there are two "islands" (1-3 and 5).

### Optimized SQL:

```
WITH login_groups AS (
    SELECT 
        user_id,
        login_date,
        -- Subtract a sequence from the date to create a constant "group" key
        login_date - INTERVAL '1 day' * ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY login_date) as grp
    FROM user_logins
)
SELECT 
    user_id,
    MIN(login_date) as streak_start,
    MAX(login_date) as streak_end,
    COUNT(*) as days_in_streak
FROM login_groups
GROUP BY user_id, grp
ORDER BY days_in_streak DESC;
```

### Explanation
The magic here is the grp calculation. When dates are consecutive, they increase at the same rate as the ROW_NUMBER. When you subtract them, the result is a static date for the entire "island." When there is a gap, the ROW_NUMBER resets the subtraction value, creating a new group.

#### Performance Considerations: Requires a sort on login_date.

Highly efficient compared to recursive CTEs for large datasets.


---

## Sessionization with Cumulative Sums
### Problem Statement
Web traffic logs show timestamps. You need to group hits into "sessions." A session ends if there is a gap of 30 minutes or more between hits.

### Optimized SQL:

```
WITH time_diffs AS (
    SELECT 
        user_id,
        timestamp,
        LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp) as prev_ts
    FROM web_logs
),
session_starts AS (
    SELECT 
        *,
        CASE WHEN timestamp > prev_ts + INTERVAL '30 minutes' OR prev_ts IS NULL 
             THEN 1 ELSE 0 END as is_new_session
    FROM time_diffs
)
SELECT 
    user_id,
    timestamp,
    SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY timestamp ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as session_id
FROM session_starts;
```

### Explanation
#### LAG: Fetches the previous event time.

#### Boolean Logic: Compares current and previous time to flag a new session.

#### Cumulative SUM: Acts as a running counter that increments only when is_new_session is 1, effectively assigning a unique ID to each session "island."

#### Performance Considerations
Uses two window function passes. In BigQuery/Snowflake, try to keep the PARTITION BY and ORDER BY identical to allow the engine to reuse the sorted data.



---

## Slowly Changing Dimensions (SCD) Type 2
### Problem Statement
You need to track the history of a customer's subscription tier. If they upgrade, you don't overwrite the old row; you expire it and create a new one to allow for accurate "as-of" reporting.

### Optimized SQL (Using Lead for Effective/Expiry Dates)

```
SELECT 
    customer_id,
    subscription_tier,
    changed_at as effective_from,
    LEAD(changed_at, 1, '9999-12-31') OVER (PARTITION BY customer_id ORDER BY changed_at) as effective_to,
    CASE WHEN LEAD(changed_at) OVER (PARTITION BY customer_id ORDER BY changed_at) IS NULL 
         THEN TRUE ELSE FALSE END as is_current
FROM subscription_history;
```

### Explanation
This pattern uses LEAD to look ahead at the next record's start date and use it as the current record's end date. The 9999-12-31 placeholder represents an active record. This is a foundational pattern for Point-in-Time (PIT) joins in analytics.

#### Performance Considerations

Indexing customer_id and changed_at is critical for lookup performance.

In dbt (Data Build Tool), this is often automated via "Snapshots," but understanding the underlying SQL is vital for debugging.

---

## N-Step Funnel Analysis

**Problem Statement:** Calculate the conversion rate and drop-off through a specific sequence of events: `Sign Up` → `Email Verified` → `First Purchase`. You need to know how many users reached each step and the percentage that "survived" from the previous step.

**Optimized SQL (Using Conditional Aggregation):**

```sql
WITH user_events AS (
    SELECT 
        user_id,
        MIN(CASE WHEN event_name = 'sign_up' THEN event_time END) as signup_at,
        MIN(CASE WHEN event_name = 'email_verified' THEN event_time END) as verified_at,
        MIN(CASE WHEN event_name = 'first_purchase' THEN event_time END) as purchase_at
    FROM events
    GROUP BY 1
),
funnel_counts AS (
    SELECT
        COUNT(signup_at) as step_1_all,
        COUNT(CASE WHEN verified_at > signup_at THEN 1 END) as step_2_verified,
        COUNT(CASE WHEN purchase_at > verified_at THEN 1 END) as step_3_purchased
    FROM user_events
)
SELECT 
    step_1_all,
    step_2_verified,
    step_3_purchased,
    ROUND(100.0 * step_2_verified / NULLIF(step_1_all, 0), 2) as pct_signup_to_verify,
    ROUND(100.0 * step_3_purchased / NULLIF(step_2_verified, 0), 2) as pct_verify_to_purchase
FROM funnel_counts;

```

**Explanation:**

* **Step 1:** Use `MIN(CASE...)` to flatten the event log into a single row per user with the first occurrence of each milestone.
* **Step 2:** Apply "Sequencing Logic" in the counts. A user only counts for Step 3 if their `purchase_at` is chronologically after their `verified_at`.
* **Step 3:** Calculate the ratios. Using `NULLIF` prevents "Division by Zero" errors if a step has no users.

**Performance Considerations:**

* This "Flattening" approach is significantly faster than performing three separate self-joins on a large event table.

---

## Classic Retention (N-Day Retention)

**Problem Statement:** For users who joined on a specific date, what percentage of them returned to the app exactly 1 day later, 7 days later, and 30 days later?

**Optimized SQL:**

```sql
WITH user_activity AS (
    -- Get the first date for every user (Cohort Date)
    SELECT 
        user_id,
        MIN(event_date) OVER(PARTITION BY user_id) as join_date,
        event_date
    FROM user_logins
),
retention_flags AS (
    SELECT 
        join_date,
        user_id,
        MAX(CASE WHEN event_date = join_date + INTERVAL '1 day' THEN 1 ELSE 0 END) as d1_retention,
        MAX(CASE WHEN event_date = join_date + INTERVAL '7 day' THEN 1 ELSE 0 END) as d7_retention
    FROM user_activity
    GROUP BY 1, 2
)
SELECT 
    join_date,
    COUNT(user_id) as cohort_size,
    SUM(d1_retention) as d1_count,
    SUM(d7_retention) as d7_count,
    ROUND(100.0 * SUM(d1_retention) / COUNT(user_id), 2) as d1_pct
FROM retention_flags
GROUP BY 1
ORDER BY 1 DESC;

```

**Explanation:**

* **Cohort Definition:** The `MIN() OVER()` window function identifies the "Birth Date" for every user.
* **Comparison:** We check if any record for that user exists exactly X days after their join date.
* **Aggregation:** We group by the `join_date` to see how different "classes" of users perform over time.

**Performance Considerations:**

* Large datasets benefit from filtering `user_activity` to a specific date range (e.g., the last 90 days) before running the window function to reduce the shuffle size.

---

## Cohort Heatmap (Month-over-Month)

**Problem Statement:** Build a matrix where rows are the "Join Month" and columns are the "Months Since Joining." This is the standard view used by VCs and Product Managers to check product-market fit.

**Optimized SQL:**

```sql
WITH cohort_source AS (
    SELECT 
        user_id,
        DATE_TRUNC('month', MIN(event_date) OVER(PARTITION BY user_id)) as cohort_month,
        DATE_TRUNC('month', event_date) as activity_month
    FROM user_logins
),
cohort_sizes AS (
    SELECT cohort_month, COUNT(DISTINCT user_id) as total_users
    FROM cohort_source
    GROUP BY 1
),
retention_table AS (
    SELECT 
        s.cohort_month,
        -- Calculate number of months between join and activity
        PERIOD_DIFF(EXTRACT(YEAR_MONTH FROM s.activity_month), 
                    EXTRACT(YEAR_MONTH FROM s.cohort_month)) as month_number,
        COUNT(DISTINCT s.user_id) as active_users
    FROM cohort_source s
    GROUP BY 1, 2
)
SELECT 
    r.cohort_month,
    sz.total_users,
    r.month_number,
    ROUND(100.0 * r.active_users / sz.total_users, 2) as retention_pct
FROM retention_table r
JOIN cohort_sizes sz ON r.cohort_month = sz.cohort_month
WHERE r.month_number <= 12  -- Limit to first year
ORDER BY 1, 3;

```

**Explanation:**

* **`month_number`:** This creates the "Index" (Month 0, Month 1, Month 2). Month 0 is always 100%.
* **Self-Correction:** By dividing the `active_users` in Month N by the `total_users` in the original cohort, you get the classic "Heatmap" values.

**Performance Considerations:**

* In columnar databases (BigQuery/Snowflake), `COUNT(DISTINCT)` is expensive. If your `user_logins` table is massive, consider creating a daily or monthly "User Activity" summary table first.

---
