============================
データバックアップ・リストア
============================

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

.. _check_replica_count_backup:

バックアップ
============

レプリカ数の確認
----------------

| 作業前のレプリカ数の確認をし、状態を記録します。

.. code-block:: bash
  :caption: コマンド

  RS_AE=`kubectl get deploy ita-by-ansible-execute -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_ALRV=`kubectl get deploy ita-by-ansible-legacy-role-vars-listup -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_ATS=`kubectl get deploy ita-by-ansible-towermaster-sync -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_CS=`kubectl get deploy ita-by-conductor-synchronize -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_MC=`kubectl get deploy ita-by-menu-create -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_PA=`kubectl get deploy platform-auth -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`

リバースプロキシの停止
----------------------

| リバースプロキシ (platform-auth) のレプリカ数を 0 に変更し、エンドユーザーからのアクセスを制限します。

.. code-block:: bash
  :caption: コマンド

  kubectl scale deployment platform-auth -n exastro --replicas=0

バックヤード処理の停止
----------------------

| バックヤード処理 (ita-by-\*\*\*) のレプリカ数を 0 に変更し、データベースの更新を停止します。

.. code-block:: bash
  :caption: コマンド
  :linenos:

  kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=0
  kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=0
  kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=0
  kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=0
  kubectl scale deployment ita-by-menu-create -n exastro --replicas=0

Pod 起動数の確認
----------------

| 上記で停止した対象の Pod 数が 0 になっていることを確認

.. code-block:: bash
  :caption: コマンド

  kubectl get replicaset -n exastro

.. code-block:: bash
  :caption: 実行結果

  NAME                                                DESIRED   CURRENT   READY   AGE
  mariadb-67dd78cc76                                  1         1         1       6d22h
  platform-web-9f9d486fd                              1         1         1       6d22h
  ita-api-admin-85b7d8f977                            1         1         1       6d22h
  ita-web-server-7dbf6fd6ff                           1         1         1       6d22h
  ita-api-organization-5c5f4b86cb                     1         1         1       6d22h
  platform-api-8655864fbf                             1         1         1       6d22h
  keycloak-7f7cdccb6b                                 1         1         1       6d22h
  platform-auth-5b57bc57bd                            0         0         0       6d22h
  ita-by-ansible-execute-6cd6d4d5fd                   0         0         0       6d22h
  ita-by-ansible-legacy-role-vars-listup-67dbf5586f   0         0         0       6d22h
  ita-by-ansible-towermaster-sync-5674448c55          0         0         0       6d22h
  ita-by-conductor-synchronize-9dc6cfbdf              0         0         0       6d22h
  ita-by-menu-create-7fccfc7f57                       0         0         0       6d22h

バックアップ
------------

| Exastro システム内で、Exastro Platform と Exastro IT Automation でデータベースを共有するか、分離するかによって手順が異なります。

.. tabs::

  .. group-tab:: データベースを共有

      1. メンテナンス用コンテナを作成します。

      | Exastro Platform のデータベース(Exastro IT Automation と共有)を :command:`mysqldump` を使ってバックアップします。

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
              - name: STORAGEPATH
                valueFrom:
                  configMapKeyRef:
                    key: STORAGEPATH
                    name: ita-params-ita-global
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

      2. データベースのバックアップを取得します。

      .. code-block:: bash
          :caption: コマンド

          kubectl exec -it platform-db-backup -n exastro -- sh -c 'mysqldump -h ${DB_HOST} -P ${DB_PORT} -u ${DB_ADMIN_USER} -p${DB_ADMIN_PASSWORD} --all-databases --add-drop-table' | gzip > mysqldump_platform-db_`date +"%Y%m%d-%H%M%S"`.sql.gz

      3. ファイルのバックアップを取得します。

      .. code-block:: bash
          :caption: コマンド

          kubectl exec -i platform-db-backup -n exastro -- sh -c 'tar zcvf - ${STORAGEPATH}' > exastro_storage_backup_`date +"%Y%m%d-%H%M%S"`.tar.gz

      4. 作業用に起動した Pod を削除します。

      .. code-block:: bash
          :caption: コマンド

          kubectl delete pod platform-db-backup -n exastro

  .. group-tab:: データベースを分離

      | Exastro Platform のデータベースを :command:`mysqldump` を使ってバックアップします。

      1. メンテナンス用コンテナを作成します。

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
              - name: STORAGEPATH
                valueFrom:
                  configMapKeyRef:
                    key: STORAGEPATH
                    name: ita-params-ita-global
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

      2. Exastro Platform のデータベースのバックアップを取得します。

      .. code-block:: bash
          :caption: 実行結果

          kubectl exec -it platform-db-backup -n exastro -- sh -c 'mysqldump -h ${DB_HOST} -P ${DB_PORT} -u ${DB_ADMIN_USER} -p${DB_ADMIN_PASSWORD} --all-databases --add-drop-table' | gzip > mysqldump_platform-db_`date +"%Y%m%d-%H%M%S"`.sql.gz

      3. ファイルのバックアップを取得します。

      .. code-block:: bash
          :caption: コマンド

          kubectl exec -i platform-db-backup -n exastro -- sh -c 'tar zcvf - ${STORAGEPATH}' > exastro_storage_backup_`date +"%Y%m%d-%H%M%S"`.tar.gz


      4. メンテナンス用コンテナを作成します。

      | Exastro IT Automation のデータベースを :command:`mysqldump` を使ってバックアップします。

      .. code-block:: bash
          :caption: コマンド
          :linenos:

          cat <<_EOF_ | kubectl apply -f - -n exastro
          apiVersion: v1
          kind: Pod
          metadata:
            name: ita-db-backup
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
                    name: ita-params-ita-database
              - name: DB_HOST
                valueFrom:
                  configMapKeyRef:
                    key: DB_HOST
                    name: ita-params-ita-database
              - name: DB_PORT
                valueFrom:
                  configMapKeyRef:
                    key: DB_PORT
                    name: ita-params-ita-database
              - name: DB_ADMIN_PASSWORD
                valueFrom:
                  secretKeyRef:
                    key: DB_ADMIN_PASSWORD
                    name: ita-secret-ita-database
              - name: DB_ADMIN_USER
                valueFrom:
                  secretKeyRef:
                    key: DB_ADMIN_USER
                    name: ita-secret-ita-database
              image: mariadb:10.9
              imagePullPolicy: IfNotPresent
              name: ita-db-backup
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

      5. Exastro IT Automation のデータベースのバックアップを取得します。

      .. code-block:: bash
          :caption: コマンド

          kubectl exec -it ita-db-backup -n exastro -- sh -c 'mysqldump -h ${DB_HOST} -P ${DB_PORT} -u ${DB_ADMIN_USER} -p${DB_ADMIN_PASSWORD} --all-databases --add-drop-table' | gzip > mysqldump_ita-db_`date +"%Y%m%d-%H%M%S"`.sql.gz

      6. ファイルのバックアップを取得します。

      .. code-block:: bash
          :caption: コマンド

          kubectl exec -it ita-db-backup -n exastro -- sh -c 'mysqldump -h ${DB_HOST} -P ${DB_PORT} -u ${DB_ADMIN_USER} -p${DB_ADMIN_PASSWORD} --all-databases --add-drop-table' | gzip > mysqldump_ita-db_`date +"%Y%m%d-%H%M%S"`.sql.gz

      7. 作業用に起動した Pod を削除します。

      .. code-block:: bash
        :caption: コマンド

        kubectl delete pod platform-db-backup -n exastro
        kubectl delete pod ita-db-backup -n exastro

サービス再開
------------

:ref:`check_replica_count_backup` で取得した各 Deployment のレプリカ数を元に戻します。


.. code-block::
  :caption: コマンド
  :linenos:

  kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=${RS_AE}
  kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=${RS_ALRV}
  kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=${RS_ATS}
  kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=${RS_CS}
  kubectl scale deployment ita-by-menu-create -n exastro --replicas=${RS_MC}
  kubectl scale deployment platform-auth -n exastro --replicas=${RS_PA}

Pod 起動数の再確認
------------------

| 上記で起動した対象の Pod 数が元に戻りすべて :kbd:`READY` になっていることを確認

.. code-block:: bash
  :caption: コマンド

  kubectl get replicaset -n exastro

.. code-block:: bash
  :caption: 実行結果

  NAME                                                DESIRED   CURRENT   READY   AGE
  mariadb-67dd78cc76                                  1         1         1       6d22h
  platform-web-9f9d486fd                              1         1         1       6d22h
  ita-api-admin-85b7d8f977                            1         1         1       6d22h
  ita-web-server-7dbf6fd6ff                           1         1         1       6d22h
  ita-api-organization-5c5f4b86cb                     1         1         1       6d22h
  platform-api-8655864fbf                             1         1         1       6d22h
  keycloak-7f7cdccb6b                                 1         1         1       6d22h
  ita-by-ansible-execute-6cd6d4d5fd                   1         1         1       6d22h
  ita-by-ansible-legacy-role-vars-listup-67dbf5586f   1         1         1       6d22h
  ita-by-ansible-towermaster-sync-5674448c55          1         1         1       6d22h
  ita-by-conductor-synchronize-9dc6cfbdf              1         1         1       6d22h
  ita-by-menu-create-7fccfc7f57                       1         1         1       6d22h
  platform-auth-5b57bc57bd                            1         1         1       6d22h


.. _check_replica_count_restore:

リストア
========

レプリカ数の確認
----------------

| 作業前のレプリカ数の確認をし、状態を記録します。

.. code-block:: bash
  :caption: コマンド

  RS_AE=`kubectl get deploy ita-by-ansible-execute -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_ALRV=`kubectl get deploy ita-by-ansible-legacy-role-vars-listup -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_ATS=`kubectl get deploy ita-by-ansible-towermaster-sync -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_CS=`kubectl get deploy ita-by-conductor-synchronize -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_MC=`kubectl get deploy ita-by-menu-create -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
  RS_PA=`kubectl get deploy platform-auth -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`

