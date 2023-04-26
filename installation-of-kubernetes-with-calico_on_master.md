# Kubernetes Setup Script
#!/bin/bash

# Get user input for k8smaster hostname and private IP address
read -p "Enter hostname for k8smaster: " k8smaster_hostname
read -p "Enter private IP address for k8smaster: " k8smaster_private_ip

# Get user input for k8sworker1 hostname and private IP address
read -p "Enter hostname for k8sworker1: " k8sworker1_hostname
read -p "Enter private IP address for k8sworker1: " k8sworker1_private_ip

# Set hostname for k8smaster
sudo hostnamectl set-hostname "$k8smaster_hostname"

# Set hostname for k8sworker1
sudo hostnamectl set-hostname "$k8sworker1_hostname"

# Update /etc/hosts with hostname and private IP entries
echo "$k8smaster_private_ip $k8smaster_hostname" | sudo tee /etc/hosts
sudo sed -i "/$k8sworker1_hostname/d" /etc/hosts
echo "$k8sworker1_private_ip $k8sworker1_hostname" | sudo tee -a /etc/hosts


# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Configure containerd
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl for Kubernetes
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

# Install Docker
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Install Kubernetes components
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/kubernetes-xenial.gpg
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Initialize Kubernetes control plane
sudo kubeadm init --control-plane-endpoint=k8smaster
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify cluster status
kubectl cluster-info
kubectl get nodes

# Install Calico network plugin
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
kubectl get pods -n kube-system
kubectl get nodes
