# What happens if columns are specified in GROUP BY, but no aggregate functions are specified?

Options:
* The query will return unique values for all columns from the table.
* The query will return an error.
* The query will return unique values for all columns listed in SELECT, and ignore the rest.
* The query will return unique values for the columns listed both in GROUP BY and SELECT.
* The query will return unique values for all columns listed in GROUP BY, and ignore the rest.

✅ Correct answer:
“The query will return unique values for the columns listed in GROUP BY, and ignore the rest.”
(Option 5)

# What is the reason why AVG(price) can return NULL, even if the column price exists?

Options:
* There is only one row in the table.
* All price values are equal to zero.
* All price values are NULL.
* The query has no GROUP BY.
* AVG does not work with the string type.

✅ Correct answer:
All price values are NULL.

The AVG() function in SQL ignores NULL values when calculating the average.
If all values in the column are NULL, then there are no non-null values to average, 
and the result is NULL.

# In which case can a LEFT JOIN return fewer rows than the table on the left?

Options:
* If SELECT * is not used.
* If the tables have a different number of columns.
* If there is a WHERE condition that filters out rows from the result.
* If we use USING instead of ON.
* If we join by a key that doesn’t exist in the right table.

✅ Correct answer:
If there is a WHERE condition that filters out rows from the result.
(Option 3)

By definition,
LEFT JOIN returns all rows from the left table,
and matching rows from the right one —
even if there are no matches (in which case right-side columns become NULL).

However, if you later apply a WHERE filter that excludes rows with NULL values (or any other condition),
you can end up with fewer rows than in the left table.

# What result will the following expression produce?

```sql
SELECT CASE 
         WHEN TRUE THEN 1 
         WHEN TRUE THEN 2 
         ELSE 3 
       END;
```

Options:
* NULL
* 3
* 1
* Error
* 2

A CASE expression in SQL is evaluated top to bottom,
and it stops at the first condition that is true.

So here’s what happens step by step:

The first condition — WHEN TRUE — is true ✅
→ So SQL immediately returns the value after THEN, i.e. 1.

The remaining conditions are not checked at all.

The result is 1.

# Which subquery will cause an error when executing the main query?

Options:

```sql
SELECT * FROM employees 
WHERE salary > ALL (SELECT salary FROM employees);
```

```sql
SELECT * FROM employees 
WHERE EXISTS (SELECT 1 FROM employees WHERE salary > 100000);
```

```sql
SELECT * FROM employees 
WHERE salary > (SELECT department, MAX(salary) 
                FROM employees GROUP BY department);
```

```sql
SELECT * FROM employees 
WHERE department IN (SELECT department FROM employees);
```

```sql
SELECT * FROM employees 
WHERE salary > (SELECT MAX(salary) FROM employees);
```

✅ Correct answer:
Option 3

This subquery:
```sql
SELECT department, MAX(salary)
FROM employees
GROUP BY department
```

returns two columns — department and MAX(salary).

However, the main query tries to compare salary > (subquery),
which expects the subquery to return a single scalar value (one column and one row).

Since it returns multiple columns, SQL will throw an error like:

“Subquery must return only one column.”

# How to update a materialized view so that it includes only the data from the last 3 months?

1. Update the materialized view using `UPDATE`.
2. Delete the view and recreate it with a filter `WHERE dt > current_date - interval '3 months'`
3. Execute `REFRESH MATERIALIZED VIEW` with a filter.
4. Apply `WHERE dt > now()` to the existing view.
5. Use `ALTER MATERIALIZED VIEW` to add a condition.

✅ Correct answer:
Option 2 – Delete the view and recreate it with a filter:
