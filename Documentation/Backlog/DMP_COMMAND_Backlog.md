# DMP COMMAND – Projektweites Backlog für größere, zurückgestellte Optimierungen

**Geltungsbereich:** Dieses Backlog gilt für das gesamte DMP COMMAND System — alle Agenten (Agent 1, Agent 2, Agent 3, Agent 4, Agent 5 — durchnummeriert am 2026-08-13, vormals Agent 3.01/3.02/3.03), die Power App (DMP COMMAND), sowie alle verwendeten Konfigurations- und Statuslisten/-dokumente (`DMP Command Configuration`, `DMP Command Agent Status`, u. a.).

**Pflege-Regel (in KI-Arbeitsregeln verankert am 2026-08-06):**
Dieses Dokument wird regelmäßig aktualisiert, sobald bei der Arbeit an irgendeiner Komponente des Systems ein Finding entsteht, das bewusst auf später verschoben wird. Es ist NICHT auf einen einzelnen Agenten beschränkt.

**Wichtiger Arbeitshinweis (gilt für jeden Punkt):**
Vor Umsetzung IMMER zuerst den dann aktuellen Stand der jeweils betroffenen Datei(en) neu einlesen und gegen die hier beschriebenen Fundstellen prüfen (Feldnamen/Ausdrücke können sich durch zwischenzeitliche manuelle Änderungen verschoben haben). Keine neuen Config-Felder oder Variablen ohne Rücksprache mit dem Nutzer einführen.

---

# 🔴 STATUS-ÜBERSICHT (2026-09-01, Sitzungsende): Alle offenen Punkte

**Kontext:** Nutzer beendet die Sitzung ("dann mache ich jetzt Schluss") – dies ist die finale, gesammelte Übersicht aller zu diesem Zeitpunkt offenen Punkte, damit nichts verloren geht.

**✅ Heute deployed (live) UND vom Nutzer final bestätigt (Stand 18:36 Uhr):**
- Agent 6 (Admin Functions) – Counter-Reset (alle 5 Fälle) auf die SharePoint-Liste `DMP Command Counters` (GUID `277307f0-195a-4e78-afc8-850dbdf956b2`) umgestellt, per `pac solution import` live importiert und vom Nutzer reaktiviert.
- App **v1.22.3** (kumulativ inkl. v1.22.1/v1.22.2) – kanonische `.msapp`-Datei vom Nutzer in Power Apps Studio geöffnet, gespeichert und veröffentlicht. **Live.** Enthält:
  - v1.22.1: Header/Next-Steps-Randausrichtung, Timer-Split-Fix, KPI-Verschiebung.
  - v1.22.2: Replace-Button-Zeilenumbruch-Fix, Domains-Buttons nach links verschoben.
  - v1.22.3: "AGENTS ACTIVE"- und Files-Panel-Namen (Emergency Report/External Domains/Counter File/Audit Trail File) waren durch die Arial-Migration abgeschnitten – Schriftgröße reduziert, behoben. Audit Trail (Detail) zeigte veraltete Recent-Critical/Warning-Zeilen, da der periodische Refresh-Timer nur auf dem Cockpit-Screen läuft – Audit-Trail-Screen holt sich jetzt bei jedem Öffnen selbst frische Daten (`OnVisible`).
- Nutzer bestätigt zusätzlich: alle Agenten und SharePoint-Listen in der App aktualisiert/verbunden und veröffentlicht.
- Neues Dokument `DMP_COMMAND_AI_Collaboration_Best_Practices.md` erstellt (Englisch) – beschreibt unsere Zusammenarbeitsmethodik (KI-Arbeitsregeln, Best Practices) INKLUSIVE der konkreten technischen Praktiken (SharePoint-Variablen/Konfiguration, Audit Trail, Warn-Mails, Release Notes, Backlog, GitHub-Nutzung) als eigener Abschnitt – für eine andere KI zur PowerPoint-Erstellung.

**🟡 Noch offen / nächste Schritte (in Prioritätsreihenfolge, für die nächste Sitzung):**
1. **Agent 1 SharePoint-Migration** (External-Domains-Schreiben, Full-Sync-Pattern: erst alle Zeilen löschen, dann neu anlegen) – noch NICHT begonnen, ist jetzt der nächste Schritt.
2. **Aufräumen (klein, risikolos):** In Agent 6 werden die alten Excel-Konfigurationswerte `CounterFolder`, `CounterFileName`, `CounterTableName`, `CounterTableColumnNamePath`, `CounterTableColumnNameCounter` (in der Liste `DMP Command Configuration`) nicht mehr referenziert. Können bei Gelegenheit entfernt werden, keine Funktionsauswirkung.
3. **Unbeantwortete Agent-3-Frage:** Ob eine Verlängerung der Wartezeit vor dem `HTTP_Recycle_WorkFile_CurrentRun`-Retry (z. B. auf 5 Minuten) das wiederkehrende `EMREPORT-002 WorkFileCleanupStillLocked`-Warning beheben würde – der relevante Code (`SCOPE_WorkFileCleanup_CurrentRun` in Agent 3) wurde lokalisiert, aber die konkrete Wartezeit-Analyse steht noch aus.
4. **Standalone-Anleitungsdatei** `ANLEITUNG_Neue_SharePoint_Listen.md` für die beiden neuen SharePoint-Listen – nicht angelegt (Listen sind bereits vom Nutzer erstellt, daher niedrige Priorität, ggf. nicht mehr nötig).
5. **Audit Trail SharePoint-Migration** – weiterhin bewusst zurückgestellt, separates Projekt (siehe Eintrag weiter oben in diesem Dokument).

---

## ✅ Update (2026-09-02): Agent 2 SharePoint-Migration fertig gebaut (git-only, noch nicht deployt)

**Agent 2 (E-Mail Inbox Treatment) komplett auf SharePoint umgestellt:**
- **Counter-Inkrement** (alle 4 Fälle: No DMP, DMP internal Sender, DMP effected Member, DMP not effected Sender): `Get_Last_<X>_ID` liest jetzt per `GetItems`+`$filter: "Title eq '<Detected Workflow Path>'"` aus der Liste `DMP Command Counters` (GUID `277307f0-195a-4e78-afc8-850dbdf956b2`), `Update_Last_<X>_ID` schreibt per `PatchItem` mit `id: first(body('Get_Last_<X>_ID')?['value'])?['ID']` und `item/NumberProcessedEmails: variables('Current Workflow Counter')`. Klassischer Lese-vor-Schreiben-Zyklus, wie im Backlog vorgesehen.
- **External-Domains-Lesen:** Der alte Top-Level-Gate-Check `Check_Existence_of_External_Domains_File` (vorher `GetFileItems` auf eine Datei in einer SharePoint-Dokumentbibliothek) liest jetzt per `GetItems` alle Zeilen der Liste `DMP Command External Domains` (GUID `dc89aed5-d87a-4e12-875a-db2adbc2cee4`). Die tiefer verschachtelte Domain-Abgleichslogik (`Load_External_Domains` + `Parse_external_domains_file_+_create_array`) wurde vereinfacht: Der separate Datei-Lese-Schritt (`Load_External_Domains`, `GetFileContentByPath`) entfällt komplett, `Parse_external_domains_file_+_create_array` verwendet jetzt direkt die bereits beim Flow-Start geladenen Listenzeilen (`select(body('Check_Existence_of_External_Domains_File')?['value'], toLower(trim(string(item()?['Title']))))`) – exakt das gleiche, bereits bewährte Muster wie beim früheren Internal-Domains-Umbau.
- **Vorab-Verfügbarkeitsprüfung** `Check_Workflow_Counter_File` (löste bei fehlender Excel-Datei einen Alarm+Terminate aus) liest jetzt per `GetItems` die Liste `DMP Command Counters` an, um weiterhin zu erkennen, wenn die Liste selbst nicht erreichbar ist.
- **Alle Alarm-E-Mail-Texte** (Counter-Update fehlgeschlagen ×4, Counter-Liste nicht erreichbar) wurden inhaltlich aktualisiert, um "DMP Command Counters SharePoint list" statt der alten Excel-Dateipfad-Beschreibung zu nennen – keine irreführenden Texte mehr für das Hotline-Team.
- **Nicht angefasst (bewusst außerhalb des Scopes):** Der Audit-Trail-Schreibzugriff (`shared_excelonlinebusiness`, Tabelle `AgentAuditSummary` u. a.) bleibt unverändert – das ist die separate, zurückgestellte Audit-Trail-Migration.
- Workflow-Version in `DMPAgent2E-MailInboxTreatmentVS-....json.data.xml` von `[1.0.7]` auf `[1.0.8]` erhöht.
- **Validiert:** JSON-Syntax gültig, keine verwaisten `runAfter`-Referenzen (Vergleich mit dem Originalstand vor der Änderung, identische Restmenge an regex-bedingten Fehlalarmen bei escaped Anführungszeichen), keine Beschreibung >255 Zeichen, `pac solution pack` erfolgreich (zweimal, vor und nach der Versionsnummer-Änderung).
- **Status:** Nur ins Git-Repository committet, **NICHT live importiert** (`pac solution import`) – wartet auf Nutzer-Freigabe zum Testen, da Agent 2 den produktiven E-Mail-Pfad verarbeitet.

---
7. **Ungeklärt:** Ob es zwei separate "DMP Command Counters"-Listen gibt (Teams-Tab zeigte "(2)" im Namen) – aus den `DataSources.json`-Daten der App gibt es nur EINEN verbundenen Eintrag, daher vermutlich nur ein Tab-Namensartefakt, aber vom Nutzer nie explizit bestätigt. Bei Gelegenheit im Teams-Kanal kurz gegenchecken.
8. **Backlog-Punkt weiter unten:** User-seitiges Zurücksetzen der Critical/Warnings-Zähler – noch offene Architekturfrage (client-lokal vs. serverseitig), siehe Detail-Eintrag direkt im Anschluss an diese Übersicht.

---


**Nutzerwunsch:** "CRITICAL und WARNINGS counter sollten durch den User 'zurückgesetzt' werden können, damit da nicht die ganze Zeit eine Zahl >0 steht!"

**Kontext:** Die Cockpit-KPI-Kacheln "Critical" und "Warnings" zeigen `varAuditFailedCount`/`varAuditWarningCount`, die aus Agent 4s Statusabfrage kommen und die Gesamtzahl aller bisher aufgetretenen Fehler/Warnungen im Audit Trail widerspiegeln (nicht nur "seit letztem Reset"). Es existiert bereits ein separater, ähnlicher Mechanismus für die "Change-Notification"-LEDs (`varLastAckCriticalCount`/`varLastAckWarningCount`, per Klick auf die Kachel bestätigt) – das ist aber nur ein rein clientseitiger "gesehen"-Marker, der die angezeigte ZAHL selbst nicht auf 0 zurücksetzt.

**Zu klären vor Umsetzung (Architekturfrage, braucht Rücksprache):**
1. Soll der Reset nur die App-seitige Anzeige beeinflussen (client-lokal, verschwindet bei App-Neustart wieder), oder soll er serverseitig im Audit Trail vermerkt werden (ähnlich dem bereits existierenden "Reset e-mail counters"-Muster in Admin Functions, das den alten Wert vor dem Reset im Audit Trail protokolliert)?
2. Soll das Reset nur die ANZEIGE beeinflussen (Baseline hochsetzen, ähnlich `varLastAckCriticalCount`) oder tatsächlich die zugrunde liegenden Audit-Trail-Fehlerzähler auf Agent-4-Seite zurücksetzen?
3. Wer darf zurücksetzen (jeder App-Nutzer oder nur Admin-Funktionen-Bereich)?

**Empfehlung:** Vermutlich am saubersten als neue Admin-Functions-Aktion analog zu "Reset e-mail counters" (mit Audit-Trail-Protokollierung des alten Werts vor dem Reset), NICHT als reiner Klick auf die Kachel selbst (das würde das bestehende Muster der Change-Notification-LED verwässern, die ja gerade "neue Warnung nach dem letzten Blick" anzeigen soll).

---



**Kontext:** Nutzer war für ~3 Tage abwesend und hatte 4 Aufträge im "Autopilot-Modus" erteilt. Diese Zusammenfassung dokumentiert, was erledigt wurde und was noch offen ist.

## ✅ Erledigt und bereits gepusht (Git, Branch `main`)
1. **App-Layout-Feinschliff + Reserved-Bereich-Redesign + 3 SharePoint-Anbindungen** – Commit `3633a69` (v1.10.0). Details siehe Eintrag weiter unten.
2. **Agent 2 + Agent 4 auf neue Internal-Domains-SharePoint-Liste umgestellt** – Commit `a4b131f`. Details siehe eigener Eintrag weiter unten.
3. **UAT_Playbook.docx aktualisiert** – 3 Testszenarien (3.5, DEE-Szenario, 3.13) verweisen jetzt auf die SharePoint-Liste statt auf `Internal_Domains.txt`. Backup liegt als `UAT_Playbook.docx.bak` im selben Ordner (`AI_Agent\UAT\`) – kann gelöscht werden, sobald das Dokument in Word erfolgreich geöffnet wurde und alles passt.

## ⏳ Noch OFFEN – benötigt eine Entscheidung/Aktion von dir

1. **App veröffentlichen (Studio Save + Publish):** Ich kann `.msapp` packen und nach Git pushen, aber das eigentliche Veröffentlichen in Power Apps Studio (Speichern + Publish) ist ein manueller Schritt – Browser-Zugriff wurde in dieser Sitzung getestet und ist durch die Sandbox-Umgebung blockiert (Sicherheitsgrenze, kein Zugriffsproblem deinerseits). **Bitte die neu gepackte App in Studio öffnen und veröffentlichen.**

2. **Agent 2 + Agent 4 Live-Deployment (`pac solution import`):** Die Flow-Änderungen sind nur im Git-Repository, NICHT in der Live-Umgebung aktiv. Bewusst zurückgehalten, da Agent 2 produktiv eingehende E-Mails klassifiziert – eine Live-Aktivierung ohne begleitetes Testen erschien zu riskant. **Bitte review/freigeben, dann führe ich `pac solution pack` + `pac solution import` aus** (oder du machst es selbst über den Power Platform Admin Center / Solution-Import).

3. **Nach Agent-2-Import: Testlauf empfohlen** – je eine Test-Mail von einer bekannten internen Domain (z. B. eurex.com) und einer bekannten externen Domain durchlaufen lassen, um die neue SharePoint-basierte Klassifizierung zu verifizieren, bevor man sich vollständig darauf verlässt.

4. **Generische "Nächste-Schritte"-Checkliste entfernt:** Beim Automation-Status-Redesign wurden die alten manuellen Checklisten-Punkte (Check Agent 2 inbox / Validate Emergency Report / Audit Trail review / End Fire Drill / Export weekly report) aus Platzgründen ersatzlos entfernt. Inhalt bleibt in der Git-Historie (Commit vor `3633a69`) erhalten. **Entscheidung offen:** Sollen diese irgendwo wieder auftauchen (z. B. eigener neuer Bereich), oder war das ohnehin veralteter Inhalt?

5. **Operational Manuals – nur teilweise bearbeitet:** `DMP_Multi_Agent_Workflow_Documentation.docx` und alle Agent-xxx-Workflow-HTML-Dateien liegen in `Documentation\ARCHIVE\` – also bereits bewusst archiviert/nicht mehr aktuell. Diese wurden NICHT angefasst, um keine toten Dokumente wiederzubeleben. **Bitte bestätigen:** Sollen diese Dokumente reaktiviert und auf den aktuellen Stand gebracht werden, oder bleibt es bei „archiviert, `UAT_Playbook.docx` ist das einzige aktive Manual"?

6. **Aufräumarbeiten (unkritisch, jederzeit möglich):**
   - `UAT_Playbook.docx.bak` (Backup-Datei) kann nach Bestätigung gelöscht werden.
   - Ungenutzte Konfigurationswerte `InternalDomainsFileName`/`InternalDomainsStorageFolder` in `DMP Command Configuration` werden von Agent 2/4 nicht mehr referenziert, aber bewusst nicht entfernt (geringes Risiko, kein Zeitdruck).
   - Alte `Internal_Domains.txt` bleibt bestehen, bis die SharePoint-Liste sich im Praxisbetrieb bewährt hat.

7. **Unverändert offene, ältere Backlog-Punkte** (aus früheren Sitzungen, nicht Teil dieser Autopilot-Runde):
   - Agent 2 Item 3 (Mailbox-Ordner-ID-Caching) – pausiert, wartet auf GUI-Entscheidung.
   - Agent-Audit-Datei/Tabellen-Hardcoding (Item 4) – bewusst zurückgestellt (Risiko > Nutzen).
   - Größere Container-für-Container-Design-/UX-Überarbeitung des Cockpits – Umfang/Reihenfolge noch mit Nutzer zu klären.

---

## 📋 NEU (2026-09-01): Backlog-Plan – Umstellung der Datei-basierten Speicher auf SharePoint-Listen

**Anlass:** Nutzerfrage nach dem erfolgreichen Migrations-Vorbild "Internal Domains" (Text-Datei → SharePoint-Liste, diese Sitzung): "Würde es Sinn machen, auch alle anderen Dateien auf SharePoint-Listen umzustellen? Würde das den Flow robuster und schneller machen?"

**Grundsätzliche Bewertung:** Ja, SharePoint-Listen sind für die App-Anbindung und für Schreibrobustheit grundsätzlich besser geeignet als Excel-Online-Business-Tabellen:
- Kein Datei-Lock-Risiko beim gleichzeitigen Schreiben mehrerer Agenten (Excel Online Business sperrt die Datei kurzzeitig – SharePoint-Listen schreiben pro Zeile unabhängig).
- Native, delegierbare `Filter()`/`CountRows()`/`Patch()`-Unterstützung in Power Fx – kein Get-Item-by-ID-Umweg wie bei Excel-Tabellen nötig.
- Die App hat bereits 3 SharePoint-Verbindungen etabliert (Configuration, Agent Status, Internal Domains) – jede weitere Liste nutzt dieselbe, bereits bewährte Anbindung.

**Aber differenziert je Datei – nicht alles ist ein guter Kandidat:**

| Datei/Tabelle | Empfehlung | Priorität | Begründung |
|---|---|---|---|
| `Counter.xlsx` (Tabelle `DMP_Email_Counter`, 4 Zeilen) | ✅ Migrieren | Hoch (klein, geringes Risiko) | Exakt gleiches Muster wie Internal Domains: wenige Zeilen, einfacher Key-Value-Aufbau (`Workflow`/`Number_Processed_Emails`). Macht auch den bereits gebauten Counter-Reset (Admin Functions) einfacher (`Patch()` statt GetItem/PatchItem-Excel-Dance). Betroffen: Agent 2 (schreibt), Agent 6 (resettet, seit v1.2.0). |
| `External_Domains.txt` | ✅ Migrieren | Hoch | Reine Textdatei, exakt dasselbe Muster wie Internal Domains. War schon vorher als Altlast im Backlog vermerkt (Punkt 6 oben: "Alte Internal_Domains.txt bleibt bestehen..." – hier das externe Pendant). Betroffen: Agent 1 (schreibt/liest), Agent 2 (liest), Cockpit (zeigt Zähler). |
| `AuditTrail.xlsx` (Tabelle mit GUID `{81828E1C-...}`, ~19 Spalten: TimestampUtc, RunId, WorkflowPath, StepName, StepStatus, KeyOutput, DurationSec, ActionType, Direction, Recipient, Decision, Sender, MessageId, SenderDomain, MatchedDomain, SubjectOut, TargetFolderId/Name, TargetMessageId, FlowId) | ⚠️ Eigenes, größeres Projekt – NICHT nebenbei | Mittel (hoher Nutzen, aber hoher Aufwand/Risiko) | Höchster Nutzen (löst nebenbei auch die App-seitige Audit-Trail-Detailanzeige elegant, da die App dann direkt filtern/zählen kann statt über Agent-4-Statuszusammenfassung), ABER: **alle 6 Agenten schreiben hierher** (viel größerer Blast-Radius als Internal Domains mit nur 2 Agenten), reiches Spaltenschema, und bei sehr großen Listen (>5000 Zeilen) braucht SharePoint indizierte Spalten, sonst wird `Filter()` selbst wieder undelegierbar (List View Threshold). Muss pro Agent einzeln umgesetzt und getestet werden – nicht in einem Rutsch. |
| `AgentAuditSummary` (Rollup-Tabelle, 6 Zeilen – eine pro Agent: FailedStepsCount, WarningStepsCount, SucceededRunsCount, FailedRunsCount, WarningRunsCount, StartedRunsCount) | 💡 Idee, kein Muss | Niedrig | Könnte in die bereits bestehende `DMP Command Agent Status`-Liste integriert werden (ist ja schon 1 Zeile pro Agent) statt einer separaten Tabelle – spart eine ganze Datenquelle. Nur sinnvoll, wenn die Audit-Trail-Migration ohnehin ansteht. |
| Emergency Report Workbook, `Status DMP Process.xlsx` (Next Steps/Milestones) | ❌ Nicht migrieren | – | Echte mehrspaltige Businessdokumente, die Menschen in Excel bearbeiten/formatieren (Formeln, Formatierung, echte Report-Inhalte). Eine Liste bringt hier keinen Mehrwert und würde die Bearbeitung für die Fachseite erschweren. |

**Empfohlene Reihenfolge:**
1. Counter.xlsx → SharePoint-Liste `DMP Command Counters` (klein, isoliert, direkt nutzbar für den bestehenden Reset-Mechanismus).
2. External_Domains.txt → SharePoint-Liste `DMP Command External Domains` (spiegelt Internal Domains 1:1).
3. Audit Trail → SEPARATES, sorgfältig geplantes Projekt: pro Agent einzeln umstellen (Schreibaktion `AddRowV2` → SharePoint `CreateItem`), Spalten als Choice-Felder wo sinnvoll (StepStatus, ActionType, Direction, Decision), mit indizierten Spalten für Delegierbarkeit bei Wachstum. Die geplante "Archivieren + Audit-Datei neu starten"-Funktion (Nutzerwunsch) sollte auf Basis der NEUEN Listen-Struktur gebaut werden (Zeilen in ein Archiv-Listen-Duplikat kopieren + Original leeren), nicht mehr für die alte Excel-Variante. **Zusätzlicher Hinweis (2026-09-01):** Der "Open full Audit Trail file"-Button auf der Audit-Trail-Seite verlinkt aktuell direkt auf die rohe `.xlsx`-Datei (löst beim Klick einen Browser-"Speichern"-Dialog statt einer Online-Ansicht aus). Sobald diese Migration abgeschlossen ist, muss dieser Button stattdessen auf die neue SharePoint-Liste verweisen (`.../Lists/<Listenname>/AllItems.aspx`, analog zu den bereits umgestellten Internal-Domains-Buttons) – bewusst noch nicht vorher geändert, da ohne echten SharePoint-Freigabelink ein Rateversuch die Funktion eher verschlechtern als verbessern würde.

**Status:** Nur geplant/dokumentiert, noch nicht umgesetzt. Wird nach Bestätigung durch den Nutzer priorisiert angegangen.

---

## 🔴 BLOCKER gefunden (2026-09-01): Fortsetzung der SharePoint-Migration braucht zuerst zwei neue, vom Nutzer angelegte Listen

**Kontext:** Nutzerauftrag "mache mit dem Einbau der SharePoint-Listen weiter" (während er in der Mittagspause war). Vor Beginn der eigentlichen Umsetzung wurde geprüft, welche SharePoint-Listen aktuell an die App angebunden sind (`DataSources.json` im `.msapr`) – **nur drei** sind verbunden: `DMP Command Agent Status`, `DMP Command Configuration`, `DMP Command Internal Domains`. Weder `DMP Command Counters` noch `DMP Command External Domains` existieren bislang.

**Warum die KI hier nicht einfach selbst weitermachen kann:** Eine neue SharePoint-Liste anzulegen UND sie als Datenquelle mit der Canvas-App zu verbinden, erfordert entweder direkten SharePoint/Graph-API-Zugriff (bereits früher als nicht funktionierend dokumentiert – 401, kein Login möglich) oder eine manuelle Aktion in Power Apps Studio (wie bereits bei "Internal Domains" geschehen: "vom Nutzer in Studio verbunden, von der KI verifiziert"). Ohne die real angelegte Liste fehlt außerdem die interne SharePoint-GUID der Liste/Tabelle, die die Flow-Aktionen (`GetItems`/`CreateItem`/`UpdateItem`) zwingend referenzieren müssen – diese lässt sich nicht im Voraus erraten oder manuell einsetzen, sie wird erst beim tatsächlichen Verbinden generiert.

**Exakte Ziel-Schemata (aus den bestehenden Excel-/Textdatei-Strukturen abgeleitet, bereit zum 1:1-Nachbau in SharePoint):**

**1) `DMP Command Counters`** (Ersatz für `Counter.xlsx`, Tabelle `DMP_Email_Counter`, 4 Zeilen):
- Spalte `Title` (Standard-SharePoint-Spalte, dient als Schlüssel) – exakte Werte: `No DMP`, `DMP internal Sender`, `DMP effected Member`, `DMP not effected Sender` (vier Zeilen anlegen, Groß-/Kleinschreibung und Leerzeichen exakt wie hier, da Agent 6 diese Strings unverändert für `idColumn`-Lookups verwendet).
- Spalte `NumberProcessedEmails` (Zahl, Startwert `0` für alle vier Zeilen).
- **Betroffene Agenten:** Agent 2 (inkrementiert den passenden Zähler bei jeder klassifizierten E-Mail – GEnauer Lese-Erhöhen-Schreiben-Zyklus, nicht nur ein einfacher manueller CRUD wie bei Internal Domains!), Agent 6 (liest+setzt auf 0 zurück, protokolliert den alten Wert vorher im Audit Trail).
- **Wichtiger Unterschied zu Internal Domains:** Bei SharePoint gibt es kein direktes "GetItem by beliebige Spalte" wie Excels `idColumn` – die Flow-Umstellung braucht statt `GetItem`/`UpdateItem` (Excel) ein `GetItems`+`Filter` (`Title eq 'No DMP'`) gefolgt von `UpdateItem` mit der von SharePoint intern vergebenen numerischen `ID` der gefundenen Zeile. Für Agent 2 (Inkrementieren) zusätzlich eine Lese-vor-Schreiben-Sequenz nötig (kein natives "+1"-Atomic-Update in SharePoint) – bei parallelen E-Mail-Verarbeitungen ließe sich das ggf. durch sehr kurze Verarbeitungszeiten pro Nachricht in der Praxis vernachlässigen, sollte aber im Test beobachtet werden.

**2) `DMP Command External Domains`** (Ersatz für `External_Domains.txt`):
- Spalte `Title` (Domain-Name, ein Eintrag pro Zeile, z. B. `example.com`).
- **Wichtiger Unterschied zur Ursprungsannahme "spiegelt Internal Domains 1:1":** Anders als Internal Domains (von Menschen manuell in SharePoint gepflegte Allow-Liste mit `Active`-Spalte) wird External_Domains.txt von **Agent 1 bei jedem Lauf komplett neu geschrieben** (alle extrahierten Domains als eine neue Datei, keine "Active"-Markierung, kein inkrementelles Update). Die SharePoint-Entsprechung braucht daher KEINE `Active`-Spalte, sondern eine **"Full Sync"-Logik in Agent 1**: bei jedem Lauf zuerst alle bestehenden Zeilen der Liste löschen (`GetItems` alle Zeilen → `DeleteItem` je Zeile, oder als Batch), dann für jede frisch extrahierte Domain eine neue Zeile per `CreateItem` anlegen. Das ist ein größerer Eingriff in Agent 1 als der reine Feld-Umzug bei Internal Domains.
- **Betroffene Agenten:** Agent 1 (schreibt komplett neu bei jedem Lauf), Agent 2 (liest die Liste für den Domain-Abgleich, analog zum bereits fertigen aber noch nicht deployten Internal-Domains-Umbau).

**Nächste Schritte (sobald der Nutzer zurück ist):**
1. Nutzer legt beide Listen in SharePoint an (exakte Spalten siehe oben) und verbindet sie in Power Apps Studio als Datenquelle (wie bei Internal Domains).
2. KI baut danach analog zum bereits fertigen (aber noch nicht deployten) Internal-Domains-Umbau: zuerst Agent 6 (Counter-Reset, geringstes Risiko, kein Produktions-E-Mail-Pfad), dann Agent 2 (Counter-Inkrement UND External-Domains-Lesen – höheres Risiko, da Produktions-E-Mail-Verarbeitung), dann Agent 1 (External-Domains-Schreiben/Full-Sync).
3. Wie beim Internal-Domains-Vorbild: Änderungen zunächst NUR im Git-Repository committen, NICHT live deployen (`pac solution import`), bis der Nutzer verfügbar ist und die Umstellung begleitet testen kann (Agent 2 verarbeitet produktive E-Mails).

---

## ✅ Update (2026-09-01): Beide Listen angelegt, Agent 6 auf SharePoint umgestellt

**Beide Listen wurden vom Nutzer angelegt und mit der App verbunden.** Die App wurde erneut veröffentlicht, wodurch die KI die internen SharePoint-GUIDs per `pac canvas download` (frischer Export der `DataSources.json`) ermitteln konnte:
- `DMP Command Counters` → GUID `277307f0-195a-4e78-afc8-850dbdf956b2`
- `DMP Command External Domains` → GUID `dc89aed5-d87a-4e12-875a-db2adbc2cee4`

**Agent 6 (Counter-Reset) fertig umgebaut:** Alle 5 Fälle (`Case_ResetCounter_NoDMP`, `_InternalSender`, `_Effected`, `_NotEffected`, `Case_ResetAllCounters`) nutzen jetzt `GetItems` mit `$filter: "Title eq '...'"` gefolgt von `PatchItem` mit `id: =first(body('GET_...')?['value'])?['ID']` und `item/NumberProcessedEmails: 0`, statt der alten Excel-`GetItem`/`PatchItem`-Aktionen. Die verwaisten Konfigurationswerte (`CounterFolder`, `CounterFileName`, `CounterTableName`, `CounterTableColumnNamePath`, `CounterTableColumnNameCounter`) werden nicht mehr referenziert – sie können bei Gelegenheit aus der `DMP Command Configuration`-Liste entfernt werden (rein aufräumend, keine Funktionsauswirkung, da nichts mehr darauf zugreift).
- Vollständig validiert (JSON-Syntax, Klammerbalance, keine Beschreibung >255 Zeichen, keine doppelten Aktionsnamen) und ins Git-Repository committet – **noch NICHT live importiert** (`pac solution import`), wartet auf Nutzer-Test.

**Noch offen:** Agent 2 (Counter-Inkrement + External-Domains-Lesen) und Agent 1 (External-Domains-Schreiben/Full-Sync) – als Nächstes in dieser Reihenfolge.

---




**Anlass:** Fortsetzung der Anmerkungsrunde ("Ich schicke dir jetzt die Anmerkungen einzeln") plus Nutzerauftrag im Autopilot-Modus ("die App wie besprochen aus[bauen], integriere auch gleich die Verwaltung der internen domains, prüfe auch, dass der reserved space sich in das Layout einpasst").

**1) Einheitliche, minimale Abstände (8px überall):** `conMain`-LayoutGap (16→8), Y-Abstand zwischen Operating State/Maintenance Domains (16→8) – vorher inkonsistent 8px horizontal vs. 16px vertikal.

**2) Maintenance Domains bereinigt:** Bindestriche vor/nach INTERNAL/EXTERNAL entfernt (Nutzer-Feedback: "keine Punkte vor oder nach Intern/External"), Label-Breite 80→100px, Container 440→400px verschmälert – Operating State im gleichen Maß mitgeschrumpft (Breitengleichheit bleibt Pflicht). Emergency-Report-LED dabei als kleines Badge auf die Replace-Button-Ecke verschoben (X=376,Y=88), da sie sonst über den neuen schmaleren Rand hinausgeragt hätte.

**3) Files-Container schmaler (490→470px)**, engere Status/Last-Updated-Spalten – behebt den zu großen Leerraum nach dem CEST-Zeitstempel.

**4) Ring-Höhen-Alignment (System Health / Emails Processed):** Beide Karten jetzt exakt 328px hoch (= Operating State 160 + Abstand 8 + Maintenance Domains 160). Dafür Ring-Padding oben/unten 16→8px reduziert und die Ringe selbst leicht verkleinert (328→312px Durchmesser), `conRow1`/`conMiddleColumn` ebenfalls auf 328px vereinheitlicht.

**5) System-Health-Tooltip ersetzt:** Nutzer meldete, die native Tooltip-Funktion funktioniere gar nicht (im Gegensatz zu Emails Processed, wo sie nur die erste Zeile zeigte). Beide Ring-Tooltips jetzt als Klick-Popup mit echten Farb-/Status-Punkten gelöst (`conHeartbeatLegendPopup`, analog zu `conEmailsLegendPopup`), da native Tooltips bei Bild-Steuerelementen unzuverlässig sind.

**6) Release-Notes-Laufleiste – zweiter Anlauf:** Der erste Fix (`LayoutOverflowY: =LayoutOverflow.Scroll`) allein reichte nicht, weil `conReleaseNotesList` keine echte Höhenbegrenzung hatte (nur `FillPortions: =1`, kein `Height`/`LayoutMinHeight`/`LayoutMaxHeight`) – ohne feste Höhe wächst der Container einfach mit, ein Overflow-Zustand tritt nie ein. Fix: `Height`/`LayoutMinHeight`/`LayoutMaxHeight` auf `Parent.Height - 120` gesetzt.

**7) Drei SharePoint-Listen direkt an die App angebunden** (vom Nutzer in Studio verbunden, von der KI verifiziert): `DMP Command Configuration`, `DMP Command Agent Status`, und die **neu angelegte** `DMP Command Internal Domains` (Spalten `Title`, `Active` als Choice-Feld Yes/No – Achtung: SharePoint liefert Choice-Spalten als Objekt mit `.Value`, nicht als Text, Formel entsprechend `Active.Value = "Yes"`).

**8) Internal-Domains-Verwaltung auf SharePoint umgestellt (nur Power-App-Seite, Agenten folgen separat):** Die "Internal Domains"-Zeile in Maintenance Domains liest die Anzahl jetzt live per `CountRows(Filter('DMP Command Internal Domains', Active.Value = "Yes"))` statt über den Agent-4-gelieferten Wert `varInternalDomainsCount`. View/Edit-Buttons verlinken jetzt direkt auf die SharePoint-Liste (`.../Lists/DMP Command Internal Domains/AllItems.aspx`) statt auf die alte Textdatei. **Wichtig:** Die Backend-Agenten (Agent 1/Agent 2) lesen/schreiben weiterhin die alte `Internal_Domains.txt` – diese Umstellung ist ausdrücklich als nächster, separater Schritt geplant (siehe offener Punkt unten).

**9) "Automation Status" komplett neu aufgebaut:** Alter, eigenständiger Container am Seitenende (`conAutomationStatus` mit `conStep1`-`conStep7`, überwiegend generische manuelle Checklisten-Punkte wie "Check Agent 2 inbox", "Validate Emergency Report", "Audit Trail review", "End Fire Drill", "Export weekly report") **ersatzlos entfernt** (Nutzer-Vorgabe: "sollte es zu viel werden, lasse lieber etwas weg und schreibe es in den Backlog"). Stattdessen kompaktes 4-Zeilen-Statuspanel im ehemaligen "RESERVED FOR FUTURE USE"-Platzhalter (jetzt in `conFilesPlaceholder` integriert, Höhe 162px reicht dafür komfortabel):
   - a) Agent 4 Live Status (Status-Punkt + Text, aus `varIsRefreshing`/`varStatusCallError`)
   - b) Internal Domains (SharePoint) – aktive Anzahl
   - c) Config Parameters active – `CountRows(Filter('DMP Command Configuration', Active = "Yes"))`
   - d) Agent Status entries – `CountRows('DMP Command Agent Status')`

   **Bewusst NICHT wiederhergestellt** (aus Platzgründen, siehe Punkt 9 oben): die generischen "Nächste manuelle Schritte"-Hinweise (Check Agent 2 inbox / Validate Emergency Report / Audit Trail review / End Fire Drill / Export weekly report). Der Inhalt bleibt über die Git-Historie (Commit vor diesem) wiederherstellbar, falls später gewünscht – ggf. als eigener, neuer Bereich statt im Reserved-Slot.

**10) Root-Cause-Fix – wiederkehrendes Agent-4-Verbindungsproblem:** Siehe eigener Rules-Eintrag vom 2026-08-28 in `DMP COMMAND_Mission_und_KI_Arbeitsregeln.md`. Kurzfassung: Die App-interne Flow-Verbindungsreferenz lag in der git-versionierten `DMP_COMMAND.msapr`-Datei, nicht in den YAML-Dateien, und war seit einer früheren Agent-4-Flow-Neuanlage veraltet (falsche `FlowNameId`) – jedes KI-seitige Deployment hat dadurch die kaputte Verbindung zurückgeschrieben. Behoben durch Re-Synchronisation der `.msapr`-Datei aus der aktuell veröffentlichten App (`pac canvas download` + `pac canvas unpack --layout SourceCode`). **Wichtig:** Dieser Sync musste im Verlauf der Sitzung ein zweites Mal wiederholt werden, weil zwischen den beiden Syncs eine dritte SharePoint-Verbindung (Internal Domains) hinzugekommen war und sonst gefehlt hätte – vor jedem zukünftigen finalen Pack-Vorgang ist ein frischer `.msapr`-Abgleich Pflicht (siehe Regelwerk).

**Offene Punkte für die nächste Runde:**
- **Agenten-Umstellung auf die neue SharePoint-Liste** – ✅ UMGESETZT (2026-09-01, siehe eigener Abschnitt unten), aber **NOCH NICHT live deployt** (bewusst zurückgehalten, siehe dort).
- Generische "Nächste Schritte"-Checkliste (siehe Punkt 9) – neuer Platz/Entscheidung nötig, ob überhaupt wieder gewünscht.
- Operational-Manuals-Aktualisierung (Nutzerauftrag 3) – noch zu prüfen, welche Dokumente betroffen sind.

**Deployment:** Alle 4 Pflichtprüfungen bestanden (Mojibake 0, Doppelpunkt-Scan 0, Geschwister-Einrückung 0, Round-Trip 0 Diff – inkl. Container-Anzahl-Gegenprobe vor/nach Entfernen von `conAutomationStatus`: 45→37, exakt die erwarteten 8 Container). `.msapp` gepackt, Version `v1.10.0`.

---

## ✅ NEU (2026-09-01): Agent 2 + Agent 4 auf neue Internal-Domains-SharePoint-Liste umgestellt (Backend, NOCH NICHT live deployt)

**Anlass:** Nutzerauftrag 2 aus der Autopilot-Anweisung ("baue die Struktur der Agenten um, die die neue SharePoint-Liste der internal domains verwenden. Du kannst auch bereits die Agenten ändern").

**Betroffene Agenten identifiziert:** Nur zwei der sechs Agenten lesen `Internal_Domains.txt` – **Agent 4** (`DMPAgent302StatusCheckVS...json`, intern noch "3.02" benannt, extern "Agent 4") für die reine Status-Anzeige (Existenz/Anzahl/Änderungsdatum), und **Agent 2** (`DMPAgent2E-MailInboxTreatmentVS...json`) für die tatsächliche Klassifizierungslogik (Sender-Domain-Abgleich gegen die Liste). Agent 1 und Agent 3 sind nicht betroffen.

**Agent 4 (`SCOPE_Internal_Domains`):** Kompletter Ersatz der Datei-basierten Prüfung (`GetFileMetadataByPath` + `GetFileContentByPath` + Zeilen-Split-Zählung) durch eine SharePoint-Listenabfrage (`GetItems`, Tabelle `1ac9e8e2-6c55-435b-a188-44093da5aa8f`, Filter `Active eq 'Yes'`). Die nach außen sichtbaren Variablen (`InternalDomainsExists`, `InternalDomainsCount`, `InternalDomainsLastModified`) blieben unverändert – nur ihre interne Berechnung wurde umgestellt (`LastModified` jetzt als `max()` über alle Zeilen-Zeitstempel statt Datei-Metadatum).

**Agent 2 (Sender-Klassifizierung):** `Check_Existence_of_Internal_Domains_File` liest jetzt dieselbe SharePoint-Liste (`GetItems`, Filter `Active eq 'Yes'`) statt eine gefilterte Datei-Metadaten-Abfrage auf die Textdatei. Die separate `Load_Internal_Domains`-Aktion (Volltext-Lesen der Datei) wurde **ersatzlos entfernt**, da die Listendaten bereits aus dem Existenz-Check verfügbar sind – spart einen kompletten SharePoint-Aufruf pro E-Mail. `Parse_internal_domains_file_+_create_array` baut das Domain-Array jetzt per `select(...)` über die `Title`-Spalte der Listenzeilen statt per Zeilen-Split der Textdatei – das Ergebnis-Array hat exakt dieselbe Form (kleingeschriebene Domain-Strings) wie zuvor, sodass die komplette nachgelagerte Abgleichslogik (`MatchedInternalDomain`, `DomainClassificationInternal`, `DecisionInternal`, Audit-Buffering) **unverändert** bleibt – bewusst so gewählt, um das Risiko bei dieser produktiven E-Mail-Klassifizierungslogik zu minimieren. Die Alarm-Mail-Formulierung ("Datei fehlt" mit Pfadangabe) wurde auf "Liste hat keine aktiven Einträge" umgestellt.

**Validierung:** Für beide JSON-Dateien vollständig durchgeführt: JSON-Syntax gültig, alle Beschreibungstexte ≤ 255 Zeichen (206 bzw. 447 geprüft), alle `runAfter`-Referenzen zeigen auf existierende Geschwister-Aktionen (rekursiv über die komplette Flow-Struktur, nicht nur den geänderten Abschnitt), alle `SetVariable`/`AppendToArrayVariable`-Namen sind per `InitializeVariable` deklariert.

**Bewusst NICHT deployt:** Die Änderungen sind nur im Git-Repository (Quelldateien) committed, **nicht** per `pac solution import` in die Live-Umgebung importiert. Agent 2 verarbeitet produktiv eingehende E-Mails – eine Live-Aktivierung ohne Möglichkeit zum begleiteten Testen wurde als zu riskant eingeschätzt und daher bewusst zurückgehalten, bis der Nutzer grünes Licht gibt bzw. selbst testen kann. Nach Freigabe: `pac solution pack` (Ordner `PowerAutomate\DMP_COMMAND_Solution\Source`) gefolgt von `pac solution import` gegen die Zielumgebung.

**Nächste Schritte / offen:**
- Freigabe zum Live-Import einholen, dann `pac solution import` ausführen.
- Nach Import: mindestens eine Test-Mail von einer bekannten internen Domain und einer bekannten externen Domain durchlaufen lassen, um die Klassifizierung zu verifizieren.
- Die jetzt ungenutzten Konfigurationswerte `InternalDomainsFileName`/`InternalDomainsStorageFolder` (weiterhin aus `DMP Command Configuration` geladen, aber von `SCOPE_Internal_Domains`/Agent 2 nicht mehr referenziert) können bei Gelegenheit aufgeräumt werden – bewusst nicht in dieser Runde entfernt, um das Risiko klein zu halten.
- Die alte `Internal_Domains.txt` bleibt vorerst bestehen (kein Löschauftrag erteilt) – erst entfernen, wenn der Nutzer bestätigt, dass die SharePoint-Liste sich bewährt hat.

---

## ✅ NEU (2026-08-28, v1.9.3): Release Notes lesbar + scrollbar + Copy-to-Clipboard, Files-Container-Rand-Fix, Ring-Padding weiter reduziert, Emails-Legende als Farb-Popup

**Anlass:** Nutzer schickte mehrere GUI-Anmerkungen nacheinander und bat ausdrücklich, mit dem Deployment zu warten, bis alle Punkte gesammelt sind ("Ich schicke dir jetzt die Anmerkungen einzeln. Bitte warte mit dem Deployment").

**Punkt 1 – Release Notes nicht lesbar:** Alle 6 "Notes"-Label (`lblReleaseV190Notes` bis `lblReleaseOlderNotes`) hatten `Wrap: =true`, aber eine manuell geschätzte, statische `Height`, die mit wachsendem Änderungsprotokoll zu klein wurde – der Text wurde sichtbar abgeschnitten statt dass der Container mitwuchs. **Fix:** `AutoHeight: =true` auf allen 6 Labels ergänzt.

**Punkt 2 – Laufleiste + Copy-to-Clipboard:**
- Laufleiste: `LayoutOverflowY: =LayoutOverflow.Scroll` auf `conReleaseNotesList` (AutoLayout-Container) gesetzt. Exakte Property wurde vorab per Recherche an mehreren echten `.pa.yaml`-Quellen verifiziert (u. a. `GroupContainer@1.5.0`-Fixtures), da falsche Property-Namen in dieser Sitzung bereits zweimal zu Studio-Öffnungsfehlern geführt hatten.
- Neuer Button `btnCopyReleaseNotesToClipboard` im Header (grün, oben rechts), der per `Copy()`-Funktion den gesamten Versionsverlauf als Klartext in die Zwischenablage kopiert, mit `IfError()`-Absicherung und `Notify()`-Rückmeldung (Erfolg/Fehler) – gemäß Microsofts eigener Empfehlung zu Clipboard-Einschränkungen in eingebetteten Hosting-Kontexten.

**Punkt 3 – Files-Container: rechter Rand nicht mehr sichtbar:** Ursache: `conFilesRow` war 530px breit bei X=448 (→ 978px), aber der Elterncontainer `conMiddleColumn` nur 948px breit – die rechten ca. 30px (inkl. Rahmen) wurden vom `conEmailsCard` daneben verdeckt. **Fix:** Breite auf 490px reduziert (Placeholder-Container `conFilesPlaceholder` darunter zwecks Höhengleichheit mitgezogen). Zusätzlich die Spaltenlücken (Status/Last Updated) weiter verengt (Name-Spalte 150→130px, Status-Spalte 100→90px bei X=180 statt 200, Last-Updated-Spalte 200→190px bei X=280 statt 310) für weiteren Platzgewinn.

**Punkt 4 – Ring-Abstände bei Heartbeat/Emails-Karten:** Auf Wunsch weiter reduziert: inneres Padding 8px→4px, Kartenbreite 344px→336px, Ring-X-Position 8→4px (Ring-Durchmesser selbst unverändert).

**Punkt 5 – Emails-Processed-Tooltip-Farben passen nicht zu den Ringsegmenten:** Die native Power-Apps-`Tooltip`-Property kann nur reinen Text ohne Farben darstellen. Nutzer entschied sich (nach Rückfrage) für die aufwendigere Variante: eigenes Klick-Popup statt Tooltip. **Umsetzung:** Unsichtbarer Klick-Button (`btnEmailsLegendToggle`) über dem Ring togglet die neue Variable `varShowEmailsLegend` (in `App.OnStart` mit `false` initialisiert); das Popup (`conEmailsLegendPopup`) zeigt 4 Zeilen mit echten farbigen Quadraten (`RGBA(60,125,190,1)` Blau/`RGBA(102,52,142,1)` Dunkellila/`RGBA(153,102,178,1)` Helllila/`RGBA(255,159,10,1)` Orange – identisch zu den SVG-Ringfarben) plus Zahl/Prozent-Text und eigenem ✕-Schließen-Button.

**Deployment:** Alle 4 Pflichtprüfungen bestanden (Mojibake 0, Doppelpunkt-Scan 0, Geschwister-Einrückung 0 – Checker dabei robuster gemacht, da die einfache Variante bei verschachtelten `Children:`-Listen fälschlich Treffer meldete –, Round-Trip 0 Diff über alle Dateien). `.msapp` gepackt, Version `v1.9.3`.

---

## ✅ NEU (2026-08-28, v1.9.2): Replace-Button-Trefferfeld vergrößert

**Anlass:** Nutzer fragte "kannst Du das Trefferfeld vergrößern? Eventuell unsichtbar?" – ohne expliziten Kontext, da der `ask_user`-Rückfragedialog fehlschlug (bekanntes Problem dieser Sitzung). Autonome Entscheidung: Bezug auf den Replace-Button (Maintenance Domains, External Domains) angenommen, da dieser seit Langem als schwer klickbar bekannt ist (siehe frühere Backlog-Einträge) und gerade erst im Zuge des Maintenance-Domains-Umbaus neu positioniert wurde.

**Analyse:** Das unsichtbare `Attachments`-Steuerelement (`attExternalDomainsReplace`, der eigentliche Klick-/Upload-Auslöser hinter dem sichtbaren "Replace"-Rahmen) war bereits geringfügig größer als der sichtbare Rahmen (95×54 vs. 90×32), aber nur nach unten/rechts versetzt – nicht als zusätzlicher Rand in alle Richtungen.

**Fix:** Trefferfeld auf 102×64px vergrößert, mit negativem X/Y-Offset (-6/-16), sodass es den sichtbaren "Replace"-Rahmen in alle Richtungen großzügig überragt (oben/unten +16px, links/rechts +6px) – bewusst asymmetrisch, um NICHT in den benachbarten Edit-Knopf (endet bei X=288, neues Trefferfeld beginnt bei X=290, 2px Abstand) oder die Emergency-Report-LED (beginnt bei X=394, Trefferfeld endet bei X=392, 2px Abstand) hineinzuragen. Weiterhin unsichtbar (Opacity 0.01 wie zuvor).

**Deployment:** Alle 4 Pflichtprüfungen bestanden, `.msapp` gepackt, Version `v1.9.2`.

**Bitte Nutzer-Feedback abwarten:** Falls sich meine Annahme (Replace-Button statt Ack-LED) als falsch herausstellt, muss dies in der nächsten Runde korrigiert werden.

---

## ✅ NACHKORREKTUR (2026-08-28, v1.9.1): Maintenance-Domains-Format exakt nach Nutzervorgabe

**Nutzer-Korrektur:** Das v1.9.0-Format ("999 - INTERNAL" gefolgt direkt von Knöpfen) hatte einen fehlenden zweiten Bindestrich. Exaktes Zielformat: `[ANZAHL] - [ORT] - [ACTIONS]`, also `"999 - INTERNAL - [View] [Edit]"` bzw. `"999 - EXTERNAL - [View] [Edit] [Replace]"`.

**Fix:** Label-Text von `"- INTERNAL"`/`"- EXTERNAL"` auf reines `"INTERNAL"`/`"EXTERNAL"` geändert, zwei eigene kleine Bindestrich-Labels ergänzt (vor UND nach dem Orts-Namen): `lblInternalDomainsDash1`/`lblInternalDomainsDash2` (analog External). Knöpfe (View/Edit), Replace-Stack und LED entsprechend nach rechts verschoben, um Platz für den zweiten Bindestrich zu schaffen – weiterhin komfortabel innerhalb der 440px-Containerbreite (12-28px Sicherheitsmarge, je nach Zeile).

**Deployment:** Alle 4 Pflichtprüfungen bestanden (0 Mojibake, 0 Doppelpunkt-Risiko, 0 Geschwister-Einrückungsfehler, Round-Trip 0 Diff), Container-Anzahl unverändert (17/17). `.msapp` gepackt, Version `v1.9.1`.

---

## ✅ NEU (2026-08-28, v1.9.0): Files/Maintenance-Domains-Tausch, kompaktes Zahl-Label-Layout, Ack-LED-Klickfläche vergrößert

**1. "Operational Mode" → "MODE":** Umbenannt wie gewünscht. Container-Breite bewusst NICHT verkleinert (Nutzer-Korrektur: "Die Container sollen schon übereinander gleich groß sein: Sonst sieht das Scheiße aus") – Operating State bleibt exakt so breit wie Maintenance Domains (440px), da beide jetzt in derselben Spalte übereinanderliegen.

**2. Files-Container:**
- "Internal Domains"-Zeile entfernt (4 Zeilen statt 5, redundant zu Maintenance Domains).
- **Ursache der abgeschnittenen Schrift gefunden:** Beim letzten Umbau wurden die Spalten-Breiten (Name/Status/Zeit) zu aggressiv verkleinert, unabhängig von der Container-Breite – das Label selbst war zu schmal für seinen Text, nicht der Container. Neu: Name 150px, Status 100px, Zeit 200px (mit Sicherheitsmarge, nach den zwei vorherigen Abschneide-Vorfällen bewusst großzügiger).
- Container-Breite jetzt 530px (eigener, realistischer Bedarf – nicht mehr künstlich an die alte 440px-Spalte gezwängt).

**3. Maintenance Domains – komplett neues, kompaktes Layout (Nutzeridee):** Statt Label + weit rechts stehender Zahl-Spalte jetzt: **Zahl (grün, fett, links) + " - INTERNAL"/" - EXTERNAL" (Großbuchstaben) direkt daneben**, gefolgt von View/Edit(/Replace/LED bei External). Das spart deutlich Platz gegenüber der alten Lösung (Label + separate rechtsbündige "19 (SharePoint)"-Spalte). Suffixe "(SharePoint)"/"(File)" entfallen (nicht mehr nötig, da Kontext durch die Kachel klar ist). Container-Breite bleibt 440px (passend zu Operating State).

**4. Tausch Files ↔ Maintenance Domains (wie angefordert):** Neue Anordnung: Operating State (oben links) + Maintenance Domains (unten links, beide 440px) | Files (oben rechts) + Reserved-Placeholder (unten rechts, beide 530px). Konzeptionell sinnvoller: "Steuerung" (Operating State + Maintenance Domains) links, "Status-Anzeige" (Files + Reserved) rechts.

**5. Ehrliche Restrechnung:** Neue Gesamtbreite ≈ 2002px (Sidebar+Padding 320 + Heartbeat 344 + Mittelspalte 978 [440+8+530] + Emails 344 + 2×Gap 16) gegenüber 1919px verfügbar bei 100% Zoom → **83px zu viel**. Das ist mehr als die 53px aus der vorherigen Runde, weil Files jetzt genug Platz für lesbaren Text bekommt (530px statt 440px) – Lesbarkeit hatte in dieser Runde bewusst Vorrang vor exaktem Breiten-Fit. Nächste mögliche Einsparung (noch nicht umgesetzt): Sidebar weiter verkleinern, oder Zeitstempel-Format in Files kürzen (z. B. ohne Sekunden).

**6. Bestätigungs-LED (Critical/Warnings/Agents Active) schwer treffbar – behoben:** Ursache: Die kleine 12px-LED selbst hatte keinen Klick-Handler, nur die Zahl/Beschriftung daneben (mit nur ~6px zufälliger Überlappung zur LED). Fix: Neue unsichtbare, großzügige Klickfläche (`btnAckCriticalHitArea`/`btnAckWarningsHitArea`/`btnAckAgentsHitArea`) über die gesamte Kachel (Zahl+Label+LED), die dieselbe Bestätigungs-Aktion auslöst – jetzt reicht ein Klick irgendwo auf die Kachel, auch direkt auf die blinkende LED.

**Deployment:** Alle 4 Pflichtprüfungen bestanden (Mojibake 0, Doppelpunkt-Scan 0 – dabei erneut 4 Treffer in einem Release-Notes-Text gefunden und korrigiert, Geschwister-Einrückung 0, Round-Trip 0 Diff). `.msapp` gepackt, Version `v1.9.0`.

**Noch offen:** Reale Sichtbarkeitsprüfung bei 100% Zoom nach Deployment; verbleibende 83px ggf. durch Sidebar-Verkleinerung oder Zeitformat-Kürzung in einer weiteren Runde schließen.

---

## ✅ GELÖST (2026-08-28, v1.8.1): Zweiter Studio-Öffnungsfehler PA1001 (Geschwister-Einrückungsfehler bei conEmailsCard)

**Fehler:** `Src\scrHome.pa.yaml(1883,24) : error PA1001 : ... While parsing a block mapping, did not find expected key.` Direkt beim ersten Öffnungsversuch von v1.8.0 in Studio aufgetreten.

**Ursache:** Beim großen Breiten-Umbau wurde der komplette `conEmailsCard`-Block (Container + alle Kind-Elemente bis zur schließenden Klammer vor `conNextSteps`) beim `edit`-Tool-Einsatz um genau 1 Leerzeichen zu wenig eingerückt kopiert – 23 statt der korrekten 24 Leerzeichen, die seine Geschwister-Container `conHeartbeatCard`/`conMiddleColumn` unter demselben `Children:`-Knoten von `conRow1` haben. Der komplette Block war intern konsistent (jede Ebene relativ zueinander korrekt), aber strukturell 1 Ebene "verrutscht" gegenüber seinen echten Geschwistern.

**Warum unentdeckt bis Studio:** Exakt dasselbe Muster wie beim vorherigen Doppelpunkt-Bug – `pac canvas pack`/`unpack` hat den fehlerhaft eingerückten Block klaglos akzeptiert (Round-Trip ergab 0 Diff), weil `pac`s YAML-Parser nachweislich toleranter ist als Studios eigener PaYaml-Parser. Round-Trip-Gleichheit beweist nur „von `pac` konsistent geparst", nicht „von Studio akzeptiert".

**Fix:** Alle betroffenen Zeilen (kompletter `conEmailsCard`-Subtree) um exakt 1 Leerzeichen nachträglich ergänzt, damit sie wieder exakt auf der Einrückungsebene ihrer Geschwister liegen. Verifiziert durch manuellen Abgleich der Einrückungstiefe gegen `conHeartbeatCard` (Referenz).

**Neuer, dauerhafter Schutzmechanismus:** Eigenständiger PowerShell-Prüfschritt geschrieben, der für jeden `Children:`-Knoten im YAML programmatisch verifiziert, dass alle direkten Geschwister-Listenelemente exakt dieselbe Einrückungstiefe haben (siehe Arbeitsregeln-Ergänzung). Über alle 9 zuletzt geänderten Dateien laufen lassen – 0 verbleibende Treffer. Dieser Check ist jetzt PFLICHT-Bestandteil derselben Validierungsroutine wie Mojibake-/Doppelpunkt-Scan/Round-Trip (insgesamt jetzt 4 Prüfungen vor jeder Auslieferung).

**Deployment:** `.msapp` neu gepackt, PowerApp-Version auf `v1.8.1`. Kein weiterer inhaltlicher/gestalterischer Unterschied zu v1.8.0 – reiner Bugfix.

---

## ✅ GROSSER BREITEN-UMBAU (2026-08-28, v1.8.0): Logo-Fix, Operating-State/Files/Maintenance-Domains-Verschlankung, Ring-Tooltips

**Anlass:** Nutzer meldete, das Cockpit-Layout sei nur bei 75% Browser-Zoom vollständig sichtbar (Standard: 100%). Gemeinsames Brainstorming (Dialogmodus) mit dem Nutzer, um systematisch Breite einzusparen, ohne erneut Texte abzuschneiden oder den Replace-Button zu beschädigen (beides bereits einmal in dieser Sitzung passiert).

**1. Logo-Fix (eigentliche Ursache endlich gefunden):** Das eingebettete Logo-PNG (1024×1024px, für Dark- und Light-Mode je eine Variante) hatte ca. 18-25% Leerraum auf allen 4 Seiten fest eingebrannt (identische Hintergrundfarbe wie der Rand, aber Teil der Bilddatei, nicht transparent) – deshalb hat jedes bisherige Vergrößern der Anzeige-Box nichts gebracht, der Leerraum wuchs proportional mit. Per `System.Drawing` pixelgenau den tatsächlichen Bildinhalt ermittelt (Farbabstand zur Eckfarbe) und auf exakt denselben Ausschnitt (+20px Sicherheitsmarge) zugeschnitten, für beide Modi identisch. Neues Seitenverhältnis 654:559 (≈1,17:1, vorher 288:194≈1,48:1 reine Box ohne Bezug zum Bildinhalt).

**2. Systematisches Breiten-Trimming (Dialogmodus, Nutzer hat jeden Schritt vorgegeben/bestätigt):**
- `conOperatingState`: "Mode" nicht mehr eigene Zeile, sondern Untertitel direkt unter dem Kachel-Titel, in der Rahmenfarbe des jeweiligen Umgebungsmodus eingefärbt (PROD+DMP=Rot, PROD+Normal=Grün, SIMU+DMP=Orange, SIMU+Normal=Grau), blinkt während des Umschaltens. Alle übrigen Labels ("Last Changed"/"Operational Mode"/"Environment") auf einheitlichen kleinen Grauschrift-Standard (Semibold, 9px, GROSSBUCHSTABEN) umgestellt, Toggle/LED näher an die Labels gerückt. Breite 550→**440px**.
- `conFilesRow`: Spalten Name/Status/Zeit verschlankt (Name 160→120px, Status 100→80px, Zeit 214→165px, Abstände verkleinert), Breite 550→**440px** (identisch zu Operating State, da beide in derselben Spalte übereinanderliegen).
- `conMaintenanceDomains`: Label "Internal"/"External" 140→70px, Knöpfe "View"/"Edit" verschmälert (70→56px / 80→60px), Zähler-Spalte 144/124→110px vereinheitlicht, Breite 620→**500px**.
- `conFilesPlaceholder`: identisch auf **500px** mitgezogen (gleiche Spalte wie Maintenance Domains).
- **Licht-Modus-Lesbarkeit:** Alle zu hellen Grautöne (`RGBA(90,88,100,1)`) in Files/Maintenance Domains/Next Steps auf dunkler `RGBA(45,43,53,1)` umgestellt (14 Fundstellen, per gezieltem Text-Replace, 1 bewusst ausgenommene invertierte Fundstelle bei „RESERVED FOR FUTURE USE" unangetastet gelassen).
- **Ring-Container (Nutzeridee):** Bei `conEmailsCard` die 4 Legenden-Labels (No DMP/External/Internal/Not Effected) komplett entfernt und durch EINEN zusammengefassten Tooltip auf dem Ring selbst ersetzt. Dadurch kann der Ring exakt gleich groß bleiben (328px, unverändert) wie bei `conHeartbeatCard`, während die Karte trotzdem stark schrumpft. Padding um beide Ringe von 24px auf **8px** reduziert. `conHeartbeatCard` bekam ebenfalls einen neuen Tooltip (6-Punkte-Aufschlüsselung: Emergency Report/Internal Domains/External Domains/Counter File/Audit Trail/Agent 6, je OK/Missing) – gab es vorher gar nicht. Beide Karten: 376px/560px → **344px**.
- **Abstände:** `conRow1`-Zeilenabstand 16→8px, Abstand zwischen linker/rechter Unterspalte in der Mitte 16→8px.
- **Sidebar:** 300→**280px** (Logo entsprechend auf 268×229px nachjustiert, weiterhin volle Breite ausfüllend).

**3. Ehrliche Restrechnung (Breitenbudget):**
| Element | Breite |
|---|---|
| Sidebar + `conMain`-Padding | 320px |
| Heartbeat-Karte | 344px |
| Mittlere Spalte (440+8+500) | 948px |
| Emails-Karte | 344px |
| 2× Zeilen-Abstand | 16px |
| **Summe** | **1972px** |

Verfügbar (vom Nutzer per neuer Diagnose-Anzeige gemeldet, 100% Zoom): 1919px → **rechnerisch ca. 53px zu viel**, gegenüber ursprünglich ermittelten 575px also ca. 91% des Fehlbetrags geschlossen. Da alle Breitenschätzungen auf Zeichenbreiten-Näherungen beruhen (keine pixelgenaue Schriftrendering-Möglichkeit ohne Live-Rendering), könnte es in der Praxis knapp reichen oder auch nicht – **muss nach Deployment über die neue `App.Width`-Diagnoseanzeige auf der Admin-Seite real geprüft werden.**

**Falls nach dem Deployment noch ein Rest fehlt – nächste, noch NICHT angetastete Reserve:** Die Toggle-Breiten in `conOperatingState` (`tglOperationalState`/`tglApplicationMode`, aktuell 110px) wurden bewusst NICHT verkleinert (Funktionssteuerelemente, höheres Risiko als reine Labels). Hier stecken ggf. noch 15-20px je Toggle, falls nötig – aber nur nach erneuter Rücksprache, nicht vorsorglich.

**Wichtiger Vorfall während der Umsetzung (selbst gefunden, behoben, Regel verschärft):** Beim Release-Notes-Eintrag für v1.8.0 enthielt eine lange Inline-Formelzeile GLEICH 3 gefährliche `": "`-Doppelpunkte, der bisherige zeilenbasierte Scan-Regex hatte aber nur 1 davon gemeldet (er zählt nur „Zeile hat mind. 1 Treffer", nicht „wie viele"). Dadurch wurden 2 Vorkommen beim ersten Durchgang übersehen. Neuer, pro-Vorkommen zählender Scan eingeführt und in den Arbeitsregeln verankert (siehe dortige Ergänzung vom 2026-08-28).

**Deployment:** Jede einzelne Änderungsrunde separat validiert (Mojibake-Scan 0, Doppelpunkt-Scan 0, Pack/Unpack-Round-Trip 0 Diff je Datei, in isoliertem Testordner). `DMP_COMMAND_Solution.msapp` final gepackt, PowerApp-Version auf `v1.8.0`, neuer Release-Notes-Eintrag ergänzt.

**Noch offen:**
- Reale Sichtbarkeitsprüfung bei 100% Zoom nach Deployment (siehe Restrechnung oben).
- Replace-Button-Fehler weiterhin ungeklärt (unverändert in dieser Runde, wie vereinbart nicht mit angefasst).
- Internal Domains → SharePoint-Liste-Migration (vom Nutzer als hohe Priorität für „heute noch" angekündigt, aber noch nicht begonnen – nach Abschluss würde die Internal-Domains-Zeile in `conFilesRow` entfallen und zusätzliche Höhe/Breite freigeben).
- Phase 2/3 des Nutzerkonzepts (Admin-Freigabe, rollenbasierte Menüpunkte) weiterhin offen.

---

## ✅ GELÖST (2026-08-28, v1.7.1): Studio-Öffnungsfehler PA1001 (YAML-Doppelpunkt-Bug) + Diagnose-Anzeige + Agent-4-Verbindungsproblem geklärt

**🔴 Kritischer Bug (Nutzer konnte die App nicht mehr öffnen):** Beim Öffnen von `v1.7.0` in Power Apps Studio: `Src\scrReleaseNotes.pa.yaml(87,62) : error PA1001 : ... While scanning a plain scalar value, found invalid mapping.` Ursache: 3 `Text: =...`-Formeln enthielten eine literale `": "`-Sequenz (Doppelpunkt+Leerzeichen) innerhalb eines unquotierten YAML-Plain-Scalars (z. B. "current version: ", "popup: Admin", "Next Steps: milestone") – YAML interpretiert das als (ungültigen) Mapping-Beginn, obwohl es PowerFx-seitig ein normaler String war. **Wichtig:** Der bisherige Validierungsweg (`pac canvas pack`/`unpack`-Round-Trip, 0 Diff) hat diesen Fehler NICHT erkannt – `pac`s Parser ist toleranter als Studios eigener PaYaml-Parser. `pac canvas validate` existiert zwar als Befehl, ist laut CLI aber „no longer supported". **Fix:** Alle 3 betroffenen Doppelpunkte entfernt/durch " - " ersetzt, alle 9 zuletzt geänderten Dateien erneut auf das Muster gescannt (0 verbleibende Treffer), Round-Trip erneut bestanden, `.msapp` neu gepackt. Neue harte Regel in den Arbeitsregeln verankert (manueller Regex-Scan als Pflichtschritt vor jeder Auslieferung).

**✅ Neu: Diagnose-Anzeige auf der Admin-Seite** (`scrAdminFunctions.pa.yaml`, neue Kachel `conDisplayDiagnostics` direkt unter dem Header): zeigt `App.Width`/`App.Height` in px an, mit Hinweistext, diese Werte zusammen mit dem aktuellen Browser-Zoom zu melden. Hintergrund: Die Browser-Konsole (`window.innerWidth`) hat beim Nutzer nicht funktioniert – diese In-App-Anzeige ist der zuverlässigere Ersatzweg.

**🔍 Agent-4-Verbindungsproblem (Nutzerfrage, warum Agent 4 nach jeder Änderung gelöscht und neu eingebunden werden muss, während bei Agent 3/5/6 ein einfaches „Aktualisieren" reicht):** Geprüft, welche Flows die Power App direkt aufruft: `DMPAgent4(StatusCheck)=>VS` (mehrfach, unter anderem in `scrHome.OnVisible`/`tmrAutoRefreshTick`), `DMPAgent3.01(EmergencyReportManagement)=>VS`, `DMPAgent3.03(OperationalStateManagement)=>VS`, `DMPAgent6(AdminFunctions)`. **Befund:** Die aktuell verwendete `SourceCode`-Unpack-Struktur (`Source\Src\*.pa.yaml` + `.msapr`) enthält KEINE `DataSources.json`/Verbindungs-Metadaten als Textdatei – diese werden nur bei der (veralteten, von `pac` selbst als „deprecated" markierten) `Experimental`-Layout-Struktur als bearbeitbare Datei exportiert. Das bedeutet: **Ich kann die Verbindungs-/Schema-Zwischenspeicherung von Studio nicht direkt reparieren** (kein Dateizugriff darauf in der aktuellen, empfohlenen Struktur). **Wahrscheinliche Ursache (Erfahrungswert, nicht 100% verifizierbar ohne Studio-Zugriff):** `DMPAgent4(StatusCheck)` liefert das mit Abstand größte/komplexeste Antwortobjekt (`varStatusResult.*` mit über 20 Feldern, u. a. der verschachtelten Tabelle `nextmilestones`) und wurde im Projektverlauf am häufigsten erweitert – Power Apps Studio invalidiert bei solchen Antwortschema-Änderungen (neue/umbenannte/verschachtelte Felder) den lokalen Cache manchmal nicht sauber per „Aktualisieren", sondern nur bei vollständigem Entfernen+Neuverbinden. Agent 3/5/6 haben deutlich kleinere, stabilere Antwortschemata, daher seltener/nie dieses Problem. **Mitigation (keine vollständige Lösung, da Studio-Verhalten):** Künftige Änderungen an Agent 4s Response-Schema nach Möglichkeit nur additiv am Ende anfügen (keine Umbenennungen/Typänderungen an bestehenden Feldern), das reduziert die Häufigkeit, in der ein harter Reconnect nötig ist. Bei jeder Agent-4-Schemaänderung zusätzlich explizit erwähnen, dass ein Reconnect wahrscheinlich nötig sein wird (nicht nur „Aktualisieren" versuchen).

---

## ✅ PHASE 1 UMGESETZT (2026-08-28): Alle 6 Menüpunkte haben jetzt eine eigene Seite + Änderungs-Hinweis-LEDs

**Umsetzung von Phase 1 des Konzeptvorschlags (siehe unten):**
- `scrAdminFunctions.pa.yaml` (neu): kompletter Inhalt von `conAdminFunctions` (Löschen-Funktion für 'PA Processed Mails'-Ordnerbaum inkl. Bestätigungsdialog) 1:1 aus `scrHome.pa.yaml` hierher verschoben. Overlay-Logik (`varShowAdminPanel`-Toggle) entfernt.
- `scrAgentMonitoring.pa.yaml`, `scrOperationalBoard.pa.yaml`, `scrAuditTrail.pa.yaml`, `scrConfiguration.pa.yaml`, `scrMaintenance.pa.yaml` (alle neu): je ein einfacher "Coming Soon"-Platzhalter-Screen mit Header + "< Back to Cockpit"-Knopf, identisches Muster wie `scrReleaseNotes.pa.yaml`. Fachlicher Inhalt wird schrittweise in künftigen Sitzungen gefüllt.
- Alle 6 `btnNav*`-Knöpfe in `scrHome.pa.yaml` von `Notify("...coming next")`/Overlay-Toggle auf echtes `Navigate(scr..., ScreenTransition.None)` umgestellt.
- `conAdminFunctions`-Block (166 Zeilen) vollständig aus `scrHome.pa.yaml` entfernt (jetzt in eigener Datei).

**Neu: Änderungs-Hinweis-LEDs (Nutzeridee, im Rahmen derselben Runde umgesetzt):**
- Anlass: Der Nutzer möchte über Auto-Refresh erkennen, WO sich seit der letzten Bestätigung etwas geändert hat (z. B. eine neue Warnung), und dies gezielt bestätigen können – eine neue Warnung soll die LED danach erneut aufleuchten lassen.
- Umsetzung: 3 neue LEDs (`dotLedCriticalChanged`, `dotLedWarningsChanged`, `dotLedAgentsChanged`) an den 3 Header-Kacheln CRITICAL/WARNINGS/AGENTS ACTIVE, exakt im bestehenden LED-Muster (`dotLedOperationalState` u. Ä. – 18px Kreis, hier 12px, blinkend über `varBlinkPhase`). Farbe bewusst Blau (`RGBA(0,120,215,1)`) gewählt, um Verwechslung mit den bereits belegten Status-Farben (Grün=gesund, Orange=Warnung, Rot=kritisch, Grau=unbekannt) zu vermeiden – die neue LED bedeutet ausschließlich "seit letzter Bestätigung geändert", keine eigene Statusaussage.
- Baseline-Logik: `varLastAckCriticalCount`/`varLastAckWarningCount`/`varLastAckAgentsHealthyCount` werden EINMALIG beim ersten erfolgreichen Refresh der Sitzung (in `scrHome.OnVisible`) auf den dann aktuellen Wert gesetzt (`varBaselineAcknowledged`-Flag verhindert ein erneutes Setzen bei jedem Auto-Refresh-Tick). Jede spätere Abweichung von dieser Baseline (Erhöhung, Verringerung, jede Änderung) lässt die passende LED aufleuchten, bis der Nutzer durch Klick auf die jeweilige Kachel (`lblKpiCriticalValue`/`Label`, `lblKpiWarningsValue`/`Label`, `lblKpiAgentsValue`/`Label` – alle mit neuem `OnSelect`) bestätigt, wodurch die Baseline auf den dann aktuellen Wert gesetzt wird.
- **Bekannte Einschränkung (transparent, da SaveData/LoadData in Canvas-Apps im Browser laut Microsoft-Dokumentation NICHT unterstützt werden):** Die Bestätigung gilt nur für die laufende Browser-Sitzung – nach einem Neuladen der Seite wird die Baseline beim ersten Refresh neu auf den dann aktuellen Stand gesetzt (keine LEDs beim Neuladen). Eine geräte-/sitzungsübergreifende Bestätigung wäre nur mit einer neuen SharePoint-Zeile pro Nutzer realisierbar (deutlich höherer Aufwand) – auf Wunsch als späterer Ausbauschritt möglich.
- `App.pa.yaml`: neue Init-Variablen `varLastAckCriticalCount`/`varLastAckWarningCount`/`varLastAckAgentsHealthyCount`/`varBaselineAcknowledged`.

**Release Notes:** Neuer Eintrag `v1.7.0 – 2026-08-28` in `scrReleaseNotes.pa.yaml` ergänzt (kein Screenshot, da vom Nutzer keiner mitgeliefert wurde für diesen Stand).

**Deployment:** Alle 9 geänderten/neuen `.pa.yaml`-Dateien einzeln auf Mojibake geprüft (0 Treffer) und per `pac canvas pack`/`unpack`-Round-Trip verifiziert (0 Diff je Datei, in einem separaten Testordner, NICHT im echten `Source`-Ordner). Anschließend das echte `DMP_COMMAND_Solution.msapp` gepackt (vorher Sicherheitskopie `DMP_COMMAND_Solution.backup-before-v1.7.0.msapp` angelegt). PowerApp-Version auf `v1.7.0` erhöht.

**Noch offen:**
- Nutzer muss `DMP_COMMAND_Solution.msapp` in Power Apps Studio öffnen, prüfen und veröffentlichen (Pflichtschritt, nicht automatisierbar).
- SharePoint-Konfigurationswert für die angezeigte App-Version auf `v1.7.0` aktualisieren.
- Phase 2/3 des Nutzerkonzepts (Admin-Freigabe, rollenbasierte Menüpunkte) weiterhin offen, siehe Konzeptvorschlag unten.
- 100%-Zoom-Breitenproblem weiterhin offen (siehe eigener Abschnitt unten) – blockiert auf Viewport-Breite vom Nutzer.

---

## 🆕 KONZEPTVORSCHLAG (2026-08-28, auf Nutzeranfrage ausgearbeitet, NICHT implementiert): Rollenbasierte Sidebar-Navigation + Admin-Freigabe + Admin-Funktionen als eigene Seite

**Anlass:** Nutzer möchte (a) die Admin-Funktionen als eigene Seite statt als Overlay über dem Cockpit, und (b) ein Nutzerkonzept, bei dem die linken Sidebar-Menüpunkte nutzerspezifisch konfiguriert werden und ein Admin-User die Nutzung freischalten muss. **Ausdrücklich bestätigt (Nutzerfrage 2026-08-28): Dies betrifft NICHT nur Admin Functions, sondern JEDEN der 5 aktuell noch als Platzhalter existierenden Menüpunkte** (Agent Monitoring, Operational Board, Audit Trail, Configuration, Maintenance) – jeder von ihnen braucht langfristig eine eigene, echte App-Seite statt eines "coming next"-Platzhalters.

**Ist-Zustand geprüft:** Von den 7 Sidebar-Knöpfen (`btnNavCockpit`, `btnNavAgentMonitoring`, `btnNavOperationalBoard`, `btnNavAuditTrail`, `btnNavConfiguration`, `btnNavMaintenance`, `btnNavAdminFunctions`) sind aktuell nur **Cockpit** (aktive Seite) und **Admin Functions** (togglet ein Overlay `conAdminFunctions` über `varShowAdminPanel`) tatsächlich funktional – die anderen 5 zeigen nur `Notify("... screen - coming next")`. Es existiert bislang nur EIN echter Screen (`scrHome`), alles läuft in einem einzigen `.pa.yaml`.

### Empfohlenes 3-Phasen-Vorgehen (kleine, risikoarme Schritte statt Großumbau in einem Zug)

**Phase 1 – JEDER Menüpunkt bekommt eine echte, eigene Seite (sofort umsetzbar, kein neues Datenmodell nötig):**
- Für Admin Functions: Neue Datei `scrAdminFunctions.pa.yaml` anlegen (gleiches Schema wie `scrHome.pa.yaml`), Inhalt von `conAdminFunctions` dorthin verschieben; `btnNavAdminFunctions.OnSelect` von `Set(varShowAdminPanel,!varShowAdminPanel)` auf `Navigate(scrAdminFunctions, ScreenTransition.None)` ändern.
- Für die 5 übrigen Platzhalter-Menüpunkte (Agent Monitoring, Operational Board, Audit Trail, Configuration, Maintenance): analog je eine eigene, zunächst noch inhaltsleere Screen-Datei anlegen (`scrAgentMonitoring.pa.yaml` usw.) mit Titel + "Coming Soon"-Hinweis, `OnSelect` der jeweiligen Knöpfe auf echtes `Navigate(...)` umstellen (statt der bisherigen `Notify(...)`-Platzhaltermeldung). Der tatsächliche fachliche Inhalt jeder Seite wird dann schrittweise, Seite für Seite, in eigenen späteren Arbeitsschritten gefüllt (nicht alle auf einmal).
- Jede neue Seite bekommt einen "← Zurück zum Cockpit"-Knopf mit `Navigate(scrHome, ScreenTransition.None)`.
- **Aufwand:** klein bis mittel (6 neue, zunächst einfache Screen-Dateien + Umstellung von 6 `OnSelect`-Formeln), **Risiko:** gering (reine Struktur-/Navigations-Änderung, keine bestehende Logik wird verändert). Kann unabhängig von Phase 2/3 sofort umgesetzt werden.

**Phase 2 – Einfache Admin-Freischaltung (nur EIN Recht: "ist Admin ja/nein"):**
- Neue SharePoint-Liste `DMP Command User Roles` mit Spalten: `UserPrincipalName` (Text/Person), `DisplayName`, `IsAdmin` (Ja/Nein), `ApprovalStatus` (Wahl: Pending/Approved/Revoked), `ApprovedBy`, `ApprovedDateUtc`, `Notes`.
- Bei `App.OnStart`: `LookUp('DMP Command User Roles', UserPrincipalName = User().Email)` in `varCurrentUserRole` puffern.
- `btnNavAdminFunctions.Visible` und der Zugriff auf `scrAdminFunctions` an `varCurrentUserRole.IsAdmin = true UND ApprovalStatus = "Approved"` knüpfen.
- Ist der Nutzer nicht in der Liste oder nicht freigegeben: normales Cockpit bleibt nutzbar (read-only Überwachung), nur der Admin-Bereich bleibt verborgen/gesperrt – **kein Vollsperren der App**, um den produktiven Betrieb nicht zu gefährden.
- **Aufwand:** mittel (1 neue Liste + wenige neue Formeln), **Risiko:** gering, da additiv (bestehende Funktionen bleiben unangetastet).

**Phase 3 – Volles Nutzerkonzept (nutzerspezifische Menüpunkte, Selbst-Registrierung, Freigabe-Workflow):**
- `DMP Command User Roles` um Spalte `VisibleMenuItems` erweitern (Mehrfachauswahl-Feld mit den 7 Menüpunkt-Schlüsseln, z. B. `Cockpit;AgentMonitoring;AuditTrail`).
- Jeder Sidebar-Knopf bekommt `Visible: =varCurrentUserRole.VisibleMenuItems... enthält diesen Schlüssel`.
- Neuer Abschnitt "User Management" im Admin-Bereich (jetzt auf eigener Seite, siehe Phase 1): Tabelle aller Nutzer mit Status Pending/Approved/Revoked, Buttons zum Freigeben/Entziehen, Dropdown zur Menüpunkt-Zuweisung pro Nutzer.
- Optional: Beim ersten Login eines unbekannten Nutzers automatisch einen "Pending"-Eintrag anlegen (per Flow oder direktem `Patch(...)` in `App.OnStart`), damit der Admin nur noch freigeben muss, statt Nutzer manuell anzulegen.
- **Aufwand:** höher (neue UI-Tabelle, Freigabe-Interaktionen, ggf. ein kleiner Flow für die Auto-Registrierung), **Risiko:** moderat – braucht sorgfältiges Testen der Sichtbarkeits-Formeln, damit kein Nutzer versehentlich ausgesperrt wird.

**Offene Entscheidungen für den Nutzer (bitte vor Umsetzungsbeginn klären):**
1. Reicht für den Start Phase 2 (nur Admin ja/nein), oder soll direkt das volle Phase-3-Konzept (individuelle Menüpunkte pro Nutzer) umgesetzt werden?
2. Sollen unbekannte Nutzer automatisch einen Zugriffsantrag auslösen (Selbstregistrierung mit Wartestatus), oder legt der Admin neue Nutzer ausschließlich manuell in der Liste an (einfacher, aber weniger komfortabel)?
3. Soll ein Nutzer OHNE Freigabe das Cockpit weiterhin (read-only) sehen dürfen, oder soll die gesamte App inklusive Cockpit gesperrt werden, bis ein Admin freigibt?

**Empfehlung:** Mit Phase 1 sofort beginnen (unabhängig, geringes Risiko, erfüllt die "eigene Seite"-Anforderung direkt). Phase 2 danach als nächsten Schritt, Phase 3 erst nach Rücksprache mit dem Chef nächste Woche (passt zeitlich zur ohnehin geplanten Besprechung der "NEXT STEPS"-Aktionsliste).

---

## ✅ NEU UMGESETZT (2026-08-28): Release-Notes-Seite (inkl. Screenshot) + Versionsanzeige als Knopf

**Anlass:** Nutzer wünschte eine Seite mit Release Notes, die bei jeder neuen Version aktualisiert wird, erreichbar über einen Knopf anstelle der reinen Versionsanzeige. Zusätzlich: jeder Release-Notes-Eintrag soll idealerweise einen Screenshot enthalten, damit künftig exakt auf einen bestimmten Versionsstand zurückreferenziert werden kann.

**Umsetzung:**
- Neue Datei `scrReleaseNotes.pa.yaml` angelegt – der erste tatsächlich umgesetzte "eigene Screen" gemäß Phase-1-Konzept oben (Muster: Header mit "< Back to Cockpit"-Knopf + Titel, darunter eine Liste von Versions-Karten).
- `lblVersionTag` (bisher reine Anzeige "v1.6.4") in `scrHome.pa.yaml` um `OnSelect: =Navigate(scrReleaseNotes, ScreenTransition.Fade)` sowie `HoverFill` ergänzt – fungiert jetzt als Knopf, ohne das bestehende Pill-Badge-Design zu verändern.
- Erster Eintrag (`v1.6.4 - 2026-08-28`) enthält vollständige Frontend-/Backend-Änderungsliste dieser Sitzung PLUS einen eingebetteten Screenshot (vom Nutzer bereitgestellt, auf 900×446px verkleinert, als `data:image/png;base64,...` direkt im `Image`-Control – gleiches Muster wie das bestehende Sidebar-Logo). Ältere Stände (`v1.6.1–v1.6.3`, `v1.5.x und früher`) als kompaktere Text-Karten ohne Screenshot (rückwirkend nicht mehr exakt rekonstruierbar).
- **Wichtiger technischer Hinweis für künftige Einträge:** Screenshots werden NUR eingebettet, wenn der Nutzer zum jeweiligen Versionsstand tatsächlich einen Screenshot mitliefert (kein automatisches Erstellen durch die KI). Aus Größengründen (jedes eingebettete Bild vergrößert die `.msapp` um ca. 150–250 KB) wird empfohlen, dies nur bei größeren/sichtbaren Versionssprüngen zu tun, nicht bei jedem Patch.
- Alles ausschließlich mit dem `create`/`edit`-Tool bzw. sicherer `.NET`-Kodierung erzeugt (kein `Get-Content`/`Set-Content`), 0 Mojibake, Pack+Unpack-Round-Trip beider betroffener Dateien (`scrHome.pa.yaml`, `scrReleaseNotes.pa.yaml`) verifiziert (0 Diff).
- **Wichtiger Zwischenfall (selbst verursacht, sofort korrigiert):** Beim ersten Round-Trip-Test wurde versehentlich der komplette Inhalt von `Source\Src\` (außer `App.pa.yaml`) über `Remove-Item -Recurse` gelöscht (inkl. `scrHome.pa.yaml`, `scrHeaderTest.pa.yaml`, `Components\Component1.pa.yaml`). Da das Verzeichnis ein Git-Repository ist, sofort über `git checkout -- <Pfade>` folgenlos wiederhergestellt (bestätigt: `git status` danach clean bis auf die neue, gewollte Datei). **Lehre:** Test-Unpacks künftig IMMER in einen separaten Zielordner schreiben und niemals vorher destruktiv in den echten Quellordner hinein aufräumen.
- PowerApp-Versionsdatei (`PowerApp_Version.txt`) auf `v1.6.5` erhöht. **Hinweis:** Die tatsächlich in der App angezeigte Version (`varAppVersion`) kommt zur Laufzeit aus der SharePoint-Konfiguration (über Agent 4), nicht aus einem Hardcode in `App.pa.yaml` – der Nutzer muss den entsprechenden Konfigurationswert nach dem Publizieren manuell auf `v1.6.5` setzen, damit Anzeige und Release-Notes-Historie zueinander passen.

**Noch offen:** Der Nutzer muss die App in Power Apps Studio öffnen/speichern/publizieren, damit `scrReleaseNotes` und der Versions-Knopf live nutzbar werden (wie bei jeder `.msapp`-Änderung nicht durch die KI automatisierbar).

---

## 🔴 OFFEN (2026-08-28): Layout passt nur bei Browser-Zoom 75%, nicht bei 100% (Standard)

**Nutzeranweisung (verbindlich):** Alle Breiten-/Layoutberechnungen sind ab sofort für Browser-Zoom 100% auszulegen (siehe neue Regel in `DMP COMMAND_Mission_und_KI_Arbeitsregeln.md`).

**Analyse (Breitenbudget von `conRow1` in `scrHome.pa.yaml`):**
- Sidebar: 300px (fix, `LayoutMinWidth`/`LayoutMaxWidth`)
- `conMain`-Padding: 2 × 20px = 40px
- `conRow1`-Inhalt: `conHeartbeatCard` 376px + `conMiddleColumn` 1186px + `conEmailsCard` 560px + 2 × 16px Gap = 2154px
- **Gesamt-Mindestbreite aktuell: 2494px** bei Zoom 100%, damit nichts abgeschnitten wird oder gescrollt werden muss.

**Blockiert durch fehlende Information:** Um eine seriöse (nicht geratene) Empfehlung zu geben, wird die tatsächliche Viewport-Breite des Nutzers bei 100% Zoom benötigt (Bildschirmauflösung + Windows-Anzeigeskalierung, oder direkt `window.innerWidth` aus der Browser-Konsole, F12). Ohne diesen Wert lässt sich nicht seriös sagen, wie viele Pixel eingespart werden müssen, ohne erneut Texte abzuschneiden (siehe bereits zweimal aufgetretene Regression durch proportionale Verkleinerung).

**Nächster Schritt sobald Wert bekannt:** Zielbreite = gemeldeter Viewport – kleiner Sicherheitsabstand (z. B. 20px für Scrollbar). Fehlbetrag wird NICHT proportional über alle Container verteilt (siehe Regel zu proportionaler Skalierung), sondern gezielt an den unkritischsten Stellen eingespart (Kandidaten: `conEmailsCard` weiter verschmälern, Legenden-Labels, ggf. `conHeartbeatCard`-Padding), einzeln geprüft auf Textabschneidung.

---

## ✅ TEILWEISE UMGESETZT (2026-08-28, Fortsetzung): Logo vergrößert, Emails-Karte verkleinert, Next-Steps-Umbruch behoben

**Logo:** Sidebar-Padding von 10/20 auf 6/12 reduziert (Innenbreite 280→288px), Logo von 280×188 auf 288×194 vergrößert (füllt jetzt exakt die neue Sidebar-Innenbreite = Breite der Menüpunkte, Seitenverhältnis 1.485:1 beibehalten).

**Emails-Karte (Agent 2 Status):** Von 620px auf 560px verkleinert (−60px/≈1.6cm, etwas weniger als die gewünschten 2cm/76px, um Abschneiden der Legenden-Texte zu vermeiden). Ring bleibt bei 328px (unverändert, weiterhin gleiche Größe wie System Health). Alle 4 Legenden-Labels (No DMP/External/Internal/Not Effected) näher an den Ring gerückt (X 372→356) und Breite leicht reduziert (224→192px) – ausreichend für Texte wie "Not Effected (14, 100%)" ohne Kürzung.

**Next Steps – Zeilenumbruch behoben:** Alle 5 Meilenstein-Label (`lblMilestone1`–`5`) von Breite 240→340px verbreitert und `Wrap: =false` ergänzt, damit z. B. "Pre-Default communication assessment" nicht mehr in 2 Zeilen umbricht.

**Noch offen (bewusst zurückgestellt):**
- `conMaintenanceDomains` (620px) vs. `conOperatingState` (550px) Breiten-Angleichung – NICHT umgesetzt in dieser Runde (Replace-Button/LED-Koordination zu riskant für eine schnelle Änderung, siehe Analyse oben). Zieht weiterhin die im vorherigen Eintrag dokumentierten Werte.
- Replace-Button-Fehler weiterhin ungeklärt (braucht Live-Diagnose).
- Admin-Funktionen als eigene Seite (nicht Overlay) – vom Nutzer erneut bestätigt als offener Punkt, noch nicht begonnen.
- Nutzerkonzept (rollenbasierte Sidebar-Menüs + Admin-Freigabe) – Vorschlag noch auszuarbeiten.

**Deployment:** Alle 12 Container verifiziert, 0 Mojibake-Zeichen (voller Zeichen-Scan), Round-Trip 0 Diff. PowerApp `v1.6.3→v1.6.4`.

---

## 🔴 OFFEN (2026-08-28, Ende Sitzung wegen Token-Limit): Replace-Button weiterhin defekt + Layout-Wünsche nicht umgesetzt

**Replace-Button (conMaintenanceDomains) öffnet den Picker weiterhin NICHT**, auch nach Bewegen der Laufleiste. Rechnerisch (X/Y/Breite aller Elemente in `conReplaceStack`, `dotLedEmergencyReport`, `lblExternalDomainsCount`) wurde KEINE Überlappung/Z-Order-Kollision gefunden – die Ursache liegt vermutlich tiefer (z. B. Attachments-Control-Verhalten bei `Fill`/`Color`-Opacity 0.01, oder ein Effekt, der nur live in Studio sichtbar ist). **Braucht eine Live-Diagnose in Studio durch den Nutzer** (z. B. Control-Baum-Inspektion, welches Element den Klick tatsächlich empfängt), bevor blind weiter an Koordinaten geschraubt wird – weiteres Raten ohne Live-Feedback verbrennt nur Tokens ohne Erfolgsgarantie.

**Noch nicht umgesetzte Nutzerwünsche aus der letzten Nachricht (2026-08-28, Layout-Feedback bei 75%-Zoom-Screenshot):**
1. Bildschirm nur bei 75% Zoom vollständig sichtbar (vorher 100%) → Gesamtbreite der oberen Kachelzeile muss weiter reduziert werden.
2. `conMaintenanceDomains` (620px) soll auf Breite von `conOperatingState` (550px) angeglichen/verkleinert werden. **Geplant (nicht umgesetzt):** Card auf 550px, `lblInternalDomainsCount` X460→390, `dotLedEmergencyReport` unverändert X458 (conReplaceStack NICHT anfassen wegen Regressionsrisiko), `lblExternalDomainsCount` X480→470/Breite124→72.
3. `conEmailsCard` (Agent 2 Status) um ca. 2cm (~76px) schmaler, z. B. 620→560px, Legende enger an den Ring (X372→356, Breite224→196).
4. Logo oben links weiter vergrößern – Zielbreite = Breite der Sidebar-Menüpunkte (aktuell Sidebar-Innenbreite ca. 280px nach Padding), aktuell Logo nur 280×188.
5. „NEXT STEPS": erster Eintrag „Pre-Default communication assessment" bricht in 2 Zeilen um – Spalte verbreitern statt Zeilenumbruch.
6. **Zukünftiges großes Feature (nicht jetzt, erst nach Chef-Gespräch nächste Woche):** „NEXT STEPS" soll zu einer verlinkten Aktionsliste werden, in der jeder Prozessschritt weitere Aktionen auslösen kann.
7. **Neues Nutzerkonzept angefragt (Vorschläge erbeten, noch nicht ausgearbeitet):** Linke Sidebar-Menüpunkte sollen nutzerspezifisch konfigurierbar sein; Freigabe der Nutzung durch einen Admin-User, der auch Admin-Funktionen ausführen darf. **TODO nächste Sitzung:** Konzeptvorschlag ausarbeiten (z. B. neue SharePoint-Liste `DMP Command User Roles` mit Spalten User/Rolle/sichtbare Menüpunkte/AdminApproved, Admin-Bereich-Erweiterung um Nutzerverwaltung).

**Nächste Sitzung: Zuerst Replace-Button-Fehler mit Live-Diagnose klären, dann Punkte 2-5 (Layout) einzeln und vorsichtig umsetzen (KEINE pauschale Skalierung mehr, nur einzeln geprüfte Werte – siehe Arbeitsregeln-Lektion von heute), danach Nutzerkonzept-Vorschlag ausarbeiten.**

---



**Nutzer-Feedback:** Nach der proportionalen Verkleinerung von Operating State/Maintenance Domains/Files waren Labels abgeschnitten ("Mo…" statt "Mode", "Operational M…" statt "Operational Mode", Zeitstempel nur noch "C…" statt "CEST") UND der Replace-Button öffnete den Datei-Picker gar nicht mehr, auch nicht nach Bewegen der Laufleiste.

**Root Cause:** Die proportionale Skalierung (Faktor auf alle X/Width-Werte) funktioniert für reine Layout-Boxen, aber NICHT für Text-tragende Labels – die Schriftgröße blieb gleich, während die Spaltenbreite schrumpfte, was zu Abschneidungen führte. Gleichzeitig wurden die absoluten Positionen von `attExternalDomainsReplace`/`dotLedEmergencyReport`/`conReplaceStack` innerhalb von `conMaintenanceDomains` mitskaliert, was die zuvor korrekt austarierte Replace-Anhang-Steuerung erneut verschob/verkleinerte und den Klickbereich zerstörte.

**Fix:** `conOperatingState` (550px), `conMaintenanceDomains` (620px), `conFilesRow` (550px), `conFilesPlaceholder` (620px) sowie `conMiddleColumn` (1186px) vollständig auf den Stand VOR der Skalierung zurückgesetzt (per `git show` aus dem Commit vor der Skalierung extrahiert und gezielt zurückgespielt, NICHT durch erneutes manuelles Nachrechnen – vermeidet Rundungsfehler). `conEmailsCard` bleibt wie gewünscht das 3. Kind von `conRow1` (oben rechts), nur die Breiten der anderen drei Container sind wieder original. **Nebenwirkung:** Die zurückgeholten Blöcke brachten die alte (bereits einmal behobene) Kodierungsbeschädigung aus dem Vor-Fix-Commit wieder mit (`Â·`/`â€¦`) – gezielt nur in den betroffenen Zeilen per String-Replace erneut korrigiert, ohne den Rest der Datei zu berühren.

**Bekannter, unveränderter Nebeneffekt:** Mit den Original-Breiten ist die Gesamtbreite der Kopfzeile wieder ca. 2554px (wie vor der gestrigen Verkleinerung) – die Emails-Karte kann daher auf schmaleren Bildschirmen erneut ganz oder teilweise abgeschnitten sein. Dies ist ein bewusster Kompromiss (Lesbarkeit/Funktionalität vor Perfektion der Positionierung) – **falls die Emails-Karte weiterhin abgeschnitten wird, braucht es einen gezielteren Ansatz** (z. B. Legende unter statt neben dem Ring, oder Nutzer-Feedback zur tatsächlich verfügbaren Bildschirmbreite), NICHT wieder eine pauschale Skalierung.

**Neue Erkenntnis für die Arbeitsregeln:** Proportionale Skalierung von Containerbreiten ist NUR für reine Geometrie-Container sicher, NIEMALS für Container mit Text-Labels/Werten – dort immer nur gezielt einzelne, nicht-kritische Elemente anpassen oder Schriftgröße mit anpassen.

**Deployment:** Alle 12 Container erneut verifiziert (je 1×), Kodierung sauber (vollständiger Scan über die GESAMTE Datei auf alle Â/â/Ã-Vorkommen: 0 Treffer – dabei wurden inzident auch zwei zuvor UNENTDECKTE Mojibake-Korruptionen behoben, die meine bisherige Prüfliste nicht abdeckte: „●" (Bullet) war zu „â—" und „–" (Gedankenstrich) zu „â€“" korrumpiert, beide lagen zufällig innerhalb des wiederhergestellten `conFilesRow`-Blocks), Round-Trip 0 Diff. PowerApp `v1.6.1→v1.6.3` (v1.6.2 war bereits als Zwischenstand vergeben, aber nie committed worden – dieser Fix inkl. der zusätzlich gefundenen Korruption wird als v1.6.3 final deployt).

---

## ✅ GELÖST (2026-08-28, Fortsetzung): Emails-Karte zurück in die Kopfzeile (oben rechts), Kodierungsbeschädigung behoben

**Nutzerwunsch:** Die "Emails Processed" (Agent 2)-Karte soll NICHT in einer eigenen Zeile darunter, sondern oben rechts in derselben Zeile wie System Health und Operating State/Maintenance Domains erscheinen. Dafür sollten `conOperatingState`, `conMaintenanceDomains` und `conFilesRow`/`conFilesPlaceholder` schmaler werden.

**Umsetzung:** `conRow2` (die gestern eingeführte Verschiebung wegen Bildschirmbreite) wieder aufgelöst; `conEmailsCard` ist jetzt wieder das 3. Kind von `conRow1`. Um Platz zu schaffen, wurden `conOperatingState` (550→420px) und `conMaintenanceDomains` (620→480px) proportional verkleinert (Skalierungsfaktor auf alle internen Kind-Positionen/-Breiten angewendet, um Überlappungen zu vermeiden), `conFilesRow` passend mitverkleinert (550→420px, gleiche Spaltenbreite wie Operating State für optische Flucht), `conFilesPlaceholder` ebenfalls (620→480px). `conMiddleColumn` dadurch 1186→916px schmaler. Neue Gesamtbreite der Kopfzeile: 376(Herzschlag)+16+916(Mittelspalte)+16+620(Emails) = 1944px zzgl. 340px Sidebar/Padding = 2284px (vorher 2554px, ca. 270px/11% schmaler).

**🔴 Kritischer Nebenfund während der Umsetzung – Datei zweimal durch PowerShell-Kodierungsfehler beschädigt:** Beim Verschieben/Skalieren der YAML-Blöcke wurde `Get-Content`/`Set-Content -Encoding UTF8` verwendet, was Mehrbyte-UTF-8-Sonderzeichen (·, …, ⟳, ✓) durch Doppel-Encoding zerstörte (z. B. „·" → „Â·", sichtbar als "Loadingâ€¦" im Screenshot des Nutzers). Dies geschah ZWEIMAL hintereinander (einmal bei der gestrigen `conRow2`-Verschiebung – seitdem bereits live und committed fehlerhaft –, einmal erneut beim heutigen Skalierungsschritt). Mit `[System.IO.File]::ReadAllText/WriteAllText` + `Windows-1252`→`UTF-8`-Rückkodierung korrigiert (nachweislich: alle 4 bekannten Mojibake-Muster auf 0 reduziert, korrekte Zeichen wiederhergestellt). **Neue Hart-Regel in den Arbeitsregeln verankert:** Für PowerShell-Dateizugriffe auf `.pa.yaml` ab sofort ausschließlich `[System.IO.File]::ReadAllText/ReadAllLines` mit `[System.Text.Encoding]::UTF8` bzw. `WriteAllText/WriteAllLines` mit `New-Object System.Text.UTF8Encoding($false)` verwenden – niemals `Get-Content`/`Set-Content`/`Out-File`, auch nicht mit explizitem `-Encoding UTF8`-Parameter.

**Deployment:** Container-Anzahl vor/nach allen Splice-Operationen verifiziert (alle 12 erwarteten Container – 11 Hauptcontainer + `conRow1`, `conRow2` jetzt 0 – exakt wie erwartet), Kodierung nach jedem PowerShell-Schritt stichprobenartig geprüft, Round-Trip verifiziert (0 Diff). PowerApp `v1.6.0→v1.6.1`.

**Noch zu beobachten:** Ob 2284px Gesamtbreite auf dem tatsächlichen Bildschirm des Nutzers ausreicht – falls die Emails-Karte immer noch abgeschnitten wird, muss weiter verkleinert werden (Rückmeldung mit Screenshot erforderlich).

---

## ✅ GELÖST (2026-08-28, Fortsetzung): Echter Copy-Paste-Bug in `Move_Email_(Audit_Write_Failed)` (Agent 1) behoben

**Fund:** Nach der Fehler-Kennung `[EC:A1-AUDITWRITE]` (siehe unten) trat ein neuer Fehler auf: *"MethodNotAllowed"* bei `Move_Email_(Audit_Write_Failed)`, mit der Roh-URL `v1.0/users/default@eurex.com/messages//move` – der **doppelte Schrägstrich** zeigt eine leere Message-ID an.

**Root Cause (echter, vermutlich schon lange bestehender Bug, nicht durch heutige Änderungen verursacht):** Die Aktion referenzierte fälschlich `outputs('Compose_Sent_Message_ID_(Failed_Domains_Extraction_-_No_Domains_Found)')` statt der korrekten `outputs('Compose_Sent_Message_ID_(Audit_Write_Failed)')` – ein klassischer Copy-Paste-Fehler: Die Aktion wurde offensichtlich aus dem strukturell identischen "No Domains Found"-Zweig kopiert (im Code direkt davor), aber die Objekt-Referenz wurde nie auf den neuen Zweig angepasst. Da `Compose_Sent_Message_ID_(Failed_Domains_Extraction_-_No_Domains_Found)` in diesem Ausführungspfad nie lief, wertete Power Automate den Verweis als leeren String aus (kein Build-Fehler, da syntaktisch gültig) – was den Move-Aufruf mit leerer ID und somit ungültiger URL auslöste.

**Systematische Prüfung auf denselben Fehlertyp:** Alle 6 `Move_Email_(...)`-Aktionen in Agent 1 einzeln gegen ihre jeweils zugehörige `Compose_Sent_Message_ID_(...)`-Aktion abgeglichen – nur DIESE eine war betroffen, die anderen 5 (`Global_Fail`, `Write_Domains_File_Failed`, `Write_Domains_File_Succeeded`, `Failed_Domains_Extraction_-_Technical_Error`, `Failed_Domains_Extraction_-_No_Domains_Found`) referenzieren korrekt ihre eigene Compose-Aktion. Agent 2 ebenfalls komplett durchgeprüft (alle 15 `Move_Email_(...)`-Aktionen) – dort ist alles korrekt. Agent 3/4/5 nutzen ein strukturell anderes, für diesen Fehlertyp nicht anfälliges Muster (eine gemeinsame `variables('AlertMessageId')`, direkt vor jedem Move gesetzt, statt einzelner Compose-Aktionen pro Zweig).

**Fix:** Einzeiliger Verweis-Tausch in `Move_Email_(Audit_Write_Failed)`'s `Uri`-Formel.

**Versionen:** Agent 1 `[1.0.6]→[1.0.7]`, Solution `7.11.31→7.11.32`.

**Deployment:** Alle 6 Agenten vor dem Pack geprüft (JSON gültig, Beschreibungslängen ≤255), Solution neu gepackt und **erfolgreich importiert** – Agent 1 wurde dabei deaktiviert und **muss vom Nutzer manuell reaktiviert werden**.

---

## ✅ GELÖST (2026-08-28, Fortsetzung): ECHTER Syntax-Bug in der Zähler-Bündelung behoben, Fehler-Kennung auf alle Agenten ausgeweitet, `conEmailsCard` aus dem sichtbaren Bereich gerutscht

**🔴 ECHTER BUG – `filter(variables(...), equals(item()?[...]))` ist in klassischem Workflow-Ausdruck außerhalb von `Query`/`Foreach` NICHT gültig:** Beim Live-Test meldete Agent 1 (und wäre bei Agent 2 identisch aufgetreten): *"Unable to process template language expressions in action 'SET_SucceededStepsDelta'..."*. `item()` ist nur innerhalb eines `Query`- (Filter-Array-) oder `Foreach`/`Until`-Aktionskontexts definiert, nicht als generische Inline-Funktion in einem `SetVariable`-Wert – ein Fehler meinerseits bei der gestrigen Zähler-Bündelungs-Optimierung, der erst durch den echten Live-Test sichtbar wurde (das bereits im Code vorhandene Vorbild `FILTER_Config_CurrentOperationMode` nutzt korrekt den `Query`-Aktionstyp, das hätte mir auffallen müssen). **Fix (Agent 1 UND Agent 2):** Je 3 neue `Query`-Aktionen (`QUERY_SucceededEvents`/`QUERY_WarningEvents`/`QUERY_FailedEvents`, `from`/`where` wie beim bestehenden Vorbild) vor die jeweilige `SET_...StepsDelta`-Aktion gehängt; die `SetVariable`-Werte lauten jetzt schlicht `length(body('QUERY_...'))` statt der fehlerhaften Inline-`filter()`-Formel. Kein API-Aufruf zusätzlich nötig (Query-Aktionen laufen rein im Flow-Speicher, kein Excel/SharePoint-Zugriff).

**🆕 Fehler-Kennung auf alle Agenten mit E-Mail-Versand ausgeweitet** (Pilot war gestern nur Agent 1): Agent 2 (15 Alarm-Mails), Agent 3 (5), Agent 4 (1), Agent 5 (2) – Agent 6 hat keine E-Mail-Versandlogik, daher keine Kennungen nötig. Insgesamt 29 eindeutige Codes nach dem Schema `[EC:A<Agentennummer>-<KÜRZEL>]`, jeweils vor `[RID:...]` bzw. ans Ende der Betreffzeile angehängt (bei Agent 3/4/5, die kein `[RID:...]` im Betreff führen). Vollständige Liste (29 Codes) im Code dokumentiert über die jeweiligen `Compose_Subject_(...)`/`AlertMailSubject`-Aktionen.

**🔴 ECHTER BUG – `conEmailsCard` (Agent 2 Ring) aus dem sichtbaren Bereich gerutscht, NICHT gelöscht:** Nutzer bestätigte, dass die App-Version `v1.5.9` korrekt live war, der Container aber trotzdem fehlte – anders als der ähnliche Vorfall vom Vortag (dort war der Container tatsächlich versehentlich gelöscht). Analyse ergab: `conRow1` (Horizontal-AutoLayout) enthielt `conHeartbeatCard`(376px) + `conMiddleColumn`(1186px) + `conEmailsCard`(620px) mit je 16px Abstand = **2214px** Gesamtbreite, zzgl. 300px Sidebar und 40px Padding = **2554px** insgesamt benötigt – das übersteigt jede normale Bildschirmbreite deutlich, sodass das zuletzt platzierte `conEmailsCard` rechts aus dem sichtbaren Bereich hinausragte (kein Lösch-Bug, sondern ein Platzierungs-/Breiten-Problem). **Fix:** `conEmailsCard` aus `conRow1` in eine neue eigene Zeile `conRow2` darunter verschoben (reiner Orts-/Struktur-Wechsel, Inhalt unverändert) – `conRow1` benötigt jetzt nur noch 376+16+1186=1578px, `conRow2` enthält nur noch die 620px breite Emails-Karte. Container-Anzahl vor/nach Verschiebung verifiziert (alle 13 erwarteten Container – die 11 Hauptcontainer plus `conRow1`/`conRow2` – exakt einmal vorhanden), Round-Trip-Verifikation bestanden (0 Diff).

**Noch zu beobachten:** Ob `conRow2` mit 620px auf allen üblichen Bildschirmbreiten sichtbar ist (in Kombination mit der 300px-Sidebar ergibt das ca. 920-960px Mindestbreite für diese Zeile, was deutlich unproblematischer ist als vorher).

**Versionen:** Agent 1 `[1.0.5]→[1.0.6]`, Agent 2 `[1.0.5]→[1.0.6]`, Agent 3 `[1.1.3]→[1.1.4]`, Agent 4 `[1.2.8]→[1.2.9]`, Agent 5 `[1.1.5]→[1.1.6]`, Solution `7.10.35→7.11.31`, Power App `v1.5.9→v1.6.0`.

**Deployment:** Alle 6 Agenten-JSONs vor dem Pack vollständig geprüft (JSON gültig, keine verbliebenen fehlerhaften `filter()`-Aufrufe, alle Beschreibungen ≤255 Zeichen, neue Query-Aktionen korrekt referenziert). Solution neu gepackt und **erfolgreich importiert** – alle 5 geänderten Agenten (1, 2, 3, 4, 5) wurden dabei deaktiviert und **müssen vom Nutzer manuell reaktiviert werden**. Power App neu gepackt, Round-Trip verifiziert (0 Diff).

---

## ✅ GELÖST (2026-08-28): Retry-Policies für Audit-Trail-Excel-Aufrufe (Agent 1+2) + Fehler-Kennungs-Schema (Pilot: Agent 1)

**Anlass:** Während einer laufenden Simulation ("FIRE DRILL") sendete Agent 1 die Fehlermail "Audit Trail Write Failed". Die genaue innere Fehlerursache des einzelnen Laufs konnte nicht direkt eingesehen werden (kein Zugriff auf den Power-Automate-Run-Verlauf), aber die neue Zähler-Bündelungslogik von gestern wurde vollständig nachgeprüft und ist strukturell korrekt.

**Ausfallsicherheit (Agent 1 + Agent 2):** Die 4 kritischen Excel-Online-Business-Aufrufe in `Audit_Trail_Processing` (RunSummary schreiben, kombinierten Zähler lesen, Events schreiben, kombinierten Zähler zurückschreiben) hatten bislang KEINE Retry-Policy – bei einer kurzen Datei-Sperre durch einen gleichzeitig laufenden anderen Agenten (plausibel bei einem Fire-Drill-Test mit mehreren aktiven Agenten) schlägt der Aufruf dadurch sofort und endgültig fehl. Fix: `retryPolicy: {type: fixed, count: 4, interval: PT10S}` auf allen 4 Aktionen in beiden Agenten ergänzt – bei einer transienten Sperre wird jetzt bis zu 4× im Abstand von 10s automatisch erneut versucht, bevor der Lauf wirklich als fehlgeschlagen gilt.

**🆕 Neues Feature (Nutzeridee, Pilot in Agent 1 umgesetzt): Eindeutige Fehler-Kennung pro Alarm-Mail.** Damit bei künftigen Fehlermails sofort erkennbar ist, an welcher Stelle im Flow der Fehler entstanden ist, wird jeder Alarm-Mail-Betreff um einen kurzen, festen Code `[EC:A<Agentennummer>-<KÜRZEL>]` ergänzt, z. B. `[EC:A1-AUDITWRITE]`. Für Agent 1 umgesetzt (alle 6 echten Fehler-Mails, die reine Erfolgsmeldung "Write_Domains_File_Succeeded" bewusst ausgenommen):
- `A1-GLOBALFAIL` – unerwarteter globaler Flow-Fehler
- `A1-MBOXSETUP` – Postfach-Ordner-Einrichtung fehlgeschlagen
- `A1-FILEWRITE` – External-Domains-Datei-Schreibvorgang fehlgeschlagen
- `A1-TECHERROR` – technischer Fehler bei der Domain-Extraktion
- `A1-NODOMAINS` – keine Domains im Quell-Arbeitsblatt gefunden
- `A1-AUDITWRITE` – zentraler Audit-Trail-Schreibvorgang fehlgeschlagen (genau der hier aufgetretene Fall)

**Noch offen (bewusst als nächster eigener Schritt zurückgestellt, um unter Zeitdruck keine Fehler in 6 Dateien auf einmal zu riskieren):** Das gleiche Kennungs-Schema muss noch auf Agent 2 (15 Alarm-Mails), Agent 3, Agent 4, Agent 5 und Agent 6 ausgeweitet werden. Empfohlene Kürzel-Konvention für die Fortsetzung: kurze, sprechende GROSSBUCHSTABEN-Kürzel ohne Sonderzeichen, pro Agent fortlaufend eindeutig, eingefügt direkt vor dem `[RID:...]`-Teil des jeweiligen Betreffs.

**Versionen:** Agent 1 `[1.0.4]→[1.0.5]`, Agent 2 `[1.0.4]→[1.0.5]`, Solution `7.10.33→7.10.35`.

**Deployment:** Alle 6 Agenten-JSONs vor dem Pack vollständig geprüft (JSON gültig, Referenzen intakt, alle Beschreibungen ≤255 Zeichen, alle neuen Variablen korrekt initialisiert). Solution neu gepackt und **erfolgreich importiert** – Agent 1 und Agent 2 wurden dabei deaktiviert und **müssen vom Nutzer manuell reaktiviert werden**.

---



**Nutzer hat für heute Schluss gemacht.** Dies ist der exakte Übergabepunkt für die nächste Sitzung.

## Was zuletzt (in dieser Reihenfolge) gemacht wurde
1. Replace-Button-Regression behoben: Datei-Picker öffnete nicht mehr, weil die unsichtbare `attExternalDomainsReplace`-Steuerung (70×200px) die neue LED (`dotLedEmergencyReport`, X=450) und `lblExternalDomainsCount` überlappte und über den Kachelrand hinausragte. Auf 54×95px verkleinert → Picker funktioniert wieder, Scroll-Bug bleibt behoben.
2. `dotLedEmergencyReport` auf Nutzerwunsch etwas nach rechts verschoben (X=450→458).
3. Files "Last Updated"-Format umgestellt: `dd.mm hh:mm` → `dd.mm.yyyy hh:mm:ss` + dynamisches `CET`/`CEST`-Suffix (alle 5 Zeitstempel-Zeilen), Spalte 200→214px verbreitert.
4. Logo oben links nochmals deutlich vergrößert: 220×148 → 280×188px, Sidebar dafür 252→300px verbreitert (Padding 16→10px).
5. **🔴 ECHTER BUG behoben:** Der vom Nutzer als Regression gemeldete Fehlalarm "Mailbox evidence folder setup failed" (Agent 3) wurde per Hintergrund-Audit-Agent auf ALLE 6 Agenten geprüft. Bestätigter Root Cause: `SET_AlertTargetFolderId` (letzte Aktion der `E-Mail_Folder_creation`-Scope) tolerierte im `runAfter` nur `["Succeeded"]` – bei leerem/fehlgeschlagenem vorgelagerten Get (z. B. wegen des harmlosen, bereits tolerierten 409 "Ordner existiert bereits" auf dem Wurzelordner) wurde die Scope fälschlich als "Failed" gewertet. **Betraf Agent 3 UND Agent 4** (identisches Muster); Agent 1/2/5 waren bereits korrekt (Agent 5 diente als Referenzimplementierung), Agent 6 hat keine Mailbox-Ordner-Logik.
   - Fix in beiden Agenten: `SET_AlertTargetFolderId` toleriert jetzt `["Succeeded","Failed","Skipped","TimedOut"]` + `coalesce(...,'')`.
   - Agent 3 zusätzlich: Alarm-Mail-Block in eine neue `IF_EmailFolderResolutionFailed`-Bedingung verschoben, die den echten Geschäftszustand (`@empty(coalesce(variables('AlertTargetFolderId'), '')) = true`) statt des Scope-Status prüft (Agent-5-Muster repliziert).
   - Agent 4 hatte keinen eigenen Alarm-Mail-Block – die Toleranz-Korrektur allein genügt dort.

## Deployment-Stand (WICHTIG für morgen)
- **Git:** Alles committed und gepusht, letzter Commit `fb7885f` auf `origin/main`.
- **Solution (Flows):** Neu gepackt (`pac solution pack`) und **erfolgreich importiert** (`pac solution import`) in `DBG Team Productivity (Dev)`. Versionen: Agent 3 `[1.1.1]→[1.1.2]`, Agent 4 `[1.2.7]→[1.2.8]`, Solution `7.10.23→7.10.25`.
  - **⚠️ TODO Nutzer: Agent 3 und Agent 4 sind nach dem Import wie immer DEAKTIVIERT und müssen manuell in Power Automate reaktiviert werden**, bevor sie wieder Uploads/Status-Checks verarbeiten.
- **Power App (Canvas):** Neu gepackt (`pac canvas pack`), Round-Trip-Verifikation bestanden (0 Diff-Zeilen), Container-Anzahl-Sanity-Check bestanden (alle 11 Haupt-Container genau 1×). Version `v1.5.7→v1.5.8`.
  - **⚠️ TODO Nutzer: Die `.msapp` muss noch in Power Apps Studio geöffnet/importiert und dort Speichern+Veröffentlichen ausgeführt werden**, damit alle heutigen GUI-Änderungen live gehen (inkl. Logo, LED-Position, Files-Zeitformat, Replace-Fix).

## Was der Nutzer morgen als Erstes prüfen sollte (Live-Test)
1. Agent 3/4 reaktivieren, dann: Emergency Report hochladen und bestätigen, dass **kein** Fehlalarm "Mailbox evidence folder setup failed" mehr kommt, wenn der Wurzelordner "PA Processed Mails" bereits existiert.
2. Replace-Button bei "External Domains" testen: Picker muss sofort ohne Mausrad-Trick öffnen.
3. LED-Position rechts vom Replace-Button optisch prüfen.
4. Files-Container: neues Zeitstempelformat (`TT.MM.JJJJ hh:mm:ss CE(S)T`) auf Lesbarkeit/Spaltenbreite prüfen.
5. Logo oben links auf neue Größe prüfen (nicht überlappend mit Operating-State-Kachel-Rand).

## Offene Backlog-Punkte (nur dokumentiert, NICHT implementiert – nächste Priorisierung mit Nutzer klären)
- 🔵 Wöchentliche/tägliche Status-Report-E-Mails + Aktivitäts-LEDs pro Kachel (vollständig spezifiziert, s. Abschnitt "GROSSE NEUE FEATURES" weiter unten).
- 🔵 Manuelles E-Mail-Template für Agent-2-Mails über Sidebar.
- 🔵 Internal/External Domains von Textdateien auf echte SharePoint-Listen umstellen + linke Sidebar-Navigation für alle Listen (inkl. der aktuell fälschlich angezeigten "19 SharePoint"-Liste, die noch nicht gepflegt ist).
- 🔵 Admin-Bereich: 5 vorbereitete Funktionen (Counter-Reset, Audit-Trail-Archivierung, Acknowledgment-Button, KPI-Verlinkung zum gefilterten Audit-Trail, Detail-Tab pro Agent) – alle bereits ausgearbeitet, s. Abschnitt weiter unten.
- 🔵 Agent-2-Performance (~4-5 Min/E-Mail) – Ursache vermutlich `WaitSecondsBeforeSentMailSearch`-Konfigwert, Nutzer muss SharePoint-Config-Liste prüfen.
- 🔵 Kachel-Verwaltung ("NEXT STEPS" etc.) in eigene Agenten auslagern – bewusst erst nach Abschluss aller GUI-Kacheln.
- 🔵 `conFilesPlaceholder`-Container (rechts neben Files, aktuell "RESERVED FOR FUTURE USE") – noch keine Idee, was hinein soll.

---

## ✅ GELÖST (2026-08-27, Fortsetzung 9): Replace-Picker-Regression, Files-Zeitstempelformat, LED-Position, Logo-Größe, ECHTER Agent-3/4-Regressions-Bug behoben

**Replace-Button öffnet den Datei-Picker nicht mehr (Regression):** Ursache war die zuvor vergrößerte unsichtbare `Attachments`-Steuerung (`attExternalDomainsReplace`, 70×200px), die inzwischen den neuen `dotLedEmergencyReport` (X=450) und `lblExternalDomainsCount` (X=480) überlappte UND vertikal über den unteren Rand von `conMaintenanceDomains` hinausragte. **Fix:** Größe auf 54×95px reduziert (bleibt klar innerhalb der Kachel und vor der LED) – Scroll-Notwendigkeit bleibt behoben, Picker öffnet wieder normal.

**LED-Position:** `dotLedEmergencyReport` auf Nutzerwunsch etwas weiter nach rechts verschoben (X=450→458).

**Files "Last Updated"-Format:** Von `dd.mm hh:mm` auf `dd.mm.yyyy hh:mm:ss` + dynamisches `CET`/`CEST`-Suffix umgestellt (`TimeZoneOffset(...)  <= -90` → Sommerzeit), für alle 5 Zeitstempel-Zeilen (Emergency Report, Internal/External Domains, Counter, Audit Trail). Spaltenbreite von 200px auf 214px erweitert, damit der längere Text nicht abgeschnitten wird.

**Logo oben links:** Auf Nutzerwunsch nochmals deutlich vergrößert (220×148 → 280×188px), Sidebar dafür von 252px auf 300px verbreitert (Padding 16→10px), ohne andere Layoutteile zu beeinträchtigen (Sidebar ist die einzige Stelle, die die 252/300-Konstante nutzt).

**🔴 ECHTER BUG behoben – "Mailbox evidence folder setup failed"-Fehlalarm-Regression (Agent 3 UND Agent 4):**
Ein zuvor beauftragter Hintergrund-Audit über alle 6 Agenten-Flows hat den Root Cause bestätigt: In Agent 3 (`DMPAgent301EmergencyReportManagementVS`) und Agent 4 (`DMPAgent302StatusCheckVS`) tolerierte die letzte Aktion der `E-Mail_Folder_creation`-Scope (`SET_AlertTargetFolderId`) im `runAfter` NUR `["Succeeded"]`. Lieferte der vorgelagerte `Get_DMP_Mailbox_Subfolder_ID_for_"Agent_X_Alerts"`-Aufruf (z. B. wegen des bereits bekannten, harmlosen HTTP-409 "Ordner existiert bereits" bei "PA Processed Mails") KEIN Ergebnis oder schlug fehl, wurde `SET_AlertTargetFolderId` übersprungen ("Skipped") und die gesamte Scope damit als "Failed" gewertet – obwohl der eigentliche Zielordner "Agent X Alerts" ganz normal existierte. Bei Agent 3 löste das den fälschlichen Alarm-Mail-Versand aus; bei Agent 4 hätte es (mangels eigenem Alarm-Mail-Block dort) zumindest den gesamten Flow-Lauf fälschlich als "Failed" markiert.

Agent 5 (`DMPAgent303OperationalStateManagementVS`) hatte dieses Problem bereits früher korrekt gelöst (Referenzimplementierung) – Agent 1 und Agent 2 sind durch abweichende Patterns (explizite Empty-Prüfung bzw. sichere `first()`-Extraktion) ebenfalls nicht betroffen. Agent 6 hat keine Mailbox-Ordner-Erstellung.

**Fix (repliziert Agent 5s bewährtes Muster in Agent 3 und Agent 4):**
- `SET_AlertTargetFolderId`: `runAfter` auf `["Succeeded","Failed","Skipped","TimedOut"]` erweitert (läuft jetzt immer), Wert per `coalesce(..., '')` gegen leere/fehlgeschlagene Vorgänger abgesichert → die Scope selbst ist jetzt IMMER "Succeeded".
- Agent 3: Der bisher direkt auf den Scope-Status reagierende Alarm-Block (`SET_AuditOutcome_EmailFolderFailed` → `AUDIT_EmailFolderCreation_Failed` → `SET_AlertMailSubject_(EmailFolderCreationFailed)` → `MAIL_Alert_(EmailFolderCreationFailed)`) wurde in eine neue `IF_EmailFolderResolutionFailed`-Bedingung verschoben, die stattdessen den ECHTEN Geschäftszustand prüft: `@empty(coalesce(variables('AlertTargetFolderId'), '')) = true`. Der Alarm wird also nur noch ausgelöst, wenn der Zielordner wirklich nicht aufgelöst werden konnte – nicht mehr bei einem harmlosen, bereits tolerierten 409 auf dem Wurzelordner.
- Agent 4: Hatte keinen eigenen Alarm-Mail-Block für diesen Fall; der Fix der `SET_AlertTargetFolderId`-Toleranz allein genügt, damit der Flow-Lauf nicht mehr fälschlich als fehlgeschlagen markiert wird.

**Versionen:** Agent 3 `[1.1.1]→[1.1.2]`, Agent 4 `[1.2.7]→[1.2.8]`. Solution-Version `7.10.23→7.10.25`. Power App `v1.5.7→v1.5.8`.

**Deployment:** Beide Flow-JSONs validiert (JSON-Syntax geprüft), Solution neu gepackt (`pac solution pack`) und **erfolgreich importiert** (`pac solution import`) – Agent 3 und Agent 4 wurden dabei wie immer deaktiviert und **müssen vom Nutzer manuell reaktiviert werden**. Power App neu gepackt (`pac canvas pack`, Round-Trip-Verifikation: 0 Diff-Zeilen, Container-Anzahl-Sanity-Check bestanden). Committed (`fb7885f`) und gepusht.

---

## 🔵 GROSSE ARCHITEKTUR-ÄNDERUNG (Backlog, NACH Abschluss aller GUI-Kacheln): Kachel-Verwaltung in eigene Agenten auslagern
**Nutzeranforderung (2026-08-27):** Die Verwaltung einzelner GUI-Kacheln (beginnend mit "NEXT STEPS"/Default-Management-Process-Meilensteine) soll NICHT weiter direkt in Agent 4 (Status Check) mitwachsen, sondern in **eigene dedizierte Agenten** ausgelagert werden – nicht nur für die reine Anzeige/Datenbeschaffung, sondern für die **komplette Steuerung der Abläufe inklusive Nutzer-Interaktionen** (z. B. einen Meilenstein direkt aus der Kachel heraus als erledigt markieren, Kommentare hinzufügen, o. ä. – konkreter Funktionsumfang noch mit Nutzer zu klären, wenn dieser Punkt ansteht).

**Zeitpunkt:** Bewusst erst NACHDEM alle aktuell laufenden GUI-Kachel-Überarbeitungen (Design/UX-Pass container-für-container) abgeschlossen sind – nicht vorher anfangen.

**Hintergrund/Auslöser:** Agent 4 ist im Laufe der Zeit zu einem sehr breiten "Sammelagenten" geworden (Konfigurationsstatus, 6 parallele Domain-/Datei-Existenzchecks, 6 parallele Audit-Summary-Reads, Versions-Datei-Read, 44-zeilige Meilenstein-Schleife, Status-Zeilen-Update) – dies wurde am 2026-08-27 als wahrscheinlicher Mitverursacher eines echten `ActionResponseTimedOut`/504-Fehlers identifiziert (siehe eigener Abschnitt weiter unten). Eine Aufteilung auf mehrere fokussierte Agenten (jeweils nur für eine Kachel zuständig) würde sowohl die Wartbarkeit als auch vermutlich die Performance verbessern.

---

## ✅ GELÖST (2026-08-27, Fortsetzung 5): Maintenance-Domains-Container – Buttons, Ausrichtung, Replace-Upload
1. **✅ Button-Stil vereinheitlicht:** View/Edit/Replace hatten 3 unterschiedliche Stile (Outline, solid-grün, solid-blau). Jetzt alle im gleichen "Outline"-Stil (transparente Füllung, farbiger Rahmen, farbiger Text) – nur die Akzentfarbe unterscheidet sich je Funktion (View=neutral, Edit=Grün, Replace=Blau). Wirkt als zusammengehörige Button-Familie statt 3 verschiedener Stile.
2. **✅ Ausrichtung korrigiert:** Kompletter Umbau auf `Variant: ManualLayout` (wie Operating State) mit festen X-Spalten – View-Buttons von Internal/External stehen jetzt exakt übereinander, ebenso Edit-Buttons.
3. **🟡 Replace-Upload-UX (Versuch, nicht live getestet):** Die unsichtbare `Attachments`-Steuerung, die hinter dem "Replace"-Button liegt, wurde von 40px auf 44px Höhe vergrößert – Hypothese: Das native Steuerelement brauchte mehr Platz, um sein "Datei hinzufügen"-Ziel ohne internes Scrollen zu zeigen. **Muss beim nächsten Live-Test geprüft werden, ob das Scroll-Problem dadurch behoben ist** – falls nicht, braucht es einen grundsätzlich anderen Ansatz (Power Apps bietet für Datei-Uploads praktisch nur die `Attachments`-Steuerung, echte native Alternativen sind sehr eingeschränkt).

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich. PowerApp-Version → `v1.5.3`, Solution-Version neu berechnet → `7.10.18` (Flows unverändert, kein Re-Import nötig).

## 🔵 NEUE BACKLOG-PUNKTE (2026-08-27, aus Maintenance-Domains-Feedback)
1. **Domains-Speicherung von Flat-Files auf echte SharePoint-Listen umstellen:** Aktuell werden Internal/External Domains als reine `.txt`-Dateien gespeichert (`Internal_Domains_Storage/Internal_Domains.txt` etc.) und über `Launch()`-Links extern in SharePoint/Excel geöffnet. Nutzer möchte stattdessen echte SharePoint-Listen, die direkt aus der App heraus gepflegt werden können. **Zusätzliche Beobachtung:** Die angezeigte Zahl "19 (SharePoint)" bei Internal Domains dürfte nicht die reale/gepflegte Datenmenge widerspiegeln, da die Liste noch nicht in SharePoint gepflegt ist – erst nach der Umstellung auf eine echte Liste sinnvoll aussagekräftig.
2. **Sidebar-Navigation für Listen-Pflege:** Nutzerwunsch: Internal Domains, External Domains sowie alle weiteren Konfigurationslisten sollen als eigene Menüpunkte in der linken Sidebar erscheinen, sodass die Pflege komplett aus der App heraus möglich ist (statt über externe `Launch()`-Links). Passt inhaltlich zu den bereits als Platzhalter vorhandenen, aber noch nicht gebauten Sidebar-Einträgen "Configuration (Lists)" und "Maintenance". **Vorschlag für spätere Umsetzung:** Sobald Punkt 1 (echte SharePoint-Listen) umgesetzt ist, für jede Liste einen einfachen In-App-Screen (Gallery/Data-Table mit Such-/Filterfunktion) bauen und hinter diesen Sidebar-Einträgen verlinken.

## 🔵 GROSSE NEUE FEATURES (Backlog, Nutzeranforderung 2026-08-27 nachmittags): Wöchentlicher/täglicher Status-Report + Aktivitäts-LEDs
**Kontext:** Das Cockpit wird voraussichtlich nicht permanent geöffnet sein – hauptsächlich während aktiver DMP-Phasen (SIMU oder PROD). Im normalen PROD-Betrieb (PROD_NODMP) wird es vermutlich gar nicht geöffnet. Daher werden zwei neue, unabhängige Features benötigt. **Ausdrücklich NICHT jetzt umgesetzt, nur als Design/Spezifikation festgehalten**, da beide Features neue, noch nicht existierende Datenstrukturen bzw. einen neuen Flow benötigen und der Nutzer explizit gebeten hat, zunächst nur zu dokumentieren.

### Feature A: Automatischer Status-Report per E-Mail
**Versandregeln (mit Nutzer abgestimmt 2026-08-27):**
- `PROD_NODMP` → **wöchentlich**
- `PROD_DMP` → **täglich**
- `SIMU_DMP` → **täglich** (da hier aktiv getestet wird)
- `SIMU_NODMP` → **NIE automatisch** (Nutzer beobachtet das Cockpit hier ohnehin live)

**Inhalt:** "Screenshot" der wesentlichen Cockpit-Parameter – Agent-Status (alle 6), System Health, Anzahl der im Berichtszeitraum abgearbeiteten E-Mails (vermutlich häufig 0 bei PROD_NODMP).

**Zähler-Regel (mit Nutzer abgestimmt):** Ein **neuer, separater periodenbezogener Zähler** wird nach jedem Versand zurückgesetzt – die bestehenden Lifetime-Betriebszähler (Agent 1/2) bleiben davon komplett unberührt.

**Architektur (mit Nutzer abgestimmt – Performance hat oberste Priorität, "darf bestehende Funktionalität nicht beeinträchtigen"):**
- Kompletter **neuer, dedizierter Agent 7 ("Reporting & Activity")** mit eigenem `Recurrence`-Trigger (z. B. täglich fixe Uhrzeit), der intern prüft, ob laut aktuellem Modus + letztem Versandzeitpunkt ein Versand fällig ist. Läuft komplett unabhängig von Agent 4 und dem Cockpit – keinerlei Performance-Einfluss auf die bestehende Funktionalität.
- **Blocker/offener Punkt:** Braucht einen neuen persistenten Speicherort für `Mode`, `LastSentUtc`, `PeriodEmailsProcessed` (Vorschlag: neue kleine Tabelle "ReportingState"). Bewusste Entscheidung 2026-08-27: **NICHT** direkt/roh in `AuditTrail.xlsx` schreiben (6 Live-Flows greifen parallel zu, Korruptionsrisiko bei gleichzeitigem Cloud-Schreibzugriff während lokaler Bearbeitung über den OneDrive-Sync). Nutzer hat der KI erlaubt, den Versuch zu unternehmen, wurde aber noch nicht abgeschlossen (Sitzung wurde unterbrochen, um erst dieses Backlog-Update zu machen) – **beim nächsten Anlauf**: entweder Nutzer legt die Tabelle selbst in Excel Online an (sicherste Variante), oder KI versucht es mit vorherigem Backup + Verifikation der unveränderten Alt-Daten.

### Feature B: Aktivitäts-LEDs pro Kachel + übergeordnete Master-LED
**Betroffene Container (mit Nutzer abgestimmt):** Maintenance Domains, Emails Processed (Agent 2), System Health, Automation Status/Next Steps – **plus eine übergeordnete "Master-LED"** (z. B. im Header), die den jeweils schlechtesten (höchsten) Status über alle Einzel-LEDs hinweg anzeigt.

**Farblogik (mit Nutzer abgestimmt):**
- Grau/unsichtbar = seit letztem bekannten Stand nichts verändert
- Grün = etwas hat sich "normal" verändert (z. B. Agent 2 E-Mail-Zähler erhöht = eine E-Mail wurde ordnungsgemäß verarbeitet)
- Gelb = ein Prozess endete mit einer Warnung
- Rot = ein Prozess endete mit Fehler oder wurde abgebrochen

**Architektur (Performance-optimiert, mit Nutzer abgestimmt):**
- Der Vergleich "aktueller Wert vs. letzter bekannter Wert" läuft **komplett clientseitig in der Power App** (Vergleich der bei jedem ohnehin stattfindenden Agent-4-Poll gelieferten Werte gegen lokal gespeicherte letzte Werte) – **keine zusätzlichen Backend-Aufrufe pro Poll**, um die heute gefundenen Performance-Probleme (siehe Agent-2-Abschnitt) nicht zu verschärfen.
- Voraussetzung: Agent 4 muss die bereits abgerufenen Pro-Agent-Rohdaten aus `SCOPE_AuditSummary_Read` (aktuell nur zu EINER Gesamtsumme aggregiert) zusätzlich ungekürzt/pro Agent im Response-Objekt mitliefern – das sind KEINE neuen API-Calls, nur zusätzliche Exposition bereits vorhandener Daten.
- Baseline-Persistenz über Sitzungen hinweg: Wiederverwendung der bereits bestehenden `AuditAcknowledgment`-Tabelle (siehe weiter unten) als Baseline-Speicher – EINMALIGER Abruf beim App-Start (nicht pro Poll), danach nur noch clientseitiger Vergleich.

**Popup-Verhalten (mit Nutzer abgestimmt):**
- Klick auf eine LED öffnet ein Popup mit High-Level-Zusammenfassung (rein aus bereits im Client vorhandenen Vergleichsdaten, kein zusätzlicher Abruf).
- Zusätzlich ein Button/Link, der auf den (noch zu bauenden) "Audit Trail (Detail)"-Screen verweist und diesen mit den relevanten Einträgen vorgefiltert öffnet (Synergie mit Admin-Punkt 4/5 weiter unten).
- Optionen im Popup: Inhalt in die Zwischenablage kopieren, oder schließen.
- **Schließen setzt die LED + den zugehörigen Log zurück** (technisch: schreibt die neue Baseline in `AuditAcknowledgment`, analog zum bereits geplanten "Bestätigen"-Button).

**Status:** Nur als Design festgehalten, KEINE Implementierung begonnen (außer dem noch unvollendeten Versuch, die Excel-Tabelle für Feature A anzulegen – abgebrochen auf Nutzerwunsch, um zuerst zu dokumentieren).

---

## ✅ GELÖST (2026-08-27, Fortsetzung 6): Agent-2-Ring, Files-Zeile umgezogen, Light-Mode-Kontraste
1. **✅ "Agent 2 - Emails Processed"-Ring:** Auf System-Health-Größe gebracht (180→328px). Externe redundante Titelzeile entfernt (Ring hat bereits eine eigene "EMAILS PROCESSED"-Beschriftung, analog zum System-Health-Ring ohne externen Titel). Rahmenfarbe auf den etablierten Grün-Akzent umgestellt.
2. **✅ Legende neu gestaltet:** Abkürzungen (No DMP/DEE/DIS/DNES) durch Klartext ersetzt (No DMP/External/Internal/Not Effected, basierend auf den offiziellen Definitionen im Operations Manual), rechts neben dem Ring platziert (statt darunter), zeigt jetzt zusätzlich Anzahl und Prozentanteil in Klammern, z. B. "External (12, 34%)".
3. **✅ Kontrast-Fix (Light Mode):** Die 4 Legende-Farben (insbesondere das helle Lila und Orange) waren im Light Mode vermutlich schlecht lesbar (fest codierte Farben ohne Rücksicht auf das Theme) – jetzt alle themenabhängig mit abgedunkelten Varianten für Light Mode.
4. **✅ Breiten neu ausbalanciert:** Operating State 600→550px, Maintenance Domains 700→620px (beide geben Raum ab), Agent 2 400→620px (braucht den Platz für Ring+Legende). Gesamtbreite der mittleren Spalte bleibt bei 1186px.
5. **✅ Files-Zeile umgezogen:** War bisher eine eigene volle Breite einnehmende Zeile unterhalb der gesamten oberen Kachelreihe. Jetzt in den bisher ungenutzten Freiraum unterhalb von Operating State/Maintenance Domains verschoben (`conMiddleColumn` dafür auf `ManualLayout` mit expliziten X/Y-Koordinaten umgestellt, analog zum bereits bewährten Muster). Rahmenfarbe und Titel auf den Grün-Akzent-Standard gebracht, alle 5 Einträge (Emergency Report/Internal Domains/External Domains/Counter/Audit Trail) exakt untereinander ausgerichtet statt in einer gedrängten Einzeilen-Leiste.

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich. PowerApp-Version → `v1.5.4`, Solution-Version neu berechnet → `7.10.19` (Flows unverändert, kein Re-Import nötig). **Wartet auf Nutzer-Begutachtung, bevor weitere Container angefasst werden.**

**Nachtrag – ECHTER BUG beim Laden in Studio gefunden und behoben:** `RadiusBottomLeft/Right/TopLeft/TopRight` wurden versehentlich auf einem `Label`-Steuerelement (`lblReplaceFake`) gesetzt – Labels unterstützen diese Eigenschaft nicht (nur Container/Button/Rectangle). `pac canvas pack` prüft das NICHT, erst Studio meldet `PA2108: Unknown property` beim echten Laden. **Neue KI-Regel:** Rahmen/Rundung für einen als Button getarnten Label immer auf den umgebenden `GroupContainer` legen, niemals direkt auf das `Label`. Zusätzlich festgestellt: `pac canvas validate` (das genau solche Fehler lokal hätte finden können) ist in der installierten CLI-Version (2.11.2) nicht mehr unterstützt („no longer supported") – Studio bleibt daher die einzige verlässliche Schema-Prüfung. Fix deployt: PowerApp-Version → `v1.5.5`, Solution-Version → `7.10.20`.

## ✅ GELÖST (2026-08-27, Fortsetzung 7): "Loading…"-Zustände, Files-Tabelle, Logo, Replace-Upload
1. **✅ Falsche Default-Werte beim initialen Laden:** Der Emails-Ring zeigte vor dem ersten erfolgreichen Agent-4-Aufruf einen irreführenden Fallback-Zustand (z. B. "1" mit 100% orangem Segment, während die Legende "0, 0%" zeigte). Ring UND alle 4 Legende-Texte sowie alle 5 Files-Einträge zeigen jetzt "Loading…" statt Fallback-Werte, solange `IsBlank(varStatusLastRefreshed)` (identisches Muster wie die KPI-"?/6"-Anzeige).
2. **✅ Files-Container komplett als Tabelle neu gebaut:** Vorher ein einzelner langer Text pro Zeile (Status/Zeit nicht ausgerichtet, Zeitstempel-Bedeutung unklar). Jetzt 3 saubere Spalten (Dateiname / Status / "LAST UPDATED"-Zeitstempel mit Spaltenüberschriften), Format von reiner Uhrzeit (`hh:mm:ss`) auf Datum+Uhrzeit (`dd.mm hh:mm`) geändert.
3. **✅ Files-Container verschmälert auf Breite von Operating State (550px)**, dafür rechts daneben einen gestrichelten Platzhalter-Container ("RESERVED FOR FUTURE USE", 620px) für künftige Ideen ergänzt.
4. **✅ Sidebar-Logo vergrößert:** 190×128 → 220×148 (füllt die verfügbare Sidebar-Breite abzüglich des bestehenden 16px-Innenabstands).
5. **🟡 Replace-Upload-Scrollproblem – 2. Versuch:** Die unsichtbare `Attachments`-Steuerung wurde nochmals deutlich vergrößert (44×90 → 70×200), da vermutlich die interne "Anhang hinzufügen"-Kachel bei zu wenig Platz eine interne Scroll-Notwendigkeit auslöst. **Weiterhin nicht live verifiziert** – falls das Problem bestehen bleibt, muss ein grundsätzlich anderer Ansatz für den Datei-Upload evaluiert werden (Power Apps bietet praktisch nur die `Attachments`-Steuerung).

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich. PowerApp-Version → `v1.5.6`, Solution-Version neu berechnet → `7.10.21` (Flows unverändert, kein Re-Import nötig).

## ✅ GELÖST (2026-08-27, Fortsetzung 8): KRITISCHER Fund – `conEmailsCard` versehentlich gelöscht, Agent-3-LED, Kontrast-Fix, Ladefehler
1. **🔴 KRITISCH – `conEmailsCard` (Agent 2 Ring) war komplett verschwunden:** Bei der Files-Zeilen-Neugestaltung (Fortsetzung 7) hatte eine Line-Range-Splice-Operation den kompletten Container mitgelöscht, da er zwischen den beiden Such-Ankern lag, ohne dass ich das bemerkt hatte (weder `pac canvas pack` noch der Round-Trip-Diff schlugen an). Vollständig rekonstruiert (Ring, Legende, Loading-Zustände) und an drei Stellen mit `grep`-Zählung verifiziert, dass alle 11 erwarteten Haupt-Container wieder genau einmal vorhanden sind. **Neue Pflichtprüfung nach jeder Splice-Operation in den Arbeitsregeln verankert.**
2. **✅ Studio-Ladefehler behoben:** `Width`-Eigenschaft des Sidebar-Logos war durch einen Tippfehler bei einer früheren Änderung 2 Leerzeichen zu wenig eingerückt (landete außerhalb von `Properties:`) – `PA1001: YamlInvalidSyntax` beim Laden in Studio. Korrigiert.
3. **✅ Agent 3 – neue blinkende LED beim Emergency-Report-Upload:** Analog zum Umschalt-Muster bei Operating State (Gelb während Verarbeitung, Grün bei Erfolg, Rot bei Fehler, Grau im Ruhezustand). Nebenbei einen Text-Bug behoben: Erfolgsmeldung sprach fälschlich von "Agent 1", obwohl tatsächlich Agent 3 aufgerufen wird.
4. **✅ Kontrast-Fix "Emails Processed" (Light Mode):** Die Ring-interne Beschriftung "EMAILS PROCESSED" nutzte eine fest codierte helle Farbe (`rgb(200,195,225)`) – auf hellem Hintergrund kaum lesbar. Jetzt themenabhängig, identisches Muster wie beim System-Health-Ring.

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich, alle 11 Haupt-Container-Namen exakt einmal vorhanden verifiziert. PowerApp-Version → `v1.5.7`, Solution-Version neu berechnet → `7.10.22` (Flows unverändert, kein Re-Import nötig).

## 🔵 Neue Backlog-Punkte (2026-08-27, Nachmittag)
- **Manuelles E-Mail-Template:** Die von Agent 2 automatisch versendeten E-Mails sollen zusätzlich als manuelles Template über einen Sidebar-Menüpunkt verfügbar gemacht werden (für Fälle, in denen manuell eine solche E-Mail ausgelöst werden soll). Noch keine Details zu Umfang/genauer Ausgestaltung geklärt.
- **Agent 3 – Datei-Sperr-Fehlalarm (2026-08-27, RunId `08584137711143656468602881005CU23`):** Nutzer erhielt die irreführende Meldung "Mailbox evidence folder setup failed", tatsächliche Ursache laut Rohfehler war aber `SPFileLockException` (HTTP 423) beim `Recycle()`-Aufruf der temporären Datei `Agent3_Work/WORK_<RunId>.xlsx`. **Nachtrag nach Code-Prüfung:** Dieser konkrete Recycle-Schritt (`SCOPE_WorkFileCleanup_CurrentRun`) behandelt einen gesperrten Work-File bereits BEWUSST als reine Warnung (`AUDIT_WorkFileCleanupStillLocked`, Status "Warning"), nicht als harten Fehler – dieser Teil ist also vermutlich NICHT die Ursache der Alert-Mail. Die eigentliche Mailbox-Ordner-Erstellung (`E-Mail_Folder_creation`, Microsoft-Graph-Aufrufe auf das Postfach, komplett unabhängig vom SharePoint-Dateisystem) hat bereits eine defensive "Create, bei Fehler stattdessen suchen"-Logik. **Der Datei-Lock ist also wahrscheinlich ein bereits sauber behandelter Nebenbefund, nicht die eigentliche Ursache.** Für die echte Ursache müssten die konkreten Fehlerdetails des `E-Mail_Folder_creation`-Schritts AUS DEMSELBEN Lauf geprüft werden (im Run-Verlauf in Power Automate) – noch offen.

---

# 🟢 SESSION-STATUS zum 2026-08-27 – SOFORT HIER WEITERLESEN, bevor irgendetwas Neues begonnen wird

## ⚠️ Wichtiger Kontext für die Fortsetzung
- Der Nutzer hat berichtet, dass beim letzten VS-Code-Update der komplette Chatverlauf dieser Sitzung verloren ging. **Dieses Backlog-Dokument ist daher die einzige verlässliche Quelle** für alles, was heute gemacht wurde – bei Gesprächsverlust hier zuerst nachlesen.
- **Copilot-Kontingent-Hinweis:** "Credits at 75%"-Meldung erhalten, nur noch wenige Arbeitstage bis Monatsende. **Weiterhin sparsam vorgehen:** weniger Zwischenschritte pro Fix, Round-Trip-Verifikationen nur bei riskanten/neuartigen Änderungen.
- **Aktueller deployter Stand:** Power App `v1.5.2` (neu gepackt, `.msapp` bereit – Nutzer muss noch manuell in Studio öffnen/importieren + Speichern/Veröffentlichen), Solution `7.10.16` (Flows unverändert in dieser Runde, daher KEIN erneuter `pac solution import` nötig). Alle 6 Flows: Agent 1/2 `[1.0.1]`, Agent 3 `[1.1.0]`, Agent 4 `[1.2.7]`, Agent 5 `[1.1.5]`, Agent 6 `[1.1.0]`. Letzter Git-Commit: `f15f3fe`.
- **WICHTIG – Canvas App ist NICHT Teil der Dataverse-Solution:** Das `.msapp` wird separat per `pac canvas pack` erzeugt und muss vom Nutzer manuell in Power Apps Studio geöffnet/aktualisiert werden. Nur die 6 Flows laufen über `pac solution import`.
- **Nutzer ist 2-3h im Meeting (ab ca. 2026-08-27 10:50):** KI arbeitet in dieser Zeit selbstständig eine vereinbarte Aufgabenliste ab (siehe unten), OHNE auf Live-GUI-Feedback zu warten. Neue GUI-Screens/Tabs werden bewusst nur VORBEREITET/dokumentiert, nicht blind fertig gebaut (Begründung: wiederholt gezeigt, dass Breiten/Schriftgrößen/Ausrichtung ohne Live-Test mehrere Anläufe brauchen).

## 🔵 Aufgabenliste für die Meeting-Zeit (vom Nutzer 2026-08-27 vorgegeben)
1. ✅ GUI-Feinschliff Operating State/Maintenance Domains (Schriftgrößen Label/Wert vereinheitlicht, Operating State 650→600px schmaler, Maintenance Domains 650→700px breiter zugunsten der dort noch unvollständig sichtbaren Labels).
2. Agent 2 (E-Mail Inbox Treatment) Performance-Untersuchung (~4-5 Min/E-Mail) – Flow-Logik-Analyse.
3. Agent 3 (Emergency Report Management) Alert-Mail-Kette – Flow-Logik-Review (kein Live-Test).
4. "NEXT STEPS"-Kachel – Analyse/Vorbereitung für spätere Auslagerung in einen dedizierten Agenten (siehe Architektur-Backlog oben).
5. Admin-Bereich – 5 neue Funktionen vorsehen/vorbereiten:
   - Zurücksetzen der Counter auf 0
   - Archivierung des Audit Trails + Zurücksetzen
   - Zurücksetzen von Warnings/Critical über einen blinkenden "Acknowledgment"-Button
   - Verlinkung von Warnings/Critical-KPIs zum gefilterten Detail-Audit-Trail
   - Detaillierter Audit-Trail pro Agent (separater Tab)

## 🔵 Admin-Bereich – 5 neue Funktionen als Vorschlag ausgearbeitet (bewusst NICHT blind implementiert)
Für alle 5 Punkte gilt: Es handelt sich entweder um destruktive/kritische Operationen auf Produktivdaten oder um neue GUI-Screens – beides wurde in dieser Sitzung wiederholt als "braucht Live-Abstimmung mit dem Nutzer" eingestuft. Daher hier nur die ausgearbeiteten, sofort umsetzbaren Vorschläge inkl. offener Fragen, damit die eigentliche Umsetzung schnell gehen kann, sobald der Nutzer zurück ist.

### 1) Zurücksetzen der Counter auf 0
Betrifft vermutlich die operativen Zähler aus Agent 1/2 (`InternalDomainsCount`, `ExternalDomainsCount`, `CounterNoDMP`, `CounterInternalSender`, `CounterNotEffected`, `CounterEffected`) – nicht zu verwechseln mit den Audit-Zählern (siehe Punkt 3).
**Bereits im Backlog als offen dokumentiert** (Abschnitt "Kontrollierter globaler Reset der Counter / des Audit Trail"): braucht **4-Augen-Prinzip** und ist mit Punkt 2 (Audit-Trail-Reset) verknüpft.
**Offene Fragen an Nutzer:** Welche Counter genau (nur die 6 oben, oder auch weitere)? Wie soll das 4-Augen-Prinzip technisch aussehen (zweiter Admin-User bestätigt? Zeitverzögerter Reset mit Abbruchmöglichkeit? Zusätzliches Passwort/Code)?

### 2) Archivierung des Audit Trails + Zurücksetzen
**Ebenfalls bereits als offen dokumentiert**, identisches 4-Augen-Erfordernis.
**Offene Fragen an Nutzer:** Wohin archivieren (neue Datei mit Datumsstempel im selben SharePoint-Ordner? Separater Archiv-Ordner? Neues Sheet in derselben Arbeitsmappe)? Soll die komplette `AuditTrail`-Tabelle geleert werden oder nur Einträge älter als X Tage?

### 3) Zurücksetzen von Warnings/Critical über blinkenden "Acknowledgment"-Button
**Gute Nachricht:** Backend-Datenmodell existiert bereits vollständig (`AuditAcknowledgment`-Tabelle, seit 24.08. angelegt, 5 Zeilen Agent01-05, je 4 Baseline-Zähler+Zeitstempel) – nicht-destruktiv, vergleicht aktuellen Zähler aus `AgentAuditSummary` gegen gespeicherte Baseline.
**Technischer Zielkonflikt gefunden (Grund, warum nicht blind umgesetzt):** Die Vergleichslogik bräuchte in Agent 4 zusätzlich 5-6 GetItem-Aufrufe auf `AuditAcknowledgment` (analog zu den bereits vorhandenen 6 GetItem-Aufrufen auf `AgentAuditSummary` in `SCOPE_AuditSummary_Read`, ca. Zeile 3079). Das steht im direkten Widerspruch zur heutigen Agent-2-Performance-Erkenntnis ("zu viele API-Aufrufe verlangsamen Flows") UND zur bereits erfolgten Agent-4-Timeout-Optimierung dieser Sitzung (Foreach→Until-Umbau, genau um Aktionen zu sparen). Größe der Abwägung sollte gemeinsam entschieden werden (z. B. Baseline-Vergleich stattdessen in Agent 6 selbst durchführen und nur ein Ergebnis-Flag an Agent 4/Cockpit weiterreichen, um Agent 4 nicht zusätzlich zu belasten).
**Vorschlag:** Neue Agent-6-Aktion `AcknowledgeAuditIssues` (PatchItem, setzt Baseline = aktueller Zähler + Zeitstempel), aufgerufen über einen blinkenden Button (gleiches Blink-Muster wie die neuen Operating-State-LEDs) neben den KPI-Kacheln CRITICAL/WARNINGS.

### 4) Verlinkung von Warnings/Critical-KPIs zum gefilterten Detail-Audit-Trail
Erfordert einen neuen Navigationspfad zur bestehenden Sidebar-Option "Audit Trail (Detail)" mit einem Filter-Parameter (z. B. `Navigate(scrAuditTrail, ScreenTransition.None, {FilterStatus: "Failed"})`). **Geprüft und bestätigt:** Es existiert noch KEIN eigener Screen dafür (nur `scrHome` und `scrHeaderTest` sind als `.pa.yaml`-Dateien vorhanden) – der Sidebar-Eintrag ist aktuell nur ein Platzhalter ohne Funktion. Muss komplett neu gebaut werden.

### 5) Detaillierter Audit-Trail pro Agent (separater Tab)
Ähnlich zu Punkt 4 – vermutlich Erweiterung desselben, noch zu bauenden "Audit Trail (Detail)"-Screens um einen Agenten-Filter/-Tab (z. B. Dropdown oder 6 Tab-Buttons "Agent 1"–"Agent 6"), gespeist aus der `AuditTrail`-Haupttabelle mit `WorkflowPath`-Filter.
**Nächster Schritt (gemeinsam):** Punkte 4+5 sind im Kern EIN neuer Screen (Audit Trail Detail mit Filtern) – sollte als gemeinsames GUI-Projekt mit Live-Feedback geplant werden, sobald die aktuellen Kacheln (Files Row, Automation Status, Sidebar) fertig sind.

## 🔵 "NEXT STEPS"-Kachel – Analyse & Vorbereitung für Auslagerung in dedizierten Agenten
**Aktueller Stand (rein lesend, keine Live-Änderung):**
- Datenquelle: Excel-Tabelle "Status DMP Process.xlsx" (Sheet "Overall Process", 44 Zeilen: Phase/ID/Milestone/Responsibility/Status).
- Gelesen von Agent 4 (Status Check) in `SCOPE_NextMilestones_Read` → `LOOP_OverallProcessRows` (Until-Schleife, bricht ab, sobald 5 offene Meilensteine gefunden wurden oder alle Zeilen durch sind – siehe frühere Performance-Optimierung diese Sitzung).
- Ergebnis wird als `nextmilestones`-Array Teil der normalen Agent-4-Statusantwort, von dort in `scrHome.pa.yaml` in 5 feste Kachel-Zeilen (`conStep1`–`conStep7`, aktuell 7 statische Slots vorhanden, nur 5 befüllt) gerendert.
- **Rein lesend/anzeigend** – keine Interaktion möglich (kein "Erledigt"-Häkchen, keine Statusänderung aus dem Cockpit heraus; Status muss weiterhin direkt in der Excel-Datei gepflegt werden).

**Erkannte Einschränkungen (Begründung für die Auslagerung):**
1. Die Meilenstein-Logik ist fest in Agent 4 (den allgemeinen "Herzschlag"/Status-Check) eingebettet – jede Änderung an der Meilenstein-Darstellung erfordert einen Agent-4-Import samt Flow-Reaktivierung, obwohl es inhaltlich nichts mit dem eigentlichen System-Health-Check zu tun hat.
2. Keine Schreibrichtung: Nutzer kann einen Meilenstein nicht aus dem Cockpit heraus als erledigt markieren – das wäre aber der naheliegende nächste Schritt für echten Mehrwert.
3. Feste 5-Zeilen-Anzeige ist ein reines GUI-Designlimit, keine fachliche Grenze.

**Vorschlag für die spätere Auslagerung (zur Abstimmung mit Nutzer, NICHT jetzt umgesetzt):**
- Neuer dedizierter Agent (z. B. "Agent 7 – Milestone Management") übernimmt: Lesen der Overall-Process-Tabelle, EIGENE Trigger-Route für "Meilenstein X als erledigt markieren" (Schreibzugriff auf die Excel-Statusspalte), eigene Audit-Trail-Einträge.
- Cockpit ruft für die NEXT-STEPS-Kachel diesen neuen Agenten separat auf (entkoppelt vom Agent-4-Heartbeat), inkl. Klick-Handler pro Zeile für die neue "Erledigt"-Aktion.
- Umfang/Aufwand ähnlich zu Agent 6 (Admin Functions) – reiner Dispatcher mit einer Aktion "MarkMilestoneComplete(id)".
- **Bewusst nicht jetzt begonnen** – laut Architektur-Backlog (siehe oben) erst nach Abschluss aller GUI-Kacheln vorgesehen, und diese Sitzung bereits mehrfach gezeigt, dass GUI-Umbauten Live-Feedback brauchen.

## ✅ Agent 3 – ECHTER BUG gefunden UND behoben: Alert-Mail-Kette unzuverlässig
Flow-Logik-Review (kein Live-Test, wie vereinbart) ergab einen bestätigten Bug: Alle 5 Alert-Mail-Aktionen (`MAIL_Alert_(MissingWorksheet)`, `MAIL_Alert_(InvalidWorkbook)`, `MAIL_Alert_(InvalidExtension)`, `MAIL_Alert_(EmailFolderCreationFailed)`, `MAIL_Alert_(Audit_Failure)`) sowie die beiden vorgelagerten `SET_AlertMailSubject_*`-Aktionen hatten `runAfter` nur auf `["Succeeded"]` ihrer jeweiligen Audit-Schreib-Aktion gesetzt. **Konsequenz:** Schlägt der vorgelagerte Audit-Schreibvorgang fehl (z. B. Excel-API-Fehler), wird die Alert-Mail NIE versendet – ausgerechnet in dem Moment, in dem eine Benachrichtigung am wichtigsten wäre. Identisches Bugmuster wie die bereits in Agent 4/5 behobenen Fälle.
**Fix:** Alle 7 betroffenen `runAfter`-Klauseln auf `["Succeeded","Failed","Skipped","TimedOut"]` erweitert (identisches Muster wie bei Agent 4). Ein geprüfter, aber NICHT fehlerhafter Verdachtspunkt (`SCOPE_Validation` sollte angeblich nur nach `MAIL_Alert_(EmailFolderCreationFailed)`-Erfolg laufen) stellte sich als bereits korrekt heraus (hatte schon alle 4 Status) – kein Fix nötig.
Agent 3 → `[1.1.1]`. Solution → `7.10.17`, gepackt und **bereits per `pac solution import` deployt** (Flow muss vom Nutzer wie gewohnt reaktiviert werden).

## 🟡 Agent 2 – Performance-Untersuchung (~4-5 Min/E-Mail): Ursachen gefunden, NICHT blind gefixt
Gründliche Flow-Analyse (9723 Zeilen) ergab mehrere plausible Ursachen, absteigend nach Einfluss:
1. **🔴 19 feste `Wait`-Aktionen** vor jeder "Sent Items"-Suche, gesteuert über Konfigurationswert `WaitSecondsBeforeSentMailSearch` (Wert nicht einsehbar, da SharePoint-Liste, kein Datei-Zugriff). **Größter Hebel, aber NICHT von der KI änderbar:** Nutzer sollte diesen Wert in der SharePoint-Liste "DMP Command Configuration" prüfen und ggf. von (vermutet) 30-60s auf 10-15s reduzieren – reine Konfigurationsänderung, kein Redeploy nötig.
   - **Präzisierung (Nutzerfrage 2026-08-28, "hängt vom Durchlaufpfad ab"):** Pro Lauf feuert nur EINER der 19 Wait-Blöcke – der, der zum tatsächlich genommenen Pfad gehört (No DMP / DIS / DEE / DNES, je Erfolgs- oder Warnungs-Zweig), nicht alle 19 kumulativ. **Ausnahme/Grenzfall:** Wird die zuerst gesendete Mail bei der ersten Sent-Items-Suche NICHT gefunden (z. B. Indexierung dauert länger als der Wait), löst der Flow eine zusätzliche "Sent Mail Not Found"-Warnmail aus UND wartet dafür ein zweites Mal denselben Wert, um DIESE Mail zu suchen/verschieben – dann stehen 2 Waits hintereinander im selben Lauf. Da alle 19 Blöcke denselben einen Konfigurationswert nutzen (kein pfadspezifischer Wert), wirkt eine Reduzierung gleichermaßen auf jeden Pfad, nur eben 1× (Normalfall) oder 2× (Nicht-gefunden-Grenzfall) pro Lauf, nie alle 19 gleichzeitig.
2. Tief verschachtelte sequenzielle `runAfter`-Ketten (~9-10 Aktionen pro Pfad, die zwingend nacheinander laufen müssen, obwohl einzelne Schritte wie Zähler-Increment/Audit-Buffer keine echte Abhängigkeit zueinander haben).
3. 3 Foreach-Schleifen mit externen API-Aufrufen pro Iteration (E-Mail-Verschieben einzeln statt gebündelt, Audit-Zeilen einzeln statt gebündelt geschrieben).
4. 104 externe API-Aufrufe insgesamt (Office365/SharePoint/Excel), keine Bündelung/Caching sichtbar.
**Bewusst NICHT umgesetzt:** Strukturelle Änderungen an einem 9700-Zeilen-Produktivflow ohne Live-Test-Möglichkeit während der Abwesenheit des Nutzers wurden als zu riskant eingestuft. **Empfehlung für die Umsetzung mit Nutzer zusammen:** Zuerst Punkt 1 (Konfigurationswert) prüfen – das ist die risikoärmste und potenziell wirkungsvollste Einzelmaßnahme.

### Nachtrag (2026-08-28): Konkrete Schritt-für-Schritt-Nachverfolgung des Erfolgspfads "DMP Internal Sender" – präzise Zahlen statt Schätzung
Der komplette sequenzielle `runAfter`-Pfad für eine ganz normal erfolgreich verarbeitete E-Mail (Zweig "DMP internal Sender", vermutlich repräsentativ für DEE/DNES/No DMP – gleiche Struktur) wurde Zeile für Zeile nachverfolgt:

**A) Vorlauf (einmal pro Run):**
- `GET_DMP_Command_Configuration` (SharePoint `GetItems`, `$top=5000`) – lädt die GESAMTE Konfigurationsliste bei jedem einzelnen E-Mail-Run neu.

**B) Der eigentliche Verarbeitungspfad (DIS-Zweig, sequenziell, jede Aktion wartet auf die vorherige):**
1. `Get_Last_DMP_Internal_Sender_ID` (Excel `GetItem` – Zähler lesen)
2. `Create_DMP_Mailbox_Subfolder_"DMP_internal_Sender"` (Graph HTTP POST – oft HTTP 409 „existiert bereits", dann trotzdem weiter)
3. `Get_DMP_Mailbox_Subfolder_ID_for_"DMP_Internal_Sender"` (Graph HTTP GET)
4. `Reply_to_email_-_DMP_internal_Sender` (Office365 `ReplyToV3` – sendet die Bestätigungsmail)
5. `Update_Last_DMP_Internal_Sender_ID` (Excel `PatchItem` – Zähler zurückschreiben)
6. `Info_to_Hotline_team_about_arrival...` (Office365 `SendEmailV2` – interne Weiterleitung ans Hotline-Team)
7. **`Delay_(Wait_for_responded_DIS_E-Mail)_`** – der o.g. `WaitSecondsBeforeSentMailSearch`-Wait, **läuft auch im ganz normalen Erfolgsfall**, nicht nur bei Fehlern/Warnungen!
8. `Search_sent_mails_(DIS)_` (Graph HTTP GET, Sent-Items-Suche)
9. `Move_all_Mails_(DIS)` – Foreach über die gefundenen gesendeten Mails (typ. 2: Reply + Info-Mail), je 1 Graph HTTP POST („move") **pro Mail einzeln, nicht gebündelt**
10. Analoge Rename/Move-Schritte für die eingehende Original-Mail selbst (`Buffer_Audit_Event_-_Rename_Inbound_...`, `..._Move_Inbound_...`)

Entlang dieses Pfads werden **7 einzelne Audit-Events gepuffert** (PathSelected, Reply, InfoMail, SearchSent, MoveSent, RenameInbound, MoveInbound) – NICHT die vermuteten 1-2, sondern 7.

**C) Abschluss `Audit_Trail_Processing` (läuft danach immer, für jeden Run):**
- `Write_Audit_Trail_to_Excel` (1× `AddRowV2` für die RunSummary-Zeile)
- `Check_whether_there_are_unwritten_buffered_events_` → `Foreach` über alle 7 gepufferten Events, **PRO Event 3 sequenzielle Excel-Aufrufe**: `AddRowV2` (Zeile schreiben) + `GetItem` (aktuellen Step-Zähler aus `AgentAuditSummary` lesen) + `PatchItem` (neuen Step-Zähler zurückschreiben) → **7 × 3 = 21 Excel-Aufrufe**
- `GET_AgentSummaryRow_ForRun` (Excel `GetItem`) + `PATCH_AgentSummaryRow_ForRun` (Excel `PatchItem`) – 2 weitere Excel-Aufrufe für den Run-Zähler
- `GET_StatusRow_Agent_02` (SharePoint `GetItems`) + `UPDATE_StatusRow_Agent_02` (SharePoint `PatchItem`) – 2 weitere Aufrufe fürs Cockpit-Status-Update

**Ergebnis: ~26 sequenzielle Excel-Online-Business-Aufrufe** (1 RunSummary + 21 aus der Audit-Event-Schleife + 2 Zähler-Update + 2 Workflow-Counter Get/Update) **in einem einzigen, ganz normalen E-Mail-Durchlauf** – zusätzlich zu ~8-10 sequenziellen Microsoft-Graph-HTTP-Aufrufen und dem einen konfigurierbaren `Wait`.

**Warum das die 4-5 Minuten plausibel erklärt:** Der `shared_excelonlinebusiness`-Konnektor ist in der Power-Automate-Community notorisch der langsamste Standard-Konnektor, da jeder Aufruf die Excel-Datei serverseitig öffnen/parsen/sperren muss (typische Erfahrungswerte: 2-8 Sekunden PRO Aufruf, teils mehr bei gleichzeitigem Zugriff mehrerer Agenten auf dieselbe `AuditTrail.xlsx`). 26 Excel-Aufrufe × ~3-5s ≈ 80-130 Sekunden allein für die Audit-Buchhaltung – noch bevor der konfigurierte `WaitSecondsBeforeSentMailSearch`-Delay und die Graph-HTTP-Aufrufe dazugerechnet werden. Das erklärt die beobachteten ~4-5 Minuten ohne dass irgendetwas "kaputt" wäre – es ist ein Architekturmerkmal (sehr granulare Pro-Schritt-Audit-Protokollierung mit synchronem Zähler-Update), keine echte Fehlfunktion.

**Konkrete Optimierungsideen für eine spätere, gemeinsam mit dem Nutzer geplante Überarbeitung (absteigend nach Aufwand/Nutzen):**
1. **Größter Hebel, geringstes Risiko:** `WaitSecondsBeforeSentMailSearch` in der Config-Liste prüfen/reduzieren (s. o., reine Konfigsache).
2. **Größter struktureller Hebel:** Die Pro-Step-Zähler-Aktualisierung (`GET_AgentSummaryRow_ForStep`/`SET_NewStepsCount`/`PATCH_AgentSummaryRow_ForStep`, 2 Excel-Aufrufe PRO Audit-Event) aus der Foreach-Schleife herausnehmen und stattdessen NACH der Schleife einmalig für alle 7 Events zusammengefasst aktualisieren (z. B. Zähler pro `StepStatus` in einer lokalen Variable aufsummieren, dann 1× `GetItem`+`PatchItem` je vorkommendem Status statt 7×). Würde die 21 Aufrufe auf ggf. 2-4 reduzieren.
3. Prüfen, ob `GET_DMP_Command_Configuration` (Top 5000, komplette Liste) durch einen gefilterten Abruf ersetzt werden kann, falls die Liste stark wächst.
4. Die 2 Move-Aktionen (Reply + Info-Mail) in der Foreach-Schleife sind strukturell nicht vermeidbar (Graph-API bietet kein Batch-Move), aber da typischerweise nur 2 Elemente, geringer Einzeleinfluss.
5. **Bewusst nicht empfohlen ohne Rücksprache:** Das granulare Pro-Schritt-Audit-Modell selbst in Frage zu stellen – das war eine explizite frühere Anforderung (lückenlose Nachvollziehbarkeit) und sollte nicht ohne Abwägung geopfert werden, nur um Zeit zu sparen.

### ✅ UMGESETZT (2026-08-28): Struktureller Fix #2 (Zähler-Bündelung) in Agent 1 UND Agent 2 implementiert und deployt
Auf Nutzerwunsch ("Ja, umsetzen und gleich auch Agent 1 analog anpassen") wurde Optimierung #2 aus der Liste oben umgesetzt, in BEIDEN Agenten (Agent 2 war ursprünglich gemeldet, Agent 1 hat identisches Muster und wurde auf Nutzerwunsch gleich mit angepasst).

**Vorher (pro Agent):** Innerhalb der Foreach-Schleife über gepufferte Audit-Events: pro Event 1× `AddRowV2` (Zeile schreiben, bleibt) + 1× `GetItem` (Step-Zähler lesen) + 1× `PatchItem` (Step-Zähler schreiben) = 3 Excel-Aufrufe/Event. Danach zusätzlich 1× `GetItem` + 1× `PatchItem` für den separaten Run-Zähler. Bei 7 Events (typischer Agent-2-Erfolgspfad): 7×3 + 2 = 23 Excel-Aufrufe für die Zähler-Verwaltung allein.

**Nachher:** Vor der Schleife EINMALIG `GET_AgentSummaryRow_Combined` (liest Steps- UND Runs-Zähler in einem Aufruf). Direkt danach 3 rein lokale `SetVariable`-Aktionen (`SET_SucceededStepsDelta`/`SET_WarningStepsDelta`/`SET_FailedStepsDelta`), die per `length(filter(variables('AuditEvents'), equals(item()?['StepStatus'], '...')))` **ohne jeden API-Aufruf** zählen, wie oft jeder Status im Puffer vorkommt (bestätigt: in beiden Agenten kommen nur genau die 3 Werte "Succeeded"/"Warning"/"Failed" als `StepStatus` vor, geprüft per Volltextsuche). Danach `SET_NewRunsCount` (unverändert in der Berechnung, liest jetzt aber aus `GET_AgentSummaryRow_Combined`). Die Foreach-Schleife selbst schreibt jetzt NUR NOCH die Zeilen (`AddRowV2`, 1 Aufruf/Event, unvermeidbar). Nach der Schleife EIN EINZIGES `PATCH_AgentSummaryRow_Combined`, das per verschachtelten `if(greater(delta,0), ...)`-Ausdrücken nur die tatsächlich betroffenen Zähler-Felder (Succeeded/Warning/Failed StepsCount + deren LastUpdateUtc) UND den Runs-Zähler in einem JSON-Objekt zusammenbaut und in einem `PatchItem`-Aufruf schreibt.

**Ergebnis:** Aus 7×3+2 = 23 Excel-Aufrufen wurden 7×1 (AddRowV2) + 1 (GetItem) + 1 (PatchItem) = 9 Excel-Aufrufe — eine Reduktion um ca. 61%. Bei den vermuteten 2-8s/Aufruf entspricht das einer geschätzten Zeitersparnis von 1-2 Minuten pro E-Mail-Durchlauf.

**Technische Details:**
- Neue Aktionen (beide Agenten): `GET_AgentSummaryRow_Combined`, `SET_SucceededStepsDelta`, `SET_WarningStepsDelta`, `SET_FailedStepsDelta`, `SET_NewRunsCount` (angepasst), `PATCH_AgentSummaryRow_Combined`.
- Entfernte Aktionen: `GET_AgentSummaryRow_ForStep`, `SET_NewStepsCount`, `PATCH_AgentSummaryRow_ForStep` (aus der Schleife), `GET_AgentSummaryRow_ForRun`, `PATCH_AgentSummaryRow_ForRun` (danach).
- Die Zähler-Semantik ist exakt erhalten geblieben: nur Status, die im aktuellen Lauf tatsächlich vorkamen, werden inkl. `LastUpdateUtc` aktualisiert (kein Zähler wird "berührt", wenn er in diesem Lauf nicht vorkam) – identisch zum alten Verhalten.
- Agent 1 hat ein leicht abweichendes `AuditOutcome`-Handling (kein "Started"-Platzhalter, direkt `variables('AuditOutcome')` für den Runs-Zähler-Spaltennamen) – das wurde 1:1 aus dem Original übernommen, nur die Zähler-Bündelung ist neu.
- Beide JSON-Dateien nach der Änderung mit `ConvertFrom-Json` auf Syntax-Gültigkeit geprüft; alte Aktionsnamen per Volltextsuche als vollständig entfernt bestätigt.

**Versionen:** Agent 1 `[1.0.1]→[1.0.2]`, Agent 2 `[1.0.1]→[1.0.2]`. Solution-Version `7.10.25→7.10.27`.

**Deployment:** Solution neu gepackt (`pac solution pack`) und **erfolgreich importiert** (`pac solution import`) – Agent 1 und Agent 2 wurden dabei wie immer deaktiviert und **müssen vom Nutzer manuell reaktiviert werden**. Noch zu committen/pushen.

**Noch offen / zu beobachten:**
- Live-Verhalten nach Reaktivierung beobachten: insbesondere prüfen, ob die `AgentAuditSummary`-Zähler (Steps + Runs) nach ein paar echten Durchläufen weiterhin korrekt hochzählen (Soll: identische Endwerte wie vorher, nur mit weniger Zwischenschritten).
- Die tatsächliche Laufzeitverbesserung sollte der Nutzer nach der Reaktivierung live beobachten (z. B. nächste normale E-Mail durchlaufen lassen und Dauer vergleichen).
- Punkt 1 (`WaitSecondsBeforeSentMailSearch`-Konfigwert prüfen/reduzieren) ist weiterhin offen und separat vom Nutzer zu prüfen – bringt vermutlich zusätzlich zur jetzigen Optimierung nochmal spürbar Zeit.
- Punkt 3 (Config-Load ohne `$top=5000`-Vollabruf) weiterhin nur dokumentiert, nicht umgesetzt.




## ✅ GELÖST (2026-08-27, Fortsetzung 4): Container-Höhe, Rahmenfarbe, LED-Ausrichtung, Spalten-Flucht (Vorher/Nachher-Screenshot vom Nutzer)
Nutzer lieferte einen präzisen Vorher/Nachher-Vergleich (links aktuell, rechts gewünscht) mit 4 konkreten Punkten:
1. **✅ ECHTER BUG – Container viel zu hoch:** `conOperatingState` (als `ManualLayout` neu gebaut) hatte zwar `Height:=160`, aber KEIN `LayoutMinHeight`/`LayoutMaxHeight` – dadurch konnte der Elterncontainer (`conMiddleColumn`, gestreckt auf die volle Zeilenhöhe 360px durch `conRow1`) die Karte trotz gesetzter `Height` auf die volle 360px hochziehen. Fix: `LayoutMinHeight`/`LayoutMaxHeight: =160` ergänzt (das gleiche Prinzip, das bei `conMaintenanceDomains` bereits vorhanden war und dort nicht auftrat).
2. **✅ Rahmenfarbe:** Auf ausdrücklichen Wunsch des Nutzers (mit Referenzbild) von der zuvor eingeführten dezenten themenabhängigen Farbe zurück auf einen klar sichtbaren grünen Rahmen (`RGBA(0,206,125,0.7)`, passend zur Akzentfarbe der Titelzeile) geändert – für beide Container.
3. **✅ LED-Vertikalausrichtung:** LED-Punkte saßen nicht auf gleicher Höhe wie die Toggle-Beschriftung ("DMP"/"SIMU"). Y-Position der LEDs an die Toggle-Mittelachse angepasst.
4. **✅ Spalten-Flucht:** Die Werte von "Mode" und "Last Changed" mussten mit der Toggle-Spalte darunter auf gleicher X-Position beginnen (vorher hing "Mode" an einer separaten, weiter links liegenden Spalte). `lblModeValue.X` von 90 auf 220 verschoben (identisch zur Toggle-Spalte), `lblLastChangedValue` von rechtsbündig auf linksbündig geändert, um ebenfalls an dieser Spalte auszurichten. Container dafür von 520 auf 650px verbreitert (Wertespalten-Breite bleibt bei bewährten 414px erhalten), `conMiddleColumn` entsprechend auf 1316px.

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich. PowerApp-Version → `v1.5.1`, Solution-Version neu berechnet → `7.10.15` (Flows unverändert, kein Re-Import nötig).

## ✅ GELÖST (2026-08-27, Fortsetzung 3): Labels immer noch abgeschnitten trotz ManualLayout + Titel-Hierarchie
Trotz des ManualLayout-Umbaus waren Labels weiterhin abgeschnitten (z. B. "Mode" → "Mo…", "Operational Mode" → "Operational M…") – das bestätigt: die reale Zeichenbreite von Segoe UI Bold/Semibold bei den verwendeten Schriftgrößen ist deutlich größer als ursprünglich angenommen, unabhängig vom Layout-Mechanismus (nicht mehr FillPortions/Flex-bezogen, sondern schlicht zu knapp kalkulierte feste Breiten).
1. **✅ Alle Label-Breiten großzügig neu berechnet** (mit deutlichem Sicherheitspuffer statt knapper Schätzung): "Operational Mode"/"Environment"-Labelspalte einheitlich auf 190px, Mode-Wertzeile auf kleinere Schriftgröße (12.5→10.5) UND mehr Breite (414px) umgestellt, da der volle ausgeschriebene Text ("SIMULATION - Normal non-DMP Operation") im Extremfall ca. 450-500px benötigt.
2. **✅ `conOperatingState` und `conMaintenanceDomains` von 440px auf 520px verbreitert** (beide weiterhin identisch groß), `conMiddleColumn` entsprechend auf 1056px.
3. **✅ Container-Überschriften werten optisch auf:** "OPERATING STATE"/"MAINTENANCE - DOMAINS" von klein/grau/Semibold (Size 11) auf die App-Akzentfarbe Grün, Bold, Size 12 geändert – vorher dominierten die fett-weißen Datenzeilen-Labels optisch über die Titelzeile, was der Nutzer zu Recht als unergonomisch bemängelte.
4. **Risiko (wächst weiter):** Gesamtbreite der mittleren Zeile jetzt ca. 376+1056+400+Abstände ≈ 1850px+Sidebar – auf Bildschirmen unter ca. 2000px Breite ist horizontales Scrollen wahrscheinlich. Muss beim nächsten Live-Test geprüft werden; ggf. müssen wir die Schriftgrößen weiter reduzieren statt die Container immer weiter zu verbreitern.

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich. PowerApp-Version → `v1.5.0`, Solution-Version neu berechnet → `7.10.14` (Flows unverändert, kein Re-Import nötig).

## ✅ GELÖST (2026-08-27, Fortsetzung 2): `conOperatingState` komplett auf manuelle Positionierung umgebaut, feste Containergrößen überall
Nach erneutem Live-Test (Labels weiterhin abgeschnitten, LED weiterhin ein Balken trotz `FillPortions:=0`-Fix) hat der Nutzer vorgeschlagen, komplett mit festen Containergrößen statt AutoLayout-Flex zu arbeiten:
1. **✅ `conOperatingState` von AutoLayout (verschachtelte Zeilen-Container) auf `Variant: ManualLayout` mit expliziten `X`/`Y`-Koordinaten umgebaut** – identisches Muster wie die bereits zuverlässig funktionierende Kopfzeile (KPI-Werte, Versions-Tag). Dadurch entfallen sämtliche verschachtelten Zeilen-Container (`conRowMode`, `conRowLastChanged`, `conRowOperationalState`, `conRowApplicationMode`) – das behebt vermutlich auch die vom Nutzer bemängelte Rahmen-Inkonsistenz zwischen Light/Dark Mode (diese Zeilen-Container hatten keine explizite `BorderColor`/`BorderThickness`, was in einigen Rendering-Situationen einen unbeabsichtigten Rahmen zeigen konnte).
2. **✅ Alle Labels bekommen jetzt großzügige, fest positionierte Breiten** (kein Konkurrieren mehr um Flex-Anteile), LED-Punkte auf 18px vergrößert.
3. **✅ Feste Containergrößen statt Flex überall in dieser Zeile:** `conOperatingState` (440×160) und `conMaintenanceDomains` (440×160, jetzt exakt gleich groß – "harmonisches Bild") stehen nebeneinander in `conMiddleColumn` (jetzt ebenfalls fest 896px breit statt flexibel). `conEmailsCard` (Agent 2 / Emails Processed) von 228px auf 400px verbreitert (mind. so breit wie System Health, plus Platz für die Legende-Labels). Toter Abstandshalter in der "Internal"-Zeile von 90px auf 40px verkleinert, um Platz zu sparen.
4. **Risiko/offener Punkt:** Die Gesamtbreite der Zeile (System Health 376px + Middle Column 896px + Emails 400px + Abstände) ist jetzt spürbar größer als vorher – auf kleineren Bildschirmen (unter ca. 1900px Breite) könnte das zu horizontalem Scrollen führen. Muss im nächsten Live-Test geprüft werden.

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich. PowerApp-Version → `v1.4.9`, Solution-Version neu berechnet → `7.9.23` (Flows unverändert, kein Re-Import nötig).

## ✅ GELÖST (2026-08-27, Fortsetzung): Echter FillPortions-Bug (LED war ein Balken), SIMU→PROD-Sicherheitslogik, Modus-Rahmen, Layout-Umbau
Nach Live-Test durch den Nutzer stellte sich heraus, dass die "LED" tatsächlich als langer grauer Balken erschien (nicht nur im Mockup, sondern auch nach dem Redesign) und mehrere Labels abgeschnitten waren ("M...", "Operational ...", "Not changed thi..."):
1. **✅ ECHTER BUG – Ursache gefunden:** In Power Apps AutoLayout wird eine explizite `Width`/`Height` bei einem Kind-Element IGNORIERT, wenn `FillPortions` nicht zusätzlich explizit auf `0` gesetzt ist – das Element wird stattdessen flexibel gleich-verteilt (identisches Muster wie der frühere `conHeartbeatCard`-Bug, diesmal aber auch bei Label-Controls, nicht nur GroupContainer). Betroffen: beide LED-Dots (`dotLedOperationalState`/`dotLedApplicationMode`) und mehrere Labels (`lblModeLabel`, `lblLastChangedLabel`, `lblOperationalStateLabel`, `lblApplicationModeLabel`). Fix: `FillPortions: =0` überall ergänzt, wo eine feste Breite gelten soll; das jeweils flexible Geschwisterelement bekommt `FillPortions: =1`.
2. **Neue Regel in den Arbeitsregeln verankert:** Diese FillPortions-Regel gilt für JEDES Kind in einem AutoLayout-Container (Label, GroupContainer, etc.), nicht nur für GroupContainer.
3. **✅ Sicherheitslogik ergänzt:** Beim Umschalten SIMU→PROD wird der Operational Mode jetzt automatisch auf "Normal" zurückgesetzt (sendet immer `PROD_NODMP`, unabhängig vom vorherigen SIMU-Zustand) – verhindert versehentliches Live-Gehen mit noch aktiver DMP-Testlogik. Nutzt den bereits etablierten `varReadyToLiftToggleGuard`-Mechanismus, um einen verzögerten Doppel-Trigger durch die Default-Änderung des anderen Toggles zu vermeiden.
4. **✅ Neuer modusabhängiger Bildschirmrahmen:** 4 neue schlanke Rahmenleisten (oben/unten/links/rechts, `conFrameTop/Bottom/Left/Right`) um den gesamten Bildschirm, Farbe UND Dicke abhängig vom aktuellen Modus: Grau/3px (SIMU-NonDMP) → Grün/4px (PROD-NonDMP) → Gelb/6px (SIMU-DMP) → Rot/9px (PROD-DMP, kritischste Kombination). Rein deklarativ aus `varEnvironmentIsPROD`/`varOperationalModeIsDMP` berechnet, keine zusätzliche Zustandspflege nötig.
5. **✅ Layout-Umbau `conMiddleColumn`:** Von vertikal auf horizontal umgestellt – "Operating State" (jetzt fest 340px breit) und "Maintenance Domains" (nimmt Restbreite) stehen jetzt nebeneinander statt untereinander. `conMaintenanceDomains` bekam außerdem den gleichen alten auffälligen grünen Rahmen wie `conOperatingState` entfernt (jetzt konsistentes dezentes Rahmenmuster).
6. **✅ `conEmailsCard` (Agent 2 / "Emails Processed") verkleinert:** Gleiches Muster wie beim System-Health-Ring – `FillPortions: =0`, feste Breite 228px (180px Ring + 24px Padding je Seite) statt gleichmäßiger Flex-Aufteilung mit der Mittelspalte.

**Deployment:** Gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen, beide Dateien) erfolgreich. PowerApp-Version → `v1.4.8`, Solution-Version neu berechnet → `7.9.22` (Flows unverändert, kein Re-Import nötig).

## ✅ GELÖST (2026-08-27): `conOperatingState`-Container komplett neu gestaltet
Nutzer bat um ergonomische Überarbeitung des "OPERATING STATE"-Containers (2. Container in der GUI-Überarbeitungsreihe nach Header + Health-Ring):
1. **Einzelne runde LED statt Balken:** Die 2 Status-Punkte (`dotLedOperationalState`/`dotLedApplicationMode`) waren bereits einzelne runde Elemente, bekamen aber einen 4. Zustand ergänzt: **Grau** = Status unklar/wird geladen (NEU – vorher startete die App fälschlich direkt mit "Grün", obwohl der echte Serverstatus noch gar nicht geladen war), Grün = bestätigt aktiv, Gelb = Umschaltung läuft, Rot = letzte Umschaltung fehlgeschlagen. Dots vergrößert (12→16px) mit dezentem dunklem Rand für einen "echten LED"-Look.
2. **Blinken bei Gelb UND Rot:** Neue Variable `varBlinkPhase`, umgeschaltet im ohnehin laufenden 1-Sekunden-Timer (`tmrAutoRefreshTick`) – LED blinkt (Opazität 100%/25% im Sekundentakt) während einer laufenden Umschaltung UND bei einem fehlgeschlagenen letzten Versuch, um Aufmerksamkeit zu erzwingen. Grau/Grün bleiben ruhig/durchgehend.
3. **Modus als ausgeschriebener Text statt Abkürzung:** `SIMU_NODMP` → z. B. "SIMULATION - Normal non-DMP Operation". Kein Zeilenumbruch (explizit `Wrap:=false`), stattdessen Zeile umstrukturiert (Label "Mode" schmal links, Wert linksbündig mit vollem Restplatz).
4. **2 Labels vergrößert:** "Operational Mode"-Zeilenlabel und "Last Changed"-Wert (beide vom Nutzer im Screenshot markiert) – Schriftgröße 12→13, Breiten angepasst, kein Umbruch mehr.
5. **Container deutlich verkleinert:** Höhe 182→150px (Innenabstände 12→6, Zeilenabstand 6→4, Zeilenhöhen leicht gestrafft).
6. **Rahmen vereinheitlicht:** Der auffällige dicke grüne Rahmen (`RGBA(0,206,125,0.55)`) ersetzt durch das im Rest der App verwendete dezente, themenabhängige Rahmenmuster – behebt gleichzeitig die vom Nutzer bemängelte Light-Mode-Übersichtbarkeit UND schafft optische Konsistenz mit allen anderen Karten.
7. **Toggle-Beschriftungskontrast (Light Mode):** `Color` der beiden Toggles (`tglOperationalState`/`tglApplicationMode`) von fest-weiß auf themenabhängig (dunkel in Light Mode, weiß in Dark Mode) geändert – vermutliche Ursache des "weiß auf grau"-Problems war, dass der Text teilweise außerhalb der farbigen Toggle-Fläche auf dem Karten-Hintergrund sitzt, der sich mit dem Theme ändert.

**Deployment:** `App.pa.yaml`+`scrHome.pa.yaml` gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen, beide Dateien) erfolgreich. PowerApp-Version → `v1.4.7`, Solution-Version neu berechnet → `7.9.21` (Flows unverändert, kein Re-Import nötig).

## ✅ GELÖST (2026-08-27): Verbindungs-/Publish-Probleme nach App-Update + 2 weitere echte Bugs
1. **`FlowNotFound`-Fehler beim Öffnen der App:** Bestätigt als bekanntes Verhalten – nach jedem `pac solution import` kann die interne Verbindungs-Referenz zu Agent 4 veralten, unabhängig davon ob sich die Schnittstelle geändert hat (frühere Annahme "nur bei Schema-Änderung nötig" war FALSCH, hiermit korrigiert). Workaround bleibt: Agent-4-Datenquelle entfernen + neu einbinden, danach **Speichern + Veröffentlichen** (sonst geht die Korrektur bei Neuladen/Schließen wieder verloren).
2. **Veröffentlichen schlug fehl ("app version was not found"):** Ursache war ein veralteter Studio-Tab-Cache (Version vor dem letzten Import). Fix: Tab komplett schließen, App über make.powerapps.com neu öffnen (nicht wiederverwenden).
3. **"Schreibgeschützt"-Meldung beim Speichern:** Bekanntes Sperr-Verhalten – `pac solution import` zählt selbst als Bearbeitungssitzung. Nutzer hat mit anderem Browser erfolgreich veröffentlicht.
4. **✅ ECHTER BUG – "Live status successfully retrieved..." erschien SOFORT beim Start des Refreshs, nicht erst nach Antwort von Agent 4:** `varStatusCallError` wurde zu Beginn des Refreshs auf `""` gesetzt (noch vor dem eigentlichen Aufruf), und genau dieses Feld steuerte die Erfolgsanzeige. Fix: `lblNextStep1Done`/`lblNextStep1Detail` prüfen jetzt zusätzlich `varIsRefreshing` und zeigen währenddessen einen neutralen "wird geladen…"-Text.
5. **✅ Rahmen verschwinden bei Zoom <100% (bestätigt: kein Browser-Unterschied, reines Zoom-Rendering):** `BorderThickness` global von `2` auf `3` erhöht (alle 23 Vorkommen in `scrHome.pa.yaml`) für mehr Toleranz beim Sub-Pixel-Runden.
6. **Kontrast der Schrift im Health-Ring (Light Mode) vom Nutzer als "ok" bestätigt** – kein weiterer Handlungsbedarf.

**Deployment:** `scrHome.pa.yaml` gepackt (`pac canvas pack`), Round-Trip-Verifikation (0 Diff-Zeilen) erfolgreich. PowerApp-Version → `v1.4.6`, Solution-Version neu berechnet → `7.9.20` (Flows unverändert, kein Re-Import nötig). Committed, noch zu pushen.

## Was heute (2026-08-26, zweite Tageshälfte) gemacht wurde – GUI-Redesign `conTopHeader` + mehrere echte Bugs
Nutzer bat um Fortsetzung der Design-/UX-Überarbeitung, beginnend bei der Kopfzeile (`conTopHeader` in `scrHome.pa.yaml`). Im Rahmen dieser Arbeit wurden mehrere ECHTE, unabhängige Funktionsfehler gefunden und behoben (nicht nur Kosmetik):

1. **✅ Kopfzeilen-Redesign:** Schrift vergrößert (Titel, KPI-Werte CRITICAL/WARNINGS/AGENTS ACTIVE, Labels), Innenabstände reduziert, neue Statuszeile "Updated: HH:MM:SS · Next update in MM:SS" ergänzt.
2. **✅ Neuer Auto-Refresh-Mechanismus:** Nutzer wollte selbst wählbares Intervall (Off/Now/2m/5m/10m/15m) statt festem Intervall. Nach mehreren Fehlversuchen (Timer-Start-Formel erkennt keinen false→true-Übergang, Timer.Duration-Laufzeit-Umschaltung unzuverlässig) wurde eine ROBUSTE Lösung gebaut: fester 1-Sekunden-Tick-Timer (`tmrAutoRefreshTick`, Duration/Repeat/AutoStart/Start alle als Literal `true`/`1000` – NIE mehr formelgebunden) plus eigene Zählvariable `varAutoRefreshSecondsRemaining`. Die Auswahl-Buttons setzen diese Variable direkt, keine Abhängigkeit mehr von `Reset()`.
3. **✅ "Now"-Button:** Manueller Sofort-Refresh, zeigt sich vor dem ersten erfolgreichen Lauf grün hervorgehoben, danach neutral/weiß wie die anderen Buttons (verhindert Verwechslung mit dem "Off"-Auswahlzustand).
4. **✅ "⟳ Refresh ongoing…"-Statusanzeige** während jedes Agent-4-Aufrufs (Start, Now-Klick, Timer-Trigger) – ein Aufruf kann bis zu 1 Minute dauern (Agent 4 liest viele Datenquellen).
5. **✅ ECHTER KRITISCHER BUG gefunden und behoben – Agent 4 schlug komplett fehl:** `SCOPE_PowerAppVersion_Read`s letzte Aktion lief nur bei Erfolg des vorherigen GET-Schritts (`runAfter: [Succeeded]`) – exakt dasselbe Scope-Status-Muster wie der frühere Agent-5-Bug, nur diesmal auf FLOW-Ebene: ein einzelner fehlgeschlagener Dateizugriff (Versions-Datei) ließ den GESAMTEN Agent-4-Lauf als "Failed" gelten, wodurch Power Apps' `IfError(...)` die komplette (eigentlich gültige) Antwort verwarf und überall Fallback-Nullen zeigte. Fix: letzte Aktion läuft jetzt immer (`Succeeded, Failed, Skipped, TimedOut`), Wert per Compose-Zwischenschritt abgesichert (SetVariable-Selbstreferenz-Regel beachtet).
6. **✅ ECHTER BUG – Versions-Box zeigte rohen Fehler-JSON (`{"status":404...}`):** `coalesce(...)` allein reichte nicht, weil eine fehlgeschlagene GET-Aktion trotzdem einen nicht-leeren Fehler-Body zurückgibt. Fix: `CMP_PowerAppVersionResolved` prüft jetzt explizit den echten Aktionsstatus (`actions(...)['status']='Succeeded'`), bevor der Body verwendet wird.
7. **✅ ECHTER BUG – Versions-Datei wurde trotzdem nicht gefunden (404):** Der Nutzer fand in der rohen Power-Automate-Eingabe einen fehlerhaften Pfad mit angehängtem `\n` (`.../PowerApp_Version.txt\n`) – vermutlich ein Zeilenumbruch in den SharePoint-Konfigurationswerten `PowerAppVersionFolderName`/`PowerAppVersionFileName`. Fix: beide Config-Werte werden jetzt vor der Pfad-Bildung mit `trim(...)` bereinigt. **Noch nicht behoben: die Konfigurationswerte selbst in der SharePoint-Liste `DMP Command Configuration` enthalten vermutlich weiterhin den Zeilenumbruch** – der `trim()`-Fix maskiert das Symptom zuverlässig, sollte aber langfristig auch an der Quelle bereinigt werden (Nutzer könnte die beiden Zellen in der Liste einmal neu eintippen, ohne Copy-Paste-Zeilenumbruch).
8. **✅ ECHTER BUG – automatischer Refresh beim App-Start funktionierte nicht:** `App.OnStart` ist zu früh im App-Lebenszyklus, Datenverbindungen manchmal noch nicht bereit (bekannte Power-Apps-Einschränkung). Fix: Aufruf nach `scrHome.OnVisible` verschoben, PLUS ein redundanter "Kickstart"-Timer (`tmrInitialKickstart`, feuert einmalig nach 800ms mit literalem `AutoStart=true`/`Start=true`) als zusätzliche Absicherung, falls `OnVisible` beim allerersten Bildschirm nicht zuverlässig feuert (dokumentierte Power-Apps-Unschärfe). Beide teilen sich die Sperre `varInitialRefreshDone`, sodass die Aktualisierung garantiert nur einmal passiert.
9. **✅ ECHTER BUG – kaputte Agent-4-Datenquellen-Verbindung:** Nach mehreren `pac solution import`-Durchläufen zeigte Power Apps Studios Datenquellen-Panel bei "DMP Agent 4 (Status Check)" keinen technischen Namen mehr an (im Gegensatz zu Agent 3/5) – ein Indiz für eine verwaiste Verbindung, identisch zum früheren Agent-6-Vorfall. Nutzer hat alle 4 Datenquellen in Studio manuell aktualisiert ("Refresh"), danach lief Agent 4 wieder zuverlässig. **Root Cause dieses wiederkehrenden Verbindungsproblems ist weiterhin nicht 100% geklärt** – tritt scheinbar nach mehrfachem Solution-Import auf. Bei erneutem Auftreten: Datenquellen-Panel in Studio prüfen, ggf. alle Flows manuell auffrischen.
10. **✅ ECHTER BUG – Spurious "Operating State switched to..."-Meldung ohne Nutzeraktion:** Die Toggle-Sperre (`varSuppressToggleEvents`) wurde direkt am Ende derselben Formel aufgehoben, in der die Toggle-`Default`-Werte gesetzt wurden – die dadurch ausgelöste `OnCheck`/`OnUncheck`-Reaktion des Controls kommt aber erst einen Render-Zyklus SPÄTER an, wodurch die Sperre zu diesem Zeitpunkt fälschlich schon offen war. Fix: Sperre wird jetzt garantiert einen vollen 1-Sekunden-Timer-Tick später aufgehoben (`varReadyToLiftToggleGuard`-Zwischenvariable).
11. **✅ Layout-Feinschliff:** KPI-Schriftgrößen vereinheitlicht (waren durch einen Zwischen-Edit inkonsistent geworden), Versions-Box mehrfach neu bemessen (aktuell 95px breit, ausreichend für längere Versionsnummern), rechte Kopfzeilen-Gruppe (Version/DARK/Toggle/LIGHT) neu positioniert um Überlappungen zu vermeiden, Button-Reihe näher an Statustext gerückt, gesamte Zeile leicht nach unten verschoben.

**Alle Änderungen committed und zu GitHub gepusht.** Wichtigste Commits (chronologisch): `ac38505`, `d21d4a4`, `5b66688`, `73cba0f`, `c3a5619`, `9c6fdf3`, `7bc9fc4`, `2ba53e9`, `1fba44a`, `bca8017`, `5586d06`, `1035a47` (jeweils gefolgt von einem kleinen "remove temp commit message file"-Cleanup-Commit).

## ⏸️ Noch offen / nächste Schritte
1. **GUI-Redesign restliche Container:** Nur `conTopHeader` ist bisher überarbeitet. Noch ausstehend (in dieser Reihenfolge sinnvoll, da so im Screen angeordnet): `conHeartbeatCard` (System-Health-Ring) → `conOperatingState`/`conMaintenanceDomains` (conMiddleColumn) → `conEmailsCard` → `conFilesRow` → `conNextSteps`/`conAutomationStatus` → `conSidebar`/`conAdminFunctions`. Genauer Umfang pro Container mit Nutzer klären, bevor Änderungen vorgeschlagen werden (wie bei `conTopHeader`: Nutzer gibt konkrete Kritik anhand von Screenshots, KI setzt gezielt um).
2. **SharePoint-Konfigurationswerte bereinigen (klein, aber real offen):** `PowerAppVersionFolderName`/`PowerAppVersionFileName` in der Liste `DMP Command Configuration` enthalten vermutlich einen Zeilenumbruch (siehe Punkt 7 oben) – Nutzer sollte die Zellen bei Gelegenheit neu eintippen, auch wenn der `trim()`-Fix das Symptom bereits zuverlässig abfängt.
3. **Agent 2 Performance-Untersuchung** (~4-5 Min/E-Mail beobachtet) – noch nicht begonnen.
4. **Agent 3 Alert-Mail-Kette** noch nicht live getestet (nur Agent 5s Kette wurde bisher bestätigt).
5. **Wiederkehrendes Datenquellen-Verbindungsproblem nach Solution-Import** (Punkt 9 oben) – Ursache nicht abschließend geklärt, nur der Workaround (manuelles Refresh in Studio) ist bekannt. Falls es wieder auftritt, IMMER zuerst das Studio-Datenquellen-Panel auf fehlende technische Namen prüfen.
6. **Preisgespräch/Kosten-Übersicht** wurde in einer früheren Teilsitzung heute besprochen (Nutzer wollte ein Gefühl für die Kosten der Arbeit mit der KI bekommen) – Details dazu nicht in diesem Dokument, ggf. beim Nutzer nachfragen falls relevant für Fortsetzung.

---



## ✅ GELÖST (2026-08-26): "NEXT STEPS"-Karte zeigt jetzt echte Default-Management-Prozess-Meilensteine
Auf Nutzerwunsch wurde die bisherige statische "NEXT STEPS"-Karte (Automatisierungs-/Konfigurations-Checks wie "Configuration loaded", "Agent 1 domain extraction") umbenannt zu **"AUTOMATION STATUS"** (Inhalt unverändert) und durch eine NEUE, echte "NEXT STEPS"-Karte ersetzt, die reale organisatorische Meilensteine des Default Management Process zeigt.

**Datenquelle:** `Status DMP Process.xlsx` (Shared Documents/General), Blatt "Overall Process" – ein 44-zeiliger Meilensteinplan (Phase/ID/Milestone/Responsibility/Status) über 3 Phasen (Pre-default, nach Termination/vor Liquidation, nach Completion of Liquidation). Die Status-Spalte wird per Excel-Formel live aus 4 separaten Team-Checklisten (CoS Leader, Infrastructure, Hotline, Content Team – je eigene SharePoint-Site) zusammengeführt; diese 4 Dateien werden NICHT direkt angebunden (fremde Sites, zusätzliche Berechtigungen nötig), nur die bereits aggregierte Overall-Process-Datei.

**Voraussetzung (vom Nutzer erledigt):** Der lose Zellbereich A1:E45 wurde einmalig in eine echte Excel-Tabelle "OverallProcess" umgewandelt (Strg+T), da die Standard-Excel-Connector-Aktionen nur mit echten Tabellen funktionieren.

**Umsetzung:**
- Neuer Scope `SCOPE_NextMilestones_Read` in Agent 4: liest alle Zeilen der `OverallProcess`-Tabelle, füllt die (aufgrund verbundener Zellen) nur einmalig pro Phasenblock gefüllte `Phase`-Spalte für jede Zeile vorwärts auf, sammelt die ersten 5 Zeilen mit Status ≠ "done" in ein neues Antwortfeld `nextmilestones` (Array aus phase/milestone/responsibility/status).
- Power App: neue Variable `varNextMilestones`, 5 fest verdrahtete, per `CountRows(...)>=N` abgesicherte Zeilen-Slots in der neuen "NEXT STEPS"-Karte, "ongoing"-Meilensteine farblich hervorgehoben, Fallback-Text bei 0 offenen Meilensteinen.

**3 echte Power-Automate-Plattform-Fehler bei der Aktivierung gefunden und behoben (wichtige Lehren, jetzt in KI-Arbeitsregeln verankert):**
1. **`InitializeVariable`-Aktionen dürfen NIEMALS innerhalb eines `Scope` verschachtelt sein** – Fehler `InvalidVariableInitialization`. Die beiden neuen Variablen (`LastSeenPhase`, `NextMilestonesArray`) mussten auf die oberste Flow-Ebene verschoben werden.
2. **`SetVariable` darf sich NIEMALS selbst referenzieren** (`variables('X')` innerhalb der eigenen Wertzuweisung von `X`) – Fehler „Self reference is not supported". Der „Phase vorwärts auffüllen"-Schritt musste über eine zwischengeschaltete `Compose`-Aktion umgeleitet werden (Wert erst berechnen, dann erst in einem separaten Schritt zuweisen).
3. **Verbindungsschema-Cache (`DataSources.json`) wieder veraltet** (gleiche Fehlerklasse wie gestern bei `agent6healthy`) – diesmal wegen eines komplexeren neuen Array-Feldes zu riskant für manuelles Patchen. Stattdessen über den offiziellen Weg gelöst: Datenquelle für Agent 4 in Studio entfernen + neu hinzufügen. Dabei traten 2 weitere Studio-Eigenheiten auf, die den Fix zunächst verschleierten: ein vorübergehender "Schreibgeschützter Modus"-Zustand (durch Neuladen der Seite gelöst) und eine hartnäckig eingefrorene Vorschau-Session, die trotz mehrfachem Vorschau-Neustart alte Variablenwerte zeigte (erst ein KOMPLETTER Browser-Neustart hat das behoben).

**Ergebnis:** Vom Nutzer live bestätigt – die Karte zeigt jetzt echte Meilensteine (z. B. „Pre-Default communication assessment · CoS Leader · not started").

**Versionshinweis:** Agent 4 blieb bei `[1.2.0]` (keine Erhöhung), da diese Version nie erfolgreich aktiviert war, bevor alle 3 Fixes eingespielt wurden – gemäß Konvention keine Versionserhöhung für einen Stand, der nie erfolgreich live war. Solution-Version → `7.10.7`.

## ✅ GELÖST (2026-08-26): Echter Root Cause der „Mailbox Evidence Folder Setup Failed"-Fehlalarme gefunden
Beim Live-Test des gestrigen Doppelauslösungs-Fixes trat ein NEUER, konkreterer Fehler auf: `Create_Mailbox_Subfolder_"PA_Processed_Mails"` schlug mit `Body: {"displayName": ""}` fehl – der Ordnername war LEER, nicht nur „bereits vorhanden".

**Root Cause:** Agent 5s eigener `GET_DMP_Command_Configuration`-Filter (`$filter`) für `Scope eq 'Global'` erlaubte nur `Title eq 'CurrentOperationMode' or Title eq 'AlertEmailRecipient' or Title eq 'SharedDMPMailbox'` – die Felder `ProcessedMailsRootFolderName`, `MailImportanceError` und `WaitSecondsBeforeSentMailSearch` (alle von Agent 5 aus `CMP_ConfigObject` referenziert) waren NIE Teil dieses Filters und wurden daher NIE geladen. Verifiziert durch direkte Prüfung der `GET_DMP_Command_Configuration`-Ausgabe im Power-Automate-Laufverlauf: Die Konfigurationszeile fehlte komplett in der Antwort, obwohl der Wert in der SharePoint-Liste selbst korrekt gesetzt war (vom Nutzer bestätigt: „PA Processed Mails" in allen 4 Modus-Spalten).

**Wichtige Einordnung:** Das war vermutlich schon SEIT LANGEM so und die eigentliche Ursache für die Alarm-Mails, die schon vor dem gestrigen Fix auftraten – der gestrige Fix (echte Ordner-ID-Prüfung statt Scope-Status) war trotzdem korrekt und notwendig, hat aber lediglich dafür gesorgt, dass dieser tieferliegende, echte Fehler jetzt KORREKT als Fehlschlag erkannt und gemeldet wird, statt (wie vorher) durch einen anderen Bug zufällig verdeckt zu werden.

**Fix:** Die 3 fehlenden Titel zum Global-Scope-Teil von Agent 5s Konfigurationsfilter ergänzt. Version → `[1.1.5]`. Deployt, vom Nutzer live bestätigt: Toggle-Test zeigt jetzt nur noch 1 Agent-5-Lauf UND keine Fehlalarm-Mail mehr.

## ✅ GELÖST (2026-08-26): Agent 6 Verbindungsfehler beim ersten echten Testlauf
Beim ersten echten Klick auf „Yes, delete" trat `InvokerConnectionOverrideFailed` auf („Could not find any valid connection for connection reference name 'shared_sharepointonline'"). Behoben durch den Nutzer über den „Aktualisieren"-Knopf bei allen 3 Datenquellen-Verbindungen von Agent 6 im Power-Apps-Studio-Datenbereich. Danach lief der komplette Löschvorgang (Delete... → Bestätigungsdialog → Yes, delete) erfolgreich durch.

## ✅ GELÖST (2026-08-26): Agent 6 vollständig in Audit Trail und Cockpit-Monitoring integriert
Auf ausdrücklichen Nutzerwunsch („Komplette Integration von Agent 6 (Audit Trail / System health etc.,!!!)") wurde Agent 6 vollständig gleichgestellt:

**1) Vorbereitung – neue Datenzeilen (vom Nutzer manuell über die normale Weboberfläche angelegt, da kein direkter Browser-Zugriff mit angemeldeter Session für die KI möglich war):**
- `AuditTrail.xlsx`, Blatt „Agent Audit Summary" (Tabelle `AgentAuditSummary`): neue Zeile `AgentKey = "Agent 06"`, alle Zähler-Spalten `0`, Zeitstempel-Spalten leer.
- `AuditTrail.xlsx`, Blatt „Audit Acknowledgment" (Tabelle `AuditAcknowledgment`): neue Zeile `AgentKey = "Agent 06"`, alle Baseline-Spalten `0`, Zeitstempel-Spalten leer.
- SharePoint-Liste `DMP Command Agent Status`: neuer Eintrag `Title = "Agent_06"`, `AgentKey = "Agent_06"`, `AgentDisplayName = "Admin Functions"`, alle übrigen Felder leer (werden vom Flow beim ersten Lauf befüllt).

**2) Agent 6 (`DMPAgent6AdminFunctions...json`) – neue Audit-/Status-Kette, nach jeder Admin-Aktion:**
- Neue Variablen `AuditOutcome`/`WorkflowPath` (Wert: `Agent6_AdminFunctions`, bewusst als Literal statt neuem Config-Feld, um keine ungefragte Config-Änderung vorzunehmen).
- `SET_AuditOutcome_FromResult` → `SCOPE_AuditTrail_Write` (`WRITE_RunSummary_To_AuditTrail`, schreibt eine Zeile pro Lauf in die zentrale `AuditTrail`-Tabelle) → `SCOPE_AuditSummary_Write` (Runs-Zähler in `AgentAuditSummary`, Zeile „Agent 06") → `SCOPE_StatusRow_Update` (Dashboard-Zeile „Agent_06" in `DMP Command Agent Status`) → `RESPOND_Result`.
- **Bewusste Design-Entscheidung:** Kein granularer Pro-Schritt-Audit wie bei Agent 1/2/3/5 (mit `LOOP_AuditEvents`/Steps-Zähler), sondern das einfachere Agent-4-Muster (nur ein Runs-Zähler pro Lauf), da Agent 6 wie Agent 4 ein linearer Ablauf ohne mehrstufige interne Verzweigung ist – sachlich passender als das komplexere Muster erzwungen nachzubilden.
- Jeder nachgelagerte Schritt läuft mit allen 4 Status (`Succeeded/Failed/Skipped/TimedOut`) der vorherigen Aktion, damit (Lehre von gestern) kein einzelner Fehlschlag die gesamte Kette blockiert. Version → `[1.1.0]`.

**3) Agent 4 (`DMPAgent302StatusCheck...json`) – Aggregation um Agent 6 erweitert:**
- Neue Aktion `GET_AuditSummary_Agent06` in `SCOPE_AuditSummary_Read`, absichtlich fehlertolerant verdrahtet (`Succeeded/Failed/Skipped/TimedOut`), damit ein (noch) nicht vorhandener Zeilen-Datensatz die gesamte Statusabfrage nicht blockiert (gleiche Lehre wie beim Agent-5-Bugfix von gestern).
- `SET_AggregatedFailedStepsCount`/`SET_AggregatedWarningStepsCount`/`CMP_TotalRunsPerAgent`/`SET_AggregatedRunSummaryCount` erweitert, um Agent 06 in die System-weiten Critical/Warnings/Gesamtlauf-Zahlen einzurechnen.
- Neue Variable/Ausgabe `Agent6Healthy` (boolean): `true`, wenn `FailedRunsCount` für „Agent 06" gleich 0 ist (Default `true`, damit ein noch nie gelaufenes Admin-Tool nicht fälschlich als „ungesund" gilt). Neues Response-Feld `agent6healthy`. Version → `[1.1.0]`.

**4) Power App – „AGENTS ACTIVE"-Kachel von X/5 auf X/6 umgestellt:**
- Neue Variable `varAgent6Healthy` (Default `true`, aus `varStatusResult.agent6healthy` aktualisiert).
- `varAgentsHealthyCount`-Formel um `If(varAgent6Healthy, 1, 0)` ergänzt, `varSystemHealthPercent` von `/5` auf `/6` umgestellt.
- Anzeige-Labels in `scrHome.pa.yaml` (Live-Kachel) und `scrHeaderTest.pa.yaml` (unbenutztes Test-Labor) von „/5" auf „/6" und Schwellenwert-Vergleich (`>=5` → `>=6`) angepasst.

**Deployment:** Alle 6 Flows + Power App gepackt (`pac canvas pack`, Round-Trip-Verifikation: 0 Diff-Zeilen), Solution gepackt/importiert (`pac solution import`, erfolgreich), committed und gepusht (Commit `9a14703`). **Noch offen:** Alle 6 Flows müssen nach diesem Import erneut manuell reaktiviert werden (wie immer nach `pac solution import`), UND die neue `DMP_COMMAND_Solution.msapp` muss noch einmal in Power Apps Studio geöffnet/importiert werden, damit die Cockpit-Änderungen live gehen.

**Nachtrag – 3 echte Fehler bei der Aktivierung/dem ersten Test gefunden und behoben (2026-08-26):**
1. **Agent 6:** `WorkflowRunActionInputsInvalidProperty` bei `WRITE_RunSummary_To_AuditTrail` – Ursache: `shared_excelonlinebusiness` wurde in 3 Aktionen verwendet, aber nie im `connectionReferences`-Block oben in der Flow-Datei registriert. Behoben durch Ergänzung des fehlenden Eintrags (analog zu den anderen Agenten).
2. **Agent 6:** `InvalidVariableOperation` bei `SET_NewRunsCount` – die Variable `NewRunsCount` wurde verwendet, aber nie per `InitializeVariable` deklariert (Kopierfehler beim Übertragen des Agent-4-Musters). Ergänzt, dabei versehentlich ein doppeltes `runAfter`-Fragment erzeugt und sofort korrigiert.
3. **Power App:** Nach dem Import zeigte das Cockpit „Configuration Load Failed" und aktualisierte keine Werte mehr. Root Cause: Das in der lokalen `.msapr`/`DataSources.json` gecachte Verbindungsschema (`WadlXml`) für `DMPAgent4(StatusCheck)=>VS` kannte das neue Response-Feld `agent6healthy` noch nicht, wodurch der `.Run()`-Aufruf in `App.OnStart` fehlschlug (von `IfError(...)` abgefangen). Behoben durch gezielte Ergänzung des Feldes im gecachten Schema (textbasierter Patch statt langsamem vollständigem JSON-Reserialize, siehe KI-Arbeitsregeln). Nach diesem Fix vom Nutzer live bestätigt: Werte aktualisieren sich korrekt, „AGENTS ACTIVE" zeigt X/6.

**Status: Vollständig abgeschlossen, live getestet und funktionsfähig.**

## ✅ GELÖST (2026-08-26): App-Versionsanzeige (Flackern „1.3.0" → „1.2.0") + GUI-Redesign Kopfzeile
**Root Cause (endgültig gefunden):** Zwei Fallback-Werte existierten parallel, nur einer war bereits auf „leer" korrigiert. Der App-seitige Fallback (`varAppVersion` in `App.OnStart`) war schon leer, ABER Agent 4s eigener Flow-interner Fallback (`VAR_PowerAppVersionText`) enthielt weiterhin den hartkodierten, veralteten Text `"v1.2.0"`. Da `RESPOND_Status` bewusst IMMER nach `SCOPE_PowerAppVersion_Read` läuft (auch bei `Failed`/`Skipped`/`TimedOut`, siehe Scope-Status-Regel), lieferte Agent 4 bei jedem Aufruf diesen alten Wert zurück und überschrieb per `Coalesce(...)` den leeren App-Fallback – das ist exakt das gemeldete Flackern.

**Fix:**
1. `VAR_PowerAppVersionText`s Fallback-Wert in `DMPAgent302StatusCheckVS-....json` von `"v1.2.0"` auf `""` (leer) geändert – Agent 4 auf `[1.2.1]` erhöht (echter Bugfix an bereits live funktionierender Version).
2. **Neue Erkenntnis mit großer Tragweite:** Die KI hat entgegen bisheriger Annahme SEHR WOHL direkten Dateizugriff auf `PowerApp_Version.txt` – die Datei liegt unter `C:\Users\...\OneDrive - Deutsche Börse AG\GO365_DMP Communication - Email Hotline\AI_Agent\PowerApp_Storage\PowerApp_Version.txt`, also im lokal per OneDrive gesyncten Spiegel genau derselben SharePoint-Bibliothek, die Agent 4 über den SharePoint-Connector ausliest. Die KI hat die Datei direkt auf `v1.3.0` aktualisiert (ohne BOM, byte-identisches Format zum Original). **Ab sofort gilt:** Bei jedem künftigen Versions-Update aktualisiert die KI diese Datei SELBST als fester Bestandteil ihrer eigenen Deploy-Routine (kein manueller Schritt des Nutzers mehr nötig) – das war die vom Nutzer geforderte „bessere Lösung" statt reiner Symptombehandlung.
3. Solution-Version neu berechnet (7 Komponenten: 6 Flows + App-Version `1.3.0`) → `7.8.8`.

**Zusätzlich im selben Zug (GUI-Redesign `conTopHeader`, erster Container der Design-/UX-Überarbeitung):**
- Schriftgrößen erhöht (Titel 20→22, Untertitel 10,5→11,5, KPI-Werte 22→24, KPI-Labels/Toggle-Texte 9→10), Container-Höhe 100→84 und alle Y-Positionen einheitlich nach oben verschoben → spürbar weniger Weißraum oben UND unten, wie gewünscht.
- Neue Zeile unterhalb von Titel/Untertitel: **„Zuletzt aktualisiert: HH:MM:SS Uhr"** (`lblStatusLastRefreshed`, neue Variable `varStatusLastRefreshed`, wird nur bei erfolgreichem Agent-4-Aufruf fortgeschrieben, damit sie den Zeitpunkt der zuletzt WIRKLICH gültigen Daten zeigt, nicht jeden Versuch).
- **Neuer Auto-Refresh mit nutzerwählbarem Intervall** (Nutzerwunsch: „Der Anwender soll das über eine Auswahl definieren können"): 5 Auswahl-Buttons „Aus/2m/5m/10m/15m" (`varAutoRefreshMinutes`, Default 5 Minuten), ein unsichtbarer `Timer`-Control (`tmrAutoRefreshTick`, 1-Sekunden-Takt) zählt herunter und löst bei 0 automatisch einen neuen Agent-4-Aufruf aus (dieselbe Coalesce-Logik wie in `App.OnStart`, dupliziert – im Team ohne Power-Fx-UDFs/benannte Formeln bewusst gewählt, siehe „Wichtiger Hinweis" unten), Countdown-Anzeige „Nächstes Update in MM:SS" (`lblAutoRefreshCountdown`). **Bewusst NICHT im Refresh enthalten:** `varOperationalModeIsDMP`/`varEnvironmentIsPROD` (Umschalter-Zustand) – das ist reine App-Start-Initialisierungslogik für die Toggle-`Default`-Bindung und wurde nicht in den periodischen Refresh übernommen, um unbeabsichtigte Seiteneffekte auf die Umschalter auszuschließen.
- **Wichtiger Hinweis zu Kosten:** Jeder automatische Refresh ist ein echter Agent-4-Flow-Lauf (Power-Automate-Lizenz-/Kapazitätsverbrauch, siehe Preis-Gespräch). Bei „5 Min" macht das ca. 288 Läufe/Tag bei durchgehend geöffneter App – dem Nutzer bewusst über die Auswahlmöglichkeit in die Hand gegeben (inkl. „Aus").
- Lokal getestet: `pac canvas pack`/`unpack`-Rundlauf für den neuen `Timer@1.1.1`-Control erfolgreich (Control-Template wird korrekt erkannt und rundtrip-stabil serialisiert).

**Deployment:** Power App neu gepackt (`pac canvas pack`), Solution neu gepackt und importiert (`pac solution import`, erfolgreich). **Nachtrag:** Der erste Import schlug beim Öffnen in Power Apps Studio mit `PA1001: YamlInvalidSyntax` fehl – Ursache: zwei neue einzeilige `Text:=...`-Formeln enthielten „Doppelpunkt + Leerzeichen" (`"Zuletzt aktualisiert: "`, `"Auto-Update: Aus"`) mitten im String-Literal, was YAMLs Plain-Scalar-Parser als (ungültigen) Mapping-Trenner missversteht – `pac canvas pack` prüft das NICHT, erst Studio meldet es beim echten Öffnen. Fix: Doppelpunkt direkt vor das schließende Anführungszeichen gesetzt, Leerzeichen als eigenes `& " " &`-Literal danach angehängt (neue KI-Regel in den Arbeitsregeln verankert). Neu gepackt, committed (`d4c4030`), gepusht.

**Ergebnis (bestätigt durch Nutzer-Screenshot):** Die neue Kopfzeile lädt jetzt strukturell korrekt in Power Apps Studio. Separat sichtbar (zunächst als "erwartet" eingestuft, war aber tatsächlich ein ECHTER, viel größerer Bug – siehe unten): "Configuration load failed" / "Agent 4 status call error" mit Fallback-Nullen.

## ✅ GELÖST (2026-08-26, kritisch): Agent 4 komplett fehlgeschlagen wegen desselben Scope-Status-Bugs wie bei Agent 5
**Root Cause (per Power-Automate-Laufhistorie vom Nutzer bestätigt):** `GET_PowerAppVersion_Content` (innerhalb `SCOPE_PowerAppVersion_Read`) schlug mit „not found" fehl (Pfad/Dateiname-Auflösung der Versions-Datei separat noch zu prüfen, siehe unten). Die anschließende `SET_PowerAppVersionText` lief per `runAfter: [Succeeded]` NUR bei Erfolg des GET – schlug dieser fehl, wurde `SET_PowerAppVersionText` übersprungen (`Skipped`), wodurch der GESAMTE `SCOPE_PowerAppVersion_Read` als „Failed" galt. Das bewirkte, dass der GESAMTE Agent-4-FLOW-LAUF als „Failed" gewertet wurde – obwohl `RESPOND_Status` (dank seines eigenen toleranten `runAfter: [Succeeded, Failed, Skipped, TimedOut]`) trotzdem korrekt lief und ein gültiges JSON zurückgab! Power Apps' `'DMPAgent4(StatusCheck)=>VS'.Run()` wertet aber den GESAMTEN Flow-Lauf-Status, nicht nur die Response-Payload – ein „Failed"-Gesamtlauf löst `IfError(...)` aus und verwirft die (eigentlich gültige!) Antwort komplett zugunsten der Fallback-Nullen. **Das ist exakt dasselbe Muster wie der bereits dokumentierte Agent-5-Bug** („Mailbox evidence folder setup failed"), nur diesmal auf Flow-Ebene statt nur auf Scope-Ebene – und hat vermutlich schon seit Einführung des Versions-Datei-Features die GESAMTE Cockpit-Live-Datenanzeige zeitweise blockiert (nicht nur die Versionsanzeige selbst!).

**Fix:** `SET_PowerAppVersionText`s `runAfter` auf `[Succeeded, Failed, Skipped, TimedOut]` erweitert (läuft jetzt IMMER), Wert per `coalesce(actions('GET_PowerAppVersion_Content')?['outputs']?['body'], variables('PowerAppVersionText'))` gegen einen fehlgeschlagenen/leeren Vorgänger abgesichert (statt des ungeschützten `body('GET_PowerAppVersion_Content')`, das bei einer fehlgeschlagenen Aktion selbst einen Auswertungsfehler auslösen kann). Agent 4 → `[1.2.2]`, Solution-Version → `7.8.9`. Deployed (`pac solution import`, erfolgreich), committed (`5b66688`), gepusht.

**Noch zu klären (nicht mehr blockierend, da der Rest der App jetzt trotzdem lädt):** Warum `GET_PowerAppVersion_Content` mit „not found" fehlschlägt, obwohl der lokale OneDrive-Spiegel die Datei exakt am erwarteten Pfad zeigt (`/Shared Documents/Email Hotline/AI_Agent/PowerApp_Storage/PowerApp_Version.txt`, identisch zum Code-Fallback-Pfad). Nutzer sollte die Werte der beiden Konfigurationszeilen `PowerAppVersionFolderName`/`PowerAppVersionFileName` (Liste `DMP Command Configuration`) auf Tippfehler prüfen – falls diese Zeilen einen ABWEICHENDEN (falschen) Wert enthalten, überschreiben sie per `coalesce(...)` den eigentlich korrekten Code-Fallback.

**⚠️ Neue Merksatz-Ergänzung für künftige Scopes:** Bei JEDEM neuen `Scope` mit mehreren Aktionen IMMER sofort prüfen, ob dessen letzte Aktion(en) ausschließlich `runAfter: [Succeeded]` von einer möglicherweise fehlschlagenden Vorgänger-Aktion haben – falls ja, sofort auf das etablierte 4-Status-Toleranz-Muster umstellen, BEVOR der erste Live-Test läuft (nicht erst reaktiv nach einem Nutzer-Fehlerbericht).

**Noch offen / als Nächstes:** Nutzer muss (1) Agent 4 im Power-Automate-Portal reaktivieren (jeder Solution-Import deaktiviert geänderte Flows automatisch), (2) die aktualisierte `DMP_COMMAND_Solution.msapp` per "Apps → Apps importieren" erneut importieren (dieser Weg hat sich als der zuverlässige erwiesen, NICHT "Datei öffnen" direkt in Studio), (3) danach visuell bestätigen: "✓ Configuration loaded" statt "! Configuration load failed", Version zeigt einen Wert (oder bleibt sauber leer statt Absturz), Countdown zählt sichtbar herunter. Reihenfolge der weiteren Container für die Design-Überarbeitung (nach `conTopHeader`) mit dem Nutzer als Nächstes klären.

---

## ⏸️ Frühere Analyse (jetzt historisch, siehe ✅-Abschnitt oben für die tatsächliche Lösung): App-Versionsanzeige
**Ursprünglicher Zustand (2026-08-25):** `varAppVersion` startet in `App.OnStart` leer (`""`) und wird per `Coalesce(varStatusResult.appversion, varAppVersion)` von Agent 4 überschrieben, der eine SharePoint-Textdatei (`PowerApp_Version.txt`) ausliest. Diese Datei enthielt noch einen alten Stand. Die damalige Annahme „Die KI hat in dieser Umgebung keinen direkten Schreibzugriff auf diese SharePoint-Datei" hat sich am 2026-08-26 als FALSCH herausgestellt (siehe ✅-Abschnitt oben) – die Datei ist über den lokalen OneDrive-Sync direkt erreichbar.

---

## Sofort zu erledigen, BEVOR inhaltlich weitergearbeitet wird
1. ✅ Alle 6 Flows (inkl. neuem Agent 6) erfolgreich per `pac solution import` deployt und vom Nutzer manuell reaktiviert (bestätigt 2026-08-25).
2. ✅ **Agent 5 „Mailbox evidence folder setup failed"-Fehlalarm behoben** (Root Cause: Scope-Gesamtstatus, siehe neuer Abschnitt unten).
3. ✅ **Neuer Agent 6 (Admin Functions) angelegt** – Admin-/Testfunktionen, aktuell: Postfach-Ordnerbaum „PA Processed Mails" komplett löschen.
4. ✅ **`pac canvas pack`-Sperre behoben** (siehe Abschnitt weiter unten) – dateibasierter Weg für Power-App-Änderungen wieder nutzbar.
5. ✅ **GUI-Panel „Admin Functions" in `scrHome` gebaut** (Sidebar-Knopf + Bestätigungsdialog, per Datei-Push/`pac canvas pack`, siehe Abschnitt weiter unten). **Noch offen:** Nutzer muss `DMP_COMMAND_Solution.msapp` noch einmal in Power Apps Studio öffnen/importieren, damit die Änderung live geht (siehe Anleitung im Abschnitt unten), UND danach die neue Datenquellenverbindung zu Agent 6 in Studio prüfen/bestätigen.
6. **Noch offen:** Agent 4 noch nicht live getestet (App neu laden). Agent 2 Performance-Beobachtung noch nicht root-caused. Agent 3 Alert-Mail-Kette noch nicht live getestet (nur Agent 5 bisher).

## ✅ GELÖST (2026-08-25, Nachtrag): Agent 5 „Mailbox evidence folder setup failed" – falscher Alarm trotz korrekt vorhandener Ordner
**Symptom:** Trotz aller vorherigen Fixes (Doppelauslösungs-Guard, Concurrency-Entfernung) sendete Agent 5 bei jedem Lauf weiterhin die kritische Alarm-Mail „Agent 5 - Mailbox evidence folder setup failed", obwohl der Nutzer per Screenshot bestätigte, dass „PA Processed Mails" und „Agent 5 Alerts" im Postfach bereits korrekt existierten (plus eine Karteileiche „PA Processed Mails2" aus einem früheren Fehlversuch).

**Root Cause:** Der Scope `E-Mail_Folder_creation` enthält absichtlich fehlertolerante Schritte (`runAfter: [Succeeded, Failed]`), weil „Ordner existiert bereits" als erwarteter Fehlschlag gilt. ABER: Der Gesamtstatus eines Power-Automate-Scopes richtet sich ausschließlich nach dem Status der LETZTEN Aktion(en) ohne weitere Abhängige darin – hier `SET_AlertTargetFolderId`, deren `runAfter` nur `[Succeeded]` des vorherigen GET-Schritts war. Schlug dieser GET-Schritt fehl, wurde `SET_AlertTargetFolderId` übersprungen (`Skipped`) → der GESAMTE Scope galt als fehlgeschlagen → löste die (fälschliche) Alarm-Mail-Kette aus, obwohl die Ordner in Wirklichkeit korrekt vorhanden waren.

**Fix:**
1. `SET_AlertTargetFolderId` läuft jetzt IMMER (`runAfter: [Succeeded, Failed, Skipped, TimedOut]`), Wert per `coalesce(...)` gegen leere Vorgänger-Ausgabe abgesichert – dadurch bleibt der Scope selbst immer „Succeeded".
2. Der bisherige Alarm-Auslöser (`SET_AuditOutcome_EmailFolderFailed` an den Scope-Status gekoppelt) wurde durch eine neue Bedingung `IF_EmailFolderResolutionFailed` ersetzt, die den ECHTEN fachlichen Erfolg prüft: `empty(coalesce(variables('AlertTargetFolderId'), ''))`. Nur wenn die Ordner-ID wirklich leer blieb, wird `AuditOutcome=Failed` gesetzt und die Alarm-Mail-Kette (`AUDIT_EmailFolderCreation_Failed` → `SET_AlertMailSubject_(EmailFolderCreationFailed)` → `MAIL_Alert_(EmailFolderCreationFailed)`) ausgelöst.
3. `SCOPE_AuditTrail_Write`s `runAfter` entsprechend auf die neue `IF_EmailFolderResolutionFailed`-Aktion umgestellt.

Agent 5 Version auf `[1.1.4]` erhöht (zwei Zwischenstände `[1.1.3]`/`[1.1.4]` wegen einer im selben Zug gefundenen zu langen Beschreibung, siehe Bug-Nachtrag unten), deployt, committet, gepusht (Commits `518139e`, `960ae21`).

**⚠️ Bug im eigenen Prüfskript gefunden und behoben:** Die programmatische Beschreibungslängen-Prüfung (KI-Pflichtregel) hatte einen echten Fehler: Eine rekursive PowerShell-Funktion befüllte eine außerhalb deklarierte Sammelvariable (`$longDescs`) per `+=` ohne `$script:`-Scope-Präfix – dadurch entstand bei jedem rekursiven Aufruf eine neue, lokale Schatten-Variable, deren Ergebnisse nie beim äußeren Aufrufer ankamen. Das Skript meldete fälschlich „keine Fehler", obwohl tatsächlich 2 zu lange Beschreibungen (287 und 322 Zeichen) im Agent-5-Flow vorhanden waren – führte zu einem echten `ActionDescriptionTooLong`-Speicherfehler bei der Aktivierung. Skript korrigiert (`$script:`-Präfix), alle 6 Flow-Dateien danach neu geprüft (sauber), neue KI-Regel verankert (siehe KI-Arbeitsregeln-Dokument).

## ✅ NEU (2026-08-25): Agent 6 (Admin Functions) angelegt – Admin-/Testfunktionen, getrennt von den 5 Produktions-Agenten
Auf Nutzerwunsch („Kannst Du mir in der GUI einen kleinen Knopf bauen, um die Ordner inclusive und unterhalb 'PA Processed Mails' komplett zu löschen?" + „ist es sinnvoll, einen eigenen Admin-Agenten zu bauen? Er soll z.B. auch Verzeichnisse als Archivierung umbenennen") wurde ein 6. Flow als dedizierter Admin-/Testfunktions-Agent angelegt:
- **Workflow:** `DMPAgent6AdminFunctions-B849D817-8AA0-F111-B8DB-000D3A25AEF5.json`, Version `[1.0.0]`.
- **Wichtige technische Erkenntnis:** Neue Cloud-Flows können nicht per Code/CLI erzeugt werden – der Nutzer musste den Flow einmalig leer im Power-Automate-Studio anlegen (nur Trigger + „Respond to a Power App or flow"), UND separat über die Solution „+ Vorhandene hinzufügen" der `DMP_COMMAND_Solution` hinzufügen (ein über „Meine Flows" angelegter Flow ist NICHT automatisch Teil der Solution). Danach per `pac solution export` + `pac solution unpack` gezogen, um die korrekt generierten GUIDs/Registrierungen zu erhalten – deutlich risikoärmer als eine komplett von Hand verfasste Solution-Registrierung.
- **Aktuelle Funktion:** `RequestedAction = "DeleteProcessedMailsFolderTree"` – findet per `$filter=startswith(displayName,'<ProcessedMailsRootFolderName>')` alle Ordner unterhalb „Inbox" (erwischt damit auch Karteileichen wie „PA Processed Mails2"), löscht sie einzeln (Graph `DELETE /mailFolders/{id}`, landet in „Gelöschte Objekte" – recoverable, kein Hard-Delete), meldet Anzahl/Namen zurück. Berücksichtigt automatisch den aktuellen PROD/SIMU-Modus (gleiche Konfigurationsauflösung wie die Produktions-Agenten), damit nie versehentlich im falschen Postfach gelöscht wird.
- **Erweiterbar:** Dispatch per `Switch`-Aktion über `RequestedAction` – künftige Admin-Funktionen (z. B. Ordner für Archivierung umbenennen, wie vom Nutzer angekündigt) können als zusätzliche `case`s ergänzt werden, ohne die Lösch-Logik anzufassen.
- **Noch offen:** GUI-Anbindung (Sidebar-Bereich „Admin Functions" links, Knopf + Bestätigungsdialog vor dem Löschen) ist NOCH NICHT umgesetzt – wegen der bestehenden `pac canvas pack`-Sperre (siehe Abschnitt zu `DMP_COMMAND.msapr`) muss dies als manuelle Studio-Bauanleitung geliefert werden, nicht per Datei-Push.

## ✅ NEU (2026-08-25): Solution-weites Versionsschema eingeführt
Auf Nutzerwunsch bekommt die Solution selbst jetzt eine eigene, aus den Komponentenversionen abgeleitete Versionsnummer (`Solution.xml` → `<Version>`), zusätzlich zur bisherigen individuellen `[x.y.z]`-Versionierung jedes einzelnen Agenten:
- **Format:** `Major.Minor.Patch`, wobei jede Ziffer die SUMME der jeweiligen Versionsziffer aller 6 Agenten-Flows PLUS der Power-App-Version (`varAppVersion` aus `App.pa.yaml`) ist.
- **Aktuelle Baseline (2026-08-25):** `7.4.7` — Details: Major 1+1+1+1+1+1+1=7 (7 Komponenten je Major-Version 1); Minor 0+0+1+0+1+0+2=4 (Agent 3=1, Agent 5=1, Power App=2 aus `v1.2.0`); Patch 1+1+0+1+4+0+0=7 (Agent 1=1, Agent 2=1, Agent 4=1, Agent 5=4 aus `[1.1.4]`).
- **Pflege-Regel:** Bei jeder künftigen Solution-Version-Aktualisierung alle 3 Ziffern anhand der dann AKTUELLEN Versionsstände aller 7 Komponenten neu berechnen (nicht nur hochzählen).


## ✅ GELÖST (2026-08-25): Agent 5 Live-Test – Doppelauslösung, Postfach-Wettlauf, GUI-Rückmeldung
Beim ersten Live-Test des heutigen Agent-3/5-Fixes durch einmaliges Umlegen eines Toggles auf `scrHome` traten 3 zusammenhängende Probleme auf:

1. **Agent 5 wurde zweimal ausgelöst statt einmal.** Root Cause: `tglOperationalState`/`tglApplicationMode` hatten `Default` an eine Variable gebunden (`varOperationalModeIsDMP`/`varEnvironmentIsPROD`), die der eigene `OnCheck`/`OnUncheck`-Handler selbst setzt – der exakt gleiche „Default-Feedback-Loop"-Bug, der schon einmal beim Dark-Mode-Toggle auftrat. **Fix:** Eine bereits vorhandene, aber nie verdrahtete Variable `varSuppressToggleEvents` (Karteileiche aus `App.OnStart`) wurde als Schutz-Flag in alle 4 betroffenen Formeln (`tglOperationalState`/`tglApplicationMode`, je `OnCheck`/`OnUncheck`) eingebaut: Beim echten Klick wird das Flag gesetzt und am Ende zurückgesetzt; eine durch den Feedback-Loop verursachte zweite (unerwünschte) Auslösung erkennt das bereits gesetzte Flag und bricht sofort ab, ohne Agent 5 erneut aufzurufen. Vom Nutzer in Studio eingefügt und veröffentlicht (bestätigt 2026-08-25).
   - **Wichtiger Nachtrag zur Formel-Auslieferung:** Beim ersten Lieferversuch trat ein separater, rein mechanischer Fehler auf („Operator erwartet" direkt bei „If") – verursacht durch ein zusätzliches führendes „=" im gelieferten Formeltext, das zusammen mit dem von Studio bereits angezeigten „=" ein ungültiges „==" ergab. Durch systematisches Testen (bis hin zu einer reinen Literalformel) vom Nutzer selbst gefunden. Neue KI-Regel verankert: Formeln für manuelles Studio-Einfügen werden ab sofort OHNE führendes „=" geliefert.
2. **Postfach-Ordner-Erstellung schlug mit `BadRequest`/`MethodNotAllowed` fehl.** Sehr wahrscheinlich eine Folge von Problem 1: zwei nahezu gleichzeitige Läufe versuchten konkurrierend, dieselben Ordner anzulegen. **Ursprünglicher Fix (zurückgenommen):** Concurrency Control auf Agent 5s Trigger (`runtimeConfiguration.concurrency.runs = 1`) wurde zunächst aktiviert, schlug aber bei der Aktivierung mit `InvalidConcurrencyConfiguration` fehl – Power Automate erlaubt Concurrency Control bei einem `PowerAppV2`-Trigger NICHT, wenn der Flow eine synchrone `Response`-Aktion enthält (Agent 5 muss der App aber synchron antworten, da die Toggle-Formeln direkt auf `varOpModeResult.success` warten). **Endgültiger Fix:** Concurrency-Control-Einstellung wieder entfernt (Version `[1.1.2]`); die Ursache der Parallel-Läufe ist ohnehin durch den `varSuppressToggleEvents`-Guard (Punkt 1) an der Wurzel behoben, sodass eine zusätzliche Trigger-seitige Absicherung nicht mehr nötig ist.
3. **GUI-Statusanzeige aktualisierte sich nicht, obwohl Power Automate den Lauf als erfolgreich meldete.** Root Cause: Die App-Antwort nutzte `success = (AuditOutcome == "Succeeded")`. Da der heutige kritische Postfach-Ordner-Fix bei einem Ordnerfehler bewusst `AuditOutcome = "Failed"` setzt (um die Alert-Mail auszulösen), meldete Agent 5 der App fälschlich einen Gesamt-Fehlschlag, obwohl die eigentliche Umschaltung des Operating State erfolgreich war. **Fix (nach Nutzerentscheidung):** Neue Variable `CoreActionOutcome` verfolgt nur den Erfolg der eigentlichen Zustandsänderung (`UPDATE_ConfigRow_CurrentOperationMode`), unabhängig vom späteren Postfach-Ordner-Status. Die App-Antwort ist jetzt `true`, sobald ENTWEDER die Zustandsänderung ODER der Gesamtlauf erfolgreich war – ein Ordner-Fehler markiert weiterhin `AuditOutcome=Failed` (Alert-Mail bleibt bestehen), verfälscht aber nicht mehr die Toggle-Rückmeldung.

Agent 5 Version wegen dieser Korrekturen auf `[1.1.1]` erhöht, deployt, committet und gepusht (Commit `93eed50`).

## ⚠️ Performance-Beobachtung (2026-08-25, noch nicht root-caused): Agent 2 brauchte ~4:51 Min. für eine einzelne E-Mail
Beim ersten Live-Test nach dem heutigen Deployment beobachtete der Nutzer, dass Agent 2 (E-Mail Inbox Treatment) fast 5 Minuten für die vollständige Verarbeitung einer einzelnen eingehenden E-Mail benötigte. Das erscheint deutlich zu langsam für den normalen Betrieb. Mögliche Ursachenkandidaten (noch nicht verifiziert, nur erste Hypothesen):
- Die mehreren `WaitSecondsBeforeSentMailSearch`-Verzögerungen vor jeder Sent-Items-Suche (mehrfach pro Zweig vorhanden, z. B. vor Reply-Bestätigung, vor Counter-Update-Fehlerbehandlung, vor Forward-Fehlerbehandlung – könnten sich aufsummieren).
- Die generelle Anzahl sequenzieller Office365/SharePoint/Excel-Connector-Aufrufe pro Zweig.
- Die historisch bekannte ineffiziente Config-Ladeschleife (siehe „Item 2: Config-Ladeschleife" weiter unten im Dokument, dort für Agent 1/2 als größter Hebel dokumentiert, aber laut Backlog bereits 2026-08-07 als abgeschlossen markiert – ggf. gilt das nicht für Agent 2 oder ist zwischenzeitlich regressiert).

**Nächster Schritt:** Vor jedem Optimierungsversuch zuerst den tatsächlichen Power-Automate-Laufverlauf mit der Aktionslaufzeiten-Aufschlüsselung ansehen (welche einzelne Aktion/welcher Zweig die meiste Zeit verbraucht), statt zu raten.

## ✅ GELÖST (2026-08-25): `pac canvas pack`-Sperre behoben – dateibasierter Weg für Power-App-Änderungen wieder nutzbar
**Ausgangslage:** Seit 2026-08-24 durfte `pac canvas pack` für den lokalen Solution-Ordner nicht mehr verwendet werden, weil die darin eingebettete `DataSources.json` (in `DMP_COMMAND.msapr`) noch die ALTEN, vor der Agenten-Umnummerierung gültigen Connector-Bindungen enthielt – ein daraus gepacktes `.msapp` hätte korrekten Formel-Text mit falschen Bindungsdaten kombiniert.

**Fix (Idee A aus dem vorherigen Abschnitt umgesetzt, mit Sicherheitsprüfungen):**
1. App-Identität VOR jedem Download zweifelsfrei bestätigt: `pac canvas list` zeigte genau EINE App namens „DMP COMMAND" im Environment (kein Duplikat/Verwechslungsrisiko wie beim früheren Beinahe-Vorfall).
2. `pac canvas download --name "DMP COMMAND" --extract-to-directory <Scratch-Ordner>` in einen ISOLIERTEN Scratch-Ordner (nicht direkt ins Projekt) ausgeführt.
3. Ergebnis auf bekannte Marker geprüft (`varSuppressToggleEvents`, `dotLedOperationalState`, `conKpiCritical`, `varAppVersion`) – alle vorhanden. `.pa.yaml`-Inhalte gegen die lokale Referenzkopie verglichen: nur 2 rein kosmetische Abweichungen (Encoding-Artefakt `Â·` statt `·`, ein harmloses `OnVisible: =false` von Studio automatisch ergänzt) – bestätigt, dass die lokale Referenzkopie bereits korrekt war.
4. `References/DataSources.json` im frischen Download enthielt die KORREKTEN aktuellen Bindungen (`DMPAgent3.01(EmergencyReportManagement)=>VS`, `DMPAgent3.03(OperationalStateManagement)=>VS`, `DMPAgent4(StatusCheck)=>VS`) sowie bereits automatisch `DMPAgent6(AdminFunctions)` (Studio listet neu erstellte, mit der Umgebung verbundene Flows automatisch als potenzielle Datenquelle, auch ohne aktive Nutzung in der App).
5. **Wichtige Struktur-Erkenntnis:** `DMP_COMMAND.msapr` ist technisch nur ein ZIP-Archiv aus `msapr-header.json` + einem `msapp/`-Unterordner mit GENAU der Struktur, die `pac canvas download --extract-to-directory` erzeugt (`Assets`, `Components`, `Controls`, `References`, `Resources`, + Root-JSONs). Der `Src/`-Ordner mit den `.pa.yaml`-Dateien liegt als GESCHWISTER-Ordner NEBEN der `.msapr`, nicht darin. `pac canvas pack` kombiniert: Formel-/Steuerelement-Text aus `Src/*.pa.yaml` (aktuell, bereits korrekt) + alle übrigen Metadaten (inkl. `References/DataSources.json`) aus dem `.msapr`-Container.
6. Den veralteten `msapp`-Unterordner innerhalb der lokal entpackten `.msapr` komplett durch den frisch heruntergeladenen ersetzt, neu als `.msapr` gezippt, testweise mit den (bereits aktuellen) lokalen `Src/*.pa.yaml`-Dateien gepackt (`pac canvas pack`) → Erfolg. Ergebnis wieder entpackt und verifiziert: `DataSources.json` enthält jetzt die korrekten Bindungen, UND der Formel-Text in `scrHome.pa.yaml`/`App.pa.yaml` ist beim Pack/Unpack-Zyklus byte-identisch geblieben (0 Diff-Zeilen) – kein Datenverlust.
7. Reparierte `.msapr` in `C:\PowerAppWork\DMP_COMMAND_Solution\PowerApp\DMP_COMMAND\Source\DMP_COMMAND.msapr` übernommen, finaler Test-Pack direkt aus dem echten Projektordner erfolgreich.

**Ergebnis:** Der dateibasierte Weg (`.pa.yaml` bearbeiten + `pac canvas pack`) ist ab sofort WIEDER der bevorzugte Weg für Power-App-Änderungen (siehe KI-Arbeitsregeln, aktualisiert). Der manuelle Studio-Weg bleibt als Fallback dokumentiert, falls dieses Problem in anderer Form wiederkehrt.

**Hinweis für künftige Bearbeiter:** Die harmlose Warnmeldung „Canvas apps packed using yaml SourceCode must be validated first by opening the app for edit within the Power Apps studio" erscheint bei JEDEM `pac canvas pack`-Aufruf mit `SourceCode`-Layout (auch bei erfolgreichen Packvorgängen) – das ist normal und kein Fehler, solange danach „Packing succeeded." erscheint. `pac canvas validate` existiert in der installierten CLI-Version (2.11.2) nicht mehr („no longer supported") – falls die Warnung ernst genommen werden soll, ist der einzige Weg weiterhin, die gepackte App einmal in Studio zu öffnen und zu speichern.

---

## ✅ NEU (2026-08-25): GUI-Panel „Admin Functions" in `scrHome` gebaut (per Datei-Push, dank behobener `pac canvas pack`-Sperre)
Auf Nutzerwunsch wurde ein neuer, dauerhaft sichtbarer Bereich in der linken Sidebar ergänzt:
- **Neuer Sidebar-Knopf `btnNavAdminFunctions`** („Admin Functions (Test Only)"), nach `btnNavMaintenance` eingefügt. Klick schaltet `varShowAdminPanel` um (grün hervorgehoben, wenn aktiv – gleiches Muster wie der aktive „Cockpit"-Knopf).
- **Neuer Panel-Container `conAdminFunctions`** (rot umrandet, `Visible: =varShowAdminPanel`), als ERSTES Kind von `conMain` eingefügt (erscheint oben, oberhalb von `conTopHeader`), enthält:
  - Titel + Warnhinweis-Text.
  - Knopf „Delete..." (`btnAdminDeleteProcessedMailsFolder`) → setzt `varConfirmDeleteMailboxFolders` auf `true`.
  - Bestätigungs-Unterbereich `conAdminConfirmDelete` (`Visible: =varConfirmDeleteMailboxFolders`) mit Warntext + zwei Knöpfen:
    - „Yes, delete" (`btnConfirmDeleteYes`): ruft `'DMPAgent6(AdminFunctions)'.Run(User().Email, "DeleteProcessedMailsFolderTree")` auf, zeigt das Ergebnis per `Notify(...)` (grün bei Erfolg, rot bei Fehler), schließt den Bestätigungsdialog.
    - „Cancel" (`btnConfirmDeleteCancel`): schließt den Bestätigungsdialog ohne Aktion.
  - Ergebnis-Label `lblAdminActionResult`, zeigt die letzte Rückmeldung von Agent 6 dauerhaft an (`Coalesce(varAdminActionResult.message, "")`).
- **App-Version** in `App.pa.yaml` von `v1.2.0` auf `v1.3.0` angehoben (echte funktionale Änderung).
- Alles per Datei-Bearbeitung + `pac canvas pack` gebaut und lokal test-gepackt/entpackt (Struktur- und Round-Trip-Verifikation erfolgreich), NICHT per manueller Studio-Eingabe.

**Noch zu erledigen (Nutzer-Aktion erforderlich):**
1. Die neu gepackte Datei `C:\PowerAppWork\DMP_COMMAND_Solution\PowerApp\DMP_COMMAND\DMP_COMMAND_Solution.msapp` in Power Apps Studio öffnen/importieren (wie bei früheren `pac canvas pack`-Runden vor dem 2026-08-24-Vorfall gehandhabt) und veröffentlichen.
2. Nach dem Import prüfen, ob die App die neue Datenquellenverbindung zu Agent 6 (`DMPAgent6(AdminFunctions)`) automatisch korrekt bindet (siehe wiederkehrendes Connector-Referenz-Risiko weiter oben) – falls rote Fehlersymbole am „Yes, delete"-Knopf erscheinen, den exakten autovervollständigten Verbindungsnamen zurückmelden.
3. Danach den kompletten Ablauf einmal live testen: Admin Functions öffnen → Delete... → Bestätigungsdialog → Yes, delete → Ergebnis-Meldung prüfen.

---

## ✅ GELÖST (2026-08-25, Nachtrag): Echter Doppelauslösungs-Bug beim App-Laden gefunden und behoben (der `varSuppressToggleEvents`-Guard vom Vormittag war unvollständig)
**Symptom (vom Nutzer nach Import des neuen `.msapp` gemeldet):** Allein durch das Laden/Starten der App (ohne einen Toggle anzufassen) liefen sofort 2 Agent-5-Läufe an, beide mit „Mailbox Evidence Folder Setup Failed"-Alarm-Mail.

**Root Cause:** `App.OnStart` setzte `varSuppressToggleEvents` ganz am Anfang auf `false` (Zeile 17), ermittelte aber ERST VIEL SPÄTER (nach dem Agent-4-Status-Abruf) die echten Werte für `varOperationalModeIsDMP`/`varEnvironmentIsPROD` – genau die beiden Variablen, an die `Default` der beiden Toggles gebunden ist. Das Setzen dieser Variablen ändert `Default`, was `OnCheck`/`OnUncheck` auslöst – aber die Guard-Variable war zu diesem Zeitpunkt bereits `false`, schützte also nicht. Zusätzlich: Die alte Guard-Logik setzte im „unterdrückt"-Zweig `varSuppressToggleEvents` sofort wieder auf `false` zurück – dadurch war der Schutz beim ERSTEN der beiden automatischen Default-Wechsel (Operating State) schon wieder aufgehoben, bevor der ZWEITE (Environment) passierte, was die zweite E-Mail erklärt.

**Fix:**
1. `App.OnStart`: `varSuppressToggleEvents` wird jetzt ganz am Anfang auf `true` gesetzt (statt `false`) und erst GANZ AM ENDE von `OnStart` (nach beiden `Default`-relevanten Set-Aufrufen) wieder auf `false` zurückgesetzt.
2. In allen 4 Toggle-Handlern (`tglOperationalState`/`tglApplicationMode` × `OnCheck`/`OnUncheck`) wurde der „unterdrückt"-Zweig von `Set(varSuppressToggleEvents, false)` auf ein reines No-Op (`false`) geändert – die Guard-Variable wird jetzt NUR NOCH an den beiden bewusst gesetzten Stellen (Anfang/Ende der echten Aktion, Anfang/Ende von `App.OnStart`) verändert, nicht mehr nebenbei durch einen unterdrückten Fake-Trigger.
3. Per Round-Trip-Test (`pac canvas pack`/`unpack`) verifiziert: alle 4 Formeln korrekt übernommen.

**Weitere kleinere Korrekturen im selben Zug:**
- **Verwaiste doppelte Agent-3-Datenquelle entfernt:** `References/DataSources.json` enthielt zusätzlich zum aktuell genutzten `DMPAgent3.01(EmergencyReportManagement)=>VS` einen ungenutzten Karteileichen-Eintrag `DMPAgent3(EmergencyReportManagement)` (Relikt aus der Zeit vor der Agenten-Umnummerierung, tauchte im Studio-Datenpanel verwirrend als „2 Versionen von Agent 3" auf). Eintrag aus der `.msapr` entfernt, Rest unverändert.
- **App-Versionsanzeige geflackert (kurz „1.3.0", dann wieder „1.2.0"):** Ursache ist NICHT der App-Code, sondern eine SharePoint-Textdatei (`PowerApp_Version.txt`, von Agent 4 gelesen und in `varStatusResult.appversion` zurückgegeben), die noch den alten Stand „v1.2.0" enthält und den frisch gesetzten Wert per `Coalesce(...)` überschreibt. Auf Nutzerwunsch („Fallback ist dann lieber leer") wurde der anfängliche Fallback-Wert in `App.OnStart` von einem hartkodierten Literal auf `""` (leer) geändert, UND die Anzeige in `scrHome.pa.yaml` (`lblVersionTag`) zeigt jetzt `varAppVersion` direkt ohne zusätzlichen Text-Fallback – dadurch erscheint bei fehlendem/veraltetem Wert lieber gar nichts als ein falscher Stand. **Noch offen:** Die eigentliche Quelle der Wahrheit (`PowerApp_Version.txt` in SharePoint) müsste ebenfalls auf „v1.3.0" aktualisiert werden, damit die Anzeige nach dem Agent-4-Refresh den korrekten Stand zeigt – das kann die KI nicht direkt (kein SharePoint-Dateizugriff in dieser Umgebung), muss vom Nutzer oder in einer künftigen Sitzung nachgezogen werden.

**Noch offen (vom Nutzer gemeldet, noch nicht geklärt):** Ein Fehlersymbol rechts oberhalb des Logos in der Sidebar (Screenshot erhalten) – genaue Ursache noch nicht identifiziert, wird in der nächsten Nachricht geklärt.

---

## ✅ GELÖST (2026-08-25, Nachtrag 2): "Coalesce weist ungültige Argumente auf"-Fehler nach Import
**Symptom:** Nach Import des `.msapp` mit dem neuen Admin-Functions-Panel zeigte Studio einen Fehler „Die Funktion 'Coalesce' weist ungültige Argumente auf".

**Root Cause:** `varAdminActionResult` (das Ergebnis-Objekt von Agent 6) wurde NIRGENDS in `App.OnStart` initialisiert, sondern nur einmalig per `Set()` innerhalb des „Yes, delete"-Knopf-Handlers gesetzt. Da `Coalesce(varAdminActionResult.message, "")` und `!IsBlank(varAdminActionResult)` aber bereits beim Laden der Seite (für das Ergebnis-Label) ausgewertet werden, konnte Power Fx den Record-Typ nicht sauber ableiten – exakt das bereits dokumentierte Muster „unbenutzte/ungetypte globale Variable verursacht Typfehler in der gesamten umgebenden Formel".

**Fix:** `varAdminActionResult` in `App.OnStart` jetzt explizit mit einem konkret typisierten Record initialisiert (`{success:false, message:"", requestedaction:"", deletedcount:0}`, passend zum Response-Schema von Agent 6), ebenso `varShowAdminPanel`/`varConfirmDeleteMailboxFolders` (beide `false`). Die Sichtbarkeits-/Text-Formeln des Ergebnis-Labels wurden entsprechend angepasst (`Visible: =Len(varAdminActionResult.message)>0` statt `!IsBlank(...)`, da das Feld jetzt nie mehr „blank" sondern höchstens ein leerer String ist).

---

## ✅ GELÖST (2026-08-25, Nachtrag 3): "Ungültige Anzahl von Argumenten. Empfangen 2, erwartet 0"-Fehler bei Agent 6
**Symptom:** Nach Behebung des Coalesce-Fehlers zeigte Studio einen neuen Fehler am `'DMPAgent6(AdminFunctions)'.Run(...)`-Aufruf: „Ungültige Anzahl von Argumenten. Empfangen 2, erwartet 0."

**Root Cause:** Die Datenquellen-Metadaten (`References/DataSources.json` in der `.msapr`) für eine Power-Automate-Verbindung enthalten ein gecachtes WADL-Schema (`WadlXml`), das die erwartete Parameteranzahl beschreibt. Dieses Schema wird nur dann aktualisiert, wenn die Verbindung tatsächlich in einer Studio-Formel referenziert/aufgerufen wird (was einen Neuabruf des echten Trigger-Schemas auslöst). Da Agent 6 zum Zeitpunkt des letzten `pac canvas download`-Laufs (für den msapr-Fix) zwar als Flow existierte, aber noch NIE in einer App-Formel referenziert worden war, blieb sein gecachtes Schema beim ursprünglichen leeren Zustand (0 Trigger-Parameter) hängen, obwohl der echte Flow inzwischen 2 Parameter (`text`/`text_1`) erwartet.

**Fix:** Das `WadlXml` für `DMPAgent6(AdminFunctions)` in `DataSources.json` manuell nach dem exakten Muster der funktionierenden Agent-5-Verbindung neu erstellt: `ManualTriggerInput` mit `text`(InitiatedBy)/`text_1`(RequestedAction), `ResponseActionOutput` mit `success`(boolean)/`message`(string)/`requestedaction`(string)/`deletedcount`(number) – passend zu Agent 6s tatsächlichem Trigger-/Response-Schema. Neu gepackt, verifiziert, übernommen.

**Für künftige neue Flow-Referenzen (Lehre für später):** Wird ein NEUER Power-Automate-Flow zum ersten Mal in einer App-Formel referenziert (egal ob per Datei-Push oder manuell in Studio), sollte VOR dem finalen `pac canvas pack`/Import geprüft werden, ob das gecachte WADL-Schema in `DataSources.json` bereits die korrekte, aktuelle Parameteranzahl/-schema des Flows widerspiegelt – sonst droht dieser exakte „erwartet 0"-Fehler erneut.

**✅ Gelöst / durch Praxis überholt (2026-08-28):** Der Nutzer hält `PowerApp_Version.txt` (SharePoint-Quelle für `varAppVersion`, gelesen über Agent 4) inzwischen zuverlässig manuell synchron zum jeweils zuletzt gelieferten Versionsstand (bestätigt durch Screenshot mit korrekt angezeigtem `v1.6.4`). **Formal entschiedene Lösung für die offene Frage "bessere Lösung für die Versionsanzeige":** Variante (a) beibehalten – manuelle Aktualisierung durch den Nutzer nach jedem `.msapp`-Publish, jeweils auf den von der KI in `PowerApp_Version.txt` hinterlegten Stand. Kein automatisierter Schreibzugriff der KI auf SharePoint verfügbar, daher keine der Alternativen (b)/(c) nötig. Feste Routine ab sofort: Nach jedem PowerApp-Release nennt die KI explizit den neuen Versionsstring UND weist auf die nötige manuelle SharePoint-Aktualisierung hin (wie in dieser Sitzung bei `v1.6.5`/`v1.7.0` bereits praktiziert).

---


- **Agent 4 (Status Check):** wird nur beim Start/Neuladen der Power App über `App.OnStart` aufgerufen (`'DMPAgent4(StatusCheck)=>VS'.Run()`). Zum Testen: die DMP COMMAND Power App einmal komplett schließen und neu öffnen (bzw. in Studio über „Wiedergeben" neu starten), dann im Power-Automate-Laufverlauf von Agent 4 prüfen, ob ein neuer Lauf erscheint.
- **Agent 5 (Operational State Management):** wird nur durch die beiden Schalter auf `scrHome` ausgelöst (Operating-State-Schalter und Environment-Schalter, jeweils `OnCheck`/`OnUncheck`, `'DMPAgent3.03(OperationalStateManagement)=>VS'.Run(...)`). Zum Testen: in der laufenden App einen der beiden Schalter einmal umlegen (z. B. Environment-Schalter kurz auf „Simulation" und zurück auf „Produktiv"), dann im Power-Automate-Laufverlauf von Agent 5 prüfen, ob ein neuer Lauf erscheint.
- Für einen **gezielten Test der neuen Alert-Mail-Kette** (Postfach-Ordner-Erstellungsfehler) müsste die Ordnererstellung selbst zum Scheitern gebracht werden (z. B. testweise fehlende Berechtigung auf dem übergeordneten Postfach-Ordner) – das ist aufwendiger zu simulieren als ein normaler Lauf. Ein normaler erfolgreicher Lauf bestätigt zumindest, dass die heutige Umstrukturierung den Standard-Erfolgspfad nicht gebrochen hat.

## 🐛 Nachtrag (2026-08-25): Deployment-Fehler nach erstem Import gefunden und behoben
Der erste `pac solution import` mit den heutigen Änderungen führte NICHT zu einer sauberen Aktivierung aller 5 Flows – Power Automate lehnte mehrere Flows beim Speichern/Aktivieren ab. Gefundene und behobene Ursachen:
1. **`ActionDescriptionTooLong` (mehrfach, in Agent 1/2/3/5):** Mehrere der heute neu ergänzten Kachel-Beschreibungen überschritten das harte 255/256-Zeichen-Limit (bis zu 291 Zeichen), obwohl diese Regel in den KI-Arbeitsregeln bereits seit 2026-08-07 dokumentiert ist. Zusätzlich wurden dabei mehrere VORBESTEHENDE, bereits bei genau 256 Zeichen mitten im Wort abgeschnittene Beschreibungen entdeckt (Datenqualitätsproblem aus einer früheren Sitzung, nicht heute verursacht). Alle betroffenen Beschreibungen gekürzt/vervollständigt; KI-Arbeitsregel verstärkt, dass künftig nach JEDER Beschreibungsänderung sofort eine programmatische Längenprüfung (≤ 255 Zeichen) als fester Bestandteil der Validierungsroutine erfolgen muss (nicht nur JSON-Syntax + `runAfter`-Referenzgraph).
2. **`InvalidTemplate` (Agent 3, echter Bug):** Die neue Aktion `AUDIT_EmailFolderCreation_Failed` referenzierte `outputs('CMP_SourceFileName')`, aber diese Compose-Aktion liegt innerhalb von `SCOPE_Validation`, das bewusst ERST NACH der neuen Alert-Mail-Kette läuft (um Race Conditions bei `AuditOutcome`/`AuditEvents` zu vermeiden) – Power Automate lehnt statische Referenzen auf Aktionen ab, die nicht nachweislich im `runAfter`-Pfad liegen. Fix: `SourceFileName` im Audit-Event liest jetzt direkt `triggerBody()?['file']?['name']` statt der abgeleiteten Compose-Aktion (Trigger-Werte sind immer verfügbar, unabhängig von der Ausführungsreihenfolge).

Nach Behebung aller Punkte wurde eine finale, kompromisslose Prüfung über alle 5 Dateien durchgeführt (JSON gültig + alle Beschreibungen ≤ 255 Zeichen + keine defekten `runAfter`-Referenzen), erneut importiert, und die Aktivierung war beim zweiten Anlauf für alle 5 Flows erfolgreich.

## Was heute (2026-08-25) inhaltlich erledigt wurde
- **`App.OnStart`-Bug behoben** (siehe Abschnitt „✅ GELÖST (2026-08-25): `App.OnStart` verhinderte kompletten Neustart der App..." weiter unten): 3 mit `Blank()` initialisierte, nirgends verwendete Variablen verhinderten unbemerkt die komplette `OnStart`-Ausführung inkl. Agent-4-Aufruf. Fix: `Blank()` → `""`. Live bestätigt funktionierend, nach GitHub gepusht (Commit `56048e2`).
- **Deutsches-Gebietsschema-Regel für Power-Fx korrigiert:** Die bisherige Annahme „nur Kommas vor nackten Zahlen werden zu Semikolon" war FALSCH und hatte zu einem echten Produktionsvorfall geführt. Korrekte Regel: JEDES Funktions-Argument-Komma wird zu `;` (auch vor Text/Boolean/Variablen/Funktionsaufrufen), nur die Anweisungsverkettung wird zu `;;`.
- **Agent 3 und Agent 5 – kritischer Postfach-Ordner-Fehler behoben** (siehe eigener Abschnitt weiter unten): Ordnererstellungsfehler lösten bisher weder Warnung noch Alert-Mail aus; bei Agent 3 wurde dadurch sogar die komplette Kernverarbeitung stillschweigend übersprungen. Neues, dauerhaftes Muster etabliert: kritische Fehler = immer „Failed" + immer Alert-Mail (nicht nur Cockpit-Zähler).
- **Vollständige Kachel-Beschreibungs-Bereinigung über alle 5 Agenten** (siehe eigener Abschnitt weiter unten): 22 (Agent 1) + 137 (Agent 2) + 13 (Agent 3) + 38 (Agent 4) + 1 (Agent 5) = 211 fehlende Beschreibungen ergänzt, jeweils JSON- und `runAfter`-Referenzgraph-validiert.
- **Alle 5 Flows erfolgreich per `pac solution import` in die Live-Umgebung deployt** (zwei Durchgänge: erster Import ohne Versionsnummer-Anpassung, zweiter Import nach Korrektur). Flow-Versionsnummern gemäß der Namenskonvention hochgezählt: Agent 1/2/4 (nur Beschreibungen bzw. defensive Fixes, keine Verhaltensänderung) → `[1.0.1]`; Agent 3/5 (echte Bugfixes mit Verhaltensänderung) → `[1.1.0]`.

## Was der Nutzer als NÄCHSTEN inhaltlichen Schwerpunkt vorgegeben hat
> „Ich würde eher alle Fehler bereinigen bevor wir uns der GUI widmen."

Dieser Fokus (Fehlerbereinigung vor GUI-Arbeit) ist mit dem heutigen Abschluss der Struktur- und Beschreibungsfixes sowie dem erfolgreichen Deployment erreicht. Nach Reaktivierung der Flows und einem kurzen Live-Test ist der Weg frei für den bereits zuvor angekündigten nächsten Schwerpunkt:
> „Ich bin mit dem Design der App noch nicht zufrieden: Wir müssen Container für Container überarbeiten. Bisher hatten wir nur die Zeile mit der Überschrift fertiggestellt. Die anderen Anzeigen müssen wir schritt für schritt durchgehen und gemeinsam auf einen besseren Stand bringen, damit das Nutzererlebnis maximal ist."

**Wichtige Klarstellung:** Das ist NICHT dasselbe wie die bereits abgeschlossene „GUI-Kachel-Review" von heute (die hat nur Datenanbindung/Bugs pro Container geprüft, nicht das visuelle Design/UX). Der Nutzer möchte jetzt eine echte **Design-/UX-Überarbeitung** container-für-container, beginnend nach `conTopHeader` (das laut Nutzer als einzige Zeile bereits „fertiggestellt" ist). Reihenfolge und genauer Umfang mit dem Nutzer morgen zuerst klären, bevor Änderungen vorgeschlagen werden.

---

# 🔴 KRITISCH: Power-Fx-Formeln, die der Nutzer manuell in Power Apps Studio eintippt/einfügt, müssen die DEUTSCHE Gebietsschema-Trennzeichen verwenden (bestätigt 2026-08-24, Regel korrigiert 2026-08-25)

**Symptom:** Nach dem Einfügen einer von der KI gelieferten Power-Fx-Formel (mit Standard-US/invarianten Trennzeichen: `,` für Funktionsargumente, `;` für Anweisungsverkettung) direkt in Power Apps Studios Formelleiste zeigten mehrere Steuerelemente rote Fehlersymbole. Bei genauerer Prüfung (Herunterladen des Live-Standes via `pac canvas download` und Vergleich mit der ursprünglich gelieferten Formel) zeigte sich: Studio hatte die Formel falsch interpretiert und beim Speichern strukturell verändert.

**Root Cause (bestätigt durch Vergleich Original-Formel vs. heruntergeladener Live-Stand):** Das Studio des Nutzers läuft mit einem **deutschen Gebietsschema (locale)**. In diesem Gebietsschema gilt eine andere Zuordnung der Formel-Trennzeichen als im US-Standard:
- **Komma `,`** wird as **Dezimaltrennzeichen** interpretiert (nicht als Funktionsargument-Trennzeichen!). Ein Komma direkt vor einer nackten Zahl (z. B. `Set(varX, 0)`) wird zu `Set(varX.0)` verstümmelt (ungültiger Ausdruck).
- **Einfaches Semikolon `;`** wird als **normales Funktionsargument-/Listentrennzeichen** interpretiert (entspricht der Rolle von `,` im US-Standard) – NICHT als Verkettungsoperator für mehrere Anweisungen in einer Formel.
- **Doppeltes Semikolon `;;`** ist der korrekte **Verkettungsoperator** für mehrere Anweisungen in einer Formel (entspricht der Rolle von einfachem `;` im US-Standard).

**⚠️ Korrektur (2026-08-25):** Die ursprüngliche Annahme vom 2026-08-24 – dass Kommas vor Text/Boolean/Variablen/Funktionsaufrufen unkritisch seien und `,` bleiben dürften – erwies sich in der Praxis als FALSCH. Bei der Einführung des App-Versionsnummer-Mechanismus führte genau diese Annahme zu einer großflächigen Fehlerkaskade (rote Fehlersymbole auf praktisch jedem Steuerelement der App, da die durch `OnStart` gesetzten Variablen alle als „fehlerhaft/unbekannt" galten). Der Nutzer hat das Problem selbst gefunden und behoben, indem er AUSNAHMSLOS JEDES Funktionsargument-Komma durch `;` ersetzt hat – auch vor Text (`"Text"`), Boolean (`true`/`false`), Variablen und verschachtelten Funktionsaufrufen. Erst danach verschwanden alle Fehlersymbole bis auf ein unabhängiges Laufzeitproblem (siehe Abschnitt weiter unten zu „Agent 4 liefert nach appversion-Erweiterung keine Daten mehr").

**Korrigierte, vollständig gültige Regel für alle künftigen Power-Fx-Formeln, die der Nutzer manuell in Studio eingibt (nicht per `pac`/Datei-Bearbeitung):**
1. **AUSNAHMSLOS jedes Komma, das als Funktionsargument-Trenner dient, MUSS durch `;` ersetzt werden** – unabhängig vom Datentyp des Arguments (Zahl, Text, Boolean, Variable, Funktionsaufruf, Array-/Tabellen-Element). Es gibt KEINE Ausnahme mehr für Text/Boolean/Variablen/Funktionsaufrufe.
2. Jedes einfache Verkettungs-Semikolon `;` (mehrere Anweisungen nacheinander in einer Formel, auch verschachtelt z. B. im Fehlerbehandlungs-Zweig von `IfError(...)`) MUSS durch `;;` ersetzt werden.
3. **Zusätzliche Lehre:** Tabellen-/Array-Literale mit mehreren kommagetrennten Elementen (z. B. `CountIf([a, b, c], Value = true)`) sind besonders fehleranfällig beim manuellen Übertragen, da hier mehrere Kommas gleichzeitig konvertiert werden müssen. Wo möglich stattdessen eine robustere, kommaärmere Formulierung verwenden – der Nutzer hat dies erfolgreich durch eine Summe aus `If(Bedingung; 1; 0) + If(...) + ...` statt `CountIf` mit Array-Literal ersetzt; dieses Muster als Vorlage für künftige, ähnliche Aggregationen übernehmen.
4. **Verifikationsmethode, um diese Art Fehler zuverlässig zu erkennen:** `pac canvas download --name "DMP COMMAND" --extract-to-directory <temp-Ordner>` ausführen und die betroffene Formel in der resultierenden `.pa.yaml` mit der ursprünglich gelieferten Formel vergleichen (Zeichen für Zeichen, nicht nur grob). Rote Fehlersymbole in Studio können außerdem VERALTETEN Browser-Cache-Status zeigen – ein Neuladen der Seite (F5) VOR jeder tieferen Fehlersuche ist Pflicht. **Zuverlässigste Methode (2026-08-25):** Den Nutzer bitten, den kompletten aktuellen Formeltext per Strg+A/Strg+C direkt aus der Studio-Formelleiste zu kopieren und als Text (nicht Screenshot) zu übermitteln – Screenshots zeigen oft nur einen Ausschnitt und verbergen die tatsächliche Fehlerursache.
5. Dateibasierte Änderungen (direktes Bearbeiten der lokalen `.pa.yaml`-Dateien + `pac canvas pack`) sind von diesem Problem NICHT betroffen, da sie das invariante Format direkt schreiben und nie durch Studios Gebietsschema-sensitive Formelleisten-Auswertung laufen. Bevorzugt werden sollte daher grundsätzlich der dateibasierte Weg – ist aber für den DMP-COMMAND-Solution-Ordner aktuell GESPERRT (siehe Abschnitt zu `DMP_COMMAND.msapr`/eingebetteter `DataSources.json` weiter unten) – bis auf Weiteres gilt daher ausschließlich der manuelle Studio-Weg für Power-App-Änderungen.

---

# ✅ GELÖST (2026-08-25): `App.OnStart` verhinderte kompletten Neustart der App – Ursache war eine mit `Blank()` initialisierte, nirgends verwendete Variable

**Symptom:** Nach der Umstellung aller Kommas auf Semikolons (siehe Abschnitt oben) verschwanden zwar alle sichtbaren roten Fehlersymbole in der App, aber Agent 4 (Status Check) wurde beim App-Start nicht mehr aufgerufen (kein neuer Lauf im Power-Automate-Verlauf, auch nach vollständigem Neuladen/neuer "Wiedergeben"-Sitzung). Das Cockpit zeigte durchgehend Fallback-Werte (0/5 Agents Active, „Agent 4 status call failed at  - showing fallback defaults ()" mit LEERER Uhrzeit UND leerem Fehlertext).

**Diagnose-Weg (zur Wiederverwendung bei ähnlichen Fällen):**
1. `pac canvas download` bestätigte: Formel war syntaktisch korrekt und sauber veröffentlicht (kanonisches en-US-Format, keine Beschädigung).
2. Connector-Bindung (`DataSources.json`, `WorkflowEntityId`) war weiterhin korrekt an Agent 4 gebunden.
3. Trigger-Schema von Agent 4 erwartet keine Parameter – `Run()` ohne Argumente ist korrekt.
4. Keine Warnsymbole bei den Datenquellen in Studio.
5. **Entscheidender Schritt:** Test-/Play-Symbol direkt in der `OnStart`-Formelleiste in Studio genutzt (führt NUR diese Formel isoliert aus und zeigt Fehler sofort inline, ohne kompletten App-Reload) → zeigte 3 unterstrichene, fehlerhafte Variablen: `varAuditLastRunTimestamp`, `varAuditLastSuccessTimestamp`, `varAuditLastFailedTimestamp`.

**Root Cause:** Diese 3 Variablen wurden in `OnStart` nur EINMALIG mit `Set(varX, Blank())` initialisiert und NIRGENDS sonst im gesamten Power-App-Quellcode (weder in `App.pa.yaml` noch `scrHome.pa.yaml`) je wieder gelesen oder mit einem konkreten Wert neu belegt (im Unterschied zu den strukturell ähnlichen `...LastModified`-Variablen, die zwar auch mit `Blank()` initialisiert werden, aber später per `Coalesce(varStatusResult.X, varY)` einen konkreten Typ zugewiesen bekommen). Ohne jede weitere Verwendung kann Power Fx dem `Blank()`-Wert keinen eindeutigen Datentyp zuordnen – das führte zu einem Typ-Fehler, der die GESAMTE `OnStart`-Formel am Ausführen hinderte (nicht nur diese eine Zeile), wodurch praktisch KEINE der `Set(...)`-Anweisungen in `OnStart` mehr durchlief, einschließlich des Agent-4-Aufrufs selbst.

**Fix (Nutzerentscheidung: Variablen bewusst für spätere Verwendung behalten, nicht löschen):** Die 3 betroffenen Zeilen von `Set(varX, Blank())` auf `Set(varX, "")` (leerer Text) umgestellt – das legt den Datentyp eindeutig auf Text fest (passend zur späteren geplanten Verwendung als Zeitstempel-Text) und behebt den Typkonflikt, ohne die Variablen zu entfernen oder Funktionalität zu verlieren. Nach dieser Änderung lief `OnStart` wieder vollständig durch, Agent 4 wird wieder korrekt aufgerufen, alle KPIs/Next-Steps zeigen wieder Live-Daten.

**Wichtige Lehre für künftige Variablen-Deklarationen in `App.OnStart` (oder vergleichbaren Formeln):**
- Eine mit `Blank()` initialisierte Variable, die NIRGENDS sonst im Code verwendet/neu zugewiesen wird, kann einen App-weiten Typ-Fehler verursachen, der die GESAMTE umgebende Formel am Ausführen hindert – nicht nur einen lokalen, isolierten Fehler an der Deklarationsstelle selbst.
- Bei „geplant für später, aber aktuell nicht verwendet"-Variablen `""` (Text) oder einen anderen eindeutig typisierten Platzhalter verwenden, NICHT `Blank()`, solange die Variable nicht zusätzlich an mindestens einer weiteren Stelle mit einem konkreten Typ verwendet wird.
- **Wichtigstes Diagnose-Werkzeug für „App startet/lädt nicht richtig, aber keine sichtbaren Fehler"-Fälle:** Das Test-/Play-Symbol direkt in der Formelleiste einer Behavior-Formel (z. B. `App.OnStart`) in Power Apps Studio nutzen – es führt die Formel isoliert aus und markiert genau die fehlerhaften Teilausdrücke inline, was weit präziser und schneller ist als das Herunterladen/Vergleichen des Live-Standes oder Testen über volle App-Neustarts.

---

# ✅ GELÖST (2026-08-25): Agent 3 und Agent 5 – Postfach-Unterordner-Erstellungsfehler blieb komplett unsichtbar (kritischer Bug, KEIN Alert-Mail-Versand)

**Symptom / Root Cause:** In beiden Flows lief die Aktion `E-Mail_Folder_creation` (legt die Postfach-Unterordnerstruktur an) als von der Audit-/Warnungs-Logik komplett ENTKOPPELTER Parallelzweig. Ein Fehlschlag dieser Aktion führte zu KEINER Cockpit-Warnung und KEINER Alert-Mail – der Fehler wäre nur durch manuelles Prüfen des Power-Automate-Laufverlaufs auffindbar gewesen.

- **Agent 5 (Operational State Management):** `SCOPE_AuditTrail_Write` hing nur von `E-Mail_Folder_creation` = `Succeeded` ab, ansonsten passierte gar nichts.
- **Agent 3 (Emergency Report Management) – SCHWERWIEGENDER:** Der komplette `SCOPE_Validation` (die GESAMTE Kernlogik für Datei-Validierung, -Verarbeitung und Audit-Schreiben) war ebenfalls nur auf `E-Mail_Folder_creation` = `Succeeded` verdrahtet. Ein Fehlschlag der Ordnererstellung hätte die komplette Emergency-Report-Verarbeitung für den gesamten Lauf übersprungen – kein Audit-Eintrag, keine Antwort, keine Mail, absolut nichts sichtbar.

**Nutzerentscheidung (neue, dauerhafte Regel für alle Agenten):** Jeder kritische Fehler – ausdrücklich einschließlich fehlgeschlagener Postfach-Unterordner-Erstellung, da dies die Housekeeping-/Archivierungsfunktion bricht – muss als „Failed" (nicht nur „Warning") im Audit-Outcome geführt werden UND IMMER eine Alert-Mail auslösen, weil das Cockpit nicht durchgehend überwacht wird. Reine Cockpit-Zähler reichen für kritische Fehler nicht aus.

**Fix (in beiden Flows identisch als Muster umgesetzt):**
1. `runAfter` der nachgelagerten Kernlogik (Agent 3: `SCOPE_Validation`; Agent 5: `SCOPE_AuditTrail_Write`) so erweitert, dass sie unabhängig vom Ausgang der Ordnererstellung ausgeführt wird (Entkopplung von `E-Mail_Folder_creation.Succeeded` als Blocker).
2. Neue Aktionskette ergänzt: `SET_AuditOutcome_EmailFolderFailed` → `AUDIT_EmailFolderCreation_Failed` → `SET_AlertMailSubject_(EmailFolderCreationFailed)` → `MAIL_Alert_(EmailFolderCreationFailed)` (Agent 3: sequenziert VOR `SCOPE_Validation`, um Race Conditions bei `AuditOutcome`/`AuditEvents` zu vermeiden – die komplette Variablen-Initialisierungskette wurde geprüft, keine Race Conditions gefunden).
3. Nebenbei 6 fragile `[...][0]['id']`-Ausdrücke (ohne Null-Sicherheit) in Agent 3/4/5 auf das bereits bewährte `?[0]?['id']`-Muster umgestellt.

**Bekannte, akzeptierte Einschränkung (Agent 3, dokumentiert, nicht behoben):** Schlägt die Ordnererstellung fehl, die anschließende Emergency-Report-Validierung aber erfolgreich durch, überschreibt der spätere, unbedingte `SET_AuditOutcome_(Success)`-Schritt das `AuditOutcome` wieder auf „Succeeded" – der finale RunSummary-/Agent-Audit-Summary-Zähler zeigt den Ordnerfehler dann u. U. nicht mehr an. Der einzelne `AUDIT_EmailFolderCreation_Failed`-Audit-Event selbst bleibt aber in jedem Fall im Audit-Log erhalten. Eine vollständige Behebung würde eine Restrukturierung der mehreren unabhängigen Erfolgs-/Fehlerzweige in Agent 3 erfordern – als zu riskant/aufwendig für diese Session eingestuft.

**Status:** Beide Fixes sind in den lokalen JSON-Quelldateien vollständig umgesetzt und validiert (JSON-Syntax + `runAfter`-Referenzgraph geprüft), aber **noch nicht per `pac solution import` in die Live-Umgebung deployt**.

---

# ✅ GELÖST (2026-08-25): Vollständige Kachel-Beschreibungs-Bereinigung über alle 5 Agenten

**Auslöser:** Nutzer-Vorgabe, dass JEDE Kachel/Aktion in allen überarbeiteten Flows eine Beschreibung enthalten muss (bestehende KI-Arbeitsregel, jetzt konsequent auf alle 5 Agenten angewendet).

**Ergebnis:** Alle 5 Agenten-Flows wurden programmatisch auf fehlende `description`-Felder gescannt (rekursiv über Scopes/If-else/Switch-Cases) und vollständig ergänzt:
- Agent 1: 22 fehlende Beschreibungen ergänzt.
- Agent 2: 137 fehlende Beschreibungen ergänzt (mit Abstand größter Umfang – 4 parallele Sender-Klassifizierungszweige (No DMP / DMP Internal Sender / DMP effected Member / DMP not effected Sender) hatten jeweils unterschiedliche Lücken, dazu diverse einzigartige Top-Level-Fehlerbehandlungen wie fehlende Counter-/Audit-Dateien).
- Agent 3: 13 fehlende Beschreibungen ergänzt.
- Agent 4: 38 fehlende Beschreibungen ergänzt (überwiegend `SET_X_From_Config`-Muster sowie einzelne `VAR_`/`IF_`-Aktionen).
- Agent 5: 1 fehlende Beschreibung ergänzt.

Insgesamt 211 fehlende Kachel-Beschreibungen über alle 5 Agenten hinweg ergänzt. Nach jeder Ergänzung wurde die jeweilige Datei erneut per PowerShell-Skript validiert: JSON-Syntax gültig, 0 verbleibende fehlende Beschreibungen, `runAfter`-Referenzgraph intakt (keine kaputten Verweise) – für alle 5 Agenten abschließend bestätigt.

**Einordnung (auf Nutzerfrage, warum so viele Lücken bestanden):** Die Lücken sind historisch, kein aktueller Regelverstoß. Die KI-Arbeitsregel „jede Kachel braucht eine Beschreibung" wurde erst am 2026-08-07 verankert; alles, was in den Flows (insbesondere im ältesten und größten Flow, Agent 2) vor diesem Datum gebaut wurde, unterlag ihr noch nicht. Zusätzlich wurde die ND/DIS/DEE/DNES-Zweigstruktur in Agent 2 offenbar mehrfach durch Kopieren eines Zweigs auf einen anderen erweitert – wurde bei einer solchen Kopie die Beschreibung nicht mitgeführt, wiederholte sich die Lücke über mehrere Zweige hinweg.

**Noch offen:** Keine der heutigen Änderungen (Agent 3/4/5-Strukturfixes + alle Beschreibungsergänzungen in Agent 1–5) wurde bereits per `pac solution import` in die Live-Umgebung deployt. Nach dem nächsten Import werden erwartungsgemäß wieder alle 5 Flows deaktiviert und müssen manuell reaktiviert werden.

---

# 🔴 KRITISCH / WIEDERKEHREND: Power-Automate-Connector-Referenzen brechen nach jedem `pac`-Import (2026-08-14)

**Symptom:** Nach `pac solution import` bzw. `pac canvas pack` + Import zeigt das Power-Apps-Studio-Datenpanel bei den 3 Agent-3/4/5-Verbindungen wieder die ALTEN, veralteten internen Namen
(`DMPAgent3(EmergencyReportManagement)`, `DMPAgent3(YESFileManagement)`, `DMPAgent3(StatusCheck)`) statt der vom Nutzer manuell korrigierten aktuellen Namen
(`DMPAgent3.01(EmergencyReportManagement)=>VS`, `DMPAgent3.03(OperationalStateManagement)=>VS`, `DMPAgent4(StatusCheck)=>VS`). Die Power-Fx-Formeln in der App (z. B. `'DMPAgent3(EmergencyReportManagement)'.Run(...)`), die auf den jeweils AKTUELLEN internen Namen verweisen müssen, brechen dadurch STILL (kein Fehler, kein "Submitted for processing", leerer Flow-Run-Verlauf) – der Nutzer merkt es nur daran, dass gar nichts passiert.

**Root Cause (Vermutung, nicht 100% verifiziert):** Die interne Verbindungs-Bindungsmetadaten (`DataSources.json` im `.msapp`-Paket) scheinen Teil des lokal gepackten Standes zu sein (aus `C:\PowerAppWork\DMP_COMMAND\Source`), der noch die ALTEN Bindungen enthält (aus einem früheren `pac canvas download`, lange vor der Agenten-Umnummerierung 2026-08-13). Manuelles Entfernen+Neuhinzufügen der Verbindung in Studio ändert die Bindung nur in der LIVE-App (im Dataverse), nicht im lokalen `pa.yaml`-Source. Beim nächsten `pac canvas pack` + Import wird der lokale (alte) Stand wieder über die Live-App gelegt, wodurch der Nutzer die manuelle Korrektur JEDES MAL wiederholen muss.

**Workaround (aktuell praktiziert, funktioniert aber ist umständlich):**
1. Nutzer korrigiert die Verbindungen in Power Apps Studio manuell (Datenquelle entfernen, neu hinzufügen, exakten neuen internen Namen per Autovervollständigung ablesen).
2. KI aktualisiert alle betroffenen Power-Fx-Formeln in den `.pa.yaml`-Dateien auf den neuen exakten Namen (mit `.`/`(`/`)`/`=`/`>` in einfachen Anführungszeichen).
3. Nach dem NÄCHSTEN Import muss Schritt 1+2 vermutlich wieder durchgeführt werden.

**Betroffene Referenzen (Stand 2026-08-14), falls das Problem wieder auftritt:**
- `App.pa.yaml` (`OnStart`): `'DMPAgent4(StatusCheck)=>VS'.Run()` (Zeile ~58)
- `scrHome.pa.yaml`: `'DMPAgent3.03(OperationalStateManagement)=>VS'.Run(...)` – 4 Stellen (Operating-State-Toggle OnCheck/OnUncheck, Environment-Toggle OnCheck/OnUncheck)
- `scrHome.pa.yaml`: `'DMPAgent3.01(EmergencyReportManagement)=>VS'.Run(...)` – 1 Stelle (Emergency-Report-Replace-Attachment `OnAddFile`)
- **Diagnose-Tipp:** Wenn ein Button/eine Aktion in der App "nichts tut" (kein Fehler, keine Notify-Meldung, leerer Flow-Run-Verlauf), zuerst hier nachsehen, BEVOR man einen Trigger-/Berechtigungsfehler vermutet (der zeigt sich anders: sichtbare Fehlermeldung `WorkflowTriggerIsNotEnabled`).

**⚠️ Update (2026-08-24): Bug bestätigt als echte Ursache der roten Fehler-Symbole nach Import, nicht nur ein flüchtiges Nach-Import-Phänomen.** Bei der Umstellung der Flow-Namen auf das neue Versionsschema (siehe Abschnitt „Flow-Namenskonvention mit Versionsnummer" weiter unten) wurde festgestellt: Die Power-Fx-Formeln für Agent 3 und Agent 5 referenzierten TATSÄCHLICH noch die alten Vor-Umnummerierungs-Namen (`DMPAgent3.03(...)`, `DMPAgent3.01(...)`) – das war nie nur ein Nach-Import-Reset-Artefakt, sondern ein seit der Umnummerierung am 2026-08-13 nie behobener echter Bug in den `.pa.yaml`-Dateien selbst. Am 2026-08-24 im Zuge der Versionsnummer-Umstellung korrigiert (siehe unten) – die alten `DMPAgent3.03(...)`/`DMPAgent3.01(...)`-Referenzen existieren nicht mehr im Quellcode.

**Weiterhin gültig:** Das grundsätzliche Nach-Import-Risiko bleibt bestehen – nach JEDEM künftigen `pac`-Import sollte weiterhin geprüft werden, ob die Verbindungen in Studio korrekt gebunden sind.

**Noch offen / für die dauerhafte Lösung (nicht umgesetzt, da riskant ohne mehr Zeit/Tests):**
- Idee A: `C:\PowerAppWork\DMP_COMMAND\Source` (lokaler pac-Arbeitsordner) einmal per `pac canvas download` NEU vom aktuellen Live-Stand ziehen (NACHDEM der Nutzer die Verbindungen zuletzt manuell korrigiert und gespeichert hat), damit der lokale Source-Stand die aktuellen Bindungen übernimmt. **ACHTUNG:** In dieser Session gab es bereits einen Beinahe-Vorfall, bei dem `pac canvas download --name "DMP COMMAND"` eine KOMPLETT ANDERE, alte App heruntergeladen hat (siehe Abschnitt weiter unten/Session-Memory) – vor jedem erneuten Download IMMER zuerst die App-Identität mit dem Nutzer bestätigen und nach dem Download sofort per grep auf bekannte aktuelle Marker prüfen (z. B. `dotLedOperationalState`, `conKpiCritical`), BEVOR irgendetwas committet wird.
- Idee B: Direktes Bearbeiten von `DMP_COMMAND.msapr` → `msapp/References/DataSources.json`, um die Bindungen dauerhaft zu korrigieren (riskant, gleiche Fehlerklasse wie oben).
- Idee C: Mit dem Nutzer klären, ob es einen Weg gibt, Verbindungsreferenzen über die Dataverse-Solution (statt über das Canvas-App-Paket) zu verwalten, damit sie nicht bei jedem Canvas-Pack zurückgesetzt werden.

**⚠️ Wichtige Konsequenz (2026-08-24):** Der lokale Solution-Quellordner (`C:\PowerAppWork\DMP_COMMAND_Solution\PowerApp\DMP_COMMAND\Source\`) enthält in `DMP_COMMAND.msapr` weiterhin eine EINGEBETTETE, veraltete `DataSources.json` (aus der Zeit vor der Agent-3/5-Namenskorrektur). Die `.pa.yaml`-Textdateien (`App.pa.yaml`, `scrHome.pa.yaml`) wurden zwar nach dem Vorfall vom 2026-08-24 mit dem korrekt reparierten Live-Stand synchronisiert, das übergeordnete `.msapr`-Containerarchiv jedoch NICHT. **Deshalb darf `pac canvas pack` aus diesem lokalen Ordner bis auf Weiteres NICHT mehr zum Deployment verwendet werden** – ein daraus gepacktes `.msapp` würde die korrekten Formel-Texte mit den falschen (alten) Connector-Bindungsdaten kombinieren. Alle künftigen Power-App-Änderungen sind bis zur Bereinigung dieses Sonderfalls ausschließlich manuell direkt in Power Apps Studio einzupflegen (KI liefert die copy-paste-fertige, lokalisierte Formel; Nutzer fügt sie in Studio ein). Die lokalen `.pa.yaml`-Dateien werden weiterhin als Referenz-/Dokumentationskopie aktuell gehalten, aber nicht mehr gepackt.

---

# ⚠️ PERMANENTE EINSCHRÄNKUNG (bestätigt 2026-08-24): Interne Power-Fx-Connector-Namen für Agent 3/4/5 sind dauerhaft an die alten Vor-Umnummerierungs-Bezeichnungen gebunden

**Kontext:** Im Zuge der Einführung einer Versionsnummer in den Flow-Anzeigenamen (`DMP Agent N (...) [1.0.0]`) wurde ausführlich getestet, ob sich der interne, in Power-Fx referenzierte Verbindungsname (`'DMPAgent3.03(OperationalStateManagement)=>VS'.Run(...)` usw.) auf einen saubereren, zum aktuellen Stand passenden Namen ändern lässt.

**Durchgeführte Testreihe (alle ergebnislos, technischer Name blieb unverändert):**
1. Flow umbenennen (Anzeigename ändern) → keine Wirkung auf den technischen Namen.
2. Verbindung in Studio entfernen + mit dem neu benannten Flow neu hinzufügen → keine Wirkung, technischer Name blieb identisch zum alten.
3. Browser-Cookies/Website-Daten gezielt für `make.powerapps.com` gelöscht, neu angemeldet, Schritt 2 wiederholt → weiterhin keine Wirkung.

**Schlussfolgerung:** Der technische Verbindungsname ist server-/mandantenseitig fest an die Flow-GUID gebunden und wird nach der ersten Verbindung nie wieder aktualisiert – unabhängig von Anzeigenamen-Änderungen, erneutem Verbinden oder Client-Cache. Einzige denkbare Lösung wäre die komplette Neuanlage der 3 Flows mit neuer GUID (z. B. über „Speichern unter"/Duplizieren in Power Automate) und anschließende Umstellung der Power-App-Formeln und des lokalen Solution-Quellordners darauf.

**Nutzerentscheidung (2026-08-24):** Nicht umgesetzt – Aufwand/Risiko (Flows duplizieren, Verbindungen neu bestätigen, gründlich nachtesten, lokalen Solution-Ordner inkl. `Solution.xml`/Datei-Umbenennungen nachziehen) steht in keinem Verhältnis zum rein kosmetischen Nutzen (die Funktion selbst arbeitet bereits fehlerfrei mit den alten technischen Namen).

**Bewusst akzeptierte Abweichung vom Prinzip „Wahrheit und Klarheit":** Der technische Verbindungsname, den man beim Öffnen der Formel in Studio sieht, zeigt weiterhin `DMPAgent3.03(OperationalStateManagement)=>VS` bzw. `DMPAgent3.01(EmergencyReportManagement)=>VS` bzw. `DMPAgent4(StatusCheck)=>VS` – diese Bezeichnungen sind historisch (Stand vor der Agenten-Umnummerierung 2026-08-13) und entsprechen NICHT mehr der aktuellen Agentenbezeichnung. Das ist rein kosmetisch/irreführend für jeden, der den Quellcode liest, hat aber KEINE funktionale Auswirkung.

**Positive Konsequenz/Erkenntnisgewinn:** Da der technische Name nachweislich unabhängig vom Flow-Anzeigenamen ist, kann der Flow-Anzeigename (inkl. der neuen Versionsnummer `[x.y.z]`) ab sofort beliebig oft geändert werden, OHNE dass jemals wieder die Power-Fx-Formeln (`'...'.Run()`) angepasst werden müssen. Die Versionsnummer im Flow-Namen und der technische Connector-Name sind damit dauerhaft und sicher voneinander entkoppelt.

**Für künftige Bearbeiter:** Sollte dieses Thema erneut aufkommen (z. B. weil ein neuer Mitarbeiter sich über die alten Namen wundert oder eine Bereinigung vorschlägt) – dieser Abschnitt dokumentiert bereits die vollständige Testreihe; nicht erneut Zeit mit Cache-Leeren o. ä. verschwenden, sondern direkt zur Flow-Neuanlage übergehen, falls eine Bereinigung gewünscht wird.

---

# ⏸️ OFFEN: Agent 5 – Postfach-Ordner-Erstellungsfehler wird nicht als Warnung in der GUI angezeigt (gefunden 2026-08-24, NICHT behoben)

**Symptom (vom Nutzer gemeldet):** Beim manuellen Ausführen von Agent 5 (Operational State Management) traten 3 Fehlermeldungen im Flow-Lauf auf:
- `Create_Mailbox_Subfolder_"PA_Processed_Mails"`: *"The folder save operation failed due to invalid property values."*
- `Create_Mailbox_Subfolder_"Agent_5_Alerts"`: **InvalidTemplate** – *"Unable to process template language expressions..."*
- `Get_DMP_Mailbox_Subfolder_ID_for_"Agent_5_Alerts"`: *"Unable to process template language expressions..."*

Trotz dieser sichtbaren Fehler zeigte das Cockpit **0 Critical/0 Warnings** – der Nutzer hatte zu Recht eine Warnung erwartet.

**Root Cause (identifiziert, Datei `DMPAgent303OperationalStateManagementVS-34532EFF-5796-F111-8075-7CED8D11CBB4.json`):**
1. Die Kachel-Gruppe `E-Mail_Folder_creation` (Postfach-Ordner für Processed Mails + Agent-5-Alerts anlegen/auflösen) läuft als **eigenständiger, paralleler Scope**, der nur von `CMP_ConfigObject` abhängt – NICHT vom eigentlichen Haupt-Ablauf (Moduswechsel → `AuditOutcome` setzen → Zähler schreiben). Ein Fehler in diesem Zweig fließt daher NIRGENDS in `AuditOutcome`, `AuditEvents` oder die Agent-Audit-Summary-Zähler ein – der Lauf wird trotzdem als `Succeeded` gezählt.
2. Vermuteter Auslöser des ursprünglichen "invalid property values"-Fehlers: ein transienter Graph-API-Fehler beim Anlegen von `"PA_Processed_Mails"` (Ordner existierte vermutlich schon, oder kurzzeitiger Dienstfehler).
3. Weil `Create_Mailbox_Subfolder_"PA_Processed_Mails"` fehlschlug, lieferte die nachfolgende `Get_DMP_Mailbox_Parent_Folder_ID`-Abfrage vermutlich ein leeres `value[]`-Array zurück. Der nachgelagerte Ausdruck `body(...)?['value'][0]['id']` (ungeschütztes Indizieren auf Position `[0]`) bricht dann mit "Unable to process template language expressions" ab, sobald das Array leer ist – das erklärt die 2 Folgefehler als Kettenreaktion des ersten.

**Noch zu tun (nicht umgesetzt, nächste Session):**
1. `E-Mail_Folder_creation`-Scope so umbauen, dass sein Erfolg/Misserfolg tatsächlich in `AuditOutcome`/`AuditEvents` einfließt (z. B. als zusätzliches Audit-Event mit eigenem `StepName`, das bei Fehlschlag `AuditOutcome` auf `Warning` statt `Succeeded` setzt – nicht zwingend `Failed`, da der Hauptzweck des Laufs – der Moduswechsel – ja weiterhin gelingen kann).
2. Die Ausdrücke `body(...)?['value'][0]['id']` (3 Fundstellen, auch in Agent 3 und Agent 4 identisch vorhanden – gleiches Muster, gleiche Schwachstelle!) robuster gegen leere Arrays machen, z. B. mit `first(body(...)?['value'])?['id']` statt `[0]`-Index, um die Kettenreaktion bei einem einzelnen fehlgeschlagenen Ordner-Anlegen zu verhindern.
3. Cross-Check: Dasselbe `E-Mail_Folder_creation`-Muster (Ordner-Erstellung als unverbundener Parallelzweig) existiert identisch in Agent 3 (`DMPAgent301EmergencyReportManagementVS-...json`) und Agent 4 (`DMPAgent302StatusCheckVS-...json`) – bei der Korrektur in Agent 5 direkt mitprüfen, ob dieselbe Lücke dort ebenfalls geschlossen werden soll.

---

# ✅ Power-App-Versionsnummer über externe Textdatei statt fest codierten Text (2026-08-24)

**Auslöser:** Nutzer wollte wissen, warum trotz mehrfachen Deployments weiterhin „v1.1.13" im Header angezeigt wurde – Ursache: dieser Text war seit jeher ein fest einprogrammierter String (`lblVersionTag.Text = "v1.1.13"`), keine automatische Build-Kennung. Da jede künftige Versionsänderung sonst wieder eine riskante manuelle Studio-Formel-Bearbeitung erfordert hätte (siehe Gebietsschema-Problem oben), wurde stattdessen ein dateibasierter Mechanismus eingeführt, den die KI ohne Studio-Zugriff selbst pflegen kann.

**Mechanismus:**
- Neue Textdatei `PowerApp_Version.txt` im neuen Ordner `PowerApp_Storage` (SharePoint-synchronisiert, analog zu `External_Domains_Storage`/`Internal_Domains_Storage`/`Emergency_Report_Storage`), Inhalt schlicht die aktuelle Versionsnummer (z. B. `v1.2.0`).
- Agent 4 (Status Check) liest diese Datei bei jedem Aufruf zusätzlich aus (neue Scope `SCOPE_PowerAppVersion_Read` mit `GET_PowerAppVersion_Content`/`SET_PowerAppVersionText`, läuft parallel zu `SCOPE_AuditSummary_Read`) und gibt den Inhalt als neues Antwortfeld `appversion` zurück. Pfad/Dateiname werden – konsistent mit dem Rest des Systems – aus 2 neuen zentralen Konfigurationsparametern gelesen, mit hartcodiertem Fallback falls diese fehlen.
- Neue Konfigurationsparameter (vom Nutzer am 2026-08-24 angelegt, Scope `PowerApps`): `PowerAppVersionFolderName` (`/Shared Documents/Email Hotline/AI_Agent/PowerApp_Storage`), `PowerAppVersionFileName` (`PowerApp_Version.txt`).
- Power App: neue Variable `varAppVersion` (Default `"v1.2.0"` in `App.OnStart`, überschrieben aus `varStatusResult.appversion` nach dem Agent-4-Aufruf); `lblVersionTag.Text` liest jetzt `Coalesce(varAppVersion, "v1.2.0")` statt eines festen Strings.

**Ergebnis:** Künftige Versionsänderungen erfordern nur noch, dass die KI den Inhalt von `PowerApp_Version.txt` ändert (reiner Dateizugriff, kein SharePoint-Login, kein Studio, kein `pac`-Import nötig) – die Power App zeigt den neuen Wert beim nächsten App-Start automatisch an.

**Noch offen:** Die beiden oben genannten Power-Fx-Formeln (`App.OnStart`, `lblVersionTag.Text`) müssen noch vom Nutzer manuell in Studio eingefügt werden (siehe Konsequenz-Hinweis oben zum aktuell nicht nutzbaren `pac canvas pack`-Weg). Aktuelle Versionsnummer in der Datei: `v1.2.0` (angehoben von der zuvor fest codierten `v1.1.13`, da an diesem Tag umfangreiche echte Änderungen vorgenommen wurden).

---

# ✅ Agenten-Umnummerierung abgeschlossen (2026-08-13): Agent 3.01/3.02/3.03 → Agent 3/4/5

**Entscheidung des Nutzers (2026-08-13):** Durchgängige sequenzielle Nummerierung aller 5 Agenten statt der bisherigen Dezimalschreibweise, um unnötige Komplexität zu vermeiden. Zuordnung: Agent 1 bleibt 1, Agent 2 bleibt 2, **Agent 3.01 → Agent 3** (Emergency Report Management), **Agent 3.02 → Agent 4** (Status Check), **Agent 3.03 → Agent 5** (Operational State Management). Interne Schlüssel 2-stellig: `Agent_01`.."Agent_05" (AgentKey). Scope-Werte initial ohne Padding (`Agent1`.."Agent5"), am selben Tag noch auf das finale, einheitliche Muster `Agent 01`.."Agent 05" umgestellt (mit Leerzeichen, Zero-Padding) – siehe Abschnitt „Scope-Muster-Vereinheitlichung" unten.

## Durchgeführt (dateibasiert, via Power-Automate-Solution `DMP_COMMAND_Solution`)
- Flow-Anzeigenamen umbenannt (`.json.data.xml` Name/LocalizedName): "DMP Agent 3 (Emergency Report Management)", "DMP Agent 4 (Status Check)", "DMP Agent 5 (Operational State Management)".
- Interne Referenzen in allen 3 betroffenen Flow-JSONs umgestellt: `StatusAgentKey`-Werte (`Agent_03_01`→`Agent_03`, `Agent_03_02`→`Agent_04`, `Agent_03_03`→`Agent_05`), `WorkflowPathAgentNNN`-Konfigurationsschlüssel-Referenzen (`WorkflowPathAgent301`→`WorkflowPathAgent3`, `...302`→`...4`, `...303`→`...5`), Beschreibungstexte ("Agent 3.01"→"Agent 3" usw.), Scope-Filter in Agent 5 (`Scope eq 'Agent3_03'`→`Scope eq 'Agent5'`).
- Gepackt und erfolgreich importiert.
- Cross-Referenz-Check: Agent 1/2 referenzieren keine der umbenannten Agenten – keine weiteren Anpassungen dort nötig. Agent 3/4 nutzen Title-basierte statt Scope-basierte Config-Filter – dort keine Scope-Anpassung nötig.

## ✅ SharePoint-Nacharbeit vom Nutzer selbst durchgeführt (bestätigt 2026-08-13, per frischem CSV-Export)
Alle ursprünglich hier gelisteten manuellen SharePoint-Änderungen (Punkte 1-8, altes Muster `Agent3_01`/`Agent3_02`/`Agent3_03` → `Agent3`/`Agent4`/`Agent5`) wurden vom Nutzer eigenständig durchgeführt – SSO/MFA-Zugriff war für die KI aus Sicherheitsgründen nicht möglich (siehe Grund unten). Der Nutzer ist danach noch einen Schritt weitergegangen und hat den `Scope`-Wert **zusätzlich auf ein einheitliches 2-stelliges Muster `Agent 01`..`Agent 05` (mit Leerzeichen, Zero-Padding) umgestellt** – siehe neuen Abschnitt unten.

**Grund für den ursprünglichen manuellen Umweg:** Direkter Browser-Zugriff auf SharePoint erforderte eine interaktive Anmeldung (SSO/MFA), die aus Sicherheitsgründen nicht durch die KI selbst durchgeführt werden darf.

---

# ✅ Flow-Namenskonvention mit individueller Versionsnummer je Agent (2026-08-24)

**Nutzerwunsch:** Ähnlich zur Power-App-Versionsanzeige (`v1.1.13` im Header) sollen auch die 5 Power-Automate-Flow-Anzeigenamen eine Versionsnummer tragen, statt des bisherigen technischen Suffix `=> VS`. Jeder Agent/Flow führt dabei eine **eigene, unabhängige** Versionsnummer (keine gemeinsame App-weite Nummer).

**Format-Entscheidung des Nutzers:** Version am Ende des Namens (nicht zwischen Agentennummer und Zweck), Klammerform `[x.y.z]` – konsistent mit der bestehenden Namenskonvention `DMP Agent N (<Zweck>)`, nur der Suffix wurde ersetzt.

**Neue Namen (alle starten bei `[1.0.0]`, danach unabhängig hochzuzählen):**
- `DMP Agent 1 (Domains Extraction) [1.0.0]`
- `DMP Agent 2 (E-Mail Inbox Treatment) [1.0.0]`
- `DMP Agent 3 (Emergency Report Management) [1.0.0]`
- `DMP Agent 4 (Status Check) [1.0.0]`
- `DMP Agent 5 (Operational State Management) [1.0.0]`

**Umgesetzt:**
- Alle 5 `.json.data.xml`-Dateien (`Name`- und `LocalizedName`-Attribut) im Solution-Quellordner `PowerAutomate\DMP_COMMAND_Solution\Source\Workflows\` umbenannt (`=&gt; VS` → `[1.0.0]`).
- Alle Power-Fx-Aufrufe in der Power App entsprechend nachgezogen: `App.pa.yaml` (1 Stelle, Agent 4), `scrHome.pa.yaml` (5 Stellen: 4× Agent 5, 1× Agent 3) – dabei gleichzeitig den unter „Connector-Referenzen brechen..." dokumentierten, seit 2026-08-13 bestehenden echten Bug behoben (Agent 3/5 referenzierten fälschlich noch die alten Vor-Umnummerierungs-Namen `DMPAgent3.03(...)`/`DMPAgent3.01(...)`).
- Solution neu gepackt (`pac solution pack`) und erfolgreich importiert (`pac solution import`); Power App neu gepackt (`pac canvas pack`).

**Noch offen:**
- Nutzer muss nach diesem Import erneut alle 5 Flows manuell reaktivieren (jeder Solution-Import deaktiviert grundsätzlich alle enthaltenen Flows).
- Nutzer muss das aktualisierte `DMP_COMMAND_Solution.msapp` erneut in Power Apps Studio öffnen/importieren und die 3 betroffenen Datenquellen-Verbindungen (Agent 3, 4, 5) prüfen – die exakte interne Bindungsnamens-Normalisierung von Klammer-Sonderzeichen (`[`, `]`, `.`) durch Power Apps ist nicht 100% verifiziert (gleiche Unsicherheit wie im Abschnitt „Connector-Referenzen brechen..." beschrieben). Falls die App nach dem Import weiterhin rote Fehler-Symbole bei den entsprechenden Aktionen zeigt, exakten autovervollständigten Namen aus Studio zurückmelden, damit die Formeln nachgezogen werden können.
- Künftige Versionserhöhungen pro Agent liegen in der Verantwortung des Nutzers/der Entwicklung – kein automatischer Zusammenhang mit Code-Änderungen an dieser Stelle festgelegt.

---

# ✅ Scope-Muster-Vereinheitlichung + Alert-Folder-Trennung + Namens-Designregel (2026-08-13)

**Entscheidung des Nutzers:** Der `Scope`-Wert in `DMP Command Configuration` wird für ALLE Agenten (auch 1 und 2, bisher `Agent1`/`Agent2` ohne Leerzeichen/Padding) einheitlich auf `Agent 01`..`Agent 05` (2-stellig, mit Leerzeichen) umgestellt. Zusätzlich erhält jeder Agent (auch 3/4/5, vorher nur 1/2) einen eigenen dedizierten `AgentNAlertFolderName`-Parameter statt eines geteilten Scopes.

## Durchgeführt
- **CSV-Verifikation:** Frischer Export geprüft – alle 7 verwaisten Yes.txt-Ära-Parameter (`RequestedActionCreate/Delete`, `RealDMPIndicatorFileName/Folder`, `YesFileCreateSuccessMessage/DeleteSuccessMessage/FailureSubject`) sind vom Nutzer bereits gelöscht. `Scope`-Spalte durchgängig `Agent 01`..`Agent 05`/`Global`/`PowerApps` – keine Reste des alten Musters (`Agent1`, `Agent3_All`, etc.) mehr vorhanden.
- **Flow-seitiger Scope-Filter-Fix (Agent 5, `GET_DMP_Command_Configuration`):** `Scope eq 'Agent5' or Scope eq 'Agent3_All'` → `Scope eq 'Agent 05'` (die alte `Agent3_All`-Design-Frage aus dem vorigen Abschnitt ist damit **aufgelöst**: durch die neuen dedizierten `Agent3/4/5AlertFolderName`-Parameter ist ein geteilter Scope nicht mehr nötig). Agent 1/2/3/4 filtern ohnehin nicht Scope-basiert (Agent 1/2/3 laden alle aktiven Zeilen, Agent 4 filtert Title-basiert) – dort war keine Anpassung nötig.
- Gepackt und erfolgreich importiert (Flow danach vom Nutzer wieder aktiviert).
- **Interne Aktionsnamen bereinigt** (Flow-JSONs Agent 3/4/5): `GET_StatusRow_Agent_3.0X`/`UPDATE_StatusRow_Agent_3.0X` → `GET_StatusRow_Agent_0X`/`UPDATE_StatusRow_Agent_0X` (reine Code-Hygiene, keine funktionale Änderung, JSON-Validität geprüft).
- **Power-App-Legende bereinigt:** Agent-Heartbeat-Legende zeigte noch "Agent 3.01"/"3.02"/"3.03" (Controls `conLegendAgent301/302/303`) – umbenannt zu `conLegendAgent3/4/5`, sichtbarer Text zu "Agent 3"/"Agent 4"/"Agent 5". Gepackt nach `DMP_COMMAND_TEST.msapp` – Import über Power-Apps-Portal steht noch aus (Nutzer-Aktion).
- **Dokumentation konsolidiert:** Alle veralteten/überholten Dateien im Documentation-Root nach `ARCHIVE/` verschoben (alte Workflow-HTML-Diagramme, alte Agent-3.01/02/03-JSON-Exports, alte Master-/Handover-Docx, veraltete Referenzdaten-Snapshots, redundanter PowerApp-Arbeitsordner). Git-Repo enthält jetzt ausschließlich die 5 aktuell gepflegten Dokumente (Backlog, 2 CSVs, Mission-Datei, Operations Manual).

## ⚠️ Noch offen – bekannte Restartefakte in der CSV (nur Beschreibungstexte/Pfadwerte, nicht funktional kritisch)
Bei der Verifikation gefunden, **nicht von der KI editiert** (CSV ist ausschließlich nutzergepflegt):
- `EmergencyReportFileName`.Description nennt noch "Agent 3.01" (sollte "Agent 3" heißen).
- `StatusCheckAuditCardEnabled`/`CounterCardEnabled`/`DomainsCardEnabled`/`EmergencyReportCardEnabled`.Description nennt noch "Agent 3.02" (sollte "Agent 4" heißen).
- `RejectedFolderAgent3`/`WorkFolderAgent3` haben Pfadwerte im alten Stil (`Agent3_Rejected`, `Agent3_Work` – kein Zero-Padding/Leerzeichen). Da dies **echte SharePoint-Ordnernamen** sind, wäre eine Umbenennung eine tatsächliche Datei-Umbenennung in SharePoint (nicht nur Konfigurationswert) – bewusst nicht automatisch angefasst, Entscheidung beim Nutzer.

## 🆕 Design-Regel: Namenskonvention für neue Agenten (ab 2026-08-13 verbindlich)
Bei Einführung eines neuen Agenten (Nummer `N`, 2-stellig als `NN`):
1. **Keine Dezimal-Unternummerierung mehr** (kein "Agent 3.01"-Stil) – fortlaufende Ganzzahl.
2. **Anzeigename:** `DMP Agent N (<Zweck>)`.
3. **`Scope`-Wert** (Configuration-Liste): `Agent NN` – 2-stellig, Zero-Padding, MIT Leerzeichen (z. B. `Agent 06`). `Global` für agentenübergreifende Parameter, `PowerApps` für reine GUI-Werte.
4. **`AgentKey`** (Agent-Status-Liste): `Agent_NN` (Unterstrich, Zero-Padding) – bewusst ANDERES Format als der Scope-Wert, um beide Listen klar auseinanderzuhalten.
5. **`WorkflowPathAgentN`-Wert:** `AgentNN_<PascalCaseZweck>` (z. B. `Agent06_NeuerZweck`).
6. **Dedizierter Alert-Ordner:** jeder Agent, der Alert-/Fehler-Mails versendet, bekommt einen eigenen `AgentNAlertFolderName`-Parameter (Scope = eigener `Agent NN`-Scope), Wert `Agent NN Alerts`. Keine geteilten Alert-Ordner mehr über mehrere Agenten hinweg.
7. **Interne Flow-Aktionsnamen** (z. B. `GET_StatusRow_...`, `UPDATE_StatusRow_...`): `_Agent_NN`-Suffix, konsistent mit dem AgentKey-Format.
8. Alle 4 Modus-Spalten (PROD/SIMU × NODMP/DMP) müssen befüllt sein, auch wenn der Wert überall identisch ist.

---

# ✅ Agent 4 (Status Check) Rebuild + Live-Datenanbindung Power App (2026-08-13)

**Auslöser:** Nutzer-Prioritätsvorgabe „Sofort abarbeiten: Agent 4 (Status Check)" + gemeldete GUI-Probleme (Agent 2 Emails Processed nicht angeschlossen, Files-Anzeige nicht angeschlossen).

## Durchgeführt
- **RealDMP-Rollback:** War in einem früheren Durchgang bereits erledigt (`SET_IsRealDMP_From_OperationMode` leitet korrekt aus `CurrentOperationMode` ab). Nur noch totes Restgerüst gefunden und entfernt: `VAR_RealDMPIndicatorFileName`/`VAR_RealDMPIndicatorFolder` + `SET_RealDMPIndicatorFileName_From_Config`/`SET_RealDMPIndicatorFolder_From_Config` (keine Live-Datei-Prüfung mehr referenziert sie), plus die beiden toten `Title eq 'RealDMPIndicator...'`-Filter aus `$filter` entfernt.
- **Select+Join-Migration (Finding 2) abgeschlossen:** Alte `APPLY_TO_EACH_ConfigItem`-Foreach-Schleife durch das bewährte `Select_ConfigEntries`/`CMP_ConfigJsonText`/`CMP_ConfigObject`-Muster (wie Agent 1/2/5) ersetzt. `VAR_ConfigObject` bleibt als Variable erhalten (`= outputs('CMP_ConfigObject')`), damit alle ~30 nachgelagerten `variables('ConfigObject')`-Referenzen unverändert funktionieren – kein Umbau an jeder einzelnen Stelle nötig.
- **Zurückgestellt (bewusst, Zeit/Nutzen-Abwägung):** Die 9 toten Heartbeat-Statusvariablen (`AuditRunSummaryCount` etc., nie befüllt) wurden NICHT entfernt – rein kosmetisch, kein funktionales Risiko.
- Gepackt und erfolgreich importiert.

## Power App jetzt an echte Daten angebunden (statt statischer Demo-Werte)
Der App fehlte kein Datenzugriff – der Connector `'DMPAgent3(StatusCheck)'` war bereits als Datenquelle registriert (per msapp-Inspektion gefunden), nur nie aufgerufen. `App.OnStart` ruft jetzt `IfError('DMPAgent3(StatusCheck)'.Run(), Blank())` auf und befüllt alle bereits vorher deklarierten (aber nie gesetzten) `varEmergencyReportExists`/`varCounterNoDMP`/etc.-Variablen daraus, mit Fallback auf die bisherigen Default-Werte bei Fehler (kein sichtbarer Error-Banner). Neu: `varAgentsHealthyCount`/`varSystemHealthPercent` (aus den 5 Exists-Flags abgeleitet).
- **Files-Zeile:** alle 5 Status-Punkte + Info-Texte jetzt an `varXExists`/`varXLastModified` gebunden.
- **Agent 2 - Emails Processed Ring:** `vNoDMP/vDEE/vDIS/vDNES` lesen jetzt `varCounterNoDMP`/`varCounterEffected`/`varCounterInternalSender`/`varCounterNotEffected` statt hartcodierter Werte (52/28/12/6).
- **Agent Heartbeat:** komplett neu gestaltet (siehe GUI-Fixes unten).

## GUI-Fixes (gleicher Durchgang, Nutzer-Feedback-Runde 2026-08-13)
- **Heartbeat-Karte:** Titel-Label + 5er-Agenten-Legende entfernt, EIN großer Ring (328×328) füllt jetzt die komplette Karte, Farbe/Prozent dynamisch aus `varSystemHealthPercent`.
- **"DMP COMMAND"-Überschrift im Light Mode unsichtbar:** Kopfleiste (`conTopHeader`) hat fixes dunkles Lila-Fill in beiden Modi, aber der Titel-Text war `If(varDarkMode, weiß, dunkel)` – dunkler Text auf dunklem Lila war unsichtbar. Fix: Farbe jetzt immer Weiß (wie der bereits korrekte Untertitel).
- **KPI-Abstand Critical/Warnings/Agents Active + Versionsnummer-Ausrichtung:** feste Breiten (64/78/100) statt Auto-Breite pro KPI-Spalte gesetzt (der eigentliche Fehler war inkonsistente Auto-Breite, nicht der Gap-Wert), `lblVersionTag`-Höhe auf 44 (wie die KPI-Spalten) für exakte vertikale Zentrierung.
- **Seitliche Scrollbar blockierte Replace-Button:** `conFilesRow` `PaddingLeft` war in einer früheren Runde auf 60 erhöht worden ("4cm nach rechts") – das verursachte einen Breitenüberlauf/horizontale Scrollbar. Zurückgesetzt auf 20.
- **"Plastischer" Look für Maintenance-Domains-Buttons:** `DropShadow: =DropShadow.Light` auf alle 4 View/Edit-Buttons + den Replace-Fake-Button ergänzt (native Power-Apps-Eigenschaft für Tiefenwirkung bei Classic-Controls).

## ⚠️ Bekannter, wiederkehrender Fallstrick: `pac solution import` deaktiviert Flows bei JEDEM Import
Nicht nur beim ersten Mal – jeder Import deaktiviert den/die betroffenen Flow(s) erneut. Nutzer muss nach JEDEM Import (auch Wiederholungsimporten) kurz in Power Automate prüfen, ob die Flows wieder eingeschaltet sind.

---

# ✅ Alert-Mail-Feature für Agent 4 und Agent 5 gebaut (2026-08-14)

**Auslöser:** Nutzerentscheidung (bestätigt): Agent 4 und Agent 5 sollen bei Fehlern das gleiche Alert-Mail-dann-Verschieben-Muster erhalten wie Agent 1/2/3 (Mail an `AlertEmailRecipient`, danach die gesendete Mail in den jeweiligen `Agent4AlertFolderName`/`Agent5AlertFolderName`-Ordner verschieben).

## Agent 5 (Operational State Management)
- Neue Variablen `AlertTargetFolderId`, `AlertMailSubject`, `AlertMessageId`.
- Neuer Scope `E-Mail_Folder_creation` (nach `CMP_ConfigObject`): legt/liest den `Agent5AlertFolderName`-Unterordner im Postfach an, ermittelt `AlertTargetFolderId` – 1:1 Muster von Agent 3 übernommen.
- Bei Fehlschlag von `UPDATE_ConfigRow_CurrentOperationMode` (`SET_AuditOutcome_Failed`): neue Kette `SET_AlertMailSubject_(ConfigUpdateFailed)` → `MAIL_Alert_(ConfigUpdateFailed)` → `Delay` → `Get_Sent_Email_By_Subject` → `SET_AlertMessageId` → `Move_Email_(ConfigUpdateFailed)`, läuft parallel zu `SCOPE_AuditTrail_Write` (blockiert `RESPOND_Result` nicht). Audit-Failure-Event ergänzt um `TargetFolderName`.
- Gepackt, importiert, erfolgreich.

## Agent 4 (Status Check)
- Gleiche neue Variablen + `SET_AlertEmailRecipient_From_Config` + `E-Mail_Folder_creation`-Scope (mit `Agent4AlertFolderName`), `$filter` um `AlertEmailRecipient`/`SharedDMPMailbox`/`ProcessedMailsRootFolderName`/`MailImportanceError`/`WaitSecondsBeforeSentMailSearch`/`Agent4AlertFolderName` erweitert.
- **Besonderheit:** Agent 4 hatte (anders als Agent 5) noch KEINE Mail-Infrastruktur und keinen `InitiatedBy`-Trigger-Input. Als Fehlerfall wurde bewusst **nur der Config-Ladefehler** (`GET_DMP_Command_Configuration` Failed/TimedOut) gewählt – nicht "Datei fehlt" (das ist normaler, erwarteter Status, kein Flow-Fehler). Da in diesem Fall `CMP_ConfigObject` selbst nicht verfügbar ist, nutzt dieser spezielle Zweig (`...(ConfigLoadFailed)`-Aktionen) bewusst **hartcodierte Fallback-Werte** (Mailbox `default@eurex.com`, Ordnername `Agent 04 Alerts`, Importance `High`, Wartezeit `10s`) statt Config-Werten – einzige Möglichkeit, da die Config ja gerade nicht geladen werden konnte.
- Gepackt, importiert, erfolgreich.

## Bewusst nicht behandelt / offen
- Weitere mögliche Fehlerzustände in Agent 4 (z. B. einzelne fehlgeschlagene Datei-Metadaten-Abrufe) lösen aktuell KEINE Alert-Mail aus – nur der komplette Config-Ladefehler. Falls granularere Alarmierung gewünscht ist, mit Nutzer abstimmen.
- Reihenfolge-Risiko (wie bereits bei Agent 3 akzeptiert): `E-Mail_Folder_creation` läuft parallel zur Hauptkette, es gibt keine explizite Abhängigkeit, dass der Ordner sicher fertig angelegt ist, bevor `Move_Email` läuft – identisches, bereits akzeptiertes Muster wie im produktiven Agent 3.

---

# ✅ Bestätigung von Warnings/Alerts + Agent-4-Datenfehler behoben (2026-08-14 begonnen, 2026-08-24 abgeschlossen)

**Kontext:** Entstanden während der Arbeit am Power-App-Header/Cockpit (Anzeige von Critical/Warnings/Agents Active).

## 1) ✅ Backend für Anwender-Bestätigung von Warnings/Alerts vorbereitet (Baseline-Diff-Mechanismus)
**Nutzerentscheidung (2026-08-24):** Zeitstempel pro einzelner Zählerspalte (nicht nur pro Agent), Bestätigung granular pro Agent UND Warnungstyp (4 bestätigbare Zustände pro Agent: FailedSteps, WarningSteps, FailedRuns, WarningRuns — nur die problematischen Zustände, Succeeded/Started brauchen keine Bestätigung).

**Umgesetzt:**
- Neue Tabelle **`AuditAcknowledgment`** in `AuditTrail.xlsx` (Sheet "Audit Acknowledgment"), 9 Spalten: `AgentKey`, `FailedStepsAckBaselineCount`/`FailedStepsAckUtc`, `WarningStepsAckBaselineCount`/`WarningStepsAckUtc`, `FailedRunsAckBaselineCount`/`FailedRunsAckUtc`, `WarningRunsAckBaselineCount`/`WarningRunsAckUtc`. 5 Zeilen `Agent 01`–`Agent 05`, alle Baselines initial `0`.
- Prinzip: Anzeige (LED/Status) vergleicht aktuellen Zähler aus `AgentAuditSummary` gegen die gespeicherte Baseline in `AuditAcknowledgment` → „Neu"/rot, wenn `aktueller Zähler > Baseline`. Ein Klick auf „Bestätigen" schreibt die aktuelle Zählerzahl als neue Baseline + Zeitstempel — kein destruktives Löschen der eigentlichen Zähler, volle Historie bleibt in `AgentAuditSummary` erhalten.
- **Noch offen:** Der eigentliche Schreibvorgang (PatchItem-Aufruf, der eine Baseline aktualisiert) braucht einen UI-Trigger (Button „Bestätigen" in der Power App). Das wird in der GUI-Phase mitgebaut, sobald die entsprechende Kachel/Karte im Cockpit angefasst wird.

## 2) Kontrollierter globaler Reset der Counter / des Audit Trail — weiterhin offen
Noch nicht umgesetzt, unverändert gegenüber der ursprünglichen Beschreibung: 4-Augen-Prinzip nötig, Archivierung vor Reset zwingend, Rollen/Zielordner noch mit Nutzer zu klären. Wird im Rahmen der GUI-Phase (Einstellungen-Bereich) mitgeplant.

## 3) ✅ Agent 4 liefert jetzt echte Critical/Warnings-Zahlen statt immer 0 — VOLLSTÄNDIG ABGESCHLOSSEN (2026-08-24)

### Finale Tabellenstruktur (`AgentAuditSummary`, Sheet "Agent Audit Summary" in `AuditTrail.xlsx`)
17 Spalten, 5 Zeilen (`Agent 01`–`Agent 05`): `AgentKey`, dann je Zählertyp Zähler + Zeitstempel — `SucceededRunsCount`/`SucceededRunsCountLastUpdateUtc`, `FailedRunsCount`/`FailedRunsCountLastUpdateUtc`, `WarningRunsCount`/`WarningRunsCountLastUpdateUtc`, `StartedRunsCount`/`StartedRunsCountLastUpdateUtc`, `SucceededStepsCount`/`SucceededStepsCountLastUpdateUtc`, `FailedStepsCount`/`FailedStepsCountLastUpdateUtc`, `WarningStepsCount`/`WarningStepsCountLastUpdateUtc`, `StartedStepsCount`/`StartedStepsCountLastUpdateUtc`.

### Was am 2026-08-24 fertiggestellt wurde
- **Agent 1, Agent 2, Agent 5:** Identisches Zähler-Muster wie Agent 3 (1:1 kopiert, Agent-Key/Variablennamen angepasst) ergänzt. Je Agent: `GET_AgentSummaryRow_ForStep`→`SET_NewStepsCount`(`add()`!)→`PATCH_AgentSummaryRow_ForStep` innerhalb der jeweiligen Audit-Event-Schleife (Steps-Zähler pro `StepStatus`), sowie einmalig pro Lauf `GET_AgentSummaryRow_ForRun`→`SET_NewRunsCount`→`PATCH_AgentSummaryRow_ForRun` (Runs-Zähler nach finalem `AuditOutcome`). Beide Patch-Aufrufe schreiben zusätzlich das passende `...LastUpdateUtc`-Feld mit `utcNow()`. Agent 2 behandelt seinen Sonderfall `AuditOutcome = 'Started'` konsistent zur bestehenden `RunSummary`-Zeilenlogik (wird wie `Succeeded` gezählt).
- **Agent 3 (bereits vorher gebaut):** Retrofit der beiden Patch-Ausdrücke um die `...LastUpdateUtc`-Felder, damit alle 5 Agenten konsistent sind.
- **Agent 4:** Hatte bisher 0 Audit-Events und keinen `AuditOutcome`. Neu ergänzt: `VAR_AuditOutcome`, `VAR_NewRunsCount`, neuer Scope `SCOPE_AuditSummary_Write` (läuft nach `RESPOND_Status`) mit `SET_AuditOutcome` (aus dem Ausführungsstatus der `RESPOND_Status`-Aktion selbst abgeleitet, da Agent 4 keine granulare Fehlerverzweigung hat) → `GET_AgentSummaryRow_ForRun`→`SET_NewRunsCount`→`PATCH_AgentSummaryRow_ForRun` (Zeile `Agent 04`).
- **Agent 4 liest jetzt zusätzlich alle 5 Zeilen:** Neuer Scope `SCOPE_AuditSummary_Read` (läuft nach `CMP_ConfigObject`, vor `RESPOND_Status`) mit 5× `GetItem` (`Agent 01`–`Agent 05`). `AuditFailedCount` = Summe aller 5 `FailedStepsCount`, `AuditWarningCount` = Summe aller 5 `WarningStepsCount`, `AuditRunSummaryCount` = Summe aller Runs-Zähler-Spalten über alle 5 Zeilen (system-weite Gesamtzahl aller Ausführungen). `RESPOND_Status` wartet jetzt auf beide Scopes (`SCOPE_StatusRow_Update` UND `SCOPE_AuditSummary_Read`).
- Alle 5 Flow-JSONs nach jeder Änderung per JSON-Validierung und vollständiger `runAfter`-Referenzprüfung (0 offene Verweise) geprüft.

**Bewusst zurückgestellt:** Der tatsächliche Import/Test der 5 aktualisierten Flows in der Live-Umgebung sowie ein `pac`-Pack/Import-Durchlauf stehen noch aus (Dateibearbeitung erfolgte direkt im lokalen Solution-Quellordner `C:\PowerAppWork\DMP_COMMAND_Solution\PowerAutomate\DMP_COMMAND_Solution\Source\Workflows`).

### 🎯 Wichtiger Fallstrick (gilt für alle künftigen Erweiterungen dieses Musters)
1. **Power Automate WDL erlaubt keinen rohen `+`-Operator** in Ausdrücken (`@int(x) + 1` schlägt beim Aktivieren fehl: "the value '+' ... cannot be converted to number"). Immer `add(x, 1)` verwenden.
2. **Neue Variablen MÜSSEN explizit per `InitializeVariable` deklariert werden**, bevor sie in einer `SetVariable`-Aktion verwendet werden dürfen (Fehler sonst: "'X' must be initialized before it can be used").

### 🐛 Nachtrag (2026-08-24, nach erstem Deployment-Versuch): Klammerfehler in Agent 4 verhinderte Aktivierung
**Symptom:** Nach `pac solution import` aktivierten sich 4 von 5 Flows problemlos; Agent 4 (Status Check) schlug beim Aktivieren fehl mit: *"The power flow's logic app flow template was invalid. Unable to parse template language expression '...': expected token 'RightParenthesis' and actual 'EndOfData'."*

**Root Cause:** Die beiden neu gebauten Summierungs-Ausdrücke in `SET_AggregatedFailedStepsCount` und `SET_AggregatedWarningStepsCount` (Summe der jeweiligen Spalte über alle 5 Agent-Zeilen) enthielten fälschlich **5 statt der korrekten 4 verschachtelten `add(...)`-Aufrufe** (eine schließende Klammer fehlte am Ende dadurch). Ursache: Fehler beim ursprünglichen manuellen Aufbau der verschachtelten Addition (5 Werte summieren = 4 binäre `add()`-Aufrufe, nicht 5).

**Fix:** Beide Ausdrücke korrigiert (`DMPAgent302StatusCheckVS-EF7E75D5-...json`), auf korrekte Struktur `add(add(add(A,B), add(C,D)), E)` reduziert. Zusätzlich **alle 5 Flow-JSONs programmatisch** (Klammer-Tiefen-Zähler über den geparsten JSON-Baum, nicht nur Sichtprüfung) auf ähnliche Ungleichgewichte in sämtlichen `"@..."`-Ausdrücken geprüft – keine weiteren Fundstellen. Solution neu gepackt und erfolgreich re-importiert (2026-08-24).

**Lehre:** Bei manuell verschachtelten `add()`-Ketten über mehr als 2 Werte IMMER die Klammer-Tiefe programmatisch verifizieren (nicht nur visuell), bevor der Flow importiert wird – Power Automate validiert die Ausdruckssyntax erst beim Aktivieren, nicht beim Speichern/Importieren des Flows, wodurch der Fehler erst spät auffällt.

**Bestätigt (2026-08-24):** Nutzer hat nach dem Re-Import alle 5 Flows reaktiviert – Agent 4 aktiviert sich jetzt fehlerfrei. Alle 5 Flows sind aktiv und produktiv auf dem aktuellen Stand.

## 4) ✅ Hell/Dunkel-Umschalter (`tglTheme` in `scrHome`) Flackern behoben (2026-08-24)
Root Cause und Fix wie ursprünglich analysiert: `Default` des Toggles war an `varDarkModeHeaderSnapshot` gebunden (Screen.OnVisible-Snapshot-Ansatz), was in `scrHome` weiterhin flackerte. Angewendet: das bereits in `scrHeaderTest` Variante A verifizierte, 100% stabile Muster — `Default: =false` (Literalwert statt Variablenbindung). Zusätzlich die dadurch überflüssig gewordene Zeile `OnVisible: =Set(varDarkModeHeaderSnapshot, varDarkMode)` entfernt (kein toter Code, „Wahrheit und Klarheit"-Prinzip). Datei: `PowerApp\DMP_COMMAND\Source\Src\scrHome.pa.yaml` (im Solution-Ordner `DMP_COMMAND_Solution`, NICHT im veralteten separaten Ordner `C:\PowerAppWork\DMP_COMMAND\Source` — dieser enthält laut früherem Fund noch alte Verbindungs-Bindungen von vor der Agenten-Umnummerierung und wird nicht mehr gepflegt). 0 verbleibende Referenzen auf `varDarkModeHeaderSnapshot` im gesamten Power-App-Quellcode verifiziert.

**Noch offen:** `pac canvas pack` + Import des aktualisierten Standes in die Live-App steht noch aus.

---

# ✅ GUI-Kachel-Review `scrHome.pa.yaml` – alle Container geprüft und gegen Live-Daten angebunden (2026-08-24)

**Kontext/Vorgehen (Nutzerentscheidung):** Jede Kachel/jeder Container wird genau EINMAL angefasst — alle nötigen Änderungen (Live-Datenanbindung, Beschreibungstexte, Fehlerbereinigungen, Backlog-Bezüge) werden pro Kachel gebündelt erledigt, damit keine Kachel doppelt aufgemacht werden muss. Reihenfolge: `conTopHeader` → `conHeartbeatCard` → `conOperatingState` → `conMaintenanceDomains` → `conEmailsCard` → `conFilesRow` → `conNextSteps` → `conSidebar`. Alle 8 Container sind jetzt durchgeprüft.

## `conTopHeader`
- **Debug-Banner `lblDebugStatusCallError` war dauerhaft sichtbar** (`Visible: =true` fest verdrahtet) – auf `=false` gesetzt (Text/Logik bleiben für künftiges gezieltes Wiedereinschalten erhalten).
- **KPI „CRITICAL"-Wert war fest grün**, unabhängig vom tatsächlichen Wert – jetzt `=If(Coalesce(varAuditFailedCount,0)>0, rot, grün)`.
- **KPI „Agents Active"-Wert war fest grün** – jetzt `=If(Coalesce(varAgentsHealthyCount,0)>=5, grün, orange)`.
- **`App.OnStart`-Fallback riskant:** Bei fehlgeschlagenem Agent-4-Aufruf fiel der Modus auf `"PROD_DMP"` zurück (würde ein aktives echtes DMP-Ereignis vortäuschen) – auf den sicheren Default `"PROD_NODMP"` geändert.

## `conHeartbeatCard`
Bereits korrekt an `varSystemHealthPercent` (live) gebunden – keine Änderung nötig.

## `conOperatingState`
- **`lblModeValue` war hart auf `"SIMU_DMP"`** codiert, unabhängig vom tatsächlichen Zustand – jetzt live berechnet aus `varEnvironmentIsPROD`/`varOperationalModeIsDMP`.
- **`lblLastChangedValue` zeigte einen erfundenen Namen + Zeitstempel** (`"14:37 · U. Lehmann"`, zusätzlich mit Mojibake-Encoding-Fehler) – ersetzt durch ehrlichen Platzhalter (`"Not changed this session"`) plus neue Session-Variable `varLastChangedText`, die nach jedem erfolgreichen Toggle (beide Schalter, je OnCheck/OnUncheck) live mit echtem Zeitstempel + Nutzername gesetzt wird.

## `conMaintenanceDomains`
- **`lblInternalDomainsCount` zeigte hart `"248 (SharePoint)"`**, obwohl `varInternalDomainsCount` bereits seit dem Agent-4-Rebuild live vorhanden war – jetzt `=Coalesce(varInternalDomainsCount,0) & " (SharePoint)"`.
- **`lblExternalDomainsCount` zeigte hart `"632 (File)"`** – analog auf `=Coalesce(varExternalDomainsCount,0) & " (File)"` umgestellt.
- „Replace"-Mechanismus (`attExternalDomainsReplace`, ruft Agent 3 zur Neugenerierung der External Domains auf) bereits korrekt verdrahtet – keine Änderung nötig.

## `conEmailsCard`
- Ring-Diagramm (`imgEmailsWheel`) war bereits korrekt live an `varCounterNoDMP`/`varCounterEffected`/`varCounterInternalSender`/`varCounterNotEffected` gebunden – keine Änderung nötig.
- **Legenden-Farbe „DIS" war identisch mit „DEE"** (`RGBA(102,52,142,1)` bei beiden), obwohl das zugehörige Ring-Segment für DIS tatsächlich `rgb(153,102,178)` verwendet – Kopierfehler behoben, Legenden-Farbe jetzt `RGBA(153,102,178,1)` (passt zum Ring-Segment).

## `conFilesRow`
Alle 5 Datei-Status-Kacheln (Emergency Report, Internal/External Domains, Counter, Audit Trail) bereits korrekt an `var...Exists`/`var...LastModified` gebunden – keine Logikänderung nötig.

## `conNextSteps`
- **Schritt 1 „Configuration loaded"-Detailtext war komplett erfunden:** `"78 active parameters loaded from SharePoint at 14:35:12 UTC"` (keine Datenquelle für „78 Parameter" existiert). Ersetzt durch echten Status: Neue Variable `varAppStartTimestamp` (`Now()` beim App-Start, `App.pa.yaml`), Text jetzt `=If(varStatusCallError="", "Live status successfully retrieved from Agent 4 at " & Text(varAppStartTimestamp,"hh:mm:ss"), "Agent 4 status call failed at ... - showing fallback defaults")`. Haken/Farbe ebenfalls jetzt abhängig vom tatsächlichen Aufrufergebnis.
- **Schritt 2 „Agent 1 domain extraction"-Detailtext war erfunden:** `"1,248 internal domains extracted successfully"` – ersetzt durch echten Live-Wert `=Coalesce(varInternalDomainsCount,0) & " internal domains currently on file" & (... last updated ...)`. Haken/Farbe jetzt abhängig von `varInternalDomainsExists`.
- Schritte 3–7 sind generische Handlungsaufforderungen ohne Datenanspruch (z. B. „Check Agent 2 inbox" / „Review pending emails...") – bewusst unverändert gelassen, da keine falsche Tatsachenbehauptung enthalten.

## `conSidebar`
Reine Navigation, 5 der 6 Buttons zeigen bewusst `Notify("... - coming next")` (dokumentierte, noch nicht gebaute Platzhalter-Screens laut Operations Manual §3.6) – keine Änderung nötig.

## 🔤 Encoding-Bereinigung (Mojibake), fileweit in `scrHome.pa.yaml`
Bei der Kachel-Review wurden zusätzlich mehrere Mojibake-Artefakte gefunden und behoben (gleiche Fehlerklasse wie der bereits bekannte `Â·`-Fall):
- `●`-Bullets in der Emails-Legende (`â—` + Steuerzeichen) → durch einfachen Bindestrich `-` ersetzt (4 Stellen).
- `●`-Status-Punkte in `conFilesRow` (`â—` + Steuerzeichen) → durch das korrekte Zeichen `●` (U+25CF) ersetzt (5 Stellen, da hier als Farb-Icon gemeint, nicht als Textmarker).
- Mittelpunkt-Trenner `Â·` → durch das korrekte Zeichen `·` (U+00B7) ersetzt (10 Stellen in `conFilesRow`-Infozeilen).
- Häkchen `✓` (war `âœ“`) und Pfeil `→` (war `â†’`) in `conNextSteps` → durch die korrekten Unicode-Zeichen ersetzt (2 bzw. 5 Stellen).
- Alle Ersetzungen wurden per Zeichencode-Analyse (nicht Sichtprüfung) durchgeführt und verifiziert (0 verbleibende `â`/`Â`-Treffer im gesamten `scrHome.pa.yaml`).

## Noch offen (nach Abschluss des gesamten Kachel-Reviews)
- `pac canvas pack` + Import des aktualisierten `scrHome.pa.yaml`/`App.pa.yaml`-Standes in die Live-App steht noch aus (alle Änderungen bisher nur im lokalen Solution-Quellordner).
- Empfehlung: Nach dem Import einen vollständigen Klick-Test aller Kacheln durchführen (insbesondere die neuen bedingten Farben/Texte in Fehler-/Warnfällen, die im Normalbetrieb evtl. nie ausgelöst wurden).
- Der „Bestätigen"-Button für die `AuditAcknowledgment`-Baseline (siehe Abschnitt oben) ist bewusst noch nicht gebaut – sollte als nächster Schritt in `conOperatingState` oder einer neuen Detail-Kachel ergänzt werden, sobald der Nutzer das priorisiert.

## 🐛 Nachtrag (2026-08-24, nach erstem Testimport): Dunkel-/Hell-Modus blinkte weiterhin – echter Bug im Test-Screen `scrHeaderTest` gefunden
**Symptom (vom Nutzer nach `pac canvas pack` + manuellem Import in Power Apps Studio gemeldet):** Beim initialen Laden zeigte der Toggle Position „Dark", der Inhalt aber Light-Farben; nach kurzer Zeit begann die gesamte App automatisch (ohne Klick auf den Toggle) durchgehend zu blinken/flackern – reproduzierbar sowohl im Bearbeitungs-Canvas als auch im echten Play-/Vorschau-Modus, im Light-Modus subjektiv sogar schneller.

**Root Cause:** Der zusätzliche, nicht in der Navigation verlinkte Test-Screen `scrHeaderTest` ("HEADER TEST LAB") enthält einen zweiten Toggle `tglThemeB`, der noch das ursprüngliche, fehlerhafte Bindungsmuster hatte: `Default: =varDarkMode` (statt eines Literalwerts) **und** dessen `OnCheck`/`OnUncheck` schrieben ebenfalls auf die echte, geteilte Variable `varDarkMode` (nicht auf eine isolierte Testvariable wie der bereits sichere `tglThemeA`, der korrekt `varDarkModeTestA` verwendet). Da in Power-Apps-Canvas-Apps die Formeln **aller** Screens durchgehend aktiv sind – unabhängig davon, ob der Screen gerade sichtbar/navigierbar ist – erzeugte diese reaktive `Default`-Bindung auf dieselbe Variable, die der Toggle selbst beschreibt, einen App-weiten Rückkopplungs-Loop, komplett unabhängig vom sichtbaren Toggle in `scrHome`.

**Fix:** `scrHeaderTest.pa.yaml`, `tglThemeB.Default` von `=varDarkMode` auf `=false` geändert (identisches, bereits als stabil erprobtes Muster wie bei `scrHome.tglTheme` und `scrHeaderTest.tglThemeA`). `OnCheck`/`OnUncheck` unverändert gelassen (schreiben weiterhin auf `varDarkMode`, das ist beabsichtigt – nur die reaktive `Default`-Bindung war der Fehler). Datei neu gepackt (`pac canvas pack`), verifiziert: keine weiteren `Default: =varDarkMode`-Bindungen irgendwo im Quellcode.

**Lehre für künftige Testscreens:** Test-/Scratch-Screens, die zur Musterfindung dienen, dürfen NIE auf dieselbe produktive globale Variable schreiben wie der finale, produktive Screen – auch wenn der Test-Screen selbst nicht navigierbar ist. Entweder eine komplett isolierte Testvariable verwenden (wie `varDarkModeTestA`) oder den Test-Screen nach Abschluss der Musterfindung vollständig löschen.

**Noch offen:** Erneuter Import des reparierten Standes und Nutzer-Bestätigung, dass das Blinken jetzt weg ist.

---

# Grundsätzliche Architektur-Entscheidung: Operativer Zustand über 2 unabhängige Schalter (neu ab 2026-08-11, oberste Priorität, gilt für Power App + ALLE Agenten)

**Entscheidung des Nutzers:** Im DMP-COMMAND-Frontend (Power App) sollen künftig **2 unabhängige Schalter** den operativen Zustand des Gesamtsystems bestimmen:
- **Schalter 1:** `SIMU` vs. `PROD`
- **Schalter 2:** `Normal` vs. `DMP`

Die Kombination beider Schalterzustände ergibt direkt einen der 4 bestehenden `CurrentOperationMode`-Werte (`PROD_NODMP`, `PROD_DMP`, `SIMU_NODMP`, `SIMU_DMP`) – **es gibt KEINE weiteren Zwischenzustände**. Die Schalter patchen `CurrentOperationMode` in der zentralen Konfiguration **direkt**; es gibt keinen Umweg mehr über Trigger-Dateien (siehe Ablösung des `Yes.txt`-Mechanismus unten).

**Priorität:** Oberste Priorität des Gesamtprojekts – aber **bewusst NACH** Abschluss der Agenten-Arbeiten einzuplanen (siehe Nutzerentscheidung direkt unten).

## Nutzerentscheidung zur Reihenfolge (2026-08-11)
„Bevor wir uns der GUI widmen, möchte ich vorher erst alle Agenten fertig haben, so dass dann nur noch Anpassungen im Zusammenhang mit der GUI-Entwicklung vorgenommen werden müssen!"

**Konsequenz für die Planung:** Alle Agenten (3.01, 3.02, 3.03, sowie die bereits laufende Item-3-Betrachtung bei Agent 2) müssen VOR dem eigentlichen GUI-Umbau in einen Zustand gebracht werden, in dem sie **ausschließlich und korrekt** `CurrentOperationMode` als alleinige Quelle der Wahrheit nutzen – danach soll am Backend nichts mehr angefasst werden müssen, wenn die neuen Schalter gebaut werden. Konkret:
- **Agent 3.02:** Der Statuswert `isrealdmp` (aktuell per `Yes.txt`-Existenz-Check ermittelt, siehe Rollback-Finding oben) MUSS auf eine `CurrentOperationMode`-Ableitung umgestellt werden – und zwar so, dass er **beide Dimensionen** (SIMU/PROD UND Normal/DMP) sauber wiedergibt, nicht nur die bisherige einzelne `IsRealDMP`-Boolean. Empfehlung: `CurrentOperationMode`-Rohwert selbst mit zurückgeben (z. B. neues Statusfeld `currentoperationmode`), plus ggf. die beiden Boolean-Ableitungen für die App-Anzeige, damit die künftigen 2 Schalter beim Laden der App direkt den richtigen Ausgangszustand zeigen können.
- **Agent 3.03:** Wird durch die neuen Schalter (die künftig direkt `CurrentOperationMode` patchen) vollständig ersetzt. **Keine weitere Optimierungsarbeit** (z. B. die sonst für 3.02/3.03 gleichermaßen vorgesehene Select+Join-Migration) mehr in diesen Agenten investieren – das wäre verlorene Arbeit, sobald der Agent beim GUI-Umbau entfällt. Der Flow selbst bleibt bis zum GUI-Umbau unangetastet **live bestehen** (nicht vorzeitig deaktivieren – siehe Begründung im Agent-3.03-Abschnitt unten), damit der aktuelle (ggf. bereits wirkungslose) Schalter in der App bis zur Ablösung nicht komplett ausfällt/Fehler wirft.
- **Agent 1, Agent 2, Agent 3.01:** Bereits jetzt bzw. nach Abschluss der laufenden Migration ausschließlich auf `CurrentOperationMode`-Basis – kein weiterer Anpassungsbedarf für diese Architekturentscheidung selbst, sofern die jeweilige Konfigurationsmigration (siehe Agent-3.01-Abschnitt) sauber abgeschlossen wird.

**Reihenfolge-Auswirkung:** Diese Entscheidung bestätigt und verschärft die bereits vorgeschlagene Priorisierung (siehe „Priorisierung Gesamt-Backlog" unten) – Agent 3.03 erhält ab sofort **keine Weiterentwicklung mehr**, nur noch Bestandserhaltung bis zum GUI-Cutover.

---

# Frontend-Vision & Brainstorming-Baseline (2026-08-11) – Ausgangspunkt für den GUI-Umbau

**Status:** Agenten-Arbeit (3.01 fertig, 3.02 pausiert, 3.03 eingefroren) laut Nutzerentscheidung als „funktional fertig" betrachtet – Fokus wechselt jetzt auf die Power App. Nutzer hat einen Auszug aus einer früheren Brainstorming-Session als Ausgangspunkt geliefert (nicht alle neuen Erkenntnisse aus dieser Session sind darin bereits eingearbeitet) sowie einen Screenshot des aktuellen Live-Stands.

## Grundidee (aus Brainstorming)
DMP COMMAND soll **kein** klassisches Power-App-Formular sein, sondern wirken wie ein professionelles Eurex Operations Command Center / Trading-Floor-Dashboard / Leitstand / Crisis Control Center / Management Cockpit / Audit Dashboard. Auf einen Blick erkennbar: Betriebsmodus, DMP-Status, Agenten-Gesundheit, Verfügbarkeit kritischer Dateien, Audit-/Counter-Funktionsfähigkeit, Handlungsbedarf. Design-Philosophie: große Karten, Status-Ampeln, Icons, klare Farbcodierung – keine SharePoint-/Excel-Formular-Optik. Eurex-Farblogik definiert (Primär `#003C78`, Sekundär `#00A3E0`, Erfolg `#107C10`, Warnung `#FF8C00`, Fehler `#D13438`, Inaktiv `#A19F9D`).

## Geplante Seitenstruktur (5 Bereiche)
1. **Global Status Bar** (volle Breite): Environment, Simulation/Fire Drill, `CurrentOperationMode`, Last Refresh – Farbwechsel je nach Modus (PROD_NODMP=Grün, PROD_DMP=Rot, SIMU_NODMP=Blau, SIMU_DMP=Orange).
2. **Agent Status Board**: 5 große Karten (Agent 1, 2, 3.01, 3.02, 3.03), je mit Status, letzter Lauf, Dauer, letzte Warnung, letzte Störung, Operation Mode.
3. **Operational Control Board** (größter Bereich, 3×3-Raster): Karten für Emergency Report, YES.txt, Internal Domains, External Domains, Counter, Audit Trail, Agent 1/2/3 (Last Run/Duration/Status). **Nachtrag des Nutzers („Neues Control-Board, aktueller Stand")**: zusätzlich Karten für **Configuration** (Config Loaded, Rows Loaded, CurrentOperationMode, Last Refresh), **Mailbox** (SharedDMPMailbox, Alert Folder, Status), **Operation Mode** (Display Name, Subject Prefix, Mail Mode Text, Environment).
4. **Audit Monitoring**: Tabelle der letzten Events (Timestamp, Agent, Workflow, Step, Result, Duration), farbcodiert nach Ergebnis.
5. **Live DMP Status**: großes Banner (PRODUCTION grün / SIMULATION blau / FIRE DRILL orange / REAL DMP EVENT rot).

**Priorisierung laut Brainstorming:** 1) Status-Ampeln, 2) Control Board, 3) Configuration Layer (komplette Steuerung über zentrale Config, keine Hardcodierungen), 4) Workflow-Transparenz (AuditEvents visualisieren), 5) Operational Command Center (Echtzeitstatus, Krisensteuerung).

## Ist-Stand laut Screenshot (2026-08-11) – Abgleich gegen die Vision
Bereits vorhanden (deckt sich mit Bereich 1 und Teilen von Bereich 3):
- Kopfbereich mit Titel/Untertitel, Status-Zeile (CURRENT STATE: ACTIVE, EVENT TYPE: SIMULATION, STREAM HEALTH: HEALTHY).
- **EVENT CONTROL:** Emergency Report-Karte (Available, Last Modified, Open/Refresh-Icons); **DMP CONTROL-Karte mit EINEM Schalter** „FIRE DRILL" (Initiated-Timestamp) – dies ist exakt der alte, in dieser Session als abzulösend identifizierte `Yes.txt`/Agent-3.03-Mechanismus.
- **CONFIGURATION:** Internal/External Domains-Karten (Available, Domain-Anzahl, Last Modified, Edit/Refresh-Icons).
- **MONITORING:** Counter-Karte (No DMP/Internal Sender/Not Effected/Effected Member-Zahlen, Last Modified) und Audit-Trail-Karte (Runs/Last Run/Last Successful/Last Failed/Warnings/Failures/Last Audit Event – aktuell alle auf 0/„-", da die zugrunde liegenden Werte in Agent 3.02 totes Gerüst sind, siehe oben).
- **AUTOMATION:** Agent-Health-Aggregat (Ready/Running/Failed-Zähler, aber keine 5 Einzelkarten).
- **NEXT ACTION CENTER:** einfache Liste, aktuell nur „Run Agent 1".

## Neue Backlog-Punkte aus dem Power-App-Redesign (2026-08-12)
- **Individuelle Farbeinstellungen:** Der Nutzer soll künftig in den App-Einstellungen seine eigenen Akzent-/Themenfarben festlegen können (Personalisierung), statt nur eines festen Eurex-Farbschemas.
- **Archivierung/Reset für Audit Trail und Counter (wichtig, aber zurückgestellt):** Es wird ein - in den Einstellungen versteckter - Schalter benötigt, um Audit Trail und Counter zu archivieren und anschließend zu leeren/zurückzusetzen. **Idealerweise mit 4-Augen-Prinzip** (zweite Person muss die Aktion bestätigen, bevor sie ausgeführt wird).
- **Power-Automate-Flows in eine eigene Solution überführen (ALM-Verbesserung):** Aktuell liegen die Agent-Flows (Agent 1, 2, 3.01, 3.02, 3.03) lose ("unmanaged") in der Umgebung "DBG Team Productivity (Dev)" - geprüft via `pac solution list`, keine eigene DMP-Solution gefunden. Würden sie einer eigenen Solution zugeordnet, könnte der gleiche automatisierte Code-Workflow wie bei der Power App genutzt werden (`pac solution unpack/pack` statt manuellem Copy-Paste in den Flow-Designer). **Achtung:** Das Hinzufügen zu einer Solution ist ein strukturell größerer ALM-Schritt (betrifft Governance/Zugriff), bewusst nicht ohne Rücksprache umgesetzt.

## Zentrale Konflikte/offene Punkte zwischen Vision, Screenshot und dieser Session's Erkenntnissen
1. **Der einzelne „FIRE DRILL"-Schalter MUSS durch die 2 unabhängigen Schalter (SIMU/PROD + Normal/DMP) ersetzt werden** (siehe Architekturentscheidung oben) – direktes Patchen von `CurrentOperationMode`, kein Agent-3.03-Aufruf mehr.
2. **Die geplante „YES.txt"-Karte (Create/Delete/Open) entfällt komplett**, sobald die 2 Schalter aktiv sind – identischer Grund wie Punkt 1.
3. **Agent Status Board (Bereich 2, 5 Karten) fehlt komplett** – die dafür nötigen Felder (Last Run, Duration, Last Warning, Last Failure, Operation Mode) liegen bereits fertig in `DMP Command Agent Status` (je eine Zeile pro Agent) vor und müssten nur noch angezeigt werden, vermutlich per direktem Read statt über Agent 3.02.
4. **Audit-Trail-Karte zeigt aktuell nur tote/leere Werte** (0/„-") – hängt direkt an der bereits identifizierten toten Heartbeat-Logik in Agent 3.02 (siehe Fund oben); muss im Zuge der Agent-3.02-Neuausrichtung mit echten Daten hinterlegt werden.
5. **Neue „Configuration"/„Mailbox"/„Operation Mode"-Karten (Nutzer-Nachtrag) noch nicht gebaut.**
6. **Bereich 4 (Audit Monitoring-Tabelle der letzten Einzel-Events) und Bereich 5 (großes Live-DMP-Status-Banner) fehlen komplett** – Bereich 4 bräuchte einen neuen Lesezugriff auf die einzelnen `AuditTrail.xlsx`-Zeilen (aktuell nirgendwo im Frontend vorhanden), Bereich 5 wäre eine reine Anzeige-/Layout-Ergänzung auf Basis des bereits vorhandenen `CurrentOperationMode`.
7. **Agent-3.02-Fund aus dieser Session passt direkt in Priorität 3 der Vision** („Configuration Layer... alle Agenten nutzen dieselbe Konfiguration") – die Frage, ob Agent 3.02 die Live-Datei-Checks behält oder auf Reads aus `DMP Command Agent Status` umgestellt wird, sollte im Zuge dieses Frontend-Umbaus mitentschieden werden.

---

# Grundsätzliche Annahme: E-Mail Importance-Stufen (neu ab 2026-08-10, gilt für ALLE Agenten)

**Entscheidung des Nutzers:** Ausgehende agentengenerierte E-Mails sollen ihre Outlook-„Importance" konsequent aus der zentralen Konfiguration beziehen (kein Hardcode `"Normal"` mehr), gestaffelt nach Schweregrad:

| Schweregrad | Bedeutung | Config-Parameter | Wert |
|---|---|---|---|
| Info | Keine User-Aktion erforderlich | `MailImportanceInfo` | `Low` |
| Warning | Hinweis, keine User-Aktion erforderlich | `MailImportanceWarning` | `Normal` |
| Error/Kritisch | User-Aktion erforderlich | `MailImportanceError` | `High` |

Technischer Hintergrund: Microsoft Graph/Outlook kennt für `importance` nur die 3 API-Werte `Low`/`Normal`/`High` (kein `Medium`). Werte sind bewusst modus-unabhängig (identisch in allen 4 Spalten `Value - PROD/SIMU (NODMP/DMP)`), da die Zuordnung eine feste Geschäftsregel ist, keine Umgebungseinstellung.

**Status:** Für Agent 2 in `DMP Command Configuration.csv` (lokale Kopie) ergänzt am 2026-08-10; Nutzer legt die 3 Zeilen zusätzlich in der Live-SharePoint-Liste an. Erstmalig angewendet auf die neu entdeckte zentrale Fehlerbehandlungs-Kategorie (Audit Write Failed / Missing Counter File / Internal Domains List Missing) sowie DIS-Info-Mail.

**Offen:** Bei jeder zukünftigen E-Mail-Aktion (auch in Agent 1, Agent 3.01/3.02/3.03) prüfen, ob `emailMessage/Importance` bereits korrekt auf einen dieser 3 Parameter verweist (statt hartkodiertem `"Normal"`), und wenn nicht, entsprechend nachziehen unter Zuordnung zum passenden Schweregrad.

---



# Documentation Maintenance (Anwenderdokumentation – laufend zu pflegen)

**Regel (in KI-Arbeitsregeln verankert am 2026-08-06):** Die Anwenderdokumentation muss durchgehend auf Englisch verfasst sein und bei jeder größeren fachlichen/technischen Änderung am System nachgezogen werden. Der Assistent soll proaktiv auf Aktualisierungsbedarf hinweisen, auch ohne explizite Nachfrage.

**Status-Update (2026-08-24):** Die ursprünglich hier genannten Dokumente (`DMP_Multi_Agent_Workflow_Documentation.docx`, `Agent1_High_Level_Workflow.html`, `Agent1_Detailed_Workflow.html`, `Agent2_High_Level_Workflow.html`, `Agent2_Detailed_Workflow.html`) wurden am 2026-08-13 im Zuge der Dokumentations-Konsolidierung nach `ARCHIVE/` verschoben und durch **`DMP_COMMAND_Operations_Manual.md`** ersetzt (aktives, laufend gepflegtes Anwenderhandbuch für das Gesamtsystem, alle 5 Agenten + Power App).

**✅ Operations Manual auf aktuellen Stand gebracht (2026-08-24):** War seit dem 13.08. selbst bereits leicht veraltet (§2.4 Agent 4, §3.6 Known Limitations, §7 Troubleshooting beschrieben noch den Vor-Rebuild-Zustand mit statischen Demo-Werten, obwohl der Agent-4-Rebuild + Live-Datenanbindung am 13.08. bereits erfolgt war). Jetzt korrigiert und ergänzt um: Agent-4-Live-Datenanbindung, `Agent Audit Summary`-Tabelle (§5.1), `Audit Acknowledgment`-Tabelle (§5.1), Dark-Mode-Fix (§3.5), Change-Log-Einträge für 13.08./14.08./24.08.

**⚠️ Weiterhin offen: `UAT_Playbook.docx`** (Pfad: `AI_Agent\UAT\UAT_Playbook.docx`, zuletzt geändert 11.06.2026). Geprüft am 2026-08-24: Deckt ausschließlich Agent 2 ab (Testfallkatalog mit Szenarien 3.1–3.x für ND/DIS/DEE/DNES-Pfade). Enthält **keine** Testfälle für:
- Agent 1, Agent 3, Agent 4, Agent 5 (existierten teils noch nicht bzw. waren nicht Gegenstand des Playbooks im Juni)
- Die komplette Agenten-Umnummerierung vom 13.08. (3.01/3.02/3.03 → 3/4/5)
- Die neuen 2-Schalter (SIMU/PROD, Normal/DMP) und den Wegfall des `Yes.txt`-Mechanismus
- Die neue `Agent Audit Summary`/`Audit Acknowledgment`-Infrastruktur (2026-08-14/24)

**Bewusst nicht in dieser Runde bearbeitet:** Eine vollständige Neufassung/Erweiterung des UAT-Playbooks um 4 weitere Agenten sowie neue Testszenarien ist ein eigenständiger, umfangreicher Aufwand, der echtes operatives Test-Know-how (Testumgebungen, Timing, Abnahmekriterien) erfordert. Sollte als eigener Arbeitsblock eingeplant werden, sobald die GUI-Arbeit abgeschlossen ist oder parallel dazu Zeit dafür eingeräumt wird.

---

# Agent 1 (Domains Extraction)

**Referenzdatei:** `Agent_01.json`
**Stand der Prüfung:** 2026-08-06

## Erledigt
- **Phase A (vom Nutzer bereits umgesetzt, Stand mündlich bestätigt am 2026-08-06):** `GET_DMP_Command_Configuration` zeigte fälschlich auf die Status-Liste (`variables('StatusListName')`) statt auf die zentrale Konfigurationsliste. Korrektur: Tabelle auf die GUID der `DMP Command Configuration`-Liste (`c3e96ba6-1c07-4f39-9b4d-0d0a92db6d6a`) gestellt. Dadurch wurde `VAR_OperationMode` vorher dauerhaft auf den Hardcode-Fallback `PROD_NODMP` eingefroren.

## In Arbeit: Phase B – Zentrales Konfigurationsobjekt für Agent 1 (Pilot für Select+Join, Strict Mode)

**Entscheidung des Nutzers:** Für Agent 1 wird bewusst KEIN Fallback auf alte `VAR_*`-Variablen gebaut (anders als bei Agent 2). Die neuen Ausdrücke lesen direkt und ausschließlich aus `CMP_ConfigObject`. Zusätzlich wird hier erstmals das neue, schnelle `Select`+`Join`-Verfahren statt der alten `Apply-to-each`-Schleife eingesetzt (siehe auch Agent-2-Punkt „Config-Ladeschleife" weiter unten – dort ist die alte Variante noch aktiv).

**Neue Arbeitsweise ab 2026-08-06 (KI-Arbeitsregel):** Änderungen werden kachelbezogen gruppiert (eine Kachel = ein Bearbeitungsdurchgang), in tatsächlicher Ablaufreihenfolge geliefert, inkl. Prüfung auf Beschreibungstext-Ergänzung und Cross-Check gegen offene Backlog-Punkte je Kachel.

### Erledigt (Stand 2026-08-06, bestätigt durch Nutzer-Feedback)
1. `GET_DMP_Command_Configuration`: Tabelle = `c3e96ba6-1c07-4f39-9b4d-0d0a92db6d6a`, Filter = `Active eq 'Yes'`, Top = `5000`. ✅
2. Neue Kachel `FILTER_Config_Row_CurrentOperationMode` (Filter array), umbenannt von ursprünglich `FILTER_Config_CurrentOperationMode` für mehr Klarheit. Von: `@body('GET_DMP_Command_Configuration')?['value']`. Bedingung: `item()?['Title']` ist gleich `CurrentOperationMode`. ✅
3. `SET_OperationMode`: „Ausführen nach" = `FILTER_Config_Row_CurrentOperationMode`; Wert-Ausdruck: `@coalesce( first(body('FILTER_Config_Row_CurrentOperationMode'))?['CurrentValue'], 'PROD_NODMP' )`. ✅
4. Neue Kachel `Select_ConfigEntries` (Select, **Textmodus**). Von: `@body('GET_DMP_Command_Configuration')?['value']`. Baut pro Config-Zeile ein Fragment `,"Key":"Value"` inkl. Escaping und Modus-Auflösung über `Value_PROD_NODMP`, `Value_x002d_PROD_x0028_DMP_x0029`, `Value_x002d_SIMU_x0028_NODMP_x0029`, `Value_x002d_SIMU_x0028_DMP_x0029`. ✅
5. Neue Kachel `CMP_ConfigJsonText` (Compose): `@concat('{"dummy":""', join(body('Select_ConfigEntries'), ''), '}')`. ✅ (verifiziert am 2026-08-06, Datei-Stand 14:53 Uhr)
6. Neue Kachel `CMP_ConfigObject` (Compose): `@json(outputs('CMP_ConfigJsonText'))`. ✅ (verifiziert)
7. „Ausführen nach" von `Real_DMP_recognition` erfolgreich auf `CMP_ConfigObject` umgestellt. ✅ (verifiziert)
8. **Schritt 7 (alle Verwendungsstellen umgestellt) – Anweisungen geliefert am 2026-08-06, kachelbezogen/sequenziell (37 Kacheln). ✅ Umsetzung durch Nutzer bestätigt und am 2026-08-07 im JSON-Re-Export vollständig verifiziert (alle 37 Kacheln korrekt auf `outputs('CMP_ConfigObject')?[...]` umgestellt, inkl. `int(...)` bei den 6 Wait-Kacheln).**
   - Mailbox-URIs (4 Kacheln) → `SharedDMPMailbox`, `ProcessedMailsRootFolderName`, `Agent1AlertFolderName`
   - Real-DMP-Indikator (1 Kachel) → `RealDMPIndicatorFolder`, `RealDMPIndicatorFileName`
   - External-Domains (2 Kacheln) → `ExternalDomainsStorageFolder`, `ExternalDomainsFileName`
   - **Finding B (neu entdeckt bei Schritt-7-Analyse):** Quell-Arbeitsblatt war in `Get_Emergency_Report_File_ID` (`ScriptParameters/sheetName`) hart auf den Text `"Emergency Contacts"` codiert — die alte Variable `SourceWorksheetName` wurde dort nie tatsächlich verwendet. Wird jetzt erstmals korrekt an `Agent1SourceWorksheetName` aus der Konfiguration angebunden.
   - Alert-Empfänger (24 Kacheln: alle `Send_...`- und `Buffer_Audit_Event_(Move_...)`-Kacheln in `Domains_File_Write_FAILED`, `Technical_Error_or_NoData`, `Error_handling`, `Handle_Audit_Error`, `Agent_1_Alerts_Folder_ID_is_empty`) → `AlertEmailRecipient`
   - Wait-Sekunden (6 `Delay_...`-Kacheln) → `WaitSecondsBeforeSentMailSearch` (mit `int(...)`)
   - Bei 12 der 24 Alert-Empfänger-Kacheln (den `Buffer_Audit_Event_(Move_..._Failed/Succeeded/NotFound)`-Kacheln ohne bisherige Beschreibung) wurden zusätzlich passende englische Beschreibungstexte formuliert und mitgeliefert.
   - **Zurückgestellt (optional, nur Anzeigetext ohne fachliche Wirkung):** ~~4 Stellen ... nutzen weiterhin `variables('SourceWorksheetName')`~~ ✅ Erledigt im Zuge von Finding D am 2026-08-07, vollständig auf `Agent1SourceWorksheetName` umgestellt.
   - **Audit-Dateiname/-Tabelle (`AuditFileName`, `AuditTableName`) NICHT Teil von Schritt 7** — siehe neuer Punkt „Finding A" unten, dort separat zurückgestellt (gleiche Komplexität wie Agent-2-Item-4).
9. **Weiterhin offene Design-Frage vom Assistenten an den Nutzer (noch nicht entschieden):** Soll nach `CMP_ConfigObject` eine kleine Prüf-/Warn-Kachel eingebaut werden, die sichtbar macht, wenn ein kritischer Config-Wert (z. B. `SharedDMPMailbox`) leer geliefert wird? Ohne Fallback und ohne diese Prüfung könnten leere Config-Werte zu denselben unklaren Graph-400-Fehlern führen wie zuvor bei Agent 2.

### Architekturänderung (2026-08-07) ✅ ABGESCHLOSSEN: `Real_DMP_recognition` und `SimulationPrefix_*` auf zentrale Config umgestellt
Fachliche Entscheidung des Nutzers: Das operative Steuerungskonzept läuft jetzt vollständig über die zentrale Konfiguration (Power-App-gesteuert), nicht mehr über eine Trigger-Datei (`YES.txt`/`Is_a_real_DMP`-Ordner).
- Kachel `Check,_if_Real_DMP_File_is_available` (SharePoint-Datei-Check, verursachte wiederholte Fehlversuche/Hänger im Testlauf) vollständig entfernt.
- `Check_Variable_-_Is_a_real_DMP` ersetzt durch direkte Ableitung aus `variables('OperationMode')` (`PROD_DMP`/`SIMU_DMP` → real DMP).
- `SimulationPrefix_Subject`/`SimulationPrefix_Body` (Compose-Kacheln) komplett entfernt; alle 7 `Compose_Subject_(X)`- und 7 Send-E-Mail-Bodys lesen jetzt direkt `outputs('CMP_ConfigObject')?['SubjectPrefix']`/`['MailModeText']`.
- `VAR_IsRealDMP` als Housekeeping mit entfernt (nach Nutzerentscheidung, agentenübergreifend zurückzubauen).
- **Cross-Agent-Fund:** Agent 2 nutzt bereits das saubere Config-Muster (kein Rückbau nötig). Agent 3.02 (Status Check) hat denselben veralteten `VAR_RealDMPIndicatorFileName`/`VAR_RealDMPIndicatorFolder`/`VAR_IsRealDMP`-Musteraufbau — **noch offen**, benötigt JSON-Export von Agent 3.02 für präzise Kachel-Anweisungen. Agent 3.03 (YES File Management) nutzt den Ordner/Datei als Kernfunktion (Datei-Verwaltung selbst) — kein Rückbau, nur bei Erstprüfung sauber einordnen.

### Finding C (2026-08-07): `Shared DMP Mailbox` außerhalb von Schritt 7 ✅ ABGESCHLOSSEN
Alle 52 ursprünglichen Fundstellen von `variables('Shared DMP Mailbox')` sowie die zusätzlich gefundenen `Shared DMP Mailbox - Processed E-Mails Folder` (12) und `... (Agent 1 Alerts)` (28) vollständig auf `outputs('CMP_ConfigObject')?['SharedDMPMailbox']` / `['ProcessedMailsRootFolderName']` / `['Agent1AlertFolderName']` umgestellt und am 2026-08-07 final verifiziert (0 verbleibende Referenzen).

### Finding D (2026-08-07): `RunPrefix`/`WorkflowPath` ✅ ABGESCHLOSSEN
Alle Referenzen auf `variables('RunPrefix')` und `variables('WorkflowPath')` (ursprünglich 22/35 Stellen) sowie die ergänzend gefundenen Anzeigetext-Reste (`SourceWorksheetName`, `ExternalDomainsFolder`, `ExternalDomainsFileName`) vollständig auf `outputs('CMP_ConfigObject')?['RunPrefixAgent1']` / `['WorkflowPathAgent1']` / `['Agent1SourceWorksheetName']` / `['ExternalDomainsStorageFolder']` / `['ExternalDomainsFileName']` umgestellt. Zusätzlich im Zuge dessen entdeckt und behoben: 5 fehlerhaft dupliziert kopierte `StepName`/`Decision`/`KeyOutput`-Werte in den „NotFound“-Varianten der Buffer-Audit-Event-Kacheln (waren identisch zur `Failed`/`Succeeded`-Variante kopiert). Housekeeping (14 nun ungenutzte `VAR_*`-Deklarationen) durch Nutzer gelöscht und am 2026-08-07 final verifiziert (0 verwaiste Referenzen, erfolgreicher Testlauf).

### Finding A (neu, 2026-08-06): Audit-Datei/Tabelle in Agent 1 hart codiert
Parallel zum bereits dokumentierten Agent-2-Item-4: Die Kachel `Write_RunSummary_To_AuditTrail` (in Scope `Audit_Trail_Processing`) verwendet `file = 01UINNLKKBG24NTK66EVBYXRQJ57RVT7XD` und `table = {81828E1C-0910-4D64-AD3B-C3AA13BE95B9}` fest codiert, obwohl die Variablen `VAR_AuditTrailFileName` (`AuditTrail.xlsx`) und `VAR_AuditTableName` (`AuditTrail`) sowie die Config-Felder `AuditFileName`/`AuditTableName` bereits existieren. **Bewusst zurückgestellt** (gleiche Komplexität wie Agent-2-Item-4: Excel-Online-Connector benötigt vermutlich zusätzlich eine `Get file metadata using path`-Aktion, um aus einem dynamischen Pfad die technische Datei-ID zu ermitteln).

### Item 3-Bezug (aus dem projektweiten Optimierungs-Backlog): Mailbox-Ordner-Setup nicht cachen
Die 4 Mailbox-URI-Kacheln in Agent 1 (`Create_Mailbox_Subfolder_"PA_Processed_Mails"_1`, `Get_DMP_Mailbox_Parent_Folder_ID`, `Create_Mailbox_Subfolder_"Agent_1_Alerts"`, `Get_DMP_Mailbox_Subfolder_ID_for_"Agent_1_Alerts"`) laufen bei jedem Run neu, obwohl sich die Ordnerstruktur praktisch nie ändert — identisches Muster wie bei Agent 2 (siehe Item 3 weiter unten). **Bleibt bewusst offen**, da Lösung neue Config-Felder erfordern würde (nur nach Rücksprache mit Nutzer).

## Zusätzliches Finding (noch nicht umgesetzt): Tote/irrelevante Statusfelder in `UPDATE_StatusRow_Agent_01`
Agent 1 (reine Domains-Extraction, keine E-Mail-Pfad-Klassifizierung) schreibt aktuell hart codiert `0` in `item/EmailsProcessed`, `item/EmailsProcessed_DMP`, `item/EmailsProcessed_NoDMP`, `item/EmailsProcessed_DNES`, `item/EmailsProcessed_DEE`. Für Agent 1 sind diese Felder fachlich irrelevant (das sind Agent-2-Kennzahlen). Empfehlung: Alle `EmailsProcessed_*`-Felder aus `UPDATE_StatusRow_Agent_01` entfernen; nur `item/DomainsExtracted` (`@string(length(variables('DomainArray')))`) behalten.

---

# Agent 2 (E-Mail Inbox Treatment)

**Referenzdatei:** `Agent_02.json`
**Stand der letzten Prüfung:** 2026-08-06
**Status:** Produktiv im Einsatz, mehrere kritische Bugs in dieser Session gefunden und behoben (Config-Fallback-URIs, Delay-int()-Fehler, KPI-Feldvertauschung, SIMU_NODMP-Konfigurationsschlüssel, JSON-Aufbau-Fehler in `APPEND_ConfigJsonProperty`). Alle bestätigt funktionsfähig nach letztem Test (DNES erfolgreich, DIS nach Fixes erfolgreich).

## Item 1: Strict Mode – Config-Fallbacks entfernen + Variablen-Bereinigung ✅ ABGESCHLOSSEN (2026-08-11)

Alle Fallback-Muster (`coalesce(CMP_ConfigObject, variables(...))`) vollständig entfernt (0 verbleibende Treffer bei finalem Full-File-Scan), inkl. der nachträglich entdeckten Audit-Trail-Kategorie (Buffer_Audit_Event_* Kacheln, die veraltete `variables()`-Werte statt der tatsächlich verwendeten Config-Werte protokollierten – "Wahrheit und Klarheit"-Fund). Anschließend 18 verwaiste `VAR_*`-Deklarationen identifiziert und vom Nutzer gelöscht, verifiziert (0 verbleibende Referenzen). Verbleibende `variables(...)`-Nutzungen im Flow (RunPrefix, Detected Workflow Path, LastSentSubject, StepCounter, AuditOutcome, Current Workflow Counter, AuditEvents, RunStartTicks, CurrentOperationMode, AuditBuffer, StatusAgentKey, StatusListName) sind legitime Laufzeit-Variablen, keine Config-Schatten – kein weiterer Handlungsbedarf.

### Kontext (historisch)
Aktuell nutzen sehr viele Ausdrücke im Flow das Muster:
```
coalesce(outputs('CMP_ConfigObject')?['<Key>'], variables('<Fallback-Variable>'))
```
Das ist betrieblich sicher (Flow bricht nicht ab, wenn Config-Wert fehlt), aber **unsichtbar**: Wenn `CMP_ConfigObject` einen Wert nicht liefert, merkt man es nicht – der Fallback greift lautlos.

### Nutzerentscheidung
- Nutzer möchte NICHT dauerhaft mit stillem Fallback arbeiten.
- Nutzer möchte NICHT jetzt schon die alten Fallback-Variablen löschen (zu viele Änderungen auf einmal).
- Vereinbart: Erst wenn eine größere Anpassungsrunde ansteht:
  1. Fallbacks aus den Ausdrücken entfernen (nur noch `outputs('CMP_ConfigObject')?['<Key>']` verwenden, ggf. mit `trim(string(...))`).
  2. Danach separat prüfen, welche der alten `VAR_*`-Variablen dann wirklich ungenutzt sind, und erst dann löschen.

### Betroffene Muster (Beispiele, Stand der Prüfung – Liste ist nicht abschließend, vor Umsetzung neu durchsuchen)
- `variables('Shared DMP Mailbox')` als Fallback in allen Mailbox-URI-Ausdrücken
- `variables('Miscellaneous Data Folder')` / `variables('Counter File Name')` im Counter-Pfad
- `variables('DMP E-Mail Counter Table Name')`, `variables('DMP E-Mail Counter Table Column Name Path')`, `variables('DMP E-Mail Counter Table Column Name Counter')` in allen Get/Update-Counter-Kacheln
- `variables('Wait Seconds Before Sent Mail Search')` in allen `Delay_...`-Kacheln
- `variables('Shared DMP Mailbox - Processed E-Mails Folder')`, `variables('Shared DMP Mailbox - Processed E-Mails Folder (Agent 2 Alerts)')`
- `variables('External Domains Folder')`, `variables('External Domains File Name')`, `variables('Internal Domains Folder')`, `variables('Internal Domains File Name')`
- `variables('DMP CAMS Team E-Mail')`, `variables('DMP Communication Stream - Hot Line Team E-Mail')`, `variables('DMP Porting Team E-Mail')`

### Umsetzungsvorschlag (später)
1. Zuerst NUR Fallbacks entfernen (Config wird verpflichtend), Flow in allen 4 Betriebsmodi testen.
2. Erst nach nachgewiesener Stabilität: ungenutzte `VAR_*`-Variablen identifizieren (Nutzungszähler) und entfernen.
3. Empfehlung zur Sichtbarkeit: Falls ein Config-Wert fehlt, sollte das im Audit-Trail sichtbar werden (z. B. Warning-Event), nicht nur ein technischer Fehlerabbruch. Details bei Umsetzung gemeinsam festlegen.

---

## Item 2: Config-Ladeschleife – Laufzeit-Optimierung (größter Hebel) ✅ ABGESCHLOSSEN (2026-08-07)
`Select ConfigEntries` (Datenvorgang – Auswählen) + `CMP_ConfigJsonText`-Umstellung erfolgreich umgesetzt, `VAR_ConfigJsonBuffer`/`Apply_to_each_ConfigRow_BuildJson` entfernt. Vom Nutzer erfolgreich in allen 4 Betriebsmodi getestet und im JSON-Re-Export am 2026-08-07 13:11 Uhr verifiziert (exakte Übereinstimmung mit Spezifikation, inkl. `SIMU_NODMP`-Sicherheits-Fallback). Aus 212 sequenziellen Aktionen wurden 3.

### Gemessene Fakten (Stand der Prüfung 2026-08-06)
- Aktive Config-Zeilen in `DMP Command Configuration.csv`: **106**
- Aktionen pro Schleifendurchlauf in `Apply_to_each_ConfigRow_BuildJson`: **2** (`CMP_ConfigResolvedValue`, `APPEND_ConfigJsonProperty`)
- **Aktionen nur für den Config-Aufbau: 106 × 2 = 212**
- **Gesamtaktionen im kompletten Flow: 458**
- **Anteil der Config-Schleife am Gesamtflow: ≈ 46 %**
- Die Schleife läuft zwingend **sequenziell** (kein `runtimeConfiguration.concurrency` gesetzt), weil `AppendToStringVariable` bei paralleler Ausführung Daten verlieren würde.

### Betroffene Aktionen (aktuelle Namen, Stand dieser Prüfung)
- `VAR_ConfigJsonBuffer` (Init: `{"dummy":""`)
- `Apply_to_each_ConfigRow_BuildJson`
  - `foreach`: `@body('GET_DMP_Command_Configuration')?['value']`
  - nested: `CMP_ConfigResolvedValue` (Compose), `APPEND_ConfigJsonProperty` (AppendToStringVariable)
- `CMP_ConfigJsonText`: `@concat(variables('ConfigJsonBuffer'), '}')`
- `CMP_ConfigObject`: `@json(outputs('CMP_ConfigJsonText'))`

### Umsetzungsvorschlag (später, größere Änderung)
Ersetze die `Apply to each`-Schleife durch die Kombination **`Select` + `Join`** — **inzwischen bei Agent 1 als Pilot erfolgreich in Umsetzung** (siehe Agent-1-Abschnitt oben; die dort final validierten Ausdrücke/Vorgehensweise 1:1 auf Agent 2 übertragen, sobald Agent 1 vollständig abgeschlossen und getestet ist):

1. **Neue Aktion `Select_ConfigEntries`** (Select, Datenvorgänge, **Textmodus** verwenden – siehe KI-Arbeitsregel-Hinweis)
   - **From:** `@body('GET_DMP_Command_Configuration')?['value']`
   - **Map:** Text-Fragment `"Key":"Value"` pro Element, gleiche Escaping-Logik wie bisher.

2. **Ersatz für `CMP_ConfigJsonText`:**
   ```
   @concat('{"dummy":""', join(body('Select_ConfigEntries'), ''), '}')
   ```

3. `VAR_ConfigJsonBuffer` und `Apply_to_each_ConfigRow_BuildJson` entfallen komplett.

4. `CMP_ConfigObject` bleibt unverändert (`@json(outputs('CMP_ConfigJsonText'))`).

**Erwarteter Effekt:** aus 212 sequenziellen Aktionen werden ca. 3 Aktionen – bei identischem fachlichem Ergebnis.

**Wichtig bei Umsetzung:** Exakte Select-Ausdrücke (inkl. Escaping) gegen die bei Agent 1 bereits erfolgreich getesteten Ausdrücke abgleichen, da die Modus-Key-Namen in der Config teils kryptische interne Spaltennamen haben (z. B. `Value_x002d_SIMU_x0028_NODMP_x0029`), die zuvor bereits mehrfach Fehlerquelle waren.

---

## Item 3: Mailbox-Ordner-Setup nicht bei jedem Run neu ausführen

### Kontext
Folgende 4 Aktionen laufen aktuell bei **jeder einzelnen eingehenden E-Mail** neu, obwohl sich die Ordnerstruktur nach dem ersten erfolgreichen Lauf praktisch nie ändert:
- `Create_Mailbox_Subfolder_(PA_Processed_Mails)`
- `Get_DMP_Mailbox_Parent_Folder_ID`
- `Create_Mailbox_Subfolder_(Agent_2_Alerts)`
- `Get_DMP_MailboxSubfolder_ID_for_"Agent_2_Alerts"`

Das kostet ca. 2–6 Sekunden zusätzliche Laufzeit pro Mail ohne fachlichen Mehrwert (nach dem ersten Mal sind die Ordner bereits vorhanden). **Gleiches Muster besteht auch bei Agent 1** (dortige Alerts-/Processed-Mails-Ordner-Kacheln).

### Abwägung (Stand 2026-08-11)
**Vorteile:** Laufzeit-Ersparnis (~2-6 Sek./Mail, 4 Graph-API-Calls entfallen), weniger API-Last/Throttling-Risiko, entfernt einen aktuell absichtlich in Kauf genommenen erwarteten Fehlerzweig (Create bei bereits existierendem Ordner).
**Nachteile/Risiken:** Erfordert 2 neue Config-Felder (Pflegeaufwand); **Stale-Cache-Risiko** - wenn der Ordner in Outlook manuell umbenannt/verschoben/gelöscht wird, zeigt die gecachte ID ins Leere und Folgeaktionen schlagen unklar fehl, bis der Cache manuell zurückgesetzt wird (neues Betriebsrisiko, das es aktuell nicht gibt); zusätzliche Condition-Logik erhöht Flow-Komplexität; einmaliges manuelles Befüllen der Cache-Felder nach erstem Lauf nötig.

### Nutzerentscheidung (2026-08-11): Erweiterung des Umsetzungsvorschlags erforderlich
Das Stale-Cache-Risiko darf nicht stillschweigend auftreten. Bevor Item 3 umgesetzt wird, brauchen wir zusätzlich:
1. **Warn-E-Mail-Struktur:** Wenn eine gecachte Ordner-ID zur Laufzeit nicht mehr auflösbar ist (Move/Get-Aktion schlägt fehl, obwohl Cache-Feld gefüllt ist), muss eine verständliche Warnmeldung an den Anwender/die Hotline gehen, die die Ursache erklärt und die nötigen Schritte zur Aktualisierung der ID beschreibt.
2. **Frontend-Integration:** Diese Warnung/Aktualisierung soll idealerweise direkt im DMP COMMAND Power-App-Frontend abgebildet werden (z. B. sichtbarer Hinweis + Eingabefeld/Aktion zum Neu-Anstoßen der ID-Ermittlung), nicht nur als E-Mail.
3. **Cross-Agent-Harmonisierung (zwingend):** Da das identische Muster auch bei Agent 1 (und potenziell weiteren Agenten mit Mailbox-Ordner-Setup) besteht, MUSS der gewählte Mechanismus (Cache-Feld-Namensschema, Warnung, Frontend-Baustein, Reset-Prozess) für alle betroffenen Agenten einheitlich sein - keine abweichenden Prozesse pro Agent.

**Status:** Noch NICHT auf der unmittelbaren Prioritätenliste umgesetzt - Design/Scope muss zuerst um obige 3 Punkte erweitert werden, bevor konkrete Config-Felder/Kachel-Änderungen geliefert werden. Siehe auch Abschnitt "Priorisierung Gesamt-Backlog" weiter unten.

### Umsetzungsvorschlag (ursprünglich, jetzt nur Teilaspekt - siehe Nutzerentscheidung oben)
- Ordner-IDs einmalig ermitteln und in der zentralen Konfiguration cachen (z. B. neue Felder wie `ProcessedMailsFolderId`, `Agent2AlertsFolderId` in `DMP Command Configuration`).
- **Wichtig:** Neue Config-Felder nur nach ausdrücklicher Rücksprache mit dem Nutzer einführen.
- Alternative ohne neue Config-Felder: Ordner-Erstellung/-Suche nur ausführen, wenn eine nachgelagerte Move-Aktion fehlschlägt (bedingte Lazy-Erstellung).

---

## Item 4: Audit-Datei/Tabelle dynamisieren (aktuell noch hart codiert)

### Kontext (bestätigt bei Prüfung am 2026-08-06)
Folgende zwei Aktionen verwenden weiterhin harte SharePoint-interne IDs statt der zentralen Konfiguration:

- `Write_Audit_Trail_to_Excel`
  - `file` = `01UINNLKKBG24NTK66EVBYXRQJ57RVT7XD`
  - `table` = `{81828E1C-0910-4D64-AD3B-C3AA13BE95B9}`
- `Write_buffered_audit_events_to_Excel` (innerhalb `Check_whether_there_are_unwritten_buffered_events_` / `All_buffered_audit_events`)
  - `file` = `01UINNLKKBG24NTK66EVBYXRQJ57RVT7XD`
  - `table` = `{81828E1C-0910-4D64-AD3B-C3AA13BE95B9}`

Beide Aktionen haben inzwischen eine englische Beschreibung (bereits ergänzt), sind aber fachlich noch nicht dynamisiert.

Die Variablen `VAR_AuditTableName` (Wert: `AuditTrail`) und `VAR_AuditTrailFileName` (Wert: `AuditTrail.xlsx`) existieren bereits im Flow, werden aber aktuell **nicht** in diesen beiden Aktionen verwendet.

Die zentrale Konfiguration (`DMP Command Configuration.csv`) enthält bereits passende aktive Felder: `AuditFileName`, `AuditTableName`, `DMPSharePointRootFolder`.

### Umsetzungsvorschlag (später)
1. Dateipfad/-ID dynamisch aus Config auflösen:
   ```
   @concat(
     coalesce(outputs('CMP_ConfigObject')?['DMPSharePointRootFolder'], variables('Miscellaneous Data Folder')),
     '/',
     coalesce(outputs('CMP_ConfigObject')?['AuditFileName'], variables('AuditTrailFileName'))
   )
   ```
2. Tabellenfeld auf:
   ```
   @coalesce(outputs('CMP_ConfigObject')?['AuditTableName'], variables('Audit Table Name'))
   ```
3. **Wichtig:** Der Excel-Online-Connector (`shared_excelonlinebusiness`) referenziert Dateien oft über interne Datei-IDs (`file`-Parameter), nicht über Pfade – ggf. ist zusätzlich eine `Get file metadata using path`-Aktion nötig, um aus dem dynamischen Pfad die technische Datei-ID zu ermitteln.

### Status: ZURÜCKGESTELLT (Nutzerentscheidung 2026-08-11) - Nutzen rechtfertigt aktuell nicht das Risiko

**Warum wollten wir das umsetzen?** `Write_Audit_Trail_to_Excel` und `Write_buffered_audit_events_to_Excel` verwenden hartcodierte interne SharePoint-IDs statt der bereits existierenden zentralen Config-Felder `AuditFileName`/`AuditTableName`/`DMPSharePointRootFolder` - ein "Wahrheit und Klarheit"-Verstoß, da die Config hier keine tatsächliche Wirkung hat.

**Abwägung (dokumentiert wie im Chat erläutert):**

*Vorteile:* Bei künftiger Umbenennung/Verschiebung/Neuanlage der Audit-Datei würde eine reine Config-Änderung genügen statt manueller Flow-Bearbeitung; Konsistenz mit dem Rest des Systems (alle anderen Datei-/Ordner-Referenzen laufen bereits über Config); Einheitlichkeit zu Agent 1 (identisches offenes Finding "Finding A" dort).

*Nachteile/Risiken (ausschlaggebend für Zurückstellung):*
- Technisch komplexer als Item 3: Der Excel-Connector benötigt für `file` eine technische Datei-ID (Drive-Item-ID), keinen Pfad-String → erfordert eine **neue Aktion** (SharePoint "Dateimetadaten anhand des Pfads abrufen"), die zur Laufzeit **zusätzlich pro Schreibvorgang** aufgerufen werden müsste - das ist das Gegenteil von Performance-Gewinn, es fügt Laufzeit UND eine neue API-Abhängigkeit hinzu.
- Neue Fehlerquelle direkt in der kritischsten Fehlerkette: Schlägt die Pfad-Auflösung fehl (Datei umbenannt, Rechte-Problem), schlägt der Audit-Write komplett fehl - und genau diese Aktion löst den "Audit Write Failed"-Alarm aus. Ein Bug hier könnte Fehlalarme in der ohnehin sensiblen zentralen Fehlerbehandlung erzeugen.
- `table`-Parameter ließe sich vermutlich einfacher direkt auf `AuditTableName` umstellen (kein ID-Problem), aber ungetestet - Excel-Connector-Felder verhalten sich im Textmodus mit reinem Namen statt GUID teils überraschend.
- Die aktuelle hartcodierte Lösung ist stabil und ändert sich (anders als Mailbox-Ordner bei Item 3) praktisch nie - das Aufwand-Nutzen-Verhältnis ist ungünstig.

**Sekundäre Kopplung:** Identisches Finding ("Finding A") ist bereits im Agent-1-Abschnitt oben dokumentiert und ebenfalls offen/zurückgestellt - bei einer künftigen Umsetzung MUSS das gleiche Muster 1:1 auf Agent 1 übertragen werden, um keine Inkonsistenz zwischen beiden Agenten zu erzeugen.

**Entscheidung:** Bewusst zurückgestellt, bis sich das Nutzen/Risiko-Verhältnis ändert (z. B. wenn eine Datei-/Tabellen-Umbenennung tatsächlich ansteht). Kein aktueller Termin.

---

## Item 5: Irreführende Kachel-Bezeichnung `SimulationPrefix_Subject` / `SimulationPrefix_Body` (Grundprinzip „Wahrheit und Klarheit", neu ab 2026-08-10)

### Kontext
Beide Kacheln liefern den modusabhängigen Betreff-Präfix bzw. Body-Hinweistext für **alle** ausgehenden Agent-2-Mails, für **jeden** der 4 Betriebsmodi (nicht nur SIMU):
```
SimulationPrefix_Subject: @if( empty(coalesce(outputs('CMP_ConfigObject')?['SubjectPrefix'], '')), '', concat(outputs('CMP_ConfigObject')?['SubjectPrefix'], ' ') )
SimulationPrefix_Body:    @if( empty(coalesce(outputs('CMP_ConfigObject')?['MailModeText'], '')), '', concat(outputs('CMP_ConfigObject')?['MailModeText'], '\n\n') )
```
Die Logik selbst ist bereits korrekt und rein konfigurationsgetrieben (kein Sonderfall für „SIMU"). Der **Name** ist aber irreführend, da er suggeriert, es ginge nur um Simulationsläufe.

### Nutzerentscheidung (2026-08-10)
Neues Grundprinzip für alle Agenten: **„Wahrheit und Klarheit"** – widersprüchliche/unsachgemäße Bezeichnungen und Beschreibungen sind zu korrigieren, wo sinnvoll auch durch Vereinfachung/zentrale Konfiguration.

### Umsetzungsvorschlag
- Umbenennen zu **„Mail Mode Subject Prefix"** bzw. **„Mail Mode Body Header"**.
- **Wichtig:** Umbenennung MUSS über den Designer (Rechtsklick → Umbenennen) erfolgen, nicht per manueller JSON-Bearbeitung, da beide Kacheln vermutlich in sehr vielen anderen Aktionen per `outputs(...)` referenziert werden und der Designer alle Referenzen automatisch mit umbenennt.
- Beschreibungen bleiben unverändert (bereits sachlich korrekt).
- Status: Vorschlag geliefert, Umbenennung durch Nutzer noch nicht bestätigt/durchgeführt.

### Cross-Check für andere Agenten (2026-08-10)
- **Agent 1:** Geprüft – hat KEINE `SimulationPrefix_*`-Kacheln (0 Fundstellen). Nutzt stattdessen 7 Stellen mit direktem Inline-Verweis auf `outputs('CMP_ConfigObject')?['SubjectPrefix']`/`['MailModeText']` in den jeweiligen `Compose_Subject_(...)`/`Send_..._Email_(...)`-Kacheln. Andere Architektur, aber keine irreführende Bezeichnung vorhanden – kein Handlungsbedarf.
- **Agent 3.01/3.02/3.03:** Noch nicht prüfbar – JSON-Exports liegen noch nicht vor (nur .docx-Stand). **TODO:** Bei Erhalt der JSON-Exports auf dasselbe oder ähnliche irreführende Namensmuster prüfen (`SimulationPrefix_*` oder vergleichbar benannte Mode-Text-Kacheln).

---

## Zusammenfassung / Reihenfolge-Empfehlung Agent 2 (Stand 2026-08-11)

| # | Thema | Status | Aufwand | Nutzen |
|---|---|---|---|---|
| 1 | Strict Mode + Variablen-Bereinigung | ✅ Abgeschlossen | Niedrig bis Mittel | Mittel (Transparenz, Code-Hygiene) |
| 2 | Config-Ladeschleife (Select+Join) | ✅ Abgeschlossen | Mittel | Sehr hoch (Laufzeit) |
| 5 | Kachel-Umbenennung `SimulationPrefix_*` | ✅ Abgeschlossen | Niedrig | Niedrig-Mittel (Klarheit) |
| 3 | Mailbox-Ordner-Setup cachen | 🔲 Offen – Design erweitert um Warn-Mechanismus/Frontend/Cross-Agent-Harmonisierung (siehe oben) | Mittel-Hoch (jetzt inkl. Frontend-Baustein) | Mittel (Laufzeit) |
| 4 | Audit-Datei/Tabelle dynamisieren | ⏸️ Zurückgestellt (2026-08-11, Risiko > Nutzen aktuell) | Mittel (Excel-Connector-Besonderheit) | Mittel (Wartbarkeit) |

---

# Priorisierung Gesamt-Backlog (agentenübergreifend, Stand 2026-08-11, aktualisiert nach 2-Schalter-Architekturentscheidung)

**Kontext der Frage:** Nutzer möchte wissen, ob Item 3/4 überhaupt als nächstes dran wären, was sonst noch offen ist, und plant zusätzlich eine größere Weiterentwicklung des Power-App-Frontends (2 neue Schalter SIMU/PROD + Normal/DMP, siehe Architekturabschnitt oben). **Explizite Nutzervorgabe: Erst ALLE Agenten fertigstellen, danach ausschließlich noch GUI-Anpassungen.**

## Was ist bereits vollständig abgeschlossen
- **Agent 1:** Phase A, Phase B (Select+Join, Strict Mode), Findings A–D, Architekturumstellung (Trigger-Datei-Ablösung), VAR_*-Housekeeping – alles ✅ abgeschlossen. Bereits jetzt vollständig kompatibel mit der 2-Schalter-Zielarchitektur (nutzt ausschließlich `CurrentOperationMode`).
- **Agent 2:** Item 1 (Strict Mode), Item 2 (Select+Join), Item 5 (Umbenennung) – alle ✅ abgeschlossen. Ebenfalls bereits kompatibel.
- **Agent 3.01:** Vollständige Config-Migration ✅ abgeschlossen (2026-08-11), getestet (Happy Flow erfolgreich), final verifiziert. Siehe Agent-3.01-Abschnitt unten für Details.

## Was ist offen, geordnet nach tatsächlicher Priorität (nicht nach Item-Nummer)

**Update 2026-08-11 (2):** Nach Detailarbeit an Agent 3.02 wurde entdeckt, dass Teile davon (Audit-Heartbeat-Felder) totes Gerüst aus einem verworfenen Architekturansatz sind, der durch `DMP Command Agent Status` abgelöst wurde. **Nutzerentscheidung: Reihenfolge geändert** – nur noch Agent 3.01 wird jetzt auf Agent-1/2-Niveau gehoben; Agent 3.02 wird pausiert, bis ein gemeinsames Frontend-Brainstorming geklärt hat, ob der Agent eine neue Ausrichtung bekommt oder wie Agent 3.03 entfällt. Ziel des Frontends: möglichst Real-/Near-Time-Status zu Agent 1/2/3.01 (Gesundheit + Verarbeitungsstand: Anzahl, Ergebnisse, Warnungen, Fehler).

| Agent | Zentrale Config genutzt? | Mechanismus | Bekannte Findings | Status (2026-08-11) |
|---|---|---|---|---|
| 3.01 (Emergency Report) | ✅ Ja (nach Migration) | `CMP_ConfigObject` via Select+Join | Vollständig migriert, mehrere Bugs gefunden und behoben (StepStatus-Vertauschung, falsche Beschreibungen, Encoding) | ✅ **Abgeschlossen und getestet** |
| 3.02 (Status Check) | ✅ Ja, aber alte Foreach-Schleife | `variables('ConfigObject')` via `APPLY_TO_EACH_ConfigItem` | RealDMPIndicator-Rollback (Select+Join+Rollback-Fixes bereits ausgearbeitet, Anwendung offen) + totes Heartbeat-Gerüst entdeckt | ⏸️ **Pausiert bis Frontend-Brainstorming** |
| 3.03 (YES File Mgmt) | ✅ Ja, aber alte Foreach-Schleife | `variables('ConfigObject')` via `APPLY_TO_EACH_ConfigItem` | Nur Select+Join-Optimierung fällig, kein Rollback nötig | ⏸️ **Entfällt beim GUI-Umbau, keine weitere Arbeit** |

### Empfohlene Reihenfolge (aktualisiert 2026-08-11 (3))
1. **Agent 3.01 – ✅ abgeschlossen.** Keine weitere Arbeit nötig.
2. **Agent 3.02 – weiterhin pausiert**, offene Frage: bereits ausgearbeitete Fixes (Select+Join-Config-Ladevorgang, RealDMP-Rollback) jetzt noch anwenden oder komplett zurückstellen? Endgültige Ausrichtung erst nach Frontend-Brainstorming.
3. **Agent 3.03 – keine weitere Arbeit**, bleibt bis GUI-Cutover unverändert live (siehe Agent-3.03-Abschnitt unten).
4. **Frontend-Brainstorming (jetzt nächster Schritt) – gemeinsames, eigenständiges Vorhaben.** Ziel: DMP-COMMAND-Frontend mit möglichst Real-/Near-Time-Status zu Agent 1/2/3.01 (Gesundheit: läuft/Fehler/Warnungen; Verarbeitungsstand: Anzahl E-Mails, Ergebnisse). Bekannter Fundus für dieses Brainstorming: `DMP Command Agent Status` hat bereits eigene Zeilen für `Audit`, `Counter`, `EmergencyReport`, `ExternalDomains`, `InternalDomains`, `YesFile` zusätzlich zu den Agent-Zeilen – potenzielle Datenquelle für eine neue Agent-3.02-Ausrichtung statt Live-Datei-Checks. Ergebnis dieses Brainstormings entscheidet auch über Agent 3.02s endgültiges Schicksal (neue Aufgabe vs. Ablösung wie Agent 3.03) sowie über die 2-Schalter-GUI-Umsetzung und Item 3 bei Agent 2.

### Weitere offene Punkte (nachrangig zu oben)
- **Agent-2-Item 3 (isolierte Umsetzung ohne Frontend)** und **Item 4 (Audit-Datei/Tabelle)** – beide bewusst nicht vor dem Frontend-Brainstorming zu priorisieren: Item 4 ist zurückgestellt (Risiko > Nutzen), Item 3 hängt jetzt am Frontend-Brainstorming.
- **Dokumentations-Nachzug** (siehe Abschnitt „Documentation Maintenance" oben): `DMP_Multi_Agent_Workflow_Documentation.docx`, Agent1/Agent2-Workflow-HTMLs und `UAT_Playbook.docx` sind seit 2026-08-06 nicht mehr aktuell – spätestens jetzt (nach Abschluss von Agent 3.01) nachziehen, damit die Doku nicht noch weiter zurückfällt.


---

# Agent 3.01 (Emergency Report Management)

**Referenzdatei:** `JSON/Agent_03.01.json`
**Stand:** ✅ **VOLLSTÄNDIG ABGESCHLOSSEN (2026-08-11)** – komplette Config-Migration durchgeführt, getestet (Happy Flow erfolgreich) und final verifiziert (0 verbleibende alte Variable-Referenzen, 0 Coalesce-Fallbacks, 73 `CMP_ConfigObject`-Referenzen).

## Ursprünglicher Architektur-Befund (jetzt behoben)
Agent 3.01 nutzte ursprünglich kein zentrales Konfigurationsobjekt (0 Treffer `CMP_ConfigObject`) und las aus der zentralen Konfiguration ausschließlich `CurrentOperationMode`; alle anderen ~13 operativen Parameter (Mailbox-Adresse, Ordnernamen, Alert-Empfänger, Audit-Datei/-Tabelle, Wartezeit, Worksheet-Name, Rejected-/Work-Ordner, WorkflowPath) waren hartcodiert. **Alle passenden Config-Felder existierten bereits** (Scope `Agent3_01`), reine Verdrahtungsarbeit ohne neue SharePoint-Zeilen nötig.

## Durchgeführte Arbeiten
1. **Select+Join-Konfigurationsaufbau** (`Select ConfigEntries`, `CMP ConfigJsonText`, `CMP ConfigObject`) neu erstellt, direkt nach `SET OperationMode`.
2. **13 Config-Werte** vollständig auf `outputs('CMP_ConfigObject')?['Key']` (Strict Mode, kein Fallback) umgestellt: `Agent3AlertFolderName`, `RequiredWorksheetName`, `EmergencyReportTargetFolder`, `SharedDMPMailbox`, `WaitSecondsBeforeSentMailSearch`, `WorkFolderAgent3`, `RejectedFolderAgent3`, `WorkflowPathAgent301`, `EmergencyReportFileName`, `AlertEmailRecipient`, `ProcessedMailsRootFolderName`, `AuditFileName`, `AuditTableName` – über ~24 Kacheln in allen Fehlerzweigen (MissingWorksheet, InvalidWorkbook, InvalidExtension, WorkFile-Cleanup, Audit Failure, Status-Zeilen-Updates).
3. **`GET DMP Command Configuration`**: Filter (`Active eq 'Yes'`) und Top (`5000`) ergänzt (fehlten komplett, verursachten Performance-Warnung).
4. **2 Duplikat-Variablenpaare konsolidiert** (`ProcessedMailsRootFolderName`/`Shared DMP Mailbox - Processed E-Mails Folder`, `AgentAlertFolderName`/`...Agent 3 Alerts`-Variante).
5. **Bugfixes gefunden und behoben:**
   - `WRITE AuditEvent`: `item/StepStatus` las fälschlich `item()?['StepName']` statt `item()?['StepStatus']`.
   - Mehrere falsch kopierte Beschreibungstexte korrigiert (u. a. `AUDIT InvalidWorkbook` hatte die Beschreibung von `Get Sent Email By Subject`; `Get Sent Email By Subject (InvalidExtension)` hatte eine komplett fachfremde Beschreibung von einem anderen Agenten/Kontext).
   - `emailMessage/Importance` in allen 4 Alert-Mails von hartcodiert `"High"` auf `MailImportanceError` (Config) umgestellt (Konsistenz mit Agent 2).
   - Encoding-Probleme in allen 4 Mail-Bodies behoben (Gedankenstrich-Mojibake → einfacher Bindestrich, siehe neue KI-Arbeitsregel).
   - Lexical-Rich-Text-Formatierung in allen 4 Mail-Bodies korrigiert (Absatz-/Zeilenumbruch-Struktur, siehe neue KI-Arbeitsregel).
6. **VAR-Housekeeping:** Alle 14 überflüssigen `VAR_*`-Deklarationen gelöscht (inkl. 3 bereits zuvor toter Variablen `ProcessedMailsRootFolderName`, `AuditFileName`, `AuditTableName`).

**Zurückgestellt (bewusst, wie bei Agent 1 Finding A / Agent 2 Item 4):** `WRITE AuditEvent`/`AUDIT_*`-Kacheln nutzen weiterhin hartcodierte SharePoint-interne Datei-/Tabellen-IDs für `AuditTrail.xlsx` statt Config – gleiche Risiko-Abwägung wie bei den anderen beiden Agenten, kein akuter Handlungsbedarf.

---

# Agent 3.02 (Status Check)

**Referenzdatei:** `JSON/Agent_03.02.json` (72.896 Zeichen)
**Stand der Prüfung:** Detailliert analysiert am 2026-08-11 (erste inhaltliche Prüfung überhaupt).

## ⏸️ PAUSIERT (2026-08-11): Weiterarbeit gestoppt bis Frontend-Brainstorming abgeschlossen ist

**Kontext der Entscheidung:** Bei der Detailarbeit wurde entdeckt, dass 9 Statusvariablen (`AuditRunSummaryCount`, `AuditWarningCount`, `AuditFailedCount`, `AuditLastRunTimestamp`, `AuditLastSuccessTimestamp`, `AuditLastFailedTimestamp`, `AuditLastWarningTimestamp`, `AuditLastFailureTimestamp`, `AuditLastAuditEventTimestamp`) sowie 8 zugehörige `SET_*_From_Config`-Kacheln (`AuditTableName`, `AuditColumnStepName/StepStatus/TimestampUtc`, `AuditStepNameRunSummary`, `AuditStatusSucceeded/Warning/Failed`) **totes Gerüst** aus einem fr��heren, verworfenen Architekturansatz sind: Der ursprüngliche Plan war, den „Heartbeat" der Agenten per Live-Abfrage/Auszählung des Audit Trails direkt im Flow zu ermitteln – das erwies sich als zu langsam und führte zu Power-Apps-Timeouts. Deshalb wurde `DMP Command Agent Status` eingeführt, das Zwischenergebnisse **direkt aus den Flows heraus** (Agent 1/2/3.01 schreiben nach jedem Lauf) für das Frontend bereitstellt – die 9 Variablen blieben als nie fertiggestelltes/nie entferntes Gerüst zurück (immer beim Init-Default 0/leer, da nie tatsächlich befüllt).

**Nutzerentscheidung (2026-08-11):** Anstatt Agent 3.02 jetzt weiter zu vertiefen (Select+Join-Migration, Bereinigung der toten Felder, mögliche Neuausrichtung auf Reads aus `DMP Command Agent Status`), wird die Reihenfolge geändert:
1. **Nur Agent 3.01** wird jetzt noch auf das Architektur-Niveau von Agent 1/2 gehoben (vollständige Config-Migration, siehe Agent-3.01-Abschnitt unten) – Agent 3.02 NICHT mehr in der aktuellen Form weiter ausbauen.
2. Danach: **gemeinsames Frontend-Brainstorming** (neues, eigenständiges Vorhaben) mit dem Ziel, dass das DMP-COMMAND-Frontend einen möglichst **Real-/Near-Time-Status** über die „Gesundheit" von Agent 1/2/3.01 sowie den Verarbeitungsstand (Anzahl, Ergebnisse, Warnungen, Fehler) liefert.
3. **Erwartung:** Aus diesem Brainstorming werden sich vermutlich Konsequenzen für Agent 3.02 ergeben – entweder eine **neue, klar definierte Aufgabe/Ausrichtung** (z. B. reine Ressourcen-Verfügbarkeitsprüfung ohne die toten Heartbeat-Felder, oder ein Umbau auf Reads aus `DMP Command Agent Status` statt Live-Datei-Checks) oder eine **vollständige Ablösung analog Agent 3.03** (falls das neue Frontend-Konzept seine Funktion komplett anders/anderswo abbildet).

**Was bereits geliefert wurde (Stand vor der Pause) und zur Diskussion steht, ob es trotzdem angewendet wird:**
- Schritt 1+2 (Select+Join statt Foreach-Schleife für den Config-Ladevorgang) – reine Performance-/Konsistenz-Verbesserung, unabhängig vom künftigen Schicksal von Agent 3.02.
- Schritt 4+5 (RealDMP-Rollback: `IsRealDMP` aus `CurrentOperationMode` statt `Yes.txt`-Datei-Check ableiten, additives `currentoperationmode`-Statusfeld) – relevant, weil `Yes.txt`/Agent 3.03 ohnehin planmäßig abgelöst werden.
- Die im Chat identifizierten 27 `SET_*_From_Config`-Kacheln sowie deren mögliche Ablösung durch direkte `outputs('CMP_ConfigObject')`-Referenzen (Konsistenz-Diskussion) – **NICHT weiter vertieft**, da diese Arbeit ggf. durch die künftige Neuausrichtung ohnehin hinfällig wird.
- **Offene Frage an den Nutzer:** Sollen die bereits vollständig ausgearbeiteten Schritte 1, 2, 4, 5 trotzdem jetzt angewendet werden (kein Mehraufwand, da schon fertig spezifiziert), oder komplett zurückgestellt bis nach dem Frontend-Brainstorming?

**Zusätzliches, unabhängiges Finding (nur dokumentiert, nicht Teil der Pause):** `DMP Command Agent Status` hat bereits eigene Zeilen für `Audit`, `Counter`, `EmergencyReport`, `ExternalDomains`, `InternalDomains`, `YesFile` (zusätzlich zu den Agent-Zeilen) – das ist vermutlich die Datenquelle, die im Rahmen des Frontend-Brainstormings für eine mögliche Neuausrichtung von Agent 3.02 relevant wird (Reads aus dieser Liste statt Live-Datei-Checks).

## Finding 1 (bekannt, jetzt bestätigt): `IsRealDMP`/`RealDMPIndicator`-Anti-Pattern noch aktiv
**Bestätigt vorhanden** (3 Treffer `IsRealDMP`, 12 Treffer `RealDMPIndicator`). Der Flow ermittelt den DMP-Real-Status weiterhin über eine Datei-Existenzprüfung (`SET_RealDMPIndicatorFileName_From_Config` + Datei-Check), obwohl Agent 1 dieses Muster bereits am 2026-08-07 durch die direkte, robustere Ableitung aus `variables('OperationMode')` (`PROD_DMP`/`SIMU_DMP` → real DMP) ersetzt hat. **Muss analog zurückgebaut werden** – exakte Kachel-Liste erst bei Detailarbeit an diesem Agenten zu ermitteln (Scope jetzt bekannt, Feinanalyse noch offen).

## Finding 2: Konfiguration wird geladen, aber über die alte, ineffiziente Schleife
Anders als Agent 3.01 lädt Agent 3.02 tatsächlich ein vollständiges Konfigurationsobjekt (`variables('ConfigObject')`, 30 Treffer) – aber über `APPLY_TO_EACH_ConfigItem` (Foreach über alle 106 aktiven Config-Zeilen mit `setProperty`/`SetVariable` pro Durchlauf), **genau das Muster, das bei Agent 2 als "Item 2" identifiziert und durch Select+Join ersetzt wurde** (aus 212 auf ~3 Aktionen). Gleicher Optimierungshebel gilt hier 1:1.
**Zusätzlicher Konsistenz-Punkt:** Das Objekt heißt `variables('ConfigObject')` statt (wie bei Agent 1/2) `outputs('CMP_ConfigObject')` – funktional äquivalent, aber abweichende Namensgebung. Sollte im Zuge der Select+Join-Migration auf `CMP_ConfigObject` vereinheitlicht werden ("Wahrheit und Klarheit"/Konsistenz über alle Agenten).

## Weitere Findings (Erstsichtung, keine Detailprüfung)
- 0 Treffer für `SimulationPrefix` – kein Rename-Bedarf.
- 0 verbleibende `coalesce(CMP_ConfigObject, variables(...))`-Fallback-Muster (aber Achtung: da hier kein `CMP_ConfigObject` existiert, ist diese Metrik hier weniger aussagekräftig als bei Agent 1/2 – bei Detailarbeit eigene Fallback-Suche auf `variables('ConfigObject')`-Basis nötig).
- Sehr viele Status-spezifische Variablen (`AuditLastSuccessTimestamp`, `AuditTrailExists`, `CounterExists`, `ExternalDomainsExists`, `EmergencyReportExists`, `InternalDomainsExists` etc.) – fachlich plausibel für einen "Status Check"-Agenten, aber noch nicht im Detail auf Korrektheit/Config-Bezug geprüft.

## Umsetzungsvorschlag
1. Finding 1 (RealDMPIndicator-Rollback) zuerst – kleinerer, klar umrissener Fix, schließt eine seit Längerem bekannte Lücke.
2. Finding 2 (Select+Join-Migration) danach – größerer Umbau, aber bereits zweimal erfolgreich erprobtes Muster (Agent 1, Agent 2), geringes Risiko.
3. Im Zuge von Punkt 2 gleich auf `CMP_ConfigObject`-Namensgebung vereinheitlichen.
4. Danach: Vertiefte Prüfung der Status-Variablen und Audit-Kacheln auf Fallback-/Stale-Value-Muster analog Agent 2.

**Aufwand:** Mittel (beide Findings folgen bereits etablierten, erprobten Mustern). **Nutzen:** Hoch (schließt bekannte fachliche Inkonsistenz + spürbare Laufzeitverbesserung bei 106 Config-Zeilen).

---

# Agent 3.03 (Operational State Management) — vormals "YES File Management"

**Referenzdatei:** `JSON/Agent_03.03.json` (45.970 Zeichen – kleinster der drei 3.x-Flows)
**Stand der Prüfung:** Detailliert analysiert am 2026-08-11 (erste inhaltliche Prüfung überhaupt).

## ✅ FINALE ENTSCHEIDUNG (2026-08-12): Umbenennung + Yes.txt-Mechanismus komplett entfernt

Der Flow wird **nicht stillgelegt**, sondern umgebaut und umbenannt: `DMP Agent 3.03 (YES File Management)` → **`DMP Agent 3.03 (Operational State Management)`**. Der `Yes.txt`-Datei-Mechanismus (Create/Delete-Logik, Validierung auf erlaubte Werte, Indicator-Datei-Variablen) wird **vollständig ausgebaut** — er wird durch den neuen Zweck ersetzt: **direktes Schreiben von `CurrentOperationMode` in die SharePoint-Liste `DMP Command Configuration`**, aufgerufen von den 2 neuen Power-App-Schaltern (Operational Mode, Environment) im "Operating State"-Panel, da ein direkter `Patch()` aus der App selbst nicht verfügbar ist (kein SharePoint-Connector in der App, siehe unten).

**Neue Zielstruktur des Flows:**
- Trigger `PowerAppV2` bleibt (2 Textfelder), `text` = InitiatedBy (User-Email, unverändert), `text_1` = neuer gewünschter `CurrentOperationMode`-Wert (z. B. `PROD_DMP`) statt vormals `Create`/`Delete`.
- Entfernt: `VAR_RealDMPIndicatorFileName`, `VAR_RealDMPIndicatorFolder`, `VAR_RequestedActionCreate`, `VAR_RequestedActionDelete`, `SCOPE_Create_or_Delete_Yes_File` (inkl. `IF_Create_Requested` und aller `GET_YesFile_*`/`CREATE_YesFile`/`DELETE_YesFile`/`AUDIT_YesFile_*`-Aktionen), die gesamte Create/Delete-Validierung in `IF_Action_Is_Valid` sowie der komplette Invalid-Action-Zweig (`SET_AuditOutcome_(InvalidAction)`, `TERMINATE_(InvalidAction)`, `GET_StatusRow_Agent_3.03_(InvalidAction)`, `UPDATE_StatusRow_Agent_3.03_(Invalid_Action)`).
- Neu: `GET_ConfigRow_CurrentOperationMode` (SharePoint Get items, Liste `DMP Command Configuration` / GUID `c3e96ba6-1c07-4f39-9b4d-0d0a92db6d6a`, Filter `Title eq 'CurrentOperationMode'`, Top 1) → `UPDATE_ConfigRow_CurrentOperationMode` (SharePoint Update item, Feld `CurrentValue` = `variables('RequestedAction')`).
- `RESPOND_Result` bleibt (Antwort an die App), Body vereinfacht (kein `indicatorfilename`/`indicatorfolder` mehr).
- Audit-Grundgerüst (`RunStartTicks`, `AuditOutcome`, `AuditEvents`, `StepCounter`, `RunPrefix`, `WorkflowPath`, `VAR_AlertEmailRecipient`) bleibt erhalten für Konsistenz mit den anderen Agenten.

**Config-Zeilen, die dadurch verwaisen und aus der Live-SharePoint-Liste `DMP Command Configuration` entfernt werden können** (nicht in der lokalen CSV-Kopie geändert, da nutzergepflegter Export): `RequestedActionCreate`, `RequestedActionDelete`, `WorkflowPathAgent303` (Wert `Agent3_YesFileManagement` → müsste ohnehin auf `Agent3_OperationalStateManagement` aktualisiert werden, falls weiter genutzt), `YesFileCreateSuccessMessage`, `YesFileDeleteSuccessMessage`, `YesFileFailureSubject`.

**Umsetzung:** Manueller Umbau in Power Automate Designer durch den Nutzer (kein Datei-basierter Unpack/Pack-Workflow für einzelne Flows verfügbar, siehe ALM-Finding oben). Nach Fertigstellung: Power-App-Schalter `tglOperationalState`/`tglApplicationMode` von `Notify()`-Stubs auf echten `'DMPAgent3(OperationalStateManagement)'.Run(User().Email, <neuer Modus-String>)`-Aufruf umstellen.

**Status:** In Umsetzung (2026-08-12) – Designer-Umbau durch Nutzer läuft.

## ✅ ENTSCHIEDEN (2026-08-11): Agent wird durch neue GUI-Schalter abgelöst – ab sofort keine weitere Investition mehr

**Ursprüngliche Nutzer-Vermutung (bestätigt):** Da die Real-DMP-Steuerung inzwischen vollständig über die zentrale Konfiguration (`CurrentOperationMode`) laufen soll, ist Agent 3.03 (der nur die Datei `Yes.txt` erstellt/löscht) obsolet. **Nutzerentscheidung (2026-08-11):** Das Frontend bekommt 2 neue, direkt auf `CurrentOperationMode` schreibende Schalter (SIMU/PROD + Normal/DMP, siehe Architekturabschnitt ganz oben im Dokument) – Agent 3.03 entfällt dadurch vollständig. **Bis zum GUI-Cutover bleibt der Flow unverändert live bestehen** (keine Stilllegung vorab, um den aktuellen App-Schalter nicht komplett auszuknipsen), erhält aber **keine Migrations-/Optimierungsarbeit mehr** (kein Select+Join, siehe Finding 1 unten – das wäre verlorene Arbeit).

**Befund nach Cross-Check über alle Agenten (bestätigt die Vermutung weitgehend):**
- **Agent 1** (der einzige, der `Yes.txt` je zur Steuerung genutzt hat) wurde am 2026-08-07 bewusst umgebaut: `Check,_if_Real_DMP_File_is_available` wurde entfernt, `Check_Variable_-_Is_a_real_DMP` liest seither ausschließlich `variables('OperationMode')`. 0 verbleibende Treffer für `Yes.txt`/`RealDMPIndicator` in `Agent_01.json`.
- **Agent 2:** 0 Treffer für `Yes.txt`/`RealDMPIndicator` – hat die Datei nie genutzt.
- **Agent 3.01:** 0 Treffer – nutzt sie ebenfalls nicht.
- **Agent 3.02** ist der EINZIGE verbleibende Leser der Datei – prüft `Yes.txt`-Existenz und meldet das Ergebnis (`IsRealDMP`, `IsRealDMPLastModified`) als reinen **Status-/Monitoring-Wert an die Power App zurück** (`RESPOND_Status`). Es wird NICHT zur internen Ablaufsteuerung von Agent 3.02 selbst verwendet – es ist ein reiner Anzeige-/Dashboard-Wert, analog zu den anderen dort gemeldeten Ressourcen-Status (Emergency Report, Domains-Listen, Counter, Audit Trail).
- **Konsequenz:** Sobald das ohnehin schon geplante Agent-3.02-Rollback-Finding (siehe oben) umgesetzt wird – d. h. `IsRealDMP` künftig ebenfalls aus `CurrentOperationMode` abgeleitet statt per Datei-Check ermittelt wird – **liest nichts mehr im gesamten Agenten-Ökosystem die Datei `Yes.txt`**. Agent 3.03 würde dann nur noch eine Datei erstellen/löschen, die niemand mehr konsumiert.

**Eine offene Unbekannte — JETZT GEKLÄRT durch direkte Prüfung des Power-App-Quellcodes (2026-08-11):**
Ich habe `DMP_COMMAND_MASTER_PowerApps_CODE.docx` direkt entpackt und den Power-Fx-Quellcode durchsucht (technisch möglich, da .docx intern ein ZIP-Container ist). Ergebnis, **deutlich gravierender als ursprünglich vermutet**:

- Der einzige Schalter für „REAL DMP" vs. „FIRE DRILL" in der App (`tglDMPMode`/`cardYesFile`) ruft beim Bestätigen **ausschließlich** `'DMPAgent3(YESFileManagement)'.Run(User().Email, varPendingAction)` auf – also **nur** Agent 3.03 (Create/Delete `Yes.txt`).
- **0 Treffer** für `CurrentOperationMode`, `Patch(`, `PROD_NODMP`, `PROD_DMP`, `SIMU_NODMP`, `SIMU_DMP`, `ForAll(`, `CurrentValue` im **gesamten** Power-App-Quellcode. Es gibt **keinerlei Mechanismus in der App, der `CurrentOperationMode` in der zentralen Konfigurationsliste setzt.**
- Der Status-Wert `varIsRealDMP`, der den Schalter visuell synchronisiert (Farbe/Text „REAL DMP"/„FIRE DRILL"), kommt ausschließlich von `'DMPAgent3(StatusCheck)'.Run().isrealdmp` zurück – also wieder aus Agent 3.02s Datei-Existenz-Check, NICHT aus `CurrentOperationMode`.

**Das bedeutet: Es existieren aktuell zwei komplett unabhängige, nicht synchronisierte Steuerungs-/Statuspfade:**
1. `CurrentOperationMode` in der zentralen Konfiguration (von Agent 1 – und perspektivisch Agent 2/3.01 – als alleinige Quelle der Wahrheit verwendet) – wird von der Power App **an keiner Stelle geschrieben**. Muss also aktuell manuell direkt in der SharePoint-Liste gepflegt werden, oder der Schreibmechanismus liegt außerhalb der geprüften App-Datei.
2. Der `Yes.txt`-Datei-Schalter in der App (Agent 3.03 erstellt/löscht, Agent 3.02 liest zurück) – **beeinflusst Agent 1 nicht mehr**, seit dessen Umbau am 2026-08-07.

**🚨 Kritische, dringende Konsequenz:** Falls `CurrentOperationMode` NICHT anderweitig zuverlässig parallel gepflegt wird, hat der „REAL DMP / FIRE DRILL"-Schalter in der App aktuell **keine Wirkung mehr auf das tatsächliche Verhalten von Agent 1** – die Bedienperson glaubt, den Modus umzuschalten, tatsächlich passiert nichts an der Stelle, die zählt. Das ist potenziell schwerwiegender als die ursprüngliche Frage „brauchen wir Agent 3.03 noch" – es geht um die Frage, ob der zentrale Umschalter der gesamten Anwendung überhaupt noch funktioniert.

**Optional, nicht mehr blockierend:** Weiterhin sinnvoll, live zu prüfen, ob der aktuelle „REAL DMP"-Schalter in der App überhaupt noch Wirkung auf Agent 1 zeigt (Schalter umlegen, `CurrentOperationMode`-`CurrentValue` in SharePoint beobachten) – rein zur Einschätzung des Ist-Zustands in der Übergangszeit bis zum GUI-Cutover, ändert aber nichts mehr an der Zielplanung.

### Konkretes weiteres Vorgehen bis zum GUI-Cutover
1. Agent 3.03 unverändert lassen (kein Select+Join, keine sonstige Optimierung) – gilt als „fertig" im Sinne von „keine weitere Arbeit mehr nötig", da er ohnehin entfällt.
2. Agent 3.02: `isrealdmp`-Ableitung auf `CurrentOperationMode` umstellen (siehe Rollback-Finding oben) – das macht Agent 3.03 als Datenquelle für den Status überflüssig, unabhängig vom GUI-Zeitpunkt.
3. Beim eigentlichen GUI-Umbau (separates, nachgelagertes Vorhaben): Neue Schalter bauen (direktes `Patch(CurrentOperationMode)`), `cardYesFile`-Steuerelement entfernen, Agent 3.03 stilllegen, Config-Felder `RealDMPIndicatorFileName`, `RealDMPIndicatorFolder`, `YesFileFailureSubject` nach Rücksprache als verwaist entfernen.

**Status:** Entschieden – keine weitere Diskussion nötig, nur noch Umsetzung gemäß obiger Schritte zum jeweils richtigen Zeitpunkt (2 + 3 zeitlich getrennt: Schritt 2 jetzt, Schritt 3 erst beim GUI-Umbau).

## Finding 1: Gleiches Select+Join-Optimierungspotenzial wie Agent 3.02 – NICHT MEHR UMZUSETZEN
Auch hier wird die Konfiguration über `APPLY_TO_EACH_ConfigItem` (alte Foreach-Schleife über alle 106 Zeilen) in `variables('ConfigObject')` geladen (11 Treffer) – identisches Muster wie Agent 3.02 (siehe dort Finding 2). **Bewusst NICHT mehr umzusetzen** (siehe Entscheidung oben) – wäre verlorene Arbeit, da der Agent beim GUI-Umbau entfällt.

## Finding 2: `RealDMPIndicatorFileName`/`-Folder` – hier KEIN Anti-Pattern, sondern (auslaufende) Kernfunktion
`Real DMP Indicator File Name` (`Yes.txt`) und `Real DMP Indicator Folder` sind hier keine Kopie des bei Agent 1 entfernten Erkennungs-Mechanismus, sondern die **bisherige Kernaufgabe** dieses Agenten (Verwaltung der YES-Datei selbst: `RequestedActionCreate`/`RequestedActionDelete`) – **diese Kernaufgabe läuft planmäßig aus**, siehe Entscheidung oben.

## Weitere Findings (Erstsichtung, keine Detailprüfung)
- 0 Treffer für `SimulationPrefix` – kein Rename-Bedarf.
- 0 Treffer für `IsRealDMP` (im Unterschied zu Agent 3.02) – bestätigt, dass hier kein Datei-Check-Status-Anti-Pattern vorliegt, sondern nur die (jetzt zu hinterfragende) legitime Datei-Verwaltung.
- 17 verschiedene `variables(...)`-Namen insgesamt – kleinster Umfang der drei 3.x-Agenten, passend zur kompakteren fachlichen Aufgabe.

**Aufwand:** Gering (reines Unverändert-Lassen bis GUI-Cutover, dann einfaches Entfernen – kein Umbau, keine Migration). **Nutzen:** Reduziert Systemkomplexität, eliminiert eine potenzielle Zwei-Quellen-Inkonsistenz zwischen `Yes.txt` und `CurrentOperationMode`, spart die sonst nötige Select+Join-Migrationsarbeit vollständig ein.

---
---

# Power App (DMP COMMAND)

**Referenzdatei:** `DMP_COMMAND_MASTER_PowerApps_CODE.docx`
**Stand der Prüfung:** Am 2026-08-06 auszugsweise geprüft (Mapping-Vergleich für Agent-2-Statusfelder ND/DIS/DNES/DEE — Ergebnis: PowerApp-Logik ist konsistent mit den vier fachlichen Counter-Feldern `varCounterNoDMP`, `varCounterInternalSender`, `varCounterNotEffected`, `varCounterEffected`; keine Nutzung von `EmailsProcessed_DMP`/`_NoDMP`). Keine weiteren Findings bisher – **aber auch keine vollständige Prüfung**, siehe „Priorisierung Gesamt-Backlog" oben.
**Neu (2026-08-11):** Nutzer plant eine größere Weiterentwicklung des Frontends ("neues Level") sowie die Integration eines Warn-/Reset-Mechanismus für gecachte Ordner-IDs (Agent-2-Item-3, agentenübergreifend zu harmonisieren). Scope/Zeitpunkt noch nicht festgelegt – siehe Priorisierungsabschnitt oben.

---

# Zentrale Listen/Dokumente

## `DMP Command Configuration` (SharePoint-Liste, 106 aktive Zeilen)
Keine offenen Findings bisher außer den bereits oben je Agent genannten Zugriffsfehlern (Tabellenreferenz-Verwechslungen).

## `DMP Command Agent Status` (SharePoint-Liste)
Bestätigte Spaltenstruktur (Stand 2026-08-06): `Title, AgentDisplayName, CurrentStatus, LastUpdateTimestamp, LastRunTimestamp, LastSuccessTimestamp, LastRunResult, LastRunDurationSec, LastFailureTimestamp, LastFailureStep, LastFailureMessage, LastWarningTimestamp, LastWarningStep, LastWarningMessage, OperationMode, WorkflowPath, DomainsExtracted, EmailsProcessedTotal, EmailsProcessed_ND, EmailsProcessed_DIS, EmailsProcessed_DNES, EmailsProcessed_DEE, AgentKey, EmergencyReportPresent, WorkbookValidationPassed, CurrentIndicatorMode, LastRunId, LastStatusUpdateSource, StatusSeverity, LastHeartbeatTimestamp, StatusMessage`. Enthält 11 Zeilen (Agent_01, Agent_02, Agent_03_01, Agent_03_02, Agent_03_03, Audit, Counter, EmergencyReport, ExternalDomains, InternalDomains, YesFile). Wichtig: Es gibt **keine** Spalten `EmailsProcessed_DMP` oder `EmailsProcessed_NoDMP` — nur `_ND`, `_DIS`, `_DNES`, `_DEE` (siehe Findings bei Agent 1 und Agent 2 oben).
