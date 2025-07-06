# KinD and kubectl Command Notes

## Key Concepts

- **Isolated Environments:** Each KinD cluster is a completely separate and isolated Kubernetes environment. They do not share namespaces, workloads, or any other resources.
- **`kubectl` Context:** Your `kubectl` command acts like a remote control that can only point to one cluster at a time. The "context" tells `kubectl` which cluster to talk to.
- **Automatic Context Switching:** When you create a new cluster with `kind create cluster`, KinD automatically updates your `kubectl` configuration to point to the newly created cluster.

## Managing KinD Clusters

### Create a Cluster

- **Create a default cluster:**
  ```bash
  kind create cluster
  ```
  The `kubectl` context will be named `kind-kind`.

- **Create a named cluster:**
  ```bash
  kind create cluster --name <cluster-name>
  ```
  *Example:* `kind create cluster --name v1`
  The `kubectl` context will be `kind-v1`.

### List Clusters

See all the KinD clusters currently running on your machine.

```bash
kind get clusters
```
*Example Output:*
```
kind
v1
```

### Delete a Cluster

- **Delete the default cluster:**
  ```bash
  kind delete cluster
  ```

- **Delete a named cluster:**
  ```bash
  kind delete cluster --name <cluster-name>
  ```
  *Example:* `kind delete cluster --name v1`

## Managing `kubectl` Context

### See All Available Contexts

List all the cluster connections `kubectl` knows about. The one with the `*` is the currently active one.

```bash
kubectl config get-contexts
```
*Example Output:*
```
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         kind-v1    kind-v1    kind-v1
          kind-kind  kind-kind  kind-kind
```

### Get the Current Context

Show only the currently active context.

```bash
kubectl config current-context
```

### Switch Between Contexts

Change the cluster that `kubectl` is pointing to.

```bash
kubectl config use-context <context-name>
```
*Example:*
```bash
# Switch to the 'v1' cluster
kubectl config use-context kind-v1

# Switch back to the default cluster
kubectl config use-context kind-kind
```

After switching contexts, all `kubectl` commands (`get pods`, `get namespaces`, etc.) will be directed to the newly selected cluster.

---

## Deploying & Accessing Applications

### 1. Apply a Deployment

Create resources (like a deployment) in your cluster from a YAML file.

```bash
kubectl apply -f <your-deployment-file>.yaml
```
*Example:* `kubectl apply -f v1-deployment.yaml`

### 2. Check the Deployment

Verify that the deployment was created successfully and its pods are running.

```bash
kubectl get deployment <deployment-name>
```
*Example:* `kubectl get deployment v1`

### 3. Check the Pods

List the running pods created by the deployment.

```bash
kubectl get pods
```

### 4. Access the Application (Port Forward)

The quickest way to access your application from your local machine for testing.

**A. Get the pod name:**
```bash
kubectl get pods
```
(Look for a pod name like `v1-xxxxxxxxxx-yyyyy`)

**B. Forward the port:**
This command connects your local port (e.g., 8080) to the pod's port (e.g., 80).
```bash
kubectl port-forward <pod-name> <local-port>:<pod-port>
```
*Example:* `kubectl port-forward v1-6d5d4c5b8f-abcde 8080:80`

**C. Test the connection:**
Open a **new terminal** and use `curl` or a browser.
```bash
curl localhost:8080
```

---

## Inspecting Resources

### Get Detailed Information (Describe)

Get a detailed, human-readable report on any resource. Essential for troubleshooting.

- **Describe a node:**
  ```bash
  kubectl describe node <node-name>
  ```
- **Describe a pod:**
  ```bash
  kubectl describe pod <pod-name>
  ```
- **Describe a deployment:**
  ```bash
  kubectl describe deployment <deployment-name>
  ```

---

## Exposing Applications: A Complete Example

This section provides a complete, end-to-end guide for exposing your application using a `NodePort` service, made accessible on `localhost` via your `kind` configuration.

### The Networking Flow

`Your Browser` -> `http://localhost:8080` -> `(Docker Network)` -> `Node:30080` -> `Service` -> `Pod:80`

### Step 1: Configure `kind` for Port Mapping

Your `kind-v1.yml` is configured to forward traffic from your local machine's port `8080` to the `NodePort` `30080` inside the cluster's control-plane node. This is done with `extraPortMappings`.

**`kind-v1.yml`:**
```yaml
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080  # The NodePort in the K8s service
    hostPort: 8080        # The port on your local machine
    protocol: TCP
```

**Action:** You must create your cluster with this configuration.
```bash
# Delete the old cluster if it exists
kind delete cluster --name v1

# Create the new cluster with the port mapping
kind create cluster --name v1 --config kind-v1.yml
```

### Step 2: Deploy Your Application

Your `v1-deployment.yaml` deploys 3 replicas of Nginx. The key part is the `labels` which the service will use to find the pods, and the `containerPort` which the service will send traffic to.

**`v1-deployment.yaml`:**
```yaml
metadata:
  labels:
    app: nginx #<-- Service selector will match this
spec:
  template:
    metadata:
      labels:
        app: nginx #<-- Service selector will match this
    spec:
      containers:
      - name: nginx
        ports:
        - containerPort: 80 #<-- Service will send traffic here
```
**Action:**
```bash
kubectl apply -f v1-deployment.yaml
```

### Step 3: Create the `NodePort` Service

The `v1-service.yaml` file creates the service that exposes the pods.

- `type: NodePort`: Exposes the service on a port on each node.
- `selector: app: nginx`: Finds all pods with the `app: nginx` label.
- `port: 80`: The service's own internal port.
- `targetPort: 80`: The port on the pod to send traffic to.
- `nodePort: 30080`: The specific high-level port to open on all nodes. This **must match** the `containerPort` in your `kind-v1.yml`.

**`v1-service.yaml`:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: v1-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```
**Action:**
```bash
kubectl apply -f v1-service.yaml
```

### Step 4: Access Your Application

After completing these steps, you can now access your Nginx service directly from your host machine.

```bash
# Test the connection
curl localhost:8080
```
You should see the "Welcome to nginx!" page.
