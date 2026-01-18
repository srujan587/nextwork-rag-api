# Kubernetes Deployment Debugging Guide

## 1. Check Deployment Status
```powershell
kubectl get deployments
kubectl get deployment rag-app-deployment
kubectl describe deployment rag-app-deployment
```

## 2. Check Pod Status
```powershell
# List all pods
kubectl get pods

# Get pods with labels
kubectl get pods -l app=rag-api

# Describe specific pod (replace POD_NAME)
kubectl describe pod <POD_NAME>

# Watch pods in real-time
kubectl get pods -w
```

## 3. View Pod Logs
```powershell
# Get logs from a specific pod
kubectl logs <POD_NAME>

# Get logs from all pods with label
kubectl logs -l app=rag-api

# Follow logs (like tail -f)
kubectl logs -f <POD_NAME>

# Get logs from previous container instance (if crashed)
kubectl logs <POD_NAME> --previous

# Get logs with timestamps
kubectl logs <POD_NAME> --timestamps
```

## 4. Check Events
```powershell
# Get events (sorted by time)
kubectl get events --sort-by='.lastTimestamp'

# Get events for a specific resource
kubectl describe pod <POD_NAME>

# Watch events in real-time
kubectl get events --watch
```

## 5. Execute Into Pod (Interactive Debugging)
```powershell
# Open shell in running pod
kubectl exec -it <POD_NAME> -- /bin/bash

# Or if bash is not available, try sh
kubectl exec -it <POD_NAME> -- /bin/sh

# Run a single command
kubectl exec <POD_NAME> -- ps aux
kubectl exec <POD_NAME> -- env
```

## 6. Check Container Status
```powershell
# Check if container is running
kubectl get pods -o wide

# Check resource usage
kubectl top pod <POD_NAME>
kubectl top node
```

## 7. Common Issues & Solutions

### Image Pull Errors

#### ErrImageNeverPull
This error means Kubernetes can't find the image locally. If using minikube, the image needs to be in minikube's Docker daemon, not your local Docker.

**Solution for Minikube:**

```powershell
# Option 1: Use minikube's Docker daemon (PowerShell)
# Configure Docker to use minikube's daemon
minikube docker-env | Invoke-Expression

# Now build the image (it will be in minikube's Docker)
docker build -t rag-app .

# Option 2: Load existing image into minikube
minikube image load rag-app

# Option 3: Build directly in minikube
minikube image build -t rag-app .
```

**Check image exists:**
```powershell
# Check local Docker images
docker images --format "{{.Repository}}:{{.Tag}}" | Select-String "rag-app"

# If using minikube, check after setting docker-env:
minikube docker-env | Invoke-Expression
docker images | Select-String "rag-app"
```

**If minikube command not found:**
- Install minikube or find where it's installed
- Or use Docker Desktop Kubernetes (may share Docker daemon)
- Or push image to a registry and use `imagePullPolicy: IfNotPresent`

### Pod CrashLoopBackOff
```powershell
# Check why pod is crashing
kubectl logs <POD_NAME> --previous

# Check pod description for errors
kubectl describe pod <POD_NAME>
```

### Port Issues
```powershell
# Check if service is exposing the port correctly
kubectl get svc
kubectl describe svc <SERVICE_NAME>

# Port forward to test locally
kubectl port-forward <POD_NAME> 8000:8000
# Then test: http://localhost:8000
```

### Configuration Issues
```powershell
# Verify deployment YAML syntax
kubectl apply --dry-run=client -f deployment.yaml

# Validate resource
kubectl get deployment rag-app-deployment -o yaml
```

## 8. Quick Debugging Workflow
```powershell
# 1. Check deployment status
kubectl get deployment rag-app-deployment

# 2. Check pods
kubectl get pods -l app=rag-api

# 3. If pod not running, describe it
kubectl describe pod <POD_NAME>

# 4. Check logs
kubectl logs <POD_NAME>

# 5. Check recent events
kubectl get events --sort-by='.lastTimestamp' | Select-Object -Last 10

# 6. If pod is running but not responding, exec into it
kubectl exec -it <POD_NAME> -- /bin/sh
```

## 9. PowerShell-Specific Commands
```powershell
# Format output as table
kubectl get pods -o wide | Format-Table

# Filter pod status
kubectl get pods | Where-Object { $_ -match "Error\|CrashLoop\|Pending" }

# Get pod names only
kubectl get pods -l app=rag-api -o jsonpath="{.items[*].metadata.name}"
```

## 10. Advanced Debugging

### Check All Resources
```powershell
kubectl get all -l app=rag-api
```

### Export Configuration
```powershell
# Export current deployment config
kubectl get deployment rag-app-deployment -o yaml > deployment-current.yaml
```

### Check Network Connectivity
```powershell
# Test connection from within pod
kubectl exec <POD_NAME> -- wget -O- http://localhost:8000
```
