# Phase 13 — Containerized Development: Docker & Kubernetes Deliverable

**Document Status:** Complete / Verified  
**Project:** SecureChain — Software Supply Chain Security Platform  
**Marks Weight:** 7 Marks  

---

## 1. Docker Application Containers

Both backend and frontend services have been containerized using hardened multi-stage Dockerfiles.

### Backend Hardened Dockerfile ([`backend/Dockerfile`](file:///c:/Users/addal/Documents/antigravity/focused-bell/backend/Dockerfile))
```dockerfile
# Multi-stage build for Python FastAPI Backend
FROM python:3.11-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Final production runtime image
FROM python:3.11-slim
WORKDIR /app

# Non-root user creation (Container Security Control #2)
RUN groupadd -g 10001 appgroup && \
    useradd -u 10001 -g appgroup -s /bin/sh appuser

COPY --from=builder /install /usr/local
COPY --chown=appuser:appgroup . /app

USER appuser
EXPOSE 8001
CMD ["python", "main.py"]
```

---

## 2. Four Container Security Practices Applied

| Practice # | Security Control | Description & Verification |
|---|---|---|
| **Practice 1** | **Minimal Base Image** | Uses `python:3.11-slim` and `node:20-alpine` instead of full distribution images, stripping package managers, compilers, and redundant binaries to minimize attack surface. |
| **Practice 2** | **Non-Root Execution** | Explicitly creates non-root application users (`appuser` with UID `10001` / `10002`), mitigating root privilege escalation container breakouts. |
| **Practice 3** | **Controlled Port Exposure** | Exposes strictly required container ports (`8001` for API, `5173` for Web UI), blocking default access to debugging or secondary ports. |
| **Practice 4** | **Decoupled Secret Handling** | Excludes passwords, JWT secret keys, and database connection strings from image layers via `.dockerignore` and environment injection. |

---

## 3. Kubernetes Deployment & Security Manifests

All manifests are configured under [`k8s/`](file:///c:/Users/addal/Documents/antigravity/focused-bell/k8s/):

1. **Namespace Isolation** ([`k8s/namespace.yaml`](file:///c:/Users/addal/Documents/antigravity/focused-bell/k8s/namespace.yaml)): Isolated `securechain-prod` namespace.
2. **Kubernetes Secret** ([`k8s/secrets.yaml`](file:///c:/Users/addal/Documents/antigravity/focused-bell/k8s/secrets.yaml)): Encrypted key-value store for system secrets.
3. **Backend Deployment** ([`k8s/backend-deployment.yaml`](file:///c:/Users/addal/Documents/antigravity/focused-bell/k8s/backend-deployment.yaml)): Multi-replica deployment with security context.
4. **Backend ClusterIP Service** ([`k8s/backend-service.yaml`](file:///c:/Users/addal/Documents/antigravity/focused-bell/k8s/backend-service.yaml)): Restricted internal network exposure.
5. **Frontend Deployment** ([`k8s/frontend-deployment.yaml`](file:///c:/Users/addal/Documents/antigravity/focused-bell/k8s/frontend-deployment.yaml)): Web application deployment.
6. **Frontend Service** ([`k8s/frontend-service.yaml`](file:///c:/Users/addal/Documents/antigravity/focused-bell/k8s/frontend-service.yaml)): NodePort service for external access.

---

## 4. Two Key Kubernetes Security Controls Demonstrated

### Control 1: Strict Security Context (`runAsNonRoot` & `readOnlyRootFilesystem`)
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
    fsGroup: 10001
  containers:
  - name: backend
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```
* **Impact**: Ensures container processes cannot run with root privileges and blocks attackers from writing malicious persistent scripts to the container root filesystem.

### Control 2: Resource Limits & Namespace Isolation
```yaml
metadata:
  namespace: securechain-prod
spec:
  containers:
  - name: backend
    resources:
      limits:
        cpu: "500m"
        memory: "512Mi"
      requests:
        cpu: "100m"
        memory: "128Mi"
```
* **Impact**: Enforces computational boundary ceilings, preventing noisy-neighbor container resource starvation or Denial of Service (DoS) CPU-exhaustion attacks.

---

## 5. Minikube / Kubernetes Deployment Execution Instructions

```bash
# 1. Apply Namespace & Secrets
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secrets.yaml

# 2. Deploy Backend & Frontend Services
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml

# 3. Verify Pod & Security Context Status
kubectl get pods -n securechain-prod
kubectl describe pod -l app=securechain-backend -n securechain-prod
```
