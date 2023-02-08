========
ログ一覧
========


| Exastro IT Automationの各コンテナの出力メッセージに関するログについて。
| 各コンテナのログの例と文字列の意味を以下に記載します。



platform-job
============

.. code-block:: 

    形式：%(asctime)s %(levelname)s (%(userid)s) %(pathname)s(%(lineno)d) %(message)s
    例　：2023/01/11 11:27:05.976995 INFO (None) /app/platform_init.py(88) platform initialize setting start


.. list-table:: 
   :widths: 10 15 20 20
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
     -  メッセージレベル
     -  INFO
     -  DEBUG, INFO, WARNING, ERROR, CRITICAL が出力される。
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



platform-api
============

.. code-block:: 

    形式：%(asctime)s %(levelname)s (%(userid)s) %(pathname)s(%(lineno)d) %(message)s
    例　：2023/01/06 10:15:47.537174 INFO (2d6aabce-04c7-4938-a616-fa283cd6693) /app/common_library/common/api_keycloak_roles.py(372) Get keycloak user list for each role. realm_name=org3, client_id=743c50ae-7656-40d2-9ac1-b6cc6e39d15c, role_name=_workspace-1-admin



.. list-table:: 
   :widths: 10 15 20 20
   :header-rows: 1
   :align: left

   * - | フォーマット文字列
     - | フォーマットの意味
     - | ログの例
     - | 備考
   * - | %\(asctime\)s
     - | ログ出力日時
     - | 2023/01/06 10:15:47.537174
     - |
   * - | %\(levelname\)s
     - | メッセージレベル
     - | INFO
     - | DEBUG, INFO, WARNING, ERROR, CRITICAL が出力される。
   * - | \(%\(userid\)s\)
     - | アクセスユーザー（Noneは指定なし）
     - | \(2d6aabce-04c7-4938-a616-fa283cd6693\)
     - |
   * - | %\(pathname\)s
     - | ログ出力元のソース
     - | /app/common_library/common/api_keycloak_roles.py
     - |
   * - | \(%\(lineno\)d\)
     - | ログ出力元の行
     - | \(372\)
     - |
   * - | %\(message\)s
     - | メッセージ
     - | platform initialize setting start
     - |



platform-web
============

| デフォルトのapacheログ形式になっています。
| 設定内容は以下の通りです。

.. code-block::

    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined \
    LogFormat "%h %l %u %t \"%r\" %>s %b" common \ 
    <IfModule logio_module> \
      # You need to enable mod_logio.c to use %I and %O \
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio \
    </IfModule>

| commonフォーマットは以下の構成になっています。
| ログの保存としてcommonを指定した場合は表の情報が一行で記録されていきます。

.. code-block::
   
   例：192.168.128.2 - - [12/Jan/2023:15:38:10 +0900] "GET /favicon.ico/platform/ HTTP/1.1" 200 9817
   "http://localhost:8000/org3/platform/roles" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
   (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36"

.. list-table:: commonでログ保存を指定した場合
   :widths: 10 15 20 20
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
   :widths: 10 15 20 20
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
     - | User Agent とは、使用しているOS・ブラウザなどの情報のことです。

| combinedioフォーマットは、combinedフォーマットに以下の項目が追加されています。

.. list-table:: combinedioフォーマットでログ保存を指定した場合
   :widths: 10 15 20 20
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


keycloak
========

.. code-block:: 
   
   形式：%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p [%c] (%t) %s%e%n
   例　：2023-01-12 09:21:49,040 INFO  [org.keycloak.events] (default task-13) type=INTROSPECT_TOKEN, realmId=org3, clientId=system-org3-auth, userId=null, ipAddress=172.18.0.14, client_auth_method=client-secret


.. list-table:: 
   :widths: 10 15 20 20
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
     - | エラーレベル（DEBUG, INFO, WARN, ERROR）
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
     - |  
     - |
   * - | %e
     - | 例外
     - |
     - |
   * - | %n
     - | 改行
     - |
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