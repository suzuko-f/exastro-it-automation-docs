===================================
Organization (オーガナイゼーション)
===================================

はじめに
========

| 本章では、Exastro Suite における Organization (オーガナイゼーション) について説明します。


オーガナイゼーションとは
========================

| Exastro IT Automation 2.0 から導入されたマルチテナント機能におけるテナントの単位のことで、論理的に組織の空間を区分する単位のことで、たとえ同一の Exastro システムであってもデータの共有がされることのない、完全にプライベートな空間です。
| Exastro Suite の各アプリケーションの利用を開始するためには、オーガナイゼーションを作成する必要があります。


.. figure:: /images/ja/diagram/overview_of_the_organization.png
    :alt: オーガナイゼーション全体図


オーガナイゼーションの作成
==========================

| オーガナイゼーションの作成方法について説明します。

目的
----

| オーガナイゼーション作成を行うことで、オーガナイゼーション管理者のアカウントが作成され、オーガナイゼーションごとのエンドポイントURLにアクセスすることができるようになります。
| また、システム内部では下記の処理が実行されます。

- 処理のながれ

  #. Keycloak に、オーガナイゼーション用のレルムデータと管理者ユーザが登録されます。
  #. MariaDB や MySQL といったリレーショナルデータベースに、オーガナイゼーション用のデータが登録されます。
  #. Exastro IT Automation の永続ボリュームに、オーガナイゼーション用のディレクトリが作成されます。
  #. GitLab に、オーガナイゼーション用のユーザが登録されます。

前提条件
--------

| 本作業には下記のコマンドが必要となるため、事前にインストールをしてください。

- 前提条件

  - インストールが完了し、keycloak の管理コンソールにログインできること
  - システム管理に必要な下記の情報があること

    - 管理コンソールの URL
    - システム管理者のユーザID
    - システム管理者のパスワード

  - 作業クライアントに必要なアプリケーションがインストールされていること

    - :kbd:`curl`
    - :kbd:`git`
    - :kbd:`jq`
 
オーガナイゼーション作成
------------------------

| オーガナイゼーションの作成方法には、下記の3通りの方法があります。

