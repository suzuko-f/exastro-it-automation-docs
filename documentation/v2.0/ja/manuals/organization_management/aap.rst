================================
Ansible Automation Platform 連携
================================

目的
====

| Ansible Automation Platform とは、PF 構築自動化ツールである Ansible にアクセスコントロール、ジョブスケジューリング、タスクの可視化などの機能を拡張した管理プラットフォームです。
| 本頁では、 Exastro IT Automation と Ansible Automation Controller との連携方法について説明します。
| Ansible Automation Platform を使用しない場合には本手順は不要です。

前提条件
========

| 連携する Ansible Automation Controller にてユーザの作成がされている必要があります。


Ansible Automation Controller の登録
====================================

| Ansible Automation Controller の登録をすることで、“プロジェクト”、”インベントリ”、”認証情報”の組合せで”ジョブテンプレート”を作成しAnsibleを実行できます。
| 複数の“ジョブテンプレート”を組み合せて”ワークフロージョブテンプレート”を作成することによって、より多彩な作業フローを表現することができます。

前提条件
--------

| 本作業には下記のコマンドが必要となるため、事前にインストールをしてください。

- 作業クライアントに必要なアプリケーション

  - :kbd:`curl`
  - :kbd:`git`
  - :kbd:`jq`
 
登録方法
--------

| Ansible Automation Controller の Exastro IT Automation への登録には、下記の2通りの方法があります。

