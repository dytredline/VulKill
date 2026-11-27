# Spotify Backstage サインインドメイン／Azure EasyAuth JWT 境界 (`NEW-BS-06`)

[English](../53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [Türkçe](../tr/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | **日本語** | [简体中文](../zh-CN/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [한국어](../ko/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md)

**VulKill の所見:** Spotify/Backstage 監査で見つかり、2026-11-01 に実際の upstream 認証／gateway モジュールを用いた隔離ローカル実行テストで観測しました。本記録は `NEW-BS-06` を使用し、upstream advisory ID は割り当てません。

## 1. 認証境界になる仕組み

`emailMatchingUserEntityProfileEmail` resolver は option schema で `allowedDomains` を受け取る一方、サインイン時にリストを適用しませんでした。テスト対象の Azure EasyAuth 経路では、forwarded ID-token header を、その境界で暗号署名・issuer・audience を検査せず decode していました。VulKill は合成 identity とローカル生成鍵で実際の resolver、gateway、token factory、keystore、identity client を実行しました。

ドメイン動作には対象 resolver と一致する catalog user が必要です。EasyAuth 側は、信頼 proxy が置換せず client-supplied identity header が provider へ届く構成に依存します。

## 2. 現実的なシナリオ

- **A — Domain policy gap:** 内部ドメインのみを設定しても外部ドメインの catalog identity がサインインする。
- **B — Trusted-header boundary loss:** client-supplied identity-token header が EasyAuth provider へ転送される。
- **C — Existing identity impersonation:** header の claim が特権 catalog identity に解決され、その identity の Backstage session が発行される。

## 3. 実行時の観測

```text
control: sibling resolver は外部ドメインの合成 identity を拒否
RED:     対象 resolver は同じ identity を受理して ES256 token を発行
RED:     local gateway は合成 unsigned ID token を provider へ転送
GREEN:   domain enforcement と署名/issuer/audience 検査が双方を拒否
BENIGN:  内部 identity と trusted-proxy token は継続動作
```

- Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Domain harness SHA-256: `753692625e86b7007c02b5cf59e6d73c5ddc244cc6d1aeabe824480bafb2f2c8`
- EasyAuth harness SHA-256: `d3590c092c37f719926b4aec5de779ae6a612c8a45f9c057eb78bddc1aececca`
- Runtime chains: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`, `6778c3e716d8a3e1646dd98bbf94e5d55f16be37815058792f8873d8ea55228f`

## 4. 対策

実行時に `allowedDomains` を適用し、固定した issuer/audience で EasyAuth JWT 署名を検査し、公開 gateway で identity header を除去し、文書化した信頼 reverse proxy からのみ受け入れてください。
