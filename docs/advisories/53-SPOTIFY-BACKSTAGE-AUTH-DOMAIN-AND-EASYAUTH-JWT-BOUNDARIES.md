# Spotify Backstage Sign-In Domain and Azure EasyAuth JWT Boundaries (`NEW-BS-06`)

**English** | [Türkçe](tr/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [日本語](ja/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [简体中文](zh-CN/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [한국어](ko/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md)

**VulKill finding:** Found during the Spotify/Backstage audit and observed against the real upstream authentication and gateway modules in isolated local runtime tests on 2026-11-01. This record uses the audit identifier `NEW-BS-06` and does not assign an upstream advisory identifier.

## 1. How the Code Defects Become Authentication Boundaries

The `emailMatchingUserEntityProfileEmail` resolver accepted `allowedDomains` in its option schema but did not enforce the list during sign-in. In the tested Azure EasyAuth path, the authenticator decoded the forwarded ID-token header without performing cryptographic signature, issuer, and audience checks at that boundary. VulKill exercised the real resolver, gateway routing, token factory, key store, and identity-client path with synthetic identities and locally generated keys.

The domain behavior requires the affected resolver and a matching catalog user. The EasyAuth behavior depends on a deployment path that permits a client-supplied identity header to reach the provider instead of replacing it at a trusted proxy.

## 2. Real-World Scenarios

- **Scenario A — Domain policy gap:** An external-domain catalog identity signs in even though the resolver configuration lists only internal domains.
- **Scenario B — Trusted-header boundary loss:** A deployment forwards a client-supplied identity-token header to the EasyAuth provider.
- **Scenario C — Existing identity impersonation:** Claims from the accepted header resolve to a privileged catalog identity and a signed Backstage session is issued for that identity.

## 3. Runtime Observation

```text
control: sibling resolver rejects an external-domain synthetic identity
RED:     affected resolver accepts the same identity and issues an ES256 Backstage token
RED:     local gateway path forwards a synthetic unsigned ID token to the EasyAuth provider
GREEN:   domain enforcement and JWT signature/issuer/audience checks reject both cases
BENIGN:  internal-domain identity and trusted proxy token continue to work
```

- Backstage source commit tested: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Domain-resolver harness SHA-256: `753692625e86b7007c02b5cf59e6d73c5ddc244cc6d1aeabe824480bafb2f2c8`
- EasyAuth/gateway harness SHA-256: `d3590c092c37f719926b4aec5de779ae6a612c8a45f9c057eb78bddc1aececca`
- Domain runtime chain SHA-256: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`
- EasyAuth runtime chain SHA-256: `6778c3e716d8a3e1646dd98bbf94e5d55f16be37815058792f8873d8ea55228f`

## 4. Defensive Guidance

Enforce `allowedDomains` at runtime, verify EasyAuth JWT signatures with pinned issuer and audience, strip identity headers at the public gateway, and accept those headers only from a documented trusted reverse proxy.