.. tabs::

   .. group-tab:: 設定ファイルとスクリプト利用

      - 特徴
       
      | Rest API を使った登録方法に比べ、利用するパラメータ情報の事前準備が不要なためユーザの操作に向いています。

      - 登録方法

      | GitHub リポジトリから取得した資材の中にある、シェルスクリプトを実行し Ansible Automation Controller を登録します。

      #. Ansible Automation Controller 登録用シェルスクリプトを、リポジトリから :kbd:`git clone` により取得します。

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform の資材を入手
            git clone https://github.com/exastro-suite/exastro-platform.git

      #. 設定ファイルの :kbd:`CONF_BASE_URL` に Exastro システムの管理用エンドポイント URL を設定します。

         .. code-block:: bash
            :caption: コマンド

            # Exastro Platform への接続のための設定情報を登録
            vi ./exastro-platform/tools/api-auth.conf

         | 例えば、:ref:`サービス公開の設定 (Ingress の設定)<ingress_setting>` をした場合は下記のようになります。

         .. code-block:: diff
            :caption: api-auth.conf
     
            - CONF_BASE_URL=http://platform-auth:8001
            + CONF_BASE_URL=http://exastro-suite-mng.example.local
              CURL_OPT=-sv
        
         .. tip::
            | 自己証明書を利用している場合、証明書エラーが発生します。
            | 設定ファイル内の :kbd:`CURL_OPT=-sv` を :kbd:`CURL_OPT=-svk` に変更することで証明書エラーを回避できますが、認証機関から発行された正しい証明書をインストールすることを推奨します。
            
      #. Ansible Automation Controller 情報の設定

         | 設定ファイルの作成は、:file:`./exastro-platform/tools/initial-settings-ansible.sample.json` を基に、作成する Ansible Automation Controller の情報を指定した JSON ファイルを基に作成します。

         .. code-block:: bash
            :caption: コマンド

            cp -pi ./exastro-platform/tools/initial-settings-ansible{.sample,}.json

            vi ./exastro-platform/tools/initial-settings-ansible.json


         .. raw:: html

            <details> 
              <summary>initial-settings-ansible.sample.json</summary>

         .. code-block:: json

            {
                "input_limit_setting": true,
                "execution_engine_list": [
                    "string"
                ],
                "initial_data": {
                    "ansible_automation_controller_host_list": [
                        {
                            "file": {
                                "ssh_private_key_file": "string"
                            },
                            "parameter": {
                                "host": "string",
                                "authentication_method": "string",
                                "user": "string",
                                "password": "string",
                                "ssh_private_key_file": "string",
                                "passphrase": "string",
                                "isolated_tower": "string",
                                "remarks": "string"
                            }
                        }
                    ],
                    "interface_info_ansible": {
                        "parameter": {
                            "execution_engine": "string",
                            "representative_server": "string",
                            "ansible_automation_controller_protocol": "string",
                            "ansible_automation_controller_port": "string",
                            "organization_name": "string",
                            "authentication_token": "string",
                            "delete_runtime_data": "string",
                            "proxy_address": "string",
                            "proxy_port": "string"
                        }
                    }
                }
            }


         .. raw:: html

            </details>

         | 各パラメータについては下記を参照してください。

         .. raw:: html

           <details> 
             <summary>Ansible Automation Controller 登録時のパラメータ(表示・非表示)</summary>

         .. include:: ../../include/api_option_initial_settings_ansible.rst

         .. tip::
             | ※ 認証方式が鍵認証（鍵交換済み）に設定する為に必要な公開鍵ファイルの配布
             | ・Ansible Core の場合
             | ansible がインストールされているサーバーの実行ユーザー「Ansible 共通コンソール=>インターフェース情報に設定されている実行ユーザー」から作業対象ホストに ssh 接続します。
             | 実行ユーザーの公開鍵ファイルをログイン先ユーザーの authorized_keys にコピーして下さい。
             |
             | ・ Ansible Automation Controller の場合
             | Ansible Automation Controller の awx ユーザーから作業対象ホストに ssh 接続しています。
             | awx ユーザーの公開鍵ファイルをログイン先ユーザーの authorized_keys にコピーして下さい。ブラウザより Ansible Automation Controller にログインし、「設定」→「ジョブ」→「分離されたジョブに公開するパス」に「/var/lib/awx/.ssh/」を設定します。

         .. raw:: html

           </details> 

        
      #. Ansible Automation Controller 作成実行

         .. code-block:: bash
            :caption: コマンド

            ./exastro-platform/tools/initial-settings-ansible.sh ./exastro-platform/tools/initial-settings-ansible.json

            organization id : INPUT-ORGANIZATION-ID-TO-SET # 設定先のオーガナイゼーションID

            your username : INPUT-YOUR-USERNAME # システム管理者のユーザ名を入力します
            your password : INPUT-USER-PASSWORD # システム管理者のパスワードを入力します

            Create an organization, are you sure? (Y/other) : Y # Y を入力すると Ansible Automation Controller の登録処理が開始します


         -  成功時の結果表示

            resultが”000-00000”が、 Ansible Automation Controller の作成に成功したことを示しています。
            
            .. code-block:: bash
               :caption: 実行結果(成功時)

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


         -  失敗時の結果表示イメージ

            .. code-block:: bash
               :caption: 実行結果(失敗時)
 
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
                 "message": "エラーメッセージ,
                 "result": "エラーコード",
                 "ts": "2022-08-18T05:29:35.643Z"
               }

   .. group-tab:: Rest API 利用

      - 特徴

      | Rest API は外部システムから Ansible Automation Controller の登録を行う際に有用です。

      - 登録方法

      1. SSH 鍵ファイルのエンコード

         | SSH 秘密鍵ファイルをアップロードする必要があるため、Base64 エンコードをします。

         .. code-block::
           :caption: コマンド
   
           MY_KEY=`base64 -w 0 my-aac-key.pem`

      2. コマンド

         | オーガナイゼーションIDを :kbd:`ORG_ID` に設定する必要があります。
         | また、Basic 認証を利用するためには、システム管理者の認証情報を :kbd:`BASE64_BASIC` に設定する必要があります。
         | 認証情報に関して、:ref:`インストール時に登録した認証情報 <create_system_manager>` で登録した内容となります。
         | cURL を使用する場合は、下記のようにコマンドを実行します。


         | 各パラメータについては下記を参照してください。

         .. raw:: html
          
            <details> 
              <summary>Ansible Automation Controller 登録時のパラメータ</summary>

         .. include:: ../../include/api_option_initial_settings_ansible.rst
          
         .. raw:: html
          
            </details> 

         .. code-block:: bash
            :caption: コマンド

            ORG_ID=org001
            BASE64_BASIC=$(echo -n "KEYCLOAK_USER:KEYCLOAK_PASSWORD" | base64)
            BASE_URL=http://exastro-suite-mng.example.local

            curl -X 'POST' \
              "http://${BASE_URL}/api/ita/${ORG_ID}/initial-settings/ansible/" \
              -H 'accept: application/json' \
              -H "Authorization: Basic ${BASE64_BASIC}" \
              -H 'Content-Type: application/json' \
              -d '{
              "input_limit_setting": true,
              "execution_engine_list": [
                "Ansible Automation Controller"
              ],
              "initial_data": {
                "ansible_automation_controller_host_list": [
                  {
                    "file": {
                      "ssh_private_key_file": "'${MY_KEY}'"
                    },
                    "parameter": {
                      "host": "aac-server01",
                      "user": "awx",
                      "authentication_method": "鍵認証（パスフレーズなし）",
                      "password": "awx-password",
                      "ssh_private_key_file": "my-aac-key.pem",
                      "passphrase": "",
                      "isolated_tower": "False",
                      "remarks": ""
                    }
                  }
                ],
                "interface_info_ansible": {
                  "parameter": {
                    "execution_engine": "Ansible Automation Controller",
                    "representative_server": "aac-server01",
                    "ansible_automation_controller_protocol": "https",
                    "ansible_automation_controller_port": "443",
                    "organization_name": "organization001",
                    "authentication_token": "LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpR...",
                    "delete_runtime_data": "True",
                    "proxy_address": "",
                    "proxy_port": ""
                  }
                }
              }
            }'

