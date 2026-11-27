# Spotify Backstage GitHub URL Reader Encoded Traversal and Credentialed API Redirection (`GHSA-fp7g-2pw8-726f`)

**English** | [Türkçe](tr/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [日本語](ja/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [简体中文](zh-CN/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [한국어](ko/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md)

**VulKill finding:** Found during the Spotify/Backstage audit and observed in local runtime tests on 2026-11-01, before the upstream publication on 2026-11-27.
**VulKill audit identifier:** `NEW-BS-03`.
**Upstream:** [`GHSA-fp7g-2pw8-726f`](https://github.com/backstage/backstage/security/advisories/GHSA-fp7g-2pw8-726f) · CVSS 3.1 `7.7 High` · `CWE-22` / `CWE-200`.

## 1. How the Code Defect Becomes a Vulnerability

The GitHub URL reader validated the decoded file path but did not apply equivalent repeated-decoding and delimiter checks to every repository component. An encoded traversal in the owner portion could survive validation; after interpolation into a GitHub API URL, normal URL processing redirected the authenticated read to a different API path. VulKill exercised the real `GithubUrlReader`, `getGithubFileFetchUrl`, and `parseGitUrlSafe` path and observed a credentialed request reach the controlled canary endpoint.

Affected: `@backstage/integration >= 0.1.3, < 2.3.0`. Fixed in `2.3.0` (Backstage `1.57.4`). The impact depends on the privileges of the configured GitHub integration credential.

## 2. Real-World Scenarios

- **Scenario A — Private repository disclosure:** A template user reads private source reachable by an organization-wide GitHub App token.
- **Scenario B — Workflow-log exposure:** A redirected API read obtains a private job log that contains deployment or service credentials.
- **Scenario C — Enterprise-wide blast radius:** A shared integration credential makes the readable scope larger than the attacker's own GitHub permissions.

## 3. Runtime Observation

```text
control: canonical file URL -> intended repository contents endpoint
RED:     encoded owner component -> attacker-selected GitHub API path with integration credential
GREEN:   repeated decoding and delimiter/traversal validation rejects the URL
BENIGN:  ordinary owner/repository/ref/file URLs continue to work
```

- Upstream source commit tested: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `dcf9f78d66b543e438dfa311509a481d78955070dd882a15bb6445552d0fd2c3`
- Chain proof SHA-256: `0e2de393ee65136719495c2824cfe5133512ce570636174ee942fc208bbdc47a`

## 4. Defensive Guidance

Upgrade `@backstage/integration` to `2.3.0` or later. As a temporary mitigation, minimize shared integration-token privileges and require user SCM credentials where supported.
