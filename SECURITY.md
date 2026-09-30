# Security

Do not report security vulnerabilities, credentials, connection strings, private
run records or authenticated browser state in public GitHub issues.

Report security vulnerabilities in Microsoft-owned repositories through the
[Microsoft Security Response Center](https://msrc.microsoft.com/create-report).
See Microsoft's [coordinated vulnerability disclosure guidance](https://aka.ms/security.md/cvd).
Include the affected version, a sanitized reproduction and the security impact.

## Supported use

This is a supervised sample for **dedicated disposable synthetic environments**,
not a production migration controller or a distributed authorization boundary.
The MongoDB source write fence is server-wide. Never run it against a shared
source or customer data. Keep the app bound to loopback and use authenticated,
host-key-verified SSH; do not expose it with a public tunnel.

The operator controls provisioning. The backend uses read-only Azure permissions
and separate scoped database identities. Azure DocumentDB's native secondary-user
roles are cluster-wide, so its target must remain dedicated to this sample.
An administrator can bypass the sample's controls; local JSON, tags and SQLite
records are not independent authorization or proof of cloud execution.

Keep credentials, state, diagnostics and archives in protected storage outside
Git. `.gitignore` is a convenience, not a security boundary. Never weaken TLS,
disable required organization controls or erase a journal to bypass a stop.

Cancellation and deallocation are not deletion or guaranteed rollback.
Independently verify process termination, source unlock and data/resource cleanup.
After a target-only write attempt, returning to the source requires a separately
designed reconciliation or reverse-sync procedure.
