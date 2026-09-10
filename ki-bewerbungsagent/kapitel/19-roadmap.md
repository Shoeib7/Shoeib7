## 19. Roadmap: Phase 0, MVP, v1, v2; Meilensteine, Definition of Done, erste Schritte

Die Roadmap übersetzt die Entscheidungen der Kapitel 6 bis 16 in eine Reihenfolge, in der Claude Code sie bauen und du sie prüfen kannst. Sie entscheidet nichts neu: Quellen und Stufen kommen aus Kapitel 6.6, Laufzeit, Datenmodell und Migrationspfad aus Kapitel 7.10, die Module aus den Kapiteln 8 bis 15, die Freigabe-Logik aus Kapitel 16.8, die Zielwerte aus Kapitel 1.3. Was hier hinzukommt, ist die Zeitachse, die Arbeitspakete mit Abhängigkeiten, die Abnahmekriterien je Phase, die Auslöser für jeden Migrationsschritt und die ersten konkreten Handgriffe.

### 19.1 Leitplanken

1. **Kleinster Durchstich zuerst, dann Breite.** Der MVP ist eine schmale, vollständige Kette von der Quelle bis zum Entwurf im Postfach, keine breite Sammlung halbfertiger Module. Jede Phase endet mit einer messbaren Definition of Done, die ausschließlich aus Tracker- und `event_log`-Daten berechnet wird (Kapitel 1.5); die Abfragen dafür stehen in 19.4.
2. **Das Freigabe-Gate steht ab Tag 1.** Auch der MVP hat keinen Codepfad, der ohne `application.status = 'freigegeben'` etwas nach außen schickt; im MVP ist der Bote ohnehin im Entwurfsmodus (Kapitel 7.8, 15.1). Der erste echte Versand ist ein v1-Meilenstein, keine Nebenwirkung.
3. **Grün-Quellen vor Gelb-Quellen.** SerpAPI, später Apify oder JSearch, kommen erst, wenn der MVP eine dokumentierte Abdeckungslücke gezeigt hat (Kapitel 6.6). Rot-Quellen nie (Kapitel 16.8).
4. **Kalibrieren statt raten.** Alle Schwellen in Dedup (Kapitel 9.4), Rubrik (Kapitel 11.9) und ATS-Prüfer (Kapitel 12.11) sind Startwerte; die Roadmap sieht nach den ersten 20 freigegebenen Bewerbungen einen festen Kalibrierungstermin vor.
5. **Ein Mensch, ein Werkzeug.** Annahme für alle Aufwandsangaben: Claude Code schreibt Code, Tests und Vorlagen; du gibst Arbeitspakete vor, prüfst Diffs, testest an echten Anzeigen und triffst die Entscheidungen aus Kapitel 22. „PT“ bezeichnet unten einen Personentag deiner Zeit (rund sechs konzentrierte Stunden), nicht die Laufzeit von Claude Code. Die Zahlen sind Planungsannahmen ohne Quelle; sie werden nach Woche 2 gegen die tatsächlich gebrauchte Zeit korrigiert.
6. **Kein Migrationsschritt ohne Messwert.** Embeddings, SerpAPI, Managed Agents, Postgres und Portal-Co-Pilot haben je einen benannten Auslöser (19.7). Solange der Auslöser nicht aus den Betriebsdaten belegt ist, bleibt der Baustein auf der MVP-Stufe. Jeder Phasenabschluss aktualisiert außerdem das Risikoregister (Kapitel 20.7) und streicht die beantworteten Fragen aus Kapitel 22.

### 19.2 Überblick und Meilensteine

| Phase | Zeitraum | Ziel | Ergebnis am Ende | Laufende Kosten (Größenordnung) |
|---|---|---|---|---|
| Phase 0 | Woche 0 | Voraussetzungen schaffen | Accounts, Server, zwei Repos, vollständiges Kandidatenprofil, geprüfte Quellenzugänge | Server ca. 20 €/Monat; API nur Testaufrufe |
| MVP | Wochen 1–4 | Durchstich: Quelle → Entwurf im Postfach | Tageslauf liefert täglich bis zu 10 Bewerbungen im Status „bereit zur Freigabe“; Freigabe im Cockpit; Bote legt Entwürfe ab; du sendest selbst | Server; API-Budget bis 10 USD je Tageslauf; Datenquellen 0 EUR |
| v1 | Monat 2–3 | Betrieb stabilisieren, Versand und Rückkanal | SMTP-Versand nach Freigabe, Tracker mit Antwortklassifikation und Nachfassen, SerpAPI, weitere ATS-Adapter, Embeddings, Cockpit als Web-App, Phoenix | + SerpAPI 25 USD/Monat, Firecrawl 0–16 USD/Monat |
| v2 | Monat 4–6 | Portale, Lernen, Mehrnutzer-Vorbereitung | Co-Pilot für Portalformulare, gelernte Scoring-Gewichte, Auth im Cockpit, Managed-Agents-Entscheidung, rechtliche Prüfung vor Öffnung | + optionale Gelb-Quellen nach Freigabe, Kapitel 6.6 |

Die belastbare Kostenrechnung je Bewerbung und je Monat steht in Kapitel 18; die Zahlen hier sind nur die Deckel, die die Architektur setzt (Kapitel 7.7.7, 6.6).

Fünf Meilensteine markieren die Übergänge. Jeder hat ein einziges Abnahmeereignis, das du selbst auslöst, und die Definition of Done der jeweiligen Phase als Bedingung:

| Meilenstein | Ende von | Abnahmeereignis | Bedingung |
|---|---|---|---|
| MS0 „Startklar“ | Woche 0 | Commit `profil/` im Daten-Repository, `pytest` grün auf dem Server | Definition of Done Phase 0 (19.3) |
| MS1 „Durchstich“ | Woche 4 | Du klickst im eigenen Mailprogramm auf Senden für einen Entwurf, den der Bote angelegt hat | Definition of Done MVP (19.4), fünf Betriebstage |
| MS2 „Erster Versand und Kalibrierung“ | Woche 8 | Erste SMTP-Sendung durch den Boten im Versandfenster; Kalibrierungstermin nach 20 Freigaben protokolliert | V-01 bis V-07 abgenommen |
| MS3 „v1 abgenommen“ | Woche 12 | Baseline für Rücklauf- und Interviewquote mit dir festgehalten (Kapitel 1.3) | Definition of Done v1 (19.5) |
| MS4 „v2 und Go/No-Go“ | Woche 26 | Go/No-Go-Entscheidung nach Kapitel 21.5 dokumentiert | Definition of Done v2 (19.6) |

```text
Woche   0    1    2    3    4    5    6    7    8    9   10   11   12  13 … 16  17 … 20  21 … 26
        │    │    │    │    │    │    │    │    │    │    │    │    │   │      │   │      │   │
Phase 0 ████
MVP          ████ ████ ████ ████
v1                               ████ ████ ████ ████ ████ ████ ████ ████
v2                                                                       ████████ ████████ ████████████
        ▲                   ▲                   ▲                   ▲                           ▲
       MS0                 MS1                 MS2                 MS3                         MS4
Betrieb (tägliche Reviews)   ├──────────────── ab Woche 4 durchgehend ─────────────────────────────►
```

### 19.3 Phase 0: Vorbereitung (Woche 0)

**Ziel:** Am Ende der Woche existiert alles, was Claude Code in Woche 1 braucht, um den Scout gegen echte Daten zu bauen, und alles, was der Autor in Woche 2 braucht, um in deiner Stimme zu schreiben. Phase 0 ist die Woche mit dem höchsten Anteil deiner eigenen Zeit, weil das Kandidatenprofil (Kapitel 8) nur du liefern kannst.

**Arbeitspakete (Checkliste):**

*Block A: Entscheidungen und Accounts (ca. 0,5 PT)*

