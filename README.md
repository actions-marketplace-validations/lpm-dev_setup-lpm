# Setup LPM

GitHub Action to configure the [LPM](https://lpm.dev) package registry for `npm install`. Supports OIDC (no secrets needed) and token-based authentication.

## Quick Start (OIDC — recommended)

No secrets to manage. Just link your GitHub account at [lpm.dev/dashboard/settings/security](https://lpm.dev/dashboard/settings/security).

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: 22
  - uses: lpm-dev/setup-lpm@v1
  - run: npm ci
```

## With Token (fallback)

For CI platforms without OIDC, or when you prefer static tokens:

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: 22
  - uses: lpm-dev/setup-lpm@v1
    with:
      oidc: "false"
      token: ${{ secrets.LPM_TOKEN }}
  - run: npm ci
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `oidc` | Use OIDC for secret-free auth (requires `id-token: write`) | `"true"` |
| `token` | LPM auth token (fallback when OIDC is unavailable) | `""` |
| `cli-version` | LPM CLI version to install | `"latest"` |

## How OIDC Works

1. GitHub Actions generates a signed identity token (JWT)
2. The action exchanges this token with LPM for a 30-minute read-only install token
3. The token is written to `.npmrc` — `npm install` works immediately
4. No secrets are stored anywhere

If OIDC exchange fails, the action falls back to `${LPM_TOKEN}` environment variable.

**Prerequisite:** Your GitHub account must be linked to your LPM account at [Settings > Security](https://lpm.dev/dashboard/settings/security).

## Full CI + Publish Example

```yaml
name: CI + Publish
on:
  push:
    branches: [main]
    tags: ["v*"]
  pull_request:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - uses: lpm-dev/setup-lpm@v1
      - run: npm ci
      - run: npm test

  publish:
    needs: build
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - uses: lpm-dev/setup-lpm@v1
      - run: npm ci
      - run: npx lpm publish
```

## License

MIT
