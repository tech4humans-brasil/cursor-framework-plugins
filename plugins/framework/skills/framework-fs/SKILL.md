---
name: t4h-framework-fs
description: How to use @t4h.framework/fs FileSystemClaim read/write and Binary. Use when handling filesystem claims or imports from @t4h.framework/fs.
---

# @t4h.framework/fs

## Requirements

- **Node.js**: `>=22`.
- **Peer dependency**: `@t4h.framework/core` (`workspace:^`).

## Import

```ts
import { FileSystemClaim } from '@t4h.framework/fs'
```

## `FileSystemClaim`

| Method | Parameters | Returns |
|--------|------------|---------|
| **`write`** | **`path`**, **`readable: Readable \| Buffer`** | **`Binary`** (sync or async) |
| **`read`** | **`path`** | **`Binary`** (sync or async) |

**`Binary`** is from **`@t4h.framework/core`**.

## With `@t4h.framework/http`

**`AbstractHttpRequestActivity`** uses **`http`** + **`fs`** claims: the response stream is written with **`fs.write`** to a generated path, then exposed as **`Body`**. Projects that use **`HttpRequestActivity`** need **`FileSystemClaim`** available alongside **`HttpClientRequestClaim`**.
