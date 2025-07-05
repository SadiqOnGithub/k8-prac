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

## Advanced Experiments

### 1. Expose Deployment with a Service

Create a stable network endpoint that load-balances traffic to your pods.

- **Expose the deployment:**
  ```bash
  kubectl expose deployment <deployment-name> --type=NodePort --port=80
  ```
  *Example:* `kubectl expose deployment v1 --type=NodePort --port=80`

- **Find the service port:**
  ```bash
  kubectl get service <deployment-name>
  ```
  (Look for the port mapped to port 80, e.g., `80:31234/TCP`)

- **Access the service:**
  ```bash
  curl localhost:<node-port>
  ```
  *Example:* `curl localhost:31234`

### 2. Test Self-Healing

Watch Kubernetes automatically replace a deleted pod.

- **Watch pods in one terminal:**
  ```bash
  kubectl get pods -w
  ```
- **Delete a pod in a second terminal:**
  ```bash
  kubectl delete pod <pod-name>
  ```

### 3. Perform a Rolling Update

Update the application to a new version with zero downtime.

- **Update the container image:**
  ```bash
  kubectl set image deployment/<deployment-name> <container-name>=<new-image>
  ```
  *Example:* `kubectl set image deployment/v1 nginx=httpd`

- **Check rollout status:**
  ```bash
  kubectl rollout status deployment/<deployment-name>
  ```
