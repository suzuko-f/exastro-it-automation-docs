
1. サービス再開

   | サービス停止時に取得した各 Deployment の Pod 起動数を元に戻します。

   .. code-block::
     :caption: コマンド
     :linenos:

     kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=${RS_AE}
     kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=${RS_ALRV}
     kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=${RS_ATS}
     kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=${RS_CS}
     kubectl scale deployment ita-by-menu-create -n exastro --replicas=${RS_MC}
     kubectl scale deployment platform-auth -n exastro --replicas=${RS_PA}

2. Pod 起動数の確認

   | 上記で起動した対象の Pod 数が元に戻りすべて :kbd:`READY` になっていることを確認

   .. code-block:: bash
     :caption: コマンド

     kubectl get deployment -n exastro

   .. code-block:: bash
     :caption: 実行結果

     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
     mariadb                                  1/1     1            1           3h7m
     ita-web-server                           1/1     1            1           3h7m
     platform-web                             1/1     1            1           3h7m
     ita-api-admin                            1/1     1            1           3h7m
     ita-api-organization                     1/1     1            1           3h7m
     platform-api                             1/1     1            1           3h7m
     keycloak                                 1/1     1            1           3h7m
     ita-by-ansible-legacy-role-vars-listup   1/1     1            1           3h7m
     ita-by-conductor-synchronize             1/1     1            1           3h7m
     ita-by-ansible-execute                   1/1     1            1           3h7m
     ita-by-ansible-towermaster-sync          1/1     1            1           3h7m
     platform-auth                            1/1     1            1           3h7m
     ita-by-menu-create                       0/0     0            0           3h7m