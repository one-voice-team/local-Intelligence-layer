# Vorschlag: Integration des Local Intelligence Layers in V2

> **Status:** Diskussions- und Planungsgrundlage. Dieses Dokument beschreibt mögliche Arbeitspakete und offene Fragen. Es ist weder eine verbindliche Architekturentscheidung noch eine Implementierungsanleitung. Die tatsächliche Lösung sollte vom zuständigen Integrationsteam anhand der bestehenden Systeme und der Anforderungen festgelegt werden.

## Kurzfassung

Eine Integration könnte V2s bestehende Performer-, Sitzplatz- und Player-Funktionen mit der vorhandenen Verhaltens- und Signalverarbeitung des Local Intelligence Layers (LIL) verbinden. V2 würde weiterhin die Teilnehmerinteraktion und die Darstellung auf den Geräten übernehmen. Der LIL könnte – wenn V2 ihn für einen konkreten Ablauf anfordert – anhand von Agenten, Zuständen, Regeln und den bereits vorhandenen Neighborhoods bestimmen, wie ein Ereignis lokal weiterwirkt.

Dafür wären insbesondere die Performer-Identität aus V2, die zugehörige Agent-Identität im LIL sowie Sitzplatz-/Positionsinformationen zuverlässig miteinander zu verknüpfen. Der konkrete Integrationsweg, die Zuständigkeit für Daten und die Schnittstellen müssten zuerst untersucht und gemeinsam beschlossen werden. Das Beispiel „Teilnehmende schütteln unten ihr Handy; nach einer Regel werden darüber sitzende Teilnehmende aktiviert“ könnte als begrenzter erster Anwendungsfall dienen; siehe auch das [Beispiel zur Crows-/Holy-Show-Aktivierung](crows-holy-show-aktivierung.md).

## Mögliche Arbeitspakete

### 1. Anforderungen und Anwendungsfall untersuchen

Als möglicher Auftakt könnte ein eintägiger Workshop mit den relevanten Rollen stattfinden. Denkbare Teilnehmende wären:

- Operator des Local Intelligence Layers
- System- bzw. Softwarearchitektur des Local Intelligence Layers
- Operator von V2
- System- bzw. Softwarearchitektur von V2
- Vertretungen betroffener Nutzergruppen, zum Beispiel Dirigenten oder Komponisten

Der Workshop könnte anhand einer Live-Vorführung beider Systeme mögliche Integrationsabläufe und notwendige Änderungen an der V2-Bedienung sammeln. Als Ergebnisse könnten ein priorisierter erster Anwendungsfall, offene fachliche und technische Fragen sowie erste Akzeptanzkriterien dokumentiert werden. Rollen und Format sind Vorschläge und sollten an die tatsächliche Projektorganisation angepasst werden.

Gemeinsam mit den betroffenen Rollen sollte geklärt werden:

- Welche konkreten Abläufe in V2 sollen den LIL nutzen, und welche bleiben vollständig in V2?
- Wer konfiguriert und startet einen LIL-gestützten Ablauf – zum Beispiel Operator, Dirigent oder eine andere Rolle?
- Welche Eingaben kann V2 tatsächlich liefern (z. B. Performer-ID, Sitzplatz, Bewegungssignal, Show-/Run-Kontext)?
- Welche Entscheidung wird vom LIL erwartet, und welches Ergebnis muss V2 erhalten, damit es einen Effekt auf den richtigen Geräten darstellen kann?
- Welche Anforderungen gelten für Reaktionszeit, Verfügbarkeit, Verhalten bei Verbindungsabbrüchen und Datenschutz?
- Welche Abläufe müssen bei Pause, Neustart, Wiederverbindung oder einem Sitzplatzwechsel gelten?

Als erster fachlicher Ablauf könnte geprüft werden: Ein Gerät meldet ein Schütteln; der LIL wertet den Agenten und seine vorhandenen Neighborhood-/Regelbeziehungen aus; V2 erhält die für die nächste Aktion relevanten Performer-Ziele und zeigt dort eine Aufforderung oder visuelle Reaktion an. Ob genau dieser Ablauf geeignet ist, sollte das Team mit den Nutzenden entscheiden.

### 2. Bestehende Architekturen und Zuständigkeiten prüfen

Vor einer Architekturentscheidung sollten die relevanten Teile beider Systeme gemeinsam nachvollzogen werden:

- V2: Erzeugung und Lebenszyklus der `performerId`, Sitzplatzplanung und QR-Code-Zuordnung, Performer-Session, Positions-/Layout-Daten, eingehende Gerätedaten sowie Player-/Ausgabeweg.
- LIL: Run-Kontext, Agent-Identität, Agent-Position, vorhandene Neighborhood-Zuordnungen, Signalverarbeitung, Regeln und Propagation.
- Welche Datenquelle ist für Performer-Identität, Sitzplatz/Position und Neighborhood-Mitgliedschaft jeweils maßgeblich?
- Wie werden Layout-Änderungen, ungültige Zuordnungen und veraltete Daten erkannt und behandelt?
- Wie bleibt V2s bestehende Grobarchitektur möglichst stabil, während die LIL-Anbindung gekapselt wird?

