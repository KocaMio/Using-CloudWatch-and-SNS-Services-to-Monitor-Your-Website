# Using CloudWatch and SNS Services to Monitor Your Website

## Scenario
The following procedures help you install an Apache web server with PHP and MySQL support on your Amazon Linux instance. You can use this server to host a static website or deploy a dynamic PHP application that reads and writes information to a database.

Please referred to following architecture of this lab.

![1.jpg](/images/1.png)

## Prerequisites
>The workshop’s region will be in ‘N.Virginia’

## Lab tutorial
### Create Your VPC
1.    In the AWS Management Console, on the service menu, click **VPC**.

2.    In the navigation pane, click **Your VPCs**.

3.	  Click **Create VPC**, enter the following details:

* Name tag: My Lab VPC
* IPv4 CIDR block: 10.0.0.0/16

4.	Click **Yes, Create**.

5.	In the navigation pane, click **Internet Gateways**.

6.	Click **Create Internet Gateway**, enter Name tag as **My Lab IGW**.

7.	Click **Create**.

8.	Choose **My Lab IGW**, click **action** button and choose **Attach to VPC**, and then choose **My Lab VPC** you created, click **Attach**.

9.	In the navigation pane, click **Subnets**.

10.	Click **Create Subnet**, enter the following details:

* Name tag: Public Subnet 1
* VPC: My Lab VPC
* Availability Zone : us-east-1a
* IPv4 CIDR block: 10.0.0.0/24

11.	Click **Create**.

12.	Click **Create Subnet**, enter the following details:

* Name tag: Public Subnet 2
* VPC: My Lab VPC
* Availability Zone : us-east-1b
* IPv4 CIDR block: 10.0.10.0/24

13.	Click **Create**.
 
14.	In the navigation pane, click **Route Tables**. Then choose route table of **My Lab VPC**, in the **Routes** tab, click **Edit**, click **Add another route**, enter **Destination: 0.0.0.0/0**, **Target: igw‐xxxxxxxx**. Click **Save**.

15.	In the **Subnet Associations** tab, click **Edit**, check both **Public Subnet 1** and **Public Subnet 2**, then click **Save**.


### Launch an EC2 instance

1.	In the AWS Management Console, on the service menu, click **EC2**.

2.	Click **Launch Instance**.

3.	In the navigation pane, choose Quick Start, in the row for **Amazon Linux AMI 2018.03.0 (HVM)** which ami id is ami-cfe4b2b0, click **Select**.

![2.png](/images/2.png)

4.    On **Step2: Choose a Instance Type page**, make sure **t2.micro** is selected and click **Next: Configure Instance Details**.

5.    On **Step3: Configure Instance Details page**, enter the following and leave all other values with their default:
* Network: My Lab VPC
* Subnet: Public Subnet 1
* Auto-assign Public IP: click **Enable**

6.    Click **Next: Add Storage**, leave all values with their default.

7.    Click **Next: Add Tag**.

8.    On **Step5: Tag Instance page**, enter the following information:

* Key: Name
* Value: Lab Server

9.    Click **Next: Configure Security Group**.

10.   On **Setp6: Configure Security Group page**, click create a new security group, enter the following information:

* Security group name: LabSecurityGroup
* Description: Enable SSH, HTTP and HTTPS access

11.   Click **Add Rule**.

12.   For Type, click SSH (22), HTTP (80) and HTTPs (443).

![3.png](/images/3.png)

13.   Click **Review and Launch**.

14.   Review the instance information and click **Launch**.

15.   Click Create a new key pair, enter the Key pair name (ex. amazonec2_keypair_virginia), click **Download Key Pair**.

16.   Click **Launch Instance**s.

17.   Scroll down and click **View Instances**.

18.   Wait until Lab Server shows 2/2 checks passed in the **Status Checks** column. This will take 3‐5 minutes. Use the refresh icon at the top right to check for updates.

