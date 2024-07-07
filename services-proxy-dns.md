To demonstrate the use of different types of Kubernetes services (LoadBalancer, ClusterIP, NodePort) with multiple pods, let's go through the steps of creating two pods, applying different services, and discussing their respective use cases and drawbacks.

### Step-by-Step Guide

#### 1. Create Namespace

Let's start by creating a namespace for our resources:

```sh
kubectl create namespace my-namespace
```

#### 2. Create Pod YAMLs

Create two Pod YAML files (`pod1.yaml` and `pod2.yaml`) for NGINX pods using `echo`:

```sh
# pod1.yaml
cat <<EOF > pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: my-namespace
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
EOF
```

```sh
# pod2.yaml
cat <<EOF > pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: my-namespace
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
EOF
```

Apply the Pod configurations:

```sh
kubectl apply -f pod1.yaml
kubectl apply -f pod2.yaml
```

#### 3. Create Services YAMLs

##### a. ClusterIP Service

Create a ClusterIP service (`service-clusterip.yaml`) using `echo`:

```sh
cat <<EOF > service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-clusterip
  namespace: my-namespace
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
EOF
```

Apply the ClusterIP service:

```sh
kubectl apply -f service-clusterip.yaml
```

##### b. NodePort Service

Create a NodePort service (`service-nodeport.yaml`) using `echo`:

```sh
cat <<EOF > service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-nodeport
  namespace: my-namespace
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080  # Specify a specific port or let Kubernetes allocate one
  type: NodePort
EOF
```

Apply the NodePort service:

```sh
kubectl apply -f service-nodeport.yaml
```

##### c. LoadBalancer Service

Create a LoadBalancer service (`service-loadbalancer.yaml`) using `echo`:

```sh
cat <<EOF > service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-loadbalancer
  namespace: my-namespace
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
EOF
```

Apply the LoadBalancer service:

```sh
kubectl apply -f service-loadbalancer.yaml
```

#### 4. Verify Services and Endpoints

Check services and endpoints:

```sh
kubectl get services -n my-namespace
kubectl get endpoints -n my-namespace
```

Note down the ClusterIP, NodePort, and external IP (for LoadBalancer service).

#### 5. Discuss Use Cases and Drawbacks

##### a. ClusterIP Service

- **Use Case:** Internal communication between microservices within the same namespace.
- **Drawbacks:** Not accessible outside the cluster without additional configurations (e.g., Ingress or LoadBalancer).

##### b. NodePort Service

- **Use Case:** Accessing applications from outside the Kubernetes cluster directly via a specific port on each node.
- **Drawbacks:** Limited by the port range (30000-32767) and potential security concerns if not properly configured.

##### c. LoadBalancer Service

- **Use Case:** Providing external access to applications via a cloud provider's load balancer.
- **Drawbacks:** Relies on cloud provider integration, may incur additional costs, and might be slower to provision compared to NodePort or ClusterIP.

Certainly! After creating the Kubernetes services (ClusterIP, NodePort, LoadBalancer) and deploying the NGINX pods as previously outlined, let's proceed to test each service using `curl`.

### Testing Each Service with `curl`

#### 1. ClusterIP Service

To test the ClusterIP service:

1. Open a shell session inside a test pod within the same namespace:

```sh
kubectl run test-pod --rm -it --image=alpine --namespace=my-namespace -- sh
```

2. Install `curl` if it's not already installed:

```sh
apk add curl
```

3. Use `curl` to access the ClusterIP service:

```sh
curl http://my-service-clusterip.my-namespace.svc.cluster.local
```

Replace `my-service-clusterip` with your actual service name and `my-namespace` with your namespace.

Example:
```sh
curl http://my-service-clusterip.my-namespace.svc.cluster.local
```

You should receive the default NGINX welcome page or the expected response from your service.

#### 2. NodePort Service

To test the NodePort service:

1. Obtain the IP address of any Kubernetes node (`<node-ip>`).

2. Use `curl` to access the NodePort service using the NodePort (`<node-port>`) you specified:

```sh
curl http://<node-ip>:<node-port>
```

Replace `<node-ip>` and `<node-port>` with the actual IP address of a node in your cluster and the NodePort assigned to your service.

Example:
```sh
curl http://192.168.1.100:30080
```

You should receive the default NGINX welcome page or the expected response from your service.

#### 3. LoadBalancer Service

To test the LoadBalancer service:

1. Check the external IP assigned to the LoadBalancer service:

```sh
kubectl get services -n my-namespace
```

2. Use `curl` to access the service using the external IP:

```sh
curl http://<external-ip>
```

Replace `<external-ip>` with the external IP address assigned to your LoadBalancer service.

Example:
```sh
curl http://123.45.67.89
```

You should receive the default NGINX welcome page or the expected response from your service.

### Summary

Testing each Kubernetes service type (ClusterIP, NodePort, LoadBalancer) using `curl` ensures that they are correctly configured and accessible within your Kubernetes cluster environment. This approach helps verify connectivity and functionality, ensuring that your applications are reachable and responsive according to their intended deployment and networking configurations. Adjust testing commands and configurations as needed based on your specific deployment requirements and service definitions.
