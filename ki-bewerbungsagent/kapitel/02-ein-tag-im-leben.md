## 2. So arbeitet der Bewerbungsagent: ein Tag im Leben

Dieses Kapitel erzählt einen Werktag des Bewerbungsagenten von der ersten Quellenabfrage in der Nacht bis zur Verarbeitung der ersten Antworten am Abend. Die Uhrzeiten sind die Zeitplan-Defaults aus Kapitel 7 (Zeitzone Europe/Berlin), die Komponentennamen und Statusbegriffe die aus dem Styleguide. Erzählt wird der Zielzustand v1, in dem der Bote nach Freigabe selbst versendet; wo der MVP davon abweicht, steht es am Ende in Abschnitt 2.12. Das durchgespielte Beispiel ist erfunden: Die „Beispiel GmbH“ existiert nicht, die Stelle „Projektleiter:in Intralogistik (m/w/d)“ ist ein Platzhalter für eine Rolle, die zu deinem Profil passt (Beruf, Branche und Zielrollen kennen wir nicht; Kapitel 22). Das Beispiel knüpft bewusst an das Briefing-Beispiel in Kapitel 11 an, damit dieselbe fiktive Bewerbung durch das ganze Dokument läuft.

### 2.1 Der Tag auf einen Blick

| Zeit | Lauf | Komponenten | Ergebnis | Status danach |
|---|---|---|---|---|
| 03:30 | Tageslauf, Teil 1 | Scout, Matcher (Vorauswahl) | 200–500 Rohtreffer, dedupliziert, gefiltert; Judge-Batch eingereicht | entdeckt → dedupliziert |
| 05:30 | Tageslauf, Teil 2 | Matcher (Judge-Ergebnis, Tagesauswahl) | Top 10 mit Begründung | bewertet → ausgewählt |
| 05:40 | Tageslauf, Teil 2 | Rechercheur (max. 3 parallel) | Dossier je Stelle mit Konfidenzen, offene Rückfragen | recherchiert |
| 06:05 | Tageslauf, Teil 2 | Autor, Kritiker, ATS-Prüfer | Anschreiben, Lebenslauf-Variante, Claims, Rubrik-Urteil, Keyword-Abgleich | geschrieben → geprüft |
| 06:35 | Tageslauf, Teil 2 | Setzer, ATS-Prüfer (Test-Parsing) | PDF/DOCX, Bewerbungsmappe, QA-Bericht, Review-Items | bereit zur Freigabe |
| 06:45 | Tageslauf, Teil 2 | Orchestrator | Benachrichtigung „10 Bewerbungen bereit“ | – |
| 07:00–09:30 (Di–Do) | Versandlauf | Bote | freigegebene Bewerbungen mit Zufallsversatz gesendet | freigegeben → gesendet |
| tagsüber | – | Review-Cockpit, du | Freigeben, Ablehnen, Später, Kommentar; Rückfragen beantworten | freigegeben / archiviert / zurück an Autor |
| 14:00–16:00 (Di–Do) | Versandlauf, 2. Fenster | Bote | Freigaben vom Vormittag gesendet | gesendet |
| alle 2 h, 08–20 Uhr | Nachlauf | Tracker | Antworten klassifiziert, Nachfass-Entwürfe, Erinnerungen | Rückmeldung / Interview / Absage |
| dauerhaft | Tracker-Daemon | Tracker | neue Mail per IMAP IDLE → Nachlauf sofort | – |

Die Zahl zehn ist das Tageslimit für die Tagesauswahl und für den Versand (konfigurierbar, Kapitel 7 und 9). An Tagen mit weniger passenden Anzeigen bleibt die Liste kürzer; der Agent füllt sie nicht mit schlechteren Stellen auf.

### 2.2 Nachts, 03:30 Uhr: Scout und Matcher sammeln und sortieren vor

