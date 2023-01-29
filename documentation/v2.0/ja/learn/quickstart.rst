================
クイックスタート
================

はじめに
=========

| 本書は、はじめて IT Automation（以下ITAと記載）に触れるユーザが、ITAのインタフェースをスムーズに体感できるクイックスタートの手順書としてご活用できます。
| システム構築においてよくあるLinuxサーバのパッケージのインストール作業を通して、構築対象サーバごとの作業とパッケージ管理を自動化・一元管理化を行い、
| 従来のシステム構築とは異なるITAを使用した効率的なシステム構築を体感できます。

.. figure::  ../../../images/learn/quickstart/common/overview1.png
      :alt: ITAでできること
      :align: left
      :width: 700px
      

本クイックスタートで体感できる主なITA機能の範囲
-----------------------------------------------

- 自動化ソフトウェア(Ansible)との連携
- パラメータ管理(メニュー作成・登録・履歴管理等)
- 変数紐付け(代入値自動登録)

  .. figure::  ../../../images/learn/quickstart/common/overview2.png
      :alt: ITAの機能範囲
      :align: left
      :width: 700px
   
本シナリオと作業範囲の位置づけ
------------------------------

| 本シナリオではAnsibleドライバを使用し、Linuxサーバ構築で実施するyumパッケージのインストール作業を構築対象サーバごとにパラメータ管理し、
| 構築作業の自動化を行う内容となっています。

- 作業環境

  ..  figure:: ../../../images/learn/quickstart/common/workenvironment.png
      :alt: 作業環境
      :align: left
      :width: 700px
    
- 使用するシステム

  - Exastro IT Automation 1.10.0
  - CentOS Linux 7.8(ITAサーバ用)
  - CentOS Linux 7.8(ターゲットマシン用)
  - Windows 10(クライアント)
  - Google Chrome (Win10側)



シナリオ実行イメージ
********************

.. figure:: ../../../images/learn/quickstart/common/executionimage1.png
    :alt: 実行イメージ①
    :align: left
    :width: 700px
    

インストール後からAnsible-Legacyを実行するまでのシナリオ
********************************************************

| シナリオと、開発者(実行前準備)／作業者(実行操作)の作業範囲については以下の通りです。

.. figure:: ../../../images/learn/quickstart/common/executionimage2.png
    :alt: 実行イメージ②
    :align: left
    :width: 700px
    
 
各種用語の説明
--------------

.. list-table:: 本シナリオに登場する主な用語
   :widths: 10  20
   :header-rows: 1
   :align: left


   * - 用語
     - 説明
    
   * - Playbook
     - | 定型業務をタスクで記述し、Ansibleに実行させるためのファイルです。
       | YAML形式で使用します。
     
   * - Ansible-Legacy
     - | ITA から Ansible を利用する機能です。
       | Legacy コンソールでは、構築コードとして単体のYAMLファイルを使う場合に使用します。

   * - オペレーション名(operation)
     - | ITA での作業実行単位です。
       | 作業予定、実行履歴などを管理することができます。

   * - Conductor
     - | ITA での一連の作業の単位です。
       | オペレーション名と関連付けて実行します。
       | Node と呼ぶ各種パーツを組み合わせて、ジョブフローを作成し、
       | 複数の機器に対して、一連の構築・設定などの作業を行います。

   * - Movement
     - | 各機器に対する構築ツールを使った構築、設定などの作業の単位です。

    

画面説明
=========

Webコンソール画面(ログイン)
---------------------------

| ITAのインストールが完了しURLへアクセスすると、ログイン画面が表示されます。
| ※インストール手順については” IT Automation オンラインインストールをご参照ください。

.. figure:: ../../../images/learn/quickstart/login&mainmenu/v1.0_login.png
    :alt: ログイン
    :align: left
    :width: 700px

画面説明(メインメニュー)
------------------------

| **画面は以下のように表示されます。**

| メイングループ、メインメニュー

.. figure:: ../../../images/learn/quickstart/login&mainmenu/v1.0_mainmenu1.png
    :alt: ログイン
    :align: left
    :width: 700px
    
