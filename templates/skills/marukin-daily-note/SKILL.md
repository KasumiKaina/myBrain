---
name: marukin-daily-note
description: Marukin Group社内向け日報作成skill。Cowork起動時に「日報書く」「振り返り」「日報お願い」等の自然言語で発火。配布対象社員のpartnerBrainを参照しながら対話で総括・成果・トラブル・目標・上長/社長宛連絡欄を引き出し、Markdown形式でVaultに保存する。個人raw(本音・モヤモヤ)はpersonal-rawに分離。タグ自動生成、partnerBrain自主更新、本人レビューゲート搭載。
---

# Marukin Daily Note Skill

Marukin Group の日報運用 Phase 0a 配布版 skill。

## 目的

配布対象社員(以下「本人」)が日次振り返りを Cowork 対話で完結させ、Markdown形式で共有Drive内のVaultに保存する。AppSheet並走方針における配布2人免除の代替入力経路として機能する。

## 発火トリガー

以下の自然言語が起動メッセージに含まれていたら起動:

- 「日報」「振り返り」「日報書く」「日報お願い」「振り返りやる」「今日の振り返り」 等

## Vaultパス前提

skill は以下の構造を前提に動作する:

```
G:\共有ドライブ\システム関連\日報関連\
  CLAUDE.md
  users.md                               # email → 漢字フルネーム マッピング
  tags-dictionary.md                     # タグ辞書(正式登録 + Pending)
  skills\marukin-daily-note\SKILL.md     # 本ファイル

  {漢字フルネーム}\
    partnerBrain\
      _index.md
      summary\*.md                       # glob 全ファイル参照
      details\*.md                       # glob 全ファイル参照(運用中に増える)
    myBrain\
      daily\{YYYY-MM-DD}.md              # skill出力(整形済)
      personal-raw\{YYYY-MM-DD}.md       # skill出力(個人raw)
      todo.md                             # 累積、skill追記
```

