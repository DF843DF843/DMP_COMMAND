# Konzept: "NEXT STEPS" Container → Working-Sub-Stream-Checklisten & Automatisierung

Status: 🔵 Teilumsetzung in Dev (C5 Agent 7 aktiviert in Solution 7.11.46 am 2026-09-21)
Zweck: Fachliches Gesamtkonzept und Referenz für die weiteren Ausbaustufen. Die Recurrence-/Occurrence-Listen, ein erster Agent-7-Flow und eine lokale Task-Occurrences-Appseite existieren bereits; die übrigen Funktionen sind weiterhin Konzept bzw. Backlog.

---

## 1. Ausgangslage (heutiger Stand, zur Erinnerung)

- "NEXT STEPS"-Kachel auf `scrHome` zeigt aktuell **nur lesend** 5 Meilensteine aus `Status DMP Process.xlsx` (Sheet "Overall Process"), geliefert von Agent 4 im Rahmen des normalen Status-Checks.
- Es gibt 2 Umschalter in der App: `varEnvironmentIsPROD` (PROD/SIMU) × `varOperationalModeIsDMP` (NoDMP/DMP) → 4 Kombinationen, die Agent 5 (`DMPAgent303OperationalStateManagement`) auslösen (Mail-Versand + Audit).
- 6 Agenten existieren bereits (Domain-Extraktion, Mail-Inbox-Behandlung, Emergency Report, Status Check, Operational State, Admin Functions) – alle nach demselben Muster: SharePoint-Konfigurationsliste, Audit-Trail, Versand über `default@eurex.com`.
- Bereits im Backlog als "Auslagerung in dedizierten Agenten" vorgemerkt (siehe Zeile ~1014 `DMP_COMMAND_Backlog.md`) – **dieses Konzept ist die konkrete Ausarbeitung davon.**

---

## 2. Fachliche Anforderungen aus dem Brainstorming (strukturiert)

### 2.1 Vier Working Sub Streams, davon 3 mit eigener Checkliste
- COS Leader, Infrastructure Team, Content Team → eigene Checkliste (Excel heute, künftig SharePoint-Liste)
- Hotline Team → aktuell keine eigene Checkliste genannt (offene Frage, siehe Abschnitt 7)
- Jede Checkliste referenziert per `Task ID` die gemeinsame Gesamtliste `Status DMP Process.xlsx` → wird zur **SharePoint-Liste "DMP Command Checklist Overall Process"**.

### 2.2 Fünf Modi statt vier
- Neu: **Pre-Default** und **Post-Default** als eigenständige Modi.
- Reihenfolge: `Normal → Pre-Default → DMP → Post-Default → Normal` (linearer Ablauf, kein freies Hin-und-Herschalten zwischen beliebigen Modi).
- Agent 2 (Mail-Behandlung) behandelt `Normal` / `Pre-Default` / `Post-Default` **gleich** (faktisch wie heutiges "NoDMP"). Nur `DMP` ist der Sonderfall.
- Damit ändert sich das bisherige 2×2-Toggle-Modell zu einem **5-Werte-Zustand** (kein einfaches Boolean-Paar mehr).

### 2.3 Popup bei Umschaltung auf DMP: "Default Case Context"
- Sobald von Normal/Pre-Default auf **DMP** umgeschaltet wird, muss ein autorisierter User ein Formular ausfüllen:
  - Termination Reason, Defaulted Clearing Member ID, Defaulted Clearing Member Name, Termination Date, Termination Time, (weitere Felder TBD).
- Diese Daten werden zu **Platzhaltern** für alle nachfolgenden E-Mail-Templates (ersetzen die bisher händisch gelb markierten Textstellen in den Word-Vorlagen).

