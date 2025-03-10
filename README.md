# Kubernetes-Minikube-Project
This is the steps involved in this project


 Step 1: Install Minikube on Windows

1. Install Hyper-V (if not already installed):
   - Open PowerShell as Administrator and enable Hyper-V by running:
    ```bash
     dism.exe /Online /Enable-Feature:Microsoft-Hyper-V-All /All /LimitAccess /Restart
     ```
     

2. Download and Install Minikube:
   - Install Minikube via `choco` (Chocolatey package manager):
     ```bash
     choco install minikube
     ```

3. Start Minikube:
   - Once Minikube is installed, start it using the following command:
     ```bash
     minikube start --driver=hyperv
     ```

4. Verify Minikube is Running:
   - To check if the Minikube cluster is running successfully, run:
     ```bash
     minikube status
     ```

---

 # Step 2: Install kubectl (Kubernetes CLI)

1. Install kubectl via Chocolatey:
   ```bash
   choco install kubernetes-cli
   ```

2. Verify kubectl Installation:
   - Run the following command to check if kubectl is working:
     ```bash
     kubectl version --client
     ```

---

# Step 3: Create a Kubernetes Deployment for Nginx

1. Create a Deployment YAML for Nginx (`nginx-deployment.yaml`):
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     replicas: 2  # Number of replicas
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
           image: nginx:latest
           ports:
           - containerPort: 80
   ```

2. Apply the Deployment:
   - Run the following command to apply the deployment to your cluster:
     ```bash
     kubectl apply -f k8s/nginx-deployment.yaml
     ```

3. Verify Deployment:
   - Check the status of the deployed pods:
     ```bash
     kubectl get pods
     ```

---

# Step 4: Expose the Nginx Deployment Using a Service

1. Create a Service YAML for Nginx (`nginx-service.yaml`):
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service
   spec:
     selector:
       app: nginx
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
     type: NodePort
   ```

2. Apply the Service:
   - Run the following command to create the service:
     ```bash
     kubectl apply -f k8s/nginx-service.yaml
     ```

3. Verify the Service:
   - Check if the service is running and note the port number:
     ```bash
     kubectl get svc
     ```

---

# Step 5: Scale the Nginx Deployment

1. Update the Deployment to Scale the Application (`nginx-deployment.yaml`):
   - Modify the `replicas` field to scale the number of Nginx pods. For example, increase it to 3:
   ```yaml
   replicas: 3
   ```

2. Apply the Updated Deployment:
   - Apply the changes to scale the deployment:
     ```bash
     kubectl apply -f k8s/nginx-deployment.yaml
     ```

3. Verify the Scaling:
   - Run the following command to see if there are now 3 pods running:
     ```bash
     kubectl get pods
     ```

---

# Step 6: Add Persistence with Persistent Volumes

1. Create a Persistent Volume YAML (`nginx-pv.yaml`):
   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: nginx-pv
   spec:
     capacity:
       storage: 1Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: /mnt/data  # Path to your desired storage location
   ```

2. Create a Persistent Volume Claim YAML (`nginx-pvc.yaml`):
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: nginx-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```

3. Update Nginx Deployment to Use the PVC:
   Modify the `nginx-deployment.yaml` to mount the persistent volume claim to the Nginx container.

   ```yaml
   volumeMounts:
   - name: nginx-storage
     mountPath: /usr/share/nginx/html  # Where data will be stored
   volumes:
   - name: nginx-storage
     persistentVolumeClaim:
       claimName: nginx-pvc  # Use the PVC defined earlier
   ```

4. Apply the Persistent Volume, PVC, and Deployment:
   - Run the following commands:
     ```bash
     kubectl apply -f k8s/nginx-pv.yaml
     kubectl apply -f k8s/nginx-pvc.yaml
     kubectl apply -f k8s/nginx-deployment.yaml
     ```

---

# Step 7: Use ConfigMaps and Secrets

# 3.1: Create a ConfigMap

