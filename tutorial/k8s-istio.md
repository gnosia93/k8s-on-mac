
## [시작하기](https://istio.io/latest/docs/setup/getting-started/#download) ##

```
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.25.1
export PATH="$PATH:/home/kube/istio-1.25.1/bin"     # ~/.profile 에 등록

istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
k get all -n istio-system
```
