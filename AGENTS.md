# AGENTS.md — api-mesh-starter-kit

Template repository that developers clone to bootstrap an API Mesh project. Not a deployed service — the repo itself is the deliverable. Contains a sample `mesh.json`, environment config, GitHub Actions workflows for deployment and load testing, and Codespaces support.

---

## Domain Concepts

### `mesh.json` + `.env` are the primary artifacts
`mesh.json` is the mesh configuration users edit to define sources, transforms, and plugins. It supports environment variable interpolation using `{{env.VARIABLE_NAME}}` syntax, resolved from `.env` at runtime. Both files must stay in sync — referencing an undefined env var fails silently at mesh startup.

### Create-vs-update is determined by string matching
The deploy workflow runs `aio api-mesh:get` and checks whether its output contains the string `"No mesh found"` to decide whether to create or update. If the get command fails for any other reason, the string won't match and the workflow will attempt an update instead of surfacing the error.

---

## Critical Gotchas

### 1. `yarn install` silently installs the AIO CLI plugin and enables telemetry
The `postinstall` hook runs `aio plugins:install @adobe/aio-cli-plugin-api-mesh` and enables telemetry in `~/.config/aio` as a background process. This happens on every install. Errors from the background process are not surfaced.

### 2. `continue-on-error: true` on the Get Mesh step can mask real failures
The workflow step that fetches the existing mesh has `continue-on-error: true`. If `aio api-mesh:get` fails for a reason other than the mesh not existing, the workflow continues and attempts an update — potentially against a mesh that is in an unexpected state.

### 3. Load test reports auto-commit to the repo if `UPLOAD_REPORT=true`
When the `loadTest.yml` workflow runs with `UPLOAD_REPORT=true`, it converts the K6 HTML report to PDF using Puppeteer and commits it directly to the repo. Repeated load test runs with this flag will grow the git history.