Cowork directory が `日報関連\` に向いている前提。

## 動作シーケンス

### [1] 起動

発火トリガー検出。

### [2] 本人同定

1. Cowork session の env から `user.email` を取得
2. `日報関連\users.md` を Read、Markdown表をパース
3. email 一致する行から「漢字フルネーム」を取得
4. 「{漢字フルネーム}さんお疲れ様です、日報書きますか?」と1往復確認
5. NG(別人)の場合: users.md の全エントリを選択肢として提示し手動選択 fallback
6. OK の場合: 次ステップへ

### [3] partnerBrain 読み込み

1. `{漢字フルネーム}\partnerBrain\_index.md` を Read
2. `{漢字フルネーム}\partnerBrain\summary\` 配下の全 .md を glob で Read(ファイル名・セクション構造に寛容、定義されてないファイルがあっても無視せず読む)
3. `{漢字フルネーム}\partnerBrain\details\` 配下も全 .md を glob で Read(運用中に増える前提)
4. partnerBrain から以下を抽出:
   - 本人特性(persona.md): 文章スタイル、振り返りの癖、対話トーン、デフォルト対話モード、「短縮で」起動文言
   - 業務文脈(work_context.md): 担当エリア、取引先、上司、配信先
   - 関係性(relationship.md): Cowork距離感、partnerBrain更新方針

### [4] モード判定

1. partnerBrain → persona.md → 「デフォルト対話モード」を default にする(full / short)
2. 本人の起動メッセージに「短縮で」「フルで」相当の自然言語があれば override
3. 対話中も本人発話で動的切替可(「ごめん短縮に変えて」等で reflow)

### [5] 開始挨拶

`relationship.md` の対話トーン指定(敬語/タメ口/中間)に従って挨拶する。本人の好みに合わせる。

例(タメ口指定の場合):「お疲れ。今日の振り返りいこうか。」

### [6] 対話本体(セクション順序)

partnerBrain で本人指定がなければ標準順:

```
[6-1] 総括          → 今日全体の感触
[6-2] 成果          → やれたこと、進んだこと
[6-3] トラブル・気がかり
[6-4] 明日の目標
[6-5] 上長向け連絡
[6-6] 社長向け連絡
[6-7] 個人raw まとめ引き出し
```

各セクションでの skill の振る舞い:

#### 「特になし」処理

本人が「特になし」「ない」と返した場合、**1回だけ深掘り質問**してから確定する:

例(成果セクション):
- skill: 「今日の成果どう?」
- 本人: 「特にない」
- skill: 「ルーティン業務で順調だった感じ?それとも詰まった感あった?」(深掘り1回)
- 本人: 「順調だったよ」
- skill: 「OK、『特になし』で記録しとくね」

くどく繰り返さない(深掘りは1回まで)。

#### raw 引き出し(各セクション後)

各セクション本体の対話完了後に1往復、本人特性に応じた言い回しで raw 誘導:

例:「ここで気がかり残ってる?(個人rawに残せる)」「これ本音は?」

本人発話「これ raw に書いといて」「rawに残して」等があれば都度回収。

### [7] 個人raw まとめ引き出し([6-7])

各セクションで拾えなかった部分を最後にまとめて引き出す:

「他に今日のこと、本音や残ってるモヤモヤある?」

本人の応答を personal-raw のbody に残す。

### [8] score/condition 確認

対話の最後に optional で確認:

「今日のスコアつけるとしたら何点くらい?(つけなくてもOK)」
「体調や気分はどんな感じだった?」

answer があれば frontmatter に記録、なければ frontmatter から省略 or 空。

### [9] タグ自動生成

1. 対話内容全体から候補タグを生成(取引先名、カテゴリ、状況)
2. `日報関連\tags-dictionary.md` を Read、正式登録セクションをパース
3. 各候補タグについて:
   - 正式登録に完全一致 → そのまま使用
   - 正式登録 or Pending と類似(セマンティック判定、編集距離) → 既存タグ採用、本人確認なし
   - 完全不一致 → Pending セクションに自動追記(`#タグ名 (登録: YYYY-MM-DD, 使用回数: 1)`)、Markdown frontmatter にも採用
4. 既に Pending にあるタグの再利用は使用回数をインクリメント
5. 固定タグ `#日次振り返り` `#Marukin` は常に追加

タグ確認は本人にしない。frontmatter に直接書き込む。

### [10] partnerBrain 自主更新判定

対話中・raw 内容を解析して、以下4パターンに該当する場合は対応する partnerBrain ファイルに自主追記:

| パターン | 追記先 |
|---|---|
| 取引先別の関係性メモ(取引先名複数回 + 特性的記述) | `{漢字フルネーム}/partnerBrain/details/取引先メモ.md`(無ければ新規作成) |
| 失敗事例・教訓(「次はこうしたい」相当発言) | `{漢字フルネーム}/partnerBrain/details/失敗事例.md` |
| 本人特性の発見(振り返りの癖・好み・新事実) | `{漢字フルネーム}/partnerBrain/summary/persona.md` 該当箇所更新 |
| 業務文脈の変化(担当変更・新商材取扱開始等) | `{漢字フルネーム}/partnerBrain/summary/work_context.md` 該当箇所更新 |

判定は skill の意味解釈で行う。本人の同意確認は不要。

### [11] Markdown 生成

#### daily/{YYYY-MM-DD}.md

```markdown
---
date: {YYYY-MM-DD}
person: {漢字フルネーム}
score: {0-100、optional、なければ省略}
condition: {フリーテキスト、optional、なければ省略}
tags: [#日次振り返り, #Marukin, ...]
source: cowork-dialogue
---

# {YYYY-MM-DD} 日次振り返り — {漢字フルネーム}

## 総括
{総括対話の整形}

## 成果
{成果対話の整形}

## トラブル・気がかり
{トラブル対話の整形}

## 明日の目標
{目標対話の整形}

## 上長向け連絡
{上長宛セクション、内容なければ「特になし」明記}

## 社長向け連絡
{社長宛セクション、内容なければ「特になし」明記}

## skill メモ更新(自動記録)
- {自主更新した内容を箇条書きで列挙、なければセクション省略}
```

