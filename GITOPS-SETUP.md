# GitOps Setup Guide

สำหรับการ setup ArgoCD GitOps workflow กับ GitHub

## 🚀 Quick Start

### 1. สร้าง GitHub Repository

```bash
cd payment-service-gitops

# สร้าง repo ด้วย GitHub CLI (ถ้ามี)
gh repo create payment-service-gitops --public --source=. --remote=origin --push

# หรือสร้างแบบ manual:
# 1. ไปที่ https://github.com/new
# 2. ชื่อ repo: payment-service-gitops
# 3. เลือก Public
# 4. อย่าเลือก Initialize this repository with
# 5. คลิก Create repository
```

### 2. เพิ่ม Remote และ Push

ถ้าสร้างแบบ manual:

```bash
cd payment-service-gitops

# แทนที่ YOUR_GITHUB_USERNAME ด้วย username จริง
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/payment-service-gitops.git
git branch -M main
git push -u origin main
```

### 3. อัพเดท GitHub Username ในไฟล์

ต้องแก้ไข `YOUR_GITHUB_USERNAME` ให้เป็น username จริงของคุณ:

**ไฟล์ที่ต้องแก้:**

1. `argocd/application.yaml` (line 19):
```yaml
repoURL: https://github.com/YOUR_GITHUB_USERNAME/payment-service-gitops.git
```

2. `backstage/catalog-info.yaml` (line 11):
```yaml
github.com/project-slug: YOUR_GITHUB_USERNAME/payment-service-gitops
```

3. `backstage/catalog-info.yaml` (line 23):
```yaml
url: https://github.com/YOUR_GITHUB_USERNAME/payment-service-gitops
```

4. `k8s/deployment.yaml` (line 16):
```yaml
backstage.io/managed-by-location: url:https://github.com/YOUR_GITHUB_USERNAME/payment-service-gitops
```

**วิธีแก้ไขทั้งหมดพร้อมกัน:**

```bash
# แทนที่ YOUR_GITHUB_USERNAME ทั้งหมดในครั้งเดียว
# แทน 'yourusername' ด้วย GitHub username จริง
find . -type f \( -name "*.yaml" -o -name "*.yml" \) -exec sed -i '' 's/YOUR_GITHUB_USERNAME/yourusername/g' {} +

# ตรวจสอบว่าแทนที่แล้ว
grep -r "yourusername" . --include="*.yaml"
```

### 4. Commit และ Push การเปลี่ยนแปลง

```bash
git add .
git commit -m "Update GitHub username"
git push origin main
```

### 5. Deploy ด้วย ArgoCD

```bash
# กลับไปที่ root directory
cd ..

# Deploy application
./argocd-manager.sh deploy

# หรือใช้ portal manager
./portal-manager.sh deploy
```

### 6. ตรวจสอบการ Deploy

```bash
# Check ArgoCD application status
./argocd-manager.sh status

# Watch pods
kubectl get pods -n payment-service -w

# Check ArgoCD UI
# http://localhost:8181
# Username: admin
# Password: ./argocd-manager.sh password
```

## 🔄 GitOps Workflow

### การอัพเดท Application

1. แก้ไขไฟล์ใน `k8s/` directory (เช่น เปลี่ยน replicas):

```yaml
# k8s/deployment.yaml
spec:
  replicas: 3  # เปลี่ยนจาก 2 เป็น 3
```

2. Commit และ Push:

```bash
git add k8s/deployment.yaml
git commit -m "Scale payment-service to 3 replicas"
git push origin main
```

3. ArgoCD จะ auto-sync ภายใน 3 นาที หรือ force sync:

```bash
./argocd-manager.sh sync
```

4. ตรวจสอบผลลัพธ์:

```bash
kubectl get pods -n payment-service
# ควรเห็น 3 pods
```

## 🎯 ทดสอบ GitOps Workflow

### Test 1: เปลี่ยน Image Tag

