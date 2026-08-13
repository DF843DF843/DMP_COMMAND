# DMP COMMAND – Projektweites Backlog für größere, zurückgestellte Optimierungen

**Geltungsbereich:** Dieses Backlog gilt für das gesamte DMP COMMAND System — alle Agenten (Agent 1, Agent 2, Agent 3, Agent 4, Agent 5 — durchnummeriert am 2026-08-13, vormals Agent 3.01/3.02/3.03), die Power App (DMP COMMAND), sowie alle verwendeten Konfigurations- und Statuslisten/-dokumente (`DMP Command Configuration`, `DMP Command Agent Status`, u. a.).

**Pflege-Regel (in KI-Arbeitsregeln verankert am 2026-08-06):**
Dieses Dokument wird regelmäßig aktualisiert, sobald bei der Arbeit an irgendeiner Komponente des Systems ein Finding entsteht, das bewusst auf später verschoben wird. Es ist NICHT auf einen einzelnen Agenten beschränkt.

**Wichtiger Arbeitshinweis (gilt für jeden Punkt):**
Vor Umsetzung IMMER zuerst den dann aktuellen Stand der jeweils betroffenen Datei(en) neu einlesen und gegen die hier beschriebenen Fundstellen prüfen (Feldnamen/Ausdrücke können sich durch zwischenzeitliche manuelle Änderungen verschoben haben). Keine neuen Config-Felder oder Variablen ohne Rücksprache mit dem Nutzer einführen.

---

# ✅ Agenten-Umnummerierung abgeschlossen (2026-08-13): Agent 3.01/3.02/3.03 → Agent 3/4/5

**Entscheidung des Nutzers (2026-08-13):** Durchgängige sequenzielle Nummerierung aller 5 Agenten statt der bisherigen Dezimalschreibweise, um unnötige Komplexität zu vermeiden. Zuordnung: Agent 1 bleibt 1, Agent 2 bleibt 2, **Agent 3.01 → Agent 3** (Emergency Report Management), **Agent 3.02 → Agent 4** (Status Check), **Agent 3.03 → Agent 5** (Operational State Management). Interne Schlüssel 2-stellig: `Agent_01`.."Agent_05" (AgentKey), Scope-Werte ohne Padding: `Agent1`.."Agent5" (passend zum bestehenden Muster von Agent1/Agent2).

## Durchgeführt (dateibasiert, via Power-Automate-Solution `DMP_COMMAND_Solution`)
- Flow-Anzeigenamen umbenannt (`.json.data.xml` Name/LocalizedName): "DMP Agent 3 (Emergency Report Management)", "DMP Agent 4 (Status Check)", "DMP Agent 5 (Operational State Management)".
- Interne Referenzen in allen 3 betroffenen Flow-JSONs umgestellt: `StatusAgentKey`-Werte (`Agent_03_01`→`Agent_03`, `Agent_03_02`→`Agent_04`, `Agent_03_03`→`Agent_05`), `WorkflowPathAgentNNN`-Konfigurationsschlüssel-Referenzen (`WorkflowPathAgent301`→`WorkflowPathAgent3`, `...302`→`...4`, `...303`→`...5`), Beschreibungstexte ("Agent 3.01"→"Agent 3" usw.), Scope-Filter in Agent 5 (`Scope eq 'Agent3_03'`→`Scope eq 'Agent5'`).
- Gepackt und erfolgreich importiert.
- Cross-Referenz-Check: Agent 1/2 referenzieren keine der umbenannten Agenten – keine weiteren Anpassungen dort nötig. Agent 3/4 nutzen Title-basierte statt Scope-basierte Config-Filter – dort keine Scope-Anpassung nötig.

## ⚠️ Noch OFFEN – manuell in SharePoint nachzuziehen (konnte nicht automatisiert werden, kein Live-Zugriff)
Folgende Änderungen an den **live** SharePoint-Listen wurden bewusst NICHT automatisch vorgenommen (kein direkter Schreibzugriff verfügbar) und müssen vom Nutzer nachgezogen werden:

