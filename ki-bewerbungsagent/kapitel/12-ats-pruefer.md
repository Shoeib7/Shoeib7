## 12. Modul ATS-Prüfer: Keyword-Abgleich, Formatregeln, Test-Parsing

Der ATS-Prüfer ist die letzte automatische Instanz vor dem Review-Cockpit. Er stellt zwei Fragen, die weder der Kritiker noch der Setzer stellen: Würde eine Maschine, die den Text buchstäblich nach Begriffen durchsucht, die richtigen Wörter finden? Und liest sich die tatsächlich erzeugte Datei – nicht der Entwurf, sondern die Bytes, die verschickt werden – so, wie ein Parser sie liest? Der Kritiker (Kapitel 11) urteilt semantisch mit einem Sprachmodell, ob eine Aussage gut begründet und ehrlich ist. Der ATS-Prüfer prüft überwiegend deterministisch, ob die Begriffe buchstäblich vorhanden sind und ob die Datei technisch intakt bleibt. Das sind unterschiedliche Fehlerarten: Ein Anschreiben kann rhetorisch überzeugend und trotzdem für einen literalen Keyword-Abgleich unsichtbar sein, weil der entscheidende Begriff nur umschrieben vorkommt; ein Lebenslauf kann inhaltlich korrekt und trotzdem technisch kaputt sein, weil eine Schriftart beim Rendern nicht eingebettet wurde. Beide Fehlerarten fallen erst beim Empfänger auf, wenn niemand sie vorher testet.

### 12.1 Aufgabe, Schnittstellen, zwei Prüfstufen

Der ATS-Prüfer arbeitet in zwei Stufen, die an unterschiedlichen Stellen der Pipeline sitzen:

- **Stufe 1 (Text, vor dem Rendern).** Läuft auf dem Bewerbungsdossier im Status „geschrieben“, unmittelbar nach der Kritiker-Rubrik (Kapitel 11.8, Schritt 3) und vor dem Übergang nach „geprüft“ – also auf reinem Text, noch bevor der Setzer eine Zeile PDF oder DOCX erzeugt. Sie prüft Begriffsabdeckung und Stuffing-Regeln (12.2–12.4) und ist billig: Code plus ein kurzer Modellaufruf. Der Übergang „geschrieben“ → „geprüft“ wird erst gesetzt, wenn Kritiker-Rubrik **und** ATS-Prüfer Stufe 1 bestanden sind (Kapitel 7.4); ein Fehlschlag hier bedeutet, dass der Autor nachbessert, bevor der Setzer überhaupt zu tun bekommt – das entspricht dem Pfeil „ATS-PRÜFER → SETZER“ im Komponentendiagramm (Kapitel 7.2).
- **Stufe 2 (Datei, nach dem Rendern).** Läuft auf den tatsächlich erzeugten Dateien im Status „geprüft“, nachdem die Setzer-QA (Kapitel 13.10) grün ist. Sie testet Formatregeln und Test-Parsing (12.5–12.7) auf den echten Bytes. Erst wenn Setzer-QA **und** ATS-Prüfer-Stufe-2 grün sind, wechselt die Stelle von „geprüft“ auf „bereit zur Freigabe“ (Kapitel 13.1).

Beide Stufen schreiben ihre Befunde in denselben ATS-Report (12.8), der im Review-Cockpit neben dem Kritikbericht angezeigt wird (Kapitel 14). Scheitert Stufe 1 an einem harten Kriterium, geht die Stelle mit der Liste der fehlenden Begriffe zurück an den Autor – in die bestehende Überarbeitungsschleife aus Kapitel 11.8, nicht als eigene, dritte Schleife. Scheitert Stufe 2 an einem Formatfehler, der plausibel am Rendern liegt (z. B. eine Tabelle im Template), geht die Stelle an die Vorlagenpflege des Setzers zurück, nicht an den Autor – dieselbe Unterscheidung, die der Setzer bei seiner eigenen QA schon trifft (Kapitel 13.10).

Warum ein eigenes Modul und keine Erweiterung des Kritikers: Der Kritiker bewertet mit einem Sprachmodell und frischem Kontext, ob ein Text gut, belegt und in der richtigen Stimme geschrieben ist (Rubrik K1–K7, Kapitel 11.9). Der ATS-Prüfer bewertet mehrheitlich mit Code, ob ein Text und eine Datei bestimmte harte, mechanische Kriterien erfüllen, die mit Textqualität nichts zu tun haben – eine hervorragende, ehrliche Bewerbung kann trotzdem an einer nicht eingebetteten Schriftart oder einem im Fließtext fehlenden Pflichtbegriff scheitern. Zwei getrennte Module mit unterschiedlicher Fehlerlogik sind robuster als ein Modul, das beides gleichzeitig prüfen soll.

