
## [시작하기](https://istio.io/latest/docs/setup/getting-started/#download) ##

```
ssh kube@control

curl -L https://istio.io/downloadIstio | sh -
cd istio-1.25.1
export PATH="$PATH:/home/kube/istio-1.25.1/bin"     # ~/.profile 에 등록

istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
k get all -n istio-system
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/istio-demo-profile-1.png)

```
k kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.2.1" | kubectl apply -f -
k apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/istio-demo-profile-2.png)

```
kubectl apply -f samples/bookinfo/gateway-api/bookinfo-gateway.yaml
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/istio-demo-profile-3.png)
```
kubectl annotate gateway bookinfo-gateway networking.istio.io/service-type=ClusterIP --namespace=default
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/istio-demo-profile-4.png)

```
kubectl port-forward svc/bookinfo-gateway-istio 8080:80
```
