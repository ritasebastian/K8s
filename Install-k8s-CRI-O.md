### Distributions using `rpm` packages

#### Define the Kubernetes version and used CRI-O stream

```bash
KUBERNETES_VERSION=v1.30
PROJECT_PATH=prerelease:/main
```

#### Add the Kubernetes repository

```bash
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/repodata/repomd.xml.key
EOF
```

#### Add the CRI-O repository

```bash
cat <<EOF | tee /etc/yum.repos.d/cri-o.repo
[cri-o]
name=CRI-O
baseurl=https://pkgs.k8s.io/addons:/cri-o:/$PROJECT_PATH/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/addons:/cri-o:/$PROJECT_PATH/rpm/repodata/repomd.xml.key
EOF
```

#### Install package dependencies from the official repositories

```bash
dnf install -y container-selinux
```

#### Install the packages

```bash
dnf install -y cri-o kubelet kubeadm kubectl
```

#### Start CRI-O

```bash
systemctl start crio.service
```

#### Bootstrap a cluster

```bash
swapoff -a
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1

kubeadm init
```

### Distributions using `deb` packages

#### Install the dependencies for adding repositories

```bash
apt-get update
apt-get install -y software-properties-common curl
```

#### Define the Kubernetes version and used CRI-O stream

```bash
KUBERNETES_VERSION=v1.30
PROJECT_PATH=prerelease:/main
```

#### Add the Kubernetes repository

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
```

#### Add the CRI-O repository

```bash
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/$PROJECT_PATH/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/$PROJECT_PATH/deb/ /" |
    tee /etc/apt/sources.list.d/cri-o.list
```

#### Install the packages

```bash
apt-get update
apt-get install -y cri-o kubelet kubeadm kubectl
```

#### Start CRI-O

```bash
systemctl start crio.service
```

#### Bootstrap a cluster

```bash
swapoff -a
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1

kubeadm init
```

