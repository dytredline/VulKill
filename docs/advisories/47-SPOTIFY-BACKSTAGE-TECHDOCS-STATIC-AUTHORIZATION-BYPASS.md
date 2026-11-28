# Spotify Backstage TechDocs Static Content Authorization Bypass (`GHSA-rg9r-hr7g-5gc2` / `CVE-2026-106489`)

**English** | [Türkçe](tr/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [日本語](ja/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [简体中文](zh-CN/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [한국어](ko/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md)

**VulKill finding:** Found during the Spotify/Backstage audit and observed against the real upstream modules in local runtime tests on 2026-11-01. The upstream advisory had been published on 2026-08-28.
**VulKill audit identifier:** `NEW-BS-02`.
**Upstream:** [`GHSA-rg9r-hr7g-5gc2`](https://github.com/backstage/backstage/security/advisories/GHSA-rg9r-hr7g-5gc2) · `CVE-2026-106489` · CVSS 3.1 `6.5 Medium` · `CWE-862`.

## 1. How the Code Defect Becomes a Vulnerability

The TechDocs static-content route normalized a crafted double-slash path differently during authorization and storage lookup. In deployments using the permission framework, an authenticated user allowed to read one documentation entity could make the external storage publisher resolve content belonging to another entity. VulKill exercised the real `createRouter`, `LocalPublish`, and `AzureBlobStoragePublish` modules: the ordinary path remained denied, while the crafted path returned the protected marker before the defensive change.

Affected: `@backstage/plugin-techdocs-backend < 2.2.4`. Fixed in `2.2.4`. Local-storage deployments and deployments without the permission framework are outside the upstream advisory's affected configuration.

## 2. Real-World Scenarios

- **Scenario A — Cross-team documentation disclosure:** A developer who can read public service docs retrieves another team's private runbook from external TechDocs storage.
- **Scenario B — Operational secret exposure:** Internal incident instructions, private hostnames, or example credentials embedded in restricted docs become visible.
- **Scenario C — Tenant boundary loss:** A shared Backstage instance enforces entity permissions in the UI, but the crafted static-content path crosses that boundary at the backend.

## 3. Runtime Observation

VulKill used synthetic markers only; no live Spotify service or third-party deployment was accessed.

```text
control: low-privileged user + canonical path -> 404
RED:     same user + crafted static path -> 200, protected marker returned
GREEN:   patched route -> 404
BENIGN:  authorized user + canonical path -> 200
```

- Upstream source commit tested: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `30bdbb5fa044c7568985a06b2e27c498296d1883344b728b99919f6736b8a665`
- Shared runtime log SHA-256: `86cec273f9c154713aa8b9afb4283ad4b65422466adc12fff4710484512af0de`

## 4. Defensive Guidance

Upgrade to `@backstage/plugin-techdocs-backend 2.2.4` or later. Until upgraded, do not rely on the affected external-storage static route to enforce per-entity confidentiality.