### 2.4 E-Mail-Automatisierung je Checklisten-Task
- Spalte "E-Mail Template" in jeder Checkliste → wird zu einer strukturierten Aktion:
  - Word-Vorlagen werden in ein **parametrisierbares Format** überführt (analog zu allen bestehenden Agenten: Text-Bausteine mit Platzhaltern, gesteuert über die Configuration-Liste je Modus).
  - Gelb markierte, situationsabhängige Textstellen → **Popup-Formular** pro Versand, das der User vor dem Senden ausfüllt.
  - Versand **erst nach Bestätigung durch den User** (kein vollautomatischer "silent send").
  - Versand technisch über `default@eurex.com` (identisch zu Agent 1–6).
  - Empfänger = feste Sammel-E-Mail-Adressen pro Sub-Stream/Anlass (Konfigurationsliste, nicht hart codiert).
  - Jede gesendete Mail wird **archiviert** (analog "Sent Items"-Verschiebung/Kopie in dedizierten Ordner, wie Agent 1–3) und **im Audit Trail dokumentiert** (gleiches Schema wie bestehend: `AuditTrail.xlsx`/-Liste).

### 2.5 Status-Pflege mit 4-Augen-Prinzip
- Jeder Sub-Stream pflegt seinen Task-Status (Not Started/Ongoing/Done) über eine **eigene Sub-Seite** in der App, die im Hintergrund eine SharePoint-Liste beschreibt.
- **4-Augen-Prinzip:** Status-Änderung darf nur durch Mitglieder des jeweiligen Sub-Streams ODER CoS Lead/Deputy erfolgen – impliziert:
  - Eine Person trägt den Status ein (z. B. "Ongoing → Done"),
  - eine zweite (andere) Person bestätigt/genehmigt, bevor der Status final wirksam wird und ggf. Folgeaktionen (E-Mails, Freischaltung nächster Meilenstein) auslöst.
- **CoS Leader Master-Seite:** aggregierte Sicht auf den Status **aller** Sub-Streams, jederzeit einsehbar (nur Lesen für ihn, außer eigene CoS-Leader-Tasks).

### 2.6 Automatische Folge-Mails bei Meilenstein-Erreichung anderer Teams
- Wenn Sub-Stream A einen Meilenstein abschließt (Status = Done, ggf. nach 4-Augen-Bestätigung), können automatisch Mails an andere Stakeholder ausgelöst werden (abhängig von Verknüpfung in der Gesamtliste).
- Voraussetzung: alle Teams pflegen ihren Status **zeitnah und korrekt** – das 4-Augen-Prinzip dient auch der Datenqualität hierfür.
- Diese Trigger-Kette kann sich **mehrfach wiederholen** (z. B. bei erneutem Statuswechsel im Laufe eines DMP) – jeweils mit ggf. unterschiedlichem Inhalt/Empfängerkreis, nicht 1:1 wiederverwendbar → braucht pro Task eine **kleine Regel-Tabelle** (welcher Status-Übergang löst welche Mail an wen aus), keine hart codierte 1:1-Logik im Flow.

### 2.7 Rollen & Rechte ("Assign Roles")
- Bestehender Task "Assign Roles" bekommt in der App eine doppelte Bedeutung:
  1. Zugehörigkeit einer Person zu genau einem (oder mehreren) Working Sub Stream(s).
  2. **Screen-/Aktions-Berechtigung**: welche Bildschirme sieht die Person, welche Aktionen (Status setzen, bestätigen, Mail auslösen, Popup ausfüllen) darf sie bedienen.
- Braucht eine **Rollen-/Rechte-Tabelle** (SharePoint-Liste), keine rein hart codierte Sicherheitsgruppen-Logik in der App (zu unflexibel für 4-Augen-Prinzip pro Sub-Stream).

---

## 3. Vorgeschlagenes Datenmodell (neue SharePoint-Listen)

