# Steps to Verify MongoDB Headless Service in Kubernetes

Follow these steps to verify that your MongoDB headless service is set up correctly in Kubernetes:

## Step 1: Verify the Headless Service

First, ensure that your headless service is created and running. You can use the `kubectl get services` command to list all services and confirm that your headless service is listed:

```sh
kubectl get svc
```

Your headless service should have `None` for the `CLUSTER-IP`, indicating that it's a headless service. For example:

```yaml
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
mongodb         ClusterIP   None         <none>        27017/TCP  10m
```

## Step 2: Verify Endpoints

Check the endpoints for the headless service to ensure that they point to the MongoDB pods:

```sh
kubectl get endpoints mongodb
```

The output should list the IPs of the MongoDB pods. For example:

```yaml
NAME      ENDPOINTS                                      AGE
mongodb   10.244.0.10:27017,10.244.0.11:27017,10.244.0.12:27017   10m
```

## Step 3: DNS Resolution

Use a test pod to check DNS resolution within your Kubernetes cluster. Create a temporary pod with a DNS utility like `busybox` or `debian`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
spec:
  containers:
  - name: dnsutils
    image: gcr.io/kubernetes-e2e-test-images/dnsutils:1.3
    command:
      - sleep
      - "3600"
  restartPolicy: Never
```

Apply this YAML file:

```sh
kubectl apply -f dnsutils.yaml
```

## Step 4: Exec into the Test Pod

Once the pod is running, exec into it to perform DNS lookups:

```sh
kubectl exec -it dnsutils -- sh
```

## Step 5: Perform DNS Lookups

Within the test pod, you can perform DNS lookups to verify that the headless service resolves to the MongoDB pod IPs. Use the `nslookup` command:

```sh
nslookup mongodb
```

You should see output listing the IPs of the MongoDB pods.

## Step 6: Connect to MongoDB

You can also try to connect to one of the MongoDB instances using `mongosh` (MongoDB Shell). If `mongosh` is not available, you can install it or use the `mongo` client.

Assuming you have `mongosh` or `mongo` installed in the `dnsutils` pod, run the following command:

```sh
mongosh mongodb-0.mongodb:27017
```

```

This markdown provides a step-by-step guide to verify your MongoDB headless service in Kubernetes.
