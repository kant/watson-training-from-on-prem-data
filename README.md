# Continuously train a cloud-based machine learning model from an on-premise database

### Loading Data from a csv to a DB2 database

1. Download the [CSV](https://github.com/IBMDataScience/buildings_blog/blob/master/buildings_data_17.csv).

1. Clean your data- I have separated data by `violation_date` Year

1. Create database, connect to user

   > Watson studio does not display admin schema objects- Log in as a non-admin user

1. Connect using new user-

1. Useradd newuser

1. PASSWD passw0rd

1. Grant connect,load on database to user newusers

1. DB2 connect to database user newuser

1. Create table

1. Load the table

### Connect to DB2 on-prem from Watson Studio

[Configure a secure gateway](https://console.bluemix.net/docs/services/SecureGateway/index.html#getting-started-with-sg)

[console.bluemix.net/catalog/services/secure-gateway](https://console.bluemix.net/catalog/services/secure-gateway)

Download the secure gateway client for your machine- (I used Ubuntu Linux)

#### To install the client

    dpkg -i ibm-securegateway-client-1.8.0fp6+client_amd64.deb

Update the config file with the gateway ID and token that you see on your
secure gateway service:

    vi /etc/ibm/sgenvironment.conf

#### Start the client

    /usr/bin/sudo /bin/systemctl start securegateway_client

Add the IP address to the Secure gateway's Access control list.

#### Configure the Destination

Use the GUI to add the IP address, Port number, choose TCP/IP , No
authentication. If you see a Green heartbeat on your gateway, It is up and
running!

Connect to DB2 from Watson Studio

#### On Watson Studio

Add to Project-> Connection-> IBM DB2

Enter the hostname, port, Db name, user name, password, select the secure
gateway

Add to Project-> Connected asset-> select the table from the DB

> Watson Machine Learning models do not support Data assets from DB2 on-prem,
> so we now have to convert the Data asset into a csv

#### Refine the asset

Data Assets-> Refine-> Add operation-> run

It is now saved as a csv

#### Create a WML model

Associate a ML service-

Associate an IBM analytics service- (Spark)

Select the Type of classification, data asset, Input and output features,
   estimators and the final model.

#### Deploy the model

Use these endpoints in your notebook on new data.

### Continuous Learning: Re-training the model

#### Configure Performance Monitoring:

> Watson Studio only supports DB2 Watson on Cloud tables as Feedback tables

#### On DB2 Warehouse on Cloud

Create a Row-organized table (Feedback table)

Load data into your table

#### Performance Monitoring

Select the feedback metric, the feedback table, Trigger event (Eg: after 50
rows are added to the table)

When the Trigger event occurs, It will pull in new data from the Feedback table
and re-train your model. If the new model performs better, this will be
deployed.

> After Watson Studio uses the Feedback table, it writes a column `_TRAINING`
> into the Feedback table, with Timestamp.

This column has a not null constraint. To load new data into the Feedback table-

1. Add a column called `_TRAINING` in your dataset

1. Alter the table and remove the NOT NULL constraint from the column

## Commands

### NON-admin user:

    Useradd newuser
    PASSWD newuser
    GRANT connect,load on WML_DB to user newuser

    db2 "CREATE TABLE violations_2018(ID INTEGER,VIOLATION_CODE VARCHAR(20),INSPECTOR_ID VARCHAR(15),INSPECTION_STATUS VARCHAR(10),INSPECTION_CATEGORY VARCHAR(10),DEPARTMENT_BUREAU VARCHAR(30),ADDRESS VARCHAR(250),LATITUDE DOUBLE,LONGITUDE DOUBLE)"

    db2 "load from /home/ibm_admin/Desktop/Shruthi/violations_2018.csv of DEL replace into Violations_2018"


### On Db2 Warehouse on Cloud

    CREATE TRIGGER feedback_trigger NO CASCADE BEFORE INSERT ON violations_feedback REFERENCING NEW AS n FOR EACH ROW SET n."_training"=CURRENT_TIMESTAMP

    CREATE TABLE violations_feedback(ID INTEGER,VIOLATION_CODE VARCHAR(20),INSPECTOR_ID VARCHAR(15),INSPECTION_STATUS VARCHAR(10),INSPECTION_CATEGORY VARCHAR(10),DEPARTMENT_BUREAU VARCHAR(30),ADDRESS VARCHAR(250),LATITUDE DOUBLE,LONGITUDE DOUBLE,"_TRAINING" TIMESTAMP NOT NULL) ORGANIZE BY ROW

    Alter table violations_feedback alter column "_TRAINING" set NOT NULL
