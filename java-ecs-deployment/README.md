# Java ECS Deployment

This workflow demonstrates a production-ready CI/CD pipeline for deploying Java applications to Amazon ECS (Elastic Container Service) using modern DevOps practices.

## 🚀 CI/CD Pipeline Overview

### Deployment Flow

```
Git Push (master) → GitHub Actions → Docker Build → ECR Push → ECS Update → Service Stability → Slack Notification
```

## 🔄 Detailed Workflow Process

### Phase 1: Trigger & Setup
1. **Event**: Code push to `master` branch
2. **Concurrency Control**: Prevents duplicate deployments with `cancel-in-progress: true`
3. **Permissions**: OIDC authentication for secure AWS access
4. **Environment Variables**: Configure AWS region, account ID, and ECR repository

### Phase 2: Build & Containerization
1. **Code Checkout**: Retrieve latest source code
2. **AWS Authentication**: Assume OIDC role for secure AWS access
3. **ECR Login**: Authenticate with Amazon Elastic Container Registry
4. **Docker Buildx**: Set up advanced Docker build environment
5. **Image Tagging**: Generate short SHA and branch-specific tags
6. **Build & Push**: Create Docker image and push to ECR with caching optimization

### Phase 3: ECS Deployment
1. **Slack Notification**: Alert team that deployment has started
2. **Task Definition Download**: Retrieve current ECS task definition
3. **Image Update**: Replace container image in task definition with new version
4. **Task Registration**: Register new task definition version
5. **Service Update**: Deploy new task definition to ECS service
6. **Stability Wait**: Monitor until service reaches stable state
7. **Status Summary**: Display deployment results and service status
8. **Slack Notification**: Alert team of deployment success/failure

## 🏗️ Architecture Components

### Continuous Integration (CI)
- **Source Control**: Git-based version management
- **Build Environment**: GitHub Actions Ubuntu runners
- **Container Registry**: Amazon ECR with OIDC authentication
- **Image Tagging**: Semantic versioning with SHA and branch tags
- **Build Optimization**: GitHub Actions caching for faster builds

### Continuous Deployment (CD)
- **Orchestration**: Amazon ECS with Fargate
- **Service Management**: Automated service updates with force deployment
- **Health Monitoring**: Service stability verification
- **Rollback Capability**: Previous task definition versions maintained
- **Notification System**: Slack integration for team awareness

### Infrastructure Components
- **Container Registry**: Amazon ECR for Docker image storage
- **Orchestration**: Amazon ECS for container management
- **Compute**: AWS Fargate for serverless container execution
- **Networking**: VPC and security groups for secure communication
- **Monitoring**: AWS CloudWatch and service health checks

## 📋 Configuration Details

### Required GitHub Secrets
- `AWS_REGION` - AWS deployment region (e.g., us-east-1)
- `AWS_ACCOUNT_ID` - AWS account number (12-digit)
- `AWS_OIDC_ROLE` - OIDC role ARN for GitHub Actions
- `ECR_REPO_TM_CORE` - ECR repository name
- `SLACK_WEBHOOK_URL` - Slack webhook URL for notifications

### AWS Resources Required
- **ECS Cluster**: `teammart-cluster`
- **ECS Service**: `tm-core-service`
- **Task Definition**: `tm-core-prod-new`
- **ECR Repository**: Configured via `ECR_REPO_TM_CORE` secret
- **IAM Role**: OIDC role with ECS/ECR permissions

### Docker Configuration
- **Dockerfile**: Expected at `./Dockerfile`
- **Build Context**: Root directory of repository
- **Multi-platform**: Supports linux/amd64 and linux/arm64
- **Provenance**: Disabled for security compliance

## 🎯 Image Tagging Strategy

### Tag Format
- **SHA Tag**: `{short-sha}` (7-character commit hash)
- **Latest Tag**: `{branch}-latest` (e.g., `prod-latest`)

### Examples
```
# Master branch deployment
1234567 (SHA)
prod-latest (Latest)

# Development branch (if enabled)
abcdefg (SHA)
dev-latest (Latest)

# Staging branch (if enabled)
hijklmn (SHA)
staging-latest (Latest)
```

## 📊 Deployment Process Flow

### 1. Build Phase
```bash
# Image building process
docker build -t 1234567 .
docker build -t prod-latest .
docker push 1234567
docker push prod-latest
```

