# Intro-to-SQL


# Introduction to SQL - Functions and GROUP BY

## SELECT statements without FROM clauses
It's actually possible to write a select statement without an accompanying FROM clause.  After the **`SELECT`** statement write a comma separated list of values. A single row table will be returned.

**Example 01**
```sql 
SELECT 
    'this is a string' as some_string, 
    5 as some_number,
    true as some_boolean;
```

### Aliasing
Notice the use of the `as` keyword. This changes the column name and is known as aliasing. Aliasing is heavily used when using functions in SQL to rename the output to something sensible.

## SELECT statements as a calculator
It is possible to use the SELECT statement as a calculator to do basic operations.

**Example 02**
```sql
SELECT 5 * (4 - 10) / 3 + 2;
```

## Functions
You can do more than just the basic arithmetic operations in SQL. Most SQL implementations come with a variety of functions that add lots of additional power to modify the output.

### Aggregating vs Non-Aggregating Functions
To better understand functions, you can make a distinction between those that perform an aggregation and those that do not. To make this distinction we need a clear definition of aggregations.

> Aggregation - Taking multiple values as inputs and returning a single value

This seems quite simple and it is. For instance, when we have a sequence of numbers and sum them together, an aggregation has been performed. If we count the number of items in our sequence, another aggregation as been performed. The five most common aggregation functions that all SQL implementations have are `SUM, MIN, MAX, COUNT, AVG`. data.world also supports `GROUP_CONCAT, STDEV, STD_SAMP, STD_POP, VARIANCE, VAR_POP, VAR_SAMP, CORRELATION`.

#### Non-aggregating functions
Non-aggregating functions return a value for each of the sequence of items passed to it. They do not summarize the sequence to a single result. For instance, taking the square root or rounding a number are examples of non-aggregating functions. 'Non-aggregating' functions are not a universal term. For instance, Oracle refers to them as [single-row functions][2].

#### Further sub-classification of Non-aggregating functions
Typically, these functions are divided into different groups based on what kind of data type that they operate on. The most common data types are strings, numbers and dates. The documentation provides a [list of all functions][3] that data.world provides.

Some examples of functions below:
##### Numeric
* `abs` - The absolute value of a numeric value. Takes one argument.
* `sqrt`- The square root function. Takes one argument.
* `cos` - The cosine function, from trigonometry. Takes one argument.

##### String
* `length` - The length of a string. Takes one argument.
* `concat` - Concatenates two strings, returning a string consisting of the first argument followed by the second. Takes two arguments.
* `regex` - Check whether a string matches a given regular expression. Similar to LIKE, but more powerful. Takes two arguments, the string and the regex to match to it.

##### Date/Time
* `month` - Extract the month value from a date or date time value, as an integer (January = 1). Takes one argument
* `date_add` - Adds a fixed duration to a date. Takes three arguments, the original date, a count, and a date part string

## Data Types
Every column in a SQL table must be only one data type. Each value in that column will be of the same data type. Each SQL implementation will have its own definitions for data types. data.world uses the following: 
`STRING, INT, DECIMAL, DOUBLE, BOOLEAN, DATE, DATETIME, TIME`.   Other databases such as oracle use much [more precise data types][4].

#### Using non-aggregating functions without a FROM clause
Let's continue to use SQL as a calculator but use some of the numeric functions.

**Example 03** - calculate the area of a circle with radius 5
```sql
SELECT pi() * pow(5, 2) as area_of_circle;
```

**Example 04** - String Functions 
```sql
select concat(left('I am using SQL string functions right now', 11), 
              right('My guitar has broken two strings.', 12));
```

Before we look at the date and time functions we need to understand the three data types DATE, TIME and DATETIME
* `DATE` - Only the year, month, day - formatted with `YYYY-MM-DD`
* `TIME` - Only the hours, minutes, seconds and microseconds - formatted with `HH:MM:SS.UUUUUU`
* `DATETIME` -  All of the above - year, month, day, hour, minute, second, mircosecond - formatted by putting the letter T between date and time `YYYY-MM-DDTHH:MM:SS.UUUUUU`

To use the date, time and datetime functions your column must be one of those specific data types. To manually create these data types you need to use the `CAST` command with the correctly formatted string. See the example query for details.
 
**Example 05** 
```sql
SELECT 
    cast('2013-05-08' as date) as date_col,
    cast('15:20:10.298348' as time) as time_col,
    cast('2016-12-01T09:24:34.87' as datetime) as datetime_col;
```

After creating a date, a time and a datetime value we can use functions on them.

**Example 06**
```sql
SELECT 
    day(cast('2013-05-08' as date)) as day_from_date,
    seconds(cast('15:20:10.298348' as time)) as seconds_from_time,
    date_sub(cast('2016-12-01T09:24:34.87' as datetime), 5, 'year') as subtract_5_years,
    now();
```

## Using functions on datasets with the FROM clause
We are finally ready to use these functions on actual data. It is very rare to use the SELECT statement on its own without the FROM clause as was done above. 

### Science Giving Dataset
We will use the [science giving dataset from fivethirtyeight][5]. Visit the link to get a summary of the dataset and a better description of the columns.

### Aggregating with `count`
The `count` function counts the number of rows in two distinct ways. If `count(*)` is used then the total number of rows are returned. If `count(column_name)` is used then the count of **non-null** values are returned. `count(1)` is the same as `count(*)`.

**Example 07**
```sql
select 
    count(*)
from 
    science_federal_giving;
```
If we use `count` with specific columns then the number of non-missing values are returned.

**Example 08**
```sql
select
    count(cmte_nm) as cmte_nm_count,
    count(cand_name) as cand_name_count,
    count(1) as count_one
from
    science_federal_giving;
```
### Other aggregate functions
Let's see examples of other aggregate functions on a numeric column like `transaction_amt`

