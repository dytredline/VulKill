# Spotify Backstage TechDocs 静的コンテンツ認可バイパス (`GHSA-rg9r-hr7g-5gc2` / `CVE-2026-106489`)

[English](../47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [Türkçe](../tr/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | **日本語** | [简体中文](../zh-CN/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [한국어](../ko/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md)

**VulKill の発見:** Spotify/Backstage 監査で見つかり、2026-11-01 に実際の upstream モジュールを用いたローカル実行テストで挙動を観測しました。Upstream advisory は 2026-08-28 に公開済みでした。
**Upstream:** [`GHSA-rg9r-hr7g-5gc2`](https://github.com/backstage/backstage/security/advisories/GHSA-rg9r-hr7g-5gc2) · `CVE-2026-106489` · CVSS 3.1 `6.5 Medium` · `CWE-862`。

## 1. 仕組み

TechDocs の静的コンテンツ経路は、二重スラッシュを含むパスを認可時とストレージ参照時で異なる形に正規化していました。権限フレームワークと外部ストレージを使う環境では、1 つの文書エンティティだけを読める利用者が別エンティティの内容へ到達できます。VulKill は実際の `createRouter`、`LocalPublish`、`AzureBlobStoragePublish` を合成マーカーで実行しました。

影響: `@backstage/plugin-techdocs-backend < 2.2.4`。修正版: `2.2.4`。

## 2. 現実的なシナリオ

- 他チーム限定の運用手順書が読まれる。
- 文書内の内部ホスト名や例示資格情報が露出する。
- UI のエンティティ権限と backend の静的配信境界が一致しなくなる。

## 3. 実行時の観測

```text
control: 低権限 + 通常パス -> 404
RED:     同じ利用者 + 細工したパス -> 200、保護マーカーを取得
GREEN:   修正後 -> 404
BENIGN:  許可済み利用者 + 通常パス -> 200
```

Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
Harness SHA-256: `30bdbb5fa044c7568985a06b2e27c498296d1883344b728b99919f6736b8a665`

## 4. 対策

`@backstage/plugin-techdocs-backend 2.2.4` 以降へ更新してください。