リバースプロキシの停止
----------------------

| リバースプロキシ (platform-auth) のレプリカ数を 0 に変更し、エンドユーザーからのアクセスを制限します。

.. code-block:: bash
  :caption: コマンド

  kubectl scale deployment platform-auth -n exastro --replicas=0

バックヤード処理の停止
----------------------

| バックヤード処理 (ita-by-\*\*\*) のレプリカ数を 0 に変更し、データベースの更新を停止します。

.. code-block:: bash
  :caption: コマンド
  :linenos:

  kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=0
  kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=0
  kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=0
  kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=0
  kubectl scale deployment ita-by-menu-create -n exastro --replicas=0

Pod 起動数の確認
----------------

| 上記で停止した対象の Pod 数が0になっていることを確認

.. code-block:: bash
  :caption: コマンド

  kubectl get replicaset -n exastro

.. code-block:: bash
  :caption: 実行結果

  NAME                                                DESIRED   CURRENT   READY   AGE
  mariadb-67dd78cc76                                  1         1         1       6d22h
  platform-web-9f9d486fd                              1         1         1       6d22h
  ita-api-admin-85b7d8f977                            1         1         1       6d22h
  ita-web-server-7dbf6fd6ff                           1         1         1       6d22h
  ita-api-organization-5c5f4b86cb                     1         1         1       6d22h
  platform-api-8655864fbf                             1         1         1       6d22h
  keycloak-7f7cdccb6b                                 1         1         1       6d22h
  platform-auth-5b57bc57bd                            0         0         0       6d22h
  ita-by-ansible-execute-6cd6d4d5fd                   0         0         0       6d22h
  ita-by-ansible-legacy-role-vars-listup-67dbf5586f   0         0         0       6d22h
  ita-by-ansible-towermaster-sync-5674448c55          0         0         0       6d22h
  ita-by-conductor-synchronize-9dc6cfbdf              0         0         0       6d22h
  ita-by-menu-create-7fccfc7f57                       0         0         0       6d22h

