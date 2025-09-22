# Three-Tier AWS Architecture with Auto Scaling & Load Balancing

## 📌 Overview
This project demonstrates how to design and deploy a **three-tier architecture on AWS**.  
It consists of:
- **Web Tier**: Serves frontend HTML/CSS via Nginx
- **App Tier**: Runs PHP with business logic
- **Database Tier**: Amazon RDS (MariaDB)

The architecture ensures **high availability, scalability, and secure access** using Auto Scaling Groups, Load Balancers, and a Jump Server.

---

## 🏗️ Architecture
1. **VPC Setup**
   - Create a VPC with 8 subnets (public + private across AZs).
   - Attach an Internet Gateway and configure Route Tables.

2. **Database Tier**
   - Create an RDS Subnet Group.
   - Launch RDS (MariaDB) inside private subnets.
   - Secure RDS with proper security group rules.

3. **Jump Server**
   - Launch a Bastion Host (EC2) in a public subnet.
   - Use it to securely connect to private resources.

4. **App Tier**
   - Launch an EC2 instance in private subnet.
   - Install PHP & Nginx.
   - Deploy `submit.php`.
   - Create an AMI of this instance.
   - Setup an **Auto Scaling Group** for the App Tier.

5. **Web Tier**
   - Launch EC2 instance for frontend (HTML + Nginx).
   - Add sample web page.
   - Create an AMI of this instance.
   - Setup **Auto Scaling Group** for Web Tier.
   - Attach an **Application Load Balancer**.

---

## 🚀 Features
- VPC with public & private subnets across AZs  
- Internet Gateway & Route Tables for routing  
- Jump Server for secure private access  
- Amazon RDS (MariaDB) in Multi-AZ setup  
- Auto Scaling Groups for Web & App tiers  
- Application Load Balancer for traffic distribution  
- Custom AMIs for fast scaling  

---

## 🛠️ Tech Stack
- **AWS Services**: VPC, EC2, RDS (MariaDB), Auto Scaling, Load Balancer, AMI  
- **Languages/Tools**: PHP, Nginx, HTML, CSS, MariaDB  

---

## 📂 Project Use Cases
- Learning **AWS Infrastructure & Cloud Architecture**  
- Demonstration of **Auto Scaling & Load Balancing**  
- Portfolio project for **Cloud/DevOps roles**  

---

## 📸 Architecture Diagram
*(Add your architecture diagram here)*

---

## ✅ Deployment Steps (Summary)
1. Create VPC with 8 subnets (public/private).  
2. Configure Internet Gateway + Route Tables.  
3. Launch RDS with a Subnet Group.  
4. Setup Jump Server in public subnet.  
5. Deploy App Tier (PHP + Nginx) → Create AMI → Auto Scaling.  
6. Deploy Web Tier (HTML + Nginx) → Create AMI → Auto Scaling.  
7. Configure Application Load Balancer for Web Tier.  
8. Test end-to-end application.  

---


