# Spotify Backstage Kubernetes リソースの秘密情報露出 (`GHSA-p795-mqf2-36mf` / `CVE-2026-106561`)

[English](../48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [Türkçe](../tr/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | **日本語** | [简体中文](../zh-CN/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [한국어](../ko/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md)

**VulKill の発見:** Spotify/Backstage 監査で見つかり、2026-11-01 に実際の Kubernetes router/fetcher を用いたローカル実行テストで観測しました。Upstream advisory は 2026-08-28 に公開済みでした。
**Kubernetes 部分の VulKill audit ID:** `NEW-BS-07B`。
**Upstream:** [`GHSA-p795-mqf2-36mf`](https://github.com/backstage/backstage/security/advisories/GHSA-p795-mqf2-36mf) · `CVE-2026-106561` · CVSS 3.1 `5.0 Medium` · `CWE-200`。

上記の GHSA は本記録の Kubernetes secret disclosure 部分のみを対象とします。以下の GitHub Events cache は関連する VulKill 監査所見であり、この GHSA の一部として提示するものではありません。

## 1. 仕組み

通常の `Secret.data` はマスクされても、Kubernetes リソースの annotation に残る同等の秘密値がすべての照会経路で除去されませんでした。標準のリソース読取権限を持つ利用者が、隠されるはずの平文値を受け取れます。VulKill は実際の `KubernetesRouter`、`KubernetesProxy`、`KubernetesClientBasedFetcher` で合成マーカーを観測しました。

影響: `@backstage/plugin-kubernetes-backend < 0.21.9`。修正版: `0.21.9`。

## 2. 現実的なシナリオ

- annotation に残った DB パスワードが開発者へ表示される。
- 生成 manifest の CI/CD token がマスクを回避する。
- 共有 cluster 内の別サービス資格情報が露出する。

## 3. 実行時の観測

```text
control: secret への直接 proxy -> 403
RED:     許可済みサービス照会 -> 200、Secret.data はマスク済みだが annotation marker は表示
GREEN:   修正後は秘密由来 annotation もマスク
BENIGN:  非機密 metadata は利用可能
```

Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
Harness SHA-256: `d6acd23d33238a8dda450ca77e744f008094e415f80f4efdc39cb9a71071a9a8`

## 4. 関連する VulKill 所見: GitHub Events の署名前 negative cache (`NEW-BS-07A`)

テスト対象の GitHub Events 経路では、`createAppIdResolver` が webhook HMAC 検査前に installation lookup を行い、`undefined` を TTL なしで cache しました。偽造 request が署名拒否前に negative entry を作ると、同じ installation ID の正しく署名された webhook も process restart まで `403` を受け続けました。

```text
control: resolver entry がある正規署名 webhook -> 202
RED:     forged webhook -> 403; 同じ ID の後続正規 webhook -> 403
GREEN:   signature-first、undefined を永続 cache しない -> 202
BENIGN:  有効な cached installation mapping は継続動作
```

- Audit ID: `NEW-BS-07A`
- Harness SHA-256: `51332b3066dd4c845480dbdd72a21acd1067e60831817ec8b595def31afbe048`
- Runtime chain SHA-256: `e41bfbbca3329aa88074c3c885234c7ce9ada5a54485296a76d82fd0f3a7b5d5`

## 5. 対策

`0.21.9` 以降へ更新し、`kubernetes.resources.read` と service-account RBAC を最小化してください。

GitHub Events では installation lookup/cache 前に HMAC を検査し、`undefined` を cache せず、negative cache が必要な場合は有限 TTL を設定してください。
