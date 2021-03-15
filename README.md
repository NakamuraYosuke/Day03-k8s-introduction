# Day03-Kubernetesを触ってみよう
## 目標
 Day3では、Red Hat OpenShift Container Platform (RHOCP) 上で実⾏される Kubernetes クラスターのアーキテクチャについて説明します。

 Kubernetes と RHOCP によって提供される主要なリソースタイプやコンテナ、Kubernetes、RHOCP のネットワーク特性、外部アクセス可能な Pod を作成するためのメカニズムをハンズオン形式で説明します。

 ## KUBERNETES と OPENSHIFT
 Kubernetes はコンテナー化されたアプリケーションのデプロイメント、管理、およびスケーリングを簡潔化するオーケストレーションサービスであることを確認しました。
 
 Kubernetes を使⽤する主な利点の 1 つは、複数のノードを使⽤して管理対象アプリケーションの復元⼒とスケーラビリティーを確保していることです。Kubernetes はコンテナーを実⾏する⼀連のノードサーバーのクラスターを形成し、複数のマスターサーバーによって集中管理されています。
 
 サーバーはマスターとノードの両⽅の役割を果たすことができますが、安定性を向上させるため、通常は役割が分けられています。


![](https://raw.githubusercontent.com/NakamuraYosuke/Day03-k8s-introduction/main/images/ocpimage.png)

- ベース OS は Red Hat CoreOS です。Red Hat CoreOS は、コンテナー実⾏のためにイミュータブルなオペレーティングシステムを提供することに焦点を当てた Linux ディストリビューションです。

- CRI-O は、OCI (Open Container Initiative) と互換性があるランタイムの使⽤を可能にするKubernetes CRI (Container Runtime Interface) の実装です。CRI-O は、runc (Docker サービスが使⽤)、libpod (Podman が使⽤)、rkt (CoreOS から) など、CRI を満たす任意のコンテナーランタイムを使⽤できます。

- Kubernetes は、コンテナーを実⾏するホスト (物理または仮想) のクラスターを管理します。複数のリソースから構成されたマルチコンテナーアプリケーションおよびそれらの相互接続⽅法を記述するリソースを使⽤します。

- Etcd は分散キーバリューストアであり、Kubernetes クラスター内のコンテナーおよびその他のリソースについての構成情報と状態情報を保存するために Kubernetes によって使⽤されます。

- カスタムリソース定義 (CRD) は、Etcd に格納され、Kubernetes によって管理されるリソースタイプです。これらのリソースタイプは、OpenShift によって管理されるすべてのリソースの状態と構成を形成します。

- コンテナー化されたサービスにより、ネットワーキングや認可をはじめとする多数の PaaS インフラストラクチャ機能が実現されます。RHOCP では、ほとんどの内部機能⽤に Kubernetesや基盤となるコンテナーランタイムのベーシックなコンテナーインフラストラクチャを使⽤します。つまり、ほとんどの RHOCP 内部サービスは、Kubernetes によってオーケストレーションされたコンテナーとして実⾏されます。

- ランタイムおよび xPaaS は、開発者がすぐに使⽤可能なベースコンテナーイメージです。それぞれが、特定のランタイム⾔語またはデータベースを使⽤して事前設定されています。xPaaS製品は、JBoss EAP や ActiveMQ などの Red Hat ミドルウェア製品⽤のベースイメージのセットです。Red Hat OpenShift アプリケーションランタイム (RHOAR) は、OpenShift のクラウドネイティブアプリケーション⽤に最適化された⼀連のランタイムです。利⽤可能なアプリケーションランタイムは、Red Hat JBoss EAP、OpenJDK、Thorntail、Eclipse Vert.x、SpringBoot、Node.js です。

- DevOps ツールとユーザーエクスペリエンス: RHOCP はユーザーアプリケーションと RHOCPサービスを管理するための Web UI と CLI の管理ツールを提供ƒします。OpenShift の Web UIツールと CLI ツールは、IDE や CI プラットフォームなどの外部ツールから使⽤できる REST API から構築されています。

### Kubernetesのリソースタイプ
Kubernetes には 6 つの主要リソースタイプがあり、それらは YAML ファイル、JSON ファイル、または OpenShift 管理ツールを使⽤することによって作成、管理できます。
- Pod (po)

    IP アドレスや永続的ストレージボリュームなどのリソースを共有するコンテナーの集合です。Pod は Kubernetes の基本的な作業単位です。

- サービス (svc)

    Pod プールへアクセスするための 1 つの IP とポートの組み合わせを定義します。デフォルトでは、サービスにおけるクライアントから Pod への接続はラウンドロビン形式で実施されます。

- レプリケーションコントローラー (rc)

    Pod を異なるノードに複製 (⽔平⽅向にスケーリング) する⽅法を定義する Kubernetes リソース。レプリケーションコントローラーは、Pod とコンテナーに⾼可⽤性を提供するための基本的な Kubernetes サービスです。

- 永続ボリューム (pv)

    Kubernetes Pod が使⽤するストレージエリアを定義します。

- 永続ボリューム要求 (pvc)

    Pod によるストレージの要求です。PVC は、通常コンテナーのファイルシステムにストレージをマウントすることによって、そのコンテナーが PV を利⽤できるように PV を Pod にリンクします。

- ConfigMaps (cm) と Secrets

    他のリソースが使⽤できるキーと値のセットが含まれています。ConfigMaps と Secrets は通常、複数のリソースで使⽤される設定値を⼀元管理するために使⽤されます。Secret はConfigMap のマップと異なり、Secret の値は常にエンコードされ (暗号化されていない)、そのアクセス権は少数の認証されたユーザーに制限されています。

### Kubernetes ネットワーキング
Kubernetes クラスターにデプロイされる各コンテナーは、内部ネットワークから割り当てられたIPアドレスを持ち、このIP アドレスはコンテナーを実⾏しているノードからのみアクセスできます。

コンテナーの⼀時的な性質により、IP アドレスは割り当てと解放を常に繰り返しています。

Kubernetes はソフトウェア定義ネットワーク (SDN) を提供します。SDN では複数のノードから成る内部コンテナーネットワークが⽣成され、任意の Pod の任意のホスト内のコンテナーが他のホストの Pod にアクセスできます。SDN へのアクセスは同⼀の Kubernetes クラスターからのみ可能です。

Kubernetes Pod 内のコンテナー同⼠が動的 IP アドレスによって直接に接続することはありません。

![](https://raw.githubusercontent.com/NakamuraYosuke/Day03-k8s-introduction/main/images/k8snetwork.png)

Kubernetesのサービスでは、ある Podのコンテナーから、他の Pod のコンテナーにネットワーク接続できるようにする働きがあります。

Pod はさまざまな原因で再起動し、毎回、前回とは異なる内部 IP アドレスが付与されます。

再起動のたびに⼀⽅の Pod が他⽅の Pod の IP アドレスを検出するのではなく、サービスが他⽅の Pod に静的 IP アドレスを⽤意していれば、再起動後にどのワーカーノードで Pod が動作しても問題は⽣じません。

![](https://raw.githubusercontent.com/NakamuraYosuke/Day03-k8s-introduction/main/images/k8ssvcnetwork.png)

アプリケーションの多くは、単⼀のPodとしては動作しません。

ユーザーの要求が増えて⽔平拡張の必要が⽣じると、同じ Pod のリソース定義に基づき、多くの Pod 上で同じコンテナーを実⾏するようになります。

サービスは⼀連の Pod を束ねて単⼀の IP アドレスを割り当て、クライアントから要求があると、各 Pod に負荷を分散します。

## APIリソースとkubectl
### Kubernetesの基礎
Kubernetesは、Kubernetes MasterとKubernetes Nodeの2種類のノードから成り立っています。
Kubernetes Master はAPIエンドポイントの提供、コンテナのスケジューリング、コンテナのスケーリングなどを担うノードです。
もう一方のKubernetes Nodeは、いわゆるDockerホストに相当し、実際にコンテナが起動するノードです。
![](https://raw.githubusercontent.com/NakamuraYosuke/Day03-k8s-introduction/main/images/kubernetesmasterandnode.png)

Kubernetesクラスタを操作するには、CLIツールのkubectlとYAML形式かJSON形式で書かれたマニフェストファイルを用いて、Kubernetes Masterにリソースの登録をおこないます。

### Kubernetesとリソース
Kubernetesのリソースの種類を下記に示します。

|  リソース種別  |  概要  |
| ---- | ---- |
|  Workloads  |  コンテナの実行に関するリソース  |
|  Discovery&LB  |  コンテナを外部公開するようなエンドポイントを提供するリソース  |
|  Config&Storage  |  設定/機密情報/永続化ボリュームなどに関するリソース  |
|  Cluster  |  セキュリティやクォータなどに関するリソース  |
|  Metadata  |  クラスタ内の他のリソースを操作するリソース |


#### Workloadsリソース
Workloadsリソースは、クラスタ上にコンテナを起動させるために利用するリソースです。

- Pod
- ReplicationController
- ReplicaSet
- Deployment
- DaemonDet
- StatefulSet
- Job
- CronJob

#### Discovery&LBリソース
Discovery&LBリソースは、コンテナのサービスディスカバリや、クラスタの外部からもアクセス可能なエンドポイントを提供するリソースです。
ServiceとIngressの２種類のDiscovery＆LBリソースがあります。
Serviceには、下記の種類のタイプが存在します。

- ClusterIP
- ExternalIP
- NodePort
- LoadBalancer
- Headless
- ExternalName
- None-Selector

#### Confit&Storageリソース
Confit&Storageリソースは、設定や機密データをコンテナに埋め込んだり永続ボリュームを提供するリソースです。
SecretとConfigMapはいずれもKey-Valueのデータ構造を持ち、保存したいデータが機密データなのか、一般的な設定情報なのかによって使い分けます。

- Secret
- ConfigMap
- PersistentVolumeClaim

#### Clusterリソース
Clusterリソースは、クラスタ自体の振る舞いを定義するリソースです。
セキュリティ周りの設定やポリシー、クラスタの管理性を高める機能のためのリソースなど様々なリソースがあります。

- Node
- Namespace
- PersistentVolume
- ResourceQuota
- ServiceAccount
- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding
- NetworkPolicy

#### Metadataリソース
Metadataリソースは、クラスタ内の他のリソースの動作を制御するためのリソースです。
例えば、Podをオートスケーリングさせるために利用されるHorizontalPodAutoscalerは、Deploymentリソースを操作してレプリカ数を変更することでオートスケーリングを実現しています。

- LimitRange
- HorizontalPodAutoscaler
- CustomResourceDefinition

### Namecpaceによる仮想的なクラスタの分離
Kubernetesには、Namespaceと呼ばれる仮想的なKubernetesクラスタの分離機能があります。
完全な分離レベルではないため使い所は限られますが、１つのKubernetesクラスタを複数チームで利用したりプロダクション環境/ステージング環境/開発環境などのように環境ごとに分離したりすることも可能です。

### kubectlを使ってみよう
kubectlツールがKubernetes Masterと通信する際には、接続先サーバの情報や認証情報が必要となります。
kubectlはkubeconfit（デフォルトでは、`~/.kube/config`に書かれている情報をしようして接続します。

```
$ more ~/.kube/config

apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://api.crc.testing:6443
  name: api-crc-testing:6443
- cluster:
    certificate-authority-data: ・・・
contexts:
- context:
    cluster: api-crc-testing:6443
    user: developer/api-crc-testing:6443
  name: /api-crc-testing:6443/developer
- context:
    cluster: api.crc.testing:6443
    namespace: default
    user: kubeadmin
  name: crc-admin
- context:
    cluster: api.crc.testing:6443
    namespace: default
    user: developer
  name: crc-developer
kind: Config
preferences: {}
users:
- name: developer
  user:
    token: sha256~_・・・
- name: developer/api-crc-testing:6443
  user:
    token: sha256~・・・
- name: kube:admin/api-crc-testing:6443
  user:
    token: sha256~・・・
- name: kubeadmin
  user:
    token: sha256~・・・
```

OpenShift（CRC）をインストールした際に、`crc-admin`と`crc-developer`というコンテキストが作成されています。

ますはkubeadminでログインしてみましょう。
```
$ kubectl config use-context crc-admin
```

プロジェクトの一覧を表示してみます。
```
$ kubectl get project
```
下記のように一覧が表示されるはずです。
```
NAME                                               DISPLAY NAME   STATUS
default                                                           Active
kube-node-lease                                                   Active
kube-public                                                       Active
kube-system                                                       Active
openshift                                                         Active
・・・
```

namespaceを作成します。
```
$ kubectl create namespace kubetest
```
kubetestのコンテキストを作成します。
```
$ kubectl config set-context kubetest/api-crc-testing:6443/kube:admin \
--cluster=api-crc-testing:6443 \
--user=kubeadmin \
--namespace=kubetest
```
作成したkubetestのnamespaceに切り替えます。
```
$ kubectl config use-context kubetest/api-crc-testing:6443/kube:admin
```
kubetestのnamespaceに変わっていることを確認します。
```
$ kubectl config get-context
```

### マニフェストとリソースの作成
kubectlが利用できるようになったところで、マニフェストを使ってコンテナを起動してみます。
今回は１つのコンテナからなるPodを「sample-pod」という名前で作成してみます。

-- sample-pod.yaml --
```
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:1.12
```

上記のsample-pod.yamlのマニフェストからPodを作成します。
```
$ kubectl create -f sample-pod.yaml
```

コンテナの起動状態を確認します。
```
$ kubectl get pods
```
作成中の場合は、
```
NAME         READY   STATUS              RESTARTS   AGE
sample-pod   0/1     ContainerCreating   0          10s
```
と表示され、正常に作成が完了すると
```
NAME         READY   STATUS    RESTARTS   AGE
sample-pod   1/1     Running   0          18s
```
上記のようにRunningとなります。

次に、マニフェストを更新します。
nginxのコンテナイメージタグを1.12から1.13に変更します。

```
< image: nginx:1.12
---
> image: nginx:1.13
```

変更を適用します。
```
$ kubectl apply -f sample-pod.yaml
```
`pod/sample-pod configured`と表示されます。

sample-podの利用しているイメージを確認します。
```
$ kubectl get pods sample-pod -o jsonpath="{.spec.containers[].image}"
```

sample-podの詳細を確認します。
```
$ kubectl describe pod sample-pod
```

### 1つのマニフェストの中に複数のリソースを記述する
先ほどのマニフェストでは、１つのリソースのみ定義しましたが、複数のリソースを１つのマニフェストに記述することもできます。

次はPodを起動するWorkloadsリソースと、そのWorkloadsリソースを外部公開するDiscovery&LBリソースを同一のマニフェスト内に記述します。

複数のリソースを定義する際は、「---」で区切るように記述します。

なお、Discovery&LBリソースはServiceとして定義します。

- ClusterIP (既定値) - クラスター内の内部IPでServiceを公開します。この型では、Serviceはクラスター内からのみ到達可能になります。

- NodePort - NATを使用して、クラスター内の選択された各ノードの同じポートにServiceを公開します。
<NodeIP>:<NodePort>を使用してクラスターの外部からServiceにアクセスできるようにします。これはClusterIPのスーパーセットです。

- LoadBalancer - 現在のクラウドに外部ロードバランサを作成し(サポートされている場合)、Serviceに固定の外部IPを割り当てます。これはNodePortのスーパーセットです。

- ExternalName - 仕様のexternalNameで指定した名前のCNAMEレコードを返すことによって、任意の名前を使ってServiceを公開します。プロキシは使用されません。このタイプはv1.7以上のkube-dnsを必要とします。

-- sample-multi-resource-manifest.yaml --
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginxinc/nginx-unprivileged:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  ports:
  - name: "http-port"
    protocol: "TCP"
    port: 8080
    targetPort: 80
  selector:
    app: nginx
```

マニフェストからリソースを作成します。
```
$ kubectl apply -f sample-multi-resource-manifest.yaml
```

Podが正常に作られているかを確認します。
```
$ kubectl get pod
```

Serviceが正常に作られているかを確認します。
```
$ kubectl get svc
```

今回はServiceのtypeとしてLoadBalancerを選択しました。
せっかくなので、NodePortやClusterIPを選択してnginxアプリケーションを外部に公開してみます。

DeploymentからServiceを作成する際には`expose`オプションを利用します。
NodePortのServiceを作成してみましょう。
```
$ kubectl expose deployment nginx-deployment --type NodePort --port 8080
```

```
$ kubectl get svc nginx-deployment
NAME               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
nginx-deployment   NodePort   10.217.4.181   <none>        8080:32345/TCP   10m
```

上記の例では、`32345`のポート番号がNodeに割り当てられた番号であり、`8080`のポート番号がnginxアプリケーションが待ち受ける番号となります。

OpenShift（CRC）のIPを用いて、アクセスしてみましょう。

```
$ crc ip
192.168.64.2
```

ブラウザのアドレス欄に、`http://192.168.64.2:32345`と入力するか、
```
$ curl http://192.168.64.2:32345
```
と入力し、`Welcome to nginx!`と表示されればOKです。


続いて、ServiceのtypeがClusterIPの場合も見てみましょう。
```
$ kubectl expose deployment nginx-deployment --type=ClusterIP --port=8080
```

ClusterIPをタイプに選択した場合、クラスター内からのみアクセス可能となるためこのままではアクセスできません。

そこでOpenShiftからrouteを作成してみます。

OpenShiftにログインします。

```
$ eval $(crc oc-env)
$ oc login -u kubeadmin -p XXXXX https://api.crc.testing:6443
```

routeを作成します。
```
$ oc expose svc nginx-deployment
route.route.openshift.io/nginx-deployment exposed
```

routeを確認します。
```
$ oc get route
NAME               HOST/PORT                                    PATH   SERVICES           PORT   TERMINATION   WILDCARD
nginx-deployment   nginx-deployment-kubetest.apps-crc.testing          nginx-deployment   8080                 None
```

HOST列に示されたノードにアクセスします。
```
$ curl http://nginx-deployment-kubetest.apps-crc.testing
```
もしくは、ブラウザのアドレス欄に、`http://nginx-deployment-kubetest.apps-crc.testing`と入力します。

先ほどのNodePortと同様、`Welcome to nginx!`と表示されればOKです。

本日は以上で終了です。
