# DMP COMMAND – Next Steps und automatisierte DMP-Ablaufsteuerung

## 1. Zielsetzung

Die bestehende „NEXT STEPS“-Funktion von DMP COMMAND wird zu einer zentralen operativen Steuerungs- und Koordinationsfunktion für den DMP-Prozess ausgebaut.

Sie zeigt dem CoS Leader jederzeit:

- welche Aufgaben zuletzt abgeschlossen wurden,
- welche Aufgaben aktuell bearbeitet werden,
- welche Aufgaben bereits ausführbar sind,
- welche Aufgaben zwar als Nächstes folgen, aber noch nicht ausführbar sind,
- welchem DMP Substream die Aufgaben zugeordnet sind,
- welche operative Aktion für eine Aufgabe vorgesehen ist,
- welche Abhängigkeiten die Ausführung einer Aufgabe noch verhindern,
- und welche Aufgaben überfällig sind oder ein Problem aufweisen.

Die Funktion nutzt und erweitert die vorhandene Streams-, Task-Occurrences-, Rollen-, E-Mail-, Audit- und Agent-7-Architektur. Es wird keine parallele Daten-, Status-, Freigabe- oder Versandlogik aufgebaut.

---

## 2. Prozessphasen und Aktivierung

Die DMP-Prozesssteuerung wird mit dem Übergang von **Normal** zu **Pre-Default** aktiviert.

Die verbindliche Reihenfolge der Betriebszustände lautet:

**Normal → Pre-Default → DMP → Post-Default → Normal**

Bereits in Pre-Default werden administrative und vorbereitende Aufgaben aktiviert, insbesondere:

- Heads-up- und Informations-E-Mails,
- Readiness Checks,
- Completeness Checks,
- weitere vorbereitende Maßnahmen.

Andere Aufgaben werden erst bei Erreichen einer späteren Prozessphase ausführbar. Beispielsweise können Aufgaben des Infrastructure Teams vom Wechsel zu DMP abhängig sein.

Der Default Case Context für die konkreten Falldaten bleibt davon fachlich getrennt. Er wird beim Übergang zum DMP-Zustand erfasst und anschließend als Datenquelle für die E-Mail-Automatisierung und weitere fallspezifische Aktionen verwendet.

---

## 3. Führende Aufgabenstruktur und globale DMP-Reihenfolge

Ausgangspunkt der fachlichen Aufgabenstruktur ist die bisherige Datei `Status DMP Process.xlsx`.

Sie wird durch die SharePoint-Liste **DMP Command Checklist Overall Process** abgelöst. Diese bildet die gemeinsame, übergeordnete Aufgabenstruktur. Die Checklisten der Working Substreams referenzieren ihre Aufgaben über die Task-ID.

Die Overall-Process-Struktur definiert die **globale fachliche DMP-Reihenfolge** sämtlicher relevanter Aufgaben und Meilensteine über alle Substreams hinweg.

Diese globale Reihenfolge ist ausdrücklich **keine zwingend sequenzielle Abarbeitungsreihenfolge**:

- Substreams können ihre Aufgaben grundsätzlich parallel bearbeiten.
- Eine niedrigere oder höhere Sequenznummer bestimmt nur die fachliche Position im Gesamtprozess.
- Die tatsächliche Ausführbarkeit wird ausschließlich anhand der für den Task definierten Abhängigkeiten bestimmt.
- Wichtige teamübergreifende Meilensteine können die Ausführbarkeit nachfolgender Tasks steuern.

Nach erfolgter Migration wird für die operative Anwendung ausschließlich die SharePoint-basierte Struktur verwendet. Es erfolgt kein Parallelbetrieb mit den bisherigen Excel-Checklisten. Die migrierten Excel-Dateien werden anschließend nur noch als eingefrorenes Archiv vorgehalten.

---

## 4. Working Substreams

Für die Prozesssteuerung werden die bereits definierten Working Substreams verwendet:

- CoS Leader
- Infrastructure Team
- Content Team

Das Hotline Team erhält entsprechend der bereits getroffenen Entscheidung keine eigene Checkliste und keine eigene Template-Automatisierung.

Jede Aufgabe der Gesamtaufgabenstruktur ist einem verantwortlichen Substream zugeordnet.

