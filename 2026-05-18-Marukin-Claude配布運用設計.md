# Marukin Claude配布運用設計

> created: 2026-05-18
> tags: #判断 #Marukin #PoC #Claude運用 #設計

## 話題

Claude memory + Obsidian運用の最新動向調査から始まって、Marukin内でのClaude配布運用設計の話に発展した。GoogleDrive経由でVault配布してた現状が「我慢できなくはない」レベルだけど、長期的には厳しい。配布相手のUX(扱いづらさ・応答速度)が問題になってきてる。

最初は「partnerBrain運用の改善」として考えてたけど、議論を進めるうちに**Kaina個人運用とMarukin配布運用は本質的に別物**だと気づいて、分離する設計に着地した。

## 全体図

![[2026-05-18-Marukin配布運用アーキテクチャ.svg]]

(2026-05-21時点の改訂版。データレイヤーを更新頻度で4層に分け、Salesforce将来層を加えた)

## 結論・方針

### 個人運用と配布運用を分離する

```
【Kaina個人運用】(現状維持)
- Vault + git + summary/details 2層 + 自律更新試行
- AIと自分の判断軸ズレ観察も継続
- 思想・運用とも今のまま

【Marukin配布運用】(新設計、C案)
- Claude Team plan(契約済み、Connector有効化済み)
- Project: Marukin-Support
- 配布相手は Claude.ai/Desktop/Mobile 内で完結
- ファイル/Vault概念は配布相手に隠蔽
```

### 配布設計の幹(C案: Claude完結型)

基盤:
- Claude Team plan
- Project "Marukin-Support"

データレイヤー:
- Project knowledge: 初期4ファイル(`marukin_overview` / `marukin_org` / `marukin_rules` / `claude_role`)
- Project memory: 自動蓄積+Kaina(管理者)剪定
- Drive Connector(Remote): 業務文書・最新資料

自動化:
- Cloud Routines(Anthropicクラウド実行、Desktop起動不要)

通知配信の多重化:
- 主: Claude モバイル/Desktopプッシュ
- 保険(外勤Pixelユーザー): Googleカレンダーイベント+通知時刻
- 保険(全員): Email通知

配布相手の使い分け:
- 内勤者: Claude Desktop主、モバイル補助
- 外勤者(営業): Claude モバイル主、音声入力推奨、Pixelカレンダー通知に乗せる

### Phase別ロードマップ

| Phase | 期間 | 内容 |
|---|---|---|
| 0 | 1週間 | Project作成、初期4ファイル作成、Calendar連動テスト、Cloud Routine試作 |
| 1 | 2週間 | Kaina個人検証、memory蓄積観察、配布相手向けガイド作成 |
| 2 | 1ヶ月 | 配布相手2人招待(内勤1+外勤1が望ましい)、フィードバック収集 |
| 3 | 2ヶ月以降 | 全社展開判断、Tasks/外部連携拡張検討 |

## 蹴った選択肢

### A案: Claude入力 + Chat出力(動線分離型)
- 配布相手はClaude.aiで質問、結果がGoogle Chatに投稿
- Kainaが「プラットフォーム分かれちゃう」と指摘 → 却下
- 入出力の動線が分かれて統一感ない

### B案: Google Chat内完結bot
- Claude API + Google Chat bot API で完全Chat内完結
- 工数大、Marukin情シスにNode.js/Python実装スキル必要
- **Claude.aiのProject knowledge/memoryに直接アクセスできない問題**
- Files API + Memory Tool で同等機能は作れるが自前実装
- 2026/6/15 のAgent SDK Team plan統合まで待てば課金は楽になる
- Phase 3以降に再検討、PoCでは投資回収できない

### D案: ハイブリッド(Drive共通化、双方向)
- Claude.ai主体 + Chat MCP併用
- データはDrive集約してC/B両対応の素地を作る
- 設計としては優秀だが、Kainaの本質要件「窓口一本化」と一致しない
- 段階移行の橋渡しとして候補に残るが、Phase 1では採用せず

### Vault概念を配布側にも残す案
- partnerBrainの2層構造(summary/details)を配布側でも維持
- 「partnerBrain自律更新運用」を配布側でも要件化
- → よく考えたら**自律更新は手段で、本来の要件は「社員リソース消費ゼロ」**だった
- 要件を再定義したらVault概念ごと配布側で捨てて良かった

