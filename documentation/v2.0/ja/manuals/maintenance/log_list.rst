========
ログ一覧
========

| Exastro IT Automationの各コンテナの出力メッセージに関するログについて。
| 各コンテナのログの例と文字列の意味を以下に記載します。

一般的なログ
============

| 一般的なログは以下の形式となっています。

.. code-block::
   :caption: 形式

   [%(asctime)s] [%(levelname)s] [%(userid)s] <フリーログ>

.. code-block::
   :caption: 例

    [2023-02-06 14:18:05,212][INFO] [USER_ID:20401] AppLog instance(stdAppLogger) is created

.. list-table:: 
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | [%(asctime)s]
     - | 日付
     - | [2023-01-05 18:05:22,875]
     - |
   * - | [%(levelname)s]
     - | ログレベル
     - | [INFO]
     - | 
   * - | [%(userid)s]
     - | ユーザーID
     - | [USER_ID:efb59f05-6f31-47d6-b28e-0f9ee236534e]
     - |
   * - | <フリーログ>
     - | 決まった値はなし
     - | AppLog instance(stdAppLogger) is created
     - | ログによって形式が異なります。

platform-job・platform-api
==========================

.. code-block:: 
   :caption: 形式
    
    %(asctime)s %(levelname)s (%(userid)s) %(pathname)s(%(lineno)d) %(message)s
    
.. code-block:: 
   :caption: 例

   2023/01/11 11:27:05.976995 INFO (None) /app/platform_init.py(88) platform initialize setting start


.. list-table:: 
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * -  フォーマット文字列
     -  フォーマットの意味
     -  ログの例
     -  備考
   * -  %\(asctime\)s
     -  ログ出力日時
     -  2023/01/11 11:27:05.976995
     -    
   * -  %\(levelname\)s
     -  ログレベル
     -  INFO
     -  DEBUG, INFO, WARNING, ERROR, CRITICAL のいずれかが出力される。
   * -  \(%\(userid\)s\)
     -  アクセスユーザー（Noneは指定なし）
     -  （None）
     -    
   * -  %\(pathname\)s
     -  ログ出力元のソース
     -  /app/platform_init.py
     -    
   * -  \(%\(lineno\)d\)
     -  ログ出力元の行
     -  \(88\)
     -   
   * -  %\(message\)s
     -  メッセージ
     -  platform initialize setting start
     -  

platform-web・ita-web-server
============================

| デフォルトのapacheログ形式になっています。
| 設定内容は以下の通りです。

.. code-block::
   :caption: 形式

    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined \
    LogFormat "%h %l %u %t \"%r\" %>s %b" common \ 
    <IfModule logio_module> \
      # You need to enable mod_logio.c to use %I and %O \
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio \
    </IfModule>


| commonフォーマットは以下の構成になっています。
| ログの保存としてcommonを指定した場合は表の情報が一行で記録されていきます。

