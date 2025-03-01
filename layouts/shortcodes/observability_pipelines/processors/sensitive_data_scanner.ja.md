Sensitive Data Scanner プロセッサはログをスキャンして、PII、PCI、カスタムの機密データなどの機密情報を検出し、マスキングまたはハッシュ化します。機密データをスキャンするには、当社のライブラリから定義済みのルールを選択するか、カスタムの正規表現ルールを入力できます。

Sensitive Data Scanner プロセッサをセットアップするには
1. **フィルタークエリ**を定義します。指定した[フィルタークエリ](#filter-query-syntax)に一致するログのみがスキャンおよび処理されます。フィルタークエリに一致するかどうかにかかわらず、すべてのログはパイプラインの次のステップに送信されます。
1. **Add Scanning Rule** をクリックします。
1. スキャンルールに名前を付けます。
1. **Select scanning rule type** フィールドで、ライブラリからルールを作成するか、カスタムルールを作成するかを選択します。
    - ライブラリからルールを作成する場合は、使用するライブラリパターンを選択します。
    - カスタムルールを作成する場合は、データに対して確認する正規表現パターンを入力します。
1. **Scan entire or part of event** セクションで、ドロップダウンメニューの **Entire Event** (イベント全体)、**Specific Attributes** (特定の属性)、**Exclude Attributes** (属性の除外) からスキャン対象を選択します。
    - **Specific Attributes** (特定の属性) を選択した場合は、**Add Field** をクリックし、スキャンする特定の属性を入力します。最大 3 つのフィールドを追加できます。ネストされたキーにアクセスするには、パス記法 (`outer_key.inner_key`) を使用します。ネストされたデータを持つ特定の属性では、すべてのネストされたデータがスキャンされます。
    - **Exclude Attributes** (属性の除外) を選択した場合は、**Add Field** をクリックし、スキャンから除外する特定の属性を入力します。最大 3 つのフィールドを追加できます。ネストされたキーにアクセスするには、パス記法 (`outer_key.inner_key`) を使用します。ネストされたデータを持つ指定された属性については、すべてのネストされたデータが除外されます。
1. **Define action on match** セクションで、一致した情報に対して実行するアクションを選択します。**注**: マスキング、部分的なマスキング、およびハッシュ化はすべて元に戻せないアクションです。
    - 情報をマスキングする場合は、一致したデータを置き換えるテキストを指定します。
    - 情報を部分的にマスキングする場合は、マスキングする文字数を指定し、部分的なマスキングを一致したデータの先頭または末尾に適用するかどうかを指定します。
    - **注**: ハッシュ化を選択した場合、一致した UTF-8 バイトは FarmHash の 64 ビットフィンガープリントでハッシュ化されます。
1. オプションとして、正規表現に一致するすべてのイベントにタグを追加し、イベントのフィルタリング、分析、アラートを行うことができます。