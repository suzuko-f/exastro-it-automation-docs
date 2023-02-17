==============================================
システム構成/環境構築ガイド Terraform-driver編
==============================================

はじめに
========

| 本書では、ITA で Terraform オプション機能（以下、Terraform driver）として運用する為のシステム構成と環境構築について説明します。

| ITA Terraform driver を利用するにあたっては、ITA 基本機能が構築済であることが前提です。ITA 基本機能の構築に関しては、「環境構築ガイド（基本編）」をご覧ください。

機能
====

| Terraform driver は以下の機能を提供します。

.. list-table:: 表 1-1-1. 機能名
    :header-rows: 1
    :align: center

    * - No
      - 機能名
      - 用途
      - WEB/AP
      - BackYard
    * - 1
      - | Terraform driver
      - | ITAから Terraform を介して、コード化したインフラストラクチャ構成について、実行計画を生成したうえで構成を実行します。
        | また、Policy as Code によるアクセスポリシーをコード化して管理することが可能です。
      - | ・
      - | ・

システム構成
============

システム構成
------------

| Terraform driver のシステム構成は、ITA システムと同じです。

| Terraform の利用については、ITA サーバとは別に Terraform Enterprise サーバか、クラウドサービスである Terraform Cloud を利用する必要があります。

| Terraform の自動構成の実行対象がクラウド上から直接通信できないオンプレミス上にある場合は、Terraform Cloud Agents を利用して投入する構成が考えられます。

| ※以下の ITA システムは省略した構成図を記載しています。
| 詳細は「システム構成／環境構築ガイド_基本編」を参照してください。

.. figure:: /images/ja/terraform-driver/common/system_configuration.png
  :width: 800px
  :align: center

システムの通信要件
------------------

| 本システム構成において、各サービス間の通信要件は以下の通りです。

| ITA システム本体の通信要件の詳細は「システム構成／環境構築ガイド_基本編」を参照してください。

.. list-table:: 表 2.2 通信要件一覧
    :header-rows: 1
    :align: center

    * - | 通信番号 
        | ※1
      - FROM
      - TO
      - | プロトコル
        | [ポート番号 ※2]
      - 主な用途
    * - ①
      - | ITA サーバ
        | (Web/AP 機能)
      - | Terraform
        | ※3
      - | https
        | [443/tcp]
      - | ・ITA 画面での Organization/Workspace を Terraform 側への登録。
        | ・ITA 画面での Organization/Workspace/Policy/PolicySet の情報取得
    * - ②
      - | ITA サーバ
        | (Backyard 機能)
      - | Terraform
        | ※3
      - | https
        | [443/tcp] 
      - 作業実行時の Terraform Enterprise への Plan/PolicyCheck/Apply の実行および結果の取得
    * - ③
      - | Terraform
        | ※3
      - 対象機器 
      - | Any
        | (利用する Terraform プロバイダにより異なる）
      - Terraform から対象機器／クラウドサービスへのAPI 投入やコマンド実行
    * - ④-1
      - Terraform Cloud Agents
      - | Terraform
        | ※3
      - | https
        | [443/tcp] 
      - Terraform Cloud Agent から Terraform 本体へ API 通信を行う。
    * - ④-2
      - Terraform Cloud Agents
      - 対象機器
      - | Any
        | (利用する Terraform プロバイダにより異なる）
      - Terraform から対象機器／クラウドサービスへの API 投入やコマンド実行

| ※1「システム構成」の構成イメージに上記番号と紐づく通信番号を記載。
| ※2 ポート番号は標準的なポート番号を記載。
| ※3 本記載の Terraform は、Terraform Enterprise および Terraform Cloud を指します。

システム要件
============

| Terraform driver（ITA 側） は ITA システムのシステム要件に準拠するため、「システム構成／環境構築ガイド_基本編」を参照してください。ここでは BackYard の必要要件を記載します。

- バージョン

  .. list-table:: 表 3-1. Terraform Enterprise システム要件
    :widths: 30 50 30
    :header-rows: 1
    :align: center

    * - パッケージ
      - バージョン
      - 注意事項
    * - Terraform Enterprise
      - 202110-1(576)以上
      - 

  .. list-table:: 表 3-2.Terraform Cloud システム要件
    :widths: 30 30 50
    :header-rows: 1
    :align: center

    * - パッケージ
      - バージョン
      - 注意事項
    * - Terraform Cloud
      - 
      - Terraform Cloud は常に最新の状態です。

- BackYard

  .. list-table:: 表 3-3. Terraform BackYard 必要 Linux コマンド
    :header-rows: 1
    :align: center

    * - コマンド
      - 注意事項
    * - tar
      - 
    * - zip
      - 

- Python ライブラリ
  
  .. list-table:: 表 3-4. Terraform driver システム要件
    :widths: 20 20 80
    :header-rows: 1
    :align: center

    * - 名称
      - 必要要件
      - 注意事項
    * - python-hcl2
      - Python3.6 以上
      - WebAP サーバと BackYard サーバに必要です。

Terraform 初期設定
==================

必要リソース準備
----------------

| Terraform Driver から Terraform に連携するために、Terraform からユーザトークンを発行する必要があります。

| ブラウザより Terraform にログインし、[User Settings] →[Tokens]→[Creat an API token]の順に押下することで発行されます。

.. figure:: /images/ja/terraform-driver/common/user_token_issuance_method.png
  :width: 800px
  :align: center

| 発行されたユーザトークンは ITA の Terraform コンソール「インターフェース情報」より設定します。
| 詳細は「利用手順マニュアル Terraform-driver」をご参照ください。