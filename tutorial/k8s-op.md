#### 1. etcd 백업 ####
```
sudo apt install etcd-client
etcdctl --version
etcdctl version: 3.3.25
API version: 2



```

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
* 메인 컨테이너
```
apiVersion: v1
kind: Pod
metadata:
  name: eshop-cart-app
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - 'i=1; while :;do echo -e "$1:" >> /var/log/cart-app.log; i=$((i+1)); sleep 3; done'
    image: busybox
    name: cart-app
    volumeMounts:
    - mountPath: /var/log
      name: varlog
  volumes:
  - emptyDir: {}
    name: varlog

kubectl apply -f eshop-cart-app.yaml
```

* 사이드 카 컨테이너
```
kubectl get pod eshop-cart-app
kubectl get pod eshop-cart-app -o yaml > eshop-cart-app.yaml

vi eshop-cart-app.yaml
metadata:
  name: eshop-cart-app
  namespace: default
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - 'i=1; while :;do echo -e "$1:" >> /var/log/cart-app.log; i=$((i+1)); sleep 3; done'
    image: busybox
    imagePullPolicy: Always
    name: cart-app
    volumeMounts:
    - mountPath: /var/log
      name: varlog
  - name: sidecar
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -F /var/log/cart-app.log']
    volumeMounts:
    - mountPath: /var/log
      name: varlog
  volumes:
  - emptyDir: {}
    name: varlog

kubectl delete pod eshop-cart-app --force 
kubectl apply -f eshop-cart-app.yaml
kubectl logs eshop-cart-app -c sidecar
```

#### 6. deployment & pod scale ####