Die Substreams arbeiten im Wesentlichen unabhängig und parallel. Abhängigkeiten zwischen Substreams bestehen nur dort, wo sie für konkrete Tasks oder teamübergreifende Meilensteine ausdrücklich definiert sind.

---

## 5. Trennung von Task-Status, Reife und Problemzustand

Task-Status und Ausführbarkeit sind fachlich getrennt zu behandeln.

### 5.1 Gespeicherter Task-Status

Die bestehende Statuslogik wird weiterverwendet, insbesondere:

- `NOT STARTED`
- `ONGOING`
- `COMPLETED` beziehungsweise `DONE`

Es werden durch diese Spezifikation keine zusätzlichen gespeicherten Task-Statuswerte beschlossen.

### 5.2 Dynamisch berechnete Reife

Ein noch nicht gestarteter Task kann abhängig von seinen Voraussetzungen entweder:

- **noch nicht reif** oder
- **reif und damit ausführbar**

sein.

Die Reife soll möglichst dynamisch aus den vorhandenen Prozess-, Task- und Occurrence-Daten berechnet werden. Sie ist nicht automatisch ein zusätzlicher persistierter Task-Status.

### 5.3 Problem- beziehungsweise Überfälligkeitszustand

Ein Task kann zusätzlich als problematisch oder überfällig dargestellt werden. Dieser Zustand übersteuert in der NEXT-STEPS-Darstellung die normale Statusfarbe.

Die konkrete technische Ermittlung eines Problems oder einer Überfälligkeit muss auf vorhandenen beziehungsweise noch fachlich festzulegenden Frist- und Fehlerinformationen beruhen. Sie darf nicht allein aus der Sequenznummer abgeleitet werden.

---

## 6. Verbindliches Abhängigkeitsmodell

Jeder DMP-(Sub-)Task besitzt mindestens eine Aktivierungsbedingung beziehungsweise Abhängigkeit.

Die minimale Abhängigkeit jedes DMP-Tasks ist:

**Der DMP-Prozess hat mindestens den Zustand Pre-Default erreicht.**

Weitere Abhängigkeiten können hinzukommen:

- erforderlicher DMP-Betriebszustand, beispielsweise `Pre-Default`, `DMP` oder `Post-Default`,
- `COMPLETED`-Status eines vorhergehenden Tasks,
- `COMPLETED`-Status mehrerer vorhergehender Tasks,
- Erreichen eines definierten Meilensteins.

Mehrere Voraussetzungen werden ausschließlich mit **UND** verknüpft.

Es gibt keine ODER-Verknüpfung von Task-Abhängigkeiten.

### 6.1 Ausführbarkeitsregel

Ein Task ist genau dann reif und ausführbar, wenn gleichzeitig:

1. der für den Task definierte DMP-Betriebszustand erreicht beziehungsweise gültig ist **UND**
2. alle für den Task definierten Vorgänger-Tasks den erforderlichen Abschlussstatus `COMPLETED` erreicht haben **UND**
3. alle weiteren ausdrücklich definierten Meilensteinbedingungen erfüllt sind.

Formal:

`Executable(Task) = DMP-Status-Bedingung erfüllt UND Vorgänger 1 COMPLETED UND Vorgänger 2 COMPLETED UND ... UND Meilensteinbedingungen erfüllt`

Die Reihenfolge beziehungsweise `SeqNo` eines Tasks ist keine automatische Abhängigkeit und macht einen Task weder reif noch blockiert.

### 6.2 Verhalten noch nicht reifer Tasks

Ein noch nicht reifer Task:

- darf bereits als zukünftiger Schritt in NEXT STEPS sichtbar sein,
- muss eindeutig als noch nicht ausführbar gekennzeichnet werden,
- darf keine ausführende Aktion anbieten,
- darf nicht allein aufgrund seiner Position in der globalen Reihenfolge gestartet werden.

Sobald sämtliche UND-verknüpften Voraussetzungen erfüllt sind, wechselt die Darstellung dynamisch von „noch nicht reif“ zu „reif“.

---

## 7. Verbindliche Farb- und Darstellungslogik

Die Reife und der Bearbeitungszustand eines Tasks werden in NEXT STEPS durch Farbe und Text dargestellt. Farbe darf niemals die einzige Informationsträgerin sein.

