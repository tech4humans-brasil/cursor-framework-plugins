---
name: t4h-framework-sftp
description: How to use @t4h.framework/sftp Sftp methods and SftpClaim-shaped operations. Use when calling SFTP from workflows or importing from @t4h.framework/sftp.
---

# @t4h.framework/sftp

## Requirements

- **Node.js**: `>=22`.
- **Peer dependency**: `@t4h.framework/core` (`workspace:^`).

Import from **`@t4h.framework/sftp`**.

## `Sftp`

**`new Sftp(connectionString)`** — stores **`readonly connectionString`** on the instance. Each method delegates to a **`History`-bound** activity that uses **`SftpClaim`**.

## Methods

| Method | Parameters | Notes |
|--------|------------|--------|
| **`append`** | **`filepath`**, **`content: Binary`** | Append to file. |
| **`chmod`** | **`path`**, **`mode: string \| number`** | Mode. |
| **`cwd`** | — | Current remote directory. |
| **`delete`** | **`path`**, **`noErrorOK?: boolean`** | Delete file. |
| **`downloadDir`** | **`path`** | Returns **`Binary`**. |
| **`exists`** | **`path`** | Existence. |
| **`get`** | **`filepath`** | Read as **`Binary`**. |
| **`list`** | **`path`** | **`SftpFileInfo[]`**. |
| **`mkdir`** | **`path`**, **`recursive?: boolean`** | Create dir. |
| **`put`** | **`filepath`**, **`content: Binary`** | Upload. |
| **`copy`** | **`srcPath`**, **`destPath`** | Copy. |
| **`realPath`** | **`path`** | Canonical path. |
| **`rename`** | **`srcPath`**, **`destPath`** | Rename/move. |
| **`rmdir`** | **`path`**, **`recursive?: boolean`** | Remove dir. |
| **`stat`** | **`path`** | **`SftpStat`**. |

## `SftpClaim`

Abstract class with the same operations; inputs are the **`*ActivityInput`** types under **`packages/sftp/src/activities`**.