### 12.2 Stufe 1: Begriffe aus der Anzeige, feiner als Muss/Kann

Der Matcher (Kapitel 9) klassifiziert ganze Anforderungssätze der Anzeige in `must_haves` und `nice_to_haves` – das ist die Ebene für Scoring und Auswahl. Der ATS-Prüfer braucht eine feinere Ebene: die einzelnen Begriffe innerhalb dieser Sätze, weil genau das die Einheit ist, in der ein literaler Keyword-Abgleich (oder ein überfliegender Mensch) tatsächlich sucht. Aus „mind. 5 Jahre Erfahrung mit SAP EWM und Lean-Methoden, SixSigma-Zertifikat von Vorteil“ entstehen die Begriffe „SAP EWM“ (Werkzeug, Muss), „Lean“ (Methode, Muss) und „Six Sigma“ (Zertifikat, Kann) – jeweils mit Typ (Werkzeug, Zertifikat, Rollentitel, Methode, Sprache) und Herkunft (Muss/Kann-ID aus Kapitel 9).

**Entscheidung:** Diese Begriffsextraktion läuft als eigener, kurzer Aufruf mit Claude Structured Output auf `must_haves` + `nice_to_haves` + dem Rohtext der Anzeige, nicht als Erweiterung des Matcher-Schemas aus Kapitel 9. **Begründung:** Das Matcher-Schema ist auf Scoring und Deduplizierung optimiert und soll dort schlank bleiben (Kapitel 9.2); die Begriffsliste ist ein eigener Verwendungszweck mit eigenem Format und wird nur für Stellen gebraucht, die den Status „geprüft“ erreichen – eine Minderheit aller gescannten Anzeigen (Kapitel 9.5). **Alternative:** Ein gemeinsames Schema für beide Zwecke – spart einen Aufruf, koppelt aber zwei Module, die aus unterschiedlichen Gründen ihr Schema ändern.

