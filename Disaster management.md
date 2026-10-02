

# Disaster Recovery Setup with AWS Backup & Route 53

## 📌 Project Overview

This project implements a **Disaster Recovery (DR) solution on Amazon Web Services (AWS)** using **AWS Backup** and **Amazon Route 53**.

The main purpose of the project is to protect critical application data, provide reliable backup and recovery, monitor application availability, and redirect users to a secondary environment when the primary environment becomes unavailable.

The solution demonstrates how cloud-based disaster recovery can help reduce **downtime** and **data loss** while maintaining application availability.

---

## 🎯 Project Objectives

The main objectives of this project are:

- To create a reliable disaster recovery architecture on AWS.
- To automatically back up critical AWS resources using AWS Backup.
- To securely store recovery points in a Backup Vault.
- To configure Amazon Route 53 for DNS-based failover.
- To monitor the health of the primary application.
- To redirect users to a secondary environment during a failure.
- To test backup restoration and application recovery.
- To understand RTO and RPO in a practical cloud environment.
- To improve application availability and business continuity.

---

## 🏗️ Architecture

The project consists of a **Primary Environment** and a **Disaster Recovery Environment**.

```text
                         USERS
                           |
                           v
                    +--------------+
                    |  Route 53    |
                    | DNS &        |
                    | Failover     |
                    +------+-------+
                           |
              +------------+------------+
              |                         |
              v                         v
      PRIMARY ENVIRONMENT          DR ENVIRONMENT
              |                         |
              v                         v
       +-------------+           +-------------+
       |     ALB     |           |     ALB     |
       +------+------+\          +------+------+
              |                         |
              v                         v
       +-------------+           +-------------+
       |    EC2      |           |    EC2      |
       | Application |           | Application |
       +------+------+\          +------+------+
              |                         |
              v                         v
       +-------------+           +-------------+
       |     RDS     |           | DR Database |
       |  Database   |           | / Recovery  |
       +-------------+           +-------------+

              |
              v
       +----------------+
       |  AWS Backup    |
       +-------+--------+
               |
               v
       +----------------+
       |  Backup Vault  |
       +----------------+
```

---

## ☁️ AWS Services Used

| AWS Service | Purpose |
|---|---|
| **Amazon VPC** | Creates the isolated network environment |
| **Amazon EC2** | Hosts the application/web server |
| **Amazon RDS** | Provides the application database |
| **Application Load Balancer** | Distributes application traffic |
| **AWS Backup** | Creates and manages backups |
| **Backup Vault** | Stores recovery points |
| **Amazon Route 53** | Provides DNS and failover routing |
| **Route 53 Health Checks** | Monitors application availability |
| **Amazon CloudWatch** | Provides monitoring and alarms |
| **AWS IAM** | Manages permissions and access |
| **Amazon S3** | Optional storage for application/backup-related objects |

---

# 🔄 Disaster Recovery Workflow

## 1. Create the Primary Environment

The first step is to create the production environment.

The environment includes:

- VPC
- Public and private subnets
- Security groups
- EC2 instance
- Application Load Balancer
- RDS database

Basic flow:

```text
VPC
 |
 +-- ALB
      |
      +-- EC2
           |
           +-- RDS
```

---

## 2. Deploy the Application

Deploy a simple web application on the EC2 instance.

For example:

```text
EC2
 |
 +-- Web Server
 |
 +-- Application
```

The application should provide a health endpoint such as:

```text
/health
```

This endpoint can return a successful response when the application is running correctly.

---

## 3. Configure the Database

Create an Amazon RDS database and connect it to the application.

```text
Application
     |
     v
   RDS
     |
     v
Application Data
```

The database contains important application data that needs to be protected.

---

# 💾 4. Configure AWS Backup

Create an AWS Backup Vault.

Example:

```text
Backup Vault Name:
DR-Backup-Vault
```

Create a Backup Plan.

Example configuration:

```text
Backup Frequency : Daily
Retention        : 7 Days
Backup Vault     : DR-Backup-Vault
```

