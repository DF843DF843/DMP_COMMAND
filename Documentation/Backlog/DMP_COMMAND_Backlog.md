# DMP COMMAND – Projektweites Backlog für größere, zurückgestellte Optimierungen

**Geltungsbereich:** Dieses Backlog gilt für das gesamte DMP COMMAND System — alle Agenten (Agent 1–6), die Power App (DMP COMMAND), sowie alle verwendeten Konfigurations- und Statuslisten/-dokumente (`DMP Command Configuration`, `DMP Command Agent Status`, u. a.).

**Pflege-Regel (in KI-Arbeitsregeln verankert am 2026-08-06):** Dieses Dokument wird regelmäßig aktualisiert, sobald bei der Arbeit an irgendeiner Komponente des Systems ein Finding entsteht, das bewusst auf später verschoben wird. Es ist NICHT auf einen einzelnen Agenten beschränkt.

**Archivierungs-Regel (ergänzt am 2026-09-04, siehe KI-Arbeitsregeln „Pflicht-Checkliste" Abschnitt G):** Vollständig erledigte, bestätigt live ausgelieferte Punkte werden regelmäßig aus diesem aktiven Dokument nach [`DMP_COMMAND_Backlog_Archive.md`](./DMP_COMMAND_Backlog_Archive.md) ausgelagert (nicht gelöscht). Der fachliche Kern jedes erledigten Punkts wird vorher in `DMP_COMMAND_Release_Notes.md` (versions-relevante Fixes/Features) bzw. `DMP_COMMAND_Operations_Manual.md` (betrieblich relevante Punkte) gesichert.

**Wichtiger Arbeitshinweis (gilt für jeden Punkt):** Vor Umsetzung IMMER zuerst den dann aktuellen Stand der jeweils betroffenen Datei(en) neu einlesen und gegen die hier beschriebenen Fundstellen prüfen (Feldnamen/Ausdrücke können sich durch zwischenzeitliche manuelle Änderungen verschoben haben). Keine neuen Config-Felder oder Variablen ohne Rücksprache mit dem Nutzer einführen.

---

## 🟠 v1.22.24 (2026-09-22, lokal gepackt, noch nicht in Studio geladen) — YAML-Fix + erste Task-Occurrences-4-Augen-Demo

1. **YAML-Fehler behoben (PA1001, „did not find expected key"):** Die in v1.22.23 hinzugefügte schließende Klammer für den Ring-Scope-Fix war exakt auf die Einrückung der folgenden `Width:`-Property eingerückt — Studios strengerer PaYaml-Parser beendete den Block-Skalar dadurch eine Zeile zu früh. Neu eingerückt; **`pac`s eigener Pack/Unpack-Rückvergleich hat das NICHT erkannt** (bekannte Grenze — nur Studio prüft PaYaml wirklich streng) — bei jeder künftigen mehrzeiligen Formel-Änderung besonders auf Einrückung achten.
2. **Task Occurrences — erste 4-Augen-Workflow-Demo (C6/C7-Vorschau):** Die Vorschau-Gallery läuft jetzt auf einer echten, patchbaren lokalen Collection (`colTaskOccurrencesPreview`, exaktes Schema der Live-Liste) statt einer statischen `Table()`. Jede Zeile hat jetzt Propose/Approve/Reject: "Propose" setzt Status weiter + `ApprovalState=Pending`; "Approve"/"Reject" erscheinen nur für einen ANDEREN Nutzer als den Vorschlagenden (4-Augen-Prinzip technisch durchgesetzt). Noch nicht mit der echten Liste verbunden (gleicher manueller "Add data"-Schritt in Studio wie bei B3 nötig).
   **Offene Design-Frage an den Nutzer:** Die echte Liste hat kein `PreviousStatus`-Feld — "Reject" setzt aktuell nur den Pending-Zustand zurück, macht die Status-Änderung selbst aber nicht rückgängig. Vor Live-Schaltung klären: neues Feld ergänzen, oder ist ein manuelles Neu-Vorschlagen nach Reject akzeptabel?

Pack/Unpack-Rückvergleich: 0 Diff auf allen 4 geänderten Dateien.

---

## 🟠 v1.22.23 (2026-09-22, historisch, in v1.22.24 gefaltet) — Ring-Scope-Bug + Agent-7-Release-Notes + Label-Fix

Aus dem 2. Live-Test-Feedback nach v1.22.22:

1. **Ring immer noch nicht sichtbar, neuer Fehler:** `vFixedSegments wird nicht erkannt`. Root Cause: `vGreenFixedSegments: CountIf(vFixedSegments, ...)` stand im GLEICHEN Record-Literal wie `vFixedSegments` selbst — Power-Fx-Records können keine Geschwister-Felder referenzieren, nur verschachtelte `With()`-Aufrufe können auf zuvor gebundene Namen zugreifen. In eine eigene `With()`-Ebene ausgelagert (gleiches Muster wie der Rest der Formel). Vermutlich seit Erstellung dieser Formel unentdeckt, nur bisher von anderen Fehlern verdeckt.
2. **Release Notes „Agent 7 fehlt" — derselbe Bug wie beim App-Tab, jetzt auch im Agent-Tab gefunden:** `varSelectedAgentRelease` war nie initialisiert, kein Switch-Case/Menü-Button für Agent 7 — fiel auf Agent 1 zurück. Fix: `btnAgentRelease7`, Switch-Case „7", Default auf „7".
3. **Popup-Label abgeschnitten:** „Termination Date + Time (local time, YYYY-MM-DD HH:M" gekürzt auf „Termination Date + Time (local) *" — das genaue Format steht bereits im Platzhaltertext des Eingabefelds.

Pack/Unpack-Rückvergleich: 0 Diff auf `scrHome.pa.yaml`, `scrReleaseNotes.pa.yaml`, `App.pa.yaml`.

---

## 🟠 v1.22.22 (2026-09-22, historisch, in v1.22.23 gefaltet) — 4 echte Root-Cause-Fixes aus Live-Test-Feedback

Der Nutzer hat v1.22.21 tatsächlich in Studio getestet und 4 konkrete Findings gemeldet — alle 4 sind jetzt mit echter Ursachenanalyse behoben, nicht nur symptomatisch:

1. **System Health Ring:** `CountIf(vFixedSegments, Color="rgb(...)")` scheiterte mit "Enum, Text nicht vergleichbar". Ursache: Das Feldname `Color` wird von Power Fx offenbar mit seinem eingebauten `Color`-Enum-Typ verwechselt, sobald es per `=` verglichen wird. Feld überall auf `SegColor` umbenannt — dabei eine ZWEITE, bisher unentdeckte Instanz desselben Bugs in der Sortier-Formel (`AddColumns(...,ColorRank, Switch(Color,...))`) gefunden und mitbehoben.
2. **Timestamp-Bug (Jahr 3926):** Das Debug-Panel bewies live: `Date(1899,12,30)+Value(rawTs,"en-US")` liefert bei einer echten Zeile (`RAW=46287.3776736574`) `3926-09-23 00:00:00` statt `2026-09-...`. Root Cause: `Date(1899,12,30)` liegt außerhalb des von Power Apps unterstützten Datumsbereichs (Minimum 1900-01-01) und lieferte bisher STILLSCHWEIGEND Müll statt eines Fehlers — dieselbe kaputte Epoche steckt seit mindestens einem Commit vom 3. September unverändert im Code (nur die Zweig-Reihenfolge wurde damals geändert, nie die Mathematik selbst). Fix: Community-Standard-Konvertierung `DateAdd(Date(1900,1,1),(Value(rawTs,"en-US")-2)*86400,TimeUnit.Seconds)`, an allen 20 Stellen in `scrAuditTrail.pa.yaml` plus im Debug-Panel angewendet.
3. **Release Notes zeigt immer v1.22.13:** Root Cause gefunden — `varSelectedAppRelease` wurde nirgends initialisiert, und der große Versions-Switch hatte seit v1.22.13 KEINEN einzigen neuen Fall mehr bekommen (jede neue Version wurde nur der scrollbaren Liste hinzugefügt, nicht diesem separaten Detail-Pane-Switch) — das war also kein Stale-Studio-Problem, sondern ein echter, bisher unentdeckter App-Bug. Fix: Default auf aktuelle Version gesetzt + neuer Switch-Case/Menü-Button ergänzt (ältere Lücke v1.22.14–v1.22.20 bewusst nicht rückwirkend gefüllt — siehe Priorität 3).
4. **B3-Popup:** Titel war lila Text auf ggf. dunklem Hintergrund (schwer lesbar im Dark Mode) — jetzt weißer Text auf farbigem Banner (gleiche Konvention wie beim Button-Kontrast-Fix aus v1.22.17). Datum/Zeit-Feld verlangte bisher UTC-Eingabe — jetzt lokale Zeit, Umrechnung nach UTC (`TimeZoneOffset`) erst beim Speichern in die Collection.

**Bewusst zurückgestellt (Priorität 3):** Die Release-Notes-Menü-/Switch-Lücke für v1.22.14–v1.22.20 (8 Versionen) wurde nicht rückwirkend aufgefüllt, da für den gemeldeten Fehler nur "zeigt die aktuelle Version standardmäßig" nötig war — bei Bedarf nachträglich ergänzbar.

Pack/Unpack-Rückvergleich: 0 Diff auf allen 5 geänderten Dateien.

---

## 🟠 v1.22.21 (2026-09-22, historisch, in v1.22.22 gefaltet) — gebündelter Fix + Debug-Panel + B3-Popup

- `vGreenFixedSegments` (System Health ring) enthielt trotz gegenteiliger Release-Notes-Behauptung (v1.22.17/18) weiterhin den ungültigen `Table(vStatusCheckColor, ...)`-Aufruf mit Text-Skalaren (vom Nutzer per Studio-Fehleranalyse gefunden) — jetzt echt korrigiert auf `CountIf(vFixedSegments, Color=...)`.
- Temporäres Panel `conFuncTimestampDebug` auf dem Admin-Functions-Screen (4 parallele Datums-Parsing-Ansätze anhand echter Live-Daten) — Quellcode-Prüfung ergab, dass `scrAuditTrail.pa.yaml` bereits die robuste numerische Vorprüfung enthält und Agent 7 in den Release Notes bereits vollständig gepflegt ist; der gemeldete alte Stand stammt sehr wahrscheinlich aus einer seit 2026-09-04 nicht neu geladenen Studio-Session.
- **B3 (Default Case Context) — erste Umsetzung:** `conDefaultCaseContextPopup` in `scrHome.pa.yaml` — Pflichtformular (Case ID, Termination Reason, Defaulted Member ID/Name, Termination Date+Time), das jetzt jeden Pre-Default→DMP-Übergang von `btnOperationalModeAdvance` gate't. **Korrektur zur bisherigen Doku:** Die Liste `DMP Command Default Case Context` (Achtung: realer Name OHNE „Checklist", anders als Recurrence Rules/Task Occurrences) existiert laut Nutzer-Screenshot bereits (Site `GO365_DMPCommunication-CoSLeader`, GUID `8fa1f858-3f89-4600-b3dc-86f9dfcaf4b8`), Spalten wurden vom Nutzer bestätigt (inkl. interner Feldname `TerminationDate` für die Anzeige-Spalte „TerminationDateTime"). Ebenso existieren laut Screenshot bereits ALLE anderen Streams-Listen (Email Templates, Recipient Groups, Role Assignments, Status Change Approvals, Email Placeholders) — die bisherige Annahme „nur Recurrence Rules/Task Occurrences existieren" war veraltet/falsch und ist hiermit korrigiert. Noch offen: Liste als echte Power-App-Datenquelle in Studio verbinden (manueller Schritt, wie bei Agent 7) — bis dahin schreibt das Popup nur in eine lokale Collection (`colDefaultCaseContextPending`, exakt gleiches Spaltenschema).
- **Lessons Learned Control-Typ:** `Classic/TextInput` wurde in dieser App noch nie verwendet. Ein erster geratener Name (`TextInput@2.3.2` ohne „Classic/"-Präfix) wurde lokal erkannt (fehlte im gepackten Template-Register) und VOR dem Test an den Nutzer gestoppt; der Nutzer hat testweise ein Textfeld in Studio eingefügt und den echten Namen `Classic/TextInput@2.3.2` geliefert — damit korrigiert, alle 5 neuen Felder nutzen jetzt die verifizierte Deklaration.
- Pack/Unpack-Rückvergleich: 0 Diff für alle 3 geänderten Screens (`scrHome`, `scrAdminFunctions`, `scrReleaseNotes`) + `App.pa.yaml`.

---


## 📌 Große Aufräum-Aktion am 2026-09-04

Dieses Dokument war auf ca. 2650 Zeilen angewachsen (chronologisches Arbeitsprotokoll seit 2026-08-06) und wurde vor einer 2-wöchigen Nutzer-Abwesenheit komplett neu strukturiert:

- Die **gesamte bisherige Historie** (alle Root-Cause-Analysen, Architektur-Entscheidungen, abgeschlossene Punkte) wurde 1:1 unverändert nach [`DMP_COMMAND_Backlog_Archive.md`](./DMP_COMMAND_Backlog_Archive.md) übernommen (Volltext-durchsuchbares Nachschlagewerk).
- Dieses Dokument enthält ab jetzt **nur noch das, was tatsächlich noch offen ist**, sortiert nach Priorität/Bereich.
- Alle als "erledigt" erkannten Punkte sind bereits in `DMP_COMMAND_Release_Notes.md` (versionsbezogen) bzw. `DMP_COMMAND_Operations_Manual.md` §7/§8 (betrieblich) nachgezogen.
- Falls ein Punkt aus der alten Historie fehlt, den du noch für offen hältst: im Archiv per Volltextsuche (Strg+F) nach Stichwort suchen und hier wieder ergänzen.

---

# 🔴 Priorität 1 – Bereit zum Deploy, wartet auf grünes Licht des Nutzers

## 🟢 B1/B2 (Streams-Konzept, Strang B – 5-Modus-Umstellung) – umgesetzt, Studio-Validierung durch Nutzer noch offen

**B1 – Configuration-Erweiterung (Variante B, 8 Spalten):** Klargestellt und vorbereitet
2026-09-22. `DMP Command Configuration` braucht 4 neue Spalten (`Value - PROD (Pre-Default)`,
`Value - PROD (Post-Default)`, `Value - SIMU (Pre-Default)`, `Value - SIMU (Post-Default)`,
Typ „Einzelne Zeile Text"), die der Nutzer manuell in SharePoint anlegen muss (kein PnP-/API-
Zugriff verfügbar). Werte für 10 bereits vollständig vorbereitete Zeilen (`AuditModeDecision`,
`AuditModeText`, `CurrentOperationMode`, `Agent2EffectiveModeMapping`,
`OperationModeAllowedNextSteps`, `MailModeText`, `ModeIconName`, `OperationModeDescription`,
`OperationModeDisplayName`, `OperationModeShortLabel`) sowie 16 neu ergänzte Zeilen
(`DashboardStatusColor`, `DMPStateDisplayName`, `EnvironmentDisplayName`, `BannerText`,
`BannerSubText`, `AlertEmailRecipient`, `MailImportanceActionRequired/Error/Info/Warning`,
`SubjectPrefix`, `SharedDMPMailbox`, `ProcessedMailsRootFolderName`,
`WaitSecondsBeforeSentMailSearch`, `Agent5AlertFolderName`, `WorkflowPathAgent5`) stehen fertig
in `DMP Command Configuration.csv` (beide Kopien) — **noch nicht in die echte SharePoint-Liste
übertragen, wartet auf Nutzer.** Kein dritter „TEST"-Umgebungswert nötig: `TEST` entspricht der
bereits getrennten Power-Platform-Umgebung `DBG Team Productivity (Dev)`/`(UAT)`, nicht einem
Konfigurationswert (Klarstellung 2026-09-22).

**Update 2026-09-22 (Nachmittag):** Nutzer hat `DMP Command Configuration.csv` neu erstellt und
alle 8 Werte-Spalten für jede Zeile befüllt (beide Doku-Kopien synchronisiert, 32 KB, vorher
26 KB). **Noch zu klären: ist das nur die Vorbereitung in der CSV-Datei, oder wurden die 4
neuen Spalten inkl. Werte auch schon in der echten SharePoint-Liste `DMP Command Configuration`
angelegt?** Die Survey-Datei (Stand vor diesem Update) zeigte dort noch nur die 4
ursprünglichen `Value - PROD/SIMU (NODMP/DMP)`-Spalten, keine der 4 neuen Pre-/Post-Default-
Spalten. Solange das nicht bestätigt ist, bleibt B1 formal offen.


**B2 – 5-Werte-Zustandsmodell in der App:** Implementiert 2026-09-22 in
`PowerApp/DMP_COMMAND/Source/Src/scrHome.pa.yaml` + `App.pa.yaml` (App-Version `v1.22.15`,
noch nicht vom Nutzer in Studio geladen/gespeichert/getestet). Der bisherige Normal/DMP-Toggle
(`tglOperationalState`) wurde durch einen einzelnen „Weiter"-Button (`btnOperationalModeAdvance`,
`Classic/Button`) ersetzt, der die aktuelle Phase und die nächste anzeigt (z. B. „Normal ->
Pre-Default") und pro Klick genau einen Schritt im zyklischen Ablauf `Normal -> Pre-Default ->
DMP -> Post-Default -> Normal` auslöst — ein direkter Sprung zwischen nicht benachbarten Phasen
ist technisch ausgeschlossen. Neue Variablen `varOperationalMode` (Text) und
`varOperationalStepCounter` (Zahl, monoton steigend, Anzeige via `Mod(...,4)`) ergänzen die
bestehende `varOperationalModeIsDMP` (bleibt aus Kompatibilitätsgründen abgeleitet erhalten,
u. a. für bestehende Farb-/Rahmen-Formeln). Environment-Wechsel zu PROD setzt weiterhin
automatisch auf „Normal" zurück (jetzt aus allen 4 Phasen, nicht nur DMP); Wechsel zu SIMU
erhält die aktuelle Phase (alle 4, nicht nur DMP/Normal).

**Bewusst KEIN nativer Power-Apps-Slider-Control verwendet** (Nutzerwunsch war ein
„Schiebeschalter"): Diese Codebasis hat noch nie einen Slider-Control-Typ verwendet; ein
Rateversuch bei Control-Typ/Version hätte dasselbe Risiko wie frühere stille Power-Fx-/Schema-
Fehler bedeutet. Stattdessen ein bereits im Screen 20+ Mal bewährter `Classic/Button`, farblich
als Pille gestylt. Falls der Nutzer nach Sichtprüfung in Studio dennoch einen echten Drag-Slider
möchte: Control in Studio selbst einfügen (garantiert korrektes Schema), danach die
Min/Max-Begrenzungslogik (`[varOperationalStepCounter, varOperationalStepCounter+1]`) darauf
verdrahten.

**Solution-Version-Prüfsumme:** Durch die Agent-7-Label-Korrektur UND die Power-App-
Versionsänderung zweimal neu berechnet und importiert: `7.11.48` → `7.34.46` → `7.34.48`
(siehe Session Restart Guide für die vollständige Komponententabelle). Offene Frage an den
Nutzer: ob künftig JEDER Power-App-Versions-Bump einen eigenen Solution-Reimport auslösen soll,
oder ob das mit dem nächsten ohnehin fälligen Flow-Import gebündelt werden darf.

**Noch offen:**
1. Nutzer muss die 20 Configuration-Zeilen (B1) manuell in SharePoint anlegen/befüllen.
2. Nutzer muss die Power App neu laden/speichern und den neuen Button in Studio visuell/
   funktional prüfen (Layout wurde nicht Studio-validiert, nur JSON/YAML-seitig auf Klammern-
   Balance und Referenz-Konsistenz geprüft).
3. Agent 7 muss nach dem zweiten Reimport (`7.34.48`) erneut geöffnet/gespeichert/aktiviert
   werden (Power Automate meldete wieder "deactivated and replaced").
4. B3 (`DMP Command Checklist Default Case Context`-Liste + Popup `popDefaultCaseContext`),
   B4 (Agent 5 Pre-/Post-Default-Mail-Logik), B5 (Agent 2 EffectiveMode-Mapping nutzen) folgen
   erst nach B1/B2-Bestätigung durch den Nutzer.

## 🔵 Agent 7 occurrence generation – activated in Dev, final C5 contract pending

**Deployed to Dev 2026-09-22, corrected to `7.34.46` same day:** Solution `7.11.48` was
imported and published in `DBG Team Productivity (Dev)` (`pac solution import
--publish-changes`, succeeded; Power Automate reported "The original workflow definition has
been deactivated and replaced" as expected for a workflow-definition update). **Agent 7
needs to be reactivated in the designer and saved** before it can run — this is the real
save/activate test for this release (unlike `7.11.47`, this version has real content
changes, so Save will not be greyed out). Immediately afterwards, `7.11.48` was found to be
a wrong, free-running version number: the Solution version MUST be the checksum (Σ Major, Σ
Minor, Σ Patch) of all 8 components (7 agents + Power App), and Agent 7's own label had
stayed stale at `[0.2.0]` since `7.11.36`. Corrected in a second, versioning-only re-import:
Agent 7 → `[0.3.1]`, Solution → **`7.34.46`** (no further flow-logic change). See
`DMP COMMAND_Mission_und_KI_Arbeitsregeln.md` section I for the formula and the component
table.

**Verified Dev state (2026-09-22, pre-`7.11.48`):** Solution `7.11.47` was imported and
published only in `DBG Team Productivity (Dev)` (Agent 7 workflow component `[0.2.0]`).
The user confirmed it opens error-free and is activated ("Ein"); Save could not be
separately tested because the designer only enables Save after a real change — which
`7.34.46` now provides.

The misleading `$schema` designer failure (pre-`7.11.46`) was isolated to an invalid
SharePoint Create Item parameter. Task Occurrences `ScheduleSlot` is a Choice and must be sent
through `item/ScheduleSlot/Value`, not `item/ScheduleSlot`. The SharePoint site binding was
also corrected to `GO365_DMPCommunication-CoSLeader`.

Implemented as of `7.11.47`: DMP gate, active Daily rule read, canonical OccurrenceId, exact
lookup-before-create idempotency, time-zone conversion to `DueUtc`, create for missing items,
**and now**: `RequestedAction` contract (`CreateTaskOccurrences` with `BusinessDate` /
`GenerateMissingOccurrences` with inclusive `FromDate`/`ToDate`), removal of the obsolete
`OccurrencesJson` trigger input, inclusive multi-day date-range expansion, explicit
validation (unsupported action, missing/out-of-order dates) surfaced via
`validationError`/`message`, and reliable `createdCount`/`skippedCount` response counters.

**Hardening implemented in `7.11.48` (imported/published in Dev 2026-09-22, pending user
activate/save confirmation):** the flow now additionally contains:
- Rule-level `TimeOfDay`/`TimeZone` existence validation inside `APPLY_Rules`
  (`CHECK_RuleScheduleFieldsValid`): a rule missing either field is skipped gracefully
  (logged into a new `ErrorMessages` array + `ErrorCount` variable) instead of failing the
  whole run in `convertTimeZone`.
- A real per-item error counter: `CREATE_Occurrence` is now wrapped in `SCOPE_CreateOccurrence`
  with a Failed/TimedOut catch branch (`COMPOSE_CreateOccurrenceError` →
  `APPEND_CreateOccurrenceError` → `INCREMENT_ErrorCount_CreateFailed`) that increments the same
  `ErrorCount`/`ErrorMessages` instead of failing the run.
- A temporary placeholder validation on the caller-supplied `CaseId` (`VALIDATE_CaseId`):
  a blank/whitespace-only `CaseId` now sets `ValidationError` before any SharePoint read/write,
  explicitly marked as a stand-in until the real active-case lookup (B3) replaces it.
- `RESPOND_Result` now also returns `errorCount` and `errorDetails` (joined `ErrorMessages`),
  and `success` is now `false` whenever `ErrorCount > 0`, not only on `ValidationError`.

`7.11.48` was imported via `pac solution import --publish-changes` and published
successfully. Power Automate reported "The original workflow definition has been deactivated
and replaced" (expected for any workflow-definition update) — **Agent 7 needs to be reopened,
saved, and reactivated by the user in the designer before it can run again.** Unlike
`7.11.47`, this version has real content changes, so Save will not be greyed out and is a
real test this time.

Still open:

1. ~~Replace/remove `OccurrencesJson` from the final production contract.~~ Done in `7.11.47`.
2. ~~Add `FromDate` and `ToDate` and inclusive multi-day generation.~~ Done in `7.11.47`.
3. ~~Rule-level `TimeOfDay`/`TimeZone` existence validation.~~ Done in `7.11.48` (see above);
   pending the user's designer save/activate confirmation.
4. Resolve authoritative active-case/mode lookup and final mode values. Central config
   `CurrentOperationMode` (Global/Runtime) already carries
   `PROD_NODMP/PROD_DMP/SIMU_NODMP/SIMU_DMP/PROD_PREDEFAULT/PROD_POSTDEFAULT/SIMU_PREDEFAULT/SIMU_POSTDEFAULT`
   and is reused by other agents, but there is still no discovered central "active
   CaseId" source anywhere in the app/config. Per the confirmed work order, B1–B3 (Streams
   concept) are inserted next to build this; until then, `VALIDATE_CaseId` (above) only
   rejects a blank `CaseId`, it does not verify the case is real/active.
5. ~~Add trustworthy created/skipped/error counters.~~ Created/skipped done in `7.11.47`;
   the per-item error counter (`Scope`/try-catch around `CREATE_Occurrence`) is done in
   `7.11.48` (see above); pending the user's designer save/activate confirmation.
6. Add a numeric `Sequence` field if checklist order must be guaranteed.
7. Connect the prepared Task Occurrences Power App screen to live SharePoint/Agent 7.
8. Run the first non-destructive Dev end-to-end and idempotency test (still fully open,
   including for the new `7.11.47` date-range contract and this hardening).

Do not reintroduce `item/TimeZone` into Create Item unless the connector schema confirms that
field. Do not restore pre-`7.11.46` Agent 7 definitions.

## P1 – Published System Health ring has a Power Fx type error (first fix next session)

**Live state (2026-09-04):** The Power App was loaded, saved, published, and deployed, but `scrHome.imgHeartbeatWheel.Image` shows **"Only record or table values can be used in that context."** The faulty formula attempts to combine a fixed table with `ForAll(vAgents As agent, {...})` through `vFixedSegments & ForAll(...)`. `pac canvas pack` did not detect this Studio-time Power Fx error.

**Required fix:** Start from a freshly downloaded live app in the local worktree. Rebuild the dynamic segment table with a Studio-supported Power Fx table composition; do not assume the previous `&` operation is valid. Test the formula directly in Power Apps Studio before creating any package.

**Approved functional design:** equal-sized segments for Status Check, Critical Events, Warnings, Operating State, Emergency Report processing, Internal Domains, External Domains, Counter, Audit Trail, plus one dynamic segment for every `DMP Command Agent Status` row whose `AgentKey` starts with `Agent_`. Agent 7 and every future agent must therefore appear automatically. Sort segments Green → Orange → Red → Grey, then alphabetically inside a color group. Emergency Report is only non-green after a processing failure; a missing report is not an error. Critical >0 is red; Warnings >0 is orange; a failed Operating-State switch is red until the next successful switch.

**Mandatory validation:** Power Apps Studio accepts the formula without error, then pack/unpack round-trip, then import/save/publish and check the live ring with at least one non-green factor.

## ✅ Erledigt am 2026-09-04: Agent 2 läuft trotz ungültiger Power-Automate-Expression nicht fachlich korrekt durch

**Nutzer-Feedback vom 2026-09-04:** Agent Monitoring zeigte Agent 2 rot, während das Cockpit weiterhin "alles grün" meldete. Die Nutzerprüfung des letzten Agent-2-Laufs bestätigte einen kritischen Fehler in `Parse internal domains file + create array`:

`Unable to process template language expressions in action 'Parse_internal_domains_file_+_create_array' inputs at line '0' and column '0': 'The template function 'select' is not defined or not valid.'`

**Root Cause / Korrektur der bisherigen Annahme:** Der frühere Fix-Hinweis "select() mit coalesce() guard" war unzureichend bzw. falsch: `select()` ist an dieser Stelle keine gültige Power-Automate-Template-Funktion. `Select` ist in Power Automate als eigene Data-Operation-Aktion zu modellieren, nicht als inline expression function in einem Compose-/Expression-Feld.

**Fachliche Auswirkung:** Agent 2 kann technisch als Lauf "durchgelaufen" erscheinen, obwohl die interne Domains-Klassifizierung nicht funktioniert hat. Das Cockpit erkennt diesen Agent-Status derzeit nicht zuverlässig; Agent Monitoring zeigt dagegen den Agent-Status rot. Zusätzlich wurde bei diesem kritischen Fehler offenbar **keine Alert-E-Mail** versendet. Das ist ein P1-Fehler in der Fehler-/Alarmierungsarchitektur, nicht nur ein UI-Problem.

**Fix umgesetzt und live importiert:** Solution `DMP_COMMAND_Solution` wurde von 7.11.34 auf **7.11.35** angehoben und in der Umgebung **DBG Team Productivity (Dev)** importiert/veröffentlicht. Agent 2 wurde von `[1.0.8]` auf **`[1.0.9]`** angehoben. Die beiden ungültigen inline-`@select(...)`-Expressions in `Parse_internal_domains_file_+_create_array` und `Parse_external_domains_file_+_create_array` wurden durch echte `Query`/Filter-array-Aktionen ersetzt, die die SharePoint-Listenzeilen auf `Title = SenderDomainNormalized` filtern. Zusätzlich wurde im vorhandenen technischen `Error_handling`-Scope eine verpflichtende kritische Alert-E-Mail `[Agent 2][FAILED] Technical Main Flow Failed [EC:A2-MAINFLOW-FAILED][RID:…]` ergänzt.

**Nicht verhandelbare Akzeptanzkriterien für den Fix:**
- Jeder technische Fehler in der internen/externalen Domains-Klassifizierung muss `AuditOutcome=Failed` setzen und als kritischer Fehler im Audit Trail erscheinen.
- Der Agent-2-Status in `DMP Command Agent Status` muss danach `CurrentStatus=Failed`, `LastRunResult=Failed`, `StatusSeverity=Critical` und eine konkrete `StatusMessage` mit Aktionsname + Power-Automate-Fehlertext enthalten.
- Das Cockpit muss diesen Zustand als kritisch erkennen; "alles grün" darf bei `StatusSeverity=Critical` oder `CurrentStatus=Failed` eines Agenten nicht möglich sein.
- Es muss eine Alert-E-Mail an `AlertEmailRecipient` mit Fehlercode, RunId und Aktionsname verschickt werden, z. B. `[Agent 2][FAILED] Internal Domains Parse Failed [EC:A2-INTDOMAINS-PARSE][RID:…]`.
- Falls der Fehler so früh passiert, dass der normale Klassifizierungs-/Audit-Pfad nicht mehr erreicht wird, nutzt Agent 2 den übergeordneten `Error_handling`-Scope mit `runAfter` auf `Failed/TimedOut`, der Statuszeile und Alert-E-Mail schreibt.

**Validiert:** Live-Solution exportiert, geändert, JSON/XML geparst, 0 verbleibende inline-`@select(`-Treffer, keine >255-Zeichen-Descriptions gefunden, `pac solution pack` erfolgreich, `pac solution import --publish-changes true` erfolgreich (`The original workflow definition has been deactivated and replaced.`). Noch offen ist nur der fachliche Live-Test mit einer Test-Mail, um Alert-Mail-/Status-/Cockpit-Verhalten end-to-end zu bestätigen.

---

## ✅ v1.22.13 gepackt und bereit — ECHTE Root Cause der kaputten Buttons gefunden + Datumsanzeige robuster gemacht

**Nutzer-Feedback nach dem ersten Laden/Veröffentlichen von v1.22.12:** „Die Buttons funktionieren noch nicht" — trotz bereits behobener `.msapr`-FlowNameId und `IfError`-Record-Shape-Probleme.

**Fund 1 — Tatsächliche Root Cause der Buttons (bestätigt durch Download der veröffentlichten Live-App und Inspektion von `References\DataSources.json`):** Der interne Power-Fx-Datenquellenname für Agent 6 lautet in der Live-App **`DMPAgent6(AdminFunctions)[1.1.0]`** (mit Versions-Suffix in eckigen Klammern) — vermutlich entstanden, als der Nutzer diesen Sommer alle Agenten in Studio entfernt und neu hinzugefügt hat; Studio übernimmt beim Neu-Hinzufügen den AKTUELLEN Anzeigenamen des Flows aus Power Automate 1:1, inklusive des dort offenbar vorhandenen „[1.1.0]"-Zusatzes. Alle 4 anderen Agenten-Datenquellen (Agent 3, 4, 5) haben KEINEN solchen Suffix. Alle Formeln im Repo riefen jedoch weiterhin `'DMPAgent6(AdminFunctions)'.Run(...)` **ohne** diesen Suffix auf — ein Name, der in der Live-App gar nicht mehr existiert. Das GUID-basierte `.msapr`-`FlowNameId`-Abgleich hatte diesen reinen String-Namens-Mismatch nicht aufgedeckt.

**Fix:** Alle 5 Fundstellen auf den exakten Live-Namen `'DMPAgent6(AdminFunctions)[1.1.0]'` korrigiert (`scrAdminFunctions.pa.yaml` ×2, `scrAuditTrail.pa.yaml` ×3).

**⚠️ Wichtiger Hinweis für die Zukunft:** Sollte der Agent-6-Flow in Power Automate jemals umbenannt werden (z. B. der „[1.1.0]"-Zusatz entfernt) und die Verbindung in Studio erneut entfernt/neu hinzugefügt werden, ändert sich dieser interne Name vermutlich wieder — dann müssten alle 5 Formelstellen erneut angepasst werden. Empfehlung: den Flow-Anzeigenamen in Power Automate dauerhaft auf `DMP Agent 6 (Admin Functions)` (ohne Versions-Zusatz) vereinheitlichen, damit dieses Muster nicht erneut auftritt.

**Fund 2 — Datumsanzeige im Audit Trail weiterhin fehlerhaft, trotz vorherigem Fix:** Die bisherige Logik versuchte zuerst `DateTimeValue(rawTs,"en-US")` (für ISO-Text) und fiel erst bei einem Fehler auf die Excel-Serial-Arithmetik zurück. Ein per Diagnose-Label sichtbar gemachter Rohwert (`"46267.5244033102"`) zeigte, dass Agent 4 für diese Zeilen weiterhin reine Excel-Seriennummern liefert (trotz `dateTimeFormat: ISO 8601` auf der zuständigen `GET_AuditTrail_AllRows`-Aktion — vermutlich weil die Spalte `TimestampUtc` in der Excel-Tabelle nicht als echter Datumstyp, sondern als allgemein/Zahl formatiert ist, wodurch der Connector-Parameter wirkungslos bleibt). Vermutlich parste `DateTimeValue` diese reine Zahl fälschlich "erfolgreich" (statt mit Fehler abzubrechen), wodurch die eigentlich korrekte Serial-Arithmetik-Fallback-Logik nie zum Zug kam und ein zusätzlicher "-1900-Jahre"-Nothelfer (aus einer früheren Sitzung) nicht zuverlässig griff.

**Fix (robuster, deterministischer Ansatz, an allen 20 Stellen in `scrAuditTrail.pa.yaml`):** Der Rohwert wird jetzt ZUERST versuchsweise als reine Zahl geparst (`Value(rawTs,"en-US")`). Gelingt das, wird er als Excel-Seriennummer behandelt (`Date(1899,12,30) + Zahl`) — eindeutig und ohne Rateheuristik. Nur wenn die Zahl-Konvertierung fehlschlägt (echter ISO-Text), greift der `DateTimeValue`-Textparser. Der bisherige "-1900-Jahre-falls-Jahr-unplausibel"-Nothelfer entfällt komplett, da die Ursache jetzt an der Wurzel vermieden wird statt nachträglich zu korrigieren.

**Fund 3 — UX-Wünsche umgesetzt:**
- Der "Loading…"-Hinweis bei den Audit-Trail-Aktionen zeigt jetzt kontextabhängigen Text (`"Resetting Critical baseline..."`, `"Resetting Warning baseline..."`, `"Resetting all baselines..."`, `"Refreshing audit data..."`) statt eines generischen `"Loading recent alerts..."`.
- Position vertikal auf Höhe der Knöpfe-Mitte zentriert (Y an Button-Mittelachse ausgerichtet), horizontal etwas weiter nach rechts verschoben.
- Das temporäre Diagnose-Label (`lblAuditTrailDiagnosticRawTimestamp`) wurde entfernt, nachdem es seinen Zweck (Aufdeckung von Fund 2) erfüllt hat.

**Versionierung:** App-Version auf **v1.22.13** angehoben (v1.22.12 bleibt als historischer, tatsächlich veröffentlichter Eintrag stehen; die danach gefundenen Fixes gehören korrekterweise in die neue Version). `PowerApp_Version.txt`, Release Notes (Dokument + In-App-Screen) aktualisiert.

**Validiert:** YAML-`": "`-Bug-Scan app-weit: 0 Treffer. Klammern-/Geschweifte-Klammern-Balance in `scrAuditTrail.pa.yaml`: ausgeglichen. Pack→Unpack-Rückvergleich: 0 Diff über alle Screens. 0 verbleibende Treffer des alten Agent-6-Namens ohne Suffix.

**Power-Automate-Seite (Agenten/Flows): BEREITS LIVE, kein weiterer Schritt nötig.** Solution-Version 7.11.34 ist sowohl im Repo (`Solution.xml`) als auch live in der Umgebung „DBG Team Productivity (Dev)" identisch — Agent 4 v1.4.4 und Agent 6 v1.3.1 sind bereits produktiv. Für die Fixes in diesem Abschnitt war **keine** Flow-Änderung nötig (alles rein App-seitig).

**Warum der letzte App-Schritt NICHT von der KI ausgeführt werden kann:** Canvas Apps sind in diesem Projekt kein Bestandteil der Dataverse-Solution; `pac` (Version 2.11.2) bietet nachweislich KEINEN Befehl, der ein bereits veröffentlichtes Canvas-App-Update direkt in die Umgebung schreibt. Der einzige Weg ist das manuelle Laden in Power Apps Studio.

**Nächster Schritt für den Nutzer:** `DMP_COMMAND_Solution.msapp` erneut in Power Apps Studio öffnen (Datei importieren/ersetzen), speichern, veröffentlichen.

---

# 🟡 Priorität 2 – Größere offene Feature-Vorhaben

## 1. DMP Command Streams (Next Steps Container) — Teilumsetzung C5 in Dev

Aus dem Brainstorming vom 2026-09-0x entstandenes Vorhaben: pro Working Sub Stream (CoS Leader, Infrastructure Team, Content Team, Hotline Team) eine eigene digitale Checkliste (analog zu den bestehenden Excel-Dateien `CoSLeader Checklist.xlsx`, `Infrastructure Checklist.xlsx`), die automatisch E-Mails versendet, Status in einer Gesamtübersicht zusammenführt, und ein 4-Augen-Prinzip je Team durchsetzt.

**Wichtige fachliche Ergänzungen aus dem Brainstorming (noch nicht in das Konzeptdokument voll eingearbeitet — beim Wiederaufsetzen zuerst prüfen):**
- **5 statt 4 Betriebsmodi:** Neue Zwischenzustände **Pre-Default** und **Post-Default** zusätzlich zu Normal/DMP. Neue Reihenfolge: `Normal → Pre-Default → DMP → Post-Default → Normal`. Agent 2 behandelt Normal/Pre-Default/Post-Default identisch (nur DMP ist fachlich unterschiedlich).
- **Formular bei Umschaltung auf DMP:** Beim Umschalten auf PROD/DMP muss ein autorisierter User ein strukturiertes Formular ausfüllen (Termination Reason, Defaulted Clearing Member ID/Name, Termination Date/Time, u. a.) — diese Werte fließen dann in die automatisch generierten E-Mails ein (Ersatz für die bisher gelb markierten, manuell auszufüllenden Freitextstellen in den Word-Templates).
- **E-Mail-Versand:** Über einen neuen Agenten (Agent 7), Versand über die bestehende `Default@eurex.com`-Mailbox (wie alle anderen Agenten), mit Bestätigungsschritt vor dem Versand, Archivierung und Audit-Trail-Dokumentation. Empfänger über Sammel-E-Mail-Adressen je Team. E-Mail-Templates müssen in ein parametrisierbares Format überführt werden (analog zu allen bestehenden Agenten-Templates, PROD/SIMU × Non-DMP/DMP).
- **Assign Roles:** Der bestehende "Assign Roles"-Task ist nicht nur eine Stream-Zuordnung, sondern muss auch steuern, welche Bildschirme/Aktionen ein Nutzer in der App sehen/bedienen darf — **direkte Überschneidung mit dem separat unten aufgeführten „User-Access-Konzept"**. Beide Anforderungen sollten in einem gemeinsamen Rollenkonzept zusammengeführt werden, nicht getrennt gebaut werden.
- **Meilenstein-getriebene E-Mails:** Sobald ein Team einen Meilenstein erreicht (Status "Done" in seiner Teilliste), müssen automatisch E-Mails an andere Teams ausgelöst werden — setzt sauber gepflegte Status voraus (4-Augen-Prinzip beim Status-Update selbst, nicht nur beim E-Mail-Versand).
- Der CoS Leader braucht eine eigene Masterseite mit Echtzeit-Einblick in den Status aller Teams.

**Bereits vorbereitet:** Konzeptdokument [`DMP_Command_Streams_Feature_Konzept.md`](../Backlog/DMP_Command_Streams_Feature_Konzept.md) und SharePoint-Listen-Spaltenvorlagen in [`Streams_ListTemplates/`](../Streams_ListTemplates) (10 CSVs + README). SharePoint-Listenanlage ist ein manueller Nutzer-Schritt (kein List-API-Zugriff aus dieser Umgebung möglich).

**Status (2026-09-21):** Die Listen `DMP Command Checklist Recurrence Rules` und
`DMP Command Checklist Task Occurrences` sind vorhanden. Agent 7 ist als erste
C5-Teilumsetzung in Dev `7.11.46` aktiviert; die Task-Occurrences-Appseite ist lokal
vorbereitet. Der übrige Streams-Umfang (Rollen, 4-Augen-Prinzip, E-Mail-Automatisierung,
Master-/Teamseiten und vollständiger C5-Datumsbereichsvertrag) bleibt offen und wird in
Etappen umgesetzt.

## 2. Audit Trail / Counter: Archivierung + Reset mit 4-Augen-Prinzip

Ein - in den Einstellungen versteckter - Schalter wird benötigt, um Audit Trail und Counter zu archivieren und danach zurückzusetzen, idealerweise mit 4-Augen-Prinzip (zweite Person bestätigt vor Ausführung). **Wichtig bei Umsetzung:** Das 4-Augen-Prinzip-Muster, das für DMP Command Streams (Team-Status-Freigabe) entwickelt wird, sollte hier wiederverwendet werden statt eine zweite, eigenständige Lösung zu bauen.

**Status:** Nur als Anforderung dokumentiert, nicht begonnen.

## 3. Error-Code-Klassifizierung für Audit Trail

Feature zur strukturierten Klassifizierung von Audit-Trail-Fehlern nach Error-Codes (Details/Scope siehe Archiv). **Status:** Nicht begonnen, Scope mit Nutzer noch nicht final abgestimmt.

## 4. User-Access-Konzept (Rollen → Bildschirme/Aktionen)

Ursprünglich als eigenständiges Vorhaben identifiziert (rollenbasierte Sichtbarkeit von Menüpunkten/Aktionen, Admin-Freigabe-Workflow). **Überschneidet sich inhaltlich vollständig mit dem "Assign Roles"-Punkt von DMP Command Streams** (siehe oben) — sollte beim Wiederaufsetzen als EIN gemeinsames Rollenkonzept behandelt werden, nicht als zwei getrennte Features.

## 5. Individuelle Farbeinstellungen (Personalisierung)

Nutzer sollen künftig eigene Akzent-/Themenfarben in den App-Einstellungen festlegen können, statt nur des festen Eurex-Farbschemas. **Priorität:** niedrig, reine Komfort-Erweiterung.

---

# 🟢 Priorität 3 – Kleinere technische Restposten (niedrige Priorität)

- **Hardcodierte AuditTrail-Datei-/Tabellen-IDs statt Config:** In Agent 1 (Finding A), Agent 2 (Item 4) und Agent 3 nutzen die `WRITE AuditEvent`/`AUDIT_*`-Aktionen weiterhin SharePoint-interne Datei-/Tabellen-IDs statt zentraler Config-Werte. Bewusst zurückgestellt (kein akutes Risiko, da sich diese IDs praktisch nie ändern), aber technische Schuld.
- **Agent 2, Item 3 – Mailbox-Ordner-Setup-Optimierung:** 4 Aktionen (Ordner anlegen/IDs abrufen) laufen bei jeder einzelnen E-Mail neu, obwohl sich die Ordnerstruktur nach dem ersten Lauf nicht mehr ändert (~2-6 Sek. Laufzeit-Ersparnis möglich pro Mail). Gleiches Muster auch bei Agent 1. Abwägung (Stale-Cache-Risiko bei manueller Ordner-Umbenennung) im Archiv dokumentiert. Nicht umgesetzt, niedrige Priorität.
- **Tote Config-Variablen bereinigen:** Einige ungenutzte Einträge (u. a. `CounterFolder`, `CounterFileName`) in `DMP Command Configuration` sollten bei Gelegenheit identifiziert und entfernt werden.
- **Agent 3 – `WorkFileCleanupStillLocked`-Wartezeit:** Offene Detailfrage, ob die aktuell konfigurierte Wartezeit vor dem Cleanup-Retry ausreichend bemessen ist. Minor, kein bekannter Vorfall.
- **E-Mail-Importance-Konsistenz:** Bei jeder künftigen neuen E-Mail-Aktion (auch in Agent 1/2/3) prüfen, ob `emailMessage/Importance` bereits korrekt auf `MailImportanceInfo/Warning/Error` (Config) verweist statt hartkodiert `"Normal"`.
- **Agent 2 – irreführende Benennung `EmailsProcessed_DMP`/`EmailsProcessed_NoDMP` (gefunden + klargestellt 2026-09-22):** In `DMPAgent2E-MailInboxTreatmentVS-...json` klassifiziert die interne Variable `Detected Workflow Path` jede Mail in einen von vier Werten: `"No DMP"` (Standard-/Normalfall – NICHT, wie zunächst angenommen, ein Fallback für eine fehlende Referenzdatei, sondern der eigentliche produktive Regelfall, solange kein aktiver DMP-Fall läuft), `"DMP internal Sender"`, `"DMP not effected Sender"`, `"DMP effected Member"` (diese drei nur relevant, wenn tatsächlich ein DMP-Fall aktiv ist). Die beiden Zähler-Spalten in `DMP Command Agent Status` sind dazu vertauscht benannt: `item/EmailsProcessed_DMP` wird bei `"No DMP"` hochgezählt, `item/EmailsProcessed_NoDMP` bei `"DMP internal Sender"` (Zeilen ~9629-9630). Nutzer-Klarstellung 2026-09-22: der Name "DMP" für den Nicht-DMP-Standardfall ist irreführend; korrekter wäre `NDMP`/`NODMP` (passend zur bestehenden Konvention `PROD_NODMP`/`Value - PROD (NODMP)` an anderer Stelle in der App). **Nicht umgesetzt, bewusst zurückgestellt:** eine Korrektur würde SharePoint-Spaltenumbenennung (manueller Schritt, kein PnP-/API-Zugriff), Anpassung der Flow-JSON-Referenzen, Solution-Reimport und Prüfung aller lesenden Stellen (Dashboard/Ring) erfordern – hohes Fehlerpotential und Aufwand für eine rein kosmetische Korrektur. Nur als mögliche künftige Verbesserung vorgemerkt, nicht als Bug behandeln.

---

# ⚠️ Dokumentations-Lücken

## UAT_Playbook.docx veraltet

Pfad: `AI_Agent\UAT\UAT_Playbook.docx`, zuletzt inhaltlich für Agent 2 gepflegt (Stand Juni 2026). Deckt **nicht ab**:
- Agent 1, Agent 3, Agent 4, Agent 5, Agent 6 (existierten teils noch nicht)
- Die Agenten-Umnummerierung vom 2026-08-13 (3.01/3.02/3.03 → 3/4/5)
- Die 2 neuen Schalter (SIMU/PROD, Normal/DMP) und den Wegfall des `Yes.txt`-Mechanismus
- Die neue `Agent Audit Summary`/`Audit Acknowledgment`-Infrastruktur

**Status:** Bewusst zurückgestellt (größerer Aufwand), noch nicht angegangen. Sollte vor dem nächsten größeren UAT-Durchlauf aktualisiert werden — spätestens vor Umsetzung von DMP Command Streams relevant, da dort ganz neue Testfälle hinzukommen werden.

---

# 📎 Verweis: Vollständige Historie

Alle Root-Cause-Analysen, abgeschlossene Punkte, Architektur-Entscheidungen und der komplette bisherige Gesprächsverlauf (2026-08-06 bis 2026-09-04) stehen unverändert in [`DMP_COMMAND_Backlog_Archive.md`](./DMP_COMMAND_Backlog_Archive.md). Bei Bedarf dort per Volltextsuche nach Agent-Name, Datum oder Stichwort suchen.
