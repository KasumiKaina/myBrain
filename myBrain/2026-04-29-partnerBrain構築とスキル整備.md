# partnerBrain構築とスキル整備

> created: 2026-04-29
> tags: #判断 #気づき #モヤモヤ #環境構築

## 話題

myBrain(自分の思考ログVault)の対になる、Claude側の人格・知識ベース「partnerBrain」を構築する話。
雛形のディレクトリ構成と、それを自動読み込みしてくれるpartner-brainスキルをセットで作る方針。
ついでに既存のobsidianスキルも、リポジトリ構成変更に合わせてパス修正する。

## 結論・方針

- `partnerBrain` Vaultを `myBrain` リポジトリ内に新規作成
  - サマリー(常時参照) + 詳細(必要時のみ) の2段構成
  - `summary/` に persona / relationship / thinking_style / context の4ファイル
  - `details/` は雛形だけ置いて、これから書き足していく
- リポジトリ構造を **「myBrain/(思考ログ) と partnerBrain/(Claude人格) の並列同居」** に変更
  - リポジトリルート: `Obsidian Vault\myBrain\`
  - 思考ログVault: `Obsidian Vault\myBrain\myBrain\`
  - Claude人格Vault: `Obsidian Vault\myBrain\partnerBrain\`
- `partner-brain` スキルを新規作成。description発火で自動的にpartnerBrain読みに行く仕様
- `obsidian` スキルは保存先パスを `myBrain\myBrain\` に修正(作り直してzip再生成)
- 当面は `partnerBrain` の中身は雛形のまま。会話を重ねながら書き足す

## 蹴った選択肢

- **シンボリックリンク方式**(指示書通り)
  → Windowsはsymlinkに管理者権限orDevModeが必要でダルい。シンプルなネスト同居の方がメンテラク
- **summary層を最初から情報で埋める**
  → 後で雛形ベースで自分で書きたい。Claudeの解釈で先に塗りつぶされるのは違う
- **partner-brainスキルを明示呼び出し限定**
  → 毎回「partnerBrain読んで」って言うのダルい。description発火で自動化したい(ベースラインの強さは妥協)
- **Coworkサンドボックスから直接git push**
  → やってみたら `.git/index.lock` が解除できず詰まる。ローカルPowerShellでpushが安定

## 判断軸

- **メンテコスト**: symlink地獄や複雑な構成は避ける。シンプルなフォルダ階層で済ませる
- **継続運用性**: 「ついで更新」が回り続ける構造に。雛形で空けておくのは無理に書かせない工夫
- **トークン効率**: 2段構成にして常時読みは軽量に保つ
- **環境差吸収**: Claude Code / Cowork / Web版でgit運用方針を分岐(自動 vs PowerShell案内)
- **「育てる」設計**: Claudeに完成形を作らせるんじゃなく、自分で書き足していけるベースを用意

## モヤモヤ・揺れ

- **ベースライン強い問題**: persona.md/thinking_style.mdに「こう振る舞え」って書いても、モデルの口調は完全には変わらない。反映されたらラッキー、で見るしかない
- **作業中の謎現象**: Cowork経由でWriteしまくってる最中に、なぜか元のmyBrain配下ファイルが`myBrain\myBrain\`にネスト化された。OneDriveか何か?でもタイミングは作業中。**再現性怖い**
- **名前ぶつかり**: リポジトリ名もmyBrain、思考ログVaultもmyBrain、Vault親フォルダもObsidian Vault\myBrain\。3階層中2回myBrainが出てくるのキモい。後から `notes` とかにrenameしたくなるかも
- **スキル間のhardcoded path**: obsidianスキルとpartner-brainスキルで同じパスを別々に持ってる。次パス変更したら2箇所直すの確定。3つ目スキル出てきたら共通設定ファイル(例: partnerBrain/summary/context.mdに集約)に逃がすべき
- **userIgnoreFiltersが効くか未検証**: `.obsidian/app.json` に書いたけど、実際にmyBrain Vaultから`partnerBrain/`が隠れてるかObsidian開いて確認してない

## 気づき・ツッコミ

- Cowork、Linuxサンドボックスとの間でファイル同期がたまに荒ぶる。Write連発するとマウントの整合が崩れることがあるっぽい。**バッチでファイル作るときは様子見ながらの方が安全**
- `.skill` ファイルは中身zipアーカイブだった。SKILL.md直接編集しても反映されない、毎回zip再パッケージ→再インストールの儀式。MVPっぽくはあるけど、頻繁に更新するならホットリロード欲しいよね
- 「人格を別Vaultで管理する」って発想、考えてみると面白い。**自分のメタ認知をmyBrainに、相手のメタ認知をpartnerBrainに**。これ続けると、Claudeの応答の癖を自分でデバッグできるようになる気がする(persona.mdに「こうじゃなかった」を書き溜める運用)
- 指示書を自分で書いた時点では「symlinkで繋ぐ」って雑に書いてたけど、Coworkに渡したら速攻で「symlinkダルい」って却下された。**指示書を書く側より、実行する側のほうが現実的判断できる**ことある
- partner-brainスキルのdescriptionに「Marukin/ArsMagia/Factorio」とか具体名詞ガッツリ並べてる。トリガー精度のためだけど、後でこれらの名詞が変わると修正必要。**固有名詞ベースの発火ロジックは脆い**
- 並列構成のリポジトリ、よく考えると `partnerBrain/` をsubmoduleにする選択肢もある。今は単一commit履歴で混ざるけど、3ヶ月後「partnerBrainだけ別履歴で見たい」って思うかも。今やる必要はないけど頭に置いとく

## 関連リンク候補

[[2026-04-29-obsidian導入にあたって]]
[[2026-04-29-obsidian-brainプラグイン化とCowork対応]]
[[2026-04-29-obsidian導入とインストール格闘]]
