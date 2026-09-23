# DMP Command Checklist – SharePoint List Templates

These CSV files contain ONLY the column headers (empty templates) for the new SharePoint lists from the concept
[DMP_Command_Streams_Feature_Konzept.md](../Backlog/DMP_Command_Streams_Feature_Konzept.md) (Section 3, data model)
and, since 2026-09-23, from [DMP_COMMAND_Next_Steps_Anforderung.md](../DMP_COMMAND_Next_Steps_Anforderung.md).

## Status (as of 2026-09-23)

- **All lists already created and published by the user, except Infrastructure/Content Checklist** (deferred – no
  SharePoint site access yet for those two team sites at the time; **please re-check this session whether those
  team sites now exist** - if not yet, say so before creating the lists so an alternative placement can be agreed).
- **New this session (2026-09-23), needed for the "Next Steps" feature - 3 new columns on the already-created
  `DMP Command Checklist Overall Process` list:** `SeqNo`, `PredecessorTaskIds`, `IsMilestone` (see the updated
  column table below). These close a data-model gap found while checking
  [DMP_COMMAND_Next_Steps_Anforderung.md](../DMP_COMMAND_Next_Steps_Anforderung.md) against the live app: the
  document's dependency/readiness model (§6) and global task ordering (§3) cannot currently be stored anywhere.
  **Please add these 3 columns to the existing live list** (do not recreate the list - just add columns).
- **Action required on the already-created lists:** (1) rename each list from its previous title to the new
  naming convention below (SharePoint: **List settings → List name and description → rename** – this only
  changes the display name/title, no data is lost, the internal URL/list ID stays the same), and (2) apply the
  structural corrections documented per list below (column swaps, new catalog lists, merged date/time field –
  all flagged as "user feedback 2026-09-03").
- The two new catalog lists (`DMP Command Checklist Screens Catalog`, `DMP Command Checklist Actions Catalog`) and
  `DMP Command Checklist Email Placeholders` are additional NEW lists that did not exist before – these need to be
  created from scratch (see CSV templates in this folder), not just renamed.

## Naming history (for context, not to be repeated)

The feature originated from the existing "NEXT STEPS" tile/container on the home screen (a read-only milestone
display, unrelated to this data model). During drafting, the SharePoint lists were first named with the prefix
"NextSteps_", then briefly "DMP Command Next Steps ". **Both were working titles only.** User feedback
(2026-09-03): this is fundamentally about **governing/controlling the four working sub streams**, not about the
home screen tile – hence the final, binding prefix is **"DMP Command Checklist "**.

## Why as files instead of created directly?

Automated SharePoint access (PnP PowerShell, Connect-PnPOnline) fails in this environment due to the tenant's
Azure AD app registration ("Application ... was not found in the directory") – this can only be resolved by a
SharePoint/Azure AD admin (app consent), not by a script. Hence the lists are prepared here as templates so they
can be created manually in a few minutes (by the user or anyone with SharePoint permissions).

## How to create a list in SharePoint

1. Open the target site (e.g. `https://deutscheboerse.sharepoint.com/teams/GO365_DMPCommunication` for the
   overall/shared lists, or the respective team site for team-specific checklists).
2. **Site contents → New → List → From Excel** (or "Import list"), upload the respective CSV file from this
   folder. Alternatively: create an empty list and add columns manually according to the CSV headers (recommended,
   since SharePoint's CSV import often creates all columns as "single line of text" – change data types afterwards,
   see table below).
3. Name the list **exactly** as documented below (this name is later referenced 1:1 as a SharePoint data source in
   Power Apps / Power Automate).

## Column data types and descriptions

The SharePoint column type dialog only offers a limited selection when creating/importing manually:
`Einzelne Zeile Text`, `Mehrere Zeilen Text`, `Zahl`, `Ja/Nein`, `Auswahl`, `Datum und Uhrzeit`, `Hyperlink`,
`Person oder Gruppe`, `Titel`, `Nicht importieren`. **No `Nachschlagen (Lookup)` in this dialog** – lookup columns
must be added afterwards via "+ Add column" → "Show more column types" → "Search" (there, any column of the
target list, not just Titel, can be chosen as the display value – only possible once the target list exists).

