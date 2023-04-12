## Step 1: To be setup in the worker nodes 
```
sudo su
```
```
echo -e '#!/bin/bash\nsudo apt-get update && sudo apt-get upgrade -y\nsudo apt-get install -y docker.io\nsudo systemctl enable docker\nsudo systemctl start docker\nsudo apt-get update && sudo apt-get install -y apt-transport-https curl\ncurl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -\necho "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list\nsudo apt-get update -y\nsudo apt-get install -y kubelet kubeadm kubectl\nsudo apt-mark hold kubelet kubeadm kubectl' > install_kubeadm.sh && chmod +x install_kubeadm.sh && ./install_kubeadm.sh
```
## Step 4: Join the worker node to the cluster (Copy Your Generated Token And Paste On Your Work Node)
1. On the worker node, execute the kubeadm join command you saved earlier. It should look like this:
```
sudo kubeadm join <control-plane-privateip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
