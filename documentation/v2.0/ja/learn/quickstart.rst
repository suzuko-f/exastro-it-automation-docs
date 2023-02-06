================
クイックスタート
================


目的
====

| 本頁は、はじめて Exastro IT Automation（以下、ITA とも記載）に触れるユーザが、 Exastro IT Automation のインタフェースをスムーズに体感することを目的とした教材です。
| システム構築においてよくある Linux サーバのパッケージのインストール作業を通して、構築対象サーバごとの作業とパッケージ管理を自動化・一元管理化を行い、従来のシステム構築とは異なる Exastro IT Automation を使用した効率的なシステム構築を体感できます。

.. figure::  /images/learn/quickstart/common/overview1.png
   :width: 600px


概要
====

| 本シナリオでは Ansible ドライバを使用した Linux サーバ上への yum パッケージのインストールを行います。
| その際に、構築対象のサーバ毎にことなるパッケージをインストールする必要がありますが、Exastro IT Automation のパラメータ管理を利用し、効率的に構築作業の自動化を行っていきます。

.. figure:: /images/learn/quickstart/common/executionimage1.png
    :width: 600px

前提条件
--------

| 本シナリオを進める前に、下記の環境を準備しておく必要があります。
| Exastro IT Automation のインストール方法は、:doc:`../installation/index` を参照してください。

- 使用するシステム

  - Exastro IT Automation 2.1
  - AlmaLinux Linux 8 (ターゲットマシン用)
  - 作業端末

    - Windows 10
    - Google Chrome

..  figure:: /images/learn/quickstart/common/workenvironment.png
    :width: 600px

   

作業内容
--------

| シナリオと、開発者(実行前準備)／作業者(実行操作)の作業内容については以下の通りです。

.. figure:: /images/learn/quickstart/common/executionimage2.png
    :width: 600px
    
 

機能の範囲
----------

| 本シナリオでは、Exastro IT Automation を利用する上で最もよく使われるであろう下記の機能に絞って紹介します。

- 本シナリオで扱う機能

  - 自動化ソフトウェア(Ansible)との連携
  - パラメータ管理(メニュー作成・登録・履歴管理等)
  - 変数紐付け(代入値自動登録)

.. figure::  /images/learn/quickstart/common/overview2.png
   :width: 600px


各種用語の説明
--------------

.. list-table:: 本シナリオに登場する主な用語
   :widths: 10  20
   :header-rows: 1


   * - 用語
     - 説明
    
   * - Playbook
     - | 定型業務をタスクで記述し、Ansible に実行させるためのファイルです。
       | YAML形式で使用します。
     
   * - Ansible-LegacyRole
     - | Exastro IT Automation から Ansible Role を利用する機能です。
       | LegacyRole コンソールでは、構築コードとして YAML ファイルのセットを使う場合に使用します。

   * - オペレーション名(operation)
     - | Exastro IT Automation での作業実行単位です。
       | 作業予定、実行履歴などを管理することができます。

   * - Conductor
     - | Exastro IT Automation での一連の作業の単位です。
       | オペレーション名と関連付けて実行します。
       | Node と呼ぶ各種パーツを組み合わせて、ジョブフローを作成し、
       | 複数の機器に対して、一連の構築・設定などの作業を行います。

   * - Movement
     - | 各機器に対する構築ツールを使った構築、設定などの作業の単位です。


オーガナイゼーション作成
========================

