Here is an example of deploying an Nginx pod in Kubernetes and exposing it via a NodePort service. This will allow you to access the Nginx server using a port on any node in the cluster.

### Step 1: Create the Nginx Pod

First, create a YAML file for the Nginx pod.

# nginx-pod.yaml

```
cat <<EOF>> nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
EOF
```

Apply the pod configuration:

```sh
kubectl apply -f nginx-pod.yaml
```

### Step 2: Create the NodePort Service

Next, create a YAML file for the NodePort service.

**nginx-service.yaml**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080  # You can specify a port in the range 30000-32767, or let Kubernetes choose one for you.
```

Apply the service configuration:

```sh
kubectl apply -f nginx-service.yaml
```

### Step 3: Verify the Deployment

Check the status of the pod:

```sh
kubectl get pods
```

Check the status of the service:

```sh
kubectl get services
```

You should see an output similar to:

```sh
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort    10.104.103.206   <none>        80:30080/TCP   1m
```

### Step 4: Access the Nginx Server

Now you can access the Nginx server using the external IP address of any node in the cluster and the node port (30080 in this example).

Get the external IP address of a node:

```sh
kubectl get nodes -o wide
```

In your web browser or using `curl`, access the Nginx server:

```sh
curl http://<node-external-ip>:30080
```

Replace `<node-external-ip>` with the actual external IP address of your node.

### Summary

- You created an Nginx pod.
- You exposed the Nginx pod using a NodePort service.
- You accessed the Nginx server through the external IP address of a node and the specified node port.

This setup allows you to expose a Kubernetes service to the external network using a specific port on each node in the cluster.
