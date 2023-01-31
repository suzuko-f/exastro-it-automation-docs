
.. list-table:: Keycloak コンテナのオプションパラメータ
   :widths: 25 25 10 20
   :header-rows: 1
   :align: left
   :class: filter-table

   * - パラメータ
     - 説明
     - 変更
     - デフォルト値・選択可能な設定値
   * - exastro-platform.keycloak.enabled
     - Keycloak コンテナのデプロイの有無
     - 可
     - | :program:`true` (デフォルト): Keycloak コンテナをデプロイします。
       | :program:`false` : Keycloak コンテナをデプロイしません。
   * - exastro-platform.keycloak.image.repository
     - コンテナイメージのリポジトリ名
     - 不可
     - "exastro/keycloak"
   * - exastro-platform.keycloak.image.tag
     - コンテナイメージのタグ
     - 不可
     - "1.0.6"
   * - exastro-platform.keycloak.image.pullPolicy
     - イメージプルポリシー
     - 可
     - | :program:`IfNotPresent` (デフォルト): コンテナイメージが存在しない場合のみプル
       | :program:`Always`: 毎回必ずプル
       | :program:`None`: プルしない
   * - exastro-platform.keycloak.resources.requests.memory
     - メモリ要求
     - 可
     - "256Mi"
   * - exastro-platform.keycloak.resources.requests.cpu
     - CPU要求
     - 可
     - "1m"
   * - exastro-platform.keycloak.resources.limits.memory
     - メモリ上限
     - 可
     - "2Gi"
   * - exastro-platform.keycloak.resources.limits.cpu
     - CPU上限
     - 可
     - "4"
