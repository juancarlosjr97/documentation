---
description: Azure 上で管理される SQL Server 用のデータベースモニタリングをインストールし、構成します。
further_reading:
- link: /integrations/sqlserver/
  tag: ドキュメント
  text: 基本的な SQL Server インテグレーション
- link: /database_monitoring/troubleshooting/?tab=sqlserver
  tag: ドキュメント
  text: よくある問題のトラブルシューティング
- link: /database_monitoring/guide/sql_deadlock/
  tag: ドキュメント
  text: デッドロックモニタリングの構成
title: Azure SQL Server のデータベースモニタリングの設定
---

Database Monitoring は、クエリメトリクス、クエリサンプル、実行計画、データベースの状態、フェイルオーバー、イベントを公開することで、Microsoft SQL Server データベースを詳細に可視化します。

データベースで Database Monitoring を有効にするには、以下の手順を実行します。

1. [Agent にデータベースへのアクセスを付与する](#grant-the-agent-access)
2. [Agent のインストールと構成](#install-and-configure-the-agent)
3. [Azure インテグレーションをインストールする](#install-the-azure-integration)

## はじめに

サポートされている SQL Server バージョン
: 2014、2016、2017、2019、2022

{{% dbm-sqlserver-before-you-begin %}}

## Agent にアクセスを付与する

Datadog Agent が統計やクエリを収集するためには、データベースサーバーへの読み取り専用のアクセスが必要となります。

{{< tabs >}}

{{% tab "Azure SQL Database" %}}

サーバーに接続するための読み取り専用ログインを作成し、必要な [Azure SQL Roles][1] を付与します。
```SQL
CREATE LOGIN datadog WITH PASSWORD = '<PASSWORD>';
CREATE USER datadog FOR LOGIN datadog;
ALTER SERVER ROLE ##MS_ServerStateReader## ADD MEMBER datadog;
ALTER SERVER ROLE ##MS_DefinitionReader## ADD MEMBER datadog;
-- Log Shipping Monitoring (Agent v7.50+ で利用可能) または
-- SQL Server Agent Monitoring (Agent v7.57+ で利用可能) のいずれも使用しない場合は、
-- 次の 3 行をコメントアウトしてください。
USE msdb;
CREATE USER datadog FOR LOGIN datadog;
GRANT SELECT to datadog;
```

このサーバー上の追加の Azure SQL Database それぞれへのアクセスを Agent に付与します。

```SQL
CREATE USER datadog FOR LOGIN datadog;
```

**注:** Microsoft Entra ID マネージドアイデンティティ認証もサポートされています。Azure SQL DB インスタンスの構成方法については、[ガイド][3]を参照してください。

Datadog Agent を構成する場合、特定の Azure SQL DB サーバーにあるアプリケーションデータベースごとに 1 つのチェックインスタンスを指定します。`master` やその他の[システムデータベース][2]は含めないでください。各データベースは分離された計算環境で実行されているため、Datadog Agent は Azure SQL DB の各アプリケーションデータベースに直接接続する必要があります。これは、`database_autodiscovery` が Azure SQL DB では機能しないことも意味するので、有効化してはいけません。

**注:** Azure SQL Database は、分離されたネットワークでデータベースをデプロイします。各データベースは単一のホストとして扱われます。つまり、Azure SQL Database をエラスティックプールで実行した場合、プール内の各データベースは個別のホストとして扱われます。

```yaml
init_config:
instances:
  - host: '<SERVER_NAME>.database.windows.net,1433'
    database: '<DATABASE_1>'
    username: datadog
    password: '<PASSWORD>'
    # プロジェクトとインスタンスを追加した後、CPU、メモリなどの追加のクラウドデータをプルするために Datadog Azure インテグレーションを構成します。
    azure:
      deployment_type: 'sql_database'
      fully_qualified_domain_name: '<SERVER_NAME>.database.windows.net'

  - host: '<SERVER_NAME>.database.windows.net,1433'
    database: '<DATABASE_2>'
    username: datadog
    password: '<PASSWORD>'
    # プロジェクトとインスタンスを追加した後、CPU、メモリなどの追加のクラウドデータをプルするために Datadog Azure インテグレーションを構成します。
    azure:
      deployment_type: 'sql_database'
      fully_qualified_domain_name: '<SERVER_NAME>.database.windows.net'
```

Datadog Agent のインストールと構成の詳細については、[Agent のインストール](#install-the-agent)を参照してください。

[1]: https://docs.microsoft.com/en-us/azure/azure-sql/database/security-server-roles
[2]: https://docs.microsoft.com/en-us/sql/relational-databases/databases/system-databases
[3]: /ja/database_monitoring/guide/managed_authentication
{{% /tab %}}

{{% tab "Azure SQL Managed Instance" %}}

サーバーに接続するために読み取り専用ログインを作成し、必要な権限を付与します。

#### SQL Server バージョン 2014+ の場合

```SQL
CREATE LOGIN datadog WITH PASSWORD = '<PASSWORD>';
CREATE USER datadog FOR LOGIN datadog;
GRANT CONNECT ANY DATABASE to datadog;
GRANT VIEW SERVER STATE to datadog;
GRANT VIEW ANY DEFINITION to datadog;
-- If not using either of Log Shipping Monitoring (available in Agent v7.50+) or
-- SQL Server Agent Monitoring (available in Agent v7.57+), comment out the next three lines:
USE msdb;
CREATE USER datadog FOR LOGIN datadog;
GRANT SELECT to datadog;
```

**注:** Azure マネージドアイデンティティ認証もサポートされています。Azure SQL DB インスタンスの構成方法については、[ガイド][1]を参照してください。

[1]: /ja/database_monitoring/guide/managed_authentication
{{% /tab %}}

{{% tab "Windows Azure VM の SQL Server" %}}

[Windows Azure VM の SQL Server][1] の場合は、[セルフホスティングの SQL Server のデータベースモニタリングを設定する][2]のドキュメントに従って、Windows Server ホスト VM に直接 Datadog Agent をインストールしてください。

[1]: https://docs.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview
[2]: /ja/database_monitoring/setup_sql_server/selfhosted/
{{% /tab %}}

{{< /tabs >}}

### パスワードを安全に保管
{{% dbm-secret %}}

## Agent のインストールと構成

Azure はホストへの直接アクセスを許可しないため、Datadog Agent は SQL Server ホストと通信可能な別のホストにインストールする必要があります。Agent のインストールと実行には、いくつかのオプションがあります。

{{< tabs >}}
{{% tab "Windows Host" %}}

SQL Server テレメトリーの収集を開始するには、まず [Datadog Agent をインストール][1]します。

SQL Server Agent のコンフィギュレーションファイル `C:\ProgramData\Datadog\conf.d\sqlserver.d\conf.yaml` を作成します。使用可能なすべての構成オプションは、[サンプルコンフィギュレーションファイル][2]を参照してください。

```yaml
init_config:
instances:
  - dbm: true
    host: '<HOSTNAME>,<SQL_PORT>'
    username: datadog
    password: 'ENC[datadog_user_database_password]'
    connector: adodbapi
    adoprovider: MSOLEDBSQL
    tags:  # オプション
      - 'service:<CUSTOM_SERVICE>'
      - 'env:<CUSTOM_ENV>'
    # プロジェクトとインスタンスを追加した後、Datadog Azure インテグレーションを構成して、CPU やメモリなどの追加のクラウドデータを取得します。
    azure:
      deployment_type: '<DEPLOYMENT_TYPE>'
      fully_qualified_domain_name: '<AZURE_INSTANCE_ENDPOINT>'
```

`deployment_type` と `name` フィールドの設定に関する追加情報は、[SQL Server インテグレーション仕様][3]を参照してください。

[Windows 認証][4]を利用する場合は、`connection_string: "Trusted_Connection=yes"` と設定し、`username` と `password` フィールドを省略します。

`service` と `env` タグを使用して、共通のタグ付けスキームでデータベースのテレメトリーを他のテレメトリーにリンクします。これらのタグが Datadog 全体でどのように使用されるかについては、[統合サービスタグ付け][5]を参照してください。

### 対応ドライバー

#### Microsoft ADO

推奨する [ADO][6] プロバイダーは、[Microsoft OLE DB Driver][7] です。Agent が動作しているホストにドライバーがインストールされていることを確認してください。
```yaml
connector: adodbapi
adoprovider: MSOLEDBSQL19  # バージョン 18 以下の MSOLEDBSQL に置き換えます
```

他の 2 つのプロバイダー、`SQLOLEDB` と `SQLNCLI` は、Microsoft によって非推奨とされており、もはや使用するべきではありません。

#### ODBC

推奨される ODBC ドライバーは [Microsoft ODBC Driver][8] です。Agent 7.51 以降、SQL Server 用 ODBC Driver 18 が Linux 用 Agent に含まれています。Windows の場合は、Agent を実行するホストにドライバーがインストールされていることを確認してください。

```yaml
connector: odbc
driver: '{ODBC Driver 18 for SQL Server}'
```

すべての Agent の構成が完了したら、[Datadog Agent を再起動][9]します。

### UpdateAzureIntegration

[Agent の status サブコマンドを実行][10]し、**Checks** セクションで `sqlserver` を探します。Datadog の[データベース][11]のページへ移動して開始します。


[1]: https://app.datadoghq.com/account/settings/agent/latest?platform=windows
[2]: https://github.com/DataDog/integrations-core/blob/master/sqlserver/datadog_checks/sqlserver/data/conf.yaml.example
[3]: https://github.com/DataDog/integrations-core/blob/master/sqlserver/assets/configuration/spec.yaml#L353-L383
[4]: https://docs.microsoft.com/en-us/sql/relational-databases/security/choose-an-authentication-mode
[5]: /ja/getting_started/tagging/unified_service_tagging
[6]: https://docs.microsoft.com/en-us/sql/ado/microsoft-activex-data-objects-ado
[7]: https://docs.microsoft.com/en-us/sql/connect/oledb/oledb-driver-for-sql-server
[8]: https://docs.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server
[9]: /ja/agent/configuration/agent-commands/#start-stop-and-restart-the-agent
[10]: /ja/agent/configuration/agent-commands/#agent-status-and-information
[11]: https://app.datadoghq.com/databases
{{% /tab %}}
{{% tab "Linux ホスト" %}}
SQL Server テレメトリーの収集を開始するには、まず [Datadog Agent をインストール][1]します。

Linux では、Datadog Agent の他に、ODBC SQL Server ドライバー (例えば、[Microsoft ODBC ドライバー][2]) がインストールされていることが必須となります。ODBC SQL Server がインストールされたら、`odbc.ini` と `odbcinst.ini` ファイルを `/opt/datadog-agent/embedded/etc` フォルダーにコピーします。

`odbc` コネクターを使用し、`odbcinst.ini` ファイルに示されているように、適切なドライバーを指定します。

SQL Server Agent のコンフィギュレーションファイル `/etc/datadog-agent/conf.d/sqlserver.d/conf.yaml` を作成します。使用可能なすべての構成オプションは、[サンプルコンフィギュレーションファイル][3]を参照してください。

```yaml
init_config:
instances:
  - dbm: true
    host: '<HOSTNAME>,<SQL_PORT>'
    username: datadog
    password: 'ENC[datadog_user_database_password]'
    connector: odbc
    driver: '<Driver from the `odbcinst.ini` file>'
    tags:  # オプション
      - 'service:<CUSTOM_SERVICE>'
      - 'env:<CUSTOM_ENV>'
    # プロジェクトとインスタンスを追加した後、Datadog Azure インテグレーションを構成して、CPU やメモリなどの追加のクラウドデータを取得します。
    azure:
      deployment_type: '<DEPLOYMENT_TYPE>'
      fully_qualified_domain_name: '<AZURE_ENDPOINT_ADDRESS>'
```

`deployment_type` と `name` フィールドの設定に関する追加情報は、[SQL Server インテグレーション仕様][4]を参照してください。

`service` と `env` タグを使用して、共通のタグ付けスキームでデータベースのテレメトリーを他のテレメトリーにリンクします。これらのタグが Datadog 全体でどのように使用されるかについては、[統合サービスタグ付け][5]を参照してください。

すべての Agent の構成が完了したら、[Datadog Agent を再起動][6]します。

### UpdateAzureIntegration

[Agent の status サブコマンドを実行][7]し、**Checks** セクションで `sqlserver` を探します。Datadog の[データベース][8]のページへ移動して開始します。


[1]: https://app.datadoghq.com/account/settings/agent/latest
[2]: https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server
[3]: https://github.com/DataDog/integrations-core/blob/master/sqlserver/datadog_checks/sqlserver/data/conf.yaml.example
[4]: https://github.com/DataDog/integrations-core/blob/master/sqlserver/assets/configuration/spec.yaml#L353-L383
[5]: /ja/getting_started/tagging/unified_service_tagging
[6]: /ja/agent/configuration/agent-commands/#start-stop-and-restart-the-agent
[7]: /ja/agent/configuration/agent-commands/#agent-status-and-information
[8]: https://app.datadoghq.com/databases
{{% /tab %}}
{{% tab "Docker" %}}
Docker コンテナで動作するデータベースモニタリング Agent を設定するには、Agent コンテナの Docker ラベルとして[オートディスカバリーのインテグレーションテンプレート][1]を設定します。

**注**: ラベルのオートディスカバリーを機能させるためには、Agent にDocker ソケットに対する読み取り権限が与えられている必要があります。

アカウントや環境に合わせて、値を置き換えます。利用可能なすべての構成オプションについては、[サンプルコンフィギュレーションファイル][2]を参照してください。

```bash
export DD_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
export DD_AGENT_VERSION=7.51.0

docker run -e "DD_API_KEY=${DD_API_KEY}" \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -l com.datadoghq.ad.check_names='["sqlserver"]' \
  -l com.datadoghq.ad.init_configs='[{}]' \
  -l com.datadoghq.ad.instances='[{
    "dbm": true,
    "host": "<HOSTNAME>,<SQL_PORT>",
    "connector": "odbc",
    "driver": "ODBC Driver 18 for SQL Server",
    "username": "datadog",
    "password": "<PASSWORD>",
    "tags": [
      "service:<CUSTOM_SERVICE>"
      "env:<CUSTOM_ENV>"
    ],
    "azure": {
      "deployment_type": "<DEPLOYMENT_TYPE>",
      "name": "<AZURE_ENDPOINT_ADDRESS>"
    }
  }]' \
  gcr.io/datadoghq/agent:${DD_AGENT_VERSION}
```

`deployment_type` と `name` フィールドの設定に関する追加情報は、[SQL Server インテグレーション仕様][3]を参照してください。

`service` と `env` タグを使用して、共通のタグ付けスキームでデータベースのテレメトリーを他のテレメトリーにリンクします。これらのタグが Datadog 全体でどのように使用されるかについては、[統合サービスタグ付け][4]を参照してください。

### UpdateAzureIntegration

[Agent の status サブコマンドを実行][5]し、**Checks** セクションで `sqlserver` を探します。または、Datadog の[データベース][6]のページへ移動して開始します。


[1]: /ja/agent/faq/template_variables/
[2]: https://github.com/DataDog/integrations-core/blob/master/sqlserver/datadog_checks/sqlserver/data/conf.yaml.example
[3]: https://github.com/DataDog/integrations-core/blob/master/sqlserver/assets/configuration/spec.yaml#L353-L383
[4]: /ja/getting_started/tagging/unified_service_tagging
[5]: /ja/agent/configuration/agent-commands/#agent-status-and-information
[6]: https://app.datadoghq.com/databases
{{% /tab %}}
{{% tab "Kubernetes" %}}
Kubernetes クラスターをお使いの場合は、データベースモニタリング用の [Datadog Cluster Agent][1] をご利用ください。

Kubernetes クラスターでクラスターチェックがまだ有効になっていない場合は、指示に従って[クラスターチェックを有効化][2]します。Cluster Agent の構成は、Cluster Agent コンテナにマウントされた静的ファイル、または Kubernetes サービスアノテーションのいずれかを使用することができます。

### Helm

以下の手順に従って、Kubernetes クラスターに [Datadog Cluster Agent][1] をインストールしてください。お使いのアカウントや環境に合わせて値を変更してください。

1. Helm の [Datadog Agent インストール手順][3]に従います。
2. YAML コンフィギュレーションファイル (Cluster Agent インストール手順の `datadog-values.yaml`) を更新して、以下を含めます。
    ```yaml
    clusterAgent:
      confd:
        sqlserver.yaml: |-
          cluster_check: true
          init_config:
          instances:
          - dbm: true
            host: <HOSTNAME>,1433
            username: datadog
            password: 'ENC[datadog_user_database_password]'
            connector: 'odbc'
            driver: '{ODBC Driver 18 for SQL Server}'
            include_ao_metrics: true  # Optional: For AlwaysOn users
            tags:  # Optional
              - 'service:<CUSTOM_SERVICE>'
              - 'env:<CUSTOM_ENV>'
            azure:
              deployment_type: '<DEPLOYMENT_TYPE>'
              fully_qualified_domain_name: '<AZURE_ENDPOINT_ADDRESS>'

    clusterChecksRunner:
      enabled: true
    ```

3. コマンドラインから上記のコンフィギュレーションファイルを使用して Agent をデプロイします。
    ```shell
    helm install datadog-agent -f datadog-values.yaml datadog/datadog
    ```

<div class="alert alert-info">
Windows の場合は、<code>helm install</code> コマンドに <code>--set targetSystem=windows</code> を追加します。
</div>

### マウントされたファイルで構成する

マウントされたコンフィギュレーションファイルを使ってクラスターチェックを構成するには、コンフィギュレーションファイルを Cluster Agent コンテナのパス `/conf.d/sqlserver.yaml` にマウントします。

```yaml
cluster_check: true  # このフラグを必ず入れてください
init_config:
instances:
  - dbm: true
    host: '<HOSTNAME>,<SQL_PORT>'
    username: datadog
    password: 'ENC[datadog_user_database_password]'
    connector: "odbc"
    driver: '{ODBC Driver 18 for SQL Server}'
    tags:  # オプション
      - 'service:<CUSTOM_SERVICE>'
      - 'env:<CUSTOM_ENV>'
    # プロジェクトとインスタンスを追加した後、Datadog Azure インテグレーションを構成して、CPU やメモリなどの追加のクラウドデータを取得します。
    azure:
      deployment_type: '<DEPLOYMENT_TYPE>'
      fully_qualified_domain_name: '<AZURE_ENDPOINT_ADDRESS>'
```

### Kubernetes サービスアノテーションで構成する

ファイルをマウントせずに、インスタンスのコンフィギュレーションを Kubernetes サービスとして宣言することができます。Kubernetes 上で動作する Agent にこのチェックを設定するには、Datadog Cluster Agent と同じネームスペースにサービスを作成します。


```yaml
apiVersion: v1
kind: Service
metadata:
  name: sqlserver-datadog-check-instances
  annotations:
    ad.datadoghq.com/service.check_names: '["sqlserver"]'
    ad.datadoghq.com/service.init_configs: '[{}]'
    ad.datadoghq.com/service.instances: |
      [
        {
          "dbm": true,
          "host": "<HOSTNAME>,<SQL_PORT>",
          "username": "datadog",
          "password": "ENC[datadog_user_database_password]",
          "connector": "odbc",
          "driver": "ODBC Driver 18 for SQL Server",
          "tags": ["service:<CUSTOM_SERVICE>", "env:<CUSTOM_ENV>"],  # オプション
          "azure": {
            "deployment_type": "<DEPLOYMENT_TYPE>",
            "fully_qualified_domain_name": "<AZURE_ENDPOINT_ADDRESS>"
          }
        }
      ]
spec:
  ports:
  - port: 1433
    protocol: TCP
    targetPort: 1433
    name: sqlserver
```

`deployment_type` と `name` フィールドの設定に関する追加情報は、[SQL Server インテグレーション仕様][4]を参照してください。

Cluster Agent は自動的にこのコンフィギュレーションを登録し、SQL Server チェックを開始します。

`datadog` ユーザーのパスワードをプレーンテキストで公開しないよう、Agent の[シークレット管理パッケージ][5]を使用し、`ENC[]` 構文を使ってパスワードを宣言します。


[1]: /ja/agent/cluster_agent
[2]: /ja/agent/cluster_agent/clusterchecks/
[3]: /ja/containers/kubernetes/installation/?tab=helm#installation
[4]: https://github.com/DataDog/integrations-core/blob/master/sqlserver/assets/configuration/spec.yaml#L353-L383
[5]: /ja/agent/configuration/secrets-management
{{% /tab %}}
{{< /tabs >}}

## Agent の構成例
{{% dbm-sqlserver-agent-config-examples %}}

## Azure インテグレーションをインストールする

Azure からより包括的なデータベースメトリクスとログを収集するには、[Azure インテグレーション][1]をインストールします。

## 参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ja/integrations/azure