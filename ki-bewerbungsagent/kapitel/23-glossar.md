## 23. Glossar

Begriffe alphabetisch. Komponentennamen, Status-Begriffe und Modellbezeichnungen wie in Kapitel 7 festgelegt; englische Fach- und Produktbegriffe bleiben unübersetzt. Rechtsbegriffe tragen eine Fundstelle, unbestätigte Recherchebefunde sind entsprechend gekennzeichnet.

**Agent SDK (Claude Agent SDK).** Python-/TypeScript-Bibliothek (`claude-agent-sdk`, MIT-Lizenz, Version 0.2.152 vom 2.9.2026), die die Claude-Code-CLI bündelt und Subagents, Hooks, Permission-Modes und Structured Outputs bereitstellt ([PyPI](https://pypi.org/project/claude-agent-sdk/)). Bildet zusammen mit dem Messages-API-SDK die technische Basis der Architektur aus Kapitel 7; Rechercheur, Autor und Kritiker laufen darüber als Subagents.

**AGG (Allgemeines Gleichbehandlungsgesetz).** Deutsches Antidiskriminierungsgesetz; macht Foto, Geburtsdatum, Familienstand und Konfession im Lebenslauf rechtlich freiwillig. Kandidatenprofil (Kapitel 8) und Setzer (Kapitel 13) lassen diese Angaben deshalb standardmäßig weg; ihre Aufnahme ist nie automatisch, sondern nur mit expliziter Freigabe erlaubt (Kapitel 16).

**AI Act (KI-VO).** EU-Verordnung (EU) 2024/1689 zur Regulierung von KI-Systemen. Der Digital Omnibus (Verordnung (EU) 2026/1744, in Kraft seit 27.7.2026) verschiebt die Hochrisiko-Pflichten für Annex-III-Systeme wie Recruiting vom 2.8.2026 auf den 2.12.2027 ([Gibson Dunn](https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/)). Diese Pflichten treffen den Arbeitgeber als Betreiber, nicht dich als Bewerber: Art. 2 Abs. 10 KI-VO nimmt rein persönliche, nicht berufliche Nutzung aus ([ai-act-law.eu](https://ai-act-law.eu/de/artikel/2/); Kapitel 16).

**ATS (Applicant Tracking System, Bewerbermanagementsystem).** Software, mit der Arbeitgeber Stellenanzeigen verwalten, Bewerbungen sammeln und teils automatisiert vorsortieren. In Deutschland sollen laut DGFP-Benchmarkstudie 2025 vier Anbieter über 5 Prozent Marktanteil liegen – Personio, SAP SuccessFactors, softgarden, rexx systems ([DGFP](https://www.dgfp.de/aktuell/recruiting-strukturen-2025-recruiting-wird-strukturierter-datengetriebener-und-technologischer); Aussage vom Faktenprüfer nicht bestätigbar, dgfp.de gesperrt, daher unbestätigt); ATS-Prüfer (Kapitel 12) und Scout (Kapitel 9) priorisieren ihre Erkennungslogik trotzdem entsprechend.

**ATS-Prüfer.** Komponente (Kapitel 12), die erzeugte Dokumente gegen Keyword- und Formatregeln prüft und über eine lokale Zwei-Parser-Gegenprobe testet, wie ein generischer Parser sie tatsächlich liest.

**AÜG (Arbeitnehmerüberlassungsgesetz).** Regelt Zeitarbeit; § 11 AÜG verpflichtet den Verleiher nur zur schriftlichen Information von Arbeitnehmer und Entleiher, nicht zu einer Kennzeichnung im Stelleninserat selbst ([§ 11 AÜG](https://www.gesetze-im-internet.de/a_g/__11.html)). Der Scout kann Zeitarbeit/Personalvermittlung deshalb nur heuristisch erkennen, nie zuverlässig allein aus dem Anzeigentext (Kapitel 9).

**Autor.** Komponente (Kapitel 11), die Anschreiben schreibt und den Lebenslauf anpasst – ausschließlich durch Umordnen, Betonen und Formulieren, nie durch Erfinden; jede Zahlenaussage bekommt eine `claims`-Referenz auf die Story-Bank.

**AVV (Auftragsverarbeitungsvertrag).** DSGVO-Vertrag nach Art. 28 zwischen Verantwortlichem und Auftragsverarbeiter (hier: dir und Anthropic). Solange der Agent nur für dich privat arbeitet, greift wahrscheinlich die Haushaltsausnahme; sobald das System für weitere Nutzer geöffnet wird, ist ein AVV mit Anthropic Pflicht (Kapitel 16, 21).

**BA-Jobsuche-API.** Inoffizielle, reverse-engineerte REST-Schnittstelle der Bundesagentur für Arbeit (`bundesAPI/jobsuche-api`, fester Header `X-API-Key: jobboerse-jobsuche`, keine dokumentierten Rate-Limits) ([GitHub](https://github.com/bundesAPI/jobsuche-api)). Rechtlich die sauberste Kernquelle des Scout (Kapitel 6, 9), technisch aber ohne SLA – kann jederzeit ohne Ankündigung geändert werden.

**Batch API.** Anthropics asynchrone Messages-Batches-Schnittstelle: 50 Prozent Rabatt auf alle Tokenpreise, Ergebnisse meist innerhalb einer Stunde, spätestens nach 24 Stunden, 29 Tage abrufbar, mit Prompt-Caching-Rabatt kombinierbar ([Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing)). Der Matcher (Kapitel 9) bewertet darüber die nächtliche Masse an Stellenanzeigen; der Judge läuft im Batch, Rechercheur, Autor und Kritiker laufen live (Kapitel 7).

**BM25.** Klassischer lexikalischer Retrieval-Algorithmus (Keyword-Ranking); die Bibliothek `bm25s` mit deutschem Stemmer bildet zusammen mit Dense-Embeddings (BGE-M3) und Reciprocal-Rank-Fusion das Hybrid-Retrieval des Matchers, bevor Reranking und Judge die Top-Auswahl treffen (Kapitel 9).

**Bote.** Komponente (Kapitel 15), die E-Mail-Entwürfe anlegt bzw. nach Freigabe versendet und Portal-Formulare vorbefüllt. Im MVP läuft er im Entwurfsmodus für E-Mail und im Co-Pilot-Modus für ATS-Formulare; erst in v1 sendet er selbst per SMTP im Versandfenster.

**Claims.** Strukturierte Liste jeder Faktenbehauptung im Anschreiben, jeweils mit Story-Bank-ID (Kapitel 8) referenziert. Der Kritiker gleicht jeden Claim gegen die Story-Bank ab (belegt / nicht belegt / übertrieben, per Claude Sonnet 5) und blockiert unbelegte Behauptungen vor der Freigabe (Kapitel 11).

**Code-Execution-Tool.** Sandboxed Container-Werkzeug der Claude-API mit vorinstallierten Bibliotheken (python-docx, pypdf, reportlab u. a.); Basis der Anthropic-Skills `docx`/`pdf`, die der Setzer nutzt. Kostenlos in Kombination mit Web-Search/-Fetch, sonst 1.550 Freistunden pro Organisation und Monat, danach 0,05 $/Stunde ([Code execution tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool); Kapitel 13, 17).

**Covered Model.** Anthropic-Kategorie für Modelle, die zwingend 30 Tage Datenspeicherung voraussetzen und ohne gesonderte Freigabe nicht unter Zero Data Retention laufen; Claude Fable 5.1 gehört dazu, Claude Opus 5, Claude Sonnet 5 und Claude Haiku 4.5 nicht ([API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)). Für die sensibelsten Verarbeitungsschritte ist Opus 5 deshalb die ZDR-fähige Alternative zu Fable 5.1 (Kapitel 16).

**Cowork (Claude Cowork).** Anthropic-Produkt mit geplanten Aufgaben, Dateizugriff und eingebautem Browser, in jedem bezahlten Abo ab Pro enthalten. Pragmatische, aber weniger scriptbare Alternative zum selbst gehosteten Agent SDK; im Masterplan nicht als Kernarchitektur gewählt (Kapitel 7).

**DIN 5008.** Deutsche Norm für den Geschäftsbrief: Das Anschriftfeld beginnt 4,5 cm vom oberen Rand, die Betreffzeile folgt zwei Leerzeilen darunter. Referenzstandard des Setzers für das Anschreiben-Layout (Kapitel 13); WeasyPrint setzt die Maße über CSS Paged Media um.

**Dossier.** Strukturiertes Rechercheergebnis des Rechercheurs: Jedes Feld trägt Quelle und Konfidenz; unter der Schwelle entsteht eine Rückfrage statt einer Annahme. Geht an Autor, Setzer und Review-Cockpit (Kapitel 10).

**DSGVO.** EU-Datenschutz-Grundverordnung. Für die rein private Jobsuche einer Person greift voraussichtlich die Haushaltsausnahme (Art. 2 Abs. 2 lit. c); sie entfällt vollständig, sobald Daten Dritter systematisch verarbeitet werden oder das System für weitere Nutzer geöffnet wird (Kapitel 16, 21).

**Embedding.** Numerische Vektordarstellung von Text für semantische Ähnlichkeitssuche. Empfohlen: BGE-M3 (MIT-Lizenz, 100+ Sprachen, 8.192 Token, selbst hostbar) als Ergänzung zu BM25 im Hybrid-Retrieval des Matchers (Kapitel 9).

**Entgeltatlas-API.** Inoffizielle API der Bundesagentur für Arbeit für Median-Gehaltsdaten nach KldB-Code, Region, Alter und Branche ([GitHub](https://github.com/bundesAPI/entgeltatlas-api)). Da nur rund 12,5 Prozent der deutschen Stellenanzeigen ein Gehalt nennen ([Indeed Hiring Lab](https://www.hiringlab.org/de/blog/2025/03/05/gehaltsangaben-bleiben-in-deutschland-die-ausnahme/)), liefert sie dem Matcher den Schätz-Fallback – immer als Schätzung mit Bandbreite gekennzeichnet, nie als Fakt (Kapitel 9).

**Entwurfsmodus.** MVP-Betriebsart des Boten: Er legt die versandfertige Mail per IMAP-APPEND im Ordner „Entwürfe“ ab, den Klick auf Senden machst du selbst – ein zusätzliches Gate ohne eigene Versandlogik (Kapitel 7, 15).

**ESCO.** Kostenlose, mehrsprachige EU-Taxonomie für Skills, Kompetenzen, Qualifikationen und Berufe; primäre Normalisierungsgrundlage des Matchers für den Skill-/Berufsabgleich, ergänzt um ein Mapping zur deutschen KldB 2010 (Kapitel 9).

**Fable 5.1 (Claude Fable 5.1).** Modell `claude-fable-5-1`, 10/50 $ pro 1 Mio. Token Input/Output, dauerhaft aktives Thinking, Covered Model mit 30-Tage-Speicherung. Wunschmodell für Rechercheur und Autor, wo Formulierungsqualität zählt (Kapitel 7, 10, 11).

**Format-Router.** Entscheidungslogik (Kapitel 11.3), die vor jedem Autor-Lauf pro Stelle festlegt, ob überhaupt ein Anschreiben entsteht und in welcher Länge und Sprache – abhängig von Anzeige, Portal-Feldern und Rechercheur-Signalen.

**Freigabe-Gate.** Zustandsmaschine aus vier Bedingungen (Freigabestatus, Hash-Gleichheit der finalen Dokumente, abgelaufenes Undo-Fenster, letzter Render nach der Freigabe-Entscheidung), die der Bote vor jedem Versand im Code prüft – kein Schritt mit Außenwirkung ohne diese Prüfung (Kapitel 7, 14).

**Ghost Job.** Stellenanzeige, die ein Unternehmen nie ernsthaft besetzen will (Pipeline-Aufbau, Schaufenster, interne Nachfolgeplanung). Für Deutschland gibt es keine belastbare Quote, nur US-Zahlen (18–22 Prozent bei Greenhouse); der Matcher wertet Alter, Wiederholungspostings und fehlende Ansprechperson deshalb als weiches Signal, nie als Hard-Filter (Kapitel 9).

**Ghosting.** Ausbleibende Rückmeldung nach einer Bewerbung – verwandt, aber nicht identisch mit Ghost Job. Nach mehreren 2025/2026-Befragungen (Stepstone, Indeed/Appinio) bleibt in Deutschland die Mehrheit der Bewerbungen ohne Rückmeldung; der Tracker markiert solche Fälle für das Nachfassen im Nachlauf (Kapitel 15).

**Haiku 4.5 (Claude Haiku 4.5).** Modell `claude-haiku-4-5`, 1/5 $ pro 1 Mio. Token, kein Covered Model. Günstigstes Modell im Stack, eingesetzt für Massenarbeit: Extraktion, Klassifikation, Keyword-Listen (Kapitel 7, 9, 12).

**Handelsregister-Scraper.** Inoffizielles Werkzeug (`bundesAPI/handelsregister`) für Firmendaten, selbst begrenzt auf 60 Abfragen pro Stunde mit ausdrücklicher Warnung vor §§ 303a/303b StGB bei Missbrauch ([GitHub](https://github.com/bundesAPI/handelsregister)). Der Rechercheur nutzt es nur im Zweifelsfall, mit hart durchgesetztem Rate-Limit (Kapitel 10).

**Haushaltsausnahme.** DSGVO-Ausnahme (Art. 2 Abs. 2 lit. c) für Verarbeitung zu rein persönlichen/familiären Zwecken. Solange der Agent nur für dich arbeitet, greift sie voraussichtlich; sie entfällt, sobald das System kommerzialisiert oder für weitere Nutzer geöffnet wird (Kapitel 16, 21).

**Hook.** Agent-SDK-Ereignis (u. a. `PreToolUse`, `PostToolUse`, `Stop`), das eigenen Code vor oder nach einem Werkzeugaufruf ausführt. Ein `PreToolUse`-Hook auf dem Versand-Werkzeug ist die technische Durchsetzung des Freigabe-Gates, nicht nur eine Anweisung im Prompt (Kapitel 7).

**Jobscamming.** Betrügerische Stellenanzeige mit dem Ziel Identitätsdiebstahl; typische Signale sind Kontaktaufnahme nur über WhatsApp/Telegram oder die Forderung nach Video-Ident bzw. Kontoeröffnung vor Vertragsschluss ([Verbraucherzentrale](https://www.verbraucherzentrale.de/jobscamming-was-tun-wenn-das-traumangebot-zur-falle-wird-110906)). Der Matcher schließt solche Anzeigen hart aus, unabhängig vom sonstigen Score (Kapitel 9).

**Judge.** LLM-als-Bewertungsschritt im Matcher: bewertet jede Stelle gegen das Kandidatenprofil mit Zahlenankern statt Adjektiven, mit Claude Sonnet 5 im Batch. Grundprinzip auch der Kritiker-Rubrik: Das bewertende Modell ist nie dasselbe wie das schreibende (Kapitel 9, 11).

**Kandidatenprofil.** Komponente (Kapitel 8): Master-Lebenslauf, Story-Bank, Stimmprofil, Präferenzen und Standardantworten – die strukturierte Datenbasis, auf die sich jede andere Komponente stützt, ohne sie zu verändern.

**Kennziffer.** Referenz-/Ausschreibungsnummer einer Stelle. Muss identisch in Betreffzeile, Anschreiben und Lebenslauf erscheinen; der Setzer prüft diese Konsistenz automatisch, bevor ein Dokument als final gilt (Kapitel 13, 14).

**KldB 2010.** Fünfstellige Berufsklassifikation der Bundesagentur für Arbeit, ohne eigenes REST-API, aber Schlüssel der Entgeltatlas-API. Der Matcher pflegt ein Mapping von ESCO zu KldB, um Gehaltsschätzungen anzubinden (Kapitel 9).

**Knockout-Frage.** Regelbasierte Muss-Frage im Bewerbungsformular (z. B. Arbeitserlaubnis, Gehaltsvorstellung), die vor jeder inhaltlichen Bewertung über Zulassung entscheidet. Bei Personio läuft die Vorselektion 2026 ausschließlich darüber, nicht über semantisches Ranking ([Personio Community](https://community.personio.de/recruiting-2/ai-im-personio-recruiting-bereich-13294)); der Bote muss sie korrekt beantworten, bevor ATS-Prüfer-Optimierung überhaupt zählt (Kapitel 4, 12, 15).

**Konfidenz.** Wert zwischen 0 und 1, den der Rechercheur jedem Feld seines Dossiers mitgibt. Unterschreitet ein Pflichtfeld die Schwelle, entsteht eine Rückfrage statt einer Annahme – Grundprinzip der Rechercheur-Regel „er rät nie“ (Kapitel 10).

**Kritiker.** Komponente (Kapitel 11), die Entwürfe mit frischem Kontext gegen Rubrik, Story-Bank, Stellenanzeige und Stimmprofil prüft und Überarbeitungen einfordert, höchstens zwei Schleifen, bevor der Fall an dich geht.

**Managed Agents (Anthropic Managed Agents).** Gehostete Beta-REST-API mit Scheduled Deployments (echte Cron-Ausdrücke), Vaults (Zugangsdaten ohne Modellzugriff im Klartext), Memory Stores (Kontext über Sessions hinweg) und Budgets (0,08 $ pro aktiver Session-Stunde) ([Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview)). Funktional reicher als das Agent SDK, aber Beta – im Masterplan für v1 vorgesehen, nicht für den MVP (Kapitel 7).

**Matcher.** Komponente (Kapitel 9), die Passung und Attraktivität jeder Stelle bewertet, die Bewertung erklärt und die Tages-Top-Liste mit Diversitäts-Kappung auswählt.

**MCP (Model Context Protocol).** Offenes Protokoll, über das Claude externe Werkzeuge und Datenquellen als Server einbindet (`.mcp.json`, stdio-/HTTP-Transport). Im Masterplan u. a. für Playwright MCP (Browser-Formulare, Kapitel 15) und einen lesenden Handelsregister-MCP-Server (Kapitel 10) genutzt.

**MinHashLSH.** Algorithmus zur Near-Duplicate-Erkennung (Bibliothek `datasketch`, MIT-Lizenz). Der Scout setzt ihn nach einem Blocking-Schritt (Firma + Titel + Ort) ein, weil Duplikate über Jobbörsen hinweg laut Textkernel-Forschung bis zu 50 bis 80 Prozent ausmachen können ([ACM](https://dl.acm.org/doi/fullHtml/10.1145/3486622.3493928); Kapitel 9).

**Nachlauf.** Geplanter Lauf zur Antwortverarbeitung und zum Nachfassen, getrennt vom Tageslauf. Der Tracker wertet hier eingegangene Rückmeldungen aus und stößt Erinnerungen an (Kapitel 2, 15).

**Opus 5 (Claude Opus 5).** Modell `claude-opus-5`, 5/25 $ pro 1 Mio. Token, kein Covered Model, damit ZDR-fähig. Eingesetzt als Kritiker (Rubrik, Stimm-Check, Leser-Test) – als vom Autor getrenntes Grader-Modell, kein separater Zweitgutachter – und über den Konfigurationsschalter MODEL_TOP als ZDR-fähige Alternative zu Fable 5.1 für Verarbeitungsschritte, bei denen 30-Tage-Speicherung vermieden werden soll (Kapitel 7, 11, 16).

**Orchestrator.** Komponente, die Zeitplan, Reihenfolge, Budget und Fehlerbehandlung des Tageslaufs steuert und jeden Schritt im `event_log` protokolliert (Kapitel 7).

**Permission-Mode.** Agent-SDK-Einstellung, die steuert, wie Werkzeugaufrufe genehmigt werden (`default`, `acceptEdits`, `plan`, `bypassPermissions`, `dontAsk`, `auto`). Der Versand-Schritt läuft nie in `bypassPermissions`, sondern über ein eigenes `canUseTool`-Gate (Kapitel 7).

**Personalvermittler.** Firma, die im eigenen Namen für einen Auftraggeber inseriert (Formulierungen wie „für unseren Kunden“, „unser Mandant“) – zu unterscheiden von Zeitarbeit/Arbeitnehmerüberlassung, für die das AÜG gilt. Der Scout markiert Verdachtsfälle nur als Hinweis; der Matcher entscheidet, im Zweifel per Rückfrage (Kapitel 9).

**Playwright MCP.** Von Microsoft gepflegter MCP-Server (Apache-2.0) für Browser-Automatisierung über Accessibility-Snapshots statt Screenshots, mit Domain-Allowlisting. Standardwerkzeug des Boten für ATS-Formulare ohne offizielle API (Kapitel 15, 17).

**Prompt Caching.** Anthropic-Funktion, die einen stabilen Prompt-Präfix zwischenspeichert; Cache-Treffer kosten nur das 0,1-Fache des Basispreises, bei Fable 5.1 sogar das 0,025-Fache. Kandidatenprofil, Story-Bank und System-Prompt werden als Präfix vor die täglich wechselnden Stellenanzeigen gecacht (Kapitel 7, 18).

**Prompt Injection.** Versteckte Anweisung in fremdem Inhalt (Stellenanzeige, Webseite), die ein Modell zu ungewollten Aktionen bewegen soll. Wird im System ausschließlich als Bedrohung behandelt, nie als eigene Taktik eingesetzt: Versteckter Weisstext im eigenen Lebenslauf ist im Masterplan explizit ausgeschlossen (Kapitel 4, 7, 16).

**Rechercheur.** Komponente (Kapitel 10), die Unternehmen, Anschrift, Ansprechpartner, Kultur und ATS-Typ recherchiert, jedes Feld mit Konfidenz versieht und bei Unsicherheit fragt statt zu raten – ohne jedes Werkzeug mit Außenwirkung.

**Reranking.** Zweite Retrieval-Stufe, die eine BM25-/Embedding-Vorauswahl neu sortiert (z. B. Cohere Rerank 3.5, Voyage rerank-2.5), bevor der Judge die finale Bewertung vornimmt (Kapitel 9).

**Review-Cockpit.** Komponente (Kapitel 14): die Oberfläche für Prüfung, Checklisten, Diff-Ansicht, Freigabe und Rückfragen – mit vier Aktionen pro Stelle (Freigeben, Ändern mit Kommentar, Ablehnen mit Grund, Rückfrage beantworten), nie einer binären Ja/Nein-Entscheidung.

**Risikoampel.** Bewertung jeder Datenquelle nach ToS-/Sperr-Risiko in Grün (bewusst öffentliche Schnittstelle), Gelb (Grauzone) und Rot (Scraping-Verbot in den AGB); steuert, welche Quellen der Scout autonom im Tageslauf abfragen darf (Kapitel 6).

**Routine (Claude Code Routine).** Geplanter Cloud-Lauf über `claude.ai/code/routines`, Mindestintervall eine Stunde, jede Ausführung mit frischem Klon ohne Permission-Prompts. Wegen fehlender Rückfrage ungeeignet für Schritte mit Außenwirkung wie den Versand; im Masterplan nur optional für den reinen Scan-Teil erwogen (Kapitel 7).

**Rubrik.** Feste Bewertungsskala des Kritikers: sieben Kriterien, Skala 1 bis 5 mit Zahlenankern statt Adjektiven, zwei davon mit hartem Gate. Sorgt dafür, dass zwei Kritiker-Läufe dasselbe messen (Kapitel 11).

**Scout.** Komponente (Kapitel 9), die Stellenanzeigen aus den Quellen aus Kapitel 6 findet, normalisiert und dedupliziert – mit kanonischen IDs, damit spätere Schritte nicht erneut raten müssen.

**Setzer.** Komponente (Kapitel 13), die aus Vorlagen PDF/DOCX erzeugt (Anschreiben, Lebenslauf, Bewerbungsmappe) und vor der Freigabe eine automatisierte Konsistenz- und Formatprüfung durchführt.

**Skill (Agent Skill, `SKILL.md`).** Wiederverwendbare Workflow-Anleitung mit YAML-Frontmatter (`name`, `description`, `allowed-tools`, `model`, `effort`), die Claude automatisch oder manuell lädt. Der Setzer nutzt die Anthropic-Skills `docx`/`pdf` über das Code-Execution-Tool für die Dokumenterzeugung (Kapitel 13, 17).

**Sonnet 5 (Claude Sonnet 5).** Modell `claude-sonnet-5`, 2/10 $ pro 1 Mio. Token, kein Covered Model. Arbeitspferd des Systems für Scan, Bewertung, Extraktion und den Claims-Abgleich (Kapitel 7, 9, 11).

**Status-Pipeline.** Feste Statusfolge jeder Stelle: entdeckt → dedupliziert → bewertet → ausgewählt → recherchiert → geschrieben → geprüft → bereit zur Freigabe → freigegeben → gesendet → Rückmeldung → Interview → Absage/Zusage/archiviert, ergänzt um „Rückfrage offen“ (Kapitel 1.2 und 7.4). Der Status „bereit zur Freigabe“ wird erst gesetzt, wenn Setzer-QA und ATS-Prüfer-Stufe 2 bestanden sind (Kapitel 12, 13). Durchzieht alle Modulkapitel als gemeinsame Sprache.

**Stimmprofil.** Teil des Kandidatenprofils (Kapitel 8): dein Schreibstil, deine Wortwahl, deine Tabus. Autor und Kritiker prüfen jeden Entwurf gegen das Stimmprofil, damit ein Anschreiben nicht generisch, sondern nach dir klingt.

**Story-Bank.** Teil des Kandidatenprofils (Kapitel 8): belegte Erfolge mit Zahlen und Beispielen, stabile IDs (`S01`, `S02`, …), Belegstatus `dokumentiert`/`dritte_bestaetigung`/`erinnerung`. Jeder Claim im Anschreiben referenziert eine Story-Bank-ID.

**Structured Outputs.** Claude-API-/SDK-Funktion, die ein JSON-Schema erzwingt (`output_config.format` bzw. `output_format`); Schemas dürfen kein Regex-`pattern`, keine Längen-/Zahlengrenzen und keine rekursiven Strukturen enthalten. Jede Zwischenstufe der Pipeline – Extraktion, Judge, Dossier, Anschreiben mit Claims, Kritik – nutzt ein eigenes Schema (Kapitel 7, 9).

**Subagent.** Isolierter Agent mit eigenem Kontext, eigenen Werkzeugen und eigenem Modell, definiert über `.claude/agents/*.md` oder programmatisch als `AgentDefinition`. Rechercheur, Autor und Kritiker sind im Masterplan als Subagents umgesetzt, jeweils mit minimaler Tool-Berechtigung (Kapitel 7).

**Tageslauf.** Der geplante tägliche Durchlauf (z. B. ab 06:00 Uhr), der Scout, Matcher, Rechercheur, Autor, Kritiker, ATS-Prüfer und Setzer in Folge anstößt, bis die Top-10-Liste zur Freigabe im Review-Cockpit liegt (Kapitel 2, 7).

**Tailoring-Log.** Protokoll jeder Lebenslauf-Anpassung des Autors (Umordnen, Betonen, Umformulieren) mit erlaubter Operation und Grenze – Beleg dafür, dass nichts erfunden wurde (Kapitel 11).

**Textkernel.** Nach der Sovren-Übernahme (2021/2023 fusioniert) führender CV-Parsing-Dienstleister im deutschen ATS-Markt, Sub-Parser hinter Personio, softgarden und d.vinci ([Textkernel](https://www.textkernel.com/sovren/)). Layoutregeln des Setzers (einspaltig, keine Tabellen/Kopfzeilen) zielen direkt auf diese Parser-Familie (Kapitel 4, 12, 13).

**Tracker.** Komponente (Kapitel 15): führt die Status-Pipeline nach dem Versand fort, wertet Rückmeldungen aus, stößt Nachfassen im Nachlauf an und liefert die Statistik.

**Typst.** Modernes, Apache-2.0-lizenziertes Satzsystem (`typst compile`) als schnellere Alternative zu LaTeX. Im Ökosystem existieren viele Lebenslauf-Vorlagen, aber keine fertige DIN-5008-Anschreiben-Vorlage – der Masterplan setzt für Anschreiben deshalb primär auf WeasyPrint (Kapitel 13).

**Undo-Fenster.** Zeitspanne nach dem Klick auf „Freigeben“ (Default 60 Sekunden), in der die Freigabe im Review-Cockpit oder per Telegram zurückgenommen werden kann, bevor der Bote die Stelle zur Abholung sieht (Kapitel 14).

**Vault (Managed-Agents-Vault).** Getrennte Zugangsdaten-Verwaltung (z. B. `mcp_oauth`, `static_bearer`) für Drittanbieter-Credentials wie SMTP/Gmail, bei der das Modell den Klartext-Wert nie sieht. Für v1 vorgesehen, sobald der Wechsel zu Managed Agents erfolgt (Kapitel 7).

**WeasyPrint.** Python-Bibliothek für HTML/CSS-zu-PDF-Konvertierung mit Unterstützung für CSS Paged Media; primärer Renderer des Setzers für DIN-5008-konforme Anschreiben. Die eigenen PDF/A- und PDF/UA-Varianten gelten laut Hersteller als experimentell – für die reine ATS-Textextraktion ohnehin nicht entscheidend (Kapitel 13).

**Web-Fetch-Tool.** Anthropic-Server-Werkzeug, das nur URLs abrufen kann, die zuvor im Kontext aufgetaucht sind (Exfiltrationsschutz), ohne JS-Rendering. Kostet nur die Token des abgerufenen Inhalts, keine separate Gebühr; der Rechercheur nutzt es für statische Firmenseiten (Kapitel 10).

**Web-Search-Tool.** Anthropic-Server-Werkzeug für Websuche, 10 $ pro 1.000 Suchen zuzüglich Tokenkosten, mit `allowed_domains`/`blocked_domains` einschränkbar ([Web search tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)). Haupt-Rechercheinstrument für Unternehmens- und Ansprechpartnersuche (Kapitel 10).

**XML-Feed.** Sammelbegriff für strukturierte Stellen-Feeds (RSS, Atom, JSON-LD/`JobPosting`) auf ATS-Karriereseiten. Der Scout pflegt eine Watchlist solcher Feeds pro Wunscharbeitgeber als rechtlich unbedenkliche Ergänzung zur BA-Jobsuche-API (Kapitel 6).

**ZDR (Zero Data Retention).** Anthropic-Option, bei der Ein- und Ausgaben nicht gespeichert werden; für Covered Models wie Fable 5.1 nur mit gesonderter Freigabe verfügbar. Für die sensibelsten Schritte (vollständiger Lebenslauf, Gehaltsangaben) ist Opus 5 als ZDR-fähiges Modell die Alternative (Kapitel 16).

**Quellen dieses Kapitels:**
- DGFP: Recruiting-Strukturen 2025 — https://www.dgfp.de/aktuell/recruiting-strukturen-2025-recruiting-wird-strukturierter-datengetriebener-und-technologischer
- PyPI: claude-agent-sdk — https://pypi.org/project/claude-agent-sdk/
- Gibson Dunn: EU AI Act Omnibus Agreement — https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/
- ai-act-law.eu: Artikel 2 KI-VO — https://ai-act-law.eu/de/artikel/2/
- bundesAPI/jobsuche-api — https://github.com/bundesAPI/jobsuche-api
- Anthropic: Batch processing — https://platform.claude.com/docs/en/build-with-claude/batch-processing
- Anthropic: Code execution tool — https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool
- Anthropic: API and data retention — https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
- bundesAPI/entgeltatlas-api — https://github.com/bundesAPI/entgeltatlas-api
- Indeed Hiring Lab: Gehaltsangaben bleiben in Deutschland die Ausnahme — https://www.hiringlab.org/de/blog/2025/03/05/gehaltsangaben-bleiben-in-deutschland-die-ausnahme/
- Verbraucherzentrale: Jobscamming — https://www.verbraucherzentrale.de/jobscamming-was-tun-wenn-das-traumangebot-zur-falle-wird-110906
- Personio Community: AI im Personio-Recruiting-Bereich — https://community.personio.de/recruiting-2/ai-im-personio-recruiting-bereich-13294
- bundesAPI/handelsregister — https://github.com/bundesAPI/handelsregister
- ACM: Duplicate Job Postings (Textkernel-Forschung) — https://dl.acm.org/doi/fullHtml/10.1145/3486622.3493928
- Anthropic: Managed Agents Overview — https://platform.claude.com/docs/en/managed-agents/overview
- Textkernel: Sovren — https://www.textkernel.com/sovren/
- Anthropic: Web search tool — https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool
- § 11 AÜG (gesetze-im-internet.de) — https://www.gesetze-im-internet.de/a_g/__11.html
