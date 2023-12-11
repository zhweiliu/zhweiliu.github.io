---
draft: false
title: "Argocd - 託管另一個 k3d cluster"
date: 2023-12-09T22:23:02+08:00
categories: ["DevOps"]
keywords: ["DevOps", "argo", "argocd"]
summary: |
  託管 k3d cluster 給 argocd 時發生錯誤 `dial tcp 0.0.0.0:26443: connect: connection refused` ， 透過手動增加 k3d cluster credential secret，並且透過 docker network 讓 argocd cluster 可以解析 k3d cluster api server ，就可以正常進行託管。
---

參考[Argocd Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)步驟 : 部署 argocd UI 版本以及安裝 argocd CLI 後，在 [5. Register A Cluster To Deploy Apps To (Optional)](https://argo-cd.readthedocs.io/en/stable/getting_started/#5-register-a-cluster-to-deploy-apps-to-optional) 可以選擇註冊另一個 kubernetes cluster 給 argocd 託管；

我用 2 個 k3d cluster 來模擬 argocd 註冊託管的功能時，發生了錯誤 `dial tcp 0.0.0.0:26443: connect: connection refused`
```shell
INFO[0002] ServiceAccount "argocd-manager" created in namespace "kube-system"
INFO[0002] ClusterRole "argocd-manager-role" created
INFO[0002] ClusterRoleBinding "argocd-manager-role-binding" created
INFO[0007] Created bearer token secret for ServiceAccount "argocd-manager"
FATA[0007] rpc error: code = Unknown desc = Get "https://0.0.0.0:26443/version?timeout=32s": dial tcp 0.0.0.0:26443: connect: connection refused
```

---
## 0.0.0.0

在 argocd 文件中描述 `argocd add cluster` 指令的用途，是將 kubeconfig 註冊給 argocd 以便託管整座 cluster
> This step registers a cluster's credentials to Argo CD, and is only necessary when deploying to an external cluster. When deploying internally (to the same cluster that Argo CD is running in), https://kubernetes.default.svc should be used as the application's K8s API server address.

而 k3d 建立的 cluster ， 則是將 kube API server 的 endpoints 設定在 **{0.0.0.0:port}** 之上，

一般看到 **0.0.0.0** 會聯想到 any host ， 實際上 **0.0.0.0** 所表達是一個 **`不清楚主機和目的網路的集合`**
有興趣可以進一步參考網路上的資料，很多文章都解釋得很清楚
- [Difference Between IP Address 127.0.0.1 and 0.0.0.0](https://www.baeldung.com/linux/difference-ip-address)
- [你/妳真的了解 127.0.0.1 與 0.0.0.0 的區別？](https://ithelp.ithome.com.tw/articles/10311096)
- [docker 面试之 ，ip地址0.0.0.0和127.0.0.1的区别](https://blog.csdn.net/yz18931904/article/details/106440444)

而 k3d 建立的 cluster，正是使用了 **`{0.0.0.0:api-port}`** 的方式，來表達 cluster API server 的位置，並透過 port-forward 的方式將 localhost 和 docker network 關聯起來。
![](/images/devops/docker-port-forward.png)

因此，當 argocd 所在的 cluster 要去驗證 apps cluster 時，無法透過 **`0.0.0.0:{api-port}`** 找到 apps cluster API server 的實際位置，導致發生錯誤`dial tcp 0.0.0.0:26443: connect: connection refused` 。

---
## Argocd Cluster Secret

從[Argocd Cluster 文件](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#clusters)可以得知，註冊託管的 cluster credentials 都是帶有 label `argocd.argoproj.io/secret-type: cluster` 的 secret ， 因此在 argocd cluster 上手動建立起需要託管的 k3d cluster token 即可。

雖然手動建立 cluster credential secret 是 workaround ，但在 local 的環境下我覺得已經足夠。

建立 cluster credential secret 的範本如下
```yaml
apiVersion: v1
kind: Secret
metadata:
  namespace: argocd
  name: k3d-apps-cluster-secret
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: k3d-apps
  server: "https://k3d-apps-server-0:6443"
  config: |
    {
      "bearerToken": "<authentication token>",
      "tlsClientConfig": {
        "insecure": false,
        "caData": "<base64 encoded certificate>"
      }
    }
```
### bearerToken 

在 `argocd cluster add k3d-apps` 指令執行的第一步，實際上已經在 `k3d-apps` cluster 上建立起 service account `argocd-manager` 

```shell
kubectx k3d-apps
```

```shell
kubectl get -n kube-system secrets
...
NAME                                  TYPE                                  DATA   AGE
k3d-apps-server-0.node-password.k3s   Opaque                                1      79m
k3d-guestbook-0.node-password.k3s     Opaque                                1      78m
k3s-serving                           kubernetes.io/tls                     2      79m
argocd-manager-token-zdzcc            kubernetes.io/service-account-token   3      73m
```

而 `argocd-manager` 的 token 就是手動建立 secret 中需要的 **`bearerToken`**

```shell
TOKEN="$( kubectl get secret -n kube-system \
        argocd-manager-token-zdzcc  -o json  | 
        jq  -r .data.token | base64 -d )"
```

如果沒有 [jq](https://jqlang.github.io/jq/) 的話，就隨手[安裝](https://formulae.brew.sh/formula/jq)吧

### caData

caData 則是查找 kubeconfig ，就會在 cluster 的 `certificate-authority-data` 找到

```yaml
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkakNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUzTURJek1EUXdPVGd3SGhjTk1qTXhNakV4TVRReE5EVTRXaGNOTXpNeE1qQTRNVFF4TkRVNApXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUzTURJek1EUXdPVGd3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFUNGE5amhTeVk3VVJXNmc2cUNnT1FZZmsxaDU4cEsvVHBRTWgrSDZXanUKOUg3b1RBdFBSd2ZNblN3b1FqUXlZVFE1NzhYWTh5eitWYy9TV1R2bWFRSTdvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVWhPTVMwdDdQVUVXSFkxc2lQdDlOCkdXRnZsWVl3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnSWwyZHdEd0xnOWFkMDBGblhlb2xyR3E2V0t4YkhjUC8KYjlMUE5lZ3h0cElDSUhDYWRuc3ZmTlJiYS9TZ3lBdC9CYUhjS2NpeEF4b2dWcG9yOCtac0p0UGgKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://0.0.0.0:26443
  name: k3d-apps
```

### Docker network connect

最後，從 Docker network 將 apps cluster api server **`k3d-apps-server-0`** 加入到 argocd 的 network 中，讓 CoreDNS 可以解析k3d-apps-server-0。

```shell
docker network connect k3d-infrastructure k3d-apps-server-0
```

接著重啟 k3d-infrastructure cluster 
```shell
k3d cluster stop infrastructure
k3d cluster start infrastructure
```

然後檢查 k3d-infrastructure 的 coredns resolv 設定
```shell
kubectx k3d-infrastructure
kubectl get -n kube-system configmap coredns -oyaml
```

```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        hosts /etc/coredns/NodeHosts {
          ttl 60
          reload 15s
          fallthrough
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
        import /etc/coredns/custom/*.override
    }
    import /etc/coredns/custom/*.server
  NodeHosts: |
    172.18.0.3 k3d-infrastructure-server-0
    192.168.65.254 host.k3d.internal
    172.18.0.2 k3d-infrastructure-tools
    172.18.0.4 k3d-argocd-0
    172.18.0.6 k3d-apps-server-0
    172.18.0.5 k3d-infrastructure-serverlb
kind: ConfigMap
```

NodeHosts 中看到 k3d-apps-server-0 的解析 record 則表示成功

## Deploy gusetbook to apps cluster

接著[步驟6. Create An Application From A Git Repository](https://argo-cd.readthedocs.io/en/stable/getting_started/) 開始，
在 `Destination Cluster` section 選擇 `https://k3d-apps-server-0:6443` ，並將 guestbook 部署到 apps cluster 的 default namespace 上，
就會看到同步成功的畫面。
![](/images/devops/argocd-deploy-guestbook-to-apps-cluster.png)

