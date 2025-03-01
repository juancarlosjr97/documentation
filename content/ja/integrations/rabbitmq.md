---
app_id: rabbitmq
app_uuid: a10b582b-71ef-4773-b7b8-b7751c724620
assets:
  dashboards:
    rabbitmq: assets/dashboards/rabbitmq_dashboard.json
    rabbitmq_screenboard: assets/dashboards/rabbitmq_screenboard_dashboard.json
  integration:
    auto_install: true
    configuration:
      spec: assets/configuration/spec.yaml
    events:
      creates_events: true
    metrics:
      check: rabbitmq.queue.messages
      metadata_path: metadata.csv
      prefix: rabbitmq.
    process_signatures:
    - rabbitmq
    - rabbitmq-server
    service_checks:
      metadata_path: assets/service_checks.json
    source_type_id: 51
    source_type_name: RabbitMQ
  monitors:
    Disk space is low: assets/monitors/disk_usage_prometheus.json
    Level of disk usage is too high for host: assets/monitors/disk_usage.json
    Messages are ready in RabbitMQ queue: assets/monitors/message_ready.json
    Messages unacknowledged rate is higher than usual: assets/monitors/message_unacknowledge_rate_anomaly.json
    RabbitMQ queue has 0 consumers: assets/monitors/consumers_at_zero.json
    Unacknowledged Messages are higher than usual: assets/monitors/message_unack_prometheus.json
  saved_views:
    pid_overview: assets/saved_views/status_overview.json
    rabbitmq_pattern: assets/saved_views/rabbitmq_pattern.json
    rabbitmq_processes: assets/saved_views/rabbitmq_processes.json
author:
  homepage: https://www.datadoghq.com
  name: Datadog
  sales_email: info@datadoghq.com
  support_email: help@datadoghq.com
categories:
- log collection
- message queues
custom_kind: インテグレーション
dependencies:
- https://github.com/DataDog/integrations-core/blob/master/rabbitmq/README.md
display_on_public_website: true
draft: false
git_integration_title: rabbitmq
integration_id: rabbitmq
integration_title: RabbitMQ
integration_version: 7.1.0
is_public: true
manifest_version: 2.0.0
name: rabbitmq
public_title: RabbitMQ
short_description: キューサイズ、コンシューマーカウント、未承認メッセージなどを追跡
supported_os:
- linux
- windows
- macos
tile:
  changelog: CHANGELOG.md
  classifier_tags:
  - Category::ログの収集
  - Category::Message Queues
  - Supported OS::Linux
  - Supported OS::Windows
  - Supported OS::macOS
  - Offering::Integration
  - Product::Data Streams Monitoring
  configuration: README.md#Setup
  description: キューサイズ、コンシューマーカウント、未承認メッセージなどを追跡
  media: []
  overview: README.md#Overview
  resources:
  - resource_type: blog
    url: https://www.datadoghq.com/blog/rabbitmq-monitoring
  - resource_type: blog
    url: https://www.datadoghq.com/blog/rabbitmq-monitoring-tools
  - resource_type: blog
    url: https://www.datadoghq.com/blog/monitoring-rabbitmq-performance-with-datadog
  support: README.md#Support
  title: RabbitMQ
---

<!--  SOURCED FROM https://github.com/DataDog/integrations-core -->


![RabbitMQ ダッシュボード][1]

## 概要

このチェックは、Datadog Agent を通じて [RabbitMQ][2] を監視します。

- キューサイズ、コンシューマー数、未承認メッセージ、再配信メッセージなどのキューベースの統計を追跡できます。
- 待機プロセス、使用されたソケット、使用されたファイルディスクリプタなどのノードベースの統計を追跡できます。
- vhost の稼働状態や接続数を監視できます。

## セットアップ

### インストール

RabbitMQ チェックは [Datadog Agent][3] パッケージに含まれています。サーバーに追加でインストールする必要はありません。

### 構成

RabbitMQ は、[RabbitMQ Management Plugin][4] と [RabbitMQ Prometheus Plugin][5] の 2 つの方法でメトリクスを公開します。Datadog インテグレーションは両方のバージョンをサポートしています。使用するバージョンに関連する構成説明については、このファイルの指示に従ってください。また、Datadog インテグレーションには、それぞれのバージョンに対応するすぐに使えるダッシュボードとモニターが付属しており、ダッシュボードとモニターのタイトルで区別されています。

#### RabbitMQ の準備

##### [RabbitMQ Prometheus Plugin][5]。

*RabbitMQ v3.8 から [RabbitMQ Prometheus Plugin][5] がデフォルトで有効になっています。*

