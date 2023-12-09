---
draft: false
title: "K3d - K3s 輕量化 wrapper"
date: 2023-12-09T17:56:49+08:00
categories: ["DevOps"]
keywords: ["DevOps", "K3d"]
summary: |
  K3d 是 K3s 的輕量化 wrapper ， 讓 K3s 得以 Docker 上執行。 
---

[K3d](https://k3d.io/v5.6.0/) 是 K3s 的輕量化 wrapper ， 讓 K3s 得以 Docker 上執行 ， 可以快速建立起 K3s cluster ， 並將 node 以 docker 的形式進行部署，實現單一 cluster multi-nodes 的模擬環境。

K3d 會為每個 cluster 建立獨立的 Docker 網路，因此也便於實現模擬 multi-cluster 間的應用，如 : [Istio](https://istio.io/) 、 Argo CD 與其生態 ， 或是如 [Cilium](https://cilium.io/) CNI (Container Network Interface ) 等。

---
## K3d Installation

為方便觀察 K3d 部屬的 docker ， 我選擇[安裝 Docker Desktop](https://www.docker.com/get-started/) 建立基礎的 Docker 執行環境。

之後再透過 brew 安裝 K3d CLI (Command Line Interface)
```shell
brew install k3d
```

---
## Create K3d cluster

除了依照 [Quick Start](https://k3d.io/v5.6.0/#quick-start) 建立 cluster ， K3d 也支援利用 YAML 宣告的 [config file](https://k3d.io/v5.6.0/usage/configfile/#config-options) 

```shell
k3d cluster create --config /home/me/my-awesome-config.yaml
```

這篇文章則是採用 Quick Start 中的範例來建立

```shell
k3d cluster create mycluster
```

觀察 cluster create 的 outputs ， 可以發現 k3d 做了以下幾件事情
1. 建立 docker network `k3d-mycluster`
2. 建立 3 個 nodes `k3d-mycluster-server-0` `k3d-mycluster-tools` `k3d-mycluster-serverlb`
3. 建立 1 個 configmap
```shell
INFO[0002] Prep: Network
INFO[0002] Created network 'k3d-mycluster'
INFO[0002] Created image volume k3d-mycluster-images
INFO[0002] Starting new tools node...
INFO[0003] Creating node 'k3d-mycluster-server-0'
INFO[0004] Pulling image 'ghcr.io/k3d-io/k3d-tools:5.6.0'
INFO[0006] Pulling image 'docker.io/rancher/k3s:v1.27.4-k3s1'
INFO[0008] Starting Node 'k3d-mycluster-tools'
INFO[0016] Creating LoadBalancer 'k3d-mycluster-serverlb'
INFO[0018] Pulling image 'ghcr.io/k3d-io/k3d-proxy:5.6.0'
INFO[0026] Using the k3d-tools node to gather environment information
INFO[0026] Starting new tools node...
INFO[0026] Starting Node 'k3d-mycluster-tools'
INFO[0027] Starting cluster 'mycluster'
INFO[0027] Starting servers...
INFO[0027] Starting Node 'k3d-mycluster-server-0'
INFO[0029] All agents already running.
INFO[0029] Starting helpers...
INFO[0029] Starting Node 'k3d-mycluster-serverlb'
INFO[0036] Injecting records for hostAliases (incl. host.k3d.internal) and for 3 network members into CoreDNS configmap...
INFO[0038] Cluster 'mycluster' created successfully!
INFO[0038] You can now use it like this:
kubectl cluster-info
```

### Docker network

透過指令觀察 docker network ， 可以看到 k3d 剛才建立的新 network `k3d-mycluster` ， 模式為 `bridge`
```shell
$ docker network ls
...
NETWORK ID     NAME            DRIVER    SCOPE
d4d82eb49ae3   ansible-net     bridge    local
c72fd13e2a1e   bridge          bridge    local
29c4c283b128   host            host      local
d6e6e905b38d   k3d-mycluster   bridge    local
6f4723eeb263   none            null      local
```

### Node containers

透過 Docker Desktop UI ，可以看到 k3d 剛才建立起的 3 個 nodes ， 都是以 container 的方式在執行
![](/images/devops/k3d-node-containers.png)

### ConfigMap

透過指令查看 K3d 建立的 ConfigMap ， 主要查看 configmap 內 `data` 的區塊
```shell
kubectl get -n kube-system cm coredns -oyaml
```

```yaml
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
    192.168.65.254 host.k3d.internal
    172.18.0.2 k3d-mycluster-tools
    172.18.0.4 k3d-mycluster-serverlb
    172.18.0.3 k3d-mycluster-server-0
```

可以看到 K3d 將 DNS `resolv.conf` 的設定放在 configmap 內，若需要修改 DNS 解析的條目時，直接更新 configmap 內容即可。

---
## Create K3d node

利用 [K3d node 相關指令](https://k3d.io/v5.6.0/usage/commands/k3d_node/#synopsis) ， 建立起第一個 K3s node

```shell
k3d node create -c mycluster --k3s-node-label app=nginx,nodepool=nginx nginx
```

在上述指令中  : 
* `-c` : 指定歸屬的 k3d cluster 給新建立的 node。 如果沒有填寫， 則會被歸納到預設的 cluster `k3d-default`
* `--k3s-node-label` : 指定 node label 給新建立的 node 。 通常搭配 `nodeSelect` `affinity` `topologySpreadConstraints` 一起使用。

當 node 建立完成後，一樣可以在 Docker Desktop 看到新的 node 被包裝成 docker container `k3d-nginx-0` 執行 

透過 `k3d node list` 指令，也能查看到目前已經建立好的 nodes 有哪些

```shell
NAME                     ROLE           CLUSTER     STATUS
k3d-mycluster-server-0   server         mycluster   running
k3d-mycluster-serverlb   loadbalancer   mycluster   running
k3d-mycluster-tools                     mycluster   running
k3d-nginx-0              agent          mycluster   running
```

最後是透過 `kubectl get node` 的指令，觀察新加入的 node 
```shell
kubectl get node
```

```shell
NAME                     STATUS   ROLES                  AGE     VERSION
k3d-mycluster-server-0   Ready    control-plane,master   26m     v1.27.4+k3s1
k3d-nginx-0              Ready    <none>                 6m14s   v1.27.4+k3s1
```

查看 node `k3d-nginx-0` 的詳細資訊
```shell
kubectl get node k3d-nginx-0 -oyaml
```

在 `node.metadata.labels` ，可以觀察到建立 node 時指定的 label `app=nginx` `nodepool=nginx`
```yaml
metadata:
    labels:
        app: nginx
        beta.kubernetes.io/arch: arm64
        beta.kubernetes.io/instance-type: k3s
        beta.kubernetes.io/os: linux
        kubernetes.io/arch: arm64
        kubernetes.io/hostname: k3d-nginx-0
        kubernetes.io/os: linux
        node.kubernetes.io/instance-type: k3s
        nodepool: nginx
```

### Deploy nginx pods

透過指令部署 Nginx 服務到新的 node 上 
```shell
kubectl deploy -f https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/application/deployment.yaml
```

觀察 nginx 部署狀況
```shell
kubectl get all
```

```shell
NAME                                   READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-cbdccf466-bfvs2   1/1     Running   0          85s
pod/nginx-deployment-cbdccf466-5v5k5   1/1     Running   0          85s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   34m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           85s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-cbdccf466   2         2         2       85s
```

查看 pods 部署到哪些 nodes
```shell
kubectl get pods -owide
```

```shell
NAME                               READY   STATUS    RESTARTS   AGE     IP          NODE                     NOMINATED NODE   READINESS GATES
nginx-deployment-cbdccf466-bfvs2   1/1     Running   0          2m57s   10.42.1.3   k3d-nginx-0              <none>           <none>
nginx-deployment-cbdccf466-5v5k5   1/1     Running   0          2m57s   10.42.0.9   k3d-mycluster-server-0   <none>           <none>
```

## Delete K3d cluster

最後將測試用的 k3d cluster 刪除
```shell
k3d cluster delete mycluster
```
