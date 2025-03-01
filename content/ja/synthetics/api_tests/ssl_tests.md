---
algolia:
  category: Documentation
  rank: 70
  subcategory: Synthetic API テスト
  tags:
  - ssl
  - ssl テスト
  - ssl テスト
aliases:
- /ja/synthetics/ssl_test
- /ja/synthetics/ssl_check
description: 世界中のロケーションから SSL 証明書を監視します
further_reading:
- link: https://www.datadoghq.com/blog/introducing-synthetic-monitoring/
  tag: ブログ
  text: Datadog Synthetic モニタリングの紹介
- link: /getting_started/synthetics/api_test
  tag: Documentation
  text: API テストの概要
- link: /synthetics/private_locations
  tag: Documentation
  text: 内部ホストで SSL テストを実行する
- link: /synthetics/guide/synthetic-test-monitors
  tag: ドキュメント
  text: Synthetic テストモニターについて
title: SSL テスト
---

## 概要

SSL テストを使用すると、SSL/TLS 証明書の有効性と有効期限をプロアクティブに監視して、主要なサービスとユーザー間の安全な接続を確保できます。証明書の有効期限が近づいているか、侵害された場合、Datadog は失敗の詳細を含むアラートを送信し、問題の根本原因をすばやく特定、修正できるようにします。

