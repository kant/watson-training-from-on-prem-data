# Continuously train a cloud-based machine learning model from an on-premise database

TODO

![Architecture](doc/source/images/architecture.png)

## Flow

1. Source data is retained in on-premise Db2 database.
1. Data is accessible to Watson Studio via a Secure Gateway.
1. Secure gateway is utilized to train a cloud-based machine learning model.
1. Training feedback is stored in Db2 Warehouse for continuous learning.

## Included components

* [IBM Db2 Warehouse on
  Cloud](https://www.ibm.com/cloud/db2-warehouse-on-cloud): An elastic,
  fully-managed cloud data warehouse service that is powered by IBM BLU
  Acceleration technology for increased performance and optimization of
  analytics at a massive scale.
* [IBM Watson Studio](https://www.ibm.com/cloud/watson-studio): Build and train
  AI models, and prepare and analyze data, in a single, integrated environment.

## Steps

1. [Create a Watson Studio instance](#create-a-watson-studio-instance)
1. [Create a Db2 Warehouse on IBM Cloud](#create-a-db2-warehouse-on-ibm-cloud)
1. [Load sample data into an on-premise Db2 database](#load-sample-data-into-an-on-premise-db2-database)
1. [Configure a secure gateway to IBM Cloud](#configure-a-secure-gateway-to-ibm-cloud)
1. [Connect to on-premise Db2 database from Watson Studio](#connect-to-on-premise-db2-database-from-watson-studio)
1. [Create a machine learning model](#create-a-machine-learning-model)
1. [Enable continuous learning](#enable-continuous-learning)

### Create a Watson Studio instance

In order to build and train your machine learning model, you'll need to [sign
up for Watson Studio](https://www.ibm.com/cloud/watson-studio), which you can
do for free.

### Create a Db2 Warehouse on IBM Cloud

Next, you'll need to [create a Db2 Warehouse
instance](https://console.bluemix.net/catalog/services/db2-warehouse). The size
of the dataset used in this pattern will not incur billing.

### Load sample data into an on-premise Db2 database

As a NON-admin user:

    Useradd newuser
    PASSWD newuser
    GRANT connect,load on WML_DB to user newuser

    db2 "CREATE TABLE violations_2018(ID INTEGER,VIOLATION_CODE VARCHAR(20),INSPECTOR_ID VARCHAR(15),INSPECTION_STATUS VARCHAR(10),INSPECTION_CATEGORY VARCHAR(10),DEPARTMENT_BUREAU VARCHAR(30),ADDRESS VARCHAR(250),LATITUDE DOUBLE,LONGITUDE DOUBLE)"

    db2 "load from /home/ibm_admin/Desktop/Shruthi/violations_2018.csv of DEL replace into Violations_2018"

1. Download the [CSV](https://github.com/IBMDataScience/buildings_blog/blob/master/buildings_data_17.csv).

1. Clean your data- I have separated data by `violation_date` Year

1. Create database, connect to user

   > Watson studio does not display admin schema objects- Log in as a non-admin user

1. Connect using new user-

1. Useradd newuser

1. PASSWD passw0rd

1. Grant connect,load on database to user newusers

1. Db2 connect to database user newuser

1. Create table

1. Load the table

### Configure a secure gateway to IBM Cloud

[Configure a secure gateway](https://console.bluemix.net/docs/services/SecureGateway/index.html#getting-started-with-sg)

[console.bluemix.net/catalog/services/secure-gateway](https://console.bluemix.net/catalog/services/secure-gateway)

Download the secure gateway client for your machine- (I used Ubuntu Linux)

#### To install the client

    dpkg -i ibm-securegateway-client-1.8.0fp6+client_amd64.deb

Update the config file with the gateway ID and token that you see on your
secure gateway service:

    vi /etc/ibm/sgenvironment.conf

`sgenvironment.conf`:

    #!/bin/bash
    #
    # Stop and Restart the client during install or upgrade (Yes or No)
    # If manually modifying the following, accepted values are: Yes, yes, No, or no
    RESTART_CLIENT=No

    # Configuration ID to connect
    # If manually modifying the following, accepted values are: "<your gateway ids>" separated by spaces

    GATEWAY_ID=$YOUR_GATEWAY_ID
    export SECGW_GATEWAYID="$GATEWAY_ID"

    # Security Token for this Configuration ID (if any)
    # If manually modifying the following, accepted values are: <your gateway id security tokens> ('none' if no token, separated by '--' otherwise)
    SECTOKEN=$YOUR_GATEWAY_ID_SECURITY_TOKEN

    # Access Control List File
    # If manually modifying the following, accepted values are the absolute path to your ACL files ('none' if no file, separated by '--' otherwise)
    ACL_FILE=/opt/ibm/securegateway/client/securegw_acl.txt

    # Logging Level
    # If manually modifying the following, accepted values are: INFO, DEBUG, ERROR, TRACE (default value is INFO. Separate with '--')
    LOGLEVEL=INFO

    # Client UI port
    # If manually modifying the following, set USE_UI to N if you dont want to launch the client UI. Default value for client port is 9003.
    USE_UI=Y
    UI_PORT=9003

    # Language
    # Default is en; acceptable values are en, de, es, fr, it, ja, ko, pt-BR, zh, zh-TW
    LANGUAGE=en

    SECGW_ARGS="--no_license --l $LOGLEVEL --service"

    #
    # Add arguments only if set
    #
    if [ 'X_'$SECTOKEN != 'X_' ]; then
        SECGW_ARGS="$SECGW_ARGS --t $SECTOKEN "
    fi
    if [ 'X_'$ACL_FILE != 'X_' ]; then
        SECGW_ARGS="$SECGW_ARGS --F $ACL_FILE "
    fi
    if [ 'X_'$USE_UI == 'X_N' ] ; then
        SECGW_ARGS="$SECGW_ARGS --noUI "
    elif [ 'X_'$UI_PORT != 'X_' ]; then
        SECGW_ARGS="$SECGW_ARGS --port $UI_PORT "
    fi

    #
    # Export the arguments
    #
    export SECGW_ARGS

    export OPERSYS=Ubuntu
    export VERSION=18

`/opt/ibm/securegateway/client/securegw_acl.txt`:

    acl allow 9.236.239.165:50000
    acl allow 127.0.1.1:50000

#### Start the client

    /usr/bin/sudo /bin/systemctl start securegateway_client

Add the IP address to the Secure gateway's Access control list.

#### Configure the Destination

Use the GUI to add the IP address, Port number, choose TCP/IP , No
authentication. If you see a Green heartbeat on your gateway, It is up and
running!

### Connect to on-premise Db2 database from Watson Studio

Add to Project-> Connection-> IBM Db2

Enter the hostname, port, Db name, user name, password, select the secure
gateway

Add to Project-> Connected asset-> select the table from the DB

> Watson Machine Learning models do not support Data assets from Db2 on-prem,
> so we now have to convert the Data asset into a csv

#### Refine the asset

Data Assets-> Refine-> Add operation-> run

It is now saved as a csv

### Create a machine learning model

#### Create a WML model

Associate a ML service-

Associate an IBM analytics service- (Spark)

Select the Type of classification, data asset, Input and output features,
   estimators and the final model.

#### Deploy the model

Use these endpoints in your notebook on new data.

### Enable continuous learning

#### Configure Performance Monitoring:

> Watson Studio only supports Db2 Watson on Cloud tables as Feedback tables

#### On Db2 Warehouse on Cloud

Create a Row-organized table (Feedback table)

Load data into your table

    CREATE TRIGGER feedback_trigger NO CASCADE BEFORE INSERT ON violations_feedback REFERENCING NEW AS n FOR EACH ROW SET n."_training"=CURRENT_TIMESTAMP

    CREATE TABLE violations_feedback(ID INTEGER,VIOLATION_CODE VARCHAR(20),INSPECTOR_ID VARCHAR(15),INSPECTION_STATUS VARCHAR(10),INSPECTION_CATEGORY VARCHAR(10),DEPARTMENT_BUREAU VARCHAR(30),ADDRESS VARCHAR(250),LATITUDE DOUBLE,LONGITUDE DOUBLE,"_TRAINING" TIMESTAMP NOT NULL) ORGANIZE BY ROW

    Alter table violations_feedback alter column "_TRAINING" set NOT NULL

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
