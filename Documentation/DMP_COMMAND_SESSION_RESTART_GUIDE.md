# DMP COMMAND – Session Restart Guide

Purpose: this document lets a future AI session immediately restart DMP COMMAND work without re-discovering the environment.

## Start trigger

When the user writes "wir arbeiten an DMP_Command weiter" or similar, first read this file and then the current backlog/release/status documents before making technical claims or changes.

## Mandatory first action: restore local worktree

**Before reading, editing, packing, or deploying any source:** work from the local Git worktree
`C:\PowerAppWork\DMP_COMMAND_Solution`, never from the OneDrive-synchronised copy. Writing source or packages into the synced SharePoint/OneDrive library was previously blocked by a storage/synchronisation error. If the user starts DMP COMMAND work without mentioning this, explicitly remind them before any technical step.

The OneDrive `Documentation` folder is the documentation/distribution copy only. Keep documentation synchronised after the local worktree change is complete; do not make it the active development worktree.

**Reconfirmed by the user on 2026-09-22 — keep reminding proactively:** The user explicitly
asked to keep being reminded that this OneDrive/SharePoint-synced location previously caused
a real storage/synchronisation blocking error. A live re-test on 2026-09-22 showed that basic
writes and even `git init` succeed here right now, but the user still decided to **keep
`C:\PowerAppWork\DMP_COMMAND_Solution` as the only Git/source working copy** rather than move
Git operations into this OneDrive folder, specifically because of that earlier real incident
and the generally known risk of OneDrive's sync engine colliding with an active Git
repository's internal file writes. Do not propose moving the Git working copy into OneDrive
again without re-raising this history first.

## ⚡ Latest session recap (2026-09-23, v1.22.30: 3 UX findings from the v1.22.29 test fixed - System Health page, Task Occurrences readability, Configuration tab completeness)

- **User tested `v1.22.29` in Studio/the running app (2 screenshots) and reported 3 findings** - importantly, the B3 popup's write to `DMP Command Default Case Context` **worked** (first live confirmation of a Person/Choice-column `Patch()` in this app, previously flagged as untested in v1.22.29).
- **Finding 1 - System Health legend not fully visible:** the ring's popup (`conHeartbeatLegendPopup`, fixed 312x300px with internal scroll) could never show all 16 monitored items readably at once. **Fix:** extracted to a new dedicated page `scrSystemHealthDetails.pa.yaml` (new sidebar entry "System Health" + the ring click now navigates there instead of toggling the old popup), grouped into Core Status / Data Sources & Files / Agents (1-7), with every status formula copied verbatim from the old popup (not re-derived) to avoid introducing a logic regression. Old popup and its `varShowHeartbeatLegend` variable fully removed (verified via a container-count check before/after the splice, per the established line-range-splice safety rule).
- **Finding 2 - Task Occurrences "Current view" unreadable, function unclear:** root cause found - `lblTaskOccurrencesContract`'s box was only 28px tall for a ~300-character description, so most of the wrapped text rendered outside the box and visually overlapped the row above it (exactly matching the user's screenshot). Fixed box height/`Wrap`/`VerticalAlign`, and found the same "hardcoded near-black colour, invisible in dark mode" bug (previously fixed on the System Health ring on 2026-09-22) recurring on `lblTaskOccurrencesModeValue` and the gallery's `lblOccurrenceStatus` - both made theme-aware. Also corrected the view's label text (was "Open and overdue occurrences", which does not match the actual `Filter(...,ApprovalState.Value<>"Rejected")` logic that also shows Done rows) to "All non-rejected occurrences, soonest due first" - a text-only correction, the underlying Filter/Sort formula was deliberately left unchanged.
- **Finding 3 - Configuration (Lists) tab incomplete/unstructured:** only 3 of the 17 live-connected SharePoint lists had a tile (Configuration, Agent Status, Internal Domains) - 14 were missing entirely, confirmed against the actually-connected data sources by extracting and inspecting the local `.msapr`'s `References/DataSources.json` directly (ground truth, not guessed). Added tiles for all 14 missing lists (Counters, External Domains, and all 12 Streams lists), reorganized into 6 thematic sections in a new scrollable body container. **Site/internal-name finding (from `DataSources.json`, cross-checked against the `NextSteps_*` legacy duplicate data source entries still present from before each list's rename):** `Checklist CoS Leader`, `Default Case Context`, `Status Change Approvals`, `Email Templates`, `Recipient Groups` and `Role Assignments` are renamed lists whose SharePoint-internal URL segment is still their old `NextSteps_*` working name, not their current display title - their "View"/"New Entry" `Launch()` URLs use the confirmed internal name accordingly (a display-name-based URL would 404 for these 6).
- **Deployment hygiene:** app-wide control-name uniqueness re-verified (850 names, 0 duplicates), full pack/unpack round-trip diff (0 diff on every changed/new file), and a dedicated regex scan for literal `": "` inside single-line `Text: ="..."` formulas (per the 2026-08-28 rule) found 4 new hits in the new section-header labels - reworded to use `" - "` before packing, since this exact pattern previously crashed Studio's YAML parser despite a clean round-trip diff.
- **Versioning:** Power App bumped to `v1.22.30` (patch, per the "at least bump patch on every change" rule); `PowerApp_Version.txt` updated in the same step (the exact rule that was missed for v1.22.25-28 last session). Rule 9b backup rotated: `v1.22.29` is now the confirmed-loading backup (`DMP_COMMAND_v1.22.29.msapp`, replacing `v1.22.28`), since the user's successful live write during testing counts as confirmation it loads/works. No Power Automate/Solution changes this session - Solution stays at `7.34.67`.
- **Still open / not yet done this session:** the user has not yet loaded `v1.22.30` in Studio - no live confirmation yet for any of the 3 fixes above, or for the new System Health/Configuration pages in general (first-ever use of a nested-AutoLayout nested-in-AutoLayout section-card pattern for Configuration, and first-ever full-page version of the health legend - both packed and round-trip-clean, but not Studio-validated).

