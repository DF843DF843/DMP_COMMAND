# DMP COMMAND - Release Notes

Automatisch aus der In-App Release-Notes-Seite (scrReleaseNotes.pa.yaml) exportiert. Diese Datei wird ab sofort bei jedem neuen App-Release aktualisiert (KI-Regel vom 2026-09-01).

## App Changes

### v1.22.34 - 2026-09-23 (current, not yet loaded/saved by user in Studio)

- **Fixed v1.22.33 compile errors (108 App-Checker errors):** the nested `AddColumns(AddColumns(AddColumns(...)))` expression used to build the Next Steps collection did not compile in Studio (`AddColumns` column formulas can only see the original source table's own columns, never a column added by a nested/chained `AddColumns` used as its `Source` argument - even one level deep). Rebuilt as 3 separate materialized stages (`colCosLeaderStage1` → `colCosLeaderStage2` → `colCosLeaderNextSteps`), each a real `ClearCollect`'d collection before the next stage references it.
- **Separately found (not caused by this session's changes):** the live `DMP Command External Domains` SharePoint list currently has no `Active` column at all - only became visible once the app's data source schema was refreshed this session. Flagged for investigation (Agent 1 recreates this list on every Emergency Report extraction).
- Checks before packing: control-name uniqueness (866, 0 duplicates), `": "` regex scan (0 hits), pack→unpack round-trip diff (0 diff on `scrHome`, `scrReleaseNotes`, `App`).

### v1.22.33 - 2026-09-23 (superseded by v1.22.34 above - had compile errors, never loaded successfully)

- **Cockpit NEXT STEPS panel replaced with a live CoS Leader view (Next Steps feature, Phase 1):** the old panel showed up to 5 static milestones sourced from Agent 4's hardcoded response (`varNextMilestones`). It now reads live from `DMP Command Checklist CoS Leader`, joined by `Titel`/TaskID with `DMP Command Checklist Overall Process` for `SeqNo`/`PredecessorTaskIds`/`IsMilestone`. Shows up to 5 recently completed tasks (sorted by confirmation time) plus up to 8 open tasks (ongoing first, then ready/executable, then not-yet-ready, each by `SeqNo`), each with a status dot and word (COMPLETED/ONGOING/PENDING/NOT STARTED - green/orange-blinking/orange/grey), milestone tasks marked with a ★. A task counts as executable once its required DMP phase has been reached AND all of its `PredecessorTaskIds` are `Done`. Shows an idle message when no DMP case is active. Refreshes on every Cockpit visit and every 30 seconds thereafter, independent of the Agent 4 status-check cycle.
- **Data-model groundwork for the Next Steps dependency model:** added 3 new columns to the already-live `DMP Command Checklist Overall Process` list - `SeqNo` (Number), `PredecessorTaskIds` (Text, semicolon-separated Task IDs that must all be `Done`), `IsMilestone` (Choice: Yes/No). Refreshed the app's data source schema from a fresh Studio publish (`pac canvas download` + unpack, identity-verified before trusting it) to pick these up, without overwriting the hand-maintained screen YAML (confirmed the Studio round-trip diff was purely cosmetic reformatting).
- **Scope decision - Phase 1 is CoS Leader only:** Infrastructure Team and Content Team checklists do not exist yet (no SharePoint write access to those team sites) and are deferred; the Overall Process → substream-checklist link works via identical `Titel` values (no extra link field needed, confirmed against the 2026-09-03 Streams design). Known Phase 1 limitations by design: no overdue/red state yet (no due-date column on the checklist), no in-app propose/confirm action yet (status changes still happen directly in SharePoint), predecessors must currently be on the same CoS Leader checklist.
- Checks before packing: app-wide control-name uniqueness (866 names, 0 duplicates), literal `": "` regex scan (0 hits), pack→unpack round-trip diff (0 diff on `scrHome`, `scrReleaseNotes`, `App`). **First-ever use in this app of nested `AddColumns`+`LookUp`+`Filter(Split(...))` for a cross-list dependency join - not yet Studio-validated, please watch for red error badges on the Cockpit after loading.**

### v1.22.32 - 2026-09-23 (confirmed loading in Studio; findings addressed in v1.22.33 above)

- **System Health (Details) - Connection Diagnostics merged in from Maintenance:** the user compared v1.22.31's System Health page against the Maintenance page's compact "Connection diagnostics" row style ("Vergleich mit Maintenance - Connection diagnostic! So ähnlich könnte das aussehen") and pointed out that connection diagnostics belongs functionally to System Health, not Maintenance. Per the user's choice, the section was **moved entirely** (not duplicated): removed from `scrMaintenance.pa.yaml`, rebuilt in `scrSystemHealthDetails.pa.yaml` covering all 17 connected SharePoint lists (previously only 3 on Maintenance), split into two cards (Core & Domain Lists / Streams Lists) plus a short "About the dots" note.
- **System Health (Details) - horizontal 2-column grid instead of a vertical stack:** per "Es können 2-4 Bereiche horizontal nebeneinander", the page now uses 3 horizontal rows of 2 cards each (`ManualLayout` row wrappers with `Width: =(Parent.Width-16)/2` per card, chosen over `AutoLayout` after past sizing issues with that variant) instead of one full-width card per row: Core Status | Data Sources & Files, Agents (1-7) | Connection Diagnostics - Core & Domain Lists, Connection Diagnostics - Streams Lists | About the dots. All 16 pre-existing status formulas copied verbatim - no logic change.
- **Configuration (Lists) - section headers now stand out:** per "Die Abschnittsüberschriften heben sich nicht hervor", each of the 6 section cards now has a solid dark-purple header bar with white bold text (`RGBA(32,23,81,1)`), replacing plain colored text on the translucent card background.
- **Configuration (Lists) - compact tile grid instead of one row per list:** per "Die Kacheln pro Liste sind viel zu groß... Pro Container die Kachel gruppieren, Horizontal maximal 4 nebeneinander", the previous one-row-per-list layout was replaced with tiles grouped up to 4 per row within each section card (`Width: =(Parent.Width-68)/4`). Since every section has at most 4 lists, each section is now exactly one tile row; all section cards share a uniform 140px height (previously 110-170px depending on list count). All 17 list names/count formulas/View-New-Entry URLs/tooltips extracted programmatically from the v1.22.31 file (not retyped) - only layout changed, no data or link changed.
- **Maintenance - Connection diagnostics section removed** (moved to System Health, see above); the page now only shows Versions and Admin portal links.
- Checks before packing: app-wide control-name uniqueness (874 names, 0 duplicates), literal `": "` regex scan in single-line `Text:`/`Tooltip:` formulas (0 hits in new/changed lines), pack→unpack round-trip diff (0 diff on all 6 changed files).
- **Separately delivered by the user, not yet implemented:** a complete 21-section specification for the "Next Steps"/CoS-Leader feature (`DMP_COMMAND_Next_Steps_Anforderung.md`), replacing the 8 open clarifying questions from the v1.22.31 Backlog entry. Checked against the live data model - implementation planned for a future session (see Backlog Priorität 2 Punkt 1 for the gap analysis).

### v1.22.31 - 2026-09-23 (confirmed loading in Studio; findings addressed in v1.22.32 above)

- **System Health (Details) page redesigned:** the v1.22.30 version used large `AutoLayout` cards with 16px dots and 13px text that rendered far too big and partly truncated (user feedback: "viel zu groß und unlesbar" after loading v1.22.30 in Studio). Rebuilt as a compact fixed-layout table matching the Cockpit's own "Automation Status" container style (14px dots, 11px text, 26px row spacing, same 3 groups: Core Status, Data Sources & Files, Agents 1-7) - all 16 status formulas copied verbatim, no logic change.
- **Configuration (Lists) tab redesigned:** the 17 list tiles (130px each, ~2450px of scrolling) were reported too large/unwieldy ("viel zu groß und unübersichtlich"). Replaced the 17 individual tiles with 6 bordered section cards (one per theme) styled like the Cockpit's "Maintenance - Domains" container; each card now holds one compact row per list (reachability dot, name, live row count, View/New Entry buttons) with the description text moved to the View button's tooltip. Same 6 groupings, same counts/URLs - no data source or link changed.
- **Task Occurrences / "Next Steps" concept - NOT changed this session, needs joint design:** the user clarified the real requirement goes beyond readability - a CoS-Leader-facing "Next Steps" view of done/upcoming tasks, from which template e-mails can be triggered, pre-filled with parameters from the new SharePoint lists (Email Templates, Recipient Groups, Default Case Context, etc.). This is a new feature concept, not a bug fix, and needs a joint brainstorming/design pass before implementation - see the Backlog for the prepared clarifying questions.

### v1.22.30 - 2026-09-23 (confirmed loading in Studio; 3 findings addressed in v1.22.31 above)

- **System Health legend moved to its own page:** the Cockpit ring's popup could only show a cramped, scrolling 300px box for all 16 monitored items (user feedback: legend not fully visible) - it is now a dedicated `scrSystemHealthDetails` page (new sidebar entry "System Health" + still reachable by clicking the ring), showing all 16 items grouped into Core Status, Data Sources & Files and Agents (1-7), with no size limit.
- **Task Occurrences "Current view" panel - real readability bug found and fixed:** the description text's fixed 28px-tall box was far too short for its ~300-character content, so most of it rendered off-box and visually overlapped the row above (reported as unreadable). Box height/wrapping fixed, and the value label's near-black-on-dark hardcoded colour (also unreadable in dark mode) is now theme-aware. The same hardcoded-colour bug was also fixed on each row's Status label.
- **Task Occurrences "Current view" value corrected to match the real filter/sort logic:** was labelled "Open and overdue occurrences", which does not describe what the Gallery actually shows - now "All non-rejected occurrences, soonest due first".
- **Configuration (Lists) tab reorganized into 6 thematic sections** (Core Runtime Configuration & Monitoring, Domain Classification, Streams - Process & Checklists, Streams - Case & Scheduling, Streams - Communication, Streams - Access Control & Catalogs) and now scrolls; added View/New tiles for all 14 previously-unlisted live SharePoint lists (External Domains, Counters, and all 12 Streams lists), so every connected list can be found and opened from this tab.

### v1.22.29 - 2026-09-23 (confirmed loading in Studio; 3 findings addressed in v1.22.30 above)

- **B3 (Default Case Context popup) - connected to the real SharePoint list:** the user added `DMP Command Default Case Context` as a live data source in Studio, so the popup's Confirm button now Patches the real list directly instead of a local collection (`colDefaultCaseContextPending` removed). Note: the list has no separate `CaseId` column - Case ID is stored in `Title`.
- **Task Occurrences screen (C6/C7) - connected to the real SharePoint list `DMP Command Checklist Task Occurrences`:** the Gallery, Propose/Approve/Reject buttons and four-eyes check (proposer cannot self-approve) now read/write the real list via `Patch()` (`colTaskOccurrencesPreview` removed). Note: the list has no separate `OccurrenceId` column either - `Title` holds it. May show 0 rows until Agent 7 creates its first real occurrences.
- Both are the first real `Patch()` calls in this app against SharePoint Person and Choice columns - not yet Studio-validated live, please test the B3 popup and a manual Task Occurrences row (if any exist) after loading this version.
- `.msapr` data source references refreshed from the just-published live app (both new lists plus several other Streams lists the user also connected).

### v1.22.28 - 2026-09-23 (confirmed working by the user; superseded by v1.22.29 above for the B3/Task Occurrences live wiring)

- **Admin Functions - removed the temporary "TIMESTAMP DEBUG" panel** (`conFuncTimestampDebug`, added in v1.22.21) now that the Audit Trail year-3926 timestamp bug is confirmed fixed.
- **B1 (Configuration - 4 new Pre-/Post-Default value columns) confirmed complete:** user confirmed the columns are live in the real SharePoint list; a full programmatic check of `DMP Command Configuration.csv` found no gaps caused by the B1 extension (5 unrelated, pre-existing empty `...OpenUrl` rows logged separately in the Backlog).
- **B5 - real bug found and fixed in 6 of 7 Power Automate flows** (see Agent Changes below, v1.0.9/1.0.10/1.1.5/1.4.5/1.1.7/1.3.2): the mode-dependent Configuration value lookup only knew 4 modes and silently fell through to the `SIMU_DMP` column whenever the app was in Pre-Default or Post-Default - fixed using the confirmed real SharePoint internal field names for the 4 new B1 columns (confirmed via a live `GET_DMP_Command_Configuration` run export, since they were not derivable from the display name).
- **Agent 2 - added explicit `maximumWaitingRuns=100`** on the shared-mailbox trigger (previously implicit default 10); degree of parallelism intentionally left at 1 (sequential) to protect the shared counter increment from race conditions.
- Solution version recomputed to **7.34.67** (not yet imported).

### v1.22.27 - 2026-09-22 (confirmed working by the user; superseded by v1.22.28 above for the Admin Diagnostics cleanup)

- **System Health ring/legend - real fix #5 (user-confirmed v1.22.26 working, two follow-up issues reported):** Critical Events and Warnings were shown as red/orange purely from the raw Audit Trail counters (`varAuditFailedCount`/`varAuditWarningCount`), ignoring the existing "Reset Critical/Warning Counter Baseline" confirm mechanism (Agent 6 AdminFunctions action, wired in `scrAuditTrail.pa.yaml`) - so a confirmed/reset event stayed red in the ring even though the rest of the app (the Critical/Warnings KPI tiles further down this same screen) already treats it as cleared. Both the ring segments and the legend rows now use the same `Max(raw-baseline,0)>0` logic already established for those KPI tiles, so a confirmed event turns green consistently everywhere.
- **System Health details popup - readability fix:** reported unreadable (no scrollbar) after being extended to all 9 fixed categories + all 7 agents in v1.22.26. Gave the popup a fixed height (300px) with vertical scroll (`LayoutOverflowY`), matching the scrollable-panel convention already used elsewhere in this app.
- `DMP Command Configuration.csv` (both copies) refreshed from the user's newly recreated file - all 8 mode-value columns (`Value - PROD/SIMU (NODMP/DMP/Pre-Default/Post-Default)`) are now filled in for every row.

### v1.22.26 - 2026-09-22 (confirmed working by the user - ring/N-A/legend content fix; superseded by v1.22.27 above for the baseline+scrollbar refinement)

- **System Health ring - real fix #4 (user-reported live bug: ring showed 34% then 87% at load with visible red/yellow segments, while the details popup listed everything as OK):** `vGreenSegments` recounted agent health directly from the raw `DMP Command Agent Status` list (`CountIf(vAgents, CurrentStatus.Value="Operational")`) instead of from the already-correctly-greyed-out `vAgentSegments`, so the shown percentage ignored the "never refreshed"/"refreshing" grey-out state entirely. Fixed to `CountIf(vAgentSegments, SegColor="rgb(0,206,125)")` (added as its own nested `With()` level to avoid the same record-self-reference bug class fixed in v1.22.23). Also added an explicit grey **"N/A"** center-text state for before the first successful refresh, matching the documented behaviour (Operations Manual 3.1.3) which was never actually implemented in the formula.
- **System Health details popup - real fix:** the popup only ever listed 6 hardcoded rows (Emergency Report by file-existence, Internal/External Domains, Counter, Audit Trail, Agent 6 only), while the ring itself has always had 9 fixed segments plus one segment per agent (currently 7). Any red/yellow ring segment for Status Check, Critical Events, Warnings, Operating State, or any agent other than Agent 6 was therefore invisible in the popup - exactly the reported symptom. Added the 4 missing fixed rows and rows for all 7 agents, each using the exact same condition as its corresponding ring segment. The Emergency Report row now reflects the ring's actual "Emergency Report Processing" condition (Red/Yellow state), keeping file-existence as a parenthetical instead of the sole (and different) condition it used before.
- **Not yet Studio-validated for visual overflow:** the popup is now taller (11 more rows); its container has no fixed height (`AutoLayout`), so it should grow correctly, but this needs the user's visual confirmation on next load.

### v1.22.25 - 2026-09-22 (confirmed loading fine in Studio by the user; superseded by v1.22.26 above for the ring/legend fix)

- **Fixed a second YAML bug (PA1001, "invalid mapping") that made v1.22.24 fail to load in Studio:** `Src\scrTaskOccurrences.pa.yaml(204,73)`. Several `Text`/`OnSelect` formulas contained an unquoted colon-space sequence in a plain YAML scalar - inside a string literal (`"...four-eyes: proposer..."`), inside a Power Fx record literal used by `UpdateIf` (`{Status: Switch(...), ...}`), and in a `"Propose: "` label prefix. YAML plain scalars cannot contain `": "` anywhere (it is ambiguous with a mapping key), even though the value is a valid Power Fx formula. Converted all five affected properties to block-scalar (`|-`) form, matching the convention already used elsewhere in this app for complex formulas. Confirmed via a clean `pac canvas pack`/`unpack` round-trip (no diff across any screen), but per the standing rule this alone is not proof - Studio's own load is still the real test.

### v1.22.24 - 2026-09-22 (superseded same day, folded into v1.22.25 above)

- **Fixed a YAML indentation bug (PA1001, "did not find expected key") introduced in v1.22.23:** the extra closing `)` added to fix the ring's record self-reference scope bug was indented to exactly match the following `Width:` property, causing Studio's stricter PaYaml parser to treat the block scalar as ended one line early and the lone `)` as an invalid new mapping key. Re-indented to stay clearly inside the block scalar; `pac`'s own pack/unpack round-trip did not catch this (known limitation - only Studio validates PaYaml strictly), so re-verify carefully after this fix.
- **Task Occurrences screen - first four-eyes workflow demonstration (C6/C7 preview):** the preview Gallery now runs on a real, patchable local collection (`colTaskOccurrencesPreview`, exact schema of the live `DMP Command Checklist Task Occurrences` list) instead of a static `Table()` literal. Each row now has a working Propose/Approve/Reject flow: "Propose" advances Status and sets `ApprovalState=Pending`; "Approve"/"Reject" only appear for a *different* user than the proposer (enforces the four-eyes rule - a proposer cannot approve their own change) and set `ApprovalState` accordingly. Still local-only (not yet connected to the real SharePoint list - needs the same one-time manual "Add data" step in Studio as the B3 Default Case Context popup). **Known open design gap, flag to user:** the real list schema has no `PreviousStatus` field, so "Reject" currently only clears the pending approval state - it does not revert `Status` to its pre-proposal value. Decide before going live whether to add such a field or accept that a rejected proposal needs a fresh manual re-propose.

### v1.22.23 - 2026-09-22 (superseded same day, folded into v1.22.24 above)

- **System Health ring - real fix #3 (root cause of "vFixedSegments wird nicht erkannt"):** `vGreenFixedSegments: CountIf(vFixedSegments, SegColor=...)` was defined in the SAME record literal as `vFixedSegments` itself (`With({vFixedSegments: ..., vAgents: ..., vGreenFixedSegments: CountIf(vFixedSegments, ...)}, ...)`). Power Fx record literals cannot reference sibling fields within the same `{...}` - only nested `With()` calls can build on a previously bound name. Split into its own nested `With()` level (matching the pattern already used correctly for every other step further down in the same formula). This was very likely present, unfixed, since this ring formula was first written - masked until now by the other errors fixed in v1.22.20-22.
- **Release Notes Agent tab - same root-cause bug as the App tab, also fixed:** `varSelectedAgentRelease` was never initialized and had no Switch case or menu button for Agent 7 - defaulted to Agent 1 instead. Added the missing `btnAgentRelease7` menu button, the "7" Switch case in both detail-pane Switches, and an app-start default of `"7"`.
- Default Case Context popup: shortened the Termination Date+Time field label (was clipped: "...YYYY-MM-DD HH:M") to "Termination Date + Time (local) *" - the exact format is already shown in the field's own placeholder text.

### v1.22.22 - 2026-09-22 (superseded same day, folded into v1.22.23 above)

- **System Health ring - real root cause found and fixed:** the "Enum, Text" comparison error the user hit in Studio was caused by naming the segment color field `Color` - Power Fx appears to infer that specific field name as its built-in `Color` enum type rather than Text once compared with `=`. Renamed the field to `SegColor` everywhere (`vFixedSegments`, `vAgentSegments`, the `AddColumns`/`Switch` sort-rank formula, and the SVG-generating `segment.SegColor` reference) - this was actually a second, previously undiscovered instance of the same bug class, not just the one the user reported.
- **Audit Trail timestamp - real root cause found and fixed:** the Excel-serial-to-date conversion used `Date(1899,12,30)+Value(rawTs,"en-US")`, which is out of Power Apps' supported date range (dates before 1900-01-01 are not supported) and silently produced a garbage result (e.g. year 3926, time truncated to 00:00:00) instead of an error. This exact epoch was already present, unfixed, all the way back to a September 3 commit that only reordered branch preference around it, never fixed the math itself. Replaced with the community-standard, Power-Apps-safe conversion: `DateAdd(Date(1900,1,1),(Value(rawTs,"en-US")-2)*86400,TimeUnit.Seconds)` (the `-2` corrects for the 1900-01-01-vs-1899-12-30 epoch difference and Excel's fake 1900 leap day). Applied to all 20 occurrences in `scrAuditTrail.pa.yaml` and to the temporary Admin Functions debug panel.
- **Release Notes screen - real root cause found and fixed:** the screen's "detail pane" selector variable `varSelectedAppRelease` was never initialized anywhere and the big version-selector `Switch` had no case for any version after v1.22.13 - so the screen always silently fell back to showing v1.22.13, regardless of how many newer versions existed in the scrollable list. This was not a stale-Studio-session issue as first assumed. Added an app-start default (`Set(varSelectedAppRelease,"V12221")`) and a new top menu entry/Switch case for the current version.
- Default Case Context popup (B3): title is now a solid colored banner with always-white text (matching the established badge/pill contrast convention) instead of colored text directly on the card background, which was hard to read in dark mode. The Termination Date+Time field is now clearly labelled and entered as local time, converted to UTC (`TimeZoneOffset`) only when staged for storage - previously it asked for UTC input directly, which most users don't know off-hand.

### v1.22.21 - 2026-09-22 (superseded same day, folded into v1.22.22 above)

- **B3 (Default Case Context popup) - first implementation.** Added `conDefaultCaseContextPopup` to `scrHome.pa.yaml`: a mandatory, centered modal form (Case ID, Termination Reason, Defaulted Member ID/Name, Termination Date+Time as `YYYY-MM-DD HH:MM`) that now gates every Pre-Default -> DMP transition of `btnOperationalModeAdvance`. Confirming validates required fields, stores the entry in a local `colDefaultCaseContextPending` collection (exact same column shape as the real SharePoint list), then re-triggers the actual mode-advance logic; Cancel aborts the transition without changing mode.
- **SharePoint list clarified:** `DMP Command Default Case Context` (note: real name drops "Checklist", unlike Recurrence Rules/Task Occurrences) already exists (site `GO365_DMPCommunication-CoSLeader`, GUID `8fa1f858-3f89-4600-b3dc-86f9dfcaf4b8`), with columns confirmed to match the prepared template exactly (internal field name for the display column "TerminationDateTime" is actually `TerminationDate`). Not yet wired as a live Power App data source (needs one manual "Add data" step in Studio); until then the popup only writes to the local collection above.
- **Control-type lesson learned:** `Classic/TextInput` had never been used in this app before (like the Slider decision earlier). A first attempt using a guessed `TextInput@2.3.2` (without the `Classic/` prefix) was caught locally (missing from the packaged template registry) before being sent for testing - the user placed one blank Text Input in Studio and saved, confirming the real control is `Classic/TextInput@2.3.2`, matching this app's Classic-control-family convention (like `Classic/Button`). All 5 new fields now use this verified, Studio-sourced declaration.
- Bundled from v1.22.19/20 (both never tested live, superseded by this version): the real `vGreenFixedSegments` fix, and the temporary Admin Functions "TIMESTAMP DEBUG" panel.

### v1.22.20 - 2026-09-22 (superseded same day, folded into v1.22.21 above)

- Added a temporary "TIMESTAMP DEBUG" panel to the Admin Functions screen (`conFuncTimestampDebug`, orange-bordered), directly below the existing Diagnostics panel. It takes the most recent real Critical/Warning row's raw `timestamp` value and shows it side-by-side with 4 parsing approaches (A: current production formula, B: always-Excel-serial, C: `DateTimeValue` with app/system locale, D: `DateTimeValue` forced to `en-US`), so the correct approach can be identified from real live data instead of guessing. Root cause context: source-side investigation this session confirmed the app-side date parser (`scrAuditTrail.pa.yaml`) already contains the numeric-first-check approach documented as fixed since v1.22.13/17 - the user-reported "still broken" symptom is most likely explained by the live Studio app not having been reloaded since 2026-09-04 (confirmed via source review, not yet confirmed against live data). This panel is meant to be removed again once the correct approach is confirmed live.

### v1.22.19 - 2026-09-22 (superseded same day, folded into v1.22.20 above)

- Fixed `vGreenFixedSegments` for real: it still contained `CountIf(Table(vStatusCheckColor, vCriticalColor, ..., vAuditTrailColor), Value="rgb(0,206,125)")`, passing nine plain text values into `Table()` (invalid - confirmed by Studio's Advanced Formula Checker via user-supplied error analysis). The v1.22.17/v1.22.18 release notes incorrectly claimed this was already fixed (`CountIf(vFixedSegments, Color=...)`); that edit was never actually saved into `scrHome.pa.yaml`. Now genuinely applied: `vGreenFixedSegments: CountIf(vFixedSegments, Color="rgb(0,206,125)")`, reusing the already correctly-typed `vFixedSegments` table instead of building a new `Table()` from scalars.

### v1.22.18 - 2026-09-22 (superseded same day, folded into v1.22.19 above)

- Reverted the System Health ring's `vHealthSegments` from `Table(vFixedSegments, vAgentSegments)` (confirmed invalid by Studio's Advanced Formula Checker: "Die Funktion `.Table` weist ungültige Argumente auf") back to the working `ForAll(Sequence(...), If(..., Index(...), Index(...)))` union approach from an earlier iteration. `Table()` does not merge existing table variables in Power Fx, only record literals - this was a wrong simplification in v1.22.16/17.
- Kept the `vGreenFixedSegments` fix (`CountIf(vFixedSegments, Color=...)`) from v1.22.17, which is unrelated and unaffected by this revert.

### v1.22.17 - 2026-09-22 (current, not yet loaded/saved by user in Studio)

- Fixed the System Health ring for real this time: the `vHealthSegments` union was simplified from an over-engineered `ForAll(Sequence(...), Index(...))` attempt (still errored) to the simple, Power-Fx-documented `Table(vFixedSegments, vAgentSegments)` (concatenates two tables with matching schema).
- Also found and fixed a second, separate instance of the exact same class of bug in the same formula: `vGreenFixedSegments: CountIf(Table(vStatusCheckColor, ...), Value=...)` passed 9 plain text values into `Table()`, which is invalid - simplified to `CountIf(vFixedSegments, Color=...)`, reusing the already-correctly-typed table instead.
- Extended the "Agents Active" KPI (Cockpit header) from `X/6` to `X/7`: added a real, client-side `varAgent7Healthy` check (`LookUp('DMP Command Agent Status', AgentKey="Agent_07").CurrentStatus.Value="Operational"`, no Agent 4 flow change needed) as a 7th term.
- Added the missing Agent 7 entry to the Release Notes screen's "Agent (Flow) Changes" tab (previously only Agents 1-6 were listed there, even though the Agent Monitoring tab and this document already covered Agent 7).
- Fixed poor text contrast on the new Operating State "advance" button: text color was following the app's dark/light theme setting instead of being readable against the button's own (always colored) background - now always white, regardless of theme, matching standard practice for colored badge/pill buttons. (Per the project's established Eurex-palette rule, the safety-critical mode colors themselves - including DMP's red - intentionally stay a generic warning red rather than a branded color.)
- Refreshed the local `.msapr` packaging container from a fresh `pac canvas download` of the live app - the previous container was stale from 2026-09-04 (before Agent 7 existed) and did not carry the Agent 7 flow connection, so every local repack silently dropped it again even after the user manually reconnected it in Studio. This should stop the Agent 7 connection from repeatedly disappearing.
- Fixed the Audit Trail (Detail) date/time display showing the wrong year again (e.g. "3926" instead of "2026") on the Recent Critical/Warning rows, a recurrence of a bug previously fixed several times (v1.22.9-v1.22.13). Root cause this time: the numeric-vs-ISO-text branch decision used `Value(rawTs, "en-US")` succeeding/failing to decide the format, but `Value()` was too lenient and partially parsed some ISO datetime strings as numbers instead of failing outright. Replaced with an explicit `IsMatch(rawTs, "^\d+(\.\d+)?$")` check (true only for a pure decimal number) to decide the branch, which cannot misfire the same way.

### v1.22.16 - 2026-09-22 (superseded same day, folded into v1.22.17 above)

- Fixed the pre-existing P1 System Health ring bug (`imgHeartbeatWheel.Image`, open since 2026-09-04): `vFixedSegments & ForAll(...)` tried to concatenate two tables with the text `&` operator (invalid). Replaced with a `ForAll(Sequence(...), If(..., Index(...), Index(...)))` pattern that correctly unions the two tables.
- Fixed the Operating State mode label/color (`lblOperatingModeText` area) which still only distinguished 2 states (Normal/DMP) after the B2 5-mode change, showing misleading text like "Normal non-DMP Operation" while actually in Pre-Default/Post-Default. Extended to all 4 modes, matching the new advance button's color scheme.
- Same fix applied to the 4 ambient screen-border strips (`conFrameTop/Bottom/Left/Right`), which also only recognized 2 modes.
- Added the missing Agent 7 tile (`conAgentTile07`) to the Agent Monitoring screen - it only had static tiles for Agents 1-6. Requires a corresponding `Agent_07` row in the `DMP Command Agent Status` SharePoint list (not yet added - user action required) to display real data instead of blanks/errors.

### v1.22.15 - 2026-09-22 (superseded same day, folded into v1.22.16 above)

- Replaced the Normal/DMP Operating State toggle with a single colored "advance" button showing the current phase and the next one (e.g. "Normal -> Pre-Default"), enforcing the linear Normal -> Pre-Default -> DMP -> Post-Default -> Normal cycle - direct jumps between non-adjacent phases are no longer possible. Part of the Streams concept's B2 (5-mode rollout).
- Switching Environment to PROD now always resets Operating State back to Normal for safety (previously this safety reset only fired from DMP); switching to SIMU now preserves any of the 4 phases, not just DMP/Normal.
- New internal state model: `varOperationalMode` (text: Normal/PreDefault/DMP/PostDefault) and `varOperationalStepCounter` (a monotonically increasing step count, mod 4, used to bound the single-step advance) added alongside the existing `varOperationalModeIsDMP` (still derived/kept for backward compatibility with existing color/border formulas elsewhere in the app).
- Deliberately did NOT introduce a native Power Apps Slider control for the "advance" interaction, despite the user's preference for a slide-switch look: this codebase has never used a Slider control before, and guessing its exact schema/version risked a repeat of past silent Power Fx/schema failures. Implemented instead as a styled `Classic/Button` (a proven, already-used control type in this app) that looks like a colored pill and always advances exactly one step per tap - functionally equivalent safety guarantee (only one step possible), lower deployment risk. A literal drag-slider can be added later if desired once this is confirmed working, ideally by having the user insert Studio's native Slider control visually and then wiring the same one-step-bounded logic to it.
- Not yet packed/loaded/saved/tested by the user in Power Apps Studio this session - the previous confirmed-good app version remains v1.22.13.

### v1.22.14 - 2026-09-04 (published; P1 follow-up required)

- Published the direct SharePoint counter refresh and zero-value Emails Processed ring fix.
- Published the first System Health dynamic-segment implementation, but it has a live Power Fx type error ("Only record or table values can be used in that context") in the dynamic table composition. This implementation is not accepted as complete; its repair is the first P1 task of the next session.

### v1.22.13 - 2026-09-04

- Found and fixed the actual root cause of the non-responsive Admin Functions and Audit Trail Reset buttons - the app's internal reference to the Agent 6 flow was named "DMPAgent6(AdminFunctions)[1.1.0]" while all 5 button formulas called it without the version suffix; all 5 call sites corrected to the exact live connector name
- Rebuilt the Audit Trail date/time parsing to be more robust - the raw timestamp is now checked first for being a plain number (Excel serial date) and converted directly; only genuine ISO-formatted text falls back to the text-based date parser, removing the previous unreliable "try text-parsing first" approach and its accompanying implausible-year workaround
- The "Loading..." indicator shown while an Audit Trail action is running now shows a context-specific message (e.g. "Resetting Critical baseline...", "Resetting Warning baseline...", "Resetting all baselines...", "Refreshing audit data..."), vertically centered with the button row instead of a generic fixed text
- Removed the temporary raw-timestamp diagnostic label from the Audit Trail screen, now that the underlying date bug is confirmed fixed

### v1.22.12 - 2026-09-04

- Fixed the Audit Trail date/time display - dates now use direct date arithmetic instead of a function that silently truncated the time-of-day to 00:00; both the date and the time portion now display correctly
- Fixed a stale Agent 6 flow connection reference in the packaged app file (re-synced from a live download after the flow was reconnected in Studio)
- Fixed the 3 Audit Trail Reset buttons and the Admin Functions counter-reset action, which failed to run at all ("IfError has invalid arguments") because their error-fallback result did not have the same fields as the flow's real response - all 5 call sites corrected
- Audit Trail (Detail) no longer triggers its own separate Agent 4 call when the tab is opened - it now just kicks off the same shared periodic refresh timer the Cockpit already uses, avoiding a duplicate refresh
- The 3 Reset buttons (Critical/Warning/All) on the Audit Trail screen now show the existing "Loading..." indicator immediately after being clicked, instead of giving no visible feedback while the call is in flight
- Help / Operational Manual screen extended with 6 new sections (Agent Monitoring, Audit Trail (Detail), Configuration (Lists), Maintenance, Admin Functions, Release Notes), matching the full screen set of the app; System Health legend corrected to 6 monitored items (added Agent 6) and the Sidebar section updated to include Release Notes
- Synced the PowerApp_Version.txt source file to the actual current version, which had been stuck at v1.22.11

### v1.22.11 - 2026-09-03

- Found the real cause of the persistent Audit Trail year-3926 date bug - the underlying timestamp is stored as ISO text (e.g. "2026-09-02T12:34:43Z"), not a plain Excel serial number as previously assumed, and the earlier fix's Value()+DateAdd() approach mis-parsed that text using the wrong internal date epoch. Now uses DateTimeValue() first (built for exactly this text format), falling back to the old numeric approach only if that fails
- Audit Trail (Detail) now shows a "Loading recent alerts..." indicator while refreshing, plus a dedicated "Refresh now" button, instead of only silently reloading
- The Critical/Warning/All reset buttons (Audit Trail) and the Admin Functions counter reset buttons now show a clear error notification if the underlying flow call itself fails, instead of silently doing nothing
- Fixed the Maintenance tab's app version link - text was wrapping into an unreadable two-line block because the button was too narrow/short for its own text

### v1.22.10 - 2026-09-03

- Found and fixed the likely root cause of several reported bugs (version number not updating, timestamps not updating, counters/baselines staying at 0) - the very first automatic status refresh at app start set its own 'done' guard immediately, before actually checking if the call succeeded, so a single early failure (e.g. connections not yet ready) permanently blocked ALL further automatic refresh attempts for the rest of the session - the guard is now only set once a refresh actually succeeds, or after 15 retries (~12s)
- Added a "Copy to Clipboard" button next to the Admin Functions Diagnostics panel's Refresh button, copying the 3 counter/audit lines in plain text
- Synced the PowerApp_Version.txt source file to the actual current version - it had been stuck at an old value for several releases, which was a second, independent reason the version badge looked frozen

### v1.22.9 - 2026-09-02

- Fixed the Emergency Report Replace action not refreshing the Cockpit's numbers afterwards (e.g. Internal/External Domains counts) - it now re-runs the full status refresh on success, same as the Now button
- The Critical/Warning/All reset buttons now immediately re-check with Agent 4 after a reset instead of only guessing the new baseline locally, so the displayed 'new since reset' count reflects confirmed data right away
- Fixed a bad follow-up bug in the Audit Trail timestamp fix - dates were showing a wildly wrong year (e.g. 3926 instead of 2026) because the conversion multiplied the value into billions before converting, which silently overflowed - switched to the simpler, standard conversion approach that avoids large intermediate numbers
- Added a read-only Diagnostics panel to Admin Functions showing the live email counter, Critical/Warning total and baseline values, to help pin down the reported counter/reset display issues

### v1.22.8 - 2026-09-02

- Audit Trail (Detail) now auto-refreshes on its own every auto-update interval while the screen is open, instead of only when the tab is first opened
- Fixed garbled timestamps in the Recent Critical/Warning lists (raw Excel serial numbers like '46225.50...') - now converted to a readable date/time, falling back to the original text if it is already a normal timestamp
- Fixed the Recent Critical/Warning lists sometimes showing older rows instead of the true last 10 - the underlying Audit Trail read now pages through the full table instead of only its first page
- Added Critical/Warning/All reset buttons on the Audit Trail screen - the Cockpit's Critical and Warnings header KPIs now show only new events since the last reset instead of the all-time total, and increase again from 0 as new Critical/Warning events occur

### v1.22.7 - 2026-09-02

- Fixed the Automation Status External Domains (and Counter) row count not updating - the periodic/manual refresh logic was still only refreshing the 3 original SharePoint data sources (Agent Status, Configuration, Internal Domains) and never refreshed the two newly added 'DMP Command Counters'/'DMP Command External Domains' lists, so the client-side row counts stayed stuck at whatever was cached on first load

### v1.22.6 - 2026-09-02

- Fixed the Emails Processed legend popup formatting - counts now show with a thousands separator (e.g. 1,000) and percentages with two decimals (e.g. 12.34%), as '<Label> (<Count> / <Percentage>)', instead of the previous unformatted raw count and rounded whole-percent display

### v1.22.5 - 2026-09-02

- Fixed the Emails Processed counter not updating - Agent 4 was still reading Counter.xlsx and External_Domains.txt instead of the new SharePoint lists after yesterday's migration, so the Cockpit kept showing frozen values
- Counter and External Domains moved out of the FILES container into AUTOMATION STATUS, since both are now SharePoint lists rather than files
- Maintenance Domains View/Edit buttons for External Domains now open the SharePoint list instead of the old, no-longer-updated text file

### v1.22.4 - 2026-09-02

- Fixed the Operating State card - the 'LAST CHANGED' label was clipped because it had too little width before the value column started right next to it - widened the label and shifted the value, both toggle switches and their status LEDs further right to make room

### v1.22.3 - 2026-09-01

- Fixed another Arial-migration text-truncation regression - the 'AGENTS ACTIVE' header KPI label and the Files-panel entry names (Emergency Report, External Domains, Counter File, Audit Trail File) were being clipped with an ellipsis - reduced their font size slightly so the full text always fits
- Fixed Audit Trail (Detail) screen showing stale Recent Critical/Warning rows - the periodic data-refresh timer only runs while the Cockpit screen is active, so the Audit Trail screen now fetches fresh status data itself every time it is opened

### v1.22.2 - 2026-09-01

- Fixed a text-wrap regression in the Maintenance Domains 'Replace' button, caused by the v1.22.0 Arial font migration (Arial is wider than Segoe UI at the same point size, so 'Replace' no longer fit on one line) - added an explicit no-wrap setting and widened the button slightly
- Internal/External Domains View, Edit and Replace buttons shifted slightly further left for a bit more breathing room from the card's right edge

### v1.22.1 - 2026-09-01

- Header and Next Steps container width now bound directly to the actual rendered width of the Cockpit's card row, instead of a separate hardcoded number - guarantees their right edges always line up exactly with the cards, regardless of any future width tuning
- Header KPI numbers (Critical/Warnings/Agents Active), the version badge and the Dark/Light toggle shifted further left for a safer margin from the header's rounded right corner
- Periodic auto-refresh timer split into its own dedicated timer (same pattern as the earlier Replace fix) - fixes the LED staying solid yellow instead of blinking during a periodic refresh
- Next Steps status pill widened to actually fill its row instead of stopping halfway

### v1.22.0 - 2026-09-01

- Typography aligned with the official Eurex Brand Manual (page 39/40) - the manual itself specifies Arial as the correct system-compatible substitute for its Replica/Noto Sans brand fonts on platforms without embedded custom fonts, which is exactly our situation (a browser-based Power App)
- Every text across all 9 screens switched from Segoe UI to Arial (367 formula updates), a purely visual change - no layout, colors, or functionality affected
- All future new text formulas should use Font.'Arial' going forward

### v1.21.0 - 2026-09-01

- Refresh indicator (gray on very first load, yellow blinking on every later refresh) extended to all reloading displays across the app - the System Health legend popup dots and all 6 Agent Monitoring status dots now follow the same convention as the Files panel and Automation Status
- Emails Processed colors reassigned to match the requested convention - Blue=No DMP, Green=Not Effected, Purple=Internal, Red=External
- Maintenance Domains - all View/Edit/Replace buttons shifted slightly left so Replace no longer sits almost flush with the card's right edge
- Audit Trail - Open full Audit Trail file button made smaller (less empty space around the text)
- Note - the Audit Trail button still opens the raw Excel file (triggers a browser Save dialog); switching it to a proper SharePoint view will happen together with the already-planned Audit Trail SharePoint migration, not as a standalone guess

### v1.20.0 - 2026-09-01

- Help / Operational Manual redesigned with the same version-menu pattern as Release Notes - a small section menu on the left (About this manual, Colour/LED legend, Top bar, Operating State, Maintenance Domains, Files, System Health, Emails Processed, Automation Status, Next Steps, Sidebar navigation), one large readable container on the right
- The right side is now split in two - a screenshot area on the left (currently a placeholder until real screenshots are provided) and the description text on the right, as requested
- No content was changed, only the layout - all existing section texts are unchanged

### v1.19.0 - 2026-09-01

- Release Notes redesigned - a small version menu now sits on the left (click any version to view it), with one large, fully readable container on the right showing the selected version's full description, for both the App Changes and Agent (Flow) Changes tabs
- Fixed the Copy to Clipboard button, which had not been updated since around v1.11.0 and was silently missing nine newer versions - it now always copies the complete history of the currently selected tab
- The previous stacked-cards layout still exists underneath (hidden) as the data source for the new menu - no release history was lost in the redesign

### v1.18.0 - 2026-09-01

- LED refresh behaviour further refined per feedback - gray only on the very first app load, yellow blinking on every later refresh (kept from v1.17.1)
- Colors aligned with the official Eurex Brand Manual - the Emails Processed donut now uses the official secondary/tertiary colors (Light Blue, Purple, Aqua Mint, Peach) instead of two non-brand shades
- The DMP/SIMU (Operating State) toggle's active color, its status text, and the four screen-edge frame bars now use Eurex Purple for the SIMU+DMP combination instead of a non-brand orange
- Genuine danger/safety colors (PROD environment, PROD+DMP combination) intentionally left as-is - the Eurex palette has no red, and a universal danger signal was judged more important than brand purity there
- All future color and text/typography decisions will reference the official Eurex Brand Manual document

### v1.17.1 - 2026-09-01

- Hotfix for a v1.17.0 import error (PA1001, YamlInvalidSyntax) - two lines in scrHome.pa.yaml had drifted one space out of indentation during an earlier automated edit, which pac canvas pack does not catch but Studio's import does
- LED refresh indicator refined per feedback - LEDs now show gray only during the very first app load (before any data has ever arrived); on every later refresh they blink yellow instead of turning gray, keeping the last known Green/Red status visible in between blinks
- No other visual changes in this release

### v1.17.0 - 2026-09-01

- Fixed Timer/LED freeze during Replace - the file-replace flow call ran inside the same 1-second ticking timer as the auto-refresh, blocking its blink-phase and countdown for the whole call. Moved to its own dedicated timer
- Automation Status column labels widened further, Agent 4 Live Status name no longer clips
- Agent Monitoring now force-refreshes the Agent Status SharePoint list every time the screen is opened, so changes made by a Replace or any agent run show up immediately instead of a stale cached copy
- Now button and the periodic auto-refresh timer now also force-refresh the Agent Status, Configuration and Internal Domains SharePoint lists (previously only Agent Monitoring's own screen open did this)
- System Health and Emails Processed wheels show REFRESHING... while a refresh is running
- All Files-panel and Automation Status LEDs now turn gray while refreshing, for a consistent refreshing indicator
- Admin Functions - Delete button moved below the text and left-aligned like No DMP, text shortened to Delete, section headers restyled to match the Cockpit style (green, uppercase)

### v1.16.1 - 2026-09-01

- Hotfix for a v1.16.0 regression - switching the SIMU/DMP or PROD/SIMU toggle no longer called its flow and the LED stayed green
- Root cause - the new background Reset() calls (Now button, periodic auto-update) ran unconditionally, even while a real toggle click's own flow call was still in flight - this could yank the toggle back to its old value mid-click and made the click appear to do nothing
- Fix - those background Reset() calls now only run when no toggle click is currently being processed
- No visual changes in this release

### v1.16.0 - 2026-09-02

- Fixed Operating State inconsistency - the header text and the Mode/Application toggles could show contradicting values (e.g. header "SIMULATION" while toggle showed "Normal"). Root cause - only the initial screen load resynced the toggles from the refreshed data; clicking "Now" or waiting for the periodic auto-update refreshed the underlying values but never told the toggle controls to redraw, since a Toggle's Default property only applies on first render
- Added an explicit Reset() of both toggles at all four places the operating mode is refreshed (initial load, kickstart timer, Now button, periodic auto-update) so the toggles always redraw in sync with the header
- No other visual changes in this release

### v1.15.0 - 2026-09-01

- Timer stability fix - removed a redundant manual Reset() on the auto-refresh timer that was fighting with the timer's own AutoStart mechanism, likely causing it to stop and not restart after navigating away from the Cockpit (e.g. clicking a KPI tile)
- Maintenance Domains - widened the count numbers further (still clipping to "1..'' for two-digit counts)
- Automation Status - widened the Agent 4 Live Status name column (was clipping to "Agent 4 Live Stat…")
- Admin Functions - widened the mailbox cleanup description so it no longer clips

### v1.14.0 - 2026-09-01

- Debug status bar turned back off (served its purpose - confirmed no Agent 4 connection error)
- Release Notes - "Agent (Flow) Changes" tab header no longer wraps; fuller version history added for Agents 1/3/5
- Maintenance Domains - INTERNAL/EXTERNAL labels shifted right so the count numbers display correctly again
- Automation Status redesigned to match the Files panel style (SOURCE/ACTIVE columns instead of one long line per item)
- Audit Trail (Detail) - fixed the clipped "Warnings" label and the wrapping "Open full Audit Trail file" button
- Admin Functions - removed Display Diagnostics (rarely needed, added clutter); Mailbox cleanup button now left-aligned next to its label instead of pushed to the far right

### v1.13.1 - 2026-09-01

- Hotfix - v1.13.0 could not be opened in Studio (a "Wrap" setting was mistakenly added to several buttons, which only Labels support, plus one line had drifted out of its correct indentation) - both fixed, no visible/functional change
- Audit Trail (Detail) page filled in - shows the 10 most recent Critical and 10 most recent Warning rows read live from the central Audit Trail, plus a link to open the full file (Agent 4 v1.4.0 now also reads and returns these rows)

### v1.13.0 - 2026-09-01

- Debug status bar temporarily made visible again on the Cockpit to help diagnose a reported refresh issue - please report back what it shows
- Sidebar - "Admin Functions (Test Only)" renamed to plain "Admin Functions"
- Maintenance Domains - the separate blinking LED next to Replace was removed; the leftmost status dot on the External row now does the slow blink itself while a replacement upload is processing
- Maintenance Domains - Internal/External rows reflowed with more room for the INTERNAL/EXTERNAL labels so they no longer get cut off with "..."
- Agent Monitoring - fixed the green agent name headings having their descenders (e.g. the letter g) clipped off
- Help / Operational Manual - added a proper scrollbar to the section list (it was missing an explicit height, so long pages were not fully readable)
- Maintenance page - fixed clipped description text in Connection Diagnostics and Versions; the app version is now a clickable link to Release Notes; Admin portal link buttons no longer wrap their text
- Admin Functions - each action (mailbox cleanup, counter reset) now has its own bordered sub-section; Display Diagnostics made smaller and less prominent; the "Internal Sender" reset button no longer wraps its text; remaining "Test"/"Test Only" wording removed
- Cockpit - clicking the CRITICAL or WARNINGS KPI number/label now also opens Audit Trail (Detail); clicking AGENTS ACTIVE now also opens Agent Monitoring - both still acknowledge the change-notification LED as before

### v1.12.1 - 2026-09-01

- Hotfix - v1.12.0 could not be opened or published in Studio (YAML parsing error on 3 new labels that had a colon immediately followed by a space inside the quoted text, e.g. the words "Active rows" followed directly by a colon and a space). Rewritten so the colon is now the very last character of its own quoted segment - parses cleanly, no visible or functional change.

### v1.12.0 - 2026-09-01

- Help / Operational Manual page filled in - a full container-by-container explanation of the Cockpit (what every tile shows, what every button does, and what the colours/LEDs mean), reachable from the sidebar
- Sidebar renamed "Operational Board" to "Help / Operational Manual"
- Configuration (Lists) page filled in - direct View/New links and live active-row counts for all 3 SharePoint configuration lists
- Maintenance page filled in - live connection diagnostics for the 3 SharePoint lists, app/agent version overview, and quick links to the Power Automate, SharePoint and Power Apps maker portals
- Admin Functions - added counter reset actions (No DMP / Internal Sender / External / Not Effected / Reset ALL), each with a confirmation step; the previous value is written to the Audit Trail as an Operational History record before the reset (Agent 6 v1.2.0)

### v1.11.0 - 2026-09-01

- New - direct SharePoint connections added for "DMP Command Configuration", "DMP Command Agent Status" and the brand-new "DMP Command Internal Domains" list (Title/Active columns)
- Internal Domains management now reads/links directly to the new SharePoint list (count + View/Edit buttons) instead of the old flat text file - backend agents still write the text file for now, that migration is planned separately
- Automation Status redesigned as a compact 4-line panel (Agent 4 Live Status, Internal Domains active count, Config Parameters active count, Agent Status entries count), moved into the former "Reserved for future use" tile
- Removed the old separate Automation Status checklist section (generic manual next-step items) to make room - see backlog for the removed items
- All container gaps unified to a single, minimal 8px spacing (previously a mix of 8/16px)
- Maintenance Domains - removed the extra dashes around INTERNAL/EXTERNAL, widened the labels, narrowed the card to match Operating State's width
- Files card narrowed further, tighter column spacing
- System Health / Emails Processed rings resized so their combined card height lines up exactly with Operating State + Maintenance Domains
- System Health ring - the plain-text tooltip was unreliable (sometimes showed nothing at all), replaced with the same click-to-open colour legend popup already used on Emails Processed
- Release Notes list scrollbar fix - it needed an explicit calculated height, not just automatic space-sharing, to actually start scrolling
- Fixed a recurring backend issue where the Agent 4 connection had to be manually deleted and re-added after almost every deployment - traced to a stale internal reference that is now kept in sync
- v1.10.1 - Release Notes cards still cut off long text - AutoHeight alone was not reliable, so each card now has its own fixed max height with a scrollbar (in addition to the outer list scrollbar)
- v1.10.1 - Automation Status - fixed the third bullet (Config Parameters) showing no text at all - the Active column in DMP Command Configuration is a Choice field, needed .Value
- v1.10.1 - Automation Status tile border changed to solid green to match all other Cockpit cards (was still the old dashed placeholder style)
- v1.10.1 - Agent 6 (Admin Functions) was stuck in Draft/Inactive state in Dataverse, which made it show up oddly in the Power Automate panel - activated
- v1.10.2 - Release Notes now also has an "Agent (Flow) Changes" tab alongside "App Changes", listing each agent's current version and recent changes
- v1.10.2 - Replace button LED - the "processing" yellow blink never actually rendered because the upload and the flow call ran in the same formula with no repaint in between; the flow call now runs a moment later via a timer so the blink is visible
- v1.10.2 - All green status dots across the Cockpit (Files, Automation Status, System Health legend, Maintenance Domains) are now the same size and shape (14px circles) - previously a mix of text bullets and different-radius shapes
- v1.10.2 - Maintenance Domains - added a status dot in front of the count, matching the Files/Automation Status pattern
- v1.10.2 - Maintenance Domains - Edit now opens a quick "add new domain" form instead of duplicating the View link
- v1.10.2 - Header bar width aligned with the card row and Next Steps width for a clean edge-to-edge look
- v1.11.0 - Agent Monitoring page filled in - 6 tiles (one per agent) showing live status, last run result/duration, and status message straight from the DMP Command Agent Status SharePoint list
- v1.11.0 - Fixed System Health and Emails Processed tooltips/popups only showing one legend entry instead of all of them - the popup rows were missing an explicit setting that told them to keep their own height instead of sharing space with their siblings

### v1.9.3 - 2026-08-28

- Files and Maintenance Domains swapped positions (Files now top-right, Maintenance Domains bottom-left)
- Files - removed the Internal Domains row (redundant with Maintenance Domains), widened Name/Status/Last Updated columns to stop text truncation
- Maintenance Domains - compact new layout - count (green) - LOCATION - actions, e.g. "19 - INTERNAL - View Edit" and "6 - EXTERNAL - View Edit Replace"
- Operating State - "Operational Mode" caption shortened to "MODE"
- Change-notification LEDs (Critical/Warnings/Agents Active) - added a large invisible click area covering the whole tile including the LED itself - previously only the small number/label caught the click, making the LED very hard to tap
- v1.9.1 - added the missing second dash (between location and actions) to the Maintenance Domains format
- v1.9.2 - enlarged the invisible click area of the Replace button (Maintenance Domains - External) - was only slightly larger than the visible button, now has a generous margin on all sides without overlapping the neighboring Edit button or LED
- v1.9.3 - Release Notes text was getting cut off on longer entries - all entries now auto-size to their real content height
- v1.9.3 - Release Notes list is now scrollable, and a new "Copy to Clipboard" button copies the full version history as plain text
- v1.9.3 - Files card - right edge was poking out past its container on some screens - narrowed the card and tightened the Status/Last Updated column gaps further
- v1.9.3 - Heartbeat and Emails Processed ring cards - reduced the inner padding further to save horizontal space
- v1.9.3 - Emails Processed ring - the old plain-text tooltip could not show colours, so it was replaced with a click-to-open legend popup with real colour swatches matching each ring segment

### v1.8.1 - 2026-08-28

- Sidebar logo fixed - the source image had ~20-25% built-in padding on every side, so no resize ever actually made it bigger; it is now cropped to the real artwork and fills the sidebar width
- Major width reduction across the whole Cockpit header row to fit 100% browser zoom (was previously only fully visible at 75%) - narrower Operating State / Maintenance Domains / Files columns, narrower buttons, smaller ring padding, tighter gaps, narrower sidebar
- Operating State - "Mode" merged into the title area (color-coded like the screen border, blinks while switching), all labels made smaller/consistent, matching the Files card style
- Emails card - legend converted to a tooltip on the ring itself, so the ring can stay full size while the card gets much narrower
- Light mode - several too-light grey texts (Files status/time, Maintenance Domains counts) darkened for readability
- New diagnostics tile on the Admin page showing App.Width/App.Height for reporting layout issues
- v1.8.1 - fixed a sibling-indentation bug in this very file that blocked Studio from opening v1.8.0

### v1.7.0 - 2026-08-28

- Every sidebar menu item now has its own real screen instead of a "coming next" popup - Admin Functions moved to its own page, and Agent Monitoring / Operational Board / Audit Trail (Detail) / Configuration (Lists) / Maintenance each got a first "Coming Soon" placeholder screen (content to be filled in step by step)
- New change-notification LEDs on the Critical / Warnings / Agents Active header tiles - a small blue LED lights up (blinking) whenever one of these values changes since you last acknowledged it - click the tile to acknowledge; a new warning or critical lights the LED again automatically
- Version badge (top right) now links to this Release Notes page

### v1.6.4 - 2026-08-28

Frontend:
- Sidebar logo enlarged (280x188 -> 288x194 px), sidebar padding reduced
- Emails card (Agent 2 status) narrowed from 620 to 560px
- Next Steps - milestone text line-wrap fixed
- Display glitch during loading (garbled characters) fixed
- Reverted an over-aggressive container shrink that had truncated labels

Backend:
- New Error-ID system (e.g. [EC:A2-MBOXFOLDER-ND]) added across all 5 agents for faster troubleshooting
- Agent 1 & 2 performance improved (Excel counters now batched instead of updated per email)
- False "folder already exists" alarm fixed in Agent 3 & 4
- Copy-paste bug in Agent 1's error-mail move action fixed

### v1.6.1 - v1.6.3 - 2026-08-27/28

- Emails card visibility issue fixed (layout width)
- Several intermediate width-tuning iterations (superseded by v1.6.4)

### v1.5.x and earlier

- Older changes are documented in detail in the backlog document (DMP_COMMAND_Backlog.md), including the timestamp-format fix and earlier layout adjustments
- Starting with v1.6.4, this Release Notes page is updated with every new version

## Agent (Flow) Changes

### Agent 7 (Streams & Milestone Management) - v0.3.1 (current Dev component name)

- Solution `7.34.46` (2026-09-22) - **Versioning correction, no functional flow change beyond
  `7.11.48`.** Established that the Solution version is a checksum (Σ Major, Σ Minor, Σ Patch
  across all 7 agent components + the Power App, 8 components total), not a free-running
  build counter. Corrected Agent 7's own component label from the stale `[0.2.0]` (unchanged
  across all `7.11.36`–`7.11.48` iterations) to `[0.3.1]`: `0.3.0` retroactively covers the
  `RequestedAction`/`FromDate`-`ToDate` contract rewrite (`7.11.47`), `0.3.1` covers the
  hardening (`7.11.48`, see below). Recomputed Solution version from all current component
  versions (Agents 1-6 unchanged, Agent 7 `0.3.1`, Power App `v1.22.13`): Σ Major=7, Σ
  Minor=34, Σ Patch=46 → `7.34.46`. See `DMP COMMAND_Mission_und_KI_Arbeitsregeln.md` section
  I for the exact formula; recompute this checksum on every future component version change.
- Solution `7.11.48` (2026-09-22) - Deployed the hardening below to Dev: rule-level
  `TimeOfDay`/`TimeZone` existence validation per Daily rule (`CHECK_RuleScheduleFieldsValid`,
  skips gracefully instead of failing the whole run in `convertTimeZone`); a real per-item
  error counter via a `Scope`/catch pattern around `CREATE_Occurrence`
  (`SCOPE_CreateOccurrence` + Failed/TimedOut handler); a temporary placeholder validation
  rejecting a blank/whitespace-only caller-supplied `CaseId` (`VALIDATE_CaseId`); and new
  `errorCount`/`errorDetails` fields on the `RESPOND_Result` output (`success` is now also
  `false` when `ErrorCount > 0`). Imported via `pac solution import --publish-changes` and
  published successfully in `DBG Team Productivity (Dev)`. Power Automate reported "The
  original workflow definition has been deactivated and replaced" (expected); the user must
  reopen, save, and reactivate Agent 7 in the designer before it can run again — unlike
  `7.11.47`, this version has real content changes, so Save will not be greyed out.
- Solution `7.11.47` (2026-09-22) - Implemented the `RequestedAction` contract:
  `CreateTaskOccurrences` (single `BusinessDate`) and `GenerateMissingOccurrences`
  (inclusive `FromDate`/`ToDate` multi-day expansion). Removed the obsolete
  `OccurrencesJson` trigger input; the trigger now exposes `InitiatedBy`,
  `RequestedAction`, `OperatingMode`, `CaseId`, `FromDate`, `BusinessDate`, `ToDate`.
  Added explicit validation (unsupported action, missing/out-of-order dates) returned as
  `validationError`/`message`, and reliable `createdCount`/`skippedCount` response
  counters. The rule-application loop now nests per validated date
  (`FOREACH_Dates` &gt; `APPLY_Rules`) instead of a single `BusinessDate`. Not yet
  imported-and-activated-confirmed by the user in the designer; not yet run end-to-end
  against real data.
- Solution `7.11.46` (2026-09-21) - Agent 7 now opens in the new designer and was
  user-confirmed as saveable and activatable in `DBG Team Productivity (Dev)`. Corrected the
  Task Occurrences `ScheduleSlot` Create Item mapping from invalid plain-text
  `item/ScheduleSlot` to Choice binding `item/ScheduleSlot/Value`.
- Solution `7.11.41`–`7.11.45` - isolated the misleading `$schema` designer failure through
  controlled deployments: minimal Power Apps V2 flow, SharePoint read, DMP Condition,
  recurrence-rule loop, OccurrenceId/duplicate lookup, then Create Item. The first four
  stages loaded; adding Create Item exposed the actual
  `OpenApiOperationParameterValidationFailed` error for `ScheduleSlot`.
- Corrected all Agent 7 SharePoint datasets to
  `https://deutscheboerse.sharepoint.com/teams/GO365_DMPCommunication-CoSLeader`.
- Current partial C5 behavior: reads active Daily recurrence rules for one `BusinessDate`,
  generates the canonical OccurrenceId, checks exact existence, and creates only missing
  Task Occurrences with UTC due time. No real-row end-to-end test was performed yet.
- Still pending: final `BusinessDate`/`FromDate`/`ToDate` contract, inclusive date-range
  generation, active-case/mode validation, response counters, live Power App binding, and
  Dev idempotency test.

### Agent 2 (E-Mail Inbox Treatment) - v1.0.10 (current)

- v1.0.10 (2026-09-23) - Configuration value lookup (`Select_ConfigEntries`) now also handles `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/`SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT` (introduced by the 5-mode switch but not yet wired here - previously fell through to the `SIMU_DMP` column); uses the confirmed real SharePoint internal field names for the 4 new Configuration value columns. Trigger: explicit `maximumWaitingRuns=100` added (previously implicit default 10); degree of parallelism stays at 1 (sequential) to protect the shared counter increment from race conditions.
- v1.0.9 (2026-09-04) - Hotfix for the invalid inline `@select(...)` Power-Automate expressions in internal/external domains classification. Replaced both parse actions with real Query/Filter-array operations and added a mandatory critical technical main-flow failure alert e-mail `[EC:A2-MAINFLOW-FAILED]`. Solution 7.11.35 imported and published in DBG Team Productivity (Dev).
- v1.0.8 (2026-09-02) - Counter increment (all 4 workflow paths) and external domains read now use the "DMP Command Counters" and "DMP Command External Domains" SharePoint lists instead of Counter.xlsx and External_Domains.txt. Follow-up finding on 2026-09-04: the attempted runtime-crash fix for "Parse internal domains file + create array" is still faulty because `select()` is not a valid inline Power-Automate template function in this context; Agent 2 needs a follow-up flow fix using a real Data Operation Select or equivalent non-`select()` expression logic.
- v1.0.7 (2026-09-01) - Internal sender classification now reads the "DMP Command Internal Domains" SharePoint list (Active = Yes) instead of the flat Internal_Domains.txt file - one fewer SharePoint call per e-mail
- v1.0.6 and earlier - audit counter writes batched (one combined read/write per run instead of per event), retry policies added to the critical Excel calls, error-ID codes ([EC:A2-...]) added for faster troubleshooting

### Agent 4 (Status Check) - v1.4.5 (current)

- v1.4.5 (2026-09-23) - Configuration value lookup (`Select_ConfigEntries`) now also handles `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/`SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT` (previously fell through to the `SIMU_DMP` column); uses the confirmed real SharePoint internal field names for the 4 new Configuration value columns.
- v1.4.4 (2026-09-04) - Fixed 2 separate bugs found via the real flow run history: (1) the Audit Trail read now requests dateTimeFormat=ISO 8601 like every write action already does, instead of returning raw Excel serial numbers for TimestampUtc; (2) replaced 6 uses of the non-existent template function filter() (confirmed invalid via the official Workflow Definition Language reference) in the Counter card and Critical/Warning baseline reads with proper Query actions - this was silently breaking the No DMP/Internal Sender/Not Effected/Effected counters and both baseline reads
- v1.4.3 (2026-09-03) - Fixed the Internal/External Domains and Counter "Last Updated" fields silently failing every run ("The template function 'select' is not defined or not valid") - replaced the select()/max() expression with a simple $orderby=Modified desc on the existing list read, no extra API call needed
- v1.4.2 (2026-09-02) - Enabled pagination on the Audit Trail read so the Recent Critical/Warning lists always reflect the true last 10 rows, not just rows within the table's first page - added CriticalCounterBaseline/WarningCounterBaseline to the response, read from the same Counters list already loaded for the Counter card, used by the Cockpit's new Critical/Warning reset feature
- v1.4.1 (2026-09-02) - Counter and External Domains status checks now read the "DMP Command Counters" and "DMP Command External Domains" SharePoint lists instead of Counter.xlsx and External_Domains.txt - fixes the Cockpit's Emails Processed counter and External Domains status not updating after Agent 1/2 were migrated to the new lists
- v1.4.0 - now also reads the central Audit Trail table directly and returns the 20 most recent Critical (Failed) and Warning rows, used by the new Audit Trail (Detail) Cockpit page
- v1.3.0 (2026-09-01) - Internal Domains status check now reads the "DMP Command Internal Domains" SharePoint list (Active = Yes) instead of the flat Internal_Domains.txt file - same output fields (Exists/Count/LastModified) as before

### Agent 1 (Domains Extraction) - v1.0.9 (current)

- v1.0.9 (2026-09-23) - Configuration value lookup (`Select_ConfigEntries`) now also handles `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/`SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT` (previously fell through to the `SIMU_DMP` column); uses the confirmed real SharePoint internal field names for the 4 new Configuration value columns.
- v1.0.8 (2026-09-02) - External domains write now uses a full-sync rewrite (delete all rows, then create one row per extracted domain) against the "DMP Command External Domains" SharePoint list instead of writing External_Domains.txt
- Audit counter writes batched (one combined read/write per run instead of per event), error-ID codes ([EC:A1-...]) added for faster troubleshooting
- 2026-08-24 - Agent Audit Summary per-outcome step/run counters added (feeds the Cockpit's Critical/Warning/Total-runs figures via Agent 4)
- 2026-08-13 - renumbered from "Agent 1" (unchanged number, but part of the system-wide sequential renumbering and Select+Join config-loading rebuild applied to all agents)

### Agent 3 (Emergency Report Management) - v1.1.5 (current)

- v1.1.5 (2026-09-23) - Configuration value lookup (`Select_ConfigEntries`) now also handles `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/`SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT` (previously fell through to the `SIMU_DMP` column); uses the confirmed real SharePoint internal field names for the 4 new Configuration value columns.
- Handles Emergency Report uploads triggered from the Cockpit's Replace button, then regenerates External Domains
- 2026-08-24 - Agent Audit Summary per-outcome step/run counters added
- 2026-08-14 - alert-mail-then-move-to-folder error pattern added (matches Agents 1/2)
- 2026-08-13 - renamed from "Agent 3.01" as part of the system-wide sequential renumbering

### Agent 5 (Operational State Management) - v1.1.7 (current)

- v1.1.7 (2026-09-23) - Configuration value lookup (`Select_ConfigEntries`) now also handles `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/`SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT` (previously fell through to the `SIMU_DMP` column, e.g. wrong mail-mode texts while in Pre-/Post-Default); uses the confirmed real SharePoint internal field names for the 4 new Configuration value columns.
- Handles the Operating State toggle (Normal / DMP Operation) triggered from the Cockpit
- 2026-08-24 - Agent Audit Summary per-outcome step/run counters added
- 2026-08-14 - alert-mail-then-move-to-folder error pattern added (matches Agents 1/2/3)
- 2026-08-13 - renamed from "Agent 3.03 (YES File Management)" - the legacy Yes.txt file mechanism was fully decommissioned in favour of the CurrentOperationMode config value

### Agent 6 (Admin Functions) - v1.3.2 (current)

- v1.3.2 (2026-09-23) - Configuration value lookup (`Select_ConfigEntries`) now also handles `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/`SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT` (previously fell through to the `SIMU_DMP` column); uses the confirmed real SharePoint internal field names for the 4 new Configuration value columns.
- v1.3.1 (2026-09-04) - Fixed the Reset All action's Critical/Warning baseline row lookup, which used the non-existent template function filter() (confirmed invalid via the official Workflow Definition Language reference) and silently failed - replaced with a proper Query action, same pattern as the individual Reset Critical/Reset Warning actions already used
- v1.3.0 (2026-09-02) - added Critical/Warning counter reset actions for the Cockpit's Audit Trail screen - each captures the current lifetime Critical/Warning total (from Agent Audit Summary) as a new baseline in the DMP Command Counters SharePoint list, so the Cockpit's KPI shows only new events since the reset
- v1.2.0 - added counter reset actions (No DMP / Internal Sender / External / Not Effected / Reset ALL) - each writes the previous value to the Audit Trail as an Operational History record before resetting to 0

v1.5.x & older
