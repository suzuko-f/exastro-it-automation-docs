.. raw:: html

   <script>
   $(window).on('load', function () {
      setTimeout(function(){
        for (var i = 0; i < $("table.filter-table").length; i++) {
          $("[id^='ft-data-" + i + "-2-r']").removeAttr("checked");
          $("[id^='select-all-" + i + "-2']").removeAttr("checked");
          $("[id^='ft-data-" + i + "-2-r'][value^='可']").prop('checked', true);
          $("[id^='ft-data-" + i + "-2-r'][value*='必須']").prop('checked', true);
          tFilterGo(i);
        }
      },200);
   });
   </script>

===================================
Exastro IT Automation on Kubernetes
===================================

目的
====

| 本章では、Exastro IT Automation を利用する際に必要となる、Exastro Platform および Exastro IT Automation を導入する手順について説明します。

前提条件
========

- クライアント要件

  | 動作確認が取れているクライアントアプリケーションのバージョンは下記のとおりです。
  
  .. csv-table:: クライアント要件
   :header: アプリケーション, バージョン
   :widths: 30, 30
  
   Helm, v3.9.x
   kubectl, 1.23

- デプロイ環境

  | 動作確認が取れているコンテナ環境の最小要求リソースとバージョンは下記のとおりです。

  .. csv-table:: デプロイ環境
   :header: リソース種別, 要求リソース
   :widths: 20, 20
  
   CPU,2 Cores (3.0 GHz)
   Memory, 4GB
   Storage (Container image size),10GB
   Kubernetes, 1.23 以上

  .. warning::
    | 要求リソースは Exastro IT Automation のコア機能に対する値です。同一クラスタ上に Keycloak や MariaDB などの外部ツールをデプロイする場合は、その分のリソースが別途必要となります。
    | データベースおよびファイルの永続化のために、別途ストレージ領域を用意する必要があります。
    | Storage サイズには、Exastro IT Automation が使用する入出力データのファイルは含まれていないため、利用状況に応じて容量を見積もる必要があります。

- 通信要件

  - クライアントからデプロイ先のコンテナ環境にアクセスできる必要があります。
  - Platform 管理者用と一般ユーザー用の2つ通信ポートが使用となります。
  - コンテナ環境からコンテナイメージの取得のために、Docker Hub に接続できる必要があります。

- 外部コンポーネント

  - MariaDB、もしくは、MySQL サーバ
  - GitLab リポジトリ、および、アカウントの払い出しが可能なこと

  .. warning::
    | GitLab 環境を同一クラスタに構築する場合は、GitLab のシステム要件に対応する最小要件を追加で容易する必要があります。
    | Database 環境を同一クラスタに構築する場合は、使用する Database のシステム要件に対応する最小要件を定義する必要があります


インストールの準備
==================

1. Helm リポジトリの登録
------------------------

| Exastro Suite は、以下の2つのアプリケーションから構成されています。
| Exastro の全ツールは同一の Helm リポジトリ上に存在しています。

- 共通基盤 (Exastro Platform)
- Exastro IT Automation

.. csv-table::
 :header: リポジトリ
 :widths: 50

 https://exastro-suite.github.io/exastro-helm/

.. code-block:: shell
   :linenos:
   :caption: コマンド

   # Exastro Suite の Helm リポジトリを登録
   helm repo add exastro https://exastro-suite.github.io/exastro-helm/ -n exastro
   # リポジトリ情報の更新
   helm repo update

2. デフォルト設定値の取得
-------------------------

| 投入するパラメータを管理しやすくするために、下記のコマンドから共通基盤 values.yaml のデフォルト値を出力します。

.. code-block:: shell
   :caption: コマンド

   helm show values exastro/exastro > exastro.yaml

.. raw:: html

   <details>
     <summary>exastro.yaml</summary>

