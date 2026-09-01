# DMP COMMAND — Operations Manual

**Scope:** This manual covers the complete DMP COMMAND system: all 6 backend agents (Power Automate flows), the DMP COMMAND Power App (Cockpit GUI, now with 7 screens - Cockpit, Agent Monitoring, Help / Operational Manual, Audit Trail (Detail), Configuration (Lists), Maintenance, Admin Functions), the central configuration list, and standard operating procedures including the Fire Drill / Emergency procedure.

**Audience:** Operations team members responsible for running, monitoring, and troubleshooting DMP COMMAND day-to-day.

**Related documents:**
- `Documentation/Backlog/DMP_COMMAND_Backlog.md` — technical backlog, architecture decisions, and detailed findings (developer-facing, German)
- `Documentation/DMP Command Configuration.csv` — periodic export of the live SharePoint configuration list
- `Documentation/DMP Command Agent Status.csv` — periodic export of the live SharePoint agent status list
- `README.md` (repository root) — developer/ALM setup (pac CLI workflow, repository structure)

---

## 1. System Overview

DMP COMMAND automates the handling of DMP (Disaster/Major Peril) communication events for Eurex. The system consists of:

- **6 Power Automate flows ("Agents")** that extract domain data, classify inbound e-mails, manage emergency report ingestion, check system status, control the operational mode, and (as of Agent 6) perform administrative test/reset actions.
- **Three central SharePoint lists** that are the single source of truth for all shared data: `DMP Command Configuration` (all operational parameters - no agent uses hardcoded values for anything that varies by environment or mode), `DMP Command Agent Status` (fast, pre-computed per-agent snapshot for the Power App dashboard), and `DMP Command Internal Domains` (migrated from a flat text file to a SharePoint list in 2026-09; Agent 2 and Agent 4 both read it directly, `Active` is a Choice column).
- **An audit trail** (`AuditTrail.xlsx`, SharePoint) that every agent appends structured, per-step audit events plus one run-summary row to, for compliance and troubleshooting. Agent 4 also reads this table directly to surface the most recent Critical/Warning rows in the Cockpit's Audit Trail (Detail) screen.
- **The DMP COMMAND Power App**, a multi-screen Cockpit used by operators to monitor agent health, view/edit domain lists, switch the operational mode, drill into audit history, and perform administrative resets - see §3 for the full screen-by-screen guide.

### 1.1 The Operating State model

The entire system's behavior is governed by **one configuration value**: `CurrentOperationMode`, which is always one of:

| Value | Meaning |
|---|---|
| `PROD_NODMP` | Production environment, normal (non-DMP) operation |
| `PROD_DMP` | Production environment, real DMP event in progress |
| `SIMU_NODMP` | Simulation/test environment, normal operation |
| `SIMU_DMP` | Simulation/test environment, DMP event simulation (e.g. Fire Drill) |

This single value is composed of **two independent dimensions**, represented in the Power App as two separate toggle switches:

1. **Environment**: `PROD` vs `SIMU`
2. **Operational Mode**: `Normal` vs `DMP`

Switching either toggle in the Power App's "Operating State" panel calls Agent 5 (Operational State Management), which writes the new combined value directly to `CurrentOperationMode` in the configuration list. All other agents read this value on every run to determine subject-line prefixes, mail wording, and DMP-specific branching — there is no separate "Fire Drill" file or mechanism; switching `CurrentOperationMode` to a `_DMP` value **is** the Fire Drill / real-DMP declaration.

### 1.2 Agent naming convention

As of 2026-08-13, agents use sequential numbering (previously Agent 1, Agent 2, Agent 3.01, Agent 3.02, Agent 3.03). Agent 6 (Admin Functions) was added later in 2026-08/09 as the system's 6th agent:

| Current Name | Previous Name | Purpose |
|---|---|---|
| Agent 1 (Domains Extraction) | Agent 1 | Extracts internal/external domain lists from the Emergency Report |
| Agent 2 (E-Mail Inbox Treatment) | Agent 2 | Classifies and routes inbound DMP mailbox e-mails |
| Agent 3 (Emergency Report Management) | Agent 3.01 | Ingests and validates new Emergency Report uploads |
| Agent 4 (Status Check) | Agent 3.02 | Reports system/file status and audit detail to the Power App |
| Agent 5 (Operational State Management) | Agent 3.03 (formerly "YES File Management") | Writes the Operating State toggle changes to central configuration |
| Agent 6 (Admin Functions) | *(new)* | Performs administrative test/reset actions triggered from the Cockpit's Admin Functions screen (mailbox cleanup, e-mail counter resets) |

---

## 2. Agent Reference

### 2.1 Agent 1 — Domains Extraction

