============================
データバックアップ・リストア
============================

目的
====

| 本頁では、Exastroシステムで利用する永続データのバックアップとリストア手順について説明します。
| パブリッククラウドの仕組みにより、データバックアップの仕組みが利用可能であればそちらを利用しても問題ありません。


前提条件
========

| 本頁で紹介するバックアップ・リストア手順では、下記の前提条件を満たしている必要があります。

条件
----

- | 下記のコマンドが利用可能なこと

  - :command:`tar`
  - :command:`kubectl`

- | 作業環境のサーバで充分なディスクの空き容量があること


暗号キーのバックアップ
----------------------

.. include:: ../../include/backup_encrypt_key_k8s.rst
  

| 取得したキーを exastro.yaml の 「ENCRYPT_KEY」に格納してください。

- | Exastro IT Automation ENCRYPT_KEY
  
.. code-block:: bash

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
      ENCRYPT_KEY: ""　# 取得した Exastro IT Automation ENCRYPT_KEY を入力

- | Exastro Platform ENCRYPT_KEY
  
.. code-block:: bash
  
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
      ENCRYPT_KEY: ""  # 取得した Exastro Platform ENCRYPT_KEY を入力


概要
====

| バックアップ・リストア対象となるデータは下記の3つです。

- | バックアップ・リストア対象

  - Exastro Platform のデータベース
  - Exastro IT Automation のデータベース
  - Exastro IT Automation の共有ファイル

| これらのデータを :command:`kubectl` コマンドを利用してバックアップ・リストアします。

| 作業の流れは、まず、ユーザからのデータの書き込みを制限するために、リバースプロキシを停止します。
| 次にバックヤード処理を停止し、データのバックアップを実施します。
| 最後に、作業前の数に Pod 数を戻します。

| リストアの際の作業も同様の流れになります。

.. danger::
  | 本手順では、サービスの停止が発生します。

バックアップ
============

サービス停止
------------

.. include:: ../../include/stop_service_k8s.rst

バックアップ
------------

| Exastro システム内で、Exastro Platform と Exastro IT Automation でデータベースを共有するか、分離するかによって手順が異なります。


#. メンテナンス用コンテナの作成

   | バックアップ作業用コンテナの作成をします。

   .. code-block:: bash
      :caption: コマンド
      :linenos:

      cat <<_EOF_ | kubectl apply -f - -n exastro
      apiVersion: v1
      kind: Pod
      metadata:
        name: exastro-maintenance
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
          - name: PF_DB_DATABASE
            valueFrom:
              configMapKeyRef:
                key: DB_DATABASE
                name: platform-params-pf-database
          - name: PF_DB_HOST
            valueFrom:
              configMapKeyRef:
                key: DB_HOST
                name: platform-params-pf-database
          - name: PF_DB_PORT
            valueFrom:
              configMapKeyRef:
                key: DB_PORT
                name: platform-params-pf-database
          - name: PF_DB_ADMIN_PASSWORD
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_PASSWORD
                name: platform-secret-pf-database
          - name: PF_DB_ADMIN_USER
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_USER
                name: platform-secret-pf-database
          - name: ITA_DB_DATABASE
            valueFrom:
              configMapKeyRef:
                key: DB_DATABASE
                name: ita-params-ita-database
          - name: ITA_DB_HOST
            valueFrom:
              configMapKeyRef:
                key: DB_HOST
                name: ita-params-ita-database
          - name: ITA_DB_PORT
            valueFrom:
              configMapKeyRef:
                key: DB_PORT
                name: ita-params-ita-database
          - name: ITA_STORAGEPATH
            valueFrom:
              configMapKeyRef:
                key: STORAGEPATH
                name: ita-params-ita-global
          - name: ITA_DB_ADMIN_PASSWORD
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_PASSWORD
                name: ita-secret-ita-database
          - name: ITA_DB_ADMIN_USER
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_USER
                name: ita-secret-ita-database
          image: mariadb:10.9
          imagePullPolicy: IfNotPresent
          name: exastro-maintenance
          resources: {}
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: false
            runAsGroup: 1000
            runAsNonRoot: true
            runAsUser: 1000
          volumeMounts:
          - mountPath: /storage
            name: volume-ita-backup-storage
        volumes:
        - name: volume-ita-backup-storage
          persistentVolumeClaim:
            claimName: pvc-ita-global
        restartPolicy: Always
        securityContext: {}
        serviceAccount: default
        serviceAccountName: default
      _EOF_

