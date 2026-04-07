---
name: t4h-framework-amber
description: Documents @t4h.framework/amber (Amber client, Ticket operations, HTTP activities, Basic auth, runtime-claims peers). Use when implementing ticket or protocol flows—integration, creation, updates, transfers, tabulation, comments, locks—or when the user asks about ticket/protocol integration, creating tickets or protocols, or similar; also when they mention @t4h.framework/amber or Amber channel APIs.
---

# @t4h.framework/amber

**Amber** is the **`@t4h.framework/amber`** client for the **Amber HTTP API** under a **tenant + channel** base path. Requests use **HTTP Basic** auth: **`username`** = **`clientId`**, **`password`** = **`clientSecret`**.

**Source of truth**: the **`@t4h.framework/amber`** package itself (activity modules, types, tests). If behavior, paths, or payloads are unclear, read those files in the installed package—do not invent URL shapes or field meanings.

## Dependencies

Install peers together with this package:

| Peer | Role |
|------|------|
| **`@t4h.framework/core`** | **`History`**, **`Claim`**, **`AsyncActivity`**, workflow identifiers. |
| **`@t4h.framework/http`** | **`AbstractHttpRequestActivity`**, **`Response`**, **`HttpClientRequestClaim`**. |
| **`@t4h.framework/runtime-claims`** | **`GetProjectUniqueIdentifierClaim`**, **`GetProjectVersionUniqueIdentifierClaim`**, **`GetProjectVersionWorkflowUniqueIdentifierClaim`**, **`GetWorkflowUniqueIdentifierClaim`**, **`GetActivityUniqueIdentifierClaim`** (used by activities that need project/workflow context). |
| **`typebox`** | Schemas on async activities (e.g. transfer). |

**Node**: `>=22` (see package **`engines`**).

## URL layout

**`AmberConfig.url`** is the **API origin** (e.g. `https://api.example/`). The client resolves the **channel base** as:

`v1/tenants/{tenant}/channels/{channel}`

(relative to **`config.url`**), then stores that string on **`AmberConnection`**. All **`Ticket`** and **`Amber`** calls use this resolved **`url`** plus REST segments such as **`/services`**, **`/services/:id`**, etc., as implemented in each activity.

## `AmberConfig`

| Field | Purpose |
|-------|--------|
| **`url`** | Amber API base URL (origin). |
| **`tenant`** | Tenant id segment in the path (`safeTenantId`). |
| **`channel`** | Channel id segment; also passed into **create** payloads. |
| **`clientId`**, **`clientSecret`** | Basic auth credentials for every call. |

## Class: `Amber`

Constructed with **`new Amber(config)`**. Exposes:

| Method | Purpose |
|--------|--------|
| **`createTicket(input)`** | **`POST`** `{url}/services` with **form** body built from **`input`** (see **`AmberCreateTicketActivity`**). Fills **`type`**, **`channel`**, **`metadata`** (project / version / workflow / run from **runtime claims**). Returns a **`Ticket`**. |
| **`getTicket({ id })`** | **`GET`** `{url}/services/{id}`. Returns a **`Ticket`**. |

**`createTicket`** input type is **`Omit<AmberCreateTicketActivityInput, 'auth' \| 'url' \| 'channel'>`** — caller does not pass **`auth`**, **`url`**, or **`channel`**; the **`Amber`** class injects them.

## Class: `Ticket`

Returned by **`createTicket`** / **`getTicket`**. Holds **`id`** and connection info; exposes operations that bind the corresponding **`History`** activities with **`auth`**, **`url`**, and **`id`**:

| Method | HTTP (see activity) |
|--------|---------------------|
| **`tabulate`** | **`POST`** `.../services/{id}/tabulations` (JSON body). |
| **`transfer`** | **`AsyncActivity`**: metadata PATCH + **`POST`** `.../services/{id}/transfers` (see **`AmberTransferTicketActivity`**). |
| **`comment`** | **`POST`** `.../services/{id}/comments`. |
| **`interact`** | **`POST`** `.../services/{id}/interactions` (**form** body). |
| **`update`** | **`PATCH`** `.../services/{id}`. |
| **`finish`** | **`update`** with **`status: 'finished'`**. |
| **`lock`** | **`POST`** `.../services/{id}/locks` (JSON includes **`ref`** from **`GetWorkflowUniqueIdentifierClaim`**). |
| **`releaseLock`** | **`POST`** `.../services/{id}/locks/current/release`. |

**Note**: The package **`index`** re-exports **`Amber`** and activities; **`Ticket`** may not appear in the barrel **`export *`**—if you need a named **`Ticket`** import, confirm **`dist/index.d.ts`** for your version or import from the path your bundler resolves.

## `AbstractAmberHttpRequestActivity` and `AmberHttpRequestActivityInput`

Most sync flows extend **`AbstractAmberHttpRequestActivity`**, which extends **`AbstractHttpRequestActivity`** from **`@t4h.framework/http`**.

**`AmberHttpRequestActivityInput`** = **`HttpRequestActivityInput`** & **`{ auth: { username: string; password: string } }`** (Basic auth is **required** on Amber requests).

**`toHttpInput`** copies **`url`**, **`headers`**, **`auth`**, optional **`method`**, **`body`**, **`retry`** into the generic HTTP activity input.

## Exported activities (public API surface)

The package entry exports these **activity classes** and their **`*Input`** types (for advanced / direct **`History.bind`** usage):

| Activity | Kind | Notes |
|----------|------|--------|
| **`AmberCreateTicketActivity`** | Sync HTTP | Uses **runtime claims** to fill **`metadata`**; **`POST`** form to **`/services`**. |
| **`AmberGetTicketActivity`** | Sync HTTP | **`GET`** **`/services/:id`**. |
| **`AmberUpdateTicketActivity`** | Sync HTTP | **`PATCH`** JSON **`/services/:id`**. |
| **`AmberCommentTicketActivity`** | Sync HTTP | **`POST`** JSON **`/services/:id/comments`**. |
| **`AmberInteractTicketActivity`** | Sync HTTP | **`POST`** form **`/services/:id/interactions`**. |
| **`AmberTabulateTicketActivity`** | Sync HTTP | **`POST`** JSON **`/services/:id/tabulations`**. |
| **`AmberLockTicketActivity`** | Sync HTTP | **`POST`** JSON **`/services/:id/locks`**; **`ref`** from claim. |
| **`AmberReleaseTicketLockActivity`** | Sync HTTP | **`POST`** JSON **`/services/:id/locks/current/release`**. |
| **`AmberTransferTicketActivity`** | **`AsyncActivity`** | **`bootstrap`**: PATCH metadata, then **`POST`** **`/services/:id/transfers`** via **`HttpClientRequestClaim`**; may **`pending`** with **TypeBox** schema until **`advance`**. |

**Status checks**: Several activities expect HTTP **`200`** and throw **`Error`** with **`cause.httpStatusCode`** when the status differs.

## What to clarify with the user (do not guess)

- **Exact Amber base URL**, **tenant**, **channel**, and **credentials** for the environment.
- **Field semantics** for **`ServiceTicketInput`**, tabulation, transfer, and interaction payloads (see the **`Service*`** type modules in this package).
- Whether **`Ticket`** should be a **named export** from the package (if consumers need `import { Ticket }`).
- Any **activity not listed in `src/index.ts`** is **not** part of the published public API until added there.

## Related plugin skills

- **`framework-http`**: generic **`HttpRequestActivityInput`**, **`Response`**, **`HttpClientRequestClaim`**.
- **`framework-core`**: **`History.bind`**, **`Claim`**, **`AsyncActivity`**, **`startWorkflow`** / **`StartWorkflowActivity`**.
- **`framework-skills-index`**: map of all **`framework-*`** skills.