## ⚡ Prior session recap (2026-09-23, v1.22.28 confirmed + B3/Task Occurrences live-wired as v1.22.29 + Solution 7.34.67 imported by AI)

- **AI capability corrections this session (read before assuming "no access" again):** (1) The
  AI has an active, authenticated `pac` session against `DBG Team Productivity (Dev)` - `pac
  solution import --publish-changes` is run by the AI directly, NOT the user (user had to
  correct this). (2) `PowerApp_Version.txt` (read by Agent 4, drives the Cockpit Status Panel's
  displayed version) is directly writable by the AI and must be kept in sync on EVERY Power App
  version bump - this was missed for v1.22.25-28 this session (regression against an existing
  rule from 2026-08-26) and caused a real, user-noticed Status-Panel-vs-Audit-Trail version
  mismatch; fixed (file now `v1.22.29`) and the rule reinforced in the Mission doc.
- **v1.22.28 confirmed by the user:** loaded, new data sources added, saved and published in
  Studio. Rule 9b backup rotated to `DMP_COMMAND_v1.22.28.msapp` (replacing `v1.22.27`).
- **Solution `7.34.67` imported and published by the AI** (not the user). **Update:** the user
  confirmed all 6 changed flows (Agent 1,2,3,4,5,6) stayed active - this time no manual
  open+save in the Power Automate designer was needed (unlike some earlier sessions' Agent 7
  incidents). Do not assume this is guaranteed for every future import - keep checking/asking.
- **B3 + Task Occurrences (C6/C7) now live-wired, packed as `v1.22.29`:** the user added `DMP
  Command Default Case Context` AND `DMP Command Checklist Task Occurrences` (plus several
  other Streams lists) as real data sources in Studio. The AI downloaded the just-published live
  app (`pac canvas download`), refreshed the repo's `.msapr` from it (checklist rule A.1), then
  rewired both screens from their local preview collections to real `Patch()`/`Filter()` calls.
  **Two schema surprises found by inspecting the live `.msapr`'s `DataSources.json` directly**
  (not guessed): neither list has a dedicated business-key column - `DMP Command Default Case
  Context` has no `CaseId` column (the Case ID goes in `Title`), and `DMP Command Checklist Task
  Occurrences` has no `OccurrenceId` column either (same pattern, `Title` holds it). Person
  columns (`SetByUser`, `ProposedBy`, `ApprovedBy`) and Choice columns (`ModeAtCreation`,
  `Status`, `ScheduleSlot`, `ApprovalState`) are patched as SharePoint-connector records
  (`{Claims:...,DisplayName:...,Email:...}` / `{Value:...}`) - confirmed against the live
  DataSources.json schema (`type: object` for both). **This is the first live Patch() against
  SharePoint Person/Choice columns anywhere in this app - not yet Studio-tested,** flag to the
  user to specifically test the B3 popup confirm and (if any row exists) Task Occurrences
  Propose/Approve/Reject after loading `v1.22.29`. The Task Occurrences gallery will likely show
  0 rows until Agent 7 actually creates real occurrences (expected, not a bug).
  `colDefaultCaseContextPending`/`colTaskOccurrencesPreview` both fully removed.

## ⚡ Prior session recap (2026-09-23, v1.22.27 confirmed + B1 closed + B5 systemic fix, v1.22.28/Solution 7.34.67 not yet deployed)

- **Power App:** the user loaded `PowerApp/DMP_COMMAND/DMP_COMMAND.msapp` (`v1.22.27`) fresh in
  Studio and confirmed "funktioniert" - loads/works error-free. Rule 9b applied: local backup
  copy rotated from `DMP_COMMAND_v1.22.26.msapp` to `DMP_COMMAND_v1.22.27.msapp` (rebuilt via
  `git worktree add` on commit `652606d`, `pac canvas pack --sources Source --msapp
  DMP_COMMAND_v1.22.27.msapp --overwrite`, moved into place, old `v1.22.26` backup deleted,
  worktree removed). Exactly one backup file exists again, as required.
- **B1 (Configuration columns) - CLOSED 2026-09-23:** the user confirmed the 4 new columns
  (`Value - PROD (Pre-Default)`, `Value - PROD (Post-Default)`, `Value - SIMU (Pre-Default)`,
  `Value - SIMU (Post-Default)`) are live in the real SharePoint list `DMP Command
  Configuration`, and asked to verify CSV completeness programmatically rather than by asking
  again. A full check of all 8 value columns across all 106 rows in `DMP Command
  Configuration.csv` found only 5 rows with any gap (`AuditTrailOpenUrl`, `CounterOpenUrl`,
  `EmergencyReportOpenUrl`, `ExternalDomainsOpenUrl`, `InternalDomainsOpenUrl`) - and those were
  already empty in ALL 8 columns plus `CurrentValue` even before the B1 column extension (an
  unrelated, pre-existing gap, now logged as its own low-priority Backlog item under
  Priorität 3, not a B1 defect). B1 is therefore marked done/confirmed in the Backlog.
- **v1.22.28 prepared (Power App, not yet deployed):** the temporary "TIMESTAMP DEBUG"
  Admin Functions panel (`conFuncTimestampDebug`, added in v1.22.21) was removed now that its
  bug is confirmed fixed. Pack→Unpack round-trip: 0 diff.
