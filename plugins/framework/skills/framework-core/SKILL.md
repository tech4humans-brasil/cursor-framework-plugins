---
name: t4h-framework-core
description: How to use @t4h.framework/core App, Workflow, Authorization, Env, Claim, History, Binary, Serializable, startWorkflow, StartWorkflowActivity, WorkflowClaim, and WorkflowRef. Use when building T4H apps, workflows, nested workflow starts, or importing from @t4h.framework/core.
---

# @t4h.framework/core

## Requirements

- **Node.js**: `>=22`.
- Optional workflow validation uses **Ajv** + **TypeBox** when a **`Workflow`** has a **`schema`**.

Exports are listed in `packages/core/src/core.ts` (e.g. `App`, `Workflow`, `Authorization`, `Env`, `History`, `Claim`, `Binary`, `startWorkflow`, `StartWorkflowActivity`, `WorkflowClaim`, `WorkflowRef`, other activities, `Serializable`, exceptions).

## `App`

**`AppOptions`**:

| Field | Required | Purpose |
|-------|----------|--------|
| **`id`** | yes | Application identifier. |
| **`slug`**, **`description`** | no | Manifest metadata. |
| **`authorization`** | no | **`Authorization`** instance (e.g. JWKS, OIDC). |
| **`workflows`** | no | **`Workflow`** instances. |

**`App`**: **`addWorkflow`** throws if two workflows share the same **`id`** when ids are set. **`toManifest()`** includes authorization, each workflow manifest, and env from **`inMemoryEnvDesignToManifest()`**.

## `Workflow`

**`WorkflowOptions`** (two-argument constructor):

| Field | Purpose |
|-------|--------|
| **`id`**, **`description`** | Optional manifest fields. |
| **`schema`** | Optional TypeBox schema; if set, AJV **`validate`** is compiled for the input type. |
| **`authorization`** | Optional per-workflow authorization. |

**Constructors**: `new Workflow(fn)` or `new Workflow(options, fn)`.

## `Authorization`

Subclass **`Authorization`** and implement **`toManifest()`** → **`{ pathToModule: string, context?: TContext }`**. **`pathToModule`** is the module that performs validation; **`context`** is the data that module reads (opaque to core).

## `Env` and `env`

- **`Env.run(entries, callback)`** runs **`callback`** with **`entries`** in async-local storage. **`entries`**: array of `[key, value]` pairs, a plain object, or a **`Map`**.
- **`env`** reads those values by property name (`string | undefined`). Outside **`Env.run`**, access throws **"No env storage"**. Symbol keys are not supported.

## `Claim`

**`Claim(map)`** returns a proxy: keys (e.g. `http`, `cache`) map to **abstract claim class constructors**. **`Claim.run(claimProviders, fn)`** installs **`{ provide: Constructor, value: Instance }[]`** for the duration of **`fn`**; property access resolves the matching instance. Missing storage or provider throws **`ClaimException`**.

## `History`

**`History.bind(ActivityClass)`** wraps activities so calls are recorded/replayed consistently (used across HTTP, ORM, SFTP, SMTP).

**`History.reconciler(ActivityClass, ...args)`** (used by helpers such as **`startWorkflow`**) reads **`AsyncActivity`** history entries and returns an **`AsyncActivityAwaitable`** with **`wait()`** when the activity is async, or **`PushActivity`** when history is exhausted and the runtime must execute the activity.

## `startWorkflow`, `StartWorkflowActivity`, and `WorkflowClaim`

**`startWorkflow(workflow, input)`** is **`History.reconciler(StartWorkflowActivity, workflow, input)`**. It starts another **`Workflow`** from the current workflow run: **`toInput`** calls **`claims.workflow.start(workflow, input)`** on the abstract **`WorkflowClaim`**, then the activity goes **async** unless **`parallel`** is set (see below). The reconciler returns an awaitable; **`await (await startWorkflow(...)).wait()`** yields a **`WorkflowRef`** when the child run completes (see **`framework/examples/basic/src/workflows/nested-start-demo.ts`** in the framework monorepo).

**`WorkflowClaim`** (claim key **`workflow`**) — abstract: implement **`start(workflow, input)`** in the runtime to enqueue or start the target workflow; return value is **`Serializable | Promise<Serializable>`**.

**`StartWorkflowActivity`** extends **`AsyncActivity`**. **`claims`**: **`new Claim({ workflow: WorkflowClaim })`**.

**`StartWorkflowActivityInput<TWorkflow>`** — **`WorkflowInput<TWorkflow>`** plus:

| Field | Notes |
|-------|--------|
| **`parallel`** | If **`true`**, **`bootstrap`** completes immediately with **`{ status: 'fulfilled', output: null }`** (no pending async step for the child result). If omitted/false, the activity stays **pending** until **`advance`** receives the async payload. |

**`advance`**: **`status: 'fulfilled'`** → fulfilled; **`'rejected'`** → rejected.

**`toOutput`**: wraps the payload in **`WorkflowRef`**.

## `WorkflowRef`

**`WorkflowRef`** — **`status`**: **`'fulfilled' | 'rejected'`**; **`output`**: **`Serializable`** (child workflow result or rejection payload).

## `Serializable` and `Binary`

- **`Serializable`**: JSON-like recursive types plus **`Binary`** and **`SerializableClass`** (see **`SerializableHelper`**).
- **`Binary`**: abstract; **`contentType`**, **`contentLength`**, **`stream()`**.
