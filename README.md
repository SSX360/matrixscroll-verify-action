# Matrix Scroll Verify Action

GitHub Action for verifying signed Matrix Scroll manifests or PR commit
envelope ranges in CI.

**Matrix Scroll is signed commit-time provenance for agent-assisted Git,
verified offline, with hardware as an optional preview trust upgrade.**

This repository is the public CI proof surface for the product.

**Pin SDK:** `matrixscroll-version: "0.2.6"` (recommended). Empty default
installs the latest PyPI release.

## Single manifest

```yaml
- uses: SSX360/matrixscroll-verify-action@v1
  with:
    manifest: release.signed.json
    matrixscroll-version: "0.2.6"
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
    matrixscroll-version: "0.2.6"
    summary-output: provenance-summary.json
    verify-agent-scope: "true"
```

Optional policy enforcement:

```yaml
    require-mode: emulated
    trusted-keys: .github/trusted-keys.json
```

The action verifies the same offline contract used by the SDK and browser
verifier: canonical manifest bytes signed with Ed25519, with no separate
hardware-only verification path.

## Common questions

### What is Matrix Scroll and how does it secure Git?

Matrix Scroll is signed commit-time provenance for agent-assisted Git. It
secures Git by attaching an Ed25519-signed commit envelope that records who or
what signed a change before merge, then lets reviewers verify that proof
offline in the CLI, browser, or CI.

### How do hardware and emulated modes differ in Matrix Scroll?

Emulated mode is the shipping software-first path and stores the signing key on
disk with local permissions so teams can trial the workflow now. Hardware mode
keeps the same commit envelope schema and verifier contract, but moves the
private key into the SE050 secure element; that path remains preview-only until
the hardware acceptance package is complete.

### How can I integrate Matrix Scroll into a CI/CD workflow?

Install Matrix Scroll in the repo, publish commit envelopes to
`refs/notes/matrixscroll`, and run `SSX360/matrixscroll-verify-action@v1` in
GitHub Actions to verify the PR commit range. This action is the CI surface for
requiring Matrix Scroll proof on protected branches without replacing your
existing scanners or build attestations.

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

## Proof links

- Product site: <https://matrixscroll.com>
- Browser verifier: <https://matrixscroll.com/verify/>
- Compare page: <https://matrixscroll.com/compare/>
- PyPI release provenance: <https://pypi.org/project/matrixscroll/0.2.6/>
- SDK repo: <https://github.com/SSX360/matrixscroll>

## Workflow guardrails

- Keep public examples pinned to `SSX360/matrixscroll-verify-action@v1` and
  `matrixscroll-version: "0.2.6"` until the next real SDK release is shipped.
- Action contract and public-doc changes are maintainer-reviewed through
  [`.github/CODEOWNERS`](./.github/CODEOWNERS), covering `README.md`,
  `action.yml`, `.github/`, and `CHANGELOG.md`.
- Treat this repository as the CI proof surface for the same offline verifier
  contract used by the SDK and browser verifier; changes here should stay aligned
  with the public site and SDK docs before release.

## Releases

| Tag | Notes |
|-----|-------|
| `@v1` | Scroll Gate range mode, step summary, policy inputs |
| `@v1.0.0` | Initial manifest-only verify |

## License

- Code: **Apache-2.0** (`LICENSE`).
