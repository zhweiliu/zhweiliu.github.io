---
draft: false
title: "K3s 設定 etcd"
date: 2023-10-29T00:30:21+08:00
categories: ["devops"]
keywords: ["DevOps", "K3s"]
summary: |
  
---
*這篇文章主要記錄 K3s cluster 設定 etcd 的方法*

先前在 "Mac M2 安裝 K3s" 介紹如何快速安裝並啟動一個 K3s cluster 。 然而最近發現 `control-plane` node 並沒有 `etcd` 的角色。
![Kubernetes cluster](https://kubernetes.io/images/docs/components-of-kubernetes.svg)

對照[K3s 文件 - Cluster Datastore](https://docs.k3s.io/datastore)，k3s cluster 使用 Kine(embedded SQLite) 作為預設的 cluster datastore。
> SQLite is the default datastore, and will be used if no other datastore configuration is present, and no embedded etcd database files are present on disk.

![](https://docs.k3s.io/assets/images/how-it-works-k3s-revised-9c025ef482404bca2e53a89a0ba7a3c5.svg)

---
# 更新既有 cluster
依據官方文件描述，只要增加 `--cluster-init` flag 並重新啟動 k3s 服務，就可以無縫轉移到 embedded etcd 服務。 具體操作流程如下

## 編輯 systemd k3s service

登入到 `control-plane` vm ， 編輯 `/etc/systemd/system/k3s.service` 檔案即可
```shell
ubuntu@server:~$ sudo vim /etc/systemd/system/k3s.service

# in vim editor
# append '--cluster-init' into ExecStart=/usr/local/bin/k3s
# save to keep change and exit file
ExecStart=/usr/local/bin/k3s \
    server \
    '--cluster-init' \

# reload service changed content and restart k3s service
ubuntu@server:~$ sudo systemctl daemon-reload
ubuntu@server:~$ sudo systemctl k3s restart

# verify node roles by kubectl
ubuntu@server:~$ sudo kubectl get node
NAME     STATUS   ROLES                       AGE     VERSION
server   Ready    control-plane,etcd,master   3d14h   v1.27.6+k3s1
worker   Ready    <none>                      3d14h   v1.27.6+k3s1
```

---
# 新建立 cluster

## 下載 etcd 
建立一台新的 Ubuntu vm 並依據 K3s 官方文件選擇安裝[etcd v.3.5.4](https://github.com/etcd-io/etcd/releases?q=3.5.4&expanded=true)版本

```shell
# create etcd-1 ubuntu 22.04 VM
$ multipass launch -n etcd-1 -c 1 -m 1G -d 3G 22.04
...
# echo etcd-1 IP
$ multipass info etcd-1 | grep IP
IPv4:           192.168.64.19

# login to etcd-1
$ multipass shell etcd-1
```

安裝 etcd shell 如下，參考 [Install and deploy etcd - systemd](http://play.etcd.io/install)
```shell
# install etcd shell
ETCD_VER=v3.5.4
PLATFORM=arm64

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-${PLATFORM}.tar.gz
rm -rf /tmp/etcd && mkdir -p /tmp/etcd
curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-${PLATFORM}.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-${PLATFORM}.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-${PLATFORM}.tar.gz -C /tmp/etcd --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-${PLATFORM}.tar.gz

# verify installation
/tmp/etcd/etcd --version
/tmp/etcd/etcdctl version
/tmp/etcd/etcdutl version

# start a local etcd server
/tmp/etcd/etcd
# terminal local etcd server
^c 

# move etcd exec file to /usr/local/bin
sudo cp /tmp/etcd/etcd* /usr/local/bin/
ETCDCTL_API=3 /tmp/etcdctl version
```


## 在 systemd 新增 etcd service

重複下列步驟，在 Ubuntu vm 上啟動 etcd-1 服務。
```shell
# create etcd.service file
cat > /tmp/etcd-1.service <<EOF
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
Conflicts=etcd.service
Conflicts=etcd2.service

[Service]
Type=notify
Restart=always
RestartSec=5s
LimitNOFILE=40000
TimeoutStartSec=0

ExecStart=/usr/local/bin/etcd --name etcd-1 \
  --data-dir /tmp/etcd/etcd-1 \
  --listen-client-urls http://192.168.64.19:2379 \
  --advertise-client-urls http://192.168.64.19:2379 \
  --listen-peer-urls http://192.168.64.19:2380 \
  --initial-advertise-peer-urls http://192.168.64.19:2380 \
  --initial-cluster etcd-1=http://192.168.64.19:2380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new

[Install]
WantedBy=multi-user.target
EOF

# move etcd-1.service to /etc/systemd/system
sudo mv /tmp/etcd-1.service /etc/systemd/system/etcd-1.service

# to start service
sudo systemctl daemon-reload
sudo systemctl cat etcd-1.service
sudo systemctl enable etcd-1.service
sudo systemctl start etcd-1.service

# to get logs from service
sudo systemctl status etcd-1.service -l --no-pager
sudo journalctl -u etcd-1.service -l --no-pager|less
sudo journalctl -f -u etcd-1.service

# Check status on vm
ETCDCTL_API=3 /usr/local/bin/etcdctl \
  --endpoints 192.168.64.19:2379 \
  endpoint health
```


## 檢查 etcd 服務
在 local 環境下載 etcdctl ，驗證 etcd vm 服務是否允許內部網路連接。 我選擇下載 [MacOS 版本](https://github.com/etcd-io/etcd/releases?q=3.5.4&expanded=true)
```shell
ETCD_VER=v3.5.4

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-darwin-amd64.zip
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-darwin-amd64.zip -o /tmp/etcd-${ETCD_VER}-darwin-amd64.zip
unzip /tmp/etcd-${ETCD_VER}-darwin-amd64.zip -d /tmp && rm -f /tmp/etcd-${ETCD_VER}-darwin-amd64.zip
mv /tmp/etcd-${ETCD_VER}-darwin-amd64/* /tmp/etcd-download-test && rm -rf /tmp/etcd-${ETCD_VER}-darwin-amd64

```

然後一樣在 local 使用 etcdctl 驗證 endpoint 是否能夠聯通
```shell
$ etcd-test ETCDCTL_API=3 ./etcdctl \
  --endpoints 192.168.64.19:2379 \
  endpoint health
192.168.64.19:2379 is healthy: successfully committed proposal: took = 8.304625ms
```

## 建立 cluster 並採用 external etcd

參考[k3s 文件 - High Availability External DB](https://docs.k3s.io/datastore/ha#2-launch-server-nodes)，步驟 1.Create an External Datastore 已經在前述過程中完成，因此從步驟 2 開始。

```shell
# create new vm
$ multipass launch -n c1-server-1 -c 1 -m 1G -d 3G 22.04
$ multipass shell c1-server-1
...

# install k3s cluster with external etcd
ubuntu@c1-server-1:~$ curl -sfL https://get.k3s.io | sh -s - server --datastore-endpoint="http://192.168.64.19:2379"
[INFO]  Finding release for channel stable
[INFO]  Using v1.27.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.27.6+k3s1/sha256sum-arm64.txt
[INFO]  Skipping binary downloaded, installed k3s matches hash
[INFO]  Skipping installation of SELinux RPM
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, already exists
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

ubuntu@c1-server-1:~$ sudo kubectl get node
NAME          STATUS   ROLES                  AGE   VERSION
c1-server-1   Ready    control-plane,master   8s    v1.27.6+k3s1

ubuntu@c1-server-1:~$ exit
```

## 驗證 external etcd 運作情況

導出新建立的 cluster `kubeconfg` 檔案，並透過 `alias` 建立指定 kubconfig 檔案的 kubectl shortcut
```shell
# export kubeconfig
$ multipass exec c1-server-1 sudo cat /etc/rancher/k3s/k3s.yaml > c1-kubeconfig

# alias kubectl shortcut with kubeconfig
$ alias kc1='kubectl --kubeconfig /Users/eric/devops/etcd-test/c1-kubeconfig'

# test alias
$ kc1 get node
NAME          STATUS   ROLES                  AGE   VERSION
c1-server-1   Ready    control-plane,master   12m   v1.27.6+k3s1

# create a new namespace of cluster
$ kc1 create ns nginx
namespace/nginx created

# verify external etcd
$ ETCDCTL_API=3 ./etcdctl --endpoints=http://192.168.64.19:2379 get / --prefix --keys-only | grep nginx
/registry/configmaps/nginx/kube-root-ca.crt
/registry/namespaces/nginx
/registry/serviceaccounts/nginx/default
```

從最後驗證 external etcd 的步驟可以看到，當 namespace `nginx` 建立之後， external etcd 就會產生對應的 key-value 紀錄。

