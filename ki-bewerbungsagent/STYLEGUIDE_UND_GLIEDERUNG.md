# Styleguide und Gliederung für den Masterplan „Persönlicher KI-Bewerbungsagent“

Dieses Dokument lesen ALLE Kapitel-Autoren vollständig, bevor sie schreiben.

## 1. Zweck des Gesamtdokuments

Ein Masterplan (Konzept + Bauplan + Roadmap) für einen persönlichen KI-Bewerbungsagenten, der für EINE Person in Deutschland funktioniert und später als Produkt verkauft werden könnte. Leser: der Auftraggeber (technikaffin, kein Entwickler-Profi, will das System mit Claude Code / Claude bauen lassen) sowie später Entwickler, die es umsetzen. Das Dokument muss so konkret sein, dass man damit die Umsetzung als Projekt starten kann: Entscheidungen, Begründungen, Werkzeuge, Schritte, Kosten, Risiken, offene Fragen.

Leitsatz des Projekts: Qualität statt Masse. Eine sorgfältig recherchierte, in der echten Stimme des Kandidaten geschriebene Bewerbung soll mehr wert sein als 100 generische Massenbewerbungen. Der Mensch gibt jede Bewerbung frei (Human-in-the-Loop); der Agent recherchiert, bewertet, schreibt, prüft, setzt und bereitet den Versand vor.

## 2. Sprache und Ton

- Deutsch. Sie-Form vermeiden; den Auftraggeber direkt mit „du“ ansprechen, wo nötig (er hat im Auftrag „du“ verwendet). Sonst neutral-sachlich.
- Produkt-, Tool- und Fachbegriffe auf Englisch belassen (ATS, Scraping, Embedding, MCP-Server, Prompt Caching). Beim ersten Vorkommen kurz erklären.
- Konkret statt vage: Zahlen mit Einheit und Datum, Produktnamen, Versionen, Paragraphen, Preise mit Währung. Keine Marketingsprache, keine Floskeln („innovativ“, „revolutionär“, „nahtlos“, „state of the art“).
- Jede Behauptung, die aus der Recherche stammt, bekommt eine Quellenangabe als Markdown-Link in Klammern oder Fußnotenstil `[Quelle](URL)`. Nur URLs verwenden, die in den Research-Dateien vorkommen. Keine URLs erfinden.
- Wenn die Recherche etwas nicht belegen konnte oder der Prüfer „refuted/unverifiable“ gesagt hat: entweder weglassen oder klar als „unbestätigt“ kennzeichnen. Prüfer-Korrekturen haben Vorrang vor der ursprünglichen Behauptung.
- Kurze Absätze. Tabellen für Vergleiche (Tool | Zweck | Zugriff | Kosten | Bewertung). Nummerierte Listen für Abläufe. Keine Emojis.
- Entscheidungen klar markieren: **Entscheidung:** … **Begründung:** … **Alternative(n):** … Wenn eine Entscheidung vom Nutzer abhängt, in „Offene Fragen“ aufnehmen und eine Default-Annahme nennen.
- Kein Selbstbezug („in diesem Kapitel werde ich…“). Keine Wiederholung anderer Kapitel; stattdessen Querverweis („siehe Kapitel 9“).

## 3. Verbindliche Terminologie (überall gleich verwenden)

Arbeitstitel des Systems: **der Bewerbungsagent** (kein Produktname; Namensfindung ist offene Frage an den Nutzer).

Komponenten (Rollen des Systems; als Subagents/Skills umsetzbar):
- **Kandidatenprofil** – strukturierte Daten über den Nutzer: Lebenslauf-Basis (Master-Lebenslauf), Story-Bank (belegte Erfolge, Zahlen, Beispiele), Stimmprofil (Schreibstil, Wortwahl, Tabus), Präferenzen (Rollen, Orte, Gehalt, Ausschlüsse), Standardantworten für Formulare.
- **Scout** – findet Stellenanzeigen aus den Quellen (Jobbörsen, Karriereseiten, Feeds), normalisiert und dedupliziert.
- **Matcher** – bewertet Passung und Attraktivität, erklärt die Bewertung, wählt die Tages-Top-Liste.
- **Rechercheur** – recherchiert Unternehmen, Standort/Anschrift, Ansprechpartner, Kultur, Neuigkeiten, ATS-Typ; liefert Konfidenzen und Rückfragen.
- **Autor** – schreibt Anschreiben und passt den Lebenslauf an (nur Umordnen/Betonen/Formulieren, nie Erfinden).
- **Kritiker** – prüft Entwürfe gegen Rubrik, Story-Bank, Stellenanzeige und Stimmprofil; fordert Überarbeitungen an.
- **ATS-Prüfer** – Keyword-/Format-Abgleich, Test-Parsing der erzeugten Dateien.
- **Setzer** – erzeugt PDF/DOCX aus Vorlagen (Anschreiben, Lebenslauf, Bewerbungsmappe).
- **Review-Cockpit** – die Oberfläche für Prüfung, Checklisten, Freigabe, Rückfragen.
- **Bote** – legt E-Mail-Entwürfe an bzw. versendet nach Freigabe; füllt Portal-Formulare vor.
- **Tracker** – Status-Pipeline, Rückmeldungen, Nachfassen, Statistik.
- **Orchestrator** – Zeitplan (Läufe), Reihenfolge, Budget, Fehlerbehandlung, Protokoll.

