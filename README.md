
# 🚀 GitOps Workflow with ArgoCD & Kubernetes  

This project demonstrates a **GitOps pipeline** using **ArgoCD** on Kubernetes (K3s/Minikube).  
Application deployments are synced automatically from a **GitHub repository → Kubernetes cluster**, ensuring the cluster state always matches the Git repository.  

---

## 🔹 Workflow Overview
1. **Deploy ArgoCD** on Kubernetes  
2. **Store manifests** (`deployment.yaml`, `service.yaml`) in a GitHub repo  
3. **Configure ArgoCD** to track the repo and auto-sync changes  
4. **Update manifests in Git** → ArgoCD auto-applies to cluster  
5. ✅ Git = **single source of truth** for deployments  

---

## 📦 Tools Used
- [K3s](https://k3s.io/) / [Minikube](https://minikube.sigs.k8s.io/) → Lightweight Kubernetes  
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) → GitOps CD tool  
- [Docker Hub](https://hub.docker.com/) → Application images  
- [GitHub](https://github.com/) → Version control  

---

## 📂 Repository Structure
```
.
├── deployment.yaml # NGINX Deployment (2 replicas)
├── service.yaml # Service (NodePort 30080)
└── README.md # Project documentation
```
---
## 👨‍💻GitOps Flow
```
   👨‍💻 Developer
         │
         │ Commit & Push
         ▼
   📂 GitHub Repo (Manifests)
         │
         │ Monitored by
         ▼
   🚀 ArgoCD (GitOps Controller)
         │
         │ Syncs Desired State
         ▼
   ☸️ Kubernetes Cluster
         │
         │ Runs
         ▼
   🌐 MyApp (NGINX Service)
```
---

## ⚙️ Kubernetes Manifests

### 🔹 Deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:latest
        ports:
        - containerPort: 80
```
### 🔹 Service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```
## 🚀 How to Deploy

✅ Recommended EC2 Instance Sizes
- t3.medium (2 vCPU, 4 GB RAM) → Best balance for ArgoCD + app testing.
- If you’re just learning/testing GitOps → t3.medium (or t3a.medium for cheaper AMD option).

### 1️⃣ Start Kubernetes K3s (lightweight, good for testing)
```
curl -sfL https://get.k3s.io | sh -
```

### 2️⃣ Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3️⃣ Access ArgoCD UI on EC2 (HTTP only)

#### Expose ArgoCD with NodePort

- Change the ArgoCD service type from ClusterIP → NodePort.
```
kubectl -n argocd edit svc argocd-server
```

- Find this part:
```
spec:
  type: ClusterIP
```

Change it to:
```
spec:
  type: NodePort
  ports:
  - name: https
    port: 443
    targetPort: 8080
    nodePort: 30080   # Choose any port between 30000–32767


```
Now you can open:
```
http://<EC2_PUBLIC_IP>:30080
```
- User: admin
- Get initial admin password:
```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

### 4️⃣ Connect GitHub Repo in ArgoCD

- Add new Application in ArgoCD UI
- Repo URL: Your GitHub repo
- Path: .
- Cluster: in-cluster
- Namespace: default
- Enable Auto-Sync ✅

### 5️⃣ Test Auto-Sync

- Update deployment.yaml (e.g., change image tag nginx:1.27.1)
- Commit & push → ArgoCD detects → Auto-deploys 🚀

## ArgoCD Dashboard:
<img width="1920" height="880" alt="Applications Tiles - Argo CD - Brave 07-09-2025 14_59_56" src="https://github.com/user-attachments/assets/0de5cff3-0394-401f-bb87-bb4a9b53a807" />

## myapp deployment: Syncing
<img width="1920" height="878" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_14_04" src="https://github.com/user-attachments/assets/62a3fd6b-dd05-4c83-a853-47ab2a418442" />


## myapp deployment: Synced
<img width="1920" height="878" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_23_11" src="https://github.com/user-attachments/assets/e7d45f98-57d4-4915-8b51-ecfc9bc990dd" />

## Application:
<img width="1920" height="882" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_37_32" src="https://github.com/user-attachments/assets/90675320-b35f-43c1-862a-0bd733682da6" />