| Liste | Zweck | Kernspalten (Auszug) |
|---|---|---|
| **DMP Command Checklist Overall Process** | Ablösung von `Status DMP Process.xlsx` | Titel (=TaskID), Phase, TaskShortDescription, ResponsibleSubStream, Status, LastChangedUtc |
| **DMP Command Checklist CoS Leader** | Ablösung `CoSLeader Checklist.xlsx` | Titel (=TaskID, FK→Overall), Phase, SeqNo, TaskDescription, EmailTemplateId (FK), Status, LastChangedBy, ConfirmedBy, ConfirmedUtc |
| **DMP Command Checklist Infrastructure** *(zurückgestellt, siehe unten)* | Ablösung `Infrastructure Checklist.xlsx` | Titel (=TaskID, FK→Overall), SeqNo, Phase, TaskDescription, EmailTemplateId (FK), DelegatedTo, Status, LastChangedBy, ConfirmedBy, ConfirmedUtc |
| **DMP Command Checklist Content** *(zurückgestellt, siehe unten)* | Analog Content Team | analog zu oben |
| **DMP Command Checklist Email Templates** | Zentrale, parametrisierte Vorlagen (ersetzt Word-Dokumente als "Quelle der Wahrheit") | Titel (=TemplateId, Unique Key), TemplateName, SubjectTemplate, BodyTemplate (mit `{{Platzhaltern}}`), PlaceholderList (FK→Email Placeholders), RecipientGroupId (FK) |
| **DMP Command Checklist Email Placeholders** *(neu, Nutzer-Feedback 2026-09-03)* | Kontrollierter Katalog aller zulässigen `{{Platzhalter}}`-Schlüssel, statt Freitext je Vorlage | Titel (=PlaceholderKey, Unique Key), Description, ExampleValue |
| **DMP Command Checklist Recipient Groups** | Sammel-E-Mail-Adressen je Anlass | Titel (=GroupId, Unique Key), GroupName, EmailAddresses, SubStream |
| **DMP Command Checklist Role Assignments** | Wer gehört zu welchem Sub-Stream + welche Screens/Aktionen erlaubt | Titel, User, SubStream, IsSubStreamLead, IsCosLeadOrDeputy, AllowedScreens (FK→Screens Catalog), AllowedActions (FK→Actions Catalog) |
| **DMP Command Checklist Screens Catalog** *(neu, Nutzer-Feedback 2026-09-03)* | Kontrollierter Katalog aller App-Bildschirme, statt Freitext in `AllowedScreens` | Titel (=ScreenName, Unique Key), Description |
| **DMP Command Checklist Actions Catalog** *(neu, Nutzer-Feedback 2026-09-03)* | Kontrollierter Katalog aller Agent-7-Aktionen, statt Freitext in `AllowedActions` | Titel (=ActionName, Unique Key), Description |
| **DMP Command Checklist Default Case Context** | Ergebnis des Popups beim Umschalten auf DMP (ein aktiver Datensatz je Case) | Titel (=CaseId), TerminationReason, DefaultedMemberId, DefaultedMemberName, TerminationDateTime, SetByUser, SetUtc, ModeAtCreation |
| **DMP Command Checklist Status Change Approvals** | 4-Augen-Protokoll (wer hat vorgeschlagen, wer bestätigt) | Titel (=RequestId), ListName, ItemId, OldStatus, NewStatus, ProposedBy, ProposedUtc, ApprovedBy, ApprovedUtc, ApprovalState |
| **AuditTrail** (bestehend) | Wiederverwendung, kein neues Schema nötig | wie heute + `WorkflowPath` = z. B. "Agent7-Streams" |

