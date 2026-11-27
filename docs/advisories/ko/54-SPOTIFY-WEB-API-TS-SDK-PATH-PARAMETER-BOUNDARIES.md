# Spotify Web API TypeScript SDK 경로 매개변수 경계 (`NEW-SDK-01`)

[English](../54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [Türkçe](../tr/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [日本語](../ja/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [简体中文](../zh-CN/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | **한국어**

**VulKill 발견:** Spotify 오픈소스 SDK 감사에서 찾았으며 2026-11-01 컴파일된 upstream SDK의 격리 로컬 HTTP 런타임 테스트에서 관찰했습니다. 이 기록은 `NEW-SDK-01`을 사용하며 upstream advisory ID를 지정하지 않습니다.

## 1. 코드 결함이 요청 경계 문제가 되는 방식

11개 endpoint class의 32개 method가 호출자 제공 resource ID를 단일 path segment로 encode하지 않고 URL path template에 삽입했습니다. 애플리케이션이 공격자 영향 아래의 ID를 전달하면 URL normalization이 도달 endpoint를 바꿀 수 있지만 SDK는 애플리케이션 bearer token을 계속 첨부합니다. VulKill은 loopback-only 합성 endpoint에서 32개 method 전체를 실행했으며 Spotify live API에는 요청하지 않았습니다.

영향은 소비 애플리케이션이 공격자 제어 ID를 허용하는지와 token scope에 따라 달라집니다.

## 2. 현실적 시나리오

- **A — Backend route confusion:** playlist ID를 받는 API wrapper가 다른 Web API route에 도달합니다.
- **B — State-changing request pivot:** mutation helper가 같은 token으로 허용된 다른 endpoint에 bearer 인증 요청을 보냅니다.
- **C — Data-read pivot:** read helper가 애플리케이션 의도와 다른 허용 route의 데이터를 반환합니다.

## 3. 런타임 관찰

```text
control: 일반 base62-like ID가 의도한 합성 route에 도달
RED:     32/32 method가 공격자 영향 ID를 대체 route로 normalize
GREEN:   중앙 path-segment 검증/encoding이 동일 입력 차단
BENIGN:  일반 ID는 방어 변경 후에도 호환
```

- Source commit: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- Harness SHA-256: `3e65e293ad938d9bf526027e6bcb1371e6f2340d6a94adfb7d83d7b1400d6685`
- Result SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Matrix: `32` methods (`10` state-changing, `22` read)

## 4. 방어

공유 request builder에서 각 resource ID를 하나의 path segment로 encode하고 URL 생성 전 traversal 및 encoded separator 형식을 거부하며 신뢰할 수 없는 요청에서 받은 ID에 소비자 측 allowlist를 유지하십시오.
