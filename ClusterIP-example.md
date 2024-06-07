To create a MongoDB pod and a ClusterIP service in Kubernetes and then test it, you can follow these steps:

1. **Create the YAML file**: Use a text editor to create a YAML file with the MongoDB pod and the ClusterIP service configurations.

2. **Apply the YAML file**: Use `kubectl apply` to deploy the MongoDB pod and create the ClusterIP service.

3. **Test the MongoDB service**: Use another pod in the same namespace to test the connectivity to the MongoDB service using the ClusterIP.

Here's a detailed guide:

1. **Create the YAML file**:

Create a file named `mongodb-clusterip.yaml` and add the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
  labels:
    app: mongodb
spec:
  containers:
  - name: mongodb
    image: mongo:latest
    ports:
    - containerPort: 27017

---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  selector:
    app: mongodb
  ports:
  - protocol: TCP
    port: 27017
```

This YAML file defines a MongoDB pod and a ClusterIP service named `mongodb`.

2. **Apply the YAML file**:

Run the following command to apply the YAML file and deploy the MongoDB pod and create the ClusterIP service:

```bash
kubectl apply -f mongodb-clusterip.yaml
```

3. **Test the MongoDB service**:

To test the MongoDB service, you can create another pod in the same namespace and use the `mongo` client to connect to the MongoDB service using the ClusterIP.

Create a temporary pod for testing:

```bash
kubectl run -i --tty temp-pod --image=mongo:latest --restart=Never -- bash
```

Once inside the temporary pod, use the `mongo` client to connect to the MongoDB service:

```bash
mongo mongodb://mongodb:27017
```

This command connects to the MongoDB service using the ClusterIP (`mongodb`) and the default MongoDB port (`27017`). You should see the MongoDB shell prompt if the connection is successful.

That's it! You've now deployed a MongoDB pod and a ClusterIP service in Kubernetes and tested the connectivity to the MongoDB service.
