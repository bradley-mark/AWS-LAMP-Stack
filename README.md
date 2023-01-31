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