| Exastro システムをインストールした直後は、Exastro IT Automation を使うためにまず、オーガナイゼーションの作成を行います。
| オーガナイゼーションの詳細については、:doc:`../manuals/platform_management/organization` を参照してください。


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

   | オーガナイゼーション作成時の初期登録情報として下記の項目を登録します。

   .. list-table:: 作成するオーガナイゼーション情報
     :widths: 25 40
     :header-rows: 1
     :align: left

     * - 設定項目
       - 設定値
     * - organization id
       - :program:`qs-org`
     * - organization name
       - :program:`Quickstart organization`
     * - organization manager's username
       - :program:`qs-admin`
     * - organization manager's email
       - :program:`qs-admin@example.com`
     * - organization manager's firstName
       - :program:`quickstart`
     * - organization manager's lastName
       - :program:`administrator`
     * - organization manager's initial password
       - :program:`password`

   .. code-block:: sh
      :caption: コマンド 

      bash ./exastro-platform/tools/create-organization.sh

   | :kbd:`your username` と :kbd:`your username` は :ref:`create_system_manager` で登録した、:kbd:`KEYCLOAK_USER` 及び :kbd:`KEYCLOAK_PASSWORD` です。

   .. code-block::
      :caption: コマンド (入力例)

      Please enter the organization information to be created
  
      organization id : qs-org                             # オーガナイゼーションIDを入力します
      organization name : Quickstart organization          # オーガナイゼーション名を入力します
      organization manager's username : qs-admin           # オーガナイゼーション管理者のユーザ名（ログインするときのID）を入力します
      organization manager's email : qs-admin@example.com  # オーガナイゼーション管理者のE-mailアドレスを入力します
      organization manager's first name : quickstart       # オーガナイゼーション管理者の名を入力します
      organization manager's last name : administrator     # オーガナイゼーション管理者の姓を入力します
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


ワークスペース作成
==================

#. 作成したオーガナイゼーションにオーガナイゼーション管理者でログインします。

   http://exastro-suite.example.local/qs-org/platform/

   .. figure:: /images/learn/quickstart/login/login.png
      :width: 300px
      :align: left

   .. list-table:: ログイン情報
     :widths: 25 50
     :header-rows: 1
     :align: left

     * - 入力項目
       - 入力値
     * - URL
       - :program:`http://exastro-suite.example.local/qs-org/platform/`
     * - ユーザ名
       - :program:`qs-admin`
     * - パスワード
       - :program:`password`

#. 初回ログイン時のみパスワードとアカウントの更新を実施します。

   .. figure:: /images/learn/quickstart/login/change-password.png
      :width: 300px
      :align: left

      パスワードの更新

   .. figure:: /images/learn/quickstart/login/change-account-info.png
      :width: 300px
      :align: left

      アカウント情報の更新

#. | :menuselection:`メインメニュー --> ワークスペース一覧` で、 :guilabel:`作成` をクリックします。

   .. figure:: /images/ja/manuals/platform/workspace/ワークスペース一覧.png
      :width: 600px
      :align: left

   .. warning::
      | オーガナイゼーション管理者以外でログインしている時は、 :guilabel:`作成` は表示されません。

#. | 新規ワークスペース画面が表示されるので、ワークスペースの情報を入力し、 :guilabel:`登録` をクリックします。

   .. figure:: /images/ja/manuals/platform/workspace/新規ワークスペース作成画面.png
      :width: 600px
      :align: left

   .. list-table:: ワークスペース設定項目
      :widths: 60 80
      :header-rows: 1
      :align: left
   
      * - 項目名
        - 説明
      * - ワークスペースID
        - :program:`quickstart`
      * - ワークスペース名
        - :program:`クイックスタート`
      * - 環境一覧
        - 
      * - 説明
        - 

Exastro IT Automation 実行前作業
================================

Exastro IT Automation 画面
--------------------------

#. | :menuselection:`メインメニュー --> ワークスペース一覧` から、 :menuselection:`quickstart` ワークスペースにある :guilabel:`Exastro IT Automation` をクリックし、ITA の操作画面に遷移します。

.. figure:: /images/learn/quickstart/login/select-workspace.png
   :width: 600px
   :align: left


画面説明
--------

| 画面は以下のように表示されます。


.. figure:: /images/learn/quickstart/login&mainmenu/v1.0_mainmenu1.png
   :width: 600px
    
   メイングループ、メインメニュー

.. figure::  /images/learn/quickstart/login&mainmenu/v1.0_mainmenu2.png
   :width: 600px
   
   サブメニュー概略①

.. figure::  /images/learn/quickstart/login&mainmenu/v1.0_mainmenu3.png
   :width: 600px

   サブメニュー概略②


Playbookをアップロードしてジョブ(Movement)に紐付け 
--------------------------------------------------

