# ANSI/T-SQL Style Guide

## Example - Use Tabs for indention! The code formats itself for text editors, etc.

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql
with interest as
(
	select
		email
		,timestamp_millis(property_beacon_interest) as expressed_interest_at
	from contact
	where property_beacon_interest is not null
), 
support_interest as
(
	select 
		c.email
		,c.created_at as expressed_interest_at
	from conversation as c
	join conversation_tag as ct
		on c.id = ct.conversation_id
	where ct.tag = 'beacon-interest'
), 
combined_interest as
(
	select *
	from interest
	union all
	select *
	from support_interest
),
final as
(
	select
		email
		,min(expressed_interest_at) as expressed_interest_at
	from combined_interest
	group by
		email
)
select *
from final
```
## Guidelines

### Code Refactoring

If someone took the time to refactor a poorly written (and poor performing) query, follow THEIR formatting. Otherwise, it looks like you were lazy, your work looks sloppy and YOU should feel bad for doing it.

### Use lowercase SQL

It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```sql
-- Good
select *
from users

-- Bad
SELECT *
FROM users

-- Bad
Select *
From users
```

### Single line vs multiple line queries

The only time you should place all of your SQL on a single line is when you're selecting one thing and there's no additional complexity in the query:

```sql
-- Good
select * from users

-- Good
select id from users

-- Good
select count(*) from users
```

Once you start adding more columns or more complexity, the query becomes easier to read if it's spread out on multiple lines:

```sql
-- Good
select
	id
	,email
	,created_at
from users

-- Good
select *
from users
where email = 'example@domain.com'

-- Good
select
	user_id
	,count(*) as total_charges
from charges
group by
	user_id

-- Bad
select id, email, created_at
from users

-- Bad
select id,
	email
from users
```

### Left align SQL keywords

Some IDEs have the ability to automatically format SQL so that the spaces after the SQL keywords are vertically aligned. This is cumbersome to do by hand (and in my opinion harder to read anyway) so I recommend just left aligning all of the keywords:

```sql
-- Good
select
	id
	,email
from users
where email like '%@gmail.com'

-- Bad
select id, email
  from users
 where email like '%@gmail.com'
```

### Use single quotes

Some SQL dialects like BigQuery support using double quotes, but for most dialects double quotes will wind up referring to column names. For that reason, single quotes are preferable:

```sql
-- Good
select *
from users
where email = 'example@domain.com'

-- Bad
select *
from users
where email = "example@domain.com"
```

### Use '!=' over '<>'

Simply because '!=' reads like "not equal" which is closer to how we'd say it out loud. However, T-SQL tends to favor '<>'

```sql
-- Good
select
	count(*) as paying_users_count
from users
where plan_name != 'free'
```

### Commas should be at the the beginning of lines

```sql
-- Good
select
	id
	,email
from users

-- Bad
select
	id,
	email
from users
```

### Indenting where conditions

When there's only one 'where' condition, leave it on the same line as 'where':

```sql
select
	email
from users
where id = 1234
```

When there are multiple:

```sql
select
	id
	,email
from users
where 1 = 1
and created_at >= '2019-03-01'
and vertical = 'work'
```

### Avoid spaces inside of parenthesis

```sql
-- Good
select *
from users
where id in (1, 2)

-- Bad
select *
from users
where id in ( 1, 2 )
```

### Break long lists of 'in' values into multiple indented lines

```sql
-- Good
select *
from users
where email in
(
	'user-1@example.com'
	,'user-2@example.com'
	,'user-3@example.com'
	,'user-4@example.com'
)
```

### Table names should be a plural snake_case of the noun

```sql
-- Good
select * from users
select * from visit_logs

-- Bad
select * from user
select * from visitLog
```

### Column names should be snake_case

```sql
-- Good
select
	id
	,email
	,timestamp_trunc(created_at, month) as signup_month
from users

-- Bad
select
	id
	,email
	,timestamp_trunc(created_at, month) as SignupMonth
from users
```

### Column name conventions

* Boolean fields should be prefixed with 'is_', 'has_', or 'does_'. For example, 'is_customer', 'has_unsubscribed', etc.
* Date-only fields should be suffixed with '_date'. For example, 'report_date'.
* Date+time fields should be suffixed with '_at'. For example, 'created_at', 'posted_at', etc.

### Column order conventions

Put the date column first, followed by primary key, then by all other columns. If the table has any system columns ('created_at', 'updated_at', 'is_deleted', etc.), put those last.

```sql
-- Good
select
	created_at
	,id
	,name
