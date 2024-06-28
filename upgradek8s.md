Upgrading a Kubernetes cluster involves several steps to ensure a smooth transition without downtime. Below are the steps to upgrade the control plane and node components on the master node from version 1.20.0 to 1.20.1, including draining and uncordoning the master node and upgrading kubelet and kubectl.

1. **Check Current Version**
   Ensure that the current version is 1.20.0.
   ```sh
   kubectl version --short
   ```

2. **Drain the Master Node**
   Drain the master node to safely evict all running pods.
   ```sh
   kubectl drain <master-node-name> --ignore-daemonsets --delete-emptydir-data
   ```

3. **Upgrade kubeadm**
   Upgrade `kubeadm` to version 1.20.1.
   ```sh
   sudo apt-get update
   sudo apt-get install -y kubeadm=1.20.1-00
   ```

4. **Verify the upgrade plan**
   Verify the upgrade plan with `kubeadm`.
   ```sh
   sudo kubeadm upgrade plan
   ```

5. **Perform the Upgrade**
   Perform the upgrade using `kubeadm upgrade apply`.
   ```sh
   sudo kubeadm upgrade apply v1.20.1
   ```

6. **Upgrade kubelet**
   Upgrade the `kubelet` package to version 1.20.1.
   ```sh
   sudo apt-get install -y kubelet=1.20.1-00
   sudo systemctl restart kubelet
   ```

7. **Upgrade kubectl**
   Upgrade the `kubectl` package to version 1.20.1.
   ```sh
   sudo apt-get install -y kubectl=1.20.1-00
   ```

8. **Uncordon the Master Node**
   After the upgrade, uncordon the master node to make it schedulable again.
   ```sh
   kubectl uncordon <master-node-name>
   ```

9. **Verify the Upgrade**
   Finally, verify that the upgrade was successful.
   ```sh
   kubectl get nodes
   kubectl version --short
   ```

### Example with Node Name
Let's assume the master node name is `master-node-1`.

1. **Check Current Version**
   ```sh
   kubectl version --short
   ```

2. **Drain the Master Node**
   ```sh
   kubectl drain master-node-1 --ignore-daemonsets --delete-emptydir-data
   ```

3. **Upgrade kubeadm**
   ```sh
   sudo apt-get update
   sudo apt-get install -y kubeadm=1.20.1-00
   ```

4. **Verify the upgrade plan**
   ```sh
   sudo kubeadm upgrade plan
   ```

5. **Perform the Upgrade**
   ```sh
   sudo kubeadm upgrade apply v1.20.1
   ```

6. **Upgrade kubelet**
   ```sh
   sudo apt-get install -y kubelet=1.20.1-00
   sudo systemctl restart kubelet
   ```

7. **Upgrade kubectl**
   ```sh
   sudo apt-get install -y kubectl=1.20.1-00
   ```

8. **Uncordon the Master Node**
   ```sh
   kubectl uncordon master-node-1
   ```

9. **Verify the Upgrade**
   ```sh
   kubectl get nodes
   kubectl version --short
   ```

These steps will ensure that your Kubernetes control plane and node components on the master node are upgraded smoothly from version 1.20.0 to 1.20.1.
