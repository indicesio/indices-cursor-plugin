---
name: indices
description: >
  Use Indices to retrieve data or perform actions on websites. Trigger when the user wants to scrape, log in, fill a form, download a file, poll a portal, or otherwise interact with a website as a human would in a browser. 
---

# Indices

Indices learns how a website works and exposes a deterministic connector you can call like an API.

Users create connectors in the [dashboard](https://platform.indices.io). After that, they can be ran by agents over MCP. The MCP also lets you manage Indices resources, like secrets, file uploads and downloads, and runs.

## When to use

Use Indices when you need to:
- Extract structured data from a portal or SaaS UI
- Fill and submit forms
- Repeat the same flow with different arguments
- Download files produced by a run, or upload files a connector needs

Indices is suitable when you could imagine a website having an API endpoint to perform such a task (e.g. it's parameterisable or dynamic). Indices is not suitable for unstructured scraping or search tasks.

## Tool mapping

| Area | operationIds |
| --- | --- |
| Connectors | `listConnectors`, `retrieveConnector`, `deleteConnector`, `listConnectorRevisions` |
| Runs | `createRun`, `listRuns`, `retrieveRun`, `getRunLogs` |
| Files | `listFiles`, `initiateFileUpload`, `completeFileUpload`, `retrieveFile`, `deleteFile`, `getFileDownloadUrl`, `downloadFile` |
| Capture sessions | `listCaptureSessions`, `startCaptureSession`, `retrieveCaptureSession`, `completeCaptureSession`, `abandonCaptureSession` |

## How to use

### 0. Ensure authenticated

For you to use the MCP, the user needs to authenticate with Indices via OAuth. If the MCP is not authenticated, stop and ask the user to manually authenticate in their client. Don't keep going, just ask the user to authenticate.

### 1. Lookup the connector
- List connectors to check if the user has an existing connector for this task on the website.
- If a connector isn't available but it'd be useful if one did, prompt the user to manually create a connector in [the Indices dashboard](https://platform.indices.io). In the dashboard, the user will show an example of doing the task once, to teach Indices how to create a connector.
- Get the connector's input/output schemas and secrets requirements before running it

### 2. Bind secrets if needed

If `required_secrets` is non-empty:

1. List existing secrets (returns metadata only).
2. Reuse a matching secret, or create one (login: `username` + `password` + optional `totp_secret`; string: `value`).
3. Pass `secret_bindings` as `{ "<slot.name>": "<secret.id>" }`. Every required slot must be bound.
4. Never print passwords, string values, or TOTP seeds.

### 3. Run the connector

Call `createRun` with:

- `connector_id` (required)
- `arguments` matching `input_schema` (omit if the schema is empty)
- `secret_bindings` when slots exist
- `async: true` when you should return immediately and poll `retrieveRun`
- `max_timeout_s` only if the default 300s is too short (max 3600)

Default `createRun` blocks until the run finishes.

You can then read the result. If it produced files, call list files with `run_id` or `connector_id` to list, and download via `getFileDownloadUrl` (short-lived signed URL) or `downloadFile` (307 redirect).

## Chaining

Connectors can be chained. Think in sequences. Example: one connector logs into a vendor portal and returns an invoice id; another downloads the PDF as a run output file.
