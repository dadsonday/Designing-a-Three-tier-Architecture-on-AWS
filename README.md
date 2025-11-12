Architecting for Scale: A Step-by-Step Guide to Building a Three-Tier Architecture on AWS

A three-tier architecture is the backbone of almost all scalable, secure, and highly available web applications. It logically separates an application into three distinct layers: the Presentation (Web) Tier, the Application Tier, and the Data Tier.
This guide walks you through deploying this robust architecture on Amazon Web Services (AWS) using core services like Amazon VPC, Amazon EC2, and Amazon RDS.

<img width="800" height="495" alt="image" src="https://github.com/user-attachments/assets/6ceb23a4-11e4-4029-a399-8f1f176199a5" />

VPC and Subnet Configuration
We will create a custom VPC (e.g., 192.168.0.0/16)  and logically divide it into a set of subnets to enforce security and availability best practices.
Public Subnet (e.g., 192.168.1.0/24): This is for resources that need direct internet access (e.g., Load Balancers, Bastion Host, Web Server).
Private Application Subnet(s) (e.g., 192.168.2.0/24, 192.168.3.0/24): These host the application servers and cannot be reached directly from the internet.
Private Database Subnet(s) (e.g., 192.168.4.0/24): These host the database and are the most restricted.

Crucially, we will place subnets across two different Availability Zones (AZs) to ensure high availability and resilience.

<img width="800" height="371" alt="image" src="https://github.com/user-attachments/assets/999acd40-fcf1-4eb6-8737-26ef42a17b24" />
<img width="1200" height="527" alt="image" src="https://github.com/user-attachments/assets/25d46d56-0860-4641-bd3f-0ed2d5f5d8b8" />
<img width="1014" height="658" alt="image" src="https://github.com/user-attachments/assets/968702cd-8860-4edb-8e56-3f94e40971ac" />
<img width="1200" height="557" alt="image" src="https://github.com/user-attachments/assets/951379e9-79de-446c-bb21-ff1755739400" />

Internet and NAT Gateways
To enable internet access for our tiers:
Internet Gateway (IGW): We create an IGW and attach it to the VPC. This is the entry and exit point for internet traffic to and from the Public Subnet.
NAT Gateway (NAT GW): We deploy a NAT Gateway in the Public Subnet and assign it an Elastic IP (EIP). This allows resources in the Private Subnets (like our App Servers) to initiate outbound connections (e.g., for updates) without exposing them to incoming internet traffic.

<img width="1200" height="557" alt="image" src="https://github.com/user-attachments/assets/69917295-4844-4764-8fc0-bc18dffd738b" />
<img width="1200" height="541" alt="image" src="https://github.com/user-attachments/assets/a3df976c-0cad-4b83-8efa-20d8cd957d7a" />
<img width="1200" height="539" alt="image" src="https://github.com/user-attachments/assets/5454a8e0-4d46-4463-8792-2036b8d2d755" />
<img width="1200" height="523" alt="image" src="https://github.com/user-attachments/assets/558db074-1735-46f2-8326-46d4e4c360bf" />

Route Tables
We then define routing rules to direct traffic:
Public Route Table: Associates the Public Subnet(s) and has a default route (0.0.0.0/0) pointing to the Internet Gateway.
Private Route Table: Associates the Private Subnet(s) and has a default route (0.0.0.0/0) pointing to the NAT Gateway

<img width="1200" height="539" alt="image" src="https://github.com/user-attachments/assets/97263fc1-29e3-47d5-a7c4-90511df088eb" />
<img width="1200" height="517" alt="image" src="https://github.com/user-attachments/assets/7514dbb6-936b-4fa9-9fa8-7478a67b3c11" />
<img width="1200" height="533" alt="image" src="https://github.com/user-attachments/assets/723359f3-2a91-4a23-8c84-04df6d43b445" />
<img width="1200" height="516" alt="image" src="https://github.com/user-attachments/assets/b99691c1-8703-4d9e-8d66-64ffdd332952" />
<img width="1200" height="541" alt="image" src="https://github.com/user-attachments/assets/ea4c3f6d-4215-4e17-805e-4444b57d0f0d" />

Enforcing Isolation: Security Groups
Security Groups (SGs) act as virtual firewalls to control inbound and outbound traffic for your instances. We create four highly restrictive SGs, implementing the principle of least privilege.

Security Group,Inbound Rules,Source
Bastion Host SG,SSH (Port 22),Your Public IP
Web Server SG,"HTTP (Port 80), HTTPS (Port 443)",0.0.0.0/0 (Internet)
App Server SG,HTTP/S (Port 80/443),"0.0.0.0/0, SSH (Port 22)"
Database SG,MySQL/Aurora (Port 3306),App Server SG and Bastion Host SG

