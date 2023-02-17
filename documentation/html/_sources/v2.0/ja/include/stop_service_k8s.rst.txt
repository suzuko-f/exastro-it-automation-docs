
1. Pod 起動数の確認

   | 作業前の Pod 起動数の確認をし、状態を記録します。

   .. code-block:: bash
     :caption: コマンド

     RS_AE=`kubectl get deploy ita-by-ansible-execute -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
     RS_ALRV=`kubectl get deploy ita-by-ansible-legacy-role-vars-listup -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
     RS_ATS=`kubectl get deploy ita-by-ansible-towermaster-sync -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
     RS_CS=`kubectl get deploy ita-by-conductor-synchronize -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
     RS_MC=`kubectl get deploy ita-by-menu-create -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`
     RS_PA=`kubectl get deploy platform-auth -o jsonpath='{@.spec.replicas}{"\n"}' -n exastro`

2. リバースプロキシの停止

   | リバースプロキシ (platform-auth) の Pod 起動数を 0 に変更し、エンドユーザーからのアクセスを制限します。

   .. code-block:: bash
     :caption: コマンド

     kubectl scale deployment platform-auth -n exastro --replicas=0

3. バックヤード処理の停止

   | バックヤード処理 (ita-by-\*\*\*) の Pod 起動数を 0 に変更し、データベースの更新を停止します。

   .. code-block:: bash
     :caption: コマンド
     :linenos:

     kubectl scale deployment ita-by-ansible-execute -n exastro --replicas=0
     kubectl scale deployment ita-by-ansible-legacy-role-vars-listup -n exastro --replicas=0
     kubectl scale deployment ita-by-ansible-towermaster-sync -n exastro --replicas=0
     kubectl scale deployment ita-by-conductor-synchronize -n exastro --replicas=0
     kubectl scale deployment ita-by-menu-create -n exastro --replicas=0

4. Pod 起動数の確認

   | 上記で停止した対象の Pod 数が 0 になっていることを確認

   .. code-block:: bash
     :caption: コマンド

     kubectl get deployment -n exastro

   .. code-block:: bash
     :caption: 実行結果

     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
     mariadb                                  1/1     1            1           3h41m
     ita-web-server                           1/1     1            1           3h41m
     platform-web                             1/1     1            1           3h41m
     ita-api-admin                            1/1     1            1           3h41m
     ita-api-organization                     1/1     1            1           3h41m
     platform-api                             1/1     1            1           3h41m
     keycloak                                 1/1     1            1           3h41m
     ita-by-menu-create                       0/0     0            0           3h41m
     ita-by-ansible-execute                   0/0     0            0           3h41m
     ita-by-ansible-legacy-role-vars-listup   0/0     0            0           3h41m
     ita-by-ansible-towermaster-sync          0/0     0            0           3h41m
     ita-by-conductor-synchronize             0/0     0            0           3h41m
     platform-auth                            0/0     0            0           3h41m