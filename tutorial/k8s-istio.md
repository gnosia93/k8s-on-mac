
## [시작하기](https://istio.io/latest/docs/setup/getting-started/#download) ##

```
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.25.1
export PATH="$PATH:/home/kube/istio-1.25.1/bin"     # ~/.profile 에 등록

istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
k get all -n istio-system
```
[결과]
```
NAME                          READY   STATUS    RESTARTS   AGE
pod/istiod-84dddcfb9b-8sxwh   1/1     Running   0          70s

NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                 AGE
service/istiod   ClusterIP   10.98.117.25   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   70s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istiod   1/1     1            1           70s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/istiod-84dddcfb9b   1         1         1       70s
```
![](https://github.com/gnosia93/k8s-on-mac/blob/main/images/istio-demo-profile.png)