.. code-block:: yaml
   :linenos:

   # Default values for Exastro.
   # This is a YAML-formatted file.
   # Declare variables to be passed into your templates.
   global:
     itaGlobalDefinition:
       name: ita-global
       enabled: true
       image:
         registry: "docker.io"
         organization: exastro
         package: exastro-it-automation
       config:
         DEFAULT_LANGUAGE: "ja"
         LANGUAGE: "en"
         CONTAINER_BASE: "kubernetes"
         TZ: "Asia/Tokyo"
         STORAGEPATH: "/storage/"
       secret:
         ENCRYPT_KEY: ""
       persistence:
         enabled: true
         accessMode: ReadWriteMany
         size: 10Gi
         volumeType: hostPath # e.g.) hostPath or AKS
         storageClass: "-" # e.g.) azurefile or - (None)
         # matchLabels:
         #   release: "stable"
         # matchExpressions:
         #   - {key: environment, operator: In, values: [dev]}
     gitlabDefinition:
       name: gitlab
       enabled: true
       config:
         GITLAB_PROTOCOL: "http"
         GITLAB_HOST: "gitlab"
         GITLAB_PORT: "80"
       secret:
         GITLAB_ROOT_TOKEN: ""
     itaDatabaseDefinition:
       name: ita-database
       enabled: true
       config:
         DB_VENDOR: "mariadb"
         DB_HOST: "mariadb"
         DB_PORT: "3306"
         DB_DATABASE: "ITA_DB"
       secret:
         DB_ADMIN_USER: ""
         DB_ADMIN_PASSWORD: ""
         DB_USER: ""
         DB_PASSWORD: ""
     pfGlobalDefinition:
       name: pf-global
       enabled: true
       image:
         registry: "docker.io"
         organization: exastro
         package: exastro-platform
       config:
         DEFAULT_LANGUAGE: "ja"
         LANGUAGE: "en"
         TZ: "Asia/Tokyo"
         PYTHONIOENCODING: utf-8
         PLATFORM_API_PROTOCOL: "http"
         PLATFORM_API_HOST: "platform-api"
         PLATFORM_API_PORT: "8000"
         PLATFORM_WEB_PROTOCOL: "http"
         PLATFORM_WEB_HOST: "platform-web"
         PLATFORM_WEB_PORT: "8000"
       secret:
         ENCRYPT_KEY: ""
       persistence:
         enabled: true
         accessMode: ReadWriteMany
         size: 10Gi
         volumeType: hostPath # e.g.) hostPath or AKS
         storageClass: "-" # e.g.) azurefile or - (None)
         # matchLabels:
         #   release: "stable"
         # matchExpressions:
         #   - {key: environment, operator: In, values: [dev]}
     keycloakDefinition:
       name: keycloak
       enabled: true
       config:
         API_KEYCLOAK_PROTOCOL: "http"
         API_KEYCLOAK_HOST: "keycloak"
         API_KEYCLOAK_PORT: "8080"
         KEYCLOAK_PROTOCOL: "http"
         KEYCLOAK_HOST: "keycloak"
         KEYCLOAK_PORT: "8080"
         KEYCLOAK_MASTER_REALM: "master"
         KEYCLOAK_DB_DATABASE: "keycloak"
       secret:
         KEYCLOAK_USER: ""
         KEYCLOAK_PASSWORD: ""
         KEYCLOAK_DB_USER: ""
         KEYCLOAK_DB_PASSWORD: ""
     itaDefinition:
       name: ita
       enabled: true
       config:
         ITA_WEB_PROTOCOL: "http"
         ITA_WEB_HOST: "ita-web-server"
         ITA_WEB_PORT: "8000"
         ITA_API_PROTOCOL: "http"
         ITA_API_HOST: "ita-api-organization"
         ITA_API_PORT: "8080"
         ITA_API_ADMIN_PROTOCOL: "http"
         ITA_API_ADMIN_HOST: "ita-api-admin"
         ITA_API_ADMIN_PORT: "8080"
     pfDatabaseDefinition:
       name: pf-database
       enabled: true
       config:
         DB_VENDOR: "mariadb"
         DB_HOST: "mariadb"
         DB_PORT: "3306"
         DB_DATABASE: "platform"
       secret:
         DB_ADMIN_USER: ""
         DB_ADMIN_PASSWORD: ""
         DB_USER: ""
         DB_PASSWORD: ""
     databaseDefinition:
       name: mariadb
       enabled: true
       secret:
         MARIADB_ROOT_PASSWORD: ""
       persistence:
         enabled: true
         reinstall: false
         accessMode: ReadWriteOnce
         size: 20Gi
         volumeType: hostPath # e.g.) hostPath or AKS
         storageClass: "-" # e.g.) azurefile or - (None)
         # matchLabels:
         #   release: "stable"
         # matchExpressions:
         #   - {key: environment, operator: In, values: [dev]}
   
   exastro-it-automation:
     ita-api-admin:
       replicaCount: 1
       image:
         repository: "exastro/exastro-it-automation-api-admin"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
       extraEnv:
         PLATFORM_API_HOST: "platform-api"
         PLATFORM_API_PORT: "8000"
   
     ita-api-organization:
       replicaCount: 1
       image:
         repository: "exastro/exastro-it-automation-api-organization"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
       extraEnv:
         PLATFORM_API_HOST: "platform-api"
         PLATFORM_API_PORT: "8000"
   
     ita-by-ansible-execute:
       replicaCount: 1
       image:
         repository: "exastro/exastro-it-automation-by-ansible-execute"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
       extraEnv:
         EXECUTE_INTERVAL: "10"
         ANSIBLE_AGENT_IMAGE: "exastro/exastro-it-automation-by-ansible-agent"
         ANSIBLE_AGENT_IMAGE_TAG: "2.0.3-beta1"
       serviceAccount:
         create: false
         name: "ita-by-ansible-execute-sa"
   
     ita-by-ansible-legacy-role-vars-listup:
       replicaCount: 1
       extraEnv:
         EXECUTE_INTERVAL: "10"
       image:
         repository: "exastro/exastro-it-automation-by-ansible-legacy-role-vars-listup"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
   
     ita-by-ansible-towermaster-sync:
       replicaCount: 1
       extraEnv:
         EXECUTE_INTERVAL: "10"
       image:
         repository: "exastro/exastro-it-automation-by-ansible-towermaster-sync"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
   
     ita-by-conductor-synchronize:
       replicaCount: 1
       extraEnv:
         EXECUTE_INTERVAL: "10"
       image:
         repository: "exastro/exastro-it-automation-by-conductor-synchronize"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
   
     ita-by-menu-create:
       replicaCount: 1
       extraEnv:
         EXECUTE_INTERVAL: "10"
       image:
         repository: "exastro/exastro-it-automation-by-menu-create"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
   
     ita-database-setup-job:
       image:
         repository: ""
         tag: ""
         pullPolicy: IfNotPresent
   
     ita-web-server:
       replicaCount: 1
       image:
         repository: "exastro/exastro-it-automation-web-server"
         tag: "2.0.3-beta1"
         pullPolicy: IfNotPresent
   
   exastro-platform:
     platform-api:
       image:
         repository: "exastro/exastro-platform-api"
         tag: "1.3.0-beta1"
   
     platform-auth:
       ingress:
         enabled: true
         hosts:
           - host: exastro-suite.example.local
             paths:
               - path: /
                 pathType: Prefix
                 backend: "http"
           - host: exastro-suite-mng.example.local
             paths:
               - path: /
                 pathType: Prefix
                 backend: "httpMng"
       service:
         type: ClusterIP
         # http:
         #   nodePort: 30080
         # httpMng:
         #   nodePort: 30081
       image:
         repository: "exastro/exastro-platform-auth"
         tag: "1.3.0-beta1"
   
     platform-setup:
       keycloak:
         image:
           repository: "exastro/exastro-platform-job"
           tag: "1.3.0-beta1"
   
     platform-web:
       image:
         repository: "exastro/exastro-platform-web"
         tag: "1.3.0-beta1"
   
     mariadb:
       enabled: true
       image:
         repository: "mariadb"
         tag: "10.9"
         pullPolicy: IfNotPresent
       resources:
         requests:
           memory: "256Mi"
           cpu: "1m"
         limits:
           memory: "2Gi"
           cpu: "4"
   
     keycloak:
       enabled: true
       image:
         repository: "exastro/keycloak"
         tag: "1.3.0-beta1"
         pullPolicy: IfNotPresent
       resources:
         requests:
           memory: "256Mi"
           cpu: "1m"
         limits:
           memory: "2Gi"
           cpu: "4"

.. raw:: html

   </details>

| 以降の手順では、この :file:`exastro.yaml` に対してインストールに必要なパラメータを設定してきいます。

.. _service_setting:

3. サービス公開の設定
---------------------

| Exastro サービスを公開するための代表的な3つの設定方法について紹介します。

- Ingress
- LoadBalancer
- NodePort

.. note:: 
  | ここで紹介する方法以外にもサービス公開方法はあります。ユーザの環境ごとに適切な構成・設定を選択してください。

パラメータ
^^^^^^^^^^

| 利用可能なパラメータについては下記を参照してください。
       
.. include:: ../include/helm_option_platform-auth.rst

設定例
^^^^^^

| 各サービス公開方法の設定例を下記に記載します。

