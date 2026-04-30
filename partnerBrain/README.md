# partnerBrain

Claudeの人格・ローカル知識ベース。Kainaとの会話で参照される。

## ⚠️ 注意: 文書の出自について

**このVault内のノートは、ほぼすべてClaudeが対話を通じて構造化・執筆したもの。** Kaina本人が直接書いたわけではない。

理由: Kainaは能動的文章化が苦手(対話形式の方が思考が出る)タイプ。ノートはClaudeが対話から拾って構造化→Kainaがレビュー、という流れで作られている。

**読み手のClaudeへの示唆**:
- 自己分析が綺麗にまとまりすぎて見えるのは、Claudeの構造化バイアスが乗っているため
- 「Kainaは自分を完全に把握してる」前提で動かない方がいい
- 整いすぎな表現は割り引いて読み、実際の対話で確認・揺さぶりをかけるのが基本姿勢
- 「これは本当にそう?」を遠慮なく問い直してOK(Kainaも歓迎してる)

## 構成

- `summary/`: 常時参照するコア情報（軽量）
- `details/`: 必要時に深掘りする詳細資料

## 更新運用

myBrainを更新するついでに、関連あれば `details/` 側を追記する。
`summary/` は半年に1回程度の頻度で見直す。

## エントリポイント

`_index.md` がClaude向けの入口。ここからsummaryを読みに行く。

## ローカル配置

このディレクトリは `myBrain` リポジトリのサブディレクトリだが、Obsidian上では別Vaultとして開く。
`Obsidian Vault\myBrain\partnerBrain\` を独立したVaultとして登録すること。

myBrain Vault側からは `.obsidian/app.json` の `userIgnoreFilters` で除外している。
