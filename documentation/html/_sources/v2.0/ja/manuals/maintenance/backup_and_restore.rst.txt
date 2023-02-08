================================
データバックアップ・リストア手順
================================

目的
====

| 本頁では、Exastroシステムで利用する永続データのバックアップとリストア手順について説明します。
| パブリッククラウドの仕組みにより、データバックアップの仕組みが利用可能であればそちらを利用しても問題ありません。


前提条件
========

| 本頁で紹介するバックアップ・リストア手順は下記の条件を満たしている必要があります。

- 条件

  - 下記のコマンドが利用可能なこと

    - :command:`tar`
    - :command:`kubectl`

  - 作業環境のサーバで充分なディスクの空き容量があること


概要
====

| バックアップ対象となるデータは下記の3つです。

- バックアップ対象

  - Exastro Platform のデータベース
  - Exastro IT Automation のデータベース
  - Exastro IT Automation の共有ファイル

| これらのデータを :command:`kubectl` コマンドを利用してインストールします。

| 作業の流れは、まず、ユーザからのデータの書き込みを制限するために、リバースプロキシを停止します。
| 次に、バックヤード処理を停止したら、データのバックアップを実施します。
| 最後に、作業前の数に Pod 数を戻します。

.. danger::
  | 本手順では、サービスの停止が発生します。

バックアップ
============

#. レプリカ数の確認

   | 作業前のレプリカ数の確認をし、状態を記録します。

   .. code-block:: bash
      :caption: コマンド

      kubectl get deploy -o jsonpath='{range .items[*]}{@.metadata.name}{"\t"}{@.spec.replicas}{"\n"}' -n exastro > before_replica_count_`date +"%Y%m%d-%H%M%S"`.tsv


#. リバースプロキシの停止

   | リバースプロキシ (platform-auth) のレプリカ数を 0 に変更し、エンドユーザーからのアクセスを制限します。

   .. code-block:: bash
      :caption: コマンド

      kubectl scale deployment platform-auth -n exastro --replicas=0


#. バックヤード処理の停止

   | バックヤード処理 (ita-by-\*\*\*) のレプリカ数を 0 に変更し、データベースの更新を停止します。

   .. code-block:: bash
      :caption: コマンド

      kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=0
      kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=0
      kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=0
      kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=0
      kubectl scale deployment ita-by-menu-create -n exastro --replicas=0


#. データベースのバックアップ

   | データベースを :command:`mysqldump` を使ってバックアップします。

   | メンテナンス用コンテナを作成します。

   .. code-block:: bash
      :caption: コマンド
      :linenos:

      cat <<_EOF_ | kubectl apply -f - -n exastro
      apiVersion: v1
      kind: Pod
      metadata:
        name: platform-db-backup
        namespace: exastro
      spec:
        containers:
        - command:
          - sh
          - -c
          args:
          - |
             #!/bin/bash
             sleep 3600
          env:
          - name: DB_DATABASE
            valueFrom:
              configMapKeyRef:
                key: DB_DATABASE
                name: platform-params-pf-database
          - name: DB_HOST
            valueFrom:
              configMapKeyRef:
                key: DB_HOST
                name: platform-params-pf-database
          - name: DB_PORT
            valueFrom:
              configMapKeyRef:
                key: DB_PORT
                name: platform-params-pf-database
          - name: DB_ADMIN_PASSWORD
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_PASSWORD
                name: platform-secret-pf-database
          - name: DB_ADMIN_USER
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_USER
                name: platform-secret-pf-database
          image: mariadb:10.9
          imagePullPolicy: IfNotPresent
          name: platform-db-backup
          resources: {}
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: false
            runAsGroup: 1000
            runAsNonRoot: true
            runAsUser: 1000
        restartPolicy: Always
        securityContext: {}
        serviceAccount: default
        serviceAccountName: default
      _EOF_

   | バックアップを取得します。

   .. code-block:: bash
      :caption: 実行結果

      kubectl exec -it platform-db-backup -- sh -c 'mysqldump -h ${DB_HOST} -u ${DB_ADMIN_USER} -p${DB_ADMIN_PASSWORD} --all-databases' | gzip > mysqldump_platform-db_`date +"%Y%m%d-%H%M%S"`.sql.gz


サービス再開
------------

| 説明
| Xは置き換えて欲しい旨の記載

.. code-block::

   kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=X
   kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=X
   kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=X
   kubectl scale deployment ita-by-menu-create -n exastro --replicas=X
   kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=X
   kubectl scale deployment platform-auth -n exastro-platform --replicas=X


リストア手順(オンプレ)
======================

deployの確認
------------

| 説明
| 確認したdeployは控えておく旨の記載がいるかもしれない

.. code-block::

   kubectl get deploy -o jsonpath='{range .items[*]}{@.metadata.name}{"\t"}{@.spec.replicas}{"\n"}' -n exastro-platform;
   kubectl get deploy -o jsonpath='{range .items[*]}{@.metadata.name}{"\t"}{@.spec.replicas}{"\n"}' -n exastro;


apache停止
----------

| 説明
| 絶対にapacheの停止ではないので確認をする

.. code-block::

   kubectl scale deployment platform-auth -n exastro-platform --replicas=0


バックヤードサービス停止
------------------------

| 説明

.. code-block::

   kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=0
   kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=0
   kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=0
   kubectl scale deployment ita-by-menu-create -n exastro --replicas=0
   kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=0


DBリストア
--------------

| 説明
| コマンドが合っているか不安なので要確認

.. code-block::

   kubectl exec -it mariadb-XXX --mysqldump -u ${DB_ADMIN_USER} -p ${DB_ADMIN_PASSWORD} --all-databases > db_backup.sql


サービス再開
------------

| 説明
| Xは置き換えて欲しい旨の記載

.. code-block:: 

   kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=X
   kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=X
   kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=X
   kubectl scale deployment ita-by-menu-create -n exastro --replicas=X
   kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=X
   kubectl scale deployment platform-auth -n exastro-platform --replicas=X

