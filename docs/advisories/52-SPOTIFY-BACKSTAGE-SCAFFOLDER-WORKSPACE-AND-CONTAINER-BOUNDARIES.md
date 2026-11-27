# Spotify Backstage Scaffolder Workspace and Container-Image Boundaries (`NEW-BS-05`)

**English** | [Türkçe](tr/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [日本語](ja/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [简体中文](zh-CN/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [한국어](ko/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md)

**VulKill finding:** Found during the Spotify/Backstage audit and observed against the real upstream Scaffolder modules in isolated local runtime tests on 2026-11-01. This record uses the audit identifier `NEW-BS-05` and does not assign an upstream advisory identifier.

## 1. How the Code Defects Become Security Boundaries

The `fetch:rails` action used a lexical `startsWith` comparison for its output path. A sibling-prefix path or an in-workspace symlink could therefore resolve outside the task workspace. The Cookiecutter action separately forwarded the caller-selected `imageName` to the configured container runner without an action-level image allowlist. VulKill exercised the real action handlers with temporary workspaces and a local, network-disabled test container.

The observed scope requires permission to execute an affected Scaffolder action and control the relevant action input. The test did not access a live Spotify service or an external container registry.

## 2. Real-World Scenarios

- **Scenario A — Sibling workspace write:** A generated project writes files into a neighboring task directory whose name shares the workspace prefix.
- **Scenario B — Symlink escape:** A path that is lexically inside the workspace resolves through a symlink and writes outside the task directory.
- **Scenario C — Unapproved generator image:** A template selects a container image that produces unexpected workflow or dependency files in the generated project.

## 3. Runtime Observation

```text
control: normal child path remains inside the temporary workspace
RED:     sibling-prefix and symlink paths write synthetic markers outside the workspace
RED:     caller-selected test image reaches the local container runner
GREEN:   safe-child resolution and image allowlisting reject the same inputs
BENIGN:  approved image and ordinary child path continue to work
```

- Backstage source commit tested: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `fa4dab592b61c8f8fcde81743694189b5d988cd6a9c7cbf29accc50c4bc097ae`
- Runtime chain SHA-256: `0e2de393ee65136719495c2824cfe5133512ce570636174ee942fc208bbdc47a`

## 4. Defensive Guidance

Resolve and confine every destination with the platform safe-child helper and real-path checks, reject symlink escapes, allowlist container images at the action boundary, and restrict who may create or execute sensitive templates.
