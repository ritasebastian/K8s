# Example: Deploying WordPress with MariaDB

You can find all the files we will work with in the [`wordpress`](https://github.com/josedom24/kubernetes/tree/master/ejemplos/wordpress) directory.

We will work in a `namespace` called *wordpress*:

```sh
kubectl create -f wordpress-ns.yaml 
namespace "wordpress" created
```

## Deploying the MariaDB Database

Next, we will create the necessary `secrets` for the database configuration. We will save it in the `mariadb-secret.yaml` file:

```sh
kubectl create secret generic mariadb-secret --namespace=wordpress \
                                --from-literal=dbuser=user_wordpress \
                                --from-literal=dbname=wordpress \
                                --from-literal=dbpassword=password1234 \
                                --from-literal=dbrootpassword=root1234 \
                                -o yaml --dry-run=client > mariadb-secret.yaml
kubectl create -f mariadb-secret.yaml 
secret "mariadb-secret" created
```

We create the service, which will be of type *ClusterIP*:

```sh
kubectl create -f mariadb-srv.yaml 
service "mariadb-service" created
```

And deploy the application:

```sh
kubectl create -f mariadb-deployment.yaml 
deployment.apps "mariadb-deployment" created
```

Check the resources we have created so far:

```sh
kubectl get deploy,service,pods -n wordpress
NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mariadb-deployment         1         1         1            1           20s

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/mariadb-service   ClusterIP   10.98.24.76   <none>        3306/TCP   20s

NAME                                     READY     STATUS    RESTARTS   AGE
pod/mariadb-deployment-844c98579-cgp84   1/1       Running   0          20s
```

## Deploying the WordPress Application

First, we create the service:

```sh
kubectl create -f wordpress-srv.yaml 
service "wordpress-service" created
```

And deploy the application:

```sh
kubectl create -f wordpress-deployment.yaml 
```

Check the created resources:

```sh
kubectl get deploy,service,pods -n wordpress
NAME                                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mariadb-deployment           1         1         1            1           6m
deployment.apps/wordpress-deployment         1         1         1            1           25s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/mariadb-service     ClusterIP   10.98.24.76      <none>        3306/TCP                     6m
service/wordpress-service   NodePort    10.111.158.165   <none>        80:30331/TCP,443:30015/TCP   25s

NAME                                        READY     STATUS    RESTARTS   AGE
pod/mariadb-deployment-844c98579-cgp84      1/1       Running   0          6m
pod/wordpress-deployment-866b7d9fd8-wf5t4   1/1       Running   0          25s
```

Finally, create the `ingress` resource that will allow access to the application using a name:

```sh
kubectl create -f wordpress-ingress.yaml 
ingress.extensions "wordpress-ingress" created

kubectl get ingress -n wordpress
NAME                HOSTS                      ADDRESS   PORTS     AGE
wordpress-ingress   wp.172.22.200.178.nip.io             80        20s
```

And access the application:

![wp](img/wp1.png)

## Issues We Encounter

These are not actually problems, but consequences of the **ephemeral nature of pods**. When a pod is deleted, its information is lost. Therefore, we may encounter some circumstances:

1. What happens if we delete the MariaDB deployment or the MariaDB pod is deleted and a new one is created? In these circumstances, **the database information is lost** and the installation process will start again.
2. What happens if we scale the database deployment and have two pods providing the database? On each access to the application, the database query will be balanced between the two pods (**one that has the installation information and another that has no information**), so on consecutive accesses, the application will alternately show and then prompt for WordPress installation.
3. If we write a post in WordPress and upload an image, that file will be stored in the pod running the application. Therefore, if the pod is deleted, **the static content will be lost**.
4. If we have a pod with static content (e.g., images) and scale the WordPress deployment to two pods, **one will contain the image, but the other will not**. Therefore, on consecutive accesses to the application, the image will be alternately shown or not shown depending on which pod responds.

To solve these problems, we will explore using volumes in Kubernetes in the following sections.
```
