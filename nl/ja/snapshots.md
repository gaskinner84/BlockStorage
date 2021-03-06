---

copyright:
  years: 2014, 2019
lastupdated: "2019-02-05"

keywords:

subcollection: BlockStorage

---
{:new_window: target="_blank"}
{:tip: .tip}
{:note: .note}
{:important: .important}

# スナップショット
{: #snapshots}

スナップショットは {{site.data.keyword.blockstoragefull}} の機能の 1 つです。 スナップショットは、特定の時点におけるボリュームの内容を表しています。 スナップショットを使用すると、パフォーマンスに影響を与えることなく、スペース消費量を最小限に抑えて、データを保護できます。 スナップショットは、データ保護のための第 1 の防衛線として考えられます。 ユーザーが、ボリュームにある重要なデータを誤って変更したり、削除したりした場合でも、データをスナップショット・コピーから容易に、しかも迅速に復元することができます。

{{site.data.keyword.blockstorageshort}}には、スナップショットを取得する 2 つの方法が用意されています。

* 1 つ目は、ストレージ・ボリュームごとにスナップショット・コピーを自動的に作成および削除する構成可能なスナップショット・スケジュールする方法です。 要件に合わせて、追加のスナップショット・スケジュールの作成、コピーの手動削除、およびスケジュールの管理もできます。
* もう 1 つの方法は、手動でスナップショットを取得することです。

スナップショット・コピーは、ある時点でのボリュームの状態を収集した、{{site.data.keyword.blockstorageshort}} LUN の読み取り専用イメージです。 スナップショット・コピーは、コピーの作成に必要な時間とストレージ・スペースの両方において、効率的です。 {{site.data.keyword.blockstorageshort}} のスナップショット・コピーの作成には数秒しかかかりません。 ボリュームのサイズやストレージ上のアクティビティーのレベルに関係なく、通常は 1 秒未満で済みます。 スナップショット・コピーが作成された後、データ・オブジェクトへの変更は、まるでスナップショット・コピーが存在しないかのように、現行のオブジェクト・バージョンへの更新に反映されます。 その間、データのコピーは安定した状態のままです。

スナップショット・コピーでは、パフォーマンスが低下することはありません。 ユーザーは、1 つの{{site.data.keyword.blockstorageshort}}・ボリュームにつき最大 50 個のスケジュールされたスナップショットと 50 個の手動スナップショットを簡単に保管でき、それらのスナップショットはすべて、読み取り専用のオンライン・バージョンのデータとしてアクセス可能です。

スナップショットがあると、以下が可能になります。

- 特定時点のリカバリー・ポイントを中断なしに作成する。
- ボリュームを以前の特定時点に戻す。

スナップショットを作成するには、そのボリューム用に一定量のスナップショット・スペースを購入する必要があります。 スナップショット・スペースは、ボリュームの最初の注文時に追加することも、後で**「ボリュームの詳細」**から追加することもできます。 スケジュールされたスナップショットと手動スナップショットがスナップショット・スペースを共有するため、十分なスナップショット・スペースを注文してください。 詳しくは、[スナップショットの注文](/docs/infrastructure/BlockStorage?topic=BlockStorage-orderingsnapshots)を参照してください。

## スナップショットのベスト・プラクティス

スナップショットの設計は、お客様の環境によって異なります。 スナップショット・コピーを計画および実装するときは、以下の設計上の考慮事項が役立ちます。
- 各ボリュームまたは LUN について、最大 50 個のスナップショットをスケジュールによって作成でき、最大 50 個を手動で作成できます。
- 過剰にスナップショットを作成しないでください。 毎時、毎日、または毎週のスナップショットをスケジュールすることにより、スケジュールしたスナップショットの頻度で、お客様の RTO および RPO のニーズとアプリケーションのビジネス要件が確実に満たされるようにしてください。
- ストレージ消費量の増加を制御するために、スナップショット自動削除を使用できます。 <br/>

  自動削除しきい値は 95% に固定されています。
  {:note}

スナップショットは、実際のオフサイト災害復旧レプリケーションや長期保存バックアップに代わるものではありません。
{:important}

## セキュリティー

すべてのスナップショットおよび暗号化された {{site.data.keyword.blockstorageshort}} のレプリカも、デフォルトで暗号化されます。 この機能は、ボリューム単位でオフにすることはできません。 プロバイダー管理の保存データの暗号化について詳しくは、[データの保護](/docs/infrastructure/BlockStorage?topic=BlockStorage-encryption)を参照してください。

## スナップショットがディスク・スペースに与える影響

スナップショット・コピーは、ファイル全体ではなく個々のブロックを保存することでディスク使用量を最小化します。 スナップショット・コピーは、アクティブ・ファイル・システム内のファイルが変更または削除された場合にのみ、追加のスペースを使用します。

アクティブ・ファイル・システムでは、変更されたブロックはディスク上の別の場所に再書き込みされるか、アクティブ・ファイル・ブロックとして全体が削除されます。 ファイルが変更されたり削除されたりすると、元のファイル・ブロックは 1 つ以上のスナップショット・コピーの一部として保持されます。 その結果、元のブロックに使用されているディスク・スペースも、変更前のアクティブ・ファイル・システムの状況を表すために保持されます。 変更されたアクティブ・ファイル・システム内のブロックに使用されているディスク・スペースに加えて、このスペースも保持されます。

<table>
    <colgroup>
      <col style="width: 33.3%;"/>
      <col style="width: 33.3%;"/>
      <col style="width: 33.3%;"/>
    </colgroup>
      <tr>
        <th colspan="3" style="border: 0.0px;text-align: center;">スナップショット・コピーの前と後でのディスク・スペース使用状況</th>
     </tr><tr>
        <td style="border: 0.0px;text-align: center;"><img src="/images/bfcircle1.png" alt="スナップショット・コピー前"></td>
        <td style="border: 0.0px;text-align: center;"><img src="/images/bfcircle3.png" alt="スナップショット・コピー後"></td>
        <td style="border: 0.0px;text-align: center;"><img src="/images/bfcircle2.png" alt="スナップショット・コピー後の変更"></td>
     </tr><tr>
        <td style="border: 0.0px;">スナップショット・コピーが作成される前は、ディスク・スペースはアクティブ・ファイル・システムによってのみ使用されます。</td>
        <td style="border: 0.0px;">スナップショット・コピーが作成された後では、アクティブ・ファイル・システムとスナップショット・コピーは同じディスク・ブロックを指します。 スナップショット・コピーは追加ディスク・スペースを使用しません。</td>
        <td style="border: 0.0px;">アクティブ・ファイル・システムから <i>myfile.txt</i> が削除された後も、スナップショット・コピーは引き続きそのファイルを含んでおり、そのディスク・ブロックを参照します。 このため、アクティブ・ファイル・システム・データを削除しても、必ずしもディスク・スペースが解放されるわけではありません。</td>
      </tr>
</table>

スナップショット・スペースの使用法について詳しくは、[スナップショットの管理](/docs/infrastructure/BlockStorage?topic=BlockStorage-managingSnapshots)を参照してください。
