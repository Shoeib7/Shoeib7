## 9. Modul Scout und Matcher: Suche, Normalisierung, Dedup, Scoring, Tagesauswahl

Scout und Matcher sind die ersten beiden Stationen der Status-Pipeline (Kapitel 3). Der Scout überführt eine Stellenanzeige von „nicht im System“ zu „entdeckt“, entfernt Duplikate (Status „dedupliziert“) und normalisiert sie in ein festes Schema. Der Matcher bewertet jede normalisierte Anzeige (Status „bewertet“) und wählt daraus die Tagesauswahl (Status „ausgewählt“), die anschließend an den Rechercheur übergeht (Kapitel 10). Die Quellenmatrix selbst — welche Jobbörse, welches ATS-Feed, welche API — ist Gegenstand von Kapitel 6; dieses Kapitel beschreibt, wie der Scout diese Quellen technisch anzapft, und wie der Matcher aus der Masse der gefundenen Anzeigen zehn belastbare Kandidaten für den Menschen herausfiltert.

### 9.1 Scout: Quellen-Adapter, Rate Limits, inkrementelles Crawlen

**Entscheidung:** Der Scout besteht aus einem festen Adapter-Interface (eine Funktion `fetch(seit_cursor) -> Liste[RohAnzeige]` pro Quelle) und je einem Adapter für BA-Jobsuche-API, die direkten ATS-Feeds (Greenhouse, Lever, Personio, Recruitee, SmartRecruiters) für eine vom Nutzer gepflegte Watchlist an Wunscharbeitgebern, sowie SerpAPI/Google for Jobs und Adzuna/Arbeitnow als breite Ergänzung. **Begründung:** Jede Quelle hat ein eigenes Antwortformat, eigene Fehlerbilder und eigene Kostenlogik; ein einheitliches Interface hält den Rest der Pipeline (Dedup, Extraktion, Scoring) quellenunabhängig. **Alternative:** ein einzelner Meta-Scraper über Drittanbieter (JSearch, Bright Data) — verworfen, weil er das Scraping-Risiko nur verlagert, nicht reduziert, und die inoffiziellen ATS-Feeds bereits ToS-unbedenklich und kostenlos sind.

Jeder Adapter arbeitet inkrementell, nicht als Vollcrawl:

- **BA-Jobsuche-API**: `X-API-Key: jobboerse-jobsuche`, Endpunkt `/pc/v6/jobs`. Der Parameter `veroeffentlichtseit` (Tage seit Veröffentlichung) begrenzt die tägliche Abfrage auf neue/aktualisierte Anzeigen; ein wöchentlicher Lauf mit größerem Fenster fängt nachträglich geänderte Einträge ab. Die API ist inoffiziell und ohne SLA — GitHub-Issues zeigen bereits Schema-Brüche (v4→v6) und temporäre 403-Fehler [bundesAPI/jobsuche-api](https://github.com/bundesAPI/jobsuche-api). Der Adapter braucht defensives Error-Handling (Retry mit Backoff, Circuit-Breaker) statt der Annahme, die API sei stabil.
- **ATS-Feeds** (Greenhouse `boards-api.greenhouse.io/v1/boards/{token}/jobs`, Lever `api.lever.co/v0/postings/{site}?mode=json`, Personio `{firma}.jobs.personio.de/xml`, Recruitee `{firma}.recruitee.com/api/offers`, SmartRecruiters `api.smartrecruiters.com/v1/companies/{id}/postings`) liefern pro Firma die vollständige aktuelle Liste ohne Auth [Greenhouse-Doku](https://developers.greenhouse.io/job-board.html), [Lever-Postings-API](https://github.com/lever/postings-api), [Personio-XML-Integration](https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML), [Recruitee-Careers-API](https://docs.recruitee.com/reference/intro-to-careers-site-api), [SmartRecruiters-Posting-API](https://developers.smartrecruiters.com/docs/posting-api). Bei einer Watchlist von einigen Dutzend Firmen ist ein täglicher Vollabruf pro Firma unproblematisch; „inkrementell“ heißt hier: Abgleich der zurückgegebenen `source_id`-Liste mit dem letzten Lauf, um neue, unveränderte und verschwundene (vermutlich besetzte) Anzeigen zu unterscheiden.
- **Workday** liefert einen POST-Endpunkt `/wday/cxs/{tenant}/{site}/jobs`, hart auf 20 Ergebnisse pro Seite begrenzt; nur durch ein Community-Projekt belegt, nicht offiziell dokumentiert [Workday-Source-Guide](https://github.com/Francis1998/agentic-career-search/blob/main/docs/guides/WORKDAY_SOURCE_GUIDE.md) — als Adapter mit „kann sich jederzeit ändern“-Kennzeichnung einplanen.
- **Google for Jobs via SerpAPI** wird nicht als Crawl, sondern als budgetierte Anzahl gezielter Suchanfragen pro Tag betrieben (Freikontingent 250 Suchen/Monat, danach ab 25 USD/1.000) [SerpAPI Google Jobs API](https://serpapi.com/google-jobs-api) — Suchbegriffe kommen aus dem Kandidatenprofil (Kapitel 8), nicht aus einem offenen Vollindex.
- **Adzuna** (App-ID/Key, DE/AT) und **Arbeitnow** (kein Auth, Fokus englischsprachige Tech-Jobs) ergänzen als schmale, saubere Zusatzquellen [Adzuna-Developer-Portal](https://developer.adzuna.com/), [Arbeitnow-API](https://arbeitnow.com/api/job-board-api).

**Rate Limits im Überblick:**

| Quelle | Limit laut Recherche | Umgang im Scout |
|---|---|---|
| BA-Jobsuche-API | keine dokumentierten Limits, aber instabile inoffizielle API | eigenes Soft-Limit, Retry mit Backoff, Circuit-Breaker |
| Lever Postings API | 429 nur bei über 2 Bewerbungs-POSTs/Sekunde | unkritisch, Scout nutzt nur GET |
| Google for Jobs via SerpAPI | Kontingent nach Tarif, kein Zeitlimit | Suchanfragen budgetiert planen, nicht pro Anzeige |
| Jooble | 500 Requests lebenslang pro Key [Jooble-API](https://jooble.org/api/about) | für Tagesbetrieb ungeeignet, nicht einsetzen |

Ein hartes, im Code erzwungenes Limit ist an anderer Stelle wichtig: Der Handelsregister-Scraper, den der Rechercheur (Kapitel 10) für Firmenprüfungen nutzt, begrenzt sich selbst auf 60 Abfragen/Stunde und warnt explizit vor möglicher Strafbarkeit nach §§ 303a/303b StGB bei Massenabfragen [bundesAPI/handelsregister](https://github.com/bundesAPI/handelsregister) — das Limit muss als Rate-Limiter im Code stehen, nicht nur als Kommentar.

### 9.2 Normalisiertes Anzeige-Schema

Jede Rohanzeige wird unabhängig von ihrer Quelle in ein festes Schema überführt, bevor sie in die Dedup- und Scoring-Stufen geht:

```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "source": {"type": "string", "enum": ["ba_jobsuche", "greenhouse", "lever", "personio", "recruitee", "smartrecruiters", "workday", "serpapi_google_jobs", "adzuna", "arbeitnow", "manuell"]},
    "source_id": {"type": "string"},
    "url": {"type": "string"},
    "title_raw": {"type": "string"},
    "company_name_raw": {"type": "string"},
    "location_city": {"type": "string"},
    "location_country": {"type": "string"},
    "remote_type": {"type": "string", "enum": ["vor_ort", "hybrid", "vollremote", "unklar"]},
    "employment_type": {"type": "string", "enum": ["vollzeit", "teilzeit", "ausbildung_dual", "praktikum_trainee", "werkstudent", "selbststaendig", "unklar"]},
    "contract_type": {"type": "string", "enum": ["befristet", "unbefristet", "unklar"]},
    "salary_min": {"type": ["number", "null"]},
    "salary_max": {"type": ["number", "null"]},
    "salary_currency": {"type": ["string", "null"]},
    "salary_period": {"type": "string", "enum": ["jahr", "monat", "stunde", "unbekannt"]},
    "salary_is_estimated": {"type": "boolean"},
    "must_haves": {"type": "array", "items": {"type": "string"}},
    "nice_to_haves": {"type": "array", "items": {"type": "string"}},
    "contact_person_name": {"type": ["string", "null"]},
    "contact_person_role": {"type": ["string", "null"]},
    "contact_email": {"type": ["string", "null"]},
    "application_channel": {"type": "string", "enum": ["email", "ats_formular", "portal_upload", "unklar"]},
    "reference_number_raw": {"type": ["string", "null"]},
    "is_temp_agency_suspected": {"type": "boolean"},
    "temp_agency_signal": {"type": ["string", "null"]},
    "posted_at_text": {"type": ["string", "null"]},
    "language": {"type": "string", "enum": ["de", "en", "andere"]}
  },
  "required": ["source", "source_id", "url", "title_raw", "company_name_raw", "remote_type",
    "employment_type", "contract_type", "salary_period", "salary_is_estimated",
    "must_haves", "nice_to_haves", "application_channel", "is_temp_agency_suspected", "language"]
}
```

Um dieses Schema herum ergänzt der Code (nicht das Modell) interne Felder: `id` (eigener Primärschlüssel), `ats_type` (regelbasiert erkannt, siehe 9.3), `dedup_group_id`, `status` (Wert aus der Status-Pipeline, Kapitel 3), `first_seen_at`/`last_seen_at`, sowie die in 9.4–9.6 berechneten Score- und Erklärungsfelder. Kontaktdaten aus Anzeigen sind personenbezogene Daten; ihre Aufbewahrung und Löschung regelt Kapitel 16.

### 9.3 Extraktion strukturierter Felder per Structured Outputs

**Entscheidung:** Die Felder aus 9.2 werden pro Anzeige per Claude Structured Output (`output_config.format`, kein Beta-Header nötig) mit **Claude Haiku 4.5** ($1/$5 pro 1 Mio. Token Input/Output) extrahiert. **Begründung:** Es handelt sich um einen Massenschritt (200–500 Anzeigen/Tag laut Zielvolumen) mit vergleichsweise einfacher Aufgabe (Feldextraktion aus vorliegendem Text); der Kostenunterschied zu Fable 5.1 ($10/$50) ist bei diesem Volumen erheblich. **Alternative:** Bei Extraktionsfehlern (leere Pflichtfelder, Format-Ausreißer) eskaliert der Code den einzelnen Fall auf Claude Sonnet 5 als Fallback, statt das gesamte Volumen teurer zu fahren.

Wichtige Einschränkung von Claude Structured Outputs: Schemas dürfen kein `pattern` (Regex), keine `minLength`/`maxLength`- oder numerischen Min/Max-Constraints und keine rekursiven Strukturen enthalten; `additionalProperties` muss `false` sein [Claude Structured Outputs](https://docs.claude.com/en/docs/build-with-claude/structured-outputs). Das Schema in 9.2 hält sich daran — Formatprüfung (z. B. das Muster einer Kennziffer) läuft deshalb nachgelagert im Code:

```python
import re

KENNZIFFER_MUSTER = re.compile(r"^[A-Z0-9][A-Z0-9\-/.]{2,19}$")

def validiere_kennziffer(roh: str | None) -> str | None:
    if not roh:
        return None
    kandidat = roh.strip()
    return kandidat if KENNZIFFER_MUSTER.match(kandidat) else None
```

Einzelne Felder brauchen zusätzliche Logik über die reine LLM-Extraktion hinaus:

- **Muss/Kann**: Das Modell klassifiziert Anforderungen anhand von Formulierungssignalen („zwingend“, „Voraussetzung“, „muss“ vs. „von Vorteil“, „wünschenswert“, „idealerweise“) in `must_haves`/`nice_to_haves`. Uneindeutige Fälle bleiben in `nice_to_haves` — ein zu Unrecht als „Muss“ eingestuftes Kriterium würde später fälschlich zum Hard-Filter-Ausschluss führen.
- **Gehalt**: Da nur rund 12,5 % der deutschen Stellenanzeigen überhaupt ein Gehalt nennen und die EU-Entgelttransparenz-Richtlinie 2023/970 in Deutschland noch nicht umgesetzt ist (Frist 7.6.2026 verstrichen, nationales Gesetz frühestens 2027 erwartet) [Indeed Hiring Lab](https://www.hiringlab.org/de/blog/2025/03/05/gehaltsangaben-bleiben-in-deutschland-die-ausnahme/), [AFA Anwalt zur Entgelttransparenz](https://www.afa-anwalt.de/news/entgelttransparenzgesetz-2026-rl-2023-970/), braucht das System für die Mehrheit der Anzeigen einen Fallback: Schätzung über die Entgeltatlas-API der Bundesagentur nach KldB-Code, Region, Alter und Branche [bundesAPI/entgeltatlas-api](https://github.com/bundesAPI/entgeltatlas-api). `salary_is_estimated: true` markiert diesen Fall immer explizit — das Feld wird nie als Fakt ausgegeben, weder im Scoring noch im Review-Cockpit (Kapitel 14).
- **Remote**: `remote_type` wird aus Formulierungen („Homeoffice“, „100 % Remote“, „hybrid“, Bürostandort ohne Hinweis) abgeleitet; bei der BA-API liefert der Parameter `arbeitszeit=ho` einen zusätzlichen Strukturhinweis.
- **Ansprechpartner**: Neben der LLM-Extraktion aus dem Anzeigentext kann ergänzend spaCy `de_core_news_lg` (deutsches NER-Modell, 84,9 % NER-F1) Personennamen erkennen [spaCy de_core_news_lg](https://github.com/explosion/spacy-models/releases/tag/de_core_news_lg-3.8.0) — als zweite, günstige Prüfinstanz gegen offensichtliche Fehlextraktionen. Name und Rolle werden zweckgebunden nur für die konkrete Bewerbung gespeichert (Kapitel 16).
- **Bewerbungsweg** und **Kennziffer** steuern später den Boten (Kapitel 15): E-Mail-Adresse vorhanden → `application_channel: email`; ATS-URL-Muster erkannt → `ats_formular`.
- **ATS-Typ**: wird nicht vom Sprachmodell, sondern regelbasiert aus URL- und HTML-Mustern erkannt (Greenhouse `boards.greenhouse.io/*`, Lever `jobs.lever.co/*`, Personio `*.jobs.personio.de`, Workday `*.wd*.myworkdayjobs.com`, SmartRecruiters `careers.smartrecruiters.com/*`). Die Fingerprint-Datenbank von webappanalyzer deckt in ihrer Kategorie „Recruitment & staffing“ Personio, Greenhouse, Lever, Recruitee, SmartRecruiters, Teamtailor und onlyfy ab, aber **nicht** softgarden, rexx systems, d.vinci, SAP SuccessFactors oder JOIN [webappanalyzer](https://github.com/enthec/webappanalyzer) — für diese verbleibt vorerst nur ein eigener, manuell gepflegter URL-Musterabgleich. Die Priorisierung nach Marktanteil ist in Kapitel 4 begründet.

### 9.4 Dedup: Schlüssel, Ähnlichkeit, Schwellen

Cross-Board-Recherche zu Duplikatraten (Textkernel) nennt 50–80 % Duplikate über Jobbörsen hinweg [Textkernel-Forschung zu Duplikaten](https://dl.acm.org/doi/fullHtml/10.1145/3486622.3493928); reines Fuzzy-Matching auf Titel und Firma allein übersieht nach Fachquellen zusätzlich rund 30 % echter Duplikate, weil Firmennamen (Rechtsformzusätze, Schreibvarianten) und Jobtitel (synonyme Formulierungen) stark variieren. Dedup ist deshalb dreistufig aufgebaut, jede Stufe billiger und robuster als eine Volltext-Ähnlichkeitssuche über alle Paare:

1. **Exakte ID** — sofern vorhanden, zählt sie als Beweis: BA-`refnr`, Greenhouse-/Lever-Job-ID, oder Domain+Pfad der `absolute_url`. Zwei Anzeigen mit gleicher ID sind dieselbe Anzeige, unabhängig von Textabweichungen.
2. **Blocking-Schlüssel** — normalisierte Firma (Rechtsformen wie GmbH, AG, KG, mbH, SE entfernt, klein geschrieben) plus normalisierter Titel plus PLZ/Ort. Innerhalb eines Blocks: Jaro-Winkler-Ähnlichkeit auf dem Titel ab **0,90** in Kombination mit exakter Firma+Ort-Übereinstimmung gilt als automatischer Merge.
3. **MinHashLSH auf dem Beschreibungstext** — für Fälle ohne gemeinsame ID und ohne exakten Blocking-Treffer (z. B. dieselbe Stelle über zwei verschiedene Jobbörsen mit leicht abweichendem Firmennamen). **Entscheidung:** `datasketch` mit `num_perm=128`, Jaccard-Schwelle **0,75–0,85** für automatischen Merge [datasketch](https://github.com/ekzhu/datasketch). **Begründung:** kostenlos, MIT-lizenziert, kein Trainingsdatenbedarf — passend für ein Einzelnutzer-Volumen von einigen Hundert Anzeigen/Tag. **Alternative:** `recordlinkage` für stärker feldbasierte statt volltextbasierte Vergleiche [recordlinkage](https://github.com/J535D165/recordlinkage); `dedupe` (Active Learning) wird bewusst **nicht** eingesetzt, weil es gelabelte Trainingspaare braucht, die bei diesem Volumen unverhältnismäßig sind [dedupe](https://github.com/dedupeio/dedupe).

Ein Grenzband (Jaro-Winkler 0,75–0,90 bzw. MinHash-Jaccard 0,50–0,75) wird **nicht** automatisch zusammengeführt, sondern als „mögliches Duplikat“ markiert und im Review-Cockpit (Kapitel 14) sichtbar gemacht — Qualität vor Quantität heißt hier: lieber eine doppelt gezeigte Anzeige als eine fälschlich verworfene Chance. Die konkreten Schwellenwerte sind Startwerte und werden während der MVP-Phase (Kapitel 19) anhand echter Fehlklassifikationen kalibriert.

### 9.5 Scoring-Modell: harte Filter und gewichtete Dimensionen

Bevor eine Anzeige einen kostenpflichtigen API-Aufruf auslöst, durchläuft sie ein Boolean-Gate aus harten Muss-Filtern — Sprache, maximale Pendelzeit/-distanz, Vertragsart, Arbeitserlaubnis. Die Schwellenwerte selbst kommen aus den Präferenzen des Kandidatenprofils (Kapitel 8); der Matcher wendet sie nur an, pflegt sie aber nicht. Ein zusätzlicher harter Ausschluss ist sicherheitsbedingt, nicht qualitätsbedingt: Kontaktaufnahme ausschließlich über WhatsApp/Telegram oder die Forderung nach Video-Ident-Verfahren bzw. Kontoeröffnung vor Vertragsabschluss gilt als Job-Scamming-Signal und führt zum sofortigen Ausschluss, unabhängig vom sonstigen Score — seriöse Arbeitgeber verlangen das laut Verbraucherzentrale nicht vor Vertragsschluss [Verbraucherzentrale: Jobscamming](https://www.verbraucherzentrale.de/jobscamming-was-tun-wenn-das-traumangebot-zur-falle-wird-110906), [Verbraucherzentrale Niedersachsen](https://www.verbraucherzentrale-niedersachsen.de/wissen/digitale-welt/datenschutz/jobscamming-so-erkennen-sie-gefaelschte-jobangebote-und-schuetzen-ihre-daten-120421).

Alles, was den Hard-Filter passiert, erhält einen gewichteten Gesamtscore aus fünf Dimensionen:

```yaml
scoring:
  hard_filter:
    - sprache_erforderlich          # aus Kandidatenprofil, Kapitel 8
    - max_pendelzeit_minuten        # aus Kandidatenprofil, Kapitel 8
    - vertragsart_ausgeschlossen    # aus Kandidatenprofil, Kapitel 8
    - arbeitserlaubnis_erforderlich
    - jobscamming_signal            # WhatsApp/Telegram-only, Video-Ident/Konto vor Vertragsschluss
  gewichte:                         # Default, siehe Offene Fragen
    passung: 0.35
    attraktivitaet: 0.20
    erfolgschance: 0.15
    arbeitgeberqualitaet: 0.15
    frische: 0.15
  schwelle_top_liste: 0.60          # darunter: nicht in Tagesauswahl, aber in Warteliste sichtbar
  schwelle_rueckfrage: [0.45, 0.60] # Band "unsicher" -> Nutzer fragen statt automatisch verwerfen
```

**Passung** wird über eine dreistufige Retrieval-Pipeline ermittelt, nicht durch einen einzelnen LLM-Aufruf pro Anzeige — das hält die Kosten bei mehreren Hundert Anzeigen/Tag beherrschbar:

1. **Hybrid-Retrieval**: BM25 (`bm25s` mit deutschem Stemmer) plus Dense-Embeddings mit **BGE-M3** — MIT-lizenziert, 100+ Sprachen, selbst gehostet über `sentence-transformers`, keine Tokenkosten [bm25s](https://github.com/xhluca/bm25s), [FlagEmbedding/BGE-M3](https://github.com/FlagOpen/FlagEmbedding), [sentence-transformers](https://github.com/UKPLab/sentence-transformers) — kombiniert per Reciprocal Rank Fusion auf die Top 30–50 Kandidaten des Tages. Skill-Begriffe aus Anzeige und Kandidatenprofil (Kapitel 8) werden dabei über die ESCO-Taxonomie normalisiert, die kostenlos in deutscher Sprache vorliegt [ESCO](https://esco.ec.europa.eu/en/use-esco/download).
2. **Reranking** der Top 30–50 auf die engere Top 20 mit **Cohere Rerank 3.5** (0,001 USD/Suche) [Cohere Rerank 3.5](https://openrouter.ai/cohere/rerank-v3.5); Voyage `rerank-2.5` ist eine Alternative mit Freikontingent, über die Bibliothek `rerankers` austauschbar implementiert, um keinen Anbieter fest zu verdrahten [rerankers](https://github.com/AnswerDotAI/rerankers).
3. **LLM-Judge** mit **Claude Sonnet 5** ($2/$10 pro 1 Mio. Token) bewertet die verbleibende Top 20 gegen eine feste Rubrik und liefert `passung_score`, `attraktivitaet_score`, `erfolgschance_score` sowie eine kurze Begründung in einem Aufruf. **Begründung der Modellwahl:** Erklärbarkeit und Rubrik-Treue sind hier wichtiger als bei der Massenextraktion, das Volumen (Top 20/Tag) ist aber klein genug, dass Sonnet 5 die Kosten niedrig hält; Fable 5.1 bleibt den schreibkritischen Schritten in Kapitel 11 vorbehalten, wo der Nutzer die höchste Qualität ausdrücklich wünscht. **Alternative:** Fable 5.1 auch hier einsetzen, wenn dir Erklärbarkeit wichtiger ist als die 30-Tage-Datenspeicherung, die Fable 5.1 erfordert (Kapitel 16).

**Erfolgschance** und **Frische** sind größtenteils regelbasiert (Posting-Alter, Zeit seit letzter Änderung, Reposting-Häufigkeit) und benötigen keinen LLM-Aufruf. **Arbeitgeberqualität** kombiniert, was ohne Vollautomatisierung verfügbar ist: Handelsregister-Status (Kapitel 10), grobe Größen-zu-offene-Stellen-Relation, und optional eine manuelle kununu-Stichprobe — eine automatisierte kununu-API existiert nicht, nur ToS-riskante Drittanbieter-Scraper [Apify kununu-Scraper](https://apify.com/lexis-solutions/kununu-scraper/api); eine Insolvenzprüfung läuft mangels offiziellem API entweder manuell auf dem Bundesportal oder über kostenpflichtiges Monitoring [Insolvenz-Radar](https://insolvenz-radar.de/funktionen/).

### 9.6 Erklärbarkeit: das „Warum“ zu jeder Bewertung

Jede bewertete Anzeige trägt ein strukturiertes Begründungsfeld, das der LLM-Judge zusätzlich zu den Scores liefert:

```json
{
  "passung_score": 78,
  "attraktivitaet_score": 65,
  "erfolgschance_score": 55,
  "begruendung_kurz": "Kernanforderungen A und B klar erfüllt; Sprachanforderung C nur teilweise durch die Story-Bank belegt.",
  "unsichere_felder": ["sprachanforderung_c"],
  "rueckfrage_notwendig": false
}
```

`begruendung_kurz` und `unsichere_felder` werden im Review-Cockpit (Kapitel 14) direkt neben der Anzeige angezeigt, nicht nur der Gesamtscore — der Nutzer soll nachvollziehen können, warum eine Stelle in der Top 10 steht, nicht nur, dass sie es tut.

### 9.7 Ghost-Job- und Personalvermittler-Heuristiken

Für Ghost Jobs (Anzeigen, die nie ernsthaft besetzt werden sollen) gibt es keine belastbare deutsche Quote — nur US-Zahlen (18–22 % aller Anzeigen laut einer Greenhouse-Analyse; eine HR-Selbstauskunft-Umfrage von LiveCareer nennt 93 % der Befragten, die zumindest gelegentlich Ghost Jobs schalten) [Ghost-Jobs-Analyse](https://www.fox5ny.com/news/ghost-jobs-greenhouse-analysis), [LiveCareer: Ghost Jobs](https://www.livecareer.de/bewerbung/ghost-jobs). **Entscheidung:** Ghost-Job-Verdacht fließt ausschließlich als **weicher Malus** in `erfolgschance` ein (Posting-Alter über einen Schwellenwert ohne Änderung, wiederkehrende Repostings, fehlende namentliche Ansprechperson), **nicht** als Hard-Filter. **Begründung:** Eine falsch als „Ghost Job“ eingestufte, tatsächlich echte Stelle würde eine reale Chance kosten — bei fehlender belastbarer Datengrundlage ist Vorsicht vor Übergeneralisierung wichtiger als Vollständigkeit der Filterung. Bei mittlerer Unsicherheit landet die Anzeige im Rückfrage-Band aus 9.5, statt automatisch verworfen zu werden.

Für Personalvermittler/Zeitarbeit gilt: Es gibt in Deutschland keine Kennzeichnungspflicht für die öffentliche Anzeige selbst — § 11 AÜG verpflichtet den Verleiher nur zur schriftlichen Information von Arbeitnehmer und Entleiher, nicht zu einem Hinweis im Stelleninserat [§ 11 AÜG](https://www.gesetze-im-internet.de/a_g/__11.html). Die Erkennung bleibt deshalb heuristisch: Formulierungen wie „für unseren Kunden“, „unser Mandant“, „im Auftrag von“ [we-hr.de: Personalvermittlung vs. Arbeitnehmerüberlassung](https://we-hr.de/unterschied-zwischen-personalvermittlung-und-arbeitnehmeruberlassung/), eine gepflegte Liste bekannter Personaldienstleister-Domains, sowie — wo verfügbar — das BA-API-Flag `zeitarbeit`. **Default-Annahme:** Anzeigen mit Personalvermittler-Verdacht werden nicht automatisch ausgeschlossen, sondern im Review-Cockpit mit dem Label „Personalvermittler (Verdacht)“ markiert; bei mittlerer Konfidenz fragt der Agent aktiv nach, statt zu klassifizieren. Ob Personalvermittler-Anzeigen grundsätzlich erwünscht sind, ist eine Präferenzentscheidung des Nutzers (siehe Offene Fragen).

### 9.8 Tagesauswahl: Top 10 mit Diversitätsregel

Aus allen Anzeigen oberhalb der Schwelle `schwelle_top_liste` wählt der Matcher die zehn höchstbewerteten aus — mit einer Kappung, damit ein einzelner Arbeitgeber oder Personalvermittler mit Multiposting nicht die ganze Liste dominiert:

```
sortiere Kandidaten nach Gesamtscore absteigend
top10 = []
zaehler_firma = {}
zaehler_personalvermittler = 0
für jeden Kandidaten in der sortierten Liste:
    wenn len(top10) == 10: stoppe
    wenn zaehler_firma[Kandidat.firma] >= 2: in Warteliste, weiter
    wenn Kandidat.ist_personalvermittler_verdacht und zaehler_personalvermittler >= 2:
        in Warteliste, weiter
    top10.anhängen(Kandidat)
    zaehler_firma[Kandidat.firma] += 1
    wenn Kandidat.ist_personalvermittler_verdacht: zaehler_personalvermittler += 1
```

Maximal zwei Anzeigen pro Arbeitgeber und maximal zwei Personalvermittler-Anzeigen sind Startwerte; übersprungene Kandidaten verschwinden nicht, sondern bleiben in der Warteliste sichtbar. Diese Top-10-Liste mit Status „ausgewählt“ geht an den Rechercheur (Kapitel 10) und wird im Review-Cockpit (Kapitel 14) präsentiert.

### 9.9 Feedback-Lernen aus Nutzerurteilen

**Entscheidung:** Jede Annahme/Ablehnung einer vorgeschlagenen Stelle im Review-Cockpit wird strukturiert geloggt (Entscheidung, betroffene Score-Dimensionen, optionaler Freitextgrund); die Scoring-Gewichte aus 9.5 werden nicht laufend, sondern in einem separaten, planbaren Schritt (z. B. wöchentlich) per einfacher logistischer Regression über die strukturierten Features nachjustiert — und jede Gewichtsänderung braucht vor der Übernahme eine Freigabe, kein stiller Drift. **Begründung:** Bei rund zehn Kuratierungen pro Tag und einem einzigen Nutzer liefert eine volle Contextual-Bandit-Infrastruktur (Thompson Sampling, UCB) keinen belegbaren Zusatznutzen gegenüber einem einfachen, nachvollziehbaren Verfahren. **Alternative:** Contextual Bandits nachrüsten, falls sich mit wachsendem Volumen zeigt, dass die einfache Gewichtsanpassung nicht mehr genügt — kein Bestandteil des MVP.

### 9.10 Modellwahl und Kosten pro 1.000 Anzeigen (grobe Schätzung)

| Schritt | Modell/Werkzeug | Kosten-Charakter |
|---|---|---|
| Extraktion normalisierter Felder | Claude Haiku 4.5 ($1/$5 pro 1 Mio. Token) | pro Anzeige, Massenschritt |
| ATS-Typ-Erkennung | regelbasiert (URL-/HTML-Muster) | $0 |
| Dedup (Blocking + MinHashLSH) | `datasketch`, lokal | $0, nur Rechenzeit |
| Hybrid-Retrieval (Embeddings) | BGE-M3, selbst gehostet | $0, Compute separat (Kapitel 18) |
| Reranking Top 30–50 → Top 20 | Cohere Rerank 3.5 (0,001 USD/Suche) | pro Anzeige im Retrieval-Fenster |
| LLM-Judge mit Begründung | Claude Sonnet 5 ($2/$10 pro 1 Mio. Token) | nur für die engste Auswahl (Top 20) |

Grobe Rechnung für die Extraktion (Haiku 4.5): bei angenommen rund 1.500 Input-Token (Anzeigetext, Schema, Prompt) und 300 Output-Token pro Anzeige ergeben sich für 1.000 Anzeigen etwa 1,5 Mio. Input- und 0,3 Mio. Output-Token, also ca. 1,5 USD + 1,5 USD ≈ **3 USD pro 1.000 Anzeigen** allein für die Extraktion. Der LLM-Judge läuft nur auf der engsten Auswahl (rund 20 von durchschnittlich 200–500 gescannten Anzeigen pro Tag), verursacht dadurch pro 1.000 gescannte Anzeigen nur einen kleinen zweistelligen Cent- bis niedrigen Dollar-Betrag; Reranking-Kosten liegen im Cent-Bereich. In Summe ist für 1.000 gescannte Anzeigen eine Größenordnung von grob **5–10 USD** plausibel — eine Schätzung zur Orientierung, keine Preiszusage; die genaue Kostenrechnung für den laufenden Monatsbetrieb steht in Kapitel 18. Zu beachten: Modelle ab Claude 4.7 (dazu zählen Opus 5, Sonnet 5, Fable 5.x) nutzen einen neuen Tokenizer, der für denselben Text tendenziell mehr Token erzeugt als ältere Modelle — bei der Kalibrierung in Kapitel 19 einen Aufschlag einplanen. Prompt Caching (Cache-Read zu 0,1x des Normalpreises, bei Fable-Modellen 0,025x) kann die Kosten für wiederholt genutzte Schema- und Rubrik-Texte weiter senken [Anthropic Pricing](https://platform.claude.com/docs/en/about-claude/pricing).

### Offene Fragen

- Sollen Personalvermittler-/Zeitarbeit-Anzeigen standardmäßig einbezogen, ausgeschlossen oder nur markiert werden? **Default-Annahme:** einbeziehen, aber mit Label und Rückfrage bei Unsicherheit (9.7).
- Sind die vorgeschlagenen Standardgewichte (Passung 35 %, Attraktivität 20 %, Erfolgschance 15 %, Arbeitgeberqualität 15 %, Frische 15 %) passend, oder sollen sie von Anfang an anders justiert werden? **Default-Annahme:** wie in 9.5 angegeben, Anpassung über das Feedback-Lernen (9.9).
- Ist die Nutzung kostenpflichtiger Rerank-/Embedding-APIs (Cohere/Voyage, geschätzt Cent-Bereich/Tag) akzeptabel, oder soll ausschließlich mit selbst gehosteten Modellen (BGE-M3) gearbeitet werden? **Default-Annahme:** BGE-M3 selbst gehostet für Embeddings, Cohere Rerank 3.5 für Reranking (Kosten vernachlässigbar).
- Wie viele „unsichere“ Kandidaten pro Tag (Rückfrage-Band aus 9.5) sind akzeptabel, bevor sie dem Nutzer vorgelegt statt automatisch verworfen werden? **Default-Annahme:** kein festes Limit, alle im Band werden angezeigt; Feinjustierung in der MVP-Phase (Kapitel 19).
- Soll automatisiertes kununu-Scraping über einen Drittanbieter (ToS-Grauzone) für die Arbeitgeberqualität genutzt werden, oder nur manuelle Stichproben? **Default-Annahme:** keine automatisierte Drittanbieter-Abfrage, nur manuelle/stichprobenartige Prüfung.

**Quellen dieses Kapitels:**
- bundesAPI/jobsuche-api (GitHub) — https://github.com/bundesAPI/jobsuche-api
- bundesAPI/entgeltatlas-api (GitHub) — https://github.com/bundesAPI/entgeltatlas-api
- bundesAPI/handelsregister (GitHub) — https://github.com/bundesAPI/handelsregister
- Greenhouse Job Board API — https://developers.greenhouse.io/job-board.html
- Lever Postings API (GitHub) — https://github.com/lever/postings-api
- Personio: Integrate jobs via XML — https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML
- Recruitee Careers Site API — https://docs.recruitee.com/reference/intro-to-careers-site-api
- SmartRecruiters Posting API — https://developers.smartrecruiters.com/docs/posting-api
- Workday Source Guide (Community) — https://github.com/Francis1998/agentic-career-search/blob/main/docs/guides/WORKDAY_SOURCE_GUIDE.md
- SerpAPI Google Jobs API — https://serpapi.com/google-jobs-api
- Adzuna Developer Portal — https://developer.adzuna.com/
- Arbeitnow Job Board API — https://arbeitnow.com/api/job-board-api
- Jooble API — https://jooble.org/api/about
- webappanalyzer (GitHub) — https://github.com/enthec/webappanalyzer
- Claude Structured Outputs — https://docs.claude.com/en/docs/build-with-claude/structured-outputs
- Anthropic Pricing — https://platform.claude.com/docs/en/about-claude/pricing
- FlagEmbedding / BGE-M3 (GitHub) — https://github.com/FlagOpen/FlagEmbedding
- sentence-transformers (GitHub) — https://github.com/UKPLab/sentence-transformers
- bm25s (GitHub) — https://github.com/xhluca/bm25s
- rerankers (AnswerDotAI, GitHub) — https://github.com/AnswerDotAI/rerankers
- Cohere Rerank 3.5 (OpenRouter) — https://openrouter.ai/cohere/rerank-v3.5
- ESCO Download/Portal — https://esco.ec.europa.eu/en/use-esco/download
- datasketch (GitHub) — https://github.com/ekzhu/datasketch
- recordlinkage (GitHub) — https://github.com/J535D165/recordlinkage
- dedupe (GitHub) — https://github.com/dedupeio/dedupe
- Textkernel-Forschung zu Duplikaten (ACM) — https://dl.acm.org/doi/fullHtml/10.1145/3486622.3493928
- Ghost-Jobs-Analyse (Fox5NY) — https://www.fox5ny.com/news/ghost-jobs-greenhouse-analysis
- LiveCareer: Ghost Jobs — https://www.livecareer.de/bewerbung/ghost-jobs
- Indeed Hiring Lab: Gehaltsangaben bleiben Ausnahme — https://www.hiringlab.org/de/blog/2025/03/05/gehaltsangaben-bleiben-in-deutschland-die-ausnahme/
- AFA Anwalt: Entgelttransparenzgesetz 2026 — https://www.afa-anwalt.de/news/entgelttransparenzgesetz-2026-rl-2023-970/
- we-hr.de: Personalvermittlung vs. Arbeitnehmerüberlassung — https://we-hr.de/unterschied-zwischen-personalvermittlung-und-arbeitnehmeruberlassung/
- § 11 AÜG (gesetze-im-internet.de) — https://www.gesetze-im-internet.de/a_g/__11.html
- Verbraucherzentrale: Jobscamming — https://www.verbraucherzentrale.de/jobscamming-was-tun-wenn-das-traumangebot-zur-falle-wird-110906
- Verbraucherzentrale Niedersachsen: Jobscamming erkennen — https://www.verbraucherzentrale-niedersachsen.de/wissen/digitale-welt/datenschutz/jobscamming-so-erkennen-sie-gefaelschte-jobangebote-und-schuetzen-ihre-daten-120421
- Apify: kununu-Scraper — https://apify.com/lexis-solutions/kununu-scraper/api
- Insolvenz-Radar — https://insolvenz-radar.de/funktionen/
- spaCy de_core_news_lg (GitHub Release) — https://github.com/explosion/spacy-models/releases/tag/de_core_news_lg-3.8.0
