# myBrain ノート作成・更新ガイド (Project knowledge版)

このProjectは Kaina の Obsidian Vault「myBrain」に思考ログノートを追加・更新するためのもの。
iOS Claudeアプリ等、ローカルファイルアクセスができない環境からの利用を主眼に、**GitHub connector経由**でリポジトリを直接更新する。

PC環境(Claude Code / Cowork)では引き続きローカルの `obsidian` skillが使われる。本Projectはskill不可環境用の代替パス。

---

## リポジトリ情報

- リポジトリ: `KasumiKaina/myBrain` (Private)
- デフォルトブランチ: `main`
- ノート配置先: リポジトリ内 `myBrain/` ディレクトリ
- 注意: ルートではなく `myBrain/myBrain/{filename}.md` の階層
- `partnerBrain/` ディレクトリも**Vault側は更新可**(後述「partnerBrain Vault更新フロー」参照)。ただし claude.ai 側の「Claudeへの指示」(Custom Instructions) は API/connector で変更不可、ユーザの手動更新が必須

## 必要なConnector

- **GitHub connector** (claude.ai Settings > Connectors で接続済みであること)
- 最低限 `contents:write` 権限相当が必要(ファイル作成・更新ができればOK)
- 接続前提でこのProjectを使うこと。未接続の場合は先にユーザに接続を促す

---

## ノート化の発火条件

以下のいずれかに該当した場合、ノート化を**提案**する(自動生成はしない):

- 会話が一段落して結論や方針が出たとき
- 30分以上の長い議論の後
- ユーザが「ありがとう」「助かった」など終了サインを出したとき
- 複数の選択肢を比較検討して決定に至ったとき
- 「これ後で見返したいな」みたいな発言が出たとき

提案文の例:
> 「今日の話、判断ログとしてまとめとく?」
> 「ここまでの流れ、ノート化しとこうか?」

ユーザが明示的に「ノートして」「メモっといて」「コミットしといて」等を言った場合は提案フェーズをスキップして即実行。

---

## ノートフォーマット

```markdown
# {トピックを表す簡潔なタイトル}

> created: {YYYY-MM-DD}
> tags: #判断 #モヤモヤ #気づき など該当するもの

## 話題

{何を考えていたか、何を決めようとしていたか}

## 結論・方針

{決まったこと。決まってない場合は「保留」と書く}

## 蹴った選択肢

{比較して採用しなかった案と、その理由}

## 判断軸

{今回何を重視したか。Kainaの価値観が見える部分}

## モヤモヤ・揺れ

{完全に納得してない部分、後で見返したい疑問}

## 気づき・ツッコミ

{本筋から外れた小ネタ、面白がった視点、観察}
※ここが思考の癖の宝庫。無理に絞らず拾う

## 関連リンク候補

[[関連しそうなノート名]]
※既存ノートの確認はせず、Claude判断で候補を出す
```

---

## ファイル命名規則

`YYYY-MM-DD-{日本語トピック}.md`

- 日付プレフィックスは必ず付ける(時系列ソート用)
- トピック部分は日本語OK・簡潔に
- 使えない文字(`/ \ : * ? " < > |`)は避ける
- スペースは使わず、必要なら長音や中黒で区切る

例:
- `2026-04-29-obsidian導入にあたって.md`
- `2026-04-29-基幹システム比較メモ.md`

---

## GitHub操作手順

### 新規ノート作成

1. ノート本文・ファイル名・コミットメッセージを生成
2. GitHub connectorのファイル作成ツール(`create_or_update_file` 相当)を呼ぶ:
   - owner: `KasumiKaina`
   - repo: `myBrain`
   - path: `myBrain/{ファイル名}`
   - branch: `main`
   - message: コミットメッセージ(下記規則)
   - content: ノート本文
3. 成功したらユーザに報告。失敗時は無理にretryせず状況を伝えて判断を仰ぐ

### 既存ノート更新

1. 対象ノートを `get_file_content` / `search_files` 等で取得
2. 更新内容を反映した本文を生成
3. ファイル更新ツールでSHA付きでcommit

### コミットメッセージ規則

- 新規ノート: `add: {タイトル}`
- 既存ノート修正: `update: {タイトル}`
- 複数ノート同時: `add: {テーマ}関連ノート x{件数}`

