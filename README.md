# octank_portal
Front-end - Clone this repo to your GitHub account or CodeCommit
I. Use this repo for hosting your website using Amplify:
  1. Go to the AWS Amplify console: https://console.aws.amazon.com/amplify/home?region=us-east-1#/
  2. Create a new app and choose the appropriate source where this repository exists
  
II. Create a DynamoDB Global table:
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
      
      
III. Create a Lambda function with code from backend_lambda.py, keep settings default and provide a name to the function - Eg: Octank_portal

IV. Create a REST API with API Gateway.
    1. Create API -> Rest API -> Build
    2. Keep options as default and provide an appropriate name to the API - eg: Octank_portal
    3. a. Actions -> Create Resource -> Provide a name -> Enable API GAteway CORS -> Create Resource
       b. Select the resource -> Actions -> Create Method -> POST
    4. Choose the Integration type as Lambda and select your Lambda function from the dropdown.
    5. Select the POST API -> Actions -> Deploy API
    6. Create a New stage, provide an appropriate Stage name
    7. Copy the invoke_url and 