### Connect to EC2 reference below
* [**AWS Documents**](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html)
* [**Appendix: Connecting to Your Instance from Windows or Linux/OSX  Using PuTTY**](#Appendix). 

### Installing and start the Lab server on EC2


1.    Update all your software package, enter:

    [ec2-user ~]$ sudo yum update -y

2.    Use yum install command to install multiple software packages and all related dependencies at the same time.

    [ec2-user ~]$ sudo yum install -y httpd24 php70 mysql56-server php70-mysqlnd

3.    Start Apache web server.

    [ec2-user ~]$ sudo service httpd start
    Starting httpd: 				[ OK ]

4.    Use the **chkconfig** command to configure the Apache web server to start at each system boot.

    [ec2-user ~]$ sudo chkconfig httpd on
    [ec2-user ~]$ chkconfig --list httpd 
    httpd		0:off	1:off	2:on	3:on	4:on	5:on	6:off

>Note: The chkconfig command does not provide any confirmation message when you successfully use it to enable a service. You can verify that **httpd** dos on by running the following command:


5.    Test your web server. In a web browser, enter Public DNS address (or the public IP address) of your EC2; you should see the Apache test page.

![10.png](/images/10.png)


### Create a VPC security group for the RDS DB Instance

1.	In the AWS Management Console, on the service menu, click **VPC**.

2.	In the navigation, click **Security Groups**.

3.	Click **Create Security Group**.

4.	In the Create Security Group dialog box, enter the following details:
* Name tag: DBSecurityGroup
* Group name: DBSecurityGroup
* Description: DB instance security group
* VPC: Click My Lab VPC

5.	Click **Yes, Create**.

6.	Select **DBSecurityGroup** you just created.

7.	Click the **Inbound Rules** tab, and then click **Edit**.

8.	Create an inbound rule with the following details:
* Type: MySQL/Auora(3306)
* Protocol: TCP(6)
* Source: Paste Security Group ID of LabSecurityGroup
>Search LabSecurityGroup to find sg-xxxxxxxx
 
9.	Click **Save**.


### Create Private Subnets for Your Amazon RDS Instances

1.    In the navigation pane, click **Subnets**.

2.	Click Create Subnet dialog box, enter the following details:
* Name tag : Private Subnet 1
* VPC : Select My Lab VPC
* Availability Zone : us-east-1a
* CIDR block : 10.0.1.0/24

3.	Click **Create**.

4.	Click **Create Subnet**.

5.	In Create Subnet dialog box, enter the following details:
* Name tag : Private Subnet 2
* VPC : Select My Lab VPC
* Availability Zone : us-east-1b
* CIDR block : 10.0.11.0/24

6.	Click **Create**.


### Create DB Subnet Group
In order to build RDS DB instance in subnetgroup, we create subnet group. 

1.	In the AWS Management Console, on the service menu, click **RDS**.

2.	In the navigation pane, click **Subnet Groups**.

3.	In the navigation pane, click **Create DB Subnet Groups**.
 
4.	On the Create DB Subnet Group page, enter the following details:

* Name: dbsubnetgroup
* Description: Lab DB Subnet Group
* VPC ID: Click My Lab VPC

5.	To add subnets to subnet group, click Availability zone, choose **us-east-1a**, click subnet, choose **10.0.1.0/24**, then click **Add subnet**.

6.	Choose another Availability Zone **us-east-1b**, click the Availability Zone you selected for Private Subnet 2. For Subnet ID, click **10.0.11.0/24**, then click **Add subnet**.

7.	Click **Create**.

8.	If you do not see your new subnet group, click the refresh icon in the upper-right corner of the console.


### Launch a RDS DB instance with MySQL engine

1.	In the AWS Management Console, on the service menu, click **RDS**.

2.	In the navigation pane, choose **Instances**.

3.	Choose **Create database**, on the **Select Engine** page choose **MySQL**.

![11.png](/images/11.png)

4.	In the **Select Engine** window, click the **Next** button for the MySQL DB engine.

5.	On the **Choose Use case** page, click **Production-MySQL**. Click **Next**.

6.	On the **Specify DB details** page, enter the following details:
* DB Instance Class: Choose db.t2.small—1 vCPU, 2GiB RAM.
* Multi-AZ Deployment: Click **Create replica in different zone**.
* DB Instance Identifier: LabDBInstance
* Master Username: labuser (Please remenber this username!)
* Master Password: labpassword
* Confirm  Password: labpassword

7.	Click **Next**.

8.	On the **Configure Advanced Setting**s page, enter the following details and leave all other values with their default:
* VPC: My Lab VPC
* Subnet Group: dbsubnetgroup
* Publicly Accessible: No
* VPC Security Group(s): Select **Choose existing VPC security groups** and select **DBSecurityGroup**, delete **default**.
* Database Name: sampledb
* Encryption: **Disable Encrytion**
* Backup: Backup retention period : **0 days**
* Monitoring: **Disable enhanced monitoring**
* Maintenance : choose **Disable auto minor version upgrade**

9.	Click **Create database**.

10.	Click **View DB Instance details**.

11.	Select **labdbinstance** and scroll down to **DETAILS** wait until **Endpoint** is available or modifying – this may take up to 10 minutes. Use the refresh icon in the top right corner to check for updates.

![12.png](/images/12.png)

12.	Copy and save the **Endpoint**, make sure not to copy the :3306 of your **Endpoint**. **Endpoint** should look similar to the following as below example: **db.choi5coyenv6.us-east-1.rds.amazonaws.com**. You will change localhost to this endpoint later.

>Note: "db.choi5coyenv6" is a random generated number.

![13.png](/images/13.png)


### Use EC2 to connect with database

1.	Log in to your **Lab Server** instance using SSH.

2.	Install MySQL tool.

	    [ec2-user ~]$ sudo yum install mysql

3.	After check state, please key **y** to start install.
    
    	Is this ok? [y/d/N]: y

4.  After installed MySQL, to log-in RDS server, key **mysql ‐h [Endpoint] –u [Username] -p**, and press enter to key **[password]**.

    	[ec2-user ~]$ mysql -h db.choi5coyenv6.us-east-1.rds.amazonaws.com -u labuser -p

>Note: "db.choi5coyenv6" is a random generated number.


5.	If you log-in to RDS, you can see the **mysql>**, and you can use the database.

  	 	mysql> select database();
    	+------------+
   		| database() |
   		+------------+
   		| NULL	|

6.	If you want to leave the RDS, please press **Ctrl+c** to exit.


### Create an alarm in CloudWatch and SNS Service

CloudWatch is a monitoring and management service built for developers. You can use CloudWatch to set high resolution alarms to ensure instances are running smoothly.

1.   In the AWS Management Console, on service menu, click **CloudWatch**.

2.	In the navigation pane, choose **Alarms**, click **Create Alarm**.

3.	For the Select **Metric** step, you can set up the metric.

4.	Choose a metric category - EC2 Metrics.

5.	Select an instance (Lab Server ID) and metric - **CPUUtilization**. Click **Next**.

![14.jpg](/images/14.jpg)

6.	For the **Define Alarm** step, you can set the alarm.

7.	Under Alarm Threshold, type a unique name for the alarm (for example, **myHighCpuAlarm**) and a description of the alarm (for example, **CPU usage exceeds 30 percent**).

8.	Under Whenever, for is, choose > and type **30**. For for:1, type **1**.

![15.jpg](/images/15.jpg)

9.	Under Additional settings, for Treat missing data as, choose **bad (breaching threshold)**, as missing data points may indicate the instance is down.

10.	Under **Actions**, for Whenever this alarm, select State is ALARM. 
* For Send notification to, Click **NEW List**.
* Send notification to: mytopic
* Sendemail : Your email

>You must confirm the subscription before notifications can be sent.

![16.png](/images/16.png)

11.	Choose **Create Alarm**.

12.	When have alarm, the mailbox will get the alarm mail which sent from AWS.

13.	To test SNS function, Log in to your Lab Server instance using SSH.

14.	You can test Alarm function. Try to increase CPU Utilization to 30%. you can Log in to your **Lab Server** though SSH.

    [ec2-user ~]$ sudo yum install stress -y

15.	Burn up CPU in 120s.

    [ec2-user ~]$ sudo stress --cpu 1 --timeout 120

16.	If CPU alarm trggers by Cloudwatch, the email box will get the alarm mail which sent from AWS.

![17.jpg](/images/17.jpg)


## Conclusion

Congratulations! You now have learned how to:
* Logged into Amazon Management Console
* Create an Amazon Linux Instance from and Amazon Machine Image (AMI).
* Find your instance in the Amazon Management Console
* Logged into your instance
* Setup network security included subnets and security group.
* Create an Amazon Relation Database (RDS).
* Logged into your DB instance.
* Monitor service states and send SNS when alarm.



## <h2 id="Appendix">Appendix</h2>

### Connect to your instance (Linux/OSX only)

>Note: This section is for Linux and Max OSX users only. If you are running Windows but have not yet connected to your instance, go back to previous step. If you have already connected to your instance, skip ahead to next step.

1. To connect to your EC2 instance, run the following commands in Terminal:
	
	chmod 400  <path and name of pem>
	ssh –i <path and name of pem> ec2-user@<public IP>

* For **path and name of pem**, substitute the path/filename to the .pem file you downloaded.
* For **public IP**, substitute the public IP address for your **Web Server** instance which you copied into a text editor earlier in the lab.


### Connect to your instance (Windows only)

1. Start PuTTYgen.exe, click **Load**. By default, PuTTYgen display only files with the extension .ppk. to locate your .pem file, select the option to display files of all types.

![4.png](/images/4.png)

2. Select your .pem file **(ex. amazonec2_keypair_virginia.pem)**, and then click **Open**. Click **OK** to dismiss the confirmation dialog box.

3. Click **Save private key** to save the key in the format that PuTTY can use. PuTTYgen displays a warning about saving the key without a passphrase, click **Yes**.

4. Specify the same name for the key that you used for the key pair **(ex. amazonec2_keypair_virginia.ppk)**. PuTTY automatically adds the .ppk extension.

5. Start **PuTTY.exe**, enter **Host Name**, find host name from AWS console. (select Lab Server, and copy the **public IP** value.).

![5.png](/images/5.png)

![6.png](/images/6.png)

6. On the navigation pane, click **Connect>SSH>Auth**, click **Browse** to choose your key pair (ex.**amazonec2_keypair_virginia.ppk**), click **Open**.

![7.jpg](/images/7.jpg)

7. Enter **ec2-user**,and press ENTER.You are now logged into your **Lab Server** instance.

![8.png](/images/8.png)


