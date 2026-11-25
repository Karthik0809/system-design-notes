# 78 · Design an Authentication Service (SSO across web and mobile)

| Category | Difficulty | Key ideas |
|---|---|---|
| Security infrastructure | Medium | OIDC authorization code + PKCE, short-lived signed access tokens, rotating refresh tokens, JWKS for local verification, session revocation within minutes, MFA |

---

## Summary
Users sign in once at a central **identity provider** (IdP). Apps use **OIDC authorization code flow with PKCE** to get a short-lived **access token** (JWT, ~10 min) and a **refresh token**. Gateways and services verify access tokens **locally** with the IdP's public keys (JWKS), so no lookup per request. Revocation works by keeping access tokens short and checking refresh tokens (and a small revocation list) on every refresh, so a logout or compromised session is cut off within minutes.

---

## Requirements
| Functional | Non-functional |
|---|---|
| Sign up, sign in (password, social login, passkeys), MFA | 100M users, sign-in peaks of 10K/s |
| Single sign-on across many apps | Token check at the gateway with no network call |
| Sign out everywhere, revoke sessions | Revocation effective within ~5 min |
| Password reset, account recovery | Very high availability (if auth is down, everything is down) |

---

## Architecture

```mermaid
flowchart LR
    APP["Web / mobile app"] -->|"redirect + PKCE"| IDP["Identity provider (login UI, OIDC endpoints)"]
    IDP --> UDB[("Users: credentials hashed with Argon2, MFA secrets")]
    IDP --> SDB[("Sessions + refresh tokens")]
    IDP --> KMS["Signing keys (KMS / HSM), rotated"]
    IDP --> JWKS["JWKS endpoint (public keys, CDN cached)"]
    APP -->|"access token"| GW["API gateway"]
    GW -->|"verify signature locally"| JWKS
    GW --> REV[("Revocation list (small, pushed to gateways)")]
    GW --> SVC["Services"]
```

---

## Sign-in flow (authorization code + PKCE)
```mermaid
sequenceDiagram
    participant A as App
    participant I as Identity provider
    participant G as Gateway
    A->>A: create code_verifier, send its hash (challenge)
    A->>I: /authorize (client_id, challenge, scopes)
    I->>I: login + MFA, create session
    I-->>A: authorization code (redirect)
    A->>I: /token (code + code_verifier)
    I-->>A: access token (10 min) + refresh token + id token
    A->>G: API call with Bearer access token
    G->>G: verify signature, expiry, audience with cached JWKS
    Note over A,I: before expiry: /token with refresh token (rotated each time)
```

---

## Deep dives
- **Why short access tokens:** they can't be revoked individually without a lookup, so their lifetime is the revocation delay. 5-15 minutes is the usual balance.
- **Refresh token rotation:** each refresh returns a new refresh token and invalidates the old one; reuse of an old one signals theft, so revoke the whole session family.
- **Revoke now:** for urgent cases (account takeover), push session IDs to a small revocation list on gateways until their tokens expire.
- **Key rotation:** publish new keys in JWKS before using them; keep old keys until all tokens signed with them expire.
- **Password storage:** Argon2/bcrypt with per-user salt; breached-password checks; rate limiting and lockouts against credential stuffing.
- **MFA and passkeys:** TOTP, push approval, WebAuthn passkeys (phishing resistant).
- **Availability:** the IdP is replicated across regions; token verification works even if the IdP is briefly down because it's local.

---

## Interview Q&A

**Q: Why PKCE?**
A: Mobile and single-page apps can't keep a client secret; PKCE proves the app redeeming the code is the one that started the login, which blocks intercepted codes.

**Q: How does "sign out of all devices" work?**
A: Delete all refresh tokens for the user and add active sessions to the revocation list; access tokens die within minutes.

**Related:** [Security](../01-fundamentals/15-security-authn-authz-encryption.md) · [API gateway](63-api-gateway.md) · [Permissions service](80-permissions-service.md)
