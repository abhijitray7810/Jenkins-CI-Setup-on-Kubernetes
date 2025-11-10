# Step 1: Create the namespace
kubectl create namespace jenkins

# Step 2: Create Jenkins Deployment
kubectl create deployment jenkins-deployment \
  --image=jenkins/jenkins \
  --replicas=1 \
  --namespace=jenkins \
  --port=8080

# Add label 'app=jenkins' to deployment
kubectl label deployment jenkins-deployment app=jenkins -n jenkins

# Step 3: Expose the deployment with NodePort service
kubectl expose deployment jenkins-deployment \
  --type=NodePort \
  --name=jenkins-service \
  --namespace=jenkins \
  --port=8080 \
  --target-port=8080 \
  --node-port=30008

# Step 4: Verify everything
kubectl get all -n jenkins

# Check Pod status until it’s running
kubectl get pods -n jenkins -w

# Step 5: Get Node IP (to access Jenkins)
kubectl get nodes -o wide

# Then open in browser:
# http://<NODE_IP>:30008