| サブメニュー概略①

.. figure::  ../../../images/learn/quickstart/login&mainmenu/v1.0_mainmenu2.png
    :alt: ログイン
    :align: left
    :width: 700px
   
| サブメニュー概略②

.. figure::  ../../../images/learn/quickstart/login&mainmenu/v1.0_mainmenu3.png
    :alt: ログイン
    :align: left
    :width: 700px


実行前準備
==========

Playbookをアップロードしてジョブ(Movement)に紐付け 
--------------------------------------------------

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

#. メインメニューより、「Ansible-Legacy」メニューグループ >>「Movement一覧」メニューをクリックします。
#. 「登録開始」ボタンをクリックします。
#. 各項目へ下表のように入力し、登録をクリックしてください。

.. figure:: ../../../images/learn/quickstart/preparation/v1.0_legacy_movement_register.png
    :alt: Movement登録
    :align: left
    :width: 700px
    
.. list-table:: Movement一覧
   :widths: 10 10 20
   :header-rows: 1
   :align: left

   * - Movement名
     - ホスト指定形式
     - オプションパラメータ
   * - パッケージインストール
     - IP
     - -vvv
       
   
「Playbook素材集」へ新規Playbookを登録
**************************************

| 次に作成したPlaybookの登録を行います。

#. 「Ansible-Legacy」メニューグループ >>「Playbook素材集」メニューをクリックします。
#. 登録開始をクリックし、各項目へ下表のように入力し登録をクリックしてください。

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_legacy_playbook_register1.png
    :alt: Playbook登録
    :align: left
    :width: 700px
    
.. list-table:: Playbook素材集
   :widths: 10  20
   :header-rows: 1
   :align: left

   * - Playbook素材名
     - Playbook素材
   * - yum_package_install 
     - yum_package_install.yml


| 次に登録したPlaybookをMovementに紐付けます。

「Movement-Playbook紐付」への登録
*********************************

#. 「Ansible-Legacy」メニューグループ >>「Movement-Playbook紐付」メニューをクリックします。
#.  各項目へ下表のように入力、選択し登録をクリックしてください。

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_legacy_playbook_register2.png
    :alt: Movement-Playbook紐づけ登録
    :align: left
    :width: 700px


.. list-table:: Movement-Playbook紐付
   :widths: 10 10 20
   :header-rows: 1
   :align: left

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

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_conductor_edit.gif
    :alt: Conductor作成
    :align: left
    :width: 700px
    

CMDBにパラメータシートを設定
----------------------------

パラメータシートを作成する
**************************

| 次にパラメーターシートの作成を行います。

#. 「メニュー作成」メニューグループ >>「メニュー定義・作成」メニューをクリックします。
#. 各項目へ下表のように入力、選択して下さい。

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_menu_create1.gif
    :alt: パラメータシート作成1
    :align: left
    :width: 500px

.. list-table:: パラメータシートの作成
   :widths: 10 10 10 5
   :header-rows: 1
   :align: left
  

   * - グループ名
     - メニュー名
     - 作業対象
     - 表示順序
   * - インストールパッケージ
     - インストール/パッケージ一覧
     - パラメータシート(ホスト/オペレーション)
     - 1
     
| 項目を追加し、各項目へ下表のように入力、選択して下さい。

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_menu_create2.png
    :alt: パラメータシート作成2
    :align: left
    :width: 500px


.. list-table:: パラメータシートの作成
   :widths: 10 10 20 
   :header-rows: 1
   :align: left

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

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_menu_create3.gif
    :alt: パラメータシート作成2
    :align: left
    :width: 700px



パラメータシートの項目とPlaybookの変数の紐付け
----------------------------------------------

「代入値自動登録設定」作成 
**************************

| 最後に代入値自動登録を行います。

#. 「Ansible-Legacy」メニューグループ >>「代入値自動登録設定」メニューをクリックします。
#. 各項目へ下表のように入力、選択して下さい。
#. 各項目へ入力、選択が完了したら登録をクリックして下さい。

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_substitution_value_automatic_registration1.png
    :alt: パラメータシート作成2
    :align: left
    :width: 500px
    