|  Exastro IT Automation のインストールが完了しURLへアクセスすると、ログイン画面が表示されます。
| ※インストール手順については” IT Automation オンラインインストールをご参照ください。

.. figure:: /images/learn/quickstart/login&mainmenu/v1.0_login.png
    :width: 600px

画面説明(メインメニュー)
------------------------

Playbookの準備
**************

| 最初に今回利用するPlaybookの作成をします。
| お好みのエディタを使用してymlを作成し自身のローカルフォルダに保存してください。  
   
-  yum_package_install.yml
 
   .. code:: yaml

    - name: install the latest version of packages
      yum:
        name: "{{ item }}"
        state: latest
      with_items:
        - "{{ VAR_packages }}
      
.. warning::
  | 文字コードは ”UTF-8 BOMなし” 、改行コードは ”LF” 、 Linuxマシンを登録拡張子は ”yml” 形式です。
  | また、インデントにご注意ください。

   
Movement一覧へ新規Movementを登録
********************************

| 次にMovementの登録を行っていきます。

#. メインメニューより、「Ansible-LegacyRole」メニューグループ >>「Movement一覧」メニューをクリックします。
#. :guilabel:`登録開始` をクリックします。
#. 各項目へ下表のように入力し、登録をクリックしてください。

.. figure:: /images/learn/quickstart/preparation/v1.0_legacy_movement_register.png
    :width: 600px
    
.. list-table:: Movement一覧
   :widths: 10 10 20
   :header-rows: 1

   * - Movement名
     - ホスト指定形式
     - オプションパラメータ
   * - パッケージインストール
     - IP
     - -vvv
       
   
「Playbook素材集」へ新規Playbookを登録
**************************************

| 次に作成したPlaybookの登録を行います。

#. 「Ansible-LegacyRole」メニューグループ >>「Playbook素材集」メニューをクリックします。
#. 登録開始をクリックし、各項目へ下表のように入力し登録をクリックしてください。

.. figure::   /images/learn/quickstart/preparation/v1.0_legacy_playbook_register1.png
    :width: 600px
    
.. list-table:: Playbook素材集
   :widths: 10  20
   :header-rows: 1

   * - Playbook素材名
     - Playbook素材
   * - yum_package_install 
     - yum_package_install.yml


| 次に登録したPlaybookをMovementに紐付けます。

「Movement-Playbook紐付」への登録
*********************************

#. 「Ansible-LegacyRole」メニューグループ >>「Movement-Playbook紐付」メニューをクリックします。
#.  各項目へ下表のように入力、選択し登録をクリックしてください。

.. figure::   /images/learn/quickstart/preparation/v1.0_legacy_playbook_register2.png
    :width: 600px


.. list-table:: Movement-Playbook紐付
   :widths: 10 10 20
   :header-rows: 1

   * - Movement
     - Playbook素材
     - インクルード順序
   * - パッケージインストール
     - yum_package_install
     - 1
    
    
     
ジョブ(Movement)をジョブフロー(Conductor)に組込み
-------------------------------------------------

「Conductor」を作成する
***********************

| 次にMovementをConductorに組み込んでいきます。

#. 「Conductor」メニューグループ >>「Conductorクラス編集」メニューをクリックします。
#. 下記の通りConducor名に「パッケージインストール」と入力、しMovementを移動、連結させ登録をクリックしてください。

.. figure::   /images/learn/quickstart/preparation/v1.0_conductor_edit.gif
    :width: 600px
    

CMDBにパラメータシートを設定
----------------------------

パラメータシートを作成する
**************************

| 次にパラメーターシートの作成を行います。

#. 「メニュー作成」メニューグループ >>「メニュー定義・作成」メニューをクリックします。
#. 各項目へ下表のように入力、選択して下さい。

.. figure::   /images/learn/quickstart/preparation/v1.0_menu_create1.gif
    :width: 600px

.. list-table:: パラメータシートの作成
   :widths: 10 10 10 5
   :header-rows: 1
  

   * - グループ名
     - メニュー名
     - 作業対象
     - 表示順序
   * - インストールパッケージ
     - インストール/パッケージ一覧
     - パラメータシート(ホスト/オペレーション)
     - 1
     