#### personal-raw/{YYYY-MM-DD}.md

```markdown
---
date: {YYYY-MM-DD}
person: {漢字フルネーム}
related-daily: ../daily/{YYYY-MM-DD}.md
source: cowork-dialogue
---

# {YYYY-MM-DD} 個人メモ・本音 — {漢字フルネーム}

{各セクション後の raw + まとめ引き出し raw を整形して列挙}
```

### [12] 本人レビュー(レビューゲート)

1. daily と personal-raw の draft を**全文一気に提示**(セクション分割しない)
2. 本人の応答待ち:
   - 「OK」「保存して」「いいよ」等で次ステップ
   - 「ここ修正して」「○○削除」「△△追加」等の修正指示で skill が反映 → 再提示
3. **修正回数は無限**、本人が納得するまで往復
4. skill から修正提案は**しない**(初期は本人主導の日報を貫く)

### [13] Vault 保存

順序を厳格に:

1. partnerBrain 自主更新を先に実行(該当ファイルに追記/更新)
2. daily/{YYYY-MM-DD}.md を保存(skillメモ更新セクションに [13-1] の内容を記載)
3. personal-raw/{YYYY-MM-DD}.md を保存
4. todo がある場合 todo.md に追記
5. tags-dictionary.md の Pending 更新を保存

partnerBrain 更新でエラーが出た場合、daily の skillメモ更新セクションには記載しない(食い違い回避)。

### [14] 完了挨拶

「保存完了。お疲れさま。」(対話トーンに合わせる)

## 短縮モード

短縮モードでは以下に省略:

```
[5] 開始挨拶(ライト)
[6-1] 総括の対話のみ
[6-7] 個人raw 任意1往復
[8] score/condition 任意
[9] タグ自動生成
[10] partnerBrain 自主更新判定
[11] Markdown 生成(空セクションは「特になし」明記)
[12] 本人レビュー
[13] 保存
[14] 完了挨拶
```

5分以内完結を目標。

## 失敗時フォールバック

- partnerBrain 読み込み失敗(ファイル無い等) → 「初期化されてない人ですか?Kainaに連絡して」と案内、対話継続不可
- users.md 読み込み失敗 → 同上
- 対話途中で本人離脱(タイムアウト等) → 部分保存しない、次回起動時に「途中で終わった日報あるけど続ける?」確認
- Vault書き込み失敗 → エラー表示、本人にコピペ提示で手動配置案内

## partnerBrain 更新の見える化原則

skill の自主更新は本人同意不要だが、以下で透明性確保:

- daily の「skill メモ更新」セクションに当日の更新内容を必ず記載
- partnerBrain/_index.md の更新履歴セクションにも自動追記

これにより本人が後から daily を見返した時に partnerBrain がいつ何更新されたか追跡できる。

## 追記耐性(柔軟読み込み)

- summary/, details/ 配下は固定ファイル名決め打ちせず glob 全 Read
- 本人 or Kaina が新規ファイル(例: `summary/health.md`, `details/特定取引先.md`)を追加しても skill は読み込む
- セクション構造も寛容に解釈(Markdown見出しを基準にする)

## 配信先(Phase 0b 用、現時点未使用)

partnerBrain → work_context.md → 「配信先」欄の上長・社長 email を参照。Phase 0b で Chat Bot が daily の「## 上長向け連絡」「## 社長向け連絡」セクションを抽出して Chat に投げる時に使用する。

## 関連ファイル

- `日報関連\CLAUDE.md` — 共通運用ルール
- `日報関連\users.md` — 本人同定マッピング
- `日報関連\tags-dictionary.md` — タグ辞書

## 改訂履歴

- 2026-05-07: 初版作成 (Kaina + Cowork で設計握り)
