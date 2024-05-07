---
layout: post
tags: ruby-on-rails postgresql
---

# Install

MacOS 
```
brew install postgresql

createdb `whoami` # create database dule
psql # it will connect to dule db

# if you want to give access to other users on the system
psql -d postgres -c "CREATE USER ljubicao WITH SUPERUSER;"
```

# Create user

On Mac create `postgres` user
```
createuser -s postgres
# login to database as user
psql --user postgres
```

On ubuntu
```
sudo -u postgres psql
create user myuser with encrypted password 'mypass';

# this is temporary default database, used when you just run `psql`
create database myuser;
grant all privileges on database myuser to myuser;
```
to allow local user to createdb and seed fixtures you need to
```
ALTER USER myuser WITH SUPERUSER;
``UPERUSER;'"
# or
sudo -u postgres psql -d postgres -c "CREATE USER igor WITH SUPERUSER;"
```

List all users
```
\du
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB                          | {}
```

```
# allow localhost connection using password (instead of socket with same
username)
sudo vi /etc/postgresql/10/main/pg_hba.conf
# change all 'peer' and 'md5' to 'trust': 
sudo service postgres reload

# test with bash
psql

# allow remote connections; find file with: SHOW config_file;
# on macOD find config folder with ps aux|grep postgres
sudo vi /etc/postgresql/10/main/postgresql.conf
# add: listen_addresses = '*'
# test with
nmap localhost
# 5432/tcp open  postgresql

# check with netstat
netstat -an | grep 5432
# tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN

# check from other computer
nmap 192.168.1.3
# if there is no port 5432 than check firewall on ubuntu or eventual router
```

test with
```
psql -U myuser -d mydb -h 192.168.1.3
```

```
# show all databases
\l
# or you can list using SELECT
SELECT datname FROM pg_database;

# list all tables
\dt

# Show table definition, describe columns
\d users

# show sequences
\ds

# find psql version (note there is a version info as out of first line when you
# enter the console, if server and client are different, both will be shown)
SELECT version();
```


# Introduction

Learning database is funny and easy, but when you want to do something more
complicated than `SELECT * FROM products` than you need to read documentation.

