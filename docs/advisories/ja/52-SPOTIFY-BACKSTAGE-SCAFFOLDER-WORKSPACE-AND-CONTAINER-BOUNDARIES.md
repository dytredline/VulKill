# Spotify Backstage Scaffolder のワークスペース／コンテナイメージ境界 (`NEW-BS-05`)

[English](../52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [Türkçe](../tr/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | **日本語** | [简体中文](../zh-CN/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [한국어](../ko/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md)

**VulKill の所見:** Spotify/Backstage 監査で見つかり、2026-11-01 に実際の upstream Scaffolder モジュールを用いた隔離ローカル実行テストで観測しました。本記録は監査 ID `NEW-BS-05` を使用し、upstream advisory ID は割り当てません。

## 1. セキュリティ境界になる仕組み

`fetch:rails` は出力先に文字列の `startsWith` 比較を使っていたため、同じ接頭辞を持つ兄弟パスやワークスペース内の symlink がタスク領域外へ解決され得ました。Cookiecutter action は別途、呼び出し側が指定する `imageName` を action 単位の許可リストなしで container runner に渡していました。VulKill は一時ワークスペースとネットワーク無効のローカル test container で実際の action handler を実行しました。

対象 action の実行権限と該当入力の制御が前提です。テストは Spotify のライブサービスや外部 registry に接続していません。

## 2. 現実的なシナリオ

- **A — 兄弟ワークスペースへの書込み:** 同じ接頭辞を持つ隣接タスクへ生成ファイルが書かれる。
- **B — Symlink escape:** 字句上は内部のパスが symlink 経由でタスク外へ書き込む。
- **C — 未承認 generator image:** template が想定外の workflow／dependency ファイルを生成する image を選ぶ。

## 3. 実行時の観測

```text
control: 通常の子パスは一時ワークスペース内に留まる
RED:     兄弟接頭辞と symlink のパスが領域外へ合成 marker を書く
RED:     呼び出し側指定の test image がローカル runner に到達する
GREEN:   safe-child 解決と image allowlist が同じ入力を拒否する
BENIGN:  承認 image と通常の子パスは継続動作する
```

- Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `fa4dab592b61c8f8fcde81743694189b5d988cd6a9c7cbf29accc50c4bc097ae`
- Runtime chain SHA-256: `0e2de393ee65136719495c2824cfe5133512ce570636174ee942fc208bbdc47a`

## 4. 対策

safe-child helper と realpath で出力先を閉じ込め、symlink escape を拒否し、action 境界で container image を許可リスト化し、機密 template の作成・実行者を制限してください。
