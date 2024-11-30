It doesn't matter how good a software developer you are. At some point, you and I have written slow queries. Sometimes, you've noticed them being slow immediately, while other times, they were fast enough until your data grew a lot. That's when you had to reach for MySQL's EXPLAIN command to understand what was happening and how to fix it. Because trying to fix a problem without knowing the root cause would be insane. We've never done that before. Riiiight?

But the EXPLAIN output is so complicated and incomprehensible for ordinary mortals. You could try to get an understandable explanation of the EXPLAIN output by your favorite AI. Yet they all have a common problem with database stuff and produce bad output, although they are improving with insane speeds in many areas: They are trained on available content and that is the problem: The existing content they use to train an AI is bad! The MySQL documentation is overly technical and describes everything in a way only understandable to database professionals. Likewise, all blog articles just replicate the technical language of the MySQL documentation without translating it into more developer-friendly descriptions.

## The Mechanics
You can prepend the EXPLAIN clause to any SELECT, DELETE, INSERT, REPLACE and UPDATE query and run it. Such a query is not really executed - contrary to common belief! It is an diagnostics command with the primary function to provide insights into how MySQL would execute your (slow) query. So MySQL stops its internal workflow right before accessing your indexes and tables to report information like:

* The tables and partitions accessed by your query.
* Every index MySQL could use to load matching rows.
* The index that was decided to be used - or none.
* An estimated number of rows that will be read from the table and how many are left after evaluating all WHERE conditions.
* Much additional information that can be useful for you or is just detailed information you will never need.

```sql
EXPLAIN SELECT * FROM table_name WHERE conditions
```

## Constraints and Limitations
The EXPLAIN command exposes a marvelous treasure of information if you know how to read it. But it is also quite limited in how it does things and what it can do:

* All information about loaded and filtered rows is just an estimate, not the actual count. However, these estimates are also the foundation of MySQL's decision-making process regarding which index to use: There is probably something wrong with MySQL when you see very far-off estimates and no index usage.
* The estimates do not reflect constraints set on a query to limit the number of rows (e.g. LIMIT 1). MySQL will lie in your face, saying that thousands of rows may be loaded and returned, even though you know this can't be true.
* The output does not include advanced features like stored procedures (custom functions you created) or triggers. There will be no indication in the output that they are executed or the queries they run.
* Many of MySQL's terms are misleading and mean different things. For example, Using index indicates that all data for that row could be loaded from the index, and the table was not used. It's easy to panic when that term is not shown for a table, as it sounds like no index was used.

## The Output
Lets run a simple query and go through the different information of the EXPLAIN output.

```sql
EXPLAIN SELECT first_name, last_name
FROM actor
WHERE actor_id IN (
    SELECT actor_id
    FROM film_actor
    WHERE film_id IN (
        SELECT film_id FROM film WHERE title like '%alone%'
    )
);
```


|id|	select_type |	table       |	partitions|	type  |	possible_keys         |	key                |key_len|	ref|	rows|	filtered|	Extra|
|--|--------------|-------------|-----------|-------|-----------------------|--------------------|-------|-----|------|---------|------|
|1|	  SIMPLE      |	actor       |		        | ALL   |	PRIMARY               |		                 |	     |     | 200  |	100.00  |      |	
|1|	  SIMPLE      |	<subquery2> |		        | eq_ref|	<auto_distinct_key>   |	<auto_distinct_key>|	2    |	sakila.actor.actor_id|	1|	100.00|	 |
|2|	  MATERIALIZED|	film        |		        | index |	PRIMARY               |	idx_title          | 514   |		 |1000|	11.11|	Using where; Using index|
|2|	  MATERIALIZED|	film_actor  |		        | ref   |	PRIMARY,idx_fk_film_id|	idx_fk_film_id     |	2	   |sakila.film.film_id|	5|	100.00|	Using index|
	
### id
The id column is a sequential counter of the SELECT used within your query. It starts with one and is incremented for every further SELECT of a subquery. The idea is to link, e.g., a table accessed many times by different subqueries in the EXPLAIN output with the exact position in your query.

However, the value is only correct when compared with the transformed query MySQL uses internally. The one used in the example may look quite similar to this:

