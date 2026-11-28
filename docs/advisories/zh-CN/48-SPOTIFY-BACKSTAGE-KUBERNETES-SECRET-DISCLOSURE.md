# Spotify Backstage Kubernetes 资源敏感信息泄露 (`GHSA-p795-mqf2-36mf` / `CVE-2026-106561`)

[English](../48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [Türkçe](../tr/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [日本語](../ja/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | **简体中文** | [한국어](../ko/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md)

**VulKill 发现：**在 Spotify/Backstage 审计中发现，并于 2026-11-01 在真实 Kubernetes router/fetcher 的本地运行测试中观察到。上游公告已于 2026-08-28 发布。
**Kubernetes 部分的 VulKill 审计编号：**`NEW-BS-07B`。
**上游：**[`GHSA-p795-mqf2-36mf`](https://github.com/backstage/backstage/security/advisories/GHSA-p795-mqf2-36mf) · `CVE-2026-106561` · CVSS 3.1 `5.0 Medium` · `CWE-200`。

上方 GHSA 仅覆盖本记录的 Kubernetes secret disclosure 部分。下方 GitHub Events cache 是相关的 VulKill 审计观察，不作为该 GHSA 的一部分呈现。

## 1. 形成机制

Backstage 会遮蔽常规 `Secret.data`，但并未在所有查询路径中清除 Kubernetes 资源 annotation 中等价的秘密值。具有标准资源读取权限的用户可能收到本应隐藏的明文。VulKill 在真实 `KubernetesRouter`、`KubernetesProxy` 和 `KubernetesClientBasedFetcher` 路径中使用合成秘密标记观察了该行为。

受影响：`@backstage/plugin-kubernetes-backend < 0.21.9`。修复版本：`0.21.9`。

## 2. 现实场景

- annotation 中遗留的数据库密码对开发人员可见。
- 生成清单中的 CI/CD token 绕过遮蔽。
- 共享集群中其他内部服务的凭据暴露。

## 3. 运行观察

```text
control: 直接 secret proxy 请求 -> 403
RED:     允许的服务资源查询 -> 200；Secret.data 已遮蔽，但 annotation 标记可见
GREEN:   修复后同时遮蔽 secret-derived annotation
BENIGN:  非敏感元数据保持可用
```

Harness SHA-256: `d6acd23d33238a8dda450ca77e744f008094e415f80f4efdc39cb9a71071a9a8`

## 4. 相关 VulKill 观察：GitHub Events 签名前 Negative Cache (`NEW-BS-07A`)

在测试的 GitHub Events 路径中，`createAppIdResolver` 在校验 webhook HMAC 前执行 installation lookup，并把 `undefined` 结果无 TTL 缓存。伪造请求可在签名被拒绝前创建 negative cache entry；之后同一 installation ID 的正确签名 webhook 在进程重启前仍返回 `403`。

```text
control: resolver entry 已存在的正确签名 webhook -> 202
RED:     伪造 webhook -> 403；同一 ID 后续正确签名 webhook -> 403
GREEN:   先校验签名且不永久缓存 undefined -> 202
BENIGN:  有效 cached installation mapping 继续工作
```

- Audit ID: `NEW-BS-07A`
- Harness SHA-256: `51332b3066dd4c845480dbdd72a21acd1067e60831817ec8b595def31afbe048`
- Runtime chain SHA-256: `e41bfbbca3329aa88074c3c885234c7ce9ada5a54485296a76d82fd0f3a7b5d5`

## 5. 防护

升级到 `0.21.9` 或更高版本，并收紧 `kubernetes.resources.read` 与服务账户 RBAC。

GitHub Events 应在 installation lookup/cache 前校验 HMAC，不缓存 `undefined`；若业务确需 negative cache，应设置有限 TTL。