**Example 09**
```sql
select
    sum(transaction_amt) as sum_amt, 
    min(transaction_amt) as min_amt,
    max(transaction_amt) as max_amt,
    avg(transaction_amt) as avg_amt,
    stdev(transaction_amt) as std_amt,
    variance(transaction_amt) as var_amt,
    count(transaction_amt) as count_amt
from
    science_federal_giving;
```
We can use non-aggregating functions as well. Here we use a regular expression to replace all the values from the first comma onwards with the empty string, effectively deleting them. This will return us the last name of each candidate.

**Example 10**
```sql
select 
    replace(cand_name, ',.*', '') as last_name
from 
    science_federal_giving 
limit 10;
```

## Aggregating with GROUP BY
We can greatly extend the capabilities of the aggregating functions by performing them on a grouping of data. Typically we will need to find aggregations, not over the entire dataset but over a subset of independent groups. We use the `GROUP BY ` clause to separate our data into independent groups before performing an aggregation on each group. There are three distinct parts when grouping:

* Grouping columns
* Aggregating columns
* Aggregating functions

For instance, let's say we want to find the average donation for each candidate. The grouping column is the candidate. The aggregating column is the donation amount and the aggregating function is the average. This example contains only one of each, but it's possible to have multiple for all three. Let's see how this is done.

**Example 11**
```sql
select 
    cand_name,
    avg(transaction_amt) as average_amt
from
    science_federal_giving_samp
group by
    cand_name
```
Notice that the grouping column must appear under `SELECT` and `GROUP BY`. Let's do another simple grouping by finding the average donation of each occupation.

**Example 12**
```sql
select 
    occupation,
    sum(transaction_amt) as total
from
    science_federal_giving_samp
group by
    occupation
```

### Multiple Aggregating Functions
We can use as many aggregating functions as we would like.

**Example 13**
```sql
select 
    cmte_pty,
    sum(transaction_amt) as total,
    count(1) as number_transactions,
    avg(transaction_amt) as avg_amt,
    max(transaction_amt) as max_amt,
    min(transaction_amt) as min_amt,
    variance(transaction_amt) as var_amt
from
    science_federal_giving_samp
group by
    cmte_pty
```

### Multiple Aggregating Columns
In the previous examples, we were only aggregating a single column, the `transaction_amt`. It is possible to use any number of columns to perform an aggregation. A popular method of aggregation is to count the unique number of values per group. You do this with `count(distinct colname)`.

**Example 14**
```sql
select 
    occupation,
    avg(transaction_amt) as avg_amount,
    max(cand_name) as max_cand_name,
    min(cand_name) as min_cand_name,
    count(distinct employer) as num_unique_employer
from
    science_federal_giving_samp
group by
    occupation
```

### Multiple Grouping Columns
Thus far we have only grouped by a single column. Let's group by multiple columns and additionally use an `ORDER BY` to help make the data easier to consume.

**Example 15**
```sql
select
    cmte_pty,
    occupation,
    sum(transaction_amt) as total
from
    science_federal_giving_samp
group by
    cmte_pty,
    occupation
order by
    total desc
```

### Grouping columns must appear both under the SELECT and GROUP BY clauses
In order to group properly, the grouping columns must appear underneath `SELECT` and `GROUPBY`. Some SQL implementations will cause an error to happen if the grouping column don't appear in the SELECT statement. Others will complete the operation but it will make for difficult interpretation.

**Example 16**
```sql
select
    cmte_pty,
    sum(transaction_amt) as total
from
    science_federal_giving_samp
group by
    cmte_pty,
    occupation
```

### Using the WHERE clause before grouping
It is possible to first filter your data with the `WHERE` clause before grouping. Let's find the total donations by occupation for just the state of Texas.

**Example 17**
```sql
select 
    occupation,
    sum(transaction_amt) as total
from
    science_federal_giving_samp
where
    state = 'TX'
group by
    occupation
```

### HAVING clause
The `HAVING` clause allows you to filter your data after you have performed your grouped aggregations. In contrast, the `WHERE` clause filters your data before you group. Let's find all the states that have at least 500 different donations.

**Example 18**
```sql
select
    state,
    count(1) as num_donations
from
    science_federal_giving_samp
group by
    state
having
    count(1) > 500
order by
    num_donations desc
```

Notice that I used the function `count(1)` in both the SELECT and HAVING clauses. Why couldn't I reference the alias `num_donations` in the HAVING clause like I did in ORDER BY. Some SQL implementations will allow this and others will not. It depends on their order of execution. You can actually use any other aggregation conditional statement in the HAVING clause.

### Putting it all together
To use everything that we have learned we can query for the total and count of all transactions by employer and party in the states of Texas and California that have at least 10 transactions ordering by party and number of transactions.

**Example 19**
```sql
select
    cmte_pty,
    employer,
    sum(transaction_amt) as total,
    count(transaction_amt) as total_transactions
from 
    science_federal_giving_samp
where
    state in ('TX', 'CA')
group by
    cmte_pty,
    employer
having
    count(transaction_amt) > 10
order by  
    cmte_pty,
    total_transactions desc;
```

# Exercises


## Resources
* [City of Houston][1] data.world account

[1]: https://data.world/houston
[2]: https://docs.oracle.com/cd/B19306_01/server.102/b14200/functions001.htm
[3]: https://docs.data.world/tutorials/dwsql/#built-in-functions
[4]: https://docs.oracle.com/cd/B28359_01/server.111/b28318/datatype.htm
[5]: https://data.world/fivethirtyeight/science-giving/workspace/project-summary