### 7.1 Grau: noch nicht reif

Bedeutung:

- Task-Status ist `NOT STARTED`.
- Mindestens eine definierte Abhängigkeit ist noch nicht erfüllt.
- Der Task ist sichtbar, aber noch nicht ausführbar.

Anzuzeigender Text, sinngemäß:

- „NOT STARTED“ oder „DORMANT“
- optional zusätzlich die noch offenen Abhängigkeiten

### 7.2 Gelb: reif

Bedeutung:

- Task-Status ist `NOT STARTED`.
- Der erforderliche DMP-Betriebszustand ist erreicht.
- Alle definierten Vorgänger-Tasks sind `COMPLETED`.
- Alle weiteren UND-verknüpften Voraussetzungen sind erfüllt.
- Der Task kann begonnen werden.

Anzuzeigender Text:

- „PENDING“ oder „WAITING“

### 7.3 Gelb blinkend: in Bearbeitung

Bedeutung:

- Task-Status ist `ONGOING`.
- Der Task wird aktuell bearbeitet.

Anzuzeigender Text:

- „ONGOING“

Die blinkende Darstellung muss dezent, zugänglich und mit den geltenden UI-Vorgaben vereinbar sein. Auch ohne Animation muss der Zustand durch Text und Symbol eindeutig erkennbar bleiben.

### 7.4 Grün: erledigt

Bedeutung:

- Task-Status ist final wirksam `COMPLETED` beziehungsweise `DONE`.
- Eine gegebenenfalls erforderliche Vier-Augen-Freigabe ist abgeschlossen.

Anzuzeigender Text:

- „COMPLETED“

### 7.5 Rot: überfällig oder Problem

Bedeutung:

- Ein Task ist überfällig oder weist einen fachlich beziehungsweise technisch festgestellten Problemzustand auf.

Anzuigender Text muss die Ursache unterscheiden, beispielsweise:

- „OVERDUE“
- „PROBLEM“
- eine vorhandene konkrete Fehlermeldung oder Begründung

Rot übersteuert in NEXT STEPS die normale Reife- oder Statusfarbe, ohne den zugrunde liegenden Task-Status zu verändern.

### 7.6 Prioritätsregel der Darstellung

Falls mehrere Darstellungsbedingungen gleichzeitig zutreffen, gilt folgende visuelle Priorität:

1. Rot: überfällig oder Problem
2. Grün: abgeschlossen
3. Gelb blinkend: in Bearbeitung
4. Gelb: reif
5. Grau: noch nicht reif

Die endgültigen HEX- beziehungsweise RGBA-Werte sind aus der bestehenden DMP-COMMAND-/Eurex-Farbdefinition zu übernehmen und nicht neu festzulegen.

---

## 8. NEXT STEPS auf der Hauptseite

Der Bereich „NEXT STEPS“ wird als kompakte, streamübergreifende operative Sicht des aktuellen DMP-Prozesses ausgestaltet.

Die Darstellung folgt der globalen DMP-Reihenfolge, ohne eine strikt sequenzielle Abarbeitung vorzutäuschen.

### 8.1 Abgeschlossene Tasks

NEXT STEPS zeigt grundsätzlich die letzten fünf abgeschlossenen Tasks.

Zusätzlich soll von jedem aktiven Substream mindestens der zuletzt abgeschlossene Task sichtbar sein. Falls diese Substream-Mindestregel mehr als fünf Einträge erfordert, hat die vollständige Substream-Repräsentation Vorrang vor dem Standardlimit.

Die Sortierung abgeschlossener Tasks erfolgt nach dem tatsächlichen Abschlusszeitpunkt, nicht allein nach `SeqNo`.

### 8.2 Offene Tasks

NEXT STEPS zeigt:

1. alle relevanten problematischen oder überfälligen Tasks,
2. alle `ONGOING`-Tasks,
3. mindestens den nächsten offenen Task jedes aktiven Substreams,
4. wichtige globale Meilensteine,
5. anschließend weitere relevante offene Tasks in der globalen fachlichen Reihenfolge.

Grundsätzlich werden etwa fünf bis zwölf offene Tasks angezeigt. Falls die Mindestdarstellung der aktiven Substreams oder die Anzeige problematischer Tasks mehr Einträge erfordert, darf dieses Standardlimit überschritten werden beziehungsweise die Ansicht muss eine geeignete weitere Darstellung ermöglichen.