**Purpose:** Extracts the current list of internal and external domains from the `Emergency Report.xlsx` worksheet ("Emergency Contacts") and writes them to `Internal_Domains.txt` / `External_Domains.txt` for use by Agent 2's e-mail classification logic.

**Trigger:** Manual/on-demand (Power Automate manual trigger), typically run after a new Emergency Report is uploaded (see Agent 3).

**Process:**
1. Loads all active configuration rows in one read (no per-agent `Scope` filter is applied at the flow level; the flow selects only the keys it needs after loading).
2. Reads the Emergency Report workbook, worksheet "Emergency Contacts" (configurable via `Agent1SourceWorksheetName`).
3. Extracts and de-duplicates domain values into an internal/external classification.
4. Writes `Internal_Domains.txt` and `External_Domains.txt` to their configured SharePoint storage locations.
5. Sends alert e-mails on failure conditions (missing worksheet, invalid workbook, invalid file extension) to `AlertEmailRecipient`.
6. Writes a run summary and per-step audit events to the Audit Trail, and updates its own row in the Agent Status list.

**Error handling:** On any failure branch (missing worksheet, invalid workbook/extension, write failure), Agent 1 sends an alert e-mail with `Importance = High` (via the central `MailImportanceError` setting) and records the failure in both the Audit Trail and the Agent Status list (`CurrentStatus = Failed`, `StatusSeverity = Critical`).

**Key configuration parameters:** `Agent1TriggerFolder`, `Agent1TriggerFileName`, `Agent1SourceWorksheetName`, `Agent1AlertFolderName`, `ExternalDomainsStorageFolder`, `ExternalDomainsFileName`, `WaitSecondsBeforeSentMailSearch`, `WorkflowPathAgent1`.

---

### 2.2 Agent 2 — E-Mail Inbox Treatment

**Purpose:** Monitors the shared DMP mailbox, classifies each inbound e-mail by sender domain against the Internal/External domain lists, and routes it into one of four categories:

| Category | Meaning |
|---|---|
| **No DMP** | Sender not matched to a monitored domain — no DMP relevance |
| **DEE** (DMP External Effected) | External sender, domain matched, DMP-relevant |
| **DIS** (DMP Internal Sender) | Internal sender, DMP-relevant |
| **DNES** (DMP Not Effected Sender) | Sender matched but classified as not affected |

**Trigger:** Runs per inbound e-mail in the shared DMP mailbox (event-driven / polling trigger on the mailbox).

**Process:**
1. Loads active configuration (Select+Join pattern, ~3 actions instead of the historical 212-action per-row loop).
2. Reads the e-mail's sender domain and matches it against the Internal Domains list (SharePoint list `DMP Command Internal Domains`, `Active` Choice column - migrated from a flat text file in 2026-09) and the External Domains list (still a flat text file, `External_Domains.txt`).
3. Moves the e-mail to the corresponding processed-mail subfolder (`No DMP`, `DEE`, `DIS`, `DNES`).
4. Increments the corresponding counter in the central counter file/table (`CounterPathNoDMP`, `CounterPathInternalSender`, `CounterPathNotEffected`, `CounterPathEffectedMember`).
5. Sends category-appropriate notification e-mails where configured (e.g. DIS info mail).
6. Buffers and writes audit events; updates the Agent Status row.

**Error handling:** Handles Graph API failures (mailbox folder resolution), audit write failures, and missing-counter-file conditions as distinct error categories, each with its own alert e-mail and audit/status entry. E-mail importance is set from the central 3-tier scheme (see §5.2).

**Key configuration parameters:** `Agent2TriggerMailbox`, `Agent2AlertFolderName`, `CounterFileName`, `CounterFolder`, `CounterTableName`, `CounterPathNoDMP`/`CounterPathInternalSender`/`CounterPathNotEffected`/`CounterPathEffectedMember`, `ExternalDomainsFileName`/`ExternalDomainsStorageFolder`, `WorkflowPathAgent2`. (`InternalDomainsFileName`/`InternalDomainsStorageFolder` are no longer used by Agent 2 since the SharePoint list migration - the list itself is referenced directly, not via these path parameters.)

---

### 2.3 Agent 3 — Emergency Report Management

**Purpose:** Accepts a newly-uploaded Emergency Report file (`.xlsx`) from the Power App's "Replace" control, validates it, stores it in the configured location, and signals that Agent 1 should be run to regenerate the domain lists.

**Trigger:** Called directly by the Power App (`'DMPAgent3(EmergencyReportManagement)'.Run(User().Email, {file: {...}})`) when a user uploads a new Emergency Report via the Maintenance-Domains "Replace" control on the Cockpit screen.

