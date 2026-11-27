# Spotify Backstage GitHub URL Reader のエンコード済み traversal (`GHSA-fp7g-2pw8-726f`)

[English](../50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [Türkçe](../tr/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | **日本語** | [简体中文](../zh-CN/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [한국어](../ko/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md)

**VulKill の発見:** Spotify/Backstage 監査で見つかり、2026-11-01 のローカル実行テストで観測しました。Upstream advisory は 2026-11-27 に公開されました。
**VulKill audit ID:** `NEW-BS-03`。
**Upstream:** [`GHSA-fp7g-2pw8-726f`](https://github.com/backstage/backstage/security/advisories/GHSA-fp7g-2pw8-726f) · CVSS 3.1 `7.7 High` · `CWE-22` / `CWE-200`。

## 1. 仕組み

GitHub URL Reader はデコード済み file path を検査しましたが、repository owner へ同等の反復デコードと delimiter 検査を適用していませんでした。検査を通った encoded traversal が GitHub API URL へ補間されると、資格情報付き read を別 API path へ向けられます。VulKill は実際の `GithubUrlReader`、`getGithubFileFetchUrl`、`parseGitUrlSafe` で controlled canary への到達を観測しました。

影響: `@backstage/integration >= 0.1.3, < 2.3.0`。修正版: `2.3.0`。

## 2. 現実的なシナリオ

- 共有 GitHub App token が読める private source を template 利用者が取得する。
- private workflow log から deployment secret が露出する。
- 組織全体 token により利用者自身の権限を超える範囲が読まれる。

## 3. 実行時の観測

```text
control: 通常 URL -> 想定 repository contents endpoint
RED:     encoded owner -> integration credential 付きの別 GitHub API path
GREEN:   反復デコードと delimiter/traversal 検査で拒否
BENIGN:  通常 URL は動作
```

Harness SHA-256: `dcf9f78d66b543e438dfa311509a481d78955070dd882a15bb6445552d0fd2c3`

## 4. 対策

`@backstage/integration 2.3.0` 以降へ更新し、共有 integration token を最小権限にしてください。
