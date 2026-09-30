# Contributing

For a bug or documentation issue, open a GitHub issue in this repository with
the version/commit, supported operator/backend platform, exact failing step and
a **sanitized synthetic reproduction**. Read [SECURITY.md](SECURITY.md) before
reporting anything sensitive. Do not attach private runtime folders, credentials,
database archives or screenshots containing private identifiers.

Discuss substantial workflow or permission changes before implementation.
Preserve the offline-only scope, explicit approvals, complete reconciliation,
bounded operations, run attribution and fail-closed cutover/rollback behavior.
Do not replace real observations with success flags or add a simulator to the
production entry point.

From the standalone repository root:

```powershell
npm ci --ignore-scripts
npm test
npm run test:browser
```

Use an installed Edge or set `MGD_BROWSER_CHANNEL=chrome` for Chrome.
CI installs its own Chromium and uses `MGD_BROWSER_CHANNEL=chromium`.
Never run a live test without its own explicitly approved sandbox plan.

Contributions to Microsoft projects may require a
[Contributor License Agreement](https://cla.opensource.microsoft.com).
Follow the maintainers' contribution/CLA requirements; this document does not
claim that a CLA bot or other organization integration is already configured.

Please follow the [Code of Conduct](CODE_OF_CONDUCT.md).
