# Support and scope

This repository is a sample, provided under the [MIT license](LICENSE), not a
production migration service or a support SLA.

For sample-code issues, use this repository's GitHub issues with a sanitized
synthetic reproduction. Use [MSRC](SECURITY.md) for security reports. For an Azure
service issue, use your normal Azure support channel rather than posting
subscription details, credentials or private run evidence publicly.

The documented setup path uses a Windows operator with PowerShell 7 and an Ubuntu
24.04 backend VM. Linux also runs the core Node.js tests; that does not imply a
tested Linux/macOS operator-provisioning path. See [setup](docs/setup.md) for
prerequisites and [validation](docs/validation.md) for exact evidence.

This sample covers only the bounded synthetic orders/inventory workload and
offline BSON migration. It does not promise universal compatibility, production
availability, online catch-up, zero downtime/data loss, automatic rollback,
automatic cloud cleanup or unlimited unattended execution.

Retention tags do not stop billing or delete resources. The named operator owns
cleanup of the exact attributed environment and must verify its outcome.
