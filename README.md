### AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY
------
#### General Overview
You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) for a fictitious company named WakaBetter that uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this. Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost. 

![](./images/Architecture-Diagram.png)

### Requirements
There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit Watch How To Do This Here
    * Create an AWS Master account. (Also known as Root Account)
    * Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this) 

    ![](./images/aws-organizations.png)

    ![](./images/aws-organizations.png)

    ![](./images/create.png)

    ![](./images/devops.png)

    * Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there).

    ![](./images/ou.png)

    ![](./images/dev.png)

    * Move the DevOps account into the Dev OU. 
    ![](./images/move.png)

    ![](./images/move-2.png)

    ![](./images/dev-devops.png)

    * Login to the newly created AWS account using the new email address.

    ![](./images/devops2.png)

2. Create a domain name for your company. I used [namecheap](https://www.namecheap.com/).
3. Create a hosted zone in AWS, and map it to your domain. 

 ![](./images/route53.png)

 ![](./images/create-hosted-zone.png)

 ![](./images/savvytek.png)

 ![](./images/ns.png)

 ![](./images/ns-2.png)

 ### SET UP A VIRTUAL PRIVATE CLOUD (VPC)
 --------
1. Create a VPC

 ![](./images/create-vpc.png)

 ![](./images/vpc-config.png)

2. Enable DNS hosting

 ![](./images/enable.png)

 3. Create internet gateway as shown in the architecture.

 ![](./images/igw.png)

 ![](./images/create%20igw.png)

 ![](./images/savvytek-igw.png)

 ![](./images/attack-to-vpc.png)

 ![](./images/attach-vpc-2.png)

4. Create the subnets as shown in the Architecture

Use this website to get the CIDR blocks easily. [ipinfo](https://ipinfo.io/ips).

![](./images/create-subnet.png)

![](./images/create-subnet-vpc-id.png)

![](./images/create-subnet-2.png)

![](./images/create-subnet-3.png)

![](./images/create-subnet-4.png)

![](./images/create-subnet-5.png)

![](./images/create-subnet-6.png)

![](./images/subnets.png)

5. Create a route table and associate it with public subnets

![](./images/route-table-1.png)

![](./images/edit-subnet-associations-1.png)

![](./images/edit-subnet-associations-2.png)

![](./images/public-route-table.png)

6. Create a route table and associate it with private subnets

![](./images/route-table-2.png)

![](./images/edit-subnet-associations-3.png)

![](./images/edit-subnet-associations-4.png)

![](./images/route-tables.png)

7. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet).

 ![](./images/edit-routes.png)

 ![](./images/edit-routes-2.png)

 8. Create 3 Elastic IPs
 ![](./images/elastic-ip-1.png)

 ![](./images/elastic-ips.png)

 9. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts).
![](./images/ngw-1.png)

![](./images/ngw-2.png)

![](./images/ngw-3.png)

![](./images/edit-route-3.png)

![](./images/edit-route-4.png)

10. Create a Security Group for:
    * External Application Load Balancer: External ALB will be accessible from the Internet

    ![](./images/create-SG.png)

    ![](./images/SG-ALB.png)


    * Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address.

    ![](./images/bastion-sg.png)

    * Nginx Reverse Proxy Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB).

    ![](./images/nginx-sg.png)

    ![](./images/nginx-sg-2.png)

    * Internal Application Load Balancer: ALB will be available from the Nginx servers

    ![](./images/internal-alb-sg.png)

    * Webservers: Access to Webservers should only be allowed from the Internal Application Load Balancer. 

    ![](./images/web-server-sg.png)

    * Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully designed – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

    ![](./images/data-layer-sg.png)

### TLS Certificates From Amazon Certificate Manager (ACM)

You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

* Navigate to AWS ACM

![](./images/acm.png)

![](./images/request-certificate.png)

![](./images/request-public-certificate.png)

![](./images/request-public-certificate-2.png)

![](./images/request-public-certificate-3.png)

![](./images/create-records-in-route53.png)

![](./images/create-records.png)

![](./images/issued.png)

Configure EFS
---
* Create a new EFS File system. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer. Associate the Security groups created earlier for data layer.

![](./images/efs.png)

![](./images/create-file-system.png)

![](./images/create-efs-customize.png)

![](./images/mount-tagets.png)

![](./images/file-system-policy.png)

![](./images/create-for-file-system.png)

![](./images/savvytek-EFS.png)

* Create 2 access points - ! for each of the website (wordpress and tooling) so that the files do not overwrite each other when we mount. 

![](./images/create-access-point.png)

![](./images/efs-details-1.png)

![](./images/efs-details-2.png)

![](./images/create-access-point-2.png)

![](./images/create-access-point-3.png)

![](./images/details-tooling.png)

![](./images/details-tooling-2.png)

![](./images/create-access-point-4.png)

### Configure RDS
-----
#### Pre-requisite:

* Create a KMS Key