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

6. Create a new paragraph and execute the below code. This will create another DynamoDB table in ap-southeast-2
```
%flink.ipyflink
#create table innovate_custfeedback
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
    TableName='innovate_custfeedback',
    KeySchema=[
        {
            'AttributeName': 'pk',
            'KeyType': 'HASH'
        },
    ],
    BillingMode='PAY_PER_REQUEST'
)
```

7. Create a new paragraph and execute the below code. This will create a kinesis data stream in ap-southeast-2
```
%flink.ipyflink
#create KDS innovate_feedback
import boto3
region='ap-southeast-2'
kinesis = boto3.client('kinesis',region_name=region)
response = kinesis.create_stream(
    StreamName='innovate_feedback',
    ShardCount=3
)
print (response)

```

## Generating random data
1. Create a new S3 bucket or upload below csv files to your S3 bucket

    a) [custfeedback.csv](sampledata/custfeedback.csv)

    b) [latlon.csv](sampledata/latlon.csv)
 
 2. Create a new paragraph on your prerequisite notebook and execute the below code. Change the S3 location as yours (bucket, key). This will upload the latlon data to a DynamoDB table you created earlier.
 
 ```
 %flink.ipyflink
#upload lanlon data
import boto3
import csv
import codecs
region='ap-southeast-2'
recList=[]
tableName='innovate_latlon'
s3 = boto3.resource('s3')
dynamodb = boto3.client('dynamodb', region_name=region)
bucket='YOUR_BUCKETNAME'
key='real-time-sentiment-flinkSQL-KDAStudio/latlon.csv'
obj = s3.Object(bucket, key).get()['Body']
batch_size = 100
batch = []
i=0

for row in csv.DictReader(codecs.getreader('utf-8')(obj)):
    pk= (row["id"])
    postcode= (row["postcode"])
    suburb= (row["suburb"])
    State= (row["State"])
    latitude= (row["latitude"])
    longitude= (row["longitude"])
    
    response = dynamodb.put_item(
        TableName=tableName,
        Item={
        'pk' : {'S':str(pk)},
        'postcode': {'S':postcode},
        'suburb': {'S':suburb},
        'State': {'S':State},
        'latitude': {'S':latitude},
        'longitude': {'S':longitude}
        }
    )
    i=i+1
    #print ('Total insert: '+ str(i))
    
print ('completed')
 ```

3. Create a new paragraph on your prerequisite notebook and execute the below code. Change the S3 location as yours (bucket, key). This will upload the customer feedback data to a DynamoDB table you created earlier.

```
%flink.ipyflink
#upload custfeedback.csv
import boto3
import csv
import codecs
region='ap-southeast-2'
recList=[]
tableName='innovate_custfeedback'
s3 = boto3.resource('s3')
dynamodb = boto3.client('dynamodb', region_name=region)
bucket='YOUR_BUCKETNAME'
key='real-time-sentiment-flinkSQL-KDAStudio/custfeedback.csv'
obj = s3.Object(bucket, key).get()['Body']
batch_size = 100
batch = []
i=0

for row in csv.DictReader(codecs.getreader('utf-8')(obj)):
    pk= (row["id"])
    feedback= (row["feedback"])
    
    response = dynamodb.put_item(
        TableName=tableName,
        Item={
        'pk' : {'S':str(pk)},
        'feedback': {'S':feedback}
        }
    )
    i=i+1
    #print ('Total insert: '+ str(i))
    
print ('completed:' + str(i))
```
