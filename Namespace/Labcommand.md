# LAB Qestions

## How to check the Namespace 
```
kubectl get namespaces
kubectl  get ns
```

## How to create Namespace with Yaml file?

```
cat > development.yaml <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: production
EOF
 ```
```
** kubectl create ns operation**
 ```
## How to check the detailed information about the Namespace?

```
kubectl describe ns development
```
```
kubectl get ns development --show-labels
```

## How to create Resource Quota for Namespace?
```
cat > resourcequota.yaml <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
    name: pods-low
    namespace: operation
spec:
    hard:
      cpu: "5"
      memory: 10Gi
      pods: "2"
EOF


```
```
kubectl describe quota -n operation
kubectl -n operation describe ResourceQuota pods-low
```
 
 ### Pod creation through Yaml file
 
**Pod1**
```
cat > pod-quota1.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-quota1
  namespace: operation
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "500Mi"
        cpu: "500m"
      limits:
        memory: "500Mi"
        cpu: "500m"
EOF
 ```
   

 
**Pod2**
```
cat > pod-quota2.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-quota2
  namespace: operation
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "500Mi"
        cpu: "500m"
      limits:
        memory: "500Mi"
        cpu: "500m"
EOF

 ```
 
**Pod3**
```
cat > pod-quota3.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-quota2
  namespace: operation
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "500Mi"
        cpu: "500m"
      limits:
        memory: "500Mi"
        cpu: "500m"
EOF

 ```
 
```
kubectl  get pod
kubectl  get pod -n operation
```
## How to set the namespace default?
```
Kubectl config set-context --current --namespace=operation
```