リストア
--------

| Exastro システム内で、Exastro Platform と Exastro IT Automation でデータベースを共有するか、分離するかによって手順が異なります。

.. tabs::

  .. group-tab:: データベースを共有

      1. メンテナンス用コンテナを作成します。

      | Exastro Platform のデータベース(Exastro IT Automation と共有)を :command:`mysql` を使ってリストアします。

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
              - name: STORAGEPATH
                valueFrom:
                  configMapKeyRef:
                    key: STORAGEPATH
                    name: ita-params-ita-global
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

      2. データベースのリストアを実施します。

      .. code-block:: bash
        :caption: コマンド

        gzip -dc mysqldump_platform-db_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i platform-db-backup -n exastro -- sh -c 'mysql -h ${DB_HOST} -P ${DB_PORT} -u ${DB_ADMIN_USER} -p${DB_ADMIN_PASSWORD}'

      3. ファイルのリストアを実施します。

      .. code-block:: bash
        :caption: コマンド

        kubectl exec -i platform-db-backup -n exastro -- sh -c 'tar zxvf - -C  ${STORAGEPATH}' < exastro_storage_backup_YYYYMMDD-HHmmss.tar.gz

      4. 作業用に起動した Pod を削除します。

      .. code-block:: bash
        :caption: コマンド

        kubectl delete pod platform-db-backup -n exastro

  .. group-tab:: データベースを分離

      1. メンテナンス用コンテナを作成します。

      | Exastro Platform のデータベースを :command:`mysql` を使ってリストアします。


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
              - name: STORAGEPATH
                valueFrom:
                  configMapKeyRef:
                    key: STORAGEPATH
                    name: ita-params-ita-global
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

      2. データベースをリストアします。

      .. code-block:: bash
          :caption: 実行結果

          gzip -dc mysqldump_platform-db_YYYYMMDD-HHmmss.sql.gz | kubectl exec -i platform-db-backup -n exastro -- sh -c 'mysql -h ${DB_HOST} -P ${DB_PORT} -u ${DB_ADMIN_USER} -p${DB_ADMIN_PASSWORD}'

      3. ファイルをリストアします。

      .. code-block:: bash
          :caption: 実行結果

          kubectl exec -i platform-db-backup -n exastro -- sh -c 'tar zxvf - -C  ${STORAGEPATH}' < exastro_storage_backup_YYYYMMDD-HHmmss.tar.gz

      4. メンテナンス用コンテナを作成します。

      | Exastro IT Automation のデータベースを :command:`mysql` を使ってリストアします。

      .. code-block:: bash
          :caption: コマンド
          :linenos:

          cat <<_EOF_ | kubectl apply -f - -n exastro
          apiVersion: v1
          kind: Pod
          metadata:
            name: ita-db-backup
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
                    name: ita-params-ita-database
              - name: DB_HOST
                valueFrom:
                  configMapKeyRef:
                    key: DB_HOST
                    name: ita-params-ita-database
              - name: DB_PORT
                valueFrom:
                  configMapKeyRef:
                    key: DB_PORT
                    name: ita-params-ita-database
              - name: DB_ADMIN_PASSWORD
                valueFrom:
                  secretKeyRef:
                    key: DB_ADMIN_PASSWORD
                    name: ita-secret-ita-database
              - name: DB_ADMIN_USER
                valueFrom:
                  secretKeyRef:
                    key: DB_ADMIN_USER
                    name: ita-secret-ita-database
              image: mariadb:10.9
              imagePullPolicy: IfNotPresent
              name: ita-db-backup
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

      5. 作業用に起動した Pod を削除します。

      .. code-block:: bash
        :caption: コマンド

        kubectl delete pod platform-db-backup -n exastro
        kubectl delete pod ita-db-backup -n exastro

