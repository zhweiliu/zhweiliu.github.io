---
draft: false
title: "Mac M2 K3s"
date: 2023-10-25T22:03:42+08:00
categories: ["DevOps"]
keywords: ["DevOps", "K3s"]
summary: |
  K3s是輕量化的 Kubernetes，由於先前我都是使用Docker Desktop Kubernetes，因為 Docker Desktop Kubernetes 是 single-node Kubernetes or Docker Swarm cluster，在local 部署 Pod 時也無法實際測試 affinity 功能，因此就想玩玩看 K3s 。
---
*這篇文章主要記錄在 Mac M2 上安裝設定 K3s 的方法*

[K3s](https://k3s.io/)是輕量化的 Kubernetes，由於先前我都是使用[Docker Desktop Kubernetes](https://docs.docker.com/desktop/kubernetes/)，因為 Docker Desktop Kubernetes 是 single-node Kubernetes or Docker Swarm cluster，在local 部署 Pod 時也無法實際測試 affinity 功能，因此就想玩玩看 K3s 。(通過 CKA 考試後也就漸漸忘了怎麼從零開始使手動建置一個 Kubernetes Cluster ... )

---
## multipass
為了實現在 local 架設一個 multi nodes 組成的 cluster ， 首先是需要在 local 建立 VMs ， 這邊我選用 [multipass](https://multipass.run/)，可以簡單並且快速的建立 Ubuntu VM ，並且編輯設定 VM 規格的方式也是簡單直覺的指令。

在 Mac M2 上可以透過 brew 安裝 multipass 
```shell
brew install multipass
```

在這個例子中，我用 multipass 啟用兩台 VM ， 分別是作為 control plane 的 `server` 以及 data plane 的 `worker`

先建立 node `server`
```shell
multipass launch -n server -c 1 -m 2G -d 5G 22.04
```
- `-n` 設定 node name 
- `-c` 設定 node vCPU
- `-m` 設定 node memory
- `-d` 設定 node 
- 22.04 設定 ubuntu image 的版本


再來建立 node `worker`
```shell
multipass launch -n worker -c 1 -m 2G -d 5G 22.04
```

透過 multipass list 指令，就能看到目前已經建立出來的 VMs
```shell
$ multipass list
Name                    State             IPv4             Image
server                  Running           192.168.64.2     Ubuntu 22.04 LTS
worker                  Running           192.168.64.3     Ubuntu 22.04 LTS
```

---
## K3s
接著分別在兩台 VM 安裝 K3s ， 首先透過 multipass 指令登入 node `server` ， 並安裝 K3s
```shell
$ multipass shell server
...
ubuntu@server:~$ curl -sfL https://get.k3s.io | sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.27.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.27.6+k3s1/sha256sum-arm64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.27.6+k3s1/k3s-arm64
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
```

安裝完成後，先打印出 node `server` 上的 [kubeconfig](https://docs.k3s.io/cluster-access) 設定 ， 以便之後可以在 local 直接透過 `kubectl` 訪問 k3s cluster
```shell
$ cat /etc/rancher/k3s/k3s.yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS...
    server: https://192.168.64.2:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: LS0tLS...
    client-key-data: LS0tLS...
```
把這邊的 `clusters` 和 `users` 內容貼回 local 端 ~/kube/config 內，新增對應的 contexts ，並記得將上面 clusters.cluster.server 的 IP 換成 node `server` 的 IP 即可。 以我的例子，server IP 是 `192.168.64.2`。
同時，也建議把上面 `cluster` `user` `context` 的 name 更換成容易識別的名稱，比如 `k3s`。設定完就可以登出 node `server`。

這時先在 local 用 kubectl config 查看剛才設定好的ㄚ k3s cluster ， 並將 kube context 切換到 k3s cluster 。
```shell
# list kube contexts
$ kubectl config get-contexts
CURRENT     NAME        CLUSTER     AUTHINFO        NAMESPACE
            k3s         k3s         k3s

# set kube context
$ kubectl config set-context k3s
Context "k3s" modified.

# confirm context switched
kubectl config get-contexts
CURRENT     NAME        CLUSTER     AUTHINFO        NAMESPACE
*           k3s         k3s         k3s
```

到這一步，可以先驗證是否可以正常訪問 k3s cluster
```shell
# verify k3s cluster-info
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.64.2:6443
CoreDNS is running at https://192.168.64.2:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://192.168.64.2:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

# verify k3s cluster nodes
$ kubectl get nodes
NAME     STATUS   ROLES                  AGE   VERSION
server   Ready    control-plane,master   18m   v1.27.6+k3s1
```

此時 k3s cluster 應該只有一台 node `server` ， 接著要將 multipass vm worker 註冊成 k3s cluster 的 node ， 為此需要 node `server` (control-plane) 的 IP 與 token ， 讓 worker 可以通過 node 的註冊驗證
```shell
# get node server ip and token
$ multipass exec server sudo cat /var/lib/rancher/k3s/server/node-token
K10ad55509e50dcc23a7e06d5a8e115668d97a31fa0b2d225462d8ee82aa89827e8::server:d91bbb9e9a3bae55e412a12ccc6ec307

$ multipass info server | grep IP
IPv4:           192.168.64.2
```

登入 node `worker` ， 安裝 k3s 時帶入 control-plane 的 token 和 IP
```shell
$ multipass shell worker
...
ubuntu@worker:~$ curl -sfL https://get.k3s.io | K3S_URL=https://192.168.64.2:6443 K3S_TOKEN="K10ad55509e50dcc23a7e06d5a8e115668d97a31fa0b2d225462d8ee82aa89827e8::server:d91bbb9e9a3bae55e412a12ccc6ec307" sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.27.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.27.6+k3s1/sha256sum-arm64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.27.6+k3s1/k3s-arm64
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent
```

登出 node `worker` ， 在 local 端執行 kubectl 指令查看 k3s cluster nodes
```shell
$ kubectl get nodes -owide
NAME     STATUS   ROLES                  AGE   VERSION        INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
server   Ready    control-plane,master   26m   v1.27.6+k3s1   192.168.64.2   <none>        Ubuntu 22.04.3 LTS   5.15.0-86-generic   containerd://1.7.6-k3s1.27
worker   Ready    <none>                 28s   v1.27.6+k3s1   192.168.64.3   <none>        Ubuntu 22.04.3 LTS   5.15.0-86-generic   containerd://1.7.6-k3s1.27
```

此時 node `worker` 也被註冊到 k3s cluster 上，且兩台 node 的 OS-IMAGE 都是先前在 multipass 建立 VM 時使用的 Ubuntu 22.04 版本，container runtime 則是 k3s 提供的 1.27 版本。