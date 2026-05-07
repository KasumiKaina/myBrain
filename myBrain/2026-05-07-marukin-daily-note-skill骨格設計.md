# marukin-daily-note skill 骨格設計

> created: 2026-05-07
> tags: #判断 #PoC #Marukin #Claude #日報 #skill設計 #SKILL.md #設計握り

## 話題

[[2026-05-07-marukin-daily-note-skill設計握り]] で5論点(A〜E)の前提握った後、Step 2として **SKILL.md 骨格+対話フロー詳細**を10論点(A〜J)で詰めた記録。

これで配布2人運用に出せる skill の動作仕様がほぼ確定。SKILL.md ファイルを別途 myBrain/templates/skills/marukin-daily-note/ に配置して、Kaina手動で `G:\共有ドライブ\システム関連\日報関連\skills\` に展開する流れ。

## 結論・方針

### 全論点(A〜J)まとめ

| 論点 | 確定内容 |
|---|---|
| **A. 本人同定** | `日報関連\users.md` に email→漢字フルネーム マッピング表、skill が起動時に env.email で照合 + 1往復確認 |
| **B1. 対話セクション順序** | 初期=標準順(総括→成果→トラブル→目標→上長連絡→社長連絡)、運用中に partnerBrain 更新で本人化(熟成) |
| **B2. 「特になし」処理** | 「特になし」明記方針、ただし1回深掘り質問してから確定(くどくならない程度) |
| **B3. raw引き出しタイミング** | A+B ハイブリッド(各セクション都度 + 対話最後にまとめて) |
| **B4. score/condition 聞きどき** | 対話最後 |
| **C. モード切替UX** | partnerBrain default + 起動時/対話中の自然言語切替(「短縮で」「フルで」) |
| **D. Markdown仕様** | daily/{date}.md + personal-raw/{date}.md + todo.md / upstream削除 / frontmatter最小 / score/condition optional / mode削除 / タグは Claude 自動生成 + Pending方式 / Obsidian 形式 `#タグ` |
| **E1. レビュー方式** | skill draft提示 → 対話で修正 → OK で保存 |
| **E2. draft 提示粒度** | 全文一気提示(初期設定) |
| **E3. 修正回数** | 無限往復、本人納得まで |
| **E4. skill 修正提案** | 初期段階では**提案しない**、本人主導の日報を貫く |
| **E5. partnerBrain 更新** | 同意不要・skill 自主追記、ただし daily 末尾の「skill メモ更新」セクションで**見える化** |
| **F. 自主追記の判断基準** | 取引先別関係性 / 失敗事例・教訓 / 本人特性 / 業務文脈変化 の4パターンで自動判定 |
| **J. 追記耐性** | summary/details 配下を **glob で全ファイル Read**、ファイル名・セクション構造に寛容 |

### Vault構造(最終)

```
G:\共有ドライブ\システム関連\日報関連\
  CLAUDE.md                              # Marukin日報運用の共通ルール
  users.md                               # email→漢字フルネーム マッピング
  tags-dictionary.md                     # タグ辞書(正式登録 + Pending)
  skills\
    marukin-daily-note\
      SKILL.md                           # 今回作る本体

  {漢字フルネームA}\                      # アクセス権: 本人 + Kaina
    partnerBrain\
      _index.md
      summary\
        persona.md
        work_context.md
        relationship.md
      details\                            # 運用中に対話駆動で深化
    myBrain\
      daily\{YYYY-MM-DD}.md              # 整形済本体、上司閲覧前提
      personal-raw\{YYYY-MM-DD}.md       # 個人raw、本人+Kainaのみ
      todo.md

  {漢字フルネームB}\                      # 同上
```

### skill 動作シーケンス(完成版)

