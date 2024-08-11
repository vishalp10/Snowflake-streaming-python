# snowflake-streaming-api using python

### Introduction: 

In today's data-driven world, real-time analytics and insights have become crucial for businesses. To facilitate this, organizations often need to stream data from various sources into a central data warehouse. However, building infrastructure and code for streaming pipelines can be very challenging due to the complex nature of streaming architectures, which often involves integrating with tools such as Apache Kafka, Flink, Spark, etc., as well as building the expertise to write high-quality code to integrate and manage these technologies.

In this article, we will explore the process of streaming data from MySQL to the Snowflake Data Cloud platform using Snowflake's Streaming Ingest API SDK. This new Streaming Ingest SDK from Snowflake is built on Java, so it requires the user to have expertise in Java programming to build an app that can stream data directly into Snowflake. (https://javadoc.io/doc/net.snowflake/snowflake-ingest-sdk/latest/index.html). However, in this guide, I'll provide you with a step-by-step approach to achieve seamless and efficient data streaming into Snowflake using Python wrapper classes and functions that will make calls to the Java API functions.

For additional background, please refer to my article on [medium](https://medium.com/@vishalrp/real-time-streaming-into-snowflake-using-python-snowflake-streaming-api-c7900d2c8d74)


#### Current Architecture of My Python application:
![image](https://github.com/user-attachments/assets/f2fe8132-c336-4648-8751-210d2b5281bf)


#### Potential Future State Architecture:
![image](https://github.com/user-attachments/assets/5a780a05-2a91-4188-9157-a1750a963b4b)


#### Prerequisites for running this notebook:
Before executing the notebook, ensure that you have the following prerequisites in place:
1. A working MySQL database with the data you want to stream (This can be any database/datawarehouse that supports ODBC connections).
2. A Snowflake account with the necessary privileges to create and manage objects.
3. Anaconda environment with Python version 3.8. 

#### Step 1: Set up the Snowflake Account and Database:
To begin, make sure you have a Snowflake account. If you don't have one, sign up for a free trial or set up a production account. Once you have access to Snowflake, create a database, a schema and tables to which you'll stream the MySQL data. Following are examples of databases, schemas and tables that I’ve used in this ingestion workflow - 

![image](https://github.com/user-attachments/assets/c89ccb80-8031-4f1c-8e15-8e7fb2e3cc42)

My source data for this POC is MySQL hosted on Azure and I had uploaded some sample people data into a database snowpark_demodb. Following is the schema and some sample records from my source table in MySQL database.

![image](https://github.com/user-attachments/assets/7a0250b3-22bf-4506-bc44-db46a04a5a24)

![image](https://github.com/user-attachments/assets/9f751e2a-958f-4236-87fe-f48776567804)


#### Step 2: Create a Conda environment with Python 3.8:
Create an anaconda python environment with python v3.8 and install the following libraries using the anaconda channel –

```
conda create --name snowpark_java -c https://repo.anaconda.com/pkgs/snowflake python=3.8
conda activate snowpark_java
```

Next install Snowpark for python, Pyjnius, mysql-connector-python & phonenumbers python libraries

```
conda install -c https://repo.anaconda.com/pkgs/snowflake snowflake-snowpark-python pandas
conda install  pyjnius, mysql-connector-python, phonenumbers
```

#### Step 3: Update the “snowflake_env_login_creds.json” file with your snowflake platform credentials and source database credentials:
The main notebook reads the configurations for source database and destination snowflake database from a json config file. Also, the authentication from your python program to Snowflake Database is performed using key pair authentication so you will need to generate a private & public key and register them in your snowflake account to securely connect using the API. Please use the following link to generate and configure key pair authentication and configure your private key in the json config file (https://docs.snowflake.com/en/user-guide/key-pair-auth#configuring-key-pair-authentication)


#### Step 4: Configure path to JAR files required to use the Streaming API SDK:

![image](https://github.com/user-attachments/assets/368b8a38-a40b-4958-abaf-9b62cab840dd)

#### Step 5: Set up MySQL Database Connection:
Next, establish a connection to your MySQL database. You can use any Python library that supports MySQL, such as `mysql-connector-python`, to connect to MySQL. Ensure that you have the necessary credentials (host, port, username, password, database name) to connect to the MySQL database.

![image](https://github.com/user-attachments/assets/3dd38a94-e61f-496b-a4a5-5a44ebc5bddb)

#### Step 6: Set up the Snowflake Streaming Ingest API Connection:
In order to use the Snowflake Streaming Ingest API, you first need to authenticate and establish a secure connection to Snowflake. Follow these steps below to set up a connection and to create a secure client to stream data directly into a Snowflake table.
 
![image](https://github.com/user-attachments/assets/98f51aae-b92c-4f02-9c73-bf921b673748)
 
#### Step 7: Prepare a payload for streaming:
The payload that is streamed into snowflake needs to be packaged as a Java hashmap, so create a hashmap that contains values for all the columns in the destination table as shown below – 
 
![image](https://github.com/user-attachments/assets/ce21fc1f-7864-4cf4-b9b1-f77d62d00b42)
  
Snowflake’s Streaming API supports both single row inserts and bulk inserts

#### Next, we will use Snowpark to perform some data completeness check and to transform our data into gold tables

#### Step 8: Use Snowpark to perform data quality checks:
Connect to your Snowflake account using Snowpark and validate that all the records from the source MySQL table were successfully ingested into the destination table in Snowflake.

![image](https://github.com/user-attachments/assets/42f727f4-3e07-49c0-a0a5-9dfeaed9acfe)

#### Step 9: Transform the raw data and create gold tables using Snowpark:
In my ingestion workflow, I am ingesting customer_id, email, phone number, and date of birth information into Snowflake. The phone number data needs some formatting, and the date of birth column needs to be converted from a varchar to a date type.

To transform the phone column into a standard format, I created a Python UDF called "parse_phone_no" and uploaded this function to a stage area within my Snowflake database. The code for uploading the UDF has been commented and is saved within the main notebook.

![image](https://github.com/user-attachments/assets/908612a5-fe3f-48fd-a6a5-0e0c2c6d81cb)

I then call this UDF in my Snowpark SQL command to transform the raw data into a final gold table, which can be used for downstream analytics.

![image](https://github.com/user-attachments/assets/a004c3f0-7e2a-44d3-b3e7-2704c4c540b7)

**The entire process, including extraction of 500K records from MySQL, loading to Snowflake, and transformation, took approximately 60 seconds to complete on my local laptop**. However, please note that I was reading data from an Azure cloud database to my local machine and then ingesting it into a Snowflake database on AWS. If this code is executed inside a container on Azure, the total execution time might be in single-digit seconds.

### Conclusion:
Streaming data to Snowflake using the Snowflake's Streaming Ingest API SDK enables organizations to maintain real-time analytics and insights. By following the steps outlined in this article, you can now build near real-time ELT pipelines using Python and Snowflake's Streaming Ingest API.


