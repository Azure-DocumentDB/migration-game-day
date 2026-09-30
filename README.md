# Migration game day

**Keep your application. Prove the move.**

A self-contained, supervised orders/inventory rehearsal from genuine MongoDB
Community to **managed Azure DocumentDB**, the MongoDB-compatible managed service.

**This first lane is OFFLINE.** Source writes are fenced before the BSON dump and
stay fenced through verification/cutover. There is no change-stream catch-up,
automatic rollback, zero-downtime promise, general compatibility score, or
unattended execution. MongoDB Community is SSPL/source-available, not
OSI-open-source; a dedicated one-member replica set gives this rehearsal
controlled fencing, not production availability.

## Start here: local verification

From this standalone repository's root (the directory containing this README
and `package.json`), use Node.js **24.13+** and npm. These checks require no
Azure sign-in, database, private configuration or previously saved run.
The browser check requires an **already installed Microsoft Edge**; it launches
a new headless browser with a temporary profile and does not use your browser
session. No browser is downloaded by `npm ci --ignore-scripts`.

```powershell
npm ci --ignore-scripts
npm test
npm run test:browser
```

If Chrome is installed instead of Edge, select it explicitly:

```powershell
$env:MGD_BROWSER_CHANNEL = "chrome"
npm run test:browser
```

`npm test` exercises the lifecycle with test doubles and real local subprocesses.
`test:browser` exercises the actual UI against clearly labeled test doubles.
Neither command launches a persistent demo nor proves a managed migration.

## Live launch: a configured environment is required

Live execution additionally requires PowerShell 7 on a Windows operator machine,
Azure CLI with an approved identity, the dedicated source VM and managed target,
verified SSH host keys, scoped database identities, verified native MongoDB tools,
and a **fully initialized private environment directory**.
The repository does not contain that environment or credentials.
`config.example.json` is a schema example, not a runnable configuration.
Do not reuse someone else's private profile, approval, identities or run ledger.
No database or Azure resource is implicitly authorized by this README.

**First-time user:** follow [the complete setup guide](docs/setup.md) before the
live launch commands below. It creates every required private artifact using
checked-in scripts, with explicit approval/review gates. Read the
[validation record](docs/validation.md) for the tested stack and coverage;
local setup-contract tests alone are not proof of an Azure deployment.

Only after an operator has completed and approved that setup, launch on the
configured backend host:

```powershell
npm start -- --config "<absolute-private-directory>\config.json"
```

Use the printed `127.0.0.1` URL, not a public tunnel. The dashboard has no mock
runtime mode. Browser/unit checks inject conspicuously labeled test doubles;
the production entry point always uses real adapters and Azure ARM preflight.
Real connection settings, archives and the SQLite journal must remain in
protected local storage **outside every Git checkout**. The placeholder
[configuration](config.example.json) deliberately cannot run.

The [infrastructure template](infra/main.bicep) creates only new, attributed
resources. It does not install an unattended experiment runner or schedule.
[Prepare-Environment.ps1](scripts/Prepare-Environment.ps1) prepares private
credentials/parameters only after environment approval. Validate the exact
template, network exposure, permissions and quotas before deploying it.
The [source bootstrap](scripts/bootstrap-source.sh) refuses an existing MongoDB
installation, verifies the vendor signing key, enables authentication and binds
MongoDB to loopback only. Replica-set/user initialization and host-key
verification must finish before the application can connect.

**Linux metadata security is a two-step setup.** Provision the VM first, then
use [Enable-SourceMetadata.ps1](scripts/Enable-SourceMetadata.ps1) to install the
supported Microsoft Linux proxy-agent extension, enable Enforce mode and verify
the actual guest service/metadata response. Never enable `proxyAgentSettings`
during initial provisioning of an image without the agent. Do not delete
policy-managed NSG rules or disable Secure Boot to fix provisioning.

## Approved VM-hosted runtime

When the local network cannot reach the managed database, do not silently weaken
TLS or switch routing. Running the same backend and native tools on the source
VM requires explicit approval of that execution location and credential transfer.
The UI remains private:
the server binds `127.0.0.1:4193` on that VM, and an authenticated, strictly
host-key-verified SSH forward exposes it only on the operator's loopback address.
No VM-to-operator Azure CLI cache, token, or browser state is copied.

The [Reader setup](scripts/Grant-PreflightReader.ps1) requires an existing,
positively identified VM system identity and grants only Reader on the six
attributed resources needed by preflight. It refuses broader existing grants.
Corporate user-assigned identities are not repurposed or modified. The entire
reviewed NSG rule set is hash-bound, including mandatory corporate management
rules; a matching rule name alone cannot authorize a changed rule.

