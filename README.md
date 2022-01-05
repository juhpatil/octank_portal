# OCTANK EXPO FLORAL EXHIBITON REGISTRATION PORTAL
## Data modelling
1. Download the NoSQL Workbench: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.settingup.html
2. Import the Octank_registration_portal.json file to visualize the model and get hands-on

# FRONT END, DYNAMODB GLOBAL TABLE
## I. Create a DynamoDB Global table:
```
  1. aws dynamodb create-table \
    --table-name Octank_registrations_2 \
    --attribute-definitions AttributeName=pk,AttributeType=S AttributeName=sk,AttributeType=S \
    --key-schema AttributeName=pk,KeyType=HASH AttributeName=sk,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST \
    --stream-specification StreamEnabled=TRUE,StreamViewType=NEW_AND_OLD_IMAGES \
    --region us-east-1
    
   2. aws dynamodb update-table --table-name Octank_registrations_2 --cli-input-json  \
      '{
          "ReplicaUpdates":
          [
            {
              "Create": {
                "RegionName": "eu-west-1"
            }
            }
          ]
      }'
    
    3. aws dynamodb update-table --table-name Octank_registrations_2 --cli-input-json  \
      '{
          "ReplicaUpdates":
          [
            {
              "Create": {
                "RegionName": "ap-southeast-1"
            }
            }
          ]
      }'
```
      
## II. Create a Lambda function with code from backend_lambda.py, keep settings default and provide a name to the function - Eg: Octank_portal

## III. Create a REST API with API Gateway.
    1. Create API -> Rest API -> Build
    2. Keep options as default and provide an appropriate name to the API - eg: Octank_portal
    3. a. Actions -> Create Resource -> Provide a name -> Enable API GAteway CORS -> Create Resource
       b. Select the resource -> Actions -> Create Method -> POST
    4. Choose the Integration type as Lambda and select your Lambda function from the dropdown.
    5. Select the POST API -> Actions -> Deploy API
    6. Create a New stage, provide an appropriate Stage name
    7. Copy the invoke_url
    
## IV. Clone this repo to your GitHub account or CodeCommit and edit the value of "var url" on line 56 of register.html to point to your API invoke_url copied earlier. 
Use this repo for hosting your website using Amplify:
  1. Go to the AWS Amplify console: https://console.aws.amazon.com/amplify/home?region=us-east-1#/
  2. Create a new app and choose the appropriate source where this repository exists

# ANALYTICS AND VISUALIZATION PIPELINE 
## I. Enable PITR on the table and export data to S3
```
  1. aws dynamodb update-continuous-backups \
    --table-name Octank_registrations_2 \
    --point-in-time-recovery-specification PointInTimeRecoveryEnabled=True 
    
  2. aws dynamodb export-table-to-point-in-time \
    --table-arn TABLE_ARN \
    --s3-bucket BUCKET_NAME \
```

## II. Create a Glue crawler
    1. Keep the options default till the Data store window -> Provide the S3 path of the exported data upto the "data/" folder in the Include path for the crawler.
    2. Create an IAM role/choose an existing role with appropriate permissions to the S3 path, as needed
    3. Choose the required database or create a new database -> Finish
    4. Select the crawler -> Choose Run crawler
    
## III. Analytical queries using Athena - Once the crawler has added tables, go to the Athena console and run the following queries:
```
    1. Data preview:
       SELECT * FROM "octank_expo_30th_dec"."data" limit 10;
      
    2. 5 most popular events:
       SELECT item.sk.s as EventID, item.event_name.s AS EventName, COUNT(item.pk.s) AS Count_of_visitors
       FROM "octank_expo_30th_dec"."data"
       where item.sk.s!='profile_visitor'
       GROUP BY item.sk.s, item.event_name.s
       ORDER BY Count_of_visitors DESC
       limit 5;
       
    3. Visitors by purpose of visit:
       SELECT item.purpose_of_visit.s as Purpose, COUNT(item.purpose_of_visit.s) AS Count
       FROM "octank_expo_30th_dec"."data"
       where item.pk.s like 'visitor_%'
       GROUP BY item.purpose_of_visit.s
       ORDER BY Count DESC
```

## IV. Visualization using Quicksight
    1. Datasets -> New dataset -> S3 -> provide a name -> Upload a manifest file (In S3manifest_30dec.json file, change the fileLocations value to your S3 data path      till the "/data" folder where we previously exported the DynamoDB table data)
    2. Analyses -> New analysis -> Choose the dataset created earlier -> Create analysis
    3. Choose required visual type and attribute - eg: Pie chart for Item.profession.s
    4. Add more visuals as needed.
    5. Once everything is added, click on Share -> Publish dashboard
