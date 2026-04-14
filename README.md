
# 🧪 README — GitOps with Helm + Argo CD (Complete Setup)

---

# 🚀 Overview

This project demonstrates a **production-style GitOps workflow** using:

* Helm (application packaging)
* Argo CD (continuous reconciliation)

```text
Git (Helm Charts) = Source of Truth  
Kubernetes Cluster = Continuously enforced state
```

---

# 🎯 Objective

* Install Argo CD using Helm
* Deploy application using Helm chart
* Manage deployment via Argo CD
* Observe **self-healing & drift correction**

---

# ⚙️ Prerequisites

* Kubernetes cluster (Minikube / Kind / EKS)
* `kubectl`
* Helm installed
* GitHub repo

---

# 🧩 Step 1: Install Argo CD using Helm

---

## ➕ Add Helm Repo

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

---

## 📦 Install Argo CD

```bash
kubectl create namespace argocd
```

```bash
helm install argocd argo/argo-cd -n argocd
```

---

## 🔐 Access Argo CD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```text
https://localhost:8080
```

---

## 🔑 Get Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

---

## 🧪 Verify Installation

```bash
kubectl get pods -n argocd
```

👉 All pods should be `Running`

---

# 📦 Step 2: Create Helm App

---

## 🛠️ Create Chart

```bash
helm create helm-app
```

---

## 📁 Structure

```text
repo/
 └── helm-app/
      ├── Chart.yaml
      ├── values.yaml
      └── templates/
           ├── deployment.yaml
           └── service.yaml
```

---

## ✏️ values.yaml

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest

service:
  type: ClusterIP
  port: 80
```

---

## 🧾 deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

---

## 🧾 service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  type: {{ .Values.service.type }}
```

---

## 📤 Push to GitHub

```bash
git add .
git commit -m "helm app setup"
git push origin main
```

---

# 🔗 Step 3: Create Argo CD Application (Helm)

---

## 📄 application.yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-helm-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: ##repo-url
    targetRevision: main
    path: helm-app
    helm:
      valueFiles:
        - values.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## ▶️ Apply

```bash
kubectl apply -f application.yaml
```

---

# ✅ Expected Result

Argo CD will:

* Pull Helm chart from Git
* Render templates
* Deploy resources
* Continuously enforce desired state

---

# 💀 Step 4: Break Things (Mandatory Learning)

---

## 💣 Scale manually

```bash
kubectl scale deployment nginx --replicas=1
```

👉 Argo CD restores to 2

---

## 💣 Change image

```bash
kubectl edit deployment nginx
```

👉 Argo CD reverts it

---

## 💣 Delete deployment

```bash
kubectl delete deployment nginx
```

👉 Argo CD recreates it

---

# 🔍 Observability

---

## Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Argo CD UI

Watch:

* Sync Status
* Health Status

---

# 🧠 Architecture Flow

```text
Git → Argo CD → Helm → Kubernetes API → Controllers → Pods
```

---

# ⚔️ Key Concepts

---

## 🔁 Reconciliation Layers

* Argo CD → Git state
* Helm → Template rendering
* Deployment → Pod lifecycle

---

## 🧩 Responsibility Split

| Component       | Responsibility      |
| --------------- | ------------------- |
| Argo CD         | Enforces Git        |
| Helm            | Generates manifests |
| K8s Controllers | Maintain runtime    |

---

# 💀 Common Mistakes

---

### ❌ Using HEAD instead of main

```yaml
targetRevision: main
```

---

### ❌ Wrong repo path

---

### ❌ Broken Helm templates

---

### ❌ Editing cluster manually

👉 Argo CD will undo it

---

# ⚡ Final Mental Model

```text
Git (Helm) → Argo CD → Render → Apply → Reconcile → Repeat forever
```

---

# 🔥 Key Insight

> Helm defines structure
> Argo CD enforces truth

---

# 💀 Brutal Truth

Without GitOps:

```text
You deploy apps
```

With GitOps:

```text
System deploys itself
```

---