### 8.3 Sichtbarkeit noch nicht reifer Tasks

Der nächste offene Task eines Substreams darf auch dann angezeigt werden, wenn er noch nicht reif ist.

Er wird dann:

- grau dargestellt,
- als „Noch nicht reif“ beschriftet,
- ohne ausführbare Aktion angezeigt,
- und kann die noch offenen Voraussetzungen anzeigen.

### 8.4 Gleitendes Prozessfenster

Sobald weitere Aufgaben abgeschlossen werden:

- rücken neue abgeschlossene Aufgaben in den Rückblick,
- fallen ältere abgeschlossene Aufgaben aus der kompakten Darstellung heraus,
- rücken weitere offene Aufgaben in die Vorschau nach,
- wird die Reife aller abhängigen Tasks erneut berechnet.

---

## 9. Zwingende Substream-Repräsentation

NEXT STEPS darf nicht lediglich die global nächsten Aufgaben ungeachtet ihrer Substream-Zugehörigkeit anzeigen.

Für jeden aktiven Substream muss sichtbar sein:

- mindestens der zuletzt abgeschlossene Task,
- mindestens ein weiterer offener Task,
- sowie jeder aktuell `ONGOING`, überfällige oder problematische Task.

Für den offenen Task gilt folgende Auswahlpriorität:

1. `ONGOING`,
2. andernfalls reifer `NOT STARTED`-Task,
3. andernfalls nächster noch nicht reifer `NOT STARTED`-Task.

So bleibt jederzeit sichtbar, wo jeder aktive Working Substream innerhalb des Gesamtprozesses steht.

---

## 10. DMP Stream Tasks

Zusätzlich zur kompakten Hauptansicht wird im linken Navigationsbereich ein eigener Funktionsbereich **„DMP Stream Tasks“** vorgesehen.

Dieser enthält separate Ansichten für die Working Substreams.

Von der NEXT-STEPS-Darstellung auf der Hauptseite führen kleine Direktzugriffe zum jeweiligen Substream.

Die Sichtbarkeit und die erlaubten Aktionen richten sich nach der bereits vorhandenen Rollen- und Rechtearchitektur mit den Berechtigungsinformationen:

- `SubStream`
- `IsSubStreamLead`
- `IsCosLeadOrDeputy`
- `AllowedScreens`
- `AllowedActions`

Der Prozess ist damit auf zwei Ebenen bedienbar:

**NEXT STEPS**  
Kompakte operative Gesamtsteuerung

**DMP Stream Tasks**  
Vollständige Arbeitsansicht des jeweiligen Substreams

---

## 11. Task-Verwaltung und Historisierung

Die Aufgaben der einzelnen Substreams müssen administrativ pflegbar sein.

Berechtigte Nutzer können insbesondere:

- Tasks hinzufügen,
- Taskbeschreibungen bearbeiten,
- Tasks in der globalen fachlichen Reihenfolge verschieben,
- Substream-Zuordnungen und operative Eigenschaften bearbeiten,
- DMP-Status-Abhängigkeiten pflegen,
- Vorgänger-Task-Abhängigkeiten pflegen,
- Tasks deaktivieren.

Ein physisches Löschen bereits verwendeter Tasks ist nicht als Standardfunktion vorgesehen.

Zwischen der Taskdefinition und einer bereits für einen konkreten DMP-Fall entstandenen Task Occurrence ist strikt zu unterscheiden:

- Die Deaktivierung einer Taskdefinition verhindert deren Berücksichtigung in zukünftigen DMP-Fällen.
- Bereits existierende Task Occurrences und historische Auditinformationen bleiben unverändert erhalten.
- Eine Aufgabe darf nicht allein durch eine spätere Stammdatenänderung aus einem laufenden oder historischen DMP-Fall verschwinden.
- Falls ein Task im laufenden DMP-Fall nicht mehr relevant ist, ist dies als fallspezifische, auditierbare Entscheidung zu behandeln und nicht durch rückwirkende Veränderung der Taskdefinition.

---

## 12. Task-Arten und Aktionen

Nicht jede Aufgabe führt dieselbe Aktion aus.