.. tabs::

   .. group-tab:: Ingress

      .. _ingress_setting:

      - 特徴

      | パブリッククラウドなどで Ingress Controller が利用可能な場合、Ingress を使ったサービス公開ができます。
      | クラスタ内にロードバランサーを構築して、ユーザ自身が運用したい場合などにメリットがあります。

      - 設定例

      | サービス公開用のドメイン情報を Ingress に登録することでDNSを使ったサービス公開を行います。
      | Azure におけるドメイン名の確認方法については :ref:`aks-dns` を確認してください。
      | クラウドプロバイダ毎に必要な :kbd:`annotations` を指定してください。
      | 下記は、AKS の Ingress Controller を使用する際の例を記載しています。

      .. code-block:: diff
         :caption: exastro.yaml
         :linenos:
         :lineno-start: 232
      
          platform-auth:
            ingress:
              enabled: true
         +    annotations:
         +      kubernetes.io/ingress.class: addon-http-application-routing
         +      nginx.ingress.kubernetes.io/proxy-body-size: 100m
         +      nginx.ingress.kubernetes.io/proxy-buffer-size: 256k
         +      nginx.ingress.kubernetes.io/server-snippet: |
         +        client_header_buffer_size 100k;
         +        large_client_header_buffers 4 100k;
              hosts:
         -      - host: exastro-suite.example.local
         +      - host: exastro-suite.xxxxxxxxxxxxxxxxxx.japaneast.aksapp.io ★ここにドメイン名を記載
                  paths:
                    - path: /
                      pathType: Prefix
                      backend: "http"
         -      - host: exastro-suite-mng.example.local
         +      - host: exastro-suite-mng.xxxxxxxxxxxxxxxxxx.japaneast.aksapp.io ★ここにドメイン名を記載
                  paths:
                    - path: /
                      pathType: Prefix
                      backend: "httpMng"

   .. group-tab:: LoadBalancer

      - 特徴

      | パブリッククラウドなどで LoadBalancer が利用可能な場合、LoadBalancer を使ったサービス公開ができます。
      | Ingress とは異なり、クラスタ外部(多くは、パブリッククラウドのサービス上)にロードバランサーがデプロイされるため、ユーザ自身が運用する必要がないことにメリットがあります。

      - 設定例

      | :kbd:`service.type` に :kbd:`LoadBalancer` を設定することで、LoadBalancer を使ったサービス公開ができます。
      | 下記は、LoadBalancer を使用する際の例を記載しています。
      
      .. code-block:: diff
         :caption: exastro.yaml
         :linenos:
         :lineno-start: 232
      
          platform-auth:
            ingress:
         -    enabled: true
         +    enabled: false
              hosts:
                - host: exastro-suite.example.local
                  paths:
                    - path: /
                      pathType: Prefix
                      backend: "http"
                - host: exastro-suite-mng.example.local
                  paths:
                    - path: /
                      pathType: Prefix
                      backend: "httpMng"
            service:
         -    type: ClusterIP
         +    type: LoadBalancer
              # http:
              #   nodePort: 30080
              # httpMng:
              #   nodePort: 30081

   .. group-tab:: NodePort

      - 特徴

      | ユーザ自身の環境でロードバランサーを準備する、もしくは、検証などの環境では NodePort を使ったサービス公開ができます。
      | Ingress や LoadBalancer とは異なり、ネイティブな Kubernetes で利用可能です。

      - 設定例

      | :kbd:`service.type` に :kbd:`NodePort` を設定することで、NodePort を使ったサービス公開ができます。
      | 下記は、NodePort を使用する際の例を記載しています。

      .. code-block:: diff
        :caption: exastro.yaml
        :linenos:
        :lineno-start: 232

          platform-auth:
            ingress:
        -    enabled: true
        +    enabled: false
              hosts:
                - host: exastro-suite.example.local
                  paths:
                    - path: /
                      pathType: Prefix
                      backend: "http"
                - host: exastro-suite-mng.example.local
                  paths:
                    - path: /
                      pathType: Prefix
                      backend: "httpMng"
            service:
        -    type: ClusterIP
        -    # http:
        -    #   nodePort: 30080
        -    # httpMng:
        -    #   nodePort: 30081
        +    type: NodePort
        +    http:
        +      nodePort: 30080
        +    httpMng:
        +      nodePort: 30081

.. _DATABASE_SETUP:

4. データベース連携
-------------------

| Exastro サービスを利用するためには、CMDB やオーガナイゼーションの管理のためのデータベースが必要となります。
| データベース利用時の3つの設定方法について説明します。

- 外部データベース
- データベースコンテナ