| 項目を追加し、各項目へ下表のように入力、選択して下さい。

.. figure::   /images/learn/quickstart/preparation/v1.0_menu_create2.png
    :width: 600px


.. list-table:: パラメータシートの作成
   :widths: 10 10 20 
   :header-rows: 1

   * - 項目名
     - 入力方式
     - 選択項目
   * - httpd
     - プルダウン選択
     - メニュー作成:選択1:\*-(ブランク)
   * - MariaDB-server
     - プルダウン選択
     - メニュー作成:選択1:\*-(ブランク)
   * - php
     - プルダウン選択
     - メニュー作成:選択1:\*-(ブランク) 
   * - perl
     - プルダウン選択
     - メニュー作成:選択1:\*-(ブランク)
   * - python
     - プルダウン選択
     - メニュー作成:選択1:\*-(ブランク) 
     
.. warning::
 | 今回はCentOS7.8を対象としています。
 | CentOS7系以外は「mariadb-server」と小文字で入力してください。

| 項目の移動が完了できたら作成をクリックして下さい。

.. figure::   /images/learn/quickstart/preparation/v1.0_menu_create3.gif
    :width: 600px



パラメータシートの項目とPlaybookの変数の紐付け
----------------------------------------------

「代入値自動登録設定」作成 
**************************

| 最後に代入値自動登録を行います。

#. 「Ansible-LegacyRole」メニューグループ >>「代入値自動登録設定」メニューをクリックします。
#. 各項目へ下表のように入力、選択して下さい。
#. 各項目へ入力、選択が完了したら登録をクリックして下さい。

.. figure::   /images/learn/quickstart/preparation/v1.0_substitution_value_automatic_registration1.png
    :width: 600px
    

.. list-table:: 代入値自動登録設定
   :widths: 10 10 3 7 7 3
   :header-rows: 1

   * - メニューグループ:メニュー
     - 項目
     - 登録方式
     - Movement
     - Key変数/変数名
     - 代入順序
     
   * - 2100011611:代入値自動登録用:3:インストールパッケージ一覧
     - パラメータ/インストールパッケージ/httpd
     -  Key型
     -  1:パッケージインストール
     -  1:VAR_packages
     -  1
       
   * - 2100011611:代入値自動登録用:3:インストールパッケージ一覧
     - パラメータ/インストールパッケージ/MariaDBserver
     - Key型
     - 1:パッケージインストール
     - 1:VAR_packages
     - 2
    
   * - 2100011611:代入値自動登録用:3:インストールパッケージ一覧
     - パラメータ/インストールパッケージ/php
     - Key型 
     - 1:パッケージインストール
     - 1:VAR_packages
     - 3
    
   * - 2100011611:代入値自動登録用:3:インストールパッケージ一覧
     - パラメータ/インストールパッケージ/perl
     - Key型 
     - 1:パッケージインストール
     - 1:VAR_packages
     - 4
      
   * - 2100011611:代入値自動登録用:3:インストールパッケージ一覧
     - パラメータ/インストールパッケージ/python
     - Key型
     - 1:パッケージインストール
     - 1:VAR_packages
     - 5
      
|

.. note::
  | 変数紐づけの登録方式は以下の3タイプがあります。
  
  - | Value型
    | 基本的なタイプであり、表の中の値を変数に紐づけるものです。
  - | Key型
    | 表の項目(列名)を変数に紐づけるものです。項目の設定値が空白の場合は紐づけ対象外になります。
  - | Key-Value型
    | 項目の名称(Key)と設定値(Value)の両方を変数に紐づけることができます。

  | 今回のシナリオでは、表の項目(列名)をPlaybookに具体値として代入したいので、登録方式は「Key型」を選択します。  

 
| 表示フィルタで5件のデータが登録できているかの確認を行って下さい。
| ここまでで実行準備は終了になります。

.. figure::   /images/learn/quickstart/preparation/v1.0_substitution_value_automatic_registration2.png
    :width: 600px
    


実行操作(1回目)
===============

機器一覧にターゲットとなるLinuxマシンを登録
--------------------------------------------

