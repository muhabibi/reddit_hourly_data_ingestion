# reddit_hourly_data_ingestion

##Introduction
The consideration of creating this data pipeline is since we only pull 100 posts from reddit forum. 
Because it's small data we can use ```serverless function```. I'll walk through about this pipeline and explain about the pros and cons of using ```serverless function```

##Serverless Functions
We will use ```AWS Lambda``` to run certain function without having to provision and maintain a server.  We write a function and deploy it to AWS Lambda.
The lambda function will run based on triggers (e.g new creation object on S3, deletion on S3, etc). Then the Lambda function will execute the logic and end.
Lambda function will have a limited time for completing the logic (curently maximum execution time is 15 mins)
##Overview
![alt text](https://github.com/muhabibi/reddit_hourly_data_ingestion/blob/master/pipeline.png?raw=true)
This project is to pull API data and put them in databaseon hourly schedule.

##Data Pipeline
###Pull data from API then save as file

###1. API endpoint : https://www.reddit.com/r/singapore/hot/
Client ID & Client Secret: At first we need to auth our app so we can get client ID & client secret, in order to read from reddit API

###2. Paginate
The general approach to make requests when pulling data from endpoint is chunk. For this usecase, it doesn't matter because the number of data that we pull is 100 data

Lambda functions is temporary, so are their storage. We will place our data in local storage in certain folders and put it to persistent storage system which is ```AWS S3```
The logic is quite straightfoward:
1. Get data from API
2. Parse JSON data
3. Write to local storage at certain directory

###Writing to AWS S3
We will create a function called lambda_handler as the entry point of lambda execution. After we run downloading data, we upload the odwnloaded local files to the certain S3 bucket that we specified

###Deploy & Schedule Lambda Functions
We will deploy functions above to a Lambda function which will execute the code on a trigger. Then set the lambda function to be triggered every 1 hour

###Glue Job: Transform
We deploy another functions that is to transform the data on S3. Let's say if we want to do cleansing, fix the certain format of columns, etc. This job will be triggered on every new file has been created on S3

###Query data from RDS
We will perform Upsert (Update & insert) job to MySQL RDS. Let' say we have a table with keys: key_a, key_b and other columns: col_c, col_d we can use the following SQL:
```insert into TABLENAME (key_a, key_b, col_c, col_d) values (?,?,?,?) ON DUPLICATE KEY UPDATE col_c=values(col_c), col_d=values(col_d)```