```bash
cd payment-service-gitops

# แก้ไข k8s/deployment.yaml
sed -i '' 's/image: nginx:alpine/image: nginx:1.25-alpine/' k8s/deployment.yaml

git add k8s/deployment.yaml
git commit -m "Update nginx to version 1.25"
git push origin main

# รอ ArgoCD sync (หรือ force)
./argocd-manager.sh sync

# ตรวจสอบ
kubectl describe pod -n payment-service | grep Image:
```

### Test 2: เพิ่ม Environment Variable

```bash
# แก้ไข k8s/deployment.yaml เพิ่ม env
# env:
# - name: NEW_VAR
#   value: "test-value"

git add k8s/deployment.yaml
git commit -m "Add new environment variable"
git push origin main

./argocd-manager.sh sync

kubectl get pod -n payment-service -o yaml | grep -A5 "env:"
```

### Test 3: Scale Up/Down

```bash
# แก้ไข replicas
sed -i '' 's/replicas: 2/replicas: 5/' k8s/deployment.yaml

git add k8s/deployment.yaml
git commit -m "Scale to 5 replicas"
git push origin main

# Watch rolling update
kubectl get pods -n payment-service -w
```

## 🐛 Troubleshooting

### ArgoCD ไม่ sync

```bash
# Check application
kubectl get application payment-service -n argocd -o yaml

# Check sync status
kubectl get application payment-service -n argocd -o jsonpath='{.status.sync.status}'

# Force refresh
kubectl patch application payment-service -n argocd -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}' --type merge

# Force sync
./argocd-manager.sh sync
```

### GitHub Authentication Error

ถ้าใช้ private repo ต้อง config GitHub token:

```bash
# Create personal access token at https://github.com/settings/tokens
# Scope: repo (full control)

kubectl create secret generic github-secret \
  -n argocd \
  --from-literal=username=YOUR_USERNAME \
  --from-literal=password=YOUR_TOKEN

# Update application.yaml to use secret
```

### Pods ไม่ Running

```bash
# Check events
kubectl get events -n payment-service --sort-by='.lastTimestamp'

# Check pod details
kubectl describe pod -n payment-service -l app=payment-service

# Check logs
kubectl logs -n payment-service -l app=payment-service
```

## 📊 Monitoring

### Watch ArgoCD Sync

```bash
# Terminal 1: Watch application
kubectl get application payment-service -n argocd -w

# Terminal 2: Watch pods
kubectl get pods -n payment-service -w

# Terminal 3: Watch events
kubectl get events -n payment-service -w
```

### ArgoCD UI

1. Start UI:
```bash
./argocd-manager.sh start
```

2. Open browser: http://localhost:8181

3. Login:
   - Username: `admin`
   - Password: `./argocd-manager.sh password`

4. Navigate to Applications > payment-service

5. View:
   - Sync status
   - Resource tree
   - Last sync info
   - Diff view

## 🔐 Security Best Practices

### ใช้ Private Repository

1. สร้าง private repo แทน public
2. เพิ่ม GitHub token ใน ArgoCD
3. Update application.yaml

### Secrets Management

อย่าเก็บ secrets ใน Git! ใช้:

- Sealed Secrets
- External Secrets Operator
- Vault
- SOPS

Example with Sealed Secrets:

```bash
# Install sealed-secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Create sealed secret
echo -n mypassword | kubectl create secret generic mysecret --dry-run=client --from-file=password=/dev/stdin -o yaml | \
  kubeseal -o yaml > mysealedsecret.yaml

# Commit sealed secret (safe to commit)
git add mysealedsecret.yaml
git commit -m "Add sealed secret"
git push
```

## 🎓 Next Steps

1. ✅ Setup GitHub repo
2. ✅ Deploy with ArgoCD
3. ✅ Test GitOps workflow
4. ⬜ Add Backstage catalog integration
5. ⬜ Configure OpenChoreo discovery
6. ⬜ Setup CI/CD pipeline
7. ⬜ Add monitoring/alerts

## 📚 Resources

- ArgoCD Docs: https://argo-cd.readthedocs.io/
- GitOps Guide: https://www.gitops.tech/
- Backstage Kubernetes Plugin: https://backstage.io/docs/features/kubernetes/
- OpenChoreo Docs: https://openchoreo.io/docs/
