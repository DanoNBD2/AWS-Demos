# Connecting AWS Glue to your On Premises Data center Demo

This repository contains a demo showcasing features of AWS Services. When launching the CloudFormation template you will be able to see and test, how AWS Glue can connect to a Database in an On Premises environment through a Site-to-Site VPN connection.

## Requirements

* **AWS Account:** If you don’t have an account nor an account provided to you, [click here](https://aws.amazon.com/es/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc).
* **Text editor (optional):** Just to make things easier and more organised.

## Architecture

![AWS Glue Demo Architecture](/images/Demo-Architecture.jpg)
What you have here are two environments. One simulating the data center (DC-VPC) and on the right, the cloud environment (Cloud-VPC). 

In the Cloud VPC we have no internet connection, so all the communication is going to be private. In the private subnet we have the Elastic Network Interfaces created by AWS Glue to provide network connectivity for the service through your VPC. Note that the number of ENIs depends on the number of data processing units (DPUs) selected for an AWS Glue ETL job. AWS Glue DPU instances communicate with each other and with your JDBC-compliant database using ENIs.

The S3 endpoint is going to be used by AWS Glue to store temporary files and ETL scripts in S3. Also, in this demo, S3 is going to be our target for processed data coming from the Database located on the data center.

On the other hand, in the DC-VPC, we have an EC2 instance configured with OpenSwan software acting as the Customer Gateway Device. The router is going to reside in the public subnet and route all the private traffic coming from the Cloud to our private subnets. In these private subnets, we have a postgres database with the sample data already loaded. A private instance is created just to test the private connection to the cloud environment through AWS Site-to-Site VPN.

## How to deploy the template

1. If you don’t have the AWS CLI installed, follow [these](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) steps. And to configure the AWS CLI, follow [these](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config). 
2. Clone the repository.
3. Export the following parameters in your CLI:
```bash
export S3NAME=<YOUR BUCKET NAME> 
export AWSREGION=<YOUR AWS REGION>
export AWSPROFILE=<YOUR AWS PROFILE>
export STACKNAME=<THE NAME OF YOUR STACK>
```
4. Create a S3 bucket:
```bash
aws s3 mb s3://$S3NAME --profile $AWSPROFILE --region $AWSREGION
```
5. Copy all the files from the cloned repository to your S3. To do that, enter the folder where you have the files that you need to copy and execute the following:
```bash
aws s3 cp . s3://$S3NAME --profile $AWSPROFILE --recursive
```
6. Create the CloudFormation stack:
```bash
aws cloudformation create-stack --stack-name $STACKNAME --template-url https://$S3NAME.s3.amazonaws.com/Templates/main.yaml --tags Key=project,Value=glue-project --profile $AWSPROFILE --region=$AWSREGION  --parameters ParameterKey=Bucket,ParameterValue=$S3NAME --capabilities CAPABILITY_IAM
```
*NOTE*: The template takes 30 min to deploy approx.

## Explore your environment

You have two VPCs, one called Cloud-VPC which is the AWS side of the infrastructure and the other one called DC-VPC which is the simulated Data Center. 

**In the Cloud-VPC note that it has:**
* 4 subnets and all of them are private (even though there are two named as public, there is no internet gateway attached to the VPC so there is no public communication). The reason is because the communication between these two environments is going to be through the VPN, so there is no need to have public subnets nor internet access. 
* 1 private route tables and inside you can see a S3 endpoint. This is for AWS Glue to access S3 privately (as this VPC does not has access to internet). 
* An EC2 instance with no public IP address. This is for you to test the VPN communication between the two environments.

**In the data center VPC:**
* Two private and two public subnets.
* 1 NatGateway 
* In the public subnet (AZ1) you have an EC2 instance already configured with OpenSwan software, just to serve as Customer’s Router (Customer Gateway Device).
* In the private subnet (AZ1) behind the router resides the postgres database to which AWS Glue is going to connect and also 2 EC2 instances, one called *PrivateInstanceToPostgresDCVPC* which is going to put the sample data inside of the DB and *DC-VPC-private-instance* which is going to be used to ping the Cloud-VPC instance to test VPN connection.

**AWS Site-to-Site VPN already created and fully configured:**
* Go to AWS VPC console and on the left pane, select Site-to-Site VPN connections and on the Tunnel Details tab check that only one tunnel is up. 
* The reason why only one tunnel is up is because of the OpenSwan configuration. This is not highly available and redundant but sufficient for this demo.

**In the AWS Glue console:**
* An AWS Glue Connection already configured with the JDBC connector to the postgres database.
* An AWS Glue Crawler which is used to scan and discover the database schema of your dataset and it is going to use the JDBC connection created.
* An AWS Glue database which is where the metadata of your dataset is going to be stored.

## Test the Communication between the two environments

Go to AWS Systems Manager and on the left pane click on Managed Instances. Select *DC-Private-Instance* instance and click Actions > Start session. A new tab is opened with the instance’s terminal. Execute this command: ping (privateI) where the private IP is the IP of the *Cloud-VPC-private-instance* instance. 
In my case the command looks like this:
```bash
ping 10.0.1.88
```
If you get this message when executing the command:

![Ping message](/images/ping_message.png)

That means that the VPN connection is working successfully and the communication is done privately.

## Test AWS Glue

Go to AWS Glue console:
* On the left pane click on Connections. You will see one connection listed called “ glueConnection-(+random string) select and click test connection. 
* If the results is successful that means AWS Glue could connect to your On premises DB instance through the VPN connection.
* Now Select Crawlers and run it. This is going to scan the dataset and discover tables and store the dataset schema in the “dbcrawler” AWS Glue database.
* Go and click in tables on the left pane and feel free to explore the dataset schema inside of the on premises database.

`YOU ARE DONE!` The connection was done successfully and you have full connectivity between AWS Glue and your On Premises enviornment.

## Perform an AWS Glue Job (Optional)

Go to AWS Glue Console:
* On the left pane click on Jobs.
* Click on Add Job.
* Put a name and the select the role called "demo-Glue-(+random string)

![Job1](/images/GlueJob_Config1.png)

* leave the rest parameters as default and click next.
* Select the data source *raw_sportstickets_public_players* and click next.
* In the next screen select these and select the bucket that was created for you (the one that has demo-glues3bucket+random string by name):

![Job2](/images/GlueJob_config2.png)

* Click save job and edit script
* In the next step click Run Job button and Run Job again
* On the right corner of console click the X button and wait for the job to finish

![Job3](/images/GlueJob_Config3.png)


* The Job has succeeded!! Now go to Amazon S3 console and click on the S3 bucket created by the template and expolore the procesed data


![Job4](/images/GlueJob_Config4.png)


* The files are loaded in Parquet format so all the ETL process is Done. `CONGRATULATIONS!`

## Authors

* Daniel Neri
* Federica Ciuffo

