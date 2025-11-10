# Jenkins Deployment on Kubernetes

## Step 1: Create Namespace
```bash
kubectl create namespace jenkins
````

## Step 2: Create Jenkins Deployment

```bash
kubectl create deployment jenkins-deployment \
  --image=jenkins/jenkins \
  --replicas=1 \
  --namespace=jenkins \
  --port=8080
```

## Step 3: Add Label

```bash
kubectl label deployment jenkins-deployment app=jenkins -n jenkins
```

## Step 4: Expose Deployment as NodePort Service

```bash
kubectl expose deployment jenkins-deployment \
  --type=NodePort \
  --name=jenkins-service \
  --namespace=jenkins \
  --port=8080 \
  --target-port=8080 \
  --node-port=30008
```

## Step 5: Verify Resources

```bash
kubectl get all -n jenkins
kubectl get pods -n jenkins -w
```

## Step 6: Access Jenkins

Find Node IP:

```bash
kubectl get nodes -o wide
```

Then open Jenkins in browser:

```
http://<NODE_IP>:30008
```

---
