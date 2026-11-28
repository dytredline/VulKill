# Spotify Backstage Catalog `validate-entity` Local File Disclosure (`GHSA-h8rc-g67j-87x3`)

**English** | [Türkçe](tr/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [日本語](ja/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [简体中文](zh-CN/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [한국어](ko/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md)

**VulKill finding:** Found during the Spotify/Backstage audit and observed in local runtime tests on 2026-11-01, before the upstream publication on 2026-11-18.
**VulKill audit identifier:** `NEW-BS-04`.
**Upstream:** [`GHSA-h8rc-g67j-87x3`](https://github.com/backstage/backstage/security/advisories/GHSA-h8rc-g67j-87x3) · CVSS 3.1 `6.5 Medium` · `CWE-200` / `CWE-863`.

## 1. How the Code Defect Becomes a Vulnerability

`POST /validate-entity` assigned a server-controlled location and then merged caller-supplied annotations after it. A caller could therefore overwrite the location context with a `file:` location. The processor treated the forged type as inherited, reached the backend file reader, and returned source lines through YAML validation diagnostics. VulKill found the behavior through the real `createRouter`, `AuthorizedValidationService`, processing orchestrator, and `FileReaderProcessor` path with low-privilege and administrator controls.

Affected: `@backstage/plugin-catalog-backend < 3.9.3` and `>= 4.0.0, < 4.0.3`. Fixed in `3.9.3`, `4.0.3`, `4.0.4`, and `4.0.5`.

## 2. Real-World Scenarios

- **Scenario A — Backend configuration disclosure:** An ordinary signed-in user reads a line from a YAML-like configuration file available to the catalog process.
- **Scenario B — Service credential exposure:** A malformed target file causes a parser diagnostic to echo a secret-bearing source line in the HTTP response.
- **Scenario C — Repository-boundary bypass:** Forged provenance annotations make placeholder reads appear to originate from a different repository context.

## 3. Runtime Observation

```text
control: unauthenticated or unprivileged guest -> 403
RED:     catalog.entity.validate user + forged location annotation -> local marker disclosed
GREEN:   server-assigned annotations take precedence; local read blocked
BENIGN:  valid URL-backed entity validation remains successful
```

- Upstream source commit tested: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `0bd744cadf97f031fd218d97eb187fa0394c7ec0b23d8c702fe7eec52f0403ce`
- Chain proof SHA-256: `a8f607e3adb9747d12b1995a3cbedd68ab2477378d52b92a5244fb2ada41cc82`

## 4. Defensive Guidance

Upgrade to a patched `@backstage/plugin-catalog-backend` release. Until then, restrict `catalog.entity.validate` to trusted users and remove the file reader if local catalog locations are not required.
