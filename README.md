
# AWS Cloud-Native Web Application Architecture

## Project Overview
This project demonstrates a highly scalable, flexible, and low-operational-overhead web application deployment on AWS. Instead of manually managing infrastructure, we leverage **Platform as a Service (PaaS)** and **Software as a Service (SaaS)** offerings to implement a "pay-as-you-go" model with Infrastructure as Code principles.

The architecture replaces traditional self-managed components (like Tomcat on EC2, MySQL on VMs) with managed AWS services such as **Elastic Beanstalk**, **RDS**, **ElastiCache**, and **Amazon MQ**.

## AWS Services Used

### Frontend Services
- **Elastic Beanstalk**: Manages EC2 instances, auto-scaling, and load balancer automatically
- **EC2 Instances**: Hosts the Tomcat application (managed by Beanstalk)
- **S3 Bucket**: Stores deployment artifacts
- **CloudWatch**: Monitors auto-scaling groups with alarms

### Backend Services
- **RDS**: Managed database service (replaces MySQL on EC2)
- **ElastiCache**: Managed caching service (replaces Memcache)
- **Amazon MQ**: Managed message broker (replaces RabbitMQ/ActiveMQ)

### Networking & Delivery
- **Route 53**: DNS management service
- **CloudFront**: Content Delivery Network (CDN) for global audience

## Service Comparison

| Traditional Component | AWS Managed Service | Benefit |
| :--- | :--- | :--- |
| **Tomcat on EC2** | **Elastic Beanstalk** | Automated patching, deployment, and capacity management |
| **Manual Load Balancer** | **Elastic Beanstalk (ALB)** | Integrated health checks and SSL termination |
| **NFS** | **EFS / S3** | Fully managed, scalable storage |
| **MySQL on EC2** | **Amazon RDS** | Automated backups, updates, and Multi-AZ failover |
| **Memcache on EC2** | **Amazon ElastiCache** | Managed in-memory caching with built-in replication |
| **RabbitMQ on EC2** | **Amazon MQ** | Managed message broker with easy migration |
| **Custom DNS** | **Route 53** | Reliable and scalable DNS management |
| **Global Delivery** | **CloudFront** | Low latency content delivery for global audience |

## Request Flow Diagram