.. tabs::

   .. tab:: 設定ファイルとスクリプトによる作成

      - 特徴

      | 対話型スクリプトによる作成方法と違い複数のオーガナイゼーション管理ユーザを登録できます。

      - 作成方法

      | GitHub リポジトリから取得した資材の中にある、シェルスクリプトを実行しオーガナイゼーションを作成します。

      #. オーガナイゼーション作成用シェルスクリプトを、リポジトリから :kbd:`git clone` により取得します。

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform の資材を入手
            git clone https://github.com/exastro-suite/exastro-platform.git

      #. 設定ファイルの :kbd:`CONF_BASE_URL` に Exastro Suite の管理用エンドポイント URL を設定します。

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform への接続のための設定情報を登録
            vi ./exastro-platform/tools/api-auth.conf

         | 例えば、:ref:`service_setting` で、Ingress を使ったサービス公開の設定をした場合は下記のようになります。

         .. code-block:: diff
            :caption: create-organization.conf
            :linenos:
            :lineno-start: 1

            - CONF_BASE_URL=http://platform-auth:8001
            + CONF_BASE_URL=http://exastro-suite-mng.example.local
              CURL_OPT=-sv
        
         .. tip::
             | 自己証明書を利用している場合、証明書エラーが発生します。
             | 設定ファイル内の :kbd:`CURL_OPT=-sv` を :kbd:`CURL_OPT=-svk` に変更することで証明書エラーを回避できますが、認証機関から発行された正しい証明書をインストールすることを推奨します。
            
      #. オーガナイゼーション情報の設定

         | オーガナイゼーション作成時の初期登録情報として下記の項目を設定できます。

         .. list-table:: オーガナイゼーション作成パラメータ
            :widths: 25 30 20 35
            :header-rows: 1
            :align: left
        
            * - 項目
              - 説明
              - 変更
              - デフォルト値・選択可能な設定値
            * - id
              - | オーガナイゼーションIDを指定。
                | 英小文字、数字、ハイフン、アンダースコアが利用可能。
                | 最大36文字。
                | ※先頭文字は英小文字であること。
                | ※予約語(後述)に合致しないこと。
              - 可
              - "org001"
            * - name
              - | オーガナイゼーション名を指定。
                | 最大255文字
              - 可
              - "org001-name"
            * - organization_managers
              - | オーガナイゼーション管理者の情報を指定。
                | ※複数名登録するときは繰り返し指定可能
              - 可
              - (オーガナイゼーション管理者のリスト)
            * - organization_managers[*].username
              - オーガナイゼーション管理者のユーザ名（ログインするときのID）を指定。
              - 可
              - "admin"
            * - organization_managers[*].email
              - オーガナイゼーション管理者のE-mailアドレスを指定。
              - 可
              - "admin@example.com"
            * - organization_managers[*].firstName
              - オーガナイゼーション管理者の名を指定。
              - 可
              - "admin"
            * - organization_managers[*].lastName
              - オーガナイゼーション管理者の姓を指定。
              - 可
              - "admin"
            * - organization_managers[*].credentials[0].type
              - 認証方式を指定。
              - 不可
              - "password"
            * - organization_managers[*].credentials[0].value
              - オーガナイゼーション管理者の初期パスワードを指定。
              - 可
              - "password"
            * - organization_managers[*].credentials[0].temporary
              - 初回ログイン時のパスワード変更の要否の有無を指定。
              - 可
              - | :program:`true` (デフォルト): パスワードの変更を要求する。 
                | :program:`false`: パスワードの変更を要求しない。
            * - plan.id
              - リソースプランを指定。
              - 可
              - ※初期状態では存在しないため指定しない。 
            * - options.sslRequired
              - SSL 接続の有無を指定。
              - 可
              - | :program:`external` (既定): プライベート IP アドレスに固定する限り、ユーザは SSL 無しで Keycloak と通信可能。
                | :program:`none`: SSL の設定なし。
                | :program:`all`: すべての IP アドレスに対し、SSL を要求。(内部の API が HTTP アクセスのため選択不可)


         | 設定ファイルの作成は、:file:`./exastro-platform/tools/create-organization.sample.json` を基に、作成するオーガナイゼーションの情報を指定した JSON ファイルを基に作成します。

         .. raw:: html

            <details>
              <summary>create-organization.sample.json</summary>

         .. code-block:: json
            :linenos:

            {
                "id"    :   "org001",
                "name"  :   "org001-name",
                "organization_managers" : [
                    {
                        "username"  :   "admin",
                        "email"     :   "admin@example.com",
                        "firstName" :   "admin",
                        "lastName"  :   "admin",
                        "credentials"   :   [
                            {
                                "type"      :   "password",
                                "value"     :   "password",
                                "temporary" :   true
                            }
                        ],
                        "requiredActions": [
                            "UPDATE_PROFILE"
                        ],
                        "enabled": true
                    }
                ],
                "plan": {
                    "id": "plan-1"
                },
                "options": {}
            }

         .. raw:: html

            </details>

         .. code-block:: bash
            :caption: コマンド

            # 設定用ファイルの作成
            cp -pi ./exastro-platform/tools/create-organization{.sample,}.json

            # 設定用ファイルの編集
            vi ./exastro-platform/tools/create-organization.json

        
         .. tip::
            | optionsの値に :program:`"sslRequired": "none"` を指定することで、オーガナイゼーションユーザが http でのアクセスが可能となります。

      #. オーガナイゼーション作成実行

         | スクリプトを実行してオーガナイゼーションを作成します。
         | :kbd:`your username` と :kbd:`your username` は :ref:`create_system_manager` で登録した、:kbd:`KEYCLOAK_USER` 及び :kbd:`KEYCLOAK_PASSWORD` です。

         .. code-block:: bash
            :caption: コマンド

             ./exastro-platform/tools/create-organization.sh ./exastro-platform/tools/create-organization.json

             your username : INPUT-YOUR-USERNAME # システム管理者のユーザ名を入力します
             your password : INPUT-USER-PASSWORD # システム管理者のパスワードを入力します

             Create an organization, are you sure? (Y/other) : Y # Y を入力するとオーガナイゼーションの作成処理が開始します

         | 成功時の結果表示は、:kbd:`result` が "000-00000”となります。
            
         .. code-block:: bash
            :caption: 実行結果 (成功時)

            ...
            < HTTP/1.1 200 OK
            < Date: Thu, 18 Aug 2022 01:49:13 GMT
            < Server: Apache/2.4.37 (Red Hat Enterprise Linux) mod_wsgi/4.7.1 Python/3.9
            < Content-Length: 107
            < Content-Type: application/json
            < 
            {
              "data": null, 
              "message": "SUCCESS", 
              "result": "000-00000", 
              "ts": "2022-08-18T01:49:17.251Z"
            }
            * Connection #0 to host platform-auth left intact

         | 失敗時の結果表示は、:kbd:`result` が "000-00000”以外となります。

         .. code-block:: bash
            :caption: 実行結果 (失敗時)

            ...
            < HTTP/1.1 400 BAD REQUEST
            < Date: Thu, 18 Aug 2022 05:29:35 GMT
            < Server: Apache/2.4.37 (Red Hat Enterprise Linux) mod_wsgi/4.7.1 Python/3.9
            < Content-Length: 252
            < Connection: close
            < Content-Type: application/json
            < 
            { [252 bytes data]
            * Closing connection 0
            {
              "data": null,
              "message": "指定されたorganization(org002)は作成済みのため、作成できません。",
              "result": "400-23001",
              "ts": "2022-08-18T05:29:35.643Z"
            }

   .. tab:: 対話型スクリプトによる作成

      - 特徴

      | 設定ファイルとスクリプトによる作成方法と違い設定ファイルの作成が不要です。

      .. tip::
        | この方法の場合、オーガナイゼーション管理者は1人のみ指定できます。
        | 複数名オーガナイゼーション管理者を作成する場合は、:menuselection:`設定ファイルとスクリプトによる作成方法` を行ってください。

      - 作成方法

      | 画面の指示に従ってオーガナイゼーション情報を指定し、オーガナイゼーションを作成します。

      | GitHub リポジトリから取得した資材の中にある、シェルスクリプトを実行しオーガナイゼーションを作成します。

      #. オーガナイゼーション作成用シェルスクリプトを、リポジトリから :kbd:`git clone` により取得します。

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform の資材を入手
            git clone https://github.com/exastro-suite/exastro-platform.git

      #. 設定ファイルの :kbd:`CONF_BASE_URL` に Exastro Suite の管理用エンドポイント URL を設定します。

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform への接続のための設定情報を登録
            vi ./exastro-platform/tools/api-auth.conf

         | 例えば、:ref:`service_setting` で、Ingress を使ったサービス公開の設定をした場合は下記のようになります。

         .. code-block:: diff
            :caption: create-organization.conf
            :linenos:
            :lineno-start: 1

            - CONF_BASE_URL=http://platform-auth:8001
            + CONF_BASE_URL=http://exastro-suite-mng.example.local
              CURL_OPT=-sv
        
         .. tip::
             | 自己証明書を利用している場合、証明書エラーが発生します。
             | 設定ファイル内の :kbd:`CURL_OPT=-sv` を :kbd:`CURL_OPT=-svk` に変更することで証明書エラーを回避できますが、認証機関から発行された正しい証明書をインストールすることを推奨します。

      #. オーガナイゼーション作成実行

         | オーガナイゼーション作成時の初期登録情報として下記の項目を設定できます。

         .. list-table:: オーガナイゼーション作成パラメータ
            :widths: 25 30 20 35
            :header-rows: 1
            :align: left
        
            * - 項目
              - 説明
              - 変更
              - デフォルト値・選択可能な設定値
            * - organization id
              - | オーガナイゼーションIDを指定。
                | 英小文字、数字、ハイフン、アンダースコアが利用可能。
                | 最大36文字。
                | ※先頭文字は英小文字であること。
                | ※予約語(後述)に合致しないこと。
              - 可
              - "org001"
            * - organization name
              - | オーガナイゼーション名を指定。
                | 最大255文字
              - 可
              - "org001-name"
            * - organization manager's username
              - オーガナイゼーション管理者のユーザ名（ログインするときのID）を指定。
              - 可
              - "admin"
            * - organization manager's email
              - オーガナイゼーション管理者のE-mailアドレスを指定。
              - 可
              - "admin@example.com"
            * - organization manager's firstName
              - オーガナイゼーション管理者の名を指定。
              - 可
              - "admin"
            * - organization manager's lastName
              - オーガナイゼーション管理者の姓を指定。
              - 可
              - "admin"
            * - organization manager's initial password
              - オーガナイゼーション管理者の初期パスワードを指定。
              - 可
              - "password"
            * - options.sslRequired
              - SSL 接続の有無を指定。
              - 可
              - | :program:`external` (既定): プライベート IP アドレスに固定する限り、ユーザは SSL 無しで Keycloak と通信可能。
                | :program:`none`: SSL の設定なし。
                | :program:`all`: すべての IP アドレスに対し、SSL を要求。(内部の API が HTTP アクセスのため選択不可)

         .. code-block:: sh
            :caption: コマンド 

            bash ./exastro-platform/tools/create-organization.sh

         | :kbd:`your username` と :kbd:`your username` は :ref:`create_system_manager` で登録した、:kbd:`KEYCLOAK_USER` 及び :kbd:`KEYCLOAK_PASSWORD` です。

         .. code-block::
            :caption: コマンド (入力例)

            Please enter the organization information to be created
        
            organization id : org001                             # オーガナイゼーションIDを入力します
            organization name : org001-name                      # オーガナイゼーション名を入力します
            organization manager's username : admin              # オーガナイゼーション管理者のユーザ名（ログインするときのID）を入力します
            organization manager's email : admin@example.com     # オーガナイゼーション管理者のE-mailアドレスを入力します
            organization manager's first name : admin            # オーガナイゼーション管理者の名を入力します
            organization manager's last name : admin             # オーガナイゼーション管理者の姓を入力します
            organization manager's initial password : password   # オーガナイゼーション管理者の初期パスワードを入力します
            organization plan id (optional) :                    # プランを指定(任意)します ※ 初期状態では未作成のため入力不要

            your username : INPUT-YOUR-USERNAME                  # システム管理者のユーザ名を入力します
            your password : INPUT-USER-PASSWORD                  # システム管理者のパスワードを入力します
      
            Create an organization, are you sure? (Y/other) : Y # "Y"を入力すると実行します


         | 成功時の結果表示は、:kbd:`result` が "000-00000”となります。
            
         .. code-block:: bash
            :caption: 実行結果 (成功時)

            ...
            < HTTP/1.1 200 OK
            < Date: Thu, 18 Aug 2022 01:49:13 GMT
            < Server: Apache/2.4.37 (Red Hat Enterprise Linux) mod_wsgi/4.7.1 Python/3.9
            < Content-Length: 107
            < Content-Type: application/json
            < 
            {
              "data": null, 
              "message": "SUCCESS", 
              "result": "000-00000", 
              "ts": "2022-08-18T01:49:17.251Z"
            }
            * Connection #0 to host platform-auth left intact

         | 失敗時の結果表示は、:kbd:`result` が "000-00000”以外となります。

         .. code-block:: bash
            :caption: 実行結果 (失敗時)

            ...
            < HTTP/1.1 400 BAD REQUEST
            < Date: Thu, 18 Aug 2022 05:29:35 GMT
            < Server: Apache/2.4.37 (Red Hat Enterprise Linux) mod_wsgi/4.7.1 Python/3.9
            < Content-Length: 252
            < Connection: close
            < Content-Type: application/json
            < 
            { [252 bytes data]
            * Closing connection 0
            {
              "data": null,
              "message": "指定されたorganization(org002)は作成済みのため、作成できません。",
              "result": "400-23001",
              "ts": "2022-08-18T05:29:35.643Z"
            }

   .. tab:: Rest API による作成

      - 特徴

      | 外部システムからオーガナイゼーションの作成を行う場合は、Rest API を使います。

      - 作成方法

      #. オーガナイゼーション作成実行

         | Rest API を使ってオーガナイゼーションを作成します。
         | 利用可能なパラメータは下記のとおりです。
         | 詳細は、:doc:`../../reference/api/system_manager/platform-api` を参照してください。

         .. list-table:: オーガナイゼーション作成パラメータ
            :widths: 25 30 20 35
            :header-rows: 1
            :align: left
        
            * - 項目
              - 説明
              - 変更
              - デフォルト値・選択可能な設定値
            * - organization id
              - | オーガナイゼーションIDを指定。
                | 英小文字、数字、ハイフン、アンダースコアが利用可能。
                | 最大36文字。
                | ※先頭文字は英小文字であること。
                | ※予約語(後述)に合致しないこと。
              - 可
              - "org001"
            * - organization name
              - | オーガナイゼーション名を指定。
                | 最大255文字
              - 可
              - "org001-name"
            * - organization manager's username
              - オーガナイゼーション管理者のユーザ名（ログインするときのID）を指定。
              - 可
              - "admin"
            * - organization manager's email
              - オーガナイゼーション管理者のE-mailアドレスを指定。
              - 可
              - "admin@example.com"
            * - organization manager's firstName
              - オーガナイゼーション管理者の名を指定。
              - 可
              - "admin"
            * - organization manager's lastName
              - オーガナイゼーション管理者の姓を指定。
              - 可
              - "admin"
            * - organization manager's initial password
              - オーガナイゼーション管理者の初期パスワードを指定。
              - 可
              - "password"
            * - options.sslRequired
              - SSL 接続の有無を指定。
              - 可
              - | :program:`external` (既定): プライベート IP アドレスに固定する限り、ユーザは SSL 無しで Keycloak と通信可能。
                | :program:`none`: SSL の設定なし。
                | :program:`all`: すべての IP アドレスに対し、SSL を要求。(内部の API が HTTP アクセスのため選択不可)


      | cURL を使って Rest API を利用する場合は、以下の様なコマンドを実行してください。
      | BASIC 認証で使用する認証情報は :ref:`create_system_manager` で登録した、:kbd:`KEYCLOAK_USER` 及び :kbd:`KEYCLOAK_PASSWORD` です。

      .. warning::
         | BASIC 認証を行うために、Exastro Platform 管理者の認証情報を :kbd:`BASE64_BASIC` に設定する必要があります。

      | また、Exastro Platform の管理用 URL 情報を :kbd:`BASE_URL` に設定する必要があります。
      | 例えば、 :ref:`サービス公開の設定 (Ingress の設定) <ingress_setting>` をした場合は下記のようになります。

      .. code-block:: bash

        BASE64_BASIC=$(echo -n "KEYCLOAK_USER:KEYCLOAK_PASSWORD" | base64)
        BASE_URL=http://exastro-suite-mng.example.local

        curl -X 'POST' \
          "${BASE_URL}/api/platform/organizations" \
          -H 'accept: application/json' \
          -H "Authorization: Basic ${BASE64_BASIC}" \
          -H 'Content-Type: application/json' \
          -d '{
          "id": "org001",
          "name": "org001-name",
          "organization_managers": [
            {
              "username": "admin",
              "email": "admin@example.com",
              "firstName": "admin",
              "lastName": "admin",
              "credentials": [
                {
                  "type": "password",
                  "value": "password",
                  "temporary": true
                }
              ],
              "requiredActions": [
                "UPDATE_PROFILE"
              ],
              "enabled": true
            }
          ],
          "plan": {},
          "options": {}
        }'


オーガナイゼーションへのアクセス
================================

| オーガナイゼーション用サイトが表示できるかWebブラウザから確認します。

.. code-block::

   # 書式
   http[s]://{Exastro Platform の管理用 URL}/{オーガナイゼーションID}/platform/

   # 具体例
   http://exastro-suite-mng.example.local/org001/platform/


その他制約事項・備考
====================

オーガナイゼーションIDの予約語
------------------------------

| 以下に示すパターンに合致するワードは、オーガナイゼーションの ID として使用できません。
  
- master
- platform
- account
- account-console
- admin-cli
- broker
- realm-management
- security-admin-console
- \*-workspaces
- system-\*-auth


オーガナイゼーション作成を再実行する場合
----------------------------------------

| オーガナイゼーション作成で失敗した場合、オーガナイゼーション作成の再実行をしても「指定されたorganization(xxx)は作成済みのため、作成できません。」というエラーが表示されることがあります。
| このように、失敗したオーガナイゼーション ID でオーガナイゼーションの作成ができない場合は、コマンドパラメータに :kbd:`--retry` オプションを付与して実行することで再作成をすることが可能です。

.. code-block:: bash

   ./exastro-platform/tools/create-organization.sh --retry

.. code-block:: bash

   ./exastro-platform/tools/create-organization.sh ./exastro-platform/tools/create-organization.sample.json
