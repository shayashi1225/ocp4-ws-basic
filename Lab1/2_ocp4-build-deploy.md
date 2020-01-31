# 2. コンテナイメージのビルドとデプロイ
アプリケーションをOpenShift4クラスターで動作させます。Developer Catalogのテンプレートを使用してアプリケーションをデプロイします。

- A) 新規プロジェクトの作成 [(2-3-1)](#2-3-1-プロジェクトnamespaceの作成)
- B) カタログからデプロイ [(2-3-2)](#2-3-2-カタログでソースを指定してビルドデプロイ)
- C) アプリのPod動作の確認 [(2-3-3)](#2-3-3-blogアプリケーションの状態を確認)
- D) Routerの作成 [(2-3-4)](#3-3-4-外部からアクセスするための-route-を作成)
- E) アプリの動作確認 [(2-3-5)](#3-3-5-アプリケーションの動作確認)

![](images/ocp4-Lab1-3_overview.png)

## 2-1. はじめに
### 2-1-1. OpenShift4におけるデプロイについて
OpenShiftでは，いくつかの方法でアプリケーションをクラスター上にデプロイすることができます。

- 既存のDockerイメージを使ってデプロイする方法
- ソースコードとS2I(ツール)を使ってビルド&デプロイする方法
- ソースコードとDockerfileを使ってビルド&デプロイする方法

### 2-1-2. 事前準備
- ターミナルへのアクセス情報
- OpenShift4クラスターへのアクセス情報

## 2-2. OpenShift4コンソールへのログイン
ブラウザを立ち上げて **OpenShift4コンソール** に接続します。

>ログイン方法を再確認したい場合は，[ブラウザからOpenShift4コンソールへのログイン](1_ocp4-tour.md#)を参照ください。  
## 2-3. コンテナイメージのビルドとデプロイ
S2I(Source-to-Image)というツールを使って以下の2つのコンポーネントからコンテナイメージを生成し，コンテナアプリケーションをデプロイします。

- **リポジトリURL** : GitHubなどソースコード格納場所
- **S2I Builder Image** : S2Iスクリプトが含まれているDockerイメージ

またOpenShift4は，カタログ機能(Developer Catalog)を備えています。JavaやPython，nginxなどのS2I Builder Imageをカタログ上で選択，あるいはカタログ上に追加でき，アプリケーションを簡単にOpenShift4上にデプロイできます。

新規にプロジェクトを作成し，サンプルのコードとPython用のS2I Builder Imageを使ってコンテナイメージを作成し，アプリケーションをOpenShift4上にデプロイしてみましょう。

### 2-3-1. プロジェクト(Namespace)の作成
OpenShift4コンソールで各自のプロジェクトを作成しましょう。  

1. [Home] > [Projects] > [Create Project] を選択します。  

    ![](images/ocp4-lab1-3-create-project.png)

    >コンソール右上のユーザー名が自身の<User_ID>であることを確認しましょう

    > - 他の人と重複しないプロジェクトを作成すること
    > - 作成したプロジェクト以外に触れる際は，参照系操作のみとすること

2. プロジェクト名(例: `blog-user00` )を指定し，**Create** を選択します。  
    
    プロジェクト名には，**必ずご自身のログイン時のユーザー名 (例: "blog-user00")** を指定してください。  
    複数人でクラスターを共有しているため，他の人と重複しないプロジェクト名を指定する必要があります。  

    ![](images/ocp4-lab1-3-create-project-blog.png)

    >Tips:
    >
    >OpenShift4ではプロジェクトを作成することで，新規Namespace(=プロジェクト名)が生成されます。NamespaceはK8sクラスターを論理的に分離させることが可能なK8sリソースの一種です。例えば，アプリA用のNamespaceを`ns_appa`，アプリB用のNamespaceを`ns_appb`のように作成することで，同一のK8sクラスター内に存在するns_appaとns_appbが干渉しないように構成することも可能です。

    ![](images/ocp4-lab1-3-create-project-result_4.2.png)

### 2-3-2. カタログでソースを指定してビルド&デプロイ
1. コンソールの左上からDeveloperコンソールに切り替えます。
![](images/ocp4-lab1-3-switch-devconsole_4.2.png)

2. [+Add] > [From Catalog] を選択します。
![](images/ocp4-lab1-3-select-fromcatlog_4.2.png)


3. [Catalog] > [Developer Catalog] > [Python] テンプレート を選択します。

    ![](images/ocp4-lab1-3-devcatalog-python_4.2.png)

    >Tips:
    >
    >Developer CatalogからPythonアプリケーションを作成することで以下のリソースが作成されます。
    >- Build config
    >    - Gitリポジトリからソースコードをビルド
    >- Image stream
    >    - ビルド済イメージのトラッキング
    >- Deployment config    
    >    - イメージ変更の際に新リビジョンにロールアウト
    >- Service
    >    - クラスター内にワークロードを公開
    >- Route
    >    - クラスター外にワークロードを公開(今回に限り、演習のためこの段階では作成しません。)

4. アプリケーションのリポジトリなどを指定して，OpenShift4上にアプリケーションをデプロイします。

    **[Create Application]**　を選択します。  
    ![](images/ocp4-lab1-3-devcatalog-python_create_4.2.png)
    
    次に，**[リポジトリなどいくつかの項目]** を指定し，最後に **[Create]** を選択します。  

    - Builder Image Version: `3.6`
    - Git Repo URL: `https://github.com/openshift-katacoda/blog-django-py` 
    - Application Name:`任意の名前(例: blog-user00-app)`
    - Name:`任意の名前(例: blog-user00)`
    - Create route: `デフォルト(チェックを外した状態)`

    ![](images/ocp4-lab1-3-devcatalog-python-create-repo_4.2-1.png)
    
    ![](images/ocp4-lab1-3-devcatalog-python-create-repo_4.2-2.png)

    以上の手順で，blogアプリケーションをOpenShift4上にデプロイできました。  
    
    >Tips:
    >
    >アプリケーションを作成すると下図のように[Topology]が表示され、アプリケーションの画像を選択すると左側に情報が表示されます。デプロイ直後は "0 of 1 pods" のように動作準備中の状態です。  
    >
    >![](images/ocp4-lab1-3-blog-pods-status-build_4.2.png)  
    >
    >少し待つと，下図のように状態が変化します。  
    >
    >![](images/ocp4-lab1-3-blog-pods-status-0of1_4.2.png)  
    >
    >![](images/ocp4-lab1-3-blog-pods-status-1of1_4.2.png)  
    >[Overview]を選択するとPodが一つ起動していることが確認できます。  
    >![](images/ocp4-lab1-3-blog-pods-status-overview_4.2.png)  


### 2-3-3. blogアプリケーションの状態を確認
1. [Reources]タブを選択して[指定したアプリのPod名 (例: blog-user00−1−9ndxw)] のように選択します。

    ![](images/ocp4-lab1-3-blog-pods_4.2.png)

    コンテナが作成され，起動していると以下のように表示されます。  
    
    ![](images/ocp4-lab1-3-blog-pods-status_4.2.png)

    >Tips:  
    >
    >Prometheus(+Grafana)のモニタリング状況を確認したり，yaml定義の確認，Eventの確認などができます。  
    >
    >さらに，Pod内のコンテナ内でコマンド実行も行えます。  
    >以下図のように [Terminal] を選択するとブラウザ上でターミナル内操作が行なえます。  
    >
    >![](images/ocp4-lab1-3-blog-pods-terminal_4.2.png)
    >
    >Pod内に複数コンテナが存在する場合はプルダウンメニューで選択するだけでコンテナを切替えてターミナル操作が可能です。問題判別を行う際には，手間を省いてくれる意外と嬉しい機能です。

### 2-3-4. 外部からアクセスするための Route を作成
現在のblogアプリケーションは，OpenShift4クラスター内に閉じた状態ですので，外部からアクセスできるように Router を作成しましょう。
1. コンソールの左上からAdministtratorコンソールに切り替えます。
![](images/ocp4-lab1-3-switch-adminconsole_4.2.png)

2. [Networking] > [Routes] > [Create Route] を選択します。

    ![](images/ocp4-lab1-3-blog-create-route_4.2.png)

3. **Router**，対象アプリ用の**Service**，**Port** を指定します。
    - Name: `任意の名前 (例: blog-user00)`
    - Service: `指定済のアプリ名 (例: blog-user00`
    - Target Port: `8080 → 8080(TCP)`
    
    ![](images/ocp4-lab1-3-blog-create-route-2_4.2.png)
    
    >Tips:
    >
    >「あれ？Service作ったっけ？」と思われた方，その感覚は正しいです。明示的には作成していません。  
    >今回は[3-3-2の手順]で，Pythonテンプレートでblogアプリケーションをデプロイした際に，Podだけでなく，"Service" も同時に作成されています。
    >その際，Service名はアプリ名と同じ名前が指定されています。
    >
    >Developer Catalogで選択したテンプレートは，Kubernetes上でアプリを動作させるために必ず必要になるリソース(PodやServiceなど)や，便利にアプリケーションを管理できるようにするための仕組みを一挙に作成できるように用意されています。
    >
    
4. 最後に **Create** を選択します。

    >Tips:
    >
    >作成したRouteを参照する場合は，[Networking] > [Routes] > [Router名] のように辿ることで確認できます。 
    >
    >![](images/ocp4-lab1-3-blog-route-detail_4.2.png)
    
### 2-3-5. アプリケーションの動作確認
1. [Networking] > [Routes] を選択し，blog用のRouter(例: `blog-user00`)行にある **Location欄のリンク** を開きます。
    例) `http://blog-user00-blog-user00.apps.group00-ocp4ws-basic.capsmalt.org`

    ![](images/ocp4-lab1-3-blog-confirm-app_4.2.png)

2. blogアプリのサンプルページに自身のPod名が表示されていることを確認します。

    ![](images/ocp4-lab1-3-blog-confirm-app-result.png)
    
    Pod名が分からない場合は，[Workloads] > [Pods] のPod一覧から確認しましょう。  
    
    ![](images/ocp4-lab1-3-blog-confirm-app-pod.png)


## 2-4. [Trial works] - OpenShift4クラスターに他アプリケーションをデプロイ
お題: 

「**OpenShift4クラスターに他アプリケーションをS2Iでビルド&デプロイしてみよう**」

コンテンツ:


- Project名(NameSpace): `trial-<yourID>` (例: `trial-user00`)
- BaseImage(BuilderImage): `Python 3.6`
- Git Repository: `https://github.com/sclorg/django-ex`
- Routes名: `trial-<yourID>` 

---
以上で，コンテナイメージのビルドとデプロイ は完了です。[Topへ](../README.md)  
