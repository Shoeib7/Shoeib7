## 14. Modul Review-Cockpit: Oberfläche, Checklisten, Diff, Freigabe, Rückfragen, Feedback

Das Review-Cockpit ist die Stelle, an der aus einem Agentenergebnis eine abgesendete Bewerbung wird – oder eben nicht. Kein Versand, kein Portal-Absenden und keine verbindliche Zusage laufen ohne einen expliziten Klick hier (Leitsatz aus Kapitel 1, Architekturentscheidung aus Kapitel 7). Das Cockpit selbst schreibt und liest ausschließlich in der SQLite-Datenbank, die laut Kapitel 7 die Quelle der Wahrheit des Systems ist; es ist keine eigenständige Anwendung mit eigenem Datenmodell, sondern eine dünne, aber sorgfältig gestaltete Schicht darüber. Alles Inhaltliche – Bewertung (Kapitel 9), Recherche (Kapitel 10), Text (Kapitel 11), ATS-Prüfung (Kapitel 12), Rendering (Kapitel 13) – ist zu diesem Zeitpunkt bereits fertig; das Cockpit erzeugt nichts, es zeigt, fragt nach und protokolliert.

### 14.1 Grundsatz: kein Timeout ist eine Freigabe

Die Statusmaschine des Systems (Kapitel 3 der Terminologie) kennt für jede Stelle einen Zustand „bereit zur Freigabe“, aus dem nur eine explizite menschliche Aktion herausführt. Bleibt eine Bewerbung liegen, bleibt sie im Zustand „bereit zur Freigabe“ – niemals rutscht sie durch reines Verstreichen von Zeit nach „freigegeben“. Diese Invariante ist wichtiger als jede Komfortfunktion: Ein vergessenes Review darf nie zum Versand führen. Eine separate „Rückfrage offen“-Bewerbung darf außerdem nie die Prüfung der übrigen Bewerbungen des Tages blockieren – jede Stelle trägt ihren Rückfrage-Zustand einzeln.

Eine zweite Regel betrifft externe Texte: Zitate aus Stellenanzeigen oder Firmenwebseiten, die im Cockpit angezeigt werden (etwa in einer Rückfrage), erscheinen immer optisch als Zitat mit Quellenvermerk „ungeprüfter externer Text“, nie als eigene Aussage des Agenten. Das folgt Anthropics eigener Leitlinie gegen Prompt-Injection, wonach Inhalte aus Web-Recherche als nicht vertrauenswürdig zu kennzeichnen sind ([Mitigate Jailbreaks](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)) – eine manipulierte Stellenanzeige soll im Cockpit nicht wie eine bereits geprüfte Tatsache wirken.

### 14.2 Entscheidung: MVP-Kanal vs. v1-Web-App

| Option | Zweck | Aufwand | Kosten | Bewertung |
|---|---|---|---|---|
| Telegram-Bot (python-telegram-bot) | Push, Freigeben/Ablehnen/Später/Kommentar per Button | gering (1–2 Tage) | kostenlos | MVP-Primärkanal |
| FastAPI + htmx-Detailseite | Diff, Checkliste, PDF-Vorschau, Statistik | gering–mittel | kostenlos | MVP-Ergänzung |
| Markdown/PDF im Git-Ordner + `git diff` | Diff und Freigabe rein über Commits/PRs | gering, aber PDFs sind binär nicht diffbar | kostenlos | verworfen für MVP |
| Claude Artifact als Cockpit | DB-, Datei- und Kommentar-Fähigkeiten ohne eigenes Hosting | sehr gering | in Claude-Nutzung enthalten | Experiment, kein Produktionsfundament |
| Notion-Spiegelung / lokale Web-App mit Kanban, Kalender, Dashboard | vollwertige Oberfläche über mehrere Ansichten | mittel–hoch | Notion freemium / eigene Web-App kostenlos | v1 |

