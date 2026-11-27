# Spotify Backstage Catalog `validate-entity` ローカルファイル露出 (`GHSA-h8rc-g67j-87x3`)

[English](../51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [Türkçe](../tr/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | **日本語** | [简体中文](../zh-CN/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [한국어](../ko/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md)

**VulKill の発見:** Spotify/Backstage 監査で見つかり、2026-11-01 のローカル実行テストで観測しました。Upstream advisory は 2026-11-18 に公開されました。
**Upstream:** [`GHSA-h8rc-g67j-87x3`](https://github.com/backstage/backstage/security/advisories/GHSA-h8rc-g67j-87x3) · CVSS 3.1 `6.5 Medium` · `CWE-200` / `CWE-863`。

## 1. 仕組み

`POST /validate-entity` は server 管理 location を付与した後、caller の annotations を上書き優先で merge していました。Caller は location context を `file:` に差し替え、processor を backend file reader へ到達させ、YAML 診断から source line を HTTP response に出せます。VulKill は実際の router、validation service、orchestrator、`FileReaderProcessor` 経路で観測しました。

影響: `@backstage/plugin-catalog-backend < 3.9.3` および `>= 4.0.0, < 4.0.3`。修正版: `3.9.3`、`4.0.3`、`4.0.4`、`4.0.5`。

## 2. 現実的なシナリオ

- 通常の signed-in user が backend 設定ファイルの行を読む。
- parser error が secret を含む source line を応答へ反映する。
- forged provenance annotation が repository context 境界を越える。

## 3. 実行時の観測

```text
control: 無権限 guest -> 403
RED:     validate 権限 + forged location annotation -> local marker が露出
GREEN:   server annotation が優先され local read を阻止
BENIGN:  正常な URL entity validation は動作
```

Harness SHA-256: `0bd744cadf97f031fd218d97eb187fa0394c7ec0b23d8c702fe7eec52f0403ce`

## 4. 対策

修正版へ更新し、それまでは `catalog.entity.validate` を信頼済み利用者へ限定してください。
