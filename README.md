# 🚀 vProfile Application - AWS Cloud Migration

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazon-aws)](https://aws.amazon.com/)
[![Java](https://img.shields.io/badge/Java-11-blue?style=for-the-badge&logo=java)](https://www.java.com/)
[![Spring](https://img.shields.io/badge/Spring-5-green?style=for-the-badge&logo=spring)](https://spring.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Service Comparison](#service-comparison)
- [Deployment Guide](#deployment-guide)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Testing](#testing)
- [Monitoring](#monitoring)
- [Cost Optimization](#cost-optimization)
- [Troubleshooting](#troubleshooting)
- [Technologies Used](#technologies-used)

---

## Project Overview

This project documents the **migration of the vProfile application** from on-premises infrastructure to AWS Cloud. The application is a multi-tier web application built with Java (Spring framework) that demonstrates a typical enterprise architecture.

### Original On-Premises Stack
| Component | Technology |
|-----------|------------|
| Web Server | Nginx (Reverse Proxy) |
| Application Server | Apache Tomcat |
| Database | MySQL |
| Caching | Memcached |
| Message Queue | RabbitMQ |

### AWS Cloud Architecture Benefits
- ✅ **High Availability** with Multi-AZ deployments
- ✅ **Auto Scaling** for variable workloads
- ✅ **Reduced Operational Overhead** with managed services
- ✅ **Pay-as-you-go** cost model
- ✅ **Infrastructure as Code** approach

---

## Architecture

### Request Flow Diagram

```mermaid
graph TD
    A[🌐 User] --> B[GoDaddy/Route53 DNS]
    B --> C[☁️ CloudFront CDN]
    C --> D[⚖️ Application Load Balancer]
    D --> E[🚀 Elastic Beanstalk<br/>Tomcat Auto Scaling]
    E --> F1[(🗄️ RDS MySQL)]
    E --> F2[(⚡ ElastiCache Memcached)]
    E --> F3[(📨 Amazon MQ RabbitMQ)]
    F1 --> E
    F2 --> E
    F3 --> E
    E --> C
    C --> A

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#bbf,stroke:#333,stroke-width:2px
    style D fill:#bbf,stroke:#333,stroke-width:2px
    style E fill:#bfb,stroke:#333,stroke-width:2px
    style F1 fill:#fbf,stroke:#333,stroke-width:2px
    style F2 fill:#fbf,stroke:#333,stroke-width:2px
    style F3 fill:#fbf,stroke:#333,stroke-width:2px



## Service Comparison

| Traditional Component | AWS Managed Service | Key Benefit | Technical Advantage |
|----------------------|---------------------|-------------|---------------------|
| Tomcat on EC2 | Elastic Beanstalk | Automated patching, deployment, and capacity management | Built-in auto-scaling, health monitoring, and rolling updates |
| Manual Load Balancer | Application Load Balancer | Integrated health checks and SSL termination | Layer 7 routing, WebSocket support, and request tracing |
| MySQL on EC2 | Amazon RDS | Automated backups, updates, and Multi-AZ failover | Point-in-time recovery, performance insights, and read replicas |
| Memcache on EC2 | Amazon ElastiCache | Managed in-memory caching with built-in replication | Automatic failure detection, cluster mode, and Redis/Memcached support |
| RabbitMQ on EC2 | Amazon MQ | Managed message broker with easy migration | Industry-standard protocols (AMQP, MQTT, STOMP), easy lift-and-shift |
| Custom DNS | Route 53 | Reliable and scalable DNS management | 100% SLA, DNS failover, latency-based routing |
| Global Delivery | CloudFront | Low latency content delivery for global audience | Edge locations (400+), DDoS protection, field-level encryption |
| Artifact Storage | Amazon S3 | Scalable, versioned artifact storage | 11 9's durability, lifecycle policies, cross-region replication |
## Deployment Guide

### 1. Prerequisites

<details>
<summary><b>📋 Click to expand prerequisites</b></summary>

#### **AWS Account Setup**
- ✅ Active AWS account with administrative access
- ✅ IAM user with appropriate permissions (AdministratorAccess or custom policy)
- ✅ AWS CLI configured locally (`aws configure`)
- ✅ Key pair for EC2 access (created in your target region)

#### **Local Tools Required**
```bash
# Required installations and their minimum versions
- AWS CLI v2          # aws --version
- Git                 # git --version  
- Maven 3.6+          # mvn --version
- Java 11             # java -version
- EB CLI              # eb --version
- Node.js (optional)  # For local testing

### 8. Domain and SSL Certificate Setup

<details>
<summary><b>🔒 Click to expand domain and certificate configuration</b></summary>

#### **8.1 Domain Registration**
- ✅ Domain name registered with GoDaddy (or any registrar)
- ✅ Domain: `yourdomain.com` (replace with your actual domain)
- ✅ Access to DNS management console

#### **8.2 Request SSL Certificate in ACM**

**Step 1: Request Public Certificate**
```bash
# Navigate to AWS Certificate Manager (ACM)
# Region must be us-east-1 for CloudFront distributions

aws acm request-certificate \
    --domain-name yourdomain.com \
    --validation-method DNS \
    --subject-alternative-names www.yourdomain.com \
    --region us-east-1

## 2. Initial AWS Setup

```bash
# Configure AWS CLI
aws configure

# Create key pair
aws ec2 create-key-pair \
    --key-name vprofile-key \
    --query 'KeyMaterial' \
    --output text > vprofile-key.pem

# Set permissions (Linux/Mac)
chmod 400 vprofile-key.pem

# For Windows PowerShell
# icacls vprofile-key.pem /inheritance:r /grant:r "%USERNAME%":R

# Create S3 bucket for artifacts
aws s3 mb s3://vprofile-artifacts --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket vprofile-artifacts \
    --versioning-configuration Status=Enabled

3. Backend Services Setup
🔹 RDS MySQL

# Create RDS instance
aws rds create-db-instance \
    --db-instance-identifier vprofile-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password YourPassword \
    --allocated-storage 20 \
    --storage-encrypted \
    --backup-retention-period 7 \
    --multi-az \
    --vpc-security-group-ids sg-backend-id

| Parameter | Value | Description |
|-----------|-------|-------------|
| Instance Class | db.t3.micro | Free tier eligible |
| Engine | MySQL 8.0 | Database engine |
| Storage | 20 GB | Minimum for production |
| Backup Retention | 7 days | Automated backups |
| Multi-AZ | Enabled | High availability |

🔹 ElastiCache Memcached

# Create ElastiCache cluster
aws elasticache create-cache-cluster \
    --cache-cluster-id vprofile-cache \
    --cache-node-type cache.t3.micro \
    --engine memcached \
    --num-cache-nodes 1 \
    --security-group-ids sg-backend-id

🔹 Amazon MQ (RabbitMQ)  

# Create Amazon MQ broker
aws mq create-broker \
    --broker-name vprofile-mq \
    --broker-instance-type t3.micro \
    --engine-type RABBITMQ \
    --engine-version 3.10.17 \
    --deployment-mode SINGLE_INSTANCE \
    --users Username=admin,Password=YourPassword123

4. Elastic Beanstalk Setup

# Install EB CLI
pip install awsebcli

# Initialize application
eb init \
    --platform "Tomcat 8.5 with Java 8" \
    --region us-east-1 \
    --keyname vprofile-key

# Create environment
eb create vprofile-env \
    --elb-type application \
    --instance-types t3.micro \
    --min-instances 2 \
    --max-instances 6 \
    --scale 20

# Set environment variables
eb setenv \
    RDS_HOST=vprofile-db.xxx.rds.amazonaws.com \
    RDS_DB=vprofile \
    RDS_USER=admin \
    RDS_PASSWORD=YourPassword123 \
    MEMCACHED_HOST=vprofile-cache.xxx.cache.amazonaws.com \
    RABBITMQ_HOST=b-xxx.mq.us-east-1.amazonaws.com

5. Security Group Configuration

| Security Group | Inbound Rules | Source |
|----------------|---------------|---------|
| ALB SG | HTTPS (443) | Internet (0.0.0.0/0) |
| Tomcat SG | HTTP (8080) | ALB Security Group |
| Backend SG | MySQL (3306), Memcache (11211), RabbitMQ (5672) | Tomcat Security Group |

# Get security group IDs after Beanstalk creation
aws ec2 describe-security-groups \
    --filters Name=group-name,Values=*beanstalk*

# Allow Beanstalk to access RDS
aws ec2 authorize-security-group-ingress \
    --group-id sg-backend-id \
    --protocol tcp \
    --port 3306 \
    --source-group sg-beanstalk-ec2-id

# Allow Beanstalk to access ElastiCache
aws ec2 authorize-security-group-ingress \
    --group-id sg-backend-id \
    --protocol tcp \
    --port 11211 \
    --source-group sg-beanstalk-ec2-id

# Allow Beanstalk to access Amazon MQ
aws ec2 authorize-security-group-ingress \
    --group-id sg-backend-id \
    --protocol tcp \
    --port 5672 \
    --source-group sg-beanstalk-ec2-id

6. Database Initialization

# Launch temporary EC2 instance
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t2.micro \
    --key-name vprofile-key \
    --security-group-ids sg-backend-id

# SSH into instance (Linux/Mac)
ssh -i vprofile-key.pem ec2-user@instance-public-ip

# Install MySQL client
sudo yum install mysql -y

# Connect to RDS and initialize database
mysql -h vprofile-db.xxx.rds.amazonaws.com -u admin -p

# Create database and import schema
CREATE DATABASE vprofile;
USE vprofile;
SOURCE src/main/resources/db_backup.sql;

# Verify
SHOW TABLES;
SELECT * FROM users LIMIT 5;

# Exit and terminate the temporary instance
exit

7. Build and Deploy

# Clean and build
mvn clean package

# Upload to S3
aws s3 cp target/vprofile-v2.war s3://vprofile-artifacts/

# Create application version
aws elasticbeanstalk create-application-version \
    --application-name vprofile \
    --version-label v1.0 \
    --source-bundle S3Bucket="vprofile-artifacts",S3Key="vprofile-v2.war"

# Deploy
aws elasticbeanstalk update-environment \
    --environment-name vprofile-env \
    --version-label v1.0

# Monitor deployment
eb events --follow

8. CloudFront and SSL Setup

# Request SSL certificate
aws acm request-certificate \
    --domain-name www.yourdomain.com \
    --validation-method DNS \
    --subject-alternative-names yourdomain.com

# Create CloudFront distribution
aws cloudfront create-distribution \
    --origin-domain-name vprofile-env.elasticbeanstalk.com \
    --default-root-object index.jsp \
    --viewer-protocol-policy redirect-to-https

9. DNS Configuration

<details> <summary><b>Option 1: Route 53</b></summary>

# Create A record
aws route53 change-resource-record-sets \
    --hosted-zone-id ZXXXXXXXXXXX \
    --change-batch '{
        "Changes": [{
            "Action": "CREATE",
            "ResourceRecordSet": {
                "Name": "www.yourdomain.com",
                "Type": "A",
                "AliasTarget": {
                    "HostedZoneId": "Z2FDTNDATAQYW2",
                    "DNSName": "vprofile-env.elasticbeanstalk.com",
                    "EvaluateTargetHealth": false
                }
            }
        }]
    }'
</details><details> <summary><b>Option 2: External Registrar (GoDaddy)</b></summary>

Create a CNAME record pointing to:

vprofile-env.elasticbeanstalk.com

</details>

Project Structure

vprofile-project/
├── 📄 README.md                       # Project documentation (this file)
├── 📁 userdata/                       # AWS EC2 user-data scripts
│   ├── backend.sh                     # Backend service setup
│   ├── memcache.sh                    # Memcached installation
│   ├── mysql.sh                       # MySQL initialization
│   ├── nginx.sh                       # Nginx configuration
│   ├── rabbitmq.sh                    # RabbitMQ setup
│   └── tomcat_ubuntu.sh               # Tomcat deployment
├── 📁 src/
│   ├── 📁 main/
│   │   ├── 📁 java/                   # Java source code
│   │   │   └── com/visualpathit/account/
│   │   │       ├── controller/        # REST controllers
│   │   │       ├── model/             # JPA entities
│   │   │       ├── repository/        # Data repositories
│   │   │       ├── service/           # Business logic
│   │   │       └── utils/             # Utility classes
│   │   ├── 📁 resources/
│   │   │   ├── application.properties # App configuration
│   │   │   ├── db_backup.sql          # Database dump
│   │   │   └── logback.xml            # Logging configuration
│   │   └── 📁 webapp/                 # JSP files and web resources
│   │       ├── WEB-INF/
│   │       │   ├── views/             # JSP templates
│   │       │   └── web.xml            # Web configuration
│   │       └── resources/             # CSS, JS, images
│   └── 📁 test/                       # Unit tests
├── 📄 al2023rmq.repo                  # Amazon Linux 2023 RabbitMQ repo
└── 📄 pom.xml                         # Maven configuration

Configuration
Application Properties

# Database configuration
db.host=${RDS_HOST}
db.port=3306
db.name=vprofile
db.username=${RDS_USER}
db.password=${RDS_PASSWORD}

# Cache configuration
memcache.host=${MEMCACHED_HOST}
memcache.port=11211

# Message queue configuration
rabbitmq.host=${RABBITMQ_HOST}
rabbitmq.port=5672
rabbitmq.username=admin
rabbitmq.password=${MQ_PASSWORD}

User-Data Script Example
#!/bin/bash
# User-data script for EC2 instance initialization
# This script runs when the instance first launches

# Update system
yum update -y

# Install Tomcat
yum install -y tomcat

# Download artifact from S3
aws s3 cp s3://vprofile-artifacts/vprofile-v2.war /usr/share/tomcat/webapps/ROOT.war

# Set environment variables from Parameter Store
export RDS_HOST=$(aws ssm get-parameter --name /vprofile/rds-host --query Parameter.Value --output text)
export RDS_PASSWORD=$(aws ssm get-parameter --name /vprofile/rds-password --with-decryption --query Parameter.Value --output text)

# Start Tomcat
systemctl start tomcat
systemctl enable tomcat

Testing
Functional Testing

# Test homepage
curl -I https://www.yourdomain.com

# Test login endpoint
curl -X POST https://www.yourdomain.com/login \
    -H "Content-Type: application/json" \
    -d '{"username":"test","password":"test"}'

# Test health check endpoint
curl https://www.yourdomain.com/login

# Verify load balancer health
aws elbv2 describe-target-health \
    --target-group-arn target-group-arn

Load Testing

# Using Apache Bench
ab -n 1000 -c 100 https://www.yourdomain.com/

# Using Siege
siege -c 50 -t 60s https://www.yourdomain.com/

# Monitor scaling during test
aws autoscaling describe-scaling-activities \
    --auto-scaling-group-name vprofile-env-asg

Performance Verification
✅ Response times under 500ms for API endpoints

✅ Database connection pool usage < 80%

✅ Cache hit rates > 90% in ElastiCache

✅ Message queue throughput within limits

Monitoring
CloudWatch Alarms

# CPU Utilization Alarm (Scale Out)
aws cloudwatch put-metric-alarm \
    --alarm-name vprofile-high-cpu \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --period 300 \
    --threshold 70 \
    --comparison-operator GreaterThanThreshold

# 5xx Error Rate Alarm
aws cloudwatch put-metric-alarm \
    --alarm-name vprofile-high-5xx \
    --metric-name HTTPCode_Target_5XX_Count \
    --namespace AWS/ApplicationELB \
    --period 300 \
    --threshold 10

Log Management

# Configure log streaming
eb logs

# Tail logs in real-time
aws logs tail /aws/elasticbeanstalk/vprofile-env/var/log/eb-activity.log --follow

# Download all logs
eb logs --all

| Log File	| Purpose |
|-----------------------------|------------------------------|
| /var/log/eb-activity.log	| Elastic Beanstalk deployment logs |
| /var/log/tomcat8/catalina.out	| Tomcat application logs |
| /var/log/cloud-init.log |	Instance initialization logs |

Cost Optimization
Compute Optimization

- Use Reserved Instances for steady-state workloads (40-60% savings)

- Right-size instances based on CloudWatch metrics

- Use Spot Instances for development environments

Storage Optimization
# Configure S3 lifecycle policies
aws s3api put-bucket-lifecycle-configuration \
    --bucket vprofile-artifacts \
    --lifecycle-configuration '{
        "Rules": [{
            "Status": "Enabled",
            "Prefix": "app-versions/",
            "Transitions": [{
                "Days": 30,
                "StorageClass": "STANDARD_IA"
            }],
            "Expiration": {"Days": 90}
        }]
    }'


Network Optimization
Enable CloudFront for static content (reduces origin load by 60-80%)

Configure compression in CloudFront

Set appropriate cache-control headers

Troubleshooting
Issue	Possible Cause	Solution
502 Bad Gateway	Application not starting	Check logs: eb logs
Database connection	Security group rules	Verify inbound rules on RDS SG
Health check failing	Wrong endpoint	Ensure /login is accessible
Deployment stuck	S3 permissions	Check bucket policy
High response time	Cache miss	Check ElastiCache connectivity
MQ connection refused	Port not open	Verify security group port 5672
504 Gateway Timeout	Load balancer timeout	Increase timeout settings
DNS resolution failed	Private hosted zone	Check VPC association

Debug Commands

# Check instance logs
ssh -i vprofile-key.pem ec2-user@instance-ip
tail -f /var/log/cloud-init-output.log

# Verify security groups
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Test DNS resolution
dig mysql.internal.myapp.local

# Check RDS connectivity
nc -zv vprofile-db.xxx.rds.amazonaws.com 3306

# Test backend connectivity from Tomcat instance
curl -I http://localhost:8080/login

## Technologies Used
| Category	| Technology |
|-----------|-------------------------------------|
| Framework	| Spring MVC, Spring Security, Spring Data JPA |
|Build Tool	|Maven|
| Frontend	| JSP, Bootstrap, CSS3, JavaScript |
| Database	| MySQL 8 |
| Caching	| Memcached (ElastiCache) |
| Messaging	| RabbitMQ (Amazon MQ) |
| Search	    | ElasticSearch |
| Cloud Services	| AWS EC2, RDS, ElastiCache, MQ, Beanstalk, CloudFront, Route53, S3, ACM, CloudWatch |
| Deployment	| Elastic Beanstalk, Auto Scaling |
| Monitoring	| CloudWatch, X-Ray | 

Contributing
Fork the repository

Create a feature branch (git checkout -b feature/amazing-feature)

Commit your changes (git commit -m 'Add amazing feature')

Push to the branch (git push origin feature/amazing-feature)

Open a Pull Request

License
This project is licensed under the MIT License - see the LICENSE file for details.

Acknowledgments
Original vProfile application team

AWS Documentation

Cloud computing best practices

Last Updated: March 2026
Version: 2.0.0