```sql
/* select#1 */
EXPLAIN SELECT actor.first_name AS first_name, actor.last_name AS last_name
FROM actor
JOIN (
    /* select#2 */
    SELECT DISTINCT actor_id
    FROM film_actor
    JOIN film ON (film.film_id = film_actor.film_id)
    WHERE film.title LIKE '%alone%'
) AS `< subquery2>` USING(actor_id);
```

So the id value is useless for the intended purpose as the query you've written differs from the one executed. But it also reveals another critical information about the order in which tables are accessed:

* Two consecutive rows with the same number are joined together. So you can see that film is joined to film_actor instead of executing subqueries - as it is more efficient.
* The ordering of the rows also indicates the ordering in which tables have been executed. The concept is quite complicated and you can dig into all the details at the algorithm section of the pt-visual-explain tool.

### select_type
The select_type is always SIMPLE if your query is only using e.g. a few joins and conditions and no subqueries or unions. However, you start seeing different values when using these advanced features. The outermost part of your query will always have the PRIMARY value, and the subqueries will have different values based on how they are executed.

```sql
/* PRIMARY */
SELECT *
FROM actor
JOIN film_actor USING(actor_id)
WHERE film_id IN(
    /* SUBQUERY */
    SELECT film_id FROM film WHERE title LIKE '%DINOSAUR%'
);
```

For subqueries, you can see these select types:

* SUBQUERY: A subquery is executed once and its result is cached. This cached result can be reused when that exact subquery is repeated multiple times in the query instead of rerunning the subquery.

```sql
SELECT * FROM tbl1 WHERE value IN (
    /* SUBQUERY */
    SELECT value FROM tbl2
)
```

* DEPENDENT SUBQUERY: The subquery is executed for every row of the outer query because some conditions within that subquery depend on table columns outside of that subquery.

```sql
SELECT *
FROM tbl1
WHERE EXISTS(
    /* DEPENDENT SUBQUERY */
    SELECT * FROM tbl2 WHERE tbl2.value = tbl1.value
)
```

* UNCACHEABLE SUBQUERY: The definition is the same as a dependent subquery, but the behavior differs: For a dependent subquery, the result of the subquery can be cached and reused for every distinct value of tbl1.value. But with an uncacheable subquery, it really has to be executed for every row.

```sql
SELECT *
FROM tbl1
WHERE EXISTS(
    /* UNCACHEABLE SUBQUERY */
    SELECT * FROM tbl2 WHERE tbl2.value = tbl1.value
)
```

* DERIVED: A subquery is used as a table and the results are stored in a temporary table. Other rows access this temporary table as table <derived{id}>.

```sql
SELECT * FROM (
  /* DERIVED */
  SELECT * FROM tbl1 WHERE value = 42
) AS tbl2
```

* DEPENDENT DERIVED: A subquery is used as a lateral join and will be executed for every row of the outer query. Unlike a DERIVED subquery, this subquery can reference values from other tables.

```sql
SELECT *
FROM tbl1
LEFT JOIN LATERAL (
    /* DEPENDENT DERIVED */
    SELECT * FROM tbl2 WHERE value = tbl1.value LIMIT 3
) AS lat ON true;
```

* MATERIALIZED: The subquery used within a condition is materialized to a temporary table. Other rows access this temporary table as table <materialized{id}>.

```sql
SELECT *
FROM tbl1
WHERE id IN (
    /* MATERIALIZED */
    SELECT id FROM tbl2 WHERE value = 42
)
```

When using unions, you will encounter these select types:

* UNION: Indicates a subquery as part of a UNION query that will be merged into the result.

```sql
SELECT * FROM tbl1 WHERE value = 42
UNION ALL
SELECT * FROM tbl2 WHERE value = 42
```

* UNION RESULT: A UNION clause (instead of UNION ALL) instructs MySQL to remove all duplicate rows when merging the queries. In this case, the UNION RESULT type indicates this duplicate removal step with a temporary table.

```sql
SELECT * FROM tbl1 WHERE value = 42
UNION
SELECT * FROM tbl2 WHERE value = 42
```

### table
The table column is self-explanatory: It contains the name of the table being accessed or the table alias used in your query. But there are also some special values:

* <union{id-1},{id-2},...{id-n}>: A temporary table that is the result of merging and deduplicating the specified subqueries by the id column.
* <derived{id}>: A temporary table that stores the result of the specified DERIVED subquery.
* <subquery{id}>: A temporary table that stores the result of the specified SUBQUERY subquery.