### 2. Task Definition Update
```bash
# Download current task definition
aws ecs describe-task-definition --task-definition tm-core-prod-new

# Update image reference
jq '.containerDefinitions[0].image = "new-image"' task-definition.json

# Register new version
aws ecs register-task-definition --cli-input-json new-task-definition.json
```

### 3. Service Deployment
```bash
# Update service with new task definition
aws ecs update-service \
  --cluster teammart-cluster \
  --service tm-core-service \
  --task-definition new-task-definition-arn \
  --force-new-deployment

# Wait for stability
aws ecs wait services-stable \
  --cluster teammart-cluster \
  --services tm-core-service
```

## 🔧 Security Features

### OIDC Authentication
- **No Static Credentials**: Eliminates need for long-lived AWS keys
- **Role-Based Access**: Temporary credentials via OIDC role assumption
- **Least Privilege**: Scoped permissions for ECS/ECR operations only
- **Automatic Rotation**: Credentials expire automatically after workflow

### Image Security
- **Provenance Disabled**: Prevents build metadata exposure
- **Private Registry**: ECR repositories are private by default
- **Tag Validation**: Regex validation for image URL format
- **Audit Trail**: All deployments logged in AWS CloudTrail

### Network Security
- **VPC Isolation**: Containers run in isolated VPC
- **Security Groups**: Network-level access controls
- **IAM Roles**: Container-level permissions via task roles

## 📈 Monitoring & Observability

### Deployment Metrics
- **Build Duration**: Time spent building and pushing images
- **Deployment Time**: Service update and stabilization duration
- **Success Rate**: Percentage of successful deployments
- **Rollback Frequency**: Number of manual rollbacks required

### Health Monitoring
- **Service Stability**: ECS service health checks
- **Container Health**: Application-level health monitoring
- **Resource Utilization**: CPU and memory usage tracking
- **Error Rates**: Application error monitoring

### Notification System
- **Deployment Start**: Immediate notification when deployment begins
- **Success Alert**: Confirmation when deployment completes successfully
- **Failure Alert**: Immediate notification with logs link on failure
- **Team Visibility**: All stakeholders informed via Slack integration

## 🚨 Error Handling & Recovery

### Automatic Recovery
- **Service Stability**: ECS automatically replaces unhealthy containers
- **Deployment Rollback**: Previous task definition versions available
- **Retry Logic**: Failed deployments can be retried automatically

### Manual Recovery
- **Task Definition Rollback**: Manually revert to previous task definition
- **Service Rollback**: Update service with previous task definition
- **Emergency Stop**: Cancel in-progress deployments via concurrency control

### Debugging Features
- **Detailed Logging**: Step-by-step deployment progress
- **Image Validation**: Format validation for ECR image URLs
- **Task Definition Summary**: Display of configuration before deployment
- **Service Status**: Real-time service health and deployment status

## 🎯 Best Practices Implemented

### CI/CD Excellence
- **Infrastructure as Code**: All deployment configuration in version control
- **Immutable Infrastructure**: New containers for each deployment
- **Automated Testing**: Build validation and image testing
- **Environment Parity**: Consistent environments across stages

### Security Best Practices
- **Zero Trust Security**: OIDC authentication with temporary credentials
- **Principle of Least Privilege**: Minimal required permissions
- **Secret Management**: Secure credential storage and rotation
- **Audit Logging**: Comprehensive deployment activity tracking

### Operational Excellence
- **Monitoring Integration**: Real-time deployment status tracking
- **Automated Notifications**: Team awareness via Slack integration
- **Rollback Capability**: Quick recovery from failed deployments
- **Documentation**: Comprehensive configuration and process documentation

## 📖 Usage Instructions

### Prerequisites
1. **AWS Account**: With ECS, ECR, and IAM configured
2. **GitHub Repository**: With workflow file in `.github/workflows/`
3. **Dockerfile**: Application containerization configuration
4. **ECS Resources**: Cluster, service, and task definition created

### Setup Steps
1. **Configure Secrets**: Add required GitHub repository secrets
2. **Create OIDC Role**: Set up IAM role for GitHub Actions
3. **Configure ECS**: Create cluster, service, and task definition
4. **Test Deployment**: Push to master branch to trigger pipeline

### Troubleshooting
- **Permission Errors**: Verify OIDC role and IAM permissions
- **Build Failures**: Check Dockerfile and application dependencies
- **Deployment Issues**: Review ECS service configuration and task definition
- **Notification Problems**: Validate Slack webhook URL and formatting
