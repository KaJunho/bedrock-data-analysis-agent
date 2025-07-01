# Bedrock Data Analysis Agent
A data analysis assistant for video game sales and reviews data analysis based on Bedrock Agent.
For video demo, check out [demo](https://amazon.awsapps.com/workdocs-amazon/index.html#/document/c186c2e260413a50726f9ff849d0d8a7d50420f69a7517a3483dda0b6d5bf7a1).

## Architecture
The architecture of this project is shown below.
![new-agent-arch](https://github.com/user-attachments/assets/f48a1eb5-7a01-4637-9f18-2827963c5276)


## Backend Deployment
### Prerequisites

Before you begin, ensure you have:

* [SAM CLI Installed](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
* [Python 3.9 or later](https://www.python.org/downloads/) 
* [Boto3 1.36 or later](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html)
* Anthropic Claude 3.5 Haiku and Sonnet models enabled in Amazon Bedrock
* Run this command to create a service-linked role for RDS:

```bash
aws iam create-service-linked-role --aws-service-name rds.amazonaws.com
```

<hr/>

### Deploy the Back-End Services with AWS SAM

Navigate to the SAM project folder (sam-bedrock-video-games-sales-assistant/) and execute::

```bash
sam build
```

> [!CAUTION]
> If you encounter a **Build Failed error**, you might need to change the Python version in the [template.yaml](./template.yaml) file. By default, the Lambda function uses **Python 3.13**. You can modify this setting on **line 121** of the **[template.yaml](./template.yaml)** file to use a Python version lower than 3.13 that you have installed.

Now deploy the SAM application under the SAM project foler (sam-bedrock-video-games-sales-assistant/):

```bash
sam deploy --guided --capabilities CAPABILITY_NAMED_IAM
```

Use the following value arguments for the deployment configuration:

- Stack Name : **sam-bedrock-video-games-sales-assistant**
- AWS Region : **us-east-1**
- Parameter PostgreSQLDatabaseName : **video_games_sales**
- Parameter AuroraMaxCapacity : **2**
- Parameter AuroraMinCapacity : **1**
- Parameter S3BucketName : **game-reviews-0630**
- Parameter KnowledgeBaseName : **game-kb-0630**
- Parameter KnowledgeBaseDescription : **Game reviews knowledge base**
- Parameter DataSourceName : **game_reviews-0630**
- Parameter DataSourceDescription : **Data Source for Amazon DA Assistant Knowledge Base**
- Confirm changes before deploy : **Y**
- Allow SAM CLI IAM role creation : **Y**
- Disable rollback : **N**
- Save arguments to configuration file : **Y**
- SAM configuration file : **samconfig.toml**
- SAM configuration environment : **default**

After the SAM project preparation and changeset created, confirm the following to start the deployment:

- Deploy this changeset? [y/N]: **Y**

After deployment completes, the following services will be created:

- Amazon Bedrock Agent configured with:
    - **[Agent Instructions](./resources/agent-instructions.txt)**
    - **[Agent API Schema that provides the tools for the Agent (Action Group)](./resources/agent-api-schema.json)**
- Lambda Function API for the agent to use
    - **[Provide the tools for the agent: runSQLQuery, getCurrentDate, and getTablesInformation](./functions/assistant-api-postgresql-haiku-35/tables_information.txt)**
- The Aurora Serverless PostgreSQL Cluster Database
- A DynamoDB Table for tracking questions and query details
- A Bedrock Knowledge Base
- A Bedrock Guardrail

> [!NOTE]
> To learn about agent creation configuration, please refer to [this tutorial](./manual_database_data_load_and_agent_creation.md), which provides step-by-step guidance for setting up an Amazon Bedrock Agent in the AWS Console.

<hr/>

### Load Sample Data into Aurora PostgreSQL Database

Set up the required environment variables:

``` bash
# Set the stack name environment variable
export STACK_NAME=sam-bedrock-video-games-sales-assistant

# Retrieve the output values and store them in environment variables
export SECRET_ARN=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" --query "Stacks[0].Outputs[?OutputKey=='SecretARN'].OutputValue" --output text)
export DATA_SOURCE_BUCKET_NAME=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" --query "Stacks[0].Outputs[?OutputKey=='DataSourceBucketName'].OutputValue" --output text)
export AURORA_SERVERLESS_DB_CLUSTER_ARN=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" --query "Stacks[0].Outputs[?OutputKey=='AuroraServerlessDBClusterArn'].OutputValue" --output text)
cat << EOF
STACK_NAME: ${STACK_NAME}
SECRET_ARN: ${SECRET_ARN}
DATA_SOURCE_BUCKET_NAME: ${DATA_SOURCE_BUCKET_NAME}
AURORA_SERVERLESS_DB_CLUSTER_ARN: ${AURORA_SERVERLESS_DB_CLUSTER_ARN}
EOF
```

Execute the following command to create the database and load the sample data:

``` bash
python3 resources/create-sales-database.py
```

The script uses the **[video_games_sales_random_avg_sales.csv]** as the data source.


<hr/>

### Sync Knowledge Base Data Source

Navigate to your Amazon Bedrock Knowledge Bases control panel:

- Click **game-kb-0630**
- In **Data source** section, check the data source **game_reviews-0630**
- Click **Sync** button on the top right


<hr/>

### Test the Agent in AWS Console

Navigate to your Amazon Bedrock Agent named **video-game-sales-data-analyst**:

- Click **Edit Agent Builder**
- In the Agent builder section click **Save**
- Click **Prepare**
- Click **Test**

Try these sample questions in Chinese:
- 你好
- 请找出总销量最佳的三款游戏。
- 2015年至2019年发布的游戏，在各地区的总销售额是多少？请以百分比形式给我数据。
- 请简单介绍一下Moon Storm这款游戏


<hr/>

### Create Agent Alias for Front-End Application

To use the agent in your front-end application:

- Go to your **Agent Overview**
- Click **Create Alias**

You can now proceed to the [Front-End Implementation - Integrating Amazon Bedrock Agent with a Ready-to-Use Data Analyst Assistant Application](../amplify-video-games-sales-assistant-bedrock-agent/). The tutorial will ask you for your **Agent Alias** along with the other services that you have created so far.

<hr/>

## Frontend Deployment

### Prerequisites

Before you begin, ensure you have:

- An **Alias** created for your Amazon Bedrock Agent from the **Generative AI Application - Data Source and Amazon Bedrock Agent Deployment** tutorial
- [Node.js version 18+](https://nodejs.org/en/download/package-manager)
- React Scripts installed:
``` bash
npm install react-scripts
```
<hr/>

### Set Up the Front-End Application

Navigate to the React application folder (amplify-video-games-sales-assistant-sample/) and install the Reac application dependencies:

``` bash
npm install
```
<hr/>

### Configure IAM User Access for Front-End Permissions

- [Create an IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)
- [Create Access key and Secret access key](https://docs.aws.amazon.com/keyspaces/latest/devguide/create.keypair.html) for programmatic access
- Add an inline policy to this user with the following JSON (replace placeholder values with your actual ARNs).

Update the values with your **<agent_arn>**, **<agent_id>**, **<account_id>** and **<question_answers_table_arn>** that you can find in the outputs from the SAM tutorial.

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "InvokeBedrockAgent",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeAgent"
            ],
            "Resource": [
                "<agent_arn>",
                "arn:aws:bedrock:*:<account_id>:agent-alias/<agent_id>/*"
            ]
        },
        {
            "Sid": "InvokeBedrockModel",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel"
            ],
            "Resource": [
                "arn:aws:bedrock:*:<account_id>:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0",
                "arn:aws:bedrock:us-east-2::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0",
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0",
                "arn:aws:bedrock:us-west-2::foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0"
            ]
        },
        {
            "Sid": "DynamoDB",
            "Effect": "Allow",
            "Action": [
                "dynamodb:Query"
            ],
            "Resource": "<question_answers_table_arn>"
        }
    ]
}
```
<hr/>

### Configure Environment Variables

- Rename the file **src/sample.env.js** to **src/env.js** and update the following environment variables:

    - AWS Credentials and Region:
        - **ACCESS_KEY_ID**
        - **SECRET_ACCESS_KEY**
        - **AWS_REGION**

    - Agent and table information that you can find in the CloudFormation Outputs from the SAM project:
        - **AGENT_ID**
        - **AGENT_ALIAS_ID**
        - **QUESTION_ANSWERS_TABLE_NAME** 

    - Also, you can update the general application description:
        - **APP_NAME**
        - **APP_SUBJECT**
        - **WELCOME_MESSAGE**

<hr/>

### Test Your Data Analyst Assistant

Start the application locally:

``` bash
npm start
```


### Application Features

Congratulations! Your Data Analyst Assistant can provide you with the following conversational experience:

- **Conversational interface with an agent responding to user questions**

![Video Games Sales Assistant](../images/preview1.png)

- **Detailed answers including the rationale behind SQL query generation**

![Video Games Sales Assistant](../images/preview2.png)

- **Raw query results displayed in tabular format**

![Video Games Sales Assistant](../images/preview3.png)

- **Chart visualization generated from the agent's answer and the data query results (created using [Apexcharts](https://apexcharts.com/))**.

![Video Games Sales Assistant](../images/preview4.png)

- **Summary and conclusion derived from the data analysis conversation**

![Video Games Sales Assistant](../images/preview5.png)



## Thank You

### License

This project is licensed under the Apache-2.0 License.
