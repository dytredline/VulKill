# Spotify Backstage Catalog `validate-entity` 本地文件泄露 (`GHSA-h8rc-g67j-87x3`)

[English](../51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [Türkçe](../tr/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [日本語](../ja/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | **简体中文** | [한국어](../ko/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md)

**VulKill 发现：**在 Spotify/Backstage 审计中发现，并于 2026-11-01 的本地运行测试中观察到；上游公告于 2026-11-18 发布。
**VulKill 审计编号：**`NEW-BS-04`。
**上游：**[`GHSA-h8rc-g67j-87x3`](https://github.com/backstage/backstage/security/advisories/GHSA-h8rc-g67j-87x3) · CVSS 3.1 `6.5 Medium` · `CWE-200` / `CWE-863`。

## 1. 形成机制

`POST /validate-entity` 先设置服务端控制的位置，随后却让调用者提供的 annotations 覆盖它。调用者可把位置上下文替换为 `file:`，处理链到达 backend file reader，并通过 YAML 诊断把源文件行返回到 HTTP 响应。VulKill 执行了真实 router、validation service、orchestrator 和 `FileReaderProcessor` 路径。

受影响：`@backstage/plugin-catalog-backend < 3.9.3` 以及 `>= 4.0.0, < 4.0.3`。修复版本：`3.9.3`、`4.0.3`、`4.0.4`、`4.0.5`。

## 2. 现实场景

- 普通已登录用户读取 backend 配置文件中的一行。
- 解析错误把包含秘密的源文件行回显到响应。
- 伪造 provenance annotation 跨越 repository context 边界。

## 3. 运行观察

```text
control: 无权限 guest -> 403
RED:     validate 权限 + 伪造 location annotation -> 本地标记暴露
GREEN:   服务端 annotation 优先，阻止本地读取
BENIGN:  合法 URL entity validation 保持工作
```

Harness SHA-256: `0bd744cadf97f031fd218d97eb187fa0394c7ec0b23d8c702fe7eec52f0403ce`

## 4. 防护

升级到修复版本；升级前将 `catalog.entity.validate` 限制给受信用户。
