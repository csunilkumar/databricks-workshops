# 06. Azure Data Factory v2 - Lab instructions

In this lab module - we will learn to automate Databricks Spark applications with Azure Data Factory v2.  We will create a simple Data Factory pipeline that runs sequential and parallel activities that include spawning a new Databricks cluster based on specs, and executing a pre-created and tested notebooks in your Databricks workspace.  We will learn to schedule the pipeline to run on a time basis, as well as run on-demand.  Finally, we will learn to monitor in Azure Data Factory v2.<br>

## A) Dependencies
1.  Provision data factory
2.  Completion of primer lab for Azure Storage
3.  Completion of primer lab for Azure SQL Database
4.  Adequate cores to provision Azure databricks cluster
5.  Notebooks in the module, loaded into your workspace
6.  Database objects - tables and stored procedures/functions, created ahead of time

## B) Database objects to be created in Azure SQL Database

1.  Create table: batch_job_history
```
DROP TABLE IF EXISTS dbo.BATCH_JOB_HISTORY; 
CREATE TABLE BATCH_JOB_HISTORY( 
batch_id int, 
batch_step_id int, 
batch_step_description varchar(100), 
batch_step_status varchar(30), 
batch_step_time varchar(30) );
```

2.  Create table: chicago_crimes_count
```
DROP TABLE IF EXISTS dbo.CHICAGO_CRIMES_COUNT; 
CREATE TABLE CHICAGO_CRIMES_COUNT( 
case_type varchar(100), 
crime_count bigint);
```

3.  Create table: chicago_crimes_count_by_year
```
DROP TABLE IF EXISTS dbo.CHICAGO_CRIMES_COUNT_BY_YEAR; 
CREATE TABLE CHICAGO_CRIMES_COUNT_YEAR( 
case_year int,
case_type varchar(100), 
crime_count bigint);
```

4a.  Create stored procedure: generate_batch_id
```
CREATE PROCEDURE generate_batch_id 
   @new_batch_id varchar(20) OUTPUT  
AS  
BEGIN  
   DECLARE @batch_count INT;

    SELECT @batch_count =  count(batch_id)
    FROM batch_job_history;
    
    IF(@batch_count = 0)
      SET @new_batch_id = 1;
    ELSE
      SELECT @new_batch_id =  max(batch_id)+1
      FROM batch_job_history;

    SELECT CONVERT(varchar(20),@new_batch_id);
END
```

4b. Test procedure: generate_batch_id
```
DECLARE @new_batch_id varchar(20);
EXEC dbo.generate_batch_id @new_batch_id
```
