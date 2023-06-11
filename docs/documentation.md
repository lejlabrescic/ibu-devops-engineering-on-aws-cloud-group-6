

# ⚡️Detailed project documentation
### Intro to the project 

Example University is preparing for the new school year. The admissions department has received complaints that their web application for student records is slow or not available during the peak admissions period because of the high number of inquiries.

The aim of the project was to build and deploy a highly scalable and available web application, with the minimum cost possible. 
It needs to work without any possible downtime and to respond quickly and accordingly, in order to handle errors. 

Since the project can become complex, the best approach is to divide it into multiple steps-phases to ensure that every functionality is working correctly. The picture of the web application can be founded below. 
![baza](https://github.com/lejlabrescic/ibu-devops-engineering-on-aws-cloud-group-6/assets/92021817/145320e5-403a-4512-8cca-badf72558c05)


##  👩‍💻 Phase 1 - Planning the design diagrams and estimating cost 
### 💬 Design diagram 
On the following picture, we can see the diagram of the project, whose detailed description can be seen in the following phases & tasks.
![initiač](https://github.com/lejlabrescic/ibu-devops-engineering-on-aws-cloud-group-6/assets/92021817/0d98bb6d-91af-49f8-84b1-f99c4f31e6fc)



### 💬 Cost estimation 
Cost estimation was pretty easy to do after finishing the design diagram for the project. For our project, we needed to calculate the total monthly price for all services that our project needs. We carefully looked not to exceed budget and not to enable additional resources that will increase the total price. The full price and total review of costs can be found on the following [link](https://github.com/lejlabrescic/ibu-devops-engineering-on-aws-cloud-group-6/blob/main/docs/Cost%20estimation%20-%20updated.pdf) . The total price after adding all the necessary resources is around *$65*. 

##  🧠 Phase 2 - Creating a basic functional web application
During this phase, we will build the solution to host web application on the AWS Cloud.
### 💬 Task 1 - Creating a virtual network
In the first task, we built a Virtual Private Cloud (VPC) to create a secure network setting for our application. When creating a VPC we chose an option to automatically create Private and Public Subnets, Internet Gateways, Route Tables, and NAT Gateway. We also needed to create private and public Security Groups. 
These services and options help us construct safe and scalable environments for our application and are crucial components of setting network infrastructure in AWS.
Two private subnets and two public subnets allow us to efficiently separate and manage various components. In order to allow connectivity between the VPC and the internet, an internet gateway was created, and route tables were put up to direct traffic inside the VPC. In addition, a NAT gateway was installed to provide private subnet resources with safe outbound internet access.

Security groups serve as virtual firewalls for  EC2 instances. They regulate incoming and outgoing traffic at the instance level, enabling to specify particular rules that decide which kinds of communications are approved or rejected. By implementing security guidelines and filtering network traffic, security groups assist in safeguarding your resources. We opened port 80 for inbound traffic.

### 💬 Task 2: Creating a virtual machine
Because EC2 offers a scalable and adaptable virtual server environment, we launched an EC2 instance in the cloud to host the web application. We set up an EC2 instance in a public subnet and configured it to host both the database and the application. 
### 💬 Task 3: Testing the deployment
In this task, we needed to perform tests on the application to ensure that it is accessible from the internet and functional.
Diagram for the phase 2 is seen below. 

![faza2](https://github.com/lejlabrescic/ibu-devops-engineering-on-aws-cloud-group-6/assets/92021817/3414e2e3-a5fe-475d-b55f-11e5d7769b1f)



##  🤔 Phase 3 - Decoupling the application components
In this phase of the project, we are dealing with the migration of database from EC2 instance to the Amazon RDS. The aim is to have the separate infrastructure for database (Amazon RDS) and web server (EC2 Instance).
In order to do that, we will use the existing instance, but we will need to reconfigure its details, in order to use it for this task. 

### 💬 Task 1 - Changing the VPC configuration
Since in the Phase 2, we have already created two private subnets in two separate availability zones, we can skip this step. By looking at the design diagrams, we can see that we have private subnets in us-east-1a region and us-east-1b region.
### 💬 Task 2: Creating and configuring the Amazon RDS database
In this task, we are creating the MySQL type Amazon RDS database. For the database, we need subnet groups and security group. We also need to provide the password and username for our database, that will be used later. After adding the required features to the database and after waiting a couple of minutes, the database is successfully created.

### 💬 Task 3: Configuring the development environment
For this task, we are creating a Cloud 9 instance in order to use the C9 console. We are choosing that EC2 creates a new instance for  Cloud 9 environment and SSH for the connection.

### 💬 Task 4: Provisioning Secrets Manager
- Secrets Manager is used to store secrets for the database. We can add it manually, or we can use the C9 console and the following command : 
```bash
  aws secretsmanager create-secret \
    --name Mydbsecret \
    --description "Database secret for web app" \
    --secret-string "{\"user\":\"<username>\",\"password\":\"<password>\",\"host\":\"<RDS Endpoint>\",\"db\":\"<dbname>\"}"
```
We can check if our command is successful, by opening the Secrets Manager in AWS CLI and seeing that Mydbsecret is showing on the secrets section.

### 💬 Task 5: Provisioning a new instance for the web server

For this task, we have used the old instance that we created in the 1st Phase, but we needed to add a couple of changes. 
The changes include adding the inbound rules for the EC2 Instance's security group, including the following : 

- adding the inbound rules that allows all ports for ICMP protocol, where the source is the Cloud 9 security group.
- SSH - opening port 22 for the database, where the source is again the Cloud 9 security group.
- opening ports for TCP, where for the source we are using C9 SG. 
The reason we are opening the following ports is to ensure the better troubleshooting and for pinging the remote hosts. 
After this task, we can work on the migration of database from EC2 to RDS, since we have enabled ports for data transfer and SQL usage.

### 💬 Task 6: Migrating the database
In order to migrate the database, we will use the C9 sonsole and paste the following script : 
```bash
  mysqldump -h <EC2instancePrivateip> -u nodeapp -p --databases STUDENTS > data.sql
```
To import the data into the database, we use: 
```bash
mysql -h <RDSEndpoint> -u nodeapp -p  STUDENTS < data.sql
```
####  👯‍♀️ Problems 
In this task, we have encountered a few problems. Firstly, we encountered an error that says 'Unknown database STUDENTS'. 
we solved this error by rewriting the first command in this way: 
```bash
mysql -h <RDSEndpoint> -u nodeapp -p  -e "CREATE DATABASE STUDENTS"
```
After this, we have been prompted to enter the password, and the data migration was successful.


### 💬 Task 7: Testing the application
In this part, we first opened a public IP address of the EC2 instance, in order to ensure that functionalities are still working correctly. We inserted and deleted a few rows, thus the migration of data was successful.

The project diagram for this phase is below. 

![faza 3](https://github.com/lejlabrescic/ibu-devops-engineering-on-aws-cloud-group-6/assets/92021817/ab28921c-f66d-4067-9551-3cf00d55f3c8)


## ⚡️ Phase 4: Implementing high availability and scalability

In this phase, after creating a separate infrastructures for database and web server, we want to ensure that our application is working correctly and that it is prone to answer to sudden changes.  We are using the ELB and its services to obtain the properties of high availability and scalability. 

### 💬 Task 1 - Creating an Application Load Balancer
In this task, we are creating an Application Load Balancer. While creating the balancer, we are also creating the AMI (Amazon Machine Image) and Target group.  We are also using 2 AZ  and 3 subntes and existing EC2's instance SG. By doing so, we can distribute the incoming traffic among multiple instances in the private subnet, ensuring better performance and fault tolerance.
We also need to create the Launch template.
### 💬 Task 2: Implementing Amazon EC2 Auto Scaling
To create the AutoScaling group, we will use the existing load balancer that we created. We put the option to have the two instances running. After creating the Auto Scaling, we performed a few tests. We tried to put one of the instances to terminated state. After that, we could have seen that the new instance was created as the backup.
### 💬 Task 3: Accessing the application
We accessed the application and saw that it's still working properly. 
### 💬 Task 4: Load testing the application
In order to load test the application, we prompted the input in C9 console to install the load test:
```bash
npm install -g loadtest
```
After that, we input the test: 
```bash
loadtest --rps 1000  -c 500 -k <<ELB URL>>
```
#### 👩‍💻 Testing 
In order to test the application, we can open the public DNS of the load balancer. If it displays the application, the options are enabled and everything is working in order. 

## ⚡️ Conclusion

The goal was to gain a thorough understanding and hands-on experience with AWS services by successfully accomplishing a project that covers multiple aspects of architecture design, cost estimation, deploying web applications, configuring networks, setting up security measures, ensuring high availability and scalability, and managing access permissions.
  
The final projects diagram can be seen below. 
  
![diagram](https://github.com/lejlabrescic/ibu-devops-engineering-on-aws-cloud-group-6/assets/92021817/d194d390-3f19-4d90-a8df-271033058b7f)


### 🚀 By finishing all the steps and tasks, our project has come to an end. 
<p align="center">
  <img src="https://www.metaltoad.com/sites/default/files/styles/large_personal_photo_870x500_/public/2020-05/aws-logo-blog-header.png?itok=t4o3meiH" /></p>


 