Mindestens zwei grundlegende Fälle sind erforderlich:

### 12.1 Einfache operative Tasks

Der Nutzer führt die Aufgabe aus und schlägt anschließend die entsprechende Statusänderung vor.

### 12.2 E-Mail-Tasks

Die Aufgabe referenziert eine definierte E-Mail-Vorlage. Über die Aufgabe wird die Vorbereitung und der Versand dieser Nachricht ausgelöst.

Weitere Task-Arten können später ergänzt werden. Die Architektur darf deshalb nicht ausschließlich auf „Bestätigen“ und „E-Mail senden“ fest codiert werden.

Ausführende Aktionen dürfen nur angeboten werden, wenn der Task reif ist und die nutzende Person über die erforderliche Berechtigung verfügt.

---

## 13. Vier-Augen-Prinzip

Für Statusänderungen wird die bestehende Vier-Augen-Logik weiterverwendet.

Der Ablauf lautet:

1. Eine berechtigte Person schlägt die Statusänderung vor.
2. Eine andere berechtigte Person bestätigt oder verwirft diese.
3. Erst die Bestätigung macht den neuen Status final wirksam.

Der Zweitfreigeber muss:

- eine andere Person als der Ersteller des Vorschlags sein,
- Mitglied desselben Substreams oder ein entsprechend berechtigter Lead sein.

Die vorhandene Propose-/Approve-/Reject-Logik der Task Occurrences ist weiterzuverwenden. Für NEXT STEPS wird keine zweite Freigabelogik entwickelt.

Abhängige Tasks dürfen erst nach dem final wirksamen `COMPLETED`-Status des Vorgänger-Tasks reif werden. Ein lediglich vorgeschlagener oder noch nicht genehmigter Abschluss erfüllt die Abhängigkeit nicht.

---

## 14. E-Mail-Automatisierung

E-Mail-bezogene Aufgaben nutzen die zentrale Template-Architektur.

Das bestehende Datenmodell enthält hierfür:

- Email Templates,
- Email Placeholders,
- Recipient Groups,
- Default Case Context,
- die Zuordnung eines `EmailTemplateId` zu Checklistenaufgaben.

Die Auswahl der Standardvorlage erfolgt automatisch über die jeweilige Aufgabe beziehungsweise deren Template-Zuordnung, nicht durch eine beliebige manuelle Auswahl beim Versand.

Die Versandaktion ist nur verfügbar, wenn:

- der Task reif ist,
- die erforderlichen Berechtigungen vorliegen,
- eine gültige Template-Zuordnung besteht,
- sämtliche erforderlichen Platzhalterwerte vorliegen beziehungsweise im Eingabedialog erfasst wurden.

---

## 15. E-Mail-Erstellung und Platzhalter

Templates enthalten:

- automatisch bestimmbare Werte, insbesondere aus dem Default Case Context,
- gegebenenfalls situativ durch den Nutzer zu erfassende Informationen.

Für situationsabhängige Inhalte wird ein Eingabe-Popup verwendet. Erst nach vollständiger Befüllung und Prüfung bestätigt der Nutzer den Versand. Ein vollständig stiller Versand ist nicht vorgesehen.

Vor dem Versand muss der Nutzer erkennen können:

- welche Vorlage verwendet wird,
- welche Empfänger vorgesehen sind,
- welche automatisch ermittelten DMP-Daten eingesetzt wurden,
- welche situationsabhängigen Werte noch fehlen,
- welchen endgültigen Inhalt die Nachricht hat.

---

## 16. Versand durch Agent 7

Der technische E-Mail-Versand erfolgt nicht als separate Direktversandlogik der Power App.

Hierfür wird die vorgesehene Agent-7-Aktion `SendChecklistEmail` verwendet beziehungsweise vervollständigt.

Agent 7 übernimmt:

- das Rendern der Vorlage,
- die Auflösung der definierten Platzhalter,
- die Ermittlung der Empfänger,
- den Versand über `default@eurex.com`,
- die Archivierung der versendeten Nachricht,
- den Audit-Trail-Eintrag,
- die strukturierte Rückmeldung des Ergebnisses an die App.

Damit bleibt die Power App die Steuerungs- und Benutzeroberfläche, während Agent 7 die kontrollierte Backend-Ausführung übernimmt.

