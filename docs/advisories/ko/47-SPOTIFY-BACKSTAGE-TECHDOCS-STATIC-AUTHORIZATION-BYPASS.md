# Spotify Backstage TechDocs 정적 콘텐츠 권한 우회 (`GHSA-rg9r-hr7g-5gc2` / `CVE-2026-106489`)

[English](../47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [Türkçe](../tr/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [日本語](../ja/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [简体中文](../zh-CN/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | **한국어**

**VulKill 발견:** Spotify/Backstage 감사에서 발견했으며 2026-11-01 실제 upstream 모듈을 사용한 로컬 런타임 테스트에서 해당 동작을 관찰했습니다. Upstream advisory는 2026-08-28 공개되었습니다.
**VulKill 감사 ID:** `NEW-BS-02`.
**Upstream:** [`GHSA-rg9r-hr7g-5gc2`](https://github.com/backstage/backstage/security/advisories/GHSA-rg9r-hr7g-5gc2) · `CVE-2026-106489` · CVSS 3.1 `6.5 Medium` · `CWE-862`.

## 1. 발생 방식

TechDocs 정적 콘텐츠 라우트는 이중 슬래시가 포함된 경로를 권한 검사와 저장소 조회에서 다르게 정규화했습니다. 권한 프레임워크와 외부 저장소를 사용하는 환경에서는 한 문서 엔터티만 읽을 수 있는 사용자가 다른 엔터티 콘텐츠에 접근할 수 있습니다. VulKill은 실제 `createRouter`, `LocalPublish`, `AzureBlobStoragePublish` 모듈과 합성 마커로 이 경로를 관찰했습니다.

영향 범위: `@backstage/plugin-techdocs-backend < 2.2.4`. 수정 버전: `2.2.4`.

## 2. 현실적 시나리오

- 다른 팀의 제한된 운영 문서를 읽습니다.
- 문서의 내부 호스트명이나 예시 자격 증명이 노출됩니다.
- 공유 Backstage에서 UI 권한과 backend 정적 콘텐츠 경계가 달라집니다.

## 3. 런타임 관찰

```text
control: 낮은 권한 + 정상 경로 -> 404
RED:     같은 사용자 + 조작 경로 -> 200, 보호 마커 반환
GREEN:   수정 후 -> 404
BENIGN:  허용 사용자 + 정상 경로 -> 200
```

Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
Harness SHA-256: `30bdbb5fa044c7568985a06b2e27c498296d1883344b728b99919f6736b8a665`

## 4. 방어

`@backstage/plugin-techdocs-backend 2.2.4` 이상으로 업데이트하십시오.
