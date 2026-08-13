# DMP COMMAND – Projektbeschreibung und technische Mission sowie KI-Arbeitsregeln

---

## Zielsetzung

Es soll eine zentrale grafische Benutzeroberfläche auf Basis von Microsoft Power Apps entwickelt und weiterentwickelt werden. Die Anwendung trägt den Namen DMP COMMAND (Communication Operations Management, Monitoring and Notification Dispatch) und dient als zentrales Steuerungs-, Überwachungs- und Kontrollsystem für den gesamten Default Management Process (DMP).

Die Anwendung soll den Benutzern ermöglichen:
- Power-Automate-Workflows (Agenten) manuell zu starten
- Laufende Prozesse zu überwachen
- Betriebszustände und Fehlersituationen zu erkennen
- Statusinformationen zentral darzustellen
- Konfigurationsparameter zu verwalten
- Audit- und Monitoringinformationen einzusehen
- Simulationen und Produktivläufe voneinander zu unterscheiden
- Den operativen Betrieb während eines DMP-Ereignisses zu unterstützen

Das System soll als zentrale Kommandozentrale für alle DMP-bezogenen Automatisierungen fungieren.

---

## Systemarchitektur

Das Gesamtsystem besteht aus drei Hauptkomponenten:

### Power Apps (DMP COMMAND)

Die Power App bildet die grafische Benutzeroberfläche.

Der aktuelle Entwicklungsstand der Anwendung ist im Dokument `DMP_COMMAND_MASTER_PowerApps_CODE.docx` beschrieben.

Die App dient als:
- Dashboard
- Monitoring-Oberfläche
- Steuerungszentrale
- Konfigurationsoberfläche
- Fehler- und Statusanzeige

### Power-Automate-Workflows (Agenten)

Die fachliche Verarbeitung erfolgt in mehreren Power-Automate-Workflows, die innerhalb des Projekts als Agenten bezeichnet werden. Jeder Agent übernimmt einen klar abgegrenzten Aufgabenbereich.

Typische Aufgaben sind beispielsweise:
- Verarbeitung eingehender E-Mails
- Erstellung und Versand von Benachrichtigungen
- Erzeugung von Berichten
- Statusüberwachung
- Dokumentenerzeugung
- Datenvalidierung
- Kommunikation mit Fachbereichen

Die technische Dokumentation der einzelnen Agenten befindet sich in Dateien nach dem Schema `DMP Agent xxx_JSON_code.docx`. Dabei steht "xxx" für die jeweilige Agentennummer. Diese Dokumente enthalten den vollständigen Power-Automate-Code und stellen die technische Referenz für den jeweiligen Agenten dar.

### SharePoint-basierte Datenhaltung

Die Agenten verwenden mehrere zentrale SharePoint-Listen als gemeinsame Datenbasis.

---

## DMP Command Configuration

Diese Liste stellt die zentrale Konfigurationsdatenbank des Gesamtsystems dar.

Die Struktur wird dokumentiert in `DMP Command Configuration.csv`.

Grundprinzip:
- Es sollen möglichst keine fest codierten Werte verwendet werden.
- Sämtliche Steuerungsparameter werden zentral verwaltet.
- Agenten lesen ihre Konfiguration zur Laufzeit aus dieser Liste.
- Änderungen an Parametern sollen ohne Anpassung des Codes möglich sein.

Typische Konfigurationswerte sind:
- E-Mail-Adressen
- Verteiler
- SharePoint-Pfade
- Betriebsmodus-Einstellungen
- Texte und Vorlagen
- Zeitparameter
- Schwellwerte
- Dashboard-Einstellungen
- Agentenspezifische Steuerungsparameter

Die Liste fungiert als zentrale "Single Source of Truth" für sämtliche Konfigurationswerte des Systems.

---

## DMP Command Agent Status

Zur Überwachung der laufenden Prozesse schreiben sämtliche Agenten Statusinformationen in eine zentrale SharePoint-Liste.

Die Struktur wird dokumentiert in `DMP Command Agent Status.csv`.

Diese Liste dient als zentrale Monitoring-Datenquelle für die Power App.

