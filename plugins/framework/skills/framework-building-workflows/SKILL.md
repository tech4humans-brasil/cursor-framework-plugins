---
name: framework-building-workflows
description: Guides assembling T4H apps with Workflow definitions, default-export App wiring, and routing to the correct framework-* skill per @t4h.framework package. Use when scaffolding workflows, structuring main plus src/workflows, nesting startWorkflow, or choosing which package skill to read for HTTP, auth, ORM, Amber, cache, FS, SFTP, or SMTP.
---

# Building T4H workflows and apps

## Start here

1. Open **`framework-skills-index/SKILL.md`** for the **full package → skill map**, layout rules (**`main`** + **`workflows/`** + **`constants/`**), and **what to ask the user** when requirements are unclear.
2. Open **`framework-core/SKILL.md`** for **`Workflow`**, **`App`**, **`History`**, **`startWorkflow`**, **`Env`**, **`Claim`**, **`Serializable`**, **`Binary`**.
3. For anything **outside core**, use the **routing table below** so the right **`framework-<name>/SKILL.md`** is loaded for API details.

## Assembly pattern (entry + workflows)

Typical shape:

- **`src/main.tsx`** or **`src/main.ts`** — **default export** **`new App({ id, description, authorization?, workflows })`** from **`@t4h.framework/core`**.
- **`src/workflows/*.ts`** — one **primary `Workflow` per file** (or a small related group); **named exports**; import them into **`main`** and list them in **`App`**’s **`workflows`** array.
- **`src/constants/environment.ts`** — typed **`env`** from **`@t4h.framework/core`**, validate required vars at startup (see **`framework-skills-index`** and project security rules).

**`App.workflows`**: list every **`Workflow`** the runtime must register (same pattern as sibling apps in your repo—often all exported workflows from **`workflows/`**).

**`authorization`**: optional; when present, subclass from **`@t4h.framework/jwks`** or **`@t4h.framework/oidc`** (see those skills). Use **manifest placeholders** (e.g. **`<<headers…>>`**, **`<<body…>>`**) rather than hardcoding secrets in source.

## `Workflow` body — which skill to open

| You need to… | Open skill | Package |
|--------------|------------|---------|
| Define **`new Workflow`**, input schema, **`startWorkflow`**, **`WorkflowRef`** | **`framework-core`** | `@t4h.framework/core` |
| Call outbound HTTP, **`http`**, OAuth2, retries | **`framework-http`** | `@t4h.framework/http` |
| Cache tokens or arbitrary values, **`CacheClaim`** | **`framework-cache`** | `@t4h.framework/cache` |
| Read/write files via **`FileSystemClaim`** (e.g. HTTP body persistence) | **`framework-fs`** | `@t4h.framework/fs` |
| JWKS manifest auth, webhook signatures | **`framework-jwks`** | `@t4h.framework/jwks` |
| OIDC discovery, client credentials, introspection | **`framework-oidc`** | `@t4h.framework/oidc` |
| **`@Entity`**, **`ORM`**, **`ORMClaim`** activities | **`framework-orm`** | `@t4h.framework/orm` |
| SFTP operations | **`framework-sftp`** | `@t4h.framework/sftp` |
| **`Smtp`**, email activities | **`framework-smtp`** | `@t4h.framework/smtp` |
| **`Amber`**, **`Ticket`**, ticket/protocol REST flows, **`runtime-claims`** | **`framework-amber`** | `@t4h.framework/amber` |

**Combined flows**: e.g. **`App`** + **`OIDCAuthorization`** + **`http`** → read **core** (structure), **oidc** (auth), **http** (calls); add **cache** if OAuth2/token storage is involved. Amber webhooks often pair **amber** + **http** + **core**.

## Structural inspiration (do not copy literals)

Use these only as **layout** references—adapt IDs, URLs, and secrets to **`env`** and team conventions:

- **Multiple workflows + OIDC-style `authorization`**: framework monorepo **`examples/basic`** — entry file imports several named workflows from **`./workflows/`** and registers them on **`App`**.
- **Single (or focused) workflow + JWKS `authorization`**: **`force-manual-process`** — entry imports workflow(s) from **`./workflows/`**, **`JWKSAuthorization`** with placeholder-driven **`url`** / signature fields, env-based base URL in **`constants`**.

Neither snippet should be pasted wholesale; mirror **file boundaries** and **import graph**, not environment-specific strings.

## Checklist before shipping

- [ ] Workflow inputs match **`Workflow`** **`schema`** when using TypeBox (see **core**).
- [ ] Required **`env`** vars are documented and validated in **`constants/environment.ts`**.
- [ ] Each **`@t4h.framework/*`** import used in workflow code has its **matching skill** consulted for types and claims.
- [ ] No secrets or production tokens committed—only placeholders or **`env`**.

## See also

- **`framework-skills-index`** — index table and **when to use which skill** quick rules.
- **`framework-core`** — **`Workflow`**, **`App`**, **`startWorkflow`**.
