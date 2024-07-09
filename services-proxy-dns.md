To demonstrate the use of different types of Kubernetes services (LoadBalancer, ClusterIP, NodePort) with multiple pods applying different services, and discussing their respective use cases and drawbacks.

### Step-by-Step Guide [start minikube tunnel and minikube service service-name --url if needed enable metalib)
1. **Create Kubernetes Deployment YAML**

   ```bash
   cat <<EOF > nginx-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
   spec:
     replicas: 1
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
           image: sasebastian/my-nginx-image:latest
           ports:
           - containerPort: 80
   EOF
   ```

2. **Create Kubernetes Services YAML**

   **NodePort Service:**

   ```bash
   cat <<EOF > nodeport-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-nodeport
   spec:
     selector:
       app: nginx
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
     type: NodePort
   EOF
   ```

   **ClusterIP Service:**

   ```bash
   cat <<EOF > clusterip-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-clusterip
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

   **LoadBalancer Service:**

   ```bash
   cat <<EOF > loadbalancer-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-loadbalancer
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

3. **Apply YAML Files to Kubernetes Cluster**

   ```bash
   kubectl apply -f nginx-deployment.yaml
   kubectl apply -f nodeport-service.yaml
   kubectl apply -f clusterip-service.yaml
   kubectl apply -f loadbalancer-service.yaml
   ```

4. **Test with `curl -I`, `iptables`, and `dig`**

   - **NodePort Service:**

     ```bash
     # Get NodePort assigned
     NODE_PORT=$(kubectl get svc nginx-nodeport -o jsonpath='{.spec.ports[0].nodePort}')

     # Test with curl
     curl -I localhost:$NODE_PORT

     # Test with iptables
     sudo iptables -A INPUT -p tcp --dport $NODE_PORT -j DROP
     ```

   - **ClusterIP Service:**

     ```bash
     # Get ClusterIP assigned
     CLUSTER_IP=$(kubectl get svc nginx-clusterip -o jsonpath='{.spec.clusterIP}')

     # Test with curl (assuming app runs on a pod within the cluster)
     curl -I $CLUSTER_IP:80

     # Test with dig
     dig $CLUSTER_IP
     ```

   - **LoadBalancer Service:**

     ```bash
     # Get LoadBalancer IP (assuming it's provisioned)
     LB_IP=$(kubectl get svc nginx-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

     # Test with curl
     curl -I $LB_IP

     # Test with dig
     dig $LB_IP
     ```

### Pros and Cons of Each Service Type

- **NodePort:**
  - **Pros:**
    - Exposes the service on a static port across the nodes.
    - Allows external access to the service directly via node IP and NodePort.
  - **Cons:**
    - Not suitable for production load balancing.
    - Exposes a port on every node, which might not be desired for security reasons.

- **ClusterIP:**
  - **Pros:**
    - Provides internal cluster-only IP, not exposed outside of the cluster.
    - Ideal for communication between different services within the cluster.
  - **Cons:**
    - Not accessible externally without additional configuration (like using an Ingress).

- **LoadBalancer:**
  - **Pros:**
    - Automatically provisions an external load balancer (if supported by cloud provider) and assigns an external IP.
    - Provides an easy way to expose services to external users.
  - **Cons:**
    - Costly and might not be available in all environments (e.g., local development clusters).
    - May have slower provisioning times compared to other service types.

These steps will help you create and test different Kubernetes service types for your `sasebastian/my-nginx-image:latest` deployment effectively.
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

To validate the functionality of Kubernetes services (ClusterIP, NodePort, LoadBalancer) using `iptables`, we'll inspect the `iptables` rules that Kubernetes sets up to understand how traffic is routed within the cluster.

### Validating Kubernetes Services with `iptables`

#### 1. ClusterIP Service

When you create a ClusterIP service, Kubernetes sets up `iptables` rules to route traffic to the service's ClusterIP. Here’s how you can validate it:

1. **List `iptables` rules related to ClusterIP service:**

   ```sh
   sudo iptables -t nat -L KUBE-SERVICES -n
   ```

   This command lists the `iptables` rules in the `KUBE-SERVICES` chain, which includes rules for each ClusterIP service in your namespace. Look for rules that direct traffic to the ClusterIP of your service.

2. **Inspect specific `iptables` rules for the service:**

   Replace `my-service-clusterip` with your actual service name and `my-namespace` with your namespace:

   ```sh
   sudo iptables -t nat -L KUBE-SVC-MY-SERVICE-CLUSTERIP -n
   ```

   This command lists the specific `iptables` rules in the `KUBE-SVC-MY-SERVICE-CLUSTERIP` chain, which routes traffic to the endpoints (pods) associated with your ClusterIP service.

#### 2. NodePort Service

For a NodePort service, `iptables` rules are set up to forward traffic from the specified NodePort on each node to the ClusterIP of the service. Here’s how to validate it:

1. **List `iptables` rules related to NodePort services:**

   ```sh
   sudo iptables -t nat -L KUBE-NODEPORTS -n
   ```

   This command lists the `iptables` rules in the `KUBE-NODEPORTS` chain, which include rules for each NodePort service in your namespace. Look for rules that map the NodePort to the ClusterIP.

2. **Inspect specific `iptables` rules for the NodePort service:**

   Replace `<node-port>` with the NodePort assigned to your service:

   ```sh
   sudo iptables -t nat -L KUBE-SVC-MY-SERVICE-NODEPORT -n
   ```

   This command lists the specific `iptables` rules in the `KUBE-SVC-MY-SERVICE-NODEPORT` chain, which forwards traffic from the NodePort to the ClusterIP of your service.

#### 3. LoadBalancer Service

LoadBalancer services interact with `iptables` differently depending on the cloud provider. Typically, `iptables` rules may be set up to forward traffic from the external load balancer to the nodes hosting the service. Here’s how to validate it:

1. **List `iptables` rules related to LoadBalancer services:**

   LoadBalancer services often rely on cloud provider integration for `iptables` setup. You can inspect general `iptables` rules, but specifics may vary based on the provider.

   ```sh
   sudo iptables -t nat -L -n
   ```

   This command lists all `iptables` rules in the NAT table, including any rules related to external traffic forwarding.

2. **Inspect specific `iptables` rules for the LoadBalancer service:**

   Depending on your cloud provider, specific `iptables` rules might be implemented differently. Consult your cloud provider's documentation for detailed insights into how traffic is managed.

### Summary

Validating Kubernetes services using `iptables` helps ensure that traffic is correctly routed within your cluster. By inspecting these rules, you gain visibility into how Kubernetes manages networking for ClusterIP, NodePort, and LoadBalancer services. 