1. Create a ConfigMap YAML (`nginx-configmap.yaml`):
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: nginx-config
   data:
     NGINX_HOST: "localhost"
     NGINX_PORT: "80"
   ```

2. Apply the ConfigMap:
   ```bash
   kubectl apply -f k8s/nginx-configmap.yaml
   ```

3. Update Nginx Deployment to Use ConfigMap:
   - Modify the `nginx-deployment.yaml` to reference the ConfigMap:
   ```yaml
   envFrom:
     - configMapRef:
         name: nginx-config
   ```

4. Apply the Deployment:
   ```bash
   kubectl apply -f k8s/nginx-deployment.yaml
   ```

# 3.2: Create a Secret

1. Base64 Encode Sensitive Data:
   - Base64 encode your sensitive data (e.g., API key) using this command in PowerShell:
     ```powershell
     [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("your-api-key"))
     ```

2. Create the Secret YAML (`nginx-secret.yaml`):
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: nginx-secret
   type: Opaque
   data:
     api-key: bXlzZWNyZXRhcGlrZXk=  # Replace with your base64 encoded API key
   ```

3. Apply the Secret:
   ```bash
   kubectl apply -f k8s/nginx-secret.yaml
   ```

4. Update the Deployment to Use the Secret:
   - Modify the `nginx-deployment.yaml` to inject the secret as an environment variable:
   ```yaml
   env:
     - name: API_KEY
       valueFrom:
         secretKeyRef:
           name: nginx-secret
           key: api-key  # Reference the secret key
   ```

5. Apply the Updated Deployment:
   ```bash
   kubectl apply -f k8s/nginx-deployment.yaml
   ```

---

# Final Verification: 

1. Check the Pods:
   - Verify that your Nginx pods are running and using the correct ConfigMap and Secret:
     ```bash
     kubectl get pods
     ```

2. Inspect the ConfigMap:
   - To check the details of the ConfigMap:
     ```bash
     kubectl get configmap nginx-config -o yaml
     ```

3. Inspect the Secret:
   - To check the details of the Secret:
     ```bash
     kubectl get secret nginx-secret -o yaml
     ```

4. Verify the Environment Variables:
   - You can describe the pod to check if the environment variables from the ConfigMap and Secret have been correctly set:
     ```bash
     kubectl describe pod <pod-name>
     ```

---

# Conclusion

The project was successfully completed the project with the following:

1. Minikube installed and a Kubernetes cluster up and running on Windows.
2. Nginx deployment created and exposed via a Service.
3. Scaling the Nginx deployment by increasing the number of replicas.
4. Persistence added with Persistent Volumes and Persistent Volume Claims.
5. Configured ConfigMaps for non-sensitive data and Secrets for sensitive data.


🔧 Challenges Encountered & Solutions 🔧
Here are some of the challenges I faced and the creative solutions I came up with:

# Challenge: Memory Allocation Error
I initially hit an error with Minikube stating that there wasn’t enough memory to start the virtual machine. The app couldn’t run due to insufficient resources.
Solution: I resolved this by adjusting the virtual machine’s memory allocation down to 2GB, allowing Minikube to start successfully.

# Challenge: Pod Failure and CrashLoopBackOff
After deploying the application, I ran into a CrashLoopBackOff issue where the pods failed to start, resulting in continuous restarts.
Solution: I checked the logs to identify the root cause and discovered missing configuration files in the deployment YAML. After fixing the configuration, the pods were successfully restarted without errors.

# Challenge: Issues with Secrets and ConfigMaps
When managing sensitive data using Secrets, I encountered an error with the base64 encoding of sensitive data, which caused the secrets to fail during deployment.
Solution: I properly encoded the values using the base64 command and added them to the YAML files. The error was resolved, and the secret was successfully created.

# Challenge: VM Deletion Failures
When trying to reset my Minikube cluster, I faced a deletion error where Minikube couldn’t find the necessary configuration files.
Solution: I manually cleaned up the leftover files and successfully deleted the Minikube cluster, allowing me to start fresh for the next deployment.

#  What I Learned from the Minikube Experience
 1. Hands-on with Kubernetes: Minikube gave me the perfect environment to practice Kubernetes without needing to manage a full cloud infrastructure.
 2.  Scaling & Self-Healing: Seeing Kubernetes automatically scale and recover applications in real-time showed me the power of container orchestration.
 3.  Efficient Resource Management: I gained insights into how Kubernetes handles resources and balances workloads in a way that keeps everything running smoothly.
 4. ConfigMaps & Secrets: Configuring and managing secrets and persistent volumes gave me a solid understanding of how to keep data safe and available in Kubernetes.