- **B5 - real systemic bug found and fixed in 6 of 7 Power Automate flows (Solution 7.34.67,
  not yet imported):** while resuming B5 (Agent 2 EffectiveMode mapping), discovered that
  Agent 1, 2, 3 (Emergency Report), 4 (Status Check), 5 (Operational State Management) and 6
  (Admin Functions) - every flow except Agent 7 - share one config-value-resolution mechanism
  (`Select_ConfigEntries`) that only ever recognised 4 modes (`PROD_NODMP`/`PROD_DMP`/
  `SIMU_NODMP`/else-`SIMU_DMP`). Since B2 introduced `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/
  `SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT`, every mode-dependent Configuration value (mail texts,
  subject prefixes, folder names, etc.) silently fell through to the `SIMU_DMP` column value
  whenever the app was actually in Pre-Default or Post-Default - a real, already-live bug since
  B2 shipped, not yet noticed because nobody had tested a mail-generating agent while the app
  was in Pre-/Post-Default. The CSV showed the user had already deliberately prepared distinct
  SIMU Pre-/Post-Default mail texts (fire-drill wording), so the fix could not just alias
  Pre-/Post-Default to NODMP - the real new B1 columns had to be read correctly. **Internal
  SharePoint field names were NOT derivable from the display name** (unlike the original 4
  columns, whose internal names merely strip spaces/hex-encode punctuation, the new 4 got
  auto-truncated by SharePoint to 26-27 characters with a collision-disambiguation suffix on
  2 of them): confirmed via a live `GET_DMP_Command_Configuration` run's raw JSON output that
  the user exported and pasted in -
  `Value_x0020__x002d__x0020_PROD_x` = PROD Pre-Default,
  `Value_x0020__x002d__x0020_PROD_x0` = PROD Post-Default,
  `Value_x0020__x002d__x0020_SIMU_x` = SIMU Pre-Default,
  `Value_x0020__x002d__x0020_SIMU_x0` = SIMU Post-Default (cross-validated against 2 different
  config rows). Fixed identically in all 6 flows (Agent 2's variant additionally wraps each
  branch in its own `coalesce(...,'')` and uses variable name `CurrentOperationMode` instead of
  `OperationMode` - handled separately). All 6 `.json` files + their `.json.data.xml` version
  labels bumped (patch+1 each); `Solution.xml` recomputed to **`7.34.67`**. **Not yet imported
  into Dev - this whole bundle (App + Solution) is still pending the next deploy.**
- **Agent 2 - trigger concurrency changed on explicit user request:** `runtimeConfiguration.
  concurrency.maximumWaitingRuns` set to `100` (was implicit default `10`); degree of
  parallelism (`runs`) deliberately kept at `1` (sequential) - Agent 2 increments a shared
  SharePoint counter per mail with a read-then-write pattern with no locking, so raising `runs`
  above 1 would risk duplicate counter/reference values under concurrent execution. Flagged to
  the user, not changed without being asked.
- **Reminder established this session:** the user asked to be addressed in German going
  forward for this project's conversation (documentation itself stays in its existing
  language mix, English recap prose / German backlog prose, per rule G - not rewritten
  ad hoc).

## ⚡ Prior session recap (2026-09-22, continuation session - ring bug hunt, baseline logic, msapp backup rule)

Continuation of the same-day evening session below, driven by the user loading each packed
version in Studio and reporting exact behavior. Current true state:

- **Power App:** local source is at `v1.22.27`, packed as `PowerApp/DMP_COMMAND/DMP_COMMAND.msapp`.
  `v1.22.25` (second PA1001 YAML bug in `scrTaskOccurrences.pa.yaml`) confirmed loading fine.
  `v1.22.26` (System Health ring showing a wrong percentage instead of "N/A", and a details
  popup missing most categories/agents) confirmed working by the user, with two follow-up
  issues reported and fixed in `v1.22.27`: Critical/Warning ring segments and legend rows must
  respect the existing Critical/Warning counter-baseline "confirm" mechanism (a confirmed event
  turns green even if the raw Audit Trail still contains it), and the legend popup needed an
  explicit scrollbar once expanded to 9 categories + 7 agents. `v1.22.27` is not yet tested by
  the user.
- **NEW rule established 2026-09-22, now applied - local backup copy of the last
  confirmed-error-free-loading Power App version:** see
  `DMP COMMAND_Mission_und_KI_Arbeitsregeln.md` section C, rule 9b, for the full rule text.
  Summary: `DMP_COMMAND.msapp` (no suffix) is always the newest, not-yet-confirmed build;
  `DMP_COMMAND_v<version>.msapp` is always exactly one file, the most recently
  user-confirmed-to-load-without-error version, replaced only once a newer version is also
  confirmed. Applied for the first time this session: rebuilt `DMP_COMMAND_v1.22.26.msapp`
  from git commit `4cfc557` (via `git worktree add` on that commit, `pac canvas pack` there,
  move the file, remove the worktree) since `v1.22.26` was the last confirmed-loading version;
  deleted the old, no-longer-relevant `DMP_COMMAND_v1.22.18.msapp`. Only ever one backup file
  should exist at a time going forward.
- **Survey on SharePoint Lists.docx** (OneDrive-only, not in Git) is now the canonical live
  SharePoint schema reference - see the "Canonical live SharePoint schema reference" note
  further below in this file for the full cross-check and its one important caveat: the user
  confirmed real Teams/SharePoint display inconsistencies can make an existing list simply not
  appear in that export (happened for `DMP Command Checklist CoS Leader`, confirmed live via a
  user screenshot despite being absent from the survey) - never conclude a list doesn't exist
  from that document alone.
- **B1 (Configuration columns):** the user recreated `DMP Command Configuration.csv` with all
  8 mode-value columns filled in for every row (both doc copies refreshed). **Still to confirm
  with the user:** whether the 4 new columns/values are also live in the real SharePoint list
  yet, or only prepared in the CSV so far - the Backlog B1 entry has the open question.
- **Agent 2 `EmailsProcessed_DMP`/`EmailsProcessed_NoDMP` naming:** investigated in detail and
  discussed with the user; genuinely confusingly named (the "No DMP" path, which is actually
  the normal/default case rather than a fallback, increments the column called `..._DMP`, and
  vice versa) but deliberately NOT fixed - the user decided the SharePoint-column-rename +
  flow-reference + reimport risk/cost isn't worth it for a cosmetic naming issue. Documented in
  full in the Backlog (Priorität 3 section) for future reference only.

## ⚡ Session recap (2026-09-22, evening session)

Continuation of the same-day afternoon session. This session was driven almost entirely by
the user actually loading the packed app in Studio and reporting exact error text/behavior
after each iteration - confirmed far more efficient than guessing. Current true state:

- **Power App:** local source is at `v1.22.24`, packed as `PowerApp/DMP_COMMAND/DMP_COMMAND.msapp`
  (filename intentionally has **no version suffix** - the user asked to stop accumulating
  multiple stale versioned `.msapp` files; old ones were deleted). Always re-`pac canvas pack`
  fresh into this exact same filename for the next delivery. **Superseded 2026-09-22 (see the
  newer recap above): exactly one versioned backup copy of the last confirmed-loading version
  is now deliberately kept alongside it (rule 9b) - this is not a reversion to "multiple stale
  files", just one single, always-current backup.**
- **Power Automate solution:** unchanged this session (still `7.34.48`, no flow edits) - only
  the Power App moved. No new Solution reimport was needed or performed.
- **Real bugs found and fixed this session (all via live user testing, not guesswork):**
  1. System Health ring `vGreenFixedSegments`: `Color` as a record field name collides with
     Power Fx's built-in `Color` enum once compared with `=` ("Enum, Text incompatible");
     renamed to `SegColor` everywhere in the ring formula.
  2. Same ring formula, second bug: `vGreenFixedSegments` referenced sibling `vFixedSegments`
     inside the same record literal - Power Fx records cannot self-reference; split into its
     own nested `With()` level.
  3. A YAML indentation bug (PA1001) that the fix for bug #2 itself introduced (the added
     closing paren matched the following property's indentation, ending the block scalar one
     line early) - **`pac`'s own pack/unpack round-trip diff=0 did NOT catch this**; only
     Studio's own load caught it. Treat pac round-trip diff=0 as necessary but not sufficient
     proof of Studio-validity for any multi-line formula edit.
  4. Audit Trail year-3926 timestamp bug: **root cause found and user-confirmed fixed.**
     `Date(1899,12,30)` is outside Power Apps' supported date range (minimum 1900-01-01) and
     silently produced garbage instead of erroring; this broken epoch was already present,
     unfixed, since a September 3 commit that only reordered branch preference around it.
     Replaced with `DateAdd(Date(1900,1,1),(Value(rawTs,"en-US")-2)*86400,TimeUnit.Seconds)`
     everywhere (20 places in `scrAuditTrail.pa.yaml` + the temporary debug panel).
  5. Release Notes screen showing stale v1.22.13 / missing Agent 7: **root cause found**, not
     a stale-Studio-session issue as first assumed. Both `varSelectedAppRelease` (App tab) and
     `varSelectedAgentRelease` (Agent tab) were never initialized anywhere, and their
     detail-pane `Switch` statements had no case for anything past v1.22.13 / Agent 6
     respectively. Fixed defaults + added the current version/Agent 7 as new cases - the
     historical gap (App tab Switch missing v1.22.14-v1.22.20 as selectable entries) was
     deliberately NOT backfilled, only flagged in the backlog, since only "show newest by
     default" was needed to resolve the user's symptom.
  6. B3 popup: purple title text was hard to read in dark mode - now a solid colored banner
     with always-white text (same convention as the v1.22.17 button-contrast fix). Date/time
     input was asking for raw UTC - now asks for local time, converts via `TimeZoneOffset`
     only when staged for storage. A clipped field label was shortened.
  - **Ring status at end of session: NOT yet re-confirmed by the user** - the last reported
    error (bug #2 above) is fixed in `v1.22.24`, not yet re-tested live.
- **B1 (Configuration columns):** still fully open - blocked on the user manually adding 4
  columns to the live `DMP Command Configuration` SharePoint list. Values for all rows are
  ready in `DMP Command Configuration.csv` (both copies). Nothing else can proceed on B1
  without this manual step.
- **B2 (5-mode advance button):** user-confirmed working live this session (button correctly
  triggers the B3 popup on the Pre-Default -> DMP transition; "Pop-up ok").
- **B3 (Default Case Context popup) - built and refined this session:**
  `conDefaultCaseContextPopup` in `scrHome.pa.yaml` gates every Pre-Default -> DMP transition.
  **Important correction to the prior session's assumption:** the SharePoint list
  `DMP Command Default Case Context` (and in fact ALL Streams lists) already exist - the user
  had already created them; only `Recurrence Rules` and `Task Occurrences` actually keep
  "Checklist" in their real name, the rest (Default Case Context, Status Change Approvals,
  Email Templates, Email Placeholders, Recipient Groups, Role Assignments) do not. Confirmed
  site `GO365_DMPCommunication-CoSLeader`, list GUID `8fa1f858-3f89-4600-b3dc-86f9dfcaf4b8`,
  columns match the prepared template exactly (internal field name for the display column
  "TerminationDateTime" is `TerminationDate`). **Not yet wired as a live Power App data
  source** - needs the same one-time manual "Add data" step in Studio as any new connection
  in this project (like Agent 7's flow connection). Until then, the popup stages captured
  values in `colDefaultCaseContextPending` (exact same column shape as the real list) - one
  clean `Patch(...)` swap once connected.
- **C6/C7 (Task Occurrences four-eyes workflow) - first demonstration built this session:**
  `scrTaskOccurrences.pa.yaml`'s preview Gallery now runs on a real, patchable local
  collection `colTaskOccurrencesPreview` (exact schema of the live
  `DMP Command Checklist Task Occurrences` list) instead of a static `Table()` literal. Each
  row has a working Propose/Approve/Reject flow; Approve/Reject only shown to a *different*
  user than the proposer (four-eyes technically enforced). Also not yet connected to the real
  list (same manual "Add data" step needed). **Open design question for the user:** the real
  list has no `PreviousStatus` field, so Reject only clears the pending approval state - it
  does not revert `Status`. Decide before going live whether to add such a field.
- **Proven risk-mitigation pattern for never-before-used control types:** when
  `Classic/TextInput` was needed (never used in this app before, same risk class as the
  earlier Slider decision), a first guessed declaration was caught locally (missing from the
  packaged template registry) before being sent to the user; the user then placed one blank
  instance in Studio and pasted back the real declaration (`Classic/TextInput@2.3.2`), which
  was then used everywhere. Repeat this exact pattern for any other new control type (e.g. a
  real `DatePicker` if the user still wants one instead of the current validated free-text
  local-time entry - they asked for this "ideally", not yet done).
- **Bundling rule hardened** in `DMP COMMAND_Mission_und_KI_Arbeitsregeln.md` section C (new
  rule 9a, 2026-09-22): no more isolated single-fix deploys: always search open/known items
  for the same file/screen/agent and bundle before packing, unless the user explicitly wants
  an immediate single hotfix or it's a production emergency.

### Still open / not confirmed by the user by end of session

1. Whether `v1.22.24` actually fixes the System Health ring for real - last known error (bug
   #2 above) is fixed; not yet re-tested live by the user.
2. B1 (SharePoint Configuration columns) - the user still needs to add them manually.
3. B3 and Task Occurrences - both need their respective SharePoint list connected as a live
   Power App data source in Studio (one manual step each) before they can move from local
   simulation to real persistence.
4. The `PreviousStatus` field decision for Task Occurrences' Reject behavior - open question.
5. Whether every Power-App-only version bump should trigger its own dedicated Solution
   reimport, or may be batched into the next Solution import that has an actual flow-content
   reason to run - **carried over from the prior session, still not answered by the user.**
   Not relevant this session since no flow changes were made, but will matter again as soon as
   an Agent flow and a Power App change need to ship together.
6. Whether the user still wants a native `DatePicker` for the B3 popup's Termination
   Date+Time field (asked "ideally") - current validated free-text local-time entry works but
   is not a literal picker control.

**Recommended immediate next step for the new session:** ask the user to load
`PowerApp/DMP_COMMAND/DMP_COMMAND.msapp` (the exact current file, no version suffix) fresh in
Studio (File → Open → Browse this computer, not an already-open tab), then report either
"works" or the exact Studio error/Formeln-panel text for anything still broken - this has been
the fastest working method all session. Then resolve item 2 (B1) with the user, and/or decide
whether to keep building C6/C7 further (still local-only) or pause to have the user do the 2
pending manual "Add data source" steps (Default Case Context + Task Occurrences) so real
persistence can be wired next.

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
- **Git repaired on 2026-09-22 (previously broken, now fixed):** The old `.git` directory was
  found completely empty (no history/remote link) even though all working files were intact.
  Root cause unknown/not investigated further. Fix applied: the broken `.git` was deleted, a
  full safety backup of the working files was made
  (`C:\PowerAppWork\DMP_COMMAND_Solution_local_backup_20260922` and
  `..._pre_reclone_staging`, safe to delete once this recovery is confirmed stable over a few
  sessions), then a fresh `git clone https://github.com/DF843DF843/DMP_COMMAND.git` was done
  directly into the canonical path. The remote's last commit (`f823e79`, dated before
  2026-09-04) was behind essentially all work done since Git broke. All local-only/newer
  files (Agent 7 full history through `7.11.47`, Agent 2/4/6 workflow updates, several
  PowerApp screens, all Documentation including this guide, the Streams concept +
  `Streams_ListTemplates`) were diffed file-by-file against the fresh clone and merged
  forward, then committed as `694d434`. `git status`/`git log` are confirmed clean/working.
  Verify with `git -C "C:\PowerAppWork\DMP_COMMAND_Solution" status` that this is still true;
  if `.git` is ever found broken/empty again, repeat this same diff-and-merge-forward
  recovery procedure — never blindly overwrite the working files from a fresh clone without
  first diffing, since local files are routinely ahead of the last pushed commit.
