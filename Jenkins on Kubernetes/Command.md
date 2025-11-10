# Jenkins on Kubernetes - Commands Reference

Complete command reference for deploying, managing, and troubleshooting Jenkins on Kubernetes.

---

## Table of Contents

1. [Deployment Commands](#deployment-commands)
2. [Verification Commands](#verification-commands)
3. [Access Commands](#access-commands)
4. [Monitoring Commands](#monitoring-commands)
5. [Troubleshooting Commands](#troubleshooting-commands)
6. [Management Commands](#management-commands)
7. [Cleanup Commands](#cleanup-commands)
8. [Advanced Commands](#advanced-commands)

---

## Deployment Commands

### Apply All Resources

```bash
# Deploy Jenkins with all resources
kubectl apply -f jenkins-setup.yaml
```

### Apply Individual Resources

```bash
# Create namespace only
kubectl create namespace jenkins

# Apply service only
kubectl apply -f jenkins-service.yaml

# Apply deployment only
kubectl apply -f jenkins-deployment.yaml
```

### Verify Deployment

```bash
# Check if all resources are created
kubectl get all -n jenkins
```

---

## Verification Commands

### Check Namespace

```bash
# List all namespaces
kubectl get namespaces

# Get specific namespace details
kubectl get namespace jenkins

# Describe namespace
kubectl describe namespace jenkins
```

### Check Deployment

```bash
# List deployments in jenkins namespace
kubectl get deployments -n jenkins

# Detailed deployment info
kubectl get deployment jenkins-deployment -n jenkins -o wide

# Describe deployment
kubectl describe deployment jenkins-deployment -n jenkins

# Check deployment status
kubectl rollout status deployment/jenkins-deployment -n jenkins
```

### Check Pods

```bash
# List all pods
kubectl get pods -n jenkins

# Get detailed pod information
kubectl get pods -n jenkins -o wide

# Watch pod status (real-time)
kubectl get pods -n jenkins -w

# Describe specific pod
kubectl describe pod <pod-name> -n jenkins

# Check pod with labels
kubectl get pods -n jenkins -l app=jenkins
```

### Check Service

```bash
# List services
kubectl get services -n jenkins

# Get service details
kubectl get service jenkins-service -n jenkins -o wide

# Describe service
kubectl describe service jenkins-service -n jenkins

# Get service endpoints
kubectl get endpoints jenkins-service -n jenkins
```

### Check ReplicaSet

```bash
# List replica sets
kubectl get replicasets -n jenkins

# Describe replica set
kubectl describe replicaset -n jenkins
```

---

## Access Commands

### Get Node Information

```bash
# Get all nodes with IP addresses
kubectl get nodes -o wide

# Get node details
kubectl describe nodes

# Get specific node details
kubectl describe node <node-name>
```

### Get Service URL

```bash
# Get service details with NodePort
kubectl get service jenkins-service -n jenkins

# Get service URL (if using minikube)
minikube service jenkins-service -n jenkins --url
```

### Get Initial Admin Password

```bash
# Method 1: Using kubectl exec
kubectl exec -n jenkins deployment/jenkins-deployment -- cat /var/jenkins_home/secrets/initialAdminPassword

# Method 2: Using pod name
kubectl exec -n jenkins <pod-name> -- cat /var/jenkins_home/secrets/initialAdminPassword

# Method 3: From logs
kubectl logs -n jenkins deployment/jenkins-deployment | grep -A 5 "Jenkins initial setup"
```

### Port Forwarding (Alternative Access)

```bash
# Forward local port 8080 to Jenkins pod
kubectl port-forward -n jenkins deployment/jenkins-deployment 8080:8080

# Access via: http://localhost:8080

# Forward to different local port
kubectl port-forward -n jenkins deployment/jenkins-deployment 9090:8080

# Access via: http://localhost:9090
```

---

## Monitoring Commands

### View Logs

```bash
# View deployment logs
kubectl logs -n jenkins deployment/jenkins-deployment

# Follow logs in real-time
kubectl logs -n jenkins deployment/jenkins-deployment -f

# View logs from specific pod
kubectl logs -n jenkins <pod-name>

# View previous pod logs (after restart)
kubectl logs -n jenkins <pod-name> --previous

# View logs with timestamps
kubectl logs -n jenkins deployment/jenkins-deployment --timestamps

# View last 100 lines of logs
kubectl logs -n jenkins deployment/jenkins-deployment --tail=100

# View logs from last hour
kubectl logs -n jenkins deployment/jenkins-deployment --since=1h
```

### Resource Usage

```bash
# Get pod resource usage
kubectl top pods -n jenkins

# Get node resource usage
kubectl top nodes

# Watch resource usage
watch kubectl top pods -n jenkins
```

### Events

```bash
# Get all events in namespace
kubectl get events -n jenkins

# Sort events by timestamp
kubectl get events -n jenkins --sort-by='.lastTimestamp'

# Watch events
kubectl get events -n jenkins -w

# Get events for specific resource
kubectl get events -n jenkins --field-selector involvedObject.name=jenkins-deployment
```

---

## Troubleshooting Commands

### Pod Troubleshooting

```bash
# Describe pod to see events and status
kubectl describe pod -n jenkins -l app=jenkins

# Check pod conditions
kubectl get pod -n jenkins -o jsonpath='{.items[0].status.conditions}'

# Get pod YAML
kubectl get pod -n jenkins <pod-name> -o yaml

# Check container status
kubectl get pod -n jenkins <pod-name> -o jsonpath='{.status.containerStatuses[*].state}'
```

### Interactive Debugging

```bash
# Execute bash in Jenkins container
kubectl exec -it -n jenkins deployment/jenkins-deployment -- /bin/bash

# Run specific command in container
kubectl exec -n jenkins deployment/jenkins-deployment -- ls -la /var/jenkins_home

# Check Java process
kubectl exec -n jenkins deployment/jenkins-deployment -- ps aux | grep java

# Check disk usage
kubectl exec -n jenkins deployment/jenkins-deployment -- df -h
```

### Network Troubleshooting

```bash
# Test service connectivity from within cluster
kubectl run test-pod --rm -it --image=busybox -n jenkins -- /bin/sh
# Then inside the pod:
# wget -O- http://jenkins-service:8080

# Check DNS resolution
kubectl exec -n jenkins deployment/jenkins-deployment -- nslookup jenkins-service

# Test network connectivity
kubectl exec -n jenkins deployment/jenkins-deployment -- curl -I localhost:8080
```

### Service Troubleshooting

```bash
# Get service endpoints
kubectl get endpoints -n jenkins

# Verify service selector matches pods
kubectl get pods -n jenkins --show-labels

# Test service from another pod
kubectl run test --rm -it --image=alpine -n jenkins -- wget -qO- http://jenkins-service:8080
```

---

## Management Commands

### Scaling

```bash
# Scale deployment to 2 replicas
kubectl scale deployment jenkins-deployment -n jenkins --replicas=2

# Scale down to 1 replica
kubectl scale deployment jenkins-deployment -n jenkins --replicas=1

# Scale to 0 (stop without deleting)
kubectl scale deployment jenkins-deployment -n jenkins --replicas=0
```

### Update Deployment

```bash
# Update container image
kubectl set image deployment/jenkins-deployment jenkins-container=jenkins/jenkins:lts -n jenkins

# Edit deployment directly
kubectl edit deployment jenkins-deployment -n jenkins

# Apply changes from updated YAML
kubectl apply -f jenkins-setup.yaml

# Restart deployment
kubectl rollout restart deployment/jenkins-deployment -n jenkins
```

### Rollback

```bash
# View rollout history
kubectl rollout history deployment/jenkins-deployment -n jenkins

# Rollback to previous version
kubectl rollout undo deployment/jenkins-deployment -n jenkins

# Rollback to specific revision
kubectl rollout undo deployment/jenkins-deployment -n jenkins --to-revision=2

# Check rollout status
kubectl rollout status deployment/jenkins-deployment -n jenkins
```

### Pause and Resume

```bash
# Pause deployment (stop auto-updates)
kubectl rollout pause deployment/jenkins-deployment -n jenkins

# Resume deployment
kubectl rollout resume deployment/jenkins-deployment -n jenkins
```

---

## Cleanup Commands

### Delete Specific Resources

```bash
# Delete deployment only
kubectl delete deployment jenkins-deployment -n jenkins

# Delete service only
kubectl delete service jenkins-service -n jenkins

# Delete pod (will be recreated by deployment)
kubectl delete pod <pod-name> -n jenkins
```

### Delete All Resources

```bash
# Delete all resources in namespace
kubectl delete all --all -n jenkins

# Delete namespace (removes all resources)
kubectl delete namespace jenkins

# Force delete namespace if stuck
kubectl delete namespace jenkins --grace-period=0 --force
```

### Delete Using File

```bash
# Delete resources defined in YAML
kubectl delete -f jenkins-setup.yaml
```

---

## Advanced Commands

### Wait Commands

```bash
# Wait for pod to be ready
kubectl wait --for=condition=ready pod -l app=jenkins -n jenkins --timeout=300s

# Wait for deployment to complete
kubectl wait --for=condition=available deployment/jenkins-deployment -n jenkins --timeout=300s

# Wait for pod deletion
kubectl wait --for=delete pod/<pod-name> -n jenkins --timeout=60s
```

### Export/Backup Configuration

```bash
# Export deployment YAML
kubectl get deployment jenkins-deployment -n jenkins -o yaml > jenkins-deployment-backup.yaml

# Export service YAML
kubectl get service jenkins-service -n jenkins -o yaml > jenkins-service-backup.yaml

# Export all resources
kubectl get all -n jenkins -o yaml > jenkins-all-backup.yaml

# Export with specific API version
kubectl get deployment jenkins-deployment -n jenkins -o yaml --export > jenkins-deployment.yaml
```

### Labels and Annotations

```bash
# Add label to deployment
kubectl label deployment jenkins-deployment -n jenkins environment=production

# Remove label
kubectl label deployment jenkins-deployment -n jenkins environment-

# Add annotation
kubectl annotate deployment jenkins-deployment -n jenkins description="Jenkins CI Server"

# View labels
kubectl get deployment jenkins-deployment -n jenkins --show-labels
```

### Resource Quotas and Limits

```bash
# Create resource quota for namespace
kubectl create quota jenkins-quota --hard=pods=10,services=5 -n jenkins

# View resource quotas
kubectl get resourcequota -n jenkins

# Describe resource quota
kubectl describe resourcequota jenkins-quota -n jenkins

# View resource usage
kubectl describe namespace jenkins
```

### ConfigMaps and Secrets

```bash
# Create ConfigMap
kubectl create configmap jenkins-config --from-literal=key=value -n jenkins

# Create Secret
kubectl create secret generic jenkins-secret --from-literal=password=mypassword -n jenkins

# View ConfigMaps
kubectl get configmaps -n jenkins

# View Secrets
kubectl get secrets -n jenkins

# Describe ConfigMap
kubectl describe configmap jenkins-config -n jenkins
```

### Copy Files

```bash
# Copy file from pod to local
kubectl cp jenkins/<pod-name>:/var/jenkins_home/config.xml ./config.xml

# Copy file from local to pod
kubectl cp ./config.xml jenkins/<pod-name>:/var/jenkins_home/config.xml

# Copy directory
kubectl cp jenkins/<pod-name>:/var/jenkins_home/jobs ./jobs-backup
```

### Debug Containers

```bash
# Create debug container (Kubernetes 1.23+)
kubectl debug -it <pod-name> -n jenkins --image=busybox --target=jenkins-container

# Run ephemeral debug container
kubectl alpha debug <pod-name> -n jenkins -it --image=ubuntu --share-processes
```

---

## Quick Reference Cheat Sheet

### One-Line Commands

```bash
# Complete deployment check
kubectl get all -n jenkins && kubectl get pods -n jenkins -o wide

# Quick health check
kubectl get pods -n jenkins && kubectl logs -n jenkins deployment/jenkins-deployment --tail=20

# Get Jenkins URL and password
echo "URL: http://$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}'):30008" && kubectl exec -n jenkins deployment/jenkins-deployment -- cat /var/jenkins_home/secrets/initialAdminPassword

# Full status check
kubectl get namespace jenkins && kubectl get all -n jenkins && kubectl get events -n jenkins --sort-by='.lastTimestamp' | tail -10
```

### Useful Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc

alias k='kubectl'
alias kgp='kubectl get pods'
alias kgd='kubectl get deployments'
alias kgs='kubectl get services'
alias kga='kubectl get all'
alias kdp='kubectl describe pod'
alias kdd='kubectl describe deployment'
alias kl='kubectl logs'
alias kx='kubectl exec -it'

# Jenkins specific
alias jenkins-logs='kubectl logs -n jenkins deployment/jenkins-deployment -f'
alias jenkins-pods='kubectl get pods -n jenkins'
alias jenkins-status='kubectl get all -n jenkins'
alias jenkins-exec='kubectl exec -it -n jenkins deployment/jenkins-deployment -- /bin/bash'
alias jenkins-password='kubectl exec -n jenkins deployment/jenkins-deployment -- cat /var/jenkins_home/secrets/initialAdminPassword'
```

---

## Environment Variables

### Useful Environment Variables

```bash
# Set default namespace
export NAMESPACE=jenkins
kubectl config set-context --current --namespace=jenkins

# Set kubectl verbosity
export KUBECTL_VERBOSITY=8

# Verify current context
kubectl config current-context

# View current namespace
kubectl config view --minify | grep namespace:
```

---

## Automation Scripts

### Deployment Script

```bash
#!/bin/bash
# deploy-jenkins.sh

echo "Deploying Jenkins to Kubernetes..."
kubectl apply -f jenkins-setup.yaml

echo "Waiting for pod to be ready..."
kubectl wait --for=condition=ready pod -l app=jenkins -n jenkins --timeout=300s

echo "Jenkins deployed successfully!"
echo "Access URL: http://$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}'):30008"
echo "Initial password:"
kubectl exec -n jenkins deployment/jenkins-deployment -- cat /var/jenkins_home/secrets/initialAdminPassword
```

### Health Check Script

```bash
#!/bin/bash
# health-check.sh

echo "=== Jenkins Health Check ==="
echo ""
echo "1. Namespace Status:"
kubectl get namespace jenkins

echo ""
echo "2. Deployment Status:"
kubectl get deployment jenkins-deployment -n jenkins

echo ""
echo "3. Pod Status:"
kubectl get pods -n jenkins

echo ""
echo "4. Service Status:"
kubectl get service jenkins-service -n jenkins

echo ""
echo "5. Recent Events:"
kubectl get events -n jenkins --sort-by='.lastTimestamp' | tail -5
```

---

## Additional Resources

### Help Commands

```bash
# Get help for kubectl
kubectl --help

# Get help for specific command
kubectl get --help
kubectl describe --help
kubectl logs --help

# Explain resource types
kubectl explain pod
kubectl explain deployment
kubectl explain service
```

### API Resources

```bash
# List all API resources
kubectl api-resources

# Get API versions
kubectl api-versions

# Get resource definitions
kubectl get --raw /api/v1/namespaces/jenkins/pods
```

---

**Note:** Replace `<pod-name>` and `<node-name>` with actual names from your cluster.

**Quick Tips:**
- Use `kubectl get pods -n jenkins` to find pod names
- Use `kubectl get nodes` to find node names
- Use `-o yaml` flag to get detailed output in YAML format
- Use `-o json` flag to get detailed output in JSON format
- Use `--watch` or `-w` flag to watch resources in real-time
- Use `--help` with any command to see available options

---

**Last Updated:** November 2025
