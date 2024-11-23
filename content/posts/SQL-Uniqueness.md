+++
date = '2024-11-22T14:34:54+01:00'
title = 'SQL Uniqueness'
tags = ['ChatGPT', 'SQL']
+++

Brief interaction with Chat Jippitee on Uniqueness in SQL, when you need to make sure you get a certain column with only unique values... there are several approaches.


## Using `ROW_NUMBER`
You can use the `ROW_NUMBER` window function to ensure a single row per `App_Id`. For example:

```sql
WITH RankedData AS (
    SELECT App_Id, 
           Owner, 
           Environment,
           ROW_NUMBER() OVER (PARTITION BY App_Id ORDER BY App_Id) AS RowNum
    FROM All_CIs_Table
)
SELECT App_Id, Owner, Environment
FROM RankedData
WHERE RowNum = 1;
```

### Pros:
- Deterministic: You can control which row to keep using the `ORDER BY` clause
- Performance: Typically more efficient than `DISTINCT` for large datasets
- Flexibility: Allows you to specify criteria for prioritizing rows

### Cons:
- Complexity: Slightly harder to read if you're not familiar with window functions
- Data Assumptions: You need a sensible `ORDER BY` clause to pick the "first" row


## Using `GROUP BY` with Aggregates
You can group by `App_Id` and pick one representative row for each group using aggregate functions.

```sql
SELECT App_Id,
       MAX(Owner) AS Owner,
       MAX(Environment) AS Environment
FROM All_CIs_Table
GROUP BY App_Id;
```

### Pros:
- Simplicity: Concise and easy to understand
- Performance: `GROUP BY` is often faster than `DISTINCT`
- Deterministic: Explicitly defines how duplicates are resolved

### Cons:
- Loss of Detail: You only get aggregated values
- Limited Control: You can't prioritize which row is retained

## Using `OUTER APPLY` or `CROSS APPLY`
You can use `APPLY` to fetch a single row for each `App_Id`.

```sql
SELECT App_Id, Owner, Environment
FROM All_CIs_Table AS A
OUTER APPLY (
    SELECT TOP 1 Owner, Environment
    FROM All_CIs_Table AS B
    WHERE A.App_Id = B.App_Id
    ORDER BY B.Owner, B.Environment
) AS Subquery;
```

### Pros:
- Deterministic: Control over which row is selected with `ORDER BY`
- Flexibility: Can apply more complex logic in the subquery

### Cons:
- Complexity: Harder to read and maintain than other approaches
- Performance: Can be slower for very large datasets


## Comparison Table

| Approach         | Clarity     | Performance     | Control Over Rows | Complexity |
|-------------------|-------------|-----------------|-------------------|------------|
| `ROW_NUMBER`     | Moderate    | High            | High              | Moderate   |
| `GROUP BY`       | High        | High            | Low               | Easy       |
| `OUTER APPLY`    | Moderate    | Moderate        | High              | High       |


### Best Practice Recommendation
- Use `ROW_NUMBER` if you need precise control over which row to keep
- Use `GROUP BY` if you only need one row per `App_Id` without caring which specific row is chosen

Choose based on the specific constraints and requirements of your dataset and use case!

## ROW_NUMBER Approach

The `ROW_NUMBER` approach provides excellent control over which row is kept by allowing you to customize the `ORDER BY` clause. Here's a breakdown of how you can control the row selection and some strategies to prioritize rows:


### Controlling by Null Values
If you want to prioritize rows with the least number of `NULL` values, you can calculate the count of `NULL` values and use that as part of the `ORDER BY` clause. For example:

```sql
WITH RankedData AS (
    SELECT App_Id, 
           Owner, 
           Environment,
           ROW_NUMBER() OVER (
               PARTITION BY App_Id 
               ORDER BY 
                   (CASE WHEN Owner IS NULL THEN 1 ELSE 0 END +
                    CASE WHEN Environment IS NULL THEN 1 ELSE 0 END) ASC,
                   App_Id -- Break ties if needed
           ) AS RowNum
    FROM All_CIs_Table
)
SELECT App_Id, Owner, Environment
FROM RankedData
WHERE RowNum = 1;
```

