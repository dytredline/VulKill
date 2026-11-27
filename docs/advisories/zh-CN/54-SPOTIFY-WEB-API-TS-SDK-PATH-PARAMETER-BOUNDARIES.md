# Spotify Web API TypeScript SDK 路径参数边界 (`NEW-SDK-01`)

[English](../54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [Türkçe](../tr/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [日本語](../ja/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | **简体中文** | [한국어](../ko/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md)

**VulKill 发现：**在 Spotify 开源 SDK 审计中发现，并于 2026-11-01 使用编译后的上游 SDK 在隔离本地 HTTP 运行测试中观察到。本记录使用 `NEW-SDK-01`，不指定上游安全公告编号。

## 1. 代码缺陷如何形成请求边界问题

11 个 endpoint class 中的32个方法把调用者提供的资源 ID 插入 URL path template，却没有把每个 ID 编码为单一路径段。当应用向这些方法传入受攻击者影响的 ID 时，URL normalization 可改变实际到达的 endpoint，而 SDK 仍附加应用 bearer token。VulKill 对仅限 loopback 的合成 endpoint 测试了全部32个方法；没有请求发送到 Spotify 在线 API。

影响取决于消费应用是否暴露攻击者可控 ID，以及 token 拥有的 scope。

## 2. 现实场景

- **场景 A — Backend route confusion：**接收 playlist ID 的 API wrapper 经规范化 SDK 路径到达不同 Web API route。
- **场景 B — State-changing request pivot：**修改 helper 向同一 token 允许的另一 endpoint 发送 bearer 认证请求。
- **场景 C — Data-read pivot：**读取 helper 返回与应用意图不同但获授权 route 的数据。

## 3. 运行时观察

```text
control: 普通 base62-like ID 到达预期合成 route
RED:     32/32 方法将攻击者影响的 ID 规范化到替代 route
GREEN:   中央路径段验证/编码拦截同样输入
BENIGN:  普通 ID 在防护变更后保持兼容
```

- Source commit: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- Harness SHA-256: `3e65e293ad938d9bf526027e6bcb1371e6f2340d6a94adfb7d83d7b1400d6685`
- Result SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Matrix: `32` methods (`10` state-changing, `22` read)

## 4. 防护

在共享 request builder 中把每个资源 ID 编码为单一路径段；URL 构建前拒绝 traversal 和编码分隔符形式；对来自不可信请求的 ID 保留消费端允许列表。