Typische Informationen:
- Agentenname
- Agentenstatus
- Startzeit
- Endzeit
- Laufzeit
- Erfolgsmeldungen
- Warnungen
- Fehler
- Betriebsmodus
- Verarbeitungsdetails
- Audit-Informationen

Die Power App nutzt diese Daten für die Darstellung von:
- Statusampeln
- Laufzeitüberwachung
- Fehleranzeigen
- Eskalationsinformationen
- Dashboard-Kennzahlen
- Historische Statusverläufe

---

## Betriebsmodi

Das System unterstützt unterschiedliche Betriebsmodi. Alle Agenten und die Power App müssen diese Modi berücksichtigen:
- PROD_NODMP
- PROD_DMP
- SIMU_NODMP
- SIMU_DMP

Je nach Modus können unterschiedliche Parameter, Empfänger, Verarbeitungsschritte oder Oberflächenelemente aktiviert werden. Die Modussteuerung erfolgt über die zentrale Konfiguration.

---

## Monitoring- und Audit-Strategie

Ein zentrales Ziel von DMP COMMAND ist die vollständige Transparenz aller automatisierten Prozesse. Daher sollen alle Agenten:
- Ihren aktuellen Status melden
- Fehler protokollieren
- Wichtige Verarbeitungsschritte dokumentieren
- Auditinformationen bereitstellen
- Verwertbare Fehlermeldungen erzeugen
- Laufzeiten nachvollziehbar protokollieren
- Warnungen und Eskalationen kennzeichnen

Die Power App soll diese Informationen in Echtzeit oder nahezu in Echtzeit visualisieren.

---

## Rolle der KI

Wenn eine KI Änderungen, Erweiterungen oder Fehleranalysen durchführen soll, gelten folgende Prioritäten:

1. Die aktuelle Power-App-Logik aus `DMP_COMMAND_MASTER_PowerApps_CODE.docx` analysieren.
2. Die relevanten Agentendokumente `DMP Agent xxx_JSON_code.docx` prüfen.
3. Die zentrale Konfiguration aus `DMP Command Configuration.csv` als maßgebliche Quelle verwenden.
4. Die Statusdatenstruktur aus `DMP Command Agent Status.csv` berücksichtigen.
5. Bestehende Parameter, Variablen, Namen und Datenstrukturen wiederverwenden.
6. Keine neuen Felder, Variablen oder Konfigurationsobjekte erfinden, solange diese nicht ausdrücklich definiert wurden.
7. Die zentrale Konfiguration ist stets gegenüber fest codierten Werten zu bevorzugen.
8. Die Agent-Status-Liste ist die primäre Quelle für Monitoring- und Dashboardinformationen.

