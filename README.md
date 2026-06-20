# Matrix Scroll Verify Action

GitHub Action for verifying signed Matrix Scroll manifests or PR commit envelope ranges in CI.

**Pin SDK:** `matrixscroll-version: "0.2.5"` (recommended). Empty default installs latest PyPI.

## Single manifest

```yaml
- uses: SSX360/matrixscroll-verify-action@v1
  with:
    manifest: release.signed.json
    matrixscroll-version: "0.2.5"
```

## PR commit range (Scroll Gate)

Developers publish envelopes to git notes before opening a PR:

```bash
matrixscroll envelope-publish-notes --base origin/main --head HEAD
git push origin refs/notes/matrixscroll
```

Workflow:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- uses: SSX360/matrixscroll-verify-action@v1
  with:
    head-ref: ${{ github.event.pull_request.head.sha }}
    base-ref: ${{ github.event.pull_request.base.sha }}
    source: notes
    notes-ref: refs/notes/matrixscroll
    fetch-notes: "true"
    matrixscroll-version: "0.2.5"
    summary-output: provenance-summary.json
    verify-agent-scope: "true"
```

Optional policy enforcement:

```yaml
    require-mode: emulated
    trusted-keys: .github/trusted-keys.json
```

## Outputs

| Output | Mode | Description |
|--------|------|-------------|
| `ok` | both | Whether verification passed |
| `device_id` | manifest | Signer device id |
| `mode` | manifest | Signature mode |
| `verified_count` | range | Verified commits |
| `agent_count` | range | Agent-authored commits |
| `human_count` | range | Human-authored commits |
| `modes` | range | Comma-separated modes seen |
| `summary_path` | range | Path to summary JSON when set |

## Exit codes

| Exit | Meaning |
|------|---------|
| 0 | Signature valid |
| 1 | Configuration error |
| 2 | Verification failed |

## Releases

| Tag | Notes |
|-----|-------|
| `@v1` | Scroll Gate range mode, step summary, policy inputs |
| `@v1.0.0` | Initial manifest-only verify |

See [matrixscroll](https://github.com/SSX360/matrixscroll) for the protocol, SDK v0.2.5, and [`docs/SOFTWARE_PRODUCTS.md`](https://github.com/SSX360/matrixscroll/blob/main/docs/SOFTWARE_PRODUCTS.md).
