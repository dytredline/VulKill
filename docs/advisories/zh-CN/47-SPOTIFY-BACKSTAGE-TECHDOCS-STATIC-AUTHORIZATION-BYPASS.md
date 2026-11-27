# Spotify Backstage TechDocs 静态内容授权绕过 (`GHSA-rg9r-hr7g-5gc2` / `CVE-2026-106489`)

[English](../47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [Türkçe](../tr/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [日本語](../ja/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | **简体中文** | [한국어](../ko/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md)

**VulKill 发现：**在 Spotify/Backstage 审计中发现，并于 2026-11-01 使用真实上游模块进行本地运行测试时观察到该行为。上游公告已于 2026-08-28 发布。
**上游：**[`GHSA-rg9r-hr7g-5gc2`](https://github.com/backstage/backstage/security/advisories/GHSA-rg9r-hr7g-5gc2) · `CVE-2026-106489` · CVSS 3.1 `6.5 Medium` · `CWE-862`。

## 1. 形成机制

TechDocs 静态内容路由在授权检查与存储查找阶段对包含双斜线的路径进行了不同的规范化。启用权限框架和外部存储时，只能读取一个文档实体的用户可能访问另一个实体的内容。VulKill 使用真实的 `createRouter`、`LocalPublish` 和 `AzureBlobStoragePublish` 模块及合成标记观察了该路径。

受影响：`@backstage/plugin-techdocs-backend < 2.2.4`。修复版本：`2.2.4`。

## 2. 现实场景

- 开发人员读取其他团队受限的运行手册。
- 文档中的内部主机名或示例凭据暴露。
- 共享 Backstage 中 UI 权限与后端静态内容边界不一致。

## 3. 运行观察

```text
control: 低权限用户 + 标准路径 -> 404
RED:     同一用户 + 构造路径 -> 200，返回受保护标记
GREEN:   修复后 -> 404
BENIGN:  已授权用户 + 标准路径 -> 200
```

Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
Harness SHA-256: `30bdbb5fa044c7568985a06b2e27c498296d1883344b728b99919f6736b8a665`

## 4. 防护

升级到 `@backstage/plugin-techdocs-backend 2.2.4` 或更高版本。
