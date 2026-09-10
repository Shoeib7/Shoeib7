## 0. Zusammenfassung für Eilige

### 0.1 Was gebaut wird

Der Bewerbungsagent ist ein persönliches System für genau eine Person in Deutschland: Er findet Stellenanzeigen, recherchiert das Unternehmen, entwirft Anschreiben und eine angepasste Lebenslauf-Fassung in der eigenen Stimme, prüft beides gegen Bewerbermanagementsysteme (ATS), setzt PDF und DOCX und legt alles täglich zur Freigabe vor. Er ist kein Auto-Apply-Werkzeug: Der Schritt „bereit zur Freigabe → freigegeben“ bleibt eine explizite menschliche Handlung, ein Zeitablauf zählt nie als Freigabe (Kapitel 1, 14). Gebaut wird es mit Claude Code, betrieben auf einem eigenen Server in Deutschland (Kapitel 7, 19).

### 0.2 Warum das besser ist als Massenbewerbung

- Recruiter bearbeiten laut dem Greenhouse „2025 AI in Hiring Report“ (über 4.100 Befragte, Deutschland eingeschlossen) fast dreimal so viele Bewerbungen pro Stelle wie 2021; 91 Prozent haben Täuschungsversuche bemerkt (Kapitel 1.1, 3.5).
- Volumen-Werkzeuge liefern schwache Ergebnisse: LazyApply 2,1/5 auf Trustpilot, Massive 1,8/5 mit dokumentierten Lebenslauf-Halluzinationen, JobCopilot Callback-Raten unter 2 Prozent und Bewerbungen an betrügerische Anzeigen (Kapitel 3.4).
- Laut Resume Genius Hiring Trends Report 2026 nennen 53 Prozent der Hiring Manager „KI-generierten Inhalt“ als größtes Red Flag im Lebenslauf (Kapitel 3.5).

Nicht KI ist das Risiko, sondern erkennbare Generik – dagegen stehen ein bis zwei nachprüfbare Recherchedetails je Anschreiben und null unbelegte Aussagen als hartes Gate (Kapitel 1.3).

### 0.3 Wie es funktioniert

Elf Komponenten und der Orchestrator führen jede Stelle durch die Status-Pipeline von „entdeckt“ bis „archiviert“; das Kandidatenprofil mit Master-Lebenslauf, Story-Bank und Stimmprofil ist die einzige Quelle der Wahrheit, in die kein Modell schreibt (Kapitel 7.4, 8). Um 03:30 Uhr sammelt und dedupliziert der Scout alle Quellen, der Matcher reicht 30 bis 50 gefilterte Kandidaten als Batch beim Judge ein. Um 05:30 Uhr entsteht die Tagesauswahl – höchstens zehn, maximal zwei je Arbeitgeber –, und der Rechercheur liefert je Stelle ein Dossier mit Quelle und Konfidenz je Feld; unter 0,7 fragt er nach, statt zu raten (Kapitel 9, 10). Ab 06:05 Uhr schreibt der Autor zwei bis drei Varianten, der Kritiker prüft sie in frischem Kontext gegen Rubrik, Story-Bank und Stimmprofil (höchstens zwei Schleifen), danach folgen ATS-Prüfer und Setzer; „bereit zur Freigabe“ wird erst nach Setzer-QA und ATS-Prüfer-Stufe 2 gesetzt (Kapitel 11–13). Um 06:45 Uhr meldet der Orchestrator per Telegram und E-Mail-Digest; du prüfst im Cockpit anhand Checkliste, Wort-Diff und Vorschau und wählst Freigeben, Ändern, Ablehnen oder Später (Kapitel 14). Rückfragen blockieren nur die betroffene Bewerbung: Erinnerung nach vier Stunden, dann täglich im Digest; mit Default löst der nächste Tageslauf sie auf, ohne Default werden sie nach fünf Werktagen archiviert. Nach Freigabe und 60 Sekunden Undo-Fenster sendet der Bote; der Tracker verarbeitet Antworten, Termine und Nachfass-Fälligkeiten (Kapitel 15).

### 0.4 Zentrale Entscheidungen

- **Tech-Stack:** Python 3.12, Claude Agent SDK für Rechercheur, Autor und Kritiker, Messages API für Batch; Hetzner-VPS CPX22 in Deutschland, SQLite, sops+age, WeasyPrint als Primär-Renderer mit python-docx (Typst nur Evaluation in v1), Cockpit als FastAPI+HTMX-Seite plus Telegram-Bot (Kapitel 7, 13, 14).
- **Quellen-Strategie:** Ampel Grün/Gelb/Rot; der MVP nutzt nur Grün: BA-Jobsuche-API, ATS-Feeds der Watchlist (Personio, Greenhouse, Lever), Adzuna, Arbeitnow, selbst abonnierte Job-Alert-Mails. SerpAPI (Gelb) erst in v1 nach dokumentierter Abdeckungslücke; automatisiertes Auslesen von LinkedIn, StepStone, Indeed, XING und Monster (Rot) nie (Kapitel 6, 16).
- **Versandstrategie:** Im MVP Entwurfsmodus: Der Bote legt die Mail per IMAP-APPEND bzw. Gmail-Drafts-API im eigenen Postfach ab, du sendest selbst. Ab v1 SMTP nach Freigabe, Di–Do 07:00–09:30 Uhr (optional 14:00–16:00), Versatz 15 bis 40 Minuten, Tageslimit 10. Portale nur im Co-Pilot-Modus; „Absenden“ und DSGVO-Einwilligung bleiben beim Menschen (Kapitel 15, 16.8).
- **Modellmix:** Rechercheur und Autor auf Claude Fable 5.1 (Recherche und Entwürfe effort high, Überarbeitung medium), der Kritiker als vom Autor getrenntes Grader-Modell auf Claude Opus 5 (Rubrik high, Stimm-/Leser-Test medium), kein Zweitgutachter. Claude Sonnet 5 für Briefing, Fakten-Check, ATS-Lesetest und Tagesauswahl (Batch), Claude Haiku 4.5 für Extraktion, Dedup, Keyword-Extraktion, Injection- und Konsistenz-Check sowie Tracker-Klassifikation mit Eskalation auf Sonnet 5 unter Konfidenz 0,7. `MODEL_TOP=claude-opus-5` ersetzt Fable 5.1 überall (Kapitel 7.6, 18).

