Level 50 - 360 view of Synapse.  

This walkthrough gives you an overview of Synapse as tool to manage your data requirements end-to-end.  It touches on the following: 
* Setting up the strucutred and unstructure data stores
* Setting up the SQL and Spark pools, think of this as the Compute
* Setting up source control
* Loading data to the database and the datalake
* Basic how to on Exploratory Data Analysis using Notebooks
* Basic integration workflow to copy data
* Power BI report to view results inside Synapse WS
* AutoML experiment to enrich data, also inside Synpase WS
* Initial data governance with Purview

# Setting up the strucutred and unstructure data stores
**This step takes about 30 mins**
* Create a single Resource Group to hold the resources
* Follow all steps on [this](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-create-workspace) page (15 mins to create create ws, 15 mins to create the SQL and Spark pools)


## Setting up Source Control
* Log on to [Azure DevOps](https://dev.azure.com/)
* If you have an organization, use it, else, create one: On the left **New Organization**. Give a name to your orgnaization and a region to host the code in.  Click **Continue** 
* Create a project, and set a visibility level that works for you.  Click on **Create Project**
* In Synapse, go to **Manage** > **Source Control** > **Git configuration**.  Click **Configure** in the middle of the screen.  
* In **Repository type** select **Azure DevOps Git**.  For **Azure Active Directory** select your directory.  Click **Continue**.  
* Select **Select Reporsitory**, and pick the **Azure DevOps organization name** that you have or created in the steps above.  Select the **Project Name** and **Repository Name**.  In the **Collaboration branch**, click **Create New**, and in the **Branch name** enter **dev**.  
* Click **Apply**
* In the **Set Working Branch**, select **Use Existing** and from the drop down list select **dev**.  

## Load 2 Million Rows of NYC Taxi Data
* Open Synapse Studio
* Navigate to the **Develop** hub, click the + button to add a new resource, then **create new SQL script**, and enter the following code: 
``` sql
CREATE TABLE [dbo].[Trip]
(
    [DateID] int NOT NULL,
    [MedallionID] int NOT NULL,
    [HackneyLicenseID] int NOT NULL,
    [PickupTimeID] int NOT NULL,
    [DropoffTimeID] int NOT NULL,
    [PickupGeographyID] int NULL,
    [DropoffGeographyID] int NULL,
    [PickupLatitude] float NULL,
    [PickupLongitude] float NULL,
    [PickupLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DropoffLatitude] float NULL,
    [DropoffLongitude] float NULL,
    [DropoffLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [PassengerCount] int NULL,
    [TripDurationSeconds] int NULL,
    [TripDistanceMiles] float NULL,
    [PaymentType] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [FareAmount] money NULL,
    [SurchargeAmount] money NULL,
    [TaxAmount] money NULL,
    [TipAmount] money NULL,
    [TollsAmount] money NULL,
    [TotalAmount] money NULL
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
);

COPY INTO [dbo].[Trip]
FROM 'https://nytaxiblob.blob.core.windows.net/2013/Trip2013/QID6392_20171107_05910_0.txt.gz'
WITH
(
    FILE_TYPE = 'CSV',
    FIELDTERMINATOR = '|',
    FIELDQUOTE = '',
    ROWTERMINATOR='0X0A',
    COMPRESSION = 'GZIP'
)
OPTION (LABEL = 'COPY : Load [dbo].[Trip] - Taxi dataset');
```
* Click the **Run** button.  


<g>**INFO**: This should take a little less than 60 seconds.  </g>



# Data Exploration (A few Scenarios)
## Scenario 1: Data in SQL Pool (Structured world) and we want to analyze it
* Let’s have a look at the data inside using our regular T-SQL, and try to also look at some statistics.  (steps 1 – 7 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-sql-pool#explore-the-nyc-taxi-data-in-the-dedicated-sql-pool))
## Scenario 2: Use an open dataset, and just explore o the fly with Spark.  
* Now lets look at our Spark world.  What we want to do is loan an open dataset and work on it on the fly – just to explore (steps 1 – 8 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-spark#analyze-nyc-taxi-data-in-blob-storage-using-spark))
## Scenario 3: Create a table in Spark using data from the SQL Pool.  
* We want to leverage the data in our structured database in Synapse; however, we want to analyze from Spark, but, we want to persist it as well there, so we will create a database in Spark as well.  (steps 1 – 6 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-spark#load-the-nyc-taxi-data-into-the-spark-nyctaxi-database))
* Now that the data is in Spark, lets see how we can use regular SQL in Spark to produce the same summary as in Scenario 1 in this document.  (steps 1 – 5 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-spark#analyze-the-nyc-taxi-data-using-spark-and-notebooks))
* Now the summary data is stored as another table in the Spark database (Spark pool)
## Scenario 4: Create a table in the SQL pool from a Spark table.  
* This is just to cover the final remaining direction, which is to move data from your Spark Database to your SQL Pool.  
    * Create a new code cell and enter the following code. Run the cell in your notebook. It copies the aggregated Spark table back into the dedicated SQL pool table.
    ``` python
    %%spark
    val df = spark.sql("SELECT * FROM nyctaxi.passengercountstats")
    df.write.sqlanalytics("SQLPOOL1.dbo.PassengerCountStats", Constants.INTERNAL )
    ```
## Scenario 5: Use the serverless SQL Pool to query data in a remote (linked) data lake (blob storage).  
* The Serverless SQL Pool is there by default, and its special, you can use it to run T-SQL on unstructured data in a datalake, that doesn’t necessary need to be loaded in your environment.  (Steps 1 – 3 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-sql-on-demand#analyze-nyc-taxi-data-in-blob-storage-using-serverless-sql-pool))
## Scenario 6: Use the serverless SQL Pool to analyze data in the Spark Database: 
* Another cool think about the Serverless SQL Pool is that it could be used to run queries on top of the spark databases, not just the SQL databases.  (Steps 1 – 3 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-storage#analyze-data-in-a-storage-account-1), also try Steps 1 - 3 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-sql-on-demand#analyze-nyc-taxi-data-in-spark-databases-using-serverless-sql-pool))
## Scenario 7: Analyze Data in a Data Lake 
* Create CSV and Parquet files in your storage account by running the following code in a notebook in a new code cell.  
``` python
%%pyspark
df = spark.sql("SELECT * FROM nyctaxi.passengercountstats")
df = df.repartition(1) # This ensure we'll get a single file during write()
df.write.mode("overwrite").csv("/NYCTaxi/PassengerCountStats_csvformat")
df.write.mode("overwrite").parquet("/NYCTaxi/PassengerCountStats_parquetformat")
```
* Explore simple queries using pyspark and SQL on the data now in you data lake using Steps 1 - 8 [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-analyze-storage)

# Integration
## Create a dataset
In the Synapse workspace, click on **Linked services** then click on **New**
* Search for **Azure Data Lake Storage Gen2**, click **Continue** at the bottom
* **Name**: LSDataLake
* **Connect via integration runtime**: AutoRsolveIntegrationRuntime
* **Authentication method**: Account Key
* **Account selection method**: From Azure subscription
* **Azure subscription**: select your subscription
* **Storage Account Name**: select the data lake you created above
* **Test Connection**: **To linked service**
* Click **Test Connection** 
* Click **Create**

## Create a Pipeline
In the Synapse workspace, click on **Integrate** on the left.  Next to the word **Integrate**, click on the New button and select **Copy Data Tool**
* **Properties > Task Name**: QuickCopyActivity
* **Properties > Task cadence or task schedule**: Run once now
* **Source > Connection**: LSDataLake
* **Source > Dataset > File or Folder** click **Browse** > double click **Users** > double click **NYCTaxi** > **PassengerCountStats_csvformat** > **part-0000-..**. (You should find a file with a long identifier), then click **Choose**
* **Source > Dataset > Binary** copy: Unchecked
* **Destination > Connection > SQLPOOL1**
* Keep clicking **next** until the last page, where the copy activity runs successfully.  
* Click **Finish** to close the tool
* Under **Pipelines**, click on the **QuickCopy** pipeline just created, and click on the Copy Data activity on the canvas to see its details.  

# Visualize with Power BI

## Create a Power BI workspace
Run all the sections on [this](https://docs.microsoft.com/en-us/azure/synapse-analytics/get-started-visualize-power-bi) page.  

# Trying out Machine Learning
## Create a Machine Learning Service Workspace

* In the resource group you created, Click **Add**, and search for **Machine Learning**.  Click Create.  Enter the following details: 
    * Workspace Name: mlworkspace
    * Region: pick a region
    * Leave the rest of the fields as default
    * If Container Registry is not populated with a new one automatically, click **Create New** and enter a unique name.  Click **Save**
    * Click **Review + create**, then **Create**
    * Leave it a few minutes to provision.  

## Create a new linked Service for the ML Service

<r>In the section **Create a service principal**, step 5, the service princial will already be created.  click on it, and copy the **Application ID**</r>

## Train an AutoML model with no-code
Follow the steps [here](https://docs.microsoft.com/en-us/azure/synapse-analytics/machine-learning/tutorial-automl)

<o>This takes longer to run</o>


# Governance of Data
https://docs.microsoft.com/en-us/azure/synapse-analytics/catalog-and-governance/quickstart-connect-azure-purview



# Git integration


# Integration Runtime