```
[1] 起動(自然言語: "日報書く" "振り返り" 等)
[2] 本人同定 (users.md 照合 + 1往復確認)
[3] partnerBrain 読み込み (summary/details 配下 glob 全 Read)
[4] モード判定 (default + 今日の指定)
[5] 開始挨拶 (relationship.md の対話トーン適用)

— 対話本体 —
[6] 総括の対話
[7] 成果の対話 (※ raw引き出し: 各セクション後に1往復誘導)
[8] トラブル・気がかりの対話
[9] 明日の目標の対話
[10] 上長向け連絡の対話
[11] 社長向け連絡の対話
[12] 個人raw まとめ引き出し対話
[13] score/condition 確認 (任意、答えなければ frontmatter から省略)

— 整形・保存 —
[14] タグ自動生成 (tags-dictionary.md 照合, Pending 自動登録, 確認なし)
[15] partnerBrain 自主更新判定 (F の4パターン)
[16] Markdown 生成 (frontmatter + セクション + skillメモ更新)
[17] 本人レビュー (全文提示 → 対話修正 → 無限往復 → OK)
[18] Vault 保存 (daily + personal-raw + todo追記 + partnerBrain 更新)
[19] 完了挨拶
```

### Markdown仕様(最終)

**daily/{YYYY-MM-DD}.md**

```markdown
---
date: 2026-05-07
person: {漢字フルネーム}
score: 75
condition: 普通
tags: [#日次振り返り, #Marukin, #配達, #○○商事]
source: cowork-dialogue
---

# 2026-05-07 日次振り返り — {漢字フルネーム}

## 総括
## 成果
## トラブル・気がかり
## 明日の目標
## 上長向け連絡
## 社長向け連絡

## skill メモ更新(自動記録)
- partnerBrain/details/取引先メモ.md 追加: ○○商事との対応パターン
- partnerBrain/summary/persona.md 更新: 振り返りの癖を「記録型」と確認
```

**personal-raw/{YYYY-MM-DD}.md**

```markdown
---
date: 2026-05-07
person: {漢字フルネーム}
related-daily: ../daily/2026-05-07.md
source: cowork-dialogue
---

# 2026-05-07 個人メモ・本音 — {漢字フルネーム}

(自由記述、対話で出てきたモヤモヤ・本音・揺れ)
```

## 蹴った選択肢

### A. 本人同定
- **案B 自己同定スキャン**(各 _index.md に email 埋込) — Phase 1以降で上司閲覧範囲広がる時、アクセス権ベースのフィルタ前提が崩れる。最初から表ベース運用に倒した。
- **案C 計算式同定** — 漢字フォルダ名と email 命名規則の一致が現実的でない。
- **案D 単独運用** — 確認なしの自動同定は誤同定 fallback がない。

### B3. raw引き出し
- **案A単独**(各セクション都度のみ) — 漏れる
- **案B単独**(対話最後のみ) — 各セクションでの本音が拾えない
- **案C本人駆動のみ** — 本人意識依存、漏れる
- A+B ハイブリッドで漏れ防止重視

### D. 連絡欄の二重保持
- **upstream/ ディレクトリ作成** — daily が上司閲覧前提なら不要、二重保持コスト不要
- daily 単一ソース化、Phase 0b の Bot 配信時にセクション抽出する設計でクリーン

### E4. skill修正提案
- **積極的に提案する** — Cowork能力的にはできるが、配布2人の継続性的に「skill にぐいぐい来られる」のは本人主導感を削ぐ
- 初期は控えめ、運用感掴んでから設計再考

### E5. partnerBrain更新
- **本人合意必須** — 都度確認は煩わしい、配布2人の運用負担増
- 同意不要だが「見える化」で透明性確保、これが Goldilocks

### J. 追記耐性
- **固定3ファイル決め打ち** — Kaina要望「ユーザが必要だと言ったら追記できる体制」と矛盾、glob 読みで柔軟性確保

## 判断軸

- **走りながら改善**(B1 熟成、F 判断基準は運用で精度上げる)
- **配布2人の継続性最優先**(対話往復の負担、レビューUXの自然さ)
- **本人主導を貫く**(E4 修正提案 OFF、本人の日報感を保つ)
- **透明性 > 同意取得**(E5 partnerBrain更新、許可じゃなく見える化)
- **柔軟性・拡張性**(J 追記耐性、B1 partnerBrain で本人化)
- **単一ソース原則**(D upstream削除、daily 単一化)
- **Goldilocks 原則**(都度確認の重さと本人放置の漏れ、間を取る)

