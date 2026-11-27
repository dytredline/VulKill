# Spotify Backstage Kubernetes Proxy Session-Credential Forwarding (`GHSA-27gw-fw83-xfpp`)

**English** | [Türkçe](tr/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [日本語](ja/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [简体中文](zh-CN/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [한국어](ko/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md)

**VulKill finding:** Found during the Spotify/Backstage audit and observed in local runtime tests on 2026-11-01, before the upstream publication on 2026-11-18.
**Upstream:** [`GHSA-27gw-fw83-xfpp`](https://github.com/backstage/backstage/security/advisories/GHSA-27gw-fw83-xfpp) · CVSS 3.1 `6.4 Medium` · `CWE-200` / `CWE-441`.

## 1. How the Code Defect Becomes a Vulnerability

The Kubernetes proxy forwarded the caller's incoming `Authorization` and `Cookie` headers whenever the selected cluster authentication strategy did not replace them. If a catalog contributor could influence the cluster API URL, opening the Kubernetes view for that entity caused the victim's Backstage session credential to be sent to the contributor-controlled endpoint. VulKill found the behavior through the real `CatalogClusterLocator` to `KubernetesRouter` path, including signed Backstage identity handling and the built-in authentication strategies.

Affected: `@backstage/plugin-kubernetes-backend >= 0.9.0, < 0.21.13`. Fixed in `0.21.13`, with supported backports `0.21.14` and `0.21.15`.

## 2. Real-World Scenarios

- **Scenario A — Contributor-controlled cluster:** A low-privileged catalog contributor adds an entity that points at a controlled Kubernetes endpoint; an administrator viewing its Kubernetes tab sends a valid Backstage token there.
- **Scenario B — Cross-plugin impersonation:** The captured short-lived identity token is replayed against catalog, scaffolder, or permission APIs with the victim's identity.
- **Scenario C — Cookie-auth deployment:** A deployment using session cookies forwards the victim's cookie to the influenced upstream cluster.

## 3. Runtime Observation

```text
control: operator-controlled cluster receives only the configured cluster credential
RED:     contributor-influenced cluster receives the caller Authorization/Cookie header
GREEN:   proxy strips caller session credentials before the upstream request
BENIGN:  explicit Backstage-Kubernetes-Authorization continues to work
```

- Upstream source commit tested: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Runtime chain SHA-256: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`
- Shared runtime log SHA-256: `86cec273f9c154713aa8b9afb4283ad4b65422466adc12fff4710484512af0de`

## 4. Defensive Guidance

Upgrade to a patched version, restrict catalog cluster creation, deny `kubernetes.proxy` to untrusted users, and prefer operator-controlled cluster locators.