**Process:**
1. Validates the file extension (`.xlsx` only — other types are rejected with an error notification in the app).
2. Stores the file to its configured SharePoint location (overwriting the previous version).
3. Buffers audit events and writes a run summary.
4. Updates its own Agent Status row.
5. Returns a structured response (`responsestatus`, `responsemessage`, `responsecode`) that the Power App displays via `Notify()`.

**Error handling:** Invalid file extensions are rejected client-side in the Power App before the flow is even called. Flow-side failures (write errors) are reported back via the structured response and logged to the audit trail.

**Key configuration parameters:** `EmergencyReportTargetFolder`, `EmergencyReportFileName`, `RequiredWorksheetName`, `SharedDMPMailbox`, `WorkflowPathAgent3`.

---

### 2.4 Agent 4 — Status Check

**Purpose:** Provides file-existence and count status for the Power App's Files band and related dashboard elements (Emergency Report, Internal/External Domains, Counter File, Audit Trail File), system-wide audit health figures (failed/warning step counts, total run count across all agents), and (as of v1.4.0) the actual most-recent Critical/Warning audit rows for the Cockpit's Audit Trail (Detail) screen.

**Trigger:** Manual/on-demand, called directly by the Power App's `scrHome.OnVisible` (first load) and periodically by `tmrAutoRefreshTick` (`'DMPAgent4(StatusCheck)=>VS'.Run()`) to refresh the Cockpit dashboard.

**Process:**
1. Loads active configuration via the Select+Join pattern (same as Agents 1, 2, 3, 5).
2. Performs live SharePoint file/list-metadata and content checks for each monitored item (existence, last-modified timestamp, and for domain data, a count of entries). Internal Domains count/existence is read from the `DMP Command Internal Domains` SharePoint list (`Active` Choice column) rather than a text file.
3. Reads the shared `Agent Audit Summary` table (see §5.1) for all 6 agent rows and aggregates `AuditFailedCount` (sum of all `FailedStepsCount`), `AuditWarningCount` (sum of all `WarningStepsCount`), and `AuditRunSummaryCount` (sum of all Runs-counter columns across all 6 rows).
4. **(v1.4.0, 2026-09)** Reads the central Audit Trail table directly (same workbook/table Agent 6 writes run-summaries to) and returns the 20 most recent rows where `StepStatus = Failed` (Critical) and the 20 most recent rows where `StepStatus = Warning`, each projected to a compact `{timestamp, workflowpath, stepname, keyoutput}` shape.
5. Returns all of the above via a structured response to the Power App.

**Connector reference name (permanent, does not change on flow rename):** The Power App's underlying data source connector for this flow is named `'DMPAgent4(StatusCheck)=>VS'`. This internal Power Fx connector name is bound permanently to the flow's GUID at the time the connection was first added to the app - it does **not** change if the flow's display name or version is later edited, nor if the SharePoint connection is removed and re-added, nor by clearing browser cache (confirmed by direct testing in 2026-08). Do not attempt to "clean up" this name; it is cosmetic only and has no effect on functionality.

**Key configuration parameters:** File name/folder parameters shared with Agent 1/Agent 2/Agent 3 (`ExternalDomainsFileName`, `CounterFileName`, `AuditFileName`, `EmergencyReportFileName`, etc.), `WorkflowPathAgent4`, `Agent4AlertFolderName`.

---

### 2.5 Agent 5 — Operational State Management

**Purpose:** Writes the Operating State toggle changes made in the Power App back to the central configuration (`CurrentOperationMode`). This agent replaced the legacy `Yes.txt` file-based Fire Drill trigger mechanism.

**Trigger:** Called directly by the Power App (`'DMPAgent3(YESFileManagement)'.Run(User().Email, "<mode>")` — note: the underlying flow connector reference name in the app has not been renamed since the app has no direct rename mechanism for connector references; the flow itself is named "DMP Agent 5 (Operational State Management)") whenever either Operating State toggle (Environment or Operational Mode) is switched.

**Process:**
1. Loads active configuration via the Select+Join pattern (same as Agents 1–4).
2. Looks up the current `CurrentOperationMode` row.
3. Writes the new mode string (e.g. `"PROD_DMP"`) supplied by the app to that row's `CurrentValue`.
4. Buffers an audit event describing the switch (`"Switch from <old> to <new>"` or `"No change - requested operational mode already active"` if no actual change occurred, or `"No mode change w/failure"` if the write failed).
5. Writes a run summary to the Audit Trail.
6. Updates its own Agent Status row (`AgentDisplayName = "Operational State Management"`).
7. Returns `success`, `auditoutcome`, `requestedaction`, and `newoperationmode` to the app.