If you want something more specific data from `products` table you need to use
[7.2. Table
Expressions](http://www.postgresql.org/docs/9.4/static/queries-table-expressions.html)
that is:

* `FROM products p INNER JOIN sales s ON p.id = s.product_id` where *p* and *s*
  are aliases. In following sql you need to use those aliases `p` instead
  `products`
* You can create column aliases for columns `SELECT *, t.lat AS latitude`
  but you can not use in `where` clause since where clause is logically before
  select clause (and before alias is created). If you need to use, you need to
  write that statement twice, or use sql wrapper wich will contain only that
  `where`, this example is using concat:

  ~~~
  # this does not work
  SELECT neededfield, CONCAT(firstname, ' ', lastname) as firstlast
  FROM users
  WHERE firstlast = "Bob Michael Jones"

  # contact needs to be written twice: in SELECT and WHERE
  SELECT neededfield, CONCAT(firstname, ' ', lastname) as firstlast
  FROM users
  WHERE CONCAT(firstname, ' ', lastname) = "Bob Michael Jones"

  # or select wrapped
  SELECT * FROM (
    SELECT neededfield, CONCAT(firstname, ' ', lastname) as firstlast
    FROM users) base
  WHERE firstLast = "Bob Michael Jones"
  ~~~

  ORDER or parsing SQL is
* 'FROM' -> 'WHERE' -> GROUP BY -> HAVING -> SELECT -> ORDER BY so you can not
  use select aliases in where or group by or having. But `HAVING` can accept
  aggregate functions (that is HAVING purpose).
* after processing *FROM* clause, this new virtual table is checked against
  search conditions *WHERE*, for example: `WHERE s.date > CURRENT_DATE -
  INTERVAL '4 weeks'` This filtering is done *before* processing with *GROUP BY*
  and *HAVING*. So you can not use alias in WHERE `SELECT id as as_id FROM table
  WHERE as_id = 1`. You can only use columns that are available to select
  `SELECT id AS as_id FROM table HAVING id = 1`.
* `GROUP BY product_id, p.name, p.price, p.cost` and `HAVING sum(p.price *
  s.units) > 5000;`. If you group by primary key column than you can *SELECT*
  any column, but if you group by non primary than you need some [9.20.
  Aggregate
  Functions](http://www.postgresql.org/docs/9.4/static/functions-aggregate.html)
  since those columns can be merged.
  * you can group by hour with `SELECT date_trunc('hour', created_at),  (which
      can be count(*)
  FROM users GROUP BY date_trunc('hour', created_at) ORDER BY date_trunc('hour',
  created_at);`. You can reference columns by select position `SELECT
  date_trunc('hour', created_at), count(*) FROM users GROUP BY 1 ORDER BY 1;`
  Although you can order by expression of columns `ORDER BY col1 + col2`, you
  can not use order by with calculated col and expression `ORDER BY sum + 1`
  https://www.postgresql.org/docs/8.3/queries-order.html
* you can group by several columns and select count > 2 and get only first
item.id `a=Item.reorder("").select('max("id") as max_id').group([:url, :title,
:feed_id]).having('count("id") > 1')`

I found very interesting paragraph in [tutorial for aggregate
functions](http://www.postgresql.org/docs/devel/static/tutorial-agg.html).

>  It is important to understand the interaction between aggregates and SQL's
>  WHERE and HAVING clauses. The fundamental difference between WHERE and HAVING
>  is this: WHERE selects input rows before groups and aggregates are computed
>  (thus, it controls which rows go into the aggregate computation), whereas
>  HAVING selects group rows after groups and aggregates are computed. Thus, the
>  WHERE clause must not contain aggregate functions; it makes no sense to try
>  to use an aggregate to determine which rows will be inputs to the aggregates.
>  On the other hand, the HAVING clause always contains aggregate functions.
>  (Strictly speaking, you are allowed to write a HAVING clause that doesn't use
>  aggregates, but it's seldom useful. The same condition could be used more
>  efficiently at the WHERE stage.)
>
> ... This is more efficient than adding the restriction to HAVING, because we
> avoid doing the grouping and aggregate calculations for all rows that fail the
> WHERE check.

Next step is to learn something about [9.8. Data Type Formatting
Functions](http://www.postgresql.org/docs/9.4/static/functions-formatting.html)
which you can use in grouping.

Some tips:

* if you want to avoid *Division by zero* you can use `CASE count(column_name) =
  0 WHEN 0 THEN 1 ELSE count(column_name) END`.  Another way is to use
  `COALESCE(NULLIF(column_name,0), 1)` .
  `NULLIF` is used to show nil, as division by nil is nil.
  `something/NULLIF(column_name,0)` If the value of column_name is 0 - result of
  entire expression will be NULL.
  Defaults for `ORDER BY ASC` is `NULLS LAST` and for desc `NULL FIRST` so you
  maybe want to reverse that.
* `FROM WHERE field IN ('a','b')` is the same as `field = 'a' OR field =
  'b'`
* you can use IN, ANY, ALL
[functions-comparisons](https://www.postgresql.org/docs/current/static/functions-comparisons.html#AEN18509)
* substring match `WHERE col LIKE '%substr%'` or in rails `.where("mobile ILIKE
  ?", '%substr%')`
* unique index regardless of order (two way combination or columns) can be
  enforced with expressions to build index
  https://stackoverflow.com/questions/30036837/postgresql-enforce-unique-two-way-combination-of-columns
  ```
    # This index does not cover two direction (oposite case)
    # add_index :interests, [:from_member_profile_id, :to_member_profile_id], unique: true, name: 'uniq_index_on_interests'
    # https://stackoverflow.com/a/30037484/287166
    execute <<-SQL
      create unique index uniq_index_on_interests on interests(greatest(from_member_profile_id,to_member_profile_id), least(from_member_profile_id,to_member_profile_id));
    SQL
  ```

Use gem to extent ActiveRecord functionality
https://github.com/georgekaraszi/ActiveRecordExtended

# Tutorial links

<https://www.toptal.com/postgresql>
Postgresql is better than NoSql because of:

  * transactionality at system level
  * aggregation functions
  * joins across different tables

NoSql has advantage of scaling (horizontal, a lot of new fields) and that you
can store semistructured data (social networks)

For full text search in Postgresql, you can use `@@` match operator `SELECT *
FROM posts WHERE tsvector(title || ‘ ‘ || content) @@
plainto_tsquery(‘query-string’);`

High availability is implemented with redudant servers *hot standby* that copy
data and accept only read only queries (only master can write data), and *warm
standby* servers that only follow the changes made to primary server (does not
accept queries) and wait to be promoted to primary server in case of master
failure.

<https://www.toptal.com/sql/interview-questions>

* `UNION` (`UNION ALL`) merges two structurally compatible tables into single
  table (with duplication)
* `INNER JOIN` (`LEFT OUTER` `FULL` `CROSS`) returns rows that match both
  tables (all rows from left table and matched rows from right, either left or
  right ie union, cartesian product each by each row)
* `SELECT count(*) WHERE cid <> '123'` will not count when cid is NULL
* `is null` is proper way to check if `null = null` or `cid = null`
* `WHERE cid NOT IN (SELECT wcid FROM t WHERE wcid IS NOT null)` we need to
  check if null is in table since `NOT IN` will return empty set

* Online exercises pgexercises. com

# Time Date

* there are [4
  types](http://www.postgresql.org/docs/9.1/static/datatype-datetime.html):
  *timestamp, date, time* and *interval*. When you define value use single
  quotes: `DATE '2016-05-22'`. For intervals you need to set unit `INTERVAL '1
  WEEK'`
* for all types there is `CURRENT_DATE, CURRENT_TIME`
* [functions](http://www.postgresql.org/docs/9.1/static/functions-datetime.html)
  allows you to:
  * create next day `DATE '2001-02-01' + INTERVAL '1 DAY'`
  * extract weekday `EXTRACT(DOW FROM TIMESTAMP '2016')`
  * check overlaps `(start1, end1) OVERLAPS (start2, end2)`
* typecast can be done: `INTERVAL '1 WEEK'` or `'1 WEEK'::INTERVAL`
  * jsonb is much reliable type cast when since json compares as strings
    [link](http://stackoverflow.com/questions/32843213/operator-does-not-exist-json-json)
    you can select json with `SELECT name->'en' from users`
    for full json search you can use type case
    ```
    where data::text ilike '%my_str%'
    ```
* to use columns (like `$1` or name `a.b`) you can use
  * string concatenation  `(a.b || ' MINUTES')::INTERVAL`
  * multiplication `a.b * INTERVAL '1 MINUTES'` (much nicer)
* you can find records for specific month and year with

  ~~~
  scope :posts_in_month_and_year, ->(month, year) {
    where('MONTH(posts.created_at) = ? AND YEAR(posts.created_at) = ?', month, year)
  ~~~

* another grouping by day can be `SELECT COUNT(*), to_char(date, 'YYY-MM-DD') as
'day' FROM orders GROUP BY 'day' ORDER BY 'day'`

Practical example in Ruby on Rails
============

Complex queries can be written inside *database view* but that is not easy to
maintain since you need migration file for each change.  Also *database view* is
created in migration, so we don't have it with `rake db:setup`.  You can get it
with `rake db:migrate:reset` but this sometimes doesn't work if we used
*ActiveRecord* in migration files and we later add some validations
which (of-course) are not satisfied in the past (and this can be solved with
`class Product<ActiveRecord::Base;end` in migration files).

It's much easier to write long SQL query and use [F.15.
hstore](http://www.postgresql.org/docs/devel/static/hstore.html) functions to
simplify representations. You need to install hstore extension
[link](https://gist.github.com/terryjray/3296171) for test and develop db:

~~~
sudo psql -d myapp_development -U myuser
CREATE EXTENSION hstore;
#or in one command: 
sudo su postgres -c "psql myapp_test -c 'CREATE EXTENSION hstore;'"
~~~

Sometime you can run `rake db:migrate:reset` to rerun all migration and check if *db/schema.rb* is in sync.

To detect N+1 queries use gem *bullet*, and to find most expensive queries use heroku postgres analytics.
To measure production queries, you can download database from heroku to `a.dump` and restore to development database

~~~
sudo su postgres
pg_restore -d myapp_development --clean --no-acl --no-owner -h localhost a.dump
~~~

and run benchmark

~~~
puts Benchmark.measure { Job.active_per_domain_per_tier(domain,1) }
~~~

or you can connect to production database (be carefull).


Dump database using DATABASE_URL
```
# -Fc custom format -Fd directory format
pg_dump -Fc --dbname=$DATABASE_URL > db.dump
# -C for create
pg_restore -C -d db_name db.dump


# when using plain format
pg_dump --dbname=$DATABASE_URL > db.sql
createdb db_name
psql -d db_name -f db.dump
```

access database_url postgres://username:password@host/db_name
```
psql -U username -d db_name -h host
# type password
```

## Statistics graph

When you want to show some statistics during some time (time points in sql is `GENERATE_SERIES`), for example number of deactivated users:

![Time series of deactivated users]({{ site.baseurl }}/assets/deactivated_users.png)

What I have learned is that when you use *series*, never apply *WHERE* since
because it will destroy *series*.  Better is to filter in *ON* statement. Each
subsequent joins should be *LEFT OUTER JOIN* so *series* survive.

We need *COALESCE* because on some days we don't have records, so we put key
`none` ( number is 0)

~~~
# put this in controller
if params[:from_date].present?
  @from_date = Time.new( * params[:from_date].split('-')).to_date
else
  @from_date = (Time.now - 1.month).to_date
end
if params[:to_date].present?
  @to_date = Time.new( * params[:to_date].split('-')).to_date
else
  @to_date = (Time.now - 1.day).to_date
end

@group_by = params[:group_by] || 'day'
case @group_by
when 'day'
  interval_period = '1 day'
  interval = 'YY-MM-DD'
when 'week'
  interval_period = '1 week'
  interval = 'YY-WW'
when 'month'
  interval_period = '1 month'
  interval = 'YY-MM'
end

query = <<-SQL
SELECT
  hstore(array_agg(COALESCE(res.role::text,'none')),array_agg(res.number::text) ) AS name_mapping,
  res.datum as datum
FROM
(
    SELECT 
      users.role,
      count(users.id),
      dt.series
    FROM (
      SELECT GENERATE_SERIES( '#{@from_date}'::date, '#{@to_date}'::date, '#{interval_period}')::date
      AS series
    ) AS dt
    LEFT OUTER JOIN users
    ON TO_CHAR(users.updated_at,'#{interval}') = TO_CHAR(dt.series,'#{interval}') AND
      users.status = 0
    GROUP BY dt.series, users.role
) AS res(role, number, datum)
GROUP BY res.datum
ORDER BY res.datum
SQL
results = User.find_by_sql(query)
# results group by created_at and domain { 'scuddle' => 3, 'indeed' => 10 } for each date
@heading = "Deactivated users (group by updated_at)"
@labels = results.map {|k| k.datum}
roles = results.map {|k| k.name_mapping.keys}.flatten.uniq.reject {|r| r=='none'}
# TODO make those data arrays in SQL so we do not need to iterate again here
@datasets = add_colors roles.map { |r| { label: User.new(role: r).role_to_s, data: results.map { |k| k.name_mapping[r].to_i} } }
render 'stats_line_graph'
~~~

~~~
# in view file
<form>
  <input type="date" name="from_date" value="<%= @from_date %>">
  <input type="date" name="to_date" value="<%= @to_date %>">
  <select name="group_by">
    <option value="day" <%= 'selected="selected"' if @group_by == 'day' %>>Day</option>
    <option value="week" <%= 'selected="selected"' if @group_by == 'week' %>>Week</option>
    <option value="month" <%= 'selected="selected"' if @group_by == 'month' %>>Month</option>
  </select>
  <input type="submit">
</form>
<h4><%= @heading %></h4>

<% if ! @data.present? && ! @datasets.present? %>
  <p>
    No results
  </p>
<% end %>
<div id="legend" class="left"></div>
<div class="left m-l-10">
  <%= @message %>
</div>
<div class="clearfix"></div>
<canvas id="myChart" width="800" height="400"></canvas>
<div id="chartjs-tooltip"></div>

<script>
  <%# http://www.chartjs.org/docs/#line-chart-data-structure%>
  var data = {
    labels: <%=raw @labels.to_json %>,
    <% if @datasets.present? %>
      <%# for multiple lines %>
      datasets: <%=raw @datasets.to_json %>,
    <% else %>
      <%# for single line %>
      datasets: [
        {
          data: <%=raw @data.to_json %>,
        }
      ]
    <% end %>
  }
  <%# https://github.com/nnnick/Chart.js/blob/master/samples/line-customTooltips.html#L53 %>
  Chart.defaults.global.customTooltips = function(tooltip) {

      var tooltipEl = $('#chartjs-tooltip');

      if (!tooltip) {
          tooltipEl.css({
              opacity: 0
          });
          return;
      }

      tooltipEl.removeClass('above below');
      tooltipEl.addClass(tooltip.yAlign);

      var innerHtml = '';
      if (tooltip.labels)
      {
        for (var i = tooltip.labels.length - 1; i >= 0; i--) {
          innerHtml += [
            '<div class="chartjs-tooltip-section">',
            '   <span class="chartjs-tooltip-key" style="background-color:' + tooltip.legendColors[i].fill + '"></span>',
            '   <span class="chartjs-tooltip-value">' + tooltip.labels[i] + ' - ' + myNewChart.datasets[i].label + '</span>',
            '</div>'
          ].join('');
        }
      }
      else
      {
        innerHtml = [
          '<div class="chartjs-tooltip-section">',
          '   <span class="chartjs-tooltip-value">' + tooltip.text + '</span>',
          '</div>'
        ].join('');
      }

      tooltipEl.html(innerHtml);

      tooltipEl.css({
          opacity: 1,
          left: tooltip.chart.canvas.offsetLeft + tooltip.x + 'px',
          top: tooltip.chart.canvas.offsetTop + tooltip.y + 'px',
          fontFamily: tooltip.fontFamily,
          fontSize: tooltip.fontSize,
          fontStyle: tooltip.fontStyle,
      });
  };

  var ctx = document.getElementById("myChart").getContext("2d");
  var myNewChart = new Chart(ctx).Line(data);

  // legend
  var legendHolder = document.getElementById("legend");
  legendHolder.innerHTML = myNewChart.generateLegend();
</script>
~~~

* to find which table has the most rows you can use this
  ```
  SELECT schemaname,relname,n_live_tup
  FROM pg_stat_user_tables
  ORDER BY n_live_tup DESC;
  ```

# Other gems

I found interesting gems:

* interesting [no sql not required data analytics tool
INSIGHTS](https://github.com/mariusandra/insights)
* automatic find indexes that are needed for a database
  https://github.com/ankane/dexterhttps://github.com/ankane/dexter
* trigger a psql function to update counter cache https://evilmartians.com/chronicles/pulling-the-trigger-how-to-update-counter-caches-in-you-rails-app-without-active-record-callbacks

* adding uuid

```
# config/initializers/generators.rb
# https://pawelurbanek.com/uuid-order-rails
Rails.application.config.generators do |g|
  g.orm :active_record, primary_key_type: :uuid
end


# db/migrate/2020....add_pgcrypto_uuid_extension.rb
class AddPgcryptoUuidExtension < ActiveRecord::Migration[6.1]
  def change
    enable_extension 'pgcrypto'
  end
end

# create_table :users, id: :uuid
# t.belongs_to ... type: :uuid
# t.references ... type: :uuid
```

* select last associated nested record, query two tables but show only last from
  second table using a `DISTINCT ON` and `ORDER BY`
  ```
  DISTINCT ON (parent_col) *
  FROM parent_table
  JOIN nested_table
  ON parent_table.id = nested_table.parent_table_id
  ORDER BY parent_table.id, nested_table.created_at DESC
  ```
  https://stackoverflow.com/questions/1703495/postgresql-select-from-2-tables-but-only-the-latest-element-from-table-2

* to skip duplicates when you want to join you can use three ways:
  ```
  # use distinct
  select distinct books.id from books join cart_items on cart_items.book_id =
  books.id

  # use group by
  select count(*) from (select books.id from books join cart_items on cart_items.book_id = books.id group by books.id) as b
  select books.id from books join cart_items on cart_items.book_id = books.id group by books.id

  # use nested select subquery
  select count(books.id) from books where id in (select distinct book_id from cart_items)
  ```
* use test data `(VALUES (1, 'Duke'),(2, 'Mike')) users (id, name)`
  ```
  # change column type
  ALTER TABLE table_name ALTER COLUMN column_name TYPE new_type;
  # delete column
  ALTER TABLE table_name DROP COLUMN column_name;
  ```
  delete all rows
  ```
  DELETE FROM table_name;
  ```
* inspect time consuming quieries
  ```
  pg_stat_statements
  ```

* load from mysql dump to postgres
  ```
  pgloader mysql://root@localhost/rails_production postgresql://dule@localhost/rails_production
  ```
  but I receive error
  ```
   mysql: Failed to connect to mysql at "localhost" (port 3306) as user "root": Condition QMYND:MYSQL-UNSUPPORTED-AUTHENTICATION was signalled.
   ```
  https://github.com/dimitri/pgloader/issues/782#issuecomment-502323324
```
# sudo vi /opt/homebrew/etc/my.cnf
sudo vi /opt/homebrew/Cellar/mysql/8.0.31/.bottle/etc/my.cnf
brew services restart mysql
mysql -u root
alter user dule@localhost identified with mysql_native_password by 'dule';

# try to login with new password
mysql -u root -p
mysql -u root # this will not work

# to revert to passwordless login use
ALTER USER 'root'@'localhost' IDENTIFIED BY '';
mysql -u root
```

creating a dump using a docker https://gist.github.com/dougvj/49a803c27530161071e7c63cbd9aca1e