---

## 17. Manuelle Notfalllösung für E-Mails

Zusätzlich zur automatisierten Verarbeitung muss für jede operative E-Mail-Vorlage eine für Menschen verständliche Backup-Version verfügbar sein.

Falls die App, Agent 7 oder eine andere benötigte Automatisierung während eines DMP nicht verfügbar ist, muss der operative Prozess manuell fortgesetzt werden können.

Dabei gilt:

**Es entsteht keine zweite, unabhängig gepflegte Quelle der Wahrheit.**

Die führende Template-Quelle bleibt das zentrale Template-Modell. Das Backup muss eindeutig derselben Vorlage und Version zugeordnet werden.

Die genaue technische Erzeugung und das endgültige Backupformat, beispielsweise Word oder Text/HTML, bleiben eine Implementierungsentscheidung und dürfen nicht ohne fachliche Entscheidung festgelegt werden.

---

## 18. Verhältnis von Overall Process, Checklisten, Task Occurrences, NEXT STEPS und Agent 7

Die bestehende Task-Occurrences-Funktion wird nicht durch eine neue konkurrierende Statusverwaltung ersetzt.

**Overall Process**  
Definiert die globale fachliche DMP-Reihenfolge, Phasen, Substream-Zuordnung und wichtige Meilensteine.

**Substream-Checklisten**  
Definieren die detaillierten operativen Aufgaben der jeweiligen Substreams.

**Task Occurrences**  
Bilden die konkrete, fallspezifische Ausführung und Freigabe innerhalb eines DMP-Falls ab, einschließlich `CaseId`, `TaskId`, `Status`, `ProposedBy`, `ApprovedBy`, `ApprovalState` und `CompletedUtc`.

**NEXT STEPS**  
Verdichtet Prozessstruktur, Task Occurrences, Reife, Problemzustände und Berechtigungen zur operativen CoS-Leader-Sicht.

**Agent 7**  
Übernimmt die serverseitige Ablauf-, Freigabe- und Kommunikationslogik.

---

## 19. Leitprinzipien für die Implementierung

- Keine parallele Daten-, Status-, Rollen-, Freigabe- oder E-Mail-Architektur.
- Vorhandene SharePoint-Listen, Feldnamen, Rollen, Task Occurrences, Agent-7-Aktionen, Betriebsmodi, Templates, Placeholders, Recipient Groups und Audit-Mechanismen sind wiederzuverwenden.
- Die globale Sequenz ist keine automatische Abhängigkeit.
- Reife wird aus DMP-Status und sämtlichen UND-verknüpften Voraussetzungen berechnet.
- Ein vorgeschlagener, aber noch nicht genehmigter Abschluss eines Vorgänger-Tasks erfüllt die Abhängigkeit nicht.
- Farbe wird immer durch Text und gegebenenfalls ein Symbol ergänzt.
- Historische Task Occurrences werden durch spätere Stammdatenänderungen nicht verändert.
- Keine neuen Listen, Spalten, Statuswerte, Variablen oder Agenten dürfen allein aufgrund dieser Spezifikation als technisch bereits beschlossen angenommen werden.
- Vor der Implementierung ist das bestehende Datenmodell gegen die Anforderungen zu prüfen. Fehlende technische Felder oder Strukturen sind als nachvollziehbarer Änderungsvorschlag auszuweisen und nicht stillschweigend einzuführen.

---

## 20. Antworten auf die offenen Konzeptfragen

### 20.1 Soll NEXT STEPS die bestehende Task-Occurrences-Seite ersetzen?

Nein. NEXT STEPS wird die operative CoS-Leader-Sicht. Die vorhandene Occurrence- und Freigabelogik wird wiederverwendet.

### 20.2 Was zählt als erledigt beziehungsweise als Nächstes?

Als erledigt gelten Tasks mit final wirksamem Status `COMPLETED` beziehungsweise `DONE`.

Als Nächstes werden problematische oder überfällige Tasks, `ONGOING`-Tasks und anschließend offene Tasks dargestellt. Bei `NOT STARTED` wird dynamisch zwischen „reif“ und „noch nicht reif“ unterschieden.

### 20.3 Welche Datenquellen speisen die Ansicht?

Es wird keine isolierte NEXT-STEPS-Liste geschaffen.