After explicit approval, upload only this demo's code/manifests and generated
demo credentials to a protected VM directory. The
[VM preparation script](scripts/prepare-vm-runtime.sh) verifies the Node.js
24.13.0 vendor archive, installs locked production dependencies, pins native
tool binary hashes and initializes/verifies scoped database users. Private
directories must be mode 0700 and credential/config files mode 0600.
Do not upload repository history, other projects, meeting content or unrelated
credentials. No app service or recurring task is installed.

Start the backend as the dedicated VM user with its verified Node binary and
explicit private config, keeping the SSH session attached. From the operator's
computer, the approved live browser check is:

```powershell
npm run test:live:remote -- --url "http://127.0.0.1:4193" --output "<existing-protected-evidence-directory>"
```

This performs two real rehearsals and consumes two of the three approved runs.
It refuses an unfinished earlier run. For any other environment, obtain its own
bounded approval first; this command is not standing permission.

For an already initialized environment, while its own approval is still open:

```powershell
.\scripts\Start-Dashboard.ps1 -Directory "<protected-local-environment-directory>" -StartVm
```

This starts only the positively identified retained VM when `-StartVm` is given,
then runs the backend and UI tunnel in the foreground with a finite lifetime.
It does not start a rehearsal, reset the allowance or extend approval.
Existing completed runs remain in its journal; check the displayed allowance
before starting another run. Deallocate the exact source VM again after use.
After approval expiry, stop and request a new bounded recovery/rehearsal scope;
changing dates or deleting the journal is not an approval-renewal mechanism.

## Runbook

1. **New rehearsal**, then **Seed synthetic data**. Each run gets a random
   database namespace and independent ownership nonce. The fixture has eight
   inventory products and 24 orders with ObjectId, Int32, Double, Int64 above
   JavaScript's safe-integer range, Decimal128, dates, binary UUIDs, timestamps,
   arrays, nulls and embedded documents. Re-seeding checks existing values and
   inserts only missing exact fixtures; it never overwrites foreign data.
2. **Assess workload**, then **Place a synthetic order**. The same driver code
   runs filtered reads, aggregation, and a transaction that reserves inventory
   and inserts an order. Database and driver versions are recorded. Assessment
   covers this workload only, not arbitrary customer applications.
3. **Fence source writes**. The dedicated one-member MongoDB primary must report
   its fsync lock. Its process/election identity is recorded and rechecked.
   Do not use this global lock against a shared/production source. A lock flag
   in SQLite, an expired lease, or an operator success checkbox is not a fence.
4. **Run native copy**. The app invokes verified MongoDB tools with protected
   config files, not URI-bearing command-line arguments. Only this run's
   `inventory` and `orders` are restored. No `--drop`, custom copy engine, or
   undocumented migration API is used. Process exit and archive hash are
   recorded; neither replaces independent database comparison.
5. **Inject +1 quantity mismatch**, then **Reconcile both databases**. Counts
   still match but BSON/known-answer checks fail and cutover stays blocked.
   **Repair exact injected mismatch** restores only that known preimage.
   Reconcile again; unrelated drift is never silently repaired.
6. **Cut over reads** after a complete fresh comparison. Cutover repeats the
   comparison immediately; it does not trust a previously green screen.
   Both full scans must be stable, the source fence unchanged, native tools
   stopped, and the single application's writers paused. No other writer or
   administrator may use either disposable database during this boundary.
7. **Return to source** is available only while no target-only application
   write was attempted and fresh complete equality still holds. The source
   lock is then explicitly released. After any target write attempt, even an
   uncertain one, return-to-source is permanently blocked for the run.
   Recovery requires a separately designed reconciliation/reverse-sync process.
8. **Clean this run's data** requires typing the exact run ID. It deletes only
   documents whose ID/nonce attribution matches the independently read marker,
   verifies zero remaining fixture documents, and releases only the owned
   source fence. Empty collections, indexes and ownership markers remain for
   attribution. It never drops databases, clusters, directories or groups.

## Migration-tool handoff

Microsoft's [current migration guidance][migration] deprecates
[MongoMigrationWebBasedUtility][legacy]. It remains useful historical prior art:
its UI orchestrates offline dump/restore, online driver copying/change streams,
and sync-only jobs, but it exposes no documented job-control API we can rely on.
No upstream source or connector is vendored here.

Native BSON `mongodump`/`mongorestore` is still a documented offline option and is
the implemented lane. For a future online lane, use the supported
[Azure DocumentDB Migration Extension][extension] as an honest operator-guided
handoff and independently establish copy/catch-up/writer-stop evidence.
That extension is not installed, automated, or declared successful by this app.

