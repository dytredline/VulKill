# Spotify SDK OAuth／PKCE／セッショントークン境界 (`NEW-SDK-02`)

[English](../55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [Türkçe](../tr/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | **日本語** | [简体中文](../zh-CN/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [한국어](../ko/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md)

**VulKill の所見:** Spotify の OSS TypeScript／iOS／Android SDK・example surface で見つかり、隔離したローカル、emulator、実機の各環境で観測しました。本記録は `NEW-SDK-02` を使用し、upstream advisory ID は割り当てません。

## 1. OAuth／セッション境界になる仕組み

テストした TypeScript SDK は `Math.random()` で PKCE verifier 材料を生成し、最初の読出しで verifier を削除しました。Web Playback example は最新 OAuth token を process-wide 変数に置き、共有 token route から返しました。監査した mobile example 経路も、callback／deep link の token state を開始 session に一貫して結び付ける前に保存 session data を置換し得ました。

影響は adopter の callback 公開方法、browser session 分離、example code の組込み方に依存します。

## 2. 現実的なシナリオ

- **A — Premature verifier eviction:** callback 形状の request が正規 callback 前に verifier を消費する。
- **B — Cross-session token mix-up:** 同一 example server を共有する二人で後の login が前 session の token を置換する。
- **C — Mobile callback confusion:** 開始 mobile session に結び付いた `state` なしで deep link／OAuth response を受理する。

## 3. 実行時の観測

```text
RED:     callback-shaped request が交換成功前に verifier を消費
RED:     二人の合成 user で process-wide token replacement を観測
RED:     mobile callback が完全な session binding 前に state/token data を受理
GREEN:   暗号乱数、成功後削除、session 別 storage、state 検査が各事象を阻止
BENIGN:  正しく binding された OAuth callback は継続動作
```

- TypeScript commit: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- iOS auth commit: `1e92eb3ddd632a3ab3b2ed852057948f8ef05067`
- Android auth commit: `eef9af9ffed5ed4e5ed63e2c162131c8da711bba`
- Web Playback commit: `d22700174073dec831635aac452acbf5b4186853`
- Runtime SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Mobile harness SHA-256: `0035b900685a91318a76117f59b476512b7e6b7f85e5b46098b08244415e60a0`
- Device result SHA-256: `fe2a5ab870c226dc8f06ad9793b2bf774868092c36f35716ae8f36fd830a7a80`

## 4. 対策

PKCE に暗号学的乱数を使い、state-bound exchange 成功まで verifier を保持し、token を認証 session ごとに保存し、OAuth `state` を完全一致で検査してください。example app にも production session control が必要です。