SSL テストは、ネットワークの外部または内部からのテストの実行の好みに応じて、[管理ロケーション](#select-locations)と[プライベートロケーション][1]の両方から実行することができます。SSL テストは、スケジュール、オンデマンド、または [CI/CD パイプライン][2]内で直接実行することができます。

## 構成

You may create a test using one of the following options:

- **Create a test from a template**:

     1. Hover over one of the pre-populated templates and click **View Template**. This opens a side panel displaying pre-populated configuration information, including: Test Details, Request Details, Assertions, Alert Conditions, and Monitor Settings.
     2. Click **+Create Test** to open the **Define Request** page, where you can review and edit the pre-populated configuration options. The fields presented are identical to those available when creating a test from scratch.
     3. Click **Save Details** to submit your API test.<br /><br>
        {{< img src="getting_started/synthetics/synthetics_templates_api_video.mp4" alt="Video of Synthetics API test landing page with templates" video="true" >}}

- **Build a test from scratch**:

   1. テストを一から作成するには、**+ Start from scratch** テンプレートをクリックし、SSL リクエストタイプを選択します。
   1. テストを実行する **Host** と **Port** を指定します。デフォルトの SSL ポートは `443` です。
   1. **Advanced Options** (オプション) をテストに追加します。
      * **Accept self-signed certificates**: 自己署名証明書に関連するサーバーエラーをバイパスします。
      * **Fail on revoked certificate in stapled OCSP**: 証明書が OCSP ステープリングによって取り消されたとラベル付けされた場合、テストに失敗します。
      * **Timeout**: テストがタイムアウトするまでの時間を秒単位で指定します。
      * **Server Name**: TLS ハンドシェイクを開始するサーバーを指定し、サーバーが同じ IP アドレスと TCP ポート番号上の複数の可能な証明書のうちの 1 つを提示することを可能にします。デフォルトでは、このパラメータは **Host** の値で埋められています。
      * **Client certificate**: クライアント証明書 (`.crt`) と `PEM` 形式の関連する秘密キー (`.key`) をアップロードして、mTLS を介して認証します。

      `openssl` ライブラリを使用して、証明書を変換することができます。例えば、`PKCS12` 形式の証明書を `PEM` 形式の秘密キーや証明書に変換することができます。

      ```
      openssl pkcs12 -in <CERT>.p12 -out <CERT_KEY>.key -nodes -nocerts
      openssl pkcs12 -in <CERT>.p12 -out <CERT>.cert -nokeys
      ```

   1. SSL テストに**名前**を付けます。

   1. SSL テストに Environment **タグ**とその他のタグを追加します。次に、これらのタグを使用して、[Synthetic Monitoring & Continuous Testing ページ][3]で Synthetic テストをフィルタリングできます。
   1. **Test Certificate** をクリックして、リクエストの構成をテストします。画面の右側に応答プレビューが表示されます。<br /><br>

      {{< img src="synthetics/api_tests/synthetics_ssl_test_cert.png" alt="SSL リクエストを定義する" style="width:90%;" >}}

   1. Click **Create Test** to submit your API test.



### スニペット

{{% synthetics-api-tests-snippets %}}

### アサーションを定義する

アサーションは、期待されるテスト結果が何であるかを定義します。**Test URL** をクリックした後、証明書の有効性、有効期限データ、TLS バージョン、`response time` の基本的なアサーションが、取得された応答に基づいて追加されます。テストで監視するには、少なくとも 1 つのアサーションを定義する必要があります。

| タイプ                  | 演算子                                                                               | 値の型                 |
|-----------------------|----------------------------------------------------------------------------------------|----------------------------|
| 証明書           | `expires in more than`、`expires in less than`                                         | _整数 (日数)_ |
| プロパティ              | `contains`、`does not contain`、`is`、`is not`、<br> `matches`、`does not match`       | _文字列_ <br> _[正規表現][4]_ |
| response time         | `is less than`                                                                         | 整数 (ms)             |
| TLS 最大バージョン   | `is less than`、`is less than or equal`、`is`、`is more than`、`is more than or equal` | _Decimal_                  |
| TLS 最小バージョン   | `is more than`、`is more than or equal`                                                | _Decimal_                  |

**New Assertion** をクリックするか、応答プレビューを直接クリックすることで、API テストごとに最大 20 個のアサーションを作成できます。

{{< img src="synthetics/api_tests/assertions_ssl.png" alt="SSL テストが成功または失敗するためのアサーションを定義する" style="width:90%;" >}}

アサーションで `OR` ロジックを実行するには、`matches regex` あるいは `does not match regex` コンパレータを使用して、`(0|100)` のように同じアサーションタイプに対して複数の期待値を設定した正規表現を定義します。プロパティアサーションの値が 0 あるいは 100 の場合、テストは成功です。

テストがレスポンス本文にアサーションを含まない場合、本文のペイロードはドロップし、Synthetics Worker で設定されたタイムアウト制限内でリクエストに関連するレスポンスタイムを返します。

テストがレスポンス本文に対するアサーションを含み、タイムアウトの制限に達した場合、`Assertions on the body/response cannot be run beyond this limit` というエラーが表示されます。

### ロケーションを選択する

SSL テストを実行する**ロケーション**を選択します。SSL テストは、ネットワークの外部または内部のどちらから証明書を監視するかの好みによって、管理ロケーションと[プライベートロケーション][1]の両方から実行できます。

{{% managed-locations %}}

### テストの頻度を指定する

SSL テストは次の頻度で実行できます。

* **On a schedule**: SSL/TLS 証明書が常に有効であり、主要なサービスのユーザーへの安全な接続が確保されるようにします。Datadog で SSL テストを実行する頻度を選択します。
* [**Within your CI/CD pipelines**][2]。
* **On-demand**: チームにとって最も意味のあるときにいつでもテストを実行します。

{{% synthetics-alerting-monitoring %}}

{{% synthetics-variables %}}

### 変数を使用する

SSL テストの URL、高度なオプション、アサーションで、[**Settings** ページで定義されたグローバル変数][9]を使用することができます。

変数のリストを表示するには、目的のフィールドに `{{` と入力します。

## テストの失敗

テストが 1 つ以上のアサーションを満たさない場合、またはリクエストが時期尚早に失敗した場合、テストは `FAILED` と見なされます。場合によっては、エンドポイントに対してアサーションをテストすることなくテストが実際に失敗することがあります。

これらの理由には以下が含まれます。

`CONNRESET`
: 接続がリモートサーバーによって突然閉じられました。Web サーバーにエラーが発生した、応答中にシステムが停止した、Web サーバーへの接続が失われた、などの原因が考えられます。

`DNS`
: テスト URL に対応する DNS エントリが見つかりませんでした。原因としては、テスト URL の誤構成や DNS エントリの誤構成が考えられます。

`INVALID_REQUEST`
: テストのコンフィギュレーションが無効です (URL に入力ミスがあるなど)。

`SSL`
: SSL 接続を実行できませんでした。[詳細については、個別のエラーページを参照してください][10]。

`TIMEOUT`
: リクエストを一定時間内に完了できなかったことを示します。`TIMEOUT` には 2 種類あります。
  - `TIMEOUT: The request couldn't be completed in a reasonable time.` は、リクエストの持続時間がテスト定義のタイムアウト (デフォルトは 60 秒に設定されています) に当たったことを示します。
  各リクエストについて、ネットワークウォーターフォールに表示されるのは、リクエストの完了したステージのみです。例えば、`Total response time` だけが表示されている場合、DNS の解決中にタイムアウトが発生したことになります。
  - `TIMEOUT: Overall test execution couldn't be completed in a reasonable time.`  は、テスト時間 (リクエスト＋アサーション) が最大時間 (60.5s) に達したことを示しています。

## 権限

デフォルトでは、[Datadog 管理者および Datadog 標準ロール][11]を持つユーザーのみが、Synthetic SSL テストを作成、編集、削除できます。Synthetic SSL テストの作成、編集、削除アクセスを取得するには、ユーザーをこれら 2 つの[デフォルトのロール][11]のいずれかにアップグレードします。

[カスタムロール機能][12]を使用している場合は、`synthetics_read` および `synthetics_write` 権限を含むカスタムロールにユーザーを追加します。

### アクセス制限

{{% synthetics_grace_permissions %}}

## その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ja/synthetics/private_locations
[2]: /ja/synthetics/cicd_integrations
[3]: /ja/synthetics/search/#search
[4]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions
[5]: /ja/monitors/notify/#configure-notifications-and-automations
[6]: https://www.markdownguide.org/basic-syntax/
[7]: /ja/monitors/notify/?tab=is_recoveryis_alert_recovery#conditional-variables
[8]: /ja/synthetics/guide/synthetic-test-monitors
[9]: /ja/synthetics/settings/#global-variables
[10]: /ja/synthetics/api_tests/errors/#ssl-errors
[11]: /ja/account_management/rbac/
[12]: /ja/account_management/rbac#custom-roles