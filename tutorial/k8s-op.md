#### 1. etcd 백업 ####


#### 2. pod 생성하기 ####
```
kubectl config use-context k8s
kubectl create ns ecommerce
kubectl run eshop-main --image=nginx:1.17 --env=DB=mysql -n ecommerce --dry-run=client -o yaml
```


#### 3. static pod 생성하기 ####
```
ssh kube@node01
cat /var/lib/kubelet/config.yaml | grep staticPodPath
staticPodPath: /etc/kubernetes/manifests

cd /etc/kubernetes/manifests/

sudo vi nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx

ssh kube@control
kubectl get pod -o wide
NAME           READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
nginx-node01   1/1     Running   0          2m19s   172.16.220.3   node01   <none>           <none>
```

#### 4. 멀티 컨테이너 pod 생성하기 ####
```
kubectl run multi --image=nginx --dry-run=client -o yaml > multi.yaml

vi multi.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi
  name: multi
spec:
  containers:
  - image: nginx
    name: multi
  - image: redis
    name: redis
  - image: memcached
    name: memcached

kubectl get pod
NAME           READY   STATUS    RESTARTS   AGE
multi          3/3     Running   0          62s
```

#### 5. 사이드카 컨테이너 ####
