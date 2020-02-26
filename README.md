# aws-asg-org-metrics
Application to gather AWS EC2 Autoscaling Group Metrics for an AWS Organization and it's member accounts.

## Requirements

* Python - boto3, json, csv, datetime
* AWS
  * Lambda must be run in root account within AWS Organization
  * IAM Role with Permissions for DescribeAutoScalingGroups and Trust Policy for AWS Organization root account must exist in all AWS Organization Member Accounts (sample below in Policies). 
  * IAM Role for Lambda Execution must have sts:AssumeRole, s3 PutObject, organizations:ListAccounts, and CloudWatch Logs write permissions (sample below in Policies)

## Configuration Items

* Lambda Environmental Variables
  * "RegionList" - User must define a comma-delimited list of AWS region names in the Lambda environment variable (i.e. us-east-1,us-west-2,us-east2)
  * ChildAccountRole - name of IAM Role deployed to all Organization member accounts with trust to Organization master account id and permission to autoscaling:DescribeAutoScalingGroups
  * MaxOrgAccountsTest - test variable to limit number of Organization accounts pulled to gather ASG metrics on. Set to 0 to pull all Active Organization accounts. This is helpful for large organizations (multiple hundreds of accounts) if you need to tweak the program.
  * S3Bucket - Valid S3 bucket name in master Organization account in the same region the Lambda function runs in. This is where the csv files will be deposited with the metrics
  
## IAM Policies
  
