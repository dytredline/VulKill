# Spotify Backstage Kubernetes Proxy 세션 자격 증명 전달 (`GHSA-27gw-fw83-xfpp`)

[English](../49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [Türkçe](../tr/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [日本語](../ja/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [简体中文](../zh-CN/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | **한국어**

**VulKill 발견:** Spotify/Backstage 감사에서 발견했으며 2026-11-01 로컬 런타임 테스트에서 관찰했습니다. Upstream advisory는 2026-11-18 공개되었습니다.
**Upstream:** [`GHSA-27gw-fw83-xfpp`](https://github.com/backstage/backstage/security/advisories/GHSA-27gw-fw83-xfpp) · CVSS 3.1 `6.4 Medium` · `CWE-200` / `CWE-441`.

## 1. 발생 방식

선택한 cluster 인증 전략이 요청 헤더를 교체하지 않으면 Kubernetes proxy가 호출자의 `Authorization` 및 `Cookie`를 upstream으로 전달했습니다. Catalog contributor가 cluster API URL에 영향을 줄 수 있는 구성에서는 피해자가 Kubernetes 화면을 열 때 Backstage 세션 자격 증명이 contributor가 제어하는 endpoint로 전송됩니다. VulKill은 실제 `CatalogClusterLocator -> KubernetesRouter` 경로를 실행했습니다.

영향 범위: `@backstage/plugin-kubernetes-backend >= 0.9.0, < 0.21.13`. 수정 버전: `0.21.13`, backport `0.21.14` / `0.21.15`.

## 2. 현실적 시나리오

- 낮은 권한 contributor가 제어 cluster entity를 추가하고, 관리자가 열 때 token을 수신합니다.
- 획득한 단기 token을 다른 Backstage plugin에서 피해자 신원으로 사용합니다.
- Cookie 인증 배포가 피해자의 session cookie를 upstream에 전달합니다.

## 3. 런타임 관찰

```text
control: 운영자 제어 cluster -> 설정된 cluster credential만 수신
RED:     contributor 영향 cluster -> 호출자 Authorization/Cookie 수신
GREEN:   proxy가 호출자 session credential 제거
BENIGN:  명시적 Backstage-Kubernetes-Authorization은 계속 동작
```

Runtime-chain SHA-256: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`

## 4. 방어

수정 버전으로 업데이트하고 cluster entity 생성 및 `kubernetes.proxy` 권한을 제한하십시오.
