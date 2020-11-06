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

On the other hand, in the DC-VPC, we have an EC2 instance configured with OpenSwan software acting as the Customer Gateway Device. The router is going to reside in the public subnet and route all the private traffic coming from the Cloud to our private subnets. In these private subnets, we have a postgres database with the sample data already loaded. A private instance is created just to test the private connection to the cloud environment through AWS Site-to-Site VPN.

## How to deploy the template

1. If you don’t have the AWS CLI installed, follow [these](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) steps. And to configure the AWS CLI, follow [these](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config). 
2. Clone the repository
3. Export the following parameters in your CLI:
```bash
export S3NAME = <YOUR BUCKET NAME> 
export AWSREGION = <YOUR AWS REGION>
export AWSPROFILE = <YOUR AWS PROFILE>
export STACKNAME = <THE NAME OF YOUR STACK>
```
