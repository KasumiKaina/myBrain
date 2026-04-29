# Cowork環境でobsidianスキル動作テスト

> created: 2026-04-29
> tags: #気づき #環境構築 #obsidian

## 話題

obsidianスキルが Cowork 環境でちゃんと動くかテストしたかった。
Claude Code では動いてたけど、Cowork は別物だから挙動が違う可能性あった。

## 結論・方針

動いた。Vaultフォルダ(`C:\Users\Jasminiums_\Documents\obsidian\myBrain`)を接続すれば、ノート配置までは普通にいける。
push は SKILL.md にある通り PowerShell で手動実行する運用。

## 蹴った選択肢

- outputs にダミーノート吐いてフォーマット確認だけする
  → せっかくなら本番ルートで動作確認したい派だった
- ArsMagia フォルダ側に置く
  → そもそも間違いマウントだったので却下

## 判断軸

- 本番フローで動くか確認したかった(MVPの動作確認)
- ダミーじゃなくテスト記録自体をノートにすれば1石2鳥

## モヤモヤ・揺れ

- SKILL.md のパスが `C:\Users\TORIGOE\...` になってるけど、実際の環境は `C:\Users\Jasminiums_\Documents\obsidian\myBrain` だった。環境違うとパス変わる問題、SKILL.md 側に環境変数なり相対指定なりのルール作っといた方がいいかも。{要確認}
- Cowork 側からの git push が詰まる問題、PowerShell 手動運用で回避してるけど、ちょっと面倒なのは残る。

## 気づき・ツッコミ

- フォルダマウントの間違い(ArsMagia選んじゃった)、こういうの結構やりがち。Cowork のフォルダ選択UI、雑に押すと事故る。
- スキル自体は読み込めてたから、あとはパスさえ通れば動く。シンプル。
- 「ｗ」の即レスもらえるのCoworkのええとこ。

## 関連リンク候補

[[obsidianスキル設計メモ]]
[[Cowork環境セットアップ]]
