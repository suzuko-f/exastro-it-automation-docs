=================================================================
Ansible Automation Platform
=================================================================

はじめに
========================================================
| 本書では、Exastro IT Automation（以下、ITAとも記載する）でAnsibleオプション機能（以下、Ansible driver）として運用する為のAnsible Automation Platformを実行エンジンとしたシステム構成と環境構築について説明します。
| Ansible Core による構成を行う場合は、「 :doc:`./ansible_core` 」をご覧ください。
| 
| ITA Ansible driverを利用するにあたっては、ITA基本機能が構築済であることが前提です。
| ITA基本機能の構築に関しては、「 :doc:`../../manuals/it_automation_base/architecture` 」をご覧ください。

機能
========================================================
| Ansible driverは以下の機能を提供します。

.. table::  機能名
   :align: left

   +----+---------------------------------+--------------------------------+----------------------------------+-------------------------------------+
   | No | 機能名                          | 用途                           | WEBコンテンツ                    | Backyardコンテンツ                  |
   +====+=================================+================================+==================================+=====================================+
   | 1  | Ansible driver                  | ITAからAnsible Core かAnsible\ | 〇                               | 〇                                  |
   |    |                                 | Automation Platform\           |                                  |                                     |
   |    |                                 | を介してサーバ、ストレージ、\  |                                  |                                     |
   |    |                                 | ネットワーク機器の構成管理を\  |                                  |                                     |
   |    |                                 | 行う                           |                                  |                                     |
   +----+---------------------------------+--------------------------------+----------------------------------+-------------------------------------+
   | 2  | Ansible driver (Agent)          | Ansible Coreを外部から操作\    | 〇                               | －                                  |
   |    |                                 | を外部から操作するため\        |                                  |                                     |
   |    |                                 | のRestAPIを提供するコンテンツ  |                                  |                                     |
   |    |                                 |                                |                                  |                                     |
   +----+---------------------------------+--------------------------------+----------------------------------+-------------------------------------+

システム構成
========================================================

システム構成パターン
--------------------------------------------------------

| Ansible driver機能のシステム構成は、ITAシステムと同じです。
|
| Ansible driver（Agent）機能は、Apache、PHP、Ansible Core を構成する必要があります。専用サーバを用意する他、ITAシステムにコンソリデーションすることが可能です。
|
| Ansible Automation Controller は、Ansible実行における拡張された機能の利用や、可用性を高めた構成で運用することが可能です。ITAシステムおよびAnsible Coreとは個別の専用サーバを用意する必要があります。
| また実行するplaybookをAnsible Vaultで暗号化するため、Ansible Core [Ansible Driver (Agent)]が必要となります。 (Backyardサーバとコンソリデーションすることも可能)
|
| Ansible実行の対象機器の接続台数が多く、Ansible実行サーバのスケールアウトが必要な場合、Ansible Automation Controllerによる構成を推奨します。
| 以下に主なAnsible driver機能利用の構成パターンと構成イメージを記載します。
| ※ITAシステムは省略した構成図を記載します。





