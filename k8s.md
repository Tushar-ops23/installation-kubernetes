## Kubernetes Installation Script

This is a bash script for installing Kubernetes on a Linux system. It performs the following tasks:

1. Checks if the script is run with root privileges using the `$EUID` environment variable.
2. Updates the system using `apt update` and `apt upgrade` commands.
3. Installs required dependencies including `apt-transport-https`, `ca-certificates`, `curl`, and `software-properties-common`.
4. Adds the Kubernetes repository using `curl` to download the GPG key and `add-apt-repository` to add the repository.
5. Installs Docker using `apt install docker.io`.
6. Enables and starts the Docker service using `systemctl enable docker` and `systemctl start docker` commands.
7. Installs Kubernetes components including `kubeadm`, `kubelet`, and `kubectl` using `apt install`.
8. Initializes the Kubernetes master node using `kubeadm init`.
9. Configures `kubectl` for the current user by creating the required directory and copying the admin.conf file.
10. Installs a network plugin (Calico in this example) using `kubectl apply` command.
11. Prints the join command for worker nodes to join the cluster using `kubeadm token create --print-join-command` command.
12. Prints the status of the Kubernetes cluster using `kubectl cluster-info` command.

After the installation, the script applies the Calico network plugin using `kubectl apply` command with the provided manifest URL, and then retrieves the pods in the `kube-system` namespace that are related to Calico using `kubectl get pods -n kube-system | grep calico` command.

*Note: Please ensure that the script is run with root privileges.*

### Usage
1. Copy the script to a Linux system.
2. Run the script with root privileges using `sudo` or by logging in as the root user.
3. Follow the instructions provided by the script for joining worker nodes to the cluster.
4. After the installation, you can check the status of the Kubernetes cluster by running `kubectl cluster-info` command.

```bash
#!/bin/bash

# Check if the script is run with root privileges
if [[ $EUID -ne 0 ]]; then
  echo "This script must be run with root privileges."
  exit 1
fi

# Update the system
apt update -y && apt upgrade -y

# Install required dependencies
apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Kubernetes repository
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"

# Install Docker
apt install -y docker.io

# Enable and start Docker service
systemctl enable docker
systemctl start docker

# Install Kubernetes components
apt update -y
apt install -y kubeadm kubelet kubectl

# Initialize Kubernetes master node
kubeadm init

# Configure kubectl for the current user
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# Install network plugin (example: Calico)
kubectl apply -f https://docs.projectcalico.org/v3.19/manifests/calico.yaml

# Print join command for worker nodes
echo "Run the following command on worker nodes to join the cluster:"
kubeadm token create --print-join-command

# Print status of the Kubernetes cluster

# After installation run this command on master plane
kubectl apply -f https://docs.projectcalico.org/v3.19/manifests/calico.yaml
kubectl get pods -n kube-system | grep calico
```
## Error Handling
> **Note**
> The connection to the server 172.31.2.185:6443 was refused - did you specify the right host or port?
> Make sure your ports is open if your ports already open then use this steps

#### Your Facing like this error then run this command 
```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

