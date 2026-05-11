# React S3 Static Deployment

This workflow demonstrates how to deploy a React application to AWS S3 static hosting with CloudFront CDN.

## 🚀 Deployment Process

### Trigger
- **Event**: Push to `master` branch
- **Environment**: Ubuntu runner

### Pipeline Steps
1. **Setup**: Node.js 20.x environment
2. **Dependencies**: Install npm packages
3. **Build**: Create production React build
4. **Deploy**: Sync build files to S3 bucket
5. **CDN**: Invalidate CloudFront cache

## 📋 Required Secrets

- `AWS_BUCKET_NAME` - S3 bucket name
- `AWS_ACCESS_KEY_ID_S3_WEB` - AWS access key
- `AWS_SECRET_ACCESS_KEY_S3_WEB` - AWS secret key
- `AWS_REGION` - AWS region
- `AWS_CLOUDFRONT_DISTRIBUTION_ID` - CloudFront distribution ID

## 🔧 Configuration

- **Source Directory**: `dist/` (React build output)
- **S3 Permissions**: Public read access
- **Cache Invalidation**: All files (`/*`)

## 🎯 Features

- ✅ Automated static site deployment
- ✅ CDN cache management
- ✅ Zero-downtime deployment
- ✅ Atomic file updates
- ✅ Rollback capability