## Evidence, cancellation and recovery

The full allowlisted collections, not a sample, are compared against each other
**and** deterministic known answers. EJSON v2 non-relaxed canonicalization keeps
BSON type distinctions and array order; document object-key order is explicitly
excluded. Index key order, names, uniqueness, sparse/hidden and supported options
matter; physical version/namespace/background/build UUID do not. The observed
target-only Boolean `statsEnabled` is planner-statistics metadata, not data/index
semantics, and is explicitly excluded from equality; its type is still checked.
The observed `storageEngine.enableOrderedIndex: true` marker is normalized only
for ordinary ascending/descending keys; false, unknown engine options and other
index types are rejected. Ordered indexes are the engine's [documented default][ordered-indexes].
See the upstream [collection metadata definition][planner-stats]. Unknown index
options, TTL, other collection options, extra collections, foreign documents, unavailable
observations and stale evidence fail closed. Index counts alone are insufficient.
Deprecated/unsupported BSON values (including undefined), untyped unsafe integers,
documents above 16 KiB, collections above 2 MiB or 1,000 documents also stop this
small fixture lane; it never silently falls back to sampling.

The local SQLite WAL journal records intents before writes, versions, hashes,
outcomes, failures, pending mutations, native-tool PIDs and the irreversible
target-write boundary. It is single-host bookkeeping, not cloud truth.
The local lock refuses a second process. Following a crash, confirm the recorded
process stopped, then use `--recover-lock`; a reused/live PID is not displaced.
**Reconcile uncertain operation** checks complete database pre/postimages.
Partial copies require an explicitly confirmed **Reset attributed target data**
before another native copy. Unknown or unowned contents block deletion.

**Cancel current operation**, Ctrl+C, a `STOP` file in the exact private runtime,
approval expiry and operation deadlines stop new work. Native-tool cancellation
waits for actual process closure, escalating only that child to forced termination
after a two-second grace period. An interrupted application does not prove a
surviving child stopped: recovery checks its recorded PID and fails closed while
it exists. Cancellation is not a transaction rollback, data cleanup, cloud
deletion, or automatic source unlock. Preserve the journal and inspect/recover.
Never erase it to regain the three-run allowance or clear a failure.

The approved implementation has at most three runs, one active operation, a
60-minute run deadline, 30-second database/action deadlines, and 120 seconds per
native tool. Only transient read operations retry (twice, within the original
action deadline); no application-level automatic write/copy retry occurs.
Manual transaction commit may use the driver's documented commit behavior.
No new work runs outside the at-most-24-hour approved window.
After the testing deadline, only existing-intent recovery and exact-data cleanup
remain available, still within that original approval window and the 30-second
action bound. Neither operation renews the testing deadline or run allowance.
The UI invalidates previously displayed application rows when a mutation or
route change begins; read the application again for a current observation.

## Retention and constraints

The requesting owner approves exact infrastructure scope and owns retention and
cleanup. This supervised environment's selected policy is seven-day resource
retention, VM deallocation between rehearsals, and 30-day private evidence
retention. **No automatic deletion or recurring schedule is installed.**
VM deallocation does not stop disk/public-IP/DocumentDB charges. Confirm exact
resource IDs and matching environment tags before separately approved deletion;
the existing resource group may contain unrelated resources.

The app never resets credentials, changes Azure roles, or expands firewall
rules. Native DocumentDB secondary users have cluster-level roles, so this
cluster must remain dedicated to synthetic demo data. Administrators can bypass
the demo's controls: the app is not a production distributed fencing system.
Source HA/failover, arbitrary applications, load performance, online migration,
target-to-source replication and unattended cloud cleanup are not implemented.
Keep the host awake during a supervised rehearsal.

See the [validation record](docs/validation.md) for exactly what ran live versus
with test doubles, and [support and platform scope](SUPPORT.md) before use.
This sample is licensed under the [MIT license](LICENSE). Dependencies, MongoDB
Community and downloaded tools retain their own licenses; this repository's
license does not relicense them. No vendor source or binary is bundled here.

[migration]: https://learn.microsoft.com/en-us/azure/documentdb/migration-options
[legacy]: https://github.com/AzureCosmosDB/MongoMigrationWebBasedUtility
[extension]: https://learn.microsoft.com/en-us/azure/documentdb/how-to-migrate-vs-code-extension
[planner-stats]: https://github.com/documentdb/documentdb/blob/main/pg_documentdb/include/metadata/collection.h
[ordered-indexes]: https://github.com/documentdb/documentdb/blob/main/CHANGELOG.md