### Claude for Small Business
- 当初「これが解かも」と挙げたが、別契約じゃなくCowork内トグル機能だった
- 中身は決算・営業ワークフロー15個のテンプレ集
- PKM/Vault運用とは方向性が違うので却下

## 判断軸

### 本質要件の再定義が決め手だった

最初「partnerBrainを配布側でも動かす」前提で議論してたけど、**Kainaの本質要件は別の場所にあった**:

1. **「社員のメンテリソース不要」**: 配布相手が「AIを自ら教育する」習慣がない、ここにリソース割いて欲しくない
2. **「窓口の一本化」**: Claudeとの会話・通知・タスクが1つのアプリ内で完結してほしい
3. **「保持できているならそれでいい」**: partnerBrain中身の可視性は社員には不要

この3つが揃った時点で、**ファイル管理思想を配布側で捨てる選択肢が現実的**になった。Vault概念を配布側に残そうとすると、要件と合わない複雑性を背負うことになる。

### メンテコスト・運用継続性を優先

複雑な設計(D案ハイブリッド、B案bot)は機能豊富だけど、メンテコストが純増する。Kainaの思考スタイル「メンテコスト重視」と真逆になる。シンプルな設計(C案)で「窓口一本化」が満たせるなら、それが正解。

### 段階移行を可能にする構造

データソースを Drive集約 する設計思想だけは残した。これでPhase 3以降にB案やTasks連携に展開する余地を残しつつ、Phase 1-2はC案でシンプルに走れる。

## モヤモヤ・揺れ

### Project memoryの実機挙動が読めない
- Anthropicドキュメントだけでは判断つかない点が複数残ってる
- Project memoryは「メンバー別」か「Project全体共有」か未確認
- memory蓄積がノイズ混入したときのcuration挙動が読めない
- Phase 1で実機検証必須

### Drive Connector検索精度の不安
- 日本語Markdownでの検索精度が報告されてない領域
- Phase 0でテスト必須
- 期待外れた場合は Project knowledge に統合する案にフォールバック

### B案を将来本当にやるかの不確実性
- Agent SDK 6/15統合後、課金は楽になる
- でも配布相手の本質要件は「窓口一本化」=C案で満たせてる
- B案投資する動機が後から本当に出てくるかは未知数
- Phase 2のフィードバック次第

### 配布相手のClaudeアプリ習慣化コスト
- 配布相手にとって新ツール導入の認知負荷はある
- Kainaのサポート工数が初期に重め
- 「サポートエージェント」としての価値が体感できるまでの離脱リスク

## 気づき・ツッコミ

### 「ファイル管理発想の罠」
最初「Project knowledgeに入れる」「Drive Connectorで読む」を同じレイヤーで話してて、Kainaに「変わらない気がする」とツッコミ入った。実は**Claudeへのコンテキスト供給メカニズムが根本的に違う**(Just-in-time vs Pre-loaded)んだけど、「保管場所はDrive」の表面で見ると同じに見える。説明する側がファイル管理レイヤーで止まってると伝わらない。

### 「セッション毎リセット」イメージは1年半遅れ
Kainaの認識が2024年で止まってた。Memory機能は2026/3に無料化、24h自動コンテキスト生成、Project別memory分離が標準装備されてる。memoryの実機UI見てもらったら「シンプル」と納得した。AI業界の動きが早すぎてユーザの認識がついていきにくい問題。

### Pixelユーザーは設計的に恵まれてる
配布相手全員Google Pixelで、Google Workspace濃度高い。これは「Anthropic公式Connector + Google公式MCP + Android標準通知」の3層が綺麗に連動する。iPhoneだとClaude→iOSリマインダー直接連動できるけど、Androidは不可。一見iPhone優位だけど、**Pixelは Google Calendar経由でAndroid標準通知が確実**に飛ぶので、運転中・ハンズフリーでも届く。営業さんのユースケースには Pixel + Calendar の組み合わせの方が強い場面ある。

### 「自律更新」は手段で目的じゃなかった
partnerBrainの自律更新運用を配布側でも維持しようとしてたけど、目的は「社員リソース消費ゼロ」。これに気づくと、自律更新じゃなくProject memory(会話から自動学習)+Kaina剪定で要件を満たせる。手段を目的化すると複雑性が膨らむ典型例。