.. code-block::
   :caption: 例

   192.168.128.2 - - [12/Jan/2023:15:38:10 +0900] "GET /favicon.ico/platform/ HTTP/1.1" 200 9817
   "http://localhost:8000/org3/platform/roles" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
   (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36"


.. list-table:: commonでログ保存を指定した場合
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | %h
     - | アクセス元のホスト名
     - | 192.168.128.2
     - |
   * - | %l
     - | クライアントの識別子
     - | -
     - |
   * - | &u
     - | 認証ユーザ名
     - | - 
     - |
   * - | %t
     - | リクエストを受け付けた時刻
     - | [12/Jan/2023:15:38:10 +0900]
     - |
   * - | \%r\
     - | リクエストの最初の行
     - | "GET /favicon.ico/platform/ HTTP/1.1"
     - |
   * - | %>s
     - | 最後のレスポンスのステータス
     - | 200
     - |
   * - | %b
     - | 送信されたバイト数
     - | 9817
     - |
  
| combinedフォーマットは、commonフォーマットに以下の項目が追加されています。

.. list-table:: combinedフォーマットでログ保存を指定した場合
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | \%{Referer}i\
     - | リファラー
     - | "http://localhost:8000/org3/platform/roles"
     - | リファラーとは参照元ページのことです。
   * - | \%{User-Agent}i\
     - | User Agent
     - | "Mozilla/5.0 \(Windows NT 10.0; Win64; x64\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/108.0.0.0 Safari/537.36"
     - | User Agent とは使用しているOS・ブラウザなどの情報のことです。

| combinedioフォーマットは、combinedフォーマットに以下の項目が追加されています。

.. list-table:: combinedioフォーマットでログ保存を指定した場合
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | &I
     - | 受け取ったバイト数
     - | - 
     - |
   * - | %O
     - | 送信したバイト
     - | - 
     - |

platform-auth
=============

| authログは、上述のapacheログとapiログの混合されたものが出力されますが、
| apacheログ（platform-web）とほとんど同じです。

.. code-block::
   :caption: 例

   [-] - 10.244.0.1 - - [08/Feb/2023:10:22:20 +0900] "GET /auth/resources/b3h1e/common/keycloak/node_modules/patternfly/dist/fonts/OpenSans-Light-webfont.woff2 HTTP/1.1" 200 63180 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"

keycloak
========

.. code-block:: 
   :caption: 形式

   %d{yyyy-MM-dd HH:mm:ss,SSS} %-5p [%c] (%t) %s%e%n

.. code-block:: 
   :caption: 例

   2023-01-12 09:21:49,040 INFO  [org.keycloak.events] (default task-13) type=INTROSPECT_TOKEN, realmId=org3, clientId=system-org3-auth, userId=null, ipAddress=172.18.0.14, client_auth_method=client-secret

.. list-table:: 
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | %d{yyyy-MM-dd HH:mm:ss,SSS}
     - | ログ出力日時 
     - | 2023-01-12 09:21:49,040
     - |
   * - | %-5p
     - | ログレベル
     - | INFO
     - |
   * - | \[%c\]
     - | ログ　カテゴリ名
     - | \[org.keycloak.events\]
     - |
   * - | \(%t\)
     - | スレッド名
     - | \(default task-13\)
     - |
   * - | %s
     - | 簡単なメッセージ
     - | - 
     - |
   * - | %e
     - | 例外
     - | -
     - |
   * - | %n
     - | 改行
     - | -
     - | 

| keycloakのログの詳細については以下のURLをご参照ください。
| https://www.keycloak.org/server/logging

platform-db
===========

| DBについては使用するデータベースのログフォーマットとなります。
| 設定もDBによりますが、デフォルトでmariadbを立ち上げた際はエラーログが出力されます。
| 詳細は下記URLをご参照ください。
| https://mariadb.com/kb/en/error-log/

ita-api-organization
====================

.. code-block:: 
  :caption: 形式 
   
   [%(asctime)s] [%(levelname)s]  <フリーログ>

.. code-block::
  :caption: 例   
   
   [2023-01-19 12:18:25,940][INFO] AppLog instance(stdAppLogger) is created

.. list-table:: 
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | %(asctime)s
     - | 日付
     - | [2023-01-19 12:18:25,940]
     - |
   * - | %(levelname)s
     - | ログレベル
     - | INFO
     - | ERROR, INFO, DEBUG が基本的なログレベル
   * - | <フリーログ>
     - | 決まった値はなし
     - | AppLog instance(stdAppLogger) is created
     - | ログによって形式が異なります。

ita-api-admin
=============

.. code-block:: 
  :caption: 形式

  [%(asctime)s] [%(levelname)s] [%(userid)s] <フリーログ>

.. code-block:: 
  :caption: 例
  
  [2023-01-05 18:05:22,875][INFO] [USER_ID:efb59f05-6f31-47d6-b28e-0f9ee236534e] [ts=2023-01-05T09:05:22.756Z][api-start]url: POST:http://ita-api-admin:8070/api/organizations/org1/ita/

.. list-table:: 
   :widths: 15 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | [%(asctime)s]
     - | 日付
     - | [2023-01-05 18:05:22,875]
     - |
   * - | [%(levelname)s]
     - | ログレベル
     - | [INFO]
     - | 
   * - | [%(userid)s]
     - | ユーザーID
     - | [USER_ID:efb59f05-6f31-47d6-b28e-0f9ee236534e]
     - |
   * - | <フリーログ>
     - | 決まった値はなし
     - | [ts=2023-01-05T09:05:22.756Z][api-start]url: POST:http://ita-api-admin:8070/api/organizations/org1/ita/
     - | ログによって形式が異なります。
