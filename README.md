# Mosquitto Message Broker Deployment with ConfigMap and Secret Volume Types

## Project Description
This project demonstrates how to deploy the Mosquitto message broker on a local Minikube Kubernetes cluster, using ConfigMap and Secret Volume types to manage configurations and passwords securely. The project showcases best practices in utilizing these volume types to ensure a secure and maintainable deployment pipeline in DevOps workflows.

---

## Requirements
### Prerequisites
1. **Minikube** installed on your local machine.
2. **kubectl** CLI tool installed and configured to manage Minikube.
3. **Docker** installed and running.
4. Basic understanding of Kubernetes resources such as ConfigMaps, Secrets, Deployments, and Services.
5. Internet connectivity to pull Docker images.

---

## Key Concepts

### Mosquitto Message Broker
- **Mosquitto** is a lightweight MQTT message broker that enables messaging between IoT devices or other systems.

### ConfigMap
- A Kubernetes object that allows you to store configuration data as key-value pairs.
- **Why ConfigMap?** Decouples configuration artifacts from application code, enabling better reusability and maintainability.

### Secret
- A Kubernetes object to store sensitive information such as passwords or API keys.
- **Why Secret?** Provides encryption and access control, ensuring sensitive data is secure.

---

## Project Steps

### Step 1: Start Minikube
1. Open your terminal.
2. Start Minikube:
   ```bash
   minikube start
   ```
3. Verify Minikube is running:
   ```bash
   kubectl get nodes
   ```

### Step 2: Create a ConfigMap
1. Define Mosquitto configuration in a file (e.g., `mosquitto.conf`):
   ```
   listener 1883
   allow_anonymous true
   persistence true
   persistence_location /mosquitto/data/
   ```
2. Create the ConfigMap:
   ```bash
   kubectl create configmap mosquitto-config --from-file=mosquitto.conf
   ```
3. Verify ConfigMap:
   ```bash
   kubectl describe configmap mosquitto-config
   ```

### Step 3: Create a Secret
1. Encode your password as Base64:
   ```bash
   echo -n "yourpassword" | base64
   ```
2. Create a YAML file for the Secret (e.g., `mosquitto-secret.yaml`):
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: mosquitto-secret
   type: Opaque
   data:
     password: eW91cnBhc3N3b3Jk  # Base64-encoded password
   ```
3. Apply the Secret:
   ```bash
   kubectl apply -f mosquitto-secret.yaml
   ```
4. Verify the Secret:
   ```bash
   kubectl describe secret mosquitto-secret
   ```

### Step 4: Create a Deployment
1. Define a Deployment YAML file (e.g., `mosquitto-deployment.yaml`):
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mosquitto
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mosquitto
     template:
       metadata:
         labels:
           app: mosquitto
       spec:
         containers:
         - name: mosquitto
           image: eclipse-mosquitto
           ports:
           - containerPort: 1883
           volumeMounts:
           - name: config-volume
             mountPath: /mosquitto/config
           - name: secret-volume
             mountPath: /mosquitto/secret
             readOnly: true
         volumes:
         - name: config-volume
           configMap:
             name: mosquitto-config
         - name: secret-volume
           secret:
             secretName: mosquitto-secret
   ```
2. Apply the Deployment:
   ```bash
   kubectl apply -f mosquitto-deployment.yaml
   ```
3. Verify the Deployment:
   ```bash
   kubectl get pods
   ```

### Step 5: Expose the Deployment as a Service
1. Create a Service to expose Mosquitto:
   ```bash
   kubectl expose deployment mosquitto --type=NodePort --port=1883
   ```
2. Get the Service URL:
   ```bash
   minikube service mosquitto --url
   ```
3. Test the connection using an MQTT client (e.g., `mosquitto_pub` and `mosquitto_sub`).

---

## Best Practices

### ConfigMap
1. Keep environment-specific configurations separate.
2. Use descriptive names for keys.
3. Avoid storing sensitive data; use Secrets for such information.

### Secret
1. Always use Secrets for sensitive data such as passwords, API keys, or certificates.
2. Restrict access to Secrets using Kubernetes RBAC.
3. Encrypt Secrets at rest using Kubernetes' encryption feature.

### General Tips
1. Use version control for all YAML manifests.
2. Validate manifests with tools like `kubectl lint` or `kubeval`.
3. Use labels and annotations for better resource management.
4. Monitor the cluster and application logs for troubleshooting.

---

## Sum
This demo project demonstrates the deployment of a Mosquitto message broker using Kubernetes ConfigMaps and Secrets. Following these steps ensures secure, maintainable, and efficient configurations while adhering to best practices in a DevOps environment.

Happy Deploying! 🚀
