---
name: t4h-framework-smtp
description: How to use @t4h.framework/smtp Smtp constructor, send options, and SmtpSendEmailActivity. Use when sending email from workflows or importing from @t4h.framework/smtp.
---

# @t4h.framework/smtp

## Requirements

- **Node.js**: `>=22`.
- **Peer dependency**: `@t4h.framework/core` (`workspace:^`).

Import from **`@t4h.framework/smtp`**.

## `Smtp`

**`new Smtp(connectionString, options?)`**

| Parameter | Purpose |
|-----------|--------|
| **`connectionString`** | First argument; passed through to the send activity / claim. |
| **`options`** | Optional **`SmtpOptions`**: default **`from`**. |

**`Smtp`** extends **`SerializableClass`**; **`toSerializable` / `fromSerializable`** round-trip **`connectionString`** and **`options`**.

## `smtp.send(options)`

Runs **`SmtpSendEmailActivity`** (**`History.bind`**). Message-level **`from`** overrides **`Smtp.options.from`**.

## `SmtpSendEmailActivityOptions`

| Field | Required | Purpose |
|-------|----------|--------|
| **`to`** | yes | **`string`** or **`readonly string[]`**. |
| **`subject`** | yes | Subject. |
| **`html`** | yes | HTML body. |
| **`from`** | no | Overrides default **`from`**. |
| **`cc`**, **`bcc`** | no | **`string`** or **`readonly string[]`**. |
| **`attachments`** | no | **`{ filename: string; content: Binary }[]`**. |

The activity normalizes **`to`**, **`cc`**, **`bcc`** to arrays and builds **`SmtpSendEmailActivityInput`**.

## `SmtpSendEmailClaim`

Abstract **`send(input: SmtpSendEmailActivityInput)`** returning **`ISmtpSendEmailOutput`** (see **`SmtpSendEmailOutput`**).
