# AWS Cost Savings Automation Project

## Overview

This project automates the process of managing AWS resources, specifically focusing on **EC2 instances** and **EBS volumes**, to reduce unnecessary costs. The primary objective is to use **AWS Lambda**, **CloudWatch Events**, and **SNS** to automate the following tasks:

1. **Stop or terminate idle EC2 instances** during off-hours.
2. **Delete unused EBS volumes or snapshots** to save on storage costs.
3. **Send notifications** before performing actions on EC2 instances or EBS volumes.

---

## Architecture

This project leverages the following AWS services:

- **AWS Lambda**: To execute code automatically in response to events.
- **Amazon EC2**: To manage virtual servers and automate stopping/terminating idle instances.
- **Amazon EBS**: To manage block storage and automate the deletion of unused volumes.
- **Amazon SNS**: To send notifications before performing any resource management tasks.
- **Amazon CloudWatch Events**: To schedule and trigger Lambda functions based on specific events or times.

---

## Prerequisites

- **AWS Account**: You need an active AWS account with the necessary permissions.
- **IAM Roles**: Lambda functions require IAM roles with permissions to manage EC2, EBS, and SNS resources.
- **AWS CLI** (optional): For testing and deploying Lambda functions via the command line.

---

## Project Steps

### 1. **Create IAM Roles and Permissions**

- **IAM Role for Lambda**: The Lambda functions need permissions to interact with EC2, EBS, and SNS. Create an IAM role with the following managed policies:
  - `AmazonEC2FullAccess`
  - `AmazonElasticBlockStoreFullAccess`
  - `AmazonSNSFullAccess`
  - `CloudWatchLogsFullAccess`

- **SNS Topic for Notifications**: Create an SNS topic to send notifications before performing any actions on resources.

---

### 2. **Create Lambda Functions**

#### Lambda 1: **Stopping Idle EC2 Instances**

- **Description**: This function identifies idle EC2 instances and stops or terminates them to save costs.
- **Code**: The Lambda function will use the `boto3` AWS SDK to interact with EC2, checking for idle instances based on criteria (e.g., no usage or based on tags).
  
  Example Python code:
  ```python
  import boto3
  ec2 = boto3.client('ec2')
  sns = boto3.client('sns')

  def send_notification(message):
      sns.publish(
          TopicArn='arn:aws:sns:your-region:your-account-id:ec2-actions-notification',
          Message=message,
          Subject="AWS EC2 Action Notification"
      )

  def lambda_handler(event, context):
      instances = ec2.describe_instances(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}])
      for reservation in instances['Reservations']:
          for instance in reservation['Instances']:
              instance_id = instance['InstanceId']
              send_notification(f"EC2 instance {instance_id} will be stopped due to inactivity.")
              ec2.stop_instances(InstanceIds=[instance_id])

###  Lambda 2: **Deleting Unused EBS Volumes**

- **Description**: This function identifies unused EBS volumes and deletes them to reduce storage costs.
- **Code**: The Lambda function will identify EBS volumes in the "available" state and delete them.

Example Python code :
 ```python
  import boto3
ec2 = boto3.client('ec2')
sns = boto3.client('sns')

def send_notification(message):
    sns.publish(
        TopicArn='arn:aws:sns:your-region:your-account-id:ec2-actions-notification',
        Message=message,
        Subject="AWS EBS Action Notification"
    )

def lambda_handler(event, context):
    volumes = ec2.describe_volumes(Filters=[{'Name': 'status', 'Values': ['available']}])
    for volume in volumes['Volumes']:
        volume_id = volume['VolumeId']
        send_notification(f"EBS volume {volume_id} will be deleted as it is unused.")
        ec2.delete_volume(VolumeId=volume_id)

## Set Up CloudWatch Events for Scheduling

### EC2 Management (Stopping Idle Instances)
- Create a **CloudWatch Event Rule** to trigger the EC2 stopping Lambda function at a specified schedule (e.g., daily at 8 PM UTC).

### EBS Cleanup (Deleting Unused Volumes)
- Create another **CloudWatch Event Rule** to trigger the EBS cleanup Lambda function at a set interval (e.g., weekly or monthly).

---

## Testing the Functions

- Test each **Lambda function** independently to ensure they work as expected.
- Check **CloudWatch Logs** for function execution details.
- Verify **SNS notifications** are received before EC2 instances are stopped and EBS volumes are deleted.

---

## Key Benefits

- **Cost Reduction**: Automatically stop idle EC2 instances and delete unused EBS volumes to save on AWS costs.
- **Automation**: Minimize manual intervention in managing AWS resources.
- **Notifications**: Get notified before any action is taken on resources, ensuring that no critical data is affected.

---

## Future Enhancements

- **Idle Detection Based on Usage**: Enhance idle detection logic based on more specific metrics like CPU usage, network traffic, etc.
- **Error Handling**: Implement better error handling and retries in the Lambda functions for more resilience.
- **Cost Reporting**: Add a cost reporting feature to monitor savings over time.

---

## Resources

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Amazon EBS Documentation](https://docs.aws.amazon.com/ebs/latest/userguide/)
- [Amazon SNS Documentation](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [CloudWatch Events Documentation](https://docs.aws.amazon.com/cloudwatch/latest/events/WhatIsCloudWatchEvents.html)

---

## Conclusion

This project offers a simple, automated solution to help AWS users manage their EC2 instances and EBS volumes efficiently. By stopping idle instances and deleting unused storage, it significantly reduces unnecessary AWS costs, while offering flexibility and control with scheduled tasks and notifications.