The backup plan can protect resources such as:

- EC2/EBS
- RDS
- Other supported AWS resources

---

# 🔐 5. Backup Protection

AWS Backup creates recovery points according to the configured backup schedule.

Example:

```text
Production Resources
        |
        v
    AWS Backup
        |
        v
  Backup Vault
        |
        +---- Recovery Point 1
        |
        +---- Recovery Point 2
        |
        +---- Recovery Point 3
```

These recovery points can be used to restore resources after a failure.

---

# 🌐 6. Configure Amazon Route 53

Create a Route 53 hosted zone for the application domain.

Example:

```text
example.com
```

Create a DNS record:

```text
www.example.com
```

Configure the record using:

```text
Routing Policy: Failover
```

Create:

- Primary record
- Secondary record

---

# ❤️ 7. Configure Health Checks

Configure a Route 53 health check for the primary application.

Example:

```text
https://www.example.com/health
```

The health check determines whether the primary application is available.

```text
Primary Application
        |
        v
Route 53 Health Check
        |
   +----+----+
   |         |
Healthy   Unhealthy
   |         |
   v         v
Primary    Failover
```

---

# 🔁 8. Configure Failover

During normal operation:

```text
User
 |
 v
Route 53
 |
 v
Primary ALB
 |
 v
Primary EC2
```

If the primary environment becomes unavailable:

```text
User
 |
 v
Route 53
 |
 X Primary Unhealthy
 |
 v
Secondary ALB
 |
 v
DR EC2
```

The user continues using the same domain name while traffic is directed toward the recovery environment.

---

# 🚨 9. Simulate a Disaster

To test the disaster recovery setup, simulate a failure in the primary environment.

For example:

- Stop the primary EC2 instance.
- Stop the application.
- Make the health endpoint unavailable.
- Simulate an application failure.

Example:

```text
Primary Application
        |
        X
     FAILURE
        |
        v
Route 53 Health Check
        |
        v
Primary = Unhealthy
```

---

# 🔄 10. Failover to DR Environment

Once the primary environment is considered unhealthy, the DNS failover configuration directs traffic toward the secondary environment.

```text
             Route 53
                 |
        +--------+--------+
        |                 |
    Primary             Secondary
      X                    |
   Failed                 v
                       DR ALB
                         |
                         v
                       DR EC2
```

---

# ♻️ 11. Restore Data Using AWS Backup

If resources or data need to be recovered, use AWS Backup recovery points.

Basic recovery process:

```text
Backup Vault
     |
     v
Recovery Point
     |
     v
Restore
     |
     v
Recovered Resource
     |
     v
Application Testing
```

The restored resources should be tested before being used for production recovery.

---

# 🧪 12. Disaster Recovery Testing

The project should include practical DR testing.

### Test 1: Backup Test

Verify that scheduled backups are created successfully.

### Test 2: Restore Test

Restore a resource from a recovery point and verify its functionality.

### Test 3: Health Check Test

Make the primary application unhealthy and verify that the health check detects the failure.

### Test 4: Failover Test

Verify that traffic is redirected to the secondary environment.

### Test 5: Recovery Test

Restore the primary environment and verify that it is functioning correctly.

### Test 6: Failback Test

Return application traffic to the primary environment after recovery.

---

# ⏱️ RTO and RPO

## Recovery Time Objective (RTO)

RTO defines the maximum acceptable time required to restore the application after a disaster.

Example:

```text
RTO = 30 minutes
```

The recovery process should aim to restore service within the defined target.

## Recovery Point Objective (RPO)

RPO defines the maximum acceptable amount of data loss measured in time.

Example:

```text
RPO = 1 hour
```

This means the recovery strategy aims to limit potential data loss to approximately one hour for the defined scenario.

---

# 📊 Monitoring

Amazon CloudWatch can be used to monitor the infrastructure.

Important metrics include:

- EC2 health
- CPU utilization
- RDS metrics
- ALB health
- Application availability
- Application errors
- Backup events

Example:

