Some of the quirks in the database.

## Database structure
The database has a *star* structure, where a central fact table (or in our case, a series of central tables, by fiscal quarters) can be linked to many reference tables (or dimension tables).

### Reference tables
#### ALIGN_DIM
This table contains restaurant related information.
The primary key is ```DW_RESTID```.
One thing to note is that ```DW_RESTID``` is different from addresses.
I.e. when a restaurant changes ownership (e.g. from company owned to franchise owned, or from one frachisee to another), the same restaurant will have a new ```DW_RESTID```, even though to customers, the restaurant is exactly the same.
Because of this, our analysis is at the unique restaurant level, where we identify a restaurant with a combination of address, ownership type, joint brand type and census tract number (census tract number is added by us based on its geospatial coordinates)

```sql
MariaDB [tacobell]> select DW_RESTID,
    ->   AGREEMENTTYPE, #3 ownership types: company owned, franchise and license (we consider the latter two the same)
    ->   CONCEPTDESC, #some restaurants are joint brands between Taco Bell and other YUM! brands, such as Pizza Hut and KFC
    ->   ADDRESS_LINE_1,CITYNAME,COUNTYNAME,STATENAME,PSTL_ZIP_CD,LATITUDE,LONGITUDE,
    ->   OPENEDDT,TEMPCLOSEDDT,REOPENDT,CLOSEDDT
    -> from ALIGN_DIM
    -> where OPENEDDT>='2007-01-01' #the table contains many historical records where data quality isn't as convincing for stores that opened a long time ago
    -> limit 10;
/*+-----------+---------------+-------------+----------------------------+-------------+------------------+-----------+-------------+----------+-----------+------------+--------------+------------+------------+
| DW_RESTID | AGREEMENTTYPE | CONCEPTDESC | ADDRESS_LINE_1             | CITYNAME    | COUNTYNAME       | STATENAME | PSTL_ZIP_CD | LATITUDE | LONGITUDE | OPENEDDT   | TEMPCLOSEDDT | REOPENDT   | CLOSEDDT   |
+-----------+---------------+-------------+----------------------------+-------------+------------------+-----------+-------------+----------+-----------+------------+--------------+------------+------------+
|    150977 | FRN           | TBC         | 33203 WEST EIGHT MILE ROAD | LIVONIA     | WAYNE            | MI        | 48152       |  42.4402 |  -83.3737 | 2008-05-16 | 0000-00-00   | 0000-00-00 | 0000-00-00 |
|    151207 | LIC           | TBC         | 2121 N. MONROE STREET      | MONROE      | MONROE           | MI        | 48162-5342  |  41.9474 |  -83.3872 | 2008-06-01 | 0000-00-00   | 0000-00-00 | 0000-00-00 |
|    151667 | LIC           | TBC         | 650 COLUMBIA AVENUE        | CHAPIN      | LEXINGTON        | SC        | 29036       |  34.1761 |  -81.3245 | 2009-05-22 | 0000-00-00   | 0000-00-00 | 0000-00-00 |
|    151391 | FRN           | TBC         | 17610 US HIGHWAY 31 N      | WESTFIELD   | HAMILTON         | IN        | 46074-9413  |  40.0431 |  -86.1356 | 2008-12-23 | 0000-00-00   | 0000-00-00 | 2012-12-18 |
|    151989 | FRN           | TBC         | 305 SOUTH WARRINGTON ROAD  | PENSACOLA   | ESCAMBIA         | FL        | 32507       |  30.4040 |  -87.2773 | 2011-05-18 | 0000-00-00   | 0000-00-00 | 0000-00-00 |
|    151115 | FRN           | KFC/TBC     | 32120 S. LAS VEGAS BLVD.   | PRIMM       | CLARK            | NV        | 89019       |  35.6100 | -115.3880 | 2008-11-24 | 0000-00-00   | 0000-00-00 | 0000-00-00 |
|    151345 | FRN           | TBC         | 4380 KEARNY MESA ROAD      | SAN DIEGO   | SAN DIEGO        | CA        | 92111       |  32.8199 | -117.1490 | 2008-11-12 | 0000-00-00   | 0000-00-00 | 2013-10-29 |
|    151161 | FRN           | TBC         | 6045 SOUTH FIRST ST        | MILAN       | GIBSON           | TN        | 38358       |  35.9064 |  -88.7526 | 2008-04-09 | 0000-00-00   | 0000-00-00 | 0000-00-00 |
|    150931 | FRN           | TBC         | 1520 NORTH HILLMAN STREET  | TULARE      | TULARE           | CA        | 93274       |  36.2271 | -119.3310 | 2007-09-07 | 2014-02-09   | 2014-03-12 | 0000-00-00 |
|    151308 | COM           | KFC/TBC     | 2971 DOUGHERTY FERRY RD.   | SAINT LOUIS | SAINT LOUIS CITY | MO        | 63122       |  38.5680 |  -90.4763 | 2009-11-16 | 0000-00-00   | 0000-00-00 | 2012-03-20 |
+-----------+---------------+-------------+----------------------------+-------------+------------------+-----------+-------------+----------+-----------+------------+--------------+------------+------------+ */
```
#### LINEITEM_DIM
This tables gives you an overview of what each line in a receipt means.
The 3rd column in the table ITEMMOD is added based on the interpretation of LINEITEMDESC, to quantify how much an item or an ingredient was present in the food.
This is to account for item swap in combo meals (e.g. someone may want a chicken taco and a beef taco when the combo meal default is 2 chicken tacos) and ingredient changes (e.g. customers may want to add extra cheese or have no lettuce in their burritos), which could affect the calorie in a food item.

