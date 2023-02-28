# Week 0 — Billing and Architecture

## Bootcamp prerequisites
- [x] Create a Free GITHUB Account
- [x] Setup MFA on your GITHUB Account
- [x] Create a Free GITPOD Account
- [x] Get the GITPOD Button using Chrome Extension
- [x] Setup GITHUB Code spaces
- [x] Create Your AWS Account
- [x] Create Your Repository from the GITHUB Template
- [x] Create Your Free Lucidchart Account
- [x] Create Your Free Honeycomb.io Account
- [x] Create Your Free Rollbar Account

## Getting the AWS CLI Working

We'll be using the AWS CLI often in this bootcamp,
so we'll proceed to installing the AWS CLI in this account.


### Install AWS CLI

- We are going to install the AWS CLI when our Gitpod enviroment lanuches.
- We are are going to set AWS CLI to use partial autoprompt mode to make it easier to debug CLI commands.
- The bash commands we are using are the same as the [AWS CLI Install Instructions]https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html


Update our `.gitpod.yml` to include the following task.

```sh
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

We'll also run these commands indivually to perform the install manually

### Create a new User and Generate AWS Credentials

- Go to (IAM Users Console](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/users) mtanvir create a new user
- `Enable console access` for the user
- Create a new `Admin` Group and apply `AdministratorAccess`
- Create the user and go find and click into the user
- Click on `Security Credentials` and `Create Access Key`
- Choose AWS CLI Access
- Download the CSV with the credentials

### Set Env Vars

We will set these credentials for the current bash terminal
```
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=us-east-1
```

We'll tell Gitpod to remember these credentials if we relaunch our workspaces
```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=us-east-1
```

### Check that the AWS CLI is working and you are the expected user

```sh
aws sts get-caller-identity
```

You should see something like this:
```json
{
    "UserId": "AIDAZL4NJWPKWUPAIYNYY",
    "Account": "644004557781",
    "Arn": "arn:aws:iam::644004557781:user/mtanvir"
}
```

## Enable Billing 

We need to turn on Billing Alerts to recieve alerts...


- In your Root Account go to the [Billing Page](https://console.aws.amazon.com/billing/)
- Under `Billing Preferences` Choose `Receive Billing Alerts`
- Save Preferences


## Creating a Billing Alarm

### Create SNS Topic

- We need an SNS topic before we create an alarm.
- The SNS topic is what will delivery us an alert when we get overbilled
- [aws sns create-topic](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html)

We'll create a SNS Topic
```sh
aws sns create-topic --name billing-alarm
```
which will return a TopicARN

We'll create a subscription supply the TopicARN and our Email
```sh
aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint your@email.com
```

Check your email and confirm the subscription

#### Create Alarm

- [aws cloudwatch put-metric-alarm](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html)
- [Create an Alarm via AWS CLI](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/)
- We need to update the configuration json script with the TopicARN we generated earlier
- We are just a json file because --metrics is is required for expressions and so its easier to us a JSON file.

```sh
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json
```

## Create an AWS Budget

[aws budgets create-budget](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html)

Get your AWS Account ID
```sh
aws sts get-caller-identity --query Account --output text
```

- Supply your AWS Account ID
- Update the json files
- This is another case with AWS CLI its just much easier to json files due to lots of nested json

```sh
aws budgets create-budget \
    --account-id AccountID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications-with-subscribers.json
```


## Homework Challenges
### Napkin Design for the CRUDDUR Micro-blogging Application

![Napkin Design Diagram](/_docs/assets/cruddur-napkin-design.jpg)

### Conceptual Design Diagram
This design diagram created using [Lucidchart App](https://lucid.app/lucidchart/92650fca-10c3-4e37-ae24-05185f4d8e36/edit?viewport_loc=84%2C-121%2C1480%2C639%2C0_0&invitationId=inv_561d6d50-565c-4d3a-abe4-3167005a08d4).
![Conceptual Design Diagram](/_docs/assets/cruddur-conceptual-diagram.png)

### Conceptual Architecture Design Diagram on AWS Cloud
This Architecture design diagram created using [Lucidchart App](https://lucid.app/lucidchart/4a0864b0-8709-45f1-9384-6da614617267/edit?viewport_loc=-116%2C10%2C2220%2C958%2C0_0&invitationId=inv_f34d3d44-a525-4382-b79d-b087e80693c7).

I have added AWS IAM, AWS KMS and AWS Secrets Manager as part of the security services in the architecture diagram.
- AWS IAM : Will be used to create role and policy to provide least previledge/access to the services ( eg. ecs service role )
- AWS KMS : Provides a highly available key storage, management, and auditing solution to encrypt the data across AWS services & within applications (eg. KMS key for RDS).
- AWS Secrets Manager : Enables you to easily store, rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle.

![Conceptual Architecture Design Diagram](/_docs/assets/cruddur-conceptual-arch-diagram-aws.png)

### CI/CD Pipeline Design Diagram on AWS Cloud
This CI/CD design diagram created using [Lucidchart App](https://lucid.app/lucidchart/ea5089ae-ffd6-42ee-a707-e6726e8762a7/edit?viewport_loc=-176%2C152%2C2220%2C958%2C0_0&invitationId=inv_eac66b45-d17d-4a4d-a41c-9c4f82aa0084).
![CI/CD Pipeline Design Diagram](/_docs/assets/cruddur-cicd-pipeline-aws.png)

## Reference Documentation used and followed
[Manage AWS Service Quotas](https://aws.amazon.com/premiumsupport/knowledge-center/manage-service-limits/)
[Monitoring AWS Health events with Amazon EventBridge](https://docs.aws.amazon.com/health/latest/ug/cloudwatch-events-health.html)
[CI/CD pipelines and deploy the application to Amazon ECS clusters](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/automatically-build-ci-cd-pipelines-and-amazon-ecs-clusters-for-microservices-using-aws-cdk.html)
