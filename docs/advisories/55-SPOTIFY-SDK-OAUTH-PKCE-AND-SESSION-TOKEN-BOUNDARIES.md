# Spotify SDK OAuth, PKCE, and Session-Token Boundaries (`NEW-SDK-02`)

**English** | [Türkçe](tr/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [日本語](ja/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [简体中文](zh-CN/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [한국어](ko/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md)

**VulKill finding:** Found across Spotify's open-source TypeScript, iOS, and Android SDK/example surfaces and observed in isolated local, emulator, and physical-device test environments. This record uses the audit identifier `NEW-SDK-02` and does not assign an upstream advisory identifier.

## 1. How the Code Defects Become OAuth and Session Boundaries

The tested TypeScript SDK generated PKCE verifier material with `Math.random()` and stored the verifier with first-read eviction. The Web Playback example stored the latest OAuth token in one process-wide variable and exposed it through a shared token route. The audited mobile example paths also accepted callback or deep-link token state without consistently binding it to the initiating session before replacing stored session data.

These observations concern the tested SDK/example commits and their integration patterns. Application impact depends on how an adopter exposes callback routes, isolates browser sessions, and incorporates the example code.

## 2. Real-World Scenarios

- **Scenario A — Premature verifier eviction:** A callback-shaped request consumes the cached PKCE verifier before the user's legitimate callback completes.
- **Scenario B — Cross-session token mix-up:** Two users share one example server process and the later login replaces the token returned to the earlier session.
- **Scenario C — Mobile callback confusion:** A deep link or OAuth response is accepted without a state value bound to the initiating mobile session.

## 3. Runtime Observation

```text
RED:     callback-shaped request consumes the cached verifier before successful exchange
RED:     two synthetic users observe process-wide token replacement in the example server
RED:     tested mobile callback paths accept state/token data before complete session binding
GREEN:   cryptographic randomness, post-success eviction, per-session token storage, and state checks block the cases
BENIGN:  correctly bound OAuth callbacks continue to complete
```

- TypeScript SDK source commit: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- iOS auth source commit: `1e92eb3ddd632a3ab3b2ed852057948f8ef05067`
- Android auth source commit: `eef9af9ffed5ed4e5ed63e2c162131c8da711bba`
- Web Playback example source commit: `d22700174073dec831635aac452acbf5b4186853`
- Runtime result SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Mobile linkage harness SHA-256: `0035b900685a91318a76117f59b476512b7e6b7f85e5b46098b08244415e60a0`
- Physical-device result SHA-256: `fe2a5ab870c226dc8f06ad9793b2bf774868092c36f35716ae8f36fd830a7a80`

## 4. Defensive Guidance

Use a cryptographic random source for PKCE, retain the verifier until a successful state-bound exchange, store tokens per authenticated session, require exact OAuth `state` matching, and treat example applications as integration references that still need production session controls.