### 業界動向は派手に見えてKainaの方が先取りしてた
今回調査で「Claude Dreaming」「obsidian-second-brain」「Agentic Knowledge Management」とか派手なトレンド出てきたけど、KainaがpartnerBrainでやってる「2層構造+自律更新+定期consolidation」は、業界が後追いで形式化してる構造そのもの。AIユーザー界隈の表層トレンドより、Kainaの直感運用の方が先に動いてた。

### 「Claude for Small Business」勘違い事件
最初これを別契約と思って案2に組み込んで提案したけど、実はCowork内トグル機能だった。情報源にあたって確認したら別物。**派手な名前の機能ほど中身を確認する**の教訓。

## 関連リンク候補

- [[2026-05-07-日報PoC設計-AppSheet軸転換とChat配信設計]] - **本ノートの前提・親ノート**
- [[partnerBrain/_index]]
- [[partnerBrain/details/marukin_deep]] - PoC配布側設計確定の追記反映済み
- [[2026-XX-XX-HR評価PoC]](今後作成予定)

## 文脈補足

本ノートは2026-05-07の日報PoC設計の「Claude側具体設計」を確定させたもの。
2026-05-07時点で「AppSheet/Vault軸+Cowork配信」戦略転換は決まってた。
今日はそのCowork配信側の具体実装(Project構造、通知配信、Phase別ロードマップ)を詰めた。

## 次のアクション

1. partnerBrain側に「PoC Vault運用は別思想で動かす」方針追記(`details/marukin_deep.md`)
2. Phase 0: Marukin-Support Project作成 + 初期4ファイル書き起こし
3. Phase 0: Google Calendar連動の動作確認(イベント作成・通知時刻設定)
4. Phase 1: Kaina個人で memory蓄積挙動の実機検証
5. 配布相手2人(内勤+外勤の組み合わせ推奨)のヒアリング先行

---

## 2026-05-21追記: データレイヤーとSalesforce将来設計

3日空けて続き。データレイヤーの中身を詰めた。あと将来のSalesforce連携、GoogleDrive構造まで。

### データレイヤーは更新メカニズムが全然違う

「データレイヤー」って一括りにしてたけど、3つ(実質4つ)で更新の動き方が全然違う。混ぜて考えると事故る。

- **準静的層**(Project knowledge): 手動更新。年単位で変わらないもの専用。会社概要・組織図・ルール
- **中頻度動的層**(Drive Connector): Drive側を更新したら即反映。議事録・案件状況
- **文脈層**(Project memory): 会話から自動蓄積+Kaina剪定。人・関係性
- **リアルタイム層**(Workspace Connector): API直結で常に最新。カレンダー・メール

判断基準はシンプルで「Claudeが手動更新を待てる頻度か?」。待てる→knowledge、待てない→Connector。

**動的データをProject knowledgeに固定するのはアンチパターン**。手動更新が追いつかなくてstale化する。memory staleの話と同根。Project knowledgeは「めったに変わらないもの」専用と割り切る。

### Project knowledge同期問題 → Google Docs形式で解決

.mdファイルをアップロードすると静的(手動再アップロード必要)。でも**Google Docs形式でProjectに追加するとDrive側更新が同期される**。だから初期4ファイルはGoogle Docs形式で作るのが正解。配布相手のGoogle Workspace環境とも親和性高い。

### Salesforce将来設計

Marukinのマスタは基幹刷新後にSalesforceが正になる予定。これが効いてくる。

- **在庫・受発注**: SF MCP(2026/4にGAで読み書き対応、書き込み前に承認ゲートあり)で参照。マスタがSFなら設計シンプル
- **日報のSF化**: 「入力UIをSFにする」じゃなく「**SFを正本に、入力UIは複数持つ**」が正解
  - AppSheetフロント+SFバックエンド(従来の全社員、浸透資産そのまま活かせる)
  - Claude音声入力→SF書き込み(配布相手・営業、話すだけで日報がSFに溜まる)
  - この2つは排他じゃなく両立。全社員の日報がSFに集約される
- B案(SF直接入力)は罠。AppSheet浸透資産を捨てることになる。蹴った

#### 配布相手とSFのアクセスは複線

配布相手がSFを使うとき、「Claude基盤を介す」か「直接操作」かは二択じゃなく複線。用途で使い分ける。