```sql
MariaDB [tacobell]> select * from LINEITEM_DIM order by DW_LINEITEM;
/*+-------------+--------------------------+---------+
| DW_LINEITEM | LINEITEMDESC             | ITEMMOD |
+-------------+--------------------------+---------+
|          -1 | N/A                      |     0.0 |
|           1 | COMBO-ITEM               |     1.0 |
|           2 | NON-COMBO-ITEM           |     1.0 |
|           3 | COMBO-DETAIL             |     1.0 |
|           4 | COMBO-DETAIL-REPLS-PLUS  |     1.0 |
|           5 | COMBO-DETAIL-REPLS-MINUS |    -1.0 |
|           6 | COMBO-DETAIL-REPLS-EQL   |     1.0 |
|           7 | COMBO-MOD-INGRD-ADD      |     1.0 |
|           8 | COMBO-MOD-INGRD-MINUS    |    -1.0 |
|           9 | COMBO-MOD-INGRD-EASY     |     0.5 |
|          10 | NON-COMBO-M-INGRD-ADD    |     1.0 |
|          11 | NON-COMBO-M-INGRD-MINUS  |    -1.0 |
|          12 | NON-COMBO-M-INGRD-EASY   |     0.5 |
|          13 | DISCOUNT-LINE            |     1.0 |
|          14 | TAX-LINE                 |     0.0 |
|          15 | COMBO-MOD-INGRD-ONLY     |     1.0 |
|          16 | NON-COMBO-M-INGRD-ONLY   |     1.0 |
+-------------+--------------------------+---------+*/
```

#### OCCASION_DIM
```SQL
MariaDB [tacobell]> select * from OCCASION_DIM;
/*+-------------+------------+--------------+
| DW_OCCASION | OCCASIONCD | OCCASIONDESC |
+-------------+------------+--------------+
|           1 | EIN        | EAT-IN       |
|          -1 | NA         | NA           |
|           3 | TKT        | TAKE-OUT     |
|           2 | DRT        | DRIVE-THRU   |
+-------------+------------+--------------+*/
```
#### PRODUCT_DIM (and PRODUCT_DETAIL_DIM_V1 and PRODUCT_MODIFICATION_DIM_V1)
This table contains food and beverage items in the database.
The primary key is DW_PRODUCT.
Each item has a brief description.
Note that the description is not unique to the DW_PRODUCT key.
See the example below.
```sql
MariaDB [tacobell]> select DW_PRODUCT,PRODUCTDESC from PRODUCT_DIM where PRODUCTDESC='MEDIUM PEPSI';
/*+------------+--------------+
| DW_PRODUCT | PRODUCTDESC  |
+------------+--------------+
|       3648 | MEDIUM PEPSI |
|       3139 | MEDIUM PEPSI |
+------------+--------------+*/
```
Tables PRODUCT_DETAIL_DIM_V1 and PRODUCT_MODIFICATION_DIM_V1 have the same content as PRODUCT_DIM, with the difference in column names.
This is important to note because it directly relates to how calories are aggreagted for combo meals and food items with ingredient modifications.
See the section on TLD_FACT tables for more information.

