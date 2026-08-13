# DMP COMMAND — Operations Manual

**Scope:** This manual covers the complete DMP COMMAND system: all 5 backend agents (Power Automate flows), the DMP COMMAND Power App (Cockpit GUI), the central configuration list, and standard operating procedures including the Fire Drill / Emergency procedure.

**Audience:** Operations team members responsible for running, monitoring, and troubleshooting DMP COMMAND day-to-day.

**Related documents:**
- `Documentation/Backlog/DMP_COMMAND_Backlog.md` — technical backlog, architecture decisions, and detailed findings (developer-facing, German)
- `Documentation/DMP Command Configuration.csv` — periodic export of the live SharePoint configuration list
- `Documentation/DMP Command Agent Status.csv` — periodic export of the live SharePoint agent status list
- `README.md` (repository root) — developer/ALM setup (pac CLI workflow, repository structure)

---

## 1. System Overview

DMP COMMAND automates the handling of DMP (Disaster/Major Peril) communication events for Eurex. The system consists of:

- **5 Power Automate flows ("Agents")** that extract domain data, classify inbound e-mails, manage emergency report ingestion, check system status, and control the operational mode.
- **A central configuration list** (`DMP Command Configuration`, SharePoint) that is the single source of truth for all operational parameters. No agent uses hardcoded values for anything that varies by environment or mode.
- **A status list** (`DMP Command Agent Status`, SharePoint) that every agent writes to after each run, providing a fast, pre-computed snapshot for the Power App dashboard (avoids slow live file/data checks in the UI layer).
- **An audit trail** (`AuditTrail.xlsx`, SharePoint) that every agent appends structured, per-step audit events plus one run-summary row to, for compliance and troubleshooting.
- **The DMP COMMAND Power App**, a single-screen Cockpit dashboard used by operators to monitor agent health, view/edit domain lists, switch the operational mode, and see file/audit status at a glance.

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

As of 2026-08-13, all 5 agents use sequential numbering (previously Agent 1, Agent 2, Agent 3.01, Agent 3.02, Agent 3.03):

| Current Name | Previous Name | Purpose |
|---|---|---|
| Agent 1 (Domains Extraction) | Agent 1 | Extracts internal/external domain lists from the Emergency Report |
| Agent 2 (E-Mail Inbox Treatment) | Agent 2 | Classifies and routes inbound DMP mailbox e-mails |
| Agent 3 (Emergency Report Management) | Agent 3.01 | Ingests and validates new Emergency Report uploads |
| Agent 4 (Status Check) | Agent 3.02 | Reports system/file status to the Power App |
| Agent 5 (Operational State Management) | Agent 3.03 (formerly "YES File Management") | Writes the Operating State toggle changes to central configuration |

---

## 2. Agent Reference

### 2.1 Agent 1 — Domains Extraction

**Purpose:** Extracts the current list of internal and external domains from the `Emergency Report.xlsx` worksheet ("Emergency Contacts") and writes them to `Internal_Domains.txt` / `External_Domains.txt` for use by Agent 2's e-mail classification logic.

**Trigger:** Manual/on-demand (Power Automate manual trigger), typically run after a new Emergency Report is uploaded (see Agent 3).

**Process:**
1. Loads active configuration rows scoped to `Agent1` and `Global`.
2. Reads the Emergency Report workbook, worksheet "Emergency Contacts" (configurable via `Agent1SourceWorksheetName`).
3. Extracts and de-duplicates domain values into an internal/external classification.
4. Writes `Internal_Domains.txt` and `External_Domains.txt` to their configured SharePoint storage locations.
5. Sends alert e-mails on failure conditions (missing worksheet, invalid workbook, invalid file extension) to `AlertEmailRecipient`.
6. Writes a run summary and per-step audit events to the Audit Trail, and updates its own row in the Agent Status list.

**Error handling:** On any failure branch (missing worksheet, invalid workbook/extension, write failure), Agent 1 sends an alert e-mail with `Importance = High` (via the central `MailImportanceError` setting) and records the failure in both the Audit Trail and the Agent Status list (`CurrentStatus = Failed`, `StatusSeverity = Critical`).

**Key configuration parameters:** `Agent1TriggerFolder`, `Agent1TriggerFileName`, `Agent1SourceWorksheetName`, `Agent1AlertFolderName`, `ExternalDomainsStorageFolder`, `ExternalDomainsFileName`, `RealDMPIndicatorFolder`/`RealDMPIndicatorFileName` (legacy naming, no longer used for control flow), `WaitSecondsBeforeSentMailSearch`, `WorkflowPathAgent1`.

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
2. Reads the e-mail's sender domain and matches it against the Internal/External domain lists.
3. Moves the e-mail to the corresponding processed-mail subfolder (`No DMP`, `DEE`, `DIS`, `DNES`).
4. Increments the corresponding counter in the central counter file/table (`CounterPathNoDMP`, `CounterPathInternalSender`, `CounterPathNotEffected`, `CounterPathEffectedMember`).
5. Sends category-appropriate notification e-mails where configured (e.g. DIS info mail).
6. Buffers and writes audit events; updates the Agent Status row.

