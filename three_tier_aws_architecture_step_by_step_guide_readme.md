# Three‑Tier AWS Architecture with Auto Scaling & Load Balancing — Step‑by‑Step Guide

## Overview
This document explains how to set up a **three‑tier architecture** (Web → App → Database) in AWS using only the **AWS Management Console** (no CLI). It includes all the steps you shared, plus details for configuring networking, security groups, RDS, Auto Scaling Groups, and testing. At the end, there’s also a `README.md` template you can put in your GitHub repository.

---

# Step‑by‑Step (AWS Console)

### 1) Create a VPC
1. Open the **VPC console**.
2. Click **Create VPC** → choose **VPC only** option.
3. Enter:
   - Name: `three-tier-vpc`
   - IPv4 CIDR block: `10.0.0.0/16`
   - Tenancy: Default
4. Click **Create VPC**.

### 2) Create 8 subnets
1. Go to **Subnets** → **Create subnet**.
2. Select your `three-tier-vpc`.
3. Add 8 subnets with the following details (spread across different AZs):
   - `public-a` (10.0.0.0/24)
   - `public-b` (10.0.1.0/24)
   - `public-c` (10.0.2.0/24)
   - `public-d` (10.0.3.0/24)
   - `app-a` (10.0.10.0/24)
   - `app-b` (10.0.11.0/24)
   - `db-a` (10.0.20.0/24)
   - `db-b` (10.0.21.0/24)
4. For the **public subnets**, enable **Auto-assign public IPv4 address**.

### 3) Create an RDS Subnet Group
1. Go to **RDS console** → **Subnet groups** → **Create DB Subnet group**.
2. Add name: `myapp-db-subnet-group`.
3. Choose the VPC `three-tier-vpc`.
4. Add the two subnets `db-a` and `db-b`.
5. Save.

### 4) Launch RDS (MariaDB)
1. Go to **Databases** → **Create database**.
2. Choose **Standard create**.
3. Engine: **MariaDB**.
4. Templates: Free tier or production depending on need.
5. DB instance identifier: `myapp-db`.
6. Master username: `admin`. Set a strong password.
7. DB instance size: e.g., `db.t3.micro`.
8. Storage: 20 GB.
9. Connectivity:
   - VPC: `three-tier-vpc`.
   - Subnet group: `myapp-db-subnet-group`.
   - Public access: **No**.
   - Create a new security group for DB (`sg-db`).
10. Create database and wait until status is **Available**.

### 5 & 6) Create Internet Gateway and attach to VPC
1. In **VPC console**, open **Internet Gateways** → **Create IGW**.
2. Name: `three-tier-igw`.
3. Select IGW → **Actions → Attach to VPC** → choose `three-tier-vpc`.

### 7 & 8) Create a Route Table and add route to IGW
1. In **Route tables** → **Create route table**.
2. Name: `public-rt`, select `three-tier-vpc`.
3. Edit routes → Add route:
   - Destination: `0.0.0.0/0`
   - Target: `three-tier-igw`

### 9) Associate Route Table to Public Subnets
1. In the route table, go to **Subnet associations**.
2. Select all public subnets (`public-a` to `public-d`).

### 10) Launch Jump Server (Bastion) in Public Subnet
1. Go to **EC2** → **Launch instance**.
2. AMI: Amazon Linux 2023.
3. Instance type: `t3.micro`.
4. Network: `three-tier-vpc`.
5. Subnet: `public-a`.
6. Auto-assign public IP: **Enabled**.
7. Security group: Allow **SSH (22)** only from your own IP.
8. Launch.

### 11) Login to Jump Server
Use your `.pem` key and connect via SSH.
```bash
ssh -i key.pem ec2-user@<jump-server-public-ip>
```

### 12) Install MariaDB Client on Jump Server
```bash
sudo dnf update -y
sudo dnf install -y mariadb
```

### 13) Connect to RDS from Jump Server
```bash
mysql -h <RDS-ENDPOINT> -P 3306 -u admin -p
```

### 14) Create Database
```sql
CREATE DATABASE myappdb;
CREATE USER 'appuser'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON myappdb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
```

### 15–17) Launch App Tier Instance and Install PHP & Nginx
1. Launch a private EC2 instance in `app-a` subnet.
2. Security group: allow **HTTP (80)** from web tier SG, and **MySQL (3306)** outbound to DB SG.
3. Use bastion to SSH into this app instance.
4. Install packages:
```bash
sudo dnf update -y
sudo dnf install -y nginx php-fpm php-mysqlnd
sudo systemctl enable --now nginx php-fpm
```
5. Configure Nginx to use PHP.

### 18) Create submit.php
Place file in `/var/www/html/submit.php` with PHP code that connects to RDS and inserts records.

### 19) Test submit.php
Use curl or web browser (via internal load balancer later).

### 20) Create AMI of App Instance
Right-click the instance → **Image and templates → Create image**.

### 21) Create App Tier Auto Scaling Group
1. Create a Launch Template using the App AMI.
2. Create Auto Scaling Group in `app-a` and `app-b` subnets.
3. Min size: 2, Max: 6.

### 22–23) Launch Web Tier Instance
1. Launch instance in `public-a` subnet.
2. Install Nginx and host HTML pages.
3. Create AMI.

### 24) Create Auto Scaling Group for Web Tier
1. Create Launch Template using Web AMI.
2. Create Auto Scaling Group in `public-a` and `public-b` subnets.
3. Attach an Application Load Balancer (ALB).

### 25) Verify Setup
- Access ALB DNS → Web tier page should load.
- Submit form → App tier writes to RDS.
- Verify data inside RDS.

---
