# Spotify SDK OAuth、PKCE 与会话 Token 边界 (`NEW-SDK-02`)

[English](../55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [Türkçe](../tr/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [日本語](../ja/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | **简体中文** | [한국어](../ko/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md)

**VulKill 发现：**在 Spotify 开源 TypeScript、iOS 与 Android SDK/example surface 中发现，并在隔离的本地、emulator 与真实设备环境中观察到。本记录使用 `NEW-SDK-02`，不指定上游安全公告编号。

## 1. 代码缺陷如何形成 OAuth 与会话边界问题

测试的 TypeScript SDK 使用 `Math.random()` 生成 PKCE verifier 材料，并在第一次读取时删除 verifier。Web Playback example 将最新 OAuth token 保存在 process-wide 变量中，并通过共享 token route 返回。受审计 mobile example 路径也可能在未一致绑定到发起 session 前接受 callback/deep-link token state 并替换已存储会话数据。

影响取决于采用者如何暴露 callback route、隔离 browser session 以及集成 example code。

## 2. 现实场景

- **场景 A — Premature verifier eviction：**callback 形态请求在用户正常 callback 完成前消费缓存的 PKCE verifier。
- **场景 B — Cross-session token mix-up：**两个用户共享 example server process，后一次登录替换返回给前一 session 的 token。
- **场景 C — Mobile callback confusion：**deep link 或 OAuth response 在没有绑定到发起 mobile session 的 `state` 时被接受。

## 3. 运行时观察

```text
RED:     callback-shaped request 在成功交换前消费缓存 verifier
RED:     两个合成用户观察到 example server 的 process-wide token replacement
RED:     mobile callback 路径在完整 session binding 前接受 state/token data
GREEN:   密码学随机数、成功后删除、每会话 token storage 与 state 校验阻止这些情况
BENIGN:  正确绑定的 OAuth callback 继续完成
```

- TypeScript commit: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- iOS auth commit: `1e92eb3ddd632a3ab3b2ed852057948f8ef05067`
- Android auth commit: `eef9af9ffed5ed4e5ed63e2c162131c8da711bba`
- Web Playback commit: `d22700174073dec831635aac452acbf5b4186853`
- Runtime SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Mobile harness SHA-256: `0035b900685a91318a76117f59b476512b7e6b7f85e5b46098b08244415e60a0`
- Device result SHA-256: `fe2a5ab870c226dc8f06ad9793b2bf774868092c36f35716ae8f36fd830a7a80`

## 4. 防护

为 PKCE 使用密码学随机源；在成功且绑定 `state` 的交换前保留 verifier；按已认证 session 存储 token；严格匹配 OAuth `state`；并为 example application 增加生产级 session control。