- Overall Process und Substream-Checklisten liefern die Prozessstruktur.
- Task Occurrences liefern den fallspezifischen Ausführungs- und Freigabestatus.
- Rollen- und Rechteinformationen steuern Sichtbarkeit und Aktionen.
- Templates, Placeholders, Recipient Groups und Default Case Context unterstützen E-Mail-Aufgaben.
- Abhängigkeiten bestimmen zusammen mit dem aktuellen DMP-Status die Reife.

### 20.4 Erfolgt der E-Mail-Versand direkt aus der App oder über Agent 7?

Der technische Versand erfolgt über Agent 7.

### 20.5 Gilt ein Vier-Augen-Prinzip?

Für Statusänderungen bleibt das bestehende Vier-Augen-Prinzip bestehen.

Für den eigentlichen E-Mail-Versand wird keine zusätzliche Vier-Augen-Regel unterstellt, solange diese nicht ausdrücklich fachlich festgelegt wird. Der Nutzer muss den vorbereiteten Versand aktiv prüfen und bestätigen.

### 20.6 Wie wird die E-Mail-Vorlage ausgewählt?

Automatisch über die der Aufgabe zugeordnete `EmailTemplateId`.

### 20.7 Welche Platzhalterquellen gelten, und wird eine Vorschau angezeigt?

Die Werte stammen aus dem Default Case Context, dem kontrollierten Placeholder-Modell und erforderlichen situativen Nutzereingaben.

Vor dem Versand wird der vollständig aufgelöste Inhalt einschließlich Vorlage, Empfängern und eingesetzten Werten angezeigt.

### 20.8 Ist der Auslöser aufgaben- oder meilensteinbezogen?

Die operative Steuerung basiert auf konkreten Tasks und Task Occurrences.

Meilenstein- und Cross-Team-Folgeaktionen bleiben Teil der Agent-7-Ablaufsteuerung und können nach final bestätigten Statusänderungen ausgelöst werden. Sie bilden keine unabhängige Parallelarchitektur.

---

## 21. Vollständigkeits- und Konsistenzprüfung

Die Spezifikation enthält nun ausdrücklich:

- Aktivierung ab Pre-Default,
- globale DMP-Reihenfolge ohne sequenzielle Zwangsabarbeitung,
- parallele Arbeit der Substreams,
- Mindestabhängigkeit jedes Tasks vom Erreichen von Pre-Default,
- zusätzliche Abhängigkeit vom DMP-Betriebszustand,
- Abhängigkeit vom final bestätigten `COMPLETED`-Status eines oder mehrerer Vorgänger-Tasks,
- ausschließlich UND-verknüpfte Voraussetzungen,
- Trennung von Task-Status und dynamischer Reife,
- vollständige Farb- und Textlogik für Grau, Gelb, gelb blinkend, Grün und Rot,
- mindestens letzten abgeschlossenen und nächsten offenen Task je aktivem Substream,
- Anzeige wichtiger Meilensteine,
- Umgang mit noch nicht reifen Tasks,
- Deaktivierung statt physischem Löschen,
- Schutz historischer Task Occurrences,
- rollenbasierte Detailansichten,
- Vier-Augen-Prinzip,
- Template-Zuordnung, Platzhalter, Vorschau und Versand über Agent 7,
- manuelle E-Mail-Notfalllösung,
- Abgrenzung der Verantwortlichkeiten der beteiligten Komponenten,
- und das Verbot stillschweigender neuer technischer Architekturen.

Noch nicht fachlich festgelegt und daher bewusst nicht als beschlossene technische Lösung formuliert sind:

- die konkreten SharePoint-Felder für DMP-Status- und Task-Abhängigkeiten,
- die konkrete Berechnung und Speicherung von Fristen beziehungsweise Überfälligkeit,
- die abschließenden HEX-/RGBA-Farbwerte,
- die technische Erzeugung und das Dateiformat der E-Mail-Backups,
- sowie die genaue Kennzeichnung, ob nur Meilensteine oder sämtliche Tasks standardmäßig in NEXT STEPS erscheinen.

Diese Punkte sind vor der Programmierung gegen den aktuellen Code- und Datenmodellstand zu prüfen und als gezielte Designentscheidungen zu dokumentieren.