#. データベースのバックアップ取得

   | データベースに対して :command:`mysqldump` バックアップを取得します。
   | Exastro Platform と Exastro IT Automation でデータベースサーバを共有するか、分離するかによって手順が異なります。

   .. tabs::

      .. group-tab:: データベースサーバを共有

          .. code-block:: bash
             :caption: Exastro 用データベースバックアップコマンド

             # アプリケーション
             kubectl exec -it exastro-maintenance -n exastro -- sh -c 'mysqldump -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD} --all-databases --add-drop-table' | gzip > exastro_mysqldump_platform_db_`date +"%Y%m%d-%H%M%S"`.sql.gz

             # ユーザ
             kubectl exec -it exastro-maintenance -n exastro -- sh -c 'mysqldump -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD} --allow-keywords mysql' | gzip > exastro_mysqldump_platform_user_`date +"%Y%m%d-%H%M%S"`.sql.gz

      .. group-tab:: データベースサーバを分離

          .. code-block:: bash
             :caption: Exastro Platform 用データベースバックアップコマンド

             # アプリケーション
             kubectl exec -it exastro-maintenance -n exastro -- sh -c 'mysqldump -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD} --all-databases --add-drop-table' | gzip > exastro_mysqldump_platform_db_`date +"%Y%m%d-%H%M%S"`.sql.gz

             # ユーザ
             kubectl exec -it exastro-maintenance -n exastro -- sh -c 'mysqldump -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD} --allow-keywords mysql' | gzip > exastro_mysqldump_platform_user_`date +"%Y%m%d-%H%M%S"`.sql.gz

          .. code-block:: bash
             :caption: Exastro IT Automation 用データベースバックアップコマンド

             # アプリケーション
             kubectl exec -it exastro-maintenance -n exastro -- sh -c 'mysqldump -h ${ITA_DB_HOST} -P ${ITA_DB_PORT} -u ${ITA_DB_ADMIN_USER} -p${ITA_DB_ADMIN_PASSWORD} --all-databases --add-drop-table' | gzip > exastro_mysqldump_ita_db_`date +"%Y%m%d-%H%M%S"`.sql.gz

             # ユーザ
             kubectl exec -it exastro-maintenance -n exastro -- sh -c 'mysqldump -h ${ITA_DB_HOST} -P ${ITA_DB_PORT} -u ${ITA_DB_ADMIN_USER} -p${ITA_DB_ADMIN_PASSWORD} --allow-keywords mysql' | gzip > exastro_mysqldump_ita_user_`date +"%Y%m%d-%H%M%S"`.sql.gz

#. ファイルのバックアップ取得

   | Exastro IT Automation のファイルのバックアップを取得します。

   .. code-block:: bash
      :caption: コマンド

      kubectl exec -i exastro-maintenance -n exastro -- sh -c 'tar zcvf - ${ITA_STORAGEPATH}' > exastro_storage_backup_ita_`date +"%Y%m%d-%H%M%S"`.tar.gz

#. メンテナンス用コンテナの削除

   | バックアップ作業用コンテナの作成をします。

   .. code-block:: bash
      :caption: コマンド

      kubectl delete pod exastro-maintenance -n exastro

サービス再開
------------

.. include:: ../../include/start_service_k8s.rst


リストア
========

.. _check_replica_count_restore:

サービス停止
------------

.. include:: ../../include/stop_service_k8s.rst

リストア
--------

| Exastro システム内で、Exastro Platform と Exastro IT Automation でデータベースを共有するか、分離するかによって手順が異なります。

