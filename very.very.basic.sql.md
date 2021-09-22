# Very Very Basic SQL for Koha Reports

<br /><br />

## Q: What does "SQL" stand form

SQL stands for Structured Query Language.  

A librarian way of thinking about it is that SQL is a controlled vocabulary for asking the database questions.

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

## The basic components of a custom report in koha

Every custom report you create in Koha is going to start with the word "SELECT"

The custom report interface in Koha is designed

~~~ JavaScript
$ ;
~~~