- The recovery commits `694d434` and `dec7f0a` were **pushed to `origin/main` on 2026-09-22**
  (`f823e79..dec7f0a main -> main`); `git status` confirms the branch is clean and in sync.
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

## User's manual/UI workflow (clarified 2026-09-22 — keep reminding, do not re-assume)

The user explicitly clarified their actual manual/UI role, correcting an earlier assumption in
this guide:

- The user only loads, opens, and saves the **Power App (Canvas App)** in Power Apps Studio.
  As of 2026-09-22 the live Power App's last-modified timestamp is still **2026-09-04 10:08**
  — it has not been reloaded/resaved since, so any local `PowerApp/DMP_COMMAND/Source/Src`
  changes made after that date are **not live** yet (ties into the still-pending local
  PowerApp sync/diff task above).
- The user only activates individual **Power Automate flows** (agents) in the designer, and
  only when asked/necessary — this is the "open/save/activate" check this guide has been
  asking for regarding Agent 7.
- The user does **not** independently manage the **Solution version** (`7.11.x`) — do not
  expect the user to bump, pack, or import solution versions on their own initiative. Always
  give an explicit, concrete instruction for exactly what to do (which flow to open, which
  button to click) rather than a general status question; the AI is responsible for deciding
  when a version bump/pack/import is due and must ask/instruct explicitly at that point.

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

