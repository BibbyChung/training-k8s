---
theme: "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "VSCode Reveal intro"
enableZoom: true
---

## k8s trainning

大家吃飽飽，可以開始認真

---

## agenda

- kubernetes 簡單介紹
- 快速產生 kubernetes 環境
- kubernetes 工具
- 安裝 prometheus 和 grafana
- Q & A

---

## kubernetes 簡單介紹

--

![](https://user-images.githubusercontent.com/8520661/158112714-f9992639-ab80-4010-a3ca-7328f8652db7.png)

<small>https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/</small>

--

## Kubernetes(Container Orchestration)

- 服務發現和負載平衡
- 儲存設定
- 自動部署和回滾
- 自動完成包裝計算
- 自動修復
- 密鑰和配置管理

<!-- <small>[參考](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes)</small> -->

<!-- ## 服務發現和負載平衡

Kubernetes 可以使用 DNS 名稱或自己的 IP 地址來披露容器。如果流入容器的流量很大,則 Kubernetes 可以加載餘額並分配網絡流量,從而穩定部署

## 儲存設定

Kubernetes 允許您自動發布您選擇的存儲系統,例如本地存儲,公共雲提供商等

## 自動部署和回滾

您可以使用 Kubernetes 來描述已部署容器的要求狀態,該狀態可以以受控速率將實際狀態更改為預期狀態。例如,您可以自動化 Kubernetes 以創建用於部署的新容器,刪除現有容器並將其所有資源用於新容器。

## 自動完成包裝計算

Kubernetes 允許您指定每個容器所需的 CPU 和內存(RAM)。當容器指定資源請求時,Kubernetes 可以做出更好的決策來管理容器的資源

## 自動修復

Kubernetes 重新啟動失敗的容器,替換容器,殺死未按用戶定義的操作狀態檢查的容器,並且在服務準備就緒之前不會將其通知客戶

## 密鑰和配置管理

Kubernetes 允許您存儲和管理敏感信息,例如密碼,OAuth 令牌和 ssh 鍵。您可以在不重建容器鏡像的情況下部署和更新密鑰和應用程序配置,並且無需在堆棧配置中公開密鑰 -->

--

## Kubernetes (白話說)

- 對於執行環境的相依性降低
- 降低 se/sre 自動化工作量

--

[架構圖](https://user-images.githubusercontent.com/8520661/158118154-fd953506-54dc-46d2-8930-b6d5be04850e.png)

--

## 架構元件

- etcd
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- kubelet
- kube-proxy

<!--
- etcd 等保留了整個集群的狀態
- kube-apiserver
提供了資源操作的唯一入口,並提供了認證,授權,訪問控制,API註冊和發現的機制

- kube-controller-manager
負責維護集群的狀態,例如故障檢測,自動擴展,滾動更新等。

- kube-scheduler
負責根據預定的調度策略將Pod安排到相應的計算機的資源調度和調度

- kubelet
負責維護容器的生命週期,還負責音量(CVI)和網絡(CNI)的管理。容器運行時負責鏡像管理以及Pod和容器(CRI)的實際操作,並且默認容器運行時為Docker

- kube-proxy
負責為服務提供集群內部服務發現和負載平衡
-->

--

## 建構元件(Building Blocks)

元件很多，首先你先認識 pod 就好了

--

## pod

Kubernetes 中最小單位的物件，也是 Kubernetes 中部署的基本單位。你也可以把 Pod 想成是應用程式的一個 instance 。而一個 Pod 會運行一個到多個容器 (container)，而運行在同一個 Pod 中的容器會

---

## 快速產生 kubernetes 測試環境

- 產生一個 vm
- 安裝 docker/k3d 套件
- 建立一個 k3s cluster
- 建立 Kubernetes 控制環境

--

## 產生一個 vm

安裝 multipass
https://github.com/canonical/multipass

```shell
# 產生一個 vm
multipass launch \
  --name vm0 \
  --cpus 2 \
  --mem 4G \
  --disk 20G \
  --cloud-init ./cloud-init.yaml

# into vm
multipass shell vm0
```

<small>ps: 資源給太小，會遇到有的沒有的問題，先給大點</small>

--

## 安裝 docker/k3d 套件

```sh
# 安裝 docker
curl -fsSL get.docker.com -o get-docker.sh && chmod 700 get-docker.sh && ./get-docker.sh

# 安裝 install k3d tools
curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash

```

--

## 建立一個 k3s cluster

```shell
k3d cluster create bb \
  --agents 1 \
  --k3s-arg "--disable=traefik@server:0" \
  --api-port <vm-ip>:6550

# test
k3d cluster list
```

--

## 建立 Kubernetes 控制環境

```shell
# 產生 Kubernetes config
# export KUBECONFIG=$(k3d kubeconfig get bb)
mkdir -p ~/.kube
k3d kubeconfig get bb > ~/.kube/config

# 建置 workspace
# --platform linux/arm64 (m1)
docker run --rm -it \
  --net=host \
  -v ~/.kube/config:/root/.kube/config \
  dtzar/helm-kubectl:3.8.0 \
  bash

# test
kubectl cluster-info
kubectl get nodes
```

---

## kubernetes 工具

- kube-exporter(web)

  - https://github.com/cnrancher/kube-explorer

- lens (application)
  - https://k8slens.dev

--

## lens (application)

![](https://user-images.githubusercontent.com/8520661/158140073-cf23af6a-264f-4e02-81b1-8836af8abeb6.png)

--

## kube-exporter(web)

```shell
curl -fSL https://github.com/cnrancher/kube-explorer/releases/download/v0.2.8/kube-explorer-linux-amd64 -o kube-explorer

chrom +x ./kube-explorer

./kube-explorer \
  --kubeconfig=/root/.kube/config \
  --http-listen-port=9898 \
  --https-listen-port=0
```
