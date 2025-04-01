# k8s-on-mac

### 1. UTM 설치 ###
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/utm.png)
#### 1.1. hostname 변경 ####
#### 1.2. 고정 IP 설정 ####
#### 1.3. ssh 서버 설정 #### 
#### 1.4. sudoer 등록 ####
#### 1.5. hosts 파일 수정 ####
#### 1.6. 패스워드 없이 ssh 로그인 ####


### 2. OS 설정 ###

#### 네트워크 설정 ####
```
#/etc/modules-load.d/k8s.conf 파일 생성
sudo cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
 
#/etc/sysctl.d/k8s.conf 파일 생성
sudo cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# /etc/sysctl.conf 파일 주석 제거
net.ipv4.ip_forward = 1                


#시스템 재시작 없이 stysctl 파라미터 반영
sudo sysctl --system
```

#### swap 메모리 제거 ####
```
swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab
sudo swapon -s
```


### [3. containerd 설치](https://tuu-lx.tistory.com/3) ###
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

sudo apt-get install -y apt-transport-https
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

#### cgroup driver(runc) 사용하기 설정 ####
```
vi /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true

sudo systemctl restart containerd
```


### 4. k8s 설치 ###

#### [4.1 kubeadm 설치](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) ####
```
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

#### [4.2 kubeadm 으로 클러스터 생성](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) ####



* 클러스터 초기화
```
sudo kubeadm init 
```
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.64.4
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
```

* static 파드 확인
```
watch sudo crictl ps
```  



* 워커노드 조인
```
kubeadm join 192.168.64.3:6443 --token u64kgq.mtooviynr83s98np \
	--discovery-token-ca-cert-hash sha256:4f5c389944b912b6a8dd9a5ad6b1f61b9cdc41b5ecba4fb7753fcce66c9cf558
```

## 참고자료 ##

* [UTM 고정 IP 할당](https://velog.io/@chosj1526/UTM-ubuntu-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%B6%94%EA%B0%80-%EB%B0%8F-%EA%B3%A0%EC%A0%95-ip-%ED%95%A0%EB%8B%B9%ED%95%98%EA%B8%B0)
* https://velog.io/@khj372/UTM-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0




