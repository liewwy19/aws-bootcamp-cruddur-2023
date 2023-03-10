# Week 0 — Billing and Architecture

## :pencil: Required Homework/Tasks

### Watched Week 0 - Live Streamed Video :white_check_mark:
Done

### Watched Chirag's Week 0 - Spend Considerations :white_check_mark:
Done

### Watched Ashish's Week 0 - Security Considerations :white_check_mark:
Done

### Recreate Conceptual Diagram in Lucid Charts or on a Napkin :white_check_mark:
![aws_cloudshell](../_docs/assets/week0/napkin_design.jpg)

### Recreate Logical Architectual Diagram in Lucid Charts :white_check_mark:
![aws_cloudshell](../_docs/assets/week0/CRUDDUR_lucidchart.png)
:link:[Lucidchart Link](https://lucid.app/lucidchart/46cefaa2-29cb-4260-a7c5-9c3959598a02/edit?viewport_loc=-137%2C47%2C1993%2C811%2C0_0&invitationId=inv_8196240e-2a89-4aec-9fc9-2dff1aceb8dd)

### Create an Admin User :white_check_mark:	 
![Admin_User_dashboard](../_docs/assets/week0/IAM_dashboard_Admin_User.png)

### Use CloudShell :white_check_mark:
![aws_cloudshell](../_docs/assets/week0/cloudshell.png)

### Generate AWS Credentials	:white_check_mark:
![aws_access_key](../_docs/assets/week0/aws_credentials.png)

### Installed AWS CLI	:white_check_mark:
![aws_cli](../_docs/assets/week0/gitpod_aws_cli.png)

### Create a Billing Alarm	:white_check_mark:
![aws_confirm_subscription](../_docs/assets/week0/confirm_subscription.png)
![aws_confirm_subscription](../_docs/assets/week0/billing_alarm.png)

### Create a Budget	:white_check_mark:
![aws_budget](../_docs/assets/week0/create_budget.png)

***

## :pencil: Homework Challenges

### :bulb:Enable MFA for roor account
![mfa_root](../_docs/assets/week0/root_mfa.png)

### :bulb:Enable MFA for admin user account
![mfa_admin_user](../_docs/assets/week0/admin_user_mfa.png)

### :bulb:Turn on Free-tier Usage alert
![mfa_root](../_docs/assets/week0/freetier_usage_n_budget_alert_notification.png)

### :bulb:Research on AWS Service Quotas
Please refer to my technical essay here: [Understanding AWS Service Quotas](_docs/understanding_aws_service_quotas.md)

### :bulb:Enable and setup AWS Organization
![mfa_root](../_docs/assets/week0/aws_organization.png)

### :bulb:Enable SCP and add new SCP policy to require MFA for AWS EC2 API call
![mfa_root](../_docs/assets/week0/aws_SCP.png)
![mfa_root](../_docs/assets/week0/attach_SCP.png)
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyStopAndTerminateEC2WhenMFAIsNotPresent",
      "Effect": "Deny",
      "Action": [
        "ec2:StopInstances",
        "ec2:TerminateInstances"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": false
        }
      }
    }
  ]
}
```

***

## Getting the AWS CLI Working

We'll be using the AWS CLI often in this bootcamp,
so we'll proceed to installing this account.


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

- Go to (IAM Users Console](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/users) andrew create a new user
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
    "UserId": "AIFBZRJIQN2ONP4ET4EK4",
    "Account": "655602346534",
    "Arn": "arn:aws:iam::655602346534:user/andrewcloudcamp"
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
    --topic-arn="arn:aws:sns:ap-southeast-1:110747955323:billing-alarm" \
    --protocol=email \
    --notification-endpoint="liewwy19@gmail.com"
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

## Git Tag

Git tags are used as reference points in your development worflow.

```sh
git tag week0
git push --tags
```

To delete a tag

#### local
```sh
git tag -d <tag_name>
```

#### remote
```sh
git push --delete origin <tag_name>
```
