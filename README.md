## Real Time Sync from MongoDB Atlas to Azure Synapse using Spark Streaming
## Background:
Azure Synapse is used by multiple customers as a one stop solution for their analytical needs. Data is ingested from disparate sources into Synapse Dedicated SQL Pools (EDW) and SQL, AI/ ML, Batch, Spark based analytics can be performed and data is further visualised using tools like Power BI. 

MongoDB has both a [Source and Sink connector](https://learn.microsoft.com/en-us/azure/data-factory/connector-mongodb?tabs=data-factory) for Synapse Pipelines which enables fetching data from MongoDB or loading data into MongoDB in batches/ micro batches.

However, currently there is no CDC connector for MongoDB in Synapse which can keep the Synapse Dedicated SQL pools synced with MongoDB data in real time. To facilitate real time analytics, MongoDB with Microsoft provided a custom solution and gave it as a few clicks and configuration based deployment as detailed [here](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/analytics/azure-synapse-analytics-integrate-mongodb-atlas). There is however still a need to provide a more seamless integration for Real time sync from MongoDB to Synapse and spark streaming connector from MongoDB can be of help here.

## Solution Overview:
MongoDB’s latest [Spark connector v10.1](https://www.mongodb.com/blog/post/introducing-mongodb-spark-connector-version-10-1)  provides [streaming capabilities](https://www.mongodb.com/docs/spark-connector/current/structured-streaming/) which allows streaming of changes from MongoDB or to MongoDB in both continuous and micro-batch modes. This eliminates the need for an Appservice to keep running and listening to the changes in the MongoDB collection and trigger Pipelines in Synapse to update the Dedicated SQL as and when changes are captured. All we need is a small piece of code that reads a stream of changes from MongoDB collection and writes to the ADLS gen2 in Synapse in a table format which can be queried like any other tables. The code is packaged as a Pipeline template and the User just needs to Import the pipeline, give the parameters to point to the MongoDB collection and table name in ADLS Gen2 and trigger the Pipeline.

Here are the steps you can follow to stream data from MongoDB to Azure Synapse using the Spark based Pipeline template:

## General Pre-requisites:
You will need the below set up before starting the Lab:
1. MongoDB Atlas cluster setup:   
Register for a new Atlas Account [here](https://www.mongodb.com/docs/atlas/tutorial/create-atlas-account/#register-a-new-service-account).   
Follow steps from 1 to 4 (*Create an Atlas account*, *Deploy a Free cluster*, *Add your IP to the IP access list* and *Create a Database user*) to set up the Atlas environment.   
Also, follow step 7 “*Load Sample Data*” to load sample data to be used in the lab.

![Picture 2](https://user-images.githubusercontent.com/104025201/230300219-6f95d9be-616f-4267-8cce-e4d3af5d1411.png)

**Note: For this lab add “0.0.0.0/0” to the IP access list so that Synapse can connect to MongoDB Atlas. In Production scenarios, it is recommended to use Vnet Peering or Private Link options.**

2. Azure account setup:    
Follow link [here](https://azure.microsoft.com/en-in/free/) to set up a free azure account

3. Azure Synapse Analytics Workspace setup:   
Follow link [here](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-create-workspace) to set up a Synapse workspace within you Azure account
## Pre-requisites specific to Spark streaming:
#### 1. Set up an Azure Spark Pool: 
Follow the instructions in the [Azure documentation](https://learn.microsoft.com/en-us/azure/synapse-analytics/quickstart-create-apache-spark-pool-studio) to set up an Azure Spark pool. This will allow you to run Spark jobs on Azure. You should have the Owner and required permissions in the Synapse workspace to run Spark Jobs.


#### 2. Install MongoDB Connector for Apache Spark: 
  - Download the latest version of MongoDB connector for Apache spark jar file from [Maven Central](https://repo1.maven.org/maven2/org/mongodb/spark/).

    ***Note: Make sure the package has same version of Scala as the Scala version of the Spark Pool.***  
  - Login to Azure Synapse Studio > Click on Manage in the left bar.
  - Click on Workspaces Packages > Upload, Upload MongoDB Spark Connector jar file which you downloaded earlier.

      ![image](https://user-images.githubusercontent.com/114057324/216914964-3598dae4-5041-4efb-9124-837a3c6895d7.png)

#### 3. Associate the package to your Spark Pool: 
  - Click on Apache Spark pools > options of your apache spark instance > Packages

    ![image](https://user-images.githubusercontent.com/114057324/216915583-4cc6a97c-abb8-4729-9f14-813cf65e28f0.png)


  - Under the Workspaces Packages tab, click on the Select from workspace packages option and choose the MongoDB Connector jar file which you uploaded earlier from the list.

    ![image](https://user-images.githubusercontent.com/114057324/216915613-b8f03aa7-33e9-4173-adfc-de97b31fca46.png)

  - Click Apply, this successfully installs the MongoDB Connector in Apache Spark pool instance.

## Workflow Steps:   
1. [Import Pipeline](#import-pipeline)  
2. [Provide the Configuration parameters](#provide-the-configuration-parameters)  
3. [Trigger the Pipeline](#trigger-the-pipeline)

#### Import Pipeline
  - Click on Integrate and import the [zip](https://github.com/mongodb-partners/Synapse-Spark-Streaming/blob/main/MongoDB%20Spark%20Streaming.zip) of the pipeline as shown. This zip file contains the Spark code required to enable streaming from MongoDB to Synapse.

	  ![image](https://user-images.githubusercontent.com/114057324/216915692-97beeceb-ea14-43ca-9199-4155466a6053.png)

		
#### Provide the Configuration parameters
  - Once imported successfully, click on the pipeline and update the below variables available under Settings > Base parameters.
  
    ![image](https://user-images.githubusercontent.com/114057324/216915949-363faae1-5c18-4e06-8580-fcab6886c488.png)

The parameters are:
````
connectionString : The connection uri for the MongoDB cluster to connect to
database : The database that the in the MongoDB cluster 
collection : The collection that needs to be streamed into Synapse
tablename : The tablename that we want to have created in the ADLS Gen2 default storage to hold the streamed data
````

#### Trigger the Pipeline

  - You can start the pipeline using Trigger Now, and run it indefinitely. All the changes will stream to ADLS Gen2 near-real time once the trigger is started.

    ![image](https://user-images.githubusercontent.com/114057324/216916399-d52aab8a-33c2-4ad9-be3e-7c96894edb56.png)


  - Now the Pipeline will keep running and keep streaming data from your MongoDB collection into Synapse ADLS Gen 2 table.

    ![image](https://user-images.githubusercontent.com/114057324/216916441-4a0d7f08-3354-4314-adda-073f328045c8.png)

        
#### Congratulations ! You have set up streaming from your operational database in MongoDB Atlas to Synapse ADLS Gen2 in less than 10 mins !

  - In order to stop the pipeline, go to Monitor >  Pipeline runs and click Cancel (shown below) on the pipeline you want to stop.

    ![image](https://user-images.githubusercontent.com/114057324/216916502-e82a1b9a-c1e0-4d13-9dd5-da372a019a25.png)


## Summary
This gives a complete guide to set-up and run Spark streaming in order to pull data from MongoDB in near real time.
For any further information, please contact partner-presales@mongodb.com
