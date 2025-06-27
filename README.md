# 🚀 Kubernetes ConfigMap Web Server

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)](https://nginx.org/)


A comprehensive example demonstrating how to deploy a web server using **Docker** and **Kubernetes** on Ubuntu, with HTML content served from a **ConfigMap**. This project showcases modern containerization and orchestration practices.

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Deployment Guide](#-deployment-guide)
- [Configuration](#-configuration)
- [Browser Output](#-browser-output)
- [Scaling & Management](#-scaling--management)
- [Monitoring](#-monitoring)
- [Troubleshooting](#-troubleshooting)
- [Production Considerations](#-production-considerations)
- [Contributing](#-contributing)
- [License](#-license)

## 🌟 Overview

This project demonstrates a complete workflow for deploying a web application on Kubernetes using ConfigMaps to manage HTML content. It's designed as a learning resource for DevOps engineers, developers, and anyone interested in Kubernetes fundamentals.

### Key Learning Objectives
- Understanding Kubernetes ConfigMaps
- Container orchestration with Kubernetes
- Service discovery and networking
- Resource management and scaling
- Production-ready deployment practices

## ✨ Features

- 🐳 **Containerized with Docker** - Lightweight nginx-alpine based container
- ☸️ **Kubernetes Native** - Uses Deployments, Services, and ConfigMaps
- 📦 **ConfigMap Integration** - HTML content served from Kubernetes ConfigMaps
- 🔄 **Auto-scaling Ready** - Horizontal Pod Autoscaler compatible
- 🛡️ **Resource Limits** - Proper CPU and memory constraints
- 📊 **Health Checks** - Readiness and liveness probes
- 🔧 **Easy Configuration** - Simple YAML-based configuration
- 📱 **Responsive Design** - Mobile-friendly web interface

## 📋 Prerequisites

Before you begin, ensure you have the following installed:

- **Operating System**: Ubuntu 20.04+ (or compatible Linux distribution)
- **Hardware**: Minimum 4GB RAM, 2 CPU cores
- **Software**:
  - Docker 20.10+
  - kubectl 1.20+
  - minikube 1.25+ (for local development)
  - Docker desktop
- **Permissions**: sudo access for installation

### System Requirements
```bash
# Check your system
lscpu | grep "CPU(s)"          # Should show 2+ cores
free -h | grep "Mem"           # Should show 4GB+ RAM
docker --version               # Should be 20.10+
kubectl version --client       # Should be 1.20+
```

## 🚀 Quick Start

Get up and running in under 5 minutes:

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/kubernetes-configmap-webserver.git
cd kubernetes-configmap-webserver

# 2. Start minikube
minikube start --driver=docker

# 3. Deploy the application
kubectl apply -f k8s/

# 4. Access the application
minikube service nginx-configmap-service --url
```


```
```

## 🔧 Deployment Guide

### Step 1: Environment Setup

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker (if not already installed)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Step 2: Start Kubernetes Cluster

```bash
# Start minikube with recommended settings
minikube start --driver=docker --memory=4096 --cpus=2

# Verify cluster is running
kubectl cluster-info
kubectl get nodes
```

### Step 3: Deploy the Application

```bash
# Apply all Kubernetes manifests
kubectl apply -f . (IT will run all YAML files in that directory)

# Alternative: Deploy step by step
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Step 4: Verify Deployment

```bash
# Check all resources
kubectl get all -l app=nginx-configmap

# Check ConfigMap
kubectl describe configmap nginx-html-config

# Check deployment status
kubectl rollout status deployment/nginx-configmap-deployment
```

### Step 5: Access the Application

```bash
# Method 1: Using minikube service
minikube service nginx-configmap-service --url

# Method 2: Port forwarding
kubectl port-forward service/nginx-configmap-service 8080:80

# Method 3: NodePort (if using external cluster)
kubectl get service nginx-configmap-service
```

## ⚙️ Configuration

### ConfigMap Structure

The HTML content is stored in a Kubernetes ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-html-config
  labels:
    app: nginx-configmap
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Custom Nginx Page</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <h1>Hello from ConfigMap!</h1>
        <p>This HTML is served from a Kubernetes ConfigMap</p>
        <p>Pod: <span id="hostname"></span></p>
        <script>
            document.getElementById('hostname').textContent = window.location.hostname;
        </script>
    </body>
    </html>
```

### Deployment Configuration

Key deployment features:
- **Replicas**: 3 pods for high availability
- **Resource Limits**: CPU (500m) and Memory (128Mi)
- **Health Checks**: Readiness and liveness probes
- **Volume Mounting**: ConfigMap mounted as volume

### Service Configuration

- **Type**: NodePort (for local development)
- **Port**: 80 (container) → 30080 (node)
- **Selector**: Routes traffic to nginx-configmap pods

## 🖥️ Browser Output

When you access the deployed application, you'll see:

![Browser Output](docs/images/browser-output.png)

### Expected Output Details:
- **Page Title**: "Custom Nginx Page"
- **Main Heading**: "Hello from ConfigMap!"
- **Content**: "This HTML is served from a Kubernetes ConfigMap"
- **Styling**: Basic HTML rendering with default browser fonts
- **Dynamic Content**: Shows the current pod hostname

### Testing the Application

```bash
# Test HTTP response
curl $(minikube service nginx-configmap-service --url)

# Test with headers
curl -I $(minikube service nginx-configmap-service --url)

# Load testing (optional)
ab -n 1000 -c 10 $(minikube service nginx-configmap-service --url)/
```

## 📈 Scaling & Management

### Horizontal Scaling

```bash
# Scale up to 5 replicas
kubectl scale deployment nginx-configmap-deployment --replicas=5

# Scale down to 2 replicas
kubectl scale deployment nginx-configmap-deployment --replicas=2

# Auto-scaling (requires metrics-server)
kubectl autoscale deployment nginx-configmap-deployment --cpu-percent=50 --min=2 --max=10
```

### Rolling Updates

```bash
# Update ConfigMap content
kubectl edit configmap nginx-html-config

# Trigger rolling update
kubectl rollout restart deployment nginx-configmap-deployment

# Monitor rollout
kubectl rollout status deployment nginx-configmap-deployment

# Rollback if needed
kubectl rollout undo deployment nginx-configmap-deployment
```

### Resource Monitoring

```bash
# View resource usage
kubectl top pods -l app=nginx-configmap
kubectl top nodes

# Describe resources
kubectl describe deployment nginx-configmap-deployment
kubectl describe service nginx-configmap-service
```

## 📊 Monitoring

### Basic Monitoring Commands

```bash
# Watch pods in real-time
kubectl get pods -l app=nginx-configmap -w

# View logs
kubectl logs -l app=nginx-configmap --tail=100

# Follow logs in real-time
kubectl logs -l app=nginx-configmap -f

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### Health Checks

The deployment includes:
- **Readiness Probe**: HTTP GET on port 80
- **Liveness Probe**: HTTP GET on port 80
- **Startup Probe**: Handles slow-starting containers

## 🐛 Troubleshooting

### Common Issues

#### Pods Not Starting
```bash
# Check pod status
kubectl describe pods -l app=nginx-configmap

# Check events
kubectl get events --field-selector involvedObject.kind=Pod

# Check logs
kubectl logs <pod-name>
```

#### Service Not Accessible
```bash
# Verify service endpoints
kubectl get endpoints nginx-configmap-service

# Check service configuration
kubectl describe service nginx-configmap-service

# Test internal connectivity
kubectl run test-pod --image=busybox --rm -it -- wget -O- nginx-configmap-service
```

#### ConfigMap Issues
```bash
# Verify ConfigMap content
kubectl get configmap nginx-html-config -o yaml

# Check volume mounts
kubectl describe pod <pod-name>
```

### Debug Mode

```bash
# Enable debug logging
kubectl patch deployment nginx-configmap-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","env":[{"name":"DEBUG","value":"true"}]}]}}}}'

# Access pod shell
kubectl exec -it <pod-name> -- /bin/sh
```

For more detailed troubleshooting, see [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md).

## 🏭 Production Considerations

### Security Best Practices

- **Non-root User**: Run nginx as non-root user
- **Resource Limits**: Set appropriate CPU/memory limits
- **Network Policies**: Implement network segmentation
- **RBAC**: Use Role-Based Access Control
- **Secrets**: Store sensitive data in Secrets, not ConfigMaps

### High Availability

- **Multiple Replicas**: Run 3+ replicas across different nodes
- **Pod Disruption Budgets**: Ensure minimum availability during updates
- **Node Affinity**: Distribute pods across availability zones
- **Health Checks**: Implement comprehensive health monitoring

### Performance Optimization

- **Resource Requests**: Set appropriate resource requests
- **Horizontal Pod Autoscaler**: Enable auto-scaling based on metrics
- **Caching**: Implement appropriate caching strategies
- **CDN Integration**: Use CDN for static content delivery

For comprehensive production guidelines, see [PRODUCTION.md](docs/PRODUCTION.md).

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

### Development Setup

```bash
# Clone your fork
git clone https://github.com/yourusername/kubernetes-configmap-webserver.git
cd kubernetes-configmap-webserver

# Create development environment
minikube start --profile=dev
kubectl config use-context dev

# Make your changes and test
kubectl apply -f k8s/
```

### Code Style

- Use descriptive commit messages
- Follow Kubernetes YAML best practices
- Include documentation for new features
- Add tests where appropriate



## 🙏 Acknowledgments

- **Kubernetes Community** for excellent documentation
- **Docker** for containerization platform
- **Nginx** for the web server
- **Contributors** who helped improve this project





---

⭐ **Star this repository** if you found it helpful!


