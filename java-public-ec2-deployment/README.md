# Java Public EC2 Deployment

This workflow demonstrates a production-ready CI/CD pipeline for deploying Java applications to public EC2 instances using Docker Compose with automated health checks and rollback capabilities.

## 🚀 Deployment Overview

### Architecture
```
GitHub Actions → ECR Registry → Public EC2 Instance → Docker Compose → Health Check
```

### Deployment Strategy
- **Build Phase**: Create and push Docker image to ECR
- **Deploy Phase**: SSH to public EC2, pull image, restart containers
- **Health Monitoring**: Automated health checks with rollback capability
- **Cleanup**: Automatic removal of old Docker images

## 🔄 CI/CD Pipeline Process

### Phase 1: Build & Push to ECR
1. **Trigger**: Code push to `master` branch
2. **Concurrency Control**: Prevent duplicate deployments
3. **AWS Authentication**: Configure credentials for ECR access
4. **Docker Build**: Build container image with multi-platform support
5. **Image Tagging**: Create SHA and branch-specific tags
6. **ECR Push**: Store image in Amazon ECR registry

### Phase 2: Deploy to Production
1. **SSH Connection**: Secure connection to public EC2 instance
2. **ECR Login**: Authenticate with Amazon ECR
3. **Environment Update**: Update image tag in .env file
4. **Container Deployment**: Pull new image and restart containers
5. **Health Check**: Monitor container health for up to 2 minutes
6. **Automatic Rollback**: Revert to previous version if health check fails
7. **Cleanup**: Remove unused Docker images

## 📋 Required Secrets

### AWS Configuration
- `AWS_REGION` - AWS region (e.g., us-east-1)
- `AWS_ACCESS_KEY_ID` - AWS access key ID
- `AWS_SECRET_ACCESS_KEY` - AWS secret access key
- `ECR_REPO_CS_KPI` - ECR repository name for KPI application

### Server Configuration
- `HOST` - Public EC2 instance IP/hostname
- `USERNAME` - SSH username for EC2 instance
- `SSHKEY_PRIVATE` - Private SSH key for EC2 access
- `PORT` - SSH port (default: 22)

## 🔧 Configuration Details

### Application Structure
```
/opt/cs-kpi/                    # Application directory
├── docker-compose.yml          # Docker Compose configuration
├── .env                       # Environment variables
└── cs-kpi-dkr-prod           # Running container
```

### Environment Variables
- `IMAGE_TAG_KPI` - Docker image tag for KPI application
- `REGION` - AWS region for ECR access
- `REPO` - ECR repository name

### Image Tagging Strategy
```
# Master branch
1234567 (SHA tag)
prod-latest (Latest tag)

# Development branch (if enabled)
abcdefg (SHA tag)
dev-latest (Latest tag)

# Staging branch (if enabled)
hijklmn (SHA tag)
staging-latest (Latest tag)
```

## 🏗️ Architecture Components

### Container Registry
- **Amazon ECR**: Private Docker image registry
- **Multi-platform Support**: linux/amd64 and linux/arm64
- **Version Control**: SHA-based and branch-based tagging
- **Access Control**: IAM-based authentication

### Deployment Infrastructure
- **Public EC2**: Internet-accessible server
- **Docker Engine**: Container runtime environment
- **Docker Compose**: Multi-container orchestration
- **Health Checks**: Container health monitoring

### Security Features
- **SSH Key Authentication**: Key-based server access
- **IAM Credentials**: Temporary AWS credentials
- **Private Registry**: Secure image storage
- **Network Isolation**: Container-level security

## 📊 Deployment Flow

### Build Phase
```bash
# Docker image creation
docker build -t 1234567 .
docker build -t prod-latest .
docker push 1234567
docker push prod-latest
```

### Deployment Phase
```bash
# SSH to EC2 instance
ssh -i private_key user@public_host

# Navigate to application directory
cd /opt/cs-kpi

# Login to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin

# Update environment file
sed -i "s|IMAGE_TAG_KPI=.*|IMAGE_TAG_KPI=1234567|" .env

# Deploy new container
docker compose pull cs-kpi
docker compose up -d cs-kpi
```

### Health Check Process
```bash
# Monitor container health
for i in {1..60}; do
  if docker inspect --format='{{.State.Health.Status}}' cs-kpi-dkr-prod | grep -q "healthy"; then
    echo "Container healthy"
    break
  fi
  sleep 2
done
```

