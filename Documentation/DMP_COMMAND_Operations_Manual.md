# DMP COMMAND — Operations Manual

**Scope:** This manual covers the complete DMP COMMAND system: all 6 backend agents (Power Automate flows), the DMP COMMAND Power App (Cockpit GUI, with 10 screens - Cockpit, Agent Monitoring, System Health (Details), Help / Operational Manual, Audit Trail (Detail), Configuration (Lists), Maintenance, Task Occurrences, Admin Functions, Release Notes), the central SharePoint lists, and standard operating procedures including the Fire Drill / Emergency procedure.

**Audience:** Operations team members responsible for running, monitoring, and troubleshooting DMP COMMAND day-to-day.

**Related documents:**
- `Documentation/DMP_COMMAND_Release_Notes.md` — full version history of every app and agent change (what changed, when, in which version). This manual deliberately does **not** repeat that history — it describes only the current, tested behavior of the system.
- `Documentation/Backlog/DMP_COMMAND_Backlog.md` — technical backlog, architecture decisions, and open items not yet built (developer-facing, German)
- `Documentation/DMP Command Configuration.csv` — periodic export of the live SharePoint configuration list
- `Documentation/DMP Command Agent Status.csv` — periodic export of the live SharePoint agent status list
- `README.md` (repository root) — developer/ALM setup (pac CLI workflow, repository structure)

**A note on scope:** This manual is a **functional specification**, not a troubleshooting guide or change log. It describes what every screen, field, button, and agent does today — including every status value, warning, and error message that is part of normal, designed behavior. It does not contain incident narratives, root-cause analyses, or "recently fixed" language; for the history of how the system reached its current state, see the Release Notes. Known gaps in current functionality are listed in §7 (Known Limitations); planned future work is listed in §8 (Planned in Future Phases) and detailed further in the Backlog.

---

## 1. System Overview

DMP COMMAND automates the handling of DMP (Disaster/Major Peril) communication events for Eurex. The system consists of:

- **6 Power Automate flows ("Agents")** that extract domain data, classify inbound e-mails, manage emergency report ingestion, check system status, control the operational mode, and (as of Agent 6) perform administrative test/reset actions.
- **Five central SharePoint lists** that are the single source of truth for all shared data: `DMP Command Configuration` (all operational parameters - no agent uses hardcoded values for anything that varies by environment or mode), `DMP Command Agent Status` (fast, pre-computed per-agent snapshot for the Power App dashboard), `DMP Command Internal Domains` (one row per internal domain, `Active` Choice column, maintained by Agent 1 and read by Agent 2/Agent 4), `DMP Command External Domains` (one row per external domain, fully rewritten by Agent 1 on every Emergency Report extraction, read by Agent 2/Agent 4), and `DMP Command Counters` (one row per e-mail-classification counter, keyed by `Title`, value column `NumberProcessedEmails`, updated by Agent 2 and reset by Agent 6).
- **An audit trail** (`AuditTrail.xlsx`, SharePoint) that every agent appends structured, per-step audit events plus one run-summary row to, for compliance and record-keeping. Agent 4 also reads this table directly to surface the most recent Critical/Warning rows in the Cockpit's Audit Trail (Detail) screen.
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

**Purpose:** Extracts the current list of external domains from the `Emergency Report.xlsx` worksheet ("Emergency Contacts", configurable via `Agent1SourceWorksheetName`) and performs a full delete-and-recreate sync of the **`DMP Command External Domains` SharePoint list**, which Agent 2's e-mail classification logic reads from for external-sender matching. (Internal domains are maintained directly as a separate SharePoint list, `DMP Command Internal Domains`, and are not touched by Agent 1.)

**Trigger:** Automatic — a SharePoint "When a file is created" trigger polling the `Emergency_Report_Storage` document library every 1 minute, filtered to files whose name equals exactly `Emergency Report.xlsx`. It fires shortly after Agent 3 stores a newly uploaded Emergency Report; there is no manual/on-demand trigger.

**Process:**
1. Loads all active configuration rows in one read (no per-agent `Scope` filter is applied at the flow level; the flow selects only the keys it needs after loading).
2. Resolves/creates the `Agent 1 Alerts` mailbox subfolder used to archive the alert e-mails this flow sends.
3. Reads the Emergency Report workbook, worksheet "Emergency Contacts" (configurable via `Agent1SourceWorksheetName`), via an Office Script that returns the extracted domain array plus a status indicator.
4. Deletes all existing rows of `DMP Command External Domains` and creates one fresh row per extracted domain (full-sync rewrite), then confirms the write.
5. Buffers per-step audit events and writes a run summary to the Audit Trail.
6. Updates its own row (`AgentKey = Agent_01`) in the Agent Status list.
7. Sends alert/notification e-mails to `AlertEmailRecipient` for every distinct outcome branch (see below); all Agent 1 e-mails set `Importance = "Normal"` (Agent 1 does not use the central `MailImportanceError`/`MailImportanceWarning` tiering that Agents 2/3/5 use).

**Outcomes & status values:**

| Condition | `AuditOutcome` | Agent Status `CurrentStatus` | `StatusSeverity` | Alert e-mail (subject contains) | Terminates run? |
|---|---|---|---|---|---|
| Domains extracted and `DMP Command External Domains` list rewritten successfully | `Succeeded` | `Operational` | `Information` | Success/info mail: `[Agent 1][SUCCESS] External Domains file written [RID:…\|SC:…]` (archived, no action needed) | No |
| Office Script / domain extraction technical error | `Failed` | `Failed` | `Critical` | `[Agent 1][FAILED] Domain extraction technical failure [EC:A1-TECHERROR][RID:…]` | Yes |
| Office Script ran but returned zero valid domains from the source worksheet (business data issue, not a technical fault) | `Failed` | `Failed` | `Critical` | Warning-style mail: `[Agent 1][FAILED] No domains found in source worksheet [EC:A1-NODOMAINS][RID:…]` | Yes |
| Rewrite of `DMP Command External Domains` list fails (e.g. SharePoint permission/connectivity issue) | `Failed` | `Failed` | `Critical` | `[Agent 1][FAILED] External Domains file write failed [EC:A1-FILEWRITE][RID:…]` | Yes |
| `Agent 1 Alerts` mailbox subfolder cannot be resolved/created at flow start | `Failed` | `Failed` | `Critical` | `[Agent 1][FAILED] Mailbox Setup Failed [EC:A1-MBOXSETUP][RID:…]` | Yes |
| Unexpected/unhandled error anywhere outside the above business branches | `Failed` | `Failed` | `Critical` | `[Agent 1][FAILED] Global Flow Failure [EC:A1-GLOBALFAIL][RID:…\|SC:…]` | Yes |
| Central Audit Trail file cannot be written to | `Warning` (unless a more severe outcome was already recorded) | `Warning` | `Warning` | `[Agent 1][WARNING]`-style Audit-Write-Failed mail (in-memory audit event preserved even though the Excel write failed) | No |

Each alert mail is looked up afterward in Sent Items (by its exact composed subject) and moved into the `Agent 1 Alerts` mailbox folder for archiving; a failure to find/move it is itself recorded as an additional (non-blocking) audit event.

**Key configuration parameters:** `Agent1SourceWorksheetName`, `Agent1AlertFolderName`, `AlertEmailRecipient`, `SharedDMPMailbox`, `ProcessedMailsRootFolderName`, `WaitSecondsBeforeSentMailSearch`, `WorkflowPathAgent1`, `SubjectPrefix`/`MailModeText` (test/prod mail-mode prefix and banner).

---

### 2.2 Agent 2 — E-Mail Inbox Treatment

**Purpose:** Monitors the shared DMP mailbox, classifies each inbound e-mail by sender domain against two SharePoint lists — `DMP Command Internal Domains` and `DMP Command External Domains` — and routes it into one of four categories:

| Category | Meaning |
|---|---|
| No DMP | Sender not matched to a monitored domain — no DMP relevance |
| DEE (DMP External Effected) | External sender, domain matched, DMP-relevant |
| DIS (DMP Internal Sender) | Internal sender, DMP-relevant |
| DNES (DMP Not Effected Sender) | Sender matched but classified as not affected |

**Trigger:** Automatic — a polling trigger (recurrence-based) on the shared DMP mailbox that picks up newly arrived e-mails one at a time.

**Process:**
1. Loads active configuration (Select+Join pattern, ~3 actions instead of the historical 212-action per-row loop).
2. Reads the e-mail's sender domain and matches it against `DMP Command Internal Domains` and `DMP Command External Domains` (both SharePoint lists).
3. Moves the e-mail to the corresponding processed-mail subfolder (`No DMP`, `DEE`, `DIS`, `DNES`).
4. Increments the corresponding counter row (keyed by `Title`, e.g. `'No DMP'`, `'DMP internal Sender'`, `'DMP effected Member'`, `'DMP not effected Sender'`; value column `NumberProcessedEmails`) in the `DMP Command Counters` SharePoint list.
5. Sends an acknowledgement reply to the sender and/or an internal notification e-mail to the Hotline team where configured (e.g. the DIS path always sends both an acknowledgement reply to the sender and an internal "new DIS mail arrived" notification to `AlertEmailRecipient`, at `MailImportanceActionRequired`).
6. Buffers and writes audit events; updates the Agent Status row.

**Outcomes & status values:**

Each of the four routing categories (ND / DIS / DEE / DNES) can independently raise the following distinct conditions; DEE additionally has a forward-failure condition. All alert e-mails go to `AlertEmailRecipient`, with `Importance` taken from the matching tier of the central 3-value scheme (`MailImportanceError`, `MailImportanceWarning`, `MailImportanceActionRequired`).

