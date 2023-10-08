# Problem Statement:

You work for XYZ Corporation. Your corporation wants to launch a new web-based application using AWS Virtual Machines. Configure the resources accordingly for the tasks.

# Tasks To Be Performed:
1. Create an instance in the US-East-1 (N. Virginia) region with an Ubuntu OS and install Nginx for making them web servers.
2. Change the default website with a page displaying the message: “HelloWorld"




# Task 1: Create an instance in the US-East-1 (N. Virginia) region with an Ubuntu OS and install Nginx for making them web servers.

# Solution:
To accomplish Task 1, follow these steps:

Login to AWS Console:

Go to the AWS Management Console (https://aws.amazon.com/).
Sign in with your AWS account credentials.
Launch an EC2 Instance:

Navigate to the EC2 Dashboard.
Click on "Launch Instance."
Select the "Ubuntu Server" as the operating system for your instance.
Choose an Instance Type:

Choose an instance type based on your application's requirements (e.g., t2.micro for a small-scale application).
Configure Instance Details:

In the "Configure Instance Details" section, choose the following:
Number of instances: 1
Network: Your desired VPC
Subnet: Your desired subnet
Auto-assign Public IP: Enable
Add Storage (Optional):

Configure the instance storage as per your needs. The default settings are usually sufficient for a basic setup.
Add Tags (Optional):

You can add tags for easier resource management.
Configure Security Group:

Create or select a security group that allows incoming traffic on ports 80 (HTTP) and 22 (SSH) for Nginx and SSH access.
Review and Launch:

Review your configuration and click "Launch."
Create a Key Pair (if needed):

If you don't have an existing key pair, create a new one. This key pair is used to connect to your instance securely.
Launch the Instance:

Click "Launch Instances."
Access the Instance:

Wait for the instance to launch.
Once the instance is running, you can connect to it using SSH and your key pair.
Install Nginx:

Connect to the instance using SSH.
Update the package list: sudo apt-get update
Install Nginx: sudo apt-get install nginx
Start the Nginx service: sudo service nginx start
Task 2: Change the default website with a page displaying the message: “Hello World"

# To accomplish Task 2, follow these steps:

Create a "Hello World" HTML Page:

Create a simple HTML page with the "Hello World" message. You can use a text editor like nano or vim to create the file.
```
sudo nano /var/www/html/index.html
```
Add the following content to index.html:

```
<!DOCTYPE html>
<html>
<head>
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
```
Save the File and Exit the Text Editor:

In nano, press Ctrl+O to save the file, then press Enter.
Press Ctrl+X to exit nano.
Restart Nginx:

After saving the HTML file, restart Nginx to apply the changes:
```
sudo service nginx restart
```
Access the Web Page:

Open a web browser and enter the public IP address or DNS name of your EC2 instance. You should see the "Hello World" message.
You have successfully created an AWS EC2 instance with Ubuntu, installed Nginx, and changed the default web page to display "Hello World." Your web server is now up and running in the US-East-1 (N. Virginia) region.