「機器一覧」へ新規ターゲットホストの登録
****************************************

#. 最初に機器一覧へ今回パッケージをインストールするターゲットホストを登録します。
#. 「基本コンソール」メニューグループ >>「機器一覧」メニューをクリックします。
#. 各項目へ下表のように入力して下さい。

.. figure::   /images/learn/quickstart/execution/v1.0_devicelist1.png
    :width: 600px

.. list-table:: 機器一覧
   :widths: 10 10 10
   :header-rows: 1

   * - HW機器種別
     - ホスト名
     - IPアドレス
   * - SV
     - (任意のホスト名)
     - (任意のIPアドレス)
  


| スクロールバーを右にスライドし各項目へ下表のように入力して下さい。

.. figure::   /images/learn/quickstart/execution/v1.0_devicelist2.png
    :width: 600px

.. list-table:: 機器一覧
   :widths: 10 10 10
   :header-rows: 1

   * - ログインユーザID
     - ログインパスワード管理
     - ログインパスワード
   * - (任意のログインユーザID)
     - ●
     - (任意のパスワード)
     

| 最後の項目へ下表のように選択し登録をクリックして下さい。

.. figure::   /images/learn/quickstart/execution/v1.0_devicelist3.png
    :width: 600px

.. list-table:: 機器一覧
   :widths: 10 
   :header-rows: 1

   * - LegacyRole/Role利用情報認証方式
   * - パスワード認証
   

.. note::
  | Ansible-LegacyRoleを実行するための必須入力項目は以下の6項目です。
  | [ホスト名][IPアドレス][ログインユーザID][ログインパスワード管理][ログインパスワード][認証方式]

作業名(Operation)の登録
-----------------------

「オペレーション一覧」へ新規オペレーション名を登録
**************************************************

| 次にオペレーション名を登録していきます。

#. 「基本コンソール」メニューグループ >>「オペレーション一覧」メニューをクリックします。
#. 各項目へ入力、選択が完了したら登録をクリックして下さい。

.. figure::   /images/learn/quickstart/execution/v1.0_operation_registration.png
    :width: 600px
    

.. list-table:: オペレーション登録
   :widths: 10 10
   :header-rows: 1

   * - オペレーション名
     - 実施予定日時
   * - オペレーション1
     - (任意の実行予定日時)
 


パラメータシートにデータを登録
------------------------------
「インストールパッケージ一覧」へ新規データを登録
************************************************

| 次に実行前準備で用意したインストールパッケージ一覧(パラーメータシート)にデータを入力していきます。

#. 「入力用」メニューグループ >>「インストールパッケージ一覧」メニューをクリックします。
#.  各項目へ入力、選択が完了したら登録をクリックして下さい。

.. figure::   /images/learn/quickstart/execution/v1.0_dataregistration1.png
    :width: 600px


.. list-table:: 入力用登録
   :widths: 10 10 5 5 5 5 5 
   :header-rows: 1

   * - ホスト名
     - オペレーション
     - httpd
     - MariaDB-server
     - php
     - perl
     - python
   * - (機器登録で登録したホスト名)
     - (選択した実行予定日時)_1:オペレーション1
     -  \*
     - 
     -  \*
     -  \*
     -  \*
      
    
「インストールパッケージ一覧」への登録
**************************************

| 実行前準備の代入値自動登録設定の時と同様、表示フィルタを開き「フィルタ」ボタンをクリックして登録したデータを確認してください。

.. figure::   /images/learn/quickstart/execution/v1.0_dataregistration2.png
    :width: 600px


Conductorの実行
---------------

Conductorの実行
***************

| いよいよ実行を行っていきます。

#. 「Conductor」メニューグループ >>「Conductor作業実行」メニューをクリックします。
#. 実行する「Conductor」と「オペレーション」を選択し実行をクリックして下さい。

.. figure::   /images/learn/quickstart/execution/v1.0_conductor1.png
    :width: 600px


実行結果確認
************

| 実行すると「Conductor作業確認」メニュー画面に切替わり、実行ステータスやログが表示されます。

.. figure::   /images/learn/quickstart/execution/v1.0_conductor2.png
    :width: 600px