| Condition (per category ND/DIS/DEE/DNES unless noted) | `AuditOutcome` | Importance tier | Alert e-mail subject pattern |
|---|---|---|---|
| Processed-mail subfolder for the category cannot be created/resolved | `Warning` | `MailImportanceError` | `[Agent 2][WARNING] Mailbox Folder Missing (…) [EC:A2-MBOXFOLDER-{ND\|DIS\|DEE\|DNES}][RID:…]` |
| The category's own outbound response e-mail cannot be found in Sent Items afterward (so it cannot be moved/archived) | `Warning` | `MailImportanceWarning` | `[Agent 2][WARNING] Sent Response Mail Not Found (…) [EC:A2-SENTNOTFOUND-{ND\|DEE\|DNES}][RID:…]` (not applicable to DIS) |
| Counter row for the category cannot be updated in `DMP Command Counters` | `Failed` | `MailImportanceError` | `[Agent 2][FAILED] Counter Update Failed (…) [EC:A2-COUNTERFAIL-{ND\|DIS\|DEE\|DNES}][RID:…]` |
| DEE only: forward of the e-mail to CAMS/Porting fails | `Failed` | `MailImportanceError` | `[Agent 2][FAILED] E-Mail Forward to CAMS / Porting Failed (DEE) [EC:A2-FWDFAILED-DEE][RID:…]` |
| Internal/external domains classification parse fails before routing decision | `Failed`, `StatusSeverity=Critical` | `MailImportanceError` | `[Agent 2][FAILED] Domains Classification Parse Failed [EC:A2-DOMAINS-PARSE][RID:…]` |
| `DMP Command Internal Domains` reference list/table is missing entirely | `Failed` (processing continues — this does not terminate the run) | `MailImportanceError` | `[Agent 2][FAILED] Missing Internal Domains Reference File [EC:A2-INTDOMAINS-MISSING][RID:…]` |
| `DMP Command Counters` list/table is missing entirely | `Failed`, run is **terminated** | `MailImportanceError` | `[Agent 2][FAILED] Missing Counter File [EC:A2-COUNTERFILE-MISSING][RID:…]` |
| Central Audit Trail file cannot be written to | `Warning` (unless a more severe outcome already recorded) | n/a | `[Agent 2][WARNING] Audit Trail Write Failed [EC:A2-AUDITWRITE][RID:…]` |
| No error condition occurred | `Succeeded` | — | none |

Agent Status row mapping (final): `CurrentStatus` = `Operational` / `Warning` / `Failed`, `StatusSeverity` = `Information` / `Warning` / `Critical`, derived directly from the final `AuditOutcome` value above (`Succeeded`→`Operational`/`Information`; `Warning`→`Warning`/`Warning`; anything else→`Failed`/`Critical`).

**Key configuration parameters:** `SharedDMPMailbox`, `AlertEmailRecipient`, `MailImportanceError`/`MailImportanceWarning`/`MailImportanceActionRequired`, `ProcessedMailsRootFolderName`, `BusinessReferencePrefixInternalSender` (and equivalent prefixes for the other categories), `SubjectPrefix`/`MailModeText`, `WorkflowPathAgent2`.

---

### 2.3 Agent 3 — Emergency Report Management

**Purpose:** Accepts a newly-uploaded Emergency Report file (`.xlsx`) from the Power App's "Replace" control, validates its file extension, workbook readability and required worksheet, stores it in the configured location, and signals that Agent 1 should be run to regenerate the domain list.

**Trigger:** Called directly by the Power App (HTTP `Request` trigger) when a user uploads a new Emergency Report via the Maintenance-Domains "Replace" control on the Cockpit screen.

**Process:**
1. Validates the uploaded file's extension, workbook readability, and the presence of the required worksheet (`RequiredWorksheetName`) — these checks are performed inside the flow itself, not only client-side in the app.
2. Stores the file to its configured SharePoint location (overwriting the previous version) if all validations pass.
3. Resolves/creates the `PA Processed Mails` / `Agent 3 Alerts` mailbox evidence subfolders used for housekeeping.
4. Buffers audit events and writes a run summary.
5. Updates its own Agent Status row.
6. Returns a structured response (`responsestatus`, `responsemessage`, `responsecode`) that the Power App displays via `Notify()`.

**Outcomes & status values:**

| Condition | `AppResponseStatus` | `AppResponseCode` | `AppResponseMessage` (exact/templated) | Agent Status `CurrentStatus` / `StatusSeverity` | Alert e-mail? |
|---|---|---|---|---|---|
| File stored successfully | `SUCCESS` | `OK` | `Emergency Report uploaded successfully.` | `Operational` / `Information` | No |
| Required worksheet not found in the uploaded workbook | `FAILED` | `MISSING_WORKSHEET` | `Invalid Emergency Report. Worksheet '<RequiredWorksheetName>' was not found.` | `Failed` / `Critical` | Yes — to `AlertEmailRecipient`, body states "Agent 3 - Emergency Report upload rejected." |
| Uploaded file is not a readable Excel workbook | `FAILED` | `INVALID_WORKBOOK` | `Invalid Emergency Report. Workbook could not be read.` | `Failed` / `Critical` | Yes |
| Uploaded file extension is not `.xlsx` | `FAILED` | `INVALID_EXTENSION` | `Invalid file type. Only .xlsx files are allowed.` | `Failed` / `Critical` | Yes |
| Unhandled/unexpected error (e.g. SharePoint store failure) | `FAILED` | `UNHANDLED_ERROR` | `Emergency Report processing did not complete.` | `Failed` / `Critical` | Yes — generic "Agent 3 - Emergency Report processing failed" alert |
| `PA Processed Mails` / `Agent 3 Alerts` mailbox evidence folders cannot be created/resolved | (does not affect the app response; logged separately) | — | — | — | Yes — `[Agent 3][FAILED] Mailbox Evidence Folder Setup Failed [EC:A3-MBOXFOLDER]`; the Emergency Report upload itself is still validated and processed independently of this housekeeping issue |

**Response fields returned to the Power App:**
- `responsestatus` — one of `SUCCESS`, `FAILED` (drives which `Notify()` banner style the app shows).
- `responsecode` — one of `OK`, `MISSING_WORKSHEET`, `INVALID_WORKBOOK`, `INVALID_EXTENSION`, `UNHANDLED_ERROR`.
- `responsemessage` — the exact human-readable text listed in the table above, shown verbatim in the app's notification.

**Key configuration parameters:** `EmergencyReportTargetFolder`, `EmergencyReportFileName`, `RequiredWorksheetName`, `SharedDMPMailbox`, `AlertEmailRecipient`, `MailImportanceError`, `Agent3AlertFolderName`/`ProcessedMailsRootFolderName`, `WorkflowPathAgent3`.

---

### 2.4 Agent 4 — Status Check

**Purpose:** Provides file-existence and count status for the Power App's Files band and related dashboard elements, system-wide audit health figures, and the actual most-recent Critical/Warning audit rows for the Audit Trail (Detail) screen.

**Trigger:** Called directly by the Power App (HTTP `Request` trigger) on load and periodically by an auto-refresh timer.

**Process:**
1. Loads active configuration via the Select+Join pattern.
2. Performs live SharePoint file/list-metadata and content checks for each monitored item (Emergency Report, Internal/External Domains lists, Counters list, Audit Trail, `IsRealDMP` indicator).
3. Reads the shared `Agent Audit Summary` table for all 6 agent rows and aggregates `AuditRunSummaryCount`, `AuditWarningCount`, `AuditFailedCount`.
4. Reads the central Audit Trail table directly and returns the most recent Critical rows (`auditrecentcritical`) and Warning rows (`auditrecentwarnings`), each projected to a compact shape (`timestamp`, `workflowpath`, `stepname`, `keyoutput`).
5. Returns all of the above, plus per-item existence flags, counts, last-modified timestamps and the current `OperationMode`, via a structured response (`RESPOND_Status`) to the Power App.

**Outcomes & status values:** Agent 4 performs no per-item business error branching of its own — it is a read-only reporting agent. After the response has been sent to the Power App, it always writes `CurrentStatus = Operational` / `StatusSeverity = Information` / `LastRunResult = Succeeded` to its own Agent Status row (`AgentKey = Agent_04`), and separately records its own run outcome in the Audit Trail as `Succeeded` if the `RESPOND_Status` action itself completed, or `Failed` otherwise (e.g. an unexpected connector/timeout failure) — this "no granular error branching" is by design, since any failure to read a monitored item is reflected as a `False`/absent value in the response body rather than as a distinct failure status.

**Key configuration parameters:** File name/folder parameters shared with Agent 1/2/3 (e.g. `EmergencyReportTargetFolder`, `EmergencyReportFileName`), `WorkflowPathAgent4`.

---

### 2.5 Agent 5 — Operational State Management

**Purpose:** Writes the Operating State toggle changes made in the Power App back to the central configuration (`CurrentOperationMode`).

**Trigger:** Called directly by the Power App (HTTP `Request` trigger) whenever either Operating State toggle (Environment or Operational Mode) is switched.

**Process:**
1. Loads active configuration via the Select+Join pattern.
2. Resolves/creates the `Agent 5 Alerts` mailbox evidence subfolder used for housekeeping.
3. Looks up the current `CurrentOperationMode` row.
4. Writes the new mode string (e.g. `"PROD_DMP"`) supplied by the app to that row's `CurrentValue`.
5. Buffers an audit event describing the switch (`"Switch from <old> to <new>"`, or `"No change - requested operational mode already active"` if no actual change occurred, or `"No mode change w/failure"` if the write failed).
6. Writes a run summary to the Audit Trail.
7. Updates its own Agent Status row.
8. Returns `success`, `auditoutcome`, `requestedaction`, and `newoperationmode` to the app.

**Outcomes & status values:**

| Condition | `AuditOutcome` | `CoreActionOutcome` | Agent Status `CurrentStatus` / `StatusSeverity` | `success` returned to app | Alert e-mail? |
|---|---|---|---|---|---|
| Operating State write succeeds | `Succeeded` | `Succeeded` | `Operational` / `Information` | `true` | No |
| Operating State write fails (SharePoint error) | `Failed` | `Failed` | `Failed` / `Critical` | `false` | No (no dedicated alert mail for this branch; surfaced via the app notification) |
| Write succeeds, but the `Agent 5 Alerts` mailbox evidence folder cannot be resolved/created afterward | `Failed` (housekeeping failure overrides the outcome shown on the dashboard) | `Succeeded` (the actual mode switch itself worked) | `Failed` / `Critical` | `true` (because `success` is computed as `AuditOutcome = 'Succeeded' OR CoreActionOutcome = 'Succeeded'`, so the app still reports success for the mode switch) | Yes — `[Agent 5][FAILED] Mailbox Evidence Folder Setup Failed [EC:A5-MBOXFOLDER]` to `AlertEmailRecipient` |

