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

```SQL
MariaDB [tacobell2]> show full tables;
+-----------------------+------------+
| Tables_in_tacobell2   | Table_type |
+-----------------------+------------+
| GC_HEADER_DIM_2015_Q4 | BASE TABLE |
| GC_HEADER_DIM_2016_Q1 | BASE TABLE |
| GC_HEADER_DIM_2016_Q2 | BASE TABLE |
| GC_HEADER_DIM_2016_Q3 | BASE TABLE |
| GC_HEADER_DIM_2016_Q4 | BASE TABLE |
| GC_HEADER_DIM_2017_Q1 | BASE TABLE |
| GC_HEADER_DIM_2017_Q2 | BASE TABLE |
| GC_HEADER_DIM_2017_Q3 | BASE TABLE |
| GC_HEADER_DIM_2017_Q4 | BASE TABLE |
| GC_HEADER_DIM_2018_Q1 | BASE TABLE |
| GC_HEADER_DIM_2018_Q2 | BASE TABLE |
| GC_HEADER_DIM_2018_Q3 | BASE TABLE |
| GC_HEADER_DIM_2018_Q4 | BASE TABLE |
| GC_HEADER_DIM_2019_Q1 | BASE TABLE |
| GC_HEADER_DIM_2019_Q2 | BASE TABLE |
| GC_HEADER_DIM_2019_Q3 | BASE TABLE |
| GC_HEADER_DIM_2019_Q4 | BASE TABLE |
| GC_HEADER_DIM_2020_Q1 | BASE TABLE |
| GC_HEADER_DIM_2020_Q2 | BASE TABLE |
| test_occasion         | BASE TABLE |
+-----------------------+------------+
```

To learn more about creating view tables, check [this](https://www.mysqltutorial.org/create-sql-views-mysql.aspx) out.
