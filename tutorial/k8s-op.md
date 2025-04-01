
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
```