Um 03:30 Uhr startet der Orchestrator den ersten Teil des Tageslaufs mit einer eigenen `run.id` und einem Budget. Der Scout fragt alle konfigurierten Quellen ab (Kapitel 6): die Jobsuche-API der Bundesagentur für Arbeit ([bundesAPI/jobsuche-api](https://github.com/bundesAPI/jobsuche-api)), die ATS-Feeds der Wunscharbeitgeber-Liste, Google for Jobs über einen SERP-Dienst und die Job-Alert-Mails im Postfach. Für die fiktive Beispiel GmbH kommt die Anzeige zweimal herein: einmal über die BA-API, einmal über den Personio-Feed der Firmenkarriereseite. Das ist der Normalfall, nicht die Ausnahme: In gecrawlten Stellenanzeigen liegt der Duplikatanteil laut Textkernel-Forschung bei bis zu 50 bis 80 Prozent ([ACM](https://dl.acm.org/doi/fullHtml/10.1145/3486622.3493928)). Der Scout normalisiert beide Treffer (Extraktion unstrukturierter Felder mit Claude Haiku 4.5), erkennt sie über den Blocking-Schlüssel Firma + Titel + Ort und die MinHash-Ähnlichkeit als eine Stelle, behält die Version mit der Original-Kennziffer der Karriereseite als kanonisch und archiviert die andere als Duplikat.

Vor jeder Bewertung läuft ein Injection- und Scam-Screen (Haiku 4.5, festes Schema): Enthält der Anzeigentext anweisungsartige Passagen, verborgenen Text oder Kontaktaufforderungen über Messenger und Video-Ident? Anzeigentexte gelangen ausschließlich als Werkzeugergebnis mit Herkunftsangabe in den Modellkontext, nie als Nutzer- oder Systemtext; eingebettete Anweisungen gelten als Daten, nicht als Befehle ([Anthropic: Mitigate jailbreaks](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)). Scam-Treffer sind ein harter Ausschluss, Injection-Treffer eine Warnung, die die Anzeige bis zu deiner Freigabe aus der Tagesauswahl nimmt (Kapitel 7.8).

Der Matcher wendet dann die Muss-Filter aus deinen Präferenzen an (Sprache, Pendelzeit oder Remote, Vertragsart, Ausschlusslisten) und zieht per BM25 plus Embedding eine Vorauswahl von 30 bis 50 Anzeigen. Gegen 03:50 Uhr reicht er sie als Message Batch beim Judge ein (Claude Sonnet 5 mit fester Rubrik; der Batch kostet die Hälfte der Tokenpreise und ist meist innerhalb einer Stunde fertig, [Preise](https://platform.claude.com/docs/en/about-claude/pricing)). Danach ist bis 05:30 Uhr Ruhe.

### 2.3 05:30 Uhr: Tagesauswahl und Rechercheur

Teil 2 des Tageslaufs holt das Batch-Ergebnis ab. Jede Anzeige trägt jetzt drei Scores (Passung, Attraktivität, Erfolgschance), eine Kurzbegründung und die Liste unsicherer Felder (Kapitel 9.6). Der Matcher bildet die Top 10 mit Diversitätsregel (höchstens zwei Anzeigen je Arbeitgeber, höchstens zwei Personalvermittler-Verdachtsfälle) und lässt Claude Fable 5.1 eine Rangfolge-Erklärung schreiben, die später im Cockpit steht. Die Beispiel GmbH landet auf Platz 2: „Muss-Kriterien Projektleitung und SAP EWM durch Story-Bank belegt; Lean-Methoden teilweise; Standort Kassel innerhalb der Pendelgrenze; Anzeige 4 Tage alt.“ Status: ausgewählt.

Ab 05:40 Uhr arbeitet der Rechercheur, ein Subagent des Claude Agent SDK mit isoliertem Kontext und ausschließlich lesenden Werkzeugen (WebSearch mit Domain-Allowlist, WebFetch, BA- und Registerabfragen; [Agent SDK: Subagents](https://code.claude.com/docs/en/agent-sdk/subagents)). Höchstens drei Recherchen laufen parallel, damit Websuchen und Budget kalkulierbar bleiben; der Abruf von Impressum und Karriereseite kostet nur Tokens, keine Suchgebühr ([Preise](https://platform.claude.com/docs/en/about-claude/pricing)). Für jede Stelle entsteht ein Dossier mit Konfidenz je Feld:

```json
{
  "stelle_id": "2026-09-08-0142",
  "firma": {"name": "Beispiel GmbH", "rechtsform": "GmbH", "quelle": "Impressum", "konfidenz": 0.95},
  "anschrift": {"wert": "Standort Kassel laut Anzeige; Sitz laut Impressum Fulda",
                "entscheidung": "Anzeige-Standort für Anschrift, Impressum bestätigt Firma",
                "konfidenz": 0.85},
  "ansprechperson": {"wert": "Dr. Yilmaz, Leitung Intralogistik", "quellen": ["Anzeige", "Teamseite"],
                     "anrede": "Sehr geehrte Frau Dr. Yilmaz", "konfidenz": 0.9},
  "ats": {"typ": "Personio", "weg": "portal_einzeln", "quelle": "Karriereseite-HTML"},
  "neuigkeiten": [{"text": "Neues Logistikzentrum Kassel, Inbetriebnahme Q1/2027",
                   "quelle": "{URL der Pressemitteilung}", "datum": "2026-08", "konfidenz": 0.9}],
  "rueckfragen": [],
  "auffaelligkeiten": []
}
```

Zwei Regeln bestimmen, wann der Rechercheur nachfragt statt zu raten (Kapitel 10): Liegt die Konfidenz für Anschrift oder Ansprechperson unter 0,7, oder widersprechen sich Anzeige, Website und Impressum, legt er einen `question`-Eintrag an und setzt das Flag `rueckfrage_offen`; die Bewerbung bleibt im Status „recherchiert“, die übrigen neun laufen weiter. Bei der Beispiel GmbH ist das nicht nötig: Name aus zwei Quellen bestätigt, Standortabweichung erklärt. Bei Bewerbung Nr. 7 des Tages dagegen nennt die Anzeige keine Ansprechperson, die Karriereseite zwei mögliche Namen ohne Funktion. Der Rechercheur rät nicht, sondern schlägt vor: namentliche Anrede an Person A (Konfidenz 0,5), an Person B (0,4) oder „Sehr geehrtes Team Recruiting der {Firma}“ (sicher). Eine falsche Anrede gilt in der deutschen Bewerbungsberatung als einer der häufigsten K.-o.-Fehler (Praxiswissen aus der Recherche, nicht unabhängig belegt); lieber korrekt-neutral als persönlich-falsch. LinkedIn und XING fragt der Rechercheur nie automatisiert ab (Nutzungsbedingungen, Kapitel 16); er liefert dir bei Bedarf einen vorbereiteten Suchlink für den manuellen Blick.

### 2.4 06:05 Uhr: Autor, Kritiker und ATS-Prüfer

Für die neun Bewerbungen ohne offene Rückfrage beginnt die Schreibschleife aus Kapitel 11. Zuerst entscheidet der Autor das Format: Die Beispiel GmbH verlangt laut Anzeige ein Anschreiben, also „voll“ (250–350 Wörter, eine Seite); bei zwei anderen Stellen des Tages ist das Anschreiben optional, dort entsteht ein Kurzprofil statt eines vollen Briefs. Aus Dossier, Anzeigen-Extrakt, den drei bis fünf passendsten Story-Bank-Einträgen und dem Stimmprofil entsteht ein Briefing; Claude Fable 5.1 schreibt daraus drei Varianten mit unterschiedlichem Einstieg (Leistung, Firmenbezug, Motiv) und die Lebenslauf-Variante, in der nur umgeordnet, betont und in der Sprache der Anzeige formuliert wird, nie erfunden.

Der Kritiker prüft in einem frischen Kontext ohne Zugriff auf den Schreib-Prompt (Modellwahl in Kapitel 7 und 11; Anthropics Eval-Leitfaden empfiehlt ein anderes Bewertungs- als Schreibmodell, [Develop tests](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)): zuerst deterministische Prüfungen (Wortzahl, Floskel-Liste, Satzlängen-Varianz), dann die Rubrik je Variante einzeln, dann der Fakten-Check. Der Fakten-Check folgt dem Anthropic-Muster gegen Halluzinationen: jede Behauptung extrahieren, ein wörtliches Zitat aus Story-Bank oder Dossier suchen, ohne Beleg zurückziehen und markieren ([Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)). Im Beispiel sind vier von fünf Claims belegt (Durchlaufzeit minus 18 Prozent aus Story S-004, zwei Standorte, Logistikzentrum 2027 aus dem Dossier, Lieferantenumstellung aus S-007). Der fünfte, „Six-Sigma-Erfahrung“, hat keinen Beleg: Die Anzeige nennt „Six Sigma Green Belt“ als wünschenswert, in der Story-Bank steht nur „Lean-Methoden“. Der Kritiker streicht den Satz, fordert eine Überarbeitung an und legt eine Rückfrage an dich an, weil deine Antwort den Master-Lebenslauf dauerhaft verbessert. Da es ein Kann-Kriterium ist, blockiert die Frage die Bewerbung nicht; sie läuft ohne den Satz weiter.

Nach der Überarbeitung (höchstens zwei Schleifen) folgen Stimm-Check und Leser-Test mit je frischem Kontext nach dem Reader-Testing-Muster des Anthropic-Skills `doc-coauthoring` ([SKILL.md](https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md)): Ein Prüfer, der nur Anzeige und Anschreiben kennt, antwortet als Recruiter auf fünf Fragen, darunter „Klingt das nach Textbaustein?“ und „Würde ich nach 30 Sekunden weiterlesen?“. Parallel gleicht der ATS-Prüfer die Keywords der Anzeige mit Lebenslauf und Anschreiben ab (Haiku 4.5 extrahiert, Code vergleicht) und meldet Lücken als Vorschlag, nicht als Auftrag zum Keyword-Stuffing (Kapitel 12). Bewerbung Nr. 5 des Tages fällt hier durch: Der Kritiker bleibt nach zwei Runden unter der Schwelle. Sie geht mit dem Kritikbericht als Fall „Kritiker unzufrieden“ ins Cockpit; du entscheidest, ob du selbst formulierst, eine dritte Runde freigibst oder die Stelle verwirfst.

### 2.5 06:35 Uhr: Setzer

Der Setzer ist reiner Code ohne Modell. Er rendert aus dem Bewerbungs-JSON das Anschreiben nach DIN 5008 und den einspaltigen, ATS-sicheren Lebenslauf als PDF und DOCX, fügt Zeugnisse an, benennt die Dateien nach dem Schema `Nachname_Rufname_Dokumenttyp.pdf` ohne Umlaute, Leerzeichen und Datum ([tabellarischer-lebenslauf.net](https://www.tabellarischer-lebenslauf.net/bewerbung-tipps/dateinamen-der-bewerbungsdokumente/)) und baut die Bewerbungsmappe je nach Bewerbungsweg: eine Gesamt-PDF für E-Mail (Ziel bis 3 MB, hart 5 MB, [stratag](https://www.stratag.de/bewerbung-dateigroesse)) oder Einzeldateien für ein Portal mit getrennten Feldern. Für die Beispiel GmbH (Personio, ein Upload-Feld „Lebenslauf“ plus Freitextfeld) entstehen Lebenslauf-PDF und `anschreiben.txt`. Danach prüft der ATS-Prüfer die fertigen Dateien: Textextraktion mit pdftotext gegen den Quelltext, eingebettete Schriften, Umlaute, Reihenfolge (Kapitel 12 und 13). Erst wenn das besteht, legt der Setzer je Bewerbung ein `review_item` mit Checkliste und Diff-Verweis an und setzt den Status „bereit zur Freigabe“. Die Dateien liegen versioniert im Datenrepository; die spätere Freigabe friert sie per SHA-256 ein.

### 2.6 06:45 Uhr: die Benachrichtigung

Der Orchestrator schließt den Lauf ab, schreibt Kosten und Dauer ins `run`-Protokoll und schickt eine Nachricht über den Telegram-Bot ([python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot)), parallel als E-Mail-Digest an dein Postfach, damit ein Ausfall des Bots dich nicht abhängt:

```text
Bewerbungsagent, Di 08.09.2026, Tageslauf 05:30 abgeschlossen (61 min)
Bereit zur Freigabe: 8 | Rückfrage offen (blockierend): 1 | Kritiker unzufrieden: 1
Zurückgestellt mit Warnung: 1 (Nachrücker A, Injection-Muster im Anzeigentext)
Gescannt 412, neu 173, nach Filtern 38, Tagesauswahl 10
Kosten dieses Laufs: {spent_usd} USD von {budget_usd} USD (Schätzung, Kapitel 18)

1. Beispiel GmbH, Projektleiter:in Intralogistik (Kassel), Score 78, Personio-Portal
2. … (9 weitere Zeilen)

Rückfragen: [Nr. 7: Ansprechperson wählen] [Nr. 2: Six Sigma Green Belt? (nicht blockierend)]
Label: Nr. 4 Personalvermittler (Verdacht)
Cockpit öffnen: https://{cockpit}/tag/2026-09-08
```

Die Rückfragen kommen als eigene Telegram-Nachrichten mit Antwort-Buttons, damit du sie unterwegs in Sekunden erledigen kannst, ohne das Cockpit zu öffnen.

### 2.7 Tagsüber: was du im Review-Cockpit siehst

Das Review-Cockpit ist eine schlanke Web-App (FastAPI mit HTMX, [FastAPI](https://github.com/fastapi/fastapi)) direkt auf der SQLite-Datenbank; der Telegram-Bot ist der Kanal für Push, Rückfragen und Schnellfreigaben, das Cockpit der Ort für die genaue Prüfung (Kapitel 14). Die Tagesliste zeigt je Bewerbung Firma, Titel, Score mit Begründung, Bewerbungsweg, offene Punkte und die Warnlabels. Die Detailansicht einer Bewerbung hat vier Bereiche: das gerenderte Anschreiben mit Quellen-Markierung je Aussage, den Wort-Diff zwischen Master-Lebenslauf und Variante, die PDF-Vorschau und die Checkliste. Für die Beispiel GmbH sieht die Checkliste so aus:

| Prüfpunkt | Was das Cockpit zeigt | Herkunft, Konfidenz | Stand |
|---|---|---|---|
| Firma, Rechtsform, Anschrift | Beispiel GmbH, Kassel (Anzeige), Sitz Fulda (Impressum) | Rechercheur, 0,85 | bestätigt |
| Ansprechperson und Anrede | „Sehr geehrte Frau Dr. Yilmaz“ | Anzeige + Teamseite, 0,9 | bestätigt |
| Alle Aussagen belegt | 4 von 4 Claims mit Story-ID bzw. Dossier-Quelle; 1 gestrichen | Kritiker Fakten-Check | bestanden |
| Recherchierte Details im Text | Logistikzentrum Kassel Q1/2027 | Dossier, 0,9, Quelle verlinkt | 1 (Ziel ≥ 1–2) |
| Format und Länge | voll, 318 Wörter, 1 Seite | Autor, deterministisch | bestanden |
| Rubrik und Leser-Test | 4,3 von 5; Recruiter-Eindruck „konkret, kein Textbaustein“ | Kritiker | bestanden |
| Stimm-Check | 2 Sätze mit Abweichung vom Stimmprofil, markiert | Kritiker | bitte lesen |
| Keyword-Abgleich | 9 von 11 Muss-Begriffen im Lebenslauf; fehlend: „Green Belt“ (Kann) | ATS-Prüfer | Hinweis |
| Test-Parsing der PDF | Text vollständig extrahierbar, Schriften eingebettet | ATS-Prüfer | bestanden |
| Bewerbungsweg | Personio-Formular, Co-Pilot am eigenen Rechner | Rechercheur | Hinweis |
| Gehalt, Eintrittstermin | Anzeige fragt Eintrittstermin; Wert aus Profil: 01.01.2027 | Kandidatenprofil | bestätigt |
| Warnungen | keine (kein Scam, keine Injection, kein Personalvermittler) | Scout, Matcher | – |
| Rückfrage offen | Six Sigma Green Belt (Kann-Kriterium, nicht blockierend) | Kritiker | offen |

Die Quellen-Markierung ist der Kern der Checkliste: Jeder Satz mit einer Zahl, einem Umfang, einem Firmenfakt oder einer Kenntnis trägt seine Quelle (Story-ID, Dossier-Feld, Profilfeld); ein Satz ohne Quelle wäre farblich als „unbelegt“ markiert, kommt aber wegen des harten Gates im Kritiker gar nicht bis hierher. So prüfst du nicht den ganzen Text auf Erfindungen, sondern liest die zwei markierten Stimm-Sätze und die Stelle mit dem Firmenfakt. Die Zielzeit aus Kapitel 1 sind unter zehn Minuten je Bewerbung.

Unter der Checkliste stehen vier getrennte Aktionen, keine binäre Freigabe: **Freigeben**, **Ablehnen** (mit Grund, geht in die Lernschleife), **Später** (bleibt ohne Zeitdruck in der Warteschlange) und **Kommentar** (Überarbeitungsauftrag an Autor und Kritiker). Ein Zeitablauf zählt nie als Freigabe. Du kannst den Anschreibentext auch direkt im Cockpit ändern; der Kritiker prüft die Änderung dann noch einmal auf Belegbarkeit, bevor der Setzer neu rendert.

**Entscheidung:** Freigeben löst den Versand nicht sofort aus, sondern startet ein Undo-Fenster von 60 Sekunden, in dem die Freigabe im Cockpit oder per Telegram zurückgenommen werden kann. **Begründung:** Ein Fehlklick auf dem Handy ist wahrscheinlicher als ein Fehler, der genau in dieser Minute auffällt; das Fenster kostet nichts, weil der Bote ohnehin auf das Versandfenster wartet. **Alternative:** Kein Undo, dafür ein zweiter Bestätigungsdialog; das verlangsamt jede der zehn Freigaben, statt nur den seltenen Fehlklick abzufangen.

### 2.8 Rückfragen: wo der Agent fragt statt zu raten

Rückfragen entstehen an festen Stellen der Pipeline, immer strukturiert, immer mit Antwortvorschlägen und Konfidenz, immer mit Ablaufdatum (Tabelle `question`, Kapitel 7):

| Auslöser | Fragt | Beispiel | Blockiert |
|---|---|---|---|
| Ansprechperson oder Anschrift unter Konfidenz 0,7; Widerspruch Anzeige/Impressum/Website | Rechercheur | „Anrede an Person A, Person B oder Team?“ | ja, nur diese Bewerbung |
| Anzeige nennt Standort, Firma hat mehrere; unklar, wo die Stelle sitzt | Rechercheur | „Anschrift Kassel oder Fulda?“ | ja |
| Ansprechperson laut Karriereseite nicht mehr im Unternehmen; Anzeige über 30 Tage offen | Rechercheur | „Trotzdem bewerben?“ | ja |
| Anzeige verlangt Kenntnis, die im Profil fehlt | Kritiker | „Hast du einen Six Sigma Green Belt?“ | nein bei Kann, ja bei Muss |
| Anzeige fragt Gehalt, Profilfeld leer | Autor | „Spanne für diese Stelle?“ (Vorschlag aus Entgeltatlas, als Schätzung markiert) | ja |
| Personalvermittler-Verdacht oder Ghost-Job-Band bei mittlerer Konfidenz | Matcher | „Direktstelle oder Vermittler?“ | nein, Label im Cockpit |
| Du-Anzeige, Profil ohne Präferenz | Autor | „Duzen wie die Anzeige oder Sie?“ | ja |
| Antwortmail unklar klassifiziert | Tracker | „Absage oder Rückfrage?“ | nein |

Die Frage zur Ansprechperson (Bewerbung Nr. 7) sieht in der Datenbank und in Telegram so aus:

```json
{
  "id": "q-2026-09-08-07",
  "application_id": "2026-09-08-0147",
  "asked_by": "rechercheur",
  "text": "Keine Ansprechperson in der Anzeige. Karriereseite nennt zwei Namen ohne Funktion. Wie anreden?",
  "options": [
    {"wert": "Sehr geehrte Frau A. Beispielmann", "konfidenz": 0.5, "quelle": "{URL Teamseite}"},
    {"wert": "Sehr geehrter Herr B. Mustermann", "konfidenz": 0.4, "quelle": "{URL Teamseite}"},
    {"wert": "Sehr geehrtes Team Recruiting der Beispiel-Zwei GmbH", "konfidenz": 0.95, "quelle": "Fallback"}
  ],
  "blocks_status": "geschrieben",
  "default_bei_ablauf": "Sehr geehrtes Team Recruiting der Beispiel-Zwei GmbH",
  "expires_at": "2026-09-09T03:30:00+02:00",
  "status": "offen"
}
```

Antwortest du um 08:20 Uhr per Button, setzt der Orchestrator die Bewerbung sofort fort: Autor, Kritiker, ATS-Prüfer und Setzer laufen für diese eine Stelle nach, und um etwa 08:35 Uhr ist sie „bereit zur Freigabe“. Antwortest du nicht, erinnert der Bot einmalig nach vier Stunden und danach täglich im Tagesdigest. Diese Frage hat einen Default (die Team-Anrede, Konfidenz 0,95): Bleibt sie unbeantwortet, löst der nächste Tageslauf sie automatisch mit diesem Default auf und markiert die Stelle zur Kontrolle in der Review-Checkliste. Rückfragen ohne Default – etwa ein Adresskonflikt zwischen Anzeige und Impressum – bleiben dagegen offen, bis du antwortest oder die Bewerbung verwirfst, und werden erst nach fünf Werktagen archiviert. Ein Timeout ist nie eine Freigabe.

**Entscheidung:** Rückfragen blockieren nur die betroffene Bewerbung, nie den Tageslauf. **Begründung:** Eine unklare Anrede darf nicht neun fertige Bewerbungen aufhalten; die Wartezeit auf deine Antwort ist der teuerste Engpass des Systems. **Alternative:** Alle Rückfragen vor dem Schreiben sammeln und den Lauf pausieren; das wäre einfacher zu bauen, würde aber jeden Morgen von deiner Reaktionszeit abhängen.

### 2.9 Freigabe, Kommentar und Versand

Um 08:10 Uhr öffnest du das Cockpit. Sechs der acht fertigen Bewerbungen gibst du nach der Checkliste direkt frei, darunter die Beispiel GmbH. Bei Nr. 3 schreibst du einen Kommentar: „Zweiter Absatz zu förmlich, die Zahl aus S-011 nach vorn.“ Der Kommentar geht als Überarbeitungsauftrag an Autor und Kritiker (Schleife ab Schritt 4 in Kapitel 11); zehn Minuten später liegt die neue Version mit Wort-Diff zur alten im Cockpit, und du gibst sie frei. Nr. 4 lehnst du ab (Personalvermittler; der Grund wird für die Lernschleife gespeichert), den zurückgestellten Nachrücker mit der Injection-Warnung verwirfst du, Nr. 5 („Kritiker unzufrieden“) legst du auf „Später“. Nr. 7 ist nach deiner Antwort auf die Anrede-Frage um 08:35 Uhr fertig und wird die achte Freigabe des Tages.

Mit dem Klick auf Freigeben friert der Orchestrator Empfängeradresse, Betreff und die SHA-256-Hashes der finalen Dateien ein und setzt nach Ablauf des Undo-Fensters den Status „freigegeben“. Der Bote, reiner Code ohne Modell, prüft vor jedem Versand vier Bedingungen im Code: Status „freigegeben“, Freigabezeitpunkt nach dem letzten Render, Hash-Gleichheit, Undo-Fenster abgelaufen (Kapitel 7.8). Ein `canUseTool`-Gate des Agent SDK blockiert jeden Sendeaufruf, der nicht aus dieser Prüfung kommt ([Agent SDK: Permissions](https://code.claude.com/docs/en/agent-sdk/permissions)). Der Versand selbst passiert nicht sofort, sondern im Versandfenster: Dienstag bis Donnerstag 07:00–09:30 Uhr, jede Mail mit einem Zufallsversatz von 15 bis 40 Minuten, damit kein Cron-Muster erkennbar ist. Das Fenster folgt einer HR-Ratgeberheuristik zum frühen Versand vor der Hauptwelle gegen 11 Uhr ([arwa.de](https://arwa.de/de/blog/wann-sollte-man-eine-bewerbung-abschicken); Konfidenz niedrig, keine Studie). Die Bewerbungen, die du um 08:10 Uhr freigibst, verlassen das Postfach noch im selben Fenster, spätestens gegen 09:30 Uhr; was du später am Vormittag freigibst, geht im zweiten Fenster 14:00–16:00 Uhr.

**Entscheidung:** Das zweite Versandfenster 14:00–16:00 Uhr ist als Default aktiv, damit Freigaben vom Vormittag noch am selben Tag hinausgehen. **Begründung:** Bei Review-Zeiten am späten Vormittag würde sonst jede Bewerbung 24 Stunden liegen, Donnerstags-Freigaben bis Dienstag. **Alternative:** Nur das Morgenfenster; das ist die reinere Form der Heuristik, kostet aber Reaktionszeit auf frische Anzeigen (offene Frage, Abschnitt 2.13).

Für die Beispiel GmbH ist der Weg ein anderer: Die Stelle läuft über ein Personio-Formular. Der Bote schickt keine E-Mail, sondern startet den Co-Pilot-Modus (Kapitel 15.5): ein sichtbares Browserfenster auf deinem Rechner, in dem das Formular mit Standardantworten aus dem Kandidatenprofil, dem Lebenslauf-Upload und dem Anschreiben im Freitextfeld vorausgefüllt ist. Die DSGVO-Einwilligung und den Klick auf „Bewerbung absenden“ machst du selbst; danach speichert der Bote den Screenshot der Bestätigungsseite und die Referenznummer. Die E-Mail-Bewerbungen des Tages gehen als eine Mail je Firma hinaus: Betreff „Bewerbung als {Position} – {Kennziffer}“, das Anschreiben als Mailtext oder kurzer Begleittext, genau eine Mappe als Anhang, keine Lesebestätigung, kein Tracking-Pixel (Tracking senkt Zustellbarkeit und wirkt unseriös, [Instantly](https://instantly.ai/blog/email-tracking-and-deliverability-why-tracking-pixels-can-hurt-your-inbox-placement/)). Jeder Versand steht mit Zeitstempel, Empfängerdomain, Betreff-Hash und eigener Message-ID im Sendeprotokoll (Kapitel 15.3).

### 2.10 Der Nachlauf: Antworten, Termine, Nachfassen

Der Tracker-Daemon hält per IMAP IDLE eine Verbindung zum Bewerbungspostfach ([imap_tools](https://github.com/ikvk/imap_tools)); jede neue Mail löst sofort einen Nachlauf aus, zusätzlich läuft er alle zwei Stunden zwischen 08 und 20 Uhr. Eingehende Mails werden über die Header Message-ID, In-Reply-To und References der Bewerbung zugeordnet und klassifiziert (Kapitel 15.6):

- 09:12 Uhr: Autoresponder „Ihre Bewerbung ist eingegangen“ von einer Firma des Vortags. Status bleibt „gesendet“, nur ein Protokolleintrag.
- 11:40 Uhr: Absage einer Bewerbung von vor zwei Wochen. Status „Absage“, Grund wird, falls genannt, gespeichert; du bekommst eine kurze Telegram-Zeile, keine Aufforderung.
- 15:05 Uhr: Einladung zum Gespräch mit ICS-Anhang. Der Termin wird nicht vom Modell aus dem Freitext geraten, sondern aus dem `text/calendar`-Teil geparst ([icalendar](https://pypi.org/project/icalendar)); Status „Interview“, Termin im Tracker, `.ics` zum Import. Du bestätigst per Button.
- 16:30 Uhr: Rückfrage einer Firma nach Gehaltsvorstellung. Status „Rückmeldung“ mit `rueckfrage_offen`; der Tracker legt eine Antwort-Vorlage an, die denselben Freigabeweg nimmt wie eine Bewerbung. Der Agent antwortet nie selbst.
- Unklare Fälle unter der Konfidenzschwelle landen als Frage bei dir, nicht als Status.

Außerdem prüft der Nachlauf, welche Bewerbungen seit N Werktagen ohne Rückmeldung sind (Default 10 Werktage; Kapitel 15.7), und legt dafür einen kurzen Nachfass-Entwurf an, der über den References-Header an die Bewerbung anknüpft. Auch er wird nur nach deiner Freigabe gesendet; ein zweites Nachfassen gibt es nicht automatisch. Um 22:00 Uhr am Sonntag läuft die Wartung: Backup, Archivierung nach Fristen, Löschkonzept für Ansprechpartnerdaten abgeschlossener Bewerbungen (Kapitel 16), Wochenstatistik und Kostenabgleich.

### 2.11 Wenn etwas nicht klappt

- **Budget erreicht.** Erreicht `spent_usd` das Lauf-Budget, bricht der Orchestrator ab, markiert den Lauf als `budget_erreicht`, und die unbearbeiteten Bewerbungen bleiben in ihrem Status; sie kommen am nächsten Morgen zuerst dran. Du bekommst die Zahl in der Benachrichtigung.
- **Quelle ausgefallen.** Antwortet die BA-API oder ein Feed nicht, läuft der Scout mit den übrigen Quellen weiter und meldet die Lücke; die inoffizielle BA-API hat keine SLA (Kapitel 6).
- **Zugangsdaten ungültig.** Apple widerruft App-spezifische Passwörter bei jeder Änderung des Apple-ID-Passworts; der Gmail-Refresh-Token läuft im OAuth-Testing-Modus nach 7 Tagen ab (Kapitel 15.4). Der Bote erkennt den Auth-Fehler, versucht es nicht endlos, und du bekommst eine konkrete Aufforderung („neues App-Passwort anlegen“).
- **Batch nicht fertig.** Liegt das Judge-Ergebnis um 05:30 Uhr nicht vor, bewertet der Matcher die 40 besten Kandidaten der Vorauswahl synchron; der Rest wird am Folgetag nachgezogen.
- **Kein Treffer.** An Tagen ohne Anzeige über der Schwelle sagt die Benachrichtigung genau das. Der Agent senkt die Schwelle nicht selbst.
- **Wochenende.** Der Quellenabruf läuft täglich, damit am Montag nichts fehlt; Tagesauswahl und Schreibschleife laufen Montag bis Freitag (Default, offene Frage).

### 2.12 Was im MVP anders ist

| Schritt | v1 (dieses Kapitel) | MVP (Kapitel 19) |
|---|---|---|
| Versand | Bote sendet per SMTP im Versandfenster nach Freigabe und Undo-Fenster | Bote legt die fertige Mail samt Anhang als Entwurf im Postfach ab (IMAP-APPEND, bei Gmail Drafts-API); du klickst in deiner Mail-App auf Senden ([iCloud Mail](https://support.apple.com/en-us/102198), [Gmail Drafts](https://developers.google.com/workspace/gmail/api/guides/drafts)) |
| Portale | Co-Pilot-Modus mit vorausgefülltem Formular | Bote liefert Link, Standardantworten und Dateien; du füllst selbst aus |
| Benachrichtigung | Telegram-Bot plus E-Mail-Digest | Telegram-Bot oder nur E-Mail-Digest (offene Frage) |
| Nachfassen | Entwurf automatisch angelegt, Freigabe nötig | nur Erinnerung „Nachfassen fällig“ |
| Lernschleife | Ablehnungsgründe justieren wöchentlich die Scoring-Gewichte (mit Freigabe) | Gründe werden nur gesammelt |

Der Tagesablauf, die Checkliste und die Rückfragen sind in beiden Stufen gleich; nur der letzte Meter zur Außenwelt ist im MVP kürzer und liegt vollständig bei dir.

### 2.13 Default-Annahmen und offene Fragen (für Kapitel 22)

- Versandtage: Dienstag bis Donnerstag in zwei Fenstern (07:00–09:30, 14:00–16:00 Uhr). Auch Montag und Freitag, damit Donnerstags-Freigaben nicht bis Dienstag warten? Default: nein.
- Ablauf offener Rückfragen: Erinnerung nach 4 Stunden, danach täglich im Digest. Rückfragen mit Default (z. B. Anrede-Fallback „Sehr geehrtes Recruiting-Team [Firma]“) löst der nächste Tageslauf automatisch mit dem Default auf und markiert das in der Review-Checkliste; Rückfragen ohne Default (z. B. Adresskonflikt) bleiben offen und werden nach 5 Werktagen archiviert. Ein Timeout ist nie eine Freigabe.
- Undo-Fenster nach Freigeben: Default 60 Sekunden.
- Benachrichtigung: Telegram-Bot plus E-Mail-Digest, oder reicht der Digest? Default: beides.
- Nachfassen: Default 10 Werktage ohne Rückmeldung, branchenunabhängig.
- Ansprechperson unter Konfidenz 0,7: blockierende Rückfrage (Default) oder neutraler Team-Fallback mit Kennzeichnung im Cockpit?
- Review-Zeitpunkt: Der Ablauf nimmt an, dass du morgens zwischen 07:00 und 10:00 Uhr 30 bis 60 Minuten für zehn Bewerbungen hast. Default: morgens; Alternative Benachrichtigung am Abend.

**Quellen dieses Kapitels:**

- bundesAPI/jobsuche-api (inoffizielle Jobsuche-API der Bundesagentur für Arbeit): https://github.com/bundesAPI/jobsuche-api
- Textkernel-Forschung zu Duplikaten in gecrawlten Stellenanzeigen (ACM): https://dl.acm.org/doi/fullHtml/10.1145/3486622.3493928
- Anthropic: Mitigate jailbreaks and prompt injections: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks
- Anthropic: Claude API Pricing (Batch-Rabatt, Web Fetch ohne Zusatzgebühr): https://platform.claude.com/docs/en/about-claude/pricing
- Claude Agent SDK: Subagents: https://code.claude.com/docs/en/agent-sdk/subagents
- Claude Agent SDK: Permissions (canUseTool): https://code.claude.com/docs/en/agent-sdk/permissions
- Anthropic: Develop tests (LLM-Grading mit anderem Modell): https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
- Anthropic: Reduce hallucinations: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
- Anthropic Skill doc-coauthoring (Reader-Testing-Muster): https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md
- Dateinamen für Bewerbungsdokumente: https://www.tabellarischer-lebenslauf.net/bewerbung-tipps/dateinamen-der-bewerbungsdokumente/
- Dateigröße von Bewerbungsanhängen (stratag): https://www.stratag.de/bewerbung-dateigroesse
- python-telegram-bot: https://github.com/python-telegram-bot/python-telegram-bot
- FastAPI: https://github.com/fastapi/fastapi
- Wann sollte man eine Bewerbung abschicken (arwa.de, Heuristik): https://arwa.de/de/blog/wann-sollte-man-eine-bewerbung-abschicken
- Instantly: Tracking-Pixel und Zustellbarkeit: https://instantly.ai/blog/email-tracking-and-deliverability-why-tracking-pixels-can-hurt-your-inbox-placement/
- imap_tools (IMAP IDLE): https://github.com/ikvk/imap_tools
- icalendar (PyPI): https://pypi.org/project/icalendar
- Apple: iCloud Mail Limits (Support 102198): https://support.apple.com/en-us/102198
- Gmail API: Drafts: https://developers.google.com/workspace/gmail/api/guides/drafts