**Error handling:** If the SharePoint write fails, `AuditOutcome` is set to `Failed`, the Agent Status row's severity is set to `Critical`, and the Power App shows an error notification with the audit outcome.

**⚠️ Legacy Yes.txt mechanism — fully removed (2026-08-13):** This agent previously created/deleted a file called `Yes.txt` as the sole mechanism to signal a real DMP event to Agent 1. That mechanism has been completely decommissioned; Agent 1 (and all other agents) now derive DMP status exclusively from `CurrentOperationMode`. The `Yes.txt` file itself, if still present in SharePoint from historical use, is no longer read or written by anything and can be archived/removed at the operator's discretion.

**Key configuration parameters:** `WorkflowPathAgent5`, `AlertEmailRecipient`, `SharedDMPMailbox`.

---

### 2.6 Agent 6 — Admin Functions

**Purpose:** Performs administrative test/reset actions that are too risky or destructive to expose as ordinary automated steps - triggered exclusively from the Power App's **Admin Functions** screen, never automatically. Currently supports two categories of action:
1. **Mailbox cleanup** - deletes the `PA Processed Mails` folder tree (and any stray duplicates) from the shared mailbox, for resetting test data.
2. **E-mail counter reset** - resets any one (or all) of the 4 e-mail-classification counters (`No DMP`, `DMP internal Sender`, `DMP effected Member`, `DMP not effected Sender`) in the central Counter workbook back to 0.

**Trigger:** Called directly by the Power App (`'DMPAgent6(AdminFunctions)'.Run(User().Email, "<RequestedAction>")`) after the operator confirms a Yes/Cancel dialog in the Admin Functions screen. `RequestedAction` is one of: `DeleteProcessedMailsFolderTree`, `ResetCounter_NoDMP`, `ResetCounter_InternalSender`, `ResetCounter_Effected`, `ResetCounter_NotEffected`, `ResetAllCounters`.

**Process (counter reset, added 2026-09):**
1. Reads the current value of the targeted counter row(s) from the Counter workbook.
2. Resets the value(s) to 0.
3. The result message (including the previous value) is written to the central Audit Trail as a normal run-summary row (`StepName = AdminAction`) - this run-summary row **is** the historization/audit record of the reset; no separate archive file exists for this today (see backlog for a possible future dedicated "Operational History" archive).

**Error handling:** Any unrecognized `RequestedAction` value returns a clear "Unknown or unsupported admin action requested" message instead of silently doing nothing. All actions (successful or not) write a run-summary row to the Audit Trail and increment Agent 6's own row in `Agent Audit Summary`.

**Key configuration parameters:** Shares `SharedDMPMailbox`, `ProcessedMailsRootFolderName`, and the Counter-related parameters (`CounterFolder`, `CounterFileName`, `CounterTableName`, `CounterTableColumnNamePath`, `CounterTableColumnNameCounter`) with Agents 2/4.

---

## 3. Power App (Cockpit) — User Guide

The app has 7 screens, reachable from the left sidebar: **Cockpit** (home/dashboard), **Agent Monitoring**, **Help / Operational Manual**, **Audit Trail (Detail)**, **Configuration (Lists)**, **Maintenance**, and **Admin Functions**.

### 3.1 Cockpit (home screen) layout

- **Header**: application title, version tag, KPI strip (Critical / Warnings / Agents Active counts - each is clickable: Critical and Warnings jump to Audit Trail (Detail), Agents Active jumps to Agent Monitoring), auto-refresh interval selector (Off/Now/2m/5m/10m/15m) with a live countdown, Dark/Light mode toggle.
- **System Health** ring (left): a segmented ring showing the proportion of the 5 monitored files/lists that are OK vs. missing; click to open a legend popup with the exact per-item status.
- **Operating State** panel: current mode, last-changed timestamp/user, and the two Operating State toggles (Operational Mode: Normal/DMP; Environment: SIMU/PROD).
- **Maintenance - Domains** panel: live counts + View/Edit links for the Internal (SharePoint list) and External (text file) domain lists, plus a "Replace" control on the External row to upload a new Emergency Report (routes to Agent 3) - the status dot on that row blinks while the upload is being processed.
- **Files** band: a compact status strip showing existence/last-modified information for the 5 key files/lists (Emergency Report, Internal Domains, External Domains, Counter File, Audit Trail File).
- **Automation Status** panel: a compact 4-line live summary (Agent 4 status, Internal Domains active count, Config Parameters active count, Agent Status entries count).
- **Emails Processed** ring (right): a segmented ring showing the proportional breakdown of No DMP / External / Internal Sender / Not Effected e-mail classifications; click to open a legend popup with exact counts/percentages.
- **Next Steps** panel: a checklist-style list of suggested operator actions, sourced from the real-world DMP process tracker.

