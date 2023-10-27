---
draft: false
title: "透過 NFS Server 在 K3s cluster 新增 Storage Class"
date: 2023-10-26T23:42:35+08:00
categories: ["devops"]
keywords: ["DevOps", "K3s"]
summary: |
  之前讀過 Shawn Ho 大大的在GKE上使用ReadWrite Many的Disk ， 突然意識到 multipass 產生的 Ubuntu VM ，不就是現成的 Filesystem ! 只要在 Ubuntu 上安裝了 NFS server ， 並在其他 VM 上安裝 NFS client ， 那應該就能新增使用 NFS 的 Storage Class 了！查了一些資料後發現可行，於是就手動實做看看。
---
*這篇文章主要記錄透過 NFS Server 在 K3s cluster 新增 Storage Class的方法*

之前讀過 Shawn Ho 大大的[在GKE上使用ReadWrite Many的Disk](https://medium.com/%E8%BC%95%E9%AC%86%E5%B0%8F%E5%93%81-pks%E8%88%87k8s%E7%9A%84%E9%BB%9E%E6%BB%B4/%E5%9C%A8gke%E4%B8%8A%E4%BD%BF%E7%94%A8readwrite-many%E7%9A%84disk-9945ff67a4d)，其中提到:
> 通過部署一套NFS Pod，該Pod先使用ReadWrite Once，再把這個NFS Pod當作是File Server提供ReadWrite Many的使用

突然意識到 multipass 產生的 Ubuntu VM ，不就是現成的 Filesystem ! 只要在 Ubuntu 上安裝了 NFS server ， 並在其他 VM 上安裝 NFS client ， 那應該就能新增使用 NFS 的 Storage Class 了！ 查了一些資料後發現可行，於是就手動實做看看。

---
## NFS Server
一樣先用 multipass 啟動一台 VM 作為 NFS Server 使用
```shell
multipass launch -n nfs-server -c 1 -m 2G -d 10G 22.04
```

登入 VM `nfs-server` 後，安裝 nfs-server
```shell
$ multipass shell nfs-server
...

# install nfs-kernel-server
ubuntu@nfs-server:~$ sudo apt-get install -y nfs-kernel-server
...

# confirm nfs-kernel-server status
ubuntu@nfs-server:~$ sudo service nfs-kernel-server status
● nfs-server.service - NFS server and services
     Loaded: loaded (/lib/systemd/system/nfs-server.service; enabled; vendor preset: enabled)
     Active: active (exited) since Fri 2023-10-27 00:24:21 CST; 1min 7s ago
   Main PID: 2296 (code=exited, status=0/SUCCESS)
        CPU: 4ms

Oct 27 00:24:19 nfs-server systemd[1]: Starting NFS server and services...
Oct 27 00:24:19 nfs-server exportfs[2295]: exportfs: can't open /etc/exports for reading
Oct 27 00:24:21 nfs-server systemd[1]: Finished NFS server and services.
```

nfs-server 安裝成功， nfs-kernel-server 服務啟動，但提示仍要設定 `/etc/exports`
```shell
# create a folder for storage nfs client files
ubuntu@nfs-server:~$ mkdir -p /home/ubuntu/storage

# edit /etc/exports
ubuntu@nfs-server:~$ sudo vim /etc/exports

# add config of file /etc/exports and save it
/home/ubuntu/storage 192.168.64.0/24(rw,sync,no_subtree_check,root_squash,insecure)

# export /etc/exports configuration
ubuntu@nfs-server:~$ sudo exportfs -a

# restart service
ubuntu@nfs-server:~$ sudo service nfs-kernel-server restart

# confirm nfs-kernel-server status
ubuntu@nfs-server:~$ sudo service nfs-kernel-server status
● nfs-server.service - NFS server and services
     Loaded: loaded (/lib/systemd/system/nfs-server.service; enabled; vendor preset: enabled)
     Active: active (exited) since Fri 2023-10-27 00:43:39 CST; 29s ago
    Process: 2802 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 2803 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
   Main PID: 2803 (code=exited, status=0/SUCCESS)
        CPU: 4ms

Oct 27 00:43:39 nfs-server systemd[1]: Starting NFS server and services...
Oct 27 00:43:39 nfs-server systemd[1]: Finished NFS server and services.
```

沒有提示需要設定 `/etc/exports` ， 表示上述設定 nfs-server 的動作完成

`/etc/exports` 的設定格式如下
```text
{share_folder_path} {allowed ip range | any host}(options)
``` 
- `{share_folder_path}` : NFS server 分享的資料夾位置，用以存放 NFS client 的檔案位置
- `{allowed ip range | any host}` : 允許可存取 `{share_folder_path}` 的 IP 來源，可用 `CIDR` 或 `*`
- `options` 有以下選項 :
下表轉載於[Akiicat 學習筆記 - 在 Ubuntu 上架設 NFS server](https://www.akiicat.com/2019/03/09/DevOps/setup-nfs-server/)，感謝作者採用[創用 CC 姓名標示-相同方式分享 4.0 國際 授權條款授權.](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hant)，讓我可以直接分享
```
選項是選擇性填寫的，有很多參數可以選
- ro：read only
- rw：read and write
- async：此選項允許 NFS Server 違反 NFS protocol，允許檔案尚未存回磁碟之前回覆請求。這個選項可以提高性能，但是有可能會讓 server 崩潰，可能會需要重新啟動 server. 或檔案遺失。
- sync：只會儲存檔案會磁碟之後才會回覆請求。
- no_subtree_check：禁用子樹檢查，會有些微的不安全，但在某些情況下可以提高可靠性。
- secure：請求的 port 必須小於 1024，這個選項是預設的。
- insecure：請求的 port 不一定要小於 1024。

User ID Mapping 參數：
- root_squash：將 uid 0 (root) 的使用者映射到 nobody (uid 65534) 匿名使用者，這個選項是預設的。
- no_root_squash：關掉 root squash 的選項，這個選項可以使用 root 身份來控制 NFS Server 的檔案。
- all_squash：所有登入 NFS 的使用者身份都會被壓縮成為 nobody。
```
更多參數的選項可以參考 [exports(5) - Linux man page](https://linux.die.net/man/5/exports)

---
## nfs-subdir-external-provisioner
我用 helm 安裝 [nfs-subdir-external-provisioner](https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner)，它會自動設定並使用上面安裝好的 NFS server ，同時產生一個 storage class ， 之後建立 PVC 時採用這個 storage class ， 就會自動產生對應的 PV ，我覺得超級方便。
```shell
$ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/ 
$ helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=192.168.64.4 \
    --set nfs.path=/home/ubuntu/storage
NAME: nfs-subdir-external-provisioner
LAST DEPLOYED: Fri Oct 27 00:58:25 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

這時檢查一下 cluster 內的 resource
```shell
$ kubectl get sc,pod
NAME                                               PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/local-path (default)   rancher.io/local-path                           Delete          WaitForFirstConsumer   false                  26h
storageclass.storage.k8s.io/nfs-client             cluster.local/nfs-subdir-external-provisioner   Delete          Immediate              true                   2m7s

NAME                                                   READY   STATUS              RESTARTS   AGE
pod/nfs-subdir-external-provisioner-7fdb777787-nc6rw   0/1     ContainerCreating   0          2m7s
```

可以看到 nfs-subdir-external-provisioner 已經建立起一個 名稱為 `nfs-client` 的 storage class

但 nfs-subdir-external-provisioner 的 pod 似乎卡在 ContainerCreating，檢查 pod 狀態
```shell
$ kubectl describe pod nfs-subdir-external-provisioner-7fdb777787-nc6rw
...
Events:
  Type     Reason       Age                   From               Message
  ----     ------       ----                  ----               -------
  Normal   Scheduled    4m56s                 default-scheduler  Successfully assigned default/nfs-subdir-external-provisioner-7fdb777787-nc6rw to worker
  Warning  FailedMount  47s (x10 over 4m57s)  kubelet            MountVolume.SetUp failed for volume "nfs-subdir-external-provisioner-root" : mount failed: exit status 32
Mounting command: mount
Mounting arguments: -t nfs 192.168.64.4:/home/ubuntu/storage /var/lib/kubelet/pods/a84c53a7-8356-4443-b40d-8c7b4d9d3767/volumes/kubernetes.io~nfs/nfs-subdir-external-provisioner-root
Output: mount: /var/lib/kubelet/pods/a84c53a7-8356-4443-b40d-8c7b4d9d3767/volumes/kubernetes.io~nfs/nfs-subdir-external-provisioner-root: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
  Warning  FailedMount  40s (x2 over 2m54s)  kubelet  Unable to attach or mount volumes: unmounted volumes=[nfs-subdir-external-provisioner-root], unattached volumes=[], failed to process volumes=[]: timed out waiting for the condition
```

提示 pod 在 mount `/home/ubuntu/storage` 時出現了錯誤
```shell
Output: mount: /var/lib/kubelet/pods/a84c53a7-8356-4443-b40d-8c7b4d9d3767/volumes/kubernetes.io~nfs/nfs-subdir-external-provisioner-root: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
```

這是因為 pod 掛載的 node 沒有安裝對應的 nfs-client 套件導致，檢查 pod 目前掛載在哪個 node
```shell
$ kubectl get pod -owide
NAME                                               READY   STATUS              RESTARTS   AGE     IP       NODE     
nfs-subdir-external-provisioner-7fdb777787-nc6rw   0/1     ContainerCreating   0          7m31s   <none>   worker   
```

登入到 node `worker` 安裝 nfs-common 套件
```shell
# login to node worker
$ multipass shell worker
...
# install nfs-common
ubuntu@worker:~$ sudo apt-get install -y nfs-common

# confirm nfs-common service status
ubuntu@worker:~$ sudo service nfs-common status
○ nfs-common.service
     Loaded: masked (Reason: Unit nfs-common.service is masked.)
     Active: inactive (dead)
```

驚了！剛安裝好的 nfs-common 竟然是 `inactive` ，檢查 nfs-common.service 檔案
```shell
ubuntu@worker:~$ file /lib/systemd/system/nfs-common.service
/lib/systemd/system/nfs-common.service: symbolic link to /dev/null
```

不知道什麼原因造成 link 到 `/dev/null` ， 嘗試刪除檔案後重新加載服務
```shell
ubuntu@worker:~$ sudo rm -f /lib/systemd/system/nfs-common.service
ubuntu@worker:~$ sudo systemctl status nfs-common
○ nfs-common.service - LSB: NFS support files common to client and server
     Loaded: loaded (/etc/init.d/nfs-common; generated)
     Active: inactive (dead)
       Docs: man:systemd-sysv-generator(8)
```

nfs-common 服務已載入，接下來重啟服務即可
```shell
ubuntu@worker:~$ sudo service nfs-common restart
ubuntu@worker:~$ sudo service nfs-common status
● nfs-common.service - LSB: NFS support files common to client and server
     Loaded: loaded (/etc/init.d/nfs-common; generated)
     Active: active (running) since Fri 2023-10-27 01:14:12 CST; 8s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 9992 ExecStart=/etc/init.d/nfs-common start (code=exited, status=0/SUCCESS)
      Tasks: 2 (limit: 2275)
     Memory: 2.4M
        CPU: 38ms
     CGroup: /system.slice/nfs-common.service
             ├─10000 /sbin/rpc.statd
             └─10012 /usr/sbin/rpc.idmapd

Oct 27 01:14:12 worker systemd[1]: Starting LSB: NFS support files common to client and server...
Oct 27 01:14:12 worker nfs-common[9992]:  * Starting NFS common utilities
Oct 27 01:14:12 worker rpc.statd[10000]: Version 2.6.1 starting
Oct 27 01:14:12 worker sm-notify[10001]: Version 2.6.1 starting
Oct 27 01:14:12 worker sm-notify[10001]: Already notifying clients; Exiting!
Oct 27 01:14:12 worker rpc.statd[10000]: Failed to read /var/lib/nfs/state: Success
Oct 27 01:14:12 worker rpc.statd[10000]: Initializing NSM state
Oct 27 01:14:12 worker rpc.idmapd[10012]: Setting log level to 0
Oct 27 01:14:12 worker nfs-common[9992]:    ...done.
Oct 27 01:14:12 worker systemd[1]: Started LSB: NFS support files common to client and server.
```

再次檢查 `nfs-subdir-external-provisioner` pod 的狀態， 可以看到已經是 running 了
```shell
$ kubectl get pod
NAME                                               READY   STATUS    RESTARTS   AGE
nfs-subdir-external-provisioner-7fdb777787-nc6rw   1/1     Running   0          16m
```

---
## 測試 storage class
透過部署一個 nginx 服務，並將 nginx log 掛載到 pv 上，測試 storage class 是否能正常使用

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-log
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
          - name: nginx-log
            mountPath: /var/log/nginx
      volumes:
      - name: nginx-log
        persistentVolumeClaim:
            claimName: nginx-log
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```

儲存成 install-nginx.yaml 檔之後，部署到 k3s cluster
```shell
# deploy install-nginx.yaml file
$ kubectl apply -f install-nginx.yaml
persistentvolumeclaim/nginx-log created
deployment.apps/nginx created
service/nginx created

# verify pvc,pv,pod,svc
$ kubectl get pvc,pv,pod,svc
persistentvolumeclaim/nginx-log   Bound    pvc-f4a17b71-d79d-44fa-9085-ad2d537dae32   100Mi      RWX            nfs-client     5m26s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
persistentvolume/pvc-f4a17b71-d79d-44fa-9085-ad2d537dae32   100Mi      RWX            Delete           Bound    default/nginx-log   nfs-client              5m26s

NAME                                                   READY   STATUS    RESTARTS   AGE
pod/nfs-subdir-external-provisioner-7fdb777787-nc6rw   1/1     Running   0          66m
pod/nginx-6dd49b6ccb-v6ggj                             1/1     Running   0          5m26s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP   27h
service/nginx        ClusterIP   10.43.195.255   <none>        80/TCP    4m3s
```

可以看到 
- nginx pod 已經部署完成並進入到 `Running` 的狀態
- 自動產生了一個 `Persistent Volume` 讓 PVC 使用，並進入可用的 `Bound` 狀態

開啟兩個 Terminal ， 執行 kubectl port-forwarding 的指令，並用 curl 打一個 request 到 nginx pod
```shell
$ kubectl port-forward service/nginx 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080

# another terminal
$ curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
```

最後登入到 VM `nfs-server` 內，查看 NFS server 分享的資料夾根目錄
```shell
$ multipass shell nfs-server
...
ubuntu@nfs-server:~$ ls storage/
default-nginx-log-pvc-f4a17b71-d79d-44fa-9085-ad2d537dae32

ubuntu@nfs-server:~$ ls storage/default-nginx-log-pvc-f4a17b71-d79d-44fa-9085-ad2d537dae32
access.log  error.log

ubuntu@nfs-server:~$ cat storage/default-nginx-log-pvc-f4a17b71-d79d-44fa-9085-ad2d537dae32/access.log
127.0.0.1 - - [27/Oct/2023:13:17:54 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.84.0" "-"
127.0.0.1 - - [27/Oct/2023:13:17:56 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.84.0" "-"
```

可以看到 `storage/` 下建立了一個資料夾 `default-nginx-log-pvc-f4a17b71-d79d-44fa-9085-ad2d537dae32` 提供給剛才建立的 `Persistent Volume` 使用 ， 並且 Nginx 也將 access log 內容成功寫入到檔案中。
