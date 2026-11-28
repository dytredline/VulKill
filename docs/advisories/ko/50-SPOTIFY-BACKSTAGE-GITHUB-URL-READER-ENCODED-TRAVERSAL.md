# Spotify Backstage GitHub URL Reader 인코딩 경로 탐색 (`GHSA-fp7g-2pw8-726f`)

[English](../50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [Türkçe](../tr/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [日本語](../ja/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [简体中文](../zh-CN/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | **한국어**

**VulKill 발견:** Spotify/Backstage 감사에서 발견했으며 2026-11-01 로컬 런타임 테스트에서 관찰했습니다. Upstream advisory는 2026-11-27 공개되었습니다.
**VulKill 감사 ID:** `NEW-BS-03`.
**Upstream:** [`GHSA-fp7g-2pw8-726f`](https://github.com/backstage/backstage/security/advisories/GHSA-fp7g-2pw8-726f) · CVSS 3.1 `7.7 High` · `CWE-22` / `CWE-200`.

## 1. 발생 방식

GitHub URL Reader는 디코딩된 file path를 검사했지만 repository owner에는 동일한 반복 디코딩과 delimiter 검사를 적용하지 않았습니다. 검사를 통과한 인코딩 traversal이 GitHub API URL에 삽입되면 자격 증명 포함 읽기가 다른 API path로 향할 수 있습니다. VulKill은 실제 `GithubUrlReader`, `getGithubFileFetchUrl`, `parseGitUrlSafe` 경로에서 controlled canary endpoint 도달을 관찰했습니다.

영향 범위: `@backstage/integration >= 0.1.3, < 2.3.0`. 수정 버전: `2.3.0`.

## 2. 현실적 시나리오

- Template 사용자가 공유 GitHub App token이 읽을 수 있는 private source를 가져옵니다.
- Private workflow log의 deployment secret이 노출됩니다.
- 조직 수준 integration token이 공격자 자신의 GitHub 권한보다 넓은 범위를 만듭니다.

## 3. 런타임 관찰

```text
control: 정상 파일 URL -> 의도한 repository contents endpoint
RED:     인코딩 owner -> integration credential이 포함된 다른 GitHub API path
GREEN:   반복 디코딩 및 delimiter/traversal 검사로 거부
BENIGN:  정상 URL은 계속 동작
```

Harness SHA-256: `dcf9f78d66b543e438dfa311509a481d78955070dd882a15bb6445552d0fd2c3`

## 4. 방어

`@backstage/integration 2.3.0` 이상으로 업데이트하고 공유 integration token 권한을 최소화하십시오.
