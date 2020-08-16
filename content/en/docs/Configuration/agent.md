---
title: "Configuring Natlas Agent"
linkTitle: "Configuring Natlas Agent"
date: 2020-08-16
weight: 2
description: >
  Learn more about configuring the Natlas Agent
---

## Environment Config

All agent configurations are controlled via environment variables. To make modifications to your agent, you can modify environment variables to match your specifications. These options should be placed in a file called `.env` that gets mounted into your container in `/opt/natlas/natlas-agent/.env`.

### Server Communication

| Variable | Default | Explanation |
|---|---|---|
| `NATLAS_SERVER_ADDRESS` | `http://127.0.0.1:5000` | The server to get work from and submit work to.
| `NATLAS_IGNORE_SSL_WARN` | `False` | Ignore certificate errors when connecting to `NATLAS_SERVER_ADDRESS`
| `NATLAS_REQUEST_TIMEOUT` | `15` (seconds) | Time to wait for the server to respond
| `NATLAS_BACKOFF_MAX` | `300` (seconds) | Maximum time for exponential backoff after failed requests
| `NATLAS_BACKOFF_BASE` | `1` (second) | Incremental time for exponential backoff after failed requests
| `NATLAS_MAX_RETRIES` | `10` | Number of times to retry a request to the server before giving up

### Agent Authentication

| Variable | Default | Explanation |
|---|---|---|
| `NATLAS_AGENT_ID` | `None` | ID string that identifies scans made by this host, required for agent authentication, optional if agent authentication is not required. Get this from your `/user/` page on the Natlas server if agent authentication is enabled.
| `NATLAS_AGENT_TOKEN` | `None` | Secret token needed when agent authentication is required. Generate this with the ID on the `/user/` page on the Natlas server.

### Agent Behavior

| Variable | Default | Explanation |
|---|---|---|
| `NATLAS_MAX_THREADS` | `3` | Maximum number of concurrent scanning threads
| `NATLAS_SCAN_LOCAL` | `False` | Don't scan local addresses

### Instrumentation

| Variable | Default | Explanation |
|---|---|---|
| `SENTRY_DSN` | `""` | If set, enables automatic reporting of all exceptions to a [Sentry.io instance](https://sentry.io/). Example: `http://mytoken@mysentry.example.com/1` |

### Development Settings

| Variable | Default | Explanation |
|---|---|---|
| `NATLAS_SAVE_FAILS` | `False` | Optionally save scan data that fails to upload for whatever reason. If you enable this, you will also want to mount a persistent data store at `/data`
| `NATLAS_VERSION_OVERRIDE` | `None` | **Danger**: This can be optionally set for development purposes to override the version string that natlas thinks it's running. Doing this can have adverse affects and should only be done with caution. The only reason to really do this is if you're developing changes to the way host data is stored and presented.

## Example ENV

At minimum, you'll want to define these configuration options. Get `NATLAS_AGENT_ID` and `NATLAS_AGENT_TOKEN` from the `/user/` profile page of your Natlas server.

```bash
NATLAS_SERVER_ADDRESS=https://natlas.io
NATLAS_MAX_THREADS=25
NATLAS_AGENT_ID=this-is-example-id
NATLAS_AGENT_TOKEN=this-is-obviously-an-example-token
```
