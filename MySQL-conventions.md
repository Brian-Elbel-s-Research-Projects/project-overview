## Create view table
At times, you may want to use a subset of a table instead of a full table, as part of your query, if you are only looking at a subset of products, or transactions.
Instead of adding a new table to the database (please talk to Erilia if you believe a new table should be added), create a *view* table.
Using a view table is exactly the same as a base table (a table that comes with the database), but a view table does not make a physical copy of the table, and therefore does not take up additional memory.

To create a view table:
```SQL
CREATE VIEW name_of_view_table AS
    SELECT 
        col1, col2
    FROM
        base_table
    GROUP by col1;
```

To differentiate view tables from base tables, you should end the table name with ```_view```.
For example, if you want to make a subset of product table with just burritos, the new view table could be named something like ```product_burritos_view```.

To see exactly which tables are base and view, query ```show full tables;```
