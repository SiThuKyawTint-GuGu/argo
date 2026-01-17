# Fullstack Application on Kubernetes

A production-ready fullstack application deployed on Kubernetes using Helm charts and managed by ArgoCD for GitOps continuous deployment.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Components](#components)
- [Installation](#installation)
- [Configuration](#configuration)
- [Deployment with ArgoCD](#deployment-with-argocd)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🎯 Overview

This project provides a complete fullstack application stack deployed on Kubernetes, including:

- **Frontend**: React-based web application
- **Backend**: Node.js API server
- **Database**: MySQL database with persistent storage
- **Monitoring**: Prometheus and Grafana for metrics and visualization
- **CI/CD**: ArgoCD for GitOps-based continuous deployment

All components are packaged as Helm charts and can be deployed independently or as a complete stack.

## 🏗️ Architecture

```
┌─────────────────┐
│   Frontend      │ (Port 80)
│   (React)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Backend API   │ (Port 3000)
│   (Node.js)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   MySQL DB      │ (Port 3306)
│   (Database)    │
└─────────────────┘

┌─────────────────┐     ┌─────────────────┐
│   Prometheus    │────▶│    Grafana      │
│   (Metrics)     │     │  (Visualization) │
└─────────────────┘     └─────────────────┘
```

### Namespaces

- `frontend`: Frontend application
- `backend`: Backend API service
- `database`: MySQL database
- `monitoring`: Prometheus and Grafana
- `argocd`: ArgoCD applications (management)

## 📦 Prerequisites

Before deploying this application, ensure you have:

1. **Kubernetes Cluster** (v1.30+)
   - Access to a running Kubernetes cluster
   - `kubectl` configured to connect to your cluster

2. **Helm** (v3.0+)
   ```bash
   # Install Helm
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   ```

3. **ArgoCD** (Optional but recommended)
   ```bash
   # Install ArgoCD
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

4. **Git Repository Access**
   - Access to the repository: `https://github.com/SiThuKyawTint-GuGu/argo.git`

## 📁 Project Structure

```
fullstack-app/
├── argocd/                    # ArgoCD Application definitions
│   ├── backend-app.yaml
│   ├── frontend-app.yaml
│   ├── monitoring-app.yaml
│   └── mysql-app.yaml
├── backend/                   # Backend Helm chart
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── configmap.yaml
│       ├── secret.yaml
│       └── tests/
├── frontend/                  # Frontend Helm chart
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       └── tests/
├── mysql/                     # MySQL Helm chart
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── secret.yaml
│       └── tests/
├── monitoring/                # Monitoring Helm chart
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── prometheus-deployment.yaml
│       ├── prometheus-service.yaml
│       ├── grafana-deployment.yaml
│       ├── grafana-service.yaml
│       ├── prometheus-config.yaml
│       ├── ingress.yaml
│       └── tests/
└── README.md
```

## 🔧 Components

### Backend

- **Image**: `guguskyler/backend:v2`
- **Port**: 3000
- **Service Type**: LoadBalancer
- **Replicas**: 2
- **Health Checks**: 
  - Liveness: `/health/live`
  - Readiness: `/health/ready`
- **Database Connection**: MySQL at `mysql.database.svc.cluster.local:3306`

### Frontend

- **Image**: `guguskyler/frontend:v3`
- **Port**: 80
- **Service Type**: LoadBalancer
- **Replicas**: 1

### MySQL

- **Image**: `mysql:8`
- **Port**: 3306
- **Service Type**: ClusterIP
- **Database**: `app_db`
- **Persistence**: 5Gi storage
- **Namespace**: `database`

### Monitoring

- **Prometheus**:
  - Port: 9090
  - Scrapes backend metrics every 15 seconds
  - Target: `backend-service:3000`

- **Grafana**:
  - Port: 3000
  - Default credentials: `admin/admin`
  - Pre-configured dashboards for application metrics

## 🚀 Installation

### Option 1: Manual Helm Installation

#### 1. Deploy MySQL (First)

```bash
cd mysql
helm install mysql . --namespace database --create-namespace
```

#### 2. Deploy Backend

```bash
cd ../backend
helm install backend . --namespace backend --create-namespace
```

#### 3. Deploy Frontend

```bash
cd ../frontend
helm install frontend . --namespace frontend --create-namespace
```

#### 4. Deploy Monitoring

```bash
cd ../monitoring
helm install monitoring . --namespace monitoring --create-namespace
```

### Option 2: ArgoCD Installation (Recommended)

#### 1. Install ArgoCD Applications

```bash
kubectl apply -f argocd/mysql-app.yaml
kubectl apply -f argocd/backend-app.yaml
kubectl apply -f argocd/frontend-app.yaml
kubectl apply -f argocd/monitoring-app.yaml
```

#### 2. Verify Applications

```bash
# Check ArgoCD applications
kubectl get applications -n argocd

# Check application status
argocd app list
```

ArgoCD will automatically:
- Sync applications from the Git repository
- Create namespaces if they don't exist
- Deploy and maintain desired state
- Self-heal if resources are modified outside of Git

## ⚙️ Configuration

### Backend Configuration

Edit `backend/values.yaml`:

```yaml
replicaCount: 2
image:
  repository: guguskyler/backend
  tag: "v2"
service:
  type: LoadBalancer
  port: 3000
configmap:
  enabled: true
  data:
    DB_HOST: mysql.database.svc.cluster.local
    DB_USER: root
    DB_NAME: app_db
```

### Frontend Configuration

Edit `frontend/values.yaml`:

```yaml
replicaCount: 1
image:
  repository: guguskyler/frontend
  tag: "v3"
service:
  type: LoadBalancer
  port: 80
```

### MySQL Configuration

Edit `mysql/values.yaml`:

```yaml
mysqlDatabase: app_db
persistence:
  size: 5Gi
secret:
  name: mysql-secret
```

### Monitoring Configuration

Edit `monitoring/values.yaml`:

```yaml
prometheus:
  enabled: true
  service:
    port: 9090
  config:
    scrape_configs:
      - job_name: "backend"
        static_configs:
          - targets:
              - backend-service:3000

grafana:
  enabled: true
  service:
    port: 3000
  adminUser: admin
  adminPassword: admin
```

## 🔄 Deployment with ArgoCD

### Sync Policy

All ArgoCD applications are configured with:

- **Automated Sync**: Applications sync automatically when changes are detected
- **Self-Heal**: Automatically corrects drift from desired state
- **Prune**: Removes resources that are no longer in Git
- **Auto-Create Namespace**: Creates namespaces if they don't exist

### Manual Sync

```bash
# Sync a specific application
argocd app sync backend

# Sync all applications
argocd app sync --all
```

### Update Application

1. Make changes to Helm chart values or templates
2. Commit and push to Git repository
3. ArgoCD automatically detects changes and syncs (if auto-sync enabled)
4. Or manually trigger sync via ArgoCD UI or CLI

## 📊 Monitoring

### Access Prometheus

```bash
# Port forward to Prometheus
kubectl port-forward -n monitoring svc/prometheus 9090:9090

# Access at http://localhost:9090
```

### Access Grafana

```bash
# Port forward to Grafana
kubectl port-forward -n monitoring svc/grafana 3000:3000

# Access at http://localhost:3000
# Default credentials: admin/admin
```

### Ingress Access (if enabled)

If ingress is enabled in `monitoring/values.yaml`:

- Prometheus: `http://<ingress-host>/prometheus`
- Grafana: `http://<ingress-host>/grafana`

### Metrics Collection

Prometheus is configured to scrape:
- Backend service metrics at `backend-service:3000`
- Scrape interval: 15 seconds

## 🔍 Troubleshooting

### Common Issues

#### 1. ArgoCD ComparisonError - Nil Pointer in Test Template

**Error:**
```
Error: template: monitoring/templates/tests/test-connection.yaml:14:62: 
executing "monitoring/templates/tests/test-connection.yaml" at <.Values.service.port>: 
nil pointer evaluating interface {}.port
```

**Solution:**
The test-connection.yaml file has been fixed to use `.Values.prometheus.service.port` instead of `.Values.service.port`. Ensure you have the latest version of the monitoring chart.

#### 2. Backend Cannot Connect to MySQL

**Check:**
```bash
# Verify MySQL is running
kubectl get pods -n database

# Check MySQL service
kubectl get svc -n database

# Verify backend configmap
kubectl get configmap -n backend
kubectl describe configmap <configmap-name> -n backend
```

**Fix:**
- Ensure MySQL is deployed in the `database` namespace
- Verify DB_HOST in backend configmap matches: `mysql.database.svc.cluster.local`
- Check MySQL secret exists and is properly referenced

#### 3. Pods Not Starting

```bash
# Check pod status
kubectl get pods -A

# Check pod logs
kubectl logs <pod-name> -n <namespace>

# Describe pod for events
kubectl describe pod <pod-name> -n <namespace>
```

#### 4. Image Pull Errors

```bash
# Check if image exists and is accessible
docker pull guguskyler/backend:v2
docker pull guguskyler/frontend:v3

# Verify imagePullPolicy in values.yaml
# Use "Always" for development, "IfNotPresent" for production
```

#### 5. ArgoCD Application Out of Sync

```bash
# Check application status
argocd app get <app-name>

# Force refresh
argocd app get <app-name> --refresh

# Manual sync
argocd app sync <app-name>
```

### Debug Commands

```bash
# List all resources in a namespace
kubectl get all -n <namespace>

# Check Helm releases
helm list -A

# Validate Helm chart
helm template . --debug

# Dry-run installation
helm install <release-name> . --dry-run --debug
```

## 🔐 Security Considerations

1. **Secrets Management**: 
   - MySQL credentials are stored in Kubernetes secrets
   - Consider using external secret management (e.g., Sealed Secrets, Vault)

2. **Image Security**:
   - Use private image registries in production
   - Scan images for vulnerabilities
   - Use specific image tags, avoid `latest`

3. **Network Policies**:
   - Implement network policies to restrict pod-to-pod communication
   - Use service mesh for advanced security (e.g., Istio)

4. **RBAC**:
   - Configure proper RBAC for service accounts
   - Limit ArgoCD permissions to necessary resources

5. **Grafana Credentials**:
   - Change default admin password in production
   - Use external authentication (OAuth, LDAP)

## 📝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with Helm
5. Submit a pull request

### Development Workflow

```bash
# Test chart locally
helm template . --debug

# Install in test namespace
helm install <release-name> . --namespace <test-namespace> --create-namespace

# Upgrade
helm upgrade <release-name> . --namespace <test-namespace>

# Uninstall
helm uninstall <release-name> --namespace <test-namespace>
```

## 📚 Additional Resources

- [Helm Documentation](https://helm.sh/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)

## 📄 License

This project is licensed under the MIT License.

## 👥 Support

For issues and questions:
- Open an issue in the repository
- Check existing issues and pull requests
- Review troubleshooting section above

---

**Note**: This is a production-ready template. Adjust configurations, resource limits, and security settings according to your specific requirements and environment.