Für Synonyme greift der ATS-Prüfer zuerst auf die `synonyme`-Tabelle des Kandidatenprofils zurück, die der Autor beim Lebenslauf-Tailoring bereits nutzt (Kapitel 11.7, Beispiel „Excel“ ↔ „MS Excel“) – dieselbe Quelle, kein zweites Mapping. Fehlt ein Begriff dort, prüft der ATS-Prüfer zusätzlich gegen ESCO (kostenlose, mehrsprachige EU-Taxonomie für Skills und Berufe, REST- und Lokal-API) ([ESCO](https://esco.ec.europa.eu/en/use-esco/download)), die bereits als Skill-Taxonomie für den Matcher vorgesehen ist (Kapitel 9). Ein Begriff ohne Treffer in Profil-Synonymen oder ESCO gilt als exakter String – keine Synonymsuche, kein Rateraum.

```json
{
  "stelle_id": "2026-09-08-0142",
  "begriffe": [
    {"text": "SAP EWM", "typ": "werkzeug", "herkunft": "must_haves", "muss_kann": "muss",
     "synonyme": ["SAP Extended Warehouse Management"], "quelle_synonym": "profil"},
    {"text": "Lean", "typ": "methode", "herkunft": "must_haves", "muss_kann": "muss",
     "synonyme": ["Lean Management", "Lean-Methoden"], "quelle_synonym": "esco"},
    {"text": "Six Sigma", "typ": "zertifikat", "herkunft": "nice_to_haves", "muss_kann": "kann",
     "synonyme": ["Six Sigma Green Belt", "6 Sigma"], "quelle_synonym": "profil"},
    {"text": "Projektleiter Intralogistik", "typ": "rollentitel", "herkunft": "title_raw", "muss_kann": "titel",
     "synonyme": [], "quelle_synonym": null}
  ]
}
```

### 12.3 Abdeckungsquote: Abgleich mit Lebenslauf und Anschreiben

Der Abgleich selbst läuft deterministisch, ohne Modellaufruf: normalisierter Text (Kleinschreibung, Umlaute erhalten, Satzzeichen entfernt) aus dem angepassten Lebenslauf und dem Anschreiben wird gegen jeden Begriff aus 12.2 sowie seine Synonyme geprüft. Weil das Parsing laut Kapitel 4.2 überwiegend den Lebenslauf betrifft, zählt ein Treffer im Lebenslauf als „belegt für den Parser“; ein Treffer nur im Anschreiben zählt separat als „belegt für den Menschen“, aber nicht für die Parser-Abdeckung.

Abdeckungsquote je Kategorie:

| Kategorie | Formel | Ziel | Bei Unterschreitung |
|---|---|---|---|
| Muss-Begriffe im Lebenslauf | erkannte Muss-Begriffe ÷ alle Muss-Begriffe | 100 % | hartes Gate → zurück an Autor mit Liste |
| Kann-Begriffe im Lebenslauf | erkannte Kann-Begriffe ÷ alle Kann-Begriffe | ≥ 50 % (Richtwert) | informativ, kein Gate |
| Rollentitel | exakter oder anerkannter Alias-Titel im Kurzprofil oder in der aktuellen Station | vorhanden | hartes Gate → zurück an Autor |
| Belegte-nur-im-Anschreiben | Muss-Begriffe, die nur im Anschreiben, nicht im Lebenslauf stehen | 0 | weicher Hinweis: „in Lebenslauf übernehmen“ |

Dass ein Muss-Begriff fehlt, sollte selten vorkommen: Der Matcher hat die Stelle nur ausgewählt, weil der Kandidat die Muss-Kriterien inhaltlich erfüllt (Kapitel 9.5), und der Autor soll sie ohnehin explizit beantworten (Regel R03, Kapitel 4.7). Die Prüfung hier ist ein zweites, unabhängiges Netz mit anderer Fehlerart als der Kritiker: Der Kritiker fragt mit einem Sprachmodell „ist das Kriterium belegt und gut formuliert“ (Rubrik K2, Kapitel 11.9); der ATS-Prüfer fragt ohne Modell „steht der Begriff oder ein anerkanntes Synonym buchstäblich im Text“. Ein Kriterium kann semantisch überzeugend belegt und trotzdem für einen literalen Abgleich unsichtbar sein, wenn der Autor umschrieben hat statt den Begriff zu nennen – genau diesen Fall fängt Stufe 1.

```json
{
  "stufe_1_text": {
    "muss_abdeckung": {"gesamt": 3, "erfuellt": 3, "quote": 1.0, "fehlend": []},
    "kann_abdeckung": {"gesamt": 1, "erfuellt": 1, "quote": 1.0},
    "rollentitel_erkannt": true,
    "nur_im_anschreiben": []
  }
}
```

### 12.4 Regeln gegen Stuffing

Belegte Präsenz ist nicht dasselbe wie Wiederholung. Kapitel 4.4 hält fest, dass keine der dokumentierten KI-Schichten (Joule, SmartAssistant, Teamtailor Co-Pilot) Wörter zählt – Stuffing hilft technisch nicht und wirkt beim lesenden Menschen generisch (Regel V01, Kapitel 4.7). Der Kritiker prüft bereits vor dem Rendern Floskeln und Musterhäufigkeit (Katalog in Kapitel 11.5). Der ATS-Prüfer wiederholt diese Prüfung nach dem Rendern nicht als Redundanz, sondern als letztes Netz gegen Fehler, die erst beim Rendern entstehen können – ein Template-Fehler, der einen Absatz verdoppelt, oder eine Cockpit-Bearbeitung nach der Kritiker-Freigabe.

| Regel | Schwelle | Schwere |
|---|---|---|
| Ein Begriff (inkl. Synonyme) erscheint außerhalb des Kernkompetenzen-Blocks mehr als zweimal in Lebenslauf und Anschreiben zusammen | > 2 | weich |
| Ein Begriff erscheint insgesamt mehr als dreimal | > 3 | hart, zurück an Autor |
| Mehr als vier verschiedene Begriffe stehen in einem Absatz oder einer Bullet-Zeile ohne verbindenden Satz (Begriffscluster) | > 4 in < 25 Wörtern | hart, Signal für Keyword-Block |
| Unsichtbarer Text: Zeichen mit Schriftgröße < 1 pt oder Textfarbe gleich Hintergrundfarbe | jedes Auftreten | hart, siehe unten |

Der letzte Punkt ist die technische Umsetzung der V02-Regel aus Kapitel 4.7 (kein Weißtext, keine versteckten Anweisungen) und läuft auf der gerenderten Datei, nicht auf dem Entwurf – Weißtext kann nur in einer echten PDF-Datei stecken, nicht in einem JSON-Dossier. Das Setzer-Modul nutzt pdfplumber bereits für Umbruch- und Textextraktions-Checks (Kapitel 13.10); der ATS-Prüfer liest mit derselben Bibliothek die Zeichen-Attribute (Schriftgröße, Nichtstrich-Farbe je Buchstabe) aus und markiert jedes verdächtige Zeichen als harten Gate-Verstoß, unabhängig davon, ob es harmlos oder böswillig entstanden ist. Da der Bewerbungsagent selbst nie Weißtext oder Prompt-Injection einsetzt (Leitsatz und Entscheidung in Kapitel 4.4), ist ein Treffer hier immer ein Fehler – Template-Bug, kopierter Formatierungsrest aus einer Fremdquelle oder eine nicht autorisierte Cockpit-Bearbeitung – und wird nie stillschweigend akzeptiert.

### 12.5 Formatregeln als Checkliste

Kapitel 4.7 (Regelkatalog A) und Kapitel 13 (Vorlagenkonzept) legen die inhaltlichen Formatregeln fest; hier werden sie zur automatisch prüfbaren Checkliste. Die Checkliste läuft in Stufe 2 auf der gerenderten Datei, weil erst dort sichtbar wird, ob eine Regel tatsächlich eingehalten wurde – ein Template kann syntaktisch korrekt sein und trotzdem beim Rendern eine Tabelle erzeugen.

| Prüfpunkt | Regel-Herkunft | Werkzeug | Bei Verstoß |
|---|---|---|---|
| Einspaltig, keine Tabelle für Kerninhalte | R05 (Kap. 4) | pdfplumber: Tabellenerkennung liefert 0 Treffer; Textpositionen liegen in einer Spalte | hart, an Vorlagenpflege |
| Standardüberschriften erkannt | R04 (Kap. 4), 13.5 | Zeichenkettenabgleich gegen die feste Liste aus Kapitel 13.5 | hart, an Vorlagenpflege |
| Kontaktdaten im Fließtext, nicht in Kopf-/Fußzeile | R05 (Kap. 4) | Positionsprüfung: Telefonnummer/E-Mail-Regex liegt im Textkörper-Bereich | hart, an Vorlagenpflege |
| PDF mit echtem Text-Layer | R06 (Kap. 4) | bereits durch Setzer-QA „Textextraktion“ geprüft (Kap. 13.10) – hier nur referenziert, nicht neu geprüft | – |
| Schrift vollständig eingebettet | 13.3 | bereits durch Setzer-QA „Schrifteinbettung“ (pdffonts) geprüft (Kap. 13.10) | – |
| Dateiname nach Konvention | 13.6 | bereits durch Setzer-QA geprüft (Kap. 13.10) | – |

Die letzten drei Punkte sind bewusst nicht doppelt implementiert: Der Setzer prüft sie bereits als Teil seiner eigenen QA (Kapitel 13.10), und der ATS-Prüfer liest deren Ergebnis aus derselben `qa.json` statt sie erneut zu berechnen. Was der ATS-Prüfer eigenständig prüft, sind die drei ersten Punkte – Layout-Eigenschaften, die für die Setzer-QA kein Thema sind, weil sie nicht die Integrität der Datei betreffen, sondern die Frage, ob ein fremder Parser sie richtig zerlegt.

### 12.6 Stufe 2: Test-Parsing der fertigen Dateien

Die relevanten deutschen ATS-Anbieter setzen mit Textkernel eine gemeinsame, aber selbst nicht öffentlich testbare Parsing-Engine ein (Personio, softgarden, d.vinci; Kapitel 4.2) – Enterprise-Zugang, keine Selbstbedienungspreise ([Textkernel Parser](https://www.textkernel.com/de/produkte-loesungen/parser/)). Der ATS-Prüfer testet deshalb mit zwei kostenlosen, selbst hostbaren Stellvertretern, die zusammen zwei unterschiedliche Fehlerarten abdecken:

1. **Apache Tika** (Apache-2.0, Version 4.0.0 final seit 18.8.2026) als generischer Text- und Metadaten-Extraktor, self-hosted als `tika-server` ([Apache Tika](https://github.com/apache/tika)). Tika ist absichtlich kein Lebenslauf-spezialisierter Parser, sondern die Art generischer Dokumentextraktion, die auch in vielen Enterprise-Pipelines als erste Stufe steckt. Der ATS-Prüfer schickt jede erzeugte PDF/DOCX-Datei an `tika-server`, vergleicht den extrahierten Text normalisiert gegen den Dossier-Text und prüft auf Lesereihenfolge, korrekte Umlaute/ß und fehlende Abschnitte.
2. **OpenResume-Parser** (AGPL-3.0, PDF.js-basiert, lokal per npm oder Docker unter `localhost:3000`, nur PDF) ([OpenResume](https://github.com/xitanggg/open-resume)) als lebenslauf-spezialisierter Parser, der Name, Kontakt, Stationen (Titel/Firma/Zeitraum) und Kenntnisse in Felder zerlegt – näher an dem, was ein echter CV-Parser versucht, als Tikas reine Textextraktion.

Beide laufen unabhängig von der Setzer-QA (Kapitel 13.10), die mit pdftotext/pdfplumber/pdffonts das eigene Rendern verifiziert: Setzer-QA prüft „hat mein eigenes Werkzeug richtig gerendert“, Tika und OpenResume prüfen „liest ein fremdes Werkzeug die Datei richtig“ – ein Bestätigungsfehler (derselbe Bug, der falsch rendert, würde auch falsch verifizieren) ist damit ausgeschlossen.

| Werkzeug | Testet | Format | Grenze |
|---|---|---|---|
| Apache Tika (self-hosted) | Textextraktion, Lesereihenfolge, Encoding, Metadaten | PDF, DOCX | generisch, keine Lebenslauf-Struktur |
| OpenResume-Parser (lokal) | Namens-, Kontakt-, Stationen-, Kenntnis-Erkennung | nur PDF | keine DOCX-Prüfung, PDF.js-basiert, primär englischsprachig trainiert |
| pyresparser (optional, dritte Meinung) | dieselben Felder wie OpenResume, spaCy/NLTK-basiert | PDF, DOCX | unklarer Wartungsstatus (33 offene Issues), primär Englisch ([pyresparser](https://github.com/OmkarPathak/pyresparser)) |
| Affinda Resume Parser | – | – | **kein Self-Service-Free-Tier** (ab ca. 800 USD/Monat); über Eden AI ab 0,07 USD/Datei zugänglich ([G2](https://www.g2.com/products/resume-parser-by-affinda/pricing), [Eden AI](https://www.edenai.co/post/best-resume-parser-apis)) |
| Jobscan / Resume Worded | – | – | US-/Englisch-fokussiert, kostenpflichtig ab 5 Scans gratis/Monat ([Jobscan-Preise](https://pitchmeai.com/blog/jobscan-pricing-plans)) |

Zur Klarstellung, weil die ursprüngliche Recherche-Annahme hier korrigiert werden muss: Affinda bietet **keinen** kostenlosen Selbstbedienungs-Free-Tier; ein direkter Affinda-Zugang ist für den MVP zu teuer und für ein Einzelnutzer-Projekt unwirtschaftlich. **Entscheidung:** Test-Parsing im MVP ausschließlich mit Apache Tika und OpenResume-Parser, beide selbst gehostet und kostenlos. **Begründung:** Beide sind Open Source, laufen lokal ohne laufende Kosten, und decken zusammen die zwei Hauptfehlerarten ab (generische Textextraktion, lebenslauf-spezifische Feldererkennung); pyresparser bleibt optionale dritte Meinung, Affinda/Jobscan/Resume Worded bleiben außen vor. **Alternative:** Eden AI als bezahlter Pay-per-Datei-Zugang zu mehreren Parsern (u. a. Affinda), falls du nach den ersten Wochen eine dritte, kommerzielle Meinung willst – Kostenordnung Cent-Bereich je Datei, kein Abonnement.

Vergleichslogik gegen den erwarteten Datensatz (acht Pflichtfelder: Name, Telefon, E-Mail, aktueller Titel, aktuelle Firma, höchster Abschluss, mindestens drei Kernkompetenzen, Sprachniveau):

| Erkennungsquote OpenResume-Parser | Bewertung | Aktion |
|---|---|---|
| ≥ 90 % der Pflichtfelder | bestanden | – |
| 70–89 % | Warnung | Hinweis im Review-Cockpit, kein Stopp |
| < 70 % | nicht bestanden | hart, an Vorlagenpflege |

Für DOCX gibt es keine OpenResume-Prüfung (Format-Grenze); dort verlässt sich der ATS-Prüfer auf Tika plus die bereits in Kapitel 13.10 beschriebene LibreOffice-Konvertierungsprüfung (DOCX → PDF, Text-Diff gegen die PDF-Fassung) – eine dokumentierte Lücke, kein verschwiegenes Risiko.

### 12.7 LLM-Kontrolle: „Lies wie ein ATS“

Test-Parsing prüft Struktur, nicht Bedeutung. Ob ein Kriterium für ein semantisches Screening „erkennbar erfüllt“ ist, testet ein kurzer Modellaufruf, der das einzige in dieser Recherche gut dokumentierte LLM-Screening eines europäischen ATS nachbildet: Teamtailor Co-Pilot bewertet jedes frei definierte Kriterium mit Haken, Kreuz oder „unknown“ plus schriftlicher Begründung ([Teamtailor Co-Pilot](https://support.teamtailor.com/en/articles/10209597-co-pilot-candidate-screening)). Der ATS-Prüfer stellt genau diese Frage, bevor ein echtes Co-Pilot-System sie stellen könnte:

Der Prüfer bekommt **nicht** das JSON-Dossier, sondern den von Tika extrahierten Rohtext (12.6) – damit prüft er, was ein Parser tatsächlich sieht, nicht was der Autor gemeint hat – plus die Muss-/Kann-Begriffsliste aus 12.2, ohne Kenntnis von Briefing oder Schreib-Prompt. Für jedes Muss- und Kann-Kriterium liefert er strukturiert Haken, Kreuz oder „unknown“ mit wörtlichem Zitat als Beleg.

```json
{
  "modell": "claude-sonnet-5",
  "kriterien": [
    {"id": "M-1", "text": "mind. 5 Jahre Projektleitung", "status": "erkannt",
     "zitat": "Projektleiterin Logistik, 8 Jahre"},
    {"id": "M-2", "text": "SAP EWM", "status": "erkannt", "zitat": "SAP EWM (fortgeschritten)"},
    {"id": "M-3", "text": "Lean-Methoden", "status": "erkannt", "zitat": "Lean Management, Kanban"},
    {"id": "K-1", "text": "Englisch verhandlungssicher", "status": "unknown",
     "zitat": null, "begruendung": "Sprachniveau steht nicht im extrahierten Text – vermutlich Tabellenformat verloren"}
  ],
  "unknown_bei_muss": 0,
  "unknown_bei_kann": 1
}
```

Ein „unknown“ bei einem Kann-Kriterium ist informativ; ein „unknown“ bei einem Muss-Kriterium ist ein weicher Hinweis (kein hartes Gate, weil das Modell selbst fehleranfällig ist und der Fakten-Check des Kritikers dasselbe Kriterium bereits gegen die Story-Bank geprüft hat – Sonnet 5, Kapitel 11.8 Schritt 5; die semantische Rubrikprüfung K2 läuft davon getrennt auf Opus 5, Kapitel 11.8 Schritt 3) – aber ein Hinweis, der im Review-Cockpit sichtbar bleibt, weil er auf ein Format- statt ein Inhaltsproblem hindeutet (im Beispiel: eine Sprachtabelle, die beim Parsing verloren ging).

### 12.8 Der ATS-Report: Schema, Score, Schwellen

Beide Stufen schreiben in denselben Report. Harte Gates entscheiden allein über „bestanden“/„nicht bestanden“; der gewichtete Score ist ein Priorisierungswert für das Review-Cockpit, kein zusätzliches Freigabekriterium – es gibt keinen Zielwert wie „ATS-Score 100 %“, weil solche Werte für die relevanten deutschen ATS nicht existieren (Regel V03, Kapitel 4.7).

```json
{
  "stelle_id": "2026-09-08-0142",
  "geprueft_version": "render/v2",
  "geprueft_am": "2026-09-08T07:40:00+02:00",
  "stufe_1_text": {
    "muss_abdeckung": {"quote": 1.0, "fehlend": []},
    "kann_abdeckung": {"quote": 1.0},
    "rollentitel_erkannt": true,
    "stuffing_befunde": [],
    "unsichtbarer_text": {"treffer": 0}
  },
  "stufe_2_datei": {
    "tika": {"text_uebereinstimmung": 0.995, "encoding_fehler": 0, "status": "bestanden"},
    "openresume": {"pflichtfelder_quote": 1.0, "status": "bestanden", "hinweis": "nur PDF getestet"},
    "format_checkliste": {"einspaltig": true, "standardueberschriften": true,
                           "kontakt_im_fliesstext": true},
    "llm_lesetest": {"modell": "claude-sonnet-5", "unknown_bei_muss": 0, "unknown_bei_kann": 1}
  },
  "score": {"gewichtet": 0.94, "ampel": "gruen"},
  "gates": {"muss_abdeckung": "ok", "unsichtbarer_text": "ok", "tika": "ok",
            "openresume": "ok", "format": "ok"},
  "gate_status": "bestanden",
  "naechster_schritt": "bereit zur Freigabe"
}
```

Score-Gewichtung (nur informativ, siehe oben): Kann-Abdeckung 20 %, OpenResume-Feldquote 30 %, Formatcheckliste 30 %, LLM-Lesetest ohne „unknown“ 20 %; Ampel grün ab 0,85, gelb 0,60–0,85, rot darunter. Die Gates selbst kennen keine Ampel, nur bestanden/nicht bestanden:

| Gate | Kriterium | Bei Verstoß, Rücksprung an |
|---|---|---|
| Muss-Abdeckung | 100 % der Muss-Begriffe im Lebenslauf | Autor |
| Unsichtbarer Text | 0 Treffer | Autor (bei Textursprung) oder Setzer (bei Template-Ursprung) |
| Tika-Textübereinstimmung | ≥ 98 % normalisierte Wortübereinstimmung | Setzer (Vorlagenpflege) |
| OpenResume-Pflichtfelder | ≥ 90 % (PDF; siehe 12.6-Tabelle für die Zwischenstufe) | Setzer (Vorlagenpflege) |
| Formatcheckliste | alle Punkte erfüllt | Setzer (Vorlagenpflege) |
| Stuffing (hart) | keine Überschreitung | Autor |

Nach zwei erfolglosen Rücksprüngen an dieselbe Komponente – dieselbe Grenze wie beim Kritiker (Kapitel 11.8) – stoppt der Orchestrator und legt den Fall mit vollständigem Report ins Review-Cockpit; du entscheidest dann, ob du selbst nachbesserst, die Schwelle für diesen Einzelfall überschreibst oder die Stelle verwirfst.

### 12.9 Modellwahl

**Entscheidung:** Begriffsextraktion (12.2) und Formatklassifikation mit Claude Haiku 4.5; der „Lies-wie-ein-ATS“-Lesetest (12.7) mit Claude Sonnet 5. **Begründung:** Die Begriffsextraktion ist eine enge, gut spezifizierte Aufgabe auf kurzem, bereits strukturiertem Text – dieselbe Kostenlogik wie beim Matcher, der aus demselben Grund Haiku 4.5 für die Anzeigen-Extraktion nutzt (Kapitel 9.3). Der Lesetest bildet dagegen das am besten dokumentierte LLM-Screening eines europäischen ATS nach (Teamtailor Co-Pilot) und ist die letzte automatische Instanz vor der Freigabe; ein falsches „erkannt“ hier fällt erst beim Empfänger auf, ein zusätzlicher Bruchteil eines US-Cent pro Bewerbung für ein zuverlässigeres Modell ist vertretbar (Kostenrechnung in Kapitel 18). **Alternative:** Haiku 4.5 auch für den Lesetest, wenn sich nach den ersten Bewerbungswellen zeigt, dass die „unknown“-Rate gegenüber Sonnet 5 nicht relevant steigt – dieselbe Kalibrierungslogik wie bei der Kritiker-Rubrik (Kapitel 11.9).

### 12.10 Grenzen: was der ATS-Prüfer nicht garantieren kann

Der Name „ATS-Prüfer“ verspricht mehr, als das Modul halten kann, und das muss hier offen stehen. Kein Werkzeug in diesem Kapitel ist die tatsächliche Ziel-Software: Textkernel (Personio, softgarden, d.vinci), SAP Joule, SmartRecruiters SmartAssistant und Workday HiredScore sind Enterprise-Systeme ohne öffentlichen Testzugang (Kapitel 4.3); Tika und OpenResume-Parser sind kostenlose Stellvertreter, der Lesetest ist eine Nachbildung des einzigen dokumentierten Falls (Teamtailor), nicht ein Test gegen Teamtailor selbst. Ein „bestanden“ im ATS-Report reduziert das Risiko groben Parsing-Versagens (fehlender Text, falsche Reihenfolge, verlorene Kontaktdaten) und literaler Begriffslücken – es sagt nichts über eine bestimmte Platzierung, ein Ranking oder eine Score-Zahl in einem echten System aus, und der Report behauptet das auch nicht.

Das ist keine Schwäche, die ein besseres Werkzeug beheben könnte, sondern die Konsequenz der Marktlage: Kapitel 4.4 zeigt, dass der Mythos „75 % aller Bewerbungen werden automatisch abgelehnt“ widerlegt ist und 92 % der befragten Recruiter bestätigen, dass ihr ATS nicht nach Formatierung oder Match-Score automatisch ablehnt ([The Interview Guys](https://blog.theinterviewguys.com/ats-resume-rejection-myth/), [HR.com](https://www.hr.com/en/app/blog/2026/04/ats-rejection-myth-debunked-92-of-recruiters-confi_mntajhyq.html)). Der ATS-Prüfer optimiert deshalb bewusst nicht auf einen erfundenen Zielwert, sondern auf die zwei Risiken, die tatsächlich belegt sind: technisches Parsing-Versagen und fehlende literale Begriffe für die Minderheit der Systeme, die tatsächlich automatisiert filtern (Knockout-Fragen, Kapitel 4.6). Ebenso ausgeschlossen bleibt jede Form von Weißtext oder Prompt-Injection als „Abkürzung“ – die Studienlage zeigt sinkende Wirksamkeit bei Verbreitung und die Erfahrung, dass 65 % der befragten Hiring Manager bereits KI-gestützte Täuschungsversuche erkannt haben (kein Entdeckungsrisiko pro Einzelfall) ([The Interview Guys](https://blog.theinterviewguys.com/job-seekers-are-hiding-secret-text-in-their-resumes/)); der ATS-Prüfer scannt aktiv danach (12.4), setzt es aber nie selbst ein.

Weil Anbieter ihre KI-Schichten schnell und undokumentiert ändern (Beispiel Personio/aurio, Kapitel 4.3), ist der Regelkatalog dieses Kapitels wie die Regelwerke aus Kapitel 4 ein versioniertes, quartalsweise zu prüfendes Dokument, kein einmal fertiges System.

### 12.11 Default-Annahmen und offene Fragen (für Kapitel 22)

1. **Schwellenwerte:** Die in 12.6 und 12.8 genannten Prozentwerte (90 % OpenResume-Pflichtfelder, 98 % Tika-Textübereinstimmung, 50 % Kann-Abdeckung) sind Startwerte ohne empirische Kalibrierung an deinen echten Dokumenten. Default: wie angegeben übernehmen und nach den ersten 20 Bewerbungen im Review-Cockpit nachjustieren (gleiche Logik wie die Kritiker-Rubrik, Kapitel 11.9). Alternative: strengere oder lockerere Startwerte, falls du das Risiko anders einschätzt.
2. **Dritte Meinung (pyresparser):** Default ist, pyresparser wegen unklarem Wartungsstatus **nicht** einzubinden. Alternative: aufnehmen, wenn dir eine dritte, unabhängige Parser-Meinung wichtiger ist als der Zusatzaufwand.
3. **Kostenpflichtiger Vergleichsmaßstab:** Default ist strikt kostenlos (Tika, OpenResume). Alternative: gelegentlich Eden AI (Cent-Bereich je Datei) für eine kommerzielle Zweitmeinung, falls du das willst.
4. **Modellwahl Lesetest:** Default ist Sonnet 5 für den „Lies-wie-ein-ATS“-Schritt (12.7, 12.9). Alternative: Haiku 4.5 durchgängig, wenn dir die Kostendifferenz wichtiger ist als die zusätzliche Zuverlässigkeit der letzten Instanz vor der Freigabe.

**Quellen dieses Kapitels:**

- ESCO: Download/Portal – https://esco.ec.europa.eu/en/use-esco/download
- Apache Tika: Repository – https://github.com/apache/tika
- Textkernel: Parser (Produktseite) – https://www.textkernel.com/de/produkte-loesungen/parser/
- OpenResume: Repository – https://github.com/xitanggg/open-resume
- pyresparser: Repository – https://github.com/OmkarPathak/pyresparser
- Affinda-Preise (G2) – https://www.g2.com/products/resume-parser-by-affinda/pricing
- Eden AI: Best Resume Parser APIs – https://www.edenai.co/post/best-resume-parser-apis
- Jobscan-Preise (pitchmeai) – https://pitchmeai.com/blog/jobscan-pricing-plans
- Teamtailor Support: Co-Pilot Candidate Screening – https://support.teamtailor.com/en/articles/10209597-co-pilot-candidate-screening
- The Interview Guys: ATS Resume Rejection Myth – https://blog.theinterviewguys.com/ats-resume-rejection-myth/
- HR.com: ATS Rejection Myth Debunked – https://www.hr.com/en/app/blog/2026/04/ats-rejection-myth-debunked-92-of-recruiters-confi_mntajhyq.html
- The Interview Guys: Job Seekers Are Hiding Secret Text – https://blog.theinterviewguys.com/job-seekers-are-hiding-secret-text-in-their-resumes/