#### PRODUCT_GROUP_DET (and PRODUCT_GROUP_DETAIL_DET_V1 and PRODUCT_GROUP_MOD_DET_V1)
Similar to PRODUCT_DIM table, each DW_PRODUCTGROUP has a brief textual description.
It's unclear how the product groups are defined, as many of them are not mutually exclusive.
For our purposes, all beverages are listed in groups 15 (smoothies), 16 (drinks) and 17 (packaged dri).

#### TIME_DAYPART_DET
Taco Bell's [daypart](https://en.wikipedia.org/wiki/Dayparting) for meal time.
```sql
MariaDB [tacobell]> select * from TIME_DAYPART_DET order by DW_DAYPART;
/*+------------+----------------+-------------+--------------+--------------+
| DW_DAYPART | DW_CURRENTFLAG | DAYPARTNAME | DAYPARTBGNTM | DAYPARTENDTM |
+------------+----------------+-------------+--------------+--------------+
|          1 |                | LATE NIGHT  | 00:00        | 03:59        |
|          2 |                | BREAKFAST   | 04:00        | 10:59        |
|          3 |                | LUNCH       | 11:00        | 13:59        |
|          4 |                | AFTERNOON   | 14:00        | 16:59        |
|          5 |                | DINNER      | 17:00        | 20:59        |
|          6 |                | EVENING     | 21:00        | 23:59        |
+------------+----------------+-------------+--------------+--------------+*/
```

#### TIME_DAY_DIM and TIME_MINUTE_DIM
These two tables are mostly used to connect DW_YEAR and DW_MONTH to human readable year and month.
Users can also use the TIME_MINUTE_DIM table to join with the TIME_DAYPART_DET table.

#### nutrition
This table is uploaded by us, which links food and beverage products to calorie information.
Of the items in the PRODUCT_DIM table, 3,515 unique items were examined.
(The others were not researched for various reasons. E.g. they were items from non-Taco Bell brands, they were not food items, etc.)

We were able to find calorie information on 741 items, which covers more than 95% of the sales from quarter to quarter.
Note that the carbohydrates and sugar information are likely inaccurate for non-typical sized beverages (i.e. outside of the typical small, medium and large sizes), as their records were sometimes imputed from different sized beverages.

Most of the time, to join nutrition information to transaction tables, use ```nutrition_view``` which excludes all items without calorie information.

#### product_category
This table is uploaded by us, which puts food and beverage items into 
```sql
MariaDB [tacobell]> select DW_CATEGORY, CATEGORY from product_category group by DW_CATEGORY order by DW_CATEGORY;
/*+-------------+--------------+
| DW_CATEGORY | CATEGORY     |
+-------------+--------------+
|           1 | beverage     |
|           2 | burrito      |
|           3 | dessert      |
|           4 | other_entree |
|           5 | salad        |
|           6 | side         |
|           7 | substitution |
|           8 | taco         |
|           9 | other        |
+-------------+--------------+*/
9 rows in set (0.03 sec)
```
### Transaction tables
#### GC_HEADER_DIM tables
#### TLD_FACT tables
- DW_GC_HEADER is no longer a unique primary key
## Notable quirks
### TLD_FACT_2007_Q01 has duplicate data
For reasons not clear to us, every row in the the TLD_FACT_2007_Q01 table (and only this table) seems to have a duplicate.
When you aggregate data from this table, e.g. total calories, you would need to divide everything by 2, except the number of transactions, which you can do by ```count(distinct DW_GC_HEADER)```
#### GC_HEADER_DIM, TLD_FACT and TLD_TEST are test tables
They should not be used in queries.
### Duplicate tables in product, product detail and product modification
### A slight mismatch between fiscal month and calendar month
### 4th quarter has 16 weeks
