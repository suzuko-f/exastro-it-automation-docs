=======================================================
Exastro IT Automation　サーバ連携／バックアップ対象一覧
=======================================================

バックアップ手順(オンプレ)
==========================

deployの確認
------------

| 説明
| 確認したdeployは控えておく旨の記載がいるかもしれない

.. code-block::

   kubectl get deploy -o jsonpath='{range .items[*]}{@.metadata.name}{"\t"}{@.spec.replicas}{"\n"}' -n exastro-platform;
   kubectl get deploy -o jsonpath='{range .items[*]}{@.metadata.name}{"\t"}{@.spec.replicas}{"\n"}' -n exastro-it-automation;


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

   kubectl scale deployment ita-by-ansible-execute -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-conductor-synchronize -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-menu-create -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro-it-automation --replicas=0


DBバックアップ
--------------

| 説明
| mariadb-XXXをどうするか要確認

.. code-block::

   kubectl exec -it mariadb-XXX --mysqldump -u ${DB_ADMIN_USER} -p ${DB_ADMIN_PASSWORD} --all-databases > db_backup.sql


サービス再開
------------

| 説明
| Xは置き換えて欲しい旨の記載

.. code-block::

   kubectl scale deployment ita-by-ansible-execute -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-conductor-synchronize -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-menu-create -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro-it-automation --replicas=X
   kubectl scale deployment platform-auth -n exastro-platform --replicas=X


リストア手順(オンプレ)
======================

deployの確認
------------

| 説明
| 確認したdeployは控えておく旨の記載がいるかもしれない

.. code-block::

   kubectl get deploy -o jsonpath='{range .items[*]}{@.metadata.name}{"\t"}{@.spec.replicas}{"\n"}' -n exastro-platform;
   kubectl get deploy -o jsonpath='{range .items[*]}{@.metadata.name}{"\t"}{@.spec.replicas}{"\n"}' -n exastro-it-automation;


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

   kubectl scale deployment ita-by-ansible-execute -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-conductor-synchronize -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-menu-create -n exastro-it-automation --replicas=0
   kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro-it-automation --replicas=0


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

   kubectl scale deployment ita-by-ansible-execute -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-conductor-synchronize -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-menu-create -n exastro-it-automation --replicas=X
   kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro-it-automation --replicas=X
   kubectl scale deployment platform-auth -n exastro-platform --replicas=X