from users

-- Bad
select
	id,
	name,
	created_at,
from users
```

### Don't include 'inner' for inner joins - don't over-complicate things

```sql
-- Good
select
	u.email
	,sum(c.amount) as total_revenue
from users as u
join charges as c
	on u.id = c.user_id

-- Bad
select
	u.email,
	sum(c.amount) as total_revenue
from users as u
inner join charges as c
	on u.id = c.user_id
```

### Joins

There's no need to specify 'outer' for 'left join' or 'right join'. Also, don't use 'right join'. This means you have the wrong table order.

```sql
-- Good
select
	u.*
from users as u
left join charges as c
	on u.id = c.user_id

-- Bad
select
	u.*
from users as u
left outer join charges as c
	on u.id = c.user_id
```sql

### For join conditions, put the table that was referenced first immediately after the 'on'

By doing it this way it makes it easier to determine if your join is going to cause the results to fan out:

```sql
-- Good
select
	...
from users as u
left join charges as c
	on u.id = c.user_id
-- primary_key = foreign_key --> one-to-many --> fanout
  
select
	...
from charges as c
left join users as u
	on c.user_id = u.id
-- foreign_key = primary_key --> many-to-one --> no fanout

-- Bad
select
	...
from users as u
left join charges as c
	on c.user_id = u.id
```

### Don't put join conditions on the same line as the join

```sql
-- Good
select
    users.email
    ,sum(charges.amount) as total_revenue
from users as u
join charges as c
	on u.id = c.user_id
group by
	email

-- Bad
select
    users.email
    ,sum(charges.amount) as total_revenue
from users as u
join charges as c on users.id = charges.user_id
group by email
```

When you have multiple join conditions, place each one on their own indented line:

```sql
-- Good
select
	users.email
	,sum(charges.amount) as total_revenue
from users as u
join charges as c
	on u.id = c.user_id
	and refunded = false
group by
	email
```

### Alias your tables!

```sql
-- Good
select
	u.email
	,sum(c.amount) as total_revenue
from users as u
join charges as c
	on u.id = c.user_id

-- Bad
select
	users.email
	,sum(charges.amount) as total_revenue
from users
join charges
	on users.id = charges.user_id
```

### Include the alias when there is a join, but omit it otherwise

When there are no join involved, there's no ambiguity around which table the columns came from so you can leave the table name out:

```sql
-- Good
select
	id
	,name
from companies

-- Bad
select
	companies.id,
	,companies.name
from companies
```

But when there are joins involved, it's better to be explicit so it's clear where the columns originated:

```sql
-- Good
select
	u.email
	,sum(c.amount) as total_revenue
from users as u
join charges as c
	on u.id = c.user_id

-- Bad
select
	users.email
	,sum(charges.amount) as total_revenue
from users
join charges
	on users.id = charges.user_id
```

### Always rename aggregates and function-wrapped arguments

```sql
-- Good
select
	count(*) as total_users
from users

-- Bad
select
	count(*)
from users

-- Good
select
	timestamp_millis(property_beacon_interest) as expressed_interest_at
from contact
where property_beacon_interest is not null

-- Bad
select
	timestamp_millis(property_beacon_interest)
from contact
where property_beacon_interest is not null
```

### Be explicit in boolean conditions

```sql
-- Good
select *
from customers
where is_cancelled = 'true'

select *
from customers
where is_cancelled = 'false'

-- Bad
select *
from customers
where is_cancelled

select *
from customers
where not is_cancelled
```

### Use 'as' to alias column names

```sql
-- Good
select
	,id
	,email
	,timestamp_trunc(created_at, month) as signup_month
from users

-- Bad
select
    id
    ,email
    ,timestamp_trunc(created_at, month) signup_month
from users
```

### Group using column names or numbers, but not both

I prefer grouping by name, but grouping by numbers is [also fine](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/).

```sql
-- Good
select
	user_id
	,count(*) as total_charges
from charges
group by
	user_id

-- Good
select
	user_id
	,count(*) as total_charges
from charges
group by
	1