*RabbitMQ の Prometheus プラグインバージョンは、Datadog Agent による Python 3 のサポートが必要なため、Agent v6 以降でのみサポート可能です。Prometheus プラグインバージョンのインテグレーションを構成する前に、Agent がアップデートされていることを確認してください。*

インスタンス構成で `prometheus_plugin` セクションを設定します。`prometheus_plugin` オプションを使用する場合、Management Plugin に関連する設定は無視されます。

 ```yaml
 instances:
   - prometheus_plugin:
       url: http://<HOST>:15692
 ```

これにより、1 つの RabbitMQ ノードで [`/metrics` エンドポイント][6]のスクレイピングが可能になります。また、Datadog は [`/metrics/detailed` エンドポイント][7]からもデータを収集することができます。

 ```yaml
 instances:
   - prometheus_plugin:
       url: http://<HOST>:15692
       unaggregated_endpoint: detailed?family=queue_coarse_metrics
 ```
 これにより、[`/metrics/detailed` エンドポイント][7]をスクレイピングして、キューの粗いメトリクスを収集することができます。

##### [RabbitMQ Management Plugin][4]。

プラグインを有効化します。Agent ユーザーは、少なくとも`monitoring`タグとこれらの必要な権限が必要です。

| アクセス許可 | コマンド            |
| ---------- | ------------------ |
| **conf**   | `^aliveness-test$` |
| **write**  | `^amq\.default$`   |
| **read**   | `.*`               |

次のコマンドで、デフォルトの vhost 用に Agent ユーザーを作成します。

```text
rabbitmqctl add_user datadog <シークレット>
rabbitmqctl set_permissions  -p / datadog "^aliveness-test$" "^amq\.default$" ".*"
rabbitmqctl set_user_tags datadog monitoring
```

ここで、`/` はデフォルトのホストを表します。これを、指定した仮想ホスト名に設定してください。詳細については、[RabbitMQ のドキュメント][8]を参照してください。

{{< tabs >}}
{{% tab "ホスト" %}}

#### ホスト

ホストで実行中の Agent に対してこのチェックを構成するには

##### メトリクスの収集

1. RabbitMQ メトリクスの収集を開始するには、[Agent のコンフィギュレーションディレクトリ][1]のルートにある `conf.d/` フォルダーの `rabbitmq.d/conf.yaml` ファイルを編集します。使用可能なすべてのコンフィギュレーションオプションの詳細については、[サンプル rabbitmq.d/conf.yaml][2] を参照してください。

   **注**: Agent は、デフォルトですべてのキュー、vhost、ノードをチェックしますが、リストまたは正規表現を指定してこれを制限できます。例については、[rabbitmq.d/conf.yaml][2] を参照してください。

2. [Agent を再起動します][3]。

##### ログ収集

_Agent バージョン 6.0 以降で利用可能_

1. デフォルトのログファイルの場所を変更するには、`RABBITMQ_LOGS` 環境変数を設定するか、以下の行を RabbitMQ 構成ファイル (`/etc/rabbitmq/rabbitmq.conf`) に追加します。

   ```conf
     log.dir = /var/log/rabbit
     log.file = rabbit.log
   ```

2. Datadog Agent で、ログの収集はデフォルトで無効になっています。以下のように、`datadog.yaml` ファイルでこれを有効にします。

   ```yaml
   logs_enabled: true
   ```

3. RabbitMQ ログの収集を開始するには、`rabbitmq.d/conf.yaml` ファイルの `logs` セクションを編集します。

   ```yaml
   logs:
     - type: file
       path: /var/log/rabbit/*.log
       source: rabbitmq
       service: myservice
       log_processing_rules:
         - type: multi_line
           name: logs_starts_with_equal_sign
           pattern: "="
   ```

4. [Agent を再起動します][3]。

[1]: https://docs.datadoghq.com/ja/agent/guide/agent-configuration-files/#agent-configuration-directory
[2]: https://github.com/DataDog/integrations-core/blob/master/rabbitmq/datadog_checks/rabbitmq/data/conf.yaml.example
[3]: https://docs.datadoghq.com/ja/agent/guide/agent-commands/#start-stop-and-restart-the-agent
{{% /tab %}}
{{% tab "コンテナ化" %}}

#### コンテナ化

Datadog の [Docker コンテナオートディスカバリー][1]を活用することができます。RabbitMQ 固有の設定は、`auto_conf.yaml` の設定例を参照してください。

Kubernetes などのコンテナ環境の場合は、[オートディスカバリーのインテグレーションテンプレート][2]のガイドを参照して、次のパラメーターを適用してください。

##### メトリクスの収集

