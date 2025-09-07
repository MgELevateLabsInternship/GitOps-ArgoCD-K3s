
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
## GitOps Flow
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

<img width="1920" height="1020" alt="Applications Tiles - Argo CD - Brave 07-09-2025 14_59_56" src="https://github.com/user-attachments/assets/b31bb1e4-3355-4c45-8211-1b28b0a2c95b" />
<img width="1920" height="1020" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_14_04" src="https://github.com/user-attachments/assets/7e360358-4860-454b-9cdb-aae4a3f1de75" />
<img width="1920" height="1020" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_14_21" src="https://github.com/user-attachments/assets/3db72ecd-8b50-4eae-9d44-fc1a67c2658e" />
<img width="1920" height="1020" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_14_31" src="https://github.com/user-attachments/assets/213c5da3-546f-4290-ad54-dc988ab36789" />
<img width="1920" height="1020" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_36_49" src="https://github.com/user-attachments/assets/751037ea-8051-4c90-ae94-35143605e814" />
<img width="1920" height="1020" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_37_12" src="https://github.com/user-attachments/assets/646cbc6d-bb9b-4f59-a314-058356f56d75" />
<img width="1920" height="1020" alt="Applications Tiles - Argo CD - Brave 07-09-2025 15_37_32" src="https://github.com/user-attachments/assets/139d08ca-c3de-4887-b076-1788a863e0fc" />


