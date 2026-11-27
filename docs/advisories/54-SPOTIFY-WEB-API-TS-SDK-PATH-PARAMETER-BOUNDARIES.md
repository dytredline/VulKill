# Spotify Web API TypeScript SDK Path-Parameter Boundaries (`NEW-SDK-01`)

**English** | [Türkçe](tr/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [日本語](ja/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [简体中文](zh-CN/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [한국어](ko/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md)

**VulKill finding:** Found during the Spotify open-source SDK audit and observed against the compiled upstream SDK in isolated local HTTP runtime tests on 2026-11-01. This record uses the audit identifier `NEW-SDK-01` and does not assign an upstream advisory identifier.

## 1. How the Code Defect Becomes a Request-Boundary Problem

Thirty-two methods across eleven endpoint classes interpolated caller-provided resource identifiers into URL path templates without encoding each identifier as a single path segment. When an application passes attacker-influenced identifiers into those methods, URL normalization can change the endpoint reached while the SDK still attaches the application's bearer token. VulKill exercised all 32 methods against loopback-only synthetic endpoints; no request was sent to Spotify's live API.

Impact depends on the consuming application exposing an attacker-controlled identifier and on the scopes carried by its token.

## 2. Real-World Scenarios

- **Scenario A — Backend route confusion:** An API wrapper accepts a playlist identifier from a request and the normalized SDK path reaches a different Web API route.
- **Scenario B — State-changing request pivot:** A mutation helper sends a bearer-authenticated request to another endpoint permitted by the same token.
- **Scenario C — Data-read pivot:** A read helper returns data from a different authorized route than the application intended.

## 3. Runtime Observation

```text
control: ordinary base62-like identifiers reach their intended synthetic routes
RED:     32/32 tested methods normalize attacker-influenced identifiers to alternate routes
GREEN:   central path-segment validation/encoding blocks the same inputs
BENIGN:  ordinary identifiers remain compatible after the defensive change
```

- SDK source commit tested: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- Runtime harness SHA-256: `3e65e293ad938d9bf526027e6bcb1371e6f2340d6a94adfb7d83d7b1400d6685`
- Runtime result SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Matrix: `32` methods (`10` state-changing, `22` read)

## 4. Defensive Guidance

Encode every resource identifier as one path segment in a shared request builder, reject traversal and encoded-separator forms before URL construction, and keep consumer-side allowlists for identifiers accepted from untrusted requests.