.. list-table:: システム構成パターン
   :widths: 5 50 80 25
   :header-rows: 1
   :align: left

   * - No
     - 構成
     - 説明
     - Ansibleスケールアウト可否
   * - 1
     - | Ansible Automation Platform (ハイブリッドパターン)
     - {{#1: 説明をお願いします。}}
     - 〇
   * - 2
     - Ansible Automation Platform (実行ノード分離パターン)
     - {{#2: 説明をお願いします。}}
     - 〇


|


Ansible Automation Controller (ハイブリッドパターン)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: /images/ja/diagram/aac.png
    :alt: Ansible Automation Platform 
    :width: 1200px

.. list-table:: システム通信要件
   :widths: 10 20 20 40 100
   :header-rows: 1
   :align: left

   * - | 通信番号
       | ※1 
     - FROM
     - TO
     - | プロトコル
       | [ポート番号　※2] 
     - 主な用途
   * - ①
     - ITAシステム
     - Hybrid ノード
     - | http(s)
       | [80(443)/tcp]
     - {{#4: 用途説明文の記載をお願いします。}}
   * - ②
     - ITAシステム
     - Hybrid ノード
     - ssh [22/tcp]
     - {{#5: 用途説明文の記載をお願いします。}}
   * - ③
     - ITAシステム
     - Git
     - | http(s)
       | [80(443)/tcp]
     - {{#6: 用途説明文の記載をお願いします。}}
   * - ④
     - Hybrid ノード
     - 対象機器
     - | Any
       | (ssh [22/tcp] telnet [23/tcp] 等 ※3）
     - 自動構成の対象機器へのコマンド実行
   * - ⑤
     - Hybrid ノード
     - Git
     - | http(s)
       | [80(443)/tcp]
     - {{#7: 用途説明文の記載をお願いします。}}

| ※1「2.1システム構成」の構成イメージに上記番号と紐づく通信番号を記載。
| ※2 ポート番号は標準的なポート番号を記載。
| ※3 代表的な例を記載。Ansibleモジュールにより利用プロトコルが異なる。


Ansible Automation Controller (実行ノード分離)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. figure:: /images/ja/diagram/aac_execution.png
    :alt: Ansible Automation Controller (実行ノード分離)
    :width: 1400px







.. list-table:: システム通信要件
   :widths: 10 20 20 40 100
   :header-rows: 1
   :align: left

   * - | 通信番号
       | ※1 
     - FROM
     - TO
     - | プロトコル
       | [ポート番号　※2] 
     - 主な用途
   * - ①
     - ITAシステム
     - Controle ノード
     - | http(s)
       | [80(443)/tcp]
     - {{#8: 用途説明文の記載をお願いします。}}
   * - ②
     - ITAシステム
     - Controle ノード
     - ssh [22/tcp]
     - {{#9: 用途説明文の記載をお願いします。}}
   * - ③
     - ITAシステム
     - Git
     - | http(s)
       | [80(443)/tcp]
     - {{#10: 用途説明文の記載をお願いします。}}
   * - ④
     - Controle ノード
     - 対象機器
     - | Any
       | (ssh [22/tcp] telnet [23/tcp] 等 ※3）
     - 自動構成の対象機器へのコマンド実行
   * - ⑤
     - Controle ノード
     - Git
     - | http(s)
       | [80(443)/tcp]
     - {{#11: 用途説明文の記載をお願いします。}}

| ※1「2.1システム構成」の構成イメージに上記番号と紐づく通信番号を記載。
| ※2 ポート番号は標準的なポート番号を記載。
| ※3 代表的な例を記載。Ansibleモジュールにより利用プロトコルが異なる。
  



システム要件
========================================================

| Ansible driver はITAシステムのシステム要件に準拠するため、「 :doc:`../../manuals/it_automation_base/architecture` 」を参照してください。
| ここではAnsible Automation Platformの要件を記載します。


.. table:: Ansible Automation Platformの動作確認済みバージョン
   :align: left

   +---------------------------------------+---------------------------------------------------------------------+
   |                                       | Ansible Automation Platform                                         |
   +=======================================+================+================+=================+=================+                       
   | |                                     | 2.0            | 2.1            | 2.2             | 2.3             |
   | | Exastro IT Automation  version: 2.0 +----------------+----------------+-----------------+-----------------+
   | |                                     |                | 〇             | 〇              |                 |
   +---------------------------------------+----------------+----------------+-----------------+-----------------+       

| 〇: 動作確認済み                   


Playbook連携
========================================================

| ITAとAnsible Automation Platform間のPlaybook連携について説明します。

.. figure:: /images/ja/diagram/playbook_link_between_aap_and_container.png
   :alt: ITAとAnsible Automation Platform2.xのPlaybook連携図
   :width: 750px


   ITAとAnsible Automatio Platform2.x間のPlaybook連携図


初期設定
========================================================
| Ansible Automation Platformインストール後、実行エンジンに応じて各設定を行ってください。
| 

.. list-table:: Ansible Core システム要件
   :widths: 45 55
   :header-rows: 1
   :align: left

   * - 設定項目
     - Ansible Automation Platform 2.x
   * - ITA作業用ディレクトリの準備
     - 〇
   * - ITA作業用ディレクトリの公開
     - 〇
   * - Ansible Automation Platformへのファイル転送ユーザーの準備 
     - 〇
   * - Ansible Automation Platformと連携するGitへのユーザーの準備
     - 〇
   * - Exastro IT Automation 連携 
     - 〇
   * - Proxy設定 
     - △


| 〇:必須　△:必要に応じて



ITA作業用ディレクトリの準備
-------------------------------------------------------------

| Ansible Automation PlatformサーバにITA作業用ディレクトリを作成してください。
| クラスタ構成の場合は、構成している全てのサーバにディレクトリを作成してください。
| ただし、Ansible Automation Platformのhop nodeにはディレクトリ作成不要です。
|

.. list-table:: ITA作業用ディレクトリの作成情報
   :widths: 35 120
   :header-rows: 1
   :align: left

   * - 項目
     - 設定値
   * - ディレクトリパス
     - /var/lib/exastro
   * - オーナー・グループ
     - awx:awx
   * - パーミッション
     - 0755


ITA作業用ディレクトリの公開
-------------------------------------------------------------

| ブラウザよりAnsible Automation Platformにログインし、「設定」→「ジョブ」→「分離されたジョブに公開するパス」に「/var/lib/exastro/」を設定します。
| 

.. figure:: /images/ja/diagram/publish_ita_operation_director.png
   :width: 5.92014in
   :height: 2.4375in


Ansible Automation Platformへのファイル転送ユーザーの準備
-------------------------------------------------------------

| ITAからAnsible Automation Platformのプロジェクトを生成する際、Ansible Automation PLatformの下記ディレクトリにPlaybook一式をファイル転送します。ファイル転送するLinuxユーザーを準備してください。
|
| ・SCM管理ディレクトリ(/var/lib/awx/projects)
| 　※Ansible Tower3.xの場合にLinuxユーザーでPlaybook一式をファイル転送します。
| ・ITA作業用ディレクトリ(/var/lib/exastro)

| Linuxユーザーは、Ansible Automation Pltformインストール時に生成されるawxユーザーにパスワードを設定し使用することを強く推奨します。また、awxユーザー以外のユーザーを用意し使用する場合、SCM管理パス(/var/lib/awx/projects)のパーミッションの変更はRedhatのサポート対象外となりますのでご注意ください。
| 準備したLinuxユーザーは、ITAシステムに登録する必要があります。「利用手順マニュアル_Ansible-driver」の「Ansible Automation Platformホスト一覧」を参照し、登録を行ってください。


Ansible Automation Platformと連携するGitへのユーザーの準備
--------------------------------------------------------------
| ITAからAnsible Automation Platformのプロジェクトを生成する際のSCMタイプをGitにしています。
| 連携先のGitリポジトリは、Ansible driverのバックヤード機能がインストールされているホストに作成されます。Ansible Automation Platformから、このGitリポジトリにssh鍵認証で接続するLinuxユーザーを準備してください。
|
| 準備したLinuxユーザーは、ITAシステムに登録する必要があります。「利用手順マニュアル_Ansible-driver」の「インターフェース情報」の「SCM管理 Git連携先情報」を参照し、登録を行ってください。
| 尚、ITAインストーラを使用してITAインストールまたはV1.10.0以降へのバージョンアップをした場合、Gitリポジトリに接続するLinuxユーザーと鍵ファイルを生成し、「インターフェース情報」の「SCM管理 Git連携先情報」の「ユーザー」、「ssh秘密鍵ファイル」を初期設定しているため、個別に作成は不要です。「ホスト名」にAnsible driverのバックヤード機能がインストールされているホスト名（またはIPアドレス）を設定してください。
|
| 別のユーザーを使用する場合は、Linuxユーザーと鍵ファイルを生成し「インターフェース情報」の「SCM管理 Git連携先情報」を更新してください。
| 

.. list-table:: ITAインストール時に生成されるssh鍵認証用Linuxユーザー情報
   :widths: 35 200
   :header-rows: 1
   :align: left

   * - **項目**
     - **項目値**
   * - ユーザー
     - awx
   * - パスワード
     - 未設定
   * - 秘密鍵
     - /home/awx/.ssh/rsa_awx_key
   * - 公開鍵
     - /home/awx/.ssh/rsa_awx_key.pub



Exastro IT Automation 連携
--------------------------

| Ansible Automation Platformに認証アプリケーションをあらかじめ登録しておく必要があります。

organization追加時の作業
^^^^^^^^^^^^^^^^^^^^^^^^^

.. _make_organization:

1. 組織作成
************************************

| organization用の組織を作成します。
| Ansible Automation Platform は admin(管理ユーザー)でログインしてください。


#. | :menuselection:`アクセス --> 組織` の :guilabel:`追加` ボタンをクリックします。
#. | 該当項目を入力し、 :guilabel:`保存` ボタンをクリックしてください。
   | 必須項目及び設定値については下記の表を参照してください。

.. list-table:: 
   :widths: 35 80 80
   :header-rows: 1
   :align: left

   * - 項目
     - 設定値
     - 備考
   * - 名前
     - 任意の名称
     - 
   * - インスタンスグループ
     - ※未選択のままにする
     - 「:ref:`connection_instance` 」で設定


.. _make_application:

2. アプリケーション登録
**************************************************

| 接続トークン払出用のアプリケーション登録をします。
| Ansible Automation Platform は admin(管理ユーザー)でログインしてください。
 
#. | :menuselection:`管理 --> アプリケーション` の :guilabel:`追加` ボタンをクリックしてください。
#. | 該当項目を入力し、 :guilabel:`保存` ボタンをクリックしてください。
   | 必須項目及び設定値については下記の表を参照してください。

.. list-table:: 
   :widths: 35 80 80
   :header-rows: 1
   :align: left

   * - 項目
     - 設定値
     - 備考
   * - 名前
     - 任意の名称
     - 「 :ref:`output_token` 」で使用する
   * - 組織
     - 「 :ref:`make_organization` 」で作成した組織を選択する
     - 
   * - 認証付与タイプ
     - リソース所有者のパスワードベースを選択
     - 
   * - クライアントタイプ
     - 秘密
     - 


.. _architecture_user:

3. ユーザー作成
**************************************************

| organization用のユーザーを作成します。
| Ansible Automation Platform は admin(管理ユーザー)でログインしてください。
 
#. | :menuselection:`アクセス --> ユーザー` の :guilabel:`追加` ボタンをクリックしてください。
#. | 該当項目を入力し、 :guilabel:`保存` ボタンをクリックしてください。
   | 必須項目及び設定値については下記の表を参照してください。


.. list-table:: 
   :widths: 35 80 80
   :header-rows: 1
   :align: left

   * - 項目
     - 設定値
     - 備考
   * - ユーザー名
     - 任意のユーザー名
     - 
   * - パスワード
     - 任意のパスワード
     - 
   * - パスワードの確認
     - 任意のパスワード
     - 
   * - ユーザータイプ
     - 標準ユーザーを選択
     - 
   * - 組織
     - 「 :ref:`make_organization` 」で作成した組織を選択する
     - 

.. _organization_roles:

4. ロール設定
**************************************************

| organization用ユーザーに紐づける組織に対してロールを設定します。
| Ansible Automation Platform は admin(管理ユーザー)でログインしてください。
 
#. | :menuselection:`アクセス --> ユーザー` より「 :ref:`architecture_user` 」で作成したユーザー名をクリックしてください。
#. | ユーザーの詳細画面に遷移されるため、:menuselection:`ロール` タブを選択し、:guilabel:`追加` ボタンをクリックしてください。
#. | 下記表の通りにユーザー権限の追加をしてください。

   #. | リソースタイプの追加 では 「組織」 を選択し、:guilabel:`Next` ボタンをクリックしてください。
   #. | リストの項目の選択 では 「:ref:`make_organization` 」 で作成した組織 を選択し、:guilabel:`Next` ボタンをクリックしてください。
   #. | 適用するロールの選択 では 「管理者」と「メンバー」の２つのロールを選択し、:guilabel:`保存` ボタンをクリックしてください
      | ※ロール内容に設定した該当組織のロールが表示されていることを確認してください。

.. list-table:: 
   :widths: 70 60 60
   :header-rows: 1
   :align: left

   * - ユーザー権限の追加手順
     - 設定値
     - 備考
   * - ① リソースタイプの追加
     - 「組織」を選択
     - 
   * - ② リストの項目の選択
     - 「 :ref:`make_organization` 」で作成した組織を選択する
     - ※「 :ref:`make_organization` 」で作成した組織以外のロールは付与しないで下さい。
   * - ③ 適用するロールの選択
     - 「管理者」と「メンバー」の２つのロールを選択
     - 



.. _output_token:

5. 認証トークン払出
**************************************************

| Ansible Automation Platform は「 :ref:`architecture_user` 」で作成したユーザーでログインしてください。

#. | :menuselection:`アクセス --> ユーザー` の :guilabel:`追加` ボタンを押下する。
#. | 該当項目を入力し、 :guilabel:`保存` ボタンを押下する。
   | 必須項目及び設定値については下記の表を参照してください。

.. list-table:: 
   :widths: 35 50 30
   :header-rows: 1
   :align: left

   * - 項目
     - 設定値
     - 備考
   * - アプリケーション
     - 「 :ref:`make_application` 」で作成したアプリケーションを選択
     - 
   * - 範囲
     - 書き込みを選択
     - 

workspace追加時の作業
^^^^^^^^^^^^^^^^^^^^^^^^^

.. _ansible_execution_environment:

1. Ansible Execution Environment を組み込む
*************************************************************


2. インスタンスグループ作成
*********************************************************************************

| ※先ほど組み込んだ Ansible Execution Environment を追加するインスタンスグループが既にある場合、次の 「:ref:`_add_insetance`」の手順に進んでください。

| Ansible Automation Platform は admin(管理ユーザー)でログインしてください。

#. | :menuselection:`管理 --> インスタンスグループ` の 「 :ref:`ansible_execution_environment` 」で組み込んだ Ansible Execution Environment を追加するインスタンスグループ選択してください。
#. | 該当項目を入力し、 :guilabel:`保存` ボタンを押下する。
   | 必須項目及び設定値については下記の表を参照してください。

.. list-table:: 
   :widths: 35 30 50
   :header-rows: 1
   :align: left

   * - 項目
     - 設定値
     - 備考
   * - 名前
     - 任意の名称
     - 命名規則については下記をご参照ください

.. _add_insetance:

3. インスタンスグループにインスタンス( Ansible Execution Environment )を追加
**********************************************************************************************

| インスタンスグループに「 :ref:`ansible_execution_environment` 」で組み込んだ Ansible Execution Environment を追加します。
| Ansible Automation Platform は admin(管理ユーザー)でログインしてください。

#. | :menuselection:`管理 --> インスタンスグループ` より、「 :ref:`ansible_execution_environment` 」で組み込んだ Ansible Execution Environment を追加するインスタンスグループ名をクリックしてください。
#. | インスタンスグループの詳細画面に遷移されるため、:menuselection:`インスタンス` タブを選択し、:guilabel:`関連付け` ボタンをクリックしてください。
#. | インスタンスの選択の画面に遷移され、「 :ref:`ansible_execution_environment` 」で組み込んだインスタンス( Ansible Execution Environment )が表示されるので選択し、:guilabel:`保存` ボタンをクリックしてください。

.. _connection_instance:

4. 組織とインスタンスグループの紐づけ
**********************************************************************************************

| 「 :ref:`make_organization` 」で作成した組織と上記で使用したインスタンスグループを紐づけます。
| Ansible Automation Platform は admin(管理ユーザー)でログインしてください。

#. | :menuselection:`アクセス --> 組織` より、「 :ref:`make_organization` 」で作成した組織名をクリックしてください。
#. | 詳細画面に遷移されるため、:guilabel:`編集` ボタンをクリックしてください。
#. | 詳細の編集の画面に遷移されるため、インスタンスグループに上記で使用したインスタンスグループを選択し、:guilabel:`保存` ボタンをクリックしてください。
   | ※複数選択可能

5. Exastro IT Automation に認証トークンと組織を登録
***********************************************************************************************************************************************************************

| 「利用手順マニュアル_Ansible-driver」の「 :ref:`general_operations_interface_information` 」を参照し、:menuselection:`Ansible共通 --> インターフェース情報` に「 :ref:`output_token` 」で作成した認証トークンと「 :ref:`make_organization` 」で作成した組織の登録を行ってください。


.. warning:: | 組織名を登録する際は、認証トークンを登録してから1分程度経過後(※)、インターフェース情報を再表示し、「 :ref:`make_organization` 」で作成した組織名を選択してください。
  ※ バックヤードで各認証トークンに対応したユーザーに紐づいている組織を収集し、プルダウンに表示しているため。


.. note:: | 「 :ref:`organization_roles` 」で作成したユーザーに複数の組織のロールを付与されていた場合、ランダムに選択された組織をデフォルト値とします。

..
  .. list-table:: Ansible Automation Platform 必要リソース一覧
   :widths: 20 30 30 50
   :header-rows: 1
   :align: left

   * - **種類**
     - **用途**
     - **名前**
     - **説明**
   * - アプリケーション
     - 認証アプリケーション
     - o_auth2_access_token
     - ITAからAnsible Automation PlatformにRestAPIで接続する場合の認証用のアプリケーション情報
   * - ユーザー
     - トークン
     - ー
     - ITAからAnsible Automation PlatformにRestAPIで接続するのに使用する接続トークン


  アプリケーション 
  ~~~~~~~~~~~~~~~~~~~~~~
  | ● Ansible Automation Platform設定値
  -  名前 　　　　　　　： o_auth2_access_token
  -  組織 　　　　　　　： Default
  -  認証付与タイプ 　　： リソース所有者のパスワードベース
  -  クライアントタイプ ： 機密
  
  ユーザートークン
  ~~~~~~~~~~~~~~~~~~~~~
  | ● Ansible Automation Platform設定値
  -  APPLICATION 　　　： o_auth2_access_token
  -  SCOPE 　　　　　　： 書き込み
  
  .. warning:: 
    | Ansible Automation Platformのログインに使用するユーザーでログインしておく必要があります。
    | 生成されたトークンは、Ansible共通コンソールのインタフェース情報の接続トークンに設定する必要があります。「利用手順マニュアル_Ansible-driver」の「 :ref:`general_operations_interface_information` 」を参照し、登録を行ってください。

Proxyの設定
-------------

| Ansible Automation Platformの設定に応じて作業実行時などにRedhat社の所定のサイトより実行環境のコンテナイメージのダウンロードが行われます。
| ブラウザよりAnsible Automation Platformにログインし、「設定」→「ジョブ」→「追加の環境変数」に下記の環境変数を設定します。

- | https_proxy
- | http_proxy
- | no_proxy
- | HTTPS_PROXY
- | HTTP_PROXY
- | NO_PROXY

|

.. figure:: /images/ja/diagram/proxy_settings.png
   :width: 6.09896in
   :height: 2.68264in


|
|

.. warning::
  | Ansible Automation PlatformがProxy環境下にある場合、Ansible Automation PlatformにProxy設定が必要です。Proxyの設定がされていない状態で作業実行を行った場合、エラー原因が取得できない場合があります。

..
  パッケージ確認
  ----------------
  
  | Ansible-driverで必要なパッケージがインストールされているかを確認します。
  | インストールされていない場合は、パッケージのインストールが必要です。
  
  | ●必要なパッケージ
  | 　pexpect
  |
  | ●確認方法
  
  .. code-block:: none
  
    su - awx
    source /var/lib/awx/venv/ansible/bin/activate
    pip list
    deactivate
  
  | ●インストール方法
  
  .. code-block:: none
  
    su - awx
    source /var/lib/awx/venv/ansible/bin/activate
    umask 0022
    pip install --upgrade pexpect
    deactivate
