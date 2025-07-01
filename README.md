# bedrock-data-analysis-agent
A data analysis assistant for video game sales&amp;reviews data analysis based on Bedrock Agent.

## Backend Installation
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
