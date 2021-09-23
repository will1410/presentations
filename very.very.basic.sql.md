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

To stretch the reference librarian analogy further, this query is asking the question "What is the information // in all of the fields // in // this table?"

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

#### Sample 001

~~~sql
SELECT
  *
FROM
  items
~~~

## A slightly more complex report

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildcard.

To use the reference librarian analogy again, this query is asking the question "What is the information // in field1, field2, and field3 // in // this table?"

- SELECT = What is the information
- field1, field2, field3 = in these fields
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

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildcard.

#### Sample 002

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

This updates the question to "What is the information // in field1, field2, and field3 // in // this table // if field1 matches X?"

- SELECT = What is the information
- field1, field2, field3 = in these fields
- FROM = in
- tablename = this table
- WHERE - if some field
- tablename.field1 = "X" = matches X

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

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildcard.

#### Sample 003

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

There are different types of basic constraints when working with numeric values:

- = (exaxt match)
- < (less than)
- \> (greater than)
- <= (less than or equal)
- \>= (greater than or equal)
- <> (does not equal)
- \!= (does not equal)
- BETWEEN x AND y

These questions fit our analogy like  "What is the information // in field1, field2, and field3 // in // this table // if field1 (matches|is less than|is greater than|etc. etc.) X?"

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

This report begins and ends with a "SELECT" and a "FROM tablename" but this time we're going to choose the fields we want instead of selecting them all with a wildcard.

#### Sample 004

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

#### Sample 005

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

#### Sample 006

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

#### Sample 007

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

#### Sample 008

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

#### Sample 009

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
#### Sample 010

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

## Constraint is not

You can also use WHERE NOT to constrain the results to anything except a specific match.  This works a lot like WHERE <> or WHERE !=

These questions fit our analogy like  "What is the information // in field1, field2, and field3 // in // this table // if field1 is not (matches|is less than|is greater than|etc. etc.) X?"

~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
WHERE NOT
  tablename.field1 = "X"
~~~

### Actual samples that will work in Koha

#### Sample 011

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber = 100
~~~

## Adding multiple constraints

We can also add multiple constraints using AND or OR

~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
WHERE
  tablename.field1 = "X" AND
  tablename.field2 = "Y"
~~~

#### OR

~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
WHERE
  tablename.field1 = "X" OR
  tablename.field2 = "Y"
~~~

#### OR

~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
WHERE
  tablename.field1 = "X" AND NOT
  tablename.field2 = "Y"
~~~

~~~sql
SELECT
  tablename.field1,
  tablename.field2,
  tablename.field3
FROM
  tablename
WHERE
  tablename.field1 = "X" OR NOT
  tablename.field2 = "Y"
~~~

### Actual samples that will work in Koha

#### Sample 012

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber = 100 AND
  items.biblionumber = 50
~~~

#### Sample 013

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber = 100 OR
  items.biblionumber = 50
~~~

#### Sample 014

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber = 100 AND NOT
  items.biblionumber = 50
~~~

#### Sample 015

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber = 100 OR NOT
  items.biblionumber = 50
~~~

## These samples are just examples

Don't limit yourself, either, to what you see in these sample reports.  All of the previous several examples are using constraints on two separate fields - itesm.itemnumber and items.biblionumber.

There's nothing to say you can use multiple constraints on the same field like:

#### Sample 016

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber = 100 OR
  items.itemnumber = 50
~~~

#### Sample 017

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber = 100 OR NOT
  items.itemnumber < 500
~~~

## Ordering the results

The next thing I'll throw in is ordering the results.

The command for this is "ORDER BY" and then you have the option to sort ascending (*ASC*) or descending (*DESC*).  The default sort order is *ASC*

To stretch the reference librarian analogy further, this query is asking the question "What is the information // in field1, field2, and field3 // in // this table // if field1 is not (matches|is less than|is greater than|etc. etc.) X; and show me the data in a certain order?"

#### Sample 018

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber < 100 OR
  items.itemnumber > 1000
ORDER BY
  items.barcode
~~~

#### Sample 019

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber < 100 OR
  items.itemnumber > 1000
ORDER BY
  items.barcode ASC
~~~

#### Sample 020

~~~sql
SELECT
  items.itemnumber,
  items.biblionumber,
  items.barcode
FROM
  items
WHERE NOT
  items.itemnumber < 100 OR
  items.itemnumber > 1000
ORDER BY
  items.barcode ASC
~~~

## Mixing up the ordering

And you can sort with multiple fields the results with some fields ascending and some fields descending

For this sample, I'm going to switch up tables:

#### Sample 021

~~~sql
SELECT
  biblio.biblionumber,
  biblio.author,
  biblio.title
FROM
  biblio
WHERE
  biblio.biblionumber < 200
ORDER BY
  biblio.author DESC,
  biblio.title ASC
~~~

## Counting

A very common and very basic thing that many Koha libraries want to do with reports is to count things.  The most basic count is to count all of the rows in a set of results.

This query is very basic and asks the question "What is the result // if I count all the rows // in // this table?"

#### Sample 022

~~~sql
SELECT
  Count(*)
FROM
  biblio
~~~

## Counting and grouping

Counting the rows in a table can get you some data, but it's usually more useful to group results based on the data in a specific field.

This query asks the question "What is the result // if I count all of the rows // in this table // and group them by the author's name?"

#### Sample 023

~~~sql
SELECT
  biblio.author,
  Count(*)
FROM
  biblio
GROUP BY
  biblio.author
~~~

## Joining tables

So far, everything I've shown you deals with one table at a time.  It's a lot more useful when you're writing reports in Koha if you can get data from multiple tables at the same time.

The command for this is "JOIN"

This query asks the question "What is the information // in table1.field1 and table2.field1 // in this table when it's joined to this other table?"

#### Sample 024

~~~sql
SELECT
  biblio.author,
  biblio.title,
  items.barcode
FROM
  biblio JOIN
  items ON items.biblionumber = biblio.biblionumber
~~~

## Joining tables

There are several different types of JOINs you can use in SQL:

- JOIN
- INNER JOIN
- LEFT JOIN
- RIGHT JOIN
- FULL OUTER JOIN

For the purposes of Koha, you really only need to know JOIN and LEFT JOIN

This query asks the question "Show me all of the information // in table1.field1 and table2.field1 // in this table when it's joined to this other table // even if there's no connection between the tables?"

#### Sample 025 types of joins

~~~sql
SELECT
  branches.branchcode,
  Count(items.itemnumber) AS Count_itemnumber
FROM
  branches LEFT JOIN
  items ON items.homebranch = branches.branchcode
GROUP BY
  branches.branchcode
~~~
