Upgrading a Kubernetes cluster involves several steps to ensure a smooth transition without downtime. Below are the steps to upgrade the control plane and node components on the master node from version 1.30.0 to 1.30.2, including draining and uncordoning the master node and upgrading kubelet and kubectl.

https://killercoda.com
https://kubernetes.io/docs/home/

### Example with Node Name
Let's assume the master node name is `controlplane`.

1. **Check Current Version**
   ```sh
   kubectl version
   ```

2. **Drain the Master Node**
   ```sh
   kubectl drain controlplane --ignore-daemonsets --delete-emptydir-data
   ```

3. **Upgrade kubeadm**
   ```sh
   sudo apt update
   sudo apt-cache madison kubeadm
   sudo apt-mark unhold kubeadm && \
   sudo apt-get update && sudo apt-get install -y kubeadm='1.30.2-*' && \
   sudo apt-mark hold kubeadm
   ```

4. **Verify the upgrade plan**
   ```sh
   sudo kubeadm upgrade plan
   ```

5. **Perform the Upgrade**
   ```sh
   sudo kubeadm upgrade apply v1.30.2
   ```

6. **Upgrade kubele and kubectl **
   ```sh
   sudo apt-mark unhold kubelet kubectl && \
   sudo apt-get update && sudo apt-get install -y kubelet='1.30.2-*' kubectl='1.30.2-*' && \
   sudo apt-mark hold kubelet kubectl
   ```
7. **Restart the kubelet **
   ```sh
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```
8. **Uncordon the Master Node**
   ```sh
   kubectl uncordon controlplane
   ```

9. **Verify the Upgrade**
   ```sh
   kubectl get nodes
   kubectl version
   ```

These steps will ensure that your Kubernetes control plane and node components on the master node are upgraded smoothly from version 1.30.0 to 1.30.1.