### 0.5 Was es kostet

Verbindlich ist Kapitel 18; Grundlage: 10 Bewerbungen/Tag, 22 Arbeitstage, 220/Monat.

| Szenario | je Bewerbung | Gesamt/Monat |
|---|---|---|
| sparsam | ≈ 1,06 € | ≈ 262 € |
| empfohlen (Standard) | ≈ 3,45 € | ≈ 789 € |
| maximal | ≈ 4,85 € | ≈ 1.151 € |

**Entscheidung:** „empfohlen“ ist der Standard-Betriebsmodus (Kapitel 18.6). Enthalten sind der Hetzner CPX22 für rund 20 €/Monat (19,49–19,99 € je nach Quelle; vor Bestellung prüfen), rund 9 € Massen-Scan und 0 € Datenquellen; je Bewerbung sind das rund 3,59 € (Kapitel 18.8).

### 0.6 Was als Erstes zu tun ist

Alle Schritte gehören zu Phase 0, sind in sechs bis acht Werktagen erledigt und enden mit Meilenstein MS0 „Startklar“ (Kapitel 19.9).

1. Blocker aus 0.7 entscheiden, in `config/entscheidungen.md` festhalten.
2. Anthropic-Zugang: Organisation, Commercial-API-Key, Ausgabenlimit, Auftragsverarbeitungsvertrag; bei Fable 5.1 die 30-Tage-Speicherung aktivieren und testen.
3. Hetzner CPX22 bestellen: Ubuntu 24.04, SSH-Schlüssel, Nutzer `agent`, Firewall nur SSH.
4. Kanäle anlegen: Telegram-Bot, E-Mail-Zugang mit Test-Entwurf per IMAP, Adzuna-Konto, Job-Alerts aufs Bewerbungspostfach.
5. Repos, Projektgerüst und Serverbereitstellung mit Claude Code bauen lassen.
6. Material sammeln (Lebenslauf, alte Anschreiben, 5 bis 10 Textproben, Zeugnisse, Watchlist mit 20 bis 50 Firmen), Onboarding-Interview in zwei Sitzungen, Profil per Commit freigeben.
7. Quellen-Smoke-Tests, ATS-Fingerprints, Zeugnisse per OCR aufbereiten.

### 0.7 Die acht Blocker vor Phase 0

Default-Annahmen vollständig in Kapitel 22.1:

1. Zielrolle(n), Branche, Senioritätsstufe – kein Default möglich.
2. Zielregion(en), Pendeldistanz, Remote – kein Default möglich.
3. Sprache der Bewerbungen – Deutsch, Englisch nur bei eindeutigem Signal.
4. Primäres E-Mail-Konto – technologieoffen umgesetzt, iCloud als einfachster Start.
5. Commercial-API-Key oder persönliches Pro/Max-Abo – Commercial-API-Key.
6. Hosting: Hetzner-VPS, Mac lokal oder Managed Agents – Hetzner-VPS in Deutschland.
7. Fable 5.1 mit 30-Tage-Speicherung oder Opus 5 mit Zero Data Retention – Fable 5.1 mit Offenlegung, `MODEL_TOP`-Umschalter bleibt.
8. Tägliches und monatliches Kostenlimit – Monatsobergrenze spätestens beim Ausgabenlimit in der Anthropic Console setzen.

### 0.8 Grenzen und Risiken

Der ATS-Prüfer testet mit kostenlosen Stellvertretern (Apache Tika, OpenResume-Parser), nicht gegen die realen Parser der Zielsysteme; ein „bestanden“ senkt das Risiko groben Parsing-Versagens, sagt aber nichts über ein Ranking (Kapitel 12.10). Die Stimm-Übertragung aus wenigen Textproben bleibt laut Forschung hinter menschlichem Schreiben zurück, und mehrere zitierte Kennzahlen sind als unbestätigt gekennzeichnet (Kapitel 8.1, 20.1). Größte Betriebsrisiken sind die inoffizielle BA-Jobsuche-API ohne SLA, ungültig werdende Zugangsdaten und der Ermüdungseffekt beim Freigeben – dagegen stehen Checkliste, Wort-Diff und die Regel, dass kein Timeout eine Freigabe ist (Kapitel 20).

**Quellen dieses Kapitels:** keine neuen Fakten; Belege in den zitierten Kapiteln, insbesondere 3.4/3.5, 18.6/18.8, 19.9 und 22.1.