.. tabs::

   .. tab:: 外部データベース

      - 特徴

      | マネージドデータベースや別途用意した Kubernetes クラスタ外のデータベースを利用します。
      | Kubernetes クラスタ外にあるため、環境を分離して管理することが可能です。

      - 設定例

      | 外部データベースを操作するために必要な接続情報を設定します。

      .. warning::
        | :command:`DB_ADMIN_USER` で指定するDBの管理ユーザには、データベースとユーザを作成する権限が必要です。
      
      .. warning::
        | 認証情報などはすべて平文で問題ありません。(Base64エンコードは不要)
      
      1.  Exastro IT Automation 用データベースの設定

          | データベースの接続情報を設定します。

          .. include:: ../include/helm_option_itaDatabaseDefinition.rst

          .. code-block:: diff
            :caption: exastro.yaml
            :linenos:
            :lineno-start: 39

              itaDatabaseDefinition:
                name: ita-database
                enabled: true
                config:
            -     DB_VENDOR: "mariadb"
            -     DB_HOST: "mariadb"
            -     DB_PORT: "3306"
            +     DB_VENDOR: "mariadb"                # mariadb or mysql
            +     DB_HOST: "your.database.endpoint"   # データベースのエンドポイント
            +     DB_PORT: "3306"                     # データベース接続ポート
                  DB_DATABASE: "ITA_DB"               # 変更不要
                secret:
            -     DB_ADMIN_USER: ""
            -     DB_ADMIN_PASSWORD: ""s
            +     DB_ADMIN_USER: "your-admin-account"      # データベースの管理権限を持つユーザ
            +     DB_ADMIN_PASSWORD: "your-admin-password" # データベースの管理権限を持つユーザのパスワード
                  DB_USER: ""
                  DB_PASSWORD: ""

      2.  Exastro 共通基盤用データベースの設定

          | データベースの接続情報を設定します。

          .. include:: ../include/helm_option_pfDatabaseDefinition.rst

          .. code-block:: diff
            :caption: exastro.yaml
            :linenos:
            :lineno-start: 112

              pfDatabaseDefinition:
                name: auth-database
                enabled: true
                config:
            -     DB_VENDOR: "mariadb"
            -     DB_HOST: "mariadb"
            -     DB_PORT: "3306"
            +     DB_VENDOR: "mariadb"                # mariadb or mysql
            +     DB_HOST: "your.database.endpoint"   # データベースのエンドポイント
            +     DB_PORT: "3306"                     # データベース接続ポート
                  DB_DATABASE: "platform"             # 変更不要
                secret:
            -     DB_ADMIN_USER: ""
            -     DB_ADMIN_PASSWORD: ""
            +     DB_ADMIN_USER: "your-admin-account"      # データベースの管理者ユーザ
            +     DB_ADMIN_PASSWORD: "your-admin-password" # データベースの管理者ユーザのパスワード
                  DB_USER: ""
                  DB_PASSWORD: ""

      3.  データベースコンテナの無効化

          | データベースコンテナが起動しないように設定します。

          .. include:: ../include/helm_option_databaseDefinition.rst

          .. code-block:: diff
            :caption: exastro.yaml
            :linenos:
            :lineno-start: 267

              mariadb:
            -   enabled: true
            +   enabled: false

   .. tab:: データベースコンテナ

      - 特徴

      | Kubernetes クラスタ内にデプロイしたデータベースコンテナを利用します。
      | Exastro と同じ Kubernetes クラスタにコンテナとして管理できます。

      - 設定例

      | データベースコンテナの root パスワードを作成し、他のコンテナからもアクセスできるように作成した root アカウントのパスワードを設定します。
      | また、データベースのデータを永続化するために利用するストレージを指定します。

      .. warning::
        | :command:`DB_ADMIN_USER` で指定するDBの管理ユーザには、データベースとユーザを作成する権限が必要です。
      
      .. warning::
        | 認証情報などはすべて平文で問題ありません。(Base64エンコードは不要)

      .. _configuration_database_container:

      1.  データベースコンテナの設定

          | データベースコンテナの root パスワードを設定します。
          | また、データベースのデータを永続化するために利用するストレージを指定します。

          .. include:: ../include/helm_option_databaseDefinition.rst

          .. tabs::

            .. tab:: Storage Class 利用

                .. code-block:: diff
                  :caption: exastro.yaml
                  :linenos:
                  :lineno-start: 125
          
                      databaseDefinition:
                        name: mariadb
                        enabled: true
                        secret:
                  -     MARIADB_ROOT_PASSWORD: ""
                  +     MARIADB_ROOT_PASSWORD: "root-password"  # データベースコンテナの root のパスワード
                        persistence:
                          enabled: true
                          reinstall: false
                          accessMode: ReadWriteOnce
                  -      size: 20Gi
                  -      volumeType: hostPath # e.g.) hostPath or AKS
                  -      storageClass: "-" # e.g.) azurefile or - (None)
                  +      size: XXGi                                       # 必要な容量に変更
                  +      volumeType: AKS                                  # AKS を選択
                  +      storageClass: "exastro-suite-azurefile-csi-nfs"  # 利用する Storage Class を指定

            .. tab:: hostPath 利用

                .. code-block:: diff
                  :caption: exastro.yaml
                  :linenos:
                  :lineno-start: 125
          
                      databaseDefinition:
                        name: mariadb
                        enabled: true
                        secret:
                  -     MARIADB_ROOT_PASSWORD: ""
                  +     MARIADB_ROOT_PASSWORD: "root-password"  # コンテナデータベースの root のパスワード
                        persistence:
                          enabled: true
                          reinstall: false
                          accessMode: ReadWriteOnce
                  -      size: 20Gi
                  -      volumeType: hostPath # e.g.) hostPath or AKS
                  +      size: XXGi                                       # 必要な容量に変更
                  +      volumeType: hostPath                             # AKS を選択
                          storageClass: "-" # e.g.) azurefile or - (None)

      2.  Exastro IT Automation 用データベースの設定

          | Exastro IT Automation コンテナがデータベースに接続できるようにするために、:ref:`DATABASE_SETUP` で作成した root アカウントのパスワードを設定します。

          .. include:: ../include/helm_option_itaDatabaseDefinition.rst

          .. code-block:: diff
            :caption: exastro.yaml
            :linenos:
            :lineno-start: 39

              itaDatabaseDefinition:
                name: ita-database
                enabled: true
                config:
                  DB_VENDOR: "mariadb"
                  DB_HOST: "mariadb"
                  DB_PORT: "3306"
                  DB_DATABASE: "ITA_DB"
                secret:
            -     DB_ADMIN_USER: ""
            -     DB_ADMIN_PASSWORD: ""
            +     DB_ADMIN_USER: "root"                    # root を指定
            +     DB_ADMIN_PASSWORD: "your-admin-password" # 「1.  データベースコンテナの設定」で設定したコンテナデータベースの root のパスワード
                  DB_USER: ""
                  DB_PASSWORD: ""

      3.  Exastro 共通基盤用データベースの設定

          | Exastro 共通基盤のコンテナがデータベースに接続できるようにするために、「1.  データベースコンテナの設定」で作成した root アカウントのパスワードを設定します。

          .. include:: ../include/helm_option_pfDatabaseDefinition.rst

          .. code-block:: diff
            :caption: exastro.yaml
            :linenos:
            :lineno-start: 112

              pfDatabaseDefinition:
                name: auth-database
                enabled: true
                config:
                  DB_VENDOR: "mariadb"
                  DB_HOST: "mariadb"
                  DB_PORT: "3306"
                  DB_DATABASE: "platform"
                secret:
            -     DB_ADMIN_USER: ""
            -     DB_ADMIN_PASSWORD: ""
            +     DB_ADMIN_USER: "root"                    # root を指定
            +     DB_ADMIN_PASSWORD: "your-admin-password" # 「1.  データベースコンテナの設定」で設定したコンテナデータベースの root のパスワード
                  DB_USER: ""
                  DB_PASSWORD: ""

.. _installation_kubernetes_Keycloak 設定:

5. アプリケーションの DB ユーザ設定
-----------------------------------

| Exastro でアプリケーションのために作成する DB ユーザの設定をします。

設定例
^^^^^^

| 下記のアプリケーションが利用・作成する DB ユーザをそれぞれ設定します。

- Exastro IT Automation
- Exastro 共通基盤
- Keycloak

.. warning::
  | 認証情報などはすべて平文で問題ありません。(Base64エンコードは不要)

1.  Exastro IT Automation 用データベースの設定

    | アプリケーションが利用・作成する DB ユーザを設定します。

    .. include:: ../include/helm_option_itaDatabaseDefinition.rst

    .. code-block:: diff
      :caption: exastro.yaml
      :linenos:
      :lineno-start: 39

        itaDatabaseDefinition:
          name: ita-database
          enabled: true
          config:
            DB_VENDOR: "mariadb"
            DB_HOST: "mariadb"
            DB_PORT: "3306"
            DB_DATABASE: "ITA_DB"
          secret:
            DB_ADMIN_USER: ""
            DB_ADMIN_PASSWORD: ""
      -     DB_USER: ""
      -     DB_PASSWORD: ""
      +     DB_USER: "ita-db-user"                # Exastro IT Automation のアプリが使うDBユーザ
      +     DB_PASSWORD: "ita-db-user-password"   # Exastro IT Automation のアプリが使うDBユーザのパスワード


