![1-PORTFOLIO PROJECTS_AWS - MODULE 3_ARCHITECTURE](https://github.com/user-attachments/assets/2baccfc4-6683-49f6-88b9-51498c7d2c6b)

# 🚀 AWS Deployment Guide

## 📌 Part 1: Deploying EC2 & RDS

### 🌐 Creating VPC & Subnets
- **VPC**: `vpc-bootcamp` | CIDR: `10.0.0.0/16`
- **Subnets:**
  - 🌍 `Public Subnet` | AZ: `us-east-1a` | CIDR: `10.0.0.0/24`
  - 🔒 `Private Subnet` | AZ: `us-east-1a` | CIDR: `10.0.1.0/24`
  - 🔒 `Private Subnet` | AZ: `us-east-1b` | CIDR: `10.0.2.0/24`

### 🖥️ Launching EC2 Instance
- **Name**: `awsuse1app01` | **OS**: Ubuntu 22.04 LTS
- **Instance Type**: `t2.micro`
- **Key Pair**: `ec2-ssh` (Store in `C:\aws-mod3`)
- **Network**: `vpc-bootcamp`, `Public Subnet`, Auto-assign Public IP ✅
- **Security Group**: `app01-sg` (Ports `22, 8080`)

### 🗄️ Setting Up RDS (MySQL 5.7)
- **DB Name**: `awsuse1db01` | Free Tier ✅
- **Instance**: `db.t2.micro` | Credentials: `admin/admin123456`
- **VPC**: `vpc-bootcamp` | AZ: `us-east-1a`
- **Port**: `3306` | Security Group: `default`

---

## 📌 Part 2: Configuring EC2 & Installing Dependencies

### 🔗 Connect to EC2
```bash
ssh -i ec2-ssh ubuntu@<EC2_PUBLIC_IP>
```

### 🌍 Set Up Internet Gateway & Routing
- Create **Internet Gateway**: `igw-mod3` → Attach to `vpc-bootcamp`
- Route Table → Add Route `0.0.0.0/0` → Target: `igw-mod3`

### 📦 Installing Packages & Dependencies
```bash
sudo sed -i "/#\$nrconf{restart} = 'i';/s/.*/\$nrconf{restart} = 'a';/" /etc/needrestart/needrestart.conf
sudo apt update && sudo apt install -y python3-dev python3-pip build-essential \
  libssl-dev libffi-dev libmysqlclient-dev unzip libpq-dev libxml2-dev \
  libxslt1-dev libldap2-dev libsasl2-dev pkg-config mysql-client
pip3 install Flask==2.3.3 wtforms flask_mysqldb passlib
export PATH=$PATH:/home/ubuntu/.local/bin/
```

---

## 📌 Part 3: Go Live! 🚀

### 🔐 Configure RDS Security
- **Security Group**: `EC2toRDS-sg`
- **Inbound Rule**: `MySQL (3306) → 0.0.0.0/0`
- **Attach SG to RDS**: Modify `awsuse1db01` → Apply changes ✅

### 🔗 Connect to RDS & Setup Database
```bash
mysql -h <RDS_ENDPOINT> -P 3306 -u admin -p
```
```sql
CREATE DATABASE wikidb;
CREATE USER 'wiki'@'%' IDENTIFIED BY 'admin123456';
GRANT ALL PRIVILEGES ON wikidb.* TO 'wiki'@'%';
FLUSH PRIVILEGES;
```

### 📥 Download & Configure Application
Save the wikiapp-en.zip and dump-en.sql in a s3 bucket and make it public
Download these files in ubuntu EC2 instance

### ✍ Edit `wiki.py`
```bash
cd wikiapp/
vi wiki.py
# Update MySQL credentials
MYSQL_HOST = '<RDS_ENDPOINT>'
MYSQL_USER = 'wiki'
```

### 🚀 Start the Application
```bash
python3 wiki.py
```

### ✅ Validate Deployment
- Open `http://<EC2_PUBLIC_IP>:8080`
- Login: `admin / admin`

🎉 **Success!** Add an article:
_“I'm conquering the MultiCloud Universe! 🌎🔥”_

---

## 🛑 Post Go Live: Cleanup Resources
- **Delete RDS** (Skip snapshots & backups)
- **Terminate EC2** (Actions → Terminate)

✨ **Congratulations! Your deployment is complete!** 🚀