Status-Pipeline einer Stelle (immer diese Begriffe): entdeckt → dedupliziert → bewertet → ausgewählt → recherchiert → geschrieben → geprüft → bereit zur Freigabe → freigegeben → gesendet → Rückmeldung → Interview → Absage / Zusage / archiviert. Zusätzlich: „Rückfrage offen“ (wartet auf Antwort des Nutzers).

Läufe: „Tageslauf“ (der geplante Durchlauf, z. B. 06:00 Uhr), „Nachlauf“ (Antwortverarbeitung, Nachfassen).

Modelle: Claude Fable 5.1 (`claude-fable-5-1`, $10/$50 pro 1M Token Input/Output), Claude Opus 5 (`claude-opus-5`, $5/$25), Claude Sonnet 5 (`claude-sonnet-5`, $2/$10), Claude Haiku 4.5 (`claude-haiku-4-5`, $1/$5). Fable 5.1 ist die Wunschbasis des Nutzers für die qualitätskritischen Schritte (Recherche-Synthese, Schreiben, Kritik); günstigere Modelle für Massenarbeit (Extraktion, Klassifikation, Dedup). Hinweis: Fable 5.1 verlangt 30-Tage-Datenspeicherung bei Anthropic (kein Zero Data Retention) und hat immer aktives Thinking; Tiefe über `effort` steuern.

## 4. Gliederung des Gesamtdokuments (Kapitelnummern verbindlich)

0. Zusammenfassung für Eilige (1 Seite: Was, Warum, Wie, Was es kostet, Was als Erstes zu tun ist)
1. Vision, Ziele, Nicht-Ziele, Erfolgskriterien (KPIs)
2. So arbeitet der Bewerbungsagent: ein Tag im Leben (Ende-zu-Ende-Ablauf mit Beispiel)
3. Markt und Wettbewerb: was es gibt, warum es nicht reicht
4. Der Gegner: ATS und KI-Screening in Deutschland (wie sie scannen, was folgt)
5. Deutsche Bewerbungskultur 2026: was HR wirklich erwartet (Produktkonsequenzen)
6. Datenquellen: Jobbörsen, Karriereseiten, APIs und ihre Zugriffswege (Matrix, Empfehlung)
7. Architektur: Komponenten, Datenmodell, Pipeline, Zeitplan, Tech-Stack-Entscheidung
8. Modul Kandidatenprofil: Master-Lebenslauf, Story-Bank, Stimmprofil, Onboarding-Interview
9. Modul Scout und Matcher: Suche, Normalisierung, Dedup, Scoring, Ghost-Jobs, Tagesauswahl
10. Modul Rechercheur: Unternehmen, Anschrift, Ansprechpartner, Rückfrage-Protokoll
11. Modul Autor und Kritiker: Anschreiben-Engine, Lebenslauf-Tailoring, Anti-Generik-Regeln, Rubrik, Qualitätsschleife
12. Modul ATS-Prüfer: Keyword-Abgleich, Formatregeln, Test-Parsing
13. Modul Setzer: Dokumentenerzeugung (PDF/DOCX), Vorlagen, DIN 5008, Dateinamen, Bewerbungsmappe
14. Modul Review-Cockpit: Oberfläche, Checklisten, Diff, Freigabe, Rückfragen, Feedback-Schleife
15. Modul Bote und Tracker: E-Mail-Anbindung, Sendeprotokoll, Portale, Antworten, Nachfassen
16. Recht, Datenschutz, Ethik: Compliance-Katalog (darf autonom / nur mit Freigabe / nie)
17. Werkzeugkasten: Skills, Subagents, MCP-Server, CLIs, Bibliotheken, APIs, Dienste (Inventar mit Zweck, Kosten, Link)
18. Kosten: pro Bewerbung, pro Monat; Infrastruktur; Datenquellen; Sparhebel
19. Roadmap: Phase 0 (Vorbereitung), MVP (Wochen 1–4), v1 (Monat 2–3), v2 (Monat 4–6); Meilensteine, Definition of Done, erste konkrete Schritte
20. Risiken und Gegenmaßnahmen
21. Später: Kommerzialisierung (kurz)
22. Offene Fragen an dich (Fragenkatalog mit Default-Annahmen)
23. Glossar
24. Quellenverzeichnis

## 5. Formale Regeln für Kapitel-Dateien

- Dateiname: `NN-kurzname.md` (NN zweistellig). Überschrift erster Ebene `## NN. Titel` (das Gesamtdokument hat nur eine `#`-Überschrift ganz oben). Unterüberschriften `###` und `####`.
- Umfang: je nach Kapitel 700–3.000 Wörter; Kapitel 7, 11, 17, 19 dürfen länger sein. Lieber vollständig als kurz, aber ohne Füllstoff.
- Am Ende jedes Kapitels ein Block `**Quellen dieses Kapitels:**` mit den verwendeten Links (Titel + URL). Kapitel 24 sammelt sie später zentral; trotzdem pro Kapitel angeben.
- Querverweise als „Kapitel N“.
- Tabellen: maximal 6 Spalten, kurze Zellen. Codeblöcke für Schemata (JSON/YAML), Ordnerstrukturen, Beispielprompts, Konfigurationen.
- Wo Beispiele helfen (Anschreiben-Struktur, Rubrik, Checkliste, Schema), Beispiele ausformulieren. Beispiel-Anschreiben nur als Struktur/Skelett mit Platzhaltern, da wir das Kandidatenprofil nicht kennen.
- Was wir über den Nutzer NICHT wissen (Branche, Beruf, Standort, Zielrollen, Sprache der Bewerbungen): nicht raten, sondern als Variable behandeln und in Kapitel 22 fragen.
