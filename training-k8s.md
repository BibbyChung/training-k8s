---
theme: "night"
transition: "slide"
highlightTheme: "monokai"
slideNumber: true
title: "k8s training"
enableChalkboard: false
enableZoom: true
enableMenu: true
---

## k8s training

大家吃飽飽，可以開始認真

---

<!-- .slide: data-background="https://user-images.githubusercontent.com/8520661/158227110-9098ced4-64c7-4772-b290-0c36f4a59b1e.png" data-background-repeat="none" data-background-size="1px" -->

## about me

My name is Bibby. Nice to meet you.

:fa-broadcast-tower:![BB](https://user-images.githubusercontent.com/8520661/158227110-9098ced4-64c7-4772-b290-0c36f4a59b1e.png)
:fa-broadcast-tower::fa-broadcast-tower::fa-broadcast-tower:

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

## 講古

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

<small>[官網](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes)</small>

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

## 架構圖

[:fa-broadcast-tower: 點我點我 :fa-broadcast-tower:](https://user-images.githubusercontent.com/8520661/158118154-fd953506-54dc-46d2-8930-b6d5be04850e.png)

--

## Kubernetes 架構元件(client/server)

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

## 快速產生 kubernetes 測試環境(k3d)

- 產生一個 vm
- 安裝 docker/k3d 套件
- 建立一個 k3s cluster
- 建立 Kubernetes 控制環境

--

![](https://user-images.githubusercontent.com/8520661/158402814-cf9b39c6-75a2-4291-966d-5917e270baa4.png)

--

## 產生一個 vm

安裝 multipass
https://github.com/canonical/multipass

```sh
# 產生一個 vm
multipass launch \
  --name vm0 \
  --cpus 2 \
  --mem 4G \
  --disk 20G \
  --cloud-init ./cloud-init.yml

# list vm
multipass list

# into vm
multipass shell vm0
```

<small>ps: 資源給太小，會遇到有的沒有的問題，先給大點</small>

--

## 安裝 docker/k3d 套件

```sh
# 安裝 docker
curl -fsSL get.docker.com -o get-docker.sh && chmod 700 get-docker.sh && ./get-docker.sh

# test docker
docker ps

# 安裝 install k3d tools
curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash

# test k3d
k3d version
```

--

## 建立一個 k3s cluster

```sh
# create cluster
k3d cluster create abc0 \
  --agents 1 \
  --k3s-arg "--disable=traefik@server:0" \
  --api-port <vm-ip>:6550

# test cluster
k3d cluster list
```

--

## 建立 Kubernetes 控制環境

```sh
# 產生 Kubernetes config
# export KUBECONFIG=$(k3d kubeconfig get abc0)
mkdir -p ~/.kube
k3d kubeconfig get abc0 > ~/.kube/config

# check listen port
lsof -i | grep 6550

# 建置 workspace
# --platform linux/arm64 (m1)
docker run --rm -it \
  --net=host \
  -v ~/.kube/config:/root/.kube/config \
  dtzar/helm-kubectl:3.8.0 \
  bash

# test
kubectl get nodes
kubectl cluster-info
```

---

## kubernetes 工具

- kube-exporter(web)
  - https://github.com/cnrancher/kube-explorer
- Lens (application)
  - https://k8slens.dev

--

## Lens (application)

![](https://user-images.githubusercontent.com/8520661/158140073-cf23af6a-264f-4e02-81b1-8836af8abeb6.png)

--

## kube-exporter(web)

```sh
# create container
docker run --rm -it \
  -w /app \
  --net=host \
  -v ~/.kube/config:/root/.kube/config \
  node:16.14-alpine \
  sh

# install kube-explorer
apk add curl --no-cache

curl -fSL https://github.com/cnrancher/kube-explorer/releases/download/v0.2.8/kube-explorer-linux-arm64 -o kube-explorer

chmod +x ./kube-explorer

# start service
./kube-explorer \
  --kubeconfig=/root/.kube/config \
  --http-listen-port=9898 \
  --https-listen-port=0

# test
open http://<vm-ip>:9898
```

---

## 安裝 prometheus 和 grafana

- prometheus (by Lens)
- grafana (by helm)

--

## 安裝 prometheus (by Lens)

![](https://user-images.githubusercontent.com/8520661/158142469-58663153-77f7-4fcc-ad98-89355cd45c23.png)

<small>install prometheus by Lens</small>

--

## 測試 prometheus

用下列的方式就可以看到介面了

![安裝](https://user-images.githubusercontent.com/8520661/158143369-02cb38a2-d3f9-4f50-ad56-fe9550b3d38b.png)

--

## 安裝 grafana (by helm)

```sh
# create namespace
kubectl create namespace monitor

# add repo
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# fetch code
helm fetch \
  --version=6.23.1 \
  --untar \
  grafana/grafana

# install
helm install \
  -n monitor \
  --set persistence.storageClassName=local-path \
  --set persistence.enabled=true \
  --set service.type=NodePort \
  --set service.nodePort=31101 \
  grafana \
  ./grafana

helm upgrade \
  -n monitor \
  --set persistence.storageClassName=local-path \
  --set persistence.enabled=true \
  --set service.type=NodePort \
  --set service.nodePort=31101 \
  grafana \
  ./grafana

# test (因為是 k3d 所以並不能用 nodeType 去打)
kubectl port-forward service/grafana 31101:80 \
  --address 0.0.0.0 \
  -n monitor
```

--

## 測試 grafana

```sh
# get password
kubectl get secret grafana -o jsonpath="{.data.admin-password}" -n monitor | base64 -d ; echo

# open by browser
open http://<vm-ip>:31101

# dashboard from grafana <K8 Cluster Detail Dashboard>
https://grafana.com/grafana/dashboards/10856
```

---

## Q & A

問我問我

--

## docker / dock-compose / kubernetes 差異

- Docker 簡單應用程序,用於單個部署
- Docker-Compose 單機/小型機部署應用程序
- k8s 群集部署高可用性應用程序

---

## references

- [官網](https://kubernetes.io/docs/setup/)
- [k8s 入門學習 30 天](https://ithelp.ithome.com.tw/users/20111580/ironman/3931)