### partitions
This column is only filled when a table is partitioned into smaller chunks. It lists all the partitions that will be accessed to find the rows matching the query's criteria.

### type
The type column is essential as it describes how a table was accessed. You'll find information about whether an index is used and which way it is used. Most of these values are just technical information exported in the EXPLAIN output and are not interesting. However, some of them hint at inefficient index usage:

* ALL The entire table was loaded and filtered based on your conditions. This is a term you never want to see because it means that no index was used! Only for tiny tables can loading all rows be faster than using an index.
* const A primary or unique index is checked against a single value. This is excellent because the result can be one row at most, and MySQL can apply some internal optimizations.
* eq_ref A join uses a primary or unique key to load all matching rows from the table. It is the fastest join index type as there can be only one or no matching row.
* fulltext A full-text index has been used for the table. It is important to know that with MySQL's full-text search any existing index is ignored and the full-text index is always used. So you can't limit the full-text search to some specific indexed conditions. They are always evaluated on the full-text search result for the entire table.
* index An index can be used to get the requested rows. However, the entire index is scanned instead of a small portion as usual. This is obviously slower than a more focused index access. You can optimize this.
* index_merge Multiple indexes can be used since MySQL 8.0, and that's what you see. The key column should list multiple indexes that have been merged together. The performance is obviously better than not using an index at all, but you should create a better-fitting single index.
* index_subquery This is a specially optimized version of the ref access type when the index column is checked again an abc IN(SELECT ...) subquery.
* range A range of the index is scanned from a starting point to an ending point. This is expected if your condition contains any range condition (<>, >, >=, <, <=, IS NULL, <=>, BETWEEN, LIKE or IN()) but unexpected and probably a good thing to look into when used with equality checks.
* ref All rows identified by the index are loaded. This is the access type you see most used and there is no reason to change anything.
* ref_or_null This access type behaves the same way as the ref type, but MySQL is performing another search for NULL values. ref_or_null is used instead of ref for nullable columns or sometimes for subqueries.
* unique_subquery This is a specially optimized version of the eq_ref access type when the index column is checked again an abc IN(SELECT ...) subquery.
* system A special system table with just one row has been used. This is very seldom seen except when searching within MySQL's internal exposed metrics and configuration tables.

### possible_keys
One of the most important ones: MySQL tells you which indexes it could use to get the rows from the table. Is the index you expected to use missing from this list? The most common reason is that you transformed an indexed column and believe the index will still be used - it won't!

```sql
SELECT * FROM invoices WHERE YEAR(created_at) = 2024
```
You have to create a specialized index for this transformation:

```sql
CREATE INDEX year ON invoices ((YEAR(created_at)))
```

### key
The key column tells you which index is used from the possible ones to load the rows - or NULL if none is used. A common thing also listed is automatically generated keys (e.g. <auto_distinct_key>) when MySQL stores a subquery's result in a temporary table and wants to access if fast in further execution steps. And rarely you see multiple keys listed when MySQL decides that one would not be efficient enough (see index_merge access type).

### key_len
If you really want to dig deep down into your EXPLAIN output, you can analyze the key_len column: It lists the bytes used from the indexes column definition. For multi-column indexes, you could infer whether all columns in the index had been used to find the rows or the subset of columns used.

### ref
Theoretically, the ref column should show the columns or values compared to the indexed columns. But you will most often see const or similar labels there. They are not of any interest to you.

### rows
MySQL shares its estimation of rows that will be loaded from the table with the rows column. Remember that this is only an estimate and not the actual value! However, this estimate is also the foundation of MySQL's decision-making process regarding which index to use. With estimates far off the actual value, MySQL may choose the wrong index or even use none.

The displayed number is also not the real estimate. Its a per-step estimate: For e.g. a join the number of loaded rows for each iteration of the loop is reported. So, with multiple joins or subqueries, the value may need to be multiplicated many times.

### filtered
The filtered column indicates the percentage of rows loaded from table (rows column) left after executing all filter conditions. So 1900 rows remain when 2000 rows have been loaded and the filter value is 95.00. This value should be near the optimum of 100.00. A more fitting index should be created whenever you see a low filter value.

### extra
The Extra column has many different values (35+) that export more low-level information about the query processing. But these values will most likely not be of any interest to you as they only tell which internal optimizations have been applied—they are meant for database professionals.