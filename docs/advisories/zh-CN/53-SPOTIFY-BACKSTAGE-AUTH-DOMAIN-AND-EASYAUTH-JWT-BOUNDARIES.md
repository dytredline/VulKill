# Spotify Backstage 登录域名与 Azure EasyAuth JWT 边界 (`NEW-BS-06`)

[English](../53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [Türkçe](../tr/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [日本語](../ja/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | **简体中文** | [한국어](../ko/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md)

**VulKill 发现：**在 Spotify/Backstage 审计中发现，并于 2026-11-01 在真实上游认证及网关模块的隔离本地运行时测试中观察到。本记录使用 `NEW-BS-06`，不指定上游安全公告编号。

## 1. 代码缺陷如何形成认证边界问题

`emailMatchingUserEntityProfileEmail` resolver 的选项 schema 接受 `allowedDomains`，但登录时未执行该列表。在测试的 Azure EasyAuth 路径中，authenticator 解码转发的 ID-token header，却未在该边界校验密码学签名、issuer 与 audience。VulKill 使用合成身份和本地生成密钥运行了真实 resolver、gateway route、token factory、keystore 和 identity-client 路径。

域名行为要求使用受影响 resolver 并存在匹配 catalog user。EasyAuth 行为取决于部署是否允许客户端提供的身份 header 在可信 proxy 未替换的情况下到达 provider。

## 2. 现实场景

- **场景 A — 域名策略缺口：**配置只列内部域名，但外部域名 catalog identity 仍可登录。
- **场景 B — 可信 header 边界丢失：**部署把客户端提供的 identity-token header 转发到 EasyAuth provider。
- **场景 C — 冒用现有身份：**被接受 header 中的 claims 解析为高权限 catalog identity，并为其签发 Backstage session。

## 3. 运行时观察

```text
control: 相邻 resolver 拒绝外部域名合成身份
RED:     受影响 resolver 接受同一身份并签发 ES256 Backstage token
RED:     本地 gateway 将合成 unsigned ID token 转发给 EasyAuth provider
GREEN:   域名执行及签名/issuer/audience 校验拒绝两种情况
BENIGN:  内部域名身份与可信 proxy token 继续工作
```

- Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Domain harness SHA-256: `753692625e86b7007c02b5cf59e6d73c5ddc244cc6d1aeabe824480bafb2f2c8`
- EasyAuth harness SHA-256: `d3590c092c37f719926b4aec5de779ae6a612c8a45f9c057eb78bddc1aececca`
- Runtime chains: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`, `6778c3e716d8a3e1646dd98bbf94e5d55f16be37815058792f8873d8ea55228f`

## 4. 防护

在运行时执行 `allowedDomains`，用固定 issuer 与 audience 校验 EasyAuth JWT 签名，在公共 gateway 清除身份 header，并只接受来自已记录可信 reverse proxy 的 header。
