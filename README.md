# flex-plugin-actions

Official reusable GitHub Actions workflow for publishing FlexStudio plugins to the marketplace.

## Usage

In your plugin repository, create `.github/workflows/publish.yml`:

```yaml
name: Publish to FlexStudio Marketplace

on:
  release:
    types: [published]

jobs:
  publish:
    uses: ENIAC-Tech/flex-plugin-actions/.github/workflows/publish.yml@v1.1.0
    with:
      flexcli-package: "https://github.com/ENIAC-Tech/flexcli/tarball/refs/heads/v2"
    secrets:
      webhook-secret: ${{ secrets.FLEX_MARKETPLACE_WEBHOOK_SECRET }}
```

## Setup

1. Register your plugin in FlexDesigner (Marketplace → My Uploads → Publish Plugin)
2. Copy the generated webhook secret
3. Add it to your repo: **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `FLEX_MARKETPLACE_WEBHOOK_SECRET`
   - Value: the secret from step 2

## Releasing

Create a GitHub Release with a semver tag (e.g. `v1.2.0`). The workflow runs automatically and:

1. Reads `manifest.json` to detect `native` and `platforms`
2. For non-native plugins: builds once on Ubuntu, produces a universal `.flexplugin`
3. For native plugins: runs a matrix build across all declared platforms in parallel
4. Uploads `.flexplugin` artifact(s) to the GitHub Release
5. Sends a signed webhook notification to the marketplace server

The marketplace server independently fetches and validates all artifacts from the Release — it does not trust the workflow payload beyond the notification event itself.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `flexcli-package` | No | `https://github.com/ENIAC-Tech/flexcli/tarball/refs/heads/v2` | Passed to `npm install -g` (tarball URL or `@eniac/flexcli@version` once published) |

## Secrets

| Secret | Required | Description |
|---|---|---|
| `webhook-secret` | Yes | HMAC-SHA256 key for signing the marketplace notification |

## Native plugins

Set `native: true` in `manifest.json` to enable the matrix build. The workflow maps each entry in `platforms` to the appropriate GitHub-hosted runner:

| Platform | Runner |
|---|---|
| `win32-x64` | `windows-latest` |
| `darwin-arm64` | `macos-latest` |
| `darwin-x64` | `macos-13` |
| `linux-x64` | `ubuntu-latest` |

## Pinning the workflow version

It is strongly recommended to pin to a specific release tag rather than `@main`:

```yaml
uses: ENIAC-Tech/flex-plugin-actions/.github/workflows/publish.yml@v1.1.0
```

The marketplace server rejects workflow references that use `@main` or other mutable refs.

## Migrating from workflow `@v1`

Reusable workflow `@v1` expects the legacy npm-only flexcli install. Plugin scaffolding that uses `flexcli plugin-v2` should pin **`@v1.1.0`** (or newer) and pass **`flexcli-package`** (see usage example above). The previous input `flexcli-version` was removed in favor of `flexcli-package`.
