<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🔗 Test Gerrit Servers

This repository contains workflows to test connectivity and replication from
production Gerrit servers using the
[gerrit-action](https://github.com/modeseven-lfreleng-actions/gerrit-action).

## Purpose

- Check Gerrit server connectivity across LF-hosted instances
- Test pull-replication plugin functionality
- Verify authentication credentials work as expected
- Ensure API path detection works for each server

## Workflow

The main workflow (`test-gerrit-servers.yaml`) runs tests against each Gerrit
server defined in the `GERRIT_SERVERS` repository variable.

### Manual Trigger

You can trigger the workflow manually with optional parameters:

- **debug**: Enable verbose debug output
- **sync_on_startup**: Trigger pull-replication after container startup

## Configuration

### Repository Variables

#### `GERRIT_SERVERS` (Required)

A JSON array defining the Gerrit servers to test. Each server object should
contain:

<!-- markdownlint-disable MD013 -->

| Field            | Required | Description                                                      |
| ---------------- | -------- | ---------------------------------------------------------------- |
| `name`           | Yes      | Display name for the server (e.g., "Linux Foundation")           |
| `slug`           | Yes      | Short identifier used for container naming and credential lookup |
| `gerrit`         | Yes      | Gerrit server hostname (e.g., "gerrit.linuxfoundation.org")      |
| `api_path`       | No       | API path prefix if not at root (e.g., "/infra", "/r")            |
| `project_filter` | No       | Regex pattern to filter projects, empty = all projects           |

<!-- markdownlint-enable MD013 -->

**Example (use this verbatim as your repository variable):**

```json
[
  {
    "name": "Linux Foundation",
    "slug": "lf",
    "gerrit": "gerrit.linuxfoundation.org",
    "api_path": "/infra",
    "project_filter": ""
  },
  {
    "name": "ONAP",
    "slug": "onap",
    "gerrit": "gerrit.onap.org",
    "api_path": "/r",
    "project_filter": ""
  },
  {
    "name": "O-RAN-SC",
    "slug": "oran",
    "gerrit": "gerrit.o-ran-sc.org",
    "api_path": "/r",
    "project_filter": ""
  },
  {
    "name": "OpenDaylight",
    "slug": "odl",
    "gerrit": "git.opendaylight.org",
    "api_path": "/gerrit",
    "project_filter": ""
  }
]
```

#### Project Filter Examples

The `project_filter` field supports:

- **Empty string** (`""`): Replicate ALL projects from the server
- **Literal name**: Match exact project name (e.g., `"releng/lftools"`)
- **Comma-separated**: Two or more projects (e.g., `"releng/lftools,ci-management"`)
- **Regex pattern**: Match projects by pattern (e.g., `"releng/.*"`)

### Repository Secrets

#### `GERRIT_CREDENTIALS` (Required)

A **base64-encoded** JSON array containing credentials for each Gerrit server.
Each entry must have a `slug` that matches the corresponding entry in
`GERRIT_SERVERS`.

> **Important:** The secret must be base64-encoded to prevent GitHub from
> applying spurious redactions to the console output. GitHub's secret masking
> can interfere with JSON structures containing passwords.

**Step 1: Create a JSON file with your credentials:**

```json
[
  {
    "slug": "lf",
    "username": "your-lf-username",
    "password": "XXXXXXXX"
  },
  {
    "slug": "onap",
    "username": "your-onap-username",
    "password": "XXXXXXXX"
  },
  {
    "slug": "oran",
    "username": "your-oran-username",
    "password": "XXXXXXXX"
  },
  {
    "slug": "odl",
    "username": "your-odl-username",
    "password": "XXXXXXXX"
  }
]
```

**Step 2: Base64-encode the JSON:**

```bash
# On Linux/macOS:
cat credentials.json | base64

# Or inline:
echo '[{"slug":"lf","username":"user","password":"pass"}]' | base64
```

**Step 3: Store the base64-encoded string as the `GERRIT_CREDENTIALS` secret.**

The workflow will automatically decode the base64 content before parsing the
JSON.

**Notes:**

- The password should be the HTTP password from your Gerrit account settings,
  not your SSO/login password
- The `slug` values must match in `GERRIT_SERVERS` and `GERRIT_CREDENTIALS`
- Ensure there are no trailing newlines in your base64 encoding (use
  `base64 -w 0` on Linux if needed)

## Outputs

Each test job outputs:

- Container connectivity status
- API path detection results
- Replication status (when `sync_on_startup` option set)
- SSH host keys for the local Gerrit container

## Related

- [gerrit-action](https://github.com/modeseven-lfreleng-actions/gerrit-action) -
  The GitHub Action used by this workflow
- [Gerrit Code Review](https://www.gerritcodereview.com/) - Gerrit documentation
- [pull-replication plugin][pull-replication] - Plugin documentation

[pull-replication]: https://gerrit.googlesource.com/plugins/pull-replication

## License

Apache-2.0