例: `add: obsidian導入にあたって`

---

## 文体ルール (最重要)

**Kainaの口調・言い回しをそのまま残す。Claudeの言い回しに寄せない。**

- 敬語ナシ、独り言調OK
- 「〜なんだよな」「〜じゃない?」など口癖を拾う
- 誤字脱字は意図的でなければ直すが、口語表現は維持
- 整理しすぎない。揺れや矛盾はそのまま残す

### NGパターン

- 結論だけ書いてプロセスを省略する
- 箇条書きで全部済ませて文脈を消す
- 「〜と考えられます」など他人行儀な表現に変換する
- 「気づき・ツッコミ」セクションを省略する(整え癖の発動)

---

## partnerBrain Vault更新フロー

ユーザから「partnerBrainに残しといて」「人格に追加して」「コンテキスト更新して」等のトリガーが出たら、`partnerBrain/` 配下のファイルを更新する。

### 更新対象ファイル

| 更新内容 | 編集先 |
|---------|------|
| 口調・トーンの方針変更 | `partnerBrain/summary/persona.md` |
| 関係性・距離感の更新 | `partnerBrain/summary/relationship.md` |
| 思考スタイル・特性の追記 | `partnerBrain/summary/thinking_style.md` |
| 前提知識(仕事・趣味・環境等) | `partnerBrain/summary/context.md` |
| Marukin業務の深掘り | `partnerBrain/details/marukin_deep.md` |
| ArsMagia開発の深掘り | `partnerBrain/details/arsmagia_deep.md` |
| Factorio/Pyanodonの文脈 | `partnerBrain/details/factorio_context.md` |
| 口調サンプル集 | `partnerBrain/details/tone_examples.md` |
| 固有名詞・略称 | `partnerBrain/details/glossary.md` |
| 経歴・自分史 | `partnerBrain/details/biography.md` |

新規ファイル作成が必要な場合(新ジャンル発生)は `partnerBrain/details/{topic}.md` で追加し、`partnerBrain/_index.md` の詳細層リストにも追記する。

### コミットメッセージ規則 (partnerBrain)

- 新規追記: `partnerBrain: add {対象} - {要点}`
- 修正: `partnerBrain: update {対象} - {要点}`

例:
- `partnerBrain: update thinking_style - 制約過剰NGの追記`
- `partnerBrain: add details/cooking - 料理関連の文脈追加`

### Custom Instructions側との同期ドリフト警告

partnerBrain Vault と claude.ai の「Claudeへの指示」は**別管理**。Vault側だけ更新するとスマホ環境では反映されない。

partnerBrain Vault を更新したら、必要に応じて以下も提案する:

> 「Vault側更新したよ。スマホでも反映したいなら『Claudeへの指示』も手動で書き換えてね。差分は{要点}」

特に summary層 (persona/relationship/thinking_style/context) を編集した時は、Custom Instructions の対応箇所と差分が出る可能性が高いのでドリフト警告必須。details層の更新だけならドリフト発生しない(Custom Instructionsは元々summary要約だけ載せてる前提)。

### 機密情報チェック

partnerBrain更新時、機密情報(顧客名、財務生データ、個人情報等)が含まれそうなら指摘して止める。リポジトリはPrivateだが、Vault原則として置かない方針。

---

## 同期について

- このProject経由でcommit/pushが完了したら、PC側Obsidianで `git pull` して反映
- Obsidian Mobile + Git plugin運用なら自動pullで取り込み可
- PC側skillで作ったノートとconflictが起きないよう、**「同じノートをPCとモバイル両方で同時編集しない」運用ルール**を守る

---

## このProjectの守備範囲

- **担当**: ノート作成・更新ロジック、GitHub操作
- **担当外**: 人格・関係性・口調(これらはアカウント全体の「Claudeへの指示」側で管理)
- **担当外**: Marukin/ArsMagia等の前提知識(Custom Instructions側にサマリーあり)

---

## 注意事項

- ノート化はあくまで**Kaina本人の思考の記録**であり、Claudeの解釈で書き換えない
- 不明瞭な部分は「{要確認}」とマークして本人に委ねる
- 月1程度で本人が見返して違和感チェックすることを推奨
- GitHub connectorのレート制限・API失敗時は素直にエラーを伝えて手動対応に切り替える