**Unique key correction (user feedback, 2026-09-03):** In `Email Templates` and `Recipient Groups`, the business
ID (`TemplateId` resp. `GroupId`) is the unique key of the list – the mandatory SharePoint `Titel` field is used
for it (not for the descriptive display name as originally drafted). The descriptive name now lives in its own
column (`TemplateName`/`GroupName`).

For `Auswahl` columns the concrete option values are listed (please create them 1:1 as options).

---

### 1) DMP Command Checklist Overall Process ✅ already set up — 🆕 3 new columns needed (2026-09-23)

**List description (English):**
"Aggregated master task list of the Default Management Process across all four working sub streams (CoS Leader, Infrastructure Team, Content Team, Hotline Team). Status is derived automatically from the individual team checklists once four-eyes approval is complete. Read-only reference for the CoS Leader master overview screen."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= TaskID) | Titel | Unique technical identifier of the task (e.g. `COS-001`, `INF-014`). Referenced by all team checklists to link the same process step (identical `Titel` value in both lists). Must not change after creation. |
| Phase | Auswahl: `Pre-Default`, `DMP (Termination & Liquidation)`, `Post-Default` | DMP process phase this task belongs to. Drives grouping/sorting in the overview and must match the app's current 5-mode operating state. |
| 🆕 SeqNo | Zahl | **New 2026-09-23.** Global sequence number across all sub streams (the "globale fachliche DMP-Reihenfolge" from the Next Steps spec, §3). NOT an automatic dependency by itself - only determines display order. |
| TaskShortDescription | Mehrere Zeilen Text | Short description of the task for the overview, as shown e.g. to the CoS Leader on the master page. |
| ResponsibleSubStream | Auswahl: `COS Leader`, `Infrastructure Team`, `Content Team`, `Hotline Team` | Working sub stream responsible for completing this task. |
| 🆕 PredecessorTaskIds | Einzelne Zeile Text | **New 2026-09-23.** Semicolon-separated list of `Titel`/TaskID values (e.g. `COS-001;INF-003`) that must all reach `Status = Done` before this task becomes executable (Next Steps spec §6, AND-only, no OR). Leave empty if this task's only precondition is reaching Pre-Default. |
| 🆕 IsMilestone | Ja/Nein | **New 2026-09-23.** Marks this task as a cross-team milestone that other tasks' `PredecessorTaskIds` can reference (Next Steps spec §8.2 point 4, §9). Default `No`. |
| Status | Auswahl: `Not Started`, `Ongoing`, `Done` | Current processing status of the task. Updated automatically from the respective team checklist once the four-eyes approval is complete – do not edit directly here. |
| LastChangedUtc | Datum und Uhrzeit | Timestamp (UTC) of the last status change, for traceability and potential escalation on tasks left open too long. |


### 2) DMP Command Checklist CoS Leader ✅ already set up

**Confirmed live 2026-09-22** (user-provided list settings screenshot), despite being absent
from the `Survey on SharePoint Lists.docx` schema export - a known Teams/SharePoint display
inconsistency can make an existing list simply not show up in that kind of export, so its
absence there must never be read as "doesn't exist" on its own. Site
`GO365_DMPCommunication-CoSLeader`, list GUID `E3E95266-BB35-4E32-A60D-06E8E9E379A1`,
internal/URL list name still `NextSteps_CosLeaderChecklist` (pre-rename technical name - only
the display title was ever renamed). Columns confirmed 1:1 against the table below.

**List description (English):**
"Working checklist of the CoS Leader sub stream. Each task references a unique TaskID linked to DMP Command Checklist Overall Process. Status changes require four-eyes confirmation (see DMP Command Checklist Status Change Approvals) before they become effective."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= TaskID) | Titel | Unique task ID, identical to the corresponding row in `DMP Command Checklist Overall Process`. Establishes the link to the overall status overview. |
| Phase | Auswahl: `Pre-Default`, `DMP (Termination & Liquidation)`, `Post-Default` | DMP process phase (same values as in Overall Process) this CoS Leader task belongs to. |
| SeqNo | Zahl | Sequential numbering of tasks within the CoS Leader checklist (processing order, formerly column B of the Excel file). |
| TaskDescription | Mehrere Zeilen Text | Detailed description of the task to be performed by the CoS Leader (formerly column D of the Excel checklist). |
| EmailTemplateId | Einzelne Zeile Text | Reference to the e-mail template to be sent (`Titel`/`TemplateId` from `DMP Command Checklist Email Templates`). Leave empty if no e-mail is required for this task. |
| Status | Auswahl: `Not Started`, `Ongoing`, `Done` | Current processing status. Only becomes final after four-eyes approval (see Status Change Approvals) – do not change directly here, use the approval process. |
| LastChangedBy | Person oder Gruppe | Person who proposed/entered the status change. |
| ConfirmedBy | Person oder Gruppe | Person who confirmed the status change under the four-eyes principle. Must be a different person than `LastChangedBy` and a member of the same sub stream or CoS Lead/Deputy. |
| ConfirmedUtc | Datum und Uhrzeit | Timestamp (UTC) of the confirmation by the second person. |

