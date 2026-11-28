# Spotify Backstage Scaffolder 작업공간 및 컨테이너 이미지 경계 (`NEW-BS-05`)

[English](../52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [Türkçe](../tr/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [日本語](../ja/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [简体中文](../zh-CN/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | **한국어**

**VulKill 발견:** Spotify/Backstage 감사에서 찾았으며 2026-11-01 실제 upstream Scaffolder 모듈을 사용한 격리 로컬 런타임 테스트에서 관찰했습니다. 이 기록은 감사 ID `NEW-BS-05`를 사용하며 upstream advisory ID를 지정하지 않습니다.

## 1. 코드 결함이 보안 경계가 되는 방식

`fetch:rails` action은 출력 경로에 문자열 `startsWith` 비교를 사용했습니다. 따라서 동일 접두사를 가진 인접 경로나 작업공간 내부 symlink가 task workspace 밖으로 해석될 수 있었습니다. Cookiecutter action은 별도로 호출자가 선택한 `imageName`을 action-level image allowlist 없이 container runner로 전달했습니다. VulKill은 임시 작업공간과 네트워크가 비활성화된 로컬 test container로 실제 action handler를 실행했습니다.

영향 범위에는 해당 Scaffolder action 실행 권한과 관련 입력 제어가 필요합니다. 테스트는 Spotify live service나 외부 container registry에 접근하지 않았습니다.

## 2. 현실적 시나리오

- **A — 인접 작업공간 쓰기:** 생성 프로젝트가 같은 접두사를 가진 이웃 task directory에 파일을 씁니다.
- **B — Symlink escape:** 문자열상 내부 경로가 symlink를 통해 task directory 밖에 기록됩니다.
- **C — 미승인 generator image:** template이 예상치 못한 workflow/dependency 파일을 만드는 image를 선택합니다.

## 3. 런타임 관찰

```text
control: 일반 하위 경로는 임시 작업공간 내부에 유지
RED:     인접 접두사와 symlink 경로가 외부에 합성 marker 기록
RED:     호출자 지정 test image가 로컬 container runner에 도달
GREEN:   safe-child 해석과 image allowlist가 동일 입력 거부
BENIGN:  승인 image와 일반 하위 경로는 계속 동작
```

- Source commit: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `fa4dab592b61c8f8fcde81743694189b5d988cd6a9c7cbf29accc50c4bc097ae`
- Runtime chain SHA-256: `0e2de393ee65136719495c2824cfe5133512ce570636174ee942fc208bbdc47a`

## 4. 방어

safe-child helper와 realpath 검사로 모든 대상을 제한하고 symlink escape를 거부하며 action 경계에서 container image를 allowlist하고 민감한 template 생성·실행 권한을 제한하십시오.
