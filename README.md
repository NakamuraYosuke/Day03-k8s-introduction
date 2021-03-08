# Day03-Kubernetesを触ってみよう
## 目標
 Day3では、Red Hat OpenShift Container Platform (RHOCP) 上で実⾏される Kubernetes クラスターのアーキテクチャについて説明します。

 Kubernetes と RHOCP によって提供される主要なリソースタイプやコンテナ、Kubernetes、RHOCP のネットワーク特性、外部アクセス可能な Pod を作成するためのメカニズムをハンズオン形式で説明します。

 ## KUBERNETES と OPENSHIFT
 Kubernetes はコンテナー化されたアプリケーションのデプロイメント、管理、およびスケーリングを簡潔化するオーケストレーションサービスであることを確認しました。
 
 Kubernetes を使⽤する主な利点の 1 つは、複数のノードを使⽤して管理対象アプリケーションの復元⼒とスケーラビリティーを確保していることです。Kubernetes はコンテナーを実⾏する⼀連のノードサーバーのクラスターを形成し、複数のマスターサーバーによって集中管理されています。
 
 サーバーはマスターとノードの両⽅の役割を果たすことができますが、安定性を向上させるため、通常は役割が分けられています。


![](https://raw.githubusercontent.com/NakamuraYosuke/Day03-k8s-introduction/main/images/ocpimage.png)

- ベース OS は Red Hat CoreOS です。Red Hat CoreOS は、コンテナー実⾏のためにイミュータブルなオペレーティングシステムを』供することに焦点を当てた Linux ディストリビューションです。

- CRI-O は、OCI (Open Container Initiative) と互換性があるランタイムの使⽤を可能にするKubernetes CRI (Container Runtime Interface) の実装です。CRI-O は、runc (Docker サービスが使⽤)、libpod (Podman が使⽤)、rkt (CoreOS から) など、CRI を満たす任意のコンテナーランタイムを使⽤できます。

- Kubernetes は、コンテナーを実⾏するホスト (物理または仮想) のクラスターを管理します。複数のリソースから構成されたマルチコンテナーアプリケーションおよびそれらの相互接続⽅法を記述するリソースを使⽤します。

- Etcd は分散キーバリューストアであり、Kubernetes クラスター内のコンテナーおよびその他のリソースについての構成情報と状態情報を保存するために Kubernetes によって使⽤されます。

- カスタムリソース定義 (CRD) は、Etcd に格納され、Kubernetes によって管理されるリソースタイプです。これらのリソースタイプは、OpenShift によって管理されるすべてのリソースの状態と構成を形成します。

- コンテナー化されたサービスにより、ネットワーキングや認可をはじめとする多数の PaaS インフラストラクチャ機能が実現されます。RHOCP では、ほとんどの内部機能⽤に Kubernetesや基盤となるコンテナーランタイムのベーシックなコンテナーインフラストラクチャを使⽤します。つまり、ほとんどの RHOCP 内部サービスは、Kubernetes によってオーケストレーションされたコンテナーとして実⾏されます。

- ランタイムおよび xPaaS は、開発者がすぐに使⽤可能なベースコンテナーイメージです。それぞれが、特定のランタイム⾔語またはデータベースを使⽤して事前設定されています。xPaaS製品は、JBoss EAP や ActiveMQ などの Red Hat ミドルウェア製品⽤のベースイメージのセットです。Red Hat OpenShift アプリケーションランタイム (RHOAR) は、OpenShift のクラウドネイティブアプリケーション⽤に最適化された⼀連のランタイムです。利⽤可能なアプリケーションランタイムは、Red Hat JBoss EAP、OpenJDK、Thorntail、Eclipse Vert.x、SpringBoot、Node.js です。

- DevOps ツールとユーザーエクスペリエンス: RHOCP はユーザーアプリケーションと RHOCPサービスを管理するための Web UI と CLI の管理ツールを』供します。OpenShift の Web UIツールと CLI ツールは、IDE や CI プラットフォームなどの外部ツールから使⽤できる REST API から構築されています。

### Kubernetesのリソースタイプ
Kubernetes には 6 つの主要リソースタイプがあり、それらは YAML ファイル、JSON ファイル、または OpenShift 管理ツールを使⽤することによって作成、管理できます。
- Pod (po)

    IP アドレスや永続的ストレージボリュームなどのリソースを共有するコンテナーの集合です。Pod は Kubernetes の基本的な作業単位です。

- サービス (svc)

    Pod プールへアクセスするための 1 つの IP とポートの組み合わせを定義します。デフォルトでは、サービスにおけるクライアントから Pod への接続はラウンドロビン形式で実施されます。

- レプリケーションコントローラー (rc)

    Pod を異なるノードに複製 (⽔平⽅向にスケーリング) する⽅法を定義する Kubernetes リソース。レプリケーションコントローラーは、Pod とコンテナーに⾼可⽤性を』供するための基本的な Kubernetes サービスです。

- 永続ボリューム (pv)

    Kubernetes Pod が使⽤するストレージエリアを定義します。

- 永続ボリューム要求 (pvc)

    Pod によるストレージの要求です。PVC は、通常コンテナーのファイルシステムにストレージをマウントすることによって、そのコンテナーが PV を利⽤できるように PV を Pod にリンクします。

- ConfigMaps (cm) と Secrets

    他のリソースが使⽤できるキーと値のセットが含まれています。ConfigMaps と Secrets は通常、複数のリソースで使⽤される設定値を⼀元管理するために使⽤されます。Secret はConfigMap のマップと異なり、Secret の値は常にエンコードされ (暗号化されていない)、そのアクセス権は少数の認証されたユーザーに制限されています。

### Kubernetes ネットワーキング
Kubernetes クラスターにデプロイされる各コンテナーは、内部ネットワークから割り当てられたIP アドレスを持ち、この IP アドレスはコンテナーを実⾏しているノードからのみアクセスできます。

コンテナーの⼀時的な性質により、IP アドレスは割り当てと解放を常に繰り返しています。

Kubernetes はソフトウェア定義ネットワーク (SDN) を提供します。SDN では複数のノードから成る内部コンテナーネットワークが⽣成され、任意の Pod の任意のホスト内のコンテナーが他のホストの Pod にアクセスできます。SDN へのアクセスは同⼀の Kubernetes クラスターからのみ可能です。

Kubernetes Pod 内のコンテナー同⼠が動的 IP アドレスによって直接に接続することはありません。

Kubernetes には、異なるワーカーの Pod 同⼠が接続することを可能にする、仮想ネットワークが、Kubernetes において、Pod が他の Pod の IP アドレスを検出することは容易ではありません。

![](https://raw.githubusercontent.com/NakamuraYosuke/Day03-k8s-introduction/main/images/k8snetwork.png)

Kubernetesのサービスでは、ある Podのコンテナーから、他の Pod のコンテナーにネットワーク接続できるようにする働きがあります。

Pod はさまざまな原因で再起動し、毎回、前回とは異なる内部 IP アドレスが付与されます。

再起動のたびに⼀⽅の Pod が他⽅の Pod の IP アドレスを検出するのではなく、サービスが他⽅の Pod に静的 IP アドレスを⽤意していれば、再起動後、どのワーカーノードで Pod が動作しても問題は⽣じません。

