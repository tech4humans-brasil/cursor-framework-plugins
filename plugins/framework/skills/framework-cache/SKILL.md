---
name: t4h-framework-cache
description: How to use @t4h.framework/cache CacheClaim get/set, TTL, and Serializable values; pairing with @t4h.framework/http OAuth2. Use when implementing caches or imports from @t4h.framework/cache.
---

# @t4h.framework/cache

## Requirements

- **Node.js**: `>=22`.
- **Peer dependency**: `@t4h.framework/core` (`workspace:^`).

## Import

```ts
import { CacheClaim, type CacheClaimSetOptions } from '@t4h.framework/cache'
```

## `CacheClaim`

| Member | Purpose |
|--------|--------|
| **`get<T extends Serializable>(key: string)`** | Returns the stored value, or `undefined` (sync or async). |
| **`set(key, value: Serializable, options?)`** | Stores a value. **`options.ttl`** is optional (milliseconds). |

**`Serializable`** comes from `@t4h.framework/core`—import it there when typing values.

## Using with `@t4h.framework/http`

**`OAuth2`** subclasses cache tokens under **`oauth2:token:${clientId}`** with a TTL derived from **`expires_in`**. Declare **`CacheClaim`** wherever **`HttpAuth` / `OAuth2`** is used so token get/set works.