.. list-table:: 代入値自動登録設定
   :widths: 10 10 3 7 7 3
   :header-rows: 1
   :align: left

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

.. figure::   ../../../images/learn/quickstart/preparation/v1.0_substitution_value_automatic_registration2.png
    :alt: 代入値自動登録確認
    :align: left
    :width: 700px
    


実行操作(1回目)
===============

機器一覧にターゲットとなるLinuxマシンを登録
--------------------------------------------

「機器一覧」へ新規ターゲットホストの登録
****************************************

#. 最初に機器一覧へ今回パッケージをインストールするターゲットホストを登録します。
#. 「基本コンソール」メニューグループ >>「機器一覧」メニューをクリックします。
#. 各項目へ下表のように入力して下さい。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_devicelist1.png
    :alt: 機器一覧登録
    :align: left
    :width: 500px

.. list-table:: 機器一覧
   :widths: 10 10 10
   :header-rows: 1
   :align: left

   * - HW機器種別
     - ホスト名
     - IPアドレス
   * - SV
     - (任意のホスト名)
     - (任意のIPアドレス)
  


| スクロールバーを右にスライドし各項目へ下表のように入力して下さい。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_devicelist2.png
    :alt: 機器一覧登録
    :align: left
    :width: 500px

.. list-table:: 機器一覧
   :widths: 10 10 10
   :header-rows: 1
   :align: left

   * - ログインユーザID
     - ログインパスワード管理
     - ログインパスワード
   * - (任意のログインユーザID)
     - ●
     - (任意のパスワード)
     

| 最後の項目へ下表のように選択し登録をクリックして下さい。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_devicelist3.png
    :alt: 機器一覧登録
    :align: left
    :width: 500px

.. list-table:: 機器一覧
   :widths: 10 
   :header-rows: 1
   :align: left

   * - Legacy/Role利用情報認証方式
   * - パスワード認証
   

.. note::
  | Ansible-Legacyを実行するための必須入力項目は以下の6項目です。
  | [ホスト名][IPアドレス][ログインユーザID][ログインパスワード管理][ログインパスワード][認証方式]

作業名(Operation)の登録
-----------------------

「オペレーション一覧」へ新規オペレーション名を登録
**************************************************

| 次にオペレーション名を登録していきます。

#. 「基本コンソール」メニューグループ >>「オペレーション一覧」メニューをクリックします。
#. 各項目へ入力、選択が完了したら登録をクリックして下さい。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_operation_registration.png
    :alt: operation
    :align: left
    :width: 500px
    

.. list-table:: オペレーション登録
   :widths: 10 10
   :header-rows: 1
   :align: left

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

.. figure::   ../../../images/learn/quickstart/execution/v1.0_dataregistration1.png
    :alt: パラメータシートにデータを登録
    :align: left
    :width: 700px


.. list-table:: 入力用登録
   :widths: 10 10 5 5 5 5 5 
   :header-rows: 1
   :align: left

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

.. figure::   ../../../images/learn/quickstart/execution/v1.0_dataregistration2.png
    :alt: インストールパッケージ一覧への登録
    :align: left
    :width: 500px


Conductorの実行
---------------

Conductorの実行
***************

| いよいよ実行を行っていきます。

#. 「Conductor」メニューグループ >>「Conductor作業実行」メニューをクリックします。
#. 実行する「Conductor」と「オペレーション」を選択し実行をクリックして下さい。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_conductor1.png
    :alt: Conductor実行
    :align: left
    :width: 500px


実行結果確認
************

| 実行すると「Conductor作業確認」メニュー画面に切替わり、実行ステータスやログが表示されます。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_conductor2.png
    :alt: 実行結果の確認
    :align: left
    :width: 700px


| ジョブ(Movement)を選択し、Doneのアイコンまたは右側のOperation statusをクリックすると詳細が表示されます。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_conductor3.png
    :alt: 実行結果の確認
    :align: left
    :width: 700px



実行結果の確認
--------------

実行ログの確認
**************


