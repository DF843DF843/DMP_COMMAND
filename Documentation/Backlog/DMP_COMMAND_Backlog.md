# DMP COMMAND – Projektweites Backlog für größere, zurückgestellte Optimierungen

**Geltungsbereich:** Dieses Backlog gilt für das gesamte DMP COMMAND System — alle Agenten (Agent 1–6), die Power App (DMP COMMAND), sowie alle verwendeten Konfigurations- und Statuslisten/-dokumente (`DMP Command Configuration`, `DMP Command Agent Status`, u. a.).

**Pflege-Regel (in KI-Arbeitsregeln verankert am 2026-08-06):** Dieses Dokument wird regelmäßig aktualisiert, sobald bei der Arbeit an irgendeiner Komponente des Systems ein Finding entsteht, das bewusst auf später verschoben wird. Es ist NICHT auf einen einzelnen Agenten beschränkt.

**Archivierungs-Regel (ergänzt am 2026-09-04, siehe KI-Arbeitsregeln „Pflicht-Checkliste" Abschnitt G):** Vollständig erledigte, bestätigt live ausgelieferte Punkte werden regelmäßig aus diesem aktiven Dokument nach [`DMP_COMMAND_Backlog_Archive.md`](./DMP_COMMAND_Backlog_Archive.md) ausgelagert (nicht gelöscht). Der fachliche Kern jedes erledigten Punkts wird vorher in `DMP_COMMAND_Release_Notes.md` (versions-relevante Fixes/Features) bzw. `DMP_COMMAND_Operations_Manual.md` (betrieblich relevante Punkte) gesichert.

**Wichtiger Arbeitshinweis (gilt für jeden Punkt):** Vor Umsetzung IMMER zuerst den dann aktuellen Stand der jeweils betroffenen Datei(en) neu einlesen und gegen die hier beschriebenen Fundstellen prüfen (Feldnamen/Ausdrücke können sich durch zwischenzeitliche manuelle Änderungen verschoben haben). Keine neuen Config-Felder oder Variablen ohne Rücksprache mit dem Nutzer einführen.

---

## 📌 Große Aufräum-Aktion am 2026-09-04

Dieses Dokument war auf ca. 2650 Zeilen angewachsen (chronologisches Arbeitsprotokoll seit 2026-08-06) und wurde vor einer 2-wöchigen Nutzer-Abwesenheit komplett neu strukturiert:

- Die **gesamte bisherige Historie** (alle Root-Cause-Analysen, Architektur-Entscheidungen, abgeschlossene Punkte) wurde 1:1 unverändert nach [`DMP_COMMAND_Backlog_Archive.md`](./DMP_COMMAND_Backlog_Archive.md) übernommen (Volltext-durchsuchbares Nachschlagewerk).
- Dieses Dokument enthält ab jetzt **nur noch das, was tatsächlich noch offen ist**, sortiert nach Priorität/Bereich.
- Alle als "erledigt" erkannten Punkte sind bereits in `DMP_COMMAND_Release_Notes.md` (versionsbezogen) bzw. `DMP_COMMAND_Operations_Manual.md` §7/§8 (betrieblich) nachgezogen.
- Falls ein Punkt aus der alten Historie fehlt, den du noch für offen hältst: im Archiv per Volltextsuche (Strg+F) nach Stichwort suchen und hier wieder ergänzen.

---

# 🔴 Priorität 1 – Bereit zum Deploy, wartet auf grünes Licht des Nutzers

## 🔵 Agent 7 occurrence generation – activated in Dev, final C5 contract pending

**Verified Dev state (2026-09-22):** Solution `7.11.47` is imported and published only in
`DBG Team Productivity (Dev)` (Agent 7 workflow component `[0.2.0]`). Imported via `pac
solution import`; the user has **not yet** opened/saved/activated this specific version in
the designer, and no end-to-end run against real data has been performed yet.

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

**Hardening prepared in local source on 2026-09-22 (NOT yet packed/imported/deployed — still
`7.11.47` live in Dev):** the flow JSON in the Git working copy now additionally contains:
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

This is a source-only change (JSON-parses-clean and runAfter-graph-checked locally); it has
**not** been packed into a new solution version or imported into Dev yet. Per the confirmed
work order, this waits until the user confirms `7.11.47` save/activate in the designer (see
Session Restart Guide) — do not pack/import until that is confirmed, to avoid stacking an
unconfirmed base with new changes.

Still open:

1. ~~Replace/remove `OccurrencesJson` from the final production contract.~~ Done in `7.11.47`.
2. ~~Add `FromDate` and `ToDate` and inclusive multi-day generation.~~ Done in `7.11.47`.
3. ~~Rule-level `TimeOfDay`/`TimeZone` existence validation.~~ Prepared in local source
   2026-09-22 (see above); not yet packed/imported.
4. Resolve authoritative active-case/mode lookup and final mode values. Central config
   `CurrentOperationMode` (Global/Runtime) already carries
   `PROD_NODMP/PROD_DMP/SIMU_NODMP/SIMU_DMP/PROD_PREDEFAULT/PROD_POSTDEFAULT/SIMU_PREDEFAULT/SIMU_POSTDEFAULT`
   and is reused by other agents, but there is still no discovered central "active
   CaseId" source anywhere in the app/config. Per the confirmed work order, B1–B3 (Streams
   concept) are inserted next to build this; until then, `VALIDATE_CaseId` (above) only
   rejects a blank `CaseId`, it does not verify the case is real/active.
5. ~~Add trustworthy created/skipped/error counters.~~ Created/skipped done in `7.11.47`;
   the per-item error counter (`Scope`/try-catch around `CREATE_Occurrence`) is prepared in
   local source 2026-09-22 (see above); not yet packed/imported.
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
