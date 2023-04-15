## To be setup in the worker nodes 
```
#!/bin/bash

# Take input for k8smaster private IP
read -p "Enter private IP for k8smaster: " master_private_ip

# Take input for k8sworker1 private IP
read -p "Enter private IP for k8sworker1: " worker1_private_ip

# Set hostname for k8smaster
sudo hostnamectl set-hostname "k8smaster"

# Set hostname for k8sworker1
sudo hostnamectl set-hostname "k8sworker1"

# Update /etc/hosts with hostname and private IP entries
echo "$master_private_ip k8smaster" | sudo tee -a /etc/hosts
echo "$worker1_private_ip k8sworker1" | sudo tee -a /etc/hosts

echo -e '#!/bin/bash\nsudo apt-get update && sudo apt-get upgrade -y\nsudo apt-get install -y docker.io\nsudo systemctl enable docker\nsudo systemctl start docker\nsudo apt-get update && sudo apt-get install -y apt-transport-https curl\ncurl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -\necho "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list\nsudo apt-get update -y\nsudo apt-get install -y kubelet kubeadm kubectl\nsudo apt-mark hold kubelet kubeadm kubectl' > install_kubeadm.sh && chmod +x install_kubeadm.sh && ./install_kubeadm.sh
```
# Then paste the token