```mermaid
sequenceDiagram
    participant User
    participant Route53
    participant CloudFront
    participant ALB as Beanstalk ALB
    participant EC2 as EC2 (Auto Scaling)
    participant Backend as Backend Services
    participant S3

    User->>Route53: 1. Access URL
    Route53->>CloudFront: 2. DNS Resolution
    CloudFront->>ALB: 3. Forward Request
    ALB->>EC2: 4. Route to Healthy Instance
    EC2->>Backend: 5. Fetch Data (RDS/ElastiCache/MQ)
    Backend-->>EC2: 6. Return Data
    EC2-->>User: 7. Serve Response
    Note over EC2, S3: Artifacts pulled from S3 during deployment

## Execution Flow
graph TD
    A[Login to AWS Account] --> B[Create Key Pair for EC2 access]
    B --> C[Create Security Groups<br/>Backend Services]
    C --> D[Launch Backend Services]
    D --> D1[Create RDS Instance]
    D --> D2[Create ElastiCache Cluster]
    D --> D3[Create Amazon MQ Broker]
    D1 --> E[Create Elastic Beanstalk Environment]
    E --> F[Update Security Groups<br/>Allow Beanstalk → Backend]
    F --> G[Launch Temporary EC2 Instance<br/>Initialize RDS Database]
    G --> H[Modify Beanstalk Config<br/>Health Check URL: /login]
    H --> I[Add HTTPS Listener<br/>Port 443 to ELB]
    I --> J[Build Artifact<br/>Inject Backend Endpoints]
    J --> K[Deploy Artifact to Beanstalk]
    K --> L[Create CloudFront Distribution<br/>with SSL Certificate]
    L --> M[Update DNS Records<br/>GoDaddy/Route 53]
    M --> N[Test Application via URL]

## Detailed Setup Instructions
1. Prerequisites
- Active AWS Account with administrative access
- AWS CLI installed and configured
- Git installed on your local machine
- Java 8 or higher (for building the application)
- Maven or Gradle (depending on your build tool)
- Domain name (optional, for production deployment)
- Basic knowledge of AWS services and networking concepts

2. Initial AWS Setup
# Configure AWS CLI with your credentials
aws configure

# Create a key pair for EC2 instance access
aws ec2 create-key-pair \
    --key-name beanstalk-key \
    --query 'KeyMaterial' \
    --output text > beanstalk-key.pem

# Set proper permissions for the key file
chmod 400 beanstalk-key.pem

# Create an S3 bucket for artifacts
aws s3 mb s3://your-app-artifacts-bucket --region us-east-1

# Enable versioning on the bucket
aws s3api put-bucket-versioning \
    --bucket your-app-artifacts-bucket \
    --versioning-configuration Status=Enabled

3. Backend Services Setup
# RDS Database
# Create RDS subnet group
aws rds create-db-subnet-group \
    --db-subnet-group-name myapp-db-subnet \
    --db-subnet-group-description "Subnet group for RDS" \
    --subnet-ids subnet-xxx subnet-yyy

# Create RDS instance
aws rds create-db-instance \
    --db-instance-identifier myapp-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password YourPassword123 \
    --allocated-storage 20 \
    --db-subnet-group-name myapp-db-subnet \
    --vpc-security-group-ids sg-backend-id

# Configuration Notes:
- Choose database engine (MySQL/PostgreSQL)
- Select appropriate instance size based on workload
- Enable automated backups (7-day retention minimum)
- Enable Multi-AZ for production workloads
- Note the endpoint for application configuration

# ElastiCache

# Create ElastiCache subnet group
aws elasticache create-cache-subnet-group \
    --cache-subnet-group-name myapp-cache-subnet \
    --cache-subnet-group-description "Subnet group for ElastiCache" \
    --subnet-ids subnet-xxx subnet-yyy

# Create ElastiCache cluster
aws elasticache create-cache-cluster \
    --cache-cluster-id myapp-cache \
    --cache-node-type cache.t3.micro \
    --engine memcached \
    --num-cache-nodes 1 \
    --cache-subnet-group-name myapp-cache-subnet \
    --security-group-ids sg-backend-id

# Configuration Notes:
- Select cluster engine (Memcached/Redis)
- Configure node type and number
- Enable multi-AZ if needed
- Note the configuration endpoint

# Amazon MQ

# Create Amazon MQ broker
aws mq create-broker \
    --broker-name myapp-mq-broker \
    --broker-instance-type t3.micro \
    --engine-type ACTIVEMQ \
    --engine-version 5.17.3 \
    --deployment-mode SINGLE_INSTANCE \
    --host-instance-type t3.micro \
    --auto-minor-version-upgrade \
    --users Username=admin,Password=YourPassword123

# Configuration Notes:
- Choose broker engine (ActiveMQ/RabbitMQ)
- Select appropriate instance type
- Configure users and permissions
- Note the broker endpoint

4. Elastic Beanstalk Environment

# Install EB CLI
pip install awsebcli

# Initialize Elastic Beanstalk application
eb init \
    --platform "Tomcat 8.5 with Java 8" \
    --region us-east-1 \
    --keyname beanstalk-key

# Create Elastic Beanstalk environment
eb create myapp-env \
    --elb-type application \
    --instance-types t3.micro \
    --min-instances 2 \
    --max-instances 6 \
    --scale 20

# Configure environment variables
eb setenv \
    RDS_ENDPOINT=myapp-db.xxx.us-east-1.rds.amazonaws.com \
    ELASTICACHE_ENDPOINT=myapp-cache.xxx.cache.amazonaws.com \
    AMAZONMQ_ENDPOINT=b-xxx.mq.us-east-1.amazonaws.com

# Create Elastic Beanstalk environment
eb init --platform "Tomcat" --region us-east-1
eb create my-app-environment --elb-type application

5. Security Group Configuration

# Get the security group IDs after Beanstalk creation
aws ec2 describe-security-groups \
    --filters Name=group-name,Values=*beanstalk*

# Allow Beanstalk instances to access RDS
aws ec2 authorize-security-group-ingress \
    --group-id sg-backend-id \
    --protocol tcp \
    --port 3306 \
    --source-group sg-beanstalk-ec2-id

# Allow Beanstalk instances to access ElastiCache
aws ec2 authorize-security-group-ingress \
    --group-id sg-backend-id \
    --protocol tcp \
    --port 11211 \
    --source-group sg-beanstalk-ec2-id

# Allow Beanstalk instances to access Amazon MQ
aws ec2 authorize-security-group-ingress \
    --group-id sg-backend-id \
    --protocol tcp \
    --port 61616 \
    --source-group sg-beanstalk-ec2-id

# Allow internal backend communication
aws ec2 authorize-security-group-ingress \
    --group-id sg-backend-id \
    --protocol tcp \
    --port 0-65535 \
    --source-group sg-backend-id

6. Database Initialization

# Launch temporary EC2 instance for database initialization
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t2.micro \
    --key-name beanstalk-key \
    --security-group-ids sg-backend-id \
    --subnet-id subnet-xxx

# SSH into the instance
ssh -i beanstalk-key.pem ec2-user@instance-public-ip

# Install MySQL client
sudo yum install mysql -y

# Connect to RDS and initialize database
mysql -h myapp-db.xxx.us-east-1.rds.amazonaws.com -u admin -p

# Create database and tables
CREATE DATABASE myapp;
USE myapp;

# Run your schema creation script
SOURCE schema.sql;

# Insert initial data if needed
SOURCE seed_data.sql;

# Verify the setup
SHOW TABLES;
SELECT * FROM users LIMIT 5;

# Exit MySQL and terminate the temporary instance
exit

7. Application Configuration

Update src/main/resources/application.properties or application.yml:

8. Build and Deploy

# Clean and build the application
mvn clean package

# Run tests
mvn test

# Verify the artifact
ls -la target/*.war

# Upload artifact to S3
aws s3 cp target/ROOT.war s3://your-app-artifacts-bucket/app-versions/

# Create application version in Elastic Beanstalk
aws elasticbeanstalk create-application-version \
    --application-name myapp \
    --version-label v1.0.0 \
    --source-bundle S3Bucket="your-app-artifacts-bucket",S3Key="app-versions/ROOT.war"

# Deploy to Beanstalk environment
aws elasticbeanstalk update-environment \
    --environment-name myapp-env \
    --version-label v1.0.0

# Monitor deployment status
eb events --follow

9. DNS Configuration

# If using Route 53
# Create a hosted zone
aws route53 create-hosted-zone \
    --name yourdomain.com \
    --caller-reference 2024-01-01-00:00:00

# Create A record pointing to Elastic Beanstalk
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
                    "DNSName": "myapp-env.elasticbeanstalk.com",
                    "EvaluateTargetHealth": false
                }
            }
        }]
    }'

# If using external registrar (GoDaddy, etc.)
# Update nameservers in your registrar's DNS settings to Route 53 nameservers
# OR create CNAME record pointing to your Beanstalk URL

# Health Check Configuration

# Configure health check path in Elastic Beanstalk
aws elasticbeanstalk update-environment \
    --environment-name myapp-env \
    --option-settings \
        Namespace=aws:elasticbeanstalk:application,OptionName=Application Healthcheck URL,Value=/login

# Verify health check is working
aws elasticbeanstalk describe-environments \
    --environment-names myapp-env \
    --query 'Environments[0].Health'

# Set health check thresholds in load balancer
aws elbv2 modify-target-group \
    --target-group-arn arn:aws:elasticloadbalancing:region:account:targetgroup/targetgroup/xxx \
    --health-check-path /login \
    --health-check-interval-seconds 30 \
    --health-check-timeout-seconds 5 \
    --healthy-threshold-count 3 \
    --unhealthy-threshold-count 3

# HTTPS Configuration

# Request SSL certificate from ACM
aws acm request-certificate \
    --domain-name www.yourdomain.com \
    --validation-method DNS \
    --subject-alternative-names yourdomain.com

# Get certificate ARN and add to load balancer listener
aws elbv2 create-listener \
    --load-balancer-arn load-balancer-arn \
    --protocol HTTPS \
    --port 443 \
    --certificates CertificateArn=arn:aws:acm:region:account:certificate/cert-id \
    --default-actions Type=forward,TargetGroupArn=target-group-arn

# Redirect HTTP to HTTPS
aws elbv2 create-listener \
    --load-balancer-arn load-balancer-arn \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=redirect,RedirectConfig={Protocol=HTTPS,Port=443,StatusCode=HTTP_301}

## Monitoring and Alarms
# CloudWatch Alarms Setup

# CPU Utilization Alarm (Scale Out)
aws cloudwatch put-metric-alarm \
    --alarm-name myapp-high-cpu \
    --alarm-description "Scale out when CPU > 70%" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --evaluation-periods 2 \
    --threshold 70 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=AutoScalingGroupName,Value=myapp-env-asg \
    --alarm-actions arn:aws:autoscaling:region:account:scalingPolicy:xxx

# Low CPU Alarm (Scale In)
aws cloudwatch put-metric-alarm \
    --alarm-name myapp-low-cpu \
    --alarm-description "Scale in when CPU < 30%" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --evaluation-periods 5 \
    --threshold 30 \
    --comparison-operator LessThanThreshold \
    --dimensions Name=AutoScalingGroupName,Value=myapp-env-asg \
    --alarm-actions arn:aws:autoscaling:region:account:scalingPolicy:yyy

# 5xx Error Rate Alarm
aws cloudwatch put-metric-alarm \
    --alarm-name myapp-high-5xx \
    --alarm-description "Alert on high 5xx errors" \
    --metric-name HTTPCode_Target_5XX_Count \
    --namespace AWS/ApplicationELB \
    --statistic Sum \
    --period 300 \
    --evaluation-periods 1 \
    --threshold 10 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=LoadBalancer,Value=myapp-env-alb \
    --alarm-actions arn:aws:sns:region:account:myapp-alerts

# Logs

# Configure log streaming to CloudWatch
aws elasticbeanstalk update-environment \
    --environment-name myapp-env \
    --option-settings \
        Namespace=aws:elasticbeanstalk:cloudwatch:logs,OptionName=StreamLogs,Value=true

# View logs from CLI
eb logs

# Tail logs in real-time
aws logs tail /aws/elasticbeanstalk/myapp-env/var/log/eb-activity.log --follow

# Download all logs
eb logs --all

# Common log locations in EC2 instances:
# /var/log/eb-activity.log - Elastic Beanstalk deployment logs
# /var/log/tomcat8/catalina.out - Tomcat application logs
# /var/log/cloud-init.log - Instance initialization logs
# /var/log/myapp/application.log - Custom application logs


## Testing the Deployment
# Functional Testing

# Test homepage
curl -I https://www.yourdomain.com

# Test login endpoint
curl -X POST https://www.yourdomain.com/login \
    -H "Content-Type: application/json" \
    -d '{"username":"test","password":"test"}'

# Test health check endpoint
curl https://www.yourdomain.com/actuator/health

# Test load balancer health
aws elbv2 describe-target-health \
    --target-group-arn target-group-arn

# Load Testing

# Using Apache Bench
ab -n 1000 -c 100 https://www.yourdomain.com/

# Using Siege
siege -c 50 -t 60s https://www.yourdomain.com/

# Monitor scaling during load test
aws autoscaling describe-scaling-activities \
    --auto-scaling-group-name myapp-env-asg

# Performance Verification

- Verify response times under 500ms for API endpoints
- Check database connection pool usage
- Monitor cache hit rates in ElastiCache
- Verify message queue throughput

## Cost Optimization
#Compute Optimization

# Use reserved instances for steady-state workloads
- RDS: 1-year reserved instance for 40% savings
- ElastiCache: 1-year reserved nodes for 30% savings
- EC2: Spot instances for development environments

# Right-size instances
- Use CloudWatch metrics to identify underutilized resources
- Downsize instances if CPU usage consistently below 20%
- Consider serverless alternatives (Aurora Serverless, ElastiCache Serverless)

# Storage Optimization

# Implement S3 lifecycle policies
aws s3api put-bucket-lifecycle-configuration \
    --bucket your-app-artifacts-bucket \
    --lifecycle-configuration '{
        "Rules": [{
            "Status": "Enabled",
            "Prefix": "app-versions/",
            "Transitions": [{
                "Days": 30,
                "StorageClass": "STANDARD_IA"
            }],
            "Expiration": {
                "Days": 90
            }
        }]
    }'

# Configure RDS automated backup retention
aws rds modify-db-instance \
    --db-instance-identifier myapp-db \
    --backup-retention-period 7

## Network Optimization

- Enable CloudFront for static content (reduces origin load by 60-80%)
- Configure compression in CloudFront
- Set appropriate cache-control headers
- Use AWS Global Accelerator for dynamic content

# Conclusion

This architecture provides a production-ready, scalable web application deployment with minimal operational overhead. By leveraging AWS managed services.

# Prerequisites
#
- JDK 11 
- Maven 3 
- MySQL 8

# Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- Tomcat
- MySQL
- Memcached
- Rabbitmq
- ElasticSearch
# Database
Here,we used Mysql DB 
sql dump file:
- /src/main/resources/db_backup.sql
- db_backup.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < db_backup.sql