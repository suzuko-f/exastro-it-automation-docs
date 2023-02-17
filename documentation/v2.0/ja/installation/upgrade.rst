==============
アップグレード
==============

目的
====

| 本頁では、Exastro システムのアップグレード方法について紹介します。

前提条件
========

- Exastro システムが導入済みであること


アップグレードの準備
====================

.. warning:: 
  | アップグレード実施前に :doc:`../manuals/maintenance/backup_and_restore` の手順に従い、バックアップを取得しておくことを推奨します。

1. Helm リポジトリの更新
------------------------

| Exastro システムの Helm リポジトリを更新します。

| 更新前のバージョンを確認します。

.. code-block:: shell
   :linenos:
   :caption: コマンド

   # リポジトリ情報の確認
   helm search repo exastro

.. code-block:: shell
   :linenos:
   :caption: 実行結果
   :emphasize-lines: 3

   helm search repo exastro
   NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
   exastro/exastro                 1.0.0           2.0.3           A Helm chart for Exastro. Exastro is an Open So...
   exastro/exastro-it-automation   1.2.0           2.0.3           A Helm chart for Exastro IT Automation. Exastro...
   exastro/exastro-platform        1.5.0           1.4.0           A Helm chart for Exastro Platform. Exastro Plat...

| Helm リポジトリを更新します。

.. code-block:: shell
   :linenos:
   :caption: コマンド

   # リポジトリ情報の更新
   helm repo update

| 更新後のバージョンを確認します。

.. code-block:: shell
   :linenos:
   :caption: コマンド

   # リポジトリ情報の確認
   helm search repo exastro

.. code-block:: shell
   :linenos:
   :caption: 実行結果
   :emphasize-lines: 3

   helm search repo exastro
   NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
   exastro/exastro                 1.0.1           2.1.0           A Helm chart for Exastro. Exastro is an Open So...
   exastro/exastro-it-automation   1.2.0           2.0.3           A Helm chart for Exastro IT Automation. Exastro...
   exastro/exastro-platform        1.5.0           1.4.0           A Helm chart for Exastro Platform. Exastro Plat...


2. デフォルト設定値の更新の確認
-------------------------------

| デフォルト値の更新を確認します。
| インストール時に作成した設定ファイル :file:`exastro.yaml` とアップグレード後の設定ファイルを比較します。

.. code-block:: shell
   :caption: コマンド

   diff exastro.yaml <(helm show values exastro/exastro)


.. code-block:: diff
   :caption: 実行結果
    
   exastro-platform:
     platform-api:
       image:
         repository: "exastro/exastro-platform-api"
          tag: ""
    
     platform-auth:
   +    extraEnv:
   +      # Please set the URL to access
   +      EXTERNAL_URL: ""
   +      EXTERNAL_URL_MNG: ""
       ingress:
         enabled: true
         hosts:
           - host: exastro-suite.example.local
             paths:

3. 設定値の更新
---------------

| デフォルト設定値の比較結果から、項目の追加などにより設定値の追加が必要な場合は更新をしてください。
| 設定値の更新が不要であればこの手順はスキップしてください。
| 例えば下記の差分確認結果から、:kbd:`exastro-platform.platform-auth.extraEnv` が追加されていますので、必要に応じて、:file:`exastro.yaml` に項目と設定値を追加します。


.. code-block:: diff
   :caption: 実行結果
    
   exastro-platform:
     platform-api:
       image:
         repository: "exastro/exastro-platform-api"
          tag: ""
    
     platform-auth:
   +    extraEnv:
   +      # Please set the URL to access
   +      EXTERNAL_URL: ""
   +      EXTERNAL_URL_MNG: ""
       ingress:
         enabled: true
         hosts:
           - host: exastro-suite.example.local
             paths:

.. _アップグレード-1:

アップグレード
==============

サービス停止
------------

.. include:: ../include/stop_service_k8s.rst

アップグレード実施
------------------

| アップグレードを実施します。

.. code-block:: bash
  :caption: コマンド

  helm upgrade exastro exastro/exastro \
    --namespace exastro \
    --values exastro.yaml

.. code-block:: bash
  :caption: 出力結果

  NAME: exastro
  LAST DEPLOYED: Sat Jan 28 15:00:02 2023
  NAMESPACE: exastro
  STATUS: deployed
  REVISION: 2
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
    http://exastro-suite-mng.example.local/auth/


    # Note: You can display this note again by executing the following command.


サービス再開
------------

.. include:: ../include/start_service_k8s.rst

アップグレード状況確認
----------------------

| コマンドラインから以下のコマンドを入力して、アップグレードが完了していることを確認します。

.. code-block:: bash
   :caption: コマンド

   # Pod の一覧を取得
   kubectl get po -n exastro

| 正常動作している場合は、すべて “Running” もしくは “Completed” となります。
| ※正常に起動するまで数分かかる場合があります。

.. code-block:: bash
   :caption: 出力結果

   NAME                                                      READY   STATUS      RESTARTS   AGE
   mariadb-67dd78cc76-mzm25                                  1/1     Running     0          61m
   ita-setup-jl6pw                                           0/1     Completed   0          61m
   platform-migration                                        0/1     Completed   0          61m
   ita-web-server-6b7d5db55f-4n4sr                           1/1     Running     0          10m
   platform-web-6d48987995-28p6b                             1/1     Running     0          10m
   platform-auth-6ff45cb6b5-7fvmk                            1/1     Running     0          10m
   ita-by-conductor-synchronize-b6b59c745-x7fkx              1/1     Running     0          10m
   ita-api-admin-57bdc864b5-wtndv                            1/1     Running     0          10m
   ita-api-organization-7c8b957f64-8ffv9                     1/1     Running     0          10m
   ita-by-ansible-legacy-role-vars-listup-67fb9fd59c-6qmnq   1/1     Running     0          10m
   ita-by-ansible-execute-5656785f6b-m2lcv                   1/1     Running     0          10m
   ita-by-ansible-towermaster-sync-7fff49df5f-wvg84          1/1     Running     0          10m
   ita-by-menu-create-6b4fc66bdf-r6cg2                       1/1     Running     0          10m
   platform-api-5468bf6674-ncxtd                             1/1     Running     0          10m
   keycloak-7dd65794c7-w5wpg                                 1/1     Running     0          10m