**Response fields returned to the Power App:**
- `success` — boolean; `true` if either the overall `AuditOutcome` or the underlying `CoreActionOutcome` (the mode switch itself) is `Succeeded`.
- `auditoutcome` — `Succeeded` or `Failed`, reflecting the full run including housekeeping.
- `requestedaction` — the operating-mode string that was requested (e.g. `"PROD_DMP"`).
- `newoperationmode` — the same requested mode string, echoed back for display confirmation.

**Key configuration parameters:** `WorkflowPathAgent5`, `AlertEmailRecipient`, `SharedDMPMailbox`, `Agent5AlertFolderName`, `ProcessedMailsRootFolderName`, `MailImportanceError`.

---

### 2.6 Agent 6 — Admin Functions

**Purpose:** Performs administrative test/reset actions triggered exclusively from the Power App's Admin Functions screen: mailbox cleanup (deletes the `PA Processed Mails` folder tree, matching by name prefix so stray duplicate folders are also cleaned up), reset of any one (or all) of the 4 e-mail-classification counters to 0, and reset of the Critical/Warning audit counter baselines shown as KPIs on the Cockpit. All counters live as rows (keyed by `Title`) in the `DMP Command Counters` SharePoint list, not in a separate workbook.

**Trigger:** Called directly by the Power App (HTTP `Request` trigger) after the operator confirms a Yes/Cancel dialog. `RequestedAction` is one of: `DeleteProcessedMailsFolderTree`, `ResetCounter_NoDMP`, `ResetCounter_InternalSender`, `ResetCounter_Effected`, `ResetCounter_NotEffected`, `ResetAllCounters`, `ResetCriticalCounterBaseline`, `ResetWarningCounterBaseline`, `ResetAllAuditCounterBaselines`.

**Process (counter reset):**
1. Reads the current value (`NumberProcessedEmails`) of the targeted counter row(s) from the `DMP Command Counters` SharePoint list.
2. Resets the value(s) to 0.
3. The result message (including the previous value) is written to the central Audit Trail as a normal run-summary row (`StepName = AdminAction`) — this run-summary row is the record of the reset.

**Outcomes & status values:**

| Condition | `ResultSuccess` | Agent Status `CurrentStatus` / `StatusSeverity` | Example `ResultMessage` |
|---|---|---|---|
| Requested counter/baseline successfully reset | `true` | `Operational` / `Information` | `Counter 'No DMP' reset from 42 to 0.` |
| `DeleteProcessedMailsFolderTree` finds no matching folders | `true` | `Operational` / `Information` | `No folder(s) matching 'PA Processed Mails*' found under Inbox - nothing to delete.` |
| `DeleteProcessedMailsFolderTree` finds and deletes one or more folders | `true` | `Operational` / `Information` | Lists the deleted folder names; `deletedcount` reflects how many were removed |
| Critical/Warning counter baseline reset | `true` | `Operational` / `Information` | `Critical counter reset - current lifetime total is <n>. The Cockpit's Critical KPI now shows 0 and will only increase again for new Critical events from this point on.` |
| `RequestedAction` value not recognized | `false` (default, unchanged) | `Failed` / `Critical` | `Unknown or unsupported admin action requested: '<value>'.` |

`ResultSuccess` defaults to `false` at flow start and is only ever set to `true` inside a matched, successfully-completed `Switch` case, so any unrecognized or partially-failed action is reported as a failure rather than silently doing nothing. All actions (successful or not) write a run-summary row to the Audit Trail and increment Agent 6's own row in `Agent Audit Summary`; Agent Status `CurrentStatus`/`StatusSeverity` are derived directly from `ResultSuccess` (`true`→`Operational`/`Information`, `false`→`Failed`/`Critical`).

**Response fields returned to the Power App:**
- `success` — boolean, taken directly from `ResultSuccess`.
- `message` — the human-readable `ResultMessage` shown in the app's confirmation/error notification.
- `requestedaction` — the exact `RequestedAction` value that was processed.
- `deletedcount` — integer count of mailbox folders actually deleted (only meaningful for `DeleteProcessedMailsFolderTree`; 0 for counter/baseline actions).

**Key configuration parameters:** Shares `SharedDMPMailbox` and `ProcessedMailsRootFolderName` with Agents 2/3/5. Counter and baseline rows are addressed by their `Title` value (e.g. `'No DMP'`) in the shared `DMP Command Counters` SharePoint list rather than via separate counter-file/table configuration parameters.

---

## 3. Power App (Cockpit) — User Guide

The app has 10 screens, reachable from the left sidebar: **Cockpit** (home/dashboard), **Agent Monitoring**, **System Health** (§3.13), **Help / Operational Manual**, **Audit Trail (Detail)**, **Configuration (Lists)**, **Maintenance**, **Task Occurrences**, **Admin Functions**, and **Release Notes** (reached from the Cockpit's version tag or the Maintenance screen's version link, not from a dedicated sidebar button).

**Note on this section's content:** Every field, button, colour, and message described below is mirrored in the app's own **Help / Operational Manual** screen (§3.3) — the two are maintained as a single source, kept in sync in both directions (see the AI working rules for this project).

### 3.1 Cockpit (home screen)

The Cockpit is the landing screen of DMP COMMAND. It is organized as a fixed left navigation sidebar plus a main working area on the right. The main area consists of a header bar, a row of status cards (System Health ring, Operating State + Maintenance-Domains + Files + Automation Status, Emails Processed ring), and a "Next Steps" checklist at the bottom. A thin coloured border frames the entire screen and changes with the current Operating State.

All live figures on this screen come from a single "Agent 4 (Status Check)" call that runs automatically when the screen opens, on a user-selectable auto-refresh timer, and when the operator clicks "Now". Until that call has succeeded at least once in the session, most values show placeholder text such as "Loading…", "?/6" or "N/A" instead of real numbers.

#### 3.1.1 Left navigation sidebar

A fixed 280px-wide panel on the far left, containing the application logo (which switches between a dark-mode and light-mode variant) and seven navigation buttons, stacked vertically:

| Button | Behaviour when clicked |
|---|---|
| **Cockpit** | Does not navigate (you are already here); shows the notification "Already on Cockpit" (Information). |
| **Agent Monitoring** | Navigates to the Agent Monitoring screen. |
| **Help / Operational Manual** | Navigates to the Operational Board / manual screen. |
| **Audit Trail (Detail)** | Navigates to the Audit Trail (Detail) screen. |
| **Configuration (Lists)** | Navigates to the Configuration screen. |
| **Maintenance** | Navigates to the Maintenance screen. |
| **Admin Functions** | Navigates to the Admin Functions screen. |

The currently active item ("Cockpit") is highlighted in solid green; the other six are shown in muted grey text and only highlight on hover.

#### 3.1.2 Header bar

A rounded, dark-purple banner across the top of the main pane. It contains:

- **Title/subtitle**: "DMP COMMAND" and, underneath it, "Communication Operations Management, Monitoring And Notification Dispatch".
- **Status line** (small text under the subtitle): while a refresh is running it reads **"⟳ Refresh ongoing…"** in green. Otherwise it reads either **"Not yet updated"** (optionally followed by **" (startup attempt 3/15)"** while the very first refresh of the session is still retrying, up to 15 attempts) or **"Updated: 14:32:07"** (the time of the last successful refresh). This is always followed by **"   ·   Auto-update off"** or **"   ·   Next update in 04:37"** (minutes:seconds countdown), depending on the auto-refresh setting.
- **Auto-refresh interval selector**: five small buttons — **Off**, **Now**, **2m**, **5m**, **10m**, **15m**. The currently active interval is highlighted in solid green; the others are translucent white.
  - **Off** stops the countdown (sets the interval to 0; the status line then shows "Auto-update off").
  - **Now** immediately triggers a full data refresh (identical to what happens automatically): it re-runs Agent 4 (Status Check), refreshes the `DMP Command Agent Status`, `DMP Command Configuration`, `DMP Command Internal Domains`, `DMP Command Counters` and `DMP Command External Domains` lists, and recalculates every KPI, ring and dot on the page. This button is highlighted in green (and slightly more prominent) whenever no successful refresh has happened yet this session.
  - **2m / 5m / 10m / 15m** each set the auto-refresh interval to that many minutes and restart the countdown; the same background refresh logic then fires automatically every time the countdown reaches zero.
- **KPI strip** (three clickable tiles), left to right:

  | Tile | Number shown | Colour rule | Click destination |
  |---|---|---|---|
  | **CRITICAL** | The number of failed audit entries currently above the session's critical baseline (0 if none) — concretely 0, 1, 3, 12… | Green when the number is 0; red when it is greater than 0 | Audit Trail (Detail) |
  | **WARNINGS** | The number of warning audit entries currently above the session's warning baseline — 0, 2, 7… | Green when 0; orange when greater than 0 | Audit Trail (Detail) |
  | **AGENTS ACTIVE** | "?/6" before the first successful refresh, otherwise "X/6" where X is the count of the 6 monitored health items currently OK (Emergency Report, Internal Domains, External Domains, Counter, Audit Trail, Agent 6) — e.g. "6/6", "4/6", "0/6" | Grey while never refreshed; green when X = 6; orange when X < 6 | Agent Monitoring |

  Each tile also carries a small round **blue "change" dot** above it that blinks (alternating solid/translucent blue roughly once per second) whenever that tile's underlying count has changed since the operator last looked at it (i.e. last clicked that tile). Clicking anywhere on the tile (number, label, or the invisible larger hit-area around it) acknowledges the current count — the blue dot disappears — and navigates to the destination screen shown above.
- **Version tag**: a small green pill showing the running app version string (e.g. "v3.4.2"); clicking it opens the Release Notes screen.
- **Dark/Light mode toggle**: labelled "DARK" on the left and "LIGHT" on the right of a switch; switching it immediately re-colours the whole app (background, panel fills, text colours) between dark and light theme. This is a purely visual, client-side setting with no server call, and is not persisted between sessions or per-user configurable (see §7 Known Limitations).

#### 3.1.3 System Health ring ("Heartbeat" card, far left of the row below the header)

A square card bordered in green containing a circular "donut" gauge:

- **Centre text**: a percentage (e.g. "0%", "50%", "83%", "100%") in large bold digits, or **"N/A"** in grey before the first successful refresh. Underneath it, a small caption that reads **"SYSTEM HEALTH"** normally, or **"REFRESHING..."** while a refresh call is in flight.
- **Segmented ring**: each health input has an equal-sized arc. The fixed inputs are Status Check, Critical Events, Warnings, Operating State, Emergency Report processing, Internal Domains, External Domains, Counter, and Audit Trail. In addition, every row in `DMP Command Agent Status` whose `AgentKey` starts with `Agent_` receives its own segment. This is dynamic: Agent 7 and every future agent automatically add a segment once their status row exists.

  | Health percentage | Ring/number colour | Meaning |
  |---|---|---|
  | Never refreshed | Grey, "N/A" | No data yet |
  | All segments green | Green | System healthy |
  | At least one non-green segment | Orange | Degraded; inspect the segment/legend |
  | No segment green | Red | Critical |

- Clicking anywhere on the ring navigates to the dedicated **System Health** screen (§3.13; same destination as the "System Health" sidebar entry), which lists all of the health inputs below and their live status without a size/scroll limit (before `v1.22.30` this was a cramped 300px-tall pop-up that could not show all 16 items readably — user feedback led to moving it to its own page):

  | Row text | Dot colour rule |
  |---|---|
  | Status Check | Green after a successful Agent 4 status call; red when that call fails |
  | Agent 1–Agent N | Green only when `CurrentStatus` is `Operational`; red for every other status. A newly added Agent 7 receives its own segment automatically. |
  | Critical Events | Green at 0; red when the total Critical count is greater than 0 |
  | Warnings | Green at 0; orange when the total Warning count is greater than 0 |
  | Operating State | Green normally; orange while switching; red after a failed operating-mode or environment switch, until a later switch succeeds |
  | Emergency Report processing | Green when no processing error exists; orange while an upload is processed; red only after an upload/processing failure. A missing report is not an error. |
  | Internal Domains, External Domains, Counter, Audit Trail | Green when reachable; red when missing or unavailable |

#### 3.1.4 Operating State panel

A green-bordered card titled **"OPERATING STATE"**, showing:

- **Mode line**: one of four texts — **"PRODUCTION - DMP Operation"**, **"PRODUCTION - Normal non-DMP Operation"**, **"SIMULATION - DMP Operation"**, **"SIMULATION - Normal non-DMP Operation"** — corresponding to the single configuration value `CurrentOperationMode` (`PROD_DMP`, `PROD_NODMP`, `SIMU_DMP`, `SIMU_NODMP`). Colour: red for PROD+DMP, green for PROD+Normal, purple for SIMU+DMP, grey for SIMU+Normal. While a switch is in progress the text fades in and out (blink) instead of staying solid.
- **LAST CHANGED**: either **"Not changed this session"** or a timestamp/user string such as **"14:32 · Jane Doe"**, updated the moment either toggle below is successfully changed.
- **MODE toggle** (labelled **Normal** / **DMP**): switching it calls Agent 5 (Operational State Management).
  - Success switching to DMP: notification **"Operating State switched to DMP mode."** (Success).
  - Success switching to Normal: notification **"Operating State switched to Normal mode."** (Success).
  - Failure (either direction): the toggle snaps back to its previous position and the notification reads **"Failed to switch Operating State. AuditOutcome=&lt;value&gt;."** (Error), where `<value>` is whatever outcome code Agent 5 returned, or "unknown" if none was returned.
- **ENVIRONMENT toggle** (labelled **SIMU** / **PROD**): blue when set to "SIMU", red when set to "PROD". Switching it also calls Agent 5 (Operational State Management).
  - Switching to PROD always forces Operational Mode back to Normal for safety; if it had been DMP, the success notification reads **"Environment switched to PROD (Operational Mode automatically reset to Normal for safety)."**; otherwise **"Environment switched to PROD."** (Success).
  - Switching to SIMU: notification **"Environment switched to SIMU."** (Success), and the Operational Mode toggle keeps its current Normal/DMP position.
  - Failure (either direction): the toggle snaps back and the notification reads **"Failed to switch Environment. AuditOutcome=&lt;value&gt;."** (Error).
- **Two small round LED dots**, one next to each toggle, each with four possible states:

  | LED colour | Meaning |
  |---|---|
  | Grey | Idle baseline / not yet touched this session |
  | Yellow, blinking | The switch request is currently being processed by Agent 5 |
  | Green | The last switch attempt succeeded (current state confirmed) |
  | Red, blinking | The last switch attempt failed |

There is no four-eyes/dual-approval confirmation before either toggle takes effect (see §7 Known Limitations).

#### 3.1.5 Maintenance – Domains panel

A green-bordered card titled **"MAINTENANCE - DOMAINS"**, directly below the Operating State panel, with two rows:

- **INTERNAL row**: a status dot, a bold count, the label "INTERNAL", and two buttons.
  - The **count** is the number of currently active rows in the `DMP Command Internal Domains` SharePoint list (rows where the Active flag is "Yes") — e.g. 0, 12, 340.
  - The **dot**: grey (never refreshed) → orange, blinking (refresh in progress) → green if the Internal Domains data source exists/loaded → red if it is missing.
  - **View** opens the `DMP Command Internal Domains` SharePoint list (AllItems view) in the browser.
  - **Edit** opens the same list's "new item" form (AllItems/NewForm.aspx) in the browser, so the operator can add or maintain domain entries directly in SharePoint.
- **EXTERNAL row**: a status dot, a bold count, the label "EXTERNAL", two buttons, and a "Replace" upload control.
  - The **count** is the current number of external domains loaded (e.g. 0, 340).
  - The **dot** reflects the Emergency Report upload/processing status rather than pure existence: green (idle/last upload OK), yellow and blinking (an upload is currently being processed by Agent 3), red and blinking (the last upload attempt failed).
  - **View** opens the `DMP Command External Domains` list (AllItems) in the browser; **Edit** opens its NewForm.aspx.
  - **Replace** (blue-bordered) lets the operator attach a single file to upload a new Emergency Report, which routes to Agent 3 (Emergency Report Management) to regenerate the External Domains list. Its tooltip reads: "Upload a new Emergency Report - triggers Agent 3 to regenerate External Domains."
    - If the selected file's name does not end in **.xlsx** (case-insensitive), the picker rejects it, clears the selection, and shows the notification **"Only .xlsx files are allowed."** (Error). This is normal, designed validation, not an error condition.
    - If the file is a valid .xlsx, the status dot on the External row turns yellow and starts blinking, and the notification **"Emergency Report submitted for processing. Agent 3 will regenerate External Domains."** (Information) appears immediately while the upload runs in the background.
    - When Agent 3 finishes: on success, the dot turns green, a success notification appears (the exact text returned by Agent 3, or the default **"Emergency Report uploaded successfully."** if none is supplied), and the screen performs a full data refresh (all counts, dots and rings update). On failure, the dot turns red and an error notification appears (Agent 3's own failure message, or the default **"Emergency Report processing failed."**, or — if Agent 3 did not respond at all — **"Agent 3 did not return a processing result."**). In every case the file picker is cleared afterwards so a new file can be attached.
  - After a successful Replace, the operator should subsequently run Agent 1 (Domains Extraction) to regenerate the domain lists from the newly stored report.

#### 3.1.6 Files band

A green-bordered card titled **"FILES"**, with column headers "STATUS" and "LAST UPDATED", listing two data sources (each with its own status dot, name, status text, and timestamp):

| Row | Status text possibilities | Timestamp shown |
|---|---|---|
| **Emergency Report** | "Loading…" (before first refresh) / "Operational" (exists) / "Missing" (does not exist) | e.g. "04.09.2026 09:12:03 CEST/CET", or "–" if unknown |
| **Audit Trail File** | Same three possibilities | Same timestamp format, or "–" if unknown |

Each row's dot follows the same four-state rule used elsewhere: grey (never refreshed) → orange, blinking (refresh in progress) → green (exists/operational) → red (missing).

#### 3.1.7 Automation Status panel

A green-bordered card titled **"AUTOMATION STATUS"**, below the Files band, with column headers "SOURCE" and "ACTIVE", listing six automated data sources with a status dot, name, source type, and a live count/state:

| Row | Source | "Active" text | Dot colour rule |
|---|---|---|---|
| **Agent 4 Live Status** | Flow | "Loading..." while refreshing; otherwise "Operational" or "Error" | Grey (never refreshed) → orange, blinking (refreshing) → green (last call succeeded) → red (last call to Agent 4 failed) |
| **Internal Domains** | SharePoint | "N active" — the count of active rows in `DMP Command Internal Domains` (e.g. "12 active") | Grey → orange, blinking → green (never turns red here) |
| **Config Parameters** | SharePoint | "N active" — the count of active rows in `DMP Command Configuration` (e.g. "8 active") | Grey → orange, blinking → green |
| **Agent Status** | SharePoint | "N entries" — the total row count of `DMP Command Agent Status` (e.g. "142 entries") | Grey → orange, blinking → green |
| **Counter** | SharePoint | "N rows" — the total row count of `DMP Command Counters` (e.g. "365 rows") | Grey → orange, blinking → green |
| **External Domains** | SharePoint | "N domains" — the total row count of `DMP Command External Domains` (e.g. "340 domains") | Grey → orange, blinking → green |

Only the Agent 4 row's dot can turn red (it reflects whether the last Agent 4 call itself failed); the five SharePoint-backed rows simply show green once any successful refresh has occurred, since they reflect list contents rather than connectivity health.

#### 3.1.8 Emails Processed ring (far right of the row)

A square card, mirroring the System Health card, containing a segmented circular gauge broken into up to four coloured arcs plus a centre total:

- **Centre text**: the total number of processed e-mails (No DMP + External + Internal Sender + Not Effected combined), with the caption **"EMAILS PROCESSED"** underneath, or **"REFRESHING..."** while a refresh is running. Before the first successful refresh the ring is shown as a flat grey circle with the text **"Loading..."**.
- **Segment colours and meaning**:

  | Segment colour | Category | Meaning |
  |---|---|---|
  | Blue | **No DMP** | E-mails processed while no DMP event was active |
  | Red | **External** | E-mails classified as involving an external sender |
  | Purple | **Internal Sender** | E-mails classified as involving an internal sender |
  | Green | **Not Effected** | E-mails not affected/impacted by the DMP process |

  Segment sizes are proportional to each category's share of the total (e.g. a category with 0 e-mails shows no visible arc).
- Clicking the ring opens/closes a **legend popup** titled "EMAILS PROCESSED - LEGEND" (with a "✕" close button) listing each category with its colour swatch, absolute count and percentage of the total, e.g.:
  - "No DMP (120 / 35.00%)"
  - "External (80 / 23.00%)"
  - "Internal (60 / 17.00%)"
  - "Not Effected (90 / 25.00%)"

  (Numbers and percentages are illustrative; the app always fills in the live counts and computes the percentage of the four-category total, treating the total as at least 1 to avoid a division-by-zero display.)

#### 3.1.9 Next Steps panel

A full-width card at the bottom of the Cockpit, titled **"NEXT STEPS · CoS Leader · Default Management Process"**. As of `v1.22.33`, this is no longer the old static Agent-4-milestone display - it now reads live from the `DMP Command Checklist CoS Leader` list (joined by `Titel`/TaskID with `DMP Command Checklist Overall Process` for `SeqNo`/`PredecessorTaskIds`/`IsMilestone`), per the "Next Steps" feature specification (`DMP_COMMAND_Next_Steps_Anforderung.md`). **Phase 1 scope: CoS Leader sub stream only** - Infrastructure Team and Content Team are not yet represented (their checklists do not exist yet, see Backlog Priorität 2 Punkt 1).

When no DMP case is active (`varOperationalMode = "Normal"`), the panel shows a single idle line: **"No active DMP case - Default Management Process is currently idle."** Otherwise it shows up to 5 recently completed tasks (sorted by confirmation time, most recent first) followed by up to 8 open tasks (sorted: ongoing first, then ready/executable, then not-yet-ready, each group by `SeqNo`). Each row shows a status dot, the task ID + description (milestone tasks prefixed with a ★), and a right-aligned status word:

| Dot colour | Status word | Meaning |
|---|---|---|
| Green | COMPLETED | `Status = Done` |
| Orange, slow-blinking | ONGOING | `Status = Ongoing` |
| Orange, steady | PENDING | `Status = Not Started`, required DMP phase reached, and all `PredecessorTaskIds` are `Done` - task is executable now |
| Grey | NOT STARTED | `Status = Not Started` but not yet executable (phase not reached and/or a predecessor is not yet `Done`) |

A task's required phase is read from its own `Phase` column (`Pre-Default`/`DMP (Termination & Liquidation)`/`Post-Default`, compared against the app's current `varOperationalStepCounter`). **Known Phase 1 limitations (by design, not a bug):** no red "overdue/problem" state yet (the checklist has no due-date column - only `Task Occurrences`, a separate feature for recurring tasks, has `DueUtc`); no in-app propose/confirm action yet (status changes still happen directly in SharePoint); cross-sub-stream predecessors are not possible yet (both the depending task and its predecessor must currently be on the CoS Leader checklist, since no other sub stream checklist exists). A refresh runs on every Cockpit visit and every 30 seconds thereafter (`tmrCosLeaderNextStepsRefresh`), independent of the Agent 4 status-check cycle.