### 3) DMP Command Checklist Infrastructure / Content Checklist 🆕 ready to create (2026-09-23)

**Update 2026-09-23:** previously deferred for lack of SharePoint site access for the Infrastructure Team / Content
Team sites - please confirm this session whether those sites now exist. If they do, these 2 lists can be created
now using the CSV templates in this folder (`DMP Command Streams Infrastructure Team Checklist.csv` /
`DMP Command Streams Content Team Checklist.csv`); the "Next Steps" feature's mandatory per-sub-stream
representation (spec §9) needs both to show a real, non-empty view for these 2 sub streams, not just CoS Leader.

**List description (English, adjust sub stream name accordingly):**
"Working checklist of the Infrastructure Team sub stream (respectively Content Team). Each task references a unique TaskID linked to DMP Command Checklist Overall Process. Status changes require four-eyes confirmation before they become effective."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= TaskID) | Titel | Unique task ID, identical to the corresponding row in `DMP Command Checklist Overall Process`. |
| SeqNo | Zahl | Sequential numbering of tasks within this team checklist (formerly column A of the Excel file). |
| Phase | Auswahl: `Pre-Default`, `DMP (Termination & Liquidation)`, `Post-Default` | DMP process phase this task belongs to. |
| TaskDescription | Mehrere Zeilen Text | Detailed description of the task to be performed by this team. |
| EmailTemplateId | Einzelne Zeile Text | Reference to the e-mail template to be sent. Leave empty if no e-mail is required. |
| DelegatedTo | Person oder Gruppe | Optional delegate to whom this task within the team has been delegated (formerly column F of the Infrastructure Excel file). Leave empty if not needed. |
| Status | Auswahl: `Not Started`, `Ongoing`, `Done` | Current processing status. Only becomes final after four-eyes approval. |
| LastChangedBy | Person oder Gruppe | Person who proposed/entered the status change. |
| ConfirmedBy | Person oder Gruppe | Person who confirmed the status change under the four-eyes principle (different person, same sub stream or CoS Lead/Deputy). |
| ConfirmedUtc | Datum und Uhrzeit | Timestamp (UTC) of the confirmation. |

*(Note: No dedicated example Excel exists yet for Content Team – structure copied 1:1 from Infrastructure, since the brainstorming described it as "analogous". Adjust later if needed.)*

### 4) DMP Command Checklist Email Templates

**List description (English):**
"Central, parameterized e-mail templates used by the Streams checklists. Templates use `{{placeholder}}` syntax for situation-dependent text; placeholder keys must match entries in DMP Command Checklist Email Placeholders. Maintained via the dedicated scrEmailTemplateEditor screen, not edited directly in SharePoint."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= TemplateId, unique key – swapped per user feedback 2026-09-03) | Titel | Unique technical identifier of the template (e.g. `TPL-COS-001`). This is now the primary key of the list and is referenced from the checklists' `EmailTemplateId` field. |
| TemplateName | Einzelne Zeile Text | Descriptive display name of the template (formerly held in `Titel`), e.g. "Pre-Default announcement to Clearing Member". Shown in the selection list of `scrEmailTemplateEditor`. |
| SubjectTemplate | Einzelne Zeile Text | E-mail subject line with placeholders, e.g. `DMP Update – {{DefaultedMemberName}} – Pre-Default Assessment`. |
| BodyTemplate | Mehrere Zeilen Text *(Option „Erweiterte Formatierung mit Bildern, Tabellen und Hyperlinks zulassen" aktivieren)* | Rich-text e-mail content with `{{placeholders}}` for situation-dependent text passages (replaces the previously yellow-highlighted Word text passages). Populated via the new maintenance screen with a RichTextEditor, not edited directly in SharePoint. |
| PlaceholderList | Einzelne Zeile Text *(user feedback 2026-09-03: values must match entries in the new `DMP Command Checklist Email Placeholders` catalog list; convert to a multi-value Lookup pointing to that list once it exists)* | Comma-separated list of placeholder keys used in `BodyTemplate`/`SubjectTemplate` (e.g. `TerminationReason, DefaultedMemberName, TerminationDateTime`), used for validation in the input popup before sending. |
| RecipientGroupId | Einzelne Zeile Text | Reference to the collective mailbox address(es) in `DMP Command Checklist Recipient Groups` (`Titel`/`GroupId`) this template is sent to by default. |

### 5) DMP Command Checklist Email Placeholders *(new list, user feedback 2026-09-03)*

**List description (English):**
"Controlled catalog of all placeholder keys allowed in e-mail templates, instead of free text per template. Prevents typos and documents where each placeholder's value comes from."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= PlaceholderKey, unique key) | Titel | Unique placeholder key used inside `{{...}}` syntax in e-mail templates (e.g. `DefaultedMemberName`). |
| Description | Mehrere Zeilen Text | Meaning of this placeholder and where its value comes from (e.g. "Name of the defaulted Clearing Member, sourced from DMP Command Checklist Default Case Context"). |
| ExampleValue | Einzelne Zeile Text | Example value for documentation/testing purposes. |

