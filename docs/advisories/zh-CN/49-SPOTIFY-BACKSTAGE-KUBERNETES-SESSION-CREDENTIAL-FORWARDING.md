# Spotify Backstage Kubernetes Proxy 转发会话凭据 (`GHSA-27gw-fw83-xfpp`)

[English](../49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [Türkçe](../tr/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [日本語](../ja/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | **简体中文** | [한국어](../ko/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md)

**VulKill 发现：**在 Spotify/Backstage 审计中发现，并于 2026-11-01 的本地运行测试中观察到；上游公告于 2026-11-18 发布。
**上游：**[`GHSA-27gw-fw83-xfpp`](https://github.com/backstage/backstage/security/advisories/GHSA-27gw-fw83-xfpp) · CVSS 3.1 `6.4 Medium` · `CWE-200` / `CWE-441`。

## 1. 形成机制

当所选集群认证策略不替换请求头时，Kubernetes proxy 会把调用者的 `Authorization` 和 `Cookie` 转发给上游。若 catalog contributor 能影响集群 API URL，受害者打开该实体的 Kubernetes 页面时，Backstage 会话凭据会发送到 contributor 控制的端点。VulKill 执行了真实 `CatalogClusterLocator -> KubernetesRouter` 路径。

受影响：`@backstage/plugin-kubernetes-backend >= 0.9.0, < 0.21.13`。修复版本：`0.21.13`，受支持回移版本 `0.21.14` / `0.21.15`。

## 2. 现实场景

- 低权限 contributor 添加受控集群实体，管理员查看时其 token 被发送到该端点。
- 获取的短期 token 被用于以受害者身份访问其他 Backstage 插件。
- Cookie 认证部署把受害者 session cookie 转发给上游。

## 3. 运行观察

```text
control: 运维方控制的集群只收到配置的集群凭据
RED:     contributor 影响的集群收到调用者 Authorization/Cookie
GREEN:   proxy 在上游请求前移除调用者会话凭据
BENIGN:  显式 Backstage-Kubernetes-Authorization 保持工作
```

Runtime-chain SHA-256: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`

## 4. 防护

升级到修复版本，限制集群实体创建与 `kubernetes.proxy` 权限。
