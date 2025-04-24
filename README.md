# RDS Monitoring and Alerting System

## 📌 Objective

This project demonstrates how to monitor an Amazon RDS instance's performance—specifically **CPU utilization**, **database connections**, and **storage metrics**—using **Amazon CloudWatch**, and how to configure **Amazon SNS** to send real-time alert notifications when thresholds are breached.

# Architecture
<img width="1019" alt="Screenshot 2025-04-23 at 9 13 23 PM" src="https://github.com/user-attachments/assets/dc959f9e-29d7-4c55-ad18-11870d3b4452" />

# Load test 
<img width="1440" alt="Screenshot 2025-04-23 at 7 22 20 PM" src="https://github.com/user-attachments/assets/9fbda2fd-a80d-4ac9-b44b-2a68abc22b55" />

# CloudWatch Alarm Trigger
<img width="1440" alt="Screenshot 2025-04-23 at 7 25 17 PM" src="https://github.com/user-attachments/assets/7430df47-b1f1-4901-89ca-398a19ba7f8c" />

# SNS Notification

<img width="1440" alt="Screenshot 2025-04-23 at 7 28 15 PM" src="https://github.com/user-attachments/assets/7f1ef515-764a-46b7-8846-68a617328473" />





## 🧰 AWS Services Used

- **Amazon RDS (MySQL 8.x)**
- **Amazon CloudWatch**
- **Amazon SNS**
- **Amazon EC2**

---

## 🛠️ Project Setup and Steps

### 🔹 Step 1: Create an RDS MySQL Instance
1. Navigate to **RDS > Create Database**
2. Select:
   - **Standard Create**
   - **Engine**: MySQL
   - **Version**: 8.x (Free Tier eligible)
   - **Template**: Free Tier
   - **Instance Class**: `db.t3.micro`
   - **Storage**: 20 GiB (General Purpose SSD)
3. Enable **Public Access** (if external access is needed)
4. Use a VPC security group that allows inbound traffic on **port 3306**
5. Set the **master username and password**
6. Launch the instance and wait for it to become **available**

---

### 🔹 Step 2: Enable Enhanced Monitoring
1. Go to **RDS > Databases > [Your DB] > Modify**
2. Scroll to **Monitoring**
3. Enable **Enhanced Monitoring**
4. Set monitoring granularity (1 or 5 seconds)
5. Attach monitoring role `rds-monitoring-role`  
   - If not available:
     - Create IAM role with:
       - **Service**: RDS
       - **Use case**: RDS - Enhanced Monitoring
       - **Policy**: `AmazonRDSEnhancedMonitoringRole`
       - **Name**: `rds-monitoring-role`
6. Apply changes **immediately**

---

### 🔹 Step 3: Create an SNS Topic
1. Go to **SNS > Topics > Create topic**
2. Select **Standard** type
3. Name the topic: `rds-alert-topic`
4. Create a **subscription**:
   - **Protocol**: Email
   - **Endpoint**: your-email@example.com
5. Confirm the subscription via the confirmation email

---

### 🔹 Step 4: Create CloudWatch Alarms
1. Go to **CloudWatch > Alarms > Create Alarm**
2. Select metric:
   - **RDS > Per-Database Metrics > CPUUtilization**
3. Define conditions:
   - Static threshold: **CPUUtilization > 80%**
   - Alarm after **2 out of 3 datapoints**
   - Treat missing data: **"missing"**
4. Notification action:
   - Send to SNS topic: `rds-alert-topic`
5. Alarm name: `High-CPU-Utilization`

---

## ✅ Step 5: Connect EC2 to RDS

1. SSH into your EC2:
```bash
ssh -i your-key.pem ec2-user@<EC2-Public-IP>
```

2. Install MySQL client if not already:
```bash
sudo dnf update -y
sudo dnf install mariadb105
sudo dnf install mariadb105-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo mysql -u root -p
```

3. Connect to RDS from EC2:

```bash
mysql -h <RDS-ENDPOINT> -u <master-username> -p
```


4. Run a benchmark query to spike CPU:
```bash
SELECT BENCHMARK(500000000, SHA2('test', 512));
```