**Suggested initial rows** (derived from `DMP Command Checklist Default Case Context`): `TerminationReason`, `DefaultedMemberId`, `DefaultedMemberName`, `TerminationDateTime`.

### 6) DMP Command Checklist Recipient Groups

**List description (English):**
"Collective mailbox addresses used as e-mail recipients for the Streams checklists, grouped by working sub stream or external recipient circle."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= GroupId, unique key – swapped per user feedback 2026-09-03) | Titel | Unique technical key of the recipient group (e.g. `GRP-INFRA-01`). This is now the primary key of the list and is referenced from `DMP Command Checklist Email Templates.RecipientGroupId`. |
| GroupName | Einzelne Zeile Text | Descriptive display name of the group (formerly held in `Titel`), e.g. "Infrastructure Team – Shared Mailbox" or "Clearing Members – DMP Broadcast". |
| EmailAddresses | Einzelne Zeile Text | One or more collective e-mail addresses, semicolon-separated when multiple recipients (e.g. `infrastructure-team@deutsche-boerse.com`). |
| SubStream | Auswahl: `COS Leader`, `Infrastructure Team`, `Content Team`, `Hotline Team`, `Extern/Clearing Members` | Assignment of which sub stream (or external recipient circle) this group belongs to – used for filtering/overview. |

### 7) DMP Command Checklist Role Assignments

**List description (English):**
"Assignment of users to working sub streams and the screens/actions they are permitted to see and use in the Streams feature, including CoS Lead/Deputy privileges."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** | Titel | Display name of the person for readability in list views, e.g. "Max Mustermann – Infrastructure Lead" (free text, no technical function). |
| User | Person oder Gruppe | Actual link to the person's Active Directory account. Used by the app/Agent 7 for permission checks (login match) – NOT the `Titel` text. |
| SubStream | Auswahl: `COS Leader`, `Infrastructure Team`, `Content Team`, `Hotline Team` | Working sub stream this person is assigned to. |
| IsSubStreamLead | Ja/Nein | Indicates whether the person is the lead of their sub stream (e.g. relevant for escalations and whether self-approval under the four-eyes principle is permitted). |
| IsCosLeadOrDeputy | Ja/Nein | Indicates whether the person is CoS Lead or their deputy – grants access to the master status page and cross-team view. |
| AllowedScreens | Auswahl *(Option „Mehrere Auswahlmöglichkeiten zulassen" aktivieren; user feedback 2026-09-03: options must match entries in the new `DMP Command Checklist Screens Catalog` list; convert to a multi-value Lookup once conveniently possible)* | Which screens this person is allowed to see. Controls sidebar visibility. |
| AllowedActions | Auswahl *(Option „Mehrere Auswahlmöglichkeiten zulassen" aktivieren; user feedback 2026-09-03: options must match entries in the new `DMP Command Checklist Actions Catalog` list; convert to a multi-value Lookup once conveniently possible)* | Which actions this person is allowed to perform. Controls which buttons/actions are active in the screens. |

### 8) DMP Command Checklist Screens Catalog *(new list, user feedback 2026-09-03)*

**List description (English):**
"Controlled catalog of all app screens that can be referenced in Role Assignments' AllowedScreens field, instead of free text."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= ScreenName, unique key) | Titel | Technical screen name as used in the app (e.g. `scrChecklistCosLeader`). |
| Description | Mehrere Zeilen Text | Purpose of the screen. |

Pre-filled with the 8 screens known from the concept (see the accompanying CSV file).

### 9) DMP Command Checklist Actions Catalog *(new list, user feedback 2026-09-03)*

**List description (English):**
"Controlled catalog of all Agent 7 actions / permissions that can be referenced in Role Assignments' AllowedActions field, instead of free text."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= ActionName, unique key) | Titel | Technical action name as used by Agent 7 (e.g. `ProposeStatusChange`). |
| Description | Mehrere Zeilen Text | Purpose of the action. |

