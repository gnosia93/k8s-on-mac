# k8s-on-mac

### 1. utm install ###

### 2. k8s install ###
```
1. sudo apt-get install -y apt-transport-https ca-certificates curl gpg
2. sudo mkdir -p -m 755 /etc/apt/keyrings
3. curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
4. echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
5. sudo apt-get update
6. sudo apt-get install -y kubelet kubeadm kubectl
```