### Canonical live SharePoint schema reference (added 2026-09-22)

`Documentation/Survey on SharePoint Lists.docx` (OneDrive copy only, not in Git - it is a live
export, not authored content) is a per-list column/type/required survey of the **actual, current**
SharePoint lists and **must be treated as ground truth over the CSV templates in
`Streams_ListTemplates/` and over this guide's older claims** whenever they conflict. **However,
the user has confirmed (2026-09-22) that there are known, serious Teams/SharePoint display
inconsistencies that can make an existing list simply not show up in this kind of survey/export -
treat any "list not found" conclusion drawn from this document alone as a hypothesis to verify
directly with the user/in SharePoint, never as confirmed absence.** Re-check the survey before
wiring any new list as a Power App/flow data source, but do not conclude a list doesn't exist from
it alone. Cross-checked against the templates and source on 2026-09-22; findings:

- 11 of the 12 `Streams_ListTemplates` CSVs have a matching live list with matching columns via the
  survey directly.
- **`DMP Command Checklist CoS Leader` (the CoS Leader working checklist) - confirmed live and
  correct by the user 2026-09-22 despite being absent from the survey document** (exactly the
  Teams/SharePoint display-inconsistency case above). Site `GO365_DMPCommunication-CoSLeader`,
  list GUID `E3E95266-BB35-4E32-A60D-06E8E9E379A1`, internal/URL list name still
  `NextSteps_CosLeaderChecklist` (pre-rename technical name, only the display title was ever
  renamed - relevant if a connector ever needs the internal name/GUID instead of the display
  title). Columns confirmed 1:1 against `Streams_ListTemplates/README.md` item 2: `Title`,
  `Phase` (Choice), `SeqNo` (Number), `TaskDescription` (Multiple lines of text), `EmailTemplateId`
  (Single line of text), `Status` (Choice), `LastChangedBy`/`ConfirmedBy` (Person or Group),
  `ConfirmedUtc` (Date and Time), plus standard `Modified`/`Created`/`Created By`/`Modified By`.
  All 12/12 Streams lists are therefore confirmed to exist.
