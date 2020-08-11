---
layout: post
title: AWS CodeBuild Custom Notification Messages
categories: AWS CodeBuild Custom Notification Messages using SNS and Lambda with Python Botocore SDK Customize AWS CloudWatch notifications.    
tags: AWS CodeBuild Custom Notification Messages using SNS and Lambda with Python Botocore SDK Customize AWS CloudWatch notifications.    
description: AWS CodeBuild Custom Notification Messages using SNS and Lambda with Python Botocore SDK. Customize AWS CloudWatch notifications.  
---

Recently we got a chance to implement AWS Code Pipeline for a project. It has four stages: CodeCommit, CodeBuild, CodeDeploy, and Test using Jenkins. There was a requirement to enable notifications for each state of the stages. So we have created a SNS Topic and added subscriptions so that we can use this Topic in the AWS services to send the notification.
<!--more-->
Enabling notifications in CodeCommit and CodeDeploy are pretty straight forward. In CodeCommit you can create a Trigger with Event "Push to existing branch" and add target as our SNS topic. Same with CodeDeploy, we can Create Trigger from Deployment Group with events (DeploymentStart, DeploymentSuccess, DeploymentFailure, DeploymentStop, DeploymentReady, DeploymentRollback) and target as our SNS Topic.  

But the problem with CodeCuild is, we don't have a built-in trigger for the CodeBuild state change. So we have to use CloudWatch events to trigger the notifications, where we have created Rule with event pattern and target as SNS topic in CloudWatch. In the Rule creation page we have to select "Event Pattern" and "Build event pattern to match events by service", where we will select CodeBuild as "Service Name" and "All Events" for "Event Type". This will generate an event pattern for CodeBuild status (IN_PROGRESS, SUCCEDED, FAILED, STOPPED).

But the cloudwatch event notifications send the emails with a static content as subject ("AWS Notification Message"), which won't be helpful for the user who subscribed for the notification, as he has to open the email and read the body to understand the state (success or failure, which build, etc.) of the CodeBuild. So we came up with a solution to customize the notification from CloudWatch.

From CloudWatch, while creating the Rule instead of giving the target as SNS Topic, we have given the target as AWS Lambda function. We will get the entire event details as json in our Lambda function, which we can use to prepare the subject and body of the notification message. We have used Python Botocore SDK to publish the message to SNS Topic. Please find below the AWS Lambda function:

<script src="https://gist.github.com/abhidsm/1f81819733ffe631b795bac5245b8564.js"></script>  


