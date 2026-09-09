## 13. Modul Setzer: Dokumentenerzeugung (PDF/DOCX), Vorlagen, DIN 5008, Dateinamen, Bewerbungsmappe

Der Setzer macht aus geprüften Texten und Profildaten die Dateien, die der Empfänger tatsächlich öffnet: Anschreiben, Lebenslauf, Anlagen und – je nach Bewerbungsweg – eine zusammengeführte Bewerbungsmappe. Er ist die einzige Komponente ohne Modellentscheidungen: deterministischer Code, der ein Datenobjekt durch feste Vorlagen rendert, das Ergebnis misst und versioniert ablegt. Alles Inhaltliche ist vorher entschieden (Autor und Kritiker, Kapitel 11); das Test-Parsing gegen ATS-Logik folgt danach (ATS-Prüfer, Kapitel 12). Der Setzer erfindet nichts, kürzt nichts, formuliert nichts um. Passt ein Text nicht auf die Seite, geht er mit Messwert zurück an den Autor.

### 13.1 Aufgabe, Schnittstellen, Grundsätze

Eingaben je Stelle im Status „geprüft“:

- **Bewerbungsdossier** (JSON): Anschreiben in Absätzen, angepasster Lebenslauf in Abschnitten und Einträgen, Betreff, Anrede, Sprache (de/en), Liste der Dokumente (das Anschreiben kann fehlen, wenn Kapitel 11 es für diese Stelle nicht vorsieht).
- **Empfängerdaten** vom Rechercheur (Kapitel 10): Firma, Postanschrift, Ansprechpartner mit Anredeform, Kennziffer – jeweils mit Konfidenz. Fehlt ein Pflichtfeld oder liegt die Konfidenz unter der Schwelle, rendert der Setzer nicht, sondern setzt „Rückfrage offen“.
- **Assets** aus dem Kandidatenprofil (Kapitel 8): vorbereitete Zeugnis-PDFs, optional Foto, optional Unterschriftsbild.
- **Bewerbungsweg** aus Scout/Rechercheur: `email`, `portal_getrennt` (ein Upload-Feld je Dokument), `portal_einzeln` (ein Feld für alles), `freitext` (Anschreiben nur als Textfeld).

Ausgaben: fertige Dateien mit Endnamen, Seitenvorschauen (PNG) für das Review-Cockpit (Kapitel 14), QA-Bericht, Manifest. Erst wenn Setzer-QA und ATS-Prüfer grün sind, wechselt die Stelle auf „bereit zur Freigabe“.

Fünf Grundsätze:

1. **Kein Modell im Renderpfad.** Der Setzer ist ein Werkzeug, im Agent SDK als In-Process-MCP-Tool `render_documents` registriert ([Custom Tools](https://code.claude.com/docs/en/agent-sdk/custom-tools)), das der Orchestrator aufruft. Kosten: Rechenzeit, keine Token. Einzige Ausnahme ist die optionale Sichtprüfung (13.10).
2. **Eine Datenquelle, mehrere Ausgaben.** Dieselbe JSON-Struktur speist PDF, DOCX und Textvariante. Kein Format wird aus einem anderen konvertiert.
3. **Echte Textebene, eingebettete Schriften, lineare Lesereihenfolge** – bei jedem Dokument.
4. **Reproduzierbar:** gleiche Daten, gleiche Vorlagenversion, gleiche Schriftdateien ergeben dieselbe Datei; Zeitstempel in den PDF-Metadaten werden fixiert.
5. **Unveränderlich nach Freigabe:** Was gesendet wurde, liegt byte-identisch und gehasht im Archiv (13.11).

### 13.2 Toolchain-Entscheidung

| Werkzeug | Ausgabe | DIN-5008-Positionierung | DOCX | Lizenz / Kosten | Bewertung |
|---|---|---|---|---|---|
| HTML/CSS + WeasyPrint (Python) | PDF | exakt in mm über CSS Paged Media | nein | Open Source, kostenlos | Primär-Renderer |
| Typst | PDF | exakt, eigenes Template nötig | nein | Apache-2.0 | Alternative für den Lebenslauf (v1) |
| RenderCV (YAML → Typst) | PDF | nur Lebenslauf, Themes international | nein | MIT | Referenz, nicht Kern |
| LaTeX moderncv / Tectonic | PDF | möglich, aufwendig | nein | frei | Fallback |
| python-docx | DOCX | über Absatzabstände, nicht mm-exakt | ja | MIT | DOCX-Renderer |
| Puppeteer/Playwright (Chromium) | PDF | wie HTML/CSS | nein | frei | unnötig, Browser-Overhead |
| docxtemplater / Carbone Cloud | DOCX, PDF | Template in Word | ja | 1.250–9.000 €/Jahr bzw. ab 29 €/Monat | vermeiden |
| Anthropic docx-/pdf-Skill | DOCX, PDF | per Modell + Code Execution | ja | source-available | Werkzeugkasten, nicht Renderpfad |

**Entscheidung:** PDF entsteht aus Jinja2-HTML-Vorlagen mit CSS, gerendert durch WeasyPrint; DOCX entsteht parallel aus derselben JSON-Struktur über python-docx. Beide Renderer teilen sich Datenmodell, Schriftdateien, Seitenränder und Textbausteine. LibreOffice headless dient nur der Prüfung des DOCX (Umwandlung zu PDF für Seitenzählung), nicht der Erzeugung. Typst wird in v1 als Zweitrenderer für den Lebenslauf evaluiert.

**Begründung:**

- WeasyPrint unterstützt CSS Paged Media und positioniert Elemente in Zentimetern; das Anschriftfeld nach DIN 5008 lässt sich damit exakt setzen, ohne Browser ([WeasyPrint](https://weasyprint.com/)). PDF/A- und PDF/UA-Varianten sind vorhanden, aber laut Doku experimentell und nicht garantiert konform ([WeasyPrint-Doku](https://doc.courtbouillon.org/weasyprint/stable/common_use_cases.html)) – für ATS ohnehin irrelevant, weil gängige Parser (Textkernel bei Personio und softgarden) die einfache Textlage lesen ([Personio](https://support.personio.de/hc/en-us/articles/360010193018-CV-parsing-for-candidate-profiles), [softgarden](https://support.softgarden.de/de/articles/680780-cv-parsing)).
- Der Stack ist Python (Agent SDK, Kapitel 7). HTML/CSS ist die Vorlagensprache, die Claude Code am zuverlässigsten schreibt und ändert; Layoutkorrekturen sind CSS-Änderungen, keine Satzsystem-Debugging-Sitzungen.
- DOCX nicht konvertieren, sondern nativ erzeugen: Konvertierung aus HTML/PDF verliert Absatzformate und erzeugt Textrahmen, die Parser stören. python-docx ist in der Claude-Sandbox vorinstalliert ([Cloud-Sandbox-Referenz](https://platform.claude.com/docs/en/managed-agents/cloud-sandboxes-reference)), auf dem eigenen VPS ein `pip install`.
- Typst ist attraktiv (eine Binärdatei, `typst compile`, schnelle Kompilierung, Apache-2.0, [GitHub](https://github.com/typst/typst)), hat aber kein deutsches DIN-5008-Template; die Lebenslauf-Vorlagen im Typst Universe (z. B. [modern-cv](https://typst.app/universe/package/modern-cv/), [brilliant-cv](https://typst.app/universe/package/brilliant-cv/)) sind international ausgerichtet und müssten stark angepasst und versionsgepinnt werden. Zwei Satzsysteme im MVP wären ein zweites Layout-Regelwerk ohne Mehrwert.
- RenderCV (MIT, YAML zu PDF, neun Themes, [GitHub](https://github.com/rendercv/rendercv)) deckt kein Anschreiben ab; Reactive Resume ist eine Web-App ohne dokumentierten Headless-Betrieb ([GitHub](https://github.com/amruthpillai/reactive-resume)). Beide dienen als Referenz für Datenmodell und Typografie.
- Kommerzielle Templating-Engines skalieren preislich nicht für eine Person ([docxtemplater](https://docxtemplater.com/pricing/), [Carbone](https://carbone.io/pricing.html)). Die Anthropic-Skills docx/pdf sind sofort nutzbar, laufen aber über Modellaufrufe plus Code Execution und stehen unter einer source-available-Lizenz, die vor kommerzieller Nutzung geprüft werden muss ([anthropics/skills](https://github.com/anthropics/skills)). Wir nutzen die darunterliegenden Bibliotheken (pypdf, pdfplumber, python-docx) direkt.

**Alternativen:** (a) Typst für alles – gewählt, wenn WeasyPrints Systemabhängigkeiten auf dem Zielserver Probleme machen; DIN-Template dann selbst bauen. (b) LaTeX moderncv mit Tectonic für reproduzierbare Builds ([Tectonic](https://github.com/tectonic-typesetting/tectonic)) – nur für konservative Branchen mit ausdrücklichem LaTeX-Wunsch. (c) Nur DOCX erzeugen und per LibreOffice zu PDF wandeln – eine Vorlage weniger, dafür schlechtere Kontrolle über Umbrüche und Anschriftfeld; verworfen.

### 13.3 Vorlagenkonzept: Daten → Template → PDF + DOCX

Das Datenmodell lehnt sich an JSON Resume an ([jsonresume.org](https://jsonresume.org/)), das seit 2014 stabil ist, aber kein Anschreiben und keine DIN-Adressfelder kennt; beides wird ergänzt. Gekürztes Skelett:

```json
{
  "schema": "bewerbungsdossier/1.0",
  "bewerbung_id": "a1b2c3",
  "sprache": "de",
  "layout": "sachlich",
  "bewerbungsweg": "email",
  "kandidat": {
    "vorname": "Anna", "nachname": "Müller", "rufname_datei": "Anna",
    "strasse": "Musterweg 12", "plz": "80331", "ort": "München",
    "telefon": "+49 170 0000000", "email": "anna.mueller@example.org",
    "profil_url": null, "foto": null, "unterschrift": null
  },
  "empfaenger": {
    "firma": "Beispiel GmbH", "abteilung": "Personalabteilung",
    "ansprechpartner": {"anrede_brief": "Frau", "titel": "Dr.", "vorname": "Lena",
                        "nachname": "Beispiel", "konfidenz": 0.9},
    "strasse": "Industriestraße 5", "plz": "80339", "ort": "München", "land": null,
    "anschrift_konfidenz": 0.95, "vermerk": null
  },
  "anschreiben": {
    "datum": "2026-09-08",
    "betreff": "Bewerbung als Senior Controller (m/w/d), Kennziffer 4711",
    "betreff_zeile2": "Ihre Stellenanzeige auf Ihrer Karriereseite vom 2. September 2026",
    "anrede": "Sehr geehrte Frau Dr. Beispiel,",
    "absaetze": ["…", "…", "…"],
    "gruss": "Mit freundlichen Grüßen",
    "unterschrift_einfuegen": false,
    "anlagen": ["Lebenslauf", "Arbeitszeugnisse", "Masterzeugnis"]
  },
  "lebenslauf": {
    "kurzprofil": "…",
    "max_seiten": 2,
    "abschnitte": [
      {"titel": "Berufserfahrung", "eintraege": [
        {"von": "2021-03", "bis": null, "titel": "Senior Controller",
         "organisation": "Beispiel AG", "ort": "München", "punkte": ["…", "…"]}]},
      {"titel": "Ausbildung", "eintraege": []},
      {"titel": "Kenntnisse", "listen": {"Software": ["SAP FI/CO (fortgeschritten)", "Excel (Pivot, Power Query)"]}},
      {"titel": "Sprachen", "listen": {"": ["Deutsch (Muttersprache)", "Englisch (C1, verhandlungssicher)"]}}
    ],
    "unterschrift_zeile": false
  },
  "anlagen": [
    {"typ": "arbeitszeugnis", "datei": "profil/zeugnisse/2024_beispiel-ag.pdf", "jahr": 2024},
    {"typ": "abschluss", "datei": "profil/zeugnisse/2016_master.pdf", "jahr": 2016}
  ]
}
```

**Drei Layoutfamilien**, jeweils mit Anschreiben- und Lebenslauf-Vorlage in HTML und einem python-docx-Builder:

| Layout | Zielgruppe | Merkmale | Default |
|---|---|---|---|
| `sachlich` | Mittelstand, Konzern, Tech, Verwaltung modern | strikt einspaltig, eine Akzentfarbe, Datum als eigene Zeile über dem Eintrag | ja |
| `klassisch` | Banken, Versicherungen, Behörden, Kanzleien | keine Farbe, tabellarischer Lebenslauf (Zeitraum links) je Eintrag als Flex-Zeile, nie als Tabelle; nur nach bestandenem Test-Parsing (Kapitel 12) | nein |
| `international-en` | englischsprachige Anzeigen, Konzernsprache Englisch | eigene Struktur: kein Foto, kein Geburtsdatum, kein Familienstand, „Professional Experience / Education / Skills“, Resume 1–2 Seiten; Cover Letter kürzer und direkter, Betreffzeile nur in britischer Variante | nein |

Die englische Vorlage ist keine Übersetzung der deutschen: Foto, Alter und Familienstand gelten im englischen CV als Ausschlussgrund, Struktur und Tonalität unterscheiden sich ([cvlotse](https://cvlotse.de/ratgeber/deutscher-vs-englischer-lebenslauf), [Indeed](https://at.indeed.com/karriere-guide/bewerbung/lebenslauf-englisch), [Cover Letter](https://lebenslaufdesigns.de/anschreiben-englisch)). Papierformat bleibt A4, weil die Empfänger in Deutschland sitzen.

**Corporate-Design-neutral:** Eine konfigurierbare Akzentfarbe (Default dunkles Blaugrau) für Überschriftenlinien, sonst Schwarz auf Weiß. Keine Icons, keine Balken, keine Hintergrundflächen, kein Logo. Skills stehen als Text mit Niveau („SAP FI/CO (fortgeschritten)“, „Englisch (C1)“), nie als Sterne oder Balken – Grafiken sind für Parser unsichtbar ([bewerbungundlebenslauf.de](https://www.bewerbungundlebenslauf.de/lebenslauf-faehigkeiten/), [easycv](https://easycv.ai/blog/de/ats-optimierter-lebenslauf-2026-der-komplette-leitfaden)); Sprachen zusätzlich mit GER-Stufe und umgangssprachlichem Begriff ([Karrierebibel](https://karrierebibel.de/sprachkenntnisse-lebenslauf/)).

**Schrift:** Ratgeber empfehlen Arial, Calibri, Helvetica, Times New Roman oder Garamond in 10–12 pt ([airesume.guru](https://airesume.guru/blog/ats-friendly-resume-fonts), [resufit](https://resufit.com/blog/best-fonts-for-resume-ats-tested/)); entscheidend ist nicht der Name, sondern vollständige Einbettung mit korrekter Unicode-Zuordnung. **Entscheidung:** Carlito 11 pt als Standard (frei lizenziert, metrisch kompatibel zu Calibri – öffnet ein Empfänger die DOCX ohne Carlito, ersetzt Word sie durch Calibri mit identischen Umbrüchen), Liberation Sans als Alternative. Beide liegen als Dateien im Repo unter `assets/fonts/` und werden per `@font-face` eingebunden; kein Renderer greift auf Systemschriften zu.

**Deutsche Typografie** wird im Renderer erzwungen, nicht dem Autor überlassen: Anführungszeichen „…“, Gedankenstrich „–“ mit Leerzeichen, Bis-Strich in Zeiträumen (03/2021 – heute), geschützte Leerzeichen in „z. B.“, „10 %“, „Dr. Müller“, Tausenderpunkt (12.500 €), Silbentrennung über `lang="de"` und `hyphens: auto`, Eigennamen und Firmen in `.nobr` ohne Trennung. Flattersatz statt Blocksatz (keine Löcher in engen Zeilen). Diese Regeln laufen als Nachbearbeitungsfunktion über jeden Textstring, bevor er ins Template geht.

### 13.4 Anschreiben nach DIN 5008

DIN 5008 bleibt Referenzstandard für den deutschen Geschäftsbrief: Anschriftfeld ab 4,5 cm vom oberen Rand, Betreff zwei Leerzeilen darunter ohne das Wort „Betreff“, Formvarianten A (Briefkopf 27 mm) und B (45 mm), Fließtext 12 pt, Kontaktzeile ca. 10 pt ([gruendung.de](https://www.gruendung.de/din-5008/), [zeitblueten.com](https://www.zeitblueten.com/news/brief-din-5008/), [leonrenner.com](https://leonrenner.com/din-5008-geschaeftsbrief/)). Die Normmaße wurden vom Faktenprüfer nicht unabhängig geprüft; vor der Template-Abnahme einmal gegen die aktuelle Normausgabe halten. Wir setzen Form B, weil sie dem gewohnten Bild eines Bewerbungsanschreibens entspricht.

| Element | Inhalt | Position und Format im Template |
|---|---|---|
| Absenderblock | Vorname Nachname, Straße Nr., PLZ Ort, Telefon, E-Mail, optional Profil-URL | oben rechts ab 20 mm, 10 pt, rechtsbündig |
| Rücksendeangabe | „Vorname Nachname · Straße 1 · 12345 Ort“ in einer Zeile, ca. 8 pt | erste Zeile des Anschriftfelds; Default aus, nur Postweg |
| Zusatz- und Vermerkzone (3 Zeilen) | leer; optional „Persönlich/Vertraulich“ | Zeilen 1–3 des Anschriftfelds (85 × 45 mm ab 45 mm Oberkante, 25 mm links) |
| Anschriftzone (6 Zeilen) | Firma · Abteilung oder „Personalabteilung“ · „Frau“/„Herrn“ Titel Vorname Nachname · Straße Nr. oder Postfach · PLZ Ort · Land nur bei Auslandsadresse | Zeilen 4–9, ohne Leerzeilen innerhalb der Zone |
| Datum | Default „München, 8. September 2026“; per Konfiguration ISO „2026-09-08“, das die Norm empfiehlt | rechtsbündig unterhalb des Anschriftfelds |
| Betreff | fett, ohne „Betreff:“; Zeile 2 optional normal („Ihre Stellenanzeige auf … vom …“) | zwei Leerzeilen unter dem Anschriftfeld |
| Anrede | „Sehr geehrte Frau Dr. Beispiel,“ / „Sehr geehrter Herr Beispiel,“; Fallback „Sehr geehrte Damen und Herren,“ nur, wenn der Rechercheur keinen Ansprechpartner belegt | zwei Leerzeilen unter dem Betreff |
| Fließtext | 3–5 Absätze, 250–350 Wörter, eine Seite ([skill-sprinters](https://skill-sprinters.de/blog/karriere/anschreiben-2026-aufbau/)) | 11 pt, Zeilenabstand 1,3, eine Leerzeile zwischen Absätzen, Flattersatz |
| Grußformel | „Mit freundlichen Grüßen“ | eine Leerzeile nach dem Text |
| Unterschrift | optional Bild (13.9) | 14 mm Raum zwischen Grußformel und Namenszeile, auch wenn leer |
| Namenszeile | Vorname Nachname maschinenschriftlich | direkt unter dem Unterschriftsraum |
| Anlagenvermerk | „Anlagen“ fett, darunter Liste | mindestens eine Leerzeile unter der Namenszeile; nur wenn Anlagen beiliegen |

Die Anredeform im Anschriftfeld ist der Akkusativ („Herrn“), in der Anrede der Nominativ („Herr“); beide Formen liefert der Rechercheur als Felder, der Setzer kombiniert sie nur. Kein Ansprechpartner wird geraten (Kapitel 10). Das Datum im Anschreiben ist das Sendedatum, das der Orchestrator beim Rendern setzt; der Lebenslauf trägt dasselbe Datum, wenn er eine Unterschriftszeile hat.

Vorlagenskelett (Jinja2, WeasyPrint):

```html
{# templates/de/sachlich/anschreiben.html #}
<!doctype html>
<html lang="de">
<head>
<meta charset="utf-8">
<title>Anschreiben {{ k.vorname }} {{ k.nachname }}</title>
<style>
  @font-face { font-family: "Carlito"; src: url("../../../assets/fonts/Carlito-Regular.ttf"); }
  @font-face { font-family: "Carlito"; font-weight: bold; src: url("../../../assets/fonts/Carlito-Bold.ttf"); }
  @page { size: A4; margin: 0; }
  body { margin: 0; font-family: "Carlito", "Liberation Sans", sans-serif; font-size: 11pt; line-height: 1.3; color: #111; }
  .seite { position: relative; width: 210mm; height: 297mm; overflow: hidden; }
  .absender { position: absolute; top: 20mm; right: 20mm; text-align: right; font-size: 10pt; }
  .anschrift { position: absolute; top: 45mm; left: 25mm; width: 85mm; height: 45mm; }
  .anschrift .ruecksende { height: 5mm; font-size: 8pt; }
  .anschrift .vermerk { height: 12.7mm; }          /* 3 Zeilen Zusatz- und Vermerkzone */
  .anschrift .adresse { height: 27.3mm; }          /* 6 Zeilen Anschriftzone */
  .datum { position: absolute; top: 92mm; right: 20mm; }
  .inhalt { position: absolute; top: 100mm; left: 25mm; right: 20mm; bottom: 20mm; hyphens: auto; }
  .betreff { font-weight: bold; margin: 0 0 1.3em 0; }
  .betreff small { display: block; font-weight: normal; font-size: 11pt; }
  p { margin: 0 0 1.3em 0; }
  .unterschrift, .unterschrift img { height: 14mm; }
  .anlagen { margin-top: 1.3em; }
  .nobr { hyphens: manual; white-space: nowrap; }
</style>
</head>
<body>
<div class="seite">
  <div class="absender"><strong>{{ k.vorname }} {{ k.nachname }}</strong><br>
    {{ k.strasse }}<br>{{ k.plz }} {{ k.ort }}<br>{{ k.telefon }}<br>{{ k.email }}</div>
  <div class="anschrift">
    <div class="ruecksende">{% if opt.ruecksendeangabe %}{{ k.vorname }} {{ k.nachname }} · {{ k.strasse }} · {{ k.plz }} {{ k.ort }}{% endif %}</div>
    <div class="vermerk">{{ e.vermerk or "" }}</div>
    <div class="adresse"><span class="nobr">{{ e.firma }}</span><br>
      {% if e.abteilung %}{{ e.abteilung }}<br>{% endif %}
      {% if e.ansprechpartner %}{{ e.ansprechpartner.anrede_brief }} {{ e.ansprechpartner.titel }} {{ e.ansprechpartner.vorname }} {{ e.ansprechpartner.nachname }}<br>{% endif %}
      {{ e.strasse }}<br>{{ e.plz }} {{ e.ort }}{% if e.land %}<br>{{ e.land }}{% endif %}</div>
  </div>
  <div class="datum">{{ k.ort }}, {{ a.datum | datum_de }}</div>
  <div class="inhalt">
    <div class="betreff">{{ a.betreff }}{% if a.betreff_zeile2 %}<small>{{ a.betreff_zeile2 }}</small>{% endif %}</div>
    <p>{{ a.anrede }}</p>
    {% for absatz in a.absaetze %}<p>{{ absatz | typo_de }}</p>{% endfor %}
    <p>{{ a.gruss }}</p>
    <div class="unterschrift">{% if a.unterschrift_einfuegen and k.unterschrift %}<img src="{{ k.unterschrift }}" alt="">{% endif %}</div>
    <p>{{ k.vorname }} {{ k.nachname }}</p>
    {% if a.anlagen %}<div class="anlagen"><strong>Anlagen</strong><br>{{ a.anlagen | join("<br>") }}</div>{% endif %}
  </div>
</div>
</body>
</html>
```

Der python-docx-Builder erzeugt dasselbe Dokument mit A4, Rändern 25/20/20 mm, Absatzabständen in Punkt statt absoluter Positionen und echten Word-Formatvorlagen; ein DOCX-Anschreiben wird nur erzeugt, wenn ein Portal es ausdrücklich verlangt (Default: nur PDF).

### 13.5 Lebenslauf: ATS-sicher setzen

Moderne Parser lesen textbasierte PDFs zuverlässig; Fehler entstehen durch Bild-PDFs, Tabellen, Mehrspaltenlayouts und Inhalte in Kopf-/Fußzeilen ([resumemate](https://www.resumemate.io/blog/pdf-vs-docx-for-resumes-in-2025-what-recruiters-ats-really-prefer/), [jobwizard](https://jobwizard.ai/blog/how-to-optimize-your-resume-for-ats-systems-in-2026-872842), [atsresumeai](https://www.atsresumeai.com/blog/ats-resume-formatting-guide)); Workday gilt als besonders empfindlich: Kontaktdaten im Fließtext, Standardüberschriften, keine Tabellen ([resufit](https://resufit.com/blog/optimizing-resume-text-for-autofill-success-a-guide-to-workday-and-ats-compatibility/)). Diese Quellen sind Ratgeber, keine Herstellerdokumentation; deshalb prüft Kapitel 12 jede Vorlage empirisch. Harte Regeln des Setzers:

1. **Eine Spalte** über die ganze Seite. Der Kopf mit Name, Anschrift, Telefon, E-Mail, Profil-URL steht als normaler Text im Seitenkörper, nicht in einer Kopfzeile.
2. **Standardüberschriften** in der Zielsprache: „Kurzprofil“, „Berufserfahrung“, „Ausbildung“, „Weiterbildung und Zertifikate“, „Kenntnisse“, „Sprachen“, optional „Engagement“; englisch „Professional Summary“, „Professional Experience“, „Education“, „Certifications“, „Skills“, „Languages“. Als echte Überschriften-Elemente (`h2`, in DOCX „Heading 2“), damit Parser und Navigationsbereich sie erkennen.
3. **Eintragsmuster** fest: Zeile 1 fett „Positionstitel – Organisation, Ort“, Zeile 2 grau „03/2021 – heute“, dann 2–5 Aufzählungspunkte mit Ergebnissen. Layout `klassisch` darf den Zeitraum links neben den Eintrag setzen, aber nur als Flex-Zeile pro Eintrag, nie als seitenweite Spalte oder Tabelle.
4. **Echte Textebene, eingebettete Schriften, kein Text in Bildern.** Keine Textboxen, keine SVG-Schriftzüge, keine Icons.
5. **Foto optional, Default aus.** Seit dem AGG ist es rechtlich freiwillig, manche Unternehmen bitten ausdrücklich um Verzicht ([Haufe](https://www.haufe.de/id/beitrag/agg-die-merkmale-rasse-und-ethnische-herkunft-23-bewerbungsfoto-HI16209081.html), [photo-bergmeister](https://www.photo-bergmeister.de/2025/08/22/bewerbungsfoto-pflicht-das-solltest-du-2025-wissen/)); Branchenquellen berichten zugleich von weiterhin hoher faktischer Erwartung in Finanzen, Recht und Mittelstand ([skill-sprinters](https://skill-sprinters.de/blog/karriere/bewerbungsfoto-2026-ja-oder-nein/), unbestätigt). Ist es aktiviert, sitzt es rechts oben im Seitenkörper als JPEG mit max. 35 × 45 mm, ohne Text im Bild, `alt=""`.
6. **Geburtsdatum und Familienstand** rendert der Setzer nur, wenn das Kandidatenprofil sie ausdrücklich freigibt; Ratgeber empfehlen das Weglassen aus AGG-Gründen ([StepStone](https://www.stepstone.de/magazin/artikel/persoenliche-daten-lebenslauf), [Karrierebibel](https://karrierebibel.de/persoenliche-daten-im-lebenslauf/)). Default: beides aus.
7. **Länge:** Default 2 Seiten, konfigurierbar bis 3. Fußzeile nur „Vorname Nachname · Lebenslauf · Seite 1/2“ – Parser ignorieren Fußzeilen, hier steht deshalb nichts Inhaltliches.
8. **Struktur schlägt Ästhetik:** Recruiter entscheiden nach Sekunden und folgen einem F-Muster (Titel, Firma, Daten zuerst; Eye-Tracking-Studie 2018, [The Ladders](https://www.theladders.com/career-advice/you-only-get-6-seconds-of-fame-make-it-count)) – daher fette Titel, klare Daten, Ergebnisse als Bullet-Points, keine Fließtextblöcke.
9. **Metadaten-Hygiene:** Titel „Lebenslauf Vorname Nachname“, Autor der Kandidat, `/Lang de-DE`, Producer/Creator neutralisiert, Erstell- und Änderungsdatum auf das Bewerbungsdatum fixiert (pypdf). Für DOCX ebenso die Kerneigenschaften.

Zusätzliche Dokumenttypen für v2: Kurzbewerbung (Anschreiben und Lebenslauf auf 1–2 Seiten, für Initiativbewerbungen) und Motivationsschreiben als eigenes Zusatzdokument ([bewerbung.net](https://bewerbung.net/kurzbewerbung), [arbeitsrechte.de](https://www.arbeitsrechte.de/kurzbewerbung/)). Ein Deckblatt wird nicht erzeugt; es ist für Online-Bewerbungen optional bis überholt ([bewerbungsgenius](https://bewerbungsgenius.de/ratgeber/bewerbungsmappe-2026-was-reingehoert-und-die-richtige-reihenfolge)), ein Konfigurationsschalter bleibt vorgesehen.

### 13.6 Dateinamenskonvention

Ratgeber und Praxis: Nachname_Vorname_Dokumenttyp, ASCII-sicher, ohne Umlaute, Leerzeichen, Sonderzeichen und ohne Datum – nur Vornamen sind bei großen Empfängern nicht eindeutig, ein Datum verrät, wie lange die Unterlagen schon kursieren ([tabellarischer-lebenslauf.net](https://www.tabellarischer-lebenslauf.net/bewerbung-tipps/dateinamen-der-bewerbungsdokumente/), [namequick](https://www.namequick.app/de/blog/how-to-name-a-resume-file)).

```text
Schema:          <Nachname>_<Rufname>_<Dokumenttyp>[_<Zusatz>].<pdf|docx>
Dokumenttypen:   Anschreiben | Lebenslauf | Zeugnisse | Zertifikate | Bewerbung (Mappe)
Transliteration: ä→ae ö→oe ü→ue ß→ss, Akzente entfernen (é→e), Leerzeichen→_,
                 alles außer [A-Za-z0-9_-] streichen, Bindestrich in Doppelnamen bleibt
Prüfregel:       ^[A-Za-z0-9-]+_[A-Za-z0-9-]+_(Anschreiben|Lebenslauf|Zeugnisse|Zertifikate|Bewerbung)(_[A-Za-z0-9-]{1,30})?\.(pdf|docx)$
Beispiele:       Mueller_Anna_Anschreiben.pdf   Meyer-Schmidt_Jonas_Lebenslauf.docx   Mueller_Anna_Bewerbung.pdf
Verboten:        Datum, „final“, „neu“, „v2“, Umlaute, Leerzeichen, Klammern
Optionaler Zusatz (Default aus): Stellenkurzform, z. B. Mueller_Anna_Bewerbung_Controlling.pdf
```

Interne Dateinamen im Archiv (13.11) dürfen Datum und Version tragen; nach außen geht nur der bereinigte Name. Der Bote (Kapitel 15) benennt nichts um.

### 13.7 Bewerbungsmappe: eine PDF oder getrennte Dateien

Für E-Mail-Bewerbungen wird eine zusammengeführte PDF empfohlen (Reihenfolge bleibt erhalten, wirkt geordnet), für Portale mit getrennten Feldern Einzeldateien ([cvlotse](https://cvlotse.de/ratgeber/bewerbungsunterlagen-pdf), [bewerbungstools.de](https://www.bewerbungstools.de/ratgeber/bewerbungsunterlagen-eine-pdf), [meine-bewerbungsvorlage.de](https://meine-bewerbungsvorlage.de/blogs/news/pdf-zusammenfuegen-bewerbung)). **Entscheidung je Bewerbungsweg:**

| Bewerbungsweg | Ausgabe | Reihenfolge | Größenbudget |
|---|---|---|---|
| `email` | eine PDF `Nachname_Rufname_Bewerbung.pdf`; Anschreiben zusätzlich als `anschreiben.txt` für den Mailtext (Kapitel 15) | Anschreiben, Lebenslauf, Zeugnisse (neuestes zuerst) | Ziel ≤ 3 MB, hart ≤ 5 MB |
| `portal_getrennt` | Einzeldateien je Feld; gibt es nur ein Anlagenfeld, werden Zeugnisse zu `…_Zeugnisse.pdf` zusammengeführt | wie Feldbezeichnung | je Datei ≤ 5 MB |
| `portal_einzeln`, Feld heißt „Lebenslauf/CV“ | nur Lebenslauf hochladen; Anschreiben in ein Nachrichten-/Freitextfeld, sonst Mappe in Variante P | Variante P: Lebenslauf, Anschreiben, Zeugnisse | je Datei ≤ 5 MB |
| `portal_einzeln`, Feld heißt „Unterlagen“ | Mappe wie E-Mail | Anschreiben, Lebenslauf, Zeugnisse | je Datei ≤ 5 MB |
| `freitext` | `anschreiben.txt` (UTF-8, Absätze durch Leerzeile, keine geschützten Leerzeichen) plus Lebenslauf-PDF | – | Zeichenlimit des Felds prüft der Bote |

Variante P stellt den Lebenslauf nach vorn, weil ein CV-Parser die ersten Seiten liest und ein vorangestelltes Anschreiben die Feldzuordnung stört. Die Budgets folgen den Ratgeberwerten (ideal 1–3 MB, akzeptabel bis 5–6 MB; [stratag](https://www.stratag.de/bewerbung-dateigroesse), [bewerbungsanschreiben.info](https://www.bewerbungsanschreiben.info/achtung-datenflut-ueber-die-richtige-dateigroesse-von-bewerbungsanlagen/)) und liegen unter dem Personio-Limit von 10 MB je Dokument ([Personio-Hilfe](https://support.personio.de/hc/en-us/articles/360017288378-Manage-candidate-documents), nicht unabhängig geprüft). Für softgarden und Workday liegen keine belegten Limits vor; 5 MB je Datei ist die Sicherheitsmarge.

Technik: Zusammenführen mit pypdf, Lesezeichen (Outline) je Dokument („Anschreiben“, „Lebenslauf“, „Arbeitszeugnis Beispiel AG (2024)“), Metadaten wie in 13.5, keine aufgedruckten Seitenzahlen über Scans. Anschreiben und Lebenslauf zusammen bleiben unter 500 KB (Text plus eingebettete Schriftteilmengen); der Rest des Budgets gehört den Anlagen.

### 13.8 Anlagen: Zeugnisse aufbereiten

Zeugnisse kommen einmalig beim Onboarding ins Kandidatenprofil (Kapitel 8) und werden dort vom Setzer vorverarbeitet, nicht bei jeder Bewerbung neu:

1. **Normalisieren:** A4 Hochformat, gerade Seitenreihenfolge, eine Datei je Zeugnis, Dateiname mit Jahr und Aussteller.
2. **Textebene per OCR** (Tesseract mit deutschem Sprachpaket; die Claude-Sandbox liefert nur Englisch, [Sandbox-Referenz](https://platform.claude.com/docs/en/managed-agents/cloud-sandboxes-reference)), damit Zeugnisse durchsuchbar sind. Der OCR-Text dient nur der Suche, nie als Inhalt für Autor oder Bote.
3. **Komprimieren:** Graustufen, 150 dpi, JPEG-Qualität um 70, Richtwert 200 KB je Seite (pdf2image + Pillow + reportlab oder Ghostscript auf dem VPS). Original bleibt unverändert im Profil, die komprimierte Fassung wird verwendet.
4. **Metadaten bereinigen** (Scanner-Modell, Software-Namen).

Auswahl pro Bewerbung durch Regel, nicht durch Modell: die letzten zwei bis drei Arbeitszeugnisse, höchster Abschluss, stellenrelevante Zertifikate aus der Anforderungsliste des Matchers (Kapitel 9); Obergrenze 12 Seiten Anlagen. Ältere Zahlen sprechen für Vollständigkeit der letzten Zeugnisse statt Detailtiefe (Job-Trends 2017, [karriereakademie.de](https://www.karriereakademie.de/arbeitszeugnis-wichtig); keine neuere Studie gefunden). Reihenfolge: Arbeitszeugnisse chronologisch absteigend, dann Abschlusszeugnisse absteigend, dann Zertifikate. Fremdsprachige Zeugnisse werden nur mit vorhandener beglaubigter Übersetzung angehängt; der Setzer erzeugt keine Übersetzung und kennzeichnet nichts als beglaubigt – das dürfen nur vereidigte Übersetzer ([mentorium.de](https://www.mentorium.de/zeugnisse-beglaubigt-uebersetzen/); Compliance in Kapitel 16).

### 13.9 Unterschrift

Eine Unterschrift ist rechtlich nicht vorgeschrieben; bei Online-Bewerbungen ist sie unüblich, bei postalischen und konservativen Bewerbungen wird sie weiterhin erwartet ([Karrierebibel](https://karrierebibel.de/bewerbung-lebenslauf-unterschreiben/), [enhancv](https://enhancv.com/de/blog/datum-und-unterschrift-auf-lebenslauf/)); eine Ratgeberquelle nennt rund 80 % der Personaler, die sie weiterhin schätzen ([Karrierebibel](https://karrierebibel.de/unterschrift-bei-online-bewerbung/), ohne Studienangabe, unbestätigt).

**Entscheidung:** Default keine Unterschrift. Ist im Profil ein Unterschriftsbild hinterlegt, fügt der Setzer es ein, wenn Layout `klassisch` gewählt ist oder das Dossier `unterschrift_einfuegen: true` trägt (Kapitel 11 setzt das nach Branchenregel). Position im Anschreiben: zwischen Grußformel und Namenszeile, 14 mm hoch; im Lebenslauf: letzte Seite unten, „Ort, Datum“ links, Unterschrift darunter – dasselbe Datum wie im Anschreiben. Format: PNG mit transparentem Hintergrund, ca. 600 px breit, freigestellt ohne Linie.

Das Unterschriftsbild ist ein missbrauchsanfälliges Asset: Es liegt ausschließlich lokal im Profilverzeichnis, geht nie in einen Prompt oder Modellkontext, wird nie in Vorschaubildern des Review-Cockpits verschickt und ist nur dem Setzer-Tool zugänglich (Kapitel 16).

### 13.10 Qualitätsprüfung im Setzer

Jede Render-Version durchläuft diese Prüfungen, bevor der ATS-Prüfer (Kapitel 12) übernimmt:

| Prüfung | Werkzeug | Grenzwert | Bei Verstoß |
|---|---|---|---|
| Platzhalter-Leck | Regex über extrahiertem Text (`{{`, `[…]`, „XXX“, „TODO“, „Muster“, „Beispiel“ außer im Firmennamen) | 0 Treffer | harter Stopp, Meldung an Kritiker |
| Seitenzahl | pypdf | Anschreiben = 1; Lebenslauf ≤ `max_seiten` | zurück an Autor mit Überlänge in Zeilen |
| Überlauf und Waisen | pdfplumber (Textbox der letzten Seite) | letzte Lebenslaufseite ≥ 25 % gefüllt; keine Überschrift ohne Inhalt am Seitenende | CSS-Umbruchregeln (`break-inside: avoid`), sonst Autor |
| Textextraktion | pdftotext oder pdfplumber gegen Dossier-Text (normalisiert) | alle Wörter in gleicher Reihenfolge, Umlaute und ß erhalten, kein „�“ | harter Stopp |
| Schrifteinbettung | pdffonts (Poppler) | alle Schriften „emb yes“, Teilmengen erlaubt | harter Stopp |
| Dateigröße | Dateisystem | Anschreiben + Lebenslauf ≤ 500 KB; Mappe ≤ Budget des Bewerbungswegs | Anlagen stärker komprimieren, sonst Anlagen reduzieren |
| Metadaten | pypdf | Titel, Autor, Sprache gesetzt; Producer/Creator neutral; Datum fixiert | automatisch korrigieren |
| Konsistenz | Vergleich mit Dossier und Rechercheur-Datensatz | Datum Anschreiben = Lebenslauf; Name, Anschrift, Ansprechpartner identisch; Betreff enthält Stellentitel oder Kennziffer | harter Stopp oder „Rückfrage offen“ |
| Dateiname | Regex aus 13.6 | Treffer | automatisch korrigieren |
| DOCX | LibreOffice headless → PDF, pandoc → Text | gleiche Seitenzahl wie PDF ± 0; Text-Diff leer | zurück an Vorlagenpflege |
| Sichtprüfung (optional) | pdftoppm → PNG → Claude Haiku 4.5 mit Bild | Frage: „Sichtbarer Layoutfehler? ja/nein, Grund“ | Hinweis im Review-Cockpit, kein Stopp |

Der Setzer verkleinert nie eigenmächtig Schrift oder Ränder, um Text passend zu machen; eine Seite mit 9-pt-Schrift fällt auf. Überlänge geht als Zahl zurück („Anschreiben: 6 Zeilen zu lang, ca. 60 Wörter kürzen“), der Autor kürzt, der Kritiker prüft erneut, der Setzer rendert Version n+1. Nach drei Schleifen stoppt der Orchestrator und legt den Fall ins Review-Cockpit.

Die Seitenvorschauen (PNG, 110 dpi) sind Teil jeder Version und werden im Review-Cockpit neben dem Text-Diff angezeigt, damit du das Dokument so siehst, wie es der Empfänger sieht.

### 13.11 Versionierung und Ablage

Erzeugte Dateien liegen im Dateisystem des VPS, indiziert in SQLite, in einem privaten Git-Repository versioniert (Architekturentscheidung Kapitel 7). Bei rund 300 Bewerbungen im Monat und unter 3 MB je Mappe sind das unter 1 GB pro Jahr.

```text
bewerbungen/
  2026-09-08_beispiel-gmbh_senior-controller_a1b2c3/
    dossier.json                 # Eingabe im Status „geprüft“
    empfaenger.json              # Rechercheur-Datensatz mit Konfidenzen
    render/
      v1/  Mueller_Anna_Anschreiben.pdf  Mueller_Anna_Lebenslauf.pdf
           Mueller_Anna_Lebenslauf.docx  anschreiben.txt  qa.json  preview/seite-01.png …
      v2/  …
    final/                       # nach Freigabe: Kopie der freigegebenen Version, schreibgeschützt
      Mueller_Anna_Bewerbung.pdf
      SHA256SUMS
    manifest.json
```

```json
{
  "bewerbung_id": "a1b2c3",
  "status": "gesendet",
  "layout": "sachlich",
  "sprache": "de",
  "bewerbungsweg": "email",
  "renderer": {
    "weasyprint": "<gepinnte Version>",
    "python_docx": "<gepinnte Version>",
    "template_version": "2026.09.1",
    "template_hash": "sha256:…"
  },
  "fonts": {"Carlito-Regular.ttf": "sha256:…", "Carlito-Bold.ttf": "sha256:…"},
  "versionen": [
    {"v": 1, "erzeugt": "2026-09-08T06:14:02+02:00", "qa": "fehlgeschlagen",
     "grund": "Anschreiben 2 Seiten, 6 Zeilen Überlauf"},
    {"v": 2, "erzeugt": "2026-09-08T06:21:40+02:00", "qa": "bestanden", "ats_pruefer": "bestanden"}
  ],
  "final": {
    "quelle": "render/v2",
    "freigegeben_am": "2026-09-08T08:02:11+02:00",
    "dateien": {"Mueller_Anna_Bewerbung.pdf": "sha256:…"}
  }
}
```

Regeln: Vorlagen tragen eine Versionsnummer und werden mit dem Repo versioniert; jede Änderung an Vorlage oder Schriftdatei erzwingt neues Test-Parsing (Kapitel 12). Der Ordner `final/` entsteht erst mit dem Status „freigegeben“ und wird danach nie überschrieben; der Bote (Kapitel 15) versendet ausschließlich aus `final/` und protokolliert die Hashes. Ältere Render-Versionen bleiben bis zum Abschluss der Bewerbung erhalten und werden mit dem Löschkonzept aus Kapitel 16 entfernt. Ein erneutes Rendern mit identischem Dossier, identischer Vorlagenversion und fixierten Zeitstempeln muss denselben Hash liefern; das ist der Integrationstest der Setzer-Pipeline.

### 13.12 Default-Annahmen dieses Moduls

Bis du anders entscheidest (Fragenkatalog Kapitel 22): kein Foto, kein Deckblatt, keine Unterschrift, kein Geburtsdatum; Layout `sachlich`; Lebenslauf maximal 2 Seiten; Datum ausgeschrieben mit Ort; nur PDF plus DOCX-Lebenslauf; Sprache der Vorlage folgt der Sprache der Stellenanzeige, Freigabe pro Bewerbung im Review-Cockpit.

**Quellen dieses Kapitels:**

- WeasyPrint – https://weasyprint.com/
- WeasyPrint-Dokumentation, Common Use Cases (PDF/A, PDF/UA experimentell) – https://doc.courtbouillon.org/weasyprint/stable/common_use_cases.html
- Typst (GitHub) – https://github.com/typst/typst
- Typst Universe: modern-cv – https://typst.app/universe/package/modern-cv/
- Typst Universe: brilliant-cv – https://typst.app/universe/package/brilliant-cv/
- RenderCV (GitHub) – https://github.com/rendercv/rendercv
- Reactive Resume (GitHub) – https://github.com/amruthpillai/reactive-resume
- Tectonic (GitHub) – https://github.com/tectonic-typesetting/tectonic
- JSON Resume – https://jsonresume.org/
- python-docx – https://python-docx.readthedocs.io/
- docxtemplater Pricing – https://docxtemplater.com/pricing/
- Carbone Pricing – https://carbone.io/pricing.html
- anthropics/skills (docx-/pdf-Skill, Lizenz) – https://github.com/anthropics/skills
- Claude Agent SDK, Custom Tools – https://code.claude.com/docs/en/agent-sdk/custom-tools
- Claude Managed Agents, Cloud-Sandbox-Referenz – https://platform.claude.com/docs/en/managed-agents/cloud-sandboxes-reference
- Personio Hilfe: CV parsing for candidate profiles – https://support.personio.de/hc/en-us/articles/360010193018-CV-parsing-for-candidate-profiles
- Personio Hilfe: Manage candidate documents – https://support.personio.de/hc/en-us/articles/360017288378-Manage-candidate-documents
- softgarden Hilfe: CV-Parsing – https://support.softgarden.de/de/articles/680780-cv-parsing
- DIN 5008 (gruendung.de) – https://www.gruendung.de/din-5008/
- DIN 5008 (zeitblueten.com) – https://www.zeitblueten.com/news/brief-din-5008/
- DIN 5008 Geschäftsbrief (leonrenner.com) – https://leonrenner.com/din-5008-geschaeftsbrief/
- Anschreiben 2026: Aufbau (skill-sprinters) – https://skill-sprinters.de/blog/karriere/anschreiben-2026-aufbau/
- Bewerbungsfoto 2026 (skill-sprinters) – https://skill-sprinters.de/blog/karriere/bewerbungsfoto-2026-ja-oder-nein/
- PDF vs. DOCX for resumes (resumemate) – https://www.resumemate.io/blog/pdf-vs-docx-for-resumes-in-2025-what-recruiters-ats-really-prefer/
- ATS-Formatierung (jobwizard) – https://jobwizard.ai/blog/how-to-optimize-your-resume-for-ats-systems-in-2026-872842
- ATS Resume Formatting Guide (atsresumeai) – https://www.atsresumeai.com/blog/ats-resume-formatting-guide
- Workday-Kompatibilität (resufit) – https://resufit.com/blog/optimizing-resume-text-for-autofill-success-a-guide-to-workday-and-ats-compatibility/
- ATS-sichere Schriften (airesume.guru) – https://airesume.guru/blog/ats-friendly-resume-fonts
- Best Fonts for Resume (resufit) – https://resufit.com/blog/best-fonts-for-resume-ats-tested/
- Fähigkeiten im Lebenslauf (bewerbungundlebenslauf.de) – https://www.bewerbungundlebenslauf.de/lebenslauf-faehigkeiten/
- ATS-optimierter Lebenslauf 2026 (easycv) – https://easycv.ai/blog/de/ats-optimierter-lebenslauf-2026-der-komplette-leitfaden
- Sprachkenntnisse im Lebenslauf (Karrierebibel) – https://karrierebibel.de/sprachkenntnisse-lebenslauf/
- AGG und Bewerbungsfoto (Haufe) – https://www.haufe.de/id/beitrag/agg-die-merkmale-rasse-und-ethnische-herkunft-23-bewerbungsfoto-HI16209081.html
- Bewerbungsfoto Pflicht? (photo-bergmeister) – https://www.photo-bergmeister.de/2025/08/22/bewerbungsfoto-pflicht-das-solltest-du-2025-wissen/
- Persönliche Daten im Lebenslauf (StepStone) – https://www.stepstone.de/magazin/artikel/persoenliche-daten-lebenslauf
- Persönliche Daten im Lebenslauf (Karrierebibel) – https://karrierebibel.de/persoenliche-daten-im-lebenslauf/
- Eye-Tracking-Studie (The Ladders, 2018) – https://www.theladders.com/career-advice/you-only-get-6-seconds-of-fame-make-it-count
- Deutscher vs. englischer Lebenslauf (cvlotse) – https://cvlotse.de/ratgeber/deutscher-vs-englischer-lebenslauf
- Lebenslauf Englisch (Indeed) – https://at.indeed.com/karriere-guide/bewerbung/lebenslauf-englisch
- Anschreiben Englisch (lebenslaufdesigns.de) – https://lebenslaufdesigns.de/anschreiben-englisch
- Kurzbewerbung (bewerbung.net) – https://bewerbung.net/kurzbewerbung
- Kurzbewerbung (arbeitsrechte.de) – https://www.arbeitsrechte.de/kurzbewerbung/
- Bewerbungsmappe 2026 (bewerbungsgenius) – https://bewerbungsgenius.de/ratgeber/bewerbungsmappe-2026-was-reingehoert-und-die-richtige-reihenfolge
- Dateinamen der Bewerbungsdokumente (tabellarischer-lebenslauf.net) – https://www.tabellarischer-lebenslauf.net/bewerbung-tipps/dateinamen-der-bewerbungsdokumente/
- How to name a resume file (namequick) – https://www.namequick.app/de/blog/how-to-name-a-resume-file
- Bewerbungsunterlagen als PDF (cvlotse) – https://cvlotse.de/ratgeber/bewerbungsunterlagen-pdf
- Bewerbungsunterlagen in einer PDF (bewerbungstools.de) – https://www.bewerbungstools.de/ratgeber/bewerbungsunterlagen-eine-pdf
- PDF zusammenfügen für die Bewerbung (meine-bewerbungsvorlage.de) – https://meine-bewerbungsvorlage.de/blogs/news/pdf-zusammenfuegen-bewerbung
- Dateigröße der Bewerbung (stratag) – https://www.stratag.de/bewerbung-dateigroesse
- Dateigröße von Bewerbungsanlagen (bewerbungsanschreiben.info) – https://www.bewerbungsanschreiben.info/achtung-datenflut-ueber-die-richtige-dateigroesse-von-bewerbungsanlagen/
- Arbeitszeugnis wichtig? (karriereakademie.de) – https://www.karriereakademie.de/arbeitszeugnis-wichtig
- Zeugnisse beglaubigt übersetzen (mentorium.de) – https://www.mentorium.de/zeugnisse-beglaubigt-uebersetzen/
- Lebenslauf unterschreiben (Karrierebibel) – https://karrierebibel.de/bewerbung-lebenslauf-unterschreiben/
- Unterschrift bei Online-Bewerbung (Karrierebibel) – https://karrierebibel.de/unterschrift-bei-online-bewerbung/
- Datum und Unterschrift auf Lebenslauf (enhancv) – https://enhancv.com/de/blog/datum-und-unterschrift-auf-lebenslauf/
