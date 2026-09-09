## 8. Modul Kandidatenprofil: Master-Lebenslauf, Story-Bank, Stimmprofil, Onboarding

Das Kandidatenprofil ist die einzige Quelle der Wahrheit über dich. Rechercheur, Autor, Kritiker, Matcher, ATS-Prüfer und Setzer lesen daraus, aber kein Modell schreibt hinein (Kapitel 7, 7.3). Was hier fehlt, kann später niemand erfinden – es entsteht eine Rückfrage oder die Bewerbung bleibt ohne diesen Punkt. Was hier falsch steht, wandert in jede Bewerbung. Die Sorgfalt beim Aufbau dieses Moduls bestimmt deshalb direkter als jeder Prompt, wie gut der Bewerbungsagent wird.

Fünf Dateien im Daten-Repository unter `profil/` (Kapitel 7, 7.9) bilden das Profil: `lebenslauf.yaml` (Master-Lebenslauf), `story_bank.yaml` (belegte Erfolge), `stimmprofil.md` (Schreibstil), `praeferenzen.yaml` (Rollen, Orte, Gehalt, Ausschlüsse) und `standardantworten.yaml` (Formularantworten), ergänzt um einen Dokumentenordner (8.7). Dieses Kapitel definiert ihren Inhalt vollständig; Kapitel 7 definiert nur, wo sie liegen und wie sie versioniert werden.

| Modul | Was es aus dem Kandidatenprofil braucht |
|---|---|
| Scout/Matcher (9) | Rollen, Orte, Pendelzeit, Sprache, Ausschlüsse für den Muss-Filter |
| Rechercheur (10) | Wechselmotiv-Rahmen, um Fragen an dich richtig zu stellen |
| Autor/Kritiker (11) | Master-Lebenslauf, Story-Bank, Stimmprofil, Präferenzen als Pflicht-Eingabe |
| ATS-Prüfer (12) | Kernkompetenzen und Synonym-Mapping aus dem Master-Lebenslauf |
| Setzer (13) | Layout-Präferenzen, Unterschrift, aufbereitete Zeugnisse |
| Bote (15) | Standardantworten für Formularfelder und Knockout-Fragen |

### 8.1 Grundprinzipien

**Der Master-Lebenslauf ist die vollständige Wahrheit, keine Vorlage.** Er enthält alles: jede Station, jeden Bullet, jede Kenntnis mit Niveau, jede Zahl mit Story-Bank-Verweis. Jede Bewerbung ist eine Projektion daraus – auswählen, betonen, umordnen, nie erweitern (Kapitel 11.7).

**Die Story-Bank ist das Fundament, nicht die Deko.** Jede Zahl, die im Anschreiben steht, muss auf einen Story-Bank-Eintrag zeigen; sonst wird sie entfernt oder zur Rückfrage (Kapitel 11.1). Das folgt direkt aus Anthropics Leitfaden gegen Halluzinationen: Behauptungen zuerst belegen, sonst zurückziehen, statt zu raten ([Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)).

**Das Stimmprofil ist eine Grenze, kein Versprechen perfekter Imitation.** Few-Shot-Stiltransfer erreicht bei Autoren mit großem öffentlichen Textkorpus deutlich höhere Trefferquoten als bei „everyday authors" mit wenigen, informellen Textproben; bei Letzteren bleibt laut einer Studie der ACL Findings 2025 eine signifikante Lücke zu echtem menschlichem Schreiben ([Catch Me If You Can? Not Yet](https://arxiv.org/pdf/2509.14543), [Jemama 2025](https://arxiv.org/abs/2509.24930)). Deshalb liefert das Stimmprofil explizite Regeln und Tabus statt eines Imitationsauftrags – der Autor bekommt Grenzen, keine Kopiervorlage (Kapitel 11.1).

**Alles liegt lokal und versioniert, nichts in der Cloud eines Dritten.** Details in 8.10.

### 8.2 Master-Lebenslauf (`lebenslauf.yaml`)

