---
name: t4h-framework-jwks
description: How to use @t4h.framework/jwks JWKSAuthorization, context fields, and the authorize module JWT vs raw signature paths. Use when configuring JWKS app authorization or imports from @t4h.framework/jwks.
---

# @t4h.framework/jwks

**`JWKSAuthorization`** extends **`Authorization`** from **`@t4h.framework/core`**. **`toManifest()`** sets **`pathToModule`** to **`@t4h.framework/jwks/authorize`** and copies **`JWKSAuthorizationContext`** into the manifest.

## Requirements

**Peer dependency**: **`@t4h.framework/core`**.

## Imports

- Package entry: **`JWKSAuthorization`**, **`JWKSAuthorizationContext`**, manifest types.
- **`@t4h.framework/jwks/authorize`**: default function **`jwks(options)`** from **`jwks-authorize.ts`**.

## `JWKSAuthorizationContext`

| Field | Purpose |
|-------|--------|
| **`url`** | JWKS document URI (e.g. **`/.well-known/jwks.json`**) → **`jwks-rsa`** **`JwksClient`** **`jwksUri`**. |
| **`signature`** | JWT string **or** raw signature bytes (see flow below). |
| **`rawPayload`** | String verified in the non-JWT path (`verifier.update(rawPayload)`). |
| **`issuer`**, **`audience`** | Optional; used in the JWT validation path. |
| **`kid`** | Required when **`signature`** is **not** a JWT (raw path). |
| **`algorithm`** | Maps to **`JWKSAlgorithm`** for **`createVerify`**; required in raw path. |
| **`signatureFormat`** | Encoding for **`verify()`** (e.g. **`base64`**); required in raw path. |

## How **`jwks-authorize`** decides

Returns **`Promise<boolean>`**.

1. **JWT**: If **`jwt.decode(signature, { complete: true })`** succeeds, **`validateJwt`** runs with the JWKS client and optional **`issuer`** / **`audience`**.
2. **Raw signature**: Otherwise **`kid`**, **`algorithm`**, and **`signatureFormat`** are required (else throws). Loads key by **`kid`**, **`createVerify(JWKSAlgorithm[algorithm])`**, **`update(rawPayload)`**, **`verify(signature, signatureFormat)`**.

Empty **`signature`** → **`false`**.

## Example: `force-manual-process`

**`micro-services/force-manual-process/src/main.tsx`** builds **`new JWKSAuthorization({ url, signature, rawPayload, kid, signatureFormat, algorithm })`** with template placeholders such as **`<<body.data.tenant.id>>`** and **`<<headers.x-kid>>`** so values are filled per request.
