# Spotify Backstage 로그인 도메인 및 Azure EasyAuth JWT 경계 (`NEW-BS-06`)

[English](../53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [Türkçe](../tr/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [日本語](../ja/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [简体中文](../zh-CN/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | **한국어**

**VulKill 발견:** Spotify/Backstage 감사에서 찾았으며 2026-11-01 실제 upstream 인증 및 gateway 모듈을 사용한 격리 로컬 런타임 테스트에서 관찰했습니다. 이 기록은 `NEW-BS-06`을 사용하며 upstream advisory ID를 지정하지 않습니다.

## 1. 코드 결함이 인증 경계가 되는 방식

`emailMatchingUserEntityProfileEmail` resolver는 option schema에서 `allowedDomains`를 받지만 로그인 시 목록을 적용하지 않았습니다. 테스트한 Azure EasyAuth 경로에서 authenticator는 전달된 ID-token header를 해당 경계의 암호학적 서명, issuer, audience 검사 없이 decode했습니다. VulKill은 합성 identity와 로컬 생성 key로 실제 resolver, gateway route, token factory, keystore, identity-client 경로를 실행했습니다.

도메인 동작에는 영향받는 resolver와 일치하는 catalog user가 필요합니다. EasyAuth 동작은 client-supplied identity header가 trusted proxy에서 교체되지 않고 provider에 도달할 수 있는 배포 경로에 따라 달라집니다.

## 2. 현실적 시나리오

- **A — 도메인 정책 공백:** 설정이 내부 도메인만 나열해도 외부 도메인 catalog identity가 로그인합니다.
- **B — 신뢰 header 경계 상실:** client-supplied identity-token header가 EasyAuth provider로 전달됩니다.
- **C — 기존 identity 사칭:** 허용된 header claim이 고권한 catalog identity로 해석되고 해당 identity의 Backstage session이 발급됩니다.

## 3. 런타임 관찰

```text
control: sibling resolver가 외부 도메인 합성 identity 거부
RED:     영향받는 resolver가 동일 identity를 수락하고 ES256 token 발급
RED:     로컬 gateway가 합성 unsigned ID token을 EasyAuth provider로 전달
GREEN:   domain enforcement 및 서명/issuer/audience 검사가 두 경우 거부
BENIGN:  내부 identity와 trusted-proxy token은 계속 동작
```

- Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Domain harness SHA-256: `753692625e86b7007c02b5cf59e6d73c5ddc244cc6d1aeabe824480bafb2f2c8`
- EasyAuth harness SHA-256: `d3590c092c37f719926b4aec5de779ae6a612c8a45f9c057eb78bddc1aececca`
- Runtime chains: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`, `6778c3e716d8a3e1646dd98bbf94e5d55f16be37815058792f8873d8ea55228f`

## 4. 방어

런타임에 `allowedDomains`를 적용하고 고정 issuer/audience로 EasyAuth JWT 서명을 검증하며 공개 gateway에서 identity header를 제거하고 문서화된 trusted reverse proxy에서만 해당 header를 수락하십시오.