**Entscheidung:** Der MVP nutzt einen Telegram-Bot als primären Push- und Freigabekanal, ergänzt um eine schlanke lokale FastAPI+htmx-Seite auf demselben Server wie das Agent SDK (Kapitel 7) für die vertiefte Prüfung (Diff, Checkliste, Seitenvorschau, Statistik). v1 baut daraus eine vollwertige lokale Web-App mit Kanban-Ansicht über alle Pipeline-Zustände und einer Statistikseite; eine optionale Notion-Spiegelung bleibt eine spätere Erweiterung, kein Bestandteil von v1 selbst.

**Begründung:** python-telegram-bot ist aktiv gepflegt (29.500 Sterne, 3.229 Commits, Stand 9.9.2026), asynchron und passt zur Python-Basis des Agent SDK ([GitHub](https://github.com/python-telegram-bot/python-telegram-bot)); ein Bot-Token ist in Minuten eingerichtet, liefert native mobile Push-Benachrichtigungen und Datei-Anhänge, ohne dass eine eigene App gebaut werden muss. Für die vertiefte Prüfung reicht Telegram nicht: Ein Wort-Diff über einen ganzen Lebenslauf oder eine mehrseitige PDF-Vorschau passt nicht sinnvoll in eine Chat-Nachricht. FastAPI ist mit 102.200 Sternen die am weitesten verbreitete, MIT-lizenzierte Basis für einen solchen Endpunkt ([GitHub](https://github.com/fastapi/fastapi)); htmx (49.400 Sterne) erlaubt Teil-Updates einer Seite über einfache HTML-Attribute, ohne ein SPA-Framework zu benötigen ([GitHub](https://github.com/bigskysoftware/htmx)) – die Lizenzangabe ließ sich per Abruf nicht eindeutig aus dem Repository extrahieren und ist vor Einsatz direkt in der LICENSE-Datei zu prüfen. Reines Markdown/PDF im Git-Ordner scheitert an einem technischen Fakt: PDFs sind binär und werden von Git nicht sinnvoll als Diff dargestellt; ohne parallele Versionierung der Textquelle wäre die geforderte Diff-Ansicht wertlos. Ein Claude Artifact ist auffällig günstig umzusetzen, weil es Datenbank-, Datei- und Kommentar-Fähigkeiten ohne eigenes Hosting mitbringt, ist aber laut dieser Recherche ein sich weiterentwickelnder Produktbereich ohne dokumentierte Stabilitätsgarantie und wird deshalb nur als Experiment behandelt, nicht als Produktionsschicht neben der SQLite-Datenbank.

**Alternativen:** NiceGUI (16.200 Sterne, MIT) und Reflex (28.900 Sterne, Apache-2.0) wären fertige Python-UI-Baukästen mit weniger eigenem HTML/CSS, aber mehr Lernaufwand für ein Ein-Personen-MVP ([NiceGUI](https://github.com/zauberzeug/nicegui), [Reflex](https://github.com/reflex-dev/reflex)). Streamlit (45.700 Sterne) ist ebenso naheliegend, aber sein Rerun-Modell – das gesamte Skript läuft bei jeder Interaktion neu – wird unhandlich, sobald zehn Zeilen je mehrere unabhängige Buttons (Freigeben/Ablehnen/Kommentar) tragen ([Streamlit](https://github.com/streamlit/streamlit)); zudem nimmt das Projekt aktuell keine externen Pull Requests mehr an. GitHub Issues/PRs als Diff- und Kommentar-Kanal ist für einen Git-affinen Nutzer attraktiv und würde zur ohnehin geplanten Repo-Versionierung passen, verschiebt aber die Prüfoberfläche in ein Werkzeug, das nicht für PDF-Vorschau gebaut ist; als v1-Option offen, nicht MVP.

### 14.3 Funktionsliste

1. Sofort-Push pro Stelle, sobald der Status „bereit zur Freigabe“ erreicht ist, plus ein Tagesdigest am Ende des Tageslaufs mit allen offenen Punkten.
2. Pro Stelle: Score und „Warum diese Stelle“-Begründung des Matchers (Kapitel 9), Kurzvorschau des Anschreibens, PDF-Anhang.
3. Automatisch vorausgefüllte Checkliste (14.4) mit Sprungmarken zu unsicheren Punkten.
4. Wort-Diff des Lebenslaufs, Master-Fassung gegen angepasste Fassung (14.5).
5. Seitenvorschau (PNG) aller erzeugten Dokumente, aus dem Setzer übernommen (Kapitel 13.10).
6. Rückfrage-Kanal mit Auswahloptionen und asynchroner Antwort (14.6).
7. Vier getrennte Aktionen: Freigeben, Ändern mit Kommentar, Ablehnen mit Grund, Später (14.7).
8. Unveränderliches Audit-Log jeder Aktion (14.8).
9. Statistik-Ansicht über Kennzahlen der laufenden Pipeline (14.9).
10. Bedienung von unterwegs über die Telegram-App; responsive Detailseite für die vertiefte Prüfung vom Telefon aus.

### 14.4 Die Review-Checkliste

Jede Bewerbung im Zustand „bereit zur Freigabe“ trägt eine Checkliste aus 20 Punkten in sieben Gruppen. Die meisten Punkte sind bereits durch vorgelagerte Module automatisch geprüft und grün oder rot markiert; das Cockpit fasst diese Ergebnisse nur zusammen und verlinkt auf die Fundstelle. Nur wenige Punkte – vor allem in der Gruppe Ton – verlangen ein tatsächliches menschliches Urteil, weil kein vorgelagertes Modul „liest sich das wie ich“ zuverlässig beantworten kann.

| Gruppe | Prüfpunkt | Herkunft |
|---|---|---|
| Fakten | Jede Aussage im Anschreiben trägt eine Quellenmarkierung (CV-Fakt / Stellenanzeige / Recherche-Fakt); unmarkierte Sätze sind rot hervorgehoben | automatisch (Autor, Kapitel 11) |
| Fakten | Zahlen, Zeiträume und Titel stimmen mit dem Master-Lebenslauf und der Story-Bank überein | automatisch (Kritiker, Kapitel 11) |
| Fakten | Keine Tätigkeit, kein Titel, kein Datum wurde gegenüber dem Master-Lebenslauf erweitert oder übertrieben | automatisch (Kritiker, Kapitel 11) |
| Adressat | Firmenname, Anschrift und Ansprechpartner stimmen mit dem Rechercheur-Datensatz überein, Konfidenz über der Schwelle | automatisch (Rechercheur, Kapitel 10) |
| Adressat | Anredeform im Anschriftfeld (Akkusativ) und in der Anrede (Nominativ) sind konsistent und korrekt | automatisch (Setzer-QA, Kapitel 13.10) |
| Adressat | Betreffzeile enthält Stellentitel und Kennziffer korrekt aus der Stellenanzeige | automatisch (Setzer-QA, Kapitel 13.10) |
| Dokumente | Layout und Sprache (sachlich/klassisch/international-en, de/en) passen zu Stellenanzeige und Branche | manuell (Du) |
| Dokumente | Lebenslauf-Diff zeigt nur Umordnung, Betonung und Formulierung, keine neuen Fakten | manuell (Du, mit Diff aus 14.5) |
| Dokumente | Seitenzahlen eingehalten, Setzer-QA vollständig grün | automatisch (Setzer, Kapitel 13.10) |
| Ton | Stimmprofil eingehalten, keine Anti-Generik-Verstöße, Text liest sich wie du selbst | manuell (Du) |
| Ton | Keine unbelegte Selbsteinschätzung („hochmotiviert“, „Teamplayer“) ohne Beleg aus der Story-Bank | manuell (Du) |
| ATS | ATS-Prüfer-Bericht ist grün: Keyword-Abdeckung und Test-Parsing bestanden | automatisch (ATS-Prüfer, Kapitel 12) |
| ATS | Textebene, Schrifteinbettung, kein Text in Tabellen oder Bildern | automatisch (Setzer-QA, Kapitel 13.10) |
| Anhänge | Ausgewählte Zeugnisse und Zertifikate sind die für diese Stelle relevanten, Reihenfolge korrekt | automatisch (Setzer-Regel, Kapitel 13.8) |
| Anhänge | Gesamtgröße innerhalb des Budgets für den gewählten Bewerbungsweg | automatisch (Setzer-QA, Kapitel 13.10) |
| Anhänge | Keine veraltete oder falsche Anlage versehentlich mitgeschickt | manuell (Du) |
| Versandweg | Bewerbungsweg (E-Mail, Portal getrennt, Portal einzeln, Freitext) korrekt erkannt und Formularfelder passend zugeordnet | automatisch (Rechercheur/Bote, Kapitel 10/15) |
| Versandweg | E-Mail- oder Portal-Entwurf stimmt inhaltlich mit dem Anschreiben überein | manuell (Du) |
| Versandweg | Sonderanforderungen aus der Stellenanzeige erfüllt (Gehaltsangabe, geforderte Anlagen, Sprache) | automatisch (Rechercheur, Kapitel 10) |
| Versandweg | Bei Portalen: Testbefüllung des Formulars ohne Absenden liegt als Nachweis vor | automatisch (Bote, Kapitel 15) |

Rot markierte oder fehlende automatische Prüfpunkte verhindern nicht den Aufruf im Cockpit, aber sie werden optisch hervorgehoben und öffnen standardmäßig aufgeklappt. Die manuellen Punkte sind bewusst wenige: Wenn Ton-Prüfung dauernd zu Ablehnungen führt, ist das ein Signal an Kapitel 11, das Stimmprofil zu schärfen – nicht ein Grund, mehr automatische Prüfpunkte zu erfinden, die am Ende doch wieder Vertrauensfragen an ein Modell wären.

### 14.5 Diff-Ansicht Lebenslauf

Für Anschreiben und Lebenslauf ist ein Wort-Diff dem üblichen zeilenweisen Diff aus der Softwareentwicklung vorzuziehen: In kurzen Prosa-Absätzen ändern sich meist einzelne Wörter oder Halbsätze, nicht ganze Zeilen; ein Zeilen-Diff würde jede kleine Änderung als großen Block darstellen. Die Detailseite berechnet den Diff serverseitig aus zwei Textversionen – Master-Lebenslauf (Kapitel 8) gegen die für diese Stelle angepasste Fassung – mit `difflib.SequenceMatcher` auf Wortebene (Python-Standardbibliothek, keine Zusatzabhängigkeit) und rendert das Ergebnis im Track-Changes-Stil: neu eingefügte Wörter grün und unterstrichen, entfernte Wörter rot und durchgestrichen, unverändertes Umfeld normal. Angezeigt wird der Diff je Lebenslauf-Eintrag (Position, Zeitraum, Aufzählungspunkte), nicht als ein einziger Fließtextblock, damit erkennbar bleibt, welcher Werdegangs-Eintrag wie stark umformuliert wurde.

Die Diff-Ansicht beantwortet die Frage, die die Fakten-Checkliste stellt, aber nicht selbst prüfen kann: Wurde nur umgeordnet und betont (erlaubt, Kapitel 11) oder ist etwas Neues aufgetaucht, das im Master-Lebenslauf nicht steht (nicht erlaubt)? Ein serverseitig berechneter Diff ist hier bewusst der Standard, damit die Prüfung nicht von einem weiteren Modellaufruf abhängt, der selbst wieder falsch liegen könnte.

### 14.6 Rückfrage-Kanal

Rückfragen entstehen in Kapitel 10 (Rechercheur, z. B. bei niedriger Konfidenz zum Ansprechpartner) und Kapitel 11 (Autor, z. B. bei fehlender Information für einen Pflichttext), nicht im Cockpit selbst; das Cockpit ist der Kanal, über den eine Rückfrage angezeigt und beantwortet wird. Jede Rückfrage ist ein eigener Datensatz:

```json
{
  "rueckfrage_id": "rf-4711",
  "bewerbung_id": "a1b2c3",
  "gestellt_von": "Rechercheur",
  "gestellt_am": "2026-09-08T06:40:00+02:00",
  "frage": "Für dieses Portal ist eine Gehaltsvorstellung Pflichtfeld. Welchen Betrag?",
  "kontext_zitat": "Feld „Gehaltswunsch p.a.“ – Pflichtangabe laut Bewerbungsformular",
  "optionen": [
    "Bandbreite aus dem Profil verwenden (65.000–72.000 €)",
    "Anderen Betrag angeben",
    "Feld mit „nach Vereinbarung“ versuchen"
  ],
  "erlaubt_freitext": true,
  "status": "offen",
  "antwort": null,
  "beantwortet_am": null,
  "faellig_bis": "2026-09-15"
}
```

Ist die Zahl der Optionen klein (in der Praxis bis zu vier bis fünf), erscheinen sie im Telegram-Kanal als Inline-Buttons; ist Freitext sinnvoll oder nötig, bietet der Bot zusätzlich eine Antwort per Textnachricht an, die der Bot dem offenen `rueckfrage_id` zuordnet. Die Antwort fließt strukturiert zurück in die Pipeline – an das Modul, das die Rückfrage gestellt hat – statt einen kompletten Neustart der Stelle auszulösen.

**Timeout-Verhalten:** Eine offene Rückfrage läuft nie in eine automatische Entscheidung. Nach vier Stunden ohne Antwort erinnert der Bot einmalig; solange die Rückfrage offen ist, erscheint sie täglich im Tagesdigest. Bleibt sie länger als eine konfigurierbare Frist (Default fünf Werktage) unbeantwortet, wechselt die betroffene Stelle auf „archiviert“ mit dem Grund „Rückfrage nicht beantwortet, Frist überschritten“ – nicht auf „gesendet“ und nicht auf eine automatisch gewählte Option. Diese Regel ist eine direkte Konsequenz aus 14.1: Verstreichen ist niemals gleich Zustimmung.

### 14.7 Aktionen

Vier Aktionen, niemals eine binäre Freigabe/Ablehnung-Entscheidung:

1. **Freigeben.** Setzt den Status auf „freigegeben“, schreibt Akteur und Zeitstempel. Ein Undo-Fenster von 60 Sekunden (Default, siehe 14.12) zeigt einen „Rückgängig“-Button; erst danach wird die Stelle für den Boten (Kapitel 15) zur Abholung sichtbar. Dieser Statuswechsel ist zugleich das Signal, das laut Kapitel 7 die Freigabesperre vor dem Versand-Werkzeug öffnet – das Cockpit setzt den Haken, der Bote führt aus.
2. **Ändern mit Kommentar.** Freitext, was geändert werden soll (z. B. „dritter Absatz zu förmlich, bitte direkter“). Die Stelle geht zurück auf den Status „geschrieben“ – die Recherche bleibt gültig, nur Autor und Kritiker (Kapitel 11) laufen mit dem Kommentar als zusätzlichem Eingabetext erneut, danach automatisch wieder Setzer (Kapitel 13) und ATS-Prüfer (Kapitel 12). Die neue Version erscheint als eigener Eintrag in `render/v{n+1}` (Kapitel 13.11) wieder im Cockpit.
3. **Ablehnen mit Grund.** Eine Kategorie aus einer festen Liste (z. B. „Rolle passt nicht“, „Unternehmen ausschließen“, „Ton nicht passend“, „Sonstiges“) plus optionaler Freitext. Status wechselt auf „archiviert“. Der Grund wird strukturiert erfasst und ist das Feedback-Signal für die Kalibrierung des Matchers (Kapitel 9) und, bei Ton-Gründen, für das Stimmprofil (Kapitel 8/11) – das Lernen selbst findet in jenen Kapiteln statt, das Cockpit liefert nur das saubere, kategorisierte Signal.
4. **Später.** Kein Statuswechsel, sondern ein Feld `zurückgestellt_bis` (Datum) auf dem bestehenden Zustand „bereit zur Freigabe“. Die Stelle verschwindet bis zu diesem Datum aus dem aktiven Digest, ohne die Prüfung der übrigen Stellen aufzuhalten, und taucht danach automatisch wieder auf.

### 14.8 Benachrichtigungen und Mobile

Primärkanal ist der Telegram-Bot: Sofort-Push pro Stelle bei Erreichen von „bereit zur Freigabe“, zusätzlich ein Tagesdigest mit allen offenen Rückfragen und zurückgestellten Stellen. Als Rückfallebene bei ausgefallenem oder verpasstem Bot-Kanal dient ein tägliches E-Mail-Digest mit Zusammenfassung und Link zur Detailseite – Telegram als einziger interaktiver Kanal wäre ein Single Point of Failure. Mobile Bedienung ist über die Telegram-App bereits abgedeckt (native Push, Buttons, Datei-Vorschau); die FastAPI+htmx-Detailseite bekommt ein einfaches, mobilfreundliches Einspalten-Layout für den Fall, dass eine vertiefte Prüfung unterwegs nötig ist, ist aber primär für Desktop/Tablet gestaltet, wo Diff und Seitenvorschau nebeneinander Platz haben.

### 14.9 Audit-Log

Jede Statusänderung – Freigeben, Ändern, Ablehnen, Später, Rückfrage-Antwort – landet als eigener Eintrag in einer Protokolltabelle:

```sql
CREATE TABLE audit_log (
  id INTEGER PRIMARY KEY,
  bewerbung_id TEXT NOT NULL,
  zeitstempel TEXT NOT NULL,      -- ISO 8601, Europe/Berlin
  akteur TEXT NOT NULL,           -- 'Nutzer' oder Modulname
  aktion TEXT NOT NULL,           -- freigeben | aendern | ablehnen | spaeter | rueckfrage_antwort
  alter_status TEXT,
  neuer_status TEXT,
  kommentar TEXT,
  kanal TEXT NOT NULL             -- telegram | web | api
);
```

SQLite kennt keine tabellenweisen Schreibrechte wie Postgres; Unveränderlichkeit wird deshalb auf Anwendungsebene erzwungen (kein UPDATE- oder DELETE-Codepfad für diese Tabelle) und zusätzlich durch einen periodischen Export in das private Git-Repository (Kapitel 13.11) abgesichert – die Git-Historie selbst wird damit zur zweiten, unabhängigen Unveränderlichkeitsschicht. Das Audit-Log ist der Nachweis der menschlichen Freigabe vor jedem Versand und damit direkt an die Compliance-Anforderungen aus Kapitel 16 angebunden.

### 14.10 Statistik-Ansicht

Bei der geplanten Größenordnung (rund zehn Bewerbungen am Tag, einige hundert im Monat) reicht einfache SQL-Aggregation auf der bestehenden SQLite-Datenbank aus, ein eigenes BI-Tool wäre unverhältnismäßig. Die Statistikseite der Detailseite zeigt: Bewerbungen pro Woche nach Status, Time-to-Review (Median der Zeit zwischen „bereit zur Freigabe“ und „freigegeben“, aus dem Audit-Log berechnet), Rückmeldequote (Anteil „Rückmeldung“/„Interview“ an „gesendet“, aus Kapitel 15 gespiegelt), sowie eine Verteilung der Ablehnungsgründe aus 14.7 – letztere ist das direkte Kalibrierungssignal für Matcher und Autor. In der MVP-Phase genügen Tabellen und einfache Balken aus reinem HTML/CSS; v1 kann bei Bedarf ein kleines eingebettetes Diagramm ergänzen.

### 14.11 Wireframes

Telegram-Nachricht pro Stelle:

```text
┌─────────────────────────────────────────────┐
│ Score 87  ·  Senior Controller (m/w/d)       │
│ Beispiel GmbH · München                      │
│                                               │
│ Warum diese Stelle: SAP FI/CO seit 2021,     │
│ Branche passt, Gehaltsband stimmt.           │
│                                               │
│ Anschreiben (Anfang):                        │
│ „Mit über vier Jahren Erfahrung im…"         │
│                                               │
│ Anhang: Mueller_Anna_Bewerbung.pdf, 3 Seiten │
│                                               │
│ [ Freigeben ] [ Kommentar ]                  │
│ [ Ablehnen  ] [ Später   ]                   │
│ [ Details (Diff, Checkliste) → Link ]        │
└─────────────────────────────────────────────┘
```

Detailseite (FastAPI + htmx), Desktop-Layout:

```text
┌───────────────────────────────────────────────────────────────┐
│ Beispiel GmbH – Senior Controller (m/w/d)   Status: bereit     │
│                                              zur Freigabe       │
├────────────────────────────────┬──────────────────────────────┤
│ CHECKLISTE (17/20 automatisch)  │ SEITENVORSCHAU                │
│ Fakten      [x][x][x]           │  ┌─────────┐  ┌─────────┐     │
│ Adressat    [x][x][x]           │  │ Seite 1 │  │ Seite 2 │     │
│ Dokumente   [x][ ][x] ← Du      │  │  PNG    │  │  PNG    │     │
│ Ton         [ ][ ]     ← Du     │  └─────────┘  └─────────┘     │
│ ATS         [x][x]              │                                │
│ Anhänge     [x][x][ ] ← Du      │                                │
│ Versandweg  [x][ ][x][x] ← Du   │                                │
├────────────────────────────────┴──────────────────────────────┤
│ LEBENSLAUF-DIFF (Wort-Diff, Master vs. angepasst)               │
│  Berufserfahrung › Beispiel AG                                  │
│  Verantwortlich für Reporting → Konzernreporting und            │
│  Budgetplanung für zwei Standorte […]                           │
├──────────────────────────────────────────────────────────────────┤
│ RÜCKFRAGEN: 0 offen                                                │
├──────────────────────────────────────────────────────────────────┤
│ [ Freigeben ]  [ Ändern mit Kommentar ]  [ Ablehnen ]  [ Später ]  │
└──────────────────────────────────────────────────────────────────┘
```

### 14.12 Datenfluss zum Tracker

Review-Cockpit und Bote/Tracker (Kapitel 15) teilen sich dieselbe SQLite-Datenbank, schreiben aber in disjunkte Felder, damit keine Wettlaufsituation entsteht:

| Feld | Schreibt | Liest |
|---|---|---|
| `status` (bis „freigegeben“) | Review-Cockpit | Orchestrator, Bote |
| `freigegeben_am`, `freigegeben_von` | Review-Cockpit | Bote, Tracker, Statistik |
| `zurückgestellt_bis` | Review-Cockpit | Orchestrator (Digest-Filter) |
| `status` (ab „gesendet“), Versandkanal, Nachweis | Bote (Kapitel 15) | Review-Cockpit (nur Anzeige) |
| `rückmeldung_status`, Interviewtermin | Tracker (Kapitel 15) | Review-Cockpit (Anzeige, Statistik) |
| `audit_log` | Review-Cockpit (Freigeben/Ändern/Ablehnen/Später/Rückfrage) und Bote/Tracker (Sende- und Rückmeldeereignisse) | alle Module, Statistik |

Das Cockpit kennt damit den weiteren Werdegang einer Bewerbung nur lesend – die Statistikseite zeigt Rückmeldequoten, greift aber nie selbst in den Versand- oder Nachfassprozess ein. Das hält die Verantwortung klar getrennt: Freigabe ist ein menschlicher Akt im Cockpit, alles danach ist Sache des Boten.

### 14.13 Default-Annahmen dieses Moduls

Bis du anders entscheidest (Fragenkatalog Kapitel 22): MVP-Kanal ist ein Telegram-Bot plus lokale FastAPI+htmx-Detailseite, keine Notion- oder GitHub-Spiegelung im MVP; Undo-Fenster nach Freigeben 60 Sekunden; unbeantwortete Rückfragen werden nach fünf Werktagen automatisch archiviert statt gesendet; Sofort-Push pro Stelle plus tägliches Digest, kein reiner Fest-Termin; keine Mehrnutzer-Freigabe, ausschließlich Einzelnutzer-Zugang; Statistik als einfache SQL-Aggregation ohne separates BI-Tool.

**Quellen dieses Kapitels:**

- python-telegram-bot (GitHub) – https://github.com/python-telegram-bot/python-telegram-bot
- FastAPI (GitHub) – https://github.com/fastapi/fastapi
- htmx (GitHub) – https://github.com/bigskysoftware/htmx
- NiceGUI (GitHub) – https://github.com/zauberzeug/nicegui
- Reflex (GitHub) – https://github.com/reflex-dev/reflex
- Streamlit (GitHub) – https://github.com/streamlit/streamlit
- Anthropic: Mitigate Jailbreaks (untrusted content policy) – https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks
