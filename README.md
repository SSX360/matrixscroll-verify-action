# Matrix Scroll Verify Action

GitHub Action for verifying signed Matrix Scroll manifests in CI.

## Usage

```yaml
- uses: SSX360/matrixscroll-verify-action@v1
  with:
    manifest: release.signed.json
    matrixscroll-version: "0.1.1"
```

## Exit codes

| Exit | Meaning |
|------|---------|
| 0 | Signature valid |
| 1 | Configuration error |
| 2 | Verification failed |

See [matrixscroll](https://github.com/SSX360/matrixscroll) for the protocol and SDK.
