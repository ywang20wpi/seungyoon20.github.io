---
layout: post
header: "AWS Parameter Store Example"
title:  "AWS Parameter Store Example"
date:   2020-05-05 09:30:02 -0400
categories: "Blog"
tags: [python, AWS, Lambda, Algolia, serverless]
icon: assets/social_icons/AWS-Systems-Manager_Documents_dark-bg.svg
intro: To avoid saving API secrets in environmental variables, use Parameter Store instead!
---
## The Story Begins from ...
Recently, I am working on a for-fun project with my friend Pusheen🐱. I am working on the backend using **AWS Lambda** functions for the backend, **AWS DynamoDB** as database, and **Algolia** for searching items. ~~Pusheen is working the frontend with React and deployed as **AWS S3 Static Page**.~~ We have a search function that is triggered by HTTP GET request, takes `keyword` from the parameters, and perform searching by keywords in our **Algolia items**.

For easier development and deployment, we are using **AWS Serverless Application Model (AWS SAM)** and **PyCharm** to develop locally and deploy to the **AWS Cloudformation**.

The difficulty we met here, was to find a safe place to save the **Algolia API Key** and **Algolia API Secret**. Since we plan to make the code public, saving it explicitly in the code or saving it in the cloudformation template `template.yaml` sounds dangerous.

## Setup and Use AWS Parameter Store
**1. Create secure parameters on AWS Systems Manager.**

Firstly, we need to navigate to **AWS System Manager > Parameter Store**.
<figure>
  <img src="{{site.baseurl}}/assets/img/parameter_store/step1.png" alt="Step 1"/>
  <figcaption> 1.1 Go to System Manager </figcaption>
</figure>
<figure>
  <img src="{{site.baseurl}}/assets/img/parameter_store/step2.png" alt="Step 2"/>
  <figcaption> 1.2 Navigate to Parameter Store</figcaption>
</figure>

**2. Setup the `template.yaml` for correct permissions.**

**3. Update the lambda function to use the parameters.**

## Why Parameter Storage So Important & Old Versions
The fact is, we didn't achieve the current solution in a single effort. It tooks us 2 ugly tries until the current version.
### Ugly Solution 0
This is the initial version without any optimization: __having the secrets in the lambda function as variables__. This is not elegant and risky that we need to ~~manually~~ remove the secret values and we may forget to hide the keys and secrets when we commit to the online repository.

```python
def lambda_handler(event, context):
    parameters = event['queryStringParameters']

    algolia_application_id = <application_id>
    algolia_search_only_key = <search_only_key>

    client = SearchClient.create(algolia_application_id, algolia_search_only_key)
    index = client.init_index('Tags')
    res = index.search(parameters['keyword'])

    return {
        'statusCode': 200,
        'body': json.dumps(res)
    }
```
### Still Ugly Solution 1
An alternative solution is to add the secrets to the **environ variables**. In this way, the secrets are no longer explicitly in the lambda source code, and are moved to the `template.yaml`.
Here is our first version of vulnerable code that we include the **Algolia API Secrets** explicitly in the **Cloudformation Template** `template.yaml`, which create an **API** called `ExampleApi`, and an **AWS Lambda** function in `Python3.6` called `SearchFunction` that allows access to function execution and DynamoDB:

```yaml
Resources:
  ExampleApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Staging
      Cors:
        AllowOrigin: "'*'"
        AllowMethods: "'OPTIONS,HEAD,GET,PUT,POST'"
        AllowHeaders: "'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'"
  SearchFunction:
    Type: AWS::Serverless::Function
    Properties:
      Environment:
        Variables:
          Algolia_Application_ID: this_is_secrete
          Algolia_Search_Only_Key: this_is_also_secrete
      Policies:
        - AWSLambdaExecute
        - AmazonDynamoDBFullAccess
      CodeUri: search/
      Handler: app.lambda_handler
      Runtime: python3.6
      Events:
        Search:
          Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
          Properties:
            Path: /search
            Method: get
            RestApiId:
              Ref: ExampleApi
```
and in the lambda function, we get the Algolia API secrets from the environment:
```python
algolia_application_id = os.environ['Algolia_Application_ID']
algolia_search_only_key = os.environ['Algolia_Search_Only_Key']
```

However, this version of code is still facing the same problem as the Solution 0 when publishing the code. The vulnerabilities made us move to using the Parameter Store.
