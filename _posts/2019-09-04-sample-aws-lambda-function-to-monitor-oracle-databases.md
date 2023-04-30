---
title: "Sample AWS Lambda Function to Monitor Oracle Databases"
layout: post
---

I wrote a simple AWS Lambda function to demonstrate how to connect an Oracle database, gather the tablespace usage information, and send these metrics to CloudWatch. First, I wrote this lambda function in Python, and then I had to re-write it in Java. As you may know, you need to use the cx_oracle module to connect Oracle Databases with Python. This extension module requires some libraries shipped by Oracle Database Client (oh God!). It’s a little bit tricky to pack it for the AWS Lambda.

Here’s the main class which a Lambda function can use:

{% gist 37cb009db0109240014cdb00648b7f52 %}

<!--more-->

Here is the step-by-step explanation of the above script:

* Line 15) I created a class named Monitoring – this will be the handler class. To be able to test it, it accepts JSON objects. This is why it implements "RequestHandler, String>”.
* Line 16) Definition of the function required to handle requests
* Line 18) Taken from the AWS documents. It’s the object which we use to push metrics to CloudWatch
* Lines 20-24) Loading the JDBC class to be able to connect an Oracle database
* Lines 26-29) Defining the connection and credentials. As you see, I fetch the required parameters from the environment because Lambda lets you define env variables
* Lines 32-33) Connecting to the database
* Line 35) Creating a statement object to run queries
* Line 37) As a sample, I query dba_tablespace_usage_metrics to get the usage percentage for tablespaces
* Line 38) Loop for each record
* Lines 40-47) Preparing the metric I want to push to CloudWatch – The metric will be stored as "Space Used (pct) for each tablespace in the "Databases” namespace in the customer metrics
* Line 49) Pushes the metric data
* Lines 53-55) Mandatory exception handling block
* Line 57) Returns a result (success)

You may notice a lot of "System.getenv” calls. I used it to read the environment parameters from the Lambda function.

[![Download](/assets/gitHub-download-button.png)](https://github.com/gokhanatil/samplelambda)

You can download the Maven project from my GitHub repository. After you build the maven project, you need to upload the JAR file to your S3 bucket:

````
aws s3 cp target/samplelambda-0.1-jar-with-dependencies.jar s3://yourbucketname/
````

I used this JAR file to create a lambda function through the AWS Lambda console. My package name is "com.gokhanatil.samplelambda”, my class name is "Monitoring”, and the function name is "handleRequest”. This is why I entered "com.gokhanatil.samplelambda.Monitoring::handleRequest” as the handler. Please note that you need to give the required permissions to grant access to CloudWatch (e.g. CloudWatchFullAccess) and AWSLambdaVPCAccessExecutionRole. You also need to enter the Virtual Private Cloud (VPC) information (Subnets: pick two subnets that can reach your Oracle instance. Security groups: select one that gives you access to your Oracle instance).

To schedule this job to run periodically, you can use "CloudWatch events”. This event will be triggered every 15 minutes and launch the lambda function.

![Metrics](/assets/cloudwatch-metrics.png)

After enabling the lambda function, new metrics appear in the CloudWatch dashboard. It’s possible to create an alarm for these metrics using the console or AWS CLI commands.