#### 3.1.10 Ambient status border (screen frame)

A thin coloured border runs around all four edges of the entire Cockpit screen, independent of any single panel, giving an at-a-glance ambient indicator of the current Operating State even when looking at the screen from a distance:

| Environment | Operational Mode | Border colour | Border thickness |
|---|---|---|---|
| PROD | DMP | Red | 9 px (thickest — highest-alert state) |
| SIMU | DMP | Purple | 6 px |
| PROD | Normal (non-DMP) | Green | 4 px |
| SIMU | Normal (non-DMP) | Grey | 3 px (thinnest — lowest-alert state) |

This border updates immediately whenever the Operating State or Environment toggles (see 3.1.4) are successfully switched.

### 3.2 Agent Monitoring screen

#### Purpose and layout
The Agent Monitoring screen gives a friendlier, per-agent alternative to reading the raw `DMP Command Agent Status` SharePoint list directly. It opens from the Cockpit and shows six identical status tiles — one per agent — arranged in two rows of three: **Row 1** = Agent 1, Agent 2, Agent 3; **Row 2** = Agent 4, Agent 5, Agent 6.

**Header bar** (top, dark purple):
- **"< Back to Cockpit" button** — returns to the Cockpit screen (`scrHome`) without a screen-transition animation.
- **"Agent Monitoring"** — page title.
- **"DMP COMMAND - detailed per-agent health and metrics"** — page subtitle.

When the screen is opened, it automatically refreshes the `DMP Command Agent Status` list before showing data (see "Loading and refreshing behaviour" below).

#### Each agent tile — what it shows
All six tiles (Agent 1–6) have an identical layout and are each bound to the one row in `DMP Command Agent Status` whose `AgentKey` matches that agent (`Agent_01` … `Agent_06`). Every tile has a fixed green accent border (decorative only — its color does not change with status).

| Element | Content | Source / behaviour |
|---|---|---|
| Status dot (small circle, top-left of tile) | Colored indicator | See "Status dot colors" table below |
| Agent name label | `"Agent <n> - " & AgentDisplayName"` — e.g. `"Agent 1 - Domains Extraction"` | `AgentDisplayName` field of the agent's row in `DMP Command Agent Status`. Always displayed in green text, regardless of status. |
| Status line | `"Status: " & CurrentStatus"` — e.g. `"Status: Operational"` | The `CurrentStatus` choice value of the agent's row. |
| Last run line | `"Last run: <date> <time> - <result> (<duration>s)"` | See "Last run line" details below |
| Status message | Free-text line under the last-run line | The `StatusMessage` field of the agent's row; shown blank if not set |

**Status dot colors** (exhaustive):

| Condition | Color | Meaning |
|---|---|---|
| The screen has not yet completed its first data load | Grey | Not-yet-loaded / initializing state (transient, before the first `OnVisible` refresh completes) |
| A refresh of `DMP Command Agent Status` is currently in progress | Amber, blinking (alternates between solid amber and 25%-opacity amber) | Data is being refreshed right now — value on screen may be momentarily stale |
| Refresh complete and the agent's `CurrentStatus` = `"Operational"` | Green | Agent is healthy / operating normally |
| Refresh complete and `CurrentStatus` is anything other than `"Operational"` | Red | Agent is reporting a non-operational state and needs attention |

**Last run line — exact format and placeholders:**
The line reads: `Last run: <LastRunTimestamp formatted as dd.mm.yyyy hh:mm:ss> - <LastRunResult> (<LastRunDurationSec formatted to one decimal>s)`.
- If `LastRunTimestamp` is blank, the date/time portion shows `-` instead of a timestamp.
- If `LastRunResult` is blank, it shows `-` instead of a result word.
- If `LastRunDurationSec` is blank, the duration shows `-` instead of a number.
- Example with data: `Last run: 04.09.2026 09:15:32 - Succeeded (12.3s)`.
- Example before any run has ever been recorded: `Last run: - - - (-s)`.

**Status message line:** shows the free-text `StatusMessage` value written by the agent (e.g. an error detail or a short note); it is simply blank (no placeholder text) if the agent has not written a message.

#### Loading and refreshing behaviour
When the screen becomes visible, it marks itself as refreshing (triggering the blinking-amber dot state on all six tiles), reloads `DMP Command Agent Status` from SharePoint, then clears the refreshing flag and marks the screen as "ever loaded." In practical terms:
- On first visit in a session, all six dots briefly show grey, then blink amber while the list is fetched, then settle to green/red based on each agent's actual `CurrentStatus`.
- On subsequent visits, dots blink amber during the reload and then settle again — there is no separate manual "Refresh" button on this screen; refreshing happens automatically each time the screen is opened.

### 3.3 Help / Operational Manual screen

An in-app screen (`scrOperationalBoard`) presenting the same content as this document, organized as a left-hand sidebar listing every section (Intro, General Colours, Top Bar, Operating State, Maintenance Domains, Files, System Health, Emails Processed, Automation Status, Next Steps, Sidebar, and one entry per additional screen) plus a detail pane on the right that shows the selected section's header and full explanatory text. Clicking a sidebar entry updates the detail pane immediately (no separate "open" step). Reachable from the left navigation sidebar at any time via **"< Back to Cockpit"** to return. There is currently no F1 keyboard shortcut (not reliably supported by the canvas app platform — browsers intercept/ignore F1 before the app can react to it).

### 3.4 Audit Trail (Detail) screen

#### Purpose and layout
This screen gives operators a live, detailed view into the central Audit Trail without opening the underlying `AuditTrail.xlsx` file, showing the 10 most recent Critical (Failed) and 10 most recent Warning rows, both read directly by Agent 4 (Status Check).

**Header bar** (top, dark purple):
- **"< Back to Cockpit" button** — returns to the Cockpit screen (`scrHome`) without a screen-transition animation.
- **"Audit Trail (Detail)"** — page title.
- **"DMP COMMAND - detailed audit trail view"** — page subtitle.
- A hidden 1-second interval timer drives the shared auto-refresh countdown used across the app; when the countdown reaches zero it triggers a fresh call to Agent 4 (Status Check) and updates all the counts and rows described below, then restarts the countdown. (If the auto-refresh interval setting is 0 or less, this periodic auto-refresh is switched off.)

Opening this screen also immediately triggers the same shared, central refresh mechanism used by the Cockpit's periodic auto-refresh timer (rather than a separate call) — so the summary counts and recent-row lists are up to date as soon as the tab opens, without waiting for the next scheduled tick.

#### Summary panel (top card, green-bordered)
| Element | Text | Meaning |
|---|---|---|
| Critical total | `"Critical (total): " & <count>` (red text) | Running total of Critical/Failed rows in the Audit Trail (`auditfailedcount`, as returned by Agent 4); shows `0` if not yet loaded |
| Warnings total | `"Warnings (total): " & <count>` (amber text) | Running total of Warning rows in the Audit Trail (`auditwarningcount`); shows `0` if not yet loaded |
| Total run-summary rows | `"Total run-summary rows: " & <count>` | Total number of run-summary entries in the Audit Trail (`auditrunsummarycount`); shows `0` if not yet loaded |
| Hint text | `"The lists below show the 10 most recent rows of each kind, read live from the central Audit Trail on every Cockpit refresh."` | Explains the scope of the two lists further down the screen |
| "New since reset" line | `"New since reset - Critical: " & <n> & "   Warnings: " & <n>` | Shows how many Critical / Warning rows have appeared **since the corresponding counter was last reset** (current total minus the saved baseline, never shown negative) |
| "Loading recent alerts..." (amber text) | — | Only visible while a refresh call is in progress (triggered by the periodic timer, the "Refresh now" button, or any of the three reset buttons) |