Pre-filled with the actions known from the concept (see the accompanying CSV file).

### 10) DMP Command Checklist Default Case Context

**List description (English):**
"Case-specific default context captured via a mandatory popup form whenever the app switches from Normal/Pre-Default to DMP mode. Provides placeholder values (e.g. defaulted member name, termination date/time) for all subsequent e-mail templates."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= CaseId) | Titel | Unique identifier of the default case, e.g. `CASE-2026-001`. A new record is created on every transition from Normal/Pre-Default to DMP. |
| TerminationReason | Mehrere Zeilen Text | Termination reason for the Clearing Member, entered by the authorized user in the popup. |
| DefaultedMemberId | Einzelne Zeile Text | ID of the defaulted Clearing Member. |
| DefaultedMemberName | Einzelne Zeile Text | Name of the defaulted Clearing Member – inserted as placeholder `{{DefaultedMemberName}}` in e-mail templates. |
| TerminationDateTime *(merged per user feedback 2026-09-03, replaces separate TerminationDate/TerminationTime)* | Datum und Uhrzeit *(Option „Zeit anzeigen" aktivieren)* | Combined date and time of the Clearing Member's termination. |
| SetByUser | Person oder Gruppe | Authorized user who filled in the popup form when switching to DMP. |
| SetUtc | Datum und Uhrzeit | Timestamp (UTC) when the record was created. |
| ModeAtCreation | Auswahl: `PROD_DMP`, `SIMU_DMP` | Active mode at the time of capture – distinguishes a real DMP case from a simulation/fire drill. |

### 11) DMP Command Checklist Status Change Approvals

**List description (English):**
"Four-eyes approval log for status changes proposed in any of the Streams checklists. A proposed change only becomes effective in the target list once confirmed by a different, authorized person."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= RequestId) | Titel | Unique identifier of the approval request, auto-assigned (e.g. `APR-000123`). |
| ListName | Auswahl: `DMP Command Checklist CoS Leader`, `DMP Command Checklist Infrastructure`, `DMP Command Checklist Content`, `DMP Command Checklist Overall Process` | Name of the list in which the actual status change is to take place. |
| ItemId | Zahl | SharePoint internal item ID of the affected entry in the list named above. |
| OldStatus | Auswahl: `Not Started`, `Ongoing`, `Done` | Previous status before the proposed change. |
| NewStatus | Auswahl: `Not Started`, `Ongoing`, `Done` | Proposed new status. |
| ProposedBy | Person oder Gruppe | Person who proposed the status change. |
| ProposedUtc | Datum und Uhrzeit | Timestamp (UTC) of the proposal. |
| ApprovedBy | Person oder Gruppe | Person who confirmed the change under the four-eyes principle (must be ≠ `ProposedBy` and the same sub stream/CoS Lead – strictly enforced by Agent 7). |
| ApprovedUtc | Datum und Uhrzeit | Timestamp (UTC) of the confirmation. |
| ApprovalState | Auswahl: `Pending`, `Approved`, `Rejected` | Current state of the approval request (already in the CSV template, missing from this table until 2026-09-22). |
| OccurrenceId | Einzelne Zeile Text | **Live column added directly in SharePoint (found via the 2026-09-22 schema survey, not originally in this template):** direct reference to the affected `DMP Command Checklist Task Occurrences` row, as an alternative/addition to the generic `ListName`+`ItemId` pair. |
| ApprovalState | Auswahl: `Pending`, `Approved`, `Rejected` | Current state of the approval process. |

