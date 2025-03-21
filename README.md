# 🚀 Seminar GitOps

**GitOps repository for managing the deployment of the Seminar Todo App using ArgoCD and Helm.**  
This repo contains the Helm chart configurations used by ArgoCD to automatically deploy and sync the Next.js application.

---

## 📌 Overview

This repository follows the **GitOps model**, where:
- All application configurations are **stored in Git**
- ArgoCD continuously syncs the state in Kubernetes to match the state defined in this repo
- Any updates to the application are deployed **automatically when changes are pushed**

---

## 🛠️ How It Works

1. Developers push changes to [`seminar-code`](https://github.com/mrfour4/seminar-code)
2. GitHub Actions updates the image tag in `seminar-gitops`
3. ArgoCD detects the new version and deploys the latest image automatically

---

## 📂 Repository Structure

```
seminar-gitops/
│── nextjs/
│   ├── Chart.yaml
│   ├── templates/
│   │   ├── deployment.yaml
│   │   ├── ingress.yaml
│   │   ├── sealed-secret.yaml
│   │   └── service.yaml
│   └── values.yaml
```

---

## 🔄 Deployment Workflow

### ✅ **1. Update Image in `seminar-code`**
- Developers push a change to `main` in the [`seminar-code`](https://github.com/mrfour4/seminar-code) repo.
- GitHub Actions builds a new Docker image and pushes it to Docker Hub.

### ✅ **2. Update `seminar-gitops` with New Image Tag**
- The GitHub Actions workflow updates `nextjs/values.yaml` with the new image tag.
- A commit is pushed to the `seminar-gitops` repository.

### ✅ **3. ArgoCD Auto-syncs the Changes**
- ArgoCD detects the new commit and updates the application in Kubernetes.
- The latest version of the Next.js app is deployed automatically.

---

## 🚀 How to Manually Trigger Sync in ArgoCD

If auto-sync is disabled, you can manually sync the application:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Then, open `https://localhost:8080`, log in, and sync the `seminar-nextjs` application.

Alternatively, use the CLI:
```bash
argocd app sync seminar-nextjs
```

---

## 🔐 Managing Secrets with Sealed Secrets

To securely manage environment variables and secrets:

### 1. Create a plain secret file
```bash
cat <<EOF > secret-plain.yaml
apiVersion: v1
kind: Secret
metadata:
  name: seminar-secret
  namespace: default
type: Opaque
stringData:
  DATABASE_URL: "your-database-url"
EOF
```

### 2. Encrypt the secret using `kubeseal`
```bash
kubeseal \
  --controller-namespace kube-system \
  --controller-name sealed-secrets-controller \
  -o yaml < secret-plain.yaml > nextjs/templates/sealed-secret.yaml
```

### 3. Securely delete the plain secret
```bash
shred -u secret-plain.yaml
```

### 4. Commit and push the sealed secret
```bash
git add nextjs/templates/sealed-secret.yaml
git commit -m "Add sealed secret"
git push
```

---

## 🏗️ Minikube Setup & Application Deployment

1. **Check Minikube status:**
   ```bash
   minikube status
   ```
2. **Start Minikube if it's not running:**
   ```bash
   minikube start
   ```
3. **Enable Ingress for Minikube:**
   ```bash
   minikube addons enable ingress
   ```
4. **Deploy the application using Helm:**
   ```bash
   helm upgrade --install seminar-nextjs nextjs/ -f nextjs/values.yaml
   ```
5. **Check running services:**
   ```bash
   kubectl get services
   ```
6. **Access the Next.js service in Minikube:**
   ```bash
   minikube service nextjs-service
   ```
7. **Verify application logs:**
   ```bash
   kubectl logs -l app=nextjs -f
   ```

---

## 📜 Useful Commands

- **List all applications in ArgoCD:**
  ```bash
  argocd app list
  ```
- **Check logs of the Next.js app:**
  ```bash
  kubectl logs -l app=nextjs -f
  ```
- **Describe the running pods:**
  ```bash
  kubectl get pods -o wide
  ```

---

## 💡 Notes

- Make sure ArgoCD is correctly set up and connected to this repo.
- Always validate `values.yaml` before pushing changes to avoid breaking deployments.

> **Built with ❤️ by [@mrfour4](https://github.com/mrfour4)**

