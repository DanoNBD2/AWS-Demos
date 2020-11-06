# Connecting AWS Glue to your On Premises Data center Demo

This repository contains a demo showcasing features of AWS Services. When launching the CloudFormation template you will be able to see and test, how AWS Glue can connect to a Database in an On Premises environment through a Site-to-Site VPN connection.

## Requirements

* AWS Account: If you don’t have an account nor an account provided to you, [click here](https://aws.amazon.com/es/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc).
* Text editor (optional): Just to make things easier and more organised.

## Architecture

![AWS Glue Demo Architecture](/Demo-Architecture.jpg)

What you have here are two environments. One simulating the data center (DC-VPC) and on the right, the cloud environment (Cloud-VPC). 

In the Cloud VPC we have no internet connection, so all the communication is going to be private. In the private subnet we have the Elastic Network Interfaces created by AWS Glue to provide network connectivity for the service through your VPC. Note that the number of ENIs depends on the number of data processing units (DPUs) selected for an AWS Glue ETL job. AWS Glue DPU instances communicate with each other and with your JDBC-compliant database using ENIs.

The S3 endpoint is going to be used by AWS Glue to store temporary files and ETL scripts in S3. Also, in this demo, S3 is going to be our target for processed data coming from the Database located on the data center.
