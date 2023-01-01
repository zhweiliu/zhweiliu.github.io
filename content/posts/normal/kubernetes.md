---
title: 整理面試常見問題 — Kubernetes
date: '2022-12-08T06:26:47.022Z'
categories: ['Kubernetes', 'Interview']
keywords: ['Kubernetes', 'Interview']
summary: |
  記錄在過去面試中常被問到的問題，以及依據自己的理解與查找到的資料，整理對應的回覆 
  Q1. K8s 有哪些 components ?
  Q2. 部署一個應用到 K8s 時，K8s 會如何運作整個流程 ?
  Q3. 對於在 K8s 上建置正式環境 (Product) 和測試環境 (Development)的規劃
showToc: true
TocOpen: true
---

記錄在過去面試中常被問到的問題，以及依據自己的理解與查找到的資料，整理對應的回覆

Q1. K8s 有哪些 components ?

`Master Node` ← 管理 cluster、儲存不同 node 的資訊、規劃 containers 的去向，以及 monitor node 上的 containers

*   `kube-apiserver` ← 管理整個 K8s 的 interface。使用者可以透過下達指令給 kube-apiserver ，以達到管理 K8s resources 的目的。
*   `etcd cluster` ← 儲存 K8s cluster 內，所有 node 、 containers 的資訊
*   `kube-scheduler` ← 依據 containers 的需求，包含但不限於: cpu、 menory、 affinity 、 taints and tolerations 、anti-* 等等，規劃將 containers 指派給符合需求的 node。
*   `controller manager` ← 管控 K8s cluster 內的 resources 。  
    如 node controller 管理 nodes : 加入新 node 到 cluster、處理 node 變的不可用的狀況

`Work Nodes` ← 託管執行應用的 containers

*   `kubelet` ← K8s cluster 在 work nodes 上的代理，偵聽 kube-apiserver 的指令，並依據需求部署或銷毀 containers。 kube-apiserver 週期性的從kubelet 獲取 work node 和 containers 的狀態。
*   `kube-proxy` ← 讓 work node 之間的 containers 可以互相溝通。
*   `container runtime engine` ← 執行包含應用的 containers

Q2. 部署一個應用到 K8s 時，K8s 會如何運作整個流程 ?

![](/images/normal/kubernetes/image_0.png)
1.  `User` 發送 deploy request 給 `kube-apiserver`。`kube-apiserver` 驗證 User 並驗證是否為有效的 request。
2.  將有效的 request 更新到 `ETCD`。
3.  `ETCD` 回覆 `kube-apiserver` 更新已完成。
4.  `kube-scheduler` 會持續監測 `kube-apiserver`，此時發現有一個新的 `pod` 還沒有被指派到 `work node`， `kube-scheduler` 會選擇符合條件的 `work node`。
5.  `kube-scheduler` 通知 `kube-apiserver` ，規畫將新的 `pod` 分配到指定的 `work node`。
6.  `kube-apiserver` 通知 `work node` 上的 `kubelet`，有新的 `pod` 需要部署到 `work node` 上。
7.  `kubelet` 嘗試將新的 `pod` 部署到 `work node` 上，並持續監測 `pod` 的狀態。
8.  新的 `pod` 開始部署到 `work node` 。
9.  新的 `pod` 部署完成。 `kubelet` 監測到 `pod` 已部署 (但不保證 `pod` 上的應用執行是否成功)。
10.  `kubelet` 通知 `kube-apiserver` 新的 `pod` 已部署到 `work node`。
11.  `kube-apiserver` 將 `pod` 部署資訊更新到 `ETCD` 。
12.  `ETCD` 回覆 `kube-apiserver` 更新已完成。
13.  `kube-apiserver` 回覆 `User` 新的 `pod` 已部署到 `work node`。

Q3. 對於在 K8s 上建置正式環境 (Product) 和測試環境 (Development)的規劃

*   在 K8s 分別建立 Product 和 Development 的 [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)，並可透過指令 kubectl apply -f {YAML file} -n [ Product | Development ] ，將需要的 resources 部署到對應的 namespace 。
*   若需要權限管理，則透過 [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) 進行設置。我常見的作法是先建立 K8s 的 service account 、建立 Role 或 ClusterRole 並設定可操作的 resources 與對應的 verbs，最後透過 RoleBinding 或 ClusterRoleBinding 將 service account 和 Role(或ClusterRole)進行綁定。