Die vorhandenen Neighborhoods des LIL sind dabei als bestehender Bestandteil zu behandeln. Zu untersuchen ist, wie die Sitzplatz- und Layout-Informationen aus V2 auf vorhandene Agenten und Neighborhoods abgebildet oder mit ihnen abgeglichen werden können – nicht, wie ein neues Neighborhood-Modell erfunden wird.

### 3. Identitäten und Daten referenzieren

Eine mögliche Richtung wäre, eine V2-`performerId` direkt auf einen LIL-Agenten abzubilden. Das sollte geprüft werden, statt es vorab festzuschreiben. Insbesondere ist zu klären:

- Ist eine Performer-ID über mehrere Shows und LIL-Runs stabil, oder gilt die Zuordnung nur innerhalb eines bestimmten Runs?
- Ist eine direkte 1:1-Zuordnung `performerId` → `agentId` eindeutig und dauerhaft genug?
- Welche Run-/Show-ID muss zusammen mit der Performer-ID verwendet werden, damit IDs nicht kontextübergreifend verwechselt werden?
- Ist die Sitzplatzposition Teil der Agent-Daten, der Run-Konfiguration oder eine Referenz auf V2-Layout-Daten?
- Wie werden spätere Sitzplatzwechsel, Ersatzgeräte und Wiederverbindungen abgebildet?

Ein zusätzliches Participant-Objekt sollte nicht automatisch vorausgesetzt werden. Es wäre nur dann sinnvoll, wenn die Untersuchung einen konkreten Bedarf an einer eigenständigen Zuordnung pro Run, Show oder Teilnahme ergibt. Die Entscheidung sollte aus den Lebenszyklen und Identitätsregeln beider Systeme folgen.

### 4. Schnittstelle und Ablauf festlegen

Zu untersuchen und als Vertrag festzuhalten wären:

- Kommunikationsmuster: beispielsweise HTTP/RPC für Anfragen oder Events/Socket-Nachrichten für laufende Interaktion.
- Eingabe an den LIL: mindestens die im konkreten Anwendungsfall benötigte Performer-Referenz, der Run-/Show-Kontext und der Trigger; Positionsdaten nur, sofern sie nicht bereits verlässlich im LIL verfügbar sind.
- Ergebnis des LIL: eine eindeutige, versionierbare Antwort, die V2 auf Performer-Ziele und konkrete Player-Aktionen abbilden kann.
- Fehler- und Randfälle: unbekannter Performer/Agent, fehlende oder veraltete Sitzplatzzuordnung, doppelter Trigger, Timeout, LIL nicht erreichbar sowie erneute Zustellung.
- Zustandsverantwortung: welche Änderungen in V2 bzw. im LIL gespeichert werden und wie beide Systeme konsistent bleiben.

Der LIL müsste nicht notwendigerweise jede Welle berechnen. Eine rein visuelle Welle könnte V2 direkt abspielen. Der Mehrwert des LIL wäre dort zu prüfen, wo ein eingehender Trigger anhand lokaler Agent-Zustände, Regeln und Neighborhood-Propagation eine Entscheidung auslösen soll.

### 5. Benutzeroberfläche und Bedienbarkeit untersuchen

Eine UI-Erweiterung für V2 könnte nötig sein, damit Operatoren den LIL sicher und ohne direkte Datenbank- oder Codearbeit verwenden können. Der genaue Umfang sollte anhand der Anforderungen festgelegt werden. Zu prüfen sind insbesondere:

- **Neighborhoods im Sitzplan:** vorhandene LIL-Neighborhoods anzeigen und – falls fachlich und technisch passend – Agenten/Performer anhand des Sitzplans zuordnen oder Zuordnungen prüfen. Die UI sollte die bestehende Neighborhood-Struktur nutzen und Änderungen nachvollziehbar machen.
- **Regeln erstellen und verwalten:** Regeln über verständliche Formulare oder visuelle Bausteine konfigurieren, statt Nutzerinnen und Nutzer JSON oder Programmcode schreiben zu lassen.
- **Lesbarkeit für Nicht-Entwickler:** Fachbegriffe erklären, Bedingungen und Aktionen in Alltagssprache zusammenfassen, Pflichtfelder und ungültige Kombinationen verständlich markieren.
- **Vorschau und sichere Prüfung:** Regeln vor Aktivierung prüfen und möglichst mit einer Vorschau oder einem Simulationslauf zeigen, welche Performer betroffen wären.
- **Betriebskontrolle:** Regeln aktivieren/deaktivieren, Änderungen versionieren oder nachvollziehbar protokollieren und im laufenden Ablauf Status bzw. Fehler verständlich anzeigen.
- **Berechtigungen und Schutz:** klären, wer Regeln bearbeiten, freigeben und während einer Show aktivieren darf.

