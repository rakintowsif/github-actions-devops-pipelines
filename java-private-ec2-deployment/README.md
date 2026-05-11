# Java Private EC2 Deployment

This workflow demonstrates how to deploy a Java scheduler application to a private EC2 instance using a bastion host for secure access.

## 🚀 Deployment Overview

### Architecture
```
GitHub Actions → Bastion Host → Private EC2 Instance → Docker Containers
```

### Security Model
- **Public Access**: Only bastion host accessible from internet
- **Private Network**: Application server in private subnet
- **SSH Tunneling**: Secure connection through bastion
- **Container Isolation**: Application runs in Docker containers

## 🔄 Deployment Process

### Trigger
- **Event**: Code push to `master` branch
- **Environment**: Ubuntu runner

### Deployment Steps
1. **SSH Connection**: Establish connection through bastion host
2. **Code Pull**: Update application repository on private server
3. **Container Deployment**: Rebuild and restart Docker containers
4. **Health Check**: Verify container is running correctly

## 📋 Required Secrets

### Primary Connection
- `HOST` - Private EC2 instance IP/hostname
- `USER` - SSH username for private EC2
- `SSH_PRIVATE_KEY` - Private SSH key for private EC2
- `PORT` - SSH port for private EC2 (default: 22)

### Bastion Configuration
- `BASTION_HOST` - Public bastion host IP/hostname
- `BASTION_USER` - SSH username for bastion host
- `BASTION_SSH_KEY` - Private SSH key for bastion host
- `BASTION_PORT` - SSH port for bastion host (default: 22)

## 🔧 Configuration Details

### Application Structure
```
/var/web/html_new/Schedular-Cardselling-Core/  # Application code
/opt/cs-core-scheduler/                      # Docker compose directory
```

### Container Management
- **Compose File**: Docker Compose configuration in `/opt/cs-core-scheduler/`
- **Container Name**: `cs-core-dkr-scheduler`
- **Deployment Strategy**: Down → Build → Up (zero-downtime)
- **Health Check**: Container process verification

## 🏗️ Architecture Components

### Network Security
- **Bastion Host**: Public-facing jump server
- **Private Subnet**: Application server isolated from internet
- **VPC**: Virtual Private Cloud for network isolation
- **Security Groups**: Firewall rules for access control

### Container Orchestration
- **Docker Engine**: Container runtime on private EC2
- **Docker Compose**: Multi-container application management
- **Volume Management**: Persistent data storage
- **Network Isolation**: Container networking configuration

### Deployment Security
- **SSH Key Authentication**: Key-based authentication only
- **Bastion Tunneling**: Encrypted connection through bastion
- **Least Privilege**: Minimal access permissions
- **No Direct Access**: Private server not internet-facing

## 📊 Deployment Flow

### 1. Connection Establishment
```bash
# GitHub Actions establishes SSH tunnel
ssh -i bastion_key -J bastion_user@bastion_host \
    -i private_key private_user@private_host
```

### 2. Code Update
```bash
# Navigate to application directory
cd /var/web/html_new/Schedular-Cardselling-Core

# Pull latest changes
git fetch origin
git reset --hard origin/master
```

### 3. Container Deployment
```bash
# Navigate to Docker directory
cd /opt/cs-core-scheduler

# Stop existing containers
docker compose down

# Build and start new containers
docker compose up -d --build
```

### 4. Health Verification
```bash
# Check if container is running
docker ps | grep cs-core-dkr-scheduler
```

## 🔒 Security Best Practices

### Network Security
- **Bastion Pattern**: Single entry point to private network
- **Private Subnets**: Application servers not internet-accessible
- **Security Groups**: Restrictive firewall rules
- **VPC Flow Logs**: Network traffic monitoring

### Access Control
- **SSH Keys Only**: No password authentication
- **Key Rotation**: Regular SSH key updates
- **User Separation**: Different users for bastion and private servers
- **Session Logging**: SSH session audit trails

### Container Security
- **Non-root Containers**: Run containers as non-root user
- **Resource Limits**: CPU and memory constraints
- **Network Isolation**: Container network segmentation
- **Image Scanning**: Regular vulnerability assessments

## 🚨 Error Handling & Recovery

### Connection Issues
- **Bastion Unreachable**: Verify bastion host status and SSH key
- **Private Server Unreachable**: Check VPC routing and security groups
- **Key Authentication**: Validate SSH key permissions and formats
- **Port Blocking**: Ensure SSH ports are open in security groups

### Deployment Failures
- **Git Pull Errors**: Check repository access and permissions
- **Docker Build Failures**: Validate Dockerfile and dependencies
- **Container Startup Issues**: Review container logs and configuration
- **Port Conflicts**: Check for port usage conflicts

### Recovery Procedures
- **Manual Rollback**: Revert to previous git commit
- **Container Restart**: Manual Docker commands for recovery
- **Service Recovery**: Individual container restart procedures
- **Emergency Access**: Direct bastion connection for troubleshooting

## 🎯 Benefits

### Security Advantages
- **Network Isolation**: Private servers not exposed to internet
- **Controlled Access**: Single entry point through bastion
- **Encrypted Traffic**: All connections encrypted via SSH
- **Audit Trail**: Complete access logging and monitoring

### Operational Benefits
- **Automated Deployment**: Zero-touch deployment process
- **Container Consistency**: Identical environments across deployments
- **Fast Rollback**: Quick reversion to previous versions
- **Health Monitoring**: Automated container status verification

### Development Benefits
- **Git Integration**: Direct integration with version control
- **Continuous Deployment**: Automatic deployment on code changes
- **Environment Parity**: Consistent deployment environment
- **Debugging Support**: Container logs and status monitoring

## 📖 Usage Instructions

### Prerequisites
1. **VPC Setup**: With public and private subnets
2. **Bastion Host**: Configured with SSH access
3. **Private EC2**: Application server in private subnet
4. **Docker Engine**: Installed and configured on private server
5. **Application Code**: Git repository with proper structure

### Setup Steps
1. **Configure Security Groups**: Allow SSH from bastion to private server
2. **Set Up SSH Keys**: Generate and distribute keys appropriately
3. **Install Docker**: On private EC2 instance
4. **Configure Application**: Set up directories and permissions
5. **Add GitHub Secrets**: Configure all required secrets

### Monitoring
- **Container Status**: `docker ps` command output
- **Application Logs**: Docker container logs
- **System Resources**: EC2 instance metrics
- **Network Connectivity**: Bastion and private server access

### Troubleshooting
- **Connection Issues**: Verify bastion and private server connectivity
- **Deployment Failures**: Check Docker compose configuration
- **Application Errors**: Review container logs and application status
- **Permission Problems**: Validate file and directory permissions
