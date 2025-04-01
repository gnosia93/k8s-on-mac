
## [시작하기](https://istio.io/latest/docs/setup/getting-started/#download) ##

### mac kubeconfig 설정 ###
mac 컴퓨터의 ~/.kube/config 파일에 UTM 에 생성한 k8s 클러스터 정보를 추가한다 (cluster, context, user) 
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/k8s-config.png)

### istio 설치 ###
마스터인 control 노드에서 아래 명령어를 수행한다. 
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

### 웹 서비스 조회 ###
mac 컴퓨터에서 아래 명령어를 수행한다. 
```
# gcp-context 로 변경
(base) automake@mini ~ % kubectl config use-context kubernetes-admin@kubernetes

# context 조회
(base) automake@mini ~ % kubectl config get-contexts
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/k8s-context.png)

```
# port forwarding 설정
(base) automake@mini ~ % kubectl port-forward svc/bookinfo-gateway-istio 8080:80
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/istio-productpage.png)

### 모니터링 addon 설치 ###
```
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system
```

