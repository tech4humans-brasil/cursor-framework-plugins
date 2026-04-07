---
name: framework-skills-index
description: Index of Cursor plugin skills under plugins/framework/skills for @t4h.framework packages, when to open each skill, recommended app layout (main entry plus workflows folder), and instructs to ask the user when requirements are ambiguous. Points to framework-building-workflows for end-to-end App and Workflow assembly. Use when onboarding to the framework skills, scaffolding apps, choosing which framework-* skill to read, or when the user mentions framework skills index or T4H package map.
---

# T4H Framework plugin skills — index

Use this skill **first** when you need a **map** from **`@t4h.framework/*`** packages to the matching **`framework-<name>/SKILL.md`**, plus **project layout** conventions. For **end-to-end workflow scaffolding** (how pieces fit together), also open **`framework-building-workflows`**.

## What this folder is

Under **`cursor-framework-plugins/plugins/framework/skills/`** there is **one subdirectory per `@t4h.framework/*` package** (plus **meta** skills and **this** index). Each **`framework-<name>/SKILL.md`** for a package documents **how that package works and how to use its APIs** (imports, types, typical flows).

| Skill folder | NPM package | Open this skill when you… |
|--------------|-------------|---------------------------|
| **`framework-building-workflows`** | *(cross-package)* | Assemble **`App`** + **`Workflow`** files, wire **`main`** and **`workflows/`**, and **route** to the right **`framework-*`** skill per import (HTTP, auth, ORM, Amber, etc.). |
| **`framework-amber`** | `@t4h.framework/amber` | Ticket or **protocol** integration and creation (**`Amber`** / **`Ticket`**, REST activities, Basic auth, **`runtime-claims`**). |
| **`framework-cache`** | `@t4h.framework/cache` | Implement or explain **`CacheClaim`**, **`get`/`set`**, TTL, or token caching with HTTP OAuth. |
| **`framework-core`** | `@t4h.framework/core` | Build **`App`**, **`Workflow`**, **`Authorization`**, **`Env`**, **`Claim`**, **`History`**, **`startWorkflow`** / **`StartWorkflowActivity`** / **`WorkflowRef`**, **`Binary`**, **`Serializable`**. |
| **`framework-fs`** | `@t4h.framework/fs` | Use **`FileSystemClaim`** (**`read`/`write`**) or understand how HTTP activities persist response bodies. |
| **`framework-http`** | `@t4h.framework/http` | Call **`http`/`HttpClient`**, **`HttpRequestActivityInput`**, **`HttpAuth`**, **`OAuth2`**, **`HttpClientRequestClaim`**. |
| **`framework-jwks`** | `@t4h.framework/jwks` | Configure **`JWKSAuthorization`**, **`JWKSAuthorizationContext`**, or the **`jwks-authorize`** flow (JWT vs raw signature). |
| **`framework-oidc`** | `@t4h.framework/oidc` | Configure **`OIDCAuthorization`**, discovery, client credentials, introspection. |
| **`framework-orm`** | `@t4h.framework/orm` | Define **`@Entity`/`@Property`**, **`ORM`**, **`find`/`create`**, **`Ref`**. |
| **`framework-sftp`** | `@t4h.framework/sftp` | Use **`Sftp`** (remote file operations) and **`SftpClaim`-shaped** activity inputs. |
| **`framework-smtp`** | `@t4h.framework/smtp` | Use **`Smtp`**, **`send`**, **`SmtpSendEmailActivityOptions`**. |
| **`framework-skills-index`** | *(this file)* | Need the **map above**, **project layout rules**, or **which** skill to read. |

**This skill does not replace the package skills** — it routes you to them. For API details, open the matching **`framework-<name>/SKILL.md`**.

## When to use which skill (quick rules)