- **Claude経由**: 曖昧・自然言語・分析・音声入力に向く。配布相手がSFの操作方法を知らなくていい(リソース消費ゼロ要件と整合)
- **直接操作(SF UI / AppSheet)**: 定型・大量・正確なトランザクションに向く。承認ゲートや解釈の揺れが邪魔になる定型業務はこっち
- **原則**: 正本がSFである限り経路は何本でもいい。「正本を1つ、経路は複数」がMarukin配布設計の一貫思想(入力UI複数・通知多重化と同じ構造)
- 経路を1本に絞ると損する: 全部Claude経由→定型大量入力が遅い/ボトルネック化、全部直接→自然言語の旨味が消える
- **第3の選択肢: Agentforce**(SF内蔵AI、コーディングモデルにClaude Sonnet採用)。SF本格運用になった時、Marukin-Support ClaudeとAgentforceの役割分担が論点になる。今決める話じゃない

![[2026-05-18-Marukin-SF将来アクセス図.svg]]

### 基幹刷新は「AI-ready設計」の好機

普通AIを後付けするとデータ構造が読みにくい問題に当たる。でもMarukinは基幹刷新でSFオブジェクトを新設計する。**「Claudeが読みやすい構造」を最初から織り込める**。基幹刷新の設計レビュー観点に「Claude連携しやすいか」を1項目入れておく価値がある。Kainaが刷新推進側だから能動的に効かせられるポイント。

### GoogleDrive構造

```
[共有Drive] Marukin-Claude-PoC
├── 01_knowledge/   準静的層(Project knowledgeへ投入、Google Docs形式)
├── 02_reference/   動的層(Drive Connector参照。議事録/業務マニュアル/案件メモ)
├── 03_output/      Claude生成物(briefing/日報ドラフト)
└── 90_admin/       Kaina専用(memory剪定ログ/設計ドキュメント)
```

- 共有Drive推奨(組織所有、退職者影響なし、権限管理が楽)
- 階層は2階層まで(Drive Connector検索が深い階層に弱い可能性)
- 個人raw領域は`02_reference/個人/{名前}/`でアクセス権分離、Kainaは全員分見える
- **Salesforceはこの構造の外**。SF MCPで直結する別系統。Driveに`04_salesforce`を作りたくなるけど罠、二重管理が始まる

### モヤモヤ・揺れ(追記分)

- Project knowledge同期は「Google Docs形式なら効く」が、調査ベース。実機で同期挙動を確認したい
- Salesforceライセンスコストが読めない。AppSheet経由なら安価なPlatformライセンスで済むか要見積もり
- 日報SF化のオブジェクト設計工数、AppSheetデータソース切り替え工数が未見積もり

### 気づき・ツッコミ(追記分)

- 「データレイヤー」を1つの箱だと思ってたのが甘かった。更新頻度で割ると配置先が自動的に決まる。抽象を1段おろすと迷いが消える典型
- 「日報をSFベースで入力」と最初考えたけど、よく聞くと欲しいのは「データがSFに集まること」で「入力UIがSFであること」じゃない。**入力UIとデータ正本を分けて考える**と、AppSheet浸透資産を捨てずに済む。partnerBrain自律更新のとき「自律更新は手段、目的は社員リソースゼロ」と気づいたのと同じ構造のほぐし方
- Phase 1-2は基幹刷新と独立して走れる。SF統合(Phase 3-4)だけが刷新完了に依存。依存関係を切り分けておくと「刷新が遅れてもPoCは止まらない」

### 関連: memory/knowledge層の仕組み

データレイヤーの設計根拠になっているClaude側の仕組み(memory層・knowledge層の管理、データ3分類、Connector選定)は別ノートに整理した: [[2026-05-21-Claude-memory-knowledge層の仕組み]]

### 次のアクション(更新)

1. Phase 0: Marukin-Support Project作成 + 初期4ファイル(Google Docs形式)書き起こし
2. Phase 0: 共有Drive「Marukin-Claude-PoC」作成、4フォルダ構造でセットアップ
3. Phase 0: Google Calendar連動の動作確認
4. Phase 1: Kaina個人で memory蓄積挙動+Drive Connector検索精度の実機検証
5. 配布相手2人(内勤+外勤)のヒアリング先行
6. (Phase 3以降)基幹刷新のSFオブジェクト設計に「Claude連携観点」を入れる