- `DMP Command Status Change Approvals` has a live `OccurrenceId` (single line of text) column not in
  the template CSV - update the template before next use; likely meant to reference `Task Occurrences`
  rows directly instead of via `ListName`+`ItemId`.
- `DMP Command Checklist Recurrence Rules.IsActive` is a live **Choice** column, not `Ja/Nein` as the
  concept implied - any future formula must compare against its choice values, not a boolean.
- `DMP Command Checklist Task Occurrences.ProposedBy`/`ApprovedBy` are live **Person or Group** columns.
  The current local-only `colTaskOccurrencesPreview` simulation stores `User().FullName` as plain text -
  this will need a proper person-claims value (not a plain string) once patched against the real list.
- `DMP Command Configuration` confirms only the original 4 `Value - PROD/SIMU (NODMP/DMP)` columns exist
  live - the 4 new `Value - PROD/SIMU (Pre-Default/Post-Default)` columns from B1 are still genuinely not
  created yet (no contradiction with the backlog, just confirmation).
- Unrelated bonus finding while cross-checking `DMP Command Agent Status`: Agent 2's flow
  (`DMPAgent2E-MailInboxTreatmentVS-...json`) writes `item/EmailsProcessed_DMP` when
  `Detected Workflow Path = 'No DMP'` and `item/EmailsProcessed_NoDMP` when
  `Detected Workflow Path = 'DMP internal Sender'` - these two look swapped based on their field
  names; not yet fixed, flagged for user confirmation before touching Agent 2 flow logic.

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
5. Remind the user to continue in a new session to reduce token/credit usage, AND output a ready-to-paste handover prompt (see the Mission/Working Rules doc, "NEUE REGEL" 2026-09-23) — this applies at the end of every session with DMP COMMAND changes, not only after a deployment.

## Current important state as of 2026-09-22 07:52

### Active deployment

- Work only in `DBG Team Productivity (Dev)`. Production was not changed.
- `DMP_COMMAND_Solution` is deployed and published in Dev at version `7.34.48`.
- Agent 7 is present as `DMP Agent 7 (Streams & Milestone Management) [0.3.1]`.
- The `[0.3.1]` suffix is Agent 7's own component version label, not the solution version —
  **but the solution version is not a free build counter either.** Per the checksum rule
  below, it must always equal the sum of all 8 component versions' segments.
- `7.11.46` was user-confirmed as saveable/activatable in the designer. `7.11.47` (the
  action/date-range contract rewrite below) was user-confirmed on 2026-09-22 to open
  error-free and to be activated ("Ein"); a separate Save-only test was not possible because
  the designer only enables Save after a real change.
- The hardening (rule TimeOfDay/TimeZone validation, per-item error counter, CaseId
  placeholder validation) was packed and imported via `pac solution import
  --publish-changes` in this session (2026-09-22) and published successfully as solution
  `7.11.48` at the time; Power Automate reported "The original workflow definition has been
  deactivated and replaced" (expected). **The user has not yet reopened, saved, or
  reactivated Agent 7 for this change** — do that check first in the next session. Unlike
  `7.11.47`, this version has real content changes, so Save will not be greyed out and is a
  genuine test.
- **Versioning correction (2026-09-22, same session):** the solution version `7.11.48` was
  itself found to be wrong — it was a free-running build counter, not the required checksum
  (see the new rule in `DMP COMMAND_Mission_und_KI_Arbeitsregeln.md` section I). Agent 7's
  own component label had also been left stale at `[0.2.0]` since `7.11.36` despite many real
  iterations. Corrected: Agent 7 → `[0.3.1]` (0.3.0 retroactively = the `7.11.47` contract
  rewrite, 0.3.1 = the `7.11.48` hardening), solution version recomputed and re-imported as
  `7.34.46` (no functional flow change from `7.11.48`, only the version labels).
