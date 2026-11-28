# Spotify Web API TypeScript SDK のパスパラメータ境界 (`NEW-SDK-01`)

[English](../54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [Türkçe](../tr/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | **日本語** | [简体中文](../zh-CN/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [한국어](../ko/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md)

**VulKill の所見:** Spotify の OSS SDK 監査で見つかり、2026-11-01 にコンパイル済み upstream SDK の隔離ローカル HTTP 実行テストで観測しました。本記録は `NEW-SDK-01` を使用し、upstream advisory ID は割り当てません。

## 1. リクエスト境界になる仕組み

11 endpoint class の32 method が、呼び出し側の resource ID を単一 path segment として encode せず URL template に補間していました。アプリが攻撃者の影響する ID を渡すと URL normalization により到達 endpoint が変わり、SDK は引き続き bearer token を付加します。VulKill は32 method 全てを loopback-only の合成 endpoint に対して実行し、Spotify のライブ API には送信していません。

影響は、利用アプリが攻撃者制御の ID を受け付けるか、token が持つ scope に依存します。

## 2. 現実的なシナリオ

- **A — Backend route confusion:** playlist ID を受け取る wrapper が別の Web API route に到達する。
- **B — State-changing pivot:** mutation helper が同じ token で許可された別 endpoint へ認証済み request を送る。
- **C — Data-read pivot:** read helper がアプリの意図と異なる許可済み route のデータを返す。

## 3. 実行時の観測

```text
control: 通常の base62-like ID は意図した合成 route に到達
RED:     32/32 method が攻撃者影響下の ID を別 route へ normalize
GREEN:   共通 path-segment 検査/encoding が同じ入力を拒否
BENIGN:  通常 ID は防御変更後も互換
```

- Source commit: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- Harness SHA-256: `3e65e293ad938d9bf526027e6bcb1371e6f2340d6a94adfb7d83d7b1400d6685`
- Result SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Matrix: `32` methods (`10` state-changing, `22` read)

## 4. 対策

共通 request builder で各 ID を単一 path segment として encode し、URL 構築前に traversal／encoded separator を拒否し、未信頼 request 由来 ID には利用側 allowlist を設けてください。