<img width="1200" height="543" alt="image" src="https://github.com/user-attachments/assets/26e86bd0-e065-40fa-8a33-83bd455766b8" />
<img width="1113" height="576" alt="image" src="https://github.com/user-attachments/assets/c5048c64-fec5-4ee3-97be-c61a720fb5f6" />
<img width="1200" height="564" alt="image" src="https://github.com/user-attachments/assets/89be075e-4368-4656-864a-fffc72df3981" />
<img width="849" height="590" alt="image" src="https://github.com/user-attachments/assets/e82b4185-3017-463a-82bc-fbf1d27147c7" />
<img width="938" height="602" alt="image" src="https://github.com/user-attachments/assets/821ec1f1-aa41-446f-a427-9072556648bc" />

Deploying the Tiers: EC2 Instances and RDS
With the network secured, we deploy the compute resources into their respective subnets.
Application and Presentation Tiers (EC2)
We launch three EC2 instances using the same base AMI (e.g., Amazon Linux 2):
Bastion Host: Deployed in the Public Subnet. It is a hardened access point, allowing us to securely SSH into the private instances.

<img width="800" height="363" alt="image" src="https://github.com/user-attachments/assets/09ade067-692c-4300-9d8f-e547259ca1a7" />

2. Web Server (Presentation Tier): Deployed in the Public Subnet. It receives public traffic, often acting as a proxy or hosting static content. We use User Data to install and start the web server stack (e.g., Apache/PHP) on launch.

   <img width="800" height="363" alt="image" src="https://github.com/user-attachments/assets/5f3eaa9f-d445-4814-86b0-c9b66978fd90" />

   App Server (Application Tier): Deployed in a Private Subnet. It contains the business logic. It's configured to run a database client (e.g., MariaDB client) to communicate with the RDS instance.

<img width="1200" height="550" alt="image" src="https://github.com/user-attachments/assets/4e05bd74-73b6-40dc-8efa-239066826cb0" />

<img width="1200" height="546" alt="image" src="https://github.com/user-attachments/assets/a4e0b891-1265-4ca7-b417-528707615660" />

Data Tier (RDS)
The most sensitive layer is hosted by Amazon RDS for a managed database experience.
DB Subnet Group: First, a DB Subnet Group is created, explicitly specifying the dedicated Private Database Subnets.
RDS Instance: We launch a managed database (e.g., MariaDB on the Free Tier).
Configuration: The RDS instance is placed in the created DB Subnet Group and must have Public access set to No for maximum security. It is associated only with the highly restrictive Database SG.


<img width="800" height="342" alt="image" src="https://github.com/user-attachments/assets/889010c9-31e5-4639-8e3b-af6640822734" />

<img width="1200" height="561" alt="image" src="https://github.com/user-attachments/assets/27954789-415e-4c60-8cc2-0c24b0d21009" />

<img width="1086" height="617" alt="image" src="https://github.com/user-attachments/assets/043865c6-9923-403d-9d9e-0b66da72e71d" />

<img width="1113" height="600" alt="image" src="https://github.com/user-attachments/assets/40119006-3f5c-4b7c-ab9d-36eb20cd1e2f" />

<img width="1093" height="584" alt="image" src="https://github.com/user-attachments/assets/e80061b5-368d-498a-98de-11b8a5df235e" />

Final Validation: Testing Connectivity
The final step is to validate that the cross-tier communication works as intended, proving the architecture is functional and secure.
Access the Bastion Host: SSH into the Bastion Host using its Public IP.
Jump to the App Server: From the Bastion Host, use its key to SSH into the App Server using the App Server's Private IP address.
Connect to the Database: From the App Server, use the MySQL client to connect to the RDS database endpoint.


<img width="744" height="484" alt="image" src="https://github.com/user-attachments/assets/788247f6-1587-47d7-b8e6-7e9399b09460" />

<img width="642" height="279" alt="image" src="https://github.com/user-attachments/assets/76721959-1769-4cb9-b1aa-6417aeea539e" />

<img width="766" height="768" alt="image" src="https://github.com/user-attachments/assets/4d678aad-01f8-41a2-8a48-247c50cea5eb" />

Successfully connecting to the database from the App Server confirms that the networking (VPC, Subnets, Route Tables, and NAT Gateway) and security (Security Groups) are configured correctly for a secure, multi-tier deployment!


<img width="698" height="650" alt="image" src="https://github.com/user-attachments/assets/76e8fcdb-01df-474e-bb8c-ee1aa5e56b0f" />

Note to Reader: This architecture provides a robust, highly available, and secure foundation. For production use, consider adding an Application Load Balancer (ALB) and Auto Scaling Groups (ASGs) to the Web and App tiers for horizontal scaling and self-healing capabilities.
Happy Learning!
