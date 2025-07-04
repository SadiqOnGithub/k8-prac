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