- **Second checksum update, same session (B2 Power App change):** the Power App itself was
  changed (see "App Changes" below) and its own version bumped `v1.22.13` → `v1.22.15`
  (skipping the already-used, buggy `v1.22.14`). Per the checksum rule the Power App counts
  as one of the 8 components, so the solution version was recomputed again and re-imported
  as **`7.34.48`** (Patch 46 → 48, +2 matching the Power App's patch delta) — again no
  Power-Automate flow content change, purely the checksum reflecting the Power App bump.
  `pac solution import --publish-changes` succeeded; Power Automate again reported "The
  original workflow definition has been deactivated and replaced" for Agent 7, so **the user
  needs to reopen/save/reactivate Agent 7 once more** before assuming it is live-current.
  Open question not yet confirmed with the user: whether every future Power-App-only version
  bump should trigger its own dedicated Solution re-import like this, or whether such
  Power-App-only checksum updates may be batched into the next Solution import that has an
  actual flow-content reason to run. Ask before assuming either way next time this comes up.

**Solution-version checksum table (current, 2026-09-23):**

| Component | Version | Major | Minor | Patch |
|---|---|---|---|---|
| Agent 1 | 1.0.9 | 1 | 0 | 9 |
| Agent 2 | 1.0.10 | 1 | 0 | 10 |
| Agent 3 | 1.1.5 | 1 | 1 | 5 |
| Agent 4 | 1.4.5 | 1 | 4 | 5 |
| Agent 5 | 1.1.7 | 1 | 1 | 7 |
| Agent 6 | 1.3.2 | 1 | 3 | 2 |
| Agent 7 | 0.3.1 | 0 | 3 | 1 |
| Power App | 1.22.28 | 1 | 22 | 28 |
| **Σ (= Solution version)** | **7.34.67** | **7** | **34** | **67** |

Bumped 2026-09-23: Agents 1,2,3,4,5,6 all patched for the same real bug (see B5 entry in the
Backlog) - the mode-dependent Configuration value lookup only recognised 4 modes and silently
fell through to the SIMU_DMP column for PROD/SIMU Pre-Default/Post-Default; fixed using the
confirmed real SharePoint internal field names for the 4 new B1 columns
(`Value_x0020__x002d__x0020_PROD_x`/`_x0`, `Value_x0020__x002d__x0020_SIMU_x`/`_x0` for
Pre-/Post-Default respectively - confirmed via a live `GET_DMP_Command_Configuration` run
export, NOT derivable from the display name, since SharePoint truncated/de-duplicated the
auto-generated internal names). Agent 2 additionally got `maximumWaitingRuns: 100` on its
trigger (explicit, previously implicit default 10); `runs` (degree of parallelism)
deliberately left at `1` to protect the shared counter increment from race conditions.

Recompute this table and the resulting solution version on every future component version
bump — never bump the solution version number in isolation.

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
- ~~Rule-level `TimeOfDay`/`TimeZone` existence validation before `convertTimeZone`~~ — done
  in the `7.34.46` hardening (see below), imported and published in Dev.
- ~~A real per-item error counter~~ — done in the `7.34.46` hardening (Scope/catch around
  `CREATE_Occurrence`, see below), imported and published in Dev.
- Authoritative active-case/mode source resolution (open user decision, see backlog).
- `Sequence` field decision, Power App screen live-binding, and the first Dev end-to-end test.

No end-to-end run that creates real occurrence rows has been performed. `7.11.46` was
user-confirmed open/save/activate in the designer; `7.11.47` was user-confirmed on
2026-09-22 to open error-free and to be activated ("Ein") — Save could not be separately
tested because the designer only enables Save after a real change. The hardening now live as
`7.34.46` (below) is the first version since `7.11.46` with real content changes, so it is
the next genuine save/activate test.

### Step 1 hardening deployed 2026-09-22 (pending user save/activate) — now solution `7.34.46`

Per the confirmed work order below, item 1 ("harden Agent 7 a little further") was
implemented in
`PowerAutomate/DMP_COMMAND_Solution/Source/Workflows/DMPAgent7StreamsMilestoneManagement-82E743CD-A2F6-4485-9E76-111D0D30544C.json`,
committed to Git, then packed and imported as solution version `7.11.48` via
`pac solution import --publish-changes` on 2026-09-22 (succeeded; Power Automate reported
"The original workflow definition has been deactivated and replaced", expected for a
workflow-definition update). Local JSON-parse, action-name-uniqueness, and runAfter-graph
checks all passed before packing. **The user still needs to reopen Agent 7 in the designer,
save, and reactivate it** — do that check first in the next session. Immediately afterwards
in the same session, `7.11.48` was found to be a wrong, free-running solution version number
(see the checksum rule in `DMP COMMAND_Mission_und_KI_Arbeitsregeln.md` section I) and
Agent 7's own component label had been left stale at `[0.2.0]`; both were corrected in a
second, versioning-only re-import to `[0.3.1]`/solution `7.34.46` — no further flow-logic
change beyond what is listed here. Changes:

- `VALIDATE_CaseId`: a new If-action after `VALIDATE_RequestedAction` that sets
  `ValidationError` when `triggerBody()?['text_3']` (CaseId) is blank/whitespace-only,
  explicitly commented as a temporary placeholder pending the real active-case lookup (B3).
- `CHECK_RuleScheduleFieldsValid`: a new If-action inside `APPLY_Rules`, right after
  `COMPOSE_OccurrenceId`, that only proceeds to the existing lookup/create logic when the
  current rule's `TimeOfDay` and `TimeZone` are both non-blank; otherwise it composes a
  per-rule error message, appends it to a new `ErrorMessages` array variable, and increments
  a new `ErrorCount` variable, without touching `convertTimeZone`.
- `SCOPE_CreateOccurrence`: `CREATE_Occurrence` is now wrapped in a `Scope`; a sibling
  Failed/TimedOut branch (`COMPOSE_CreateOccurrenceError` → `APPEND_CreateOccurrenceError` →
  `INCREMENT_ErrorCount_CreateFailed`) catches a real SharePoint create failure for one
  occurrence, logs it, and increments `ErrorCount` instead of failing the whole run.
- Two new variables (`ErrorCount` integer, `ErrorMessages` array) initialized alongside the
  existing counters.
- `RESPOND_Result` now also returns `errorCount` and `errorDetails` (joined `ErrorMessages`);
  `success` is `false` whenever `ValidationError` is set **or** `ErrorCount > 0`.
- `SET_ResultMessage_Success`'s text now also reports the error count.

- `VALIDATE_CaseId`: a new If-action after `VALIDATE_RequestedAction` that sets
  `ValidationError` when `triggerBody()?['text_3']` (CaseId) is blank/whitespace-only,
  explicitly commented as a temporary placeholder pending the real active-case lookup (B3).
- `CHECK_RuleScheduleFieldsValid`: a new If-action inside `APPLY_Rules`, right after
  `COMPOSE_OccurrenceId`, that only proceeds to the existing lookup/create logic when the
  current rule's `TimeOfDay` and `TimeZone` are both non-blank; otherwise it composes a
  per-rule error message, appends it to a new `ErrorMessages` array variable, and increments
  a new `ErrorCount` variable, without touching `convertTimeZone`.
- `SCOPE_CreateOccurrence`: `CREATE_Occurrence` is now wrapped in a `Scope`; a sibling
  Failed/TimedOut branch (`COMPOSE_CreateOccurrenceError` → `APPEND_CreateOccurrenceError` →
  `INCREMENT_ErrorCount_CreateFailed`) catches a real SharePoint create failure for one
  occurrence, logs it, and increments `ErrorCount` instead of failing the whole run.
- Two new variables (`ErrorCount` integer, `ErrorMessages` array) initialized alongside the
  existing counters.
- `RESPOND_Result` now also returns `errorCount` and `errorDetails` (joined `ErrorMessages`);
  `success` is `false` whenever `ValidationError` is set **or** `ErrorCount > 0`.
- `SET_ResultMessage_Success`'s text now also reports the error count.

### Confirmed work order for the rest of C5/Streams (user-approved 2026-09-22)

Do NOT reorder this without asking again; the user explicitly approved this sequence in
response to the open `CaseId`/active-case-source question:

1. Harden the current C-strand (Agent 7) a little further, without inventing a real
   active-case source yet: rule-level `TimeOfDay`/`TimeZone` existence validation, a real
   per-item error counter, and a simple non-empty validation on the caller-supplied `CaseId`
   (explicitly marked as a temporary placeholder).
2. Insert **B1–B3** from the Streams concept (Section 8) next, ahead of finishing the rest of
   C: extend `DMP Command Configuration` with the two extra Pre-/Post-Default mode columns
   (B1), move the app from the 2×2 boolean toggle to a 5-value operating-mode enum with a
   guard-flag pattern (B2), then create the `DMP Command Checklist Default Case Context` list
   and the `popDefaultCaseContext` popup (B3). B3 is what finally gives Agent 7 a real,
   authoritative `CaseId` source instead of a caller-supplied placeholder.
3. Only then replace Agent 7's raw `CaseId` trigger input with a real lookup against
   `Default Case Context`, and finish C4/C6/C7 (per-occurrence status/approval actions, the
   remaining Agent 7 actions, and the live Power App binding for `scrTaskOccurrences`).
4. Run C8 (the first real non-destructive Dev end-to-end/idempotency test) — only meaningful
   once Post-Default actually exists as a real mode (i.e. after B2).
5. B4–B5 (Agent 5 and Agent 2 adjustments for the 5 modes) can run in parallel with 3./4.
6. The A-strand (CoS Leader checklist, 4-eyes principle, e-mail automation) is independent and
   has no urgency relative to B/C — start whenever, per user priority at the time.

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
3. **PowerApp local sync still pending (requested 2026-09-22, not yet done):** The user asked
   to also clean up and rebuild the local PowerApp Canvas App sync (there are several stale
   `.msapp` fragments in `PowerApp\DMP_COMMAND\` — `DMP_COMMAND.msapp`,
   `DMP_COMMAND_Solution_counter_and_legend_fix.msapp`,
   `DMP_COMMAND_Solution_counter_fix.msapp`, `DMP_COMMAND_Solution_pending_fixes.msapp` —
   likely leftovers from earlier debugging sessions). **Important before touching this:** a
   fresh `pac canvas download` + `unpack` would overwrite `PowerApp\DMP_COMMAND\Source\Src\`,
   which currently contains real, not-yet-published-to-Studio local edits (at minimum
   `scrTaskOccurrences.pa.yaml`, and possibly other pending screen changes) — Canvas Apps can
   only be published back to the environment by the user manually loading the packed
   `.msapp` into Power Apps Studio (no `pac` command does this automatically). Do the
   download/unpack into a separate temporary folder first, diff against the current
   `Source\Src`, and only then decide with the user what to keep/replace, exactly like the
   Git recovery above — never blindly overwrite.
4. **Ask the user to open, save, and reactivate Agent 7 (now solution `7.34.46`) in the
   designer** (this is the first version since `7.11.46` with real content changes, so Save
   is a genuine test this time) and report any error before making further changes.
5. Re-read the current Agent 7 JSON; do not restore any earlier `7.11.36`–`7.11.48` package.
6. Continue with the confirmed work order above (harden C a little → B1–B3 → rest of C → C8 →
   B4–B5 → A-strand), using small designer-validated increments. Never introduce several
   unvalidated connector fields in one deployment.

### Other previously open state

- The older Agent 2 and System Health ring follow-ups remain in the active backlog unless
  separately verified and closed.
- The Power App Task Occurrences screen is prepared locally but is not live-data-bound.