**Error handling:** Handles Graph API failures (mailbox folder resolution), audit write failures, and missing-counter-file conditions as distinct error categories, each with its own alert e-mail and audit/status entry. E-mail importance is set from the central 3-tier scheme (see §5.2).

**Key configuration parameters:** `Agent2TriggerMailbox`, `Agent2AlertFolderName`, `CounterFileName`, `CounterFolder`, `CounterTableName`, `CounterPathNoDMP`/`CounterPathInternalSender`/`CounterPathNotEffected`/`CounterPathEffectedMember`, `InternalDomainsFileName`/`InternalDomainsStorageFolder`, `ExternalDomainsFileName`/`ExternalDomainsStorageFolder`, `WorkflowPathAgent2`.

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

**Purpose:** Provides file-existence and count status for the Power App's Files band and related dashboard elements (Emergency Report, Internal/External Domains, Counter File, Audit Trail File).

**Trigger:** Manual/on-demand, intended to be called by the Power App when the Cockpit screen needs a status refresh.

**Process:** Performs live SharePoint file-metadata and content checks for each monitored file (existence, last-modified timestamp, and for domain files, a count of entries), and returns them via a structured response.

**⚠️ Known limitation:** Agent 4's live file-check approach is what previously caused a Power Apps request timeout when it was first used from the app. Because of this, the Power App does **not** currently call Agent 4 directly for its live dashboard data; instead, the Files band, Agent Heartbeat wheel, and KPI values on the Cockpit screen currently use static placeholder/demo values.
**Planned change (not yet implemented):** Agent 4 should be rebuilt to read the pre-computed `DMP Command Agent Status` list (a fast, simple `Get items` query) instead of performing live file checks, removing the timeout risk while still providing real data to the app.

**Key configuration parameters:** File name/folder parameters shared with Agent 1/Agent 2/Agent 3 (`ExternalDomainsFileName`, `InternalDomainsFileName`, `CounterFileName`, `AuditFileName`, `EmergencyReportFileName`, etc.), `WorkflowPathAgent4`.

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

## 3. Power App (Cockpit) — User Guide

### 3.1 Layout

The Cockpit screen is organized as follows:
- **Sidebar** (left): navigation (Cockpit, Agent Monitoring, Operational Board, Audit Trail (Detail), Configuration (Lists), Maintenance).
- **Header**: application title, KPI strip (Critical / Warnings / Agents Active counts), version tag, Dark/Light mode toggle.
- **Agent Heartbeat** panel: a segmented ring showing aggregate agent health, with a colored-dot legend for all 5 agents.
- **Operating State** panel: current mode, last-changed timestamp/user, and the two Operating State toggles (Operational Mode: Normal/DMP; Environment: SIMU/PROD).
- **Maintenance - Domains** panel: View/Edit links for the Internal and External domain lists, plus a "Replace" control to upload a new Emergency Report (routes to Agent 3).
- **Agent 2 - Emails Processed** panel: a segmented ring showing the proportional breakdown of No DMP / DEE / DIS / DNES email classifications.
- **Files** band: a compact status strip showing existence/last-modified information for the 5 key files (Emergency Report, Internal Domains, External Domains, Counter File, Audit Trail File).
- **Next Steps** panel: a checklist-style list of suggested operator actions.

### 3.2 Switching the Operating State

1. Locate the **Operating State** panel.
2. To simulate or declare a real DMP event, toggle **Operational Mode** to **DMP**. To end one, toggle back to **Normal**.
3. To switch between the test/simulation environment and production, toggle **Environment** between **SIMU** and **PROD**.
4. Each toggle immediately calls Agent 5 and writes the combined mode to `CurrentOperationMode`. A success or error notification appears at the top of the screen.
5. There is currently **no four-eyes/dual-approval confirmation** before a switch takes effect (this has been raised as a backlog item — see §6).

### 3.3 Viewing/editing domain lists

Use the **View** buttons to open the current Internal/External domain files read-only, or **Edit** to open them for editing directly in SharePoint/Excel Online.

### 3.4 Replacing the Emergency Report

Click **Replace** next to the External row in the Maintenance - Domains panel, select a `.xlsx` file. Only `.xlsx` files are accepted — other file types are rejected with an error message. On success, Agent 3 stores the file and the operator should subsequently run Agent 1 to regenerate the domain lists from the new report.

### 3.5 Dark/Light mode

Use the toggle in the top-right of the header to switch between dark and light color themes. This is a purely visual, per-session preference (not currently persisted or per-user configurable — see backlog item on individual color settings).

### 3.6 Known current limitations of the Cockpit screen (as of 2026-08-13)

- The Agent Heartbeat wheel, Files band, and email-classification ring currently show **static/demo data**, not live figures, because the Power App has no direct SharePoint read connector and Agent 4 (the intended data source) is not yet rebuilt to avoid its timeout risk (see §2.4).
- Agent Monitoring, Operational Board, Audit Trail (Detail), Configuration (Lists), and Maintenance sidebar items are navigation placeholders and are not yet built out as separate screens.

---

## 4. Central Configuration List (`DMP Command Configuration`)

This SharePoint list is the single source of truth for every operational parameter used by all 5 agents. Columns:

| Column | Purpose |
|---|---|
| `ParameterName` | Unique key, referenced by agents as `outputs('CMP_ConfigObject')?['ParameterName']` |
| `Active` | `Yes`/`No` — only `Active = Yes` rows are loaded by any agent |
| `Category` | Grouping for readability (Mail, File, Path, Audit, Flow, Runtime, ...) |
| `CurrentValue` | Used only by mode-independent runtime parameters (notably `CurrentOperationMode` itself) |
| `Description` | Human-readable explanation |
| `ParameterType` | Text / Email / Path / Number |
| `Scope` | Which agent(s) the parameter applies to (`Agent1`, `Agent2`, `Agent3`, `Agent4`, `Agent5`, `Agent3_All` [shared], `Global`) |
| `Value - PROD (NODMP)`, `Value - PROD (DMP)`, `Value - SIMU (NODMP)`, `Value - SIMU (DMP)` | The 4 mode-specific values; agents pick the correct column at runtime based on the current `CurrentOperationMode` |

**Editing rule:** Only the operations team edits this list directly in SharePoint. Do not hardcode values in any flow — if a new parameter is needed, add it here first (with all 4 mode columns populated) before referencing it in a flow.

**Note on `Agent3_All` scope:** This scope value predates the 2026-08-13 agent renumbering and was used for parameters shared across the former Agent 3.01/3.02/3.03 family (now Agents 3/4/5). It has been intentionally left unchanged pending a decision on whether to rename it (e.g. to `Global` or a new shared scope name) — see the open item in §7.

---

## 5. Cross-Cutting Standards

### 5.1 Audit Trail

Every agent run logs to a shared `AuditTrail.xlsx` table with a fixed 20-column schema: `TimestampUtc, RunId, MessageId, WorkflowPath, StepName, StepStatus, Flow ID, KeyOutput, DurationSec, ActionType, Direction, Recipient, SubjectOut, TargetMessageId, TargetFolderName, TargetFolderId, MatchedDomain, Decision, Sender, SenderDomain`. Every significant step buffers an event; at the end of each run, one additional **Run Summary** row is written (`StepName = "RunSummary"`, `ActionType = "Summary"`) summarizing the overall outcome. `Flow ID` is intentionally left blank across all agents (unused legacy column).

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
6. **Review**: Check the Audit Trail for warnings or failures during the event window (see §5.1). A dedicated "Audit Trail (Detail)" screen is planned but not yet built — for now, review `AuditTrail.xlsx` directly in SharePoint.

---

## 7. Troubleshooting

| Symptom | Likely Cause | Action |
|---|---|---|
| Operating State toggle shows an error notification | Agent 5 flow failure, or the SharePoint connection used by the flow lacks permission | Check the flow's run history in Power Automate; check `AuditOutcome`/`StatusMessage` in the Agent Status row for Agent 5 |
| Cockpit dashboard shows static/unchanging numbers | Expected — Agent 4 rebuild to provide live data is not yet complete (see §2.4) | No action; this is a known, tracked limitation |
| Domain counts look outdated after uploading a new Emergency Report | Agent 1 was not run after the Agent 3 upload | Manually trigger Agent 1 |
| "Only .xlsx files are allowed" error when using Replace | Wrong file type selected | Re-select a genuine `.xlsx` Emergency Report file |
| A config value appears to have "no effect" after being changed in SharePoint | `Active` column is not set to `Yes`, or the wrong mode-specific column was edited | Verify `Active = Yes` and that the value was entered in the column matching the *currently active* `CurrentOperationMode` |
| Agent's WorkflowPath / audit label still shows old naming (e.g. "Agent3_YesFileManagement") after the 2026-08-13 renumbering | The corresponding `WorkflowPathAgentN` row in the live SharePoint configuration list has not yet been updated to match the flow-side rename | Update the SharePoint row (see the precise list of required manual SharePoint changes tracked separately for this migration) |

**Open items pending manual SharePoint updates (as of 2026-08-13 renumbering):** Config `Scope` values (`Agent3_01`→`Agent3` etc. — already reflected on the flow side but not yet in the live list), `AgentKey` values in the Agent Status list, and the `WorkflowPathAgentN` parameter names/values. See the dedicated instructions provided to the operations team for the exact rows and values to change.

---

## 8. Change Log (Manual)

| Date | Change |
|---|---|
| 2026-08-13 | Initial version. Reflects the 5-agent sequential renumbering, Agent 5 (formerly 3.03) Yes.txt decommissioning, real Operating State toggle wiring, and known Agent 4/Cockpit live-data limitations. |