-- Bad
select
	timestamp_trunc(created_at, month) as signup_month
	,vertical
	,count(*) as users_count
from users
group by
	1
	,vertical
```

### Take advantage of lateral column aliasing when grouping by name 'ANSI-SQL ONLY'

```sql
-- Good
select
	timestamp_trunc(com_created_at, year) as signup_year
	,count(*) as total_companies
from companies
group by
	signup_year

-- Bad
select
	timestamp_trunc(com_created_at, year) as signup_year
	,count(*) as total_companies
from companies
group by
	timestamp_trunc(com_created_at, year)
```

### Grouping columns should go first

```sql
-- Good
select
	timestamp_trunc(com_created_at, year) as signup_year
	,count(*) as total_companies
from companies
group by
	signup_year

-- Bad
select
	count(*) as total_companies
	,timestamp_trunc(com_created_at, year) as signup_year
from helpscout_companies
group by
	signup_year
```

### Aligning case/when statements

```sql
-- Good
select
	case	when event_name = 'viewed_homepage' then 'Homepage'
			when event_name = 'viewed_editor' then 'Editor'
	else 'Other'
	end as page_name
from events

-- Good
select
	case	when event_name = 'viewed_homepage'
				then 'Homepage'
			when event_name = 'viewed_editor'
				then 'Editor'
	else 'Other'            
    end as page_name
from events

-- Good
select
	case	when event_name = 'viewed_homepage' then 'Homepage' else 'Other' end as page_name
from events
```

### Use CTEs

When using CTEs, pad the query with new lines. 

If you use any CTEs, always have a CTE named 'final' and 'select * from final' at the end. That way you can quickly inspect the output of other CTEs used in the query to debug the results.

Closing CTE parentheses should use the same indentation level as 'with' and the CTE names.

```sql
-- Good
;with ordered_details as
(
	select
		user_id
		,name
		,row_number() over (partition by user_id order by date_updated desc) as details_rank
	from billing_stored_details
),
final as
(
	select
		user_id
		,name
	from ordered_details
	where details_rank = 1
)
select *
from final
```

### Use meaningful CTE names

```sql
-- Good
with ordered_details as
(

-- Bad
with d1 as
(
```

### Sub-queries

```sql
-- Good
select
	subquery_name.*
from	(
			select
				row_number() over (partition by name order by date_field desc) as rn
			from table_b
		) as subquery_name
join	(
			select *
			from table_c
		) as join_c
	on subquery_name.field = join_c.field

-- Bad
select *
from
(
	select row_number() over (partition by name order by date_field desc) as rn from table_b
) subquery_name

select *
from ( select row_number() over (partition by name order by date_field desc) as rn from table_b ) as subquery_name
```

### Window functions

You can leave it all on its own line or break it up into multiple depending on its length:

```sql
-- Good
select
	user_id
	,name
	,row_number() over (partition by user_id order by date_updated desc) as details_rank
from billing_stored_details
```

### T-SQL Variables

```sql
declare	@variable_name date = getdate()
		,@another_variable_name	bigint = 1
		,@yet_another_variable_name varchar(25) = 'Super Bomb-Bomb';
```

### Logging - Don't reinvent the wheel and use an existing framework 'Example in T-SQL'

```sql
-- Logging inside a stored procedure
declare @processName nvarchar(100) = object_name(@@procid)		
		,@processStartTime	datetime = getdate()
		,@message	varchar(500)
		,@startTime	datetime
		,@rowCount	bigint
		,@sysLog	tinyint = 0
		,@tableLog	tinyint = 1

exec DBAdmin.dbo.TimeMessageLogger_sp @processName, 'Starting', null, null, @sysLog, @tableLog;

select	@message = 'Populating ' + quotename('table_name', char(39))
		,@startTime = getdate();
exec DBAdmin.dbo.TimeMessageLogger_sp @processName, @message, null, null, @sysLog, @tableLog;

select	@rowCount = @@rowcount
		,@message = 'Inserted ' + format(@@rowcount, 'N0') + ' records into ' + quotename('table_name', char(39));
exec DBAdmin.dbo.TimeMessageLogger_sp @processName, @message, @startTime, null, @sysLog, @tableLog;

select	@message = 'Process complete';
exec DBAdmin.dbo.TimeMessageLogger_sp @processName, @message, @processStartTime, null, @sysLog, @tableLog;
```