2.  Keycloak 用データベースの設定

    | アプリケーションが利用・作成する DB ユーザを設定します。

    .. include:: ../include/helm_option_keycloakDefinition.rst

    .. code-block:: diff
      :caption: exastro.yaml
      :linenos:
      :lineno-start: 82

        keycloakDefinition:
          name: keycloak
          enabled: true
          config:
            API_KEYCLOAK_PROTOCOL: "http"
            API_KEYCLOAK_HOST: "keycloak"
            API_KEYCLOAK_PORT: "8080"
            KEYCLOAK_PROTOCOL: "http"
            KEYCLOAK_HOST: "keycloak"
            KEYCLOAK_PORT: "8080"
            KEYCLOAK_MASTER_REALM: "master"
            KEYCLOAK_DB_DATABASE: "keycloak"
          secret:
            KEYCLOAK_USER: ""
            KEYCLOAK_PASSWORD: ""
      -     KEYCLOAK_DB_USER: ""
      -     KEYCLOAK_DB_PASSWORD: ""
      +     KEYCLOAK_DB_USER: "keycloak-db-user"               # Keycloak が使うDBユーザ
      +     KEYCLOAK_DB_PASSWORD: "keycloak-db-user-password"  # Keycloak が使うDBユーザのパスワード


3.  Exastro 共通基盤用データベースの設定

    | アプリケーションが利用・作成する DB ユーザを設定します。

    .. include:: ../include/helm_option_pfDatabaseDefinition.rst

    .. code-block:: diff
      :caption: exastro.yaml
      :linenos:
      :lineno-start: 112

        pfDatabaseDefinition:
          name: auth-database
          enabled: true
          config:
            DB_VENDOR: "mariadb"
            DB_HOST: "mariadb"
            DB_PORT: "3306"
            DB_DATABASE: "platform"
          secret:
            DB_ADMIN_USER: ""
            DB_ADMIN_PASSWORD: ""
      -     DB_USER: ""
      -     DB_PASSWORD: ""
      +     DB_USER: "pf-db-user"           # Exastro 共通基盤が使うDBユーザ
      +     DB_PASSWORD: "pf-db-password"   # Exastro 共通基盤が使うDBユーザのパスワード


.. _installation_kubernetes_gitlablinkage:

6. GitLab 連携設定
------------------

| GitLab 連携のための接続情報を登録します。

.. include:: ../include/helm_option_gitlabDefinition.rst

.. warning::
  | GITLAB_ROOT_TOKEN は下記の権限スコープが割り当てられたトークンが必要です。
  | ・api
  | ・write_repository
  | ・sudo

| 下記は、GitLab 連携の設定例を記載しています。

.. code-block:: diff
  :caption: exastro.yaml
  :linenos:
  :lineno-start: 30

      gitlabDefinition:
        name: gitlab
        enabled: true
        config:
    -     GITLAB_PROTOCOL: "http"
    -     GITLAB_HOST: "gitlab"
    -     GITLAB_PORT: "80"
    +     GITLAB_PROTOCOL: "接続プロトコル http or https"
    +     GITLAB_HOST: "接続先"
    +     GITLAB_PORT: "接続ポート"
        secret:
    -     GITLAB_ROOT_TOKEN: ""
    +     GITLAB_ROOT_TOKEN: "GitLabのRoot権限を持ったトークン"

.. _create_system_manager:

7. Exastro システム管理者の作成
-------------------------------

| Keycloak セットアップ時に Exastro システム管理者の初期ユーザを作成するための情報を設定します。

.. include:: ../include/helm_option_keycloakDefinition.rst

.. code-block:: diff
  :caption: exastro.yaml
  :linenos:
  :lineno-start: 82

    keycloakDefinition:
      name: keycloak
      enabled: true
      config:
        API_KEYCLOAK_PROTOCOL: "http"
        API_KEYCLOAK_HOST: "keycloak"
        API_KEYCLOAK_PORT: "8080"
        KEYCLOAK_PROTOCOL: "http"
        KEYCLOAK_HOST: "keycloak"
        KEYCLOAK_PORT: "8080"
        KEYCLOAK_MASTER_REALM: "master"
        KEYCLOAK_DB_DATABASE: "keycloak"
      secret:
  -     KEYCLOAK_USER: ""
  -     KEYCLOAK_PASSWORD: ""
  +     KEYCLOAK_USER: "admin"               # Exastro システムの管理者
  +     KEYCLOAK_PASSWORD: "admin-password"  # Exastro システムの管理者のパスワード
        KEYCLOAK_DB_USER: ""
        KEYCLOAK_DB_PASSWORD: ""

.. _persistent_volume:

8. 永続ボリュームの設定
-----------------------

| データベースのデータ永続化 (クラスタ内コンテナがある場合)、および、ファイルの永続化のために、永続ボリュームを設定する必要があります。
| 永続ボリュームの詳細については、 `永続ボリューム - Kubernetes <https://kubernetes.io/ja/docs/concepts/storage/persistent-volumes/#%E6%B0%B8%E7%B6%9A%E3%83%9C%E3%83%AA%E3%83%A5%E3%83%BC%E3%83%A0>`_ を参照してください。

| ストレージ利用時の2つの方法について説明します。

- マネージドディスク
- Kubernetes ノードのディレクトリ

.. tabs::

   .. tab:: マネージドディスク

      - 特徴
       
      | パブリッククラウドで提供されるストレージサービスを利用することでストレージの構築や維持管理が不要となります。

      - 設定例

      | Azure のストレージを利用する場合、下記のように StorageClass を定義することで利用が可能です。
      | 詳細は、 `Azure Kubernetes Service (AKS) でのアプリケーションのストレージ オプション <https://learn.microsoft.com/ja-jp/azure/aks/concepts-storage#storage-classes>`_ を参照してください。

      .. code-block:: diff
        :caption: storage-class-exastro-suite.yaml
        :linenos:

        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: exastro-suite-azurefile-csi-nfs
        provisioner: file.csi.azure.com
        allowVolumeExpansion: true
        parameters:
          protocol: nfs
        mountOptions:
          - nconnect=8

      .. code-block:: diff
        :caption: exastro.yaml
        :linenos:
        :lineno-start: 5

          itaGlobalDefinition:
            persistence:
              enabled: true
              accessMode: ReadWriteMany
              size: 10Gi
              volumeType: hostPath # e.g.) hostPath or AKS
        -      storageClass: "-" # e.g.) azurefile or - (None)
        +      storageClass: "azurefile" # e.g.) azurefile or - (None)

      | ※ 下記は、:ref:`DATABASE_SETUP` で設定済みです。

      .. code-block:: diff
        :caption: exastro.yaml
        :linenos:
        :lineno-start: 39

          databaseDefinition:
            persistence:
              enabled: true
              reinstall: false
              accessMode: ReadWriteOnce
              size: 20Gi
              volumeType: hostPath # e.g.) hostPath or AKS
        -      storageClass: "-" # e.g.) azurefile or - (None)
        +      storageClass: "exastro-suite-azurefile-csi-nfs" # e.g.) azurefile or - (None)

   .. tab:: Kubernetes ノードのディレクトリ

      - 特徴

      | Kubernetes のノード上のストレージ領域を利用するため、別途ストレージを調達する必要はありませんが、この方法は非推奨のため検証や開発時のみの利用

      .. danger::
          | データの永続化自体は可能ですが、コンピュートノードの増減や変更によりデータが消えてしまう可能性があるため本番環境では使用しないでください。
          | また、Azure で構築した AKS クラスタは、クラスタを停止すると AKS クラスターの Node が解放されるため、保存していた情報は消えてしまいます。そのため、Node が停止しないように注意が必要となります。

      - 利用例

      | hostPath を使用した例を記載します。

      .. code-block:: diff
        :caption: pv-database.yaml (データベース用ボリューム)
        :linenos:

        # pv-database.yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: pv-database
        spec:
          capacity:
            storage: 20Gi
          accessModes:
            - ReadWriteOnce
          persistentVolumeReclaimPolicy: Retain
          hostPath:
            path: /var/data/exastro-suite/exastro-platform/database
            type: DirectoryOrCreate

      .. code-block:: diff
        :caption: pv-ita-common.yaml (ファイル用ボリューム)
        :linenos:

        # pv-ita-common.yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: pv-ita-common
        spec:
          capacity:
            storage: 10Gi
          accessModes:
            - ReadWriteMany
          persistentVolumeReclaimPolicy: Retain
          hostPath:
            path: /var/data/exastro-suite/exastro-it-automation/ita-common
            type: DirectoryOrCreate

