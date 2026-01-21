# Payment Service - GitOps Repository

This repository contains Kubernetes manifests for the Payment Service, managed by ArgoCD in a GitOps workflow.

## 📁 Repository Structure

```
payment-service-gitops/
├── k8s/                    # Kubernetes manifests
│   ├── namespace.yaml      # Namespace definition
│   ├── deployment.yaml     # Deployment specification
│   └── service.yaml        # Service definition
├── backstage/              # Backstage catalog
│   └── catalog-info.yaml   # Component metadata
├── argocd/                 # ArgoCD configuration
│   └── application.yaml    # ArgoCD Application manifest
└── README.md               # This file
```

## 🚀 Quick Start

### Prerequisites
- k3d cluster running (openchoreo-quick-start)
- ArgoCD installed in cluster
- kubectl configured

### Deploy with ArgoCD

1. **Create ArgoCD Application:**
```bash
kubectl apply -f argocd/application.yaml
```

2. **Verify Deployment:**
```bash
kubectl get pods -n payment-service
kubectl get svc -n payment-service
```

3. **Check ArgoCD Sync Status:**
```bash
kubectl get application payment-service -n argocd
```

### Manual Deployment (without ArgoCD)

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

## 🔄 GitOps Workflow

1. **Make Changes:**
   - Edit files in `k8s/` directory
   - Commit and push to GitHub

2. **Auto-Sync:**
   - ArgoCD detects changes automatically
   - Syncs cluster state with Git repository
   - Self-heals if manual changes occur

3. **Verify:**
   - Check ArgoCD dashboard
   - Verify pods are running
   - Check OpenChoreo for discovery

## 🏷️ Labels & Annotations

### OpenChoreo Labels
- `openchoreo.io/managed: "true"` - Marks resources for OpenChoreo discovery
- `openchoreo.io/environment: production` - Environment classification

### Backstage Labels
- `backstage.io/kubernetes-id: payment-service` - Links to Backstage component
- `app.kubernetes.io/name: payment-service` - Standard Kubernetes label
- `app.kubernetes.io/part-of: ecommerce-platform` - System grouping

### ArgoCD Labels
- `app.kubernetes.io/managed-by: argocd` - Managed by ArgoCD

## 🔗 Integration Points

### Backstage
- Component: `payment-service`
- System: `ecommerce-platform`
- Owner: `payment-team`

### OpenChoreo
- Auto-discovery via labels
- Environment: Production
- API Type: REST

### ArgoCD
- Application: `payment-service`
- Project: `default`
- Sync Policy: Automated with prune and self-heal

## 📊 Monitoring

### Check Pod Status
```bash
kubectl get pods -n payment-service -w
```

### View Logs
```bash
kubectl logs -n payment-service -l app=payment-service -f
```

### Check Service
```bash
kubectl get svc -n payment-service
kubectl describe svc payment-service -n payment-service
```

## 🔧 Troubleshooting

### ArgoCD Not Syncing
```bash
# Check application status
kubectl get application payment-service -n argocd -o yaml

# Force sync
kubectl patch application payment-service -n argocd -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"HEAD"}}}' --type merge
```

### Pods Not Starting
```bash
# Check events
kubectl get events -n payment-service --sort-by='.lastTimestamp'

# Check pod details
kubectl describe pod -n payment-service -l app=payment-service
```

### OpenChoreo Not Discovering
- Verify all labels are present
- Check OpenChoreo API logs
- Ensure OAuth is configured (if required)

## 📝 Notes

- Default image: `nginx:alpine` (replace with actual payment service image)
- Replicas: 2 (adjust based on load)
- Resources: 64Mi-128Mi memory, 100m-200m CPU (tune as needed)
- Health probes configured on port 80

## 🔐 Security Notes

- Update GitHub username in manifests before pushing
- Review and adjust resource limits
- Configure proper RBAC in production
- Use secrets for sensitive data (not hardcoded)
