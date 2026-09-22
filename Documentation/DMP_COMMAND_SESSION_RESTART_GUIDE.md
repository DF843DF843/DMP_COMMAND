# DMP COMMAND – Session Restart Guide

Purpose: this document lets a future AI session immediately restart DMP COMMAND work without re-discovering the environment.

## Start trigger

When the user writes "wir arbeiten an DMP_Command weiter" or similar, first read this file and then the current backlog/release/status documents before making technical claims or changes.

## Mandatory first action: restore local worktree

**Before reading, editing, packing, or deploying any source:** work from the local Git worktree
`C:\PowerAppWork\DMP_COMMAND_Solution`, never from the OneDrive-synchronised copy. Writing source or packages into the synced SharePoint/OneDrive library was previously blocked by a storage/synchronisation error. If the user starts DMP COMMAND work without mentioning this, explicitly remind them before any technical step.

The OneDrive `Documentation` folder is the documentation/distribution copy only. Keep documentation synchronised after the local worktree change is complete; do not make it the active development worktree.

## Mandatory first reads

1. `Documentation/DMP COMMAND_Mission_und_KI_Arbeitsregeln.md`
2. `Documentation/Backlog/DMP_COMMAND_Backlog.md`
3. `Documentation/DMP_COMMAND_Release_Notes.md`
4. `Documentation/DMP_COMMAND_Operations_Manual.md`
5. `Documentation/DMP Command Configuration.csv`
6. `Documentation/DMP Command Agent Status.csv`

If a change touches the Power App help/manual content, keep `DMP_COMMAND_Operations_Manual.md` and `PowerApp/DMP_COMMAND/Source/Src/scrOperationalBoard.pa.yaml` synchronized.

## Local locations

- Git working copy: `C:\PowerAppWork\DMP_COMMAND_Solution`
- Git remote: `https://github.com/DF843DF843/DMP_COMMAND.git`
- **Known issue (found 2026-09-22):** `C:\PowerAppWork\DMP_COMMAND_Solution\.git` exists as a
  directory but is completely empty (0 entries) — there is currently no working local Git
  history/remote link, even though all working files (`Documentation`, `PowerApp`,
  `PowerAutomate`, `bin`) are present and intact. This does not block file-based work or
  `pac` deployments, but no commits/history are possible until repaired (for example by
  re-cloning the remote into a sibling folder and reconciling any local-only changes). Verify
  with `git -C "C:\PowerAppWork\DMP_COMMAND_Solution" status` before assuming Git works.
- OneDrive/team-file root: `C:\Users\df843\OneDrive - Deutsche Börse AG\GO365_DMP Communication - Email Hotline\AI_Agent`
- OneDrive documentation copy: `C:\Users\df843\OneDrive - Deutsche Börse AG\GO365_DMP Communication - Email Hotline\AI_Agent\Documentation`
- Power App version file in synced SharePoint library: `C:\Users\df843\OneDrive - Deutsche Börse AG\GO365_DMP Communication - Email Hotline\AI_Agent\PowerApp_Storage\PowerApp_Version.txt`

Documentation under `Documentation\` exists in two copies (Git repo and OneDrive). Keep both copies synchronized for every documentation change.

## Power Platform environment

- Active environment normally used for DMP COMMAND development: `DBG Team Productivity (Dev)`
- Environment URL: `https://teamproductivity-dev.crm4.dynamics.com/`
- Environment ID: `53617fa1-2ee3-e3c9-99ba-c508b1a68246`
- Unique name: `unq50e812937714448f90f99950c3458`
- Check authentication with:

```powershell
pac auth list
pac env list
```

Do not assume access is missing. Test the exact access path first.

## SharePoint / OneDrive context

- SharePoint site used by the flows/lists: `https://deutscheboerse.sharepoint.com/teams/GO365_DMPCommunication`
- Synced document-library root is the OneDrive/team-file root listed above.
- SharePoint document-library files may be accessible locally through OneDrive sync; SharePoint lists are different and normally require Power Platform/connector/Graph-style access.
- Central lists:
  - `DMP Command Configuration`
  - `DMP Command Agent Status`
  - `DMP Command Internal Domains`
  - `DMP Command External Domains`
  - `DMP Command Counters`

## Source layout in Git

