# People Info API

## Intro

The People Info API is a small API based on Typescript and Fastify. API allows to receive data and posts it to DynamoDB. It also uses Redis cluster as cache layer between application and database.

## Prerequisites

- Node.js 20+
- DynamoDB already created and runs in AWS (**app_iac** stack) or locally.
- Redis cluster already created and runs in AWS (**app_iac** stack) or locally.

Current code implementation has DynamoDb and Redis as hard requirements. Application will fail to start without them.

## Running locally
It is possible to choose where we run application, locally or in AWS. For that reason, we can utilize env var ENV. It is set either in **.env** or during npm run. 
If ENV=aws, application will be run in AWS (by default). If ENV=local, it will be run locally.

In order to satisfy dependencies locally, we need to install local DynamoDB and Redis images. To run application locally, we need:

```sh
docker-compose -f docker-compose-dynamo-redis.yml up -d
aws dynamodb create-table \
  --table-name PeopleInfo \
  --billing-mode PAY_PER_REQUEST \
  --attribute-definitions AttributeName=person_surname,AttributeType=S \
  --key-schema AttributeName=person_surname,KeyType=HASH \
  --endpoint-url http://localhost:8000 --region eu-west-1
npm install
npm run start:dev
```
Application is available at 3000 port `localhost:3000/status`.

## Running in AWS
After **basic_iac** and **app_iac** stacks are created, next steps shall be done:
1. Find out endpoint of Redis cluster in AWS.
2. **Important!!!** Update host for redis cluster in **.env**. variable AWS_REDIS_HOST.
3. Push changes to main branch.
4. Wait until workflow is done.
5. Find created ALB in AWS console or using terminal

## How to use app

If you use application locally, then instead of {ALB_URl}, use localhost:3000.

#### 1. Status Endpoint

Run the following command to verify the server status:

```bash
curl http://{ALB_URl}/status
```
**Expected**: Server status.

#### 2. Posting Data to `/data` Endpoint

**Request**:

Use the following `curl` command to post data:

```bash
curl -X POST http://{ALB_URl}/data -H "Content-Type: application/json" -d '{
  "person_name": "John",
  "person_surname": "Doe",
  "person_birthdate": "1990-01-01",
  "person_address": "123 Main St"
}'
```

**Expected**: Appropriate success response indicating the data has been saved/processed. The response should also reply with the data submitted.

#### 3. Retrieve Data by Surname

**Request**:

To get data by the person name, use:

```bash
curl http://{ALB_URl}/data?person_surname=<Person Surname>
```

Replace `<Person Surname>` with the actual name of the person you wish to query.

**Expected**: JSON response containing the details related to the specified person.

#### 4. Retrieve Prometheus client metrics

```bash
curl http://{ALB_URl}/metrics
```

## Tests
Test suit for checking **/status** endpoint is available. It runs in CI as part of **application_build_push_deploy** workflow. It can be run locally as well:

```bash
npm install
npm run test
```

## DynamoDB and Redis
1. Application saves data to DynamoDB and also puts it to Redis cache level. 
2. Get request is made to Redis first and then to DynamoDB.