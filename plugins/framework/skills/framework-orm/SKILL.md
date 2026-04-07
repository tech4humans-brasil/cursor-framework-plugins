---
name: t4h-framework-orm
description: How to use @t4h.framework/orm @Entity, @Property, ORM create/findOne/find/exists, Ref, RefCollection, and ORMClaim-shaped activities. Use when defining entities or querying with ORM.
---

# @t4h.framework/orm

## Requirements

- **Node.js**: `>=22`.
- **Peer dependency**: `@t4h.framework/core` (`workspace:^`).

Import from **`@t4h.framework/orm`**.

## `ORMConfig` and **`new ORM(config)`**

| Field | Purpose |
|-------|--------|
| **`connectionString`** | Passed through to persistence (your environment defines the format). |
| **`entities`** | Entity class constructors to register. |

**`ORM`** extends **`SerializableClass`**: default serialization round-trips **`connectionString`** only.

## Repository API

| Method | Purpose |
|--------|--------|
| **`create(entity, data)`** | One insert → **`Ref<T>`**; array → **`RefCollection<T>`**. |
| **`findOne(entity, options)`** | **`FindOneOptions`**: **`where`**, optional **`orderBy`**. **`Ref<T> \| null`**. |
| **`find(entity, options?)`** | Adds **`limit`**, **`offset`**. **`RefCollection<T>`**. |
| **`exists(entity, options)`** | Same options as **`findOne`** → **`boolean`**. |

Implementation goes through **`History`-bound** activities that use **`ORMClaim`**.

## Find options

- **`FindOneOptions<T>`**: **`where`** (**`FindWhere<T>`**), optional **`orderBy`** (**`'asc' \| 'desc'`** per field).
- **`FindOptions<T>`**: **`FindOneOptions`** + optional **`limit`**, **`offset`**.

## Decorators

- **`@Entity(options?)`**: optional **`tableName`**; default from **`getEntityTableName`** (class name if unset).
- **`@Property(options?)`**: **`PropertyMetadata`** (**`type`**, **`nullable`**, **`unique`**, **`default`**, …). See **`metadata`** exports.

## `ORMClaim`

Abstract API used by the activities: **`find`**, **`create`**, **`getSnapshot`**, **`update`**, **`remove`** with types from **`typings/ORM`**.

## `Ref` and `RefCollection`

Handles for rows; **`Ref`** supports snapshot/update/remove via the corresponding activities (see **`packages/orm/src/models`**).

## Barrel exports

**`orm.ts`** exports **`ORMCreateActivity`**, **`ORMFindActivity`**, **`ORMGetSnapshotActivity`**, **`ORMUpdateActivity`**, **`ORMRemoveActivity`**, and **`./typings/ORM`**. **`ORMFindOneActivity`** is used internally by **`ORM.findOne`** but is not in the main barrel—import its module path only if needed.
