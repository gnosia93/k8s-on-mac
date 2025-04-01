
```
kubectl config use-context k8s
kubectl create ns ecommerce
kubectl run eshop-main --image=nginx:1.17 --env=DB=mysql --namespace ecommerce --dry-run=client -o yaml
```