Zusätzlich gelten folgende Vorgaben:
- Vor jeder Änderung muss die bestehende Architektur verstanden werden.
- Vorhandene Komponenten sind nach Möglichkeit wiederzuverwenden.
- Änderungen sollen möglichst zentral und wartbar implementiert werden.
- Parameter dürfen nicht mehrfach gepflegt werden.
- Doppelte Logik ist zu vermeiden.
- Konfigurationswerte haben Vorrang vor fest codierten Werten.
- Monitoring, Auditierbarkeit und Fehleranalyse haben höchste Priorität.
- Bestehende Funktionalität darf nur verändert werden, wenn dies fachlich erforderlich ist.
- Vor jeder technischen Empfehlung müssen die vorhandene Dokumentation und der aktuelle Codestand geprüft werden.
- Keine Annahmen über Variablen, Felder, Listenstrukturen oder Konfigurationswerte treffen.
- Den Nutzer bei Power-Automate-Anpassungen regelmäßig an das Abspeichern der Änderungen und einen anschließenden Code-Refresh (aktualisierte JSON-Exportdatei) erinnern, inklusive Empfehlung für Zwischentests nach größeren Anpassungsschritten.
- Sichtbar darauf hinweisen, wenn bei einer Kachel im Power-Automate-Designer der Wechsel in den Textmodus ("In Textmodus umschalten") statt der Standardansicht mit Schlüssel/Wert-Paaren erforderlich ist, bevor ein Ausdruck eingefügt werden kann.
- Das projektweite Backlog-Dokument für größere, zurückgestellte Optimierungen regelmäßig aktualisieren. Dieses Backlog bezieht sich auf das gesamte DMP COMMAND System (alle Agenten, die Power App, sowie alle verwendeten Konfigurations- und Statuslisten/-dokumente), nicht nur auf den zuletzt bearbeiteten Agenten.
- Die Anwenderdokumentation regelmäßig mit dem tatsächlichen Codestand abgleichen und aktualisieren, sobald sich Agenten, die Power App oder zentrale Konfigurations-/Statuslisten ändern. Dies betrifft insbesondere `DMP_Multi_Agent_Workflow_Documentation.docx`, alle Agent-xxx-Workflow-HTML-Dateien (High Level und Detailed) sowie das `UAT_Playbook.docx`. Diese Dokumente dienen als Anwenderhandbuch und sind durchgehend auf Englisch zu verfassen. Bei größeren fachlichen oder technischen Änderungen proaktiv auf den Aktualisierungsbedarf hinweisen, auch wenn nicht explizit danach gefragt wird.
- Bei Umbauten an Power-Automate-Kacheln die Änderungen kachelbezogen gruppieren (nicht nach Konfigurationsschlüssel), sodass jede Kachel nur einmal geöffnet und bearbeitet werden muss. Die Reihenfolge der Anweisungen muss der tatsächlichen Ablaufreihenfolge im Flow entsprechen, damit der Nutzer nicht zwischen Abschnitten springen muss. Bei jeder so bearbeiteten Kachel zusätzlich prüfen, ob der Beschreibungstext ergänzt oder aktualisiert werden muss, ob weitere bekannte Fehler oder Optimierungen an genau dieser Kachel eingearbeitet werden können, und ob sich ein offener Backlog-Punkt auf diese Kachel bezieht. Falls ja, den Backlog-Punkt in derselben Änderung mit adressieren und im Backlog-Dokument als erledigt streichen.
- Bei kachelbezogenen Anpassungen nicht nachfragen, ob fehlende oder fehlerhafte Beschreibungstexte ergänzt bzw. korrigiert werden sollen – dies ist selbstverständlich und wird automatisch mitgeliefert (harte Obergrenze: maximal 255 Zeichen je Beschreibungstext). *(verankert am 2026-08-07)*
- Vor der Auslieferung von Kachel-Änderungsanweisungen ZWINGEND eine Vollständigkeits-/Überschneidungsprüfung durchführen: alle aktuell bekannten offenen Findings (Backlog-Einträge UND in derselben Sitzung neu entdeckte Punkte) auf identische Kachelnamen abgleichen, bevor die Anweisungen geliefert werden. Wird festgestellt, dass ein bereits an anderer Stelle behandeltes oder gleichzeitig zu bearbeitendes Finding dieselbe Kachel betrifft, MÜSSEN beide Änderungen in einer einzigen, zusammengeführten Anweisung für diese Kachel geliefert werden – niemals in getrennten Durchgängen, die ein zweites Öffnen derselben Kachel erfordern würden. Jede Kachel ist in der Ausgabe immer mit ihrem vollständigen technischen Namen zu benennen (nie nur durch Verweis wie „siehe oben" oder „aus Schritt X"), damit die Anweisung eigenständig und ohne Rückblättern umsetzbar ist. *(verankert am 2026-08-07)*
- Kachel-Änderungsanweisungen dürfen NIEMALS über Muster-Buchstaben, Legenden, Kürzel oder Verweistabellen (z. B. „Kachel X → Muster A, D, E") ausgeliefert werden, auch nicht als Kompaktierung bei vielen betroffenen Kacheln. Jede einzelne Kachel MUSS als eigener, in sich vollständiger Block erscheinen, der pro betroffenem Feld den vollständigen, tatsächlich einzusetzenden Ausdruck direkt copy-paste-fertig enthält – ohne dass der Nutzer eine Legende, ein Muster oder eine andere Kachel nachschlagen muss. Dies gilt unabhängig davon, wie lang die resultierende Ausgabe dadurch wird oder wie viel Text sich wiederholt; Kürze hat in diesem Punkt keine Priorität gegenüber Vollständigkeit und direkter Umsetzbarkeit. Vor dem Absenden einer Kachel-Änderungsantwort intern prüfen, ob irgendein Verweis wie „Muster X", „siehe Gruppe Y" oder eine Buchstaben-/Nummern-Legende verwendet wurde, und falls ja, die Antwort vor dem Absenden entsprechend auflösen. *(verankert am 2026-08-07)*
- Bei der Angabe eines zu ändernden Feldes/Wertes (z. B. JSON-Werte, Ausdrücke in Buffer-Audit-Event-Kacheln, Bodys, etc.) immer den VOLLSTÄNDIGEN Feldinhalt/Textblock inklusive der Änderung als fertigen Gesamttext liefern – niemals nur das isolierte geänderte Teilstück, das der Nutzer selbst in einen größeren Kontext einfügen oder zusammensuchen müsste. Ziel: Der gelieferte Text muss 1:1 per Copy/Paste in das jeweilige Feld übernommen werden können, ohne manuelles Zusammenfügen. *(verankert am 2026-08-07)*
- JSON-Wertobjekte (insbesondere `value`-Objekte in Buffer-Audit-Event-artigen Kacheln) immer als eingerücktes, mehrzeiliges JSON ausgeben – ein Schlüssel/Wert-Paar pro Zeile, nicht als einzeilig komprimiertes JSON. Beispielformat:
  ```
  {
    "TimestampUtc": "@{utcNow()}",
    "RunId": "@{workflow()?['run']?['name']}",
    "MessageId": "",
    "WorkflowPath": "@{variables('WorkflowPath')}"
  }
  ```
  Ziel: bessere Lesbarkeit beim späteren Nachschlagen. *(verankert am 2026-08-07)*
- Vor jeder Auslieferung von Kachel-/Code-Änderungen an den Nutzer ZWINGEND eine Plausibilitätsprüfung der gelieferten Werte durchführen (z. B. auf Duplikate, widersprüchliche oder aus dem Kontext ersichtlich falsche Werte wie identische `Decision`/`StepName`-Werte für eigentlich unterschiedliche Ergebniszweige). Wird bei dieser Prüfung eine Unklarheit oder ein nicht zweifelsfrei auflösbarer Sachverhalt festgestellt, NICHT eigenmächtig raten oder rekonstruieren, sondern den Nutzer aktiv nach der korrekten/gewünschten Entscheidung fragen, bevor die Änderung ausgeliefert wird. *(verankert am 2026-08-07)*
- Referenzen auf Aktionsnamen, die selbst wörtliche Anführungszeichen enthalten (z. B. `Get_DMP_Mailbox_Subfolder_ID_for_"Agent_1_Alerts"`), gegenüber dem Nutzer IMMER mit unescapten, wörtlichen Anführungszeichen ausgeben (z. B. `body('Get_DMP_Mailbox_Subfolder_ID_for_"Agent_1_Alerts"')?['value'][0]['id']`) – NIEMALS mit Backslash-Escaping (`\"`), wenn der Nutzer den Text in eine Kachel im Power-Automate-Designer einfügen soll (auch im Textmodus einer Kachel). Wichtige Klarstellung: Das Backslash-Escaping (`\"`) in der rohen Flow-JSON-Exportdatei selbst ist normal und KEIN Fehler (Standard-JSON-String-Escaping auf Dateiebene) – es darf nur niemals in Text übernommen werden, der dem Nutzer zum Einfügen in eine Kachel gegeben wird. Gilt für alle Agenten und alle künftigen Lieferungen mit solchen Aktionsnamen. *(verankert am 2026-08-07, bestätigt durch Speicherfehler in der Praxis und Nutzer-Klarstellung)*
- Bei der Vergabe NEUER Kachelnamen (wenn eine neue Aktion im Power-Automate-Designer angelegt werden soll) den Namen mit echten Leerzeichen angeben (z. B. „Get DMP Mailbox Parent Folder ID"), NICHT mit Unterstrichen. Erst wenn diese Kachel anschließend in einem Ausdruck referenziert wird (`outputs(...)`, `body(...)`, `actions(...)`), wandelt Power Automate die Leerzeichen automatisch in Unterstriche um (z. B. `Get_DMP_Mailbox_Parent_Folder_ID`) – diese Unterstrich-Schreibweise gilt ausschließlich für Referenzen/Verweise, nicht für die Namensvergabe selbst. *(verankert am 2026-08-07)*
- Der Nutzer verwendet die DEUTSCHE Sprachversion von Power Automate und Power Apps. Bei der Anweisung, eine NEUE Kachel/Aktion anzulegen, IMMER den deutschen Aktionstyp-Namen angeben (nicht den englischen), da nur dieser im Designer des Nutzers so erscheint. Format: „Kategorie – Aktionsname" (z. B. „Datenvorgang – Auswählen" statt „Select", „Bedingung" statt „Condition", „Variable initialisieren" statt „Initialize variable"). Bei Unsicherheit über die exakte deutsche Bezeichnung eines selteneren Aktionstyps NICHT raten, sondern die englische Typbezeichnung zusätzlich in Klammern angeben und den Nutzer um Bestätigung/Korrektur der genauen deutschen Bezeichnung bitten. *(verankert am 2026-08-07, Format vom Nutzer bestätigt)*
- **Diese Regel gilt genauso für JEDES einzelne Eingabefeld innerhalb einer Kachel, nicht nur für den Aktionstyp beim Neuanlegen.** Bei Verweisen auf ein zu bearbeitendes Feld (z. B. bei HTTP-Aktionen „Uri", bei E-Mail-Versand „emailMessage/To", bei Datei verschieben „parameters/destinationFolderPath", bei Verzögerung „count"/„unit") IMMER die deutsche Feldbezeichnung, wie sie im Designer des Nutzers erscheint, gemeinsam mit dem technischen JSON-Parameternamen angeben (z. B. „Feld **An** (`emailMessage/To`)"), damit die Anweisung eindeutig und ohne Rätselraten umsetzbar ist. Bei Unsicherheit über die exakte deutsche Feldbezeichnung dies offen benennen und den JSON-Schlüssel als eindeutigen Ersatz angeben, statt stillschweigend zu raten. *(verankert am 2026-08-11, nach expliziter Beanstandung durch den Nutzer)*
- NIEMALS auf zuvor in derselben oder einer früheren Sitzung gelieferten Text verweisen (z. B. „exakt derselbe Ausdruck wie bei Agent X, Schritt Y" oder „siehe oben"), auch nicht zur Vermeidung von Wiederholung. JEDER copy-paste-fertige Ausdruck, Wert oder Textblock MUSS bei jeder Auslieferung vollständig und eigenständig erneut ausgeschrieben werden, selbst wenn er identisch zu einem bereits an anderer Stelle gelieferten Text ist. Der Nutzer darf zu keinem Zeitpunkt gezwungen sein, in der Konversation zurückzublättern oder nach einer früheren Nachricht zu suchen, um eine Anweisung vollständig umsetzen zu können. *(verankert am 2026-08-11, nach expliziter Beanstandung durch den Nutzer)*

---

## Übergeordnetes Ziel

DMP COMMAND soll eine hochgradig transparente, wartbare, konfigurierbare und zentral steuerbare Plattform für den gesamten Default Management Process bereitstellen.

Alle Agenten arbeiten mit einer gemeinsamen Konfiguration, liefern standardisierte Statusinformationen und werden über eine einzige Power-App-Oberfläche gesteuert und überwacht.

Dadurch entsteht eine zentrale DMP-Kommandozentrale für:
- Operative Steuerung
- Monitoring
- Fehlerbehandlung
- Auditierung
- Simulationen
- Eskalationsmanagement
- Reporting
- Transparenz über alle automatisierten DMP-Prozesse
- Zentrale Konfigurationsverwaltung
- Agentenübergreifende Statusüberwachung

Das langfristige Ziel besteht darin, sämtliche DMP-relevanten Automatisierungen, Statusinformationen, Konfigurationen und Betriebszustände in einer einzigen zentralen Plattform zusammenzuführen und damit eine durchgängige Überwachung, Steuerung, Analyse und Auditierung aller DMP-Aktivitäten zu ermöglichen.
