付録
====

Conductor通知先定義
-------------------

Conductor通知先定義設定例
~~~~~~~~~~~~~~~~~~~~~~~~
.. table:: Teams設定例
   :align: left

   +-----------------------+--------------------------------------------------+
   | 通知名称              | 通知サンプル                                     |
   +=======================+==================================================+
   | 通\                   |  通知先のTeamsのWebhook URLを入力して下さい。    |
   | 知先(CURLOPT_URL)     |                                                  |
   +-----------------------+--------------------------------------------------+
   | ヘッダー              |  [ "Content-Type: application/json" ]            |
   | (CURLOPT_HTTPHEADER)  |                                                  |
   +-----------------------+--------------------------------------------------+
   | メッセージ(C\         | {"text": "通知名：__NOTICE_NAME__, <br>          |
   | URLOPT_POSTFIELDS)    | Conductor名称: \__CONDUCTOR_NAME__, <br>         |
   |                       | Con                                              |
   |                       | ductorインスタンスID:__CONDUCTOR_INSTANCE_ID__,  |
   |                       | <br> オペレーションID: \__OPERATION_ID__,        |
   |                       | <br>オペレーション名:__OPERATION_NAME__,         |
   |                       | <br>ステータスID: \__STATUS_ID__,                |
   |                       | <br>ステータス: \__STATUS_NAME__,                |
   |                       | <br>実行ユーザー: \__EXECUTION_USER__, <br>      |
   |                       | 予約日時: \__TIME_BOOK__, <br>開始日時:          |
   |                       | \__TIME_START__, <br>終了日時: \__TIME_END__,    |
   |                       | <br>緊急停止フラグ: \__ABORT_FLAG__, <br>        |
   |                       | 作業URL: \__JUMP_URL__, <br> "}                  |
   +-----------------------+--------------------------------------------------+
   | PROXY / URL           | http://proxy.co.jp                               |
   | (CURLOPT_PROXY)       |                                                  |
   +-----------------------+--------------------------------------------------+
   | PROXY / PORT          | 8080                                             |
   | (\                    |                                                  |
   | CURLOPT_PROXYPORT)    |                                                  |
   +-----------------------+--------------------------------------------------+
   | 作業確認URL(FQDN)     | http://exastro-it-automation.local               |
   +-----------------------+--------------------------------------------------+
   | その他                |                                                  |
   +-----------------------+--------------------------------------------------+
   | 開始日時              |                                                  |
   +-----------------------+--------------------------------------------------+
   | 終了日時              |                                                  |
   +-----------------------+--------------------------------------------------+
|

.. figure:: ./conductor/image56.png
   :width: 600px
   :alt: Teams通知表示例

   Teams通知表示例

.. table:: Slack設定例
   :align: left
   
   +--------------------+-------------------------------------------------+
   | 通知名称           | 通知サンプル                                    |
   +====================+=================================================+
   | 通\                | 通知先のSlackのWebhook URLを入力して下さい。    |
   | 知先(CURLOPT_URL)  |                                                 |
   +--------------------+-------------------------------------------------+
   | ヘッダー(C\        | [ "Content-Type: application/json" ]            |
   | URLOPT_HTTPHEADER) |                                                 |
   +--------------------+-------------------------------------------------+
   | メッセージ(C\      | {                                               |
   | URLOPT_POSTFIELDS) |                                                 |
   |                    | "username": "ITAConductor実行通知",             |
   |                    |                                                 |
   |                    | "text": "通知名：__NOTICE_NAME__, \\n           |
   |                    | Conductor名称: \__CONDUCTOR_NAME__, \\n         |
   |                    | Con                                             |
   |                    | ductorインスタンスID:__CONDUCTOR_INSTANCE_ID__, |
   |                    | \\n オペレーションID: \__OPERATION_ID__,        |
   |                    | \\nオペレーション名:__OPERATION_NAME__,         |
   |                    | \\nステータスID: \__STATUS_ID__, \\nステータス: |
   |                    | \__STATUS_NAME__, \\n実行ユーザー:              |
   |                    | \__EXECUTION_USER__, \\n 予約日時:              |
   |                    | \__TIME_BOOK__, \\n開始日時: \__TIME_START__,   |
   |                    | \\n終了日時: \__TIME_END__, \\n緊急停止フラグ:  |
   |                    | \__ABORT_FLAG__, \\n 作業URL: \__JUMP_URL_\_ "  |
   |                    |                                                 |
   |                    | }                                               |
   +--------------------+-------------------------------------------------+
   | PROXY / URL        | http://proxy.co.jp                              |
   | (CURLOPT_PROXY)    |                                                 |
   +--------------------+-------------------------------------------------+
   | PROXY / PORT       | 8080                                            |
   | (\                 |                                                 |
   | CURLOPT_PROXYPORT) |                                                 |
   +--------------------+-------------------------------------------------+
   | 作業確認URL(FQDN)  | http://exastro-it-automation.local              |
   +--------------------+-------------------------------------------------+
   | その他             |                                                 |
   +--------------------+-------------------------------------------------+
   | 開始日時           |                                                 |
   +--------------------+-------------------------------------------------+
   | 終了日時           |                                                 |
   +--------------------+-------------------------------------------------+


.. figure:: ./conductor/image57.png
   :width: 600px
   :alt: Slack通知表示例

   Slack通知表示例

.. table:: 設定サンプル(Proxy設定、通知抑止設定、その他設定あり)
   :align: left

   +--------------------+-------------------------------------------------+
   | 通知名称           | 通知サンプル                                    |
   +====================+=================================================+
   | 通\                | https://sample.webhook.xxx.com/yyyyyyyy         |
   | 知先(CURLOPT_URL)  |                                                 |
   +--------------------+-------------------------------------------------+
   | ヘッダー(C\        | [ "Content-Type: application/json" ]            |
   | URLOPT_HTTPHEADER) |                                                 |
   +--------------------+-------------------------------------------------+
   | メッセージ(C\      | {"text": "通知内容"}                            |
   | URLOPT_POSTFIELDS) |                                                 |
   +--------------------+-------------------------------------------------+
   | PROXY / URL        | http://proxy.co.jp                              |
   | (CURLOPT_PROXY)    |                                                 |
   +--------------------+-------------------------------------------------+
   | PROXY / PORT       | 8080                                            |
   | (\                 |                                                 |
   | CURLOPT_PROXYPORT) |                                                 |
   +--------------------+-------------------------------------------------+
   | 作業確認URL(FQDN)  | http://exastro-it-automation.local              |
   +--------------------+-------------------------------------------------+
   | その他             | {"CURLOPT_TIMEOUT":"10"}                        |
   +--------------------+-------------------------------------------------+
   | 開始日時           | 2020/01/01 00:00:00                             |
   +--------------------+-------------------------------------------------+
   | 終了日時           | 2020/01/01 00:00:00                             |
   +--------------------+-------------------------------------------------+
   | 備考               | 自由記述欄です                                  |
   +--------------------+-------------------------------------------------+

.. _conductor_notification_log:
通知ログ出力例
~~~~~~~~~~~~~

通知ログの構造
**************


.. code-block:: 

   YYYY-MM-dd HH:ii:ss 通知実行結果(<ID:通知名称>,<ID:ステータス名称>) 
                                                                       
   Array                                                               
                                                                       
   (                                                                   
                                                                       
   [RETURN_MSG] =>　 ：通知実行時の返り値                              
                                                                       
   [OPTION] => Array 　　　　　　　　　　　 ：通知実行時のオプション   
                                                                       
   (                                                                   
                                                                       
   [CURLOPT_XXXXXXXX] =>                                               
                                                                       
   ・・・・・・・・略・・・・・・・・                                  
                                                                       
   )                                                                   
                                                                       
   [RESSULT] => Array ：通知実行結果                                   
                                                                       
   (                                                                   
                                                                       
   [url] => ：通知先URL                                                
                                                                       
   [http_code] => 　：HTTPステータスコード                             
                                                                       
   ・・・・・・・・略・・・・・・・・                                  
                                                                       
   )                                                                   
                                                                       
   )                                                                 

例) 通知実行ログ(正常)
^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: 

   2021-11-05 15:10:22 通知実行結果(2:通知サンプル,5:正常終了)           
                                                                         
   Array                                                                 
                                                                         
   (                                                                     
                                                                         
   [RETURN_MSG] => 1                                                     
                                                                         
   [OPTION] => Array                                                     
                                                                         
   (                                                                     
                                                                         
   [CURLOPT_CUSTOMREQUEST] => POST                                       
                                                                         
   [CURLOPT_HEADER] =>                                                   
                                                                         
   [CURLOPT_SSL_VERIFYPEER] =>                                           
                                                                         
   [CURLOPT_SSL_VERIFYHOST] => 0                                         
                                                                         
   [CURLOPT_TIMEOUT] => 5                                                
                                                                         
   [CURLOPT_CONNECTTIMEOUT] => 2                                         
                                                                         
   [CURLOPT_RETURNTRANSFER] => 1                                         
                                                                         
   [CURLOPT_HTTPPROXYTUNNEL] => 1                                        
                                                                         
   [CURLOPT_URL] => https://sample.webhook.xxx.com/yyyyyyyy              
                                                                         
   [CURLOPT_HTTPHEADER] => Array                                         
                                                                         
   (                                                                     
                                                                         
   [0] => Content-Type: application/json                                 
                                                                         
   )                                                                     
                                                                         
   [CURLOPT_POSTFIELDS] => {"text": "通知名：通知サンプル2, <br>         
   Conductor名称: NULL, <br> ConductorインスタンスID:3, <br>             
   オペレーションID: 1, <br>オペレーション名:OP_NULL, <br>ステータスID:  
   5, <br>ステータス: 正常終了, <br>実行ユーザー: システム管理者, <br>   
   予約日時: , <br>開始日時: 2021/11/05 15:10:08, <br>終了日時:          
   2021/11/05 15:10:18, <br>緊急停止フラグ: 未発令, <br> 作業URL:        
   http://exastro-it-automation.lo                                       
   cal/default/menu/01_browse.php?no=2100180005&conductor_instance_id=3, 
   <br> "}                                                               
                                                                         
   [CURLOPT_PROXY] => https://sample.proxy.xxx.com                       
                                                                         
   [CURLOPT_PROXYPORT] => 8080                                           
                                                                         
   )                                                                     
                                                                         
   [RESSULT] => Array                                                    
                                                                         
   (                                                                     
                                                                         
   [url] => https://sample.webhook.xxx.com/yyyyyyyy                      
                                                                         
   [content_type] => text/plain; charset=utf-8                           
                                                                         
   [http_code] => 200                                                    
                                                                         
   [header_size] => 834                                                  
                                                                         
   [request_size] => 1005                                                
                                                                         
   [filetime] => -1                                                      
                                                                         
   [ssl_verify_result] => 0                                              
                                                                         
   [redirect_count] => 0                                                 
                                                                         
   [total_time] => 1.519411                                              
                                                                         
   [namelookup_time] => 0.083714                                         
                                                                         
   [connect_time] => 0.107712                                            
                                                                         
   [pretransfer_time] => 0.44203                                         
                                                                         
   [size_upload] => 560                                                  
                                                                         
   [size_download] => 1                                                  
                                                                         
   [speed_download] => 0                                                 
                                                                         
   [speed_upload] => 368                                                 
                                                                         
   [download_content_length] => 1                                        
                                                                         
   [upload_content_length] => 560                                        
                                                                         
   [starttransfer_time] => 1.519364                                      
                                                                         
   [redirect_time] => 0                                                  
                                                                         
   [redirect_url] =>                                                     
                                                                         
   [primary_ip] => XXX.XXX.XXX.XXX                                       
                                                                         
   [certinfo] => Array                                                   
                                                                         
   (                                                                     
                                                                         
   )                                                                     
                                                                         
   [primary_port] => 8080                                                
                                                                         
   [local_ip] => XXX.XXX.XXX.XXX                                         
                                                                         
   [local_port] => 39874                                                 
                                                                         
   )                                                                     
                                                                         
   )


例) 通知実行ログ(異常)
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: 

   2021-11-05 15:10:20 通知実行結果(1:通知サンプル,5:正常終了)           
                                                                         
   Array                                                                 
                                                                         
   (                                                                     
                                                                         
   [RETURN_MSG] =>                                                       
                                                                         
   [OPTION] => Array                                                     
                                                                         
   (                                                                     
                                                                         
   [CURLOPT_CUSTOMREQUEST] => POST                                       
                                                                         
   [CURLOPT_HEADER] =>                                                   
                                                                         
   [CURLOPT_SSL_VERIFYPEER] =>                                           
                                                                         
   [CURLOPT_SSL_VERIFYHOST] => 0                                         
                                                                         
   [CURLOPT_TIMEOUT] => 5                                                
                                                                         
   [CURLOPT_CONNECTTIMEOUT] => 2                                         
                                                                         
   [CURLOPT_RETURNTRANSFER] => 1                                         
                                                                         
   [CURLOPT_HTTPPROXYTUNNEL] => 1                                        
                                                                         
   [CURLOPT_URL] => https://sample.webhook.xxx.com/yyyyyyyy              
                                                                         
   [CURLOPT_HTTPHEADER] => Array                                         
                                                                         
   (                                                                     
                                                                         
   [0] => Content-Type: application/json                                 
                                                                         
   )                                                                     
                                                                         
   [CURLOPT_POSTFIELDS] => {"text": "通知名：通知サンプル, <br>          
   Conductor名称: NULL, <br> ConductorインスタンスID:3, <br>             
   オペレーションID: 1, <br>オペレーション名:OP_NULL, <br>ステータスID:  
   5, <br>ステータス: 正常終了, <br>実行ユーザー: システム管理者, <br>   
   予約日時: , <br>開始日時: 2021/11/05 15:10:08, <br>終了日時:          
   2021/11/05 15:10:18, <br>緊急停止フラグ: 未発令, <br> 作業URL:        
   http://exastro-it-automation.lo                                       
   cal/default/menu/01_browse.php?no=2100180005&conductor_instance_id=3, 
   <br> "}                                                               
                                                                         
   [CURLOPT_PROXY] =>                                                    
                                                                         
   [CURLOPT_PROXYPORT] =>                                                
                                                                         
   )                                                                     
                                                                         
   [RESSULT] => Array                                                    
                                                                         
   (                                                                     
                                                                         
   [url] => https://sample.webhook.xxx.com/yyyyyyyy                      
                                                                         
   [content_type] =>                                                     
                                                                         
   [http_code] => 0                                                      
                                                                         
   [header_size] => 0                                                    
                                                                         
   [request_size] => 0                                                   
                                                                         
   [filetime] => -1                                                      
                                                                         
   [ssl_verify_result] => 0                                              
                                                                         
   [redirect_count] => 0                                                 
                                                                         
   [total_time] => 2.011686                                              
                                                                         
   [namelookup_time] => 0.532318                                         
                                                                         
   [connect_time] => 0                                                   
                                                                         
   [pretransfer_time] => 0                                               
                                                                         
   [size_upload] => 0                                                    
                                                                         
   [size_download] => 0                                                  
                                                                         
   [speed_download] => 0                                                 
                                                                         
   [speed_upload] => 0                                                   
                                                                         
   [download_content_length] => -1                                       
                                                                         
   [upload_content_length] => -1                                         
                                                                         
   [starttransfer_time] => 0                                             
                                                                         
   [redirect_time] => 0                                                  
                                                                         
   [redirect_url] =>                                                     
                                                                         
   [primary_ip] => XXX.XXX.XXX.XXX                                       
                                                                         
   [certinfo] => Array                                                   
                                                                         
   (                                                                     
                                                                         
   )                                                                     
                                                                         
   [primary_port] => 443                                                 
                                                                         
   [local_ip] =>                                                         
                                                                         
   [local_port] => 0                                                     
                                                                         
   )                                                                     
                                                                         
   )                                                                     