### 3.2 Agent Monitoring screen

Six tiles (one per agent), each showing live status, last-run result and duration, and status message straight from `DMP Command Agent Status` - a friendlier, per-agent alternative to reading the raw SharePoint list directly.

### 3.3 Help / Operational Manual screen

An in-app, English-language, container-by-container explanation of everything on the Cockpit: what each panel shows, what every button does, and what the status colours/LEDs mean. Reachable from the sidebar at any time; there is currently no F1 keyboard shortcut (not reliably supported by the canvas app platform - browsers intercept/ignore F1 before the app can react to it).

### 3.4 Audit Trail (Detail) screen

Shows live summary counts (Critical/Warnings/total run-summary rows) plus the 10 most recent Critical (Failed) and 10 most recent Warning rows, each read directly from the central Audit Trail table by Agent 4 (v1.4.0+). A button opens the full `AuditTrail.xlsx` file in SharePoint for anything beyond the most recent 10 of each kind.

### 3.5 Configuration (Lists) screen

Direct access to the 3 central SharePoint lists: live active-row counts plus View/New-entry links for `DMP Command Configuration`, `DMP Command Agent Status`, and `DMP Command Internal Domains`.

### 3.6 Maintenance screen

Connection diagnostics (reachability check for all 3 SharePoint lists), an app/agent version overview, and quick links to the Power Automate, SharePoint, and Power Apps maker portals.

### 3.7 Admin Functions screen

Destructive, operator-confirmed actions only, each in its own bordered sub-section, calling Agent 6:
- **Mailbox cleanup**: deletes the `PA Processed Mails` folder tree, with a Yes/Cancel confirmation.
- **Reset e-mail counters**: five buttons (No DMP / Internal Sender / External / Not Effected / Reset ALL), each with its own confirmation; the previous value is recorded in the Audit Trail before the reset.
- A small "Display diagnostics" strip (App.Width/App.Height) for reporting layout issues, kept deliberately unobtrusive.

### 3.8 Switching the Operating State

1. Locate the **Operating State** panel on the Cockpit.
2. To simulate or declare a real DMP event, toggle **Operational Mode** to **DMP**. To end one, toggle back to **Normal**.
3. To switch between the test/simulation environment and production, toggle **Environment** between **SIMU** and **PROD**.
4. Each toggle immediately calls Agent 5 and writes the combined mode to `CurrentOperationMode`. A success or error notification appears at the top of the screen.
5. There is currently **no four-eyes/dual-approval confirmation** before a switch takes effect (this has been raised as a backlog item — see §6).

### 3.9 Viewing/editing domain lists

Use the **View** buttons to open the current Internal (SharePoint list) / External (text file) domain data read-only, or **Edit** to add a new entry (Internal, via a quick-add form) or open the file for editing directly (External).

### 3.10 Replacing the Emergency Report

Click **Replace** next to the External row in the Maintenance - Domains panel, select a `.xlsx` file. Only `.xlsx` files are accepted — other file types are rejected with an error message. The status dot on that row blinks orange while Agent 3 processes the upload, then returns to green (or red on failure). On success, Agent 3 stores the file and the operator should subsequently run Agent 1 to regenerate the domain lists from the new report.

### 3.11 Dark/Light mode

Use the toggle in the top-right of the header to switch between dark and light color themes. This is a purely visual, per-session preference (not currently persisted or per-user configurable — see backlog item on individual color settings).

### 3.12 Known current limitations of the Cockpit (as of 2026-09-01)

- There is currently no in-app mechanism to reset the Audit Trail itself (archive + start fresh) - only the 4 e-mail counters can be reset today (§3.7); a dedicated "Operational History" archive-and-reset for the Audit Trail file is planned but not yet built (see backlog).
- There is currently no four-eyes/dual-approval confirmation before an Operating State switch takes effect.
- Auto-refresh relies on a 1-second canvas app Timer; browser tab throttling of background/inactive tabs can make the countdown appear frozen even though the underlying mechanism is correctly configured - keep the app tab in the foreground for reliable live updates.

---

## 4. Central Configuration List (`DMP Command Configuration`)

This SharePoint list is the single source of truth for every operational parameter used by all 6 agents. Columns:

| Column | Purpose |
|---|---|
| `ParameterName` | Unique key, referenced by agents as `outputs('CMP_ConfigObject')?['ParameterName']` |
| `Active` | `Yes`/`No` Choice column — only `Active = Yes` rows are loaded by any agent (formulas must use `Active.Value`, not `Active`, when read from Power Apps) |
| `Category` | Grouping for readability (Mail, File, Path, Audit, Flow, Runtime, ...) |
| `CurrentValue` | Used only by mode-independent runtime parameters (notably `CurrentOperationMode` itself) |
| `Description` | Human-readable explanation |
| `ParameterType` | Text / Email / Path / Number |
| `Scope` | Which agent(s) the parameter applies to: `Agent 01`–`Agent 06` (unified 2-digit, zero-padded, with a space — as of 2026-08-13), `Global` (cross-agent), or `PowerApps` (GUI-only values) |
| `Value - PROD (NODMP)`, `Value - PROD (DMP)`, `Value - SIMU (NODMP)`, `Value - SIMU (DMP)` | The 4 mode-specific values; agents pick the correct column at runtime based on the current `CurrentOperationMode` |

**Editing rule:** Only the operations team edits this list directly in SharePoint (or via the Power App's **Configuration (Lists)** screen, which links directly to it). Do not hardcode values in any flow — if a new parameter is needed, add it here first (with all 4 mode columns populated) before referencing it in a flow.

**Related list — `DMP Command Internal Domains` (added 2026-09):** A separate SharePoint list, not part of `DMP Command Configuration`, holding one row per internal domain (`Title` = domain name, `Active` Choice column = `Yes`/`No`). Replaces the former flat `Internal_Domains.txt` file. Agent 2 (classification) and Agent 4 (status/count) both read it directly; the Power App's Maintenance - Domains panel and Configuration (Lists) screen both link to it.

**Resolved (2026-08-13):** The former shared `Agent3_All` scope value (used by parameters shared across the pre-renumbering Agent 3.01/3.02/3.03 family) has been retired. Agent 3, 4, and 5 now each have their own dedicated `Agent3AlertFolderName` / `Agent4AlertFolderName` / `Agent5AlertFolderName` parameter with its own `Agent 03` / `Agent 04` / `Agent 05` scope — no agent shares an alert folder with another. See §9 for the naming design rule that formalizes this pattern for future agents.

---

## 5. Cross-Cutting Standards

### 5.1 Audit Trail

Every agent run logs to a shared `AuditTrail.xlsx` table with a fixed 20-column schema: `TimestampUtc, RunId, MessageId, WorkflowPath, StepName, StepStatus, Flow ID, KeyOutput, DurationSec, ActionType, Direction, Recipient, SubjectOut, TargetMessageId, TargetFolderName, TargetFolderId, MatchedDomain, Decision, Sender, SenderDomain`. Every significant step buffers an event; at the end of each run, one additional **Run Summary** row is written (`StepName = "RunSummary"`, `ActionType = "Summary"`) summarizing the overall outcome. `Flow ID` is intentionally left blank across all agents (unused legacy column).

**Agent Audit Summary (added 2026-08-14, completed 2026-08-24):** `AuditTrail.xlsx` also contains a second table, `Agent Audit Summary` (17 columns, one row per agent `Agent 01`–`Agent 05`), that keeps a running, pre-aggregated count of steps and runs by outcome (`SucceededStepsCount`, `FailedStepsCount`, `WarningStepsCount`, `StartedStepsCount`, and the equivalent `...RunsCount` columns), each paired with a `...LastUpdateUtc` timestamp column recording when that specific counter was last incremented. Every agent increments its own row after each run — this is what Agent 4 reads and aggregates for the Cockpit's system-wide Critical/Warning/Total-runs figures (see §2.4), avoiding the need to scan the full detailed log at request time.

**Agent Audit Acknowledgment (backend prepared 2026-08-24, UI not yet built):** A third table, `Audit Acknowledgment` (9 columns, one row per agent), stores a "baseline" count and acknowledgment timestamp for each of the 4 problematic counter categories per agent (`FailedSteps`, `WarningSteps`, `FailedRuns`, `WarningRuns`). The planned Cockpit UI will treat a category as "new/unacknowledged" whenever the live counter in `Agent Audit Summary` exceeds the stored baseline, and will let an operator "acknowledge" a category (updating the baseline to the current count) without ever deleting the underlying counters — full history remains intact. The write side (a button in the Power App) is planned as part of the upcoming GUI work; see the backlog for details.

### 5.2 E-mail Importance

Outbound agent e-mails set Outlook `Importance` from central configuration, never hardcoded:

| Severity | Meaning | Config Parameter | Value |
|---|---|---|---|
| Info | No action required | `MailImportanceInfo` | `Low` |
| Warning | Advisory, no action required | `MailImportanceWarning` | `Normal` |
| Error/Critical | Action required | `MailImportanceError` | `High` |

### 5.3 Status Reporting

Every agent updates its own row in `DMP Command Agent Status` after each run (`CurrentStatus`, `LastRunTimestamp`, `LastRunResult`, `LastRunDurationSec`, `LastFailureTimestamp`/`Step`/`Message`, `StatusSeverity`, `StatusMessage`, `OperationMode`). This is the fast, pre-computed data source intended for the Power App dashboard.

---

## 6. Fire Drill / Emergency DMP Event Procedure

1. **Declare the event**: In the Power App Cockpit, switch the **Operational Mode** toggle to **DMP**. Confirm the success notification and that the "Mode" value in the Operating State panel now shows the expected `*_DMP` value.
2. **Choose the correct environment**: Ensure the **Environment** toggle correctly reflects whether this is a real production event (`PROD`) or a drill/test (`SIMU`). **Verify this carefully before declaring** — there is currently no dual-approval safeguard.
3. **If a new Emergency Report is available**: Upload it via **Maintenance - Domains → Replace** (Agent 3), then manually run **Agent 1** to regenerate the domain lists.
4. **Monitor**: Agent 2 will now classify inbound mail using the DMP-specific subject/wording rules and the refreshed domain lists.
5. **Ending the event**: Once the drill/event is over, switch **Operational Mode** back to **Normal**. This is explicitly listed as a "Next Steps" reminder item on the Cockpit screen.
6. **Review**: Check the **Audit Trail (Detail)** screen in the Power App for the most recent Critical/Warning rows during the event window (see §5.1, §3.4), or review `AuditTrail.xlsx` directly in SharePoint for the full history.

---

## 7. Troubleshooting

| Symptom | Likely Cause | Action |
|---|---|---|
| Operating State toggle shows an error notification | Agent 5 flow failure, or the SharePoint connection used by the flow lacks permission | Check the flow's run history in Power Automate; check `AuditOutcome`/`StatusMessage` in the Agent Status row for Agent 5 |
| Cockpit dashboard Critical/Warning counts look wrong or unexpectedly zero | An agent's `Agent Audit Summary` row was not updated after a recent run (flow failure before reaching the summary-write step), or Agent 4 has not been re-run since | Check the flow's run history for the affected agent in Power Automate; manually re-run Agent 4 to refresh the aggregated figures |
| Domain counts look outdated after uploading a new Emergency Report | Agent 1 was not run after the Agent 3 upload | Manually trigger Agent 1 |
| "Only .xlsx files are allowed" error when using Replace | Wrong file type selected | Re-select a genuine `.xlsx` Emergency Report file |
| A config value appears to have "no effect" after being changed in SharePoint | `Active` column is not set to `Yes`, or the wrong mode-specific column was edited | Verify `Active = Yes` and that the value was entered in the column matching the *currently active* `CurrentOperationMode` |
| Agent 5's Operating State write silently fails to pick up its own alert/scope config | Live `Scope` value in SharePoint doesn't match the flow's filter exactly (e.g. missing zero-padding or space) | Verify the flow's `$filter` expression matches the exact live `Scope` string (`Agent 05`, not `Agent5` or `agent 05`) |

**Resolved (2026-08-13):** All manual SharePoint updates required by the agent renumbering (Scope values, `AgentKey` values, `WorkflowPathAgentN` parameter names/values) have been completed by the operations team, confirmed via a fresh configuration export. The Scope pattern was further unified the same day to `Agent 01`–`Agent 05` (see §4 and §9).

**Resolved (2026-08-13):** The Cockpit dashboard no longer shows static/demo numbers — Agent 4 is called directly by the Power App and all dashboard figures reflect live data (see §2.4).

---

## 8. Change Log (Manual)

| Date | Change |
|---|---|
| 2026-08-13 | Initial version. Reflects the 5-agent sequential renumbering, Agent 5 (formerly 3.03) Yes.txt decommissioning, real Operating State toggle wiring, and known Agent 4/Cockpit live-data limitations. |
| 2026-08-13 | Scope pattern unified to `Agent 01`–`Agent 05` (2-digit, zero-padded, with space) across all agents; retired the shared `Agent3_All` scope in favor of dedicated per-agent `AgentNAlertFolderName` parameters; fixed Agent 5's config filter accordingly; added §9 naming design rule for future agents; cleaned up internal flow action names and the Power App Agent Heartbeat legend to drop the old "Agent 3.01/3.02/3.03" labels; consolidated `Documentation/` by archiving superseded files into `ARCHIVE/`. |
| 2026-08-13 | Agent 4 rebuilt: Select+Join config loading (replacing the per-row loop), removed dead RealDMP indicator-file scaffolding. Power App Cockpit now calls Agent 4 directly and binds its dashboard (Files band, Agent 2 email-classification ring, Agent Heartbeat) to live data instead of static placeholders. Several Cockpit GUI fixes (heartbeat card layout, header title contrast in light mode, KPI column alignment, sidebar scrollbar, button drop shadows). |
| 2026-08-14 | Added alert-mail-then-move pattern (mirroring Agents 1–3) to Agent 4 and Agent 5 for error notifications. |
| 2026-08-24 | Added the `Agent Audit Summary` table to `AuditTrail.xlsx` (per-agent, per-outcome step/run counters with a last-update timestamp per counter) and wired all 5 agents to maintain it; Agent 4 now aggregates real Critical/Warning/Total-runs figures from this table instead of always returning 0 (§2.4, §5.1, §7 updated accordingly). Prepared the backend (`Audit Acknowledgment` table) for a future warning/alert acknowledgment feature (UI not yet built). Fixed a dark/light theme toggle flicker on the Cockpit screen (§3.5). |
| 2026-08-24 | Completed a full tile-by-tile review of the Cockpit screen (`scrHome.pa.yaml`, all 8 containers): fixed hardcoded KPI colors, a hardcoded operating-mode label, a fabricated "last changed" name/timestamp, two hardcoded domain counts, a legend color mismatch in the email ring, two fabricated "Next Steps" detail texts, and multiple text-encoding (mojibake) artifacts (§3.6). All Cockpit tiles now show genuinely live data or non-data-claiming guidance text. |
| 2026-08-25 – 2026-08-28 | Agent 6 (Admin Functions) introduced (mailbox cleanup action). Warning/Critical acknowledgment (baseline-diff, click-to-acknowledge) shipped on the Critical/Warnings/Agents Active KPI tiles. Several Timer/refresh reliability fixes (auto-refresh interval selector, countdown, toggle-guard flicker). |
| 2026-09-01 (v1.10.0–v1.10.2) | Internal Domains migrated from `Internal_Domains.txt` to the new `DMP Command Internal Domains` SharePoint list (Agent 2 + Agent 4 updated to match). Automation Status redesigned as a compact live panel. Release Notes screen added (App Changes + Agent Changes tabs, per-card scrolling). Replace-button processing LED fixed (now reliably blinks). All status dots across the Cockpit unified to the same 14px-circle size. |
| 2026-09-01 (v1.11.0–v1.13.1) | **Agent Monitoring**, **Help / Operational Manual**, **Configuration (Lists)**, and **Maintenance** screens built out (previously navigation placeholders — see §3 for the current, accurate description of each). **Admin Functions** extended with e-mail counter reset actions (Agent 6 v1.2.0), each its own bordered sub-section, "Test"/"Test Only" wording removed. **Audit Trail (Detail)** screen built out: live summary counts plus the 10 most recent Critical/Warning rows, read directly by Agent 4 (v1.4.0) from the central Audit Trail table. Clicking the Critical/Warnings/Agents Active KPIs now also navigates to the relevant detail screen. Maintenance Domains layout reflowed to fix truncated INTERNAL/EXTERNAL labels; Replace LED simplified (single blinking status dot instead of a separate LED). Documentation folder in the git repository resynced with this working copy (had drifted since 2026-08-14) and a documentation-sync AI rule added to prevent recurrence. |

---

## 9. Design Rule: Naming Convention for New Agents

When a new agent (`N`, written 2-digit as `NN`) is introduced, apply these rules consistently:

1. **Agent number:** Sequential integer, no decimal sub-numbering (the old "Agent 3.01" style is retired as of 2026-08-13).
2. **Display name:** `DMP Agent N (<Purpose>)`.
3. **`Scope` value** (Configuration list): `Agent NN` — always 2-digit, zero-padded, **with a space** (e.g. `Agent 06`, not `Agent6` or `Agent 6`). Use `Global` for cross-agent shared parameters, `PowerApps` for GUI-only display values.
4. **`AgentKey`** (Agent Status list): `Agent_NN` (underscore, zero-padded) — deliberately a *different* format from the Scope value, to keep the two lists visually distinct.
5. **`WorkflowPathAgentN` value** (audit trail `WorkflowPath` column): `AgentNN_<PascalCasePurpose>`, e.g. `Agent06_NewPurpose`.
6. **Dedicated Alert Folder:** every agent that sends alert/error e-mails gets its own `AgentNAlertFolderName` parameter (Scope = that agent's own `Agent NN` scope), value `Agent NN Alerts`. Never share an alert folder across agents.
7. **Internal flow action names:** status-board read/write actions use an `_Agent_NN` suffix (e.g. `GET_StatusRow_Agent_06`, `UPDATE_StatusRow_Agent_06`), consistent with the AgentKey format.
8. All 4 mode-specific value columns must be populated, even if the value is identical across all 4 modes.