| ジョブ(Movement)を選択し、Doneのアイコンまたは右側のOperation statusをクリックすると詳細が表示されます。

.. figure::   /images/learn/quickstart/execution/v1.0_conductor3.png
    :width: 600px



実行結果の確認
--------------

実行ログの確認
**************


| 詳細画面の進行状況(実行ログ)でAnsibleの実行ログを確認していきます。

.. figure::   /images/learn/quickstart/execution/v1.0_executionresult1.png
    :width: 600px

| httpd,php,perl,pythonをインストールされているか実行ログから確認して下さい。

進行状況(実行ログ)の一部の例
****************************

.. code-block:: bash

   ～～～～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～
   Installed:
 
       httpd.x86_64 0:2.4.6-97.el7.centos
   Dependency Installed: 
       httpd-tools.x86_64 0:2.4.6-97.el7.centos mailcap.noarch 0:2.1.41-2.el7
   Complete! 
   "]}
   ～～～～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～
   Installed: 
       php.x86_64 0:5.4.16-48.el7 
   Dependency Installed: 
       libzip.x86_64 0:0.10.1-8.el7 php-cli.x86_64 0:5.4.16-48.el7 php-common.x86_64 0:5.4.16-48.el7
   Complete! 
   "]}
   ～～～～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～ 
   Updated:
       perl.x86_64 4:5.16.3-299.el7_9 
   Dependency Updated: 
       perl-libs.x86_64 4:5.16.3-299.el7_9 
   Complete!
   "]}
   ～～～～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～
   Updated:
       python.x86_64 0:2.7.5-90.el7 
   Dependency Updated: 
       python-libs.x86_64 0:2.7.5-90.el7 
   Complete! 
   "]}
 


実行結果の確認
--------------

| ターゲットマシンでもパッケージがインストールできていることを確認して下さい。

.. code-block:: bash

   $ yum list installed httpd
 Loaded plugins: fastestmirror, langpacks
 Loading mirror speeds from cached hostfile
  * base: ftp-srv2.kddilabs.jp
  * extras: ftp-srv2.kddilabs.jp
  * updates: ftp-srv2.kddilabs.jp
 Installed Packages
 httpd.x86_64                    2.4.6-97.el7.centos                     @updates


実行操作(2回目)
===============

作業名(Operation)の登録
-----------------------

「オペレーション一覧」へ新規オペレーション名
********************************************

| ここからは1回目のオペレーション名登録以降の作業と同様になります。

#. 「基本コンソール」メニューグループ >>「オペレーション一覧」メニューをクリックします。
#. 各項目へ入力、選択が完了したら登録をクリックして下さい。
 
.. figure::   /images/learn/quickstart/execution2/v1.0_operation2.png
    :width: 600px


.. list-table:: オペレーション登録
   :widths: 10 10
   :header-rows: 1

   * - オペレーション名
     - 実施予定日時
   * - オペレーション2
     - (任意の実行予定日時)
    

パラメータシートにデータを登録
------------------------------

「インストールパッケージ一覧」新規データを登録
**********************************************

#. 「入力用」メニューグループ >>「インストールパッケージ一覧」メニューをクリックします。
#. 各項目へ入力、選択が完了したら登録をクリックして下さい。

| ※1回目とインストールするパッケージが異なっているので注意して下さい。

.. figure::   /images/learn/quickstart/execution2/v1.0_dataregistration3.png
    :width: 600px


.. list-table:: 入力用登録
   :widths: 10 10 5 5 5 5 5 
   :header-rows: 1

   * - ホスト名
     - オペレーション
     - httpd
     - MariaDB-server
     - php
     - perl
     - python
   * - (機器登録で登録したホスト名)
     - (選択した実行予定日時)_2:オペレーション2
     -  \*
     -  \*
     -  \*
     -  \*
     -  \*

Conductorの実行
---------------

| 2回目の実行も行っていきます

#.  「Conductor」メニューグループ >>「Conductor作業実行」メニューをクリックします。
#.  実行する「Conductor」と「オペレーション」を選択し実行をクリックして下さい。

