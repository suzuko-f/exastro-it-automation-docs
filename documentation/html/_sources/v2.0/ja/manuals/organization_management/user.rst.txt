========
ユーザー
========


はじめに
========

| 本章では、Exastro Suite におけるユーザーについて説明します。


ユーザーとは
============

| Exastro システムの利用者のことを指します。ユーザーのタイプには大きく分けて下記の3種類が存在します。

- システム管理者
- オーガナイゼーション管理者
- ユーザー

.. list-table:: Exastro システムの利用者
    :widths: 20, 40
    :header-rows: 1
    :align: left

    * - ユーザーのタイプ
      - 内容
    * - システム管理者
      - | ・Exastro システム全体の管理者です。
        | ・システム全体に関わる操作を実行出来ます（オーガナイゼーションやオーガナイゼーション管理者の作成、リソースプランの設定）。
        | ・オーガナイゼーション内のワークスペース、および、Exastro IT Automation に関する操作は実行出来ません。
    * - オーガナイゼーション管理者
      - | ・オーガナイゼーションの管理者です。
        | ・システム管理者により作成されたオーガナイゼーションに対してユーザーの作成やロールの割当といったオーガナイゼーションの管理をするための操作を実行出来ます。
    * - ユーザー
      - | ・上記2つ以外のすべてのユーザーです。
        | ・Exastro の各ツールの操作を実行出来ます。
        
        
| 上記のユーザー毎に作成方法が異なりますので、それぞれの方法について説明します。


システム管理者
==============

システム管理者の作成
--------------------

| システム管理者は、インストール時に :ref:`installation_kubernetes_Keycloak 設定` で :kbd:`global.keycloakDefinition.secret.KEYCLOAK_USER` に指定しインストールした際に作成されます。


システム管理者の追加
--------------------

| 登録済みのシステム管理者アカウントから新たなシステム管理者アカウントを追加します。

手順
~~~~
1. | keycloak管理コンソール(masterレルム)に接続します。
   
- | 接続URL
  
.. code-block:: bash
   
   {システム管理者用サイトアドレス}/auth/admin/master/console/#/realms/master/users


2. | keycloakのログイン画面から登録済みのシステム管理者アカウントでログインします。

.. figure:: /images/ja/manuals/platform/keycloak/keycloak_login.png
   :width: 600px
   :alt: keycloakログイン画面 

   keycloakログイン画面

.. note:: | システム管理者アカウントを未登録の場合は、インストール時に platform-secret.yaml に設定し、keycloakの環境変数へ登録されている値でログインします。
   
   .. figure:: /images/ja/diagram/keycloak_user_password.png
        :width: 400px
        :alt: platform-secret.yaml 

        platform-secret.yaml


1. | アカウントの追加を行います。

- | :menuselection:`「keycloak管理コンソール」 --> 「ユーザー」画面` の :guilabel:`ユーザーの追加` ボタンをクリックします。

.. figure:: /images/ja/manuals/platform/keycloak/keycloak_management_console.png
   :width: 600px
   :alt: keycloak管理コンソール_ユーザー画面 

   keycloak管理コンソール_ユーザー画面


- | :menuselection:`「ユーザー追加」画面` で、登録するシステム管理者の情報を入力し :guilabel:`保存` ボタンをクリックします。
  
.. figure:: /images/ja/manuals/platform/keycloak/keycloak_add_user.png
   :width: 600px
   :alt: ユーザー追加画面

   ユーザー追加画面

- | ユーザーの追加後の画面で :guilabel:`クレデンシャルタブ` をクリックします。

.. figure:: /images/ja/manuals/platform/keycloak/keycloak_credential_tab.png
   :width: 600px
   :alt: クレデンシャルタブ

   クレデンシャルタブ

- | クレデンシャルタブの画面で、パスワード、新しいパスワード（確認）を入力し :guilabel:`パスワードを設定` をクリックします。

.. figure:: /images/ja/manuals/platform/keycloak/keycloak_password_setting.png
   :width: 600px
   :alt: パスワード設定

   パスワード設定


.. note:: | 一時的「オン」：該当ユーザーが初回ログイン時、パスワード変更が要求されます。（推奨）
      | 一時的「オフ」：入力したパスワードを、そのまま利用することができます。

- | :guilabel:`ロールマッピング` タブを選択し、レルムロールの中の「admin」をアサイン済みロールに設定します。

.. figure:: /images/ja/manuals/platform/keycloak/keycloak_role_mapping.png
   :width: 600px
   :alt: ロールマッピング

   ロールマッピング

4. |  システム管理者アカウントの追加完了です。

| ユーザーの追加、パスワードの設定、追加したユーザーの「admin」ロールへのアサインをもってシステム管理者アカウントの追加は完了です。

オーガナイゼーション管理者
==========================

オーガナイゼーション管理者の作成
--------------------------------

| オーガナイゼーション管理者は、オーガナイゼーション作成時に :doc:`../platform_management/organization` で :kbd:`organization_managers` に指定することで作成されます。


ユーザー
========

ユーザーの作成
--------------

| ユーザーは、下記の方法で作成します。

#. | Exastro Platformにオーガナイゼーション管理者でログインします。


#. | Exastro Platformのメニューより :menuselection:`ユーザー管理` をクリックします。

   .. image:: /images/ja/manuals/platform/platform_menu.png
      :width: 200px
      :align: left

   .. note:: | ユーザー管理権限を有するロールに紐づくユーザー以外でログインしている時は、メニューに :menuselection:`ユーザー管理` は表示されません。
      | ユーザー管理権限を有するロールは以下の3つです。
      | _orgnization-manager,_orgnization-user-manager,_orgnization-user-role-manager

#. | keycloakのユーザー画面が表示されるので、 :guilabel:`ユーザーの追加` ボタンをクリックします。

   .. figure:: /images/ja/manuals/platform/keycloak/keycloak_management_console.png
      :width: 600px
      :align: left

#. | keycloakのユーザーの追加画面が表示されるので、ユーザーの情報を入力して :guilabel:`保存` ボタンをクリックします。

   .. figure:: /images/ja/manuals/platform/keycloak/keycloak_add_user_member01.png
      :width: 600px
      :align: left


#. | 登録が正常に終了すると、以下の画面が表示されるので、 :menuselection:`クレデンシャル` タブをクリックします。

   .. figure:: /images/ja/manuals/platform/keycloak/keycloak_credential_tab_member01.png
      :width: 600px
      :align: left


#. | 「パスワード」および「新しいパスワード（確認）」に初期パスワードを入力し :guilabel:`パスワードを設定` をクリックします。

   .. figure:: /images/ja/manuals/platform/keycloak/keycloak_password_setting_member01.png
      :width: 600px
      :align: left

   .. note:: | 一時的「オン」：該当ユーザーが初回ログイン時、パスワード変更が要求されます。（推奨）
      | 一時的「オフ」：入力したパスワードを、そのまま利用することができます。

#. | 確認ダイアログが表示されるので、 :guilabel:`Set password` ボタンをクリックします。

   .. figure:: /images/ja/manuals/platform/keycloak/keycloak_set_password.png
      :width: 600px
      :align: left
