# AWS LAMP-Stack

**What is does**

We will create a fault tolerant WordPress site with high availability. A WP Admin will manage the site and content directly accessing on an EC2 instance that will upload both code and media to S3. Users will access the website through a load balancer to a group of EC2 instances that will download the code and media from S3. CloudFront will host and cache media files for performance.

This could be a typical small **proof of concept** example for a customer. We can use this as a real-life case study to create a fault tolerant WordPress site. We'll skip through the steps in AWS and will not document every small configuration detail. Please note this is a temporary environment and security considerations may be open. Production would use HTTPS 443 instead of HTTP port 80 and other restrictions.

**High Level Design**

![image](https://user-images.githubusercontent.com/91480603/215809343-22e29ffb-bcd4-4a9a-bcf0-3e7906bfddeb.png)

**Compnents Used**

- IAM Roles
- Security Groups
- VPC
- EC2
- S3
- Elastic Load Balancer
- Availability Zones
- Auto Scaling Groups
- Amazon RDS Multi-AZ MySQL
- CloudFront
- Route 53
- Linux
- Apache
- PHP
- WordPress

**Set up**

**Login to AWS Console**

![image](https://user-images.githubusercontent.com/91480603/215811431-b53970e6-28ce-4e11-8f16-86785840973a.png)

**Networking & Content Delivery > VPC > Security Groups**

Create two new security groups in my default VPC
- WebDMZ - port 22, 80 - source: 0.0.0.0/0
- RDS-MYSQL/Aurora - port 3306 - source: Security group name for WebDMZ

**Database > RDS > Create database**

![image](https://user-images.githubusercontent.com/91480603/215812295-88421dd4-e644-4603-8180-d314b9224fac.png)

**MySQL – version 5.7.22**

Production Template which will allow Multi-AZ DB instance

![image](https://user-images.githubusercontent.com/91480603/215812544-e5e31656-d2eb-4fba-8afb-ed021af2c43b.png)

Enter a DB instance identifier name with Master name and password

For test and billing purposes we will use a T2 Micro with 20GB General Purpose SSD

Create DB instance that is not publicly accessible and in Default VPC choosing existing VPC security groups - **RDS-MYSQL/Aurora**

**S3 > Create bucket**
Create two S3 buckets
**markbradley-wp-code** / Bucket and objects not public
**markbradley-wp-media** / Public Access

**CloudFront > Distribution > Create distribution**

Create a distribution and use the S3 media bucket for the Origin Name and S3 FQDN for Origin Domain

![image](https://user-images.githubusercontent.com/91480603/215813193-6660ed0f-1ddd-4bc2-b4a7-57ce80c655b9.png)

**Security, Identity, & Compliance > IAM > Roles > Create role**

AWS service - **EC2**
Permissions policies > **AmazonS3FullAccess**
Role name: **S3forWP**

**EC2**
**Launch instance > Amazon Linux 2 AMI**

![image](https://user-images.githubusercontent.com/91480603/215813900-8575cb40-8a88-4745-8ac2-5c5408d21b04.png)

T2 micro - all default except IAM role: **S3ForWP** - Advanced Details - User data - **bootstrap script**

![image](https://user-images.githubusercontent.com/91480603/215814195-c3fa4397-2095-43fe-bf22-03ed1551596f.png)

**Bootstrap script**

Update
Install Apache, PHP, PHP-MySQL packages
Copy a healthy.html file in the /var/www/html folder
Download/install WordPress

Add Tag: Name / **WPWriteNode**
Select **WebDMZ** for Security Group (port 22, 80)

**Login to EC2 using Public IPv4 address with Key pair**

![image](https://user-images.githubusercontent.com/91480603/215814875-6024780c-5bc7-4b5c-9066-84f9c86b47a1.png)

Elevate privileges to <root> sudo su
Check WordPress files present
Check Apache service is started

**WordPress Configuration**
**Database > RDS > Databases** - Select database and copy Endpoint info
Open browser and EC2 IP address for WordPress GUI
Enter *endpoint* for Database Host and enter Database name, username, and password