### 12) DMP Command Checklist Task Occurrences

**List description (English):**
"Concrete execution instances of recurring or one-time checklist tasks, scoped to a default case, business date and schedule slot. Each occurrence is independently status-tracked, four-eyes approved and auditable."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= OccurrenceId) | Titel | Idempotent unique key in the canonical form `CASE-2026-001-20260921-21-MIDDAY`. This is the SharePoint business key and must be indexed/unique. |
| CaseId | Einzelne Zeile Text | Reference to `DMP Command Checklist Default Case Context`. |
| TaskId | Einzelne Zeile Text | Reference to the task definition in the relevant sub-stream checklist. |
| OccurrenceDate | Datum und Uhrzeit | Business date in the configured schedule time zone; store the date component only. |
| ScheduleSlot | Einzelne Zeile Text | Stable slot key, e.g. `MIDDAY`, `EVENING`, `CONFERENCE`, `WRAP-UP`. |
| TimeZone | Einzelne Zeile Text | Time zone used to calculate the business date and due time. |
| DueUtc | Datum und Uhrzeit | Calculated due time stored in UTC. |
| Status | Auswahl: `Not Started`, `Ongoing`, `Done` | Status of this concrete execution instance. |
| ProposedBy | Person oder Gruppe | Person who proposed the latest status change. |
| ProposedUtc | Datum und Uhrzeit | Timestamp (UTC) of the proposal. |
| ApprovedBy | Person oder Gruppe | Second person who confirmed the status change. |
| ApprovedUtc | Datum und Uhrzeit | Timestamp (UTC) of the confirmation. |
| ApprovalState | Auswahl: `Not Required`, `Pending`, `Approved`, `Rejected` | Approval state for the current occurrence status. |
| CompletedUtc | Datum und Uhrzeit | Timestamp (UTC) when the occurrence became `Done`. |
| Notes | Mehrere Zeilen Text | Result, deviation or operational note. |

**Idempotency rule:** Agent 7 must first query this list by the exact `OccurrenceId`. If a row exists, it must not be updated or recreated, regardless of its status. If no row exists, exactly one row may be created. `OccurrenceId` is normalized as `CaseId-Date(TaskDate in TimeZone)-TaskId-Upper(ScheduleSlot)` with separators `-`; spaces are removed from `TaskId` and `ScheduleSlot`.

### 13) DMP Command Checklist Recurrence Rules

**List description (English):**
"Controlled recurrence rules for generating checklist task occurrences. Rules are data-driven so schedule changes do not require flow logic changes."

| Spalte | Typ | Description |
|---|---|---|
| **Titel** (= RuleId) | Titel | Unique rule key, e.g. `RULE-COS-021-MIDDAY`. |
| TaskId | Einzelne Zeile Text | Task definition to which the rule applies. |
| RecurrenceType | Auswahl: `Daily`, `Once` | Recurrence mode. C5 initially consumes `Daily`. |
| ScheduleSlot | Einzelne Zeile Text | Stable business slot key, e.g. `MIDDAY` or `EVENING`. |
| TimeOfDay | Uhrzeit / Einzelne Zeile Text | Local scheduled time in `TimeZone`, used to calculate `DueUtc`. Required for active `Daily` rules. |
| TimeZone | Einzelne Zeile Text | Time zone used for schedule calculation. |
| StartMode | Auswahl: `DMP` | Mode in which generation is allowed to start. |
| ActiveFrom | Datum und Uhrzeit | Optional start boundary. |
| ActiveUntil | Datum und Uhrzeit | Optional end boundary; the active DMP case end applies otherwise. |
| IsActive | Ja/Nein | Enables or disables the rule without deleting history. |

**Generation rule:** only active `Daily` rules whose window includes the target business date and whose case is in `DMP` are eligible. `Post-Default` is an explicit stop condition; it must not create new occurrences. A scheduler retry must use the same canonical key and therefore leave existing rows byte-for-byte unchanged.

## C5 Agent-7 action contract

The new Agent-7 dispatcher must expose these Power Apps actions:

### `CreateTaskOccurrences`

Required inputs:

- `CaseId`
- `BusinessDate` in `yyyy-MM-dd` format, interpreted in each rule's `TimeZone`
- `RequestedBy`

Processing contract:

1. Reject the request unless the case is active and the effective operating mode is `PROD_DMP` or `SIMU_DMP`.
2. Read all active `Daily` recurrence rules in one SharePoint query.
3. For each eligible rule, resolve the task definition and calculate the canonical `OccurrenceId`.
4. Query `DMP Command Checklist Task Occurrences` by the exact `OccurrenceId`.
5. Create one row only when no row exists. Existing rows are returned unchanged.
6. Return `CreatedCount`, `ExistingCount`, `SkippedCount`, `OccurrenceIds` and a machine-readable `Result` (`Succeeded`, `NoEligibleRules` or `Failed`).

### `GenerateMissingOccurrences`

Required inputs:

- `CaseId`
- `FromDate` and `ToDate` in `yyyy-MM-dd` format
- `RequestedBy`

The action applies the same rules for every business date in the inclusive range. It must not update existing rows and must stop generation after the case enters Post-Default. A failure must return the failing `RuleId`, `OccurrenceId` (when calculable), SharePoint action name and connector error text; it must not return a success-shaped response.

The canonical key is built as:

`<CaseId>-<yyyyMMdd business date>-<TaskId without spaces>-<UPPER ScheduleSlot>`

The business date is calculated in `TimeZone`; `DueUtc` is then stored as UTC. The exact key is the only idempotency check. No status, due time, notes or audit fields may be changed when the key already exists.

## C5 Power Apps GUI preparation

The Canvas App source now contains `scrTaskOccurrences` and a Cockpit navigation entry
named `Task Occurrences`. The screen provides the operational layout for the occurrence
views and shows the canonical occurrence identifier, task, schedule slot, status and due
timestamp fields.

The current screen intentionally uses preview rows and does not declare an unverified
SharePoint or Agent-7 binding. Replace the preview `Items` formula with the validated
Task Occurrences data source and connect the Agent-7 actions only after the final flow
trigger/response metadata is imported and confirmed.

## Order (per concept, Section 8)

1. ✅ `DMP Command Checklist Overall Process` (migrate content from `Status DMP Process.xlsx`)
2. ✅ `DMP Command Checklist CoS Leader` (migrate content from `CoSLeader Checklist.xlsx`, establish TaskID link to 1.)
3. `DMP Command Checklist Screens Catalog`, `DMP Command Checklist Actions Catalog` (small catalogs, needed before Role Assignments)
4. `DMP Command Checklist Role Assignments`, `DMP Command Checklist Status Change Approvals`
5. `DMP Command Checklist Email Placeholders` (needed before Email Templates)
6. `DMP Command Checklist Email Templates`, `DMP Command Checklist Recipient Groups`
7. ⏸ `DMP Command Checklist Infrastructure`, `DMP Command Checklist Content` (deferred – no site access yet)
8. `DMP Command Checklist Default Case Context` (Strang B, once the 5-mode switch-over is implemented)
9. `DMP Command Checklist Recurrence Rules`, `DMP Command Checklist Task Occurrences` (Strang C, before recurring-task automation)

Once the lists are created, the actual app/flow development (Agent 7, new screens) can begin.

## Agent 7 occurrence scaffold (Dev)

The local solution contains an activation-compatible Agent 7 source scaffold:
`PowerAutomate/DMP_COMMAND_Solution/Source/Workflows/DMPAgent7StreamsMilestoneManagement-82E743CD-A2F6-4485-9E76-111D0D30544C.json`.
Dev solution version `7.11.46` was successfully opened, saved, and activated in the new
Power Automate designer. The current flow reads active Daily DMP rules for one supplied
`BusinessDate`, derives the canonical
`CaseId-yyyyMMdd-TaskIdWithoutSpaces-UPPER(ScheduleSlot)` key, performs an exact lookup, and
creates a row only when that key does not exist.

The activation failure was caused by sending the Task Occurrences `ScheduleSlot` Choice as
plain text through `item/ScheduleSlot`. The validated connector mapping is
`item/ScheduleSlot/Value`. The flow must not add an `item/TimeZone` creation parameter unless
that column is independently confirmed in the target list connector schema.

This remains a partial C5 implementation. Inclusive `FromDate`/`ToDate` expansion, the final
action contract, authoritative case/mode validation, counters, and live Power Apps binding
remain pending.