Die UI sollte keine technische Komplexität verstecken, die für einen sicheren Betrieb relevant ist; sie sollte diese verständlich und handhabbar machen.

### 6. Prototyp und Schätzung auf eine belastbare Basis stellen

Nach den Untersuchungen könnte das Team einen eng begrenzten Ablauf als Proof of Concept auswählen. Dafür wären Change Requests bzw. Akzeptanzkriterien festzuhalten, die Performer-Agent-Zuordnung und ein vorhandener Neighborhood-Pfad zu integrieren und der Ablauf mit V2 und dem realen Player-Weg zu testen. Der gemessene Aufwand kann anschließend als bessere Schätzbasis für weitere Trigger, UI-Funktionen und Betriebsanforderungen dienen.

## Vorgeschlagene Validierung und Abschlussprüfungen

Vor einer Freigabe sollte das Team unter anderem prüfen:

- Performer-ID wird dem erwarteten Agenten im richtigen Show-/Run-Kontext zugeordnet.
- Sitzplatz-/Positionsdaten und vorhandene Neighborhood-Zuordnungen stimmen überein.
- Ein realer Trigger erreicht den LIL und erzeugt die erwartete Entscheidung, ohne unbeabsichtigte Nachbarn zu aktivieren.
- V2 kann die LIL-Ziele zuverlässig den verbundenen Performer-Geräten zuordnen und den Player-Effekt ausführen.
- Wiederholte, verspätete oder doppelte Trigger führen nicht zu unerwartetem Verhalten.
- Pause, Wiederverbindung, Ausfall eines Geräts und – falls relevant – Sitzplatzwechsel werden definiert und getestet.
- Regeln sind für die vorgesehenen Nutzer verständlich, validierbar und sicher aktivierbar.
- Regressionstests bestätigen, dass bestehende V2-Abläufe ohne LIL unverändert funktionieren.
- Ende-zu-Ende-Tests decken den vollständigen Weg vom QR-/Performer-Kontext und Geräte-Trigger bis zur sichtbaren Reaktion ab.

## Mögliche Team- und Zeitplanung

Es wird vorgeschlagen, **ein dediziertes Integrationsteam** mit klarer Verantwortung für diese Arbeit einzusetzen, statt mehrere Teams parallel an überlappenden Teilen zu arbeiten. Relevante Rollen aus V2, LIL, Produkt/Betrieb und betroffenen Nutzergruppen sollten in diesem Team vertreten oder gezielt eingebunden sein. Die bestehenden V2- und LIL-Verantwortlichen sollten Architektur- und Schnittstellenentscheidungen gemeinsam abstimmen.

Als unverbindliche Planungsbasis könnte das Team folgende Phasen betrachten:

| Phase | Inhalt | Grobe Dauer |
|---|---|---:|
| Anforderungen und Anwendungsfall | Vorgeschlagener Workshop mit relevanten Rollen, erster Ablauf, offene Fragen und Akzeptanzkriterien | ca. 1 Arbeitstag |
| Architektur- und Datenanalyse | V2- und LIL-Lebenszyklen, Identitäten, Layout/Neighborhoods, Schnittstellen | ca. 1 Woche |
| Eingrenzung und Prototyp | Change Requests, Adapter-/Contract-Prototyp, kleiner vertikaler Ablauf | abhängig vom Ergebnis; initial ca. 1–2 Wochen |
| Integration und UI | vereinbarte Schnittstelle, Zuordnungs-/Regelbedienung, Fehlerbehandlung | nach Prototyp neu schätzen |
| Abschlusstests und Stabilisierung | Ende-zu-Ende-, Regression-, Last-/Ausfall- und Nutzerprüfungen, Fehlerbehebung | separat einplanen; initial ca. 3–5 Arbeitstage |

Diese Dauern sind grobe Orientierung, keine Zusage. Umfang, technische Befunde, Verfügbarkeit der Systeme und Testumgebung können sie wesentlich verändern. Die Abschlusstests und Fehlerbehebungszeit sollten ausdrücklich eingeplant und nicht als Restaufgabe behandelt werden.

## Zusammenarbeit und Rückfragen

Dieses Dokument ist als Vorschlag für die gemeinsame Erarbeitung gedacht. Architektur, Datenverantwortung, Schnittstellen, UI-Umfang und Zeitplan sollten vom Integrationsteam validiert und entschieden werden.

Für Rückfragen zum Local Intelligence Layer bin ich jederzeit über Discord (DaBrian) oder per E-Mail erreichbar: **brianick95@gmail.com**.
