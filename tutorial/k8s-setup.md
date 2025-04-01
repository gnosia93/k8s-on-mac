# k8s-on-mac


### 1. 인프라 프로비저닝 ###

* UTM 설치 - https://mac.getutm.app/
* Ubuntu 22.04.5 LTS (Jammy Jellyfish) ARM - https://cdimage.ubuntu.com/releases/jammy/release/
* VM 3개 만들기 - control/node01/node02
* sudoer 등록 
* 패스워드 없이 ssh 로그인 

![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/kubeadm-setup.png)


### 2. VM 설정 ###

#### 고정 IP 설정 ####
control 노드는 192.168.64.2/24 으로 node01 노드는 192.168.64.3/24 으로, node02 노드는 192.168.64.4/24 으로 설정한다.
```
$ sudo vi /etc/netplan/00-installer-config.yaml

# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp0s1:
            addresses:
            - 192.168.64.2/24
            routes:
            - to: default
              via: 192.168.64.1
            nameservers:
              addresses:
              - 8.8.8.8
              - 8.8.4.4
    version: 2
```

```
$ sudo netplan apply 
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 3e:10:b4:af:ef:68 brd ff:ff:ff:ff:ff:ff
    inet 192.168.64.3/24 brd 192.168.64.255 scope global enp0s1
       valid_lft forever preferred_lft forever
    inet 192.168.64.6/24 metric 100 brd 192.168.64.255 scope global secondary dynamic enp0s1
       valid_lft 2880sec preferred_lft 2880sec
    inet6 fdca:7282:6d1f:6525:3c10:b4ff:feaf:ef68/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 2591909sec preferred_lft 604709sec
    inet6 fe80::3c10:b4ff:feaf:ef68/64 scope link
       valid_lft forever preferred_lft forever
```


#### 네트워크 파라미터 설정 ####
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


### 3. 컨테이너 런타임 인터페이스(CRI) ###

#### containerd 설치 ####
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

#### cgroup driver(runc) 설정 ####
```
vi /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true

sudo systemctl restart containerd
```


### 4. 쿠버네티스 설치 ###
#### 툴 설치 ####
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

sudo systemctl start kubelet
sudo systemctl enable kubelet
```

#### 클러스터 설정 ####

* 마스터 노드 초기화
```
sudo kubeadm init 

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

curl -O https://raw.githubusercontent.com/gasida/KANS/main/kans3/calico-kans.yaml
kubectl apply -f calico-kans.yaml
```

* 워커노드 조인
```
kubeadm join 192.168.64.2:6443 --token w5xx5j.ll0a847nghlx00q8 \
	--discovery-token-ca-cert-hash sha256:49a3064aea3bded19c6b31a46dcfeaa7c7cebbc204fc1f3b4c9ab9a130fa8920
```

### 5. 설치 확인 ###

![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/kubectl-rs.png)

UTM VM 의 network mode 는 shared 모드로 설정되어 있고, enp0s1 인터페이스에는 두개의 IP 가 할당된다.
* 192.168.64.5/24 는 UTM 이 dhcp로 자동 생성한 IP 이고,
* 192.168.64.2/24 는 [고정 IP 설정] 에서 할당한 IP 로, VM 간의 통신 또는 외부와의 통신에서 사용된다.

![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/ip-addr.png)

## 참고자료 ##

* [UTM 고정 IP 할당](https://velog.io/@chosj1526/UTM-ubuntu-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%B6%94%EA%B0%80-%EB%B0%8F-%EA%B3%A0%EC%A0%95-ip-%ED%95%A0%EB%8B%B9%ED%95%98%EA%B8%B0)
* [UTM 이용해서 쿠버네티스 환경 구성하기](https://velog.io/@khj372/UTM-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0)
* [Calico CNI 설치](https://kschoi728.tistory.com/255)