.. figure::   /images/learn/quickstart/execution2/v1.0_conductor4.png
    :width: 600px


| **作業結果の確認**
| 実行すると「Conductor作業確認」メニュー画面に切替わり、実行ステータスやログが表示されます。

.. figure::   /images/learn/quickstart/execution/v1.0_conductor2.png
    :width: 600px

.. note::
  | 実行ステータスやログをリアルタイムで確認可能です。

| ジョブ(Movement)を選択し、Doneのアイコンまたは右側のOperation statusをクリックすると詳細が表示されます。

.. figure::   /images/learn/quickstart/execution2/v1.0_conductor5.png
    :width: 600px

実行結果の確認
--------------

| 詳細画面の進行状況(実行ログ)でAnsibleの実行ログを確認します。

実行ログの確認
**************

| 新たにMariaDBのインストールと他のパッケージとの依存関係の解決、他の4つのパッケージ(httpd,php,perl,python)の
| バージョンアップが行われていることを確認して下さい。

.. figure::   /images/learn/quickstart/execution2/v1.0_exectuionresult2.png
    :width: 600px

| 新たにMariaDBのインストールと他のパッケージとの依存関係の解決、他の4つのパッケージ(httpd,php,perl,python)のバージョンアップが行われていることを確認して下さい。

.. code-block:: bash

  ～～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～ 
  "results": ["All packages providing httpd are up to date",
   ""]}
  ～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～ 
  Installed:
      MariaDB-server.x86_64 0:10.8.4-1.el7.centos
  Dependency Installed: 
      perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7 
      perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7 
      perl-DBD-MySQL.x86_64 0:4.023-6.el7 
      perl-DBI.x86_64 0:1.627-4.el7 
      perl-IO-Compress.noarch 0:2.061-2.el7 
      perl-Net-Daemon.noarch 0:0.48-5.el7 
      perl-PlRPC.noarch 0:0.2020-14.el7
  Complete!
  "]}
  ～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～ 
  "results": ["All packages providing php are up to date",
   ""]} 
  ～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～ 
  "results": ["All packages providing perl are up to date", 
  ""]}
   ～～～～～～～～～～～～～～～～～～～～～省略～～～～～～～～～～～～～～～～～～～～～～～ 
   "results": ["All packages providing python are up to date", 
  ""]} 


CMDBパラメータの履歴確認
========================

作業実行と履歴管理
------------------

履歴管理と本シナリオのポイント
******************************

|  Exastro IT Automation はCMDBに「誰が・いつ・何をしたのか？」を履歴管理し、その時の時点でシステムのパラメータはどうなっているのかを抽出できる機能があります。
| パラメータの履歴管理をすることにより、設計者や運用者がストレスなくシステム更改を行うことができます。

.. figure:: /images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory1.png
    :width: 600px
    
CMDBパラメータの履歴を確認する
------------------------------

履歴確認
********

| 実際にパラメータが管理できているかどうか確認をしていきます。
| 「参照用」メニューグループ >>「インストールパッケージ一覧」メニューをクリックします。
|  まずは基準日付を入力せずにフィルタをかけます。

.. figure:: /images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory2.png
    :width: 600px

| 次に2回目の実行を行った基準日時より前の日付を入力してフィルタをかけます。

.. figure:: /images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory3.png
    :width: 600px

| 最後に1回目の実行を行った基準日時より前の日付を入力してフィルタをかけます。

.. figure:: /images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory4.png
    :width: 600px


A. 付録
=======

参考① 【Ansible-LegacyRole】単体実行
-------------------------------------

作業実行
********

| Ansible-LegacyRoleは「作業実行」メニューがあり、Movementごとに個別実行や、ドライランが可能です。

.. figure:: /images/learn/quickstart/reference/v1.0_singleexecution.png
    :width: 600px



参考② 【Ansible-LegacyRole】実行確認
-------------------------------------

作業結果確認
*************

| 実行(またはドライラン)すると画面が切替わり、実行ステータスやログが表示されます。

.. figure:: /images/learn/quickstart/reference/v1.0_executionconfirmation.png
    :width: 600px

| クイックスタートは、以上となります。








