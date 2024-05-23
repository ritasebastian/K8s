
# Setting Up MongoDB Replica Set on Kubernetes

This guide will walk you through the steps to set up a MongoDB replica set on Kubernetes using a StatefulSet and a headless service.

## 1. Headless Service Configuration

Create a file named `headless-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    app: mongo
spec:
  ports:
  - name: mongo
    port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    app: mongo
```

## 2. StatefulSet Configuration

Create a file named `mongodb-stateful.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: mongo
        image: mongo
        command:
        - mongod
        - "--bind_ip_all"
        - "--replSet"
        - rs0
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-volume
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongo-volume
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## 3. Apply Configurations

Apply the headless service and StatefulSet:

```sh
kubectl apply -f headless-service.yaml
kubectl apply -f mongodb-stateful.yaml
```

## 4. Initialize the Replica Set

1. **Exec into the first MongoDB pod:**

    ```sh
    kubectl exec -it mongo-0 -- mongosh
    ```

2. **Initialize the replica set:**

    ```javascript
    rs.initiate()
    ```

3. **Reconfigure the replica set:**

    ```javascript
    var cfg = rs.conf()
    cfg.members[0].host = "mongo-0.mongo:27017"
    rs.reconfig(cfg)
    ```

4. **Add the other members:**

    ```javascript
    rs.add("mongo-1.mongo:27017")
    rs.add("mongo-2.mongo:27017")
    ```

## 5. Expose the MongoDB Pods

Expose each MongoDB pod:

```sh
kubectl expose pod mongo-0 --port=27017 --target-port=27017 --type=LoadBalancer
kubectl expose pod mongo-1 --port=27017 --target-port=27017 --type=LoadBalancer
kubectl expose pod mongo-2 --port=27017 --target-port=27017 --type=LoadBalancer
```

## 6. Verify Connectivity

1. **Get the service URL for each pod:**

    ```sh
    minikube service mongo-0 --url
    minikube service mongo-1 --url
    minikube service mongo-2 --url
    ```

2. **Test connectivity:**

    Use `curl` or `telnet` to test the connection to each service URL:

    ```sh
    curl -vv telnet://<service-url>:27017
    ```

## 7. Check Replica Set Status

After initializing and reconfiguring the replica set, verify the status again:

```sh
kubectl exec -it mongo-0 -- mongosh --eval "rs.status()"
```

## Troubleshooting Tips

- **Pod Logs:** Check logs if there are any issues with pods not starting correctly:

    ```sh
    kubectl logs mongo-0
    kubectl logs mongo-1
    kubectl logs mongo-2
    ```

- **Network Policies:** Ensure there are no network policies blocking communication between the pods.

- **DNS Resolution:** Ensure that the DNS within the cluster is resolving correctly. You can use a utility pod like `dnsutils` or `busybox` to test DNS resolution.

By following these steps carefully, you should be able to set up and verify a MongoDB replica set in Kubernetes, ensuring that all pods are correctly communicating and that the replica set is functioning as expected.
```