**Update 2026-09-22 (local source only, not yet packed/imported into Dev):** on top of the
`RequestedAction`/`FromDate`-`ToDate` contract shipped in `7.11.47`, the local source now also
contains: (a) a rule-level `TimeOfDay`/`TimeZone` existence check inside the rule-application
loop that skips a misconfigured Daily rule gracefully (logging into new `ErrorCount`/
`ErrorMessages` variables) instead of failing the whole run in `convertTimeZone`; (b) a real
per-item error counter using a `Scope` wrapped around the SharePoint `CREATE_Occurrence` call
with a Failed/TimedOut catch branch, so a technical create failure for one occurrence is
counted and logged rather than stopping the run; (c) a temporary placeholder validation that
rejects a blank/whitespace-only caller-supplied `CaseId` before any SharePoint call, explicitly
marked as a stand-in until the real active-case lookup (B3 below) exists; and (d) `errorCount`/
`errorDetails` added to the `RESPOND_Result` output, with `success` now also `false` whenever
`ErrorCount > 0`. This is pending the user's designer save/activate confirmation for `7.11.47`
before being packed into a new solution version.

### Confirmed SharePoint bindings

The following bindings were supplied from the SharePoint List Settings pages and are now used
by the local Agent-7 source:

| Resource | Confirmed value |
|---|---|
| Site | `https://deutscheboerse.sharepoint.com/teams/GO365_DMPCommunication-CoSLeader` |
| `DMP Command Checklist Recurrence Rules` list ID | `728f1f87-2cc6-476f-be56-0f1feb05158f` |
| `DMP Command Checklist Task Occurrences` list ID | `5c413852-2dab-4844-b886-a8cf8334fcf6` |

The internal field names and exact choice values are still to be confirmed from the column
settings URLs before import. The connector parameters must not be assumed to match display
names for fields such as `CaseId`, `TaskId`, `DueUtc`, `Status` and `ApprovalState`.

### Screenshot-confirmed column types

The supplied SharePoint List Settings screenshots confirm the following visible columns and
types:

| List | Column | Type |
|---|---|---|
| Task Occurrences | `Title`, `CaseId`, `TaskId`, `Notes` | Single line of text, except `Notes` which is multiple lines of text |
| Task Occurrences | `OccurrenceDate`, `DueUtc`, `ProposedUtc`, `ApprovedUtc`, `CompletedUtc` | Date and Time |
| Task Occurrences | `ScheduleSlot`, `Status`, `ApprovalState` | Choice |
| Task Occurrences | `ProposedBy`, `ApprovedBy` | Person or Group |
| Recurrence Rules | `Title`, `TaskId`, `TimeZone`, `TimeOfDay` | Single line of text |
| Recurrence Rules | `RecurrenceType`, `StartMode`, `IsActive` | Choice |
| Recurrence Rules | `ActiveFrom`, `ActiveUntil` | Date and Time |

The screenshots do not expose the column settings URLs and therefore do not prove the
internal `Field=` names or the choice option values. Those remain a pre-import verification
step.

The Recurrence Rules column links subsequently confirmed these internal field names:
`Title`, `TaskId`, `RecurrenceType`, `ScheduleSlot`, `TimeZone`, `StartMode`, `ActiveFrom`,
`ActiveUntil`, `IsActive` and `TimeOfDay`. The built-in `Modified`, `Created`, `Author` and
`Editor` fields are not used by the Agent-7 occurrence contract.

The Task Occurrences column links subsequently confirmed these internal field names:
`Title`, `CaseId`, `TaskId`, `OccurrenceDate`, `ScheduleSlot`, `DueUtc`, `Status`,
`ProposedBy`, `ProposedUtc`, `ApprovedBy`, `ApprovedUtc`, `ApprovalState`, `CompletedUtc`
and `Notes`. Agent 7 writes only the creation fields required by the occurrence contract;
proposal, approval and completion audit fields remain unchanged.

### Remaining references and decisions required for full C5

The following must be obtained from the finalized SharePoint/Power Platform solution. Display
names are not sufficient:

1. The authoritative active-case and operating-mode source, including whether mode values are
   `DMP`/`Post-Default` or `PROD_DMP`/`SIMU_DMP`.
2. The final Power Apps trigger/response contract for `CreateTaskOccurrences` and
   `GenerateMissingOccurrences`, including inclusive `FromDate`/`ToDate`.
3. A persisted numeric sequence field if checklist order must be guaranteed independently of
   SharePoint item order.
