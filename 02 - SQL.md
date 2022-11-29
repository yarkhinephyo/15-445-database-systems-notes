**DML**: How to retrieve and modify data
**DDL**: How to define a database
**DCL**: Security and access control in the database

**Bags**: SQL is based on bags (duplicates) not sets. For example, once you take an output of a query, primary key may not be included anymore.

**Query definitions**

1. AGGREGATES - Bag of tuples to a single value. Outputs from other columns are undefined.
2. GROUP BY - Project tuples into subsets and calculate aggregates against subsets. Non-aggregated values in SELECT output clause must appear in GROUP BY clause.
3. HAVING - Filter tuples based on aggregating computation.
4. ORDER BY - Order output tuples by values in one or more columns.

**Strings**

SQL standard uses || for concatenation but is not followed consistently.

String comparison is not implemented consistently. In MySQL, strings are casted into BINARY to filter for exact match.

**Knowing native types of data**

If you are using programmatic bindings such as ORM, there will be metadata on what types the data should be.

**Output redirection**

Store query results in another table. Inner SELECT must generate same columns as target table.

In PostgreSQL, there is TEMPORARY keyword to create temporary tables which will be dropped at the end of the session or transaction.

**Nested queries**

Difficult to optimize. Database has to figure out how to run the query as a JOIN instead of sequential queries.

To find existence of tuples in inner queries, use WHERE NOT EXISTS(\<subquery\>).

**Window functions**

Perform a "sliding" calculation across a set of tuples that are related. For example, use ROW_NUMBER() OVER(PARTITION BY \<col\>) to specify groups.

**Common table expressions (CTE)**

Write auxiliary statements for use in a larger query. Alternative way to write nested queries.

Bind output columns to names before the AS keyword.

With a RECURSIVE clause, CTE can refer to its own column names.

![](images/Pasted%20image%2020220901130851.png)