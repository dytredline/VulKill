# Spotify Backstage GitHub URL Reader 编码路径穿越 (`GHSA-fp7g-2pw8-726f`)

[English](../50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [Türkçe](../tr/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [日本語](../ja/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | **简体中文** | [한국어](../ko/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md)

**VulKill 发现：**在 Spotify/Backstage 审计中发现，并于 2026-11-01 的本地运行测试中观察到；上游公告于 2026-11-27 发布。
**上游：**[`GHSA-fp7g-2pw8-726f`](https://github.com/backstage/backstage/security/advisories/GHSA-fp7g-2pw8-726f) · CVSS 3.1 `7.7 High` · `CWE-22` / `CWE-200`。

## 1. 形成机制

GitHub URL Reader 检查了解码后的文件路径，却没有对 repository owner 应用同等的重复解码与分隔符检查。编码穿越通过检查后，被插入 GitHub API URL，使带凭据的读取指向其他 API 路径。VulKill 在真实 `GithubUrlReader`、`getGithubFileFetchUrl` 和 `parseGitUrlSafe` 路径中观察到请求到达受控 canary 端点。

受影响：`@backstage/integration >= 0.1.3, < 2.3.0`。修复版本：`2.3.0`。

## 2. 现实场景

- 模板用户读取共享 GitHub App token 可访问的私有源码。
- 私有 workflow 日志中的部署秘密暴露。
- 组织级集成 token 扩大到攻击者自身 GitHub 权限之外。

## 3. 运行观察

```text
control: 标准文件 URL -> 预期 repository contents 端点
RED:     编码 owner -> 携带集成凭据的其他 GitHub API 路径
GREEN:   重复解码及分隔符/穿越检查拒绝 URL
BENIGN:  标准 URL 保持工作
```

Harness SHA-256: `dcf9f78d66b543e438dfa311509a481d78955070dd882a15bb6445552d0fd2c3`

## 4. 防护

升级到 `@backstage/integration 2.3.0` 或更高版本，并最小化共享集成 token 权限。
