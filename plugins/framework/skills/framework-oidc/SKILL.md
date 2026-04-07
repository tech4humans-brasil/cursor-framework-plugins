---
name: t4h-framework-oidc
description: How to use @t4h.framework/oidc OIDCAuthorization, context fields, and the authorize module discovery plus client-credentials and introspection. Use when configuring OIDC authorization or imports from @t4h.framework/oidc.
---

# @t4h.framework/oidc

**`OIDCAuthorization`** extends **`Authorization`**. **`toManifest()`** sets **`pathToModule`** to **`@t4h.framework/oidc/authorize`** and passes **`OIDCAuthorizationContext`**.

## Requirements

**Peer dependency**: **`@t4h.framework/core`**.

## Imports

- Entry: **`OIDCAuthorization`**, **`OIDCAuthorizationContext`**, manifest types.
- **`@t4h.framework/oidc/authorize`**: default **`oidc(context)`** in **`oidc-authorize.ts`**.

## `OIDCAuthorizationContext`

| Field | Purpose |
|-------|--------|
| **`discoveryUrl`** | OpenID discovery document URL (**GET**). JSON must include **`token_endpoint`** and **`introspection_endpoint`**. |
| **`clientId`**, **`clientSecret`** | **`grant_type: client_credentials`** at **`token_endpoint`**. |
| **`token`** | Caller credential; expect **`Bearer …`**. Only the part after **`Bearer `** is sent as the introspection **`token`**. |
| **`scope`** | Optional **`string[]`** → joined with spaces for the token **POST**. |
| **`resource`** | Optional; added to the client-credentials body when set. |

## Flow in **`oidc-authorize`**

1. **GET** **`discoveryUrl`** → read **`token_endpoint`**, **`introspection_endpoint`** (errors throw on non-OK).
2. **POST** **`token_endpoint`**, **`application/x-www-form-urlencoded`**: **`grant_type`**, **`client_id`**, **`client_secret`**, optional **`scope`**, **`resource`** → read **`access_token`** from JSON.
3. **POST** **`introspection_endpoint`**, **`application/json`**: body **`{ token }`** where **`token`** is **`getAccessToken(context.token)`** (substring after **`Bearer `**). Header **`Authorization: Bearer <access_token from step 2>`** (client-credentials token).
4. **`true`** iff introspection JSON has **`active: true`**; else **`false`**. Non-OK HTTP throws.

If **`token`** does not start with **`Bearer `**, the introspection body sends **`null`** for **`token`**.