### Rollback Process
```bash
# Revert to previous image tag
sed -i "s|IMAGE_TAG_KPI=.*|IMAGE_TAG_KPI=previous_tag|" .env
docker compose pull cs-kpi
docker compose up -d cs-kpi
```

## 🔒 Security Features

### Authentication & Access
- **SSH Key Only**: No password authentication
- **IAM Credentials**: Temporary AWS access via GitHub Actions
- **Private Registry**: ECR repositories are private by default
- **Key Rotation**: Regular credential updates recommended

### Container Security
- **Non-root Execution**: Containers run as non-privileged users
- **Health Monitoring**: Automated container health checks
- **Resource Limits**: CPU and memory constraints
- **Network Isolation**: Container-level networking

### Network Security
- **Security Groups**: EC2 firewall configuration
- **VPC**: Virtual Private Cloud isolation
- **SSH Port Control**: Configurable SSH access
- **Audit Logging**: AWS CloudTrail integration

## 📈 Monitoring & Health Checks

### Container Health Monitoring
- **Health Status**: Docker health check integration
- **Timeout**: 2-minute health check window
- **Automatic Rollback**: Failed deployments revert automatically
- **Status Reporting**: Real-time health status updates

### Deployment Metrics
- **Build Duration**: Time spent building and pushing images
- **Deployment Time**: Container restart and health check duration
- **Success Rate**: Percentage of successful deployments
- **Rollback Frequency**: Number of automatic rollbacks

### Resource Management
- **Image Cleanup**: Automatic removal of old images
- **Disk Space**: Optimized storage usage
- **Memory Usage**: Container resource monitoring
- **Network Traffic**: Application performance metrics

## 🚨 Error Handling & Recovery

### Automatic Recovery
- **Health Check Failures**: Automatic rollback to previous version
- **Container Crashes**: Docker Compose automatic restarts
- **Network Issues**: Retry logic for failed connections
- **Image Pull Failures**: Fallback to cached images

### Manual Recovery
- **Emergency Rollback**: Manual reversion to previous version
- **Container Management**: Manual Docker commands for recovery
- **Service Restart**: Individual container restart procedures
- **Debug Access**: Direct server access for troubleshooting

### Error Scenarios
- **Build Failures**: Docker build errors and dependency issues
- **Push Failures**: ECR authentication and network problems
- **Deploy Failures**: SSH connection and container issues
- **Health Check Failures**: Application startup problems

## 🎯 Benefits & Features

### Deployment Excellence
- **Zero-Downtime**: Seamless container updates
- **Automated Rollback**: Failed deployments auto-revert
- **Health Monitoring**: Real-time application health checks
- **Version Control**: Complete deployment history tracking

### Operational Benefits
- **Automated Cleanup**: Efficient disk space management
- **Concurrency Control**: Prevents overlapping deployments
- **Environment Consistency**: Identical deployment environments
- **Fast Recovery**: Quick rollback capabilities

### Security Advantages
- **Private Registry**: Secure image storage
- **Key Authentication**: Secure server access
- **IAM Integration**: Temporary credential management
- **Audit Trail**: Complete deployment logging

## 📖 Usage Instructions

### Prerequisites
1. **AWS Account**: With ECR and IAM configured
2. **Public EC2**: With Docker and Docker Compose installed
3. **Application**: With Dockerfile and docker-compose.yml
4. **SSH Access**: Key-based authentication configured
5. **Network**: Security groups allowing SSH access

### Setup Steps
1. **Configure Secrets**: Add all required GitHub repository secrets
2. **Create ECR Repository**: Set up private Docker registry
3. **Set Up EC2**: Configure server with Docker and SSH
4. **Deploy Application**: Initial manual deployment setup
5. **Test Pipeline**: Push to master branch to trigger CI/CD

### Monitoring Commands
```bash
# Check container status
docker ps

# View container logs
docker logs cs-kpi-dkr-prod

# Monitor health status
docker inspect --format='{{.State.Health.Status}}' cs-kpi-dkr-prod

# View deployment history
grep "IMAGE_TAG_KPI" .env.bak
```

### Troubleshooting
- **SSH Connection**: Verify host, user, and SSH key configuration
- **ECR Access**: Check AWS credentials and permissions
- **Docker Issues**: Review Dockerfile and application dependencies
- **Health Check Failures**: Examine application logs and startup process
- **Rollback Problems**: Verify previous image tags and environment variables
