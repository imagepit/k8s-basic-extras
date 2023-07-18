# Kubernetes基礎研修のハンズオンテキスト修正点修正点

## Docker Desktopのインストール

_コマンド実行_
```bash
choco install docker-desktop
```

## kindのインストール

kindは、Kubernetesをローカル環境で実行するためのツールです。

_コマンド実行_
```bash
choco install kind
```

次のコマンドでkindのバージョンを確認し正常にインストールされていることを確認します。

_コマンド実行_
```bash
kind version
```

## kubectlのインストール

_コマンド実行_
```bash
choco install kubernetes-cli
```

次のコマンドでkubectlのバージョンを確認し正常にインストールされていることを確認します。

_コマンド実行_
```bash
kubectl version --short
```

## helmのインストール

helmは、Kubernetesのパッケージマネージャです。

_コマンド実行_
```bash
choco install kubernetes-helm
```

次のコマンドでhelmのバージョンを確認し正常にインストールされていることを確認します。

_コマンド実行_
```bash
helm version
```

## Kubernetesクラスタの構築

任意の場所にディレクトリを作成し、そのディレクトリに移動してcluster.yamlファイルを作成します。

_コマンド実行_
```bash
mkdir kind
cd kind
touch cluster.yaml
```

_kind/cluster.yaml_
```yaml
//▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼追加▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: 127.0.0.1
  apiServerPort: 6443
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
  - containerPort: 30080
    hostPort: 30080
    protocol: TCP
  extraMounts:
    - hostPath: ./data
      containerPath: /var/local-path-provisioner
- role: worker
- role: worker
kubeadmConfigPatches:
- |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
        authorization-mode: "AlwaysAllow"
//▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲追加▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲
```


## LoadBalancerの構築修正部分

## metallbのインストール

metallbはバージョンをv0.13.10に固定してインストールします。

_コマンド実行_
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

KubernetesクラスタのノードのIPアドレスを確認します。

_コマンド実行_
```bash
docker network inspect -f '{{.IPAM.Config}}' kind
[{172.19.0.0/16  172.19.0.1 map[]} {fc00:f853:ccd:e793::/64  fc00:f853:ccd:e793::1 map[]}]
```

ノードのネットワークセグメントで使用可能なIPアドレスの範囲を指定します。今回は`172.19.255.200`から`172.19.255.250`の範囲を指定します。

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.19.255.200-172.19.255.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
```

## Ingressの構築

Ingressは今回はkind用のingress-nginxを使用します。

_コマンド実行_
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