- Power Automate solution source: `C:\PowerAppWork\DMP_COMMAND_Solution\PowerAutomate\DMP_COMMAND_Solution\Source`
- Flow JSON files: `C:\PowerAppWork\DMP_COMMAND_Solution\PowerAutomate\DMP_COMMAND_Solution\Source\Workflows`
- Solution metadata: `C:\PowerAppWork\DMP_COMMAND_Solution\PowerAutomate\DMP_COMMAND_Solution\Source\Other\Solution.xml`
- Canvas App source: `C:\PowerAppWork\DMP_COMMAND_Solution\PowerApp\DMP_COMMAND\Source`
- Canvas screens: `C:\PowerAppWork\DMP_COMMAND_Solution\PowerApp\DMP_COMMAND\Source\Src`

## Deployment discipline

Before any pack, version bump, import, or publish:

1. Search the active backlog for the same agent/screen/deploy target.
2. Identify already diagnosed, low-risk backlog items that can be bundled safely.
3. Implement safe bundled items in the same deployment package unless the user explicitly asks for an immediate single hotfix or the issue is a production emergency.
4. Only then bump versions, pack, and deploy.
5. If a production emergency forces a single hotfix, state explicitly which backlog-bundling check was skipped/deferred.

After every deployment:

1. State exactly what was deployed and to which environment.
2. State version numbers before/after.
3. State validation performed.
4. State the required live test.
5. Remind the user to continue in a new session to reduce token/credit usage.

## Current important state as of 2026-09-22 07:11

### Active deployment

- Work only in `DBG Team Productivity (Dev)`. Production was not changed.
- `DMP_COMMAND_Solution` is deployed and published in Dev at version `7.11.47`.
- Agent 7 is present as `DMP Agent 7 (Streams & Milestone Management) [0.2.0]`.
- The `[0.2.0]` suffix is the workflow component display name, not the solution version.
- `7.11.46` was user-confirmed as saveable/activatable in the designer. `7.11.47` (the
  action/date-range contract rewrite below) was imported via `pac solution import` in this
  session but has **not yet** been opened/saved/activated by the user in the designer — do
  that check first in the next session before assuming it behaves like `7.11.46`.

### Agent 7 activation root cause and verified fix

The initial Agent 7 imports (`7.11.36` through `7.11.38`) could not be loaded or activated.
The designer first reported:

```text
The definition request is missing required field 'definition'.
```

It later reported:

```text
JsonSerializationException: Required property '$schema' not found in JSON. Path ''.
```

These messages were misleading. The packed and Dataverse-stored `clientdata` both contained
`properties.definition` and the correct workflow `$schema`. The official Power Platform
Solution Checker also returned zero findings.

The issue was isolated by deploying progressively larger designer-compatible definitions:

- `7.11.40`: minimal Power Apps V2 trigger + Compose + root Response; loaded successfully.
- `7.11.42`: added one SharePoint Get Items; loaded successfully.
- `7.11.43`: added the DMP Condition around the SharePoint read; loaded successfully.
- `7.11.44`: added rule loop, OccurrenceId, exact duplicate lookup, and nested existence
  Condition; loaded successfully.
- `7.11.45`: added Create Item; the designer then exposed the real validation error:
  `item/ScheduleSlot` supplied a string where the connector expected an object.
- `7.11.46`: changed the binding to `item/ScheduleSlot/Value`; the user confirmed save and
  activation succeed.

Do not revert this mapping:

```text
Task Occurrences ScheduleSlot (Choice):
item/ScheduleSlot/Value
```

The SharePoint dataset URL was also corrected everywhere from the invalid path without an
underscore to:

```text
https://deutscheboerse.sharepoint.com/teams/GO365_DMPCommunication-CoSLeader
```

Confirmed list IDs:

- Recurrence Rules: `728f1f87-2cc6-476f-be56-0f1feb05158f`
- Task Occurrences: `5c413852-2dab-4844-b886-a8cf8334fcf6`
- Connection reference: `dmpcmnd_sharedsharepointonline_626f1`

The Task Occurrences Create Item action intentionally does not send `item/TimeZone`, because
that parameter was not confirmed by the target list connector schema. Time zone remains read
from each recurrence rule to calculate `DueUtc`.

### Current Agent 7 implementation

Source:
`PowerAutomate/DMP_COMMAND_Solution/Source/Workflows/DMPAgent7StreamsMilestoneManagement-82E743CD-A2F6-4485-9E76-111D0D30544C.json`

Currently implemented as of `7.11.47` (JSON-valid, `runAfter`-graph-checked,
description-length-checked, variable-initialization-checked locally; **not yet
designer-opened/activated by the user**):

- Power Apps V2 trigger with seven inputs: `InitiatedBy`, `RequestedAction`, `OperatingMode`,
  `CaseId`, `FromDate`, `BusinessDate`, `ToDate`. `OccurrencesJson` was removed.
