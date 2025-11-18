# Section 15: Container Orchestration using Kubernetes

This section provides Kubernetes manifest files to deploy the application. These YAML files are instructions that tell a Kubernetes cluster how to run, configure, and expose our microservices.

Below is a detailed breakdown of the `5_accounts.yml` file, with comments explaining each property.

### How It Works in Brief

1.  The **`Deployment`** object acts like a manager for the application's pods (running instances). It ensures that the correct number of pods are always running and knows how to create new ones based on the provided template.
2.  The **`Service`** object acts like a network receptionist. It provides a single, stable address (like `http://accounts:8080`) and automatically directs traffic to one of the healthy pods managed by the Deployment.

---

### `Deployment` Object: The Pod Manager

```yaml
# 5_accounts.yml

# --- DEPLOYMENT ---
apiVersion: apps/v1      # The version of the Kubernetes API to use for this object.
kind: Deployment         # The type of Kubernetes object we are defining.
metadata:
  name: accounts-deployment # The unique name for this Deployment object.
spec:                    # The specification that describes the desired state.
  replicas: 1            # Ensure that exactly one pod is running at all times.
  selector:
    matchLabels:
      app: accounts      # This Deployment manages any pod with the label "app: accounts".
  template:              # This is the blueprint for the pods that will be created.
    metadata:
      labels:
        app: accounts    # Assigns the label "app: accounts" to each pod. This is how the selector finds them.
    spec:
      containers:        # A list of containers to run inside each pod.
      - name: accounts     # A name for the container within the pod.
        image: eazybytes/accounts:s12 # The Docker image to pull and run.
        ports:
        - containerPort: 8080 # The port the application listens on inside the container.
        env:               # A list of environment variables to set inside the container.
        - name: SPRING_APPLICATION_NAME # The name of the environment variable.
          valueFrom:       # The value is not hardcoded; it's sourced from elsewhere.
            configMapKeyRef: # The source is a ConfigMap.
              name: eazybank-configmap # The name of the ConfigMap to use.
              key: ACCOUNTS_APPLICATION_NAME # The specific key within that ConfigMap to get the value from.
        - name: SPRING_PROFILES_ACTIVE
          valueFrom: 
            configMapKeyRef:
              name: eazybank-configmap
              key: SPRING_PROFILES_ACTIVE
        - name: SPRING_CONFIG_IMPORT
          valueFrom: 
            configMapKeyRef:
              name: eazybank-configmap
              key: SPRING_CONFIG_IMPORT
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          valueFrom: 
            configMapKeyRef:
              name: eazybank-configmap
              key: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
```

---

### `Service` Object: The Network Receptionist

```yaml
# --- SERVICE ---
apiVersion: v1           # The core, stable API version for fundamental objects like Services.
kind: Service            # The type of Kubernetes object.
metadata:
  name: accounts         # The name of the Service. This also becomes its internal DNS name.
spec:
  selector:
    app: accounts        # Routes traffic to any pod with the label "app: accounts". This links the Service to the Deployment's pods.
  type: LoadBalancer     # Exposes the service outside the cluster using a cloud provider's load balancer.
  ports:
    - protocol: TCP      # The IP protocol.
      port: 8080         # The port that the Service will expose on the load balancer.
      targetPort: 8080   # The port on the pod that traffic should be forwarded to. (Traffic to Service:8080 -> Pod:8080)
```
