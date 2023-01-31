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

![image](https://user-images.githubusercontent.com/91480603/215847368-a6a28d53-084c-4427-b265-fb32f7baeea9.png)

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

Copy a **healthy.html** file in the **/var/www/html** folder

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

![image](https://user-images.githubusercontent.com/91480603/215815676-f081d2e9-cfdb-4ed9-881a-5e5378067907.png)

**Sorry, but I can’t write the wp-config.php file** error screen  - Copy/paste text
  
![image](https://user-images.githubusercontent.com/91480603/215815846-f2928cd6-b56a-4ecb-b0aa-b41f7e8af0ee.png)

nano wp-config.php <edit> Copy/Paste - Ctrl-X - Y
  
Go back to GUI and click “Run the Installation”

Complete the welcome screen to install WP

![image](https://user-images.githubusercontent.com/91480603/215816080-85eae22c-1fcf-4639-b219-a1eec3f0f98b.png)

Login to WP
  
![image](https://user-images.githubusercontent.com/91480603/215816292-b0e382f3-6d0c-4df1-b363-7bd495c0bd1c.png)

See the WP Dashboard
  
![image](https://user-images.githubusercontent.com/91480603/215816479-89527d93-aa79-4dee-b223-199483bb4423.png)

Add post
Add photos to Media
Check website post
Right click copy image address
Paste in browser and confirm EC2 Public IP address

![image](https://user-images.githubusercontent.com/91480603/215816883-269236e8-7db7-498e-9ea3-984ffbe11abc.png)

Media files should be located on EC2 in wp-content folder
  
![image](https://user-images.githubusercontent.com/91480603/215817075-60f89b1f-3922-4a46-9223-1fedfd637ecc.png)

**CMD line copy files to S3**

aws s3 cp /var/www/html/wp-content/uploads/2022/02 s3://markbradley-wp-media –recursive
  
aws s3 cp /var/www/html s3://markbradley-wp-code –recursive

After copy

Check copy

aws s3 ls s3://markbradley-wp-code

**URL rewrite rule for CloudFront**
  
**Networking & Content Delivery > CloudFront** -  Distributed domain name – Copy/Paste
  
![image](https://user-images.githubusercontent.com/91480603/215841340-4cbe2dac-9f3a-4c93-9603-17deb2021281.png)

cd /var/www/html

nano .htaccess

![image](https://user-images.githubusercontent.com/91480603/215841533-65ca0966-df39-42c0-a898-4d1e150a74ef.png)

Edit current CloudFront URL to include CloudFront Distributed domain name > Copy/Paste

Ctrl+X – Y

aws s3 sync /var/www/html s3://markbradley-wp-code

This will upload and change the edited .htaccess file in S3

This file will now point to CloudFront

**Configure Apache for URL rewrites**

cd /etc/httpd

ls
  
cd conf

ls

![image](https://user-images.githubusercontent.com/91480603/215841906-90a2aaab-5f38-4360-bbf5-bc6f373d1ac2.png)

nano httpd.conf
  
![image](https://user-images.githubusercontent.com/91480603/215842024-9ed6322b-2c33-44af-9550-fca22a0544db.png)

Find
  
![image](https://user-images.githubusercontent.com/91480603/215842200-d8fca013-5c4a-417f-859b-7ca0e1d2e187.png)

Edit AllowOverride ALL
  
![image](https://user-images.githubusercontent.com/91480603/215842320-562253b0-d7ed-4d0c-a4a3-8523cdf1b3d3.png)

Ctrl+X - Y

service httpd restart

**Validate CloudFront is now hosting media files**
  
Open WP in browser and check post and images 

Right click copy image address

Paste in browser and confirm CloudFront address and not EC2 instance

![image](https://user-images.githubusercontent.com/91480603/215842509-0997af6f-0782-4084-9ddc-e6a40b807355.png)

**Create Application Load balancer**
  
**Compute > EC2 > Load balancers** – Create load balancer
 
![image](https://user-images.githubusercontent.com/91480603/215842978-3a634865-e6d5-40da-b052-66b50f2249eb.png)

Application Load Balancer – **HTTP HTTPS**
  
Internet Facing - **IPv4 – HTTP - All Availability Zones**

Select Existing Security Group - **WebDMZ**

New Target Group - **MyWPInstances**

Target Type: Instance - Protocol: **HTTP - Port: 80**

Health Check - Protocol: **HTTP** - Path: **/healthy.html**
  
![image](https://user-images.githubusercontent.com/91480603/215843619-9a126d91-963b-4551-bfad-e60fadcce224.png)

**Configure ReadNode EC2 for AMI to download S3 content and use for Auto Scale Group**

**Configure WriteNode EC2 to upload S3 content**
  
**Setup a cron job**
  
The cron command-line utility is a job scheduler that can run tasks at fixed times, dates, or intervals

**Login EC2**
  
cd /etc

nano crontab
  
![image](https://user-images.githubusercontent.com/91480603/215844027-05c9f25a-e0d4-43f3-8981-4c01b1aa82a5.png)

*/1 * * * * root aws s3 sync --delete s3://markbradley-wp-code /var/www/html

Ctrl+X > Y  > service crond restart
  
![image](https://user-images.githubusercontent.com/91480603/215844236-615e9ec3-adbb-4b93-a4d3-4d1ca40e0519.png)

This task will download the S3 folder contents for the read node instance – Copy from S3 to local folder

**Test update to S3**
  
Add file hello.txt to s3://markbradley-wp-code

Check EC2

cd /var/www/html
 
hello.txt

cat hello.txt > displays text content

**Create Template for WPReadNode EC2**
  
**Compute > EC2** - Select instance > Action > Image > Create Image

Image Name: WPReadNode

Image Description: This is the default read node for WP

**Images > AMIs** - check image is available

![image](https://user-images.githubusercontent.com/91480603/215844728-e6aaaaf0-6cae-46b0-9cd7-ca613ae210d5.png)

**Login to WP WriteNode EC2**
  
sudo su

cd /var/www/html/etc

clear

nano crontab

*/1 * * * * root aws s3 sync --delete /var/www/html s3://markbradley-wp-code

*/1 * * * * root aws s3 sync --delete /var/www/html/wp-content/uploads s3://markbradley-wp-media

Ctrl+X – Y

This task will upload the local folder contents to S3 for the write node instance.

**Test update to S3**
  
cd /var/www/html

echo "This is a TEST" > test.txt

service crond restart

service httpd status
  
**Check S3**
  
Check s3://markbradley-wp-code has test.txt file

**Setup Auto Scaling Groups using new AMI (read nodes)**

**Compute > EC2 > Auto Scaling Groups**

**Create Launch Template** – LCforWP - My AMIs – WPReadNode – T2.Micro – IAM role: S3ForWP Security Group: WebDMZ

![image](https://user-images.githubusercontent.com/91480603/215845389-81eef13d-7af9-48f4-afb1-4a4c407accad.png)

**Compute > EC2 > Target Groups** – Create target group

MyWPInstances – Target type: instance – HTTP – 80 – Default VPC – HTTP - /healthy.html

Actions – Register and deregister instance / IP targets

Check registered targets – healthy

![image](https://user-images.githubusercontent.com/91480603/215845672-8838dcbb-61d9-4dde-a075-6ea9d5598f6b.png)

**Create Auto Scaling Group** – ASGforWP – 3 instances – default VPC – all subnets

Check - Receive traffic from one or more load balancers

Target Group: MyWPInstances Health Check Type: ELB

![image](https://user-images.githubusercontent.com/91480603/215845868-c9dea79e-db22-439f-8876-6effd7a15d5d.png)

**Post implementation validation checks**

**EC2 Dashboard**
  
Check instances running

**Target Groups**

Check *all* registered targets - WPReadNode – healthy

**WP website**
  
Validate and open HTTP page with domain to view WordPress site hosted via Load Balancer

Open image file to see page and validate CloudFront address.
  
Edit Write Node EC2 and content is auto copied to S3 and S3 downloaded to Read Nodes EC2

Website is updated with new content

Check autoscaling by terminating running RN instances and verify new replacement instances

Auto Scaling Group – Activity History will confirm terminated instances and new EC2 instances

Check RDS by rebooting instance - primary will failover to secondary

  
**Optional Route 53 > Hosted zones > Create hosted zone**
  
Create a hosted zone for your domain name

**Add Domain Name to WP site**

After creating Load balancer – Route 53 Alias

**Networking & Content Delivery > Route 53**

Hosted zones – Domain name

Create Record Set

A

Alias - Y

Alias Target: <name of load balancer>

Routing Policy: Simple

Evaluate Target Health - No

![image](https://user-images.githubusercontent.com/91480603/215846557-36a12414-0cec-4a13-b665-b2becf973d80.png)


**Troubleshooting**
  
WordPress versions

**https://codex.wordpress.org/WordPress_Versions**

WordPress and PHP Version Compatibility

**https://make.wordpress.org/core/handbook/references/php-compatibility-and-wordpress-versions/**


