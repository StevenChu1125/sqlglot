# SparkSQL to Doris SQL Migration Guide

## Table of Contents
- [Introduction](#introduction)
- [Basic Usage](#basic-usage)
- [Data Types](#data-types)
- [Date and Time Functions](#date-and-time-functions)
- [String Functions](#string-functions)
- [Aggregate Functions](#aggregate-functions)
- [Array Functions](#array-functions)
- [Window Functions](#window-functions)
- [Conditional Expressions](#conditional-expressions)
- [Common Patterns](#common-patterns)
- [Known Limitations](#known-limitations)
- [Best Practices](#best-practices)

## Introduction

This guide provides a comprehensive reference for migrating SQL queries from SparkSQL to Doris SQL using SQLGlot. SQLGlot is a powerful SQL parser and transpiler that can automatically convert queries between different SQL dialects.

Apache Doris is a high-performance, real-time analytical database. While it shares many SQL features with SparkSQL, there are important differences in syntax and function implementations that SQLGlot handles automatically.

## Basic Usage

To convert a SparkSQL query to Doris SQL:

```python
import sqlglot

# Single query conversion
spark_sql = "SELECT CURRENT_DATE, CURRENT_TIMESTAMP"
doris_sql = sqlglot.transpile(spark_sql, read='spark', write='doris')[0]
print(doris_sql)
# Output: SELECT CURRENT_DATE(), NOW()

# Multiple queries
queries = sqlglot.transpile(
    "SELECT 1; SELECT 2",
    read='spark',
    write='doris'
)
```

You can also use the `parse` and `sql` methods for more control:

```python
import sqlglot

# Parse SparkSQL and generate Doris SQL
parsed = sqlglot.parse_one("SELECT CURRENT_DATE", read='spark')
doris_sql = parsed.sql(dialect='doris')
print(doris_sql)
# Output: SELECT CURRENT_DATE()
```

## Data Types

SQLGlot automatically converts data types between SparkSQL and Doris:

| SparkSQL Type | Doris Type | Notes |
|--------------|-----------|-------|
| `STRING` | `STRING` | Direct mapping |
| `INT` | `INT` | Direct mapping |
| `BIGINT` | `BIGINT` | Direct mapping |
| `DOUBLE` | `DOUBLE` | Direct mapping |
| `FLOAT` | `FLOAT` | Direct mapping |
| `BOOLEAN` | `BOOLEAN` | Direct mapping |
| `TIMESTAMP` | `DATETIME` | Doris uses DATETIME |
| `DATE` | `DATE` | Direct mapping |
| `DECIMAL` | `DECIMAL` | Direct mapping |
| `ARRAY<T>` | `ARRAY<T>` | Supported in Doris |

### Example

```python
spark_sql = "SELECT CAST(col AS STRING), CAST(num AS INT)"
doris_sql = sqlglot.transpile(spark_sql, read='spark', write='doris')[0]
# Output: SELECT CAST(col AS STRING), CAST(num AS INT)
```

## Date and Time Functions

### Current Date and Time

**SparkSQL:**
```sql
SELECT CURRENT_DATE, CURRENT_TIMESTAMP
```

**Doris SQL:**
```sql
SELECT CURRENT_DATE(), NOW()
```

### DATE_ADD

Add days to a date.

**SparkSQL:**
```sql
SELECT DATE_ADD(date_col, 5)
```

**Doris SQL:**
```sql
SELECT DATE_ADD(date_col, 5)
```

### DATEDIFF

Calculate the difference between two dates.

**SparkSQL:**
```sql
SELECT DATEDIFF(end_date, start_date)
```

**Doris SQL:**
```sql
SELECT DATEDIFF(end_date, start_date)
```

### DATE_TRUNC

Truncate a date to a specified unit.

**SparkSQL:**
```sql
SELECT DATE_TRUNC('month', date_col)
```

**Doris SQL:**
```sql
SELECT DATE_TRUNC(date_col, 'MONTH')
```

Note: The argument order is reversed in Doris.

### FROM_UNIXTIME

Convert Unix timestamp to datetime.

**SparkSQL:**
```sql
SELECT FROM_UNIXTIME(unix_ts)
```

**Doris SQL:**
```sql
SELECT FROM_UNIXTIME(unix_ts, '%Y-%m-%d %T')
```

Note: Doris requires an explicit format string.

### UNIX_TIMESTAMP

Convert datetime to Unix timestamp.

**SparkSQL:**
```sql
SELECT UNIX_TIMESTAMP(datetime_col)
```

**Doris SQL:**
```sql
SELECT UNIX_TIMESTAMP(datetime_col)
```

## String Functions

Most string functions have direct equivalents between SparkSQL and Doris.

### CONCAT

Concatenate strings.

**SparkSQL:**
```sql
SELECT CONCAT(col1, col2, col3)
```

**Doris SQL:**
```sql
SELECT CONCAT(col1, col2, col3)
```

### CONCAT_WS

Concatenate strings with a separator.

**SparkSQL:**
```sql
SELECT CONCAT_WS(',', col1, col2, col3)
```

**Doris SQL:**
```sql
SELECT CONCAT_WS(',', col1, col2, col3)
```

### SUBSTRING

Extract a substring.

**SparkSQL:**
```sql
SELECT SUBSTRING(str, 1, 5)
```

**Doris SQL:**
```sql
SELECT SUBSTRING(str, 1, 5)
```

### UPPER / LOWER

Convert case.

**SparkSQL:**
```sql
SELECT UPPER(col), LOWER(col)
```

**Doris SQL:**
```sql
SELECT UPPER(col), LOWER(col)
```

### LENGTH

Get string length.

**SparkSQL:**
```sql
SELECT LENGTH(str)
```

**Doris SQL:**
```sql
SELECT LENGTH(str)
```

## Aggregate Functions

### COUNT

Count rows or non-null values.

**SparkSQL:**
```sql
SELECT COUNT(*), COUNT(DISTINCT col)
```

**Doris SQL:**
```sql
SELECT COUNT(*), COUNT(DISTINCT col)
```

### SUM, AVG, MIN, MAX

Standard aggregate functions.

**SparkSQL:**
```sql
SELECT SUM(amount), AVG(amount), MIN(amount), MAX(amount)
```

**Doris SQL:**
```sql
SELECT SUM(amount), AVG(amount), MIN(amount), MAX(amount)
```

### COLLECT_SET

Collect unique values into an array.

**SparkSQL:**
```sql
SELECT COLLECT_SET(col)
```

**Doris SQL:**
```sql
SELECT COLLECT_SET(col)
```

Note: This function is available in Doris as a direct equivalent.

### COLLECT_LIST / ARRAY_AGG

Collect all values into an array.

**SparkSQL:**
```sql
SELECT COLLECT_LIST(col)
-- or
SELECT ARRAY_AGG(col)
```

**Doris SQL:**
```sql
SELECT COLLECT_LIST(col)
```

Note: `ARRAY_AGG` is automatically converted to `COLLECT_LIST` in Doris.

## Array Functions

Doris supports array operations, and SQLGlot handles the conversion appropriately.

### ARRAY

Create an array.

**SparkSQL:**
```sql
SELECT ARRAY(1, 2, 3)
```

**Doris SQL:**
```sql
SELECT ARRAY(1, 2, 3)
```

### SIZE / ARRAY_LENGTH

Get array size.

**SparkSQL:**
```sql
SELECT SIZE(array_col)
```

**Doris SQL:**
```sql
SELECT ARRAY_LENGTH(array_col)
```

Note: `SIZE` is converted to `ARRAY_LENGTH` in Doris.

### ARRAY_CONTAINS

Check if array contains a value.

**SparkSQL:**
```sql
SELECT ARRAY_CONTAINS(array_col, value)
```

**Doris SQL:**
```sql
SELECT ARRAY_CONTAINS(array_col, value)
```

## Window Functions

Window functions are well-supported in both SparkSQL and Doris.

### ROW_NUMBER

Assign sequential row numbers.

**SparkSQL:**
```sql
SELECT ROW_NUMBER() OVER (PARTITION BY col1 ORDER BY col2)
```

**Doris SQL:**
```sql
SELECT ROW_NUMBER() OVER (PARTITION BY col1 ORDER BY col2)
```

### LAG / LEAD

Access previous or next row values.

**SparkSQL:**
```sql
SELECT 
  LAG(col, 1) OVER (ORDER BY date_col),
  LEAD(col, 1) OVER (ORDER BY date_col)
```

**Doris SQL:**
```sql
SELECT 
  LAG(col, 1, NULL) OVER (ORDER BY date_col),
  LEAD(col, 1, NULL) OVER (ORDER BY date_col)
```

Note: Doris requires an explicit default value (third parameter).

### RANK / DENSE_RANK

Assign rankings.

**SparkSQL:**
```sql
SELECT 
  RANK() OVER (ORDER BY score),
  DENSE_RANK() OVER (ORDER BY score)
```

**Doris SQL:**
```sql
SELECT 
  RANK() OVER (ORDER BY score),
  DENSE_RANK() OVER (ORDER BY score)
```

## Conditional Expressions

### CASE WHEN

**SparkSQL:**
```sql
SELECT 
  CASE 
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE result3
  END
```

**Doris SQL:**
```sql
SELECT 
  CASE 
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE result3
  END
```

### IF

**SparkSQL:**
```sql
SELECT IF(condition, true_value, false_value)
```

**Doris SQL:**
```sql
SELECT IF(condition, true_value, false_value)
```

### COALESCE

**SparkSQL:**
```sql
SELECT COALESCE(col1, col2, default_value)
```

**Doris SQL:**
```sql
SELECT COALESCE(col1, col2, default_value)
```

## Common Patterns

### Basic SELECT with WHERE

**SparkSQL:**
```sql
SELECT 
  user_id,
  user_name,
  created_date
FROM users
WHERE status = 'active'
  AND created_date >= '2024-01-01'
ORDER BY created_date DESC
LIMIT 100
```

**Doris SQL:**
```sql
SELECT 
  user_id,
  user_name,
  created_date
FROM users
WHERE status = 'active'
  AND created_date >= '2024-01-01'
ORDER BY created_date DESC
LIMIT 100
```

### JOIN Operations

**SparkSQL:**
```sql
SELECT 
  o.order_id,
  u.user_name,
  o.total_amount
FROM orders o
INNER JOIN users u ON o.user_id = u.user_id
WHERE o.order_date >= '2024-01-01'
```

**Doris SQL:**
```sql
SELECT 
  o.order_id,
  u.user_name,
  o.total_amount
FROM orders AS o
INNER JOIN users AS u ON o.user_id = u.user_id
WHERE o.order_date >= '2024-01-01'
```

### GROUP BY with Aggregations

**SparkSQL:**
```sql
SELECT 
  category,
  COUNT(*) AS item_count,
  SUM(amount) AS total_amount,
  AVG(amount) AS avg_amount
FROM sales
GROUP BY category
HAVING SUM(amount) > 10000
ORDER BY total_amount DESC
```

**Doris SQL:**
```sql
SELECT 
  category,
  COUNT(*) AS item_count,
  SUM(amount) AS total_amount,
  AVG(amount) AS avg_amount
FROM sales
GROUP BY category
HAVING SUM(amount) > 10000
ORDER BY total_amount DESC
```

### Subqueries and CTEs

**SparkSQL:**
```sql
WITH monthly_sales AS (
  SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    SUM(amount) AS total
  FROM sales
  GROUP BY DATE_TRUNC('month', sale_date)
)
SELECT 
  month,
  total,
  LAG(total, 1) OVER (ORDER BY month) AS prev_month_total
FROM monthly_sales
ORDER BY month
```

**Doris SQL:**
```sql
WITH monthly_sales AS (
  SELECT 
    DATE_TRUNC(sale_date, 'MONTH') AS `month`,
    SUM(amount) AS total
  FROM sales
  GROUP BY DATE_TRUNC(sale_date, 'MONTH')
)
SELECT 
  `month`,
  total,
  LAG(total, 1, NULL) OVER (ORDER BY `month`) AS prev_month_total
FROM monthly_sales
ORDER BY `month`
```

## Known Limitations

### 1. Complex Data Types
While both SparkSQL and Doris support arrays, Doris has more limited support for nested structures (MAP, STRUCT). SQLGlot will attempt to convert these, but manual review may be needed.

### 2. User-Defined Functions (UDFs)
Custom UDFs cannot be automatically transpiled. You will need to reimplement UDFs in Doris or find equivalent built-in functions.

### 3. Some Specialized Functions
Certain SparkSQL-specific functions may not have direct Doris equivalents. SQLGlot will preserve the function name, which may result in runtime errors in Doris. Examples include:
- Some DataFrame API-specific functions
- Certain Spark ML functions
- Specialized Hive-compatibility functions

### 4. Performance Optimizations
While the SQL syntax will be converted, performance characteristics may differ between Spark and Doris. Query plans and optimizations should be reviewed in Doris.

## Best Practices

### 1. Test After Conversion
Always test converted queries in a Doris environment to ensure they work as expected:

```python
import sqlglot

spark_queries = [
    "SELECT COUNT(*) FROM users",
    "SELECT MAX(created_date) FROM orders",
    # ... more queries
]

for query in spark_queries:
    doris_query = sqlglot.transpile(query, read='spark', write='doris')[0]
    print(f"Original: {query}")
    print(f"Converted: {doris_query}")
    print()
```

### 2. Handle Batch Conversions
For large-scale migrations, process queries in batches and log any errors:

```python
import sqlglot

def convert_queries(spark_queries):
    converted = []
    errors = []
    
    for i, query in enumerate(spark_queries):
        try:
            doris_query = sqlglot.transpile(query, read='spark', write='doris')[0]
            converted.append({
                'original': query,
                'converted': doris_query,
                'index': i
            })
        except Exception as e:
            errors.append({
                'query': query,
                'error': str(e),
                'index': i
            })
    
    return converted, errors
```

### 3. Validate Data Types
Ensure that data type conversions preserve the intended precision and scale, especially for DECIMAL types:

```python
import sqlglot

# Check type mappings
query = "SELECT CAST(col AS DECIMAL(10, 2))"
result = sqlglot.transpile(query, read='spark', write='doris')[0]
print(result)
```

### 4. Review Window Functions
Pay special attention to window function conversions, especially LAG and LEAD which require default values in Doris.

### 5. Use Pretty Printing
For better readability, use the `pretty=True` option:

```python
import sqlglot

complex_query = "SELECT a, b, c FROM table1 JOIN table2 ON table1.id = table2.id WHERE status = 'active'"
pretty_doris = sqlglot.transpile(complex_query, read='spark', write='doris', pretty=True)[0]
print(pretty_doris)
```

### 6. Preserve Comments
SQLGlot preserves SQL comments, which can be helpful for documentation:

```python
import sqlglot

query_with_comments = """
-- Get active users
SELECT user_id, user_name
FROM users
WHERE status = 'active' -- Only active users
"""

result = sqlglot.transpile(query_with_comments, read='spark', write='doris')[0]
print(result)
```

### 7. Incremental Migration
Consider migrating queries incrementally:
1. Start with simple SELECT queries
2. Move to queries with JOINs
3. Then handle complex aggregations and window functions
4. Finally, tackle queries with advanced features

### 8. Documentation
Document any manual changes required after automatic conversion, especially for:
- UDFs that need reimplementation
- Performance tuning adjustments
- Queries that couldn't be automatically converted

## Conclusion

SQLGlot provides a powerful and reliable way to migrate SparkSQL queries to Doris SQL. By understanding the differences between these dialects and following the best practices outlined in this guide, you can streamline your migration process and reduce errors.

For more information:
- [SQLGlot Documentation](https://sqlglot.com/)
- [Doris SQL Reference](https://doris.apache.org/docs/sql-manual/sql-reference/)
- [Spark SQL Reference](https://spark.apache.org/docs/latest/sql-ref.html)