### Explanation:
- The `(CASE WHEN ... IS NULL THEN 1 ELSE 0 END)` counts `NULL` values for each column
- Rows with fewer `NULL` values are prioritized (sorted in ascending order)
- Add additional `ORDER BY` criteria (e.g., `App_Id`) to resolve ties


### Controlling by Specific Column Values
If you want to prioritize rows based on a specific column (e.g., `Environment`), you can use `ORDER BY` directly:

```sql
WITH RankedData AS (
    SELECT App_Id, 
           Owner, 
           Environment,
           ROW_NUMBER() OVER (
               PARTITION BY App_Id 
               ORDER BY 
                   Environment DESC, -- Example: prioritize specific values
                   App_Id
           ) AS RowNum
    FROM All_CIs_Table
)
SELECT App_Id, Owner, Environment
FROM RankedData
WHERE RowNum = 1;
```

### Example Scenarios:
- Sort `Environment` values alphabetically or by specific logic
- Use a similar approach for other columns based on business rules


### Controlling by Timestamp
To select the "latest" or "earliest" row for each `App_Id`, you can sort by a timestamp column:

```sql
WITH RankedData AS (
    SELECT App_Id, 
           Owner, 
           Environment,
           ROW_NUMBER() OVER (
               PARTITION BY App_Id 
               ORDER BY LastModifiedDate DESC -- Latest row
           ) AS RowNum
    FROM All_CIs_Table
)
SELECT App_Id, Owner, Environment
FROM RankedData
WHERE RowNum = 1;
```

### Explanation:
- Replace `LastModifiedDate` with any timestamp column that represents the creation or update time of rows


### Custom Tie-Breaking Logic
To ensure deterministic results when rows tie on other criteria, add secondary and tertiary sorting levels:

```sql
WITH RankedData AS (
    SELECT App_Id, 
           Owner, 
           Environment,
           ROW_NUMBER() OVER (
               PARTITION BY App_Id 
               ORDER BY 
                   LastModifiedDate DESC, -- Primary sort (e.g., latest row)
                   Environment,          -- Secondary sort (alphabetical)
                   Owner                 -- Tertiary sort (alphabetical)
           ) AS RowNum
    FROM All_CIs_Table
)
SELECT App_Id, Owner, Environment
FROM RankedData
WHERE RowNum = 1;
```


### Controlling by Business Logic
Sometimes you may have complex criteria, such as prioritizing rows where `Owner` has a specific value. You can use a `CASE` statement in the `ORDER BY` clause:

```sql
WITH RankedData AS (
    SELECT App_Id, 
           Owner, 
           Environment,
           ROW_NUMBER() OVER (
               PARTITION BY App_Id 
               ORDER BY 
                   CASE WHEN Owner = 'PreferredOwner' THEN 1 ELSE 2 END ASC, -- Prioritize specific owners
                   LastModifiedDate DESC -- Tie-break by latest timestamp
           ) AS RowNum
    FROM All_CIs_Table
)
SELECT App_Id, Owner, Environment
FROM RankedData
WHERE RowNum = 1;
```

### Explanation:
- Rows with `Owner = 'PreferredOwner'` are sorted first
- Additional criteria handle ties


### General Guidelines for Using `ORDER BY` in `ROW_NUMBER`
- Combine Logic: Use `CASE` statements to encode priority rules
- Handle Ties Explicitly: Always include additional sorting fields to ensure deterministic results
- Performance Considerations: Optimize indexes on the columns used in `ORDER BY` where possible

These strategies allow you to tailor the row selection logic to your specific requirements while maintaining uniqueness on the `App_Id` column.