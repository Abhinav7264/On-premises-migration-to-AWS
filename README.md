![1-PORTFOLIO PROJECTS_AWS - MODULE 3_ARCHITECTURE](https://github.com/user-attachments/assets/2baccfc4-6683-49f6-88b9-51498c7d2c6b)
# Migration of a Workload running in a Corporate Data Center to AWS using the Amazon EC2 and RDS service
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

### 🌍 Set Up Internet Gateway & Routing
- Create **Internet Gateway**: `igw-mod3` → Attach to `vpc-bootcamp`
- Route Table → Add Route `0.0.0.0/0` → Target: `igw-mod3`

### 🔗 Connect to EC2
```bash
ssh -i ec2-ssh ubuntu@<EC2_PUBLIC_IP>
```

### 👁️ **Installing the application's dependencies**  

**Ubuntu 22.04** may prompt ‘pop-ups’ after installing or updating packages, asking:  

🔹 *“What service should be restarted?”*  

By default, this is set to **"interactive"** mode, causing interruptions in scripts.  

To change this behavior, update the `/etc/needrestart/needrestart.conf` file:  

📝 **Modify the following lines:**  

```bash
#$nrconf{restart} = 'i';  # Default (interactive)
$nrconf{restart} = 'a';  # Changed to automatic
```

➡️ *‘i’ = interactive* | *‘a’ = automatic*  

---

### ⚙️ **Apply Changes Using `sed` Command**  

```bash
sudo sed -i "/#\$nrconf{restart} = 'i';/s/.*/\$nrconf{restart} = 'a';/" /etc/needrestart/needrestart.conf
```

### ✅ **Verify Changes Made**  

```bash
cat /etc/needrestart/needrestart.conf | grep -i nrconf{restart}
sudo apt update
sudo apt install python3-dev -y
sudo apt install python3-pip -y
```

---

### 🔧 **Install Required Packages**  

```bash
sudo apt install build-essential libssl-dev libffi-dev -y
sudo apt install libmysqlclient-dev -y
sudo apt install unzip -y
sudo apt install libpq-dev libxml2-dev libxslt1-dev libldap2-dev -y
sudo apt install libsasl2-dev libffi-dev -y
```

---

### 🛠️ **Install Python Packages**  

```bash
pip install Flask==2.3.3
```

⚠ **Potential Warning & Fix:**  

```bash
export PATH=$PATH:/home/ubuntu/.local/bin/
```

```bash
pip3 install wtforms
sudo apt install pkg-config
pip3 install flask_mysqldb
pip3 install passlib
```

---

### 🗃️ **Install MySQL Client**  

```bash
sudo apt-get install mysql-client -y
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

The Final product looks like this
<img width="1225" alt="Screenshot 2025-03-15 at 1 15 08 AM" src="https://github.com/user-attachments/assets/983b31bb-b8f4-4033-82a2-d9386bca1ae1" />

---

## 🛑 Post Go Live: Cleanup Resources
- **Delete RDS** (Skip snapshots & backups)
- **Terminate EC2** (Actions → Terminate)

✨ **Congratulations! Your deployment is complete!** 🚀


