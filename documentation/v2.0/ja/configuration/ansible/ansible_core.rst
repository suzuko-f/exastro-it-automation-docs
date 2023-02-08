========================================================
Ansible Core
========================================================

はじめに
========================================================
| 本書では、Exastro IT Automation（以下、ITAとも記載する）でAnsibleオプション機能（以下、Ansible driver）として運用する為のAnsible Coreを実行エンジンとしたシステム構成と環境構築について説明します。
| Ansible Automation Platform による構成を行う場合は、「 :doc:`./ansible_automation_platform` 」をご覧ください。
|
| {{#1: 下記内容について基本機能はどこでマージするのでしょうか}}
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
   |    |                                 | Automation Controller\         |                                  |                                     |
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

| Ansible driver機能のシステム構成は、ITAシステムと同じです。
| Ansible driver（Agent）機能は、Apache、PHP、Ansible Core を構成する必要があります。専用サーバを用意する他、ITAシステムにコンソリデーションすることが可能です。
|
| Ansible実行の対象機器の接続台数が多く、Ansible実行サーバのスケールアウトが必要な場合は、Ansible Automation Controllerによる構成を推奨します。
|
| 以下にAnsible Coreにおける構成パターンと構成イメージを記載します。

..
  | Ansible driver機能のシステム構成は、ITAシステムと同じです。
  |
  | Ansible driver（Agent）機能は、Apache、PHP、Ansible Core と構成する必要があります、専用サーバを用意するか、ITAシステムにコンソリデーションすることが可能です。
  | Ansible Automation Platformにより、Ansible実行における拡張された機能の利用や、可用性を高めた構成で運用することが可能です。
  |
  | ITAシステムおよびAnsible Coreとは個別の専用サーバを用意する必要があります。
  | また実行するplaybookをAnsible Vaultで暗号化するため、Ansible Core [Ansible Driver (Agent)]が必要となります。 (Backyardサーバとコンソリデーションすることも可能)
  |
  | Ansible実行の対象機器の接続台数が多く、Ansible実行サーバのスケールアウトが必要な場合、Ansible Automation Platformによる構成を推奨します。
  | 以下に主なAnsible driver機能利用の構成パターンと構成イメージを記載します。
  | ※ITAシステムは省略した構成図を記載します。



システム構成パターン
--------------------------------------------------------

.. list-table:: システム構成パターン
   :widths: 50 80 25
   :header-rows: 1
   :align: left

   * - 構成
     - 説明
     - Ansibleスケールアウト可否
   * - Ansible Core
     - {{#2: 説明をお願いします。}}
     - ×


システム構成イメージ
--------------------------------------------------------

.. figure:: /images/ja/diagram/ansible_core.png
    :alt: Ansible Core
    :width: 800px



.. list-table:: システム通信要件
   :widths: 10 20 20 40 80
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
     - Ansible Core
     - ストレージ機器
     - | ファイルアクセス
       | (tcp or ストレージ I/O)
     - | Ansible コマンド実行時の実行情報
       | (Playbook host_vars 等)の参照
   * - ②
     - Ansible Core
     - 対象機器
     - | Any
       | (ssh [22/tcp] telnet [23/tcp] 等 ※3）
     - 自動構成の対象機器へのコマンド実行

| ※1「2.1システム構成」の構成イメージに上記番号と紐づく通信番号を記載。
| ※2 ポート番号は標準的なポート番号を記載。
| ※3 代表的な例を記載。Ansibleモジュールにより利用プロトコルが異なる。


システム要件
========================================================

| Ansible driver はITAシステムのシステム要件に準拠するため、「 :doc:`../../manuals/it_automation_base/architecture` 」を参照してください。
| ここではAnsible Coreの要件を記載します。


Ansible Core
-------------------------------------------------------------

.. list-table:: Ansible Core システム要件
   :widths: 35 20 45 
   :header-rows: 1
   :align: left

   * - パッケージ
     - バージョン
     - 注意事項
   * - Ansible
     - 2.5 以上
     - 
   * - Python
     - 3.0 以上
     - 
   * - pywinrm
     - 
     - Python モジュールです。Yum でインストールできない場合、pip を使用してインストールしてください。
   * - Pexpect
     - 
     - Python モジュールです。
   * - talnet
     - ー
     - 構成対象に telnet 接続する場合に必要です。
   * - Apache
     - 2.4 系
     - | ITA システムと異なるサーバで運用の場合に必要です。
       | パッケージ/バージョンは ITA システムサーバに合わせてください。


.. list-table:: Ansible Driver必要Linuxコマンド
   :widths: 45 120 
   :header-rows: 1
   :align: left

   * - **コマンド**
     - **注意事項**
   * - expect
     - 




Playbook連携
========================================================

| ITAとAnsible Core 間のPlaybook連携について説明します。

.. figure:: /images/ja/diagram/playbook_link_containers.png
   :alt: ITAとAnsible Core間のPlaybook連携図
   :width: 750px


   ITAとAnsible Core間のPlaybook連携図



初期設定
========================================================
| Ansible Core インストール後、ITAとAnsible Coreからアクセス可能な共有ディレクトリを準備してください。
| インストールマニュアルの「 :ref:`persistent_volume` 」で作成したボリューム内に共有ディレクトリを作成します。