## モヤモヤ・揺れ

### 対話往復回数のコスト感
- フルで6セクション × 平均2-3往復 + raw誘導 + レビュー = 20-30往復、所要時間 20-30分?
- 配布2人の現実運用で耐えるか未知数
- 短縮モード活用と「特になし」深掘り1回までで時短設計入れたが、初期運用で手応え見て調整必要

### タグ自動生成の精度問題
- 表記ゆれ、類義語、粒度ブレ、すべて運用しながら辞書育てる前提
- 初期運用1ヶ月のPending肥大化が想定される、Kainaのレビュー作業量が読めない
- 自動的に「使用回数3回以上で正式登録候補」みたいな閾値も入れた方がいいかも

### partnerBrain 自主更新の暴走リスク
- F の4パターンに該当すれば skill が勝手に追記する設計
- 「該当」の判定が緩いと毎日 partnerBrain が膨らんでメンテ不可
- 「該当」の判定が厳しいと知見溜まらない
- daily の skill メモ更新セクションで履歴は見えるが、partnerBrain 自体の肥大化への対処は別軸

### draft 全文提示の対話 UX
- 配布2人が draft 一気に見て「ここ修正」と的確に指示出せるか未検証
- IT苦手中堅層にはそもそも全文レビュー自体が重い可能性
- partnerBrain で「draft セクション分割提示」も指定可にする逃げ道残すべきかも

### skill メモ更新セクションの上司閲覧
- daily 末尾に skill 更新履歴載るけど、上司が「これ何?」と思う可能性
- Phase 0b で配信フロー作る時、Bot が daily を読む時にこのセクションも一緒に流れる
- 本人にとっては自分の partnerBrain 更新履歴、上司には見せたくない情報かも
- → 配信時はこのセクション抽出スキップ、daily 自体には残す、で対応可能

### 短縮モード時の raw 引き出し
- フルだと A+B ハイブリッドだが、短縮だと「総括だけ → raw任意 → score任意」で raw 引き出し弱くなる
- 短縮モードで rawが拾えないなら、その日のモヤモヤ拾えない
- 配布2人が「短縮モード多用 + 翌週まとめて raw」みたいな運用始めたら設計見直し

### partnerBrain 更新失敗時のリカバリ
- skill が partnerBrain 更新中にエラーで止まったらどう戻す
- daily の skill メモ更新セクションだけ書かれて、実際の partnerBrain は更新されてない、みたいな食い違い
- → SKILL.md に「partnerBrain 更新が成功してから skill メモ更新セクションに書く」順序明記必要

### Cowork未起動日のリカバリ(H 論点、未着手)
- 出張・直行直帰・体調不良で PC前にいない日、翌朝まとめ書きする時の date 扱い
- skill に「日付指定モード」入れる必要、別軸で詰める

## 気づき・ツッコミ

### 10論点詰め切ったら SKILL.md 骨格が一気に書ける状態になった
- A〜J 全論点クローズ → SKILL.md は機械的に組み立てられる
- 設計握りに2セッションかけた価値、骨格設計の決定コストが極小化された
- 設計と実装を分離する Step 設計、めちゃくちゃ機能してる

### partnerBrain 更新「同意不要、見える化」は Kaina的回答だった
- Kaina の許容認知負荷からして「都度確認」は耐えられない
- 「許可なし + 透明性」が Kaina感覚の Goldilocks
- 配布2人にも転写される設計、配布対象拡大時に「都度確認派」の人が出てきたら設計見直し点

### E4「skill 修正提案 OFF」は配布2人主導感の死守
- 提案ON にするのは技術的には簡単
- Kaina判断で「本人主導の日報」を意図的に守る選択
- これは Marukin 文化(押し付けがましさNG、本人決め)とも整合してそう
- partnerBrain で個別 ON/OFF 切替可能にしておけば、対話派の人だけ後で ON 化できる

