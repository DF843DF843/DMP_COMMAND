# DMP COMMAND – Projektweites Backlog für größere, zurückgestellte Optimierungen

**Geltungsbereich:** Dieses Backlog gilt für das gesamte DMP COMMAND System — alle Agenten (Agent 1–6), die Power App (DMP COMMAND), sowie alle verwendeten Konfigurations- und Statuslisten/-dokumente (`DMP Command Configuration`, `DMP Command Agent Status`, u. a.).

**Pflege-Regel (in KI-Arbeitsregeln verankert am 2026-08-06):** Dieses Dokument wird regelmäßig aktualisiert, sobald bei der Arbeit an irgendeiner Komponente des Systems ein Finding entsteht, das bewusst auf später verschoben wird. Es ist NICHT auf einen einzelnen Agenten beschränkt.

**Archivierungs-Regel (ergänzt am 2026-09-04, siehe KI-Arbeitsregeln „Pflicht-Checkliste" Abschnitt G):** Vollständig erledigte, bestätigt live ausgelieferte Punkte werden regelmäßig aus diesem aktiven Dokument nach [`DMP_COMMAND_Backlog_Archive.md`](./DMP_COMMAND_Backlog_Archive.md) ausgelagert (nicht gelöscht). Der fachliche Kern jedes erledigten Punkts wird vorher in `DMP_COMMAND_Release_Notes.md` (versions-relevante Fixes/Features) bzw. `DMP_COMMAND_Operations_Manual.md` (betrieblich relevante Punkte) gesichert.

**Wichtiger Arbeitshinweis (gilt für jeden Punkt):** Vor Umsetzung IMMER zuerst den dann aktuellen Stand der jeweils betroffenen Datei(en) neu einlesen und gegen die hier beschriebenen Fundstellen prüfen (Feldnamen/Ausdrücke können sich durch zwischenzeitliche manuelle Änderungen verschoben haben). Keine neuen Config-Felder oder Variablen ohne Rücksprache mit dem Nutzer einführen.

---

## 🟠 v1.22.34 (2026-09-23, lokal gepackt) — v1.22.33-Fix: verschachteltes AddColumns durch materialisierte Zwischen-Collections ersetzt

Nutzer hat `v1.22.33` in Studio geladen — **108 App-Checker-Fehler** (5 rote Badges): "Die Funktion „AddColumns" weist ungültige Argumente auf" / "Der Name ist ungültig. „OverallProcessMatch"/„IsDoneCalc" wird nicht erkannt", außerdem Vermutung, ein neuer Timer lasse die blauen KPI-Change-LEDs (Critical/Warnings/Agents Active) blinken.

1. **Root Cause:** `AddColumns` kann in Power Fx/Studio NUR auf die Spalten der ursprünglichen Quelltabelle zugreifen — NIE auf eine Spalte, die eine verschachtelte/vorgelagerte `AddColumns`-Aufruf als `Source`-Argument neu hinzugefügt hat, selbst nur eine Ebene tief nicht. Der 3-fach verschachtelte `AddColumns(AddColumns(AddColumns(...)))`-Aufruf aus v1.22.33 kompilierte deshalb nicht; alle nachgelagerten Verweise auf die berechneten Spalten (Gallery-Template) schlugen kaskadierend fehl.
2. **Fix:** Formel auf 3 echte Zwischen-Collections umgebaut (`colCosLeaderStage1` → `colCosLeaderStage2` → `colCosLeaderNextSteps`), jede Stufe per eigenem `ClearCollect` materialisiert, bevor die nächste Stufe per `AddColumns` darauf zugreift — identisch in `OnVisible` und `tmrCosLeaderNextStepsRefresh` dupliziert.
3. **Separater Fund (nicht durch diese Sitzung verursacht):** Die Live-Liste `DMP Command External Domains` hat aktuell **kein** `Active`-Feld mehr (nur Standard-SharePoint-Metadaten-Spalten) — verifiziert direkt in der aktualisierten `.msapr`/`DataSources.json` (GUID-genau, nicht über die verwirrenden `_1/_2/_3`-Duplikat-Einträge). Das erklärt die 3 separaten `scrConfiguration`-Fehler zu `lblConfigTileExternalDomainsCount`. Wurde erst durch den Schema-Refresh dieser Sitzung sichtbar (vorher lief die App mit einem veralteten Schema-Cache). **Nicht selbst behoben** — Agent 1 legt diese Liste bei jeder Emergency-Report-Extraktion komplett neu an; zu prüfen, ob dieser Prozess das `Active`-Feld verliert. Nutzer um Prüfung gebeten.
4. **Blinkende KPI-LEDs:** Ursache nicht bestätigt — vermutlich Nebeneffekt des defekten Formel-Zustands in v1.22.33 (Studio verhält sich bei Kompilierfehlern teils unvorhersehbar), nicht direkt durch den neuen Timer verursacht (dieser setzt `varBlinkPhase` nirgends). Nutzer nach dem erneuten Laden bitten, zu prüfen, ob das Verhalten mit v1.22.34 verschwunden ist.
5. Kontrollen vor Auslieferung: App-weite Control-Namens-Eindeutigkeit (866 Namen, 0 Duplikate), `": "`-Regex-Scan (0 Treffer), Pack→Unpack-Rückvergleich (0 Diff auf `scrHome`, `scrReleaseNotes`, `App`).
6. `PowerApp_Version.txt` auf `v1.22.34` aktualisiert. Regel 9b: Backup bleibt bei `DMP_COMMAND_v1.22.32.msapp` (weder v1.22.33 noch v1.22.34 vom Nutzer bestätigt ladend). Solution unverändert bei `7.34.67`.

---

## 🟢 v1.22.33 (2026-09-23, lokal gepackt) — Next Steps Phase 1: Cockpit-Panel jetzt live auf Basis der CoS-Leader-Checkliste

Direkte Fortsetzung nach v1.22.32-Bestätigung ("Jetzt mit Next Steps weitermachen"). Umfangreicher Abstimmungsprozess mit dem Nutzer (siehe Punkt darunter unter Priorität 2 Punkt 1) zu Datenmodell-Lücken und Architekturfragen, dann konkrete Umsetzung:

1. **Datenmodell-Lücken geschlossen:** Nutzer hat 3 neue Spalten auf `DMP Command Checklist Overall Process` angelegt (`SeqNo` Zahl, `PredecessorTaskIds` Text, `IsMilestone` Auswahl Yes/No) und die Datenquelle in Studio aktualisiert+veröffentlicht. KI hat die veröffentlichte App heruntergeladen (`pac canvas download` + unpack), Identität verifiziert (bekannte v1.22.32-Marker), den Src-Diff als reine Studio-Reformatierung bestätigt (keine Inhaltsänderung) und **nur** die aktualisierte `.msapr`-Datei ins Repo übernommen.
2. **Scope-Entscheidung:** Infrastructure Team/Content Team-Checklisten existieren noch nicht (kein Schreibzugriff auf die Team-Sites) — Nutzer hat sich entschieden, Next Steps vorerst NUR für den CoS-Leader-Substream zu bauen, Infrastructure/Content folgen später.
3. **Architekturentscheidung Task-Status-Quelle:** Die bestehende Seite "Task Occurrences" ist explizit nur für wiederkehrende Aufgaben gedacht (von Agent 7 über Checklist Recurrence Rules gespeist) — die meisten CoS-Leader-Aufgaben sind aber einmalig und hätten nie eine Occurrence. Nutzer hat sich für Phase 1 entschieden: Live-Status direkt aus der CoS-Leader-Checkliste lesen (funktioniert sofort für alle Aufgaben); ein Agent/Job, der automatisch pro DMP-Fall eine Task Occurrence je Aufgabe anlegt, ist als eigener, späterer Schritt vorgemerkt.
4. **Cockpit-Umsetzung:** Der alte statische "NEXT STEPS"-Container (`varNextMilestones`, aus Agent 4) wurde durch eine live berechnete Ansicht ersetzt: `ClearCollect` mit verschachtelten `AddColumns`/`LookUp`/`Filter(Split(...))` verknüpft jede CoS-Leader-Aufgabe mit ihrer Overall-Process-Zeile (per `Titel`), berechnet Phasen-Erreichbarkeit (`varOperationalStepCounter` vs. Aufgaben-`Phase`) und Vorgänger-Erfüllung, zeigt bis zu 5 zuletzt abgeschlossene + 8 offene Aufgaben (Ongoing zuerst, dann bereit, dann nicht bereit) mit Punkt+Text je Status (COMPLETED/ONGOING/PENDING/NOT STARTED). Neuer leichter Timer `tmrCosLeaderNextStepsRefresh` (30s) hält die Ansicht aktuell, unabhängig vom Agent-4-Refresh-Zyklus.
5. **Bewusste Phase-1-Grenzen (kein Bug):** kein Rot/Overdue-Zustand (Checkliste hat kein Fälligkeitsdatum-Feld), keine In-App-Freigabe-Aktion (Statusänderungen weiterhin nur in SharePoint), Vorgänger müssen aktuell auf derselben CoS-Leader-Checkliste liegen.
6. **Encoding-Stolperfalle gefunden und behoben:** Ein PowerShell-Skript zur Entfernung verwaister YAML-Zeilen hat beim Neuschreiben der gesamten Datei Sonderzeichen (`·`, `★`) durch falsches Encoding beschädigt (Mojibake, gleiches Muster wie die UTF-16-Falle vom Anfang dieser Sitzung). Über gezieltes `ReadAllText`/`WriteAllText` mit explizitem UTF-8 behoben, danach per Pack→Unpack-Diff verifiziert (0 Diff).
7. Kontrollen vor Auslieferung: App-weite Control-Namens-Eindeutigkeit (866 Namen, 0 Duplikate), `": "`-Regex-Scan (0 Treffer), Pack→Unpack-Rückvergleich (0 Diff auf `scrHome`, `scrReleaseNotes`, `App`). **Erste Verwendung von verschachteltem `AddColumns`+`LookUp`+`Filter(Split(...))` in dieser App — noch nicht Studio-validiert, Nutzer bitten, nach dem Laden auf rote Fehler-Badges am Cockpit zu achten.**
8. `PowerApp_Version.txt` auf `v1.22.33` aktualisiert. Regel 9b: Backup bleibt bei `DMP_COMMAND_v1.22.32.msapp` (v1.22.33 ist noch nicht vom Nutzer bestätigt ladend). Solution unverändert bei `7.34.67`, keine Power-Automate-Änderungen diese Sitzung.

---

## 🟢 v1.22.32 (2026-09-23, vom Nutzer bestätigt ladend) — System Health/Configuration nach v1.22.31-Feedback erneut überarbeitet (Connection Diagnostics zusammengeführt, Kachel-Grid, Header-Bars)

**Update (2026-09-23, gleiche Sitzung):** Nutzer hat `v1.22.32` erfolgreich in Studio geladen. System Health (Details) OK, Maintenance OK, Configuration (Lists) "erst mal ok, noch nicht optimal" (siehe Priorität 3 für 2 kosmetische Nachmeldungen: abgeschnittener Kartentitel + Container-Platzierung). Regel 9b: Backup auf `DMP_COMMAND_v1.22.32.msapp` rotiert (ersetzt `v1.22.31`).

Nutzer hat `v1.22.31` in Studio geladen (Feedback zu UX, kein Ladefehler) und 3 Anmerkungen gegeben:

1. **System Health - Details: Connection Diagnostics fachlich integriert + horizontales Grid.** Nutzer: "Diese Übersicht ist so nicht brauchbar. Vergleich mit Maintenance - Connection diagnostic! So ähnlich könnte das aussehen. Zudem gehört Connection diagnostic fachlich zu System Health! Auch hier: Es können 2-4 Bereiche horizontal nebeneinander." Nach Rückfrage (per `ask_user`) hat der Nutzer sich für **komplette Verschiebung** entschieden (nicht Duplizierung): Der komplette `conMaintenanceDiagnostics`-Block wurde aus `scrMaintenance.pa.yaml` entfernt und als neuer, auf alle 17 Listen erweiterter Bereich in `scrSystemHealthDetails.pa.yaml` neu aufgebaut (vorher nur 3 Listen: Configuration, Agent Status, Internal Domains). Layout von einer langen vertikalen Kartenkette auf 3 horizontale Zeilen zu je 2 Karten umgebaut (`conSystemHealthRow1/2/3`, `ManualLayout` mit `X`/`Width`-Formeln `=(Parent.Width-16)/2` statt AutoLayout, um die in v1.22.30/31 gemachten Erfahrungen mit AutoLayout-Größenproblemen nicht zu wiederholen): Row1 = Core Status | Data Sources & Files, Row2 = Agents (1-7) | Connection Diagnostics - Core & Domain Lists (9 Listen), Row3 = Connection Diagnostics - Streams Lists (8 Listen) | Hinweis-Karte ("About the dots above"). Alle 16 bestehenden Status-Formeln unverändert übernommen; die 17 neuen Connection-Diagnostics-Zeilen nutzen dieselbe `IfError(CountRows(...)>=0,...)`-Logik wie zuvor in Configuration/Maintenance.
2. **Configuration (Lists): Header-Bars + Kachel-Grid statt Zeilen.** Nutzer: "Die Abschnittsüberschriften heben sich nicht hervor. Die Kacheln pro Liste sind viel zu groß. Jeden Abschnitt in einen eigenen Container. Pro Container die Kachel gruppieren, Horizontal maximal 4 nebeneinander." Jede der 6 Abschnittskarten hat jetzt eine eigene farbige Kopfleiste (`HeaderBar`, dunkelviolett `RGBA(32,23,81,1)`, weißer fetter Text) statt nur farbigem Text auf transparentem Hintergrund. Die bisherigen vollbreiten Einzeilen-pro-Liste wurden durch ein Kachel-Raster ersetzt (max. 4 Kacheln nebeneinander, `Width: =(Parent.Width-68)/4`); da jeder der 6 Abschnitte ohnehin höchstens 4 Listen enthält, passt jeder Abschnitt jetzt in genau eine Kachelzeile. Alle 17 Listennamen/Zähl-Formeln/View-/New-Entry-URLs/Tooltips 1:1 aus der v1.22.31-Datei extrahiert (PowerShell-Parser, kein manuelles Abtippen) und unverändert übernommen — nur Layout geändert. Kartenhöhe jetzt einheitlich 140px pro Abschnitt (vorher 110-170px je nach Listenzahl).
3. **Maintenance: Connection Diagnostics entfernt (siehe Punkt 1).** Seite zeigt jetzt nur noch Versions + Admin-Portal-Links.
4. Kontrollen vor Auslieferung: App-weite Control-Namens-Eindeutigkeit (874 Namen, 0 Duplikate), Regex-Scan auf literales `": "` in einzeiligen `Text:`/`Tooltip:`-Formeln (0 Treffer in allen neuen/geänderten Zeilen), Pack→Unpack-Rückvergleich (0 Diff auf allen 6 geänderten Dateien: `scrSystemHealthDetails`, `scrConfiguration`, `scrMaintenance`, `scrReleaseNotes`, `scrOperationalBoard`, `App`), Kontrollzahlen vor/nach (System Health: 16→33 Punkte, 21→42 Labels, 3→6 Abschnittskarten; Configuration: 17/17/17/17/13 Dots/Namen/Counts/View/New unverändert, 6 neue HeaderBars).
5. `PowerApp_Version.txt` auf `v1.22.32` aktualisiert. Regel 9b: Backup auf `DMP_COMMAND_v1.22.31.msapp` rotiert (v1.22.31 gilt als vom Nutzer bestätigt ladend — die Anmerkungen betrafen nur UX, kein Ladefehler).
6. Solution unverändert bei `7.34.67`, keine Power-Automate-Änderungen diese Sitzung.
7. **Separat vom Nutzer geliefert, noch nicht umgesetzt:** eine vollständige 21-Abschnitte-Spezifikation für das "Next Steps"/CoS-Leader-Feature (`DMP_COMMAND_Next_Steps_Anforderung.md`, ersetzt die 8 offenen Klärungsfragen unter Priorität 2 Punkt 1). Gegen das echte Datenmodell geprüft (siehe Punkt darunter) — Umsetzung nächste Sitzung.

---

## 🟢 v1.22.31 (2026-09-23, lokal gepackt) — 2 von 3 v1.22.30-Findings mit Redesign behoben, 1 Finding braucht Brainstorming

Nutzer hat `v1.22.30` in Studio geladen (lädt fehlerfrei) und 3 Anmerkungen zu den v1.22.30-Fixes gegeben, während er 2h in einem Meeting war ("baue soviel wie möglich weiter, wo keine Interaktion nötig ist"):

1. **System Health - Details: komplett neu gebaut, jetzt kompakt.** Die v1.22.30-Version nutzte `AutoLayout`-Karten mit 16px-Punkten/13px-Text, die viel zu groß und teils abgeschnitten rendern ("viel zu groß und unlesbar"). Umgebaut auf dasselbe kompakte Festlayout-Tabellen-Design wie der Cockpit-eigene "Automation Status"-Container (14px-Punkte, 11px-Text, 26px-Zeilenabstand, Breite `=Parent.Width` statt AutoLayout-Stretch). Alle 16 Status-Formeln 1:1 unverändert übernommen (nur Layout/Größe geändert, keine Logikänderung) — verifiziert per Vorher/Nachher-Kontrollzahlen (16 Punkte, 19 Labels inkl. 3 Section-Header, 3 Sections, unverändert).
2. **Configuration (Lists): komplett neu gebaut, jetzt gruppiert und kompakt.** Die 17 Einzelkacheln (je 130px, ~2450px Gesamthöhe) waren "viel zu groß und unübersichtlich". Umgebaut auf 6 umrandete Abschnittskarten (eine je Thema) im Stil des Cockpit-eigenen "Maintenance - Domains"-Containers; jede Karte enthält jetzt eine kompakte Zeile je Liste (Erreichbarkeits-Punkt via `IfError(CountRows(...)>=0,...)`, Name, Live-Zeilenzahl, View-/New-Entry-Buttons; Beschreibungstext in das Tooltip des View-Buttons verschoben). Alle Namen/Zähl-Formeln/URLs 1:1 aus der v1.22.30-Version übernommen (automatisiert aus der Datei extrahiert, nicht neu abgetippt, um Tippfehler bei den 6 umbenannten Listen-URLs auszuschließen). Gesamthöhe ca. 890px statt 2450px.
3. **Task Occurrences / "Next Steps"-Konzept - bewusst NICHT umgesetzt, siehe Priorität 2 Punkt 1.** Der Nutzer hat klargestellt, dass die eigentliche Anforderung über Lesbarkeit hinausgeht (CoS-Leader-Ansicht "erledigt/als nächstes" + automatischer E-Mail-Versand aus Vorlagen+Parametern). Das ist ein neues Feature-Konzept, kein Bugfix - braucht ein gemeinsames Brainstorming, bevor etwas gebaut wird. Klärungsfragen sind unter Priorität 2 Punkt 1 vorbereitet.
4. Kontrollen vor Auslieferung: App-weite Control-Namens-Eindeutigkeit (827 Namen, 0 Duplikate — Rückgang gegenüber 847 durch das Entfernen der 16 Health-Row-Wrapper-Container und der 17 Configuration-Beschreibungs-Labels, teilweise ausgeglichen durch 17 neue Dot-Controls), Regex-Scan auf literales `": "` in einzeiligen `Text:`/`Tooltip:`-Formeln (0 Treffer in den neu geschriebenen Zeilen; 1 Treffer im neuen Release-Notes-Eintrag vorsorglich auf `" - "` umgestellt), alle 17 SharePoint-Listen als vorhandene Datenquellen in der lokalen `.msapr` verifiziert (keine neue Datenquelle nötig, da nur bereits verwendete Listen referenziert werden).
5. `PowerApp_Version.txt` auf `v1.22.31` aktualisiert. Regel 9b: Backup auf `DMP_COMMAND_v1.22.30.msapp` rotiert (v1.22.30 gilt als vom Nutzer bestätigt ladend — lädt fehlerfrei in Studio, die 3 Anmerkungen betrafen nur UX, nicht Ladefehler).
6. Solution unverändert bei `7.34.67`, keine Power-Automate-Änderungen diese Sitzung.

---

## 🟢 v1.22.30 (2026-09-23, lokal gepackt) — 3 Findings aus v1.22.29-Test behoben (System-Health-Seite, Task-Occurrences-Lesbarkeit, Configuration-Tab-Vollständigkeit)

Nutzer hat `v1.22.29` in Studio/App getestet (Screenshots) und 3 Findings gemeldet; das Schreiben in `DMP Command Default Case Context` (B3-Popup) war dabei **erfolgreich** (erster echter Live-Test der Person-/Choice-Spalten-Patches aus v1.22.29 - funktioniert).

1. **System-Health-Legende auf eigene Seite ausgelagert:** Das alte Popup (`conHeartbeatLegendPopup`, 312×300px mit internem Scroll) konnte alle 16 überwachten Punkte nie gleichzeitig lesbar zeigen. Neuer Screen `scrSystemHealthDetails.pa.yaml` (neuer Sidebar-Eintrag "System Health" + weiterhin per Klick auf den Ring erreichbar) zeigt alle 16 Punkte thematisch gruppiert (Core Status: Status Check/Critical/Warnings/Operating State; Data Sources & Files: Emergency Report/Internal Domains/External Domains/Counter/Audit Trail; Agents 1–7), ohne Höhenlimit, mit denselben Live-Formeln wie zuvor (1:1 aus dem Popup übernommen, nicht neu geraten). Das alte Popup inkl. `varShowHeartbeatLegend` vollständig entfernt.
2. **Task Occurrences "Current view"-Panel - echter Lesbarkeits-Bug gefunden und behoben:** Die Beschreibungs-Box (`lblTaskOccurrencesContract`) war nur 28px hoch für ~300 Zeichen Text - der Grossteil rendert daher ausserhalb der Box und überlappte optisch die Zeile darüber (vom Nutzer als "unlesbar" gemeldet). Box-Höhe auf 78px erhöht, `Wrap:true` + `VerticalAlign:Top` ergänzt, Panel-Gesamthöhe 88→140px (Gallery-Höhenformel entsprechend nachgezogen). Zusätzlich hatte `lblTaskOccurrencesModeValue` (und im Gallery-Template `lblOccurrenceStatus`) eine fest codierte dunkelviolette Farbe ohne Dark-Mode-Unterscheidung - im Dark Mode nahezu unsichtbar (derselbe Bug-Typ wie der System-Health-Ring-Fix vom 2026-09-22) - beide jetzt themafähig.
3. **Task Occurrences "Current view"-Wert korrigiert ("Funktion unklar"):** Der Text "Open and overdue occurrences" beschrieb NICHT, was die Gallery tatsächlich zeigt (`Filter(..., ApprovalState.Value<>"Rejected")`, sortiert nach `DueUtc` aufsteigend - zeigt auch bereits erledigte/"Done"-Zeilen). Text korrigiert auf "All non-rejected occurrences, soonest due first" (Formel selbst bewusst NICHT geändert, um keine Daten zu verstecken, die der Nutzer ggf. noch braucht - reine Text/Doku-Korrektur).
4. **Configuration (Lists)-Tab: alle 17 live verbundenen SharePoint-Listen jetzt verlinkt, thematisch strukturiert.** Bisher nur 3 Kacheln (Configuration, Agent Status, Internal Domains); 14 fehlten komplett. Neuer scrollbarer Body-Container (`conConfigurationBody`, `LayoutOverflowY:Scroll`) mit 6 Abschnitten: Core Runtime Configuration & Monitoring (Configuration, Agent Status, **Counters neu**), Domain Classification (Internal Domains, **External Domains neu**), Streams - Process & Checklists (**Checklist Overall Process, Checklist CoS Leader neu**), Streams - Case & Scheduling (**Default Case Context, Checklist Recurrence Rules, Checklist Task Occurrences, Status Change Approvals neu**), Streams - Communication (**Email Templates, Email Placeholders, Recipient Groups neu**), Streams - Access Control & Catalogs (**Role Assignments, Screens Catalog, Actions Catalog neu**). Jede neue Kachel hat einen "View"-Link (SharePoint `AllItems.aspx`); "New Entry" nur bei manuell befüllbaren Listen (nicht bei Agent-7-generierten Task Occurrences/Status Change Approvals/Counters, analog zum bestehenden Agent-Status-Muster).
   **Site/Interner-Name-Fund (verifiziert direkt aus der lokalen `.msapr`-`DataSources.json`, nicht geraten):** `DMP Command Counters/External Domains/Checklist Overall Process/Screens Catalog/Actions Catalog` liegen auf der Haupt-Site `GO365_DMPCommunication` unter ihrem Anzeigenamen als URL-Segment. Alle übrigen 8 neuen Listen liegen auf `GO365_DMPCommunication-CoSLeader`; davon behalten `Checklist Recurrence Rules`, `Checklist Task Occurrences` und `Email Placeholders` ihren Anzeigenamen auch als internen URL-Namen, aber `Checklist CoS Leader` (`NextSteps_CosLeaderChecklist`), `Default Case Context` (`NextSteps_DefaultCaseContext`), `Status Change Approvals` (`NextSteps_StatusChangeApprovals`), `Email Templates` (`NextSteps_EmailTemplates`) und `Recipient Groups`/`Role Assignments` (`NextSteps_RecipientGroups`/`NextSteps_RoleAssignments`) sind umbenannte Listen, deren interner URL-Name noch der alte Arbeitsname ist - die "View"-Links dieser 6 Kacheln nutzen daher bewusst den internen Namen, nicht den Anzeigenamen.
5. Kontrollen vor Auslieferung: App-weite Control-Namens-Eindeutigkeit (847+ Namen, 0 Duplikate), Pack→Unpack-Rückvergleich (0 Diff auf allen geänderten/neuen Dateien), Regex-Scan auf literales `": "` in einzeiligen `Text: ="..."`-Formeln (4 Treffer in den neuen Abschnittsüberschriften gefunden und auf `" - "` umgestellt, bevor gepackt wurde).
6. `PowerApp_Version.txt` auf `v1.22.30` aktualisiert. Regel 9b: Backup auf `DMP_COMMAND_v1.22.29.msapp` rotiert (v1.22.29 gilt als vom Nutzer bestätigt ladend, da er beim Testen erfolgreich in die Liste geschrieben hat).

---

## 🟢 v1.22.29 (2026-09-23, lokal gepackt) — B3 + Task Occurrences live an echte SharePoint-Listen angebunden

Nutzer hat `DMP Command Default Case Context` UND `DMP Command Checklist Task Occurrences` (plus mehrere weitere Streams-Listen) als Live-Datenquellen in Studio hinzugefügt, gespeichert und veröffentlicht (**v1.22.28 damit vom Nutzer bestätigt ladend/speicherbar** - Backup gemäß Regel 9b auf `DMP_COMMAND_v1.22.28.msapp` rotiert).

1. **`.msapr` aktualisiert:** aktuelle Live-App per `pac canvas download` heruntergeladen, `References\DataSources.json` geprüft - beide neuen Listen sowie weitere vom Nutzer verbundene Streams-Listen (Checklist CoS Leader, Recurrence Rules, Email Templates/Placeholders, Recipient Groups, Role Assignments, Status Change Approvals) sind jetzt als Datenquellen vorhanden. `.msapr` 1:1 ins Repo übernommen (Pflicht-Checkliste A.1).
2. **B3 (Default Case Context) - jetzt Patch() gegen die echte Liste:** `Collect(colDefaultCaseContextPending, ...)` ersetzt durch `Patch('DMP Command Default Case Context', Defaults(...), {...})`. **Wichtiger Schema-Fund:** die Liste hat KEINE eigene `CaseId`-Spalte - der Case-ID-Wert gehört in `Title`. Person-Feld `SetByUser` und Choice-Feld `ModeAtCreation` entsprechend als Record gepatcht (`{Claims:...,DisplayName:...,Email:...}` bzw. `{Value:...}`). Bei Patch-Fehlschlag (`IsBlank(Ergebnis)`) wird die SharePoint-Fehlermeldung im bestehenden Validierungslabel angezeigt. `colDefaultCaseContextPending` vollständig entfernt.
3. **Task Occurrences (C6/C7) - jetzt Patch()/Filter() gegen die echte Liste:** Gallery liest jetzt `SortByColumns(Filter('DMP Command Checklist Task Occurrences', ApprovalState.Value<>"Rejected"), "DueUtc", SortOrder.Ascending)` statt der lokalen Collection. Propose/Approve/Reject patchen jetzt direkt (`Patch(..., ThisItem, {...})`), Vier-Augen-Prüfung über `ThisItem.ProposedBy.Email<>User().Email`. **Wichtiger Schema-Fund:** auch hier keine eigene `OccurrenceId`-Spalte - der Business-Key steht in `Title`. `colTaskOccurrencesPreview` vollständig entfernt. **Hinweis:** die echte Liste ist voraussichtlich noch leer (Agent 7 hat noch keinen echten Lauf gemacht) - die Gallery kann also 0 Zeilen zeigen, das ist erwartet, kein Bug.
4. **Erstmalige echte Person-/Choice-Spalten-Patches in dieser App** - noch nicht Studio-live-validiert. Bitte nach dem Laden gezielt die B3-Popup-Bestätigung testen (schreibt einen echten Datensatz) und - falls eine Testzeile in Task Occurrences existiert - Propose/Approve/Reject.
5. `PowerApp_Version.txt` auf `v1.22.29` aktualisiert (Regel eingehalten). Pack/Unpack-Rückvergleich: 0 Diff auf allen geänderten Dateien.

---

## 🟠 v1.22.28 + Solution 7.34.67 (2026-09-23, lokal gepackt/gepatcht, Solution bereits importiert von der KI) — Admin-Diagnostics-Abbau + B5 systemischer Fix (6 Flows) + Agent-2-Concurrency

**Update:** Solution `7.34.67` wurde von der KI selbst per authentifizierter `pac`-Session importiert und veröffentlicht (nicht vom Nutzer - siehe Regel-Korrektur in den Arbeitsregeln). **Noch offen:** die 6 geänderten Flows (Agent 1,2,3,4,5,6) müssen vom Nutzer je einmal im Power-Automate-Designer geöffnet/gespeichert werden ("deactivated and replaced").

1. **Admin Functions - "TIMESTAMP DEBUG"-Panel entfernt:** `conFuncTimestampDebug` (seit v1.22.21 als temporäres Diagnose-Panel für den Jahr-3926-Timestamp-Bug) komplett ausgebaut, da der Bug bestätigt gefixt ist. Pack/Unpack-Rückvergleich: 0 Diff.
2. **B1 endgültig abgeschlossen** (siehe eigener Abschnitt oben) - Backlog/Guide entsprechend aktualisiert.
3. **B5 - echter systemischer Bug gefunden und in 6 von 7 Flows gefixt (Agent 1, 2, 3, 4, 5, 6 - nur Agent 7 nicht betroffen):** Beim Weiterbau von B5 (Agent-2-EffectiveMode-Mapping) entdeckt, dass alle 6 Flows denselben Mechanismus (`Select_ConfigEntries`) zur modusabhängigen Config-Werte-Auflösung nutzen, der bisher NUR 4 Modi kannte (`PROD_NODMP`/`PROD_DMP`/`SIMU_NODMP`/sonst-`SIMU_DMP`). Seit B2 kann `CurrentOperationMode` aber auch `PROD_PREDEFAULT`/`PROD_POSTDEFAULT`/`SIMU_PREDEFAULT`/`SIMU_POSTDEFAULT` sein - dafür fiel die Formel bisher IMMER auf die `SIMU_DMP`-Spalte zurück. Das ist ein echter, bereits seit B2 scharfgeschalteter Bug (alle modusabhängigen Config-Werte wie Mailtexte, Betreff-Präfixe, Ordnernamen waren in Pre-/Post-Default falsch), bisher unbemerkt, weil niemand einen mailgenerierenden Agenten während Pre-/Post-Default getestet hatte. Die CSV zeigte, dass für SIMU Pre-/Post-Default bereits bewusst eigene (Fire-Drill-)Texte vorbereitet waren - ein einfaches Ummappen auf NODMP wäre also falsch gewesen, die echten neuen B1-Spalten mussten korrekt eingebunden werden.
   **Kritischer Fund dabei:** Die internen SharePoint-Feldnamen der 4 neuen Spalten waren NICHT aus dem Anzeigenamen ableitbar (anders als bei den 4 ursprünglichen Spalten) - SharePoint hat die automatisch generierten internen Namen auf ca. 26–27 Zeichen gekürzt und bei 2 davon einen Kollisions-Suffix angehängt. Erst durch einen vom Nutzer bereitgestellten Rohdaten-Export eines echten `GET_DMP_Command_Configuration`-Laufs bestätigt (gegen 2 verschiedene Config-Zeilen kreuzgeprüft):
   `Value_x0020__x002d__x0020_PROD_x` = PROD Pre-Default, `Value_x0020__x002d__x0020_PROD_x0` = PROD Post-Default, `Value_x0020__x002d__x0020_SIMU_x` = SIMU Pre-Default, `Value_x0020__x002d__x0020_SIMU_x0` = SIMU Post-Default.
   Identisch in allen 6 Flows gefixt (Agent 2 hat eine leicht abweichende Variante mit `coalesce(...,'')` pro Zweig und Variablenname `CurrentOperationMode` statt `OperationMode` - separat behandelt). JSON-Syntax aller 6 Dateien geprüft (gültig), neue Feldreferenz je genau 1× bestätigt.
   Versionslabels gebumpt: Agent 1 1.0.8→1.0.9, Agent 2 1.0.9→1.0.10, Agent 3 1.1.4→1.1.5, Agent 4 1.4.4→1.4.5, Agent 5 1.1.6→1.1.7, Agent 6 1.3.1→1.3.2. Solution-Checksumme neu berechnet: **7.34.67** (siehe Session Restart Guide für die volle Tabelle).
4. **Agent 2 - Concurrency auf expliziten Nutzerwunsch geändert:** `maximumWaitingRuns` auf `100` gesetzt (vorher impliziter Standard `10`). `runs` (Parallelitätsgrad) bewusst bei `1` belassen (sequenziell) - Agent 2 erhöht einen gemeinsamen SharePoint-Zähler nach dem Muster Lesen→+1→Schreiben ohne Locking; ein höherer Parallelitätsgrad würde das Risiko doppelt vergebener Zähler-/Referenznummern erzeugen. Dem Nutzer mitgeteilt, nicht eigenmächtig geändert.

**Noch offen:**
1. ~~Power App `DMP_COMMAND.msapp` (v1.22.28) muss vom Nutzer in Studio geladen/gespeichert/bestätigt werden.~~ Erledigt - Nutzer hat geladen, Datenquellen ergänzt, gespeichert und veröffentlicht (siehe v1.22.29-Eintrag oben).
2. ~~Solution 7.34.67 muss importiert werden~~ Von der KI selbst importiert/veröffentlicht (siehe Regel-Korrektur). **Update 2026-09-23:** Nutzer hat bestätigt, die Agenten waren (weiterhin) aktiv - diesmal war entgegen früherer Sessions KEIN manuelles Öffnen/Speichern im Power-Automate-Designer nötig. Erledigt.
3. B5 ist damit fachlich korrigiert, aber die ursprüngliche B5-Idee (Agent-2-Mapping über eine `EffectiveMode`-Konfigurationszeile) wurde NICHT umgesetzt - stattdessen wurden die echten neuen B1-Spalten direkt eingebunden (technisch der robustere Weg, da SIMU Pre-/Post-Default eigene Werte brauchen). Der in Backlog/Guide erwähnte Parameter `Agent2EffectiveModeMapping` existiert nicht als echte Config-Zeile und wird nicht mehr benötigt - aus Doku entfernen, sobald dieser Fund final bestätigt ist.
4. B4 (Agent 5 ruft bei DMP-Übergang zuerst Agent-7-Aktion `SetDefaultCaseContext` auf) weiterhin offen - diese Agent-7-Aktion existiert noch nicht.

---




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

## ✅ B1/B2 (Streams-Konzept, Strang B – 5-Modus-Umstellung) – umgesetzt und vom Nutzer bestätigt

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

**Update 2026-09-23:** Nutzer hat bestätigt, dass die 4 neuen Spalten jetzt live in der echten
SharePoint-Liste `DMP Command Configuration` angelegt sind. Auf Nutzerhinweis wurde die CSV
programmatisch auf Vollständigkeit der 8 Werte-Spalten über alle 106 Zeilen geprüft: bis auf 5
Zeilen (`AuditTrailOpenUrl`, `CounterOpenUrl`, `EmergencyReportOpenUrl`,
`ExternalDomainsOpenUrl`, `InternalDomainsOpenUrl`) sind alle Zeilen vollständig befüllt. Diese
5 Zeilen sind bereits seit vor der B1-Erweiterung durchgängig in ALLEN 8 Werte-Spalten UND in
`CurrentValue` leer (kein B1-spezifisches Problem, siehe neuer Punkt in Priorität 3 unten).
**B1 damit erledigt und vom Nutzer bestätigt.**


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

**🟡 Neue Konkretisierung durch den Nutzer (2026-09-23, v1.22.30-Test, Finding 2):** Die
bestehende Task-Occurrences-Seite (Lesbarkeits-Fix in v1.22.30, s.o.) ist NICHT das, was der
Nutzer eigentlich braucht — der Nutzer sagt wörtlich: *"Ich möchte, dass der CoS Leader die
Aufgaben im 'Next Steps'-Bereich sieht. Was ist erledigt, was kommt als nächstes. Von dort
soll er automatisch E-Mails versenden können, basierend auf den Vorlagen und befüllt mit den
Parametern aus den neuen SharePoint Listen!"* Das ist ein eigenständiges Feature-Konzept
(CoS-Leader-Arbeitsansicht + E-Mail-Auslösung), keine reine Lesbarkeits-Korrektur — **explizit
zurückgestellt, bis im Brainstorming-Modus mit dem Nutzer gemeinsam geklärt.** Noch nicht
implementiert; die bestehende `scrTaskOccurrences.pa.yaml` (Propose/Approve/Reject,
Vier-Augen-Prinzip) wurde in v1.22.30 nur lesbarer gemacht, aber inhaltlich nicht verändert.

**Vorbereitete Klärungsfragen für das nächste Brainstorming (von der KI vorformuliert, 2026-09-23):**
1. Ist die "Next Steps"-Ansicht ein Ersatz/Umbau der bestehenden `scrTaskOccurrences`-Seite, oder ein zusätzlicher, separater CoS-Leader-Screen (die bestehende Seite bliebe dann als reine Vier-Augen-Freigabe-Ansicht für alle Nutzer erhalten)?
2. Was genau bedeutet "erledigt" vs. "als nächstes"? Nur der `ApprovalState`/Status der einzelnen Task Occurrence, oder soll auch der Status aus `Checklist Overall Process`/`Checklist CoS Leader` einfließen?
3. Welche SharePoint-Liste(n) sollen die "Next Steps"-Liste inhaltlich speisen — nur `Checklist Task Occurrences`, oder zusätzlich `Checklist CoS Leader`/`Checklist Overall Process`?
4. Soll der E-Mail-Versand direkt aus der Power App erfolgen (z. B. Office365-Outlook-Connector), oder soll die App nur einen Agent-7-Flow anstoßen, der den eigentlichen Versand übernimmt (Vorbild: bestehendes Freigabe-Muster mit Audit-Trail-Dokumentation)?
5. Braucht der E-Mail-Versand ein Vier-Augen-Prinzip (wie Status-Änderungen), oder darf der CoS Leader allein per Klick versenden?
6. Woher kommt die Auswahl des passenden `Email Templates` je Aufgabe — automatisch anhand eines Felds in der Task Occurrence, oder wählt der CoS Leader die Vorlage manuell aus?
7. Welche Platzhalter-Quelle gilt je Vorlage — nur `Default Case Context` (Termination-Daten), oder auch `Recipient Groups` (Empfänger) und ggf. weitere Listen? Soll der Nutzer die befüllte E-Mail vor dem Versand noch sehen/anpassen können, oder komplett automatisch?
8. Ist der Auslöser pro einzelner Task Occurrence gedacht (eine Aufgabe = eine E-Mail), oder eher meilenstein-/status-getrieben (z. B. sobald ein ganzer Stream auf "Done" wechselt)?

**🟢 Update (2026-09-23, gleiche Sitzung wie v1.22.32):** Der Nutzer hat statt einzelner Antworten auf die 8 Fragen eine vollständige, 21-Abschnitte-Spezifikation geliefert:
[`DMP_COMMAND_Next_Steps_Anforderung.md`](../DMP_COMMAND_Next_Steps_Anforderung.md) (Kopie auch im
OneDrive-Dokumentationsordner). Kernpunkte: Aktivierung ab Pre-Default; global-fachliche Reihenfolge
ohne Zwangs-Sequenzialität (Sequenznummer ≠ Abhängigkeit); Mindestabhängigkeit jedes Tasks vom
Erreichen von Pre-Default, zusätzlich UND-verknüpfte Vorgänger-/Meilenstein-Bedingungen (keine
ODER-Verknüpfung); 5-Farben-Logik (Grau=noch nicht reif, Gelb=reif/PENDING, Gelb blinkend=ONGOING,
Grün=COMPLETED, Rot=OVERDUE/PROBLEM übersteuert alles andere); NEXT STEPS zeigt ~5 letzte
abgeschlossene + 5-12 offene Tasks mit zwingender Mindest-Repräsentation jedes aktiven Substreams
(mind. 1 erledigter + 1 offener Task je Substream, auch wenn noch nicht reif); zusätzlicher
Navigationsbereich "DMP Stream Tasks" für die Substream-Detailarbeit; bestehendes Vier-Augen-Prinzip
und bestehende Task-Occurrences-Logik werden wiederverwendet, nicht ersetzt; E-Mail-Versand
ausschließlich über Agent 7 (`SendChecklistEmail`), mit Vorschau vor Versand, kein zusätzliches
Vier-Augen-Prinzip für den Versand selbst; Template-Auswahl automatisch über `EmailTemplateId`;
manuelle Notfall-Backup-Vorlage pro E-Mail-Template gefordert (Format noch offen). Antworten auf
alle 8 obigen Fragen sind in Abschnitt 20 des Dokuments explizit enthalten.

**Abgleich gegen das echte Datenmodell (KI, 2026-09-23, aus der live `.msapr`/`DataSources.json`,
nicht den CSV-Vorlagen):** `Task Occurrences.DueUtc` existiert bereits live (deckt Fristen/Überfälligkeit
ab); `Role Assignments` hat bereits exakt die in Abschnitt 10 geforderten Felder (`SubStream`,
`IsSubStreamLead`, `IsCosLeadOrDeputy`, `AllowedScreens`, `AllowedActions`); `EmailTemplateId` existiert
bereits auf der CoS-Leader-Checkliste. **Es fehlt noch vollständig:** ein Vorgänger-/Abhängigkeits-Feld
(kein `PredecessorTaskId`/`Dependency`/`Milestone`-Feld auf irgendeiner Liste); `SeqNo` fehlt auf
`Checklist Overall Process` (existiert nur auf der CoS-Leader-Checkliste) — die im Dokument geforderte
globale Reihenfolge kann aktuell nicht gespeichert werden. Das deckt sich mit Abschnitt 21 des
Dokuments selbst, das genau diese Punkte als "noch nicht fachlich festgelegt" benennt. **Nächster
Schritt:** vor Implementierungsbeginn mit dem Nutzer klären, welche neuen Felder/Listen für das
Abhängigkeitsmodell angelegt werden (Regel: keine neuen Config-Felder ohne Rücksprache).

**🟢 Update (2026-09-23, direkt im Anschluss):** Konkrete Vorschlagsliste erarbeitet und in
[`Streams_ListTemplates/README.md`](../Streams_ListTemplates/README.md) dokumentiert (Nutzer hat sich für
"Vorschlagsliste vorbereiten, dann in einem Rutsch in SharePoint anlegen" entschieden): 3 neue Spalten auf der
bereits live existierenden Liste `DMP Command Checklist Overall Process` (`SeqNo`, `PredecessorTaskIds`,
`IsMilestone`) sowie 2 neue Checklisten-Listen (`DMP Command Checklist Infrastructure Team`,
`DMP Command Checklist Content Team` — Schema war schon seit 2026-09-03 vorbereitet, nur wegen fehlendem
SharePoint-Site-Zugriff zurückgestellt; CSV-Vorlagen jetzt ergänzt: `DMP Command Streams Infrastructure Team
Checklist.csv` / `DMP Command Streams Content Team Checklist.csv`). Verknüpfung Overall Process ↔
Substream-Checkliste läuft weiterhin über identische `Titel`/TaskID-Werte (kein zusätzliches Link-Feld nötig —
war schon 2026-09-03 so im README dokumentiert). **Nächster Schritt: Nutzer legt die Spalten/Listen in SharePoint
an, danach Implementierung der Next-Steps-Screens.**

**🟢 Update (2026-09-23, v1.22.33) — Phase 1 umgesetzt:** Nutzer hat die 3 Overall-Process-Spalten angelegt
und die Datenquelle aktualisiert+veröffentlicht. Infrastructure-/Content-Team-Checklisten konnten NICHT
angelegt werden (kein Schreibzugriff auf die Team-Sites) — Nutzer-Entscheidung: Next Steps vorerst nur für
CoS Leader, Infrastructure/Content Team folgen später (entweder sobald Site-Zugriff da ist, oder alternativ
auf der CoS-Leader-Site als Platzhalter). Zusätzliche Architekturfrage geklärt: die bestehende Task-Occurrences-
Seite ist nur für wiederkehrende Aufgaben gedacht (Agent 7 + Recurrence Rules) — für Next Steps wird
stattdessen direkt der Live-Status aus der jeweiligen Substream-Checkliste gelesen (Status/ConfirmedBy),
nicht aus Task Occurrences. Cockpit-NEXT-STEPS-Panel ist jetzt live (siehe Release Notes v1.22.33).
**Offen für eine spätere Sitzung:**
- Infrastructure Team/Content Team-Checklisten anlegen, sobald Site-Zugriff geklärt ist.
- Ein Agent/Job, der pro DMP-Fall automatisch eine Task Occurrence je Checklisten-Aufgabe anlegt (nicht nur
  für wiederkehrende) — damit könnte Next Steps später auf die reichhaltigeren Task-Occurrences-Daten
  (Fälligkeit, Fall-Bezug) umgestellt werden, inkl. echtem Rot/Overdue-Zustand.
- In-App-Vorschlagen/Bestätigen (Vier-Augen) direkt aus dem Next-Steps-Panel heraus (aktuell nur Leseansicht,
  Statusänderungen weiterhin nur in SharePoint).
- "DMP Stream Tasks"-Navigationsbereich (§10 der Spezifikation) für die Substream-Detailarbeit.
- E-Mail-Automatisierung über Agent 7 (§14-17 der Spezifikation).

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

- **v1.22.32-Feedback, kosmetisch, noch nicht behoben (2026-09-23):** Nutzer bestätigte v1.22.32 lädt fehlerfrei, meldete aber 2 rein optische Punkte, bewusst nicht sofort behoben, um mit "Next Steps" weiterzumachen: (1) In System Health (Details) ist der Kartentitel "Connection Diagnostics - Core & Domain Lists" abgeschnitten (Label `Width: =400` reicht bei 15pt Bold nicht für den vollen Text - Fix: Width erhöhen oder Text kürzen). (2) Die Karten/Container in System Health (Details) sind "nicht gut platziert" (genaue Abweichung vom Nutzer nicht spezifiziert - beim nächsten Öffnen genauer nachfragen/Screenshot einholen, bevor gefixt wird).
- **Hardcodierte AuditTrail-Datei-/Tabellen-IDs statt Config:** In Agent 1 (Finding A), Agent 2 (Item 4) und Agent 3 nutzen die `WRITE AuditEvent`/`AUDIT_*`-Aktionen weiterhin SharePoint-interne Datei-/Tabellen-IDs statt zentraler Config-Werte. Bewusst zurückgestellt (kein akutes Risiko, da sich diese IDs praktisch nie ändern), aber technische Schuld.
- **Agent 2, Item 3 – Mailbox-Ordner-Setup-Optimierung:** 4 Aktionen (Ordner anlegen/IDs abrufen) laufen bei jeder einzelnen E-Mail neu, obwohl sich die Ordnerstruktur nach dem ersten Lauf nicht mehr ändert (~2-6 Sek. Laufzeit-Ersparnis möglich pro Mail). Gleiches Muster auch bei Agent 1. Abwägung (Stale-Cache-Risiko bei manueller Ordner-Umbenennung) im Archiv dokumentiert. Nicht umgesetzt, niedrige Priorität.
- **Tote Config-Variablen bereinigen:** Einige ungenutzte Einträge (u. a. `CounterFolder`, `CounterFileName`) in `DMP Command Configuration` sollten bei Gelegenheit identifiziert und entfernt werden.
- **5 durchgängig leere `...OpenUrl`-Parameter (gefunden 2026-09-23 bei der B1-Vollständigkeitsprüfung):** `AuditTrailOpenUrl`, `CounterOpenUrl`, `EmergencyReportOpenUrl`, `ExternalDomainsOpenUrl`, `InternalDomainsOpenUrl` (Kategorie GUI) haben in `DMP Command Configuration.csv` weder `CurrentValue` noch irgendeine der 8 Werte-Spalten befüllt und werden aktuell nirgends im App-/Agenten-Code referenziert. Kein B1-Regressionsproblem (die Zeilen waren schon vor der B1-Spaltenerweiterung komplett leer), sondern vermutlich vorbereitete, aber nie mit den echten SharePoint-/Teams-Links befüllte Platzhalter. Nicht umgesetzt, niedrige Priorität — bei Bedarf mit dem Nutzer klären, ob diese Parameter noch gebraucht werden oder entfernt werden können.
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
