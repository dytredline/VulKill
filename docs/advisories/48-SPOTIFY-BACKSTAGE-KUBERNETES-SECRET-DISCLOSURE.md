# Spotify Backstage Kubernetes Resource Secret Disclosure (`GHSA-p795-mqf2-36mf` / `CVE-2026-106561`)

**English** | [Türkçe](tr/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [日本語](ja/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [简体中文](zh-CN/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [한국어](ko/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md)

**VulKill finding:** Found during the Spotify/Backstage audit and observed against the real upstream Kubernetes router and fetcher in local runtime tests on 2026-11-01. The upstream advisory had been published on 2026-08-28.
**Upstream:** [`GHSA-p795-mqf2-36mf`](https://github.com/backstage/backstage/security/advisories/GHSA-p795-mqf2-36mf) · `CVE-2026-106561` · CVSS 3.1 `5.0 Medium` · `CWE-200`.

## 1. How the Code Defect Becomes a Vulnerability

Backstage masked the normal `Secret.data` field but did not consistently remove equivalent secret material embedded in returned Kubernetes resource annotations. An authenticated user with the standard Kubernetes resource-read permission could therefore receive plaintext values that the UI and backend were expected to hide. VulKill found the behavior through the real `KubernetesRouter`, `KubernetesProxy`, and `KubernetesClientBasedFetcher` path with a synthetic secret marker.

Affected: `@backstage/plugin-kubernetes-backend < 0.21.9`. Fixed in `0.21.9`. Exposure is limited to resource types and namespaces that the configured Backstage cluster credential can read.

## 2. Real-World Scenarios

- **Scenario A — Database credential exposure:** A developer allowed to inspect a service's Kubernetes resources sees a password retained in a deployment annotation.
- **Scenario B — CI/CD token disclosure:** A generated manifest stores a token in an annotation even though the primary secret data field is masked.
- **Scenario C — Shared-cluster lateral movement:** A user with ordinary Backstage access obtains credentials for another internal service reachable from the same cluster.

## 3. Runtime Observation

```text
control: direct secret proxy request -> 403
RED:     permitted service-resource query -> 200; Secret.data masked, annotation marker visible
GREEN:   patched fetcher masks the secret-derived annotation value
BENIGN:  non-sensitive resource metadata remains available
```

- Upstream source commit tested: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `d6acd23d33238a8dda450ca77e744f008094e415f80f4efdc39cb9a71071a9a8`
- Shared runtime log SHA-256: `86cec273f9c154713aa8b9afb4283ad4b65422466adc12fff4710484512af0de`

## 4. Defensive Guidance

Upgrade to `@backstage/plugin-kubernetes-backend 0.21.9` or later, restrict `kubernetes.resources.read`, and ensure the Backstage service account cannot read unnecessary sensitive resource types.