.. _インストール-1:

インストール
============

1. 永続ボリュームの作成
-----------------------

| :ref:`persistent_volume` で作成したマニフェストファイルを適用し、ボリュームを作成します。

.. code-block:: bash

    # pv-database.yaml
    kubectl apply -f pv-database.yaml

    # pv-ita-common.yaml
    kubectl apply -f pv-ita-common.yaml

.. code-block:: bash

    # 確認
    kubectl get pv

.. code-block:: bash

    NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
    pv-database     20Gi       RWO            Retain           Available                                   19s
    pv-ita-common   10Gi       RWX            Retain           Available                                   9s

2. インストール
---------------

| インストール時にサービスの公開方法によって、アクセス方法が異なります。
| Ingress, LoadBalancer, NodePort それぞれの方法について説明します。

.. tabs::

   .. group-tab:: Ingress

      1. Helm コマンドを使い Kubernetes 環境にインストールを行います。
      
         .. code-block:: bash
            :caption: コマンド

            helm install exastro exastro/exastro \
              --namespace exastro --create-namespace \
              --values exastro.yaml
      
         .. code-block:: bash
            :caption: 出力結果
      
            NAME: exastro
            LAST DEPLOYED: Sat Jan 28 15:00:02 2023
            NAMESPACE: exastro
            STATUS: deployed
            REVISION: 1
            TEST SUITE: None
            NOTES:
            Exastro install completion!

            1. Execute the following command and wait until the pod becomes "Running" or "Completed":

              # NOTE: You can also append "-w" to the command or wait until the state changes with "watch command"

              kubectl get pods --namespace exastro

            2. Get the ENCRYPT_KEY by running these commands:

              # Exastro IT Automation ENCRYPT_KEY
              kubectl get secret ita-secret-ita-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

              # Exastro Platform ENCRYPT_KEY
              kubectl get secret platform-secret-pf-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

              !!! Please save the output ENCRYPT_KEY carefully. !!!

            3. Run the following command to get the application URL and go to the URL or go to the displayed URL:
              *************************
              * Service Console       *
              *************************
              http://exastro-suite.example.local/
            
              *************************
              * Administrator Console *
              *************************
              http://exastro-suite-mng.example.local/
            
            
             # Note: You can display this note again by executing the following command.


         | 以下、上記の出力結果に従って操作をします。

      2. インストール状況確認

         | コマンドラインから以下のコマンドを入力して、インストールが完了していることを確認します。
        
         .. code-block:: bash
            :caption: コマンド

            # Pod の一覧を取得
            kubectl get po -n exastro
        
         | 正常動作している場合は、すべて “Running” もしくは “Completed” となります。
         | ※正常に起動するまで数分かかる場合があります。
        
         .. code-block:: bash
            :caption: 出力結果

            NAME                                                      READY   STATUS      RESTARTS   AGE
            ita-by-menu-create-7fccfc7f57-7cc2f                       1/1     Running     0          11m
            ita-by-conductor-synchronize-9dc6cfbdf-vp64z              1/1     Running     0          11m
            ita-api-admin-85b7d8f977-cxr58                            1/1     Running     0          11m
            ita-api-organization-5c5f4b86cb-rmf4g                     1/1     Running     0          11m
            ita-by-ansible-execute-6cd6d4d5fd-wkxjd                   1/1     Running     0          11m
            ita-by-ansible-legacy-role-vars-listup-67dbf5586f-dhdb2   1/1     Running     0          11m
            ita-by-ansible-towermaster-sync-5674448c55-t9592          1/1     Running     0          11m
            ita-web-server-7dbf6fd6ff-2s7s4                           1/1     Running     0          11m
            platform-auth-5b57bc57bd-h4k2g                            1/1     Running     0          11m
            platform-web-9f9d486fd-zf5vb                              1/1     Running     0          11m
            mariadb-67dd78cc76-nthtf                                  1/1     Running     0          11m
            platform-api-8655864fbf-t5xxf                             1/1     Running     0          11m
            ita-setup-wv2t6                                           0/1     Completed   0          11m
            keycloak-7f7cdccb6b-rf4rj                                 1/1     Running     0          11m
            platform-setup-8vv2x                                      0/1     Completed   0          11m

      3. 暗号化キーのバックアップ

         | Exastro システムのパスワードや認証情報といった機密情報はすべて暗号化されています。
         | 必ず、下記で取得した暗号化キーをバックアップして、適切に保管してください。

         .. danger::
           | 暗号化キーを紛失した場合、バックアップデータからシステムを復旧した際にデータの復号ができなくなります。

         .. code-block:: bash
            :caption: コマンド

            # Exastro IT Automation ENCRYPT_KEY
            kubectl get secret ita-secret-ita-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

         .. code-block:: bash
            :caption: 出力結果

            JnIoXzJtPic2MXFqRl1yI1chMj8hWzQrNypmVn41Pk8=

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform ENCRYPT_KEY
            kubectl get secret platform-secret-pf-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

         .. code-block:: bash
            :caption: 出力結果

            bHFZe2VEVVM2PmFeQDMqNG4oZT4lTlglLjJJekxBTHE=

      4. 接続確認

         | 出力結果に従って、:menuselection:`Administrator Console` の URL にアクセスします。
         | 下記は、実行例のため :ref:`service_setting` で設定したホスト名に読み替えてください。

         .. code-block:: bash
            :caption: 出力結果

            *************************
            * Service Console       *
            *************************
            http://exastro-suite.example.local/
            
            *************************
            * Administrator Console *
            *************************
            http://exastro-suite-mng.example.local/auth/

         .. list-table:: 接続確認用URL
            :widths: 20 40
            :header-rows: 0
            :align: left

            * - 管理コンソール
              - http://exastro-suite-mng.example.local/auth/

   .. group-tab:: LoadBalancer

      1. Helm コマンドを使い Kubernetes 環境にインストールを行います。
      
         .. code-block:: bash
            :caption: コマンド
        
            helm install exastro exastro/exastro \
              --namespace exastro --create-namespace \
              --values exastro.yaml
  
         .. code-block:: bash
            :caption: 出力結果
      
            NAME: exastro
            LAST DEPLOYED: Sat Jan 28 15:00:02 2023
            NAMESPACE: exastro
            STATUS: deployed
            REVISION: 1
            TEST SUITE: None
            NOTES:
            Exastro install completion!

            1. Execute the following command and wait until the pod becomes "Running" or "Completed":

              # NOTE: You can also append "-w" to the command or wait until the state changes with "watch command"

              kubectl get pods --namespace exastro

            2. Get the ENCRYPT_KEY by running these commands:

              # Exastro IT Automation ENCRYPT_KEY
              kubectl get secret ita-secret-ita-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

              # Exastro Platform ENCRYPT_KEY
              kubectl get secret platform-secret-pf-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

              !!! Please save the output ENCRYPT_KEY carefully. !!!

            3. Run the following command to get the application URL and go to the URL or go to the displayed URL:
              # NOTE: It may take a few minutes for the LoadBalancer IP to be available.
              #       You can watch the status of by running 'kubectl get --namespace exastro svc -w platform-auth'

              export NODE_SVC_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[0].nodePort}")
              export NODE_MGT_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[1].nodePort}")
              export NODE_IP=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.status.loadBalancer.ingress[0].ip}")
              echo "*************************"
              echo "* Administrator Console *"
              echo "*************************"
              echo http://$NODE_IP:$NODE_MGT_PORT/auth/
              echo ""
              echo "*************************"
              echo "* Service Console       *"
              echo "*************************"
              echo http://$NODE_IP:$NODE_SVC_PORT

         | 以下、上記の出力結果に従って操作をします。

      2. インストール状況確認

         | コマンドラインから以下のコマンドを入力して、インストールが完了していることを確認します。
        
         .. code-block:: bash
            :caption: コマンド

            # Pod の一覧を取得
            kubectl get po -n exastro
        
         | 正常動作している場合は、すべて “Running” もしくは “Completed” となります。
         | ※正常に起動するまで数分かかる場合があります。
        
         .. code-block:: bash
            :caption: 出力結果

            NAME                                                      READY   STATUS      RESTARTS   AGE
            ita-by-menu-create-7fccfc7f57-7cc2f                       1/1     Running     0          11m
            ita-by-conductor-synchronize-9dc6cfbdf-vp64z              1/1     Running     0          11m
            ita-api-admin-85b7d8f977-cxr58                            1/1     Running     0          11m
            ita-api-organization-5c5f4b86cb-rmf4g                     1/1     Running     0          11m
            ita-by-ansible-execute-6cd6d4d5fd-wkxjd                   1/1     Running     0          11m
            ita-by-ansible-legacy-role-vars-listup-67dbf5586f-dhdb2   1/1     Running     0          11m
            ita-by-ansible-towermaster-sync-5674448c55-t9592          1/1     Running     0          11m
            ita-web-server-7dbf6fd6ff-2s7s4                           1/1     Running     0          11m
            platform-auth-5b57bc57bd-h4k2g                            1/1     Running     0          11m
            platform-web-9f9d486fd-zf5vb                              1/1     Running     0          11m
            mariadb-67dd78cc76-nthtf                                  1/1     Running     0          11m
            platform-api-8655864fbf-t5xxf                             1/1     Running     0          11m
            ita-setup-wv2t6                                           0/1     Completed   0          11m
            keycloak-7f7cdccb6b-rf4rj                                 1/1     Running     0          11m
            platform-setup-8vv2x                                      0/1     Completed   0          11m

      3. 暗号化キーのバックアップ

         | Exastro システムのパスワードや認証情報といった機密情報はすべて暗号化されています。
         | 必ず、下記で取得した暗号化キーをバックアップして、適切に保管してください。

         .. danger::
            | 暗号化キーを紛失した場合、バックアップデータからシステムを復旧した際にデータの復号ができなくなります。

         .. code-block:: bash
            :caption: コマンド

            # Exastro IT Automation ENCRYPT_KEY
            kubectl get secret ita-secret-ita-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

         .. code-block:: bash
            :caption: 出力結果

            JnIoXzJtPic2MXFqRl1yI1chMj8hWzQrNypmVn41Pk8=

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform ENCRYPT_KEY
            kubectl get secret platform-secret-pf-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

         .. code-block:: bash
            :caption: 出力結果

            bHFZe2VEVVM2PmFeQDMqNG4oZT4lTlglLjJJekxBTHE=

      4. 接続確認

         | 1. で実行した :command:`helm install` の出力結果のコマンドをコンソール上に貼り付けて実行します。

         .. code-block:: bash
            :caption: コマンド

            # helm install コマンドの実行結果を貼り付けて実行
            export NODE_SVC_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[0].nodePort}")
            export NODE_MGT_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[1].nodePort}")
            export NODE_IP=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.status.loadBalancer.ingress[0].ip}")
            echo "*************************"
            echo "* Administrator Console *"
            echo "*************************"
            echo http://$NODE_IP:$NODE_MGT_PORT/auth/
            echo ""
            echo "*************************"
            echo "* Service Console       *"
            echo "*************************"
            echo http://$NODE_IP:$NODE_SVC_PORT

         | 出力結果に従って、:menuselection:`Administrator Console` の URL にアクセスします。
         | 下記は、実行例のため実際のコマンド実行結果に読み替えてください。

         .. code-block:: bash
            :caption: 出力結果

            *************************
            * Administrator Console *
            *************************
            http://172.16.20.XXX:32031/auth/

            *************************
            * Service Console       *
            *************************
            http://172.16.20.XXX:31798

         .. list-table:: 接続確認用URL
            :widths: 20 40
            :header-rows: 0
            :align: left

            * - 管理コンソール
              - http://172.16.20.xxx:32031/auth/

   .. group-tab:: NodePort

      1. Helm コマンドを使い Kubernetes 環境にインストールを行います。
      
         .. code-block:: bash
            :caption: コマンド
        
            helm install exastro exastro/exastro \
              --namespace exastro --create-namespace \
              --values exastro.yaml
  
         .. code-block:: bash
            :caption: 出力結果
      
            NAME: exastro
            LAST DEPLOYED: Sun Jan 29 12:18:02 2023
            NAMESPACE: exastro
            STATUS: deployed
            REVISION: 1
            TEST SUITE: None
            NOTES:
            Exastro install completion!

            1. Execute the following command and wait until the pod becomes "Running" or "Completed":

              # NOTE: You can also append "-w" to the command or wait until the state changes with "watch command"

              kubectl get pods --namespace exastro

            2. Get the ENCRYPT_KEY by running these commands:

              # Exastro IT Automation ENCRYPT_KEY
              kubectl get secret ita-secret-ita-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

              # Exastro Platform ENCRYPT_KEY
              kubectl get secret platform-secret-pf-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

              !!! Please save the output ENCRYPT_KEY carefully. !!!

            3. Run the following command to get the application URL and go to the URL or go to the displayed URL:
              export NODE_SVC_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[0].nodePort}")
              export NODE_MGT_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[1].nodePort}")
              export NODE_IP=$(kubectl get nodes --namespace exastro -o jsonpath="{.items[0].status.addresses[0].address}")
              echo "*************************"
              echo "* Administrator Console *"
              echo "*************************"
              echo http://$NODE_IP:$NODE_MGT_PORT/auth/
              echo ""
              echo "*************************"
              echo "* Service Console       *"
              echo "*************************"
              echo http://$NODE_IP:$NODE_SVC_PORT

            # Note: You can display this note again by executing the following command.

         | 以下、上記の出力結果に従って操作をします。

      2. インストール状況確認

         | コマンドラインから以下のコマンドを入力して、インストールが完了していることを確認します。
        
         .. code-block:: bash
            :caption: コマンド

            # Pod の一覧を取得
            kubectl get po -n exastro
        
         | 正常動作している場合は、すべて “Running” もしくは “Completed” となります。
         | ※正常に起動するまで数分かかる場合があります。
        
         .. code-block:: bash
            :caption: 出力結果

            NAME                                                      READY   STATUS      RESTARTS   AGE
            ita-by-menu-create-7fccfc7f57-7cc2f                       1/1     Running     0          11m
            ita-by-conductor-synchronize-9dc6cfbdf-vp64z              1/1     Running     0          11m
            ita-api-admin-85b7d8f977-cxr58                            1/1     Running     0          11m
            ita-api-organization-5c5f4b86cb-rmf4g                     1/1     Running     0          11m
            ita-by-ansible-execute-6cd6d4d5fd-wkxjd                   1/1     Running     0          11m
            ita-by-ansible-legacy-role-vars-listup-67dbf5586f-dhdb2   1/1     Running     0          11m
            ita-by-ansible-towermaster-sync-5674448c55-t9592          1/1     Running     0          11m
            ita-web-server-7dbf6fd6ff-2s7s4                           1/1     Running     0          11m
            platform-auth-5b57bc57bd-h4k2g                            1/1     Running     0          11m
            platform-web-9f9d486fd-zf5vb                              1/1     Running     0          11m
            mariadb-67dd78cc76-nthtf                                  1/1     Running     0          11m
            platform-api-8655864fbf-t5xxf                             1/1     Running     0          11m
            ita-setup-wv2t6                                           0/1     Completed   0          11m
            keycloak-7f7cdccb6b-rf4rj                                 1/1     Running     0          11m
            platform-setup-8vv2x                                      0/1     Completed   0          11m

      3. 暗号化キーのバックアップ

         | Exastro システムのパスワードや認証情報といった機密情報はすべて暗号化されています。
         | 必ず、下記で取得した暗号化キーをバックアップして、適切に保管してください。

         .. danger::
            | 暗号化キーを紛失した場合、バックアップデータからシステムを復旧した際にデータの復号ができなくなります。

         .. code-block:: bash
            :caption: コマンド

            # Exastro IT Automation ENCRYPT_KEY
            kubectl get secret ita-secret-ita-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

         .. code-block:: bash
            :caption: 出力結果

            JnIoXzJtPic2MXFqRl1yI1chMj8hWzQrNypmVn41Pk8=

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform ENCRYPT_KEY
            kubectl get secret platform-secret-pf-global -n exastro -o jsonpath='{.data.ENCRYPT_KEY}' | base64 -d

         .. code-block:: bash
            :caption: 出力結果

            bHFZe2VEVVM2PmFeQDMqNG4oZT4lTlglLjJJekxBTHE=

      4. 接続確認

         | 1. で実行した :command:`helm install` の出力結果のコマンドをコンソール上に貼り付けて実行します。

         .. code-block:: bash
            :caption: コマンド

            export NODE_SVC_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[0].nodePort}")
            export NODE_MGT_PORT=$(kubectl get services platform-auth --namespace exastro -o jsonpath="{.spec.ports[1].nodePort}")
            export NODE_IP=$(kubectl get nodes --namespace exastro -o jsonpath="{.items[0].status.addresses[0].address}")
            echo "*************************"
            echo "* Administrator Console *"
            echo "*************************"
            echo http://$NODE_IP:$NODE_MGT_PORT/auth/
            echo ""
            echo "*************************"
            echo "* Service Console       *"
            echo "*************************"
            echo http://$NODE_IP:$NODE_SVC_PORT

         | 出力結果に従って、:menuselection:`Administrator Console` の URL にアクセスします。
         | 下記は、実行例のため実際のコマンド実行結果に読み替えてください。

         .. code-block:: bash
            :caption: 出力結果

            *************************
            * Administrator Console *
            *************************
            http://172.16.20.xxx:30081/auth/

            *************************
            * Service Console       *
            *************************
            http://172.16.20.xxx:30080


         .. list-table:: 接続確認用URL
            :widths: 20 40
            :header-rows: 0
            :align: left

            * - 管理コンソール
              - http://172.16.20.xxx:30081/auth/

3. 管理コンソールへのログイン
-----------------------------

| 以下の画面が表示された場合、:menuselection:`Administration Console` を選択して、ログイン画面を開きます。

.. figure:: /images/ja/manuals/platform/keycloak/administrator-console.png
  :alt: administrator-console
  :width: 600px
  :name: 管理コンソール

| ログイン ID とパスワードは :ref:`create_system_manager` で登録した、:kbd:`KEYCLOAK_USER` 及び :kbd:`KEYCLOAK_PASSWORD` です。

.. figure:: /images/ja/manuals/platform/login/exastro-login.png
  :alt: login
  :width: 300px
  :name: ログイン画面

| Keycloak の管理画面が開きます。

.. figure:: /images/ja/manuals/platform/keycloak/keycloak-home.png
  :alt: login
  :width: 600px
  :name: Keycloak 管理画面

| ログインが確認できたら、:doc:`../manuals/platform_management/organization` の作成を行います。