| 詳細画面の進行状況(実行ログ)でAnsibleの実行ログを確認していきます。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_executionresult1.png
    :alt: 実行結果の確認
    :align: left
    :width: 500px

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
 
.. figure::   ../../../images/learn/quickstart/execution2/v1.0_operation2.png
    :alt: Operation登録
    :align: left
    :width: 500px


.. list-table:: オペレーション登録
   :widths: 10 10
   :header-rows: 1
   :align: left

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

.. figure::   ../../../images/learn/quickstart/execution2/v1.0_dataregistration3.png
    :alt: Operation登録
    :align: left
    :width: 700px


.. list-table:: 入力用登録
   :widths: 10 10 5 5 5 5 5 
   :header-rows: 1
   :align: left

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

.. figure::   ../../../images/learn/quickstart/execution2/v1.0_conductor4.png
    :alt: Conducorの実行
    :align: left
    :width: 700px


| **作業結果の確認**
| 実行すると「Conductor作業確認」メニュー画面に切替わり、実行ステータスやログが表示されます。

.. figure::   ../../../images/learn/quickstart/execution/v1.0_conductor2.png
    :alt: Conducorの実行
    :align: left
    :width: 700px

.. note::
  | 実行ステータスやログをリアルタイムで確認可能です。

| ジョブ(Movement)を選択し、Doneのアイコンまたは右側のOperation statusをクリックすると詳細が表示されます。

.. figure::   ../../../images/learn/quickstart/execution2/v1.0_conductor5.png
    :alt: 作業結果確認
    :align: left
    :width: 700px

実行結果の確認
--------------

| 詳細画面の進行状況(実行ログ)でAnsibleの実行ログを確認します。

実行ログの確認
**************

| 新たにMariaDBのインストールと他のパッケージとの依存関係の解決、他の4つのパッケージ(httpd,php,perl,python)の
| バージョンアップが行われていることを確認して下さい。

.. figure::   ../../../images/learn/quickstart/execution2/v1.0_exectuionresult2.png
    :alt: Conducorの実行
    :align: left
    :width: 700px

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

| ITAはCMDBに「誰が・いつ・何をしたのか？」を履歴管理し、その時の時点でシステムのパラメータはどうなっているのかを抽出できる機能があります。
| パラメータの履歴管理をすることにより、設計者や運用者がストレスなくシステム更改を行うことができます。

.. figure:: ../../../images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory1.png
    :alt: 履歴確認
    :align: left
    :width: 700px
    
CMDBパラメータの履歴を確認する
------------------------------

履歴確認
********

| 実際にパラメータが管理できているかどうか確認をしていきます。
| 「参照用」メニューグループ >>「インストールパッケージ一覧」メニューをクリックします。
|  まずは基準日付を入力せずにフィルタをかけます。

.. figure:: ../../../images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory2.png
    :alt: 履歴確認
    :align: left
    :width: 700px

| 次に2回目の実行を行った基準日時より前の日付を入力してフィルタをかけます。

.. figure:: ../../../images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory3.png
    :alt: 履歴確認
    :align: left
    :width: 700px

| 最後に1回目の実行を行った基準日時より前の日付を入力してフィルタをかけます。

.. figure:: ../../../images/learn/quickstart/Historycheck_CMDB_parameters/v1.0_checkhistory4.png
    :alt: 履歴確認
    :align: left
    :width: 700px


A. 付録
=======

参考① 【Ansible-Legacy】単体実行
---------------------------------

作業実行
********

| Ansible-Legacyは「作業実行」メニューがあり、Movementごとに個別実行や、ドライランが可能です。

.. figure:: ../../../images/learn/quickstart/reference/v1.0_singleexecution.png
    :alt: 単体実行
    :align: left
    :width: 700px



参考② 【Ansible-Legacy】実行確認
---------------------------------

作業結果確認
*************

| 実行(またはドライラン)すると画面が切替わり、実行ステータスやログが表示されます。

.. figure:: ../../../images/learn/quickstart/reference/v1.0_executionconfirmation.png
    :alt: 実行確認
    :align: left
    :width: 700px

| クイックスタートは、以上となります。