- `RequestedAction` contract: `CreateTaskOccurrences` requires `BusinessDate` (single-date);
  `GenerateMissingOccurrences` requires `FromDate`/`ToDate` (inclusive multi-day range, built
  via `ticks()`/`range()`/`addDays()` into an `OccurrenceDates` array). Any other
  `RequestedAction`, or missing/out-of-order dates, sets `ValidationError` and is returned to
  the caller without generating anything.
- DMP-only gate (unchanged: `OperatingMode` must equal `DMP`).
- Read active Daily DMP recurrence rules once, then iterate `OccurrenceDates` (outer
  `FOREACH_Dates`) x rules (inner `APPLY_Rules`, unchanged logic per date).
- Derive the canonical key:
  `CaseId-yyyyMMdd-TaskIdWithoutSpaces-UPPER(ScheduleSlot)` per date.
- Query Task Occurrences by exact `Title`/OccurrenceId before creating.
- Create only if no existing item is found; `CreatedCount`/`SkippedCount` variables are
  incremented reliably on each branch.
- Convert `OccurrenceDate + TimeOfDay` from the rule `TimeZone` to `DueUtc` (unchanged
  connector fields/binding from `7.11.46`, only the date source changed from
  `triggerBody()?['text_5']` to `items('FOREACH_Dates')`).
- Response returns `success`, `message`, `createdCount`, `skippedCount`, `validationError`.

Still NOT implemented in `7.11.47` (see backlog for the full current list):
- Rule-level `TimeOfDay`/`TimeZone` existence validation before `convertTimeZone` (a rule
  missing either field will still fail the whole run instead of a graceful per-rule error).
- A real per-item error counter (needs a Scope/try-catch pattern around `CREATE_Occurrence`).
- Authoritative active-case/mode source resolution (open user decision, see backlog).
- `Sequence` field decision, Power App screen live-binding, and the first Dev end-to-end test.

No end-to-end run that creates real occurrence rows has been performed. `7.11.46` was
user-confirmed open/save/activate in the designer; `7.11.47` has not been designer-confirmed
yet — this is the mandatory first check in the next session.

### C5 work still open

`7.11.47` implements the trigger/date-range contract (former items 1–3, partially) and
reliable created/skipped counters (former item 6, partially). Still open:

1. Rule-level `TimeOfDay`/`TimeZone` existence validation on each Recurrence Rule before
   `convertTimeZone`, with a graceful per-rule technical error instead of failing the run.
2. Resolve the authoritative active-case and operating-mode source. `CurrentOperationMode`
   (Global/Runtime config, values `PROD_NODMP/PROD_DMP/SIMU_NODMP/SIMU_DMP/
   PROD_PREDEFAULT/PROD_POSTDEFAULT/SIMU_PREDEFAULT/SIMU_POSTDEFAULT`) is confirmed as the
   already-existing, reused central mode source. No central "active CaseId" source was found
   anywhere in the app/config — needs a user decision.
3. Add a real per-item error counter (Scope/try-catch pattern around `CREATE_Occurrence`);
   `CreatedCount`/`SkippedCount` are already reliable, `ErrorCount` is not yet implemented.
4. Decide whether checklist order must be guaranteed. Dynamic membership is supported, but
   there is no persisted order; add a numeric `Sequence` field to Recurrence Rules if order
   matters.
5. Connect `scrTaskOccurrences.pa.yaml` to the live SharePoint list and Agent 7. It currently
   uses preview rows.
6. Run a non-destructive Dev end-to-end test (covers both `CreateTaskOccurrences` and
   `GenerateMissingOccurrences`):
   - first run creates the expected rows for the requested date(s);
   - identical second run creates zero duplicates;
   - Post-Default/non-DMP creates no rows;
   - historical rows remain unchanged.

### Recommended first actions in the next chat

1. Read this guide, the active backlog, release notes, and
   `Documentation/Streams_ListTemplates/README.md`.
2. Confirm PAC still targets `DBG Team Productivity (Dev)`.
3. **Ask the user to open, save, and activate Agent 7 `7.11.47` in the designer first** (not
   yet confirmed) and report any error before making further changes.
4. Re-read the current Agent 7 JSON; do not restore any earlier `7.11.36`–`7.11.46` package.
5. Continue with the still-open C5 items above using small designer-validated increments.
   Never introduce several unvalidated connector fields in one deployment.

### Other previously open state

- The older Agent 2 and System Health ring follow-ups remain in the active backlog unless
  separately verified and closed.
- The Power App Task Occurrences screen is prepared locally but is not live-data-bound.
