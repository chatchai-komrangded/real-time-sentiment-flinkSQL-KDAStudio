# Real-time sentiment analysis on customer feedback

## Reference architecture

![kda1](/images/kda1.PNG)

## Create a Kinesis Data Analytics Studio notebook
1. Go to Kinesis Data Analytics Console: console.aws.amazon.com/kinesisanalytics
2. Click on Studio tab
3. Click on create Studio notebook
4. Choose "Quick create with sample code" as create method
5. Enter a notebook name
6. For AWS Gluedatabse click on the refresh button and select Default Glue database. If the list is still empty, create a new Glue database.
7. Notedown the IAM role name. Click on Create Studio Notebook.

![kda1](/images/kda2.png)


## Configure IAM
1. Go to IAM: https://console.aws.amazon.com/iam
2. Click on role and search for the role that KDA Studio has created earlier
3. Click on Attach Policies and add Administrator Access. (This is not recommended for your production workload.)

![kda1](/images/kda3.png)

## Working with Kinesis Data Analytics Studio
1. Go to Kinesis Data Analytics Console: console.aws.amazon.com/kinesisanalytics
2. Click on Studio tab and select the notebook you have created in the previous step
3. Click on Run and Click on Open in Apache Zeppelin once the statue of the Notebook is running
![kda4](/images/kda4.png)

## Working with Kinesis Data Analytics Studio - create prerequisite notebook
1. In the notebook console create a new note
2. enter a name of your notebook- "prerequisite"
3. Select Default Interprete as Flink and create the notebook
![kda5](/images/kda5.png)

** We are going to use the notebook to provisioned somes AWS resources, for example DynamoDB table, Kinesis Data Streams etc. For that we are using boto3. 
4. execute the following code to install boto3
```
%flink.ipyflink

pip install boto3
```

5. Create a new paragraph and execute the below code. This will create a DynamoDB table in ap-southeast-2
```
%flink.ipyflink
#create table innovate_latlon
import boto3
region='ap-southeast-2'
dynamodb = boto3.resource('dynamodb',region_name=region)
response = dynamodb.create_table(
    AttributeDefinitions=[
        {
            'AttributeName': 'pk',
            'AttributeType': 'S'
        },
    ],
    TableName='innovate_latlon',
    KeySchema=[
        {
            'AttributeName': 'pk',
            'KeyType': 'HASH'
        },
    ],
    BillingMode='PAY_PER_REQUEST'
)
```
![kda6](/images/kda6.png)

