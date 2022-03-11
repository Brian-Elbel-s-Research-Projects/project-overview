## Using High Performing Cluster (HPC) at NYU Langone

Get started [here](http://bigpurple-ws.nyumc.org/wiki/index.php/BigPurple_HPC_Cluster). 

### Apply for an account
You would need to apply to get an account on HPC, and request access to:
- the team's shared lab space at gpfs/data/elbellab
- database access: tacobell and tacobell2

Write to <hpc_admins@nyumc.org> and apply for an account with the email template below. 
Once you have an account, write to Brian and request access to the team's shared lab space and databases (as outlined above).
When you get the approval, forward it to the HPC admins, and request access.

##### Email template for account application
1.	The new user’s full name
2.	The new user’s MCIT-assigned Kerberos ID (usually the user’s initials or part of the user’s name followed by a number of digits)
3.	The new user’s email address (to receive important announcements via the BigPurple-HPCF mailing list)
4.	The department/center of the new user: **department of population health**
5.	Their division/section within the department: **section on health choice, policy and evaluation**
6.	The name of the new user’s PI: **Dr. Brain Elbel**
7.	The title of the project for which BigPurple-HPC resources are to be used: **using national sales data to understand the influence of menu labeling policy**
8.	A description of the research conducted by the new user (one short paragraph): **our research utilizes national sales data at a receipt level from a large fast food restaurant chain. We plan to use this data to assess whether consumers changed their orders and purchases in response to the menu labeling mandate, which requires fast food restaurant chains to label their menu board with calorie information.**
9.	A description of the BigPurple-HPC needs such as data storage, compute power, data transfers, etc. (one short paragraph): **I need a standard personal account to test my scripts, as well as access to the lab space under Dr. Elbel at gpfs/data/elbellab. I also need access to the two databases named tacobell and tacobell2.**

### Access tacobell database
#### Through PuTTY or command line
Most projects involving menu labeling research will use the ```tacobell``` database.
To use the database, log into your HPC account.
```bash
module load mariadb/5.5.64
mysql -p -h db -P 33061 -u username tacobell #replace username with your Kerberos ID
```
You'll then be prompted to enter your password.

Now that you are in the database environment, use MySQL queries to explore the data.
Just make sure to end your queries with ```;```
```SQL
show tables; #see a list of all tables in the database
describe ALGIN_DIM; #get metadata of a particular table
select * from ALIGN_DIM; #read in the whole table
select DW_RESTID,ADDRESS_LINE_1,CITYNAME,STATENAME,PSTL_ZIP_CD from ALIGN_DIM; #read select columns
```

#### Through ```RMySQL``` library in the interactive RStudio
The command line isn't the most friendly environment to test your code.
You can use an interactive [RStudio](https://rstudio.hpc.nyumc.org/) instead.
In order to connect to the ```tacobell``` database through this environment,
```R
library(RMySQL)
tb <- dbConnect(drv = RMySQL::MySQL(), dbname="tacobell",
                username="username", #replace username with your Kerberos ID
                password=readline(prompt="password: "), #do not store your password in plain text here
                host="db", port=33061)
```
To explore the database in this environment,
```R
dbListTables(tb) #see a list of all tables in the database
dbListFields(tb, "ALIGN_DIM") #get a list of all columns in one table
restaurant <- dbReadTable(tb, "ALIGN_DIM") #read in the whole table
#read select columns, or run a query
query <- "select DW_RESTID,ADDRESS_LINE_1,CITYNAME,STATENAME,PSTL_ZIP_CD from ALIGN_DIM"
restaurant <- dbGetQuery(tb, query)
```
Note that queries through the ```RMySQL``` library do not need to end with ```;```

### Schedule a job
Most of the times, we schedule scripts (i.e. jobs) to be run on HPC in the background, instead of in the interactive environment.
This is usually much faster, and makes sure that your script won't be interrupted.
Once you are done code testing, submit the job to HPC following these instructions [here](http://bigpurple-ws.nyumc.org/wiki/index.php/Job-Scheduler#Scheduling_background_jobs).
Each job has a unique ID, and you can check the status of your script with ```squeue -u username```.

##### Adding print statements to your script
It's helpful to have ```print()``` statements in the scripts in case you need to debug the code.
For example, you may want to consider something like this,
```R
library(RMySQL)
print("RMySQL loaded")

tb <- dbConnect(drv = RMySQL::MySQL(), dbname="tacobell",
                username="wue04", password="",
                host="db", port=33061)
print("database connected")

print("mean calories")
for (i in 2007:2015) {
    for (j in 1:4) {
        tryCatch(
            if(i==2015&j==4) {stop("file doesn't exist")} else
            {
                query <- paste0("select t.DW_RESTID, y.DW_MONTH, 
                                sum(coalesce(n1.CALORIES,0)*t.ACTQTYSOLD*l.ITEMMOD + coalesce(n2.CALORIES,0)*t.ACTMODQTY*l.ITEMMOD) as cal,
                                count(distinct DW_GC_HEADER) as count
                                from TLD_FACT_", i, "_Q0", j," t
                                inner join restaurant_in_use_match_drive_thru r using(DW_RESTID)
                                left join TIME_DAY_DIM y using(DW_DAY)
                                left join nutrition n1 on n1.DW_PRODUCT=t.DW_PRODUCTDETAIL
                                left join nutrition n2 on n2.DW_PRODUCT=t.DW_PRODUCTMOD
                                group by DW_RESTID, DW_MONTH")
                sales <- dbGetQuery(tb, query)
                print(paste0("sample query works, ",i,"Q", j))
                write.csv(sales, row.names = FALSE,
                          paste0("/gpfs/home/wue04/tb-data/mean-calorie-w-mod/mean-calorie_restid_",i,"_Q",j,".csv"))
                print(paste0("file saved: , ",i,"Q", j))
           }, error=function(e){cat(paste0("ERROR : ",i,"Q",j),conditionMessage(e), "\n")}
       )
    }
}
dbDisconnect(tb)
```
There are ```print()``` statements embedded throughout the script.
If a job you submitted was aborted before it was done, these statements help you understand which part of the script ran and where the error may have occurred.

## Make git repo on HPC
Often time, when you run queries on HPC, and analyze the exported data on your local machine, you want to version control and track those scripts in both of these locations.
To track a folder on HPC, simply clone the GitHub repo onto your destination folder on HPC.
In the HPC terminal:

```bash
module load git/2.17.0
cd <destination_folder> #change directory to the one to be tracked
git clone <REPO_URI> temp #copy the URI from the new repo you just created on GitHub
mv temp/.git . # Copy the hidden git directory
mv temp/* .    # Copy every non-hidden file
rm -rf temp    # Cleanup
```

If this is the first time you're using git on HPC, you may want to change your username and email address.
```bash
git config --global user.name "user_name"
git config --global user.email "email_address"
```

## Multi-core processing 

## Create a view table
