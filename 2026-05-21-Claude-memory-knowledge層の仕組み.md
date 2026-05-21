# Claude memory / knowledge層の仕組み

> created: 2026-05-21
> tags: #気づき #Claude運用 #memory #設計

## 話題

Marukin配布運用設計の議論の流れで、Claudeのmemoryの中身を覗き見したら「あまり多く書かれてない」印象だった。そこから「memoryって拡張できないの?」「構造化されたmemoryがいちばんいいよね?」と気になって、memory層・knowledge層がそれぞれどう管理されてるのかを整理した。Marukin設計から少し離れた、汎用的な仕組みの理解メモ。

## 結論・方針

### memory層 — 拡張するものじゃなく、薄く保つもの

- Claude.aiのmemoryが薄いのは仕様。**圧縮された要約**として保持される。生ログじゃない
- 容量を増やす設定はない。手動で項目追加・編集はできるけど、根本が「核だけ持つ」思想
- なぜ薄く保つか = **常駐コスト**。memory/CLAUDE.md/Project knowledgeはコンテキスト窓に常駐して毎ターン消費される。5,000トークンのCLAUDE.mdは2メッセージでも200メッセージでも毎ターン5,000トークン食う
- 「拡張したい情報」は memory じゃなく knowledge レイヤーに置くのが設計思想

### 「構造化memory」の正しい解釈

「構造化されたmemoryがいちばんいい」は半分正しくて半分ズレてた。

- ❌ memory自体を階層化して大きく持つ → 常駐コスト膨らむ、junk drawer化
- ✅ memoryは薄い核に保ち、構造化された詳細は別レイヤーに階層分離 → 必要時だけ都度ロード

常駐層(memory/knowledge)は薄く、都度ロード層(details/Connector/リンク先)は厚くてOK。これは自分がpartnerBrainでやってる「summary層は肥大化させない、詳細はdetails層に逃がす」と同じ。

### knowledge層 — Project単位、2モード自動切替

- knowledge層は基本「Project単位」管理。Project=ひとつの「箱」、箱にknowledge(資料)とproject memory(記憶)がぶら下がる
- 箱が違えば中身は完全に独立
- knowledge層はサイズで2モードを**自動で**行き来する:
  - **in-context**: 小さいうちはファイル全部をコンテキストに載せる(常駐、毎ターン消費、確実に全部読まれる)
  - **RAG**: context window限界に近づくと自動で検索モードに切替。容量を最大10倍に拡張、検索ヒット分だけ取得。閾値を下回ると自動でin-contextに戻る
- 切替閾値: 公式は「context window限界」、実報告では「約13ファイル」説もある(確度低め、bug reportレベル)
- AnthropicのContextual Retrieverは、検索チャンクに文脈情報を付加してから使う。retrieval error 67%削減との主張

### 「箱の中/外」の整理

- 箱の中 = knowledge、project memory(Project単位)
- 箱の外 = global memory(ユーザー単位)、Connector(アカウント単位)

### データの3分類(これが丸く収まる最終形)

> 変わらない事実は knowledge、変わり続けるデータは Connector、人のことは memory。

- **knowledge** = 変わらない事実(会社概要、組織図、業務ルール)→ 静的
- **Connector** = 変わり続けるデータ(議事録、在庫、予定、メール)→ 動的
- **memory** = 人についての理解(誰が何担当か、好み、過去のやりとりの文脈)→ 更新頻度では測れない第3カテゴリ

最初「knowledge=静的 / Connector=動的」の2分法で考えてたけど、memoryを足して三角形にすると完成形。

### Connector(動的データの置き場)の選定 — MarukinはDrive+SFで閉じる

Drive以外の代替を調査した結果、**Marukinでは新サービスを足さないのが正解**。

- 代替候補: Notion(構造化DB・読み書き可)、Microsoft 365(読み取り専用)、Box/Dropbox/Confluence等
- Notionには確かに構造化の優位性がある(ファイルベースのDriveより整理しやすい)
- でもMarukinは「SFをマスタ」「Google Workspace中心」が既定路線。構造化動的データはSF、ファイル系動的データはDriveで既にカバー済み
- Notionを足すとSFと役割が被る → 採用しない

## 蹴った選択肢

- **memoryを構造化して大きく持つ**: 常駐コストが膨らむ。memoryは薄い核に保つのが正解
- **Notionを動的データ層に採用**: 構造化の優位性は本物。でもMarukinはSF+Driveで動的データを既にカバー済み、役割が被る。サービス追加のメンテコスト(ライセンス・学習・権限二重化)が優位性を上回る
- **Microsoft 365 connector**: 読み取り専用な上、Marukinの基盤(Google Workspace)と不一致

## 判断軸

- **常駐コスト vs 都度ロード**: コンテキスト窓に常駐して毎ターン消費されるものは薄く、必要時だけ読まれるものは厚くてOK。この線引きが全ての設計判断の根っこ
- **サービスを増やさないことが運用優位性**: 新サービス1つ = ライセンス・権限・学習・障害切り分けが全部増える。便利さより「置き場は少ないほどいい」。「正本1つ・経路は複数」と同じ思想で「置き場も少なく」
- メンテコスト・運用継続性を、個別機能の便利さより優先

## モヤモヤ・揺れ

- knowledge層のin-context⇄RAG切替閾値が曖昧。「約13ファイル」説はbug reportレベルで確度低い。Marukin配布の初期4ファイルなら確実にin-contextだけど、増やしたとき何ファイルで切り替わるかは実機で見たい
- memoryの手動編集UIは見たけど、Project memoryの蓄積挙動(何が自動で記憶されるか)はまだ実機で観察できてない

## 気づき・ツッコミ

- memoryを「もっと書けるようにしたい」と思った時点で発想がズレてた。memoryは「拡張するもの」じゃなく「薄く保つもの」。覗いて薄いと感じたのは仕様通りで、むしろ正しい状態だった
- 「構造化memory」を直感で求めてたけど、答えは「memoryを構造化する」じゃなく「薄い核+別レイヤーに階層分離」。これ自分がpartnerBrainで既にやってること。業界のCLAUDE.mdベストプラクティス(lean に保て/lookup tableのように/modular hierarchical)も同じことを言ってる。直感は信じていい、業界が後追いしてる構図
- knowledge層が「小さいと全ロード、大きいとRAG」を勝手に切り替えてるのは知らなかった。memoryが「常に圧縮要約」なのと設計思想が違う。memory=核を常に持つ、knowledge=量に応じて処理を最適化する
- 「優位性のあるサービスを確認したい」と思って調べたら、結論が「サービスを増やさないのが優位性」という逆説になった。便利な機能を見つけると足したくなるけど、足すコストの方を見る癖は大事

## 関連リンク候補

- [[2026-05-18-Marukin-Claude配布運用設計]] - この仕組み理解を使った具体設計
- [[partnerBrain/_index]] - summary/details の2層構造がまさにこの「薄い核+階層分離」の実装