**Buttons in the summary panel:**

| Button | Action | Notification shown |
|---|---|---|
| **Reset Critical** | Calls Agent 6 (Admin Functions) with action `ResetCriticalCounterBaseline`, which resets the "new since reset" baseline for Critical rows to the current Critical total | On success: the message text returned by Agent 6. On failure to reach Agent 6: `"Reset call failed: " & <technical error text>` |
| **Reset Warning** | Calls Agent 6 (Admin Functions) with action `ResetWarningCounterBaseline`, resetting the Warning baseline | Same pattern as above |
| **Reset All** | Calls Agent 6 (Admin Functions) with action `ResetAllAuditCounterBaselines`, resetting both baselines at once | Same pattern as above |
| **Refresh now** | Immediately re-calls Agent 4 (Status Check) and refreshes all counts and both recent-row lists | On success: `"Audit Trail refreshed."`. On failure: `"Refresh failed: " & <technical error text>` |
| **Open full Audit Trail file** | Opens `AuditTrail.xlsx` directly in SharePoint (in a new browser tab), for anything beyond the most recent 10 Critical/Warning rows shown on this screen | — (no in-app notification; it is an external link) |

All three reset actions and the "Refresh now" action briefly show the amber "Loading recent alerts..." text while running.

#### Recent Critical (Failed) rows panel
Left-hand card, titled **"Recent Critical (Failed) rows"** (red border and title text). It lists up to 10 rows, most recent first, taken from Agent 4's `auditrecentcritical` result set.
- If no Critical rows exist in the Audit Trail, the panel shows the single line: **"No Critical rows found in the Audit Trail."**
- Otherwise, rows 1–10 are shown one per line, each only appearing if the data actually contains that many rows (row 7 is hidden if there are only 6 Critical rows, etc.). Each visible row is formatted as:

  `<timestamp> UTC - <WorkflowPath (or StepName if WorkflowPath is blank)> - <KeyOutput>`

  For example, a row could read: `2026-09-04 07:12:45 UTC - Agent2/InboxTreatment - Message forwarded to case team`.
  - **Timestamp**: the row's `TimestampUtc` value from the Audit Trail, shown as `yyyy-mm-dd hh:mm:ss UTC`.
  - **Workflow/Step**: the `WorkflowPath` column value; if that is blank, the `StepName` column value is shown instead.
  - **Key output**: the `KeyOutput` column value (a short description of the step's outcome), blank if not present.

#### Recent Warning rows panel
Right-hand card, titled **"Recent Warning rows"** (amber border and title text). Structurally and functionally identical to the Critical panel above, but sourced from Agent 4's `auditrecentwarnings` result set (StepStatus = Warning rows from the Audit Trail).
- If no Warning rows exist, the panel shows: **"No Warning rows found in the Audit Trail."**
- Otherwise, up to 10 rows are shown, newest first, in the exact same format as the Critical panel: `<timestamp> UTC - <WorkflowPath or StepName> - <KeyOutput>`.

Both panels scroll independently if their content exceeds the visible card height.

### 3.5 Configuration (Lists) screen

#### Purpose and layout
This screen gives direct, at-a-glance access to the SharePoint lists that back DMP COMMAND. No data is edited in-app — all editing happens in SharePoint via the "View" / "New Entry" links. The screen covers **all 17 live-connected lists**, organized into a scrollable body with 6 thematic sections. As of `v1.22.31`, each section was a single bordered card holding one compact row per list; as of `v1.22.32`, each card additionally has a solid dark-purple header bar with white bold text (previously the section title was plain colored text on the card background, reported as not standing out), and the per-list rows were replaced with a compact tile grid (up to 4 tiles per row within each card — every section has at most 4 lists, so each section is now exactly one tile row, with a uniform 140px card height regardless of list count). Each tile has a small reachability dot (`IfError(CountRows(list)>=0, green, red)`), the list name, a live row-count label, and View / New Entry buttons. Each list's description text (what it is used for) is in the View button's tooltip - hover it to read the description. Each tile's count reads either `"Active rows: " & <count>` (lists with an `Active` Yes/No choice column — currently Configuration, Internal Domains, External Domains) or `"Total rows: " & <count>` (all other lists, which have no such flag).

**Header bar** (top, dark purple):
- **"< Back to Cockpit" button** — returns to the Cockpit screen (`scrHome`) without a screen-transition animation.
- **"Configuration (Lists)"** — page title.
- **"DMP COMMAND - configuration and list management"** — page subtitle.

#### Section 1 — Core Runtime Configuration & Monitoring
- **DMP Command Configuration** — runtime configuration parameters used by the agents (Mode, Environment, thresholds, ...); Active rows count; View + New Entry.
- **DMP Command Agent Status** — live status/health rows written by each agent (see Agent Monitoring for a friendlier view); Total rows count; View only (agents write these rows automatically).
- **DMP Command Counters** — shared reference-number and cross-agent counters (see Admin Functions, §3.7, for resets); Total rows count; View only.

#### Section 2 — Domain Classification
- **DMP Command Internal Domains** — domains treated as internal by Agent 1/Agent 2's classification logic; Active rows count; View + New Entry.
- **DMP Command External Domains** — domains treated as external by Agent 1/Agent 2's classification logic; Active rows count; View + New Entry.

#### Section 3 — Streams: Process & Checklists
- **DMP Command Checklist Overall Process** — master status overview linking each Streams sub-checklist by `TaskID`; Total rows count; View + New Entry.
- **DMP Command Checklist CoS Leader** — CoS Leader working checklist; status changes require four-eyes confirmation; Total rows count; View + New Entry.

#### Section 4 — Streams: Case & Scheduling
- **DMP Command Default Case Context** — case-specific context (defaulted member, termination reason/date) captured via the Cockpit's B3 popup on the Pre-Default → DMP transition; Total rows count; View + New Entry.
- **DMP Command Checklist Recurrence Rules** — rules driving Agent 7's automatic generation of recurring Task Occurrences; Total rows count; View + New Entry.
- **DMP Command Checklist Task Occurrences** — recurring task instances (see the dedicated Task Occurrences screen for the working Propose/Approve/Reject view); Total rows count; View only.
- **DMP Command Status Change Approvals** — four-eyes approval records for checklist status changes across all Streams sub-checklists; Total rows count; View only.

#### Section 5 — Streams: Communication
- **DMP Command Email Templates** — parameterized `{{placeholder}}` e-mail templates used by the Streams checklists; Total rows count; View + New Entry.
- **DMP Command Email Placeholders** — catalog of valid `{{placeholder}}` keys used in Email Templates; Total rows count; View + New Entry.
- **DMP Command Recipient Groups** — collective mailbox addresses referenced by Email Templates; Total rows count; View + New Entry.

#### Section 6 — Streams: Access Control & Catalogs
- **DMP Command Role Assignments** — per-person screen/action permissions for the Streams checklists; Total rows count; View + New Entry.
- **DMP Command Screens Catalog** — catalog of selectable screen names used by Role Assignments; Total rows count; View + New Entry.
- **DMP Command Actions Catalog** — catalog of selectable action names used by Role Assignments; Total rows count; View + New Entry.

All "View" and "New Entry" links behave identically across every row: they use `Launch()` to open the corresponding SharePoint list page in the user's default browser; no confirmation dialog or in-app notification is shown, and no data is refreshed on return — reopening or refreshing the Configuration screen will pick up any changes made in SharePoint. **Note on link targets:** six of the Streams lists (`Checklist CoS Leader`, `Default Case Context`, `Status Change Approvals`, `Email Templates`, `Recipient Groups`, `Role Assignments`) were renamed after creation — their SharePoint-internal URL segment is still the old pre-rename working name, not the current display title, so their links point to that internal name (confirmed against the live `.msapr`'s data source references, not guessed).

### 3.6 Maintenance screen

The Maintenance screen is reached from the Cockpit's "Maintenance" tile/link and gives operators an overview of version numbers and one-click access to the three administrative web portals behind DMP COMMAND. It uses the same dark-purple rounded header bar as every other screen, followed by two stacked, green-bordered "card" panels. **As of `v1.22.32`, the former "Connection diagnostics" panel (Panel 1 below) was moved to the System Health (Details) screen (§3.13)**, where it now covers all 17 connected SharePoint lists instead of just 3 - per user feedback that connection diagnostics belongs functionally to System Health.

#### Header bar

| Element | Description |
|---|---|
| **"< Back to Cockpit" button** | Top-left of the header. Returns immediately to the Home/Cockpit screen (no confirmation needed, non-destructive navigation). |
| **Title: "Maintenance"** | Static page title, white bold text. |
| **Subtitle** | Static text: *"DMP COMMAND - versions and admin links"* — a one-line summary of what the screen contains. |

#### Panel 1 — "Versions"

| Element | Description |
|---|---|
| **"Cockpit app: `<version>` (see Release Notes)" button** | A text-style (borderless, green) button. The version portion is the app's own current version number (the same value shown on the Release Notes screen's subtitle, e.g. "1.22.12"; it can appear blank if the value has not yet been loaded). Clicking it navigates directly to the Release Notes screen. Tooltip on hover: "Open Release Notes". |
| **Agent versions hint** | Static gray text: *"Agent versions - see Release Notes > Agent Changes, or Agent Monitoring for live per-agent status"* — tells the operator where to find per-agent (Agent 1–6 flow) version numbers, since this screen itself does not list them individually. |

#### Panel 2 — "Admin portal links"

Three equally-sized, green-outlined pill buttons in a row, each opening an external site in a new browser tab/window via a direct link (no confirmation, non-destructive, read access only):

| Button text | Destination opened |
|---|---|
| "Power Automate flows" | The Power Automate flow list for the DMP COMMAND environment (where Agents 1–6 live as cloud flows). |
| "SharePoint site" | The team's SharePoint site hosting the mailbox/document libraries and lists used by the app. |
| "Power Apps maker portal" | The maker portal home for the same environment, used to open/edit the app itself. |

### 3.7 Admin Functions screen

The Admin Functions screen groups every destructive, operator-triggered maintenance action in one place. It uses the standard header bar ("< Back to Cockpit", title "Admin Functions", subtitle *"DMP COMMAND - administrative reset operations (destructive, use with care)"*), followed by a body section titled "Admin Functions" with the hint text *"These actions are destructive and cannot be easily undone. Use with care."* Below that sit three bordered sub-panels (mailbox cleanup, counter resets, diagnostics) and, at the very bottom, a shared result message line. All destructive actions call **Agent 6 (Admin Functions)** as `'DMPAgent6(AdminFunctions)'.Run(User().Email, "<RequestedAction>")`.

