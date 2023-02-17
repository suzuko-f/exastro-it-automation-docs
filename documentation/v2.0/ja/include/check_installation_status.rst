| コマンドラインから以下のコマンドを入力して、インストールが完了していることを確認します。

.. code-block:: bash
    :caption: コマンド
    
    # Pod の一覧を取得
    kubectl get po -n exastro
    
    | 正常動作している場合は、すべて “Running” もしくは “Completed” となります。
    | ※正常に起動するまで数分かかる場合があります。


.. code-block:: bash
    :caption: 出力結果
    
    NAME                                                      READY   STATUS      RESTARTS   AGE
    ita-by-menu-create-7fccfc7f57-7cc2f                       1/1     Running     0          11m
    ita-by-conductor-synchronize-9dc6cfbdf-vp64z              1/1     Running     0          11m
    ita-api-admin-85b7d8f977-cxr58                            1/1     Running     0          11m
    ita-api-organization-5c5f4b86cb-rmf4g                     1/1     Running     0          11m
    ita-by-ansible-execute-6cd6d4d5fd-wkxjd                   1/1     Running     0          11m
    ita-by-ansible-legacy-role-vars-listup-67dbf5586f-dhdb2   1/1     Running     0          11m
    ita-by-ansible-towermaster-sync-5674448c55-t9592          1/1     Running     0          11m
    ita-web-server-7dbf6fd6ff-2s7s4                           1/1     Running     0          11m
    platform-auth-5b57bc57bd-h4k2g                            1/1     Running     0          11m
    platform-web-9f9d486fd-zf5vb                              1/1     Running     0          11m
    mariadb-67dd78cc76-nthtf                                  1/1     Running     0          11m
    platform-api-8655864fbf-t5xxf                             1/1     Running     0          11m
    ita-setup-wv2t6                                           0/1     Completed   0          11m
    keycloak-7f7cdccb6b-rf4rj                                 1/1     Running     0          11m
    platform-setup-8vv2x                                      0/1     Completed   0          11m
