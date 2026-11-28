# Spotify SDK OAuth, PKCE 및 세션 Token 경계 (`NEW-SDK-02`)

[English](../55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [Türkçe](../tr/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [日本語](../ja/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [简体中文](../zh-CN/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | **한국어**

**VulKill 발견:** Spotify 오픈소스 TypeScript, iOS, Android SDK/example surface에서 찾았으며 격리 로컬, emulator, 실제 device 환경에서 관찰했습니다. 이 기록은 `NEW-SDK-02`를 사용하며 upstream advisory ID를 지정하지 않습니다.

## 1. 코드 결함이 OAuth 및 세션 경계가 되는 방식

테스트한 TypeScript SDK는 `Math.random()`으로 PKCE verifier material을 만들고 첫 읽기에서 verifier를 제거했습니다. Web Playback example은 최신 OAuth token을 process-wide 변수 하나에 저장하고 공유 token route로 노출했습니다. 감사한 mobile example 경로도 callback/deep-link token state를 시작 session에 일관되게 binding하기 전에 저장된 세션 데이터를 교체할 수 있었습니다.

영향은 채택 애플리케이션이 callback route를 노출하는 방법, browser session 격리, example code 통합 방식에 따라 달라집니다.

## 2. 현실적 시나리오

- **A — Premature verifier eviction:** callback 형태 요청이 정상 callback 완료 전에 cached PKCE verifier를 소비합니다.
- **B — Cross-session token mix-up:** 두 사용자가 example server process 하나를 공유하며 나중 login이 이전 session의 token을 교체합니다.
- **C — Mobile callback confusion:** 시작 mobile session에 binding된 `state` 없이 deep link/OAuth response가 수락됩니다.

## 3. 런타임 관찰

```text
RED:     callback-shaped request가 성공적 교환 전 cached verifier 소비
RED:     두 합성 사용자가 example server의 process-wide token replacement 관찰
RED:     mobile callback 경로가 완전한 session binding 전 state/token data 수락
GREEN:   암호학적 난수, 성공 후 제거, session별 token storage, state 검사가 차단
BENIGN:  올바르게 binding된 OAuth callback은 계속 완료
```

- TypeScript commit: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- iOS auth commit: `1e92eb3ddd632a3ab3b2ed852057948f8ef05067`
- Android auth commit: `eef9af9ffed5ed4e5ed63e2c162131c8da711bba`
- Web Playback commit: `d22700174073dec831635aac452acbf5b4186853`
- Runtime SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Mobile harness SHA-256: `0035b900685a91318a76117f59b476512b7e6b7f85e5b46098b08244415e60a0`
- Device result SHA-256: `fe2a5ab870c226dc8f06ad9793b2bf774868092c36f35716ae8f36fd830a7a80`

## 4. 방어

PKCE에 암호학적 난수를 사용하고 성공적인 state-bound exchange까지 verifier를 유지하며 인증 session별로 token을 저장하고 OAuth `state`를 정확히 일치시키십시오. example application에도 production session control이 필요합니다.