#### Two-step confirm pattern (applies to every destructive action)

Every destructive action uses the same "arm, then confirm" pattern:
1. **First click** on the red action button only *arms* the action — it sets a flag/variable and reveals an inline confirmation box directly beneath the row (no flow call happens yet, nothing is changed).
2. The confirmation box shows an explanatory sentence plus two buttons, **"Yes, `<action>`"** (red) and **"Cancel"** (outlined, neutral).
   - **Cancel** simply hides the confirmation box again; no data is touched and no flow is called.
   - **"Yes, …"** calls Agent 6 with the specific `RequestedAction`, hides the confirmation box, and shows a toast notification built from the flow's response: green (`NotificationType.Success`) if `varAdminActionResult.success` is true, red (`NotificationType.Error`) otherwise. The same response's `message` text is also written into the result line at the bottom of the screen.

#### Sub-panel 1 — "MAILBOX CLEANUP" (red-bordered)

| Element | Description |
|---|---|
| **Row label** | "Delete 'PA Processed Mails' folder tree" |
| **"Delete" button** | First-click step: arms the confirmation box (`varConfirmDeleteMailboxFolders = true`). |
| **Confirmation text** | *"Are you sure? This deletes 'PA Processed Mails' and everything below it (including 'Agent 5 Alerts') in the shared mailbox. This cannot be easily undone."* |
| **"Yes, delete"** | Calls Agent 6 with `RequestedAction = "DeleteProcessedMailsFolderTree"`. On success shows a green toast with the flow's own success message; on failure a red toast with the flow's own error message. The message text also stays visible in the bottom result line until the next action runs. |
| **"Cancel"** | Hides the confirmation box; no folders are deleted. |

#### Sub-panel 2 — "RESET E-MAIL COUNTERS" (red-bordered)

Hint text under the panel title: *"The previous value is recorded in the Audit Trail (Operational History) before each reset."* Five red buttons in a row, each of which arms the **same shared** confirmation box (only one reset can be pending at a time) with a button-specific action code and label:

| Button text | `RequestedAction` sent to Agent 6 | Label used inside the confirmation sentence |
|---|---|---|
| "No DMP" | `ResetCounter_NoDMP` | "No DMP" |
| "Internal Sender" | `ResetCounter_InternalSender` | "DMP internal Sender" |
| "External" | `ResetCounter_Effected` | "DMP effected Member (External)" |
| "Not Effected" | `ResetCounter_NotEffected` | "DMP not effected Sender" |
| "Reset ALL" (darker red, bold — visually marked as the most impactful option) | `ResetAllCounters` | "ALL counters" |

Confirmation text shown (with the label substituted, e.g. for "Reset ALL"): *"Are you sure? This resets the 'ALL counters' counter(s) to 0. The current value is written to the Audit Trail (as an Operational History record) before the reset, so it is not lost."*

- **"Yes, reset"**: calls Agent 6 with the pending `RequestedAction`. If the call itself cannot be completed (e.g. connectivity error), the screen builds its own fallback result locally (a "call failed" message together with the technical error text) and shows it as a red toast — this is normal, designed behaviour, not a special case the operator needs to react to differently. Otherwise the toast/result line show the flow's own `message`, green for success, red for failure.
- **"Cancel"**: hides the confirmation box only; the counter is not reset.

#### Sub-panel 3 — "DIAGNOSTICS (live values, read-only)" (blue-bordered)

A read-only reporting strip, intentionally styled in blue (not red) to distinguish it from the destructive actions above it.

| Element | Description |
|---|---|
| **"Refresh now" button** | Calls **Agent 4 (Status Check)** (`'DMPAgent4(StatusCheck)=>VS'.Run()`) and refreshes the counters and audit baselines shown below from the response. If the call fails, each value simply keeps showing its last known figure (no error is surfaced) — a resilient, designed fallback. |
| **"Copy to Clipboard" button** | Copies the text of the three lines below (counters, Critical, Warning) to the clipboard. On success shows a green toast confirming the copy. If the clipboard is not available in the current run context (e.g. some embedded/preview contexts), it instead shows a yellow warning toast that copying is not available in that context. (Both toast texts currently display in German — see §7 Known Limitations.) |
| **"Email counters" line** | Text pattern: `Email counters - No DMP=<n> Internal=<n> Effected=<n> Not Effected=<n>`, e.g. "Email counters - No DMP=0 Internal=12 Effected=3 Not Effected=7". Defaults to 0 for any counter not yet loaded. |
| **"Critical" line** | Text pattern: `Critical - total=<n> baseline=<n> new since reset=<n>`, where "new since reset" is the total minus the baseline (floored at 0) — i.e. how many new Critical audit events have occurred since the Critical baseline was last reset. |
| **"Warning" line** | Same pattern as Critical, for Warning-level audit events: `Warning - total=<n> baseline=<n> new since reset=<n>`. |

#### Bottom result line

A single gray text line beneath all three panels shows the message from the most recently completed action (mailbox cleanup or any counter reset). It is only visible when a message is present (i.e. it appears the first time any action is confirmed, and is then overwritten by each subsequent action); the line's own color does not change between success and failure — the color distinction is carried by the (temporary) toast notification only, not by this persistent line.

### 3.8 Release Notes screen

The Release Notes screen documents the app's own version history and, separately, the version history of each backend Power Automate agent (Agent 1–6). It is reached from the Cockpit's version tag and from the Maintenance screen's "Cockpit app: … (see Release Notes)" link.

#### Header bar

| Element | Description |
|---|---|
| **"< Back to Cockpit" button** | Returns to Home/Cockpit. |
| **Title** | "Release Notes - Version History" |
| **Subtitle** | "DMP COMMAND - current version `<app version>`" — shows the app's own current version number (matches the value shown on the Maintenance screen). |
| **"Copy to Clipboard" button** (top-right, green) | Copies the release-note text of whichever tab is currently active to the clipboard: for the "App Changes" tab it concatenates the header + notes of *every* app version card, from the newest down to the oldest "v1.5.x & older" bucket; for the "Agent (Flow) Changes" tab it concatenates the header + notes of all six agent cards (Agent 1 through Agent 6). On success: a green toast confirms the copy. If the clipboard is unavailable in the current context: a yellow warning toast. (Both toast texts currently display in German — see §7 Known Limitations.) |

#### Tab selector

Two pill buttons directly under the header, functioning as a simple two-way switch (only one tab is shown at a time):

| Tab button | Selects |
|---|---|
| "App Changes" | The DMP COMMAND Power App's own version history (default tab shown when the screen first opens). |
| "Agent (Flow) Changes" | The six backend agents' (Agent 1–6) version histories. |

The active tab's button is highlighted (solid green fill, bold, dark text); the inactive tab is shown in a muted, theme-adapted tint.

#### Layout pattern (both tabs): scrollable version menu + detail pane

Each tab uses the same **master/detail (sidebar + content) layout**, filling the remaining screen height below the header/tabs:

- **Left column — version menu** (narrow, ~170px wide, independently vertically scrollable): a stacked list of small pill buttons, one per item. Clicking an entry selects it (highlighted green/bold when selected, muted otherwise) and immediately updates the detail pane on the right — there is no separate "open" step.
- **Right column — detail pane** (fills remaining width, independently vertically scrollable, bordered card): shows, for the currently selected item, a bold green **header line** followed by a **notes block** of one or more bullet-style lines (each change is its own line, generally starting with "- ", separated by line breaks). If no item has been explicitly clicked yet, the detail pane defaults to showing the newest/first entry in the list (the top of the menu for the App tab, "Agent 1" for the Agent tab).

**App Changes tab specifics:**
- The version menu lists every released app version as its own entry, newest first, e.g. "v1.22.12", "v1.22.11", "v1.22.10", … down through entries like "v1.13.1", "v1.6.1-163" (a combined entry for a range of very close patch releases), ending in a catch-all "v1.5.x & older" entry that bundles all earlier history into one card.
- The detail header line combines the version number, its release date, and — **only for the single newest version** — a trailing "(current)" tag, e.g. *"v1.22.12 - 2026-09-04 (current)"* versus an older entry such as *"v1.13.1 - 2026-09-01"* (no tag).
- The detail notes block lists the functional changes made in that version as separate lines (one bullet per change; a version can bundle more than one bullet, e.g. a hotfix note plus a feature note in the same card).

**Agent (Flow) Changes tab specifics:**
- The version menu is a fixed list of exactly six entries, one per agent, labelled simply "Agent 1" through "Agent 6" (no dates or version numbers in the menu itself).
- The detail header line names the agent and its function together with its current version and the "(current)" tag, e.g. *"Agent 2 (E-Mail Inbox Treatment) - v1.0.8 (current)"*, *"Agent 6 (Admin Functions) - v1.3.1 (current)"* — since only the current build is tracked per agent (there is no older/newer selector within an agent card).
- The detail notes block is that agent's changelog: a chronological, newest-first bullet list of past changes for that single agent, each bullet typically starting with its version number and date, e.g. *"- v1.4.4 (2026-09-04) - …"*, followed by older bullets further down (some older bullets carry only a date, no version number, for changes predating strict versioning).

#### Scrolling behavior

Both the version menu and the detail pane scroll independently and vertically whenever their content is taller than the available space — e.g. scrolling the App tab's long version menu does not move the detail pane, and a long notes block in the detail pane scrolls within its own card without affecting the menu. The overall split area itself is sized to fill the remaining vertical space beneath the header and tab selector.

### 3.9 Quick reference: Switching the Operating State

1. Locate the **Operating State** panel on the Cockpit (§3.1.4).
2. To simulate or declare a real DMP event, toggle **Operational Mode** to **DMP**. To end one, toggle back to **Normal**.
3. To switch between the test/simulation environment and production, toggle **Environment** between **SIMU** and **PROD** (switching to PROD always resets Operational Mode to Normal for safety).
4. Each toggle immediately calls Agent 5 and writes the combined mode to `CurrentOperationMode`. A success or error notification appears at the top of the screen (see §3.1.4 for the exact wording of every case).

### 3.10 Quick reference: Viewing/editing domain lists

