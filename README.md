# Real-time sentiment analysis on customer feedback

## Reference architecture

![kda1](/images/kda1.PNG)

## Create a Kinesis Data Analytics Studio nobook
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
3. Click on Attach Policies and add Administrator Access. (This is not recommended for your production workload. Specify the requirement permission needed)

![kda1](/images/kda3.png)
