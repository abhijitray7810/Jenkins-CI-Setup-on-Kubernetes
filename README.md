# Jenkins-CI-Setup-on-Kubernetes
# Jenkins on Kubernetes - Setup Guide

This guide provides instructions for deploying Jenkins CI server on a Kubernetes cluster.

## Overview

This deployment creates: 
- A dedicated `jenkins` namespace
- A Jenkins deployment with 1 replica
- A NodePort service exposing Jenkins on port 30008
 
## Prerequisites

- Kubernetes cluster up and running
- `kubectl` configured to communicate with the cluster
- Sufficient cluster resources (minimum 2GB RAM recommended for Jenkins)

## Architecture


![image](https://github.com/abhijitray7810/Jenkins-CI-Setup-on-Kubernetes/blob/e1b983fd593817bc96ef4bd59a3d784c4f41516c/Architecture%20Image/Jenkins%20CI%20Setup%20on%20Kubernetes.png)



## Deployment Instructions

### Step 1: Apply the Kubernetes Manifests

Save the YAML configuration to a file (e.g., `jenkins-setup.yaml`) and apply it:

```bash
kubectl apply -f jenkins-setup.yaml
```

Expected output:
```
namespace/jenkins created
service/jenkins-service created
deployment.apps/jenkins-deployment created
```

### Step 2: Verify Namespace Creation

```bash
kubectl get namespace jenkins
```

Expected output:
```
NAME      STATUS   AGE
jenkins   Active   10s
```

### Step 3: Check Deployment Status

```bash
kubectl get deployments -n jenkins
```

Expected output:
```
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
jenkins-deployment   1/1     1            1           30s
```

### Step 4: Monitor Pod Status

Check if the pod is running:

```bash
kubectl get pods -n jenkins
```

Or watch the pod status in real-time:

```bash
kubectl get pods -n jenkins -w
```

Press `Ctrl+C` to exit watch mode once the pod status shows `Running`.

Expected output:
```
NAME                                  READY   STATUS    RESTARTS   AGE
jenkins-deployment-xxxxxxxxxx-xxxxx   1/1     Running   0          1m
```

### Step 5: Verify Service Creation

```bash
kubectl get service -n jenkins
```

Expected output:
```
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins-service   NodePort   10.xx.xxx.xxx   <none>        8080:30008/TCP   2m
```

### Step 6: Wait for Jenkins to Be Fully Ready

Use this command to wait for the pod to be ready:

```bash
kubectl wait --for=condition=ready pod -l app=jenkins -n jenkins --timeout=300s
```

Check Jenkins logs to confirm it's fully initialized:

```bash
kubectl logs -n jenkins deployment/jenkins-deployment
```

Look for messages indicating Jenkins has started successfully.

## Accessing Jenkins

### Get Node IP Address

```bash
kubectl get nodes -o wide
```

Note the `INTERNAL-IP` or `EXTERNAL-IP` of any node.

### Access Jenkins Web Interface

Open your browser and navigate to:

```
http://<NODE_IP>:30008
```

Replace `<NODE_IP>` with the actual IP address from the previous command.

### Retrieve Initial Admin Password

Jenkins requires an initial admin password on first login. Retrieve it using:

```bash
kubectl exec -n jenkins deployment/jenkins-deployment -- cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy the password and paste it into the Jenkins web interface.

## Configuration Details

### Namespace
- **Name:** `jenkins`

### Service
- **Name:** `jenkins-service`
- **Namespace:** `jenkins`
- **Type:** `NodePort`
- **Port:** `8080`
- **NodePort:** `30008`
- **Selector:** `app=jenkins`

### Deployment
- **Name:** `jenkins-deployment`
- **Namespace:** `jenkins`
- **Replicas:** `1`
- **Labels:** `app=jenkins`
- **Container Name:** `jenkins-container`
- **Image:** `jenkins/jenkins`
- **Container Port:** `8080`

## Troubleshooting

### Pod Not Starting

Check pod events:
```bash
kubectl describe pod -n jenkins -l app=jenkins
```

### Check Pod Logs

View detailed logs:
```bash
kubectl logs -n jenkins deployment/jenkins-deployment -f
```

### Service Not Accessible

Verify service endpoints:
```bash
kubectl get endpoints -n jenkins
```

Check if pod is running and ready:
```bash
kubectl get pods -n jenkins -o wide
```

### Port Already in Use

If port 30008 is already in use, you'll need to modify the `nodePort` value in the service definition.

## Cleanup

To remove the Jenkins deployment:

```bash
kubectl delete namespace jenkins
```

This will delete the namespace and all resources within it.

## Important Notes

- **First Startup:** Jenkins may take 1-2 minutes to fully initialize on first startup
- **Persistence:** This basic setup does not include persistent storage. Jenkins data will be lost if the pod is deleted
- **Security:** For production use, consider adding:
  - Persistent Volume Claims for data persistence
  - Resource limits and requests
  - Security contexts
  - Network policies
  - Ingress controller for HTTPS access
  - RBAC configurations

## Next Steps

After successfully logging in to Jenkins:

1. Install recommended plugins
2. Create the first admin user
3. Configure Jenkins URL
4. Set up your first CI/CD pipeline

## Support

For issues related to:
- **Kubernetes deployment:** Check cluster logs and pod events
- **Jenkins configuration:** Refer to [Jenkins Documentation](https://www.jenkins.io/doc/)

## Version Information

- **Jenkins Image:** `jenkins/jenkins` (latest)
- **Kubernetes API Version:** `v1` (core), `apps/v1` (deployment)

---

**Created for:** Nautilus DevOps Team  
**Purpose:** Jenkins CI Server on Kubernetes Cluster
