# Very Very Basic SQL for Koha Reports

<br /><br />

## Q: What does "SQL" stand form

SQL stands for Structured Query Language.  

One way of thinking about it is that SQL is a controlled vocabulary for asking the database questions.

A reference librarian analogy would be that SQL is the language you use to ask reference questions to a database.

## Q: What does SQL have to do with Koha?

Koha uses SQL for custom reports.  To create a custom report in the Koha staff client, click on __*Reports*__, then __*Create from SQL*__.

## Q: What can you do with a custom report that you can't do with one of the built-in reports in Koha?

The Statistics wizard, Top lists, Inactive, Other reports, and reports made with the "Create guided report" tool can do a lot, but they have limitations.  Particularly the "Created guided report" tool.  That tool only offers you a handful of database tables that you can work with and what you can do with them.  Custom reports allow you to base reports on any data saved anywhere in the Koha database.

## Q: How do I learn about writing SQL

There is a very good general SQL tutorial at [W3 Schools](https://www.w3schools.com/sql/){:target="\_blank"}

There is also one at [Tutorials point](https://www.tutorialspoint.com/sql/index.htm){:target="\_blank"}

And LinkedIn Learning has a full "SQL for Non-Programmers" course at [https://www.linkedin.com/learning/sql-for-non-programmers/how-can-sql-answer-your-data-questions](https://www.linkedin.com/learning/sql-for-non-programmers/how-can-sql-answer-your-data-questions){:target="\_blank"}

## Q: Cheat sheets

There is a good cheat sheet at [https://learnsql.com/blog/standard-sql-functions-cheat-sheet/standard-sql-functions-cheat-sheet-letter.pdf](https://learnsql.com/blog/standard-sql-functions-cheat-sheet/standard-sql-functions-cheat-sheet-letter.pdf) but it is not Koha specific

## __*Special note*__

The custom report interface in Koha is designed only to read data from Koha -- you will not be able to use any SQL commands that create, alter, or delete any data from Koha's reports module.  For this reason you can't use any of these words when creating an SQL report in Koha:

- UPDATE
- DELETE
- DROP
- INSERT
- SHOW
- CREATE

## The most basic report possible

Every custom report you create in Koha is going to start with the word "SELECT"; followed by an asterix (wildcard for "all"); followed by the word "FROM"; followed by the table name.

To stretch the reference librarian analogy further, this query is asking the question "What is // all of the information // in // this table?"

- SELECT = What is
- \* = all of the information
- FROM = in
- tablename = this table

~~~sql
SELECT
  *
FROM
  tablename
~~~

### An actual sample that will work in Koha

This is just a sample.  It's not a very useful report.

This question is "What is all of the information in the *items* table"

~~~sql
SELECT
  *
FROM
  items
~~~

## A slightly more complex report

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildcard.

To use the reference librarian analogy again, this query is asking the question "What are // field1, field2, and field3 // in // this table?"

- SELECT = What is
- field1, field2, field3 = these fields
- FROM = in
- tablename = this table

~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
~~~

### An actual sample that will work in Koha

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildaard.

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
~~~

## Adding constraints

We can add "WHERE tablename.fieldname = 'X'"

This updates the question to "What are // field1, field2, and field3 // in // this table // if one specific field is 'X'?"

- SELECT = What is
- field1, field2, field3 = these fields
- FROM = in
- tablename = this table
- WHERE - IF
- tablename.field1 = "X"

~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
WHERE
  tablename.field1 = "X"
~~~

### An actual sample that will work in Koha

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildaard.

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber = 1
~~~

## Constraints besides equals

There are different types of basic constraints:

- = (exaxt match)
- < (less than)
- \> (greater than)
- <= (less than or equal)
- \>= (greater than or equal)
- <> (does not equal)
- \!= (does not equal)
- BETWEEN x AND y


~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
WHERE
  tablename.field1 = "X"
~~~

### Actual samples that will work in Koha

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildaard.

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber < 100
~~~

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber > 100
~~~

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber <= 100
~~~

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber >= 100
~~~

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber <> 100
~~~

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber != 100
~~~

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE
  items.itemnumber BETWEEN 100 AND 150
~~~
