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