| パラメーター            | 値                                        |
| -------------------- | -------------------------------------------- |
| `<INTEGRATION_NAME>` | `rabbitmq`                                   |
| `<INIT_CONFIG>`      | 空白または `{}`                                |
| `<INSTANCE_CONFIG>`  | `{"prometheus_plugin": {"url": "http://%%host%%:15692"}}` |

##### ログ収集

_Agent v6.0 以降で利用可能_

Datadog Agent で、ログの収集はデフォルトで無効になっています。有効にする方法については、[Kubernetes ログ収集][3]を参照してください。

| パラメーター      | 値                                                                                                                                               |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `<LOG_CONFIG>` | `{"source": "rabbitmq", "service": "rabbitmq", "log_processing_rules": [{"type":"multi_line","name":"logs_starts_with_equal_sign", "pattern": "="}]}` |

[1]: https://docs.datadoghq.com/ja/containers/docker/integrations/?tab=dockeradv2
[2]: https://docs.datadoghq.com/ja/agent/kubernetes/integrations/
[3]: https://docs.datadoghq.com/ja/agent/kubernetes/log/
{{% /tab %}}
{{< /tabs >}}

### 検証

[Agent の status サブコマンドを実行][9]し、Checks セクションで `rabbitmq` を探します。

## 収集データ

### メトリクス
{{< get-metrics-from-git "rabbitmq" >}}


### イベント

### サービスチェック
{{< get-service-checks-from-git "rabbitmq" >}}


## トラブルシューティング

### Prometheus Plugin への移行

Prometheus Plugin は、Management Plugin とは異なるメトリクスを公開します。
ここでは、Management Plugin から Prometheus Plugin に移行する際に注意すべき点を説明します。

- [この表][10]でメトリクスを検索してください。メトリクスの説明に `[OpenMetricsV2]` タグが含まれていれば、Prometheus Plugin で利用可能です。Management Plugin のみで利用可能なメトリクスは、説明にタグを持ちません。
- Management Plugin のメトリクスを使用しているダッシュボードやモニターは機能しません。*OpenMetrics Version* と表示されているダッシュボードやモニターに切り替えてください。
- デフォルトの構成では、集計されたメトリクスが収集されます。これは、例えば、キューによってタグ付けされたメトリクスが存在しないことを意味します。オプション `prometheus_plugin.unaggregated_endpoint` を構成すると、集計せずにメトリクスを取得することができます。
- サービスチェックの `rabbitmq.status` は `rabbitmq.openmetrics.health` に置き換えられました。サービスチェックの `rabbitmq.aliveness` は、Prometheus Plugin では同等のものはありません。

Prometheus Plugin では、いくつかのタグが変更されます。以下の表は、より一般的なタグの変更点を説明したものです。

| 管理          | Prometheus                               |
|:--------------------|:-----------------------------------------|
| `queue_name`        | `queue`                                  |
| `rabbitmq_vhost`    | `vhost`、`exchange_vhost`、`queue_vhost` |
| `rabbitmq_exchange` | `exchange`                               |

詳しくは、[タグファミリーに基づいて RabbitMQ キューをタグ付け][11]を参照してください。

ご不明な点は、[Datadog のサポートチーム][12]までお問合せください。

## その他の参考資料

お役に立つドキュメント、リンクや記事:

- [RabbitMQ 監視のキーメトリクス][13]
- [RabbitMQ 監視ツールでメトリクスを収集][14]
- [Datadog を使用した RabbitMQ パフォーマンスの監視][15]


[1]: https://raw.githubusercontent.com/DataDog/integrations-core/master/rabbitmq/images/rabbitmq_dashboard.png
[2]: https://www.rabbitmq.com
[3]: https://app.datadoghq.com/account/settings/agent/latest
[4]: https://www.rabbitmq.com/management.html
[5]: https://www.rabbitmq.com/prometheus.html
[6]: https://www.rabbitmq.com/prometheus.html#default-endpoint
[7]: https://www.rabbitmq.com/prometheus.html#detailed-endpoint
[8]: https://www.rabbitmq.com/rabbitmqctl.8.html#set_permissions
[9]: https://docs.datadoghq.com/ja/agent/guide/agent-commands/#agent-status-and-information
[10]: https://docs.datadoghq.com/ja/integrations/rabbitmq/?tab=host#metrics
[11]: https://docs.datadoghq.com/ja/integrations/faq/tagging-rabbitmq-queues-by-tag-family/
[12]: https://docs.datadoghq.com/ja/help/
[13]: https://www.datadoghq.com/blog/rabbitmq-monitoring
[14]: https://www.datadoghq.com/blog/rabbitmq-monitoring-tools
[15]: https://www.datadoghq.com/blog/monitoring-rabbitmq-performance-with-datadog