#. メンテナンス用コンテナの作成

   | リストア作業用コンテナの作成をします。

   .. code-block:: bash
      :caption: コマンド
      :linenos:

      cat <<_EOF_ | kubectl apply -f - -n exastro
      apiVersion: v1
      kind: Pod
      metadata:
        name: exastro-maintenance
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
          - name: PF_DB_DATABASE
            valueFrom:
              configMapKeyRef:
                key: DB_DATABASE
                name: platform-params-pf-database
          - name: PF_DB_HOST
            valueFrom:
              configMapKeyRef:
                key: DB_HOST
                name: platform-params-pf-database
          - name: PF_DB_PORT
            valueFrom:
              configMapKeyRef:
                key: DB_PORT
                name: platform-params-pf-database
          - name: PF_DB_ADMIN_PASSWORD
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_PASSWORD
                name: platform-secret-pf-database
          - name: PF_DB_ADMIN_USER
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_USER
                name: platform-secret-pf-database
          - name: ITA_DB_DATABASE
            valueFrom:
              configMapKeyRef:
                key: DB_DATABASE
                name: ita-params-ita-database
          - name: ITA_DB_HOST
            valueFrom:
              configMapKeyRef:
                key: DB_HOST
                name: ita-params-ita-database
          - name: ITA_DB_PORT
            valueFrom:
              configMapKeyRef:
                key: DB_PORT
                name: ita-params-ita-database
          - name: ITA_STORAGEPATH
            valueFrom:
              configMapKeyRef:
                key: STORAGEPATH
                name: ita-params-ita-global
          - name: ITA_DB_ADMIN_PASSWORD
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_PASSWORD
                name: ita-secret-ita-database
          - name: ITA_DB_ADMIN_USER
            valueFrom:
              secretKeyRef:
                key: DB_ADMIN_USER
                name: ita-secret-ita-database
          image: mariadb:10.9
          imagePullPolicy: IfNotPresent
          name: exastro-maintenance
          resources: {}
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: false
            runAsGroup: 1000
            runAsNonRoot: true
            runAsUser: 1000
          volumeMounts:
          - mountPath: /storage
            name: volume-ita-backup-storage
        volumes:
        - name: volume-ita-backup-storage
          persistentVolumeClaim:
            claimName: pvc-ita-global
        restartPolicy: Always
        securityContext: {}
        serviceAccount: default
        serviceAccountName: default
      _EOF_

#. データベースのリストア実施

   | データベースに対して :command:`mysqldump` リストアを実施します。
   | Exastro Platform と Exastro IT Automation でデータベースサーバを共有するか、分離するかによって手順が異なります。

   .. tabs::

      .. group-tab:: データベースサーバを共有

          .. code-block:: bash
             :caption: Exastro 用データベースリストアコマンド

             # ユーザ
             gzip -dc exastro_mysqldump_platform_user_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i exastro-maintenance -n exastro -- sh -c 'mysql -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD} mysql'

             # アプリケーション
             gzip -dc exastro_mysqldump_platform_db_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i exastro-maintenance -n exastro -- sh -c 'mysql -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD}'

      .. group-tab:: データベースサーバを分離

          .. code-block:: bash
             :caption: Exastro Platform 用データベースリストアコマンド

             # ユーザ
             gzip -dc exastro_mysqldump_platform_user_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i exastro-maintenance -n exastro -- sh -c 'mysql -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD} mysql'

             # アプリケーション
             gzip -dc exastro_mysqldump_platform_db_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i exastro-maintenance -n exastro -- sh -c 'mysql -h ${PF_DB_HOST} -P ${PF_DB_PORT} -u ${PF_DB_ADMIN_USER} -p${PF_DB_ADMIN_PASSWORD}'

          .. code-block:: bash
             :caption: Exastro IT Automation 用データベースリストアコマンド

             # ユーザ
             gzip -dc exastro_mysqldump_ita_user_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i exastro-maintenance -n exastro -- sh -c 'mysql -h ${ITA_DB_HOST} -P ${ITA_DB_PORT} -u ${ITA_DB_ADMIN_USER} -p${ITA_DB_ADMIN_PASSWORD} mysql'

             # アプリケーション
             gzip -dc exastro_mysqldump_ita_db_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i exastro-maintenance -n exastro -- sh -c 'mysql -h ${ITA_DB_HOST} -P ${ITA_DB_PORT} -u ${ITA_DB_ADMIN_USER} -p${ITA_DB_ADMIN_PASSWORD}'

#. ファイルのリストア実施

   | Exastro IT Automation のファイルのバックアップを取得します。

   .. code-block:: bash
      :caption: コマンド

      kubectl exec -i exastro-maintenance -n exastro -- sh -c 'tar zxvf - -C ${ITA_STORAGEPATH}' < exastro_storage_backup_ita_YYYYMMDD-HHmmss.tar.gz

#. メンテナンス用コンテナの削除

   | バックアップ作業用コンテナの作成をします。

   .. code-block:: bash
      :caption: コマンド

      kubectl delete pod exastro-maintenance -n exastro

サービス再開
------------

.. include:: ../../include/start_service_k8s.rst
