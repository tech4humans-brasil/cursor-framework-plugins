---
name: t4h-framework-http
description: How to use @t4h.framework/http HttpClient, http singleton, HttpRequestActivityInput, HttpAuth, OAuth2, HttpClientRequestClaim. Use when calling HTTP from workflows or extending OAuth2.
---

# @t4h.framework/http

**`HttpClient`** methods go through **`HttpRequestActivity`** (**`History.bind`**): each call performs an HTTP request via **`HttpClientRequestClaim`** and writes the response body through **`FileSystemClaim`** for the returned **`Body`**.

## Dependencies

Install **`@t4h.framework/cache`**, **`@t4h.framework/core`**, and **`@t4h.framework/fs`** together with this package.

## Default client: `http`

**`http`** is a singleton **`Http`** (extends **`HttpClient`**) with empty **`config`**. **`extend(partialConfig)`** returns a new **`HttpClient`** merged with **`HttpClientConfig`**.

## `HttpClientConfig`

| Field | Purpose |
|-------|--------|
| **`baseUrl`** | Optional base for **`new URL(url, baseUrl)`**. |
| **`auth`** | **`HttpAuth`** or **`{ username, password }`** (Basic **`Authorization`** header). |

## `HttpClient` methods

**`request(url, config?)`** merges **`url`**, client **`auth`** / **`baseUrl`**, and **`HttpRequestActivityInput`** fields.

**`get`**, **`post`**, **`put`**, **`patch`**, **`delete`** set **`method`** (and **`body`** where applicable). **`get`** has no **`body`**.

## `HttpRequestActivityInput`

| Field | Notes |
|-------|--------|
| **`url`** | Required (path or absolute URL). |
| **`baseUrl`** | Overrides client default when set. |
| **`method`** | Defaults to **`GET`** in the activity if omitted. |
| **`body`** | **`{ json: Serializable }`**, **`{ text: string }`**, or **`{ form: nested object }`** (supports **`Binary`**, nesting). |
| **`headers`** | **`Record<string, string \| string[] \| undefined>`**. |
| **`auth`** | Same as **`HttpClientConfig.auth`** → **`Authorization`** header. |
| **`redirect`** | **`'follow' \| 'manual' \| number`**; default **`'follow'`**. |
| **`retry`** | **`true`** (default) or **`HttpRequestActivityInputRetryOptions`** (**`limit`**, **`methods`**, **`status`**, **`backoffLimit`**, **`jitter`**, …). |

## `HttpAuth`

Abstract: implement **`getAuthorizationHeader()`** (full value, e.g. **`Bearer …`**). The base class wires a **`Claim`** that includes **`CacheClaim`** (used by **`OAuth2`** for token storage).

## OAuth2

**`OAuth2<TConfig extends OAuth2Config>`** extends **`HttpAuth`**, uses **`HttpClientRequestClaim`** + **`CacheClaim`**. **`OAuth2Config`**: **`tokenUrl`**, **`clientId`**. Default **`toToken()`**: **`POST`** **`application/x-www-form-urlencoded`** from **`toTokenBody()`**, JSON response as **`OAuth2Token`**, cache key **`oauth2:token:${clientId}`**, TTL from **`expires_in`**, refresh via **`refresh_token`** when present and expired.

| Class | Behavior |
|-------|----------|
| **`OAuth2ClientCredentials`** | **`grant_type: client_credentials`** + **`client_secret`**. |
| **`OAuth2GrantResourceOwnerPassword`** | **`grant_type: password`** + **`clientSecret`**, **`username`**, **`password`** (on top of **`OAuth2Config`**). |

**`OAuth2Token`**: **`access_token`**, optional **`refresh_token`**, **`expires_in`**, **`token_type`**.

## `HttpClientRequestClaim`

**`request(input: HttpClientRequestClaimInput)`**: **`url`** is a **`URL`**; **`body`**: **`Buffer`**, **`FormData`**, **`Readable`**, **`URLSearchParams`**, or **`undefined`**; **`headers`**, **`redirect`**, **`retry`**. Response: **`status`**, **`headers`**, **`body`** as **`Readable`**.

Override **`toToken()`** in **`OAuth2`** subclasses when the token endpoint expects something other than form body (then call **`this.claims.http.request(...)`**).

## Example: `force-manual-process`

**`micro-services/force-manual-process/src/workflows/service-tabulate.ts`**: **`http.patch`** with auth. **`LapisLazuliOAuth2ClientCredentials`** (extends **`OAuth2ClientCredentials`**) overrides **`toToken()`** to **`POST`** JSON to the token URL and parse **`OAuth2Token`**.