- [ ] P0-01 Fragenkatalog Kapitel 22 durchgehen; mindestens die Blocker beantworten: primäres E-Mail-Konto, VPS oder Mac, Fable 5.1 mit 30-Tage-Speicherung oder Opus 5 (Offenlegung und Zustimmung nach Kapitel 16.2 schriftlich festhalten), Budget je Tageslauf, Zielrollen und Orte. Für alles Übrige gelten die Default-Annahmen der Kapitel.
- [ ] P0-02 Anthropic Console: Organisation anlegen, Commercial-API-Key erzeugen, Ausgabenlimit setzen (Kapitel 7.7.7 Stufe 3), Auftragsverarbeitungsvertrag akzeptieren (Kapitel 16.2). Wenn Fable 5.1 eingesetzt wird: 30-Tage-Datenspeicherung für Covered Models aktivieren, sonst antwortet die API mit Fehler 400 ([API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)). Kein Consumer-Abo-Token für den Tageslauf (Kapitel 16.8).
- [ ] P0-03 Hetzner-Konto, CPX22 in Falkenstein oder Nürnberg bestellen (Preis vor Bestellung im Konfigurator prüfen; Drittquellen nennen 19,49 bzw. 19,99 €/Monat nach der Preisanpassung vom 15.6.2026, [Hetzner](https://docs.hetzner.com/de/general/infrastructure-and-availability/price-adjustment/), [Northflank](https://northflank.com/blog/hetzner-cloud-server-price-increases)), Ubuntu 24.04, SSH-Schlüssel, Systemnutzer `agent`, kein offener Port außer SSH (Kapitel 7.7.3).
- [ ] P0-04 Telegram-Bot bei BotFather anlegen, Token notieren; deine Chat-ID ermitteln (Kapitel 14.2, [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot)).
- [ ] P0-05 E-Mail-Zugang für den Entwurfsmodus: bei iCloud Zwei-Faktor-Authentifizierung prüfen und App-spezifisches Passwort erzeugen (SMTP `smtp.mail.me.com:587`, IMAP `imap.mail.me.com:993`, [Apple](https://support.apple.com/en-us/102198)); bei Gmail Google-Cloud-Projekt im OAuth-Testing-Modus mit dir als einzigem Testnutzer, Scope `gmail.compose` oder `gmail.modify` (beide „Restricted“, im Testing-Modus ohne Verifizierung; Refresh-Token läuft nach 7 Tagen ab, [Gmail-Scopes](https://developers.google.com/workspace/gmail/api/auth/scopes), [OAuth-Testing](https://support.google.com/cloud/answer/15549945?hl=en)).
- [ ] P0-06 Adzuna-Entwicklerkonto (App-ID und App-Key kostenlos; Rate Limits bei der Registrierung notieren, sie sind nicht öffentlich dokumentiert, [Adzuna](https://developer.adzuna.com/)). SerpAPI-Konto nur anlegen, noch nicht einbinden: der Free-Tier mit 250 Suchen im Monat dient am Ende des MVP der Abdeckungsmessung (Kapitel 6.5, [SerpAPI Pricing](https://serpapi.com/pricing)).
- [ ] P0-07 Job-Alerts bei StepStone, Indeed, LinkedIn und XING mit deinen Zielrollen abonnieren, Zieladresse ist das Bewerbungspostfach (Kapitel 6.5).

*Block B: Repos, Secrets, Projektgerüst (ca. 0,5 PT, überwiegend Claude Code)*

- [ ] P0-08 Zwei private Git-Repositories anlegen: `bewerbungsagent` (Code) und `bewerbungen-data` (Profil, Bewerbungen, Datenbank; nie veröffentlichen), Struktur nach Kapitel 7.9.
- [ ] P0-09 age-Schlüsselpaar erzeugen, privaten Schlüssel nur auf dem Server (`/etc/bewerbungsagent/age.key`, 0400) und im Passwortmanager; `.sops.yaml` und `config/secrets.enc.yaml` mit API-Key, E-Mail-Passwort, Telegram-Token, Adzuna-Keys ([sops](https://github.com/getsops/sops)).
- [ ] P0-10 Claude-Code-Projekt initialisieren: `CLAUDE.md` mit Statusbegriffen, den zehn Sicherheitsregeln als Verbotsliste, Testpflicht (Kapitel 7.9); `.claude/settings.json` mit Deny-Regeln und `PreToolUse`-Hook (Kapitel 7.8); `pyproject.toml` mit gepinnten Abhängigkeiten (`claude-agent-sdk` 0.2.152, Python 3.10+, [PyPI](https://pypi.org/project/claude-agent-sdk/)); `schema.sql` aus Kapitel 7.3 als erste Migration; leere Modulordner; `pytest` mit einem ersten Test, der die Migration ausführt.
- [ ] P0-11 Systempakete auf dem Server: Python 3.12, Node (für Playwright MCP ab v2), WeasyPrint-Abhängigkeiten, Poppler (`pdftotext`, `pdffonts`), Java für `tika-server` (Apache Tika 4.0.0, [Tika](https://github.com/apache/tika)), LibreOffice headless nur für die DOCX-Prüfung (Kapitel 13.2), Tesseract mit deutschem Sprachpaket (Kapitel 13.8). Schriftdateien Carlito und Liberation Sans nach `assets/fonts/` (Kapitel 13.3).
- [ ] P0-12 systemd-Units aus Kapitel 7.5 anlegen, aber die Timer noch deaktiviert lassen.

*Block C: Kandidatenprofil (ca. 2,5–3 PT, deine Zeit)*

- [ ] P0-13 Material sammeln: aktueller Lebenslauf, alle alten Anschreiben, 5 bis 10 echte Textproben (berufliche E-Mails, Beiträge), Zeugnis-Scans, Zertifikate; alles in einen Eingangsordner (Kapitel 8.9 Schritt 1).
- [ ] P0-14 Skill `.claude/skills/profil-onboarding/SKILL.md` anlegen lassen, dann Onboarding-Interview Block A bis C (Kontakt, Werdegang, Erfolge) in einer Claude-Code-Sitzung (Kapitel 8.8). Ergebnis: `lebenslauf.yaml` mit `story_id` an jedem Bullet, `story_bank.yaml` mit mindestens 8, besser 10 Einträgen mit `beleg.status`.
- [ ] P0-15 Zweite Sitzung: Block D bis H (Stimme, Präferenzen, Ausschlüsse, Standardantworten, Betrieb). Stimmprofil-Extraktion aus den Textproben mit Satzlängen-Statistik und Tabus `NUTZER-001…` als Vorschlag, den du korrigierst (Kapitel 8.4, 8.9 Schritt 2); Muster wie im `doc-coauthoring`-Skill: Kontext sammeln, Entwurf, Nutzer kuratiert ([SKILL.md](https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md)).
- [ ] P0-16 Jede Profildatei lesen und korrigieren; Git-Commit im Daten-Repository ist die Freigabe; Skript `candidate_profile` mit Hashes befüllen (Kapitel 8.10). Ab hier schreibt kein Modell mehr in `profil/`.
- [ ] P0-17 Dokumente: Zeugnisse einmalig normalisieren, OCR mit deutschem Tesseract-Sprachpaket, komprimieren (Kapitel 13.8). Bei ausländischem Abschluss den Anabin-Status prüfen und gegebenenfalls die ZAB-Bewertung beantragen, weil sie Monate dauert ([Anabin](https://anabin.kmk.org/kurzanleitung/ich-moechte-feststellen-wie-mein-auslaendischer-hochschulabschluss-in-deutschland-bewertet-wird.html)); fremdsprachige Zeugnisse nur mit beglaubigter Übersetzung (Kapitel 8.7).
- [ ] P0-18 Watchlist: 20 bis 50 Wunscharbeitgeber mit Karriereseiten-URL in `config/watchlist.yaml` (Default aus Kapitel 6.9); für die ersten zehn den ATS-Typ von Hand zuordnen.

*Block D: Quellen-Smoke-Tests (ca. 0,5 PT, mit Claude Code)*

- [ ] P0-19 BA-Jobsuche-API einmal live mit deiner Zielrolle aufrufen (`/pc/v6/jobs`, Header `X-API-Key: jobboerse-jobsuche`) und die Antwort als Fixture speichern; der Faktenprüfer konnte die Erreichbarkeit nicht testen (Kapitel 6.3, [bundesAPI](https://github.com/bundesAPI/jobsuche-api)).
- [ ] P0-20 Je einen echten Arbeitgeber mit Personio-XML, Greenhouse- und Lever-Feed aufrufen und als Fixture ablegen; die Endpunkte sind dokumentiert, aber in der Prüfsitzung nicht live verifiziert (Kapitel 6.2.2, [Greenhouse](https://developers.greenhouse.io/job-board.html), [Lever](https://github.com/lever/postings-api), [Personio](https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML)). Teamtailor an einer realen Seite testen und den Widerspruch aus der Recherche auflösen (Kapitel 6.2.2).
- [ ] P0-21 Fingerprints für softgarden, rexx, d.vinci, SAP SuccessFactors und JOIN an je drei Beispielseiten sammeln und in `ats_detection_rules.yaml` eintragen (Kapitel 4.6, 6.4); webappanalyzer liefert für diese Anbieter keine Muster ([categories.json](https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json)). DGFP-Marktanteile direkt in der Studie nachsehen (Kapitel 4, offene Frage 1; vom Prüfer nicht verifizierbar).
- [ ] P0-22 Formalia prüfen, die die Recherche nicht klären konnte: htmx-Lizenz in der LICENSE-Datei (Kapitel 14.2), DIN-5008-Maße gegen die Normausgabe (Kapitel 13.4), Adzuna-Limits (P0-06).

Summe Phase 0: rund 4 bis 4,5 PT deiner Zeit, davon 2,5 bis 3 PT für das Kandidatenprofil.

**Definition of Done (Phase 0):**

- Alle Blocker aus P0-01 sind entschieden und in `config/` oder Kapitel 22 festgehalten.
- `pytest` läuft auf dem Server grün mit mindestens der Migration; `sops exec-env` liefert die Umgebungsvariablen; ein Testaufruf gegen die Anthropic API mit dem gewählten Top-Modell antwortet ohne Fehler 400.
- `bewerbungen-data/profil/` ist committet; `candidate_profile.version = 1`; Story-Bank mit mindestens 8 Einträgen; Stimmprofil mit mindestens 5 Textproben und errechnetem Satzlängen-Korridor.
- Fixtures für BA-API, Personio, Greenhouse, Lever liegen unter `tests/fixtures/` und sind anonymisiert.
- Telegram-Bot antwortet auf `/status`; der Entwürfe-Ordner des Postfachs ist per IMAP erreichbar (ein Test-Entwurf angelegt und wieder gelöscht).

**Testplan:** Reine Smoke-Tests: ein Aufruf je Quelle, ein Aufruf je Modell, ein IMAP-APPEND, eine Telegram-Nachricht. Kein Modul wird schon gebaut.

**Risiken dieser Phase:** Das Onboarding ist der einzige Schritt, den niemand für dich beschleunigen kann; unvollständige Story-Bank bedeutet dünne Belege in jeder Bewerbung (Kapitel 8.9 Schritt 6). Die inoffizielle BA-API kann anders antworten als die OpenAPI-Beschreibung (Kapitel 6.3); dann wird der Adapter in Woche 1 gegen die reale Antwort gebaut, nicht gegen die Spezifikation. Hetzner-Preise und Verfügbarkeit einzelner Linien waren nicht abschließend belegbar (Kapitel 7.7.3).

### 19.4 MVP (Wochen 1–4): der kleinste Durchstich, der echten Wert liefert

**Ziel:** Ein Tageslauf, der aus BA-API, Watchlist-Feeds, Adzuna, Arbeitnow und Job-Alert-Mails eine Tagesauswahl bildet, sie recherchiert, schreibt, prüft, setzt und dir im Cockpit vorlegt; nach deiner Freigabe legt der Bote die fertige E-Mail als Entwurf im Postfach ab, und du sendest sie selbst. Genau die Kette aus Kapitel 7.2, mit den Stufen-Festlegungen aus Kapitel 6.6 und 7.10.

**Im MVP enthalten:** Scout-Adapter BA-API, Personio, Greenhouse, Lever, Adzuna, Arbeitnow, Job-Alert-Mails (nur Metadaten, Volltext erst nach Klick); Extraktion Haiku 4.5, Dedup mit datasketch, Injection-Screen; Matcher mit Muss-Filter, BM25 und Judge auf Sonnet 5 im Batch, Tagesauswahl mit Diversitätskappung; Rechercheur, Autor, Kritiker als Agent-SDK-Subagents; Claims-Abgleich und Anti-Generik-Katalog; ATS-Prüfer Stufe 1 und 2 mit Tika und OpenResume; Setzer mit WeasyPrint und python-docx, Layout `sachlich`, Mappe; Cockpit als Telegram-Bot plus FastAPI/HTMX-Detailseite mit Checkliste, Wort-Diff, Seitenvorschau und den vier Aktionen; Bote im Entwurfsmodus; `event_log`, Budget je Lauf, systemd-Timer. Portal-Bewerbungen im MVP: Der Bote liefert Link, Standardantworten und Dateien, du füllst das Formular selbst aus (Kapitel 2.12, 15.5).

**Nicht im MVP:** SMTP-Versand, SerpAPI und alle Gelb-Quellen, Embeddings und Reranker, Antwortklassifikation und Nachfassen, Portalformular-Vorbefüllung, Kanban und Statistikseite, Phoenix, Zweitgutachter, Lernschleife (Gründe werden nur gesammelt), Motivationsschreiben, englische Vorlage nur, wenn deine Zielrollen sie brauchen (Kapitel 22).

**Arbeitspakete (Checkliste):**

- [ ] M-01 Orchestrator-Kern: CLI `bewerbungsagent`, `run` und `event_log` mit Triggern, Statusmaschine mit den erlaubten Übergängen aus Kapitel 7.4, `--budget-usd` mit hartem Abbruch, `--resume`, Commit ins Daten-Repository nach jedem Statuswechsel (Kapitel 7.5, 7.7.5).
- [ ] M-02 Scout-Adapter BA-API mit Schema-Validierung, Backoff, Tagesnotiz bei Einbruch der Trefferzahl (Kapitel 6.3); Adapter Personio, Greenhouse, Lever für die Watchlist; Adzuna, Arbeitnow; IMAP-Leser für Job-Alert-Mails (Kapitel 6.5). Jeder Adapter liefert den Rohtreffer aus Kapitel 6.7 und hat einen Fixture-Test.
- [ ] M-03 Normalisierung und Extraktion mit Haiku 4.5 und Structured Output nach dem Schema aus Kapitel 9.2; Eskalation auf Sonnet 5 bei leeren Pflichtfeldern; pydantic-Nachprüfung (Kennziffer, PLZ).
- [ ] M-04 Dedup dreistufig (ID, Blocking mit Jaro-Winkler 0,90, MinHashLSH 0,75–0,85 mit `datasketch`, [datasketch](https://github.com/ekzhu/datasketch)), Grenzband als „mögliches Duplikat“ (Kapitel 9.4).
- [ ] M-05 Injection-Screen und Scam-Signal mit Haiku 4.5 vor dem Judge (Kapitel 7.8 Regel 5, 9.5); Fixtures mit eingebetteten Anweisungen und Weißtext.
- [ ] M-06 Matcher: Muss-Filter aus `praeferenzen.yaml`, BM25-Vorauswahl mit `bm25s` ([bm25s](https://github.com/xhluca/bm25s)), Judge auf Sonnet 5 als Message Batch mit gecachtem Präfix (Kapitel 7.6, [Batch](https://platform.claude.com/docs/en/build-with-claude/batch-processing), [Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)); synchroner Fallback für die Top-40; Tagesauswahl Top-10 mit maximal zwei Anzeigen je Arbeitgeber (Kapitel 9.8); Begründungsfeld je Anzeige (Kapitel 9.6).
- [ ] M-07 Rechercheur als Subagent (Definition aus Kapitel 7.6): Stufenfolge Anzeige, Karriereseite, Impressum, Register nur im Zweifel (Kapitel 10.1); `allowed_domains`, `max_uses: 8`; Dossier nach `schemas/dossier.json`; Rückfragen als `question`-Datensätze mit den drei Fragetypen und Defaults aus Kapitel 10.4; `contact.delete_after` (Kapitel 10.7).
- [ ] M-08 Autor als Subagent: Format-Router (Kapitel 11.3), Briefing-Schritt auf Sonnet 5, drei Varianten in einem Aufruf, Lebenslauf-Tailoring mit Log (Kapitel 11.7), `claims` mit Story-IDs; Skills `anschreiben`, `lebenslauf-tailoring`; System-Prompt-Skelett aus Kapitel 11.10.
- [ ] M-09 Kritiker: deterministische Prüfungen und `anti_generik.yaml` (Kapitel 11.5), Rubrik K1–K7 mit Gates (Kapitel 11.9), Fakten-Check gegen Story-Bank, Stimm-Check und Leser-Test im frischen Kontext, Konsistenz-Check (Kapitel 11.8); höchstens zwei Schleifen, danach Cockpit. Modell für Kritiker und Zweitgutachter nach Kapitel 11.10 in `config/modelle.yaml` konfigurieren, nicht im Code festschreiben.
- [ ] M-10 ATS-Prüfer Stufe 1: Begriffsextraktion mit Haiku 4.5, Abdeckungsquote deterministisch, Stuffing-Regeln (Kapitel 12.2–12.4). Stufe 2: `tika-server` und OpenResume-Parser lokal, Formatcheckliste, Lesetest auf Sonnet 5, ATS-Report (Kapitel 12.5–12.8, [OpenResume](https://github.com/xitanggg/open-resume)).
- [ ] M-11 Setzer: Jinja2-Vorlagen `sachlich` für Anschreiben (DIN 5008 Form B) und Lebenslauf, WeasyPrint, python-docx-Builder für den Lebenslauf, Typografie-Nachbearbeitung, QA-Tabelle aus Kapitel 13.10, Mappe mit pypdf, Dateinamen nach Kapitel 13.6, `render/vN`, `final/` mit Hashes, Manifest (Kapitel 13.11, [WeasyPrint](https://weasyprint.com/)).
- [ ] M-12 Cockpit: Telegram-Bot mit Push je Stelle, Tagesdigest, vier Aktionen als Inline-Buttons, Rückfragen als Buttons oder Freitext (Kapitel 14.6–14.8); FastAPI/HTMX-Detailseite über SSH-Tunnel mit Checkliste aus Kapitel 14.4, Wort-Diff (`difflib`, Kapitel 14.5), Seitenvorschau, Undo-Fenster 60 Sekunden; Audit-Einträge in `event_log`; E-Mail-Digest als Rückfallkanal.
- [ ] M-13 Bote im Entwurfsmodus: E-Mail mit Betreff nach Konvention, Anschreiben als Mailtext aus `anschreiben.txt`, Mappe als Anhang, IMAP-APPEND in „Entwürfe“ oder Gmail-Drafts-API ([Gmail Drafts](https://developers.google.com/workspace/gmail/api/guides/drafts)); nur aus `final/`, nur bei Hash-Gleichheit und `review_item.status = freigegeben` (Kapitel 7.8, 15.1). Auth-Fehler 535 erzeugt eine Warnung statt Wiederholungen (Kapitel 15.4). Für Portal-Stellen: Nachricht mit Link, Dateipfaden und `standardantworten.yaml`-Auszug.
- [ ] M-14 Betrieb: Timer für Tageslauf Teil 1 und 2 aktivieren, Wartungslauf mit `sqlite3 .backup`, wöchentlicher Kostenabgleich über die Usage-and-Cost-API ([Cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)), Telegram-Zusammenfassung am Ende von Teil 2 (Kapitel 7.7.7).
- [ ] M-15 Abdeckungsmessung: in der letzten MVP-Woche täglich bis zu 8 SerpAPI-Suchen aus dem Free-Tier mit deinen Zielrollen; Treffer gegen die Grün-Quellen abgleichen und die Lücke dokumentieren (Kapitel 6.5). Das Ergebnis entscheidet, ob SerpAPI in v1 kommt.

**Abhängigkeiten und Parallelisierung.** Die Pakete sind über Schemas (`schemas/`) und Fixtures (`tests/fixtures/`) entkoppelt; sobald M-01 steht, können vier Stränge in getrennten Claude-Code-Sitzungen laufen:

```text
M-01 Orchestrator (Statusmaschine, run, event_log, Budget)
 ├─ Strang A  M-02 Adapter → M-03 Extraktion → M-04 Dedup → M-05 Screen → M-06 Matcher
 ├─ Strang B  M-07 Rechercheur → M-08 Autor → M-09 Kritiker      (Eingabe: Fixture-Anzeigen + Profil)
 ├─ Strang C  M-11 Setzer ⇄ M-10 ATS-Prüfer                       (Eingabe: Beispiel-Dossier aus Kapitel 13.3)
 └─ Strang D  M-12 Cockpit                                        (Eingabe: Datensätze aus schema.sql)
Zusammenführung: A+B+C+D → M-13 Bote → M-14 Betrieb → M-15 Messung
```

Strang C braucht keinen Modellaufruf und kann ganz am Anfang laufen; Strang B braucht das committete Profil aus Phase 0.

**Sprintplan (Annahme: eine Person, Claude Code baut, du prüfst):**

| Woche | Schwerpunkt | Arbeitspakete | Zwischenergebnis | Deine Zeit |
|---|---|---|---|---|
| 1 | Daten rein | M-01 bis M-06 | `bewerbungsagent tageslauf --teil 1` läuft gegen echte Quellen; morgens steht eine Top-10 mit Begründung in der Datenbank und als Telegram-Text | ca. 3 PT (Adapter gegen echte Antworten prüfen, Judge-Begründungen an 30 Anzeigen lesen, Muss-Filter nachjustieren) |
| 2 | Text raus | M-07 bis M-09 | Für drei echte Anzeigen liegen Dossier, Anschreiben-Varianten, Tailoring-Log, Kritikbericht und Fakten-Check als Dateien vor | ca. 3 PT (Dossiers auf Fehler prüfen, Anschreiben gegen dein Stimmprofil lesen, Rückfrage-Fälle durchspielen) |
| 3 | Datei raus | M-10, M-11 | Erste Mappe als PDF, Setzer-QA und ATS-Report grün, Reproduzierbarkeits-Hash stabil | ca. 3 PT (Vorlagenabnahme am Bildschirm und im Druck, DOCX in Word öffnen, Test-Parsing-Berichte lesen) |
| 4 | Mensch dazu | M-12 bis M-15 | Vollständiger Tageslauf bis „bereit zur Freigabe“, Freigabe per Telegram, Entwurf im Postfach, fünf Betriebstage, Abdeckungsmessung | ca. 4 PT (tägliche Reviews, Freigaben, Fehlerprotokoll, Messung) |

Summe MVP: rund 13 PT deiner Zeit über vier Wochen, plus ab Woche 4 täglich 30 bis 60 Minuten Review (Kapitel 2.13). Claude-Code-Sitzungen: je Arbeitspaket ein bis drei Sitzungen zu zwei bis vier Stunden, insgesamt etwa 25 bis 35 Sitzungen.

**Definition of Done (MVP), gemessen aus Tracker- und `event_log`-Daten (Kapitel 1.5):**

1. Zwei aufeinanderfolgende Tagesläufe liefern ohne manuellen Eingriff eine Tagesauswahl; jede ausgewählte Stelle erreicht „bereit zur Freigabe“ oder trägt eine begründete Rückfrage beziehungsweise einen Cockpit-Eintrag „Kritiker unzufrieden“.
2. Median der Review-Zeit („bereit zur Freigabe“ bis „freigegeben“) unter 10 Minuten (Kapitel 1.3).
3. Null unbelegte Aussagen in freigegebenen Dokumenten (Kritiker-Fakten-Check plus Stichprobe von dir an mindestens fünf Bewerbungen).
4. ATS-Parsing-Gates in 100 Prozent der freigegebenen Dokumente bestanden.
5. Null Vorgänge mit Außenwirkung ohne Freigabe: Der Bote hat ausschließlich Entwürfe angelegt, jeder mit `review_item.status = freigegeben` und passendem Hash.
6. Kosten je Tageslauf innerhalb des Budgets (Default 10 USD, Kapitel 7.11) und nach dem ersten Wochenabgleich mit der Usage-Seite ohne Abweichung über 20 Prozent gegenüber dem `event_log` (Schwelle ist eine Annahme).
7. Abdeckungsmessung dokumentiert (M-15); Ablehnungsgründe aus dem Cockpit liegen strukturiert vor.

Die Punkte 2, 4, 5 und 6 berechnet ein Skript `scripts/abnahme.py` aus dem Schema in Kapitel 7.3; die Abfragen gehören zum Arbeitspaket M-14, damit die Abnahme nicht von Hand zusammengesucht wird:

```sql
-- DoD 2: Median der Review-Zeit in Minuten (bereit zur Freigabe → freigegeben)
WITH z AS (
  SELECT e.entity_id,
         (julianday(MAX(CASE WHEN e.to_status = 'freigegeben' THEN e.ts END))
        - julianday(MAX(CASE WHEN e.to_status = 'bereit zur Freigabe' THEN e.ts END))) * 1440 AS minuten
  FROM event_log e
  WHERE e.entity_type = 'application' AND e.action = 'status_change'
  GROUP BY e.entity_id
  HAVING minuten IS NOT NULL
)
SELECT minuten FROM z ORDER BY minuten LIMIT 1 OFFSET (SELECT COUNT(*) FROM z) / 2;

-- DoD 5: Bote-Aktionen ohne freigegebenes review_item (muss 0 sein)
SELECT COUNT(*)
FROM event_log e
LEFT JOIN review_item r ON r.application_id = e.entity_id AND r.status = 'freigegeben'
WHERE e.actor = 'bote' AND e.action IN ('send', 'draft') AND r.id IS NULL;

-- DoD 4: finale Dokumente ohne bestandenes ATS-Gate (muss 0 sein)
SELECT COUNT(*) FROM document
WHERE is_final = 1 AND json_extract(ats_check, '$.gate_status') <> 'bestanden';

-- DoD 6: Kosten und Status der letzten zehn Tagesläufe
SELECT id, started_at, status, spent_usd, budget_usd
FROM run WHERE kind = 'tageslauf' ORDER BY started_at DESC LIMIT 10;
```

**Testplan (MVP):**

| Ebene | Was | Werkzeug |
|---|---|---|
| Unit | jeder Adapter gegen Fixture; Normalisierung; Dedup-Schwellen an konstruierten Paaren; Statusmaschine weist unerlaubte Übergänge ab | pytest, Fixtures aus Phase 0 |
| Golden | jeder Schema-Aufruf (Extraktion, Judge, Dossier, Kritik, Antwort) mit gespeicherter Antwort; Schemaänderung bricht den Test bewusst | pytest, Structured-Output-Schemas ([Agent SDK](https://code.claude.com/docs/en/agent-sdk/structured-outputs)) |
| Sicherheit | Anzeige mit eingebetteter Anweisung landet mit `signals.injection` außerhalb der Tagesauswahl; Rechercheur meldet Auffälligkeit statt zu handeln; `Bash(curl:*)` und `Write(./profil/**)` werden vom Hook abgewiesen ([Hooks](https://code.claude.com/docs/en/hooks)); Bote verweigert Versand bei fehlender Freigabe, altem `decided_at` oder Hash-Abweichung | Fixtures, Hook-Test, negative Bote-Tests |
| Setzer | gleiche Daten, gleiche Vorlagenversion, gleicher Hash; Platzhalter-Leck; Umlaute und ß in `pdftotext`; alle Schriften eingebettet (`pdffonts`) | Kapitel 13.10 |
| Ende zu Ende | Trockenlauf mit 20 anonymisierten Anzeigen und einem Testpostfach; danach der erste echte Lauf mit Budget 5 USD | `tageslauf --teil 1/2`, Testkonto |
| Betrieb | `--resume` nach künstlichem Abbruch; Budgetabbruch mit Status `budget_erreicht`; Batch nicht fertig, synchroner Fallback greift | Orchestrator-Tests |

**Risiken (MVP):** Die BA-API bricht oder ändert das Schema (Kapitel 6.3); dann laufen Watchlist und Adzuna weiter, und die Tagesnotiz meldet den Einbruch. Der Kritiker lehnt zu oft ab und der Tageslauf liefert wenig: Ursache ist meist ein zu dünnes Stimmprofil oder zu strenge Startschwellen; Gegenmaßnahme ist die Kalibrierung nach 20 Bewerbungen (Kapitel 11.9), nicht das Lockern der Gates. Kosten laufen über das Budget, weil der ab Claude 4.7 verwendete Tokenizer rund 30 Prozent mehr Tokens erzeugt ([Pricing](https://platform.claude.com/docs/en/about-claude/pricing)); das Lauf-Budget bricht hart ab, und Kapitel 18 rechnet mit dem Aufschlag. Das App-spezifische Passwort wird bei einer Apple-ID-Passwortänderung ungültig; der Bote erkennt 535 und fordert dich auf (Kapitel 15.4). Die Wochenaufwände sind Annahmen; wenn Woche 1 mehr als 4 PT kostet, wird Woche 4 auf zwei Wochen gestreckt statt Pakete zu streichen.

### 19.5 v1 (Monat 2–3): Betrieb, Versand, Rückkanal, Breite

**Ziel:** Aus dem Durchstich wird ein System, das du im Alltag laufen lässt: Der Bote sendet nach Freigabe selbst, der Tracker liest Antworten, klassifiziert sie und schlägt Nachfassen vor, die Quellen decken den Markt breiter ab, das Cockpit zeigt die ganze Pipeline, und die Kosten sind nachvollziehbar. v1 beginnt mit vier Wochen Betrieb des MVP, in denen Fehler, Ablehnungsgründe und Kosten gesammelt werden; die Erweiterungen kommen parallel dazu, aber nur die, deren Voraussetzung aus dem Betrieb belegt ist (19.7).

**Arbeitspakete (Checkliste):**

*Betrieb und Kalibrierung (Wochen 5–8)*

- [ ] V-01 Zwanzig Werktage Tagesläufe mit Fehlerprotokoll; jeder Abbruch wird als Fixture in `tests/` übernommen. Ausstiegskriterium des MVP-Quellenmix aus Kapitel 6.6.
- [ ] V-02 Kalibrierungstermin nach 20 freigegebenen Bewerbungen: Rubrik-Schwellen und effort-Stufen (Kapitel 11.9, 11.10 Punkt 9), Dedup-Schwellen an echten Fehlklassifikationen (Kapitel 9.4), ATS-Prüfer-Schwellen (Kapitel 12.11), Satzlängen-Korridor gegen deine Freigaben (Kapitel 11.5). Jede Änderung ist ein Commit mit Begründung.
- [ ] V-03 Kostenabgleich: `event_log` gegen Usage-and-Cost-API; Cache-Trefferquote je Modell messen; Präfixe kürzen, wenn die Quote unter der Erwartung liegt (Kapitel 7.6).

*Versand und Rückkanal*

- [ ] V-04 Bote SMTP-Versand mit `aiosmtplib`: Versandlauf Di–Do 07:00–09:30 mit Jitter 15–40 Minuten, Tageslimit 10, Undo-Fenster, Sendeprotokoll mit Domain und Betreff-Hash statt Klartext (Kapitel 15.3); `canUseTool`-Gate öffnet nur für konkret freigegebene Datensätze (Kapitel 15.1, [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions)). Kopie in „Gesendet“ per IMAP-APPEND; eigene `Message-ID` für die Zuordnung von Antworten.
- [ ] V-05 Tracker-Daemon mit IMAP IDLE über `imap_tools` ([imap_tools](https://github.com/ikvk/imap_tools)); Zuordnung über `In-Reply-To`/`References`; Klassifikation nach Kapitel 15.6 (Haiku 4.5, Eskalation auf Sonnet 5 unter Konfidenz 0,7) mit Schema `{kategorie, konfidenz, aktion, termin}`; ICS-Anhänge mit `icalendar` parsen ([icalendar](https://pypi.org/project/icalendar)); Status-Übergänge nur mit deiner Bestätigung bei niedriger Konfidenz.
- [ ] V-06 Nachlauf: Nachfass-Fälligkeit nach Default 10 Werktagen (Kapitel 15.7), Nachfass-Entwurf durch Autor und Kritiker, gleiche Freigabe wie eine Erstbewerbung; Archivierung nach Frist ohne Antwort (Kapitel 7.4); `.ics`-Export für Interviewtermine, native Kalenderanbindung erst danach (Kapitel 15.6).
- [ ] V-07 Reauth-Erinnerungen: wöchentlicher Hinweis bei Gmail-Testing-Modus, 535-Erkennung bei iCloud (Kapitel 15.4).

*Quellen und Matching*

- [ ] V-08 SerpAPI-Adapter mit Volumendeckel (30 Suchen je Tageslauf, 1.000 je Monat für 25 USD), nur wenn M-15 eine Lücke gezeigt hat und du die Gelb-Quelle in `config/quellen.yaml` freigibst (Kapitel 6.5, 6.6).
- [ ] V-09 Adapter Recruitee, SmartRecruiters, Workday (POST-Endpunkt, Seitengröße 20, als inoffiziell gekennzeichnet), Teamtailor nach dem Test aus P0-20; generischer JSON-LD-Adapter für Karriereseiten mit `JobPosting`; Jina Reader beziehungsweise Firecrawl ohne Stealth als Markdown-Vorstufe mit Haiku-4.5-Extraktion (Kapitel 6.4, 6.6, [Jina](https://jina.ai/reader/), [Firecrawl-Preise](https://scrapegraphai.com/blog/firecrawl-pricing)); Einzelabruf des Volltexts nach Klick im Cockpit.
- [ ] V-10 ATS-Detektor vervollständigen: webappanalyzer-Fingerprints, eigene Muster aus P0-21, LLM-Klassifikation als Fallback mit Konfidenz höchstens 0,6, „unbekannt“ als gültiges Ergebnis (Kapitel 4.6, 6.4).
- [ ] V-11 Semantisches Matching: LanceDB neben SQLite ([LanceDB](https://github.com/lancedb/lancedb)), Embeddings mit BGE-M3 lokal oder per API (Kapitel 7.7.4, 9.5; ein lokales BGE-M3 braucht mehr Arbeitsspeicher als der CPX22, siehe Risiken), Reciprocal Rank Fusion mit BM25, Reranker über `rerankers` austauschbar (Cohere Rerank 3.5, [OpenRouter](https://openrouter.ai/cohere/rerank-v3.5)), ESCO-Normalisierung der Skill-Begriffe ([ESCO](https://esco.ec.europa.eu/en/use-esco/download)). Erst einführen, wenn V-01 zeigt, dass BM25 relevante Anzeigen verfehlt.

*Cockpit, Qualität, Betrieb*

- [ ] V-12 Cockpit als vollwertige lokale Web-App: Kanban über alle Status, Statistikseite mit Review-Zeit, Rückmeldequote, Ablehnungsgründen (Kapitel 14.10); Feedback-Tags werden zu `profilvorschlag`-Einträgen, die du bestätigst (Kapitel 7.8 Regel 6, 7.10); Formular für einzelne Profilfelder statt erneutem Interview (Kapitel 8.9 Alternative).
- [ ] V-13 Zweitgutachter auf Opus 5 vor der Freigabe als Option (Kapitel 7.6, 11.10); Entscheidung nach Kosten-Nutzen aus V-02.
- [ ] V-14 Arize Phoenix mit SQLite-Backend und Anthropic-Instrumentierung ([Phoenix](https://github.com/Arize-ai/phoenix)); Langfuse bleibt wegen des Ressourcenbedarfs außen vor (Kapitel 7.7.7).
- [ ] V-15 Backup verschlüsselt an einen zweiten Ort, etwa Hetzner Object Storage (4,99 €/Monat inklusive 1 TB, [Hetzner](https://www.hetzner.com/storage/object-storage/)); Wiederherstellung einmal geübt.
- [ ] V-16 Optional: Typst als Zweitrenderer für den Lebenslauf evaluieren (Kapitel 13.2); Layouts `klassisch` und `international-en` nur nach bestandenem Test-Parsing (Kapitel 13.3), englischer Anti-Generik-Katalog E01–E36 (Kapitel 11.11). Optional: eigene Bewerbungsdomain über mailbox.org (laut Anbieter ab rund 3 €/Monat) oder Fastmail (rund 6 $/Monat) mit Warm-up, wenn Zustellbarkeit oder Seriosität es verlangen (Kapitel 15.2, [mailbox.org](https://mailbox.org/en/news/new-price-plans-available-mailboxorg/), [Fastmail](https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates); beide Preise vom Faktenprüfer nicht geprüft).
- [ ] V-17 Optional: Rechercheur als Managed-Agents-Scheduled-Deployment mit Budget je Session testen (Cron minutengenau, Jitter, `budget_reached` pausiert die Session; [Scheduled Deployments](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments), [Budgets](https://platform.claude.com/docs/en/managed-agents/budgets)); nur der Rechercheur, nie der Bote (Kapitel 7.10).

**Sprintplan v1 (Wochen 5–12):**

| Wochen | Schwerpunkt | Arbeitspakete | Deine Zeit |
|---|---|---|---|
| 5–6 | Betrieb, erste Kalibrierung, SMTP-Versand | V-01, V-03, V-04, V-07 | ca. 1,5 PT je Woche (tägliche Reviews, erste echte Versendungen) |
| 7–8 | Tracker und Nachfassen; Kalibrierungstermin | V-02, V-05, V-06 | ca. 2 PT je Woche (Klassifikationen prüfen, Nachfass-Entwürfe lesen, Schwellen entscheiden) |
| 9–10 | Quellenbreite | V-08, V-09, V-10 | ca. 1,5 PT je Woche (Abdeckung vergleichen, neue Adapter an echten Firmen prüfen) |
| 11–12 | Cockpit, Matching, Observability | V-11, V-12, V-13, V-14, V-15, optional V-16/V-17 | ca. 1,5 PT je Woche |

Summe v1: rund 13 PT über acht Wochen plus die tägliche Review-Routine. Ab Woche 5 ist die Review-Routine der größte Zeitposten, nicht der Bau.

**Definition of Done (v1):**

1. Mindestens vier Wochen Tracker-Daten mit lückenlosen Statuswechseln; Baseline für Rücklauf- und Interviewquote berechnet und mit dir als Zielwert festgehalten (Kapitel 1.3).
2. Nachfass-Vorschlagsrate 100 Prozent für Bewerbungen ohne Rückmeldung nach der Frist; kein Nachfassen ohne Freigabe.
3. Jede gesendete E-Mail steht im Sendeprotokoll mit Zeitpunkt im Versandfenster; null Sendungen außerhalb des Gates; Undo-Fenster nachweislich wirksam (mindestens ein bewusst zurückgenommener Fall im Test).
4. Antwortklassifikation: Stichprobe von 30 Antworten mit höchstens zwei Fehlklassifikationen, jede davon mit Konfidenz unter 0,7 eskaliert (Zielwert ist eine Annahme, in V-02 zu bestätigen).
5. Watchlist-Abdeckung: mindestens 90 Prozent der Firmen mit funktionierendem Adapter (Kapitel 6.6).
6. Median der Review-Zeit unter 7 Minuten (Kapitel 1.3); ATS-Parsing weiterhin 100 Prozent.
7. Kosten je Bewerbung liegen im Korridor aus Kapitel 18 oder die Abweichung ist erklärt und im Budget berücksichtigt.

**Testplan (v1):** Zusätzlich zu den MVP-Tests: Versandtests gegen ein eigenes Zweitpostfach mit Header-Prüfung (`Message-ID`, `References`, kein Tracking, Anhang unter 5 MB); Klassifikations-Golden-Set aus 30 echten, anonymisierten Antworten (Absage, Einladung mit und ohne Termin, Rückfrage, Autoresponder); Nachfass-Fälligkeit mit manipulierten Zeitstempeln; Wiederanlauf des IDLE-Daemons nach Verbindungsabbruch; Adapter-Tests für jede neue Quelle mit Live-Fixture; Vergleichstest BM25 gegen Hybrid-Retrieval an 100 bewerteten Anzeigen, bevor V-11 produktiv wird; Wiederherstellung aus dem Offsite-Backup.

**Risiken (v1):** Der erste echte Versand ist der Moment mit der größten Außenwirkung; deshalb starten Woche 5 und 6 mit Budget 5 USD und Tageslimit 3, bevor die Defaults greifen. Ein lokales BGE-M3 passt nicht in 4 GB RAM (Kapitel 7.7.3); die Entscheidung ist API-Embedding, ein größerer Server oder Verzicht, und sie fällt erst nach V-01. SerpAPI parst Googles Ergebnisseiten; ein SERP-Umbau bricht den Adapter, weshalb er als Gelb-Quelle mit Deckel läuft (Kapitel 6.5). Der Refresh-Token im Gmail-Testing-Modus läuft alle 7 Tage ab; ohne V-07 bleibt der Tracker still stehen.

### 19.6 v2 (Monat 4–6): Portale, Lernen, Mehrnutzer-Vorbereitung

**Ziel:** Die drei Dinge, die in MVP und v1 bewusst ausgeklammert blieben, weil sie Außenwirkung, Modellrisiko oder Architekturänderungen mitbringen: Formulare auf Portalen, Lernen aus deinem Feedback, und die Vorbereitung auf mehr als einen Nutzer, ohne den Einzelbetrieb zu gefährden. Am Ende steht die Go/No-Go-Entscheidung aus Kapitel 21.5.

**Arbeitspakete (Checkliste):**

*Portal-Co-Pilot*

- [ ] W-01 Playwright-Skript mit persistentem Chrome-Profil (`launchPersistentContext`, sichtbar, auf deinem Rechner mit residentieller IP), Vorbefüllung aus `standardantworten.yaml`, Datei-Upload über `page.setInputFiles()`, Screenshot der Bestätigungsseite, Referenz-ID ins Sendeprotokoll (Kapitel 15.5); Testbefüllung ohne Absenden als Nachweis in der Checkliste (Kapitel 14.4). Der Klick auf „Absenden“ und die DSGVO-Einwilligung bleiben bei dir.
- [ ] W-02 Reihenfolge nach Risiko: zuerst Personio, softgarden und JOIN (vollständige Vorbefüllung), dann SAP SuccessFactors und Workday im Co-Pilot-Modus mit Zugangsdaten-Tresor je Arbeitgeber (Kontoanlage manuell, Kapitel 15 offene Frage), Plattform-Schnellbewerbungen nur als Co-Pilot mit geringer Frequenz oder gar nicht (Kapitel 15.5, 16.8).
- [ ] W-03 Playwright MCP mit `--allowed-hosts` als Werkzeug für Erkundung und Einzelseiten; `browser_file_upload` ist dokumentiert ([Playwright MCP README](https://raw.githubusercontent.com/microsoft/playwright-mcp/main/README.md)); Deny-Regel `mcp__playwright__*` bleibt für alle Subagents mit Drittinhalten (Kapitel 7.8).
- [ ] W-04 Unbekannte Karriereseiten: Fallback markieren und manuell ausfüllen; browser-use oder Skyvern nur nach Einzelprüfung von Nutzen und Lizenz (Kapitel 7.7.9; Skyvern AGPL-3.0, [Skyvern](https://github.com/Skyvern-AI/skyvern)).

*Lernen aus Feedback*

- [ ] W-05 Wöchentlicher Lernschritt: logistische Regression über die strukturierten Ablehnungs- und Freigabesignale, Vorschlag neuer Scoring-Gewichte, Übernahme nur nach deiner Freigabe (Kapitel 9.9); Vergleich alt gegen neu an den letzten 100 Bewertungen, bevor die Gewichte gelten.
- [ ] W-06 Rubrik- und effort-Sweep mit deinen Freigabedaten (Kapitel 11.10 Punkt 9); Stimmprofil-Überarbeitung, wenn der Kritiker wiederholt dieselbe Korrektur meldet (Kapitel 8.10); Anti-Generik-Katalog um deine Cockpit-Rückmeldungen „nie wieder“ erweitern (Kapitel 11.5).
- [ ] W-07 Zusätzliche Dokumenttypen: Motivationsschreiben auf Anforderung der Anzeige, Kurzbewerbung für Initiativbewerbungen (Kapitel 11.3, 13.5); optional GPTZero als nicht blockierender Hinweis, nur auf deinen Wunsch (Kapitel 11.2).
- [ ] W-08 Quellen: Playwright MCP für einzelne JS-lastige Karriereseiten, JOIN-Adapter, Crawl4AI statt Firecrawl bei Kosten, Apify oder JSearch nur nach ausdrücklicher Freigabe mit Volumendeckel (Kapitel 6.6).

*Mehrnutzer-Vorbereitung*

- [ ] W-09 Cockpit mit Anmeldung und Mandantenfeld; `candidate_profile.id` als Fremdschlüssel in allen Tabellen konsequent nutzen; Mandantentrennung im Daten-Repository (ein Repository je Nutzer).
- [ ] W-10 Managed-Agents-Entscheidung anhand des Beta-Status: Vollmigration oder Verbleib beim Agent SDK; bei Migration Vaults für E-Mail-Zugangsdaten, Memory Store `read_only` in Läufen mit Drittinhalten, Webhooks für „Session wartet auf Eingabe“ (Kapitel 7.10, [Memory](https://platform.claude.com/docs/en/managed-agents/memory), [Webhooks](https://platform.claude.com/docs/en/managed-agents/webhooks)); Beta-Header `agent-memory-2026-07-22` und `managed-agents-2026-04-01` nicht kombinieren.
- [ ] W-11 Postgres mit pgvector nur, wenn mehrere Nutzer real anstehen (Kapitel 7.7.4); Files-API-Objekte sind workspace-weit sichtbar, also ein Workspace je Mandant ([Files API](https://platform.claude.com/docs/en/build-with-claude/files)).
- [ ] W-12 Rechtliche Prüfung vor jeder Öffnung: § 7 UWG bei Bewerbungsmails, XING-AGB im Volltext, aktuelle Durchsetzung der Anthropic-Consumer-Terms, Lizenz der source-available Anthropic-Skills und der AGPL-Komponenten (OpenResume, gegebenenfalls Skyvern) bei kommerzieller Nutzung, volle DSGVO-Verantwortlichkeit ohne Haushaltsausnahme (Kapitel 16.10, 21.4).
- [ ] W-13 Go/No-Go nach Kapitel 21.5 mit den Zahlen aus dem Eigenbetrieb.

**Sprintplan v2 (Wochen 13–26):**

| Wochen | Schwerpunkt | Arbeitspakete | Deine Zeit |
|---|---|---|---|
| 13–16 | Portal-Co-Pilot | W-01 bis W-04 | ca. 1,5 PT je Woche (jede Portalfamilie an zwei echten Bewerbungen begleiten) |
| 17–20 | Lernen und Dokumenttypen | W-05 bis W-08 | ca. 1 PT je Woche (Gewichtsvorschläge prüfen, Sweep-Ergebnisse lesen) |
| 21–26 | Mehrnutzer-Vorbereitung, Entscheidungen | W-09 bis W-13 | ca. 1 PT je Woche; W-12 zusätzlich externe Beratungszeit |

Summe v2: rund 16 PT über 14 Wochen plus Review-Routine; Zeitbedarf für die anwaltliche Prüfung nicht enthalten.

**Definition of Done (v2):**

1. Mindestens zehn Portalbewerbungen im Co-Pilot-Modus mit Screenshot-Nachweis und Referenz-ID; null automatische Klicks auf „Absenden“; null Automatisierung auf LinkedIn, Indeed, StepStone, XING (Kapitel 16.8).
2. Zwei Lernzyklen mit dokumentierter Gewichtsänderung und Vorher-Nachher-Vergleich; jede Änderung mit Freigabe.
3. Cockpit-Anmeldung aktiv; ein zweiter Testnutzer mit eigenem Profil und eigenem Daten-Repository durchläuft den Tageslauf ohne Datenvermischung (Testprofil, keine reale Person).
4. Entscheidung Managed Agents dokumentiert; falls migriert, laufen Rechercheur-Sessions mit Budget und Vault ohne Klartext-Secret im Kontext.
5. Rechtliche Offen-Punkte aus Kapitel 16.10 sind beantwortet oder als Blocker für Kapitel 21 markiert.

**Testplan (v2):** Trockenlauf des Co-Piloten auf einem Test-Formular (eigene Personio-Demo oder lokale HTML-Kopie) mit Upload und Screenshot; negative Tests: Skript darf keinen Submit-Button auslösen; Mandantentest mit zwei Profilen und Prüfung, dass keine Abfrage ohne `candidate_profile.id`-Filter existiert; A/B-Vergleich der Scoring-Gewichte auf historischen Daten; bei Managed-Agents-Test ein Lauf mit absichtlich kleinem Budget, der in `budget_reached` pausiert.

**Risiken (v2):** Portale ändern Formulare ohne Ankündigung; jeder Co-Pilot-Adapter ist Wartung. Ein Konto pro Arbeitgeber bei SuccessFactors und Workday vervielfacht Zugangsdaten (Kapitel 15.5). Lernen aus wenigen hundert Urteilen kann Zufall als Muster deuten; deshalb Vergleich auf historischen Daten und Freigabe je Änderung. Beta-Produkte (Managed Agents, Routines) können Preise und Verhalten ändern (Kapitel 7.7.2); der Abstraktionslayer `llm/agent_runner.py` hält die Migration rückgängig machbar.

### 19.7 Abgleich mit dem Migrationspfad aus Kapitel 7.10: Auslöser je Baustein

Kapitel 7.10 legt fest, welcher Baustein in welcher Stufe welche Form hat. Die Roadmap ergänzt dazu nur zwei Dinge: das Arbeitspaket, das den Wechsel umsetzt, und den Auslöser, ohne den der Wechsel nicht stattfindet (Leitplanke 6). Ein Auslöser ist immer ein Messwert aus `event_log`, Tracker oder einer dokumentierten Messung, nie ein Bauchgefühl.

| Baustein | MVP-Paket | v1-Paket | v2-Paket | Auslöser für den nächsten Schritt |
|---|---|---|---|---|
| Laufzeit | M-01, M-14 (Agent SDK + Messages API, systemd) | V-17 optional: nur Rechercheur als Scheduled Deployment | W-10 Vollmigration oder Verbleib | Managed Agents laut Anthropic-Doku nicht mehr Beta, und ein konkreter Bedarf an Vaults oder Webhooks; nie vor abgenommenem v1 |
| Modelle | M-06, M-08, M-09 (Fable 5.1, Sonnet 5, Haiku 4.5; Kritiker-Modell in `config/modelle.yaml`) | V-02 effort-Sweep, V-13 Zweitgutachter Opus 5 | W-06 Sweep mit Freigabedaten | 20 freigegebene Bewerbungen liegen vor (Kalibrierungstermin) |
| Quellen | M-02 (BA-API, Personio, Greenhouse, Lever, Adzuna, Arbeitnow, Alert-Mails) | V-08 SerpAPI, V-09 weitere Adapter, V-10 Detektor | W-08 Playwright MCP, JOIN, Gelb-Quellen nach Freigabe | M-15 belegt eine Abdeckungslücke; Watchlist-Abdeckung unter 90 %; für Gelb zusätzlich deine Freigabe in `config/quellen.yaml` |
| Matching | M-06 (Muss-Filter, BM25, Judge im Batch) | V-11 Embeddings, LanceDB, Reranker | W-05 gelernte Gewichte | Vergleichstest an 100 bewerteten Anzeigen zeigt, dass BM25 relevante Treffer verfehlt; für Lernen mindestens 100 strukturierte Cockpit-Urteile |
| Versand | M-13 Entwurfsmodus | V-04 SMTP nach Freigabe, Sendeprotokoll | W-01 bis W-04 Portal-Co-Pilot | MVP-DoD Punkt 5 erfüllt und 20 Werktage stabil; für Portale: Anteil der Portal-Stellen an der Tagesauswahl über vier Wochen mindestens ein Drittel (Annahme, im Tracker zählbar) |
| Datenbank | SQLite | + LanceDB nur mit V-11 | W-11 Postgres/pgvector nur bei Mehrnutzer | zweiter realer Nutzer steht an (Go nach Kapitel 21.5) |
| Secrets | P0-09 sops + age | unverändert | Vaults nur bei Migration | wie Laufzeit |
| Observability | M-14 `event_log`, Digest, Wochenabgleich | V-14 Phoenix | Langfuse nur mit größerem Server | Kosten- oder Latenzabweichung lässt sich aus `event_log` nicht mehr erklären; sonst kein Wechsel |
| Gedächtnis und Lernen | Profil-Dateien, `company.last_contacted_at`, Blacklist | V-12 Feedback-Tags werden `profilvorschlag` | W-05, W-06; Memory Store `read_only` bei Migration | mindestens 100 strukturierte Urteile; jede Profiländerung weiterhin nur durch dich |
| Cockpit | M-12 Telegram + Detailseite | V-12 Kanban, Statistik, Profilformular | W-09 Anmeldung, Mandant | Review-Zeit-Median über 10 Minuten oder verlorene Rückfragen ziehen V-12 vor; Mehrnutzer nur bei Go |
| Dokumente | M-11 WeasyPrint, python-docx, `sachlich` | V-16 Typst-Evaluation, `klassisch`, `international-en` | W-07 Motivationsschreiben, Kurzbewerbung | Zielrollen verlangen Englisch oder konservatives Layout (Kapitel 22); Tracker zählt wiederholt Anzeigen, die ein Motivationsschreiben fordern |

Der Wechsel ist bausteinweise möglich, weil alle Modellaufrufe hinter `src/bewerbungsagent/llm/` liegen (Kapitel 7.10) und alle Quellen hinter dem Adapter-Interface aus Kapitel 9.1. Rückwärts geht es genauso: Ein Baustein, dessen Auslöser sich später als Fehlmessung erweist, fällt ohne Codeänderung an anderer Stelle auf die MVP-Stufe zurück (Konfigurationsschalter in `config/quellen.yaml`, `config/modelle.yaml`, `config/zeitplan.yaml`).

### 19.8 Arbeitsweise mit Claude Code

Die Aufwandsannahmen dieses Kapitels gelten nur, wenn die Bauarbeit in einer festen Form läuft. Vier Regeln:

1. **Ein Arbeitspaket, eine Sitzung, ein Auftrag.** Jede Claude-Code-Sitzung bekommt genau ein Paket aus 19.3 bis 19.6 mit Ziel, Eingaben, Ausgaben, Abnahme und Sperrliste. Die Sitzung endet erst, wenn `pytest` grün ist und die Abnahmebedingung erfüllt ist; halbfertige Pakete werden nicht committet.
2. **`CLAUDE.md` ist Kontext, Settings und Hooks sind die Regel.** `CLAUDE.md` enthält die Statusbegriffe, die zehn Sicherheitsregeln und die Testpflicht (Kapitel 7.9; [Memory](https://code.claude.com/docs/en/memory)); erzwungen wird über `.claude/settings.json` und den `PreToolUse`-Hook, die auch beim Bauen aktiv sind. Claude Code schreibt beim Bauen nie in `profil/`, sieht nie `config/secrets.enc.yaml` im Klartext und baut keinen Versandpfad ohne die negativen Bote-Tests aus dem Testplan.
3. **Definition of Ready je Paket.** Ein Paket startet erst, wenn die Schemas, Fixtures und Konfigurationsdateien, die es braucht, im Repo liegen; sonst baut Claude Code gegen Annahmen. Für Strang B in 19.4 heißt das: committetes Profil und ein Beispiel-Dossier; für Strang C: das Dossier-Skelett aus Kapitel 13.3.
4. **Deine Prüfroutine je Paket, in dieser Reihenfolge:** Diff lesen (Ziel: 15 Minuten), Tests selbst laufen lassen, das Paket an drei echten Fällen ausprobieren, dann Commit. Was du dabei findest, wird als Fixture oder Test festgehalten, nicht als Notiz.

Arbeitsauftrag-Vorlage, mit der jede Sitzung beginnt (Datei `docs/auftraege/<paket>.md`, damit sie versioniert ist):

```text
Arbeitspaket:  M-04 Dedup (Kapitel 9.4)
Ziel:          Rohtreffer aus mehreren Quellen in kanonische Anzeigen zusammenführen;
               Grenzband als "mögliches Duplikat" markieren, nie automatisch verwerfen.
Eingaben:      schemas/extraktion.json, tests/fixtures/anzeigen/*.json (anonymisiert),
               config/rubrik.yaml (Schwellen: jaro_winkler 0.90, minhash 0.75–0.85)
Ausgaben:      src/bewerbungsagent/scout/dedup.py, tests/test_dedup.py,
               Migration nur, wenn ein neues Feld nötig ist (dann in db/migrations/)
Regeln:        CLAUDE.md; kein Modellaufruf außer Zweifelsfall (Haiku 4.5, Schema
               {gleiche_stelle, grund}); Schwellen aus config, nicht im Code
Abnahme:       pytest grün; 20 konstruierte Paare korrekt; Grenzband markiert;
               Statuswechsel entdeckt → dedupliziert bzw. archiviert(duplikat) im event_log
Nicht anfassen: profil/, config/secrets.enc.yaml, bote/, tracker/
Offen lassen:  Fragen an mich als Kommentar "FRAGE:" im Code, nicht raten
```

Wochenrhythmus im MVP: Montag Pakete der Woche festlegen und Aufträge schreiben (30 Minuten), Dienstag bis Donnerstag Bau in Sitzungen mit Prüfroutine, Freitag Abnahme des Zwischenergebnisses aus dem Sprintplan und Fehlerprotokoll. Ab Woche 4 kommt die tägliche Review-Routine aus Kapitel 2.13 dazu; sie hat Vorrang vor dem Bau, weil die Betriebsdaten die Auslöser aus 19.7 liefern.

### 19.9 Die ersten 10 konkreten Schritte ab morgen

Alle Schritte gehören zu Phase 0 und lassen sich in fünf bis sechs Werktagen erledigen. Zeitangaben sind Annahmen für deine eigene Zeit.

1. **Blocker entscheiden (P0-01, 30 Minuten).** Fünf Antworten aufschreiben: primäres E-Mail-Konto (iCloud oder Gmail), Server (Hetzner-VPS oder Mac), Top-Modell (Fable 5.1 mit 30-Tage-Speicherung oder Opus 5), Budget je Tageslauf (Default 10 USD), Zielrollen und Orte. Die Datei `config/entscheidungen.md` im Daten-Repository hält sie fest.
2. **Anthropic-Zugang einrichten (P0-02, 45 Minuten).** Organisation, Commercial-API-Key, Ausgabenlimit, Auftragsverarbeitungsvertrag; bei Fable 5.1 die 30-Tage-Speicherung aktivieren und einen Testaufruf machen, der ohne Fehler 400 antwortet.
3. **Server bestellen (P0-03, 1 Stunde).** Hetzner CPX22 in Deutschland, Ubuntu 24.04, SSH-Schlüssel, Nutzer `agent`, Firewall nur SSH; Preis im Konfigurator festhalten.
4. **Kanäle anlegen (P0-04, P0-05, 1 Stunde).** Telegram-Bot bei BotFather mit Chat-ID; bei iCloud App-spezifisches Passwort, bei Gmail OAuth-Projekt im Testing-Modus mit den Scopes aus P0-05. Ein Test-Entwurf per IMAP-APPEND, danach wieder löschen.
5. **Quellenkonten und Job-Alerts (P0-06, P0-07, 45 Minuten).** Adzuna-Konto mit App-ID und Key (Rate Limits notieren), SerpAPI-Konto nur anlegen, Job-Alerts bei StepStone, Indeed, LinkedIn und XING auf das Bewerbungspostfach.
6. **Projektgerüst bauen lassen (P0-08 bis P0-12, 2 bis 3 Stunden, erste Claude-Code-Sitzung).** Zwei Repositories, `CLAUDE.md`, `.claude/settings.json` mit Deny-Regeln und Hook, `pyproject.toml`, `schema.sql` als Migration, erster `pytest`, sops mit age, Systempakete und deaktivierte systemd-Units auf dem Server. Auftrag nach der Vorlage in 19.8.
7. **Material sammeln (P0-13, P0-18, 1 bis 2 Stunden).** Lebenslauf, alte Anschreiben, 5 bis 10 Textproben, Zeugnis-Scans in einen Eingangsordner; dazu 20 bis 50 Wunscharbeitgeber mit Karriereseiten-URL als erste `watchlist.yaml`.
8. **Onboarding, Teil 1 (P0-14, 2 bis 3 Stunden, zweite Claude-Code-Sitzung).** Skill `profil-onboarding` anlegen lassen, dann Blöcke A bis C: Kontakt, Werdegang, 8 bis 10 Erfolge mit Zahl und Beleg. Ergebnis sind `lebenslauf.yaml` und `story_bank.yaml` als Entwürfe.
9. **Onboarding, Teil 2 und Freigabe (P0-15, P0-16, 2 bis 3 Stunden, dritte Sitzung).** Blöcke D bis H, Stimmprofil-Vorschlag aus den Textproben korrigieren, Tabus `NUTZER-001…` setzen, alle fünf Dateien lesen, Commit im Daten-Repository, `candidate_profile` mit Hashes füllen. Danach schreibt kein Modell mehr in `profil/`.
10. **Quellen-Smoke-Tests und Zeugnisse (P0-17, P0-19 bis P0-22, 2 Stunden, vierte Sitzung).** BA-API live aufrufen und als Fixture speichern, je einen Personio-, Greenhouse- und Lever-Feed abrufen, Teamtailor testen, Fingerprints für softgarden, rexx, d.vinci, SuccessFactors und JOIN sammeln; Zeugnisse normalisieren und OCR mit deutschem Sprachpaket; htmx-Lizenz und DIN-5008-Maße prüfen. Damit ist MS0 erreicht, und Woche 1 beginnt mit M-01 und M-02.

### 19.10 Default-Annahmen und offene Fragen (für Kapitel 22)

Bis du anders entscheidest, gilt für die Roadmap:

- Start sofort mit Phase 0; Umsetzung durch Claude Code, Prüfung durch dich; rund 3 PT deiner Zeit je Woche im MVP, danach 1 bis 2 PT je Woche plus tägliche Review-Routine.
- Phasenzuschnitt wie in 19.2; kein Baustein wechselt die Stufe ohne den Auslöser aus 19.7.
- MVP im Entwurfsmodus (du sendest selbst); erster SMTP-Versand in Woche 5 mit Budget 5 USD und Tageslimit 3, danach die Defaults aus Kapitel 7.11.
- Kalibrierungstermin nach 20 freigegebenen Bewerbungen als gemeinsamer Termin von rund 2 PT (V-02).
- Deutsche Vorlage `sachlich` im MVP; `international-en` und `klassisch` erst in v1 nach Test-Parsing.
- Portale im MVP manuell mit Link und Dateien; Co-Pilot erst in v2, zuerst Personio, softgarden, JOIN.
- Managed-Agents-Test in v1 nur für den Rechercheur und nur optional; Entscheidung in v2.
- Anwaltliche Prüfung erst bei einer Go-Entscheidung nach Kapitel 21.5, nicht vor dem Einzelbetrieb.

Offene Fragen an dich, gesammelt in Kapitel 22:

1. Wann startet Phase 0, und wie viel Zeit je Woche kannst du verbindlich einplanen? Default: Start diese Woche, 3 PT je Woche im MVP.
2. Fable 5.1 mit 30-Tage-Speicherung oder Opus 5 für Rechercheur, Autor und Kritiker? Default: Fable 5.1 mit dokumentierter Zustimmung (Kapitel 16.2); Umschalter `MODEL_TOP` bleibt.
3. Hetzner-VPS oder eigener Mac für Betrieb und Entwicklung? Default: VPS; der Mac nur für den Portal-Co-Pilot in v2 (residentielle IP).
4. Primäres E-Mail-Konto: iCloud per IMAP-APPEND oder Gmail per Drafts-API? Default: das Konto, das du heute für Bewerbungen nutzt; beide Wege werden gebaut.
5. Budget je Tageslauf und Monatsbudget für Datenquellen? Default: 10 USD je Tageslauf, 0 EUR Datenquellen im MVP, bis 41 USD in v1.
6. Sind englischsprachige Zielrollen relevant, sodass `international-en` schon im MVP gebraucht wird? Default: nein, v1.
7. Ist der Start des echten Versands in Woche 5 mit Tageslimit 3 und Budget 5 USD akzeptabel, oder soll der Entwurfsmodus länger laufen? Default: Woche 5.
8. Onboarding in zwei Sitzungen zu je 2 bis 3 Stunden oder verteilt über die Woche? Default: zwei Sitzungen.
9. Kannst du in Phase 0 eine Watchlist mit 20 bis 50 Wunscharbeitgebern liefern? Default: ja; sonst startet der MVP nur mit BA-API, Adzuna, Arbeitnow und Alert-Mails.
10. Zweitgutachter auf Opus 5 in v1 grundsätzlich gewünscht, oder nur, wenn V-02 einen Nutzen zeigt? Default: nur nach V-02.
11. Soll der Managed-Agents-Test (V-17) überhaupt stattfinden? Default: optional, nur Rechercheur, nur wenn Zeit bleibt.
12. Welche Portalfamilien zuerst im Co-Pilot (W-02)? Default: Personio, softgarden, JOIN; SuccessFactors und Workday danach; Plattform-Schnellbewerbungen nur nach ausdrücklicher Entscheidung.
13. Soll die anwaltliche Prüfung (W-12) vor v2 budgetiert werden oder erst bei einer Go-Entscheidung? Default: erst bei Go.

**Quellen dieses Kapitels:**

- Anthropic: API and data retention (Covered Models, 30 Tage) – https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
- Anthropic: Pricing (Modellpreise, Tokenizer ab Claude 4.7) – https://platform.claude.com/docs/en/about-claude/pricing
- Anthropic: Batch processing – https://platform.claude.com/docs/en/build-with-claude/batch-processing
- Anthropic: Prompt caching – https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Anthropic: Files API (workspace-weite Sichtbarkeit) – https://platform.claude.com/docs/en/build-with-claude/files
- Anthropic Managed Agents: Scheduled deployments – https://platform.claude.com/docs/en/managed-agents/scheduled-deployments
- Anthropic Managed Agents: Budgets – https://platform.claude.com/docs/en/managed-agents/budgets
- Anthropic Managed Agents: Memory – https://platform.claude.com/docs/en/managed-agents/memory
- Anthropic Managed Agents: Webhooks – https://platform.claude.com/docs/en/managed-agents/webhooks
- Claude Agent SDK (PyPI, Version 0.2.152) – https://pypi.org/project/claude-agent-sdk/
- Claude Agent SDK: Cost tracking – https://code.claude.com/docs/en/agent-sdk/cost-tracking
- Claude Agent SDK: Permissions (canUseTool) – https://code.claude.com/docs/en/agent-sdk/permissions
- Claude Agent SDK: Structured outputs – https://code.claude.com/docs/en/agent-sdk/structured-outputs
- Claude Code: Hooks – https://code.claude.com/docs/en/hooks
- Claude Code: Memory (CLAUDE.md) – https://code.claude.com/docs/en/memory
- Hetzner: Preisanpassung 15.6.2026 – https://docs.hetzner.com/de/general/infrastructure-and-availability/price-adjustment/
- Northflank: Hetzner cloud server price increases – https://northflank.com/blog/hetzner-cloud-server-price-increases
- Hetzner Object Storage – https://www.hetzner.com/storage/object-storage/
- python-telegram-bot (GitHub) – https://github.com/python-telegram-bot/python-telegram-bot
- Apple: iCloud Mail Limits und Einstellungen – https://support.apple.com/en-us/102198
- Gmail API: Scopes – https://developers.google.com/workspace/gmail/api/auth/scopes
- Gmail API: Drafts – https://developers.google.com/workspace/gmail/api/guides/drafts
- Google Cloud: OAuth-Testing-Modus – https://support.google.com/cloud/answer/15549945?hl=en
- Adzuna Developer Portal – https://developer.adzuna.com/
- SerpAPI Pricing – https://serpapi.com/pricing
- sops (GitHub) – https://github.com/getsops/sops
- Apache Tika (GitHub, 4.0.0) – https://github.com/apache/tika
- anthropics/skills: doc-coauthoring SKILL.md – https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md
- Anabin-Kurzanleitung (KMK) – https://anabin.kmk.org/kurzanleitung/ich-moechte-feststellen-wie-mein-auslaendischer-hochschulabschluss-in-deutschland-bewertet-wird.html
- bundesAPI/jobsuche-api (GitHub) – https://github.com/bundesAPI/jobsuche-api
- Greenhouse Job Board API – https://developers.greenhouse.io/job-board.html
- Lever Postings API (GitHub) – https://github.com/lever/postings-api
- Personio: Jobs via XML integrieren – https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML
- webappanalyzer categories.json – https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json
- datasketch (GitHub) – https://github.com/ekzhu/datasketch
- bm25s (GitHub) – https://github.com/xhluca/bm25s
- OpenResume (GitHub) – https://github.com/xitanggg/open-resume
- WeasyPrint – https://weasyprint.com/
- imap_tools (GitHub) – https://github.com/ikvk/imap_tools
- icalendar (PyPI) – https://pypi.org/project/icalendar
- Jina Reader – https://jina.ai/reader/
- Firecrawl Pricing (scrapegraphai) – https://scrapegraphai.com/blog/firecrawl-pricing
- LanceDB (GitHub) – https://github.com/lancedb/lancedb
- Cohere Rerank 3.5 (OpenRouter) – https://openrouter.ai/cohere/rerank-v3.5
- ESCO: Download – https://esco.ec.europa.eu/en/use-esco/download
- Arize Phoenix (GitHub) – https://github.com/Arize-ai/phoenix
- mailbox.org: Preispläne – https://mailbox.org/en/news/new-price-plans-available-mailboxorg/
- Fastmail: Preise und Pläne – https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates
- Playwright MCP README (browser_file_upload) – https://raw.githubusercontent.com/microsoft/playwright-mcp/main/README.md
- Skyvern (GitHub, AGPL-3.0) – https://github.com/Skyvern-AI/skyvern
