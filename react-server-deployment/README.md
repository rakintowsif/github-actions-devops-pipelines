# React Server Deployment

This workflow demonstrates how to deploy a React application to a web server via SSH with Nginx configuration.

## 🚀 Deployment Process

### Trigger
- **Event**: Push to `master` branch
- **Environment**: Ubuntu runner

### Pipeline Steps
1. **Setup**: Node.js 20.x environment
2. **Dependencies**: Install npm packages
3. **Build**: Create production React build
4. **Transfer**: Copy files to production server via SCP
5. **Deploy**: Move files to web directory and set permissions
6. **Service**: Reload Nginx to serve new files

## 📋 Required Secrets

- `HOST_PRODUCTION` - Production server IP/hostname
- `USERNAME` - SSH username
- `SSHKEY_PRIVATE` - Private SSH key
- `PORT` - SSH port (default: 22)

## 🔧 Configuration

- **Source Directory**: `dist/` (React build output)
- **Remote Target**: `/home/ubuntu/cicd_front_production`
- **Web Directory**: `/var/web/html_new/production_frontend`
- **Web Server**: Nginx
- **File Owner**: `nginx:cardselling`

## 🎯 Features

- ✅ Secure SSH deployment
- ✅ Atomic file updates
- ✅ Proper file permissions
- ✅ Nginx service management
- ✅ Zero-downtime deployment

## name