Ansible Automation Contoller 連携の確認
---------------------------------------

#. オーガナイゼーション作成結果を確認します。

.. tabs::

   .. group-tab:: 設定ファイルとスクリプト利用

      .. code-block:: bash
         :caption: コマンド
    
         ./exastro-platform/tools/get-initial-settings-ansible.sh
          
         organization id : INPUT-ORGANIZATION-ID-TO-SET # 設定先のオーガナイゼーションID
    
         your username : INPUT-YOUR-USERNAME # システム管理者のユーザ名を入力します
         your password : INPUT-USER-PASSWORD # システム管理者のパスワードを入力します

      -  結果表示

         resultが”000-00000”が、 Ansible Automation Controller の作成に成功したことを示しています。
          
         .. code-block:: bash
            :caption: 実行結果(成功例)

            {
              "result": "000-00000",
              "data": {
                "input_limit_setting": true,
                "execution_engine_list": [
                  "string"
                ],
                "initial_data": {
                  "ansible_automation_controller_host_list": [
                    {
                      "file": {
                        "ssh_private_key_file": "string"
                      },
                      "parameter": {
                        "host": "string",
                        "authentication_method": "string",
                        "user": "string",
                        "password": "string",
                        "ssh_private_key_file": "string",
                        "passphrase": "string",
                        "isolated_tower": "string",
                        "remarks": "string"
                      }
                    }
                  ],
                  "interface_info_ansible": {
                    "parameter": {
                      "execution_engine": "string",
                      "representative_server": "string",
                      "ansible_automation_controller_protocol": "string",
                      "ansible_automation_controller_port": "string",
                      "organization_name": "string",
                      "authentication_token": "string",
                      "delete_runtime_data": "string",
                      "proxy_address": "string",
                      "proxy_port": "string"
                    }
                  }
                }
              },
              "message": "string"
            }        


      -  失敗時の結果表示イメージ

         .. code-block:: bash
            :caption: 実行結果(失敗例)

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
              "message": "エラーメッセージ,
              "result": "エラーコード",
              "ts": "2022-08-18T05:29:35.643Z"
            }

   .. group-tab:: Rest API 利用

      .. code-block:: bash
         :caption: コマンド
    
         curl -X 'GET' \
           'http://exastro-suite-mng.example.local/api/ita/org001/initial-settings/ansible/' \
            -H 'accept: application/json'

      -  結果表示

         resultが”000-00000”が、 Ansible Automation Controller の作成に成功したことを示しています。
          
         .. code-block:: bash
            :caption: 実行結果(成功例)

            {
              "result": "000-00000",
              "data": {
                "input_limit_setting": true,
                "execution_engine_list": [
                  "string"
                ],
                "initial_data": {
                  "ansible_automation_controller_host_list": [
                    {
                      "file": {
                        "ssh_private_key_file": "string"
                      },
                      "parameter": {
                        "host": "string",
                        "authentication_method": "string",
                        "user": "string",
                        "password": "string",
                        "ssh_private_key_file": "string",
                        "passphrase": "string",
                        "isolated_tower": "string",
                        "remarks": "string"
                      }
                    }
                  ],
                  "interface_info_ansible": {
                    "parameter": {
                      "execution_engine": "string",
                      "representative_server": "string",
                      "ansible_automation_controller_protocol": "string",
                      "ansible_automation_controller_port": "string",
                      "organization_name": "string",
                      "authentication_token": "string",
                      "delete_runtime_data": "string",
                      "proxy_address": "string",
                      "proxy_port": "string"
                    }
                  }
                }
              },
              "message": "string"
            }        


      -  失敗時の結果表示イメージ

         .. code-block:: bash
            :caption: 実行結果(失敗例)

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
              "message": "エラーメッセージ,
              "result": "エラーコード",
              "ts": "2022-08-18T05:29:35.643Z"
            }


その他制約事項・備考
--------------------

Ansible Automation Controller 登録を再実行する場合
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Ansible Automation Controller 登録の再実行する場合は、設定ファイルを編集後、再度スクリプトを実行して下さい。

.. code-block:: bash
   :caption: コマンド

   vi ./exastro-platform/tools/initial-settings-ansible.json

.. code-block:: bash
   :caption: コマンド

   ./exastro-platform/tools/initial-settings-ansible.sh ./exastro-platform/tools/initial-settings-ansible.sample.json