サービス再開
------------

:ref:`check_replica_count_restore` で取得した各 Deployment のレプリカ数を元に戻します。


.. code-block::
  :caption: コマンド
  :linenos:

  kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=${RS_AE}
  kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=${RS_ALRV}
  kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=${RS_ATS}
  kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=${RS_CS}
  kubectl scale deployment ita-by-menu-create -n exastro --replicas=${RS_MC}
  kubectl scale deployment platform-auth -n exastro --replicas=${RS_PA}

Pod 起動数の再確認
------------------

| 上記で起動した対象の Pod 数が元に戻りすべて :kbd:`READY` になっていることを確認

.. code-block:: bash
  :caption: コマンド

  kubectl get replicaset -n exastro

.. code-block:: bash
  :caption: 実行結果

  NAME                                                DESIRED   CURRENT   READY   AGE
  mariadb-67dd78cc76                                  1         1         1       6d22h
  platform-web-9f9d486fd                              1         1         1       6d22h
  ita-api-admin-85b7d8f977                            1         1         1       6d22h
  ita-web-server-7dbf6fd6ff                           1         1         1       6d22h
  ita-api-organization-5c5f4b86cb                     1         1         1       6d22h
  platform-api-8655864fbf                             1         1         1       6d22h
  keycloak-7f7cdccb6b                                 1         1         1       6d22h
  ita-by-ansible-execute-6cd6d4d5fd                   1         1         1       6d22h
  ita-by-ansible-legacy-role-vars-listup-67dbf5586f   1         1         1       6d22h
  ita-by-ansible-towermaster-sync-5674448c55          1         1         1       6d22h
  ita-by-conductor-synchronize-9dc6cfbdf              1         1         1       6d22h
  ita-by-menu-create-7fccfc7f57                       1         1         1       6d22h
  platform-auth-5b57bc57bd                            1         1         1       6d22h