======
ロール
======

はじめに
========

| 本章では、Exastro Suite におけるロールについて説明します。


ロールとは
==========

| ロールとは、ユーザやワークスペース内のデータといった Exastro システムにおけるリソースに対する操作(作成、更新、削除)権限の集合のことです。
| Exastro Suiteのロールの種類としては以下のものがあります。

- | オーガナイゼーションロール
  | オーガナイゼーションの管理を行うロールで以下の３つがあります

.. list-table:: オーガナイゼーションロール
   :widths: 20 30
   :header-rows: 1
   :align: left
      
   * - **ロール**
     - **説明**
   * - | オーガナイゼーション管理者
       | （_organization-manager)
     - ワークスペースの追加やユーザーおよびロールの管理が可能です。
   * - | ユーザー・ロール管理者
       | （_organization-user-role-manager)
     - ユーザーおよびロールの管理が可能です。
   * - | ユーザー管理者
       | (_organization-user-manager)
     - ユーザーの管理が可能です
  

- | ワークスペースロール

.. list-table:: ワークスペースロール
   :widths: 20 30
   :header-rows: 1
   :align: left
      
   * - **ロール**
     - **説明**
   * - | ワークスペース管理者
       | (_{ワークスペースID}-admin)
     - ワークスペースの管理が可能です。
   * - カスタムロール
     - | ユーザーが作成したロールで使用可能なワークスペースを指定および、Exastro IT Automationの使用可能なメニューを指定出来ます。
       | （Exastro IT Automationの使用可能なメニューの設定はXXXXを参照）


ロール作成とユーザへの紐づけ
============================

| ロールの新規作成方法とユーザへの紐づけ方法について下記の流れで説明します。

#. | :ref:`role_workspace`
   | ロールごとにワークスペースへのアクセス権限をメンテナンスできます。 
#. | :ref:`role_user`
   | ユーザ毎にアクセスを許可するロールを付与することで、ユーザごとにワークスペースへのアクセスを制御することができます。

.. _role_workspace:

ロール追加・ワークスペース紐づけ
--------------------------------

#. | Exastro Platformにオーガナイゼーション管理者 または _${ws-id}-adminロールに紐づくユーザでログインします。


#. | Exastro Platformのメニューより :menuselection:`ロール管理` をクリックします。

   .. image:: /images/ja/manuals/platform/platform_menu.png
      :width: 200px
      :align: left
      
   .. note:: | オーガナイゼーション管理者（ロール管理・ロール付与権限を有するユーザー）またはワークスペース管理者のユーザーでログインしている時のみ、メニューに :menuselection:`ロール管理` が表示されます。

#. | :menuselection:`ロール一覧` 画面が表示されるので、:guilabel:`作成` ボタンをクリックします。

   .. figure:: /images/ja/manuals/platform/role/platform_role_create.png
      :width: 600px
      :align: left

#. | :menuselection:`新規ロール` 画面が表示されるので、ロールの情報を入力し、:guilabel:`登録` ボタンをクリックします。

   .. figure:: /images/ja/manuals/platform/role/platform_role_register.png
      :width: 600px
      :align: left

| :menuselection:`使用ワークスペース` にワークスペースの一覧が表示されるので、当該ロールに紐づけるワークスペースを選択します。
    
.. figure:: /images/ja/manuals/platform/role/platform_role_workspace_used.png
   :width: 600px
   :align: left

.. note:: | ワークスペース管理者でログインしているときは、ログインしているユーザーがワークスペース管理者となっているワークスペースのみ選択可能です。

.. _role_user:

ユーザ・ロール紐づけ
--------------------

#. | Exastro Platformにオーガナイゼーション管理者 または _${ワークスペースID}-adminロールに紐づくユーザでログインします。
#. | Exastro Platformのメニューより :menuselection:`ロール管理` をクリックします。

   .. image:: /images/ja/manuals/platform/platform_menu.png
      :width: 200px
      :align: left

   .. note:: | オーガナイゼーション管理者、_${ws-id}-adminロールに紐づくユーザ以外でログインしている時は、メニューに :menuselection:`ロール管理` は表示されません。

#. | :menuselection:`ロール一覧` 画面が表示されるので、ユーザとロールの紐づけを変更したいロールを選択し :guilabel:`ユーザ` ボタンをクリックします。

   .. figure:: /images/ja/manuals/platform/role/platform_role_list.png
      :width: 600px
      :align: left

#. | :menuselection:`ロール付与・解除` 画面が表示されるので、ロールとの紐づけを変更したいユーザを選択し、 :guilabel:`付与` ボタンをクリックします。

   .. figure:: /images/ja/manuals/platform/role/platform_role_grant.png
      :width: 600px
      :align: left
