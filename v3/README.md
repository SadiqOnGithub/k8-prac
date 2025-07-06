# V3: Advanced Kind Experiments

This directory demonstrates advanced features in the `kind.yml` configuration file.

## Key Features in `v3/kind.yml`

1.  **Node Labels:** We have labeled our worker nodes:
    *   One worker has the label `nodetype: frontend`.
    *   The other has `nodetype: backend`.
    This allows us to control where our pods are scheduled.

2.  **Multiple Port Mappings:** We have exposed two ports from the control-plane to your `localhost`:
    *   `localhost:8080` -> `node:30080` (for a primary web app)
    *   `localhost:9090` -> `node:30090` (for a secondary service, like an admin panel)

3.  **Local Directory Mounts (`extraMounts`):**
    *   The local directory `./v3/app` on your computer is mounted into the control-plane node at the path `/app-data`.

## How to Experiment

### Experiment 1: Node Scheduling

1.  Create a deployment for a "frontend" pod and use a `nodeSelector` to force it to run only on the frontend node.

    **`frontend-deployment.yml`:**
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: frontend-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: frontend
      template:
        metadata:
          labels:
            app: frontend
        spec:
          nodeSelector:
            nodetype: frontend # <-- This is the key!
          containers:
          - name: nginx
            image: nginx
    ```

2.  Apply it and check which node the pod is running on:
    ```bash
    kubectl apply -f frontend-deployment.yml
    kubectl get pods -o wide
    ```
    You will see it running only on the worker node with the `nodetype: frontend` label.

### Experiment 2: Local Code Development

1.  Create a file in the local `./v3/app` directory:
    ```bash
    mkdir -p ./v3/app
    echo "Hello from my local machine!" > ./v3/app/index.html
    ```

2.  Create a pod that uses a `hostPath` volume to read from the mounted directory.

    **`local-dev-pod.yml`:**
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: local-dev-test
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: app-volume
          mountPath: /usr/share/nginx/html # Mount into nginx's web root
      volumes:
      - name: app-volume
        hostPath:
          path: /app-data # This path is from inside the node
          type: Directory
    ```

3.  Apply the pod and create a service to access it. Then, you can `curl` the service and see the content from your local file. If you change the local file, the content served by Nginx will change instantly, without rebuilding the container.