**In `DMP Command Configuration`:**
1. Zeile `WorkflowPathAgent301` → `ParameterName` umbenennen zu `WorkflowPathAgent3`, `Scope` von `Agent3_01` zu `Agent3`.
2. Zeile `WorkflowPathAgent302` → `ParameterName` zu `WorkflowPathAgent4`, `Scope` von `Agent3_02` zu `Agent4`.
3. Zeile `WorkflowPathAgent303` → `ParameterName` zu `WorkflowPathAgent5`, `Scope` von `Agent3_03` zu `Agent5`, `CurrentValue`/alle 4 Modus-Spalten von `Agent3_YesFileManagement`/`Agent3_OperationalStateManagement` zu `Agent5_OperationalStateManagement`.
4. Alle übrigen Zeilen mit `Scope = Agent3_01` → `Agent3`; `Scope = Agent3_02` → `Agent4`; `Scope = Agent3_03` → `Agent5` (Zeilen vorher in der Liste filtern/identifizieren).
5. **Offene Design-Frage:** Zeile `Agent3AlertFolderName` nutzt `Scope = Agent3_All` (geteilt zwischen den ehemaligen 3.01/3.02/3.03-Unteragenten). Da diese jetzt vollständig unabhängige Agenten 3/4/5 sind, ergibt "Agent3_All" konzeptionell keinen Sinn mehr – **bewusst unverändert gelassen**, da Agent 3 und Agent 4 ohnehin titelbasiert filtern (nicht scope-basiert) und Agent 5 den Wert `Agent3_All` weiterhin explizit in seinem Filter erwartet (unverändert gelassen). Empfehlung: Bei nächster Gelegenheit gemeinsam entscheiden, ob umbenannt (z. B. zu `Global` oder `SharedAgent345`) oder bewusst so belassen.

**In `DMP Command Agent Status`:**
6. Zeile mit `AgentKey = Agent_03_01` → `Agent_03`, `AgentDisplayName` zu "Emergency Report Management".
7. Zeile mit `AgentKey = Agent_03_02` → `Agent_04`, `AgentDisplayName` zu "Status Check".
8. Zeile mit `AgentKey = Agent_03_03` → `Agent_05`, `AgentDisplayName` bereits "Operational State Management" (unverändert).

**Grund für die Verzögerung:** Direkter Browser-Zugriff auf SharePoint erforderte eine interaktive Anmeldung (SSO/MFA), die aus Sicherheitsgründen nicht durch die KI selbst durchgeführt werden darf.



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

**Regel (in KI-Arbeitsregeln verankert am 2026-08-06):** Die folgenden drei Dokumente dienen als Anwenderhandbuch für DMP COMMAND, müssen durchgehend auf Englisch verfasst sein und sind bei jeder größeren fachlichen/technischen Änderung am System nachzuführen. Der Assistent soll proaktiv auf Aktualisierungsbedarf hinweisen, auch ohne explizite Nachfrage.

**Betroffene Dokumente:**
- `DMP_Multi_Agent_Workflow_Documentation.docx`
- `Agent1_High_Level_Workflow.html`, `Agent1_Detailed_Workflow.html`
- `Agent2_High_Level_Workflow.html`, `Agent2_Detailed_Workflow.html`
- `UAT_Playbook.docx` (Pfad: `AI_Agent\UAT\UAT_Playbook.docx`) — zusätzlich um neue Testfälle zu erweitern, nicht nur zu aktualisieren.

**Aktueller Rückstand (Stand 2026-08-06 — noch nicht nachgezogen):**
- `DMP_Multi_Agent_Workflow_Documentation.docx` (zuletzt geändert 07.07.2026) und die Agent1/Agent2-Workflow-HTML-Dateien (zuletzt geändert 02.07.2026) spiegeln **nicht** die in dieser Session vorgenommenen Änderungen wider:
  - Agent 2: Bugfixes an Mailbox-URI-Aufbau, Delay-Sekunden-Konvertierung, Status-KPI-Feldern (ND/DIS/DNES/DEE), SIMU_NODMP-Konfigurationsschlüssel, JSON-Konfigurationsaufbau (`APPEND_ConfigJsonProperty`).
  - Agent 1: Phase A (Korrektur der Konfigurationstabellen-Referenz in `GET_DMP_Command_Configuration`) sowie der begonnene Phase-B-Umbau (neues zentrales `CMP_ConfigObject` via `Select`+`Join`, umbenannte Kachel `FILTER_Config_Row_CurrentOperationMode`).
- `UAT_Playbook.docx` (zuletzt geändert 11.06.2026) enthält voraussichtlich noch keine Testfälle für die oben genannten, in dieser Session gefundenen und behobenen Fehlerbilder (z. B. Graph-400 bei Mailbox-Ordner-URIs, `int()`-Konvertierungsfehler bei Delay-Kacheln, KPI-Feldvertauschung). Empfehlung: Bei nächster größerer Anpassungsrunde entsprechende Testfälle ergänzen.

**Umsetzungsvorschlag:** Dokumentations-Update als eigener Punkt in die nächste größere Anpassungsrunde aufnehmen (z. B. direkt im Anschluss an den Abschluss von Agent-1-Phase-B), damit Doku und Code nicht weiter auseinanderlaufen.

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
