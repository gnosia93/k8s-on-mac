# k8s-on-mac

### 1. UTM 설치 ###
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/utm.png)
#### 1.1. hostname 변경 ####
#### 1.2. 고정 IP 설정 ####
#### 1.3. ssh 서버 설정 #### 
#### 1.4. sudoer 등록 ####
#### 1.5. hosts 파일 수정 ####
#### 1.6. 패스워드 없이 ssh 로그인 ####


### [2. containerd 설치](https://tuu-lx.tistory.com/3) ###
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install containerd
sudo apt-get install containerd.io
systemctl status containerd
sudo systemctl start containerd
sudo systemctl enable containerd
```


### 3. swap 메모리 제거 ###
```
# 파일시스템 설정
sudo vi /etc/fstab 

# 마지막 행에 주석처리
#/swap.img      none    swap    sw      0       0


# 리부팅 후 메모리 확인
$ free
               total        used        free      shared  buff/cache   available
Mem:         3996640      322084     3485432        5328      338392     3674556
Swap:              0           0           0
```



### 4. k8s 설치 ###

#### [4.1 kubeadm 설치](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) ####
```
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

#### [4.2 kubeadm 으로 클러스터 생성](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) ####

* root 유저로 아래 명령어 실행
```
sudo su
echo "1" > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward

vi /etc/containerd/config.toml
disabled_plugins = ["cri"]

systemctl restart containerd
cd; kubeadm init
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/kubeadm-control.png)

* 일반 유저 (eg. kube) 실행
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

* [Weave CNI 설치](https://github.com/rajch/weave#using-weave-on-kubernetes)
```
kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.31/net.yaml

kubeadm join 192.168.64.3:6443 --token u64kgq.mtooviynr83s98np \
	--discovery-token-ca-cert-hash sha256:4f5c389944b912b6a8dd9a5ad6b1f61b9cdc41b5ecba4fb7753fcce66c9cf558
```


## 참고자료 ##

* https://blog.psnote.co.kr/202
* https://medium.com/finda-tech/overview-8d169b2a54ff 




