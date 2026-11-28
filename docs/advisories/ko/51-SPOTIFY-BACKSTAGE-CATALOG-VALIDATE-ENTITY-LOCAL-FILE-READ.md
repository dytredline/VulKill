# Spotify Backstage Catalog `validate-entity` 로컬 파일 노출 (`GHSA-h8rc-g67j-87x3`)

[English](../51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [Türkçe](../tr/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [日本語](../ja/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [简体中文](../zh-CN/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | **한국어**

**VulKill 발견:** Spotify/Backstage 감사에서 발견했으며 2026-11-01 로컬 런타임 테스트에서 관찰했습니다. Upstream advisory는 2026-11-18 공개되었습니다.
**VulKill 감사 ID:** `NEW-BS-04`.
**Upstream:** [`GHSA-h8rc-g67j-87x3`](https://github.com/backstage/backstage/security/advisories/GHSA-h8rc-g67j-87x3) · CVSS 3.1 `6.5 Medium` · `CWE-200` / `CWE-863`.

## 1. 발생 방식

`POST /validate-entity`는 server가 제어하는 location을 설정한 뒤 caller 제공 annotations가 이를 덮어쓰도록 병합했습니다. Caller는 location context를 `file:`로 바꾸고 처리 경로를 backend file reader까지 도달시켜 YAML 진단을 통해 source line을 HTTP 응답에 노출할 수 있습니다. VulKill은 실제 router, validation service, orchestrator, `FileReaderProcessor` 경로를 실행했습니다.

영향 범위: `@backstage/plugin-catalog-backend < 3.9.3` 및 `>= 4.0.0, < 4.0.3`. 수정 버전: `3.9.3`, `4.0.3`, `4.0.4`, `4.0.5`.

## 2. 현실적 시나리오

- 일반 로그인 사용자가 backend 설정 파일 한 줄을 읽습니다.
- Parser error가 비밀을 포함한 source line을 응답에 반영합니다.
- Forged provenance annotation이 repository context 경계를 넘습니다.

## 3. 런타임 관찰

```text
control: 권한 없는 guest -> 403
RED:     validate 권한 + forged location annotation -> local marker 노출
GREEN:   server annotation 우선 적용, local read 차단
BENIGN:  정상 URL entity validation은 계속 동작
```

Harness SHA-256: `0bd744cadf97f031fd218d97eb187fa0394c7ec0b23d8c702fe7eec52f0403ce`

## 4. 방어

수정 버전으로 업데이트하고 그 전까지 `catalog.entity.validate`를 신뢰 사용자로 제한하십시오.