```text
CloudWatch
    |
    +-- EC2 Monitoring
    |
    +-- RDS Monitoring
    |
    +-- ALB Monitoring
    |
    +-- Alarms
```

---

# 🔐 Security

Security is an important part of the disaster recovery architecture.

The project uses:

- IAM roles and policies
- Security groups
- Private subnets for database resources
- Encryption for protected data where required
- Least-privilege access
- Secure database credentials
- Protected backup resources

Sensitive credentials should **never be hardcoded in application code or uploaded to GitHub**.

---

# 🧰 Prerequisites

Before starting the project, you should have:

- AWS account
- Basic AWS knowledge
- Basic networking knowledge
- Understanding of EC2
- Understanding of RDS
- Understanding of Route 53
- Understanding of AWS Backup
- Basic Linux commands
- A web browser

### Optional

VS Code is **not required** for the AWS Console-based implementation.

VS Code becomes useful if you decide to add:

- Terraform
- CloudFormation
- Application source code
- Shell scripts
- Configuration files

---

# 📁 Suggested Project Structure

If you are maintaining documentation along with the AWS project:

```text
disaster-recovery-aws/
│
├── README.md
│
├── architecture/
│   └── architecture-diagram.png
│
├── screenshots/
│   ├── vpc.png
│   ├── ec2.png
│   ├── rds.png
│   ├── aws-backup.png
│   ├── backup-vault.png
│   ├── route53.png
│   ├── health-check.png
│   └── failover.png
│
└── documentation/
    └── project-report.pdf
```

---

# 🚀 Implementation Steps

The complete implementation can be performed in the following order:

```text
1. Create AWS VPC
       ↓
2. Create Subnets
       ↓
3. Configure Security Groups
       ↓
4. Launch EC2
       ↓
5. Deploy Application
       ↓
6. Create RDS
       ↓
7. Configure ALB
       ↓
8. Test Application
       ↓
9. Create AWS Backup Vault
       ↓
10. Create Backup Plan
       ↓
11. Configure Backup Schedule
       ↓
12. Verify Recovery Points
       ↓
13. Create DR Environment
       ↓
14. Configure Route 53
       ↓
15. Configure Health Check
       ↓
16. Configure Failover Records
       ↓
17. Simulate Primary Failure
       ↓
18. Verify DR Failover
       ↓
19. Test Backup Restoration
       ↓
20. Restore Primary Environment
       ↓
21. Perform Failback
       ↓
22. Monitor and Document Results
```

---

# 📈 Expected Results

After completing the project:

- Critical resources are backed up automatically.
- Recovery points are available for restoration.
- Application health can be monitored.
- Route 53 can detect an unhealthy primary endpoint through the configured health-check/failover setup.
- Traffic can be directed toward the DR environment.
- Resources can be restored from backups.
- Disaster recovery procedures can be tested.
- Application downtime and data-loss exposure can be managed according to the project's RTO/RPO targets.

---

# 💡 Key Learning Outcomes

Through this project, you will learn:

- AWS disaster recovery concepts
- Backup and restore strategies
- AWS Backup
- Backup Vaults
- EC2 recovery
- RDS recovery
- Route 53 DNS
- Route 53 failover routing
- Health checks
- RTO and RPO
- CloudWatch monitoring
- IAM security
- AWS networking
- Disaster recovery testing
- Business continuity concepts

---

# 📝 Project Summary

**Disaster Recovery Setup with AWS Backup & Route 53** is a cloud-based disaster recovery project designed to protect application resources and maintain service availability during failures.

AWS Backup provides centralized backup and recovery capabilities, while Amazon Route 53 provides DNS-based failover and health-check functionality. Together with EC2, RDS, VPC, ALB, IAM, and CloudWatch, the project demonstrates a practical approach to designing and testing a disaster recovery solution on AWS.

---

# 👩‍💻 Author

**Bhagyashree Manglekar**

**Project:** Disaster Recovery Setup with AWS Backup & Route 53

**Platform:** Amazon Web Services (AWS)

---