- **New app shell and wiring workflows into `App`** (entry file, `workflows/` imports, which package skill to read next) → **`framework-building-workflows`**, then the package skills below as needed.
- **New app shell** (`App`, default export, workflows list) → **`framework-core`**.
- **Starting a child workflow from a workflow** (`startWorkflow`, **`WorkflowClaim`**, nested runs) → **`framework-core`**.
- **Incoming request auth** (manifest + `Authorization` subclasses) → **`framework-jwks`** or **`framework-oidc`** depending on JWKS vs OIDC discovery/introspection.
- **Outbound HTTP from a workflow** (`http.get`, `http.post`, OAuth2) → **`framework-http`**; if you subclass OAuth2 or rely on cached tokens → also **`framework-cache`**.
- **Email** → **`framework-smtp`**. **SFTP** → **`framework-sftp`**. **Database entities / ORM** → **`framework-orm`**.
- **Low-level FS or HTTP plumbing** (`FileSystemClaim`, response body handling) → **`framework-fs`** (and **`framework-http`** for the client surface).
- **Amber tickets / protocols and channel API** (`Amber`, `Ticket`, ticket activities) → **`framework-amber`** (and **`framework-http`** / **`framework-core`** for HTTP and `History`/`AsyncActivity`).

If the task spans multiple packages (e.g. `App` + `OIDCAuthorization` + `http`), read **core**, **oidc**, and **http** in that order for structure, then auth, then calls.

## Recommended project layout (workflows + main entry)

Follow these rules unless the user explicitly asks for a different structure:

1. **One entry file** at the app root (e.g. **`src/main.tsx`** or **`src/main.ts`**) that **default-exports** **`new App({ ... })`**.
2. **Workflows live in `src/workflows/`**, **one primary workflow per file** (or a small, clearly related group). Export named **`Workflow`** instances from each file.
3. **Import those workflows into the entry file** and pass them in **`App`**’s **`workflows`** array.
4. **Shared env / constants** in something like **`src/constants/environment.ts`** (using **`env`** from **`@t4h.framework/core`** where applicable).

**Example structure** (follow this shape in the app you are editing):

```text
src/
  main.tsx              # or main.ts — default export: new App({ ... })
  constants/
    environment.ts      # env.* and validation
  workflows/
    my-workflow.ts      # one file per workflow (or closely related group)
    ...
```

**`src/constants/environment.ts`** — read typed env and fail fast if required vars are missing:

```ts
import { env } from '@t4h.framework/core'

export const API_URL = env.API_URL!
if (!API_URL) throw new TypeError('API_URL is not defined')
```

**`src/workflows/my-workflow.ts`** — define the workflow here; export it by name:

```ts
import { Workflow } from '@t4h.framework/core'

export const myWorkflow = new Workflow({ id: 'my-workflow' }, async () => {
  // workflow body
})
```

**`src/main.tsx`** — wire **`App`**, optional **`authorization`**, and import workflows from **`./workflows/`**:

```ts
import { App } from '@t4h.framework/core'
import { OIDCAuthorization } from '@t4h.framework/oidc'

import { CLIENT_ID, CLIENT_SECRET, DISCOVERY_URL } from './constants/environment.js'
import { myWorkflow } from './workflows/my-workflow.js'

export default new App({
  id: 'my-app',
  description: '…',
  authorization: new OIDCAuthorization({
    discoveryUrl: DISCOVERY_URL,
    clientId: CLIENT_ID,
    clientSecret: CLIENT_SECRET,
    token: '<<headers.authorization>>',
  }),
  workflows: [myWorkflow],
})
```

Use **`.js` extensions in import specifiers** when the build emits ESM and resolves specifiers that way (e.g. `'./workflows/my-workflow.js'`).

## What to clarify with the user (do not guess)

When anything below is missing or ambiguous, **stop and ask the user** before implementing or changing behavior:

- **Auth model**: JWKS vs OIDC vs none; tenant-specific URLs; which headers or body fields map to **`token`**, **`signature`**, etc.
- **IDs and URLs**: tenant IDs, API base URLs, webhook URLs, skill/channel UUIDs (example code often contains **placeholders**).
- **Environment variables** required at runtime and how they are set in their deployment.
- **File naming**: **`main.ts` vs `main.tsx`** — match the existing project or ask which the user prefers.

## Node and dependencies

- Framework packages target **Node `>=22`** (see each package’s **`package.json`**).
- Install **`@t4h.framework/*`** packages that you **import**; respect **peer dependencies** (e.g. **`http`** peers **`cache`**, **`core`**, **`fs`**).
