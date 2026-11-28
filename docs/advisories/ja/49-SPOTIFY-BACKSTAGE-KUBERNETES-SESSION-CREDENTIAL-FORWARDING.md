# Spotify Backstage Kubernetes Proxy のセッション資格情報転送 (`GHSA-27gw-fw83-xfpp`)

[English](../49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [Türkçe](../tr/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | **日本語** | [简体中文](../zh-CN/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [한국어](../ko/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md)

**VulKill の発見:** Spotify/Backstage 監査で見つかり、2026-11-01 のローカル実行テストで観測しました。Upstream advisory は 2026-11-18 に公開されました。
**VulKill audit ID:** `NEW-BS-01`。
**Upstream:** [`GHSA-27gw-fw83-xfpp`](https://github.com/backstage/backstage/security/advisories/GHSA-27gw-fw83-xfpp) · CVSS 3.1 `6.4 Medium` · `CWE-200` / `CWE-441`。

## 1. 仕組み

選択した cluster 認証方式が置換しない場合、Kubernetes proxy は呼出元の `Authorization` と `Cookie` を upstream へ転送していました。Catalog contributor が cluster API URL に影響できる構成では、被害者が Kubernetes 画面を開くと Backstage セッション資格情報が contributor 管理 endpoint へ送信されます。VulKill は実際の `CatalogClusterLocator -> KubernetesRouter` 経路を実行しました。

影響: `@backstage/plugin-kubernetes-backend >= 0.9.0, < 0.21.13`。修正版: `0.21.13`、backport `0.21.14` / `0.21.15`。

## 2. 現実的なシナリオ

- 低権限 contributor が管理 cluster entity を登録し、閲覧した管理者の token を受信する。
- 取得した短期 token で他の Backstage plugin に被害者としてアクセスする。
- Cookie 認証環境で被害者の session cookie が upstream へ転送される。

## 3. 実行時の観測

```text
control: 運用者管理 cluster -> 設定済み cluster credential のみ
RED:     contributor 影響 cluster -> 呼出元 Authorization/Cookie を受信
GREEN:   proxy が呼出元 session credential を除去
BENIGN:  明示的 Backstage-Kubernetes-Authorization は動作
```

Runtime-chain SHA-256: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`

## 4. 対策

修正版へ更新し、cluster entity 作成権限と `kubernetes.proxy` 権限を制限してください。
