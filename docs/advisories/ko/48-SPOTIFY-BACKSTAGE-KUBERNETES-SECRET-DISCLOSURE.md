# Spotify Backstage Kubernetes 리소스 비밀 정보 노출 (`GHSA-p795-mqf2-36mf` / `CVE-2026-106561`)

[English](../48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [Türkçe](../tr/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [日本語](../ja/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [简体中文](../zh-CN/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | **한국어**

**VulKill 발견:** Spotify/Backstage 감사에서 발견했으며 2026-11-01 실제 Kubernetes router/fetcher의 로컬 런타임 테스트에서 관찰했습니다. Upstream advisory는 2026-08-28 공개되었습니다.
**Kubernetes 부분의 VulKill 감사 ID:** `NEW-BS-07B`.
**Upstream:** [`GHSA-p795-mqf2-36mf`](https://github.com/backstage/backstage/security/advisories/GHSA-p795-mqf2-36mf) · `CVE-2026-106561` · CVSS 3.1 `5.0 Medium` · `CWE-200`.

위 GHSA는 이 기록의 Kubernetes secret disclosure 부분만 다룹니다. 아래 GitHub Events cache는 관련 VulKill 감사 관찰이며 해당 GHSA의 일부로 제시하지 않습니다.

## 1. 발생 방식

Backstage는 일반 `Secret.data`를 마스킹하지만 Kubernetes 리소스 annotation에 남은 동일한 비밀 값을 모든 조회 경로에서 제거하지 않았습니다. 표준 리소스 읽기 권한을 가진 사용자가 숨겨져야 할 평문 값을 받을 수 있습니다. VulKill은 실제 `KubernetesRouter`, `KubernetesProxy`, `KubernetesClientBasedFetcher` 경로에서 합성 비밀 마커를 관찰했습니다.

영향 범위: `@backstage/plugin-kubernetes-backend < 0.21.9`. 수정 버전: `0.21.9`.

## 2. 현실적 시나리오

- annotation에 남은 DB 암호가 개발자에게 표시됩니다.
- 생성 manifest의 CI/CD token이 마스킹을 우회합니다.
- 공유 cluster의 다른 내부 서비스 자격 증명이 노출됩니다.

## 3. 런타임 관찰

```text
control: secret 직접 proxy 요청 -> 403
RED:     허용 서비스 리소스 조회 -> 200; Secret.data는 마스킹되지만 annotation 마커 노출
GREEN:   수정 후 secret-derived annotation도 마스킹
BENIGN:  비민감 metadata는 계속 사용 가능
```

Harness SHA-256: `d6acd23d33238a8dda450ca77e744f008094e415f80f4efdc39cb9a71071a9a8`

## 4. 관련 VulKill 관찰: GitHub Events 서명 전 Negative Cache (`NEW-BS-07A`)

테스트한 GitHub Events 경로에서 `createAppIdResolver`는 webhook HMAC 검증 전에 installation lookup을 수행하고 `undefined` 결과를 TTL 없이 cache했습니다. 위조 요청이 서명 거부 전에 negative entry를 만들면 동일 installation ID의 올바르게 서명된 webhook도 process restart 전까지 계속 `403`을 받았습니다.

```text
control: resolver entry가 있는 올바른 서명 webhook -> 202
RED:     forged webhook -> 403; 같은 ID의 후속 올바른 webhook -> 403
GREEN:   signature-first 처리와 undefined 영구 cache 금지 -> 202
BENIGN:  유효 cached installation mapping은 계속 동작
```

- Audit ID: `NEW-BS-07A`
- Harness SHA-256: `51332b3066dd4c845480dbdd72a21acd1067e60831817ec8b595def31afbe048`
- Runtime chain SHA-256: `e41bfbbca3329aa88074c3c885234c7ce9ada5a54485296a76d82fd0f3a7b5d5`

## 5. 방어

`0.21.9` 이상으로 업데이트하고 `kubernetes.resources.read` 및 service-account RBAC를 최소화하십시오.

GitHub Events에서는 installation lookup/cache 전에 HMAC을 검증하고 `undefined`를 cache하지 마십시오. negative cache가 필요하면 제한된 TTL을 적용하십시오.
