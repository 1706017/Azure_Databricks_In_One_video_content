# Azure_Databricks_In_One_video_content
#Date: 17th Feb 2025
=====================

Q1) What is apache spark?

Sol:
	•  Apache Spark is a general purpose,in memory , compute engine 
	•  It has support for multiple language like python,scala,java etc
	•  It supports both batch and stream processing 
	•  It is 10 times faster than map reduce as it does computation in memory only 
=============================================================================
Q2) Explain Spark architecture?

Sol:
	•  Architecture consists of one Driver node, one cluster manager and multiple worker nodes
        
	• Worker nodes consists of executors and these executors are the actual components that executes the task
	 
	• When a user submits the code it first goes to driver node and then driver node analyse the code and break it into smaller tasks and then communicates with cluster manager regarding the worker nodes that needs to be created and then cluster manager will create that number of worker nodes and will also assign the tasks to be performed by each worker node

	• Now once the worker nodes executed all the tasks assigned to them then they return back the result directly to driver program 

![image](https://github.com/user-attachments/assets/58c8adf9-8539-4753-bf97-641db068055f)

=============================================================================
Q3) What is the difference between azure blob storage and azure data lake ?
Sol: 
	• The main difference between blob storage and data lake is that in datalake hierarchical namespace is enabled but in blob storage hierarchihcal namespace is not enabled 

	• Also hierarchihcal namespace means ability to create subfolder like structure inside the storage account container 
=============================================================================

Q4) What are magic commands in databricks ?
Sol:

	• Using magic commands we can execute the sql code inside a python notebook and vice versa

	• Some of the common magic commands like %sql, %python, %md etc
============================================================================

Q5) How to create dataframe explicitly ?
Sol:

 my_data = [(1,'aa',40),(2,'bb',50),(3,'cc',60)]

 my_schema = 'id INT, name STRING, age INT'

  df = spark.createDataFrame(my_data,schema=my_schema)

 display(df) #to print the dataframe
 =============================================================================

Q6) What is DBFS?
Sol:
	• It stands for databricks file system 

	• It is like abstraction layer on top of cloud storage (datalake) where we can easily access the data just by providing the path to the data 
	
It is just an abstraction layer but the actual data still resides in the internal cloud storage 

=============================================================================

Q7) How to access the data that is residing in our data lake and we want to access the data in databricks notebook ?
Sol:

	• We can access the data inside the datalake via databricks through service principle (also known as app registration)

Go to search then search for app registration -> click on create new registration -> then give name to your service principle/app registartion then click on create button


	• Once the app registration is create go to all application tab and see your service principle that you have created click on that it will get open then just copy the application id and tenant id of the service principle 

	• Then click on the certificates and secrets tab on the left side to create a new client secret once the secret is created just copy the value as it will get disappear the second time you open it 

	• Now we need to go to azure datalake -> go to IAM tab from the left side of datalake -> click on +add button -> then search for a role named as storage blob contributor -> then search for the service principle that you have created and assign the access to it as storage blob data contributor 


	• Now just using the microsoft documentation create the mount points and provide the values for app id, tenant id and client secret that we have stored 
============================================================================================================================================================================================

Q8) What is Databricks Utilities?
Sol:

There are many db utilities like 

A)  dbutils.fs.ls()
     
      dbutils.fs.ls("abfss://container@storage_Account_name.dfs.core.windows.net/")

B) dbutils.secrets()
	• In Azure we create azure key vault and that has all the secerts and In Databricks we create a secret scope this secret scope links the databricks to the secret in the azure key vault without hardcoding the credentials or secrets that means your secrets will be directly pulled from keyvault rather from hardcoded in notebook 

Once you have created an azure key vault -> go to the left side of key vault and click on IAM tab and give yourselft as a role of key vault admin then only you will be able to save the secrets in key vault


	• Now click on secrets from left side of key vault to save your secrets in keyvault 

	• Now to create a secret scope of the databricks just open your databricks and copy the url and past it in the browser and at the end append with #secrets/createScope 
	Then give name to your scope 
	Then it will ask for the DNS of key vault just copy the vault uri from key vault and paste here
	then provide the resource id of key vault as well then click on create and your databricks secret scope is created 

	•  dbutils.secrets.list(scope='your_databicks_secret_scope_name')
	 dbutils.secrets.get(scope='your_databricks_secret_scope_name',key='key_name_for_the_secret_that_you_have_created_in_keyvault') #and you can use this in place of hardcoding the values of secrets 
=============================================================================

Q9) How to read data and load it into dataframe using spark reader api ?
Sol:

  df = spark.read.format('csv')\
          .option('header',True)\
          .option('inferSchema',True)\
          .load('abfss://container@storage_account_name.dfs.core.windows.net/filename.csv')

=============================================================================
Date:18th Feb 2025
Day-02
=================
Q10) Transformations that we will use here are mentioned below ?

Sol:

 from pyspark.sql.functions import *
 from pyspark.sql.types import *

 #converting a column into a list separated by space
 df2 = df1.withColumn('Item_Type',split(col('Item_Type'),' '))

#adding a new flag column having constant value
 df3 = df2.withColumn('flag',lit('yes')) 

#change the datatype of a column
 df4 = df3.withColumn('ItemVisiblity',col('ItemVisiblity').cast(StringType()))

============================================================================
Q11) How to access one notebook from another notebook?
Sol: We just need to use %run '/workspace_name/notebook_1' 
        so now we are able to access notebook_1 from notebook_2

============================================================================
Q12) How to write the transformed data into sink in delta lake format?

Sol: df4.write.format('delta')\
             .mode('append/overwrite/errorIfExists/ignore')\
             .option('path','abfss://sink@dataamritlake.dfs.core.windows.net/destination/')\
             .save() 
===========================================================================
Q13) What is the difference between Managed delta tables vs External Delta tables in databricks?

Sol: 
	• When we create any delta tables in databricks by default it is managed delta tables and in this delta table the metadata is stored inside the metastore and real data gets stored inside the default cloud storage account of azure and if we delete or drop this managed table both metadata and date gets deleted 

	• On the other hand in case of external delta tables only metadata gets deleted but the real data will be available as the data is stored externally and not managed by databricks

%sql
 CREATE DATABASE salesDB;
 
%md
##Creation of delta table

%sql
 CREATE TABLE salesDB.mantable
 (
  id INT,
  name STRING,
  marks INT
 )
 USING DELTA;

%md
##Creation of External Delta Tables

CREATE TABLE salesDB.exttable
 (
  id INT,
  name STRING,
  marks INT
 )
 USING DELTA
 LOCATION '/path/to/storage/location'
================================================================================================================
 Q14) How to know all the changes that has been done on the delta tables?
Sol:

DESCRIBE HISTORY table_name; #to know all the changes that have been done on delta table
 
RESTORE TABLE table_name TO VERSION AS OF 2; #It will restore to the version 2 of the table

 VACUUM Table_name ; #to delete the version of delta tables that we do not want and those are more than 7 days

 OPTIMIZE table_name; #it basically skip data and helpful for faster reads 

 OPTIMIZE table_name ZORDER BY id; #zorder indexing

 ![image](https://github.com/user-attachments/assets/b4ef25a7-1e4b-4127-9054-941180300701)