### users.md と tags-dictionary.md は同じ「共有メタ」レイヤー
- どちらも 配布2人共通、Kaina管理
- 設計上は users が同定情報、tags が分類情報、別物
- けど運用上は同じレイヤー(共有メタ)、まとめてフォルダ作って管理した方が綺麗
- → `日報関連\meta\` 配下に置く案も後で検討

### 「特になし」の1回深掘りは思想的に深い
- 「能動的に書く」を回避しつつ「書くべきこと」は引き出す、対話駆動の旨味
- AppSheet との差別化点、構造化フォーム入力じゃ出せないところ
- skill の対話質問パターンを partnerBrain で本人化していくと、深掘り精度が育つ
- 1ヶ月運用後に「深掘り効きすぎ」「効かなさすぎ」のフィードバックループ要

### Vault 構造「{漢字フルネーム}\partnerBrain\ + myBrain\」の二層化はメンテ思想として綺麗
- partnerBrain = Claude側の業務理解(変動少)
- myBrain = 本人の思考ログ(変動多)
- 両方が個人root下に並ぶことで、本人スコープが明確
- Kaina個人myBrain Vault と同じ思想を組織展開した形、整合

### skill メモ更新セクションは Marukin社内の「Claude透明性」設計の最初の例
- Claude が自主判断で書き換えた内容を本人が後追い確認できる
- これ Phase 1以降に他の skill (営業支援・在庫分析等) でも転用できる思想
- 「Claude が何やってるか見える」は Marukin AI推進の根幹原則になりそう

## 次のアクション

1. **SKILL.md 骨格ファイルを myBrain/templates/skills/marukin-daily-note/ に作成**(本セッション継続)
2. **Kaina手動で `日報関連\skills\marukin-daily-note\` に展開**
3. **users.md 初版作成**(配布2人ぶん、Kaina手動)
4. **tags-dictionary.md 初版作成**(配布2人と相談、初期は薄くてOK)
5. **配布2人ぶん partnerBrain 雛形展開**(Step 3で作った雛形、{漢字フルネーム}\partnerBrain\ にコピー)
6. **配布2人の基礎情報を Kaina手動で埋める**(役職・担当エリア・取引先群・上司)
7. **Kaina自身で1人ぶん試運転**(skill実装後、ドッグフーディング)
8. **配布2人へリリース・運用開始**(Phase 0a)
9. **0a期間中の調整事項リスト化**(SKILL.md 改修・partnerBrain 深化・タグ辞書整理)
10. **AppSheet並走免除の社長合意取得**(Phase 0a開始前)
11. **Cowork未起動日リカバリ(H論点)を別軸で詰める**(運用見ながら)
12. **配信(Phase 0b)準備**(Chat Bot+スペース構造設計、別ノート)

## 関連リンク候補

- [[2026-05-07-marukin-daily-note-skill設計握り]] — Step 1、A〜Eの前提握り
- [[2026-05-07-日報PoC設計-AppSheet軸転換とChat配信設計]] — 親、本ノートの上流
- [[2026-05-07-日報PoC設計-Chat連携フロー]] — 同日先行、Chat連携軸の検討
- [[2026-04-30-claude日報社内展開PoC設計]] — 初期構想・原則
- [[partnerBrain]]
- [[Marukin基幹刷新]]

## 要確認メモ

- {要確認} skill 対話往復の所要時間(配布2人で実測、フル20-30分目安)
- {要確認} タグ Pending の肥大化ペース(初期運用1ヶ月で観測)
- {要確認} partnerBrain 自主更新の頻度・暴走リスク
- {要確認} draft 全文提示の対話 UX(IT苦手層にきついか)
- {要確認} skill メモ更新セクションの上司閲覧時の見え方
- {要確認} 短縮モード時の raw 引き出し弱化の運用影響
- {要確認} `skills/`, `meta/` フォルダ命名(共有メタレイヤーの整理)
- {要確認} AppSheet並走免除の社長合意タイミング
- {要確認} Cowork未起動日のリカバリ動線(H論点)
