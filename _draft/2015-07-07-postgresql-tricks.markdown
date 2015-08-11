[7.2. Table Expressions](http://www.postgresql.org/docs/9.4/static/queries-table-expressions.html)
[9.20. Aggregate Functions](http://www.postgresql.org/docs/9.4/static/functions-aggregate.html)
[9.8. Data Type Formatting Functions](http://www.postgresql.org/docs/9.4/static/functions-formatting.html)


Migration to create a database view https://rietta.com/blog/2013/11/28/rails-and-sql-views-for-a-report/
http://www.rigelgroupllc.com/blog/2014/09/14/working-with-complex-sql-statements/
http://stackoverflow.com/questions/13020251/aggregate-hstore-column-in-postresql


http://www.postgresql.org/docs/8.1/static/tutorial-agg.html
It is important to understand the interaction between aggregates and SQL's WHERE and HAVING clauses. The fundamental difference between WHERE and HAVING is this: WHERE selects input rows before groups and aggregates are computed (thus, it controls which rows go into the aggregate computation), whereas HAVING selects group rows after groups and aggregates are computed. Thus, the WHERE clause must not contain aggregate functions; it makes no sense to try to use an aggregate to determine which rows will be inputs to the aggregates. On the other hand, the HAVING clause always contains aggregate functions. (Strictly speaking, you are allowed to write a HAVING clause that doesn't use aggregates, but it's seldom useful. The same condition could be used more efficiently at the WHERE stage.)

In the previous example, we can apply the city name restriction in WHERE, since it needs no aggregate. This is more efficient than adding the restriction to HAVING, because we avoid doing the grouping and aggregate calculations for all rows that fail the WHERE check.

Starting with PSQL 9.1 it's sufficient to list the columns of the primary key in the group by clause


To optimize SQL use gem bullet
heroku postgres most expensive queries
