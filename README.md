# aws-asg-org-metrics
Application to gather AWS EC2 Autoscaling Group Metrics for an AWS Organization and its member accounts. The Lambda will gather a list of the current Active AWS accounts in the Organization where the Lambda is deployed, assume a Role into each account, gather the ASG metrics into a csv file, then upload the file to the configured S3 bucket with a timestamp-based filename. Current metrics gathered per AutoScaling group are the following items:
* AccountId
* Region
* ASG Name
* Current Time (time of execution)
* CreatedTime (time ASG was created in account)
* DesiredCapacity
* MinSize
* MaxSize

Note: Modifications to the get_asg_metrics() and convert_list_csv() functions can be made to add additional fields from the DescribeAutoScalingGroups API call. [The full API result set is listed here at this link.](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DescribeAutoScalingGroups.html#API_DescribeAutoScalingGroups_Example_1)

## Requirements

* Python 3.8 (tested version) - boto3, json, csv, os, datetime
* AWS
  * Lambda must be run in root account within AWS Organization
  * IAM Role with Permissions for DescribeAutoScalingGroups and Trust Policy for AWS Organization root account must exist in all AWS Organization Member Accounts (sample below in Policies). 
  * IAM Role for Lambda Execution must have sts:AssumeRole, s3:PutObject, organizations:ListAccounts, and CloudWatch Logs write permissions (sample below in Policies)

## Configuration Items

* Lambda Environmental Variables
  * RegionList - User must define a comma-delimited list of AWS region names in the Lambda environment variable (i.e. us-east-1,us-west-2,us-east2)
  * ChildAccountRole - name of IAM Role deployed to all Organization member accounts with trust to Organization master account id and permission to autoscaling:DescribeAutoScalingGroups
  * MaxOrgAccountsTest - test variable to limit number of Organization accounts pulled to gather ASG metrics on. Set to 0 to pull all Active Organization accounts. This is helpful for large organizations (multiple hundreds of accounts) if you need to tweak the program.
  * S3Bucket - Valid S3 bucket name in master Organization account in the same region the Lambda function runs in. This is where the csv files will be deposited with the metrics
  
## IAM Policies

### Lambda Execution Role

* Trust Policy/Relationship

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

* Permissions Policy

Note: Replace 'NAME_OF_YOUR_S3_BUCKET' with your S3 bucket name that csv's will be uploaded to in the Resource section of the s3:PutObject Statement

```
{
    "Version": "2012-10-17",
    "Statement": [
    {
        "Effect": "Allow",
        "Action": [
            "organizations:Describe*",
            "organizations:List*"
        ],
        "Resource": "*"
    },
    {
        "Effect": "Allow",
        "Action": "s3:PutObject",
        "Resource": [
            "arn:aws:s3:::NAME_OF_YOUR_S3_BUCKET/*"
        ]
    },
    {
        "Effect": "Allow",
        "Action": [
            "logs:CreateLogGroup",
            "logs:CreateLogStream",
            "logs:PutLogEvents"
        ],
        "Resource": "*"
    },
    {
        "Effect": "Allow",
        "Action": "sts:AssumeRole",
        "Resource": "*"
    }
  ]
}
```
### ChildAccountRole

* Trust Policy/Relationship

Note: Replace the master AWS Organization AccountId where the Lambda is deployed in the Principal value

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::MASTER_ORG_ACCOUNT_WITH_LAMBDA:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```

* Permissions Policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "autoscaling:Describe*",
            "Resource": "*"
        }
    ]
}
```