**Namenskonvention – finale Klarstellung (Nutzer-Feedback 2026-09-03):** Der Präfix „NextSteps_" (und die zwischenzeitliche Variante „DMP Command Next Steps ") wurden als verwirrend zurückgemeldet, da „Next Steps" nur der ursprüngliche Arbeitstitel der bestehenden Home-Screen-Kachel war und nichts mit der eigentlichen fachlichen Funktion (Steuerung/Governance der 4 Working Sub Streams) zu tun hat. **Verbindlicher, finaler Präfix: „DMP Command Checklist "** (konsistent mit der bestehenden Liste „DMP Command Configuration"). Interne Feld-/Variablennamen (`TaskID`, `EmailTemplateId` usw.) bleiben unverändert – nur die SharePoint-Listentitel und der Agent-/Screen-Name ändern sich.

**Hotline Team:** Keine eigene Checkliste (siehe Entscheidung Abschnitt 7.1) – daher taucht "Hotline" hier bewusst nicht als eigene Liste auf.

**Infrastructure/Content Checklist zurückgestellt (Nutzer-Feedback 2026-09-03):** Der Nutzer hat noch keinen SharePoint-Zugriff auf die jeweiligen Team-Sites – diese beiden Listen werden erst angelegt, sobald der Zugriff vorhanden ist. Alle anderen Listen sind davon unabhängig und wurden bereits umgesetzt.

**Primärschlüssel-Korrektur (Nutzer-Feedback 2026-09-03):** In `Email Templates` und `Recipient Groups` ist jeweils die fachliche ID (`TemplateId` bzw. `GroupId`) der eindeutige Schlüssel der Liste – dafür wird das SharePoint-Pflichtfeld `Titel` verwendet (nicht wie ursprünglich für den sprechenden Anzeigenamen). Der sprechende Name wandert in eine eigene Spalte (`TemplateName`/`GroupName`).

**Wichtig:** Die 4-Augen-Genehmigung wird NICHT direkt im Statusfeld der Checkliste vollzogen, sondern über die Zwischentabelle `StatusChangeApprovals` — verhindert, dass ein einzelner User den Status direkt "durchschreibt".

---

## 4. Vorgeschlagene Architektur (neuer Agent + Screens)

### 4.1 Neuer Agent: "Agent 7 – Streams & Milestone Management"
Analog zu Agent 6 (Admin Functions) als reiner Dispatcher mit mehreren `RequestedAction`-Zweigen:
- `GetChecklist(subStream)` – liefert Zeilen für die jeweilige Sub-Seite
- `ProposeStatusChange(listItem, newStatus, user)` – schreibt Vorschlag in `StatusChangeApprovals`, löst KEINE Mail aus
- `ApproveStatusChange(requestId, approver)` – 4-Augen-Check (approver ≠ proposer, approver hat Berechtigung), schreibt finalen Status in Checkliste + Overall-Process, prüft Regel-Tabelle für Folge-Mails
- `SendChecklistEmail(templateId, placeholderValues, confirmedByUser)` – rendert Template, sendet über `default@eurex.com`, archiviert, schreibt Audit-Eintrag
- `SetDefaultCaseContext(formValues)` – wird beim Umschalten auf DMP aufgerufen, bevor Agent 5 die Mode-Mails verschickt
- `GetMasterStatus()` – aggregierte Sicht für CoS-Leader-Masterseite

Begründung für **einen neuen Agenten statt Erweiterung von Agent 4/5**: gleiches Argument wie im Backlog bereits vermerkt – Entkopplung vom Status-Heartbeat, eigene Audit-Spur, unabhängige Weiterentwicklung ohne Agent-4-Redeploy.

### 4.2 Anpassung bestehender Agenten
- **Agent 5 (Operational State Management):** Zustandsmodell von 2×2 Boolean auf **5-Werte-Enum** (`Normal/PreDefault/DMP/PostDefault`) umstellen; erzwingt lineare Übergangsreihenfolge (kein Sprung z. B. direkt Normal→DMP); ruft vor dem eigentlichen Modus-Wechsel-Mailversand ggf. Agent 7 `SetDefaultCaseContext` auf (nur beim Übergang **zu** DMP).
- **Agent 2 (Mail Inbox Treatment):** Modus-Zuordnung anpassen – überall wo bisher `NODMP` geprüft wird, künftig `Normal ODER Pre-Default ODER Post-Default` als gleichbehandelt interpretieren (Konfigurationsspalten ggf. um 2 weitere Werte-Spalten ergänzen oder Mapping-Tabelle nutzen statt harter 4-Spalten-Struktur).
- **Configuration-Liste (`DMP Command Configuration`):** Die bisherigen 4 Modus-Spalten (`PROD/NODMP`, `PROD/DMP`, `SIMU/NODMP`, `SIMU/DMP`) müssten um Pre-/Post-Default erweitert werden – das sprengt das bisherige "4-Spalten"-Schema. Empfehlung: neue Zusatzspalte/-liste statt 5×2=10 Spalten pro Parameter (siehe offene Frage 7.2).

### 4.3 Neue Screens (Power Apps)
1. `scrStreamsOverview` – Master-Statusseite für CoS Leader (aggregiert alle Sub-Streams, nur Lesen fremder Streams)
2. `scrChecklistCosLeader`, `scrChecklistInfrastructure`, `scrChecklistContent` – je Sub-Stream (kein Hotline-Screen, siehe Entscheidung 7.1), gefiltert nach `DMP Command Checklist Role Assignments`
3. `popDefaultCaseContext` – Formular-Popup, erscheint beim Umschalten Normal/Pre-Default → DMP
4. `popEmailPlaceholderInput` – strukturiertes Formular für die "gelb markierten" situativen Textteile vor Mail-Versand
5. `popStatusApproval` – 4-Augen-Bestätigungsdialog (zeigt Vorschlag, Bestätigen/Ablehnen)
6. `scrEmailTemplateEditor` – Pflege-Screen für E-Mail-Vorlagen: `RichTextEditor`-Control + Platzhalter-Galerie zum Einfügen von `{{Platzhaltern}}` an der Cursorposition, speichert in `DMP Command Checklist Email Templates.BodyTemplate`; nur für Nutzer mit `AllowedActions` = `EditEmailTemplates`
7. Erweiterung Sidebar-Navigation um die neuen Screens, sichtbar nur je nach `AllowedScreens` aus Rollenzuweisung

---

## 5. Phasenplan (Vorschlag, iterativ, mit Nutzer-Feedback nach jeder Phase)

**Phase 1 – Datenbasis (dein genannter Startpunkt):**
1. SharePoint-Liste `DMP Command Checklist Overall Process` anlegen, Inhalte aus `Status DMP Process.xlsx` migrieren
2. SharePoint-Liste `DMP Command Checklist CoS Leader` anlegen, Inhalte aus `CoSLeader Checklist.xlsx` migrieren, TaskID-Verknüpfung zu Overall Process herstellen
3. Nur Lesen in der App (Ersatz der heutigen NEXT-STEPS-Kachel durch echte Liste, noch ohne Schreiben/Mails)

**Phase 2 – Sub-Seite CoS Leader mit Status-Schreiben + 4-Augen-Prinzip**
- `DMP Command Checklist Role Assignments` + `DMP Command Checklist Status Change Approvals` einführen
- Agent 7 (Basisversion): `GetChecklist`, `ProposeStatusChange`, `ApproveStatusChange`
- `scrChecklistCosLeader` + `popStatusApproval`

**Phase 3 – E-Mail-Automatisierung für CoS-Leader-Checkliste**
- 1. Word-Template als Pilot in `DMP Command Checklist Email Templates` überführen
- `popEmailPlaceholderInput`, `SendChecklistEmail`, Archivierung + Audit

**Phase 4 – 5-Modus-Umstellung (Pre-/Post-Default) inkl. Popup `Default Case Context`**
- Bewusst nach Phase 1–3, da hiervon Agent 2/5 UND Configuration-Schema betroffen sind (höheres Risiko, siehe Backlog-Erfahrung mit Toggle-Feedback-Loop-Bugs)

**Phase 5 – Rollout auf Infrastructure Team, Content Team (und ggf. Hotline)**
- Wiederverwendung des in Phase 1–3 geschaffenen generischen Musters (Liste + Screen + Agent-7-Aktionen), nur Konfigurationsdaten unterscheiden sich

**Phase 6 – Automatische Cross-Team-Folge-Mails bei Meilenstein-Erreichung**
- Regel-Tabelle "welcher Status-Übergang löst welche Mail bei welchem anderen Team aus" – erst sinnvoll, wenn alle Sub-Streams ihre Checkliste digital pflegen (Phase 5 abgeschlossen)

---

## 6. Wichtige Lehren aus dem bestehenden Projekt, die wir hier direkt vermeiden

- **Toggle-Feedback-Loop-Bug** (Backlog 2026-08-25): Beim 5-Werte-Zustand für Operating State muss von Anfang an ein Guard-Flag-Muster (`varSuppressToggleEvents`) korrekt integriert werden – nicht wie beim ersten Mal nachträglich reparieren.
- **Zu viele API-Aufrufe verlangsamen Flows** (Agent 2 Performance-Erkenntnis): Agent 7 sollte Statusabfragen bündeln (z. B. eine `GetItems`-Filterabfrage statt Schleifen mit vielen `GetItem`-Aufrufen).
- **YAML-Doppelpunkt-Bug** (kritischer Studio-Parser-Fehler): Bei allen neuen Text-Formeln in `.pa.yaml` auf `": "`-Sequenzen in unquotierten Strings achten.
- **`runAfter`-Lücken bei Alert-Mails** (mehrfach in Agent 3/4/5 gefunden): Bei Agent 7 von Anfang an alle Mail-/Audit-Aktionen mit vollständigem `runAfter` (`Succeeded/Failed/Skipped/TimedOut`) verketten.

---

## 7. Entscheidungen (Rückmeldung Nutzer, 2026-09-02)

1. **Hotline Team:** ✅ Bewusst **keine** eigene Checkliste/Templates. Bleibt außen vor bei `DMP Command Checklist *Checklist`-Listen und Mail-Automatisierung.
2. **Configuration-Schema für 5 Modi:** ✅ Bestehende 4 Modus-Spalten in `DMP Command Configuration` werden um **2 weitere Spalten** ergänzt → 6 Werte-Spalten total (`PROD-PreDefault`, `PROD-DMP`, `PROD-PostDefault`, `SIMU-PreDefault`, `SIMU-DMP`, `SIMU-PostDefault` o. ä., "Normal" entspricht dem bisherigen `NODMP`-Wert). Betrifft nur Parameter, die Agent 5 nutzt.
3. **Template-Pflege:** ✅ **Neuer dedizierter Pflege-Screen** `scrEmailTemplateEditor` statt Word-Datei oder reinem HTML-Freitext:
   - Power Apps `RichTextEditor`-Control (Word-ähnliches Editier-Erlebnis: fett, Absätze, Listen).
   - Daneben eine **Platzhalter-Auswahlliste** (Dropdown/Galerie mit z. B. `{{TerminationReason}}`, `{{DefaultedMemberName}}`, `{{TerminationDate}}` …), Klick fügt Platzhalter an Cursorposition im Editor ein.
   - Gespeichert wird der resultierende HTML/Rich-Text in `DMP Command Checklist Email Templates.BodyTemplate` – die Liste bleibt technische "Quelle der Wahrheit", der Screen ist nur die komfortable Editier-Oberfläche dafür (kein Word-Datei-Import/Export nötig).
   - Nur Nutzer mit passender Berechtigung (`AllowedActions` enthält z. B. `EditEmailTemplates`) sehen/nutzen diesen Screen.
4. **4-Augen-Prinzip:** ✅ Streng: Zweitfreigeber muss (a) eine **andere Person** als der Ersteller sein **und** (b) **Mitglied desselben Sub-Streams** (oder dessen Lead) sein. Wird in `ApproveStatusChange` (Agent 7) hart geprüft (`ApprovedBy ≠ ProposedBy` UND `ApprovedBy.SubStream == item.SubStream`), sonst Ablehnung mit Fehlermeldung.
5. **Migration:** ✅ Ab Phase 1 ist die neue SharePoint-Liste **sofort exklusiv** die Quelle der Wahrheit – kein Parallelbetrieb mit den Excel-Dateien. Die 3 Excel-Checklisten werden einmalig migriert und anschließend als "eingefroren/nur Archiv" markiert (z. B. in SharePoint verschieben/schreibschützen), damit niemand versehentlich dort weiterpflegt.
6. **Reihenfolge:** ✅ **Parallel starten** – CoS-Leader-Checkliste (Phase 1-3) UND 5-Modus-Umstellung (Phase 4) werden als zwei unabhängige Teilstränge gleichzeitig vorangetrieben (siehe Abschnitt 8, angepasster Phasenplan).

---

## 8. Angepasster Phasenplan (nach Entscheidung: 2 parallele Teilstränge)

Da beide Stränge unterschiedliche Teile der Lösung berühren (Strang A: neue Listen/Screens/Agent 7 rein additiv; Strang B: bestehende Agenten 2/5 + Configuration-Liste), ist Parallelarbeit risikoarm möglich – sie teilen sich nur die neue Liste `DMP Command Checklist Default Case Context` und das Popup `popDefaultCaseContext` als Schnittstelle.

### Strang A – CoS-Leader-Checkliste & Next-Steps-Grundgerüst
- **A1:** `DMP Command Checklist Overall Process` + `DMP Command Checklist CoS Leader` anlegen, Excel-Inhalte migrieren, Excel-Dateien anschließend schreibschützen/archivieren. "NEXT STEPS"-Kachel auf neue Liste umstellen (rein lesend, Ersatz von Agent 4 Overall-Process-Teil).
- **A2:** `DMP Command Checklist Role Assignments` + `DMP Command Checklist Status Change Approvals` anlegen. Agent 7 Basisversion (`GetChecklist`, `ProposeStatusChange`, `ApproveStatusChange` mit strengem 4-Augen-Check). Neue Screens `scrChecklistCosLeader` + `popStatusApproval`.
- **A3:** `DMP Command Checklist Email Templates` + `DMP Command Checklist Recipient Groups` anlegen. Neuer Screen `scrEmailTemplateEditor` (RichTextEditor + Platzhalter-Galerie). Agent-7-Aktion `SendChecklistEmail` inkl. Archivierung + Audit, Popup `popEmailPlaceholderInput` für situative Texteingaben.
- **A4:** `scrStreamsOverview` (CoS-Leader-Masterseite, aggregiert alle Sub-Streams).
- **A5:** Rollout des in A1-A3 geschaffenen generischen Musters auf Infrastructure Team und Content Team (neue Listen + Screens, gleiche Agent-7-Aktionen wiederverwendet).
- **A6:** Cross-Team-Regeltabelle für automatische Folge-Mails bei Meilenstein-Erreichung (erst sinnvoll nach A5).

### Strang B – 5-Modus-Umstellung (Pre-/Post-Default)
- **B1:** `DMP Command Configuration`-Liste um 2 Werte-Spalten erweitern (6 Spalten total für betroffene Agent-5-Parameter); Konfigurationswerte für Pre-/Post-Default befüllen (i. d. R. identisch zu "Normal"/NODMP).
- **B2:** Zustandsmodell in der App von 2×2-Boolean auf **5-Werte-Enum** umstellen (`varOperationalMode`: Normal/PreDefault/DMP/PostDefault), inkl. sauberem Guard-Flag-Muster von Anfang an (Lehre aus Backlog, siehe Abschnitt 6), UI erzwingt lineare Übergangsreihenfolge (kein Direktsprung Normal→DMP o. ä.).
- **B3:** `DMP Command Checklist Default Case Context`-Liste + Popup `popDefaultCaseContext` – erscheint beim Übergang **zu** DMP, blockiert den Wechsel bis Pflichtfelder ausgefüllt sind.
- **B4:** Agent 5 anpassen: bei Übergang zu DMP zunächst Agent 7 `SetDefaultCaseContext` aufrufen (Werte aus Popup), danach bestehende Mode-Mail-Logik. Bei Übergang zu Pre-/Post-Default: gleiche Mail-Behandlung wie bisher "Normal".
- **B5:** Agent 2 Mapping anpassen: überall wo bisher `NODMP` geprüft wurde, künftig `Normal ODER Pre-Default ODER Post-Default` als eine Gruppe behandeln (z. B. über eine kleine Lookup-Function/Konfigurationsspalte `EffectiveMode`, die die 3 Werte auf "NODMP-Verhalten" mappt) statt Code-Verzweigungen zu verdreifachen.

**Reihenfolge innerhalb der Stränge ist verbindlich** (A1 vor A2 vor A3 …), **zwischen den Strängen A und B gibt es keine Abhängigkeit** außer der gemeinsamen Nutzung von `DMP Command Checklist Default Case Context`/Agent 7 in B3/B4 (baut auf A2, da Agent 7 dort erstmals entsteht).

### Strang C – Wiederkehrende Aufgaben und Tagesausführungen
- **C1:** Die CoS-Leader-Definitionen für Task 21–24 als wiederkehrende Aufgaben markieren; keine täglichen Statuswerte in den Definitionszeilen speichern.
- **C2:** `DMP Command Checklist Recurrence Rules` mit einer aktiven Daily-Regel je Task und ScheduleSlot anlegen. Uhrzeit und Zeitzone werden konfiguriert, nicht im Flow hart codiert.
- **C3:** `DMP Command Checklist Task Occurrences` mit den dokumentierten Spalten anlegen. `OccurrenceId` ist der unveränderliche Geschäftsschlüssel im Format `CaseId-Date(TaskDate in TimeZone)-TaskId-Upper(ScheduleSlot)`.
- **C4:** `DMP Command Checklist Status Change Approvals` um `OccurrenceId` ergänzen. Jede Statusfreigabe bezieht sich auf eine konkrete Tagesausführung.
- **C5:** Agent 7 um `CreateTaskOccurrences` und `GenerateMissingOccurrences` erweitern. Vor jeder Anlage erfolgt eine exakte Suche nach `OccurrenceId`; vorhandene Zeilen werden niemals aktualisiert oder überschrieben. Nur aktive Daily-Regeln im DMP-Modus sind zulässig. Post-Default beendet die Erzeugung neuer Ausführungen.
- **C6:** Agent-7-Aktionen für `GetOpenTaskOccurrences`, `ProposeOccurrenceStatusChange`, `ApproveOccurrenceStatusChange` und `CompleteTaskOccurrence` ergänzen.
- **C7:** Power-App-Tagesansicht für offene, erledigte und überfällige Occurrences ergänzen; die Masteransicht aggregiert nur die aktuellen Ausführungen, die Historie bleibt separat filterbar.
- **C8:** Übergänge testen: DMP-Start erzeugt die Ausführungen, erneuter Scheduler-Lauf erzeugt keine Duplikate, Post-Default erzeugt keine neuen Daily-Ausführungen, bestehende Historie bleibt lesbar.

---

## 9. Nächster Schritt

Start mit **A1** (Listen + Migration + Kachel-Umstellung) und **B1** (Configuration-Liste erweitern) – beide unabhängig voneinander und risikoarm (rein additiv bzw. reine Konfigurationsdaten-Ergänzung ohne Flow-Logikänderung). Sag Bescheid, ob ich direkt mit A1 (SharePoint-Listen anlegen + Excel-Migration) beginnen soll, oder ob du das Anlegen der SharePoint-Listen selbst übernimmst und ich dir dabei nur die exakten Spalten-/Datentyp-Definitionen liefere (da ich keinen direkten SharePoint-Zugriff aus dieser Umgebung habe).
