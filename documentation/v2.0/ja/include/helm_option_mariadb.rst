
.. list-table:: MariaDB コンテナのオプションパラメータ
   :widths: 25 25 10 20
   :header-rows: 1
   :align: left
   :class: filter-table

   * - パラメータ
     - 説明
     - 変更
     - デフォルト値・選択可能な設定値
   * - exastro-platform.mariadb.enabled
     - MariaDB コンテナのデプロイの有無
     - 可
     - | :program:`true` (デフォルト): MariaDB コンテナをデプロイします。
       | :program:`false` : MariaDB コンテナをデプロイしません。
   * - exastro-platform.mariadb.image.repository
     - コンテナイメージのリポジトリ名
     - 不可
     - "mariadb"
   * - exastro-platform.mariadb.image.tag
     - コンテナイメージのタグ
     - 不可
     - "10.9"
   * - exastro-platform.mariadb.image.pullPolicy
     - イメージプルポリシー
     - 可
     - | :program:`IfNotPresent` (デフォルト): コンテナイメージが存在しない場合のみプル
       | :program:`Always`: 毎回必ずプル
       | :program:`None`: プルしない
   * - exastro-platform.mariadb.resources.requests.memory
     - メモリ要求
     - 可
     - "256Mi"
   * - exastro-platform.mariadb.resources.requests.cpu
     - CPU要求
     - 可
     - "1m"
   * - exastro-platform.mariadb.resources.limits.memory
     - メモリ上限
     - 可
     - "2Gi"
   * - exastro-platform.mariadb.resources.limits.cpu
     - CPU上限
     - 可
     - "4"