Use the **View** buttons (Maintenance – Domains panel, §3.1.5) to open the current Internal (`DMP Command Internal Domains` SharePoint list) / External (`DMP Command External Domains` SharePoint list) domain data in SharePoint's "All Items" view. Use **Edit** to open the corresponding list's "New Item" form directly in SharePoint, for adding a new domain entry. Both links open in the browser; no data is edited inside the Power App itself.

### 3.11 Quick reference: Replacing the Emergency Report

Click **Replace** next to the External row in the Maintenance – Domains panel (§3.1.5), select a `.xlsx` file. Only `.xlsx` files are accepted — other file types are rejected with the message "Only .xlsx files are allowed." The status dot on that row blinks yellow while Agent 3 processes the upload, then returns to green (or red on failure). On success, Agent 3 stores the file; the operator should subsequently run Agent 1 to regenerate the domain lists from the new report.

### 3.12 Quick reference: Dark/Light mode

Use the toggle in the top-right of the header (§3.1.2) to switch between dark and light color themes. This is a purely visual, per-session, client-side setting.

### 3.13 System Health (Details) screen

Reached either by clicking the System Health ring on the Cockpit (§3.1.3) or via the **"System Health"** entry in the left navigation sidebar. Introduced in `v1.22.30` to replace the ring's old pop-up, which was fixed at 312×300px with an internal scrollbar and could not show all 16 monitored items readably at once (user feedback). The screen has the same "< Back to Cockpit" header pattern as the other detail screens, followed by a scrollable body of bordered cards, grouping the same 16 items and status colours the ring itself is built from (green/amber/red, identical thresholds - this screen adds no new logic for those items, it only presents the existing ring formulas at full size), plus (as of `v1.22.32`) the connection-reachability dots formerly on the Maintenance screen. As of `v1.22.32`, the cards are arranged as 3 horizontal rows of 2 cards each (per user feedback that "2-4 Bereiche" should sit side by side, instead of one full-width card per row):

- **Row 1 - Core Status | Data Sources & Files:** Status Check, Critical Events, Warnings, Operating State | Emergency Report Processing, Internal Domains, External Domains, Counter, Audit Trail.
- **Row 2 - Agents (1-7) | Connection Diagnostics - Core & Domain Lists:** one row per agent, each showing the agent's display name and `Operational`/`Not Responding` state from `DMP Command Agent Status` | reachability dot (`IfError(CountRows(list)>=0, green, red)`) for 9 of the 17 connected SharePoint lists (Configuration, Agent Status, Counters, Internal/External Domains, Checklist Overall Process, Checklist CoS Leader, Default Case Context, Checklist Recurrence Rules).
- **Row 3 - Connection Diagnostics - Streams Lists | About the dots:** reachability dot for the remaining 8 lists (Checklist Task Occurrences, Status Change Approvals, Email Templates, Email Placeholders, Recipient Groups, Role Assignments, Screens Catalog, Actions Catalog) | a short note explaining that a red dot means the app itself lost its own SharePoint connection to that list, not necessarily that the list is broken.

Connection Diagnostics was moved here from the Maintenance screen in `v1.22.32` (previously covered only 3 of the 17 lists there) per user feedback that it belongs functionally to System Health rather than Maintenance.

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

**Related lists (each a separate SharePoint list, not part of `DMP Command Configuration` itself):**
- **`DMP Command Internal Domains`:** one row per internal domain (`Title` = domain name, `Active` Choice column = `Yes`/`No`). Agent 1 writes/maintains it; Agent 2 (classification) and Agent 4 (status/count) both read it directly; the Power App's Maintenance - Domains panel and Configuration (Lists) screen both link to it.
- **`DMP Command External Domains`:** one row per external domain. Fully deleted and recreated by Agent 1 on every Emergency Report extraction; Agent 2 (classification) and Agent 4 (status/count) both read it directly; the Power App's Maintenance - Domains panel and Configuration (Lists) screen (§3.5, has its own row in the "Domain Classification" card, since `v1.22.30`) both link to it, as does the Automation Status panel (§3.1.5/§3.1.7).
- **`DMP Command Counters`:** one row per e-mail-classification counter, keyed by `Title` (e.g. `'No DMP'`, `'DMP internal Sender'`, `'DMP effected Member'`, `'DMP not effected Sender'`), value column `NumberProcessedEmails`. Agent 2 increments the matching row per classified e-mail; Agent 6 resets one or all rows to 0 on operator request (§3.7); Agent 4 reads it for the Cockpit's Emails Processed ring (§3.1.8) and Automation Status panel (§3.1.7).

**Alert folder parameters:** Each of Agent 3, 4, and 5 has its own dedicated `Agent3AlertFolderName` / `Agent4AlertFolderName` / `Agent5AlertFolderName` parameter with its own `Agent 03` / `Agent 04` / `Agent 05` scope — no agent shares an alert folder with another. See §9 for the naming design rule that formalizes this pattern for future agents.

---

## 5. Cross-Cutting Standards

### 5.1 Audit Trail

Every agent run logs to a shared `AuditTrail.xlsx` table with a fixed 20-column schema: `TimestampUtc, RunId, MessageId, WorkflowPath, StepName, StepStatus, Flow ID, KeyOutput, DurationSec, ActionType, Direction, Recipient, SubjectOut, TargetMessageId, TargetFolderName, TargetFolderId, MatchedDomain, Decision, Sender, SenderDomain`. Every significant step buffers an event; at the end of each run, one additional **Run Summary** row is written (`StepName = "RunSummary"`, `ActionType = "Summary"`) summarizing the overall outcome. `Flow ID` is intentionally left blank across all agents (unused legacy column).

**Agent Audit Summary:** `AuditTrail.xlsx` also contains a second table, `Agent Audit Summary` (17 columns, one row per agent `Agent 01`–`Agent 05`), that keeps a running, pre-aggregated count of steps and runs by outcome (`SucceededStepsCount`, `FailedStepsCount`, `WarningStepsCount`, `StartedStepsCount`, and the equivalent `...RunsCount` columns), each paired with a `...LastUpdateUtc` timestamp column recording when that specific counter was last incremented. Every agent increments its own row after each run — this is what Agent 4 reads and aggregates for the Cockpit's system-wide Critical/Warning/Total-runs figures (see §2.4), avoiding the need to scan the full detailed log at request time.

**Audit Acknowledgment table:** A third table, `Audit Acknowledgment` (9 columns, one row per agent), stores a "baseline" count and acknowledgment timestamp for each of the 4 problematic counter categories per agent (`FailedSteps`, `WarningSteps`, `FailedRuns`, `WarningRuns`). It is used today by the Audit Trail (Detail) screen's Reset Critical/Reset Warning/Reset All buttons (§3.4) to store the baseline that the "new since reset" figures are calculated against.

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

## 7. Known Limitations

- **No four-eyes/dual-approval confirmation** before an Operating State or Environment switch takes effect (§3.1.4, §6) — a single operator's toggle click is sufficient to change `CurrentOperationMode`.
- **No in-app mechanism to archive-and-reset the Audit Trail file itself.** Today, only the 4 e-mail counters (Admin Functions, §3.7) and the Critical/Warning KPI *baselines* (Audit Trail (Detail), §3.4) can be reset; the underlying `AuditTrail.xlsx` detail rows are never deleted or archived from within the app.
- **Dark/Light mode (§3.1.2, §3.12) is a purely visual, client-side, per-session setting** — it is not persisted between sessions and cannot be configured per user; every session starts in the platform/app default theme.
- **Two clipboard-copy confirmation toasts are still in German** while the rest of the UI is in English: the "Copy to Clipboard" actions on the Admin Functions screen (§3.7) and the Release Notes screen (§3.8) both show German success/warning toast text.
- **No F1 keyboard shortcut** to open the Help / Operational Manual screen (§3.3) — not reliably supported by the canvas app platform, since browsers intercept/ignore F1 before the app can react to it.
- **Auto-refresh relies on a 1-second canvas app Timer** (§3.1.2); browser tab throttling of background/inactive browser tabs can make the visible countdown appear to freeze even though the underlying mechanism is correctly configured. Keep the app tab in the foreground for reliable live updates.
- **Individual/per-user accent colour themes are not supported** — the app uses a single fixed Eurex colour scheme (plus the Dark/Light toggle above), with no per-user customization.
- **The "Assign Roles" task and screen/action-level access control are not yet implemented** — every authenticated user who can open the app currently sees the same screens and the same buttons; there is no role-based restriction on who may, for example, switch the Operating State or use Admin Functions.

## 8. Planned in Future Phases

The items below are scoped or partially designed but not yet built. Full technical detail, architecture decisions, and status live in `Documentation/Backlog/DMP_COMMAND_Backlog.md`; this section only summarizes what is planned, not how it will be implemented.

- **DMP Command Streams:** a new feature area giving each working sub-stream (CoS Leader, Infrastructure Team, Content Team, Hotline Team) its own digital checklist (mirroring the existing SharePoint Excel checklists), feeding a master status overview, with automated e-mail dispatch on task completion and on Operating Mode changes, enforced by a four-eyes principle per team. Includes two new intermediate Operating Modes, **Pre-Default** and **Post-Default** (planned full sequence: Normal → Pre-Default → DMP → Post-Default → Normal), and a structured "Assign Roles" concept controlling which screens/actions a user may access (see next item).
- **User-Access-Konzept (roles → screens/actions):** role-based visibility and permission control for screens and destructive actions (e.g. Operating State switching, Admin Functions), to be delivered together with the DMP Command Streams "Assign Roles" design as a single, unified role model.
- **Audit Trail / Counter archiving and reset:** a settings-level, four-eyes-protected mechanism to archive and then clear the Audit Trail file (distinct from the counter/baseline resets that already exist today, §3.4/§3.7).
- **Error-code classification for the Audit Trail:** structured classification of Audit Trail failures by error code, for easier filtering/analysis. Scope not yet finalized with the business owner.
- **Individual colour settings:** letting operators choose their own accent/theme colours in app settings, instead of the single fixed Eurex colour scheme.
- **UAT Playbook refresh:** `AI_Agent\UAT\UAT_Playbook.docx` currently only covers Agent 2 (as of June 2026) and needs new test cases for Agent 1/3/4/5/6, the 2-switch Operating State model, and the Audit Summary/Acknowledgment infrastructure — planned ahead of the next major UAT cycle, and mandatory before DMP Command Streams testing begins.

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