**Entscheidung:** Der Master-Lebenslauf wird als YAML-Datei geführt, mit einem an [JSON Resume](https://jsonresume.org/) angelehnten, aber eigenständig erweiterten Schema. **Begründung:** JSON Resume ist ein offenes, seit 2014 stabiles Schema mit den vertrauten Blöcken `basics`, `work`, `education`, `skills`, `languages` und einer breiten Theme-Landschaft, deckt aber keine Story-Bank-Verweise, Synonym-Mappings, Titel-Aliase oder Lücken-Wortlaute ab – genau die Felder, die Kapitel 11 für das Tailoring braucht. YAML statt JSON, weil es in Git lesbar bleibt und sich im Editor direkt pflegen lässt; RenderCV, der für den Setzer vorgesehene YAML-zu-PDF-Renderer (Kapitel 13), erwartet ohnehin YAML ([RenderCV](https://rendercv.com/)). **Alternative:** Strikte JSON-Resume-Konformität, die aber Zusatzfelder nur über eine Extension (`meta`) statt nativ erlauben würde und die Story-Bank-Kopplung verschleiert.

```yaml
# profil/lebenslauf.yaml
version: 3
master_version: "2026-08-30"          # Datum der letzten inhaltlichen Änderung
basics:
  name: "Vorname Nachname"
  email: "..."
  telefon: "..."
  ort: "Stadt, Land"
  profile_links:
    linkedin: "..."
    xing: "..."
  foto_verwenden: false                # Default aus, AGG-Empfehlung (Kapitel 5, 13)
  geburtsdatum_anzeigen: false
  familienstand_anzeigen: false
work:
  - firma: "..."
    titel: "..."
    titel_alias: null                  # z.B. "Senior Consultant" statt "Specialist II"
    von: "2019-03"
    bis: "2022-06"                      # null = laufend
    ort: "..."
    bullets:
      - text: "..."
        story_id: "S01"                 # Pflichtverweis, sonst kein Beleg
luecken:
  - von: "2022-07"
    bis: "2022-09"
    bezeichnung: "Elternzeit"           # dein Wortlaut, nie vom Modell ergänzt
education:
  - institution: "..."
    abschluss: "..."
    note: null                          # nur wenn du sie zeigen willst
    von: "..."
    bis: "..."
    auslaendischer_abschluss: false
    anabin_status: null                 # H+, unklar, nicht geprüft (siehe 8.7)
skills:
  - name: "Projektmanagement"
    niveau: "fortgeschritten"           # Anfänger | fortgeschritten | Experte
    synonyme: ["PM", "Project Management"]   # exakte Spiegelung der Anzeige (R02)
languages:
  - sprache: "Englisch"
    ger_niveau: "C1"
    umgangssprachlich: "verhandlungssicher"
zertifikate:
  - name: "..."
    aussteller: "..."
    jahr: 2024
    zeugnis_datei: "profil/dokumente/zeugnisse/2024_zertifikat_xyz.pdf"
```

Jeder Bullet trägt zwingend eine `story_id`; ohne sie darf der Autor die Zahl nicht verwenden (Kapitel 11.1). `titel_alias` und Synonyme werden einmalig im Onboarding festgelegt, nicht vom Modell nachträglich erfunden (Kapitel 11.7). `luecken` benennt Lücken mit deinem Wortlaut – Standardbegriffe sind „Elternzeit", „Weiterbildung", „Bewerbungsphase"; fehlt ein Eintrag, entsteht bei der ersten betroffenen Bewerbung eine Rückfrage statt eines Platzhaltertexts.

### 8.3 Story-Bank (`story_bank.yaml`)

Die Story-Bank ist eine Liste belegter Erfolge im STAR/CAR-Muster (Situation/Kontext – Aufgabe – Handlung – Ergebnis), wie es sich für Bewerbungs- und Interview-Vorbereitung etabliert hat ([CareerScribe STAR Story Bank Template](https://blog.careerscribeai.com/star-story-bank-template/)). Empfehlung aus der Recherche: mindestens 8 bis 10 Einträge über verschiedene Kompetenzbereiche verteilt, damit für unterschiedliche Stellentypen passende Erfolge zur Verfügung stehen; fünf gelten als Mindestausstattung für Interviews, für ein Bewerbungsjahr mit zehn Bewerbungen pro Tag ist mehr Varianz nötig.

```yaml
# profil/story_bank.yaml
- id: "S01"
  titel: "Durchlaufzeit im SAP-EWM-Rollout gesenkt"
  kontext: "Situation/Aufgabe in 2-3 Sätzen: Ausgangslage, Auftrag"
  handlung: "Was du konkret getan hast (dein Anteil, nicht das Team pauschal)"
  ergebnis:
    text: "Durchlaufzeit gesenkt"
    zahl: 18
    einheit: "Prozent"
    ca: false                          # true = im Text als "rund"/"etwa" ausgeben
    zeitraum: "6 Monate"
  beleg:
    status: "dokumentiert"             # dokumentiert | dritte_bestaetigung | erinnerung
    quelle: "Arbeitszeugnis 2022, S. 1"
    zeugnis_datei: "profil/dokumente/zeugnisse/2022_arbeitszeugnis_ag.pdf"
  tags: ["SAP EWM", "Prozessoptimierung", "Projektleitung"]
  kompetenzbereich: "Prozess/Effizienz"
  sprachvarianten:
    en: "Reduced cycle time by 18% within six months during an SAP EWM rollout."
```

`beleg.status` unterscheidet, worauf du dich im Ernstfall stützen kannst: `dokumentiert` (Zeugnis, Projektbericht), `dritte_bestaetigung` (Referenz, aber kein Papier) oder `erinnerung` (nur dein Gedächtnis). Der Kritiker verlangt für harte Zahlen im Anschreiben keinen bestimmten Status, aber der Status wandert ins Tailoring-Log (Kapitel 11.7) und ist relevant, falls du im Vorstellungsgespräch nachweisen musst, was im Anschreiben steht – die rechtliche Grenze zur arglistigen Täuschung nach § 123 BGB behandelt Kapitel 16. `ca: true` erzwingt eine Rundungsformulierung im Text, nie eine präzisere Zahl als in der Story-Bank hinterlegt (Kapitel 11.7). `sprachvarianten.en` ist optional, aber ohne eigene englische Formulierung ist die Stimmtreue bei englischen Bewerbungen geringer (Kapitel 11.11).

**Entscheidung:** Story-Bank-IDs sind stabil, kurz (`S01`, `S02`, …) und werden nie geändert oder wiederverwendet, auch wenn ein Eintrag veraltet. **Begründung:** Claims, Tailoring-Logs und Kritikberichte referenzieren die ID dauerhaft; eine geänderte ID würde alte Nachweise entwerten. **Alternative:** Sprechende Slugs (`s-sap-rollout-2022`) – lesbarer, aber instabiler bei Umbenennungen.

### 8.4 Stimmprofil (`stimmprofil.md`)

Das Stimmprofil ist Markdown, keine YAML-Struktur, weil es überwiegend aus Fließtext und Beispielen besteht, die eine strikte Schema-Form nur verschlechtern würde.

**Inhalt:**

1. **Grundhaltung**: wie du grundsätzlich über dich schreibst (zurückhaltend, direkt, humorvoll-sachlich, …), in eigenen Worten.
2. **Anrede-Präferenz**: Standard Du/Sie, wenn die Anzeige keine klare Präferenz zeigt (Default: Sie, siehe Kapitel 11.4); Ausnahmefälle, in denen du bewusst „Du" erlaubst.
3. **Grußformeln**: zwei bis drei Varianten, die du tatsächlich benutzt, nicht „Mit freundlichen Grüßen" als einzige Option, wenn du eigentlich variierst.
4. **Beispieltexte**: 5 bis 10 echte, von dir geschriebene Texte (alte Anschreiben, berufliche E-Mails, LinkedIn-Beiträge), jeweils mit Datum und Anlass. Sie sind die wichtigste Grundlage, weil Stiltransfer aus wenigen informellen Proben laut Forschung unsicherer ist als aus einem größeren Korpus (8.1) – mehr Proben schließen die Lücke eher als mehr Regeln.
5. **Lieblingswörter und -wendungen**: Formulierungen, die zu dir passen und im Zweifel bevorzugt werden.
6. **Tabus**: Wörter oder Muster, die nie vorkommen dürfen, mit fester ID `NUTZER-001`, `NUTZER-002`, … Diese IDs fließen direkt in den Anti-Generik-Katalog des Kritikers ein (Kapitel 11.5) und werden dort als harte Gates behandelt.
7. **Satzrhythmus-Zielkorridor**: aus den Beispieltexten errechnete mittlere Satzlänge und Standardabweichung, als Zahlenpaar hinterlegt; der Kritiker prüft neue Entwürfe gegen diesen Korridor statt gegen einen generischen Zielwert (Kapitel 11.5).
8. **Englisches Pendant** (falls vorhanden): eigene englische Textproben oder, in Abwesenheit, die Anweisung, im Englischen direkter zu formulieren als im Deutschen (Kapitel 11.11).

```yaml
# eingebetteter Kopf in stimmprofil.md, maschinenlesbarer Teil
tabus:
  - id: "NUTZER-001"
    muster: '\bspannend\w*\b'
    schwere: "hart"
satzlaenge_stichprobe:
  mittel: 14.2
  stdev: 6.8
  n_proben: 8
  berechnet_am: "2026-09-01"
anrede_default: "Sie"
```

Die Ableitung folgt demselben Grundmuster wie ein Skill zur Stilerfassung aus gesendeten Nachrichten und Dokumenten: Proben sammeln, Stilmerkmale analysieren (Satzlänge, Wortwahl, Struktur), einen Entwurf vorlegen, den du korrigierst, statt ihn stillschweigend zu übernehmen. Details zum Ablauf in 8.9. Der iterative Rhythmus – Kontext sammeln, Optionen entwerfen, vom Nutzer kuratieren lassen, mit einem unabhängigen Leser-Test prüfen – folgt dem Muster, das Anthropics eigener `doc-coauthoring`-Skill für Ko-Autorenschaft beschreibt ([doc-coauthoring/SKILL.md](https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md)).

### 8.5 Präferenzen und Ausschlüsse (`praeferenzen.yaml`)

```yaml
# profil/praeferenzen.yaml
rollen:
  zieltitel: ["...", "..."]             # Suchbegriffe für den Scout (Kapitel 9)
orte:
  staedte: ["..."]
  remote_ok: true
  max_pendelzeit_minuten: 45            # Muss-Filter im Matcher (Kapitel 9)
gehalt:
  waehrung: "EUR"
  spanne_von: 0
  spanne_bis: 0
  zielwert: 0
  basis: "brutto_jahr"
  darf_geschaetzt_werden: false          # nie automatisch schätzen (Kapitel 5, 11.4)
sprachen:
  sprache_erforderlich: ["de", "en"]     # Sprachen, in denen du arbeiten willst/kannst
ausschluesse:
  branchen: []
  firmen_blacklist: []
  vertragsart_ausgeschlossen: ["Zeitarbeit"]
wechselmotiv:
  allgemein: "dein ehrlicher Grund, in eigenen Worten"
  ueber_aktuellen_arbeitgeber_erlaubt: "nichts, außer explizit unten freigegeben"
  freigegebene_aussagen: []
```

`max_pendelzeit_minuten`, `sprache_erforderlich` und `vertragsart_ausgeschlossen` sind die Felder, die der Matcher als harten Muss-Filter liest (Kapitel 9); Änderungen hier wirken sofort auf die Tagesauswahl. `wechselmotiv.allgemein` ist die einzige zulässige Quelle für Sätze über deinen Wechselgrund im Anschreiben (Kapitel 11.4); ohne Eintrag bleibt dieser Punkt im Text leer statt erraten. Foto-, Geburtsdatum- und Familienstand-Präferenzen liegen bereits in `lebenslauf.yaml` (8.2), weil sie direkt am Dokument hängen; die AGG-Begründung für den Default „aus" steht in Kapitel 5 und wird vom Setzer umgesetzt (Kapitel 13.6).

### 8.6 Standardantworten für Formulare (`standardantworten.yaml`)

Bewerbungsformulare bei ATS-Systemen wie SAP SuccessFactors oder Workday enthalten regelmäßig K.-o.-Fragen (Ja/Nein oder Dropdown zu Arbeitserlaubnis, Gehalt, Verfügbarkeit), die vor jeder menschlichen Prüfung disqualifizieren können, wenn sie falsch oder gar nicht beantwortet sind ([QuickCV: ATS Knockout Questions](https://quickcv.io/blog/ats-knockout-questions), [onapply: K.-o.-Fragen](https://www.onapply.de/recruiting-wissen/k-o-fragen)). Ein generischer Platzhalter reicht nicht, weil diese Fragen oft harte Ausschlusskriterien des Arbeitgebers abbilden.

```yaml
# profil/standardantworten.yaml
kuendigungsfrist_wochen: 12
fruehester_eintritt: "2026-12-01"       # oder Bezug zur Kündigungsfrist
arbeitserlaubnis: "EU-Staatsbürger, keine Einschränkung"
umzugsbereitschaft: false
reisebereitschaft_prozent: 10
fuehrerschein: ["B"]
wie_haben_sie_von_uns_erfahren: "Jobportal / Recherche"
schwerbehinderung_angeben: false          # Default aus, sensibel (Kapitel 16)
```

Gehalt und Eintrittstermin dürfen im Anschreiben nur erscheinen, wenn die Anzeige explizit danach fragt, und immer aus diesem Feld beziehungsweise aus `praeferenzen.gehalt`, niemals geschätzt ([bewerbung.net](https://bewerbung.net/gehaltsvorstellung-bewerbung), [JobTeaser](https://www.jobteaser.com/de/advices/gehaltsvorstellung-in-der-bewerbung-formulieren-so-geht-s), [Karrierebibel](https://karrierebibel.de/bewerbung-eintrittstermin-nennen-sofort/)); fehlt ein benötigtes Feld, entsteht eine Rückfrage statt eines Platzhaltertexts (Kapitel 11.4). `schwerbehinderung_angeben` und vergleichbare AGG-sensible Felder sind grundsätzlich deaktiviert und werden nur nach ausdrücklicher Entscheidung von dir aktiviert; die rechtliche Einordnung solcher Angaben behandelt Kapitel 16.

### 8.7 Dokumentenablage: Zeugnisse, Zertifikate, Foto, Unterschrift

```text
profil/dokumente/
├── zeugnisse/                     # Originale, versioniert in Git
│   ├── 2022_arbeitszeugnis_ag.pdf
│   └── 2024_abschlusszeugnis.pdf
├── unterschrift.png               # optional, transparenter Hintergrund
└── foto.jpg                       # optional, Default: nicht verwendet
```

Die Originale bleiben unverändert im Profil; komprimierte, durchsuchbare Fassungen erzeugt der Setzer einmalig beim Onboarding und cacht sie außerhalb von Git, weil sie aus den Originalen regenerierbar sind (Kapitel 13.8, ergänzt `profil/dokumente/.cache/` in `.gitignore` neben der SQLite-Datenbank aus Kapitel 7.9). Für die Auswahl pro Bewerbung (letzte zwei bis drei Arbeitszeugnisse, höchster Abschluss, stellenrelevante Zertifikate) und die technische Aufbereitung gilt Kapitel 13.8 vollständig; hier zählt nur, was du beim Onboarding bereitstellen musst.

Zwei Punkte gehören zwingend in die Checkliste vor dem ersten Bewerbungslauf, nicht erst wenn eine passende Stelle gefunden ist:

- **Ausländischer Bildungsabschluss:** Prüfe den Status in der Datenbank Anabin; fehlt ein Eintrag oder ist er unklar (Status ungleich H+), ist eine kostenpflichtige Zeugnisbewertung bei der Zentralstelle für ausländisches Bildungswesen (ZAB) nötig, die derzeit mehrere Monate dauert ([Anabin-Kurzanleitung](https://anabin.kmk.org/kurzanleitung/ich-moechte-feststellen-wie-mein-auslaendischer-hochschulabschluss-in-deutschland-bewertet-wird.html), [KMK zur Anerkennung](https://www.kmk.org/themen/anerkennung-auslaendischer-abschluesse.html)). Der `anabin_status` in `lebenslauf.yaml` (8.2) hält das Ergebnis fest.
- **Fremdsprachige Zeugnisse:** Nur mit vorliegender beglaubigter Übersetzung durch vereidigte Übersetzer verwenden; der Agent erstellt selbst höchstens einen Rohentwurf und kennzeichnet ihn nie als beglaubigt (Kapitel 13.8, [mentorium.de](https://www.mentorium.de/zeugnisse-beglaubigt-uebersetzen/)).

Das Unterschriftsbild ist ein sensibles Asset: Es verlässt `profil/dokumente/` nie in Richtung eines Prompts oder Modellkontexts und wird nie in einer Vorschau des Review-Cockpits verschickt; nur das Setzer-Werkzeug liest es direkt von der Platte (Kapitel 13.9, 16).

### 8.8 Onboarding-Interview: Fragenkatalog

Der Katalog wird einmalig vor dem ersten Tageslauf durchgegangen und danach bei Bedarf ergänzt (8.10). Er ist absichtlich in Blöcke geteilt: Ein Block pro Sitzung ist realistischer als alles an einem Abend.

**A – Kontakt und Formalia**

1. Name, Kontaktdaten, LinkedIn-/XING-Profil-Link (→ `basics`).
2. Soll ein Foto verwendet werden, trotz AGG-Empfehlung dagegen (→ `foto_verwenden`)?
3. Geburtsdatum und Familienstand im Lebenslauf zeigen (→ `geburtsdatum_anzeigen`, `familienstand_anzeigen`)?
4. Liegt ein ausländischer Bildungsabschluss vor? Ist der Anabin-Status bekannt (→ `anabin_status`)?
5. Liegen fremdsprachige Zeugnisse als beglaubigte Übersetzung vor, oder ist das noch offen (→ Checkliste 8.7)?

**B – Werdegang (Master-Lebenslauf)**

6. Alle Stationen mit Firma, Titel, Zeitraum, Ort, vollständig, auch die, die du normalerweise weglässt.
7. Gibt es Stationen mit einem internen Titel, der nach außen unverständlich ist? Welcher marktübliche Titel ist als Alias vertretbar (→ `titel_alias`)?
8. Für jede Station: drei bis fünf Bullets, was du konkret getan hast, nicht nur die Stellenbeschreibung.
9. Gibt es zeitliche Lücken? Wie sollen sie benannt werden (→ `luecken`)?
10. Alle Aus- und Weiterbildungen mit Institution, Abschluss, Zeitraum.
11. Alle Kenntnisse mit ehrlichem Niveau; welche Synonyme oder Kurzformen verwendest du oder erwartest du in Anzeigen (→ `synonyme`)?
12. Sprachkenntnisse mit GER-Stufe und dem umgangssprachlichen Begriff, den du selbst benutzt.
13. Zertifikate mit Aussteller und Jahr, samt Zeugnis-Datei.

**C – Erfolge (Story-Bank)**

14. Nenne 8 bis 10 Erfolge aus verschiedenen Phasen und Kompetenzbereichen.
15. Für jeden: Was war die Ausgangslage, was war deine konkrete Aufgabe?
16. Was genau hast du getan – dein Anteil, nicht das Team pauschal?
17. Welches messbare Ergebnis kam heraus, mit welcher Zahl, welcher Einheit, welchem Zeitraum?
18. Worauf stützt sich die Zahl – Zeugnis, Projektdokument, nur Erinnerung (→ `beleg.status`)?
19. Welche Erfolge lassen sich auch auf Englisch ehrlich und ohne Übertreibung formulieren?

**D – Stimme und Schreibstil**

20. Schicke 5 bis 10 echte Texte: alte Anschreiben, berufliche E-Mails, LinkedIn-Beiträge.
21. Bevorzugst du grundsätzlich Sie oder Du, wenn die Anzeige keine klare Präferenz zeigt?
22. Welche Grußformeln verwendest du tatsächlich?
23. Welche Wörter oder Wendungen sind typisch für dich?
24. Welche Wörter oder Muster sollen nie vorkommen (→ Tabus, `NUTZER-*`)?
25. Gibt es Formulierungen aus früheren Bewerbungen, die du nie wieder lesen willst?
26. Schreibst du auf Englisch anders (direkter, kürzer) als auf Deutsch? Gibt es englische Textproben?
27. Wie viel Kontrolle über die Stilregeln willst du sichtbar im Prompt haben, auch wenn mehr Regeln den Text steifer machen können (Kapitel 11.2)?

**E – Präferenzen: Rollen, Orte, Gehalt**

28. Welche Jobtitel oder Rollen sind Zielsuchbegriffe?
29. Welche Städte oder Regionen, und ist Remote-Arbeit akzeptabel?
30. Wie viel Pendelzeit ist maximal akzeptabel?
31. Gehaltsspanne und Zielwert, brutto oder netto, jährlich oder monatlich?
32. Darf eine Gehaltsspanne aus Marktdaten vorgeschlagen werden, oder soll immer aktiv nachgefragt werden?
33. In welchen Sprachen willst oder kannst du arbeiten?
34. Was ist dein ehrlicher Wechselmotiv-Text, und was darfst du öffentlich über deinen aktuellen Arbeitgeber sagen?

**F – Ausschlüsse und rote Linien**

35. Gibt es Branchen oder einzelne Firmen, die grundsätzlich ausgeschlossen sind?
36. Welche Vertragsarten kommen nicht in Frage (Zeitarbeit, befristet, …)?
37. Sollen bei rechtlichen Grauzonen im Lebenslauf-Tailoring immer Rückfragen an dich gestellt werden, oder legst du selbst eine rote Linie fest, die der Agent automatisch respektiert?
38. Soll standardmäßig ein Motivationsschreiben zusätzlich erstellt werden, auch wenn die Anzeige es nicht fordert?

**G – Standardantworten für Formulare**

39. Aktuelle Kündigungsfrist und frühester Eintrittstermin.
40. Arbeitserlaubnis-Status.
41. Umzugsbereitschaft und Reisebereitschaft in Prozent.
42. Führerscheinklassen.
43. Standardantwort auf „Wie haben Sie von uns erfahren?"
44. Sollen AGG-sensible Angaben wie Schwerbehinderung überhaupt abgefragt und gespeichert werden?

**H – Betrieb und Datenschutz**

45. Nutzt du bereits ein gepflegtes LinkedIn- oder XING-Profil, das als Ausgangsbasis dienen kann?
46. Soll das System dich vor dem ersten Lauf auf Inkonsistenzen zwischen Lebenslauf und Profil hinweisen?
47. Wie oft willst du die Story-Bank aktiv erweitern – nach jedem neuen Erfolg, oder in festen Abständen?
48. Wer außer dir soll jemals Lese- oder Schreibzugriff auf `profil/` bekommen?
49. Reicht dir ein lokales Backup des Daten-Repositorys, oder soll zusätzlich ein verschlüsseltes Offsite-Backup existieren (Kapitel 7)?
50. Digitale Signaturvorlage vorhanden? Soll sie automatisch eingefügt werden?

### 8.9 Ablauf des Onboardings

**Entscheidung:** Das Onboarding läuft als geführte, interaktive Claude-Code-Sitzung, nicht über das Review-Cockpit. **Begründung:** Ein mehrstufiges Interview mit Rückfragen, Dateien lesen (alte Anschreiben), Dateien schreiben (Profil-Entwürfe) und Fortschritt sichtbar machen ist im MVP-Cockpit (Kapitel 14: Freigabe, Diff, Rückfragen) nicht vorgesehen und würde es unnötig aufblähen. **Alternative:** Ein Webformular im Cockpit für Wiederholungs-Updates einzelner Felder ohne Chat – sinnvoll ab v1, sobald das Cockpit steht (Kapitel 7.10), aber kein Ersatz für das erste, ausführliche Interview.

Ablauf in sechs Schritten, angelehnt an das Muster aus Anthropics `doc-coauthoring`-Skill (Kontext sammeln, iterativ entwerfen, mit einem unabhängigen Test prüfen, [SKILL.md](https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md)):

1. **Vorbereitung:** Du legst alte Anschreiben, E-Mails, den aktuellen Lebenslauf und Zeugnis-Scans in einen Ordner; ein Skill `.claude/skills/profil-onboarding/SKILL.md` führt Block für Block durch 8.8.
2. **Stimmprofil-Extraktion als eigener Teilschritt:** Ein zweiter Skill (analog zum Muster, nach dem eine Stilerfassung aus gesendeten Nachrichten funktioniert) liest die eingereichten Textproben, berechnet Satzlängen-Statistik und häufige Wendungen, und legt einen Rohentwurf für `stimmprofil.md` vor – nie als Endversion, immer als Vorschlag zur Korrektur.
3. **Entwurf:** Claude Code schreibt Entwürfe für alle fünf Dateien direkt in `profil/`. Das ist der einzige Moment, in dem ein Modell in dieses Verzeichnis schreibt, weil du in derselben Sitzung unmittelbar danebensitzt – kein autonomer Tageslauf, keine unbeobachtete Ausführung.
4. **Prüfung:** Du liest jede Datei, korrigierst Wortlaut, Zahlen und Tabus direkt. Der Kritiker (Kapitel 11) macht an dieser Stelle noch nichts; er prüft erst Bewerbungstexte, nicht das Profil selbst.
5. **Freigabe durch Commit:** Ein Git-Commit im Daten-Repository ist die Freigabe (8.10). Ab diesem Zeitpunkt gilt wieder uneingeschränkt die Regel aus Kapitel 7.3: Kein Modell schreibt mehr in `profil/`; Änderungsvorschläge aus der Feedback-Schleife (Kapitel 14) landen als `profilvorschlag` und warten auf deine Bestätigung.
6. **Story-Bank-Reserve:** Mit weniger als 8 Einträgen oder ohne Beleg-Status läuft der erste Tageslauf trotzdem, aber der Kritiker markiert entsprechende Bewerbungen als dünn belegt; das ist ein bewusster Kompromiss für einen schnellen Start, keine Dauerlösung.

### 8.10 Pflege, Versionierung, Datenschutz

**Versionierung:** Jede Änderung an `profil/` ist ein Git-Commit im privaten Daten-Repository (Kapitel 7.9), niemals im veröffentlichbaren Code-Repository. Nach jedem Commit aktualisiert ein kleines Skript die Tabelle `candidate_profile` (Kapitel 7.3): Version hochzählen, SHA-256-Hashes von `lebenslauf.yaml`, `story_bank.yaml` und `stimmprofil.md` neu berechnen, aktuellen Inhalt von `praeferenzen.yaml` und `standardantworten.yaml` als JSON in die entsprechenden Spalten kopieren. So weiß jede erzeugte Bewerbung, mit welcher Profilversion sie entstand, ohne dass die Datenbank die Profildateien selbst redundant vorhält.

**Wann aktualisieren:** Nach jedem neuen nennenswerten Erfolg ein Story-Bank-Eintrag ergänzen, nicht erst wenn eine passende Stelle gefunden ist – frisch erinnerte Details sind belegbarer. Nach jeder Beförderung, jedem Jobwechsel oder jeder neuen Qualifikation den Master-Lebenslauf aktualisieren. Das Stimmprofil ändert sich seltener; eine Überarbeitung lohnt sich, wenn der Kritiker wiederholt dieselbe Art von Korrektur meldet (Feedback-Schleife, Kapitel 14).

**Datenschutz:** `profil/` liegt ausschließlich im privaten Daten-Repository auf deinem eigenen Server oder Rechner (Kapitel 7.9), nie in einem SaaS-Werkzeug eines Dritten und nie im Code-Repository, das später veröffentlicht werden könnte. Eine etwaige Notion- oder Kanban-Spiegelung des Cockpits (Kapitel 14) überträgt nur Status und Metadaten einzelner Bewerbungen, nie die Profildateien selbst. Master-Lebenslauf, Story-Bank und Stimmprofil gehen als Kontext in jeden Autor- und Kritiker-Aufruf ein; läuft dieser Aufruf über Claude Fable 5.1, gilt dafür die in Kapitel 7 und 16 beschriebene 30-Tage-Datenspeicherung bei Anthropic als bewusst in Kauf genommene Bedingung, nicht als Standardverhalten der API. Unterschriftsbild und Foto sind davon ausgenommen: Sie erreichen nach 8.7 nie einen Prompt.

### 8.11 Default-Annahmen und offene Fragen

Bis du anders entscheidest (Fragenkatalog Kapitel 22): Story-Bank-Ziel zehn Einträge vor dem ersten produktiven Lauf; Anrede-Default „Sie"; AGG-sensible Felder (Foto, Geburtsdatum, Familienstand, Schwerbehinderung) grundsätzlich deaktiviert; Gehalt wird nie automatisch aus Marktdaten geschätzt, sondern muss von dir gepflegt oder aktiv erfragt werden; das Onboarding läuft als Claude-Code-Sitzung, nicht als Cockpit-Formular.

Offene Fragen an dich, gesammelt in Kapitel 22: Wie viele echte Textproben kannst du für das Stimmprofil bereitstellen, und dürfen sie dauerhaft gespeichert werden? Liegt ein ausländischer Bildungsabschluss vor, und ist der Anabin-Status bereits bekannt? Wie streng sollen die eigenen Tabus im Anti-Generik-Katalog durchgesetzt werden? Soll ein Motivationsschreiben standardmäßig zusätzlich entstehen? Wie oft willst du die Story-Bank aktiv pflegen?

**Quellen dieses Kapitels:**

- Anthropic: Reduce hallucinations – https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
- Anthropic: doc-coauthoring Skill – https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md
- Catch Me If You Can? Not Yet (Few-Shot-Stiltransfer, ACL Findings 2025) – https://arxiv.org/pdf/2509.14543
- Ergänzende Studie zum Stiltransfer – https://arxiv.org/abs/2509.24930
- CareerScribe: STAR Story Bank Template – https://blog.careerscribeai.com/star-story-bank-template/
- JSON Resume – https://jsonresume.org/
- RenderCV – https://rendercv.com/
- QuickCV: ATS Knockout Questions – https://quickcv.io/blog/ats-knockout-questions
- onapply: K.-o.-Fragen – https://www.onapply.de/recruiting-wissen/k-o-fragen
- bewerbung.net: Gehaltsvorstellung in der Bewerbung – https://bewerbung.net/gehaltsvorstellung-bewerbung
- JobTeaser: Gehaltsvorstellung formulieren – https://www.jobteaser.com/de/advices/gehaltsvorstellung-in-der-bewerbung-formulieren-so-geht-s
- Karrierebibel: Eintrittstermin nennen – https://karrierebibel.de/bewerbung-eintrittstermin-nennen-sofort/
- Anabin-Kurzanleitung (KMK) – https://anabin.kmk.org/kurzanleitung/ich-moechte-feststellen-wie-mein-auslaendischer-hochschulabschluss-in-deutschland-bewertet-wird.html
- KMK: Anerkennung ausländischer Abschlüsse – https://www.kmk.org/themen/anerkennung-auslaendischer-abschluesse.html
- mentorium.de: Zeugnisse beglaubigt übersetzen – https://www.mentorium.de/zeugnisse-beglaubigt-uebersetzen/
- Karrierebibel: Sprachkenntnisse im Lebenslauf – https://karrierebibel.de/sprachkenntnisse-lebenslauf/
