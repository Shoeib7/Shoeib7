## 17. Werkzeugkasten: Skills, Subagents, MCP-Server, CLIs, Bibliotheken, APIs, Dienste

Dieses Kapitel ist das vollständige Inventar aller Werkzeuge, die in den Kapiteln 6 bis 16 entschieden, erwogen oder verworfen wurden. Es trifft keine neuen Architekturentscheidungen; es sammelt die Verdikte der Modulkapitel an einem Ort, damit Claude Code beim Bauen nicht raten muss, was installiert, konfiguriert oder bewusst weggelassen wird. Jede Zeile nennt Zweck bei uns, Zugriff oder Installation, Kosten mit Quelle und eine Bewertung. Wo zwei Kapitel dasselbe Werkzeug unterschiedlich zuordnen, steht hier die Entscheidung mit Verweis; die Begründungen bleiben in den Modulkapiteln.

### 17.1 Lesehilfe und Regeln für das Inventar

**Bewertungsskala.** *Kern (MVP)*: ohne dieses Werkzeug läuft der Durchstich aus Kapitel 19.4 nicht. *Kern (v1)* und *Kern (v2)*: fest eingeplant, aber erst in der genannten Phase. *Optional*: nur bei belegtem Bedarf, mit Bedingung in der Zeile. *Vermeiden*: aus Rechts-, Kosten-, Lizenz- oder Wartungsgründen ausgeschlossen; die Zeile bleibt stehen, damit niemand die Prüfung wiederholt.

**Preise.** Nur mit Quelle aus der Recherche, Stand September 2026. „Unbestätigt“ heißt, dass der Faktenprüfer die Zahl nicht live prüfen konnte; sie ist dann Planungsannahme, kein Fakt. Werkzeuge, deren Preise die Recherche nicht belegen konnte, tragen keinen Preis.

**Lizenz- und Wartungsregeln** (verbindlich für Kapitel 19 bis 21):

1. Bevorzugt MIT, Apache-2.0, BSD, MPL. AGPL-Projekte (Skyvern, OpenResume, ApplyPilot) werden nur betrieben oder als Referenz gelesen, nie in den eigenen Code kopiert; das gilt für eine spätere Kommerzialisierung (Kapitel 21).
2. Die Anthropic-Dokumentskills docx/pdf/pptx/xlsx sind „source-available“, nicht Open Source; vor jeder kommerziellen Nutzung ist die LICENSE.txt je Skill zu prüfen ([anthropics/skills](https://github.com/anthropics/skills)).
3. Archivierte oder seit mehr als zwölf Monaten unveröffentlichte Projekte kommen nicht in den Produktivbetrieb (Beispiele: Gmail-MCP von GongRzhe, archiviert 3.3.2026, [Repo](https://github.com/GongRzhe/Gmail-MCP-Server); Browserbase-MCP-Repo, archiviert 20.7.2026, [Repo](https://github.com/browserbase/mcp-server-browserbase); python-jobspy, letzte Version 1.1.82 vom 28.7.2025, [PyPI](https://pypi.org/project/python-jobspy/)).
4. Halbjährliche Abhängigkeitsprüfung im Wartungslauf (Kapitel 7.5, 20.5): Archiv-Status, letzte Version, Lizenzänderung, Preisänderung.
5. Versionen werden in `pyproject.toml` und `.mcp.json` gepinnt. Stand der Recherche:

| Werkzeug | Gepinnte Version | Quelle |
|---|---|---|
| claude-agent-sdk (Python) | 0.2.152 vom 2.9.2026, Python 3.10+ | [PyPI](https://pypi.org/project/claude-agent-sdk/) |
| Apache Tika | 4.0.0 vom 18.8.2026 (3.3.0 ist überholt) | [CHANGES.txt](https://raw.githubusercontent.com/apache/tika/main/CHANGES.txt) |
| @playwright/mcp | 0.0.80 (mit `browser_file_upload`) | [README](https://raw.githubusercontent.com/microsoft/playwright-mcp/main/README.md) |
| datasketch | 2.0.0 vom Juli 2026, Python 3.9+ | [GitHub](https://github.com/ekzhu/datasketch) |
| sentence-transformers | 6.0.1 vom 31.8.2026 | [GitHub](https://github.com/UKPLab/sentence-transformers) |
| LanceDB | 0.38.0 vom 31.8.2026 | [GitHub](https://github.com/lancedb/lancedb) |
| spaCy de_core_news_lg | 3.8.0 | [Release](https://github.com/explosion/spacy-models/releases/tag/de_core_news_lg-3.8.0) |
| Crawl4AI | 0.9.3 vom 31.8.2026 | [PyPI](https://pypi.org/project/crawl4ai/) |
| pgvector | 0.8.6 | [GitHub](https://github.com/pgvector/pgvector) |

### 17.2 (A) Claude-Bausteine

#### 17.2.1 Modelle und ihr Einsatz

Preise laut Preisseite: Fable 5.1 $10/$50, Opus 5 $5/$25, Sonnet 5 $2/$10 (seit 1.9.2026 dauerhaft), Haiku 4.5 $1/$5 je 1 Mio. Token Input/Output; Cache-Lesen 0,1x des Eingabepreises, bei Fable 5.1 0,025x; Cache-Schreiben 1,25x (5 Minuten) oder 2x (1 Stunde); Batch 50 Prozent Rabatt; Modelle ab Claude 4.7 (also Opus 5, Sonnet 5, Fable 5.x) erzeugen mit dem neuen Tokenizer rund 30 Prozent mehr Tokens für denselben Text ([Pricing](https://platform.claude.com/docs/en/about-claude/pricing), [Prompt Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching), [Batch](https://platform.claude.com/docs/en/build-with-claude/batch-processing)). Fable 5.1, Fable 5, Mythos 5 und Mythos 5.1 sind „Covered Models“ mit Pflicht zur 30-Tage-Speicherung; Opus 5, Sonnet 5 und Haiku 4.5 sind es nicht ([Data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)).

| Modell (ID) | Preis In/Out, Cache-Lesen | Einsatz bei uns | Hinweise |
|---|---|---|---|
| Claude Fable 5.1 (`claude-fable-5-1`) | $10/$50; 0,025x | Rechercheur (effort high); Autor: Entwürfe high, Überarbeitung medium, Lebenslauf-Tailoring medium; Matcher-Tagesauswahl mit Begründung (medium) | Thinking immer aktiv, Tiefe nur über `effort`; 30-Tage-Speicherung muss im Konto aktiviert sein, sonst Fehler 400; `stop_reason: refusal` möglich, Fallback Opus 5 (Kapitel 11.10) |
| Claude Opus 5 (`claude-opus-5`) | $5/$25; 0,1x | Kritiker: Rubrik (high), Stimm-Check und Leser-Test (medium); Refusal-Fallback; Umschalter `MODEL_TOP=claude-opus-5` für alle Fable-Rollen | ZDR-fähig; Alternative als Autor, falls keine 30-Tage-Speicherung gewünscht (Kapitel 16.2) |
| Claude Sonnet 5 (`claude-sonnet-5`) | $2/$10; 0,1x | Matcher-Judge im Batch (medium); Autor-Briefing (medium); Fakten-Check (medium, Structured Output); ATS-Lesetest; Eskalation bei Extraktions- und Klassifikationsfehlern (low) | Arbeitspferd für Masse mit Erklärbarkeit |
| Claude Haiku 4.5 (`claude-haiku-4-5`) | $1/$5; 0,1x | Extraktion normalisierter Anzeigen; Dedup-Zweifelsfälle; Injection-Screen; Keyword-Extraktion im ATS-Prüfer; Konsistenz-Check (low); Tracker-Antwortklassifikation (Standard); optionale Sichtprüfung der PDF-Vorschau | kein `effort`-Parameter (Kapitel 7.6); Cache-Mindestlänge 4.096 Token |

Zuordnung je Rolle, konsolidiert aus den Modulkapiteln:

| Komponente, Schritt | Modell, effort | Aufrufweg | Kapitel |
|---|---|---|---|
| Scout: Extraktion, Dedup-Zweifel, Injection-Screen | Haiku 4.5 (Eskalation Sonnet 5) | Messages API, Batch | 7.6, 9.3, 7.8 |
| Matcher: Judge | Sonnet 5, medium | Messages Batch, gecachter Präfix | 7.6, 9.5 |
| Matcher: Tagesauswahl-Begründung | Fable 5.1, medium | Messages API | 7.6, 9.8 |
| Rechercheur | Fable 5.1, high | Agent SDK Subagent | 7.6, 10 |
| Autor: Briefing | Sonnet 5, medium | Messages API | 11.8 |
| Autor: Entwürfe / Überarbeitung / Tailoring | Fable 5.1, high / medium / medium | Agent SDK Subagent | 11.8, 11.10 |
| Kritiker: Rubrik; Stimm-Check und Leser-Test | Opus 5, high; Opus 5, medium | Agent SDK Subagent, frischer Kontext | 11.8, 11.10 |
| Kritiker: Fakten-Check; Konsistenz-Check | Sonnet 5, medium; Haiku 4.5, low | Messages API, Structured Output | 11.8 |
| ATS-Prüfer: Begriffsextraktion; Lesetest | Haiku 4.5; Sonnet 5 | Messages API | 12.9 |
| Tracker: Antwortklassifikation | Haiku 4.5, Eskalation Sonnet 5 unter Konfidenz 0,7 | Messages API | 15.6 |
| Setzer, Bote, Cockpit, Orchestrator | kein Modell | Code | 13, 14, 15 |

**Entscheidung** zu zwei Widersprüchen zwischen Kapiteln: Kapitel 7.6 nennt für den Kritiker Fable 5.1 mit Opus 5 als optionalem Zweitgutachter und für die Tracker-Klassifikation Sonnet 5; die Modulkapitel 11 und 15 legen Opus 5 als Kritiker und Haiku 4.5 mit Eskalation für den Tracker fest, und Kapitel 12, 19 und 20 folgen ihnen. Das Inventar folgt den Modulkapiteln. **Begründung:** Kapitel 11 begründet die Wahl mit Anthropics Empfehlung, Grader und Generator zu trennen, und mit der ZDR-Fähigkeit von Opus 5 ([Develop tests](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)); Kapitel 15 begründet Haiku 4.5 mit der Massenlogik des Styleguides. **Alternative:** Kapitel 7.6 anpassen; Kapitel 24 sollte den Stand vereinheitlichen. Alle Zuordnungen liegen in `config/modelle.yaml` (Kapitel 7.9), nicht im Code, und werden nach 20 Bewerbungen kalibriert (Kapitel 19.5, V-02).

#### 17.2.2 Funktionen der Claude-API

| Funktion | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| Messages API mit `anthropic`-Python-SDK | alle zustandslosen, schemagebundenen Aufrufe (Extraktion, Judge, Fakten-Check, Klassifikation) | Commercial-API-Key aus `sops exec-env`, nie Abo-Token (Kapitel 16.8) | Tokenpreise, siehe 17.2.1 | Kern (MVP) |
| Message Batches ([Doku](https://platform.claude.com/docs/en/build-with-claude/batch-processing)) | nächtlicher Judge über 200–500 Anzeigen | asynchron, meist unter 1 h, spätestens 24 h, 29 Tage abrufbar | 50 % Rabatt, stapelbar mit Caching | Kern (MVP) |
| Prompt Caching ([Doku](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)) | System-Prompt, Rubrik, Profil, Stimmprofil als 1-h-Präfix | `cache_control`, max. 4 Breakpoints, Mindestlängen 512/1.024/4.096 Token, 100 % identischer Präfix | Lesen 0,1x bzw. 0,025x | Kern (MVP) |
| Structured Outputs ([Doku](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)) | jedes Zwischenergebnis als JSON-Schema unter `schemas/` | `output_config.format`, kein Beta-Header; kein `pattern`, kein `minLength`/`maxLength`, keine Rekursion, `additionalProperties: false`; Formatprüfung mit pydantic | kein Aufpreis | Kern (MVP) |
| Web Search ([Doku](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)) | Rechercheur: Firma, Karriereseite, News | Server-Tool, `max_uses: 8`, `allowed_domains`/`blocked_domains`; Version 20260209+ mit Dynamic Filtering | $10 je 1.000 Suchen plus Tokens; fehlgeschlagene Suchen kostenlos | Kern (MVP) |
| Web Fetch ([Doku](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool)) | Impressum, Karriereseite, Kununu-Profil, Anzeigenseite | Server-Tool, aktuell `web_fetch_20260318`; nur URLs aus Nutzernachricht oder Werkzeugergebnis, nicht aus System-Prompt oder Modellausgabe; kein JavaScript; `max_content_tokens` | keine Zusatzgebühr, nur Tokens | Kern (MVP) |
| Code Execution ([Doku](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool)) | nicht im Renderpfad (Setzer ist Code); Basis für Anthropic-Dokumentskills bei interaktiver Nutzung | Sandbox mit python-docx, pypdf, pdfplumber, reportlab | kostenlos mit Web Search/Fetch 20260209+; sonst 1.550 Freistunden je Organisation und Monat, danach $0,05 je Stunde, Mindestabrechnung 5 Minuten | optional |
| Agent Skills über `container.skills` ([Skills Guide](https://platform.claude.com/docs/en/build-with-claude/skills-guide)) | docx/pdf per API ohne Claude Code | max. 20 Skills je Request; Ergebnisdateien als `file_id` | Code-Execution-Zeit plus Tokens | optional; Setzer nutzt die Bibliotheken direkt (Kapitel 13.2) |
| Files API und PDF-Support ([Files](https://platform.claude.com/docs/en/build-with-claude/files), [PDF](https://platform.claude.com/docs/en/build-with-claude/pdf-support)) | Einlesen bestehender Lebenslauf-PDFs beim Onboarding; Rückgabe von Skill-Dateien | `file_id` ist workspace-weit sichtbar, nicht nutzer- oder sitzungsgebunden; PDFs max. 32 MB, 600 Seiten | im Tokenpreis | optional; bei Mehrnutzerbetrieb ein Workspace je Mandant (Kapitel 19.6, W-11) |
| Usage and Cost API, Console ([Cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)) | wöchentlicher Abgleich mit `event_log`; Ausgabenlimit im Konto | Console, API | – | Kern (MVP) |
| Effort ([Doku](https://platform.claude.com/docs/en/build-with-claude/effort)) | Denktiefe je Rolle (17.2.1) | `output_config.effort` bzw. `effort` im SDK | steuert Tokenverbrauch | Kern (MVP) |
| Rate Limits ([Doku](https://platform.claude.com/docs/en/api/rate-limits)) | keine Maßnahme nötig | Tiers Start/Build/Scale; Spend-Cap Start $500/Monat (mittlere Konfidenz) | – | Hinweis: bei 10 Bewerbungen am Tag kein Engpass |
| Usage Policy ([AUP](https://www.anthropic.com/aup)) | Beschäftigungs-Screening ist Hochrisiko mit HITL-Pflicht; Spam-Verbot | – | – | Freigabe-Gate erfüllt HITL (Kapitel 16.7) |

#### 17.2.3 Claude Code: Skills, Subagents, Hooks, Einstellungen

Claude Code ist zugleich Bauwerkzeug (schreibt und testet den Code) und Laufzeitbaustein (Skills, Subagents und Hooks werden vom Agent SDK aus denselben Dateien geladen). Skills liegen als `SKILL.md` mit Frontmatter im Projekt-Scope `.claude/skills/` und werden über die `description` automatisch oder per `/name` manuell ausgelöst; Frontmatter-Felder umfassen `name`, `description`, `allowed-tools`, `disable-model-invocation`, `model`, `effort`, `context: fork` ([Skills](https://code.claude.com/docs/en/skills), [Agent SDK: Skills](https://code.claude.com/docs/en/agent-sdk/skills)). Subagents liegen in `.claude/agents/*.md` mit `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `effort` und starten mit frischem, isoliertem Kontext ([Subagents](https://code.claude.com/docs/en/sub-agents)). Hooks kennen über 30 Ereignisse und fünf Typen (command, http, mcp_tool, prompt, agent) und sind, anders als CLAUDE.md, erzwungene Kontrolle ([Hooks](https://code.claude.com/docs/en/hooks), [Memory](https://code.claude.com/docs/en/memory)).

**Skills (Projekt-Scope, versioniert):**

| Skill (`.claude/skills/…/SKILL.md`) | Zweck | Trigger | Modell, allowed-tools |
|---|---|---|---|
| `anschreiben` (in Kapitel 11.10 als `anschreiben-schreiben` referenziert; ein Name, hier `anschreiben`) | Struktur-Skelette A/B/C, Schreibregeln in Kurzform, Anti-Generik-Hinweise für den Autor | automatisch im Autor-Subagent, wenn Status „recherchiert“ und Briefing vorliegt | Fable 5.1; Read, Write nur im Stellenordner |
| `lebenslauf-tailoring` | erlaubte Operationen, Tailoring-Log, Synonym-Spiegelung | automatisch im Autor-Subagent | Fable 5.1, medium; Read, Write |
| `rubrik-kritik` | Rubrik K1–K7, Gates, Kritik-Schema, Leser-Test-Fragen | automatisch im Kritiker-Subagent | Opus 5; nur Read |
| `din-5008-check` | Prüfliste Anschriftfeld, Betreff, Anrede-Formen, Anlagenvermerk | manuell `/din-5008-check` bei Vorlagenabnahme; automatisch nicht nötig, weil der Setzer Code ist | Haiku 4.5; Read |
| `rueckfrage-protokoll` | wann fragen statt raten, drei Fragetypen, Defaults | automatisch im Rechercheur-Subagent | Fable 5.1; Read |
| `profil-onboarding` | Fragenkatalog Kapitel 8.8 Block für Block, Entwürfe der fünf Profildateien | nur manuell `/profil-onboarding` (`disable-model-invocation: true`) | interaktive Sitzung; Read, Write nur in `profil/` |
| `stimmprofil-extraktion` | Textproben lesen, Satzlängen-Statistik, Wendungen, Tabu-Vorschläge als Entwurf | nur manuell aus dem Onboarding | interaktiv; Read, Write nur `stimmprofil.md` |
| Anthropic `document-skills` (docx, pdf) ([anthropics/skills](https://github.com/anthropics/skills)) | interaktive Hilfe beim Bauen und Prüfen der Vorlagen; nicht im Renderpfad | `/plugin marketplace add anthropics/skills`, `/plugin install document-skills@anthropic-agent-skills` | source-available, Lizenz vor kommerzieller Nutzung prüfen; optional |
| Anthropic `doc-coauthoring` ([SKILL.md](https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md)) | Muster für Onboarding und Kritiker-Leser-Test (Kontext sammeln, Entwurf, unabhängiger Leser) | als Vorlage gelesen, nicht installiert | Apache-2.0; Referenz |

Skills enthalten Regeln und Beispiele, nie Fakten über den Kandidaten (Kapitel 7.6). Die beiden Onboarding-Skills sind die einzigen, die in `profil/` schreiben dürfen, und nur in einer interaktiven Sitzung (Kapitel 8.9).

**Subagents (`.claude/agents/`, Definitionen in Kapitel 7.6 und 11.10):**

| Subagent | Modell, effort | tools / disallowedTools | Weitere Felder | Ausgabe |
|---|---|---|---|---|
| `rechercheur` | Fable 5.1, high | WebSearch, WebFetch, Read, `mcp__bundesapi__handelsregister_suche`, `mcp__bundesapi__jobdetails` / Write, Edit, Bash | `maxTurns: 25`, `permissionMode: default`, Skill `rueckfrage-protokoll` | `schemas/dossier.json`, `question`-Einträge |
| `autor` | Fable 5.1, high | Read, Write (nur Stellenordner) / Bash, WebSearch, WebFetch | Skills `anschreiben`, `lebenslauf-tailoring` | `entwuerfe.md`, `tailoring_log.json`, Claims |
| `kritiker` | Opus 5, high | Read / Write, Edit, Bash, Netz | frischer Kontext, Skill `rubrik-kritik`, `output_format` JSON | `kritik.json` |

Der eingebaute `general-purpose`-Subagent wird nicht verwendet; jede Rolle hat eine eigene Definition mit minimalen Rechten ([Agent SDK: Subagents](https://code.claude.com/docs/en/agent-sdk/subagents)).

**Hooks und Einstellungen (`.claude/settings.json`, Auszug in Kapitel 7.8):**

| Hook / Regel | Typ | Zweck | Bewertung |
|---|---|---|---|
| `PreToolUse` mit Matcher `.*` | command: `bewerbungsagent hook pre-tool --deny-outside bewerbungen/ --log` | blockiert Pfade außerhalb der Whitelist, protokolliert jeden Werkzeugaufruf in `event_log` | Kern (MVP) |
| `PostToolUse` | command | Tokens, Dauer, Ergebnisgröße je Aufruf nachtragen | optional (v1, mit Phoenix) |
| `SubagentStop` | command | Transkript-Pfad und Kosten des Subagents in `event_log` | optional (v1) |
| `permissions.deny` | Liste | `Bash(curl:*)`, `Bash(wget:*)`, `Bash(ssh:*)`, `mcp__bote__*`, `mcp__playwright__*`, `Write(./profil/**)`, `Edit(./profil/**)`, `Write(./config/**)` | Kern (MVP) |
| Permission-Modus | `default` | nie `bypassPermissions`; im SDK zusätzlich `canUseTool`-Callback als hartes Gate für den Boten | Kern (MVP) |
| `CLAUDE.md` | Kontext | Statusbegriffe, zehn Sicherheitsregeln, Testpflicht; kein Ersatz für Hooks | Kern (MVP) |
| Sandbox ([Security](https://code.claude.com/docs/en/security)) | bubblewrap unter Linux | Netzwerk-Allowlist auf `api.anthropic.com` und Quellen-Domains für SDK-Läufe (mittlere Konfidenz) | Kern (MVP) |
| Headless-Modus ([Headless](https://code.claude.com/docs/en/headless)) | `claude -p "<prompt>" --output-format json --allowedTools …`; `--bare` ohne Hooks/Skills/MCP mit `ANTHROPIC_API_KEY` | Golden-Tests und Trockenläufe einzelner Prompts | Kern (Entwicklung) |
| Plugins ([Plugins](https://code.claude.com/docs/en/plugins), [Marketplace](https://github.com/anthropics/claude-plugins-official)) | `/plugin marketplace add`, `/plugin install` | document-skills; offizielle Marketplace enthält browser-use, firecrawl, exa, github, kein E-Mail-Plugin | optional |

#### 17.2.4 MCP-Server

Model Context Protocol (MCP) ist die Schnittstelle, über die Claude-Modelle Werkzeuge aufrufen. `.mcp.json` unterstützt stdio-, http- und WebSocket-Transporte in drei Scopes (local, project, user) mit `${VAR}`-Expansion; Werkzeugausgaben sind auf 25.000 Token begrenzt (`MAX_MCP_OUTPUT_TOKENS`), und MCP-Tool-Beschreibungen gelten als nicht vertrauenswürdiger Input ([MCP](https://code.claude.com/docs/en/mcp)). Eigene Werkzeuge laufen im Agent SDK als In-Process-Server ohne separaten Prozess (`@tool`, `create_sdk_mcp_server`, Namensschema `mcp__<server>__<tool>`; [Custom Tools](https://code.claude.com/docs/en/agent-sdk/custom-tools)).

**Eigene In-Process-Server (Code im Repo, kein Fremdbetrieb):**

| Server, Tools | Zweck bei uns | Auth | Reife | Kosten | Bewertung |
|---|---|---|---|---|---|
| `bundesapi`: `jobdetails`, `handelsregister_suche`, später `entgeltatlas` | lesende Werkzeuge für den Rechercheur; Wrapper um `deutschland` und bundesAPI-Clients mit hartem Rate-Limiter (Handelsregister max. 60 Abfragen je Stunde) | keine; BA-Header `X-API-Key: jobboerse-jobsuche` im Code | eigener Code | 0 EUR | Kern (MVP) |
| `setzer`: `render_documents` | Setzer als Werkzeug des Orchestrators (Kapitel 13.1) | – | eigener Code | Rechenzeit | Kern (MVP) |
| `bote`: `create_draft` (MVP), `send_email` (v1), `prefill_portal` (v2) | einzige Werkzeuge mit Außenwirkung; für alle Subagents per Deny gesperrt; `canUseTool` öffnet nur bei `status = freigegeben`, Hash-Gleichheit, abgelaufenem Undo-Fenster (Kapitel 7.8) | Zugangsdaten aus Umgebung, nie im Kontext | eigener Code | 0 EUR | Kern (MVP) |

**Externe MCP-Server:**

| Server (Repo) | Zweck bei uns | Auth / Install | Reife | Kosten | Bewertung |
|---|---|---|---|---|---|
| Playwright MCP ([microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp)) | Erkundung und Einzelabruf JS-lastiger Karriereseiten; Entwicklung des Portal-Co-Piloten | `npx @playwright/mcp@latest --headless --allowed-hosts …`; kein Key | aktiv, Apache-2.0, 0.0.80 mit `browser_file_upload`; laut Doku „keine Sicherheitsgrenze“ | 0 EUR | optional (v2); für alle Subagents mit Drittinhalten gesperrt |
| Firecrawl MCP ([firecrawl/firecrawl-mcp-server](https://github.com/firecrawl/firecrawl-mcp-server)) | Markdown-Vorstufe für Karriereseiten ohne Feed | `FIRECRAWL_API_KEY`, `npx -y firecrawl-mcp` oder gehostet | aktiv | Free 1.000 Credits/Monat, Hobby 16 USD/Monat ([scrapegraphai](https://scrapegraphai.com/blog/firecrawl-pricing)) | optional (v1); Adapter ruft die REST-API direkt, kein MCP nötig |
| Exa MCP ([exa-labs/exa-mcp-server](https://github.com/exa-labs/exa-mcp-server)) | semantische Zweitsuche des Rechercheurs | gehostet `https://mcp.exa.ai/mcp`, Key optional | aktiv | Preise in der Recherche nicht belegt | optional (v2) |
| Tavily MCP ([tavily-ai/tavily-mcp](https://github.com/tavily-ai/tavily-mcp)) | RAG-freundliche Zweitsuche | `TAVILY_API_KEY`, `npx -y tavily-mcp@latest` | aktiv | Preise in der Recherche nicht belegt | optional (v2) |
| Brave Search MCP ([brave/brave-search-mcp-server](https://github.com/brave/brave-search-mcp-server)) | günstige Zweitsuche | `BRAVE_API_KEY`, Kreditkarte Pflicht | aktiv | Free-Tier seit Februar 2026 abgeschafft; $5 je 1.000 Anfragen, $5 Gratis-Guthaben je Monat ([implicator](https://www.implicator.ai/brave-drops-free-search-api-tier-puts-all-developers-on-metered-billing/), [agentdeals](https://agentdeals.dev/vendor/brave-search-api)) | optional |
| Adzuna MCP ([folathecoder/adzuna-job-search-mcp](https://github.com/folathecoder/adzuna-job-search-mcp)) | – | Adzuna-Keys | Community | 0 EUR | vermeiden: Scout ruft Adzuna direkt per REST (Kapitel 6.5) |
| Google Workspace MCP ([taylorwilsdon/google_workspace_mcp](https://github.com/taylorwilsdon/google_workspace_mcp)) | Gmail-Entwürfe per MCP | `uvx workspace-mcp`, OAuth 2.1, Google-Cloud-Projekt | aktiv, MIT, 120+ Tools | 0 EUR | optional; nicht nötig, weil der Bote Code ohne Modell ist (Kapitel 7.2) |
| Gmail MCP ([GongRzhe/Gmail-MCP-Server](https://github.com/GongRzhe/Gmail-MCP-Server)) | – | npx | archiviert 3.3.2026, 32 offene PRs | – | vermeiden |
| Google-eigener Gmail-MCP ([Doku](https://developers.google.com/workspace/gmail/api/guides/configure-mcp-server)) | Entwürfe über offiziellen Server | Developer Preview | Preview, Verfügbarkeit unbestätigt | – | optional, erst nach Prüfung |
| Notion MCP ([makenotion/notion-mcp-server](https://github.com/makenotion/notion-mcp-server)) | Kanban-Spiegel des Cockpits | gehostet mit OAuth oder `NOTION_TOKEN` | offiziell, MIT | 0 EUR, Notion-Plan | optional (nach v1, Kapitel 14.2) |
| Slack MCP ([korotovsky/slack-mcp-server](https://github.com/korotovsky/slack-mcp-server)), Telegram MCP ([chigwell/telegram-mcp](https://github.com/chigwell/telegram-mcp)) | – | Token bzw. Telethon-Nutzerkonto | aktiv | 0 EUR | vermeiden: Benachrichtigung ist Bot-Code ohne Modellzugriff (Kapitel 7.7.7) |
| Postgres MCP Pro ([crystaldba/postgres-mcp](https://github.com/crystaldba/postgres-mcp)) | – | `uvx postgres-mcp`, `DATABASE_URI` | aktiv, MIT | 0 EUR | vermeiden bis Postgres im Mehrnutzerbetrieb (Kapitel 7.7.4) |
| Apify Actors MCP ([apify/actors-mcp-server](https://github.com/apify/actors-mcp-server)) | vorgefertigte Scraper | gehostet, `APIFY_TOKEN` | aktiv | je Actor, ca. 0,05–5 USD je 1.000 Ergebnisse | optional (v2), nur nach Freigabe als Gelb-Quelle |
| Browserbase MCP ([browserbase/mcp-server-browserbase](https://github.com/browserbase/mcp-server-browserbase)) | – | gehosteter Endpunkt | Repo archiviert 20.7.2026 | nutzungsabhängig | vermeiden |
| Referenzserver ([modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)): Filesystem, Git, Fetch, Memory | Experimente in der Entwicklung | npx/uvx | laut Repo nicht für Produktion; Postgres/Slack/Brave archiviert | 0 EUR | nur Entwicklung |
| job-apply-plugin ([neonwatty/job-apply-plugin](https://github.com/neonwatty/job-apply-plugin)) | Referenzmuster für HITL-Stopps und lokale Profildaten | Claude-Code-Plugin | Community, ATS-Workflows „unverified“, keine deutschen ATS | 0 EUR | Referenz, nicht integrieren |

Beispiel `.mcp.json` (Projekt-Scope, versioniert; Playwright erst in v2 aktiv):

```json
{
  "mcpServers": {
    "bundesapi": {
      "command": ".venv/bin/python",
      "args": ["-m", "bewerbungsagent.llm.mcp_bundesapi"],
      "env": { "BA_RATE_LIMIT_PER_HOUR": "60" }
    },
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@0.0.80", "--headless",
               "--allowed-hosts", "${PORTAL_HOSTS:-localhost}"]
    }
  }
}
```

#### 17.2.5 Claude Agent SDK und Managed Agents

| Baustein | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| Claude Agent SDK Python `claude-agent-sdk` ([PyPI](https://pypi.org/project/claude-agent-sdk/), [Doku](https://code.claude.com/docs/en/agent-sdk/python)) | Werkzeugschleife für Rechercheur, Autor, Kritiker; `query()` je Lauf, `ClaudeSDKClient` für Mehrschritt; `ClaudeAgentOptions` mit `allowed_tools`, `permission_mode`, `agents`, `hooks`, `output_format`, `max_budget_usd`, `model`, `effort`, `mcp_servers`, `skills` | `pip install claude-agent-sdk`; bündelt die Claude-Code-CLI, kein Node nötig; MIT | 0 EUR, Tokens separat | Kern (MVP) |
| Permission-Auswertung ([Permissions](https://code.claude.com/docs/en/agent-sdk/permissions)) | sechs Stufen: Hooks → Deny → Ask → Modus → Allow → `canUseTool`; Modi default/acceptEdits/plan/bypassPermissions/dontAsk/auto | Konfiguration | – | Kern (MVP); `canUseTool` ist das Gate des Boten |
| Structured Outputs im SDK ([Doku](https://code.claude.com/docs/en/agent-sdk/structured-outputs)) | `output_format` mit JSON-Schema, Feld `structured_output`, automatische Wiederholung bei Verstoß | Option | – | Kern (MVP) |
| Cost Tracking ([Doku](https://code.claude.com/docs/en/agent-sdk/cost-tracking)) | `total_cost_usd` je Session, `max_budget_usd` als Session-Deckel | Option | clientseitige Schätzung, nicht autoritativ | Kern (MVP), plus Usage-API-Abgleich |
| Session Storage, Hosting ([Storage](https://code.claude.com/docs/en/agent-sdk/session-storage), [Hosting](https://code.claude.com/docs/en/agent-sdk/hosting)) | JSONL-Transkripte unter `~/.claude/projects/`; je Session ein CLI-Subprozess | Dateisystem des VPS | – | Kern (MVP); SessionStore-Adapter erst bei Mehrhostbetrieb |
| Managed Agents ([Overview](https://platform.claude.com/docs/en/managed-agents/overview)) | vollverwaltete Alternative: Sessions, Environments, Events | REST, Beta-Header `managed-agents-2026-04-01` | Tokens ohne Batch-Rabatt plus $0,08 je aktiver Session-Stunde plus $10 je 1.000 Websuchen ([Budgets](https://platform.claude.com/docs/en/managed-agents/budgets)) | optional (v1-Test für den Rechercheur, v2-Entscheidung, Kapitel 7.10) |
| Scheduled Deployments ([Doku](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments)) | Cron minutengenau, IANA-Zeitzone, Jitter bis 15 % (5 s bis 9 min), Budget je gestarteter Session, max. 1.000 Deployments je Organisation | REST | siehe oben | optional (v1-Test) |
| Budgets ([Doku](https://platform.claude.com/docs/en/managed-agents/budgets)) | harter Deckel in USD-Cent, Session pausiert mit `budget_reached` | nur bei Session-Erstellung setzbar | – | optional |
| Vaults ([Doku](https://platform.claude.com/docs/en/managed-agents/vaults)) | Credentials (`mcp_oauth`, `static_bearer`, `environment_variable`), Klartext nie im Kontext | REST; `environment_variable` nicht mit self-hosted Sandboxes | im Preis enthalten | optional (v2 bei Migration) |
| Memory Stores ([Doku](https://platform.claude.com/docs/en/managed-agents/memory)) | Gedächtnis über Läufe; max. 8 je Session, 10.000 Einträge, 100 kB je Eintrag, 30 Tage Historie | Beta-Header `agent-memory-2026-07-22`, nicht mit `managed-agents-2026-04-01` kombinierbar (400) | im Preis enthalten | optional (v2, nur `read_only` in Läufen mit Drittinhalten) |
| Webhooks ([Doku](https://platform.claude.com/docs/en/managed-agents/webhooks)) | Ereignisse `session.status_idled`, `budget_reached`, `deployment_run.*` | HMAC-signiert, 3 Zustellversuche, kein durables Log | – | optional |
| Cloud Sandboxes ([Referenz](https://platform.claude.com/docs/en/managed-agents/cloud-sandboxes-reference)) | Ubuntu 24.04, bis 8 GB RAM, 10 GB Disk; Python, Node, Playwright mit Chromium, LibreOffice, Poppler, TeX Live, pandoc vorinstalliert; Tesseract nur Englisch | – | Session-Stunden | Hinweis für die Migration; ohne persistente DB |

#### 17.2.6 Zeitsteuerung auf Claude-Seite

| Mechanismus | Mindestintervall, Ort | Freigabe-Gate | Bewertung |
|---|---|---|---|
| systemd-Timer auf dem VPS (Kapitel 7.5) | frei, eigener Server | eigener Code, Hooks, `canUseTool` | Kern (MVP) |
| `/loop` ([Scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks)) | 1 Minute, nur bei offener Session, verfällt nach 7 Tagen | Session-Permissions | nur Entwicklung |
| Desktop-Scheduled-Tasks ([Doku](https://code.claude.com/docs/en/desktop-scheduled-tasks)) | 1 Minute, Rechner muss laufen, lokale Dateien | Permission-Modus konfigurierbar | optional (Entwicklung auf dem Mac) |
| Claude Code Routines ([Doku](https://code.claude.com/docs/en/routines)) | 1 Stunde, Cloud, frischer Repo-Klon je Lauf, Tagesdeckel je Konto (Zahl nicht dokumentiert) | keine Permission-Prompts; Connectors dürfen ohne Nachfrage schreiben | nur Lese-Experimente, nie für Versand |
| Claude Cowork ([Produktseite](https://claude.com/product/cowork)) | Tages-/Wochenaufgaben, Desktop-App | Nutzer sieht Ergebnisse | vermeiden: nicht scriptbar, nicht prüfbar (mittlere Konfidenz zur Verfügbarkeit) |
| Managed Agents Scheduled Deployments | minutengenau, Anthropic-Cloud | Tool-Confirmation, Budget, Vault | optional (v1-Test) |

### 17.3 (B) Datenquellen-APIs

Die vollständige Matrix mit Endpunkten, Risikoampel und Stufenstrategie steht in Kapitel 6; hier nur das Inventar mit Verdikt.

| Quelle | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| BA-Jobsuche-API ([bundesAPI/jobsuche-api](https://github.com/bundesAPI/jobsuche-api), [openapi.yaml](https://raw.githubusercontent.com/bundesAPI/jobsuche-api/main/openapi.yaml)) | Primärquelle des Scouts | REST GET `/pc/v6/jobs`, Header `X-API-Key: jobboerse-jobsuche`; inoffiziell, kein SLA, Schema-Brüche v4→v6 ([Issues](https://github.com/bundesAPI/jobsuche-api/issues?q=is%3Aissue)) | 0 EUR | Kern (MVP) |
| ATS-Feeds Personio ([XML](https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML)), Greenhouse ([API](https://developers.greenhouse.io/job-board.html)), Lever ([API](https://github.com/lever/postings-api)) | Watchlist-Adapter | GET ohne Auth; Personio/Greenhouse in Phase 0 live prüfen | 0 EUR | Kern (MVP) |
| ATS-Feeds Recruitee ([API](https://docs.recruitee.com/reference/intro-to-careers-site-api)), SmartRecruiters ([API](https://developers.smartrecruiters.com/docs/posting-api)), Workday CXS ([Community-Guide](https://github.com/Francis1998/agentic-career-search/blob/main/docs/guides/WORKDAY_SOURCE_GUIDE.md)) | Watchlist-Adapter | GET ohne Auth; Workday POST, Seitengröße 20, nur Community-belegt | 0 EUR | Kern (v1) |
| Teamtailor ([Doku](https://docs.teamtailor.com/)), JOIN ([join.com](https://join.com)), generischer JSON-LD-Adapter ([JobPosting](https://developers.google.com/search/docs/appearance/structured-data/job-posting)) | Watchlist-Rest | Teamtailor-Feed unbestätigt (Widerspruch in der Recherche); JOIN inoffiziell | 0 EUR | Kern (v1) nach Live-Test; JOIN v2 |
| Job-Alert-Mails (StepStone, Indeed, LinkedIn, XING) | Metadaten aus dem eigenen Postfach; Volltext nur nach Klick | IMAP-Leser des Scouts | 0 EUR | Kern (MVP) |
| Adzuna API ([developer.adzuna.com](https://developer.adzuna.com/)) | offizielle Ergänzungsquelle DE/AT | App-ID und App-Key; Rate Limits nicht öffentlich, bei Registrierung notieren | kostenloser Tarif | Kern (MVP) |
| Arbeitnow API ([Doku](https://arbeitnow.com/api/job-board-api)) | englischsprachige Tech-Rollen | REST ohne Auth | 0 EUR | Kern (MVP), sofern Zielrollen passen |
| Google for Jobs via SerpAPI ([Google Jobs API](https://serpapi.com/google-jobs-api), [Pricing](https://serpapi.com/pricing)) | Meta-Index über StepStone, Indeed, Firmenseiten | API-Key; Gelb-Quelle mit Volumendeckel 30 Suchen je Tageslauf | Free 250 Suchen/Monat; 25 USD/1.000; 75 USD/5.000; 150 USD/15.000; 275 USD/30.000 ([costbench](https://costbench.com/software/web-scraping/serpapi/)) | Kern (v1) nach Abdeckungsmessung |
| DataForSEO Google Jobs ([Pricing](https://dataforseo.com/pricing/serp/google-jobs-serp-api)) | Preisalternative zu SerpAPI | Pay-as-you-go | nicht recherchiert | optional |
| Entgeltatlas-API ([bundesAPI/entgeltatlas-api](https://github.com/bundesAPI/entgeltatlas-api)) | Gehaltsschätzung nach KldB-Code, immer als Schätzung markiert (Kapitel 9.3) | REST, OAuth2 Client-Credentials oder X-API-Key; inoffiziell | 0 EUR | optional (v1); nicht in der MVP-Liste von Kapitel 19.4 |
| KldB 2010 ([Destatis](https://www.destatis.de/DE/Methoden/Klassifikationen/Berufe/klassifikation-berufe-kldb-2010.html)) | Schlüssel für Entgeltatlas, Mapping zu ESCO | Download | 0 EUR | optional (v1, mit Entgeltatlas) |
| ESCO ([Download/API](https://esco.ec.europa.eu/en/use-esco/download)) | Skill- und Berufsnormalisierung (Matcher v1), Synonyme im ATS-Prüfer | REST oder lokale Kopie, kostenlos, mehrsprachig | 0 EUR | Kern (MVP als lokale Kopie für Synonyme; v1 für Matching) |
| JSearch/RapidAPI ([Pricing](https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch/pricing)) | Cross-Check-Aggregator | RapidAPI-Key; DE-Abdeckung unbestätigt | Free 200 Requests; ca. 10–200 USD/Monat | optional (v2), nur nach Freigabe |
| Jooble ([API](https://jooble.org/api/about)) | – | Key je Länderportal, POST-only | 500 Requests lebenslang je Key | vermeiden |
| Google Cloud Talent Solution ([Doku](https://docs.cloud.google.com/talent-solution/job-search/v3/docs/basics)) | – | nur eigene Jobdaten | nutzungsabhängig | vermeiden (falscher Anwendungsfall) |
| StepStone, Indeed, LinkedIn, XING/onlyfy, Monster, Glassdoor, Jobware, stellenanzeigen.de, meinestadt.de, Kimeta | – | keine Lese-API; AGB verbieten Scraping ([StepStone](https://www.stepstone.de/ueber-stepstone/nutzungsbedingungen-2022-03/), [Indeed](https://www.indeed.com/legal), [LinkedIn](https://www.linkedin.com/help/linkedin/answer/a1341387), [Monster](https://www.monster.com/inside/terms-of-use)) | – | vermeiden (Rot, Kapitel 6.1, 16.8) |
| Apify-Actors ([Indeed](https://apify.com/misceres/indeed-scraper), [LinkedIn](https://apify.com/bebity/linkedin-jobs-scraper), [StepStone](https://apify.com/jupri/stepstone-scraper)) | Pay-per-Result-Scraper Dritter | Apify-Token | ca. 0,05–5 USD je 1.000 Ergebnisse je Actor | optional (v2), nur nach ausdrücklicher Freigabe |
| python-jobspy ([GitHub](https://github.com/speedyapply/JobSpy)) | – | pip; Proxys für LinkedIn „a must“ | 0 EUR | vermeiden (Rot-Quellen, 13 Monate ohne Release) |
| Bright Data Jobs Scraper ([Produkt](https://brightdata.com/products/web-scraper/jobs-scraper)), ScraperAPI ([Pricing](https://www.scraperapi.com/pricing/)) | – | API | 0,75–1,50 USD je 1.000 Records; ScraperAPI ab 49 USD/Monat | vermeiden (überdimensioniert; ScraperAPI umgeht Bot-Abwehr) |
| softgarden Frontend-API ([dev.softgarden.de](https://dev.softgarden.de/career-websites-api/jobs-api/)) | – | ClientID nur vom Arbeitgeber | – | vermeiden; generischer Adapter stattdessen |

### 17.4 (C) Recherche-APIs und -Dienste

Standardpfad des Rechercheurs sind Web Search und Web Fetch (17.2.2) in der Stufenfolge aus Kapitel 10.1; alles Weitere ist Ergänzung.

| Dienst | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| Impressum nach § 5 DDG ([Gesetzestext](https://www.gesetze-im-internet.de/ddg/__5.html)) | verbindlichste Quelle für Firmenname, Rechtsform, Sitz | Web Fetch auf `{domain}/impressum` | 0 EUR | Kern (MVP); Rechtslage in der Recherche nicht live geprüft |
| Handelsregister über bundesAPI-Scraper ([bundesAPI/handelsregister](https://github.com/bundesAPI/handelsregister)) und Paket `deutschland` ([GitHub](https://github.com/bundesAPI/deutschland)) | Einzelabfrage im Zweifelsfall (Stufe 4) | Python; selbst auferlegtes Limit 60 Abfragen je Stunde, Warnung vor §§ 303a/b StGB bei Massenabfragen | 0 EUR | Kern (MVP), Rate-Limiter hart im Code |
| handelsregister.de ([Portal](https://www.handelsregister.de/)) | – | AGB untersagen automatisierten Abruf | Dokumentabruf teils gebührenpflichtig | vermeiden für Automatisierung; nur manuell |
| Northdata ([northdata.de](https://www.northdata.de/)) | Registerdaten bei Widerspruch | öffentliche Ergebnisseite per Web Search/Fetch; API kostenpflichtig | Preise nicht verifiziert | optional, Einzelfall |
| OpenCorporates ([API](https://opencorporates.com/)) | internationale Firmen | API-Key | Free Tier limitiert, Preise nicht verifiziert | optional; DE-Abdeckung lückenhaft |
| Unternehmensregister/Bundesanzeiger ([Portal](https://www.unternehmensregister.de/)) | Bonitäts- und Größencheck | manuell | teils gebührenpflichtig | optional, manuell |
| Kununu ([kununu.com](https://www.kununu.com/)) | Kultur, Ton, Bewerbungsprozess | ein Web-Fetch-Aufruf je Firma, kein Massen-Scraping | 0 EUR | Kern (MVP), punktuell |
| Glassdoor ([glassdoor.de](https://www.glassdoor.de/)) | Zweitquelle für US-verwurzelte Arbeitgeber | Web Search | 0 EUR | optional |
| LinkedIn, XING (Personenprofile) | Bestätigung, ob Ansprechperson noch im Unternehmen | ausschließlich manuell durch dich; Rechercheur liefert nur Suchlinks (Kapitel 10.1) | 0 EUR | Kern (manuell); Automatisierung vermeiden |
| Jina Reader ([r.jina.ai](https://jina.ai/reader/)) | HTML→Markdown vor der Haiku-Extraktion | `GET r.jina.ai/{url}`; ohne Key ca. 20 Anfragen je Minute | kostenlos; mit Key 10 Mio. Gratis-Tokens, danach Pay-as-you-go | Kern (v1) |
| Firecrawl ([firecrawl.dev](https://www.firecrawl.dev)) | Scrape/Crawl/Extract für Karriereseiten ohne Feed, ohne Stealth-Modus | API-Key | Free 1.000 Credits/Monat; Hobby 16 USD/Monat; Standard 83 USD/Monat ([scrapegraphai](https://scrapegraphai.com/blog/firecrawl-pricing)) | Kern (v1) als Fallback |
| Crawl4AI ([GitHub](https://github.com/unclecode/crawl4ai)) | Self-hosted Alternative zu Firecrawl | Docker/Python, Playwright-basiert; „Undetected-Chrome“ nicht nutzen | 0 EUR plus Betrieb | optional (v2) |
| Exa ([exa.ai](https://exa.ai/)), Tavily ([tavily.com](https://tavily.com/)) | semantische bzw. RAG-freundliche Zweitsuche | API-Key | Preise in der Recherche nicht belastbar belegt | optional (v2) |
| Brave Search API ([brave.com](https://brave.com/search/api/)) | günstige Zweitsuche | API-Key, Kreditkarte | $5 je 1.000, $5 Gratis-Guthaben je Monat (siehe 17.2.4) | optional |
| Perplexity Sonar ([perplexity.ai](https://www.perplexity.ai/)) | aktuelle Firmennews mit Zitaten | API-Key | Preise nicht verifiziert | optional |
| webappanalyzer ([GitHub](https://github.com/enthec/webappanalyzer), [categories.json](https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json)) | ATS-Detektor Stufe 3: Fingerprints der Kategorie 101 für Personio, Greenhouse, Lever, Recruitee, SmartRecruiters, Teamtailor, onlyfy; nicht für softgarden, rexx, d.vinci, SuccessFactors, JOIN | JSON-Dateien im Repo, eigener Matcher | 0 EUR | Kern (v1) |
| BuiltWith ([builtwith.com](https://builtwith.com/)), Wappalyzer ([wappalyzer.com](https://www.wappalyzer.com/)) | Tech-Stack der Zielfirma | Web-Lookup kostenlos, API kostenpflichtig | Preise nicht verifiziert | optional |
| OpenStreetMap Nominatim ([Doku](https://nominatim.org/release-docs/latest/api/Overview/)) | Adress-Existenzprüfung | max. 1 Anfrage je Sekunde, Pflicht-User-Agent (unbestätigt) | 0 EUR | optional |
| Google Places / Address Validation ([Doku](https://developers.google.com/maps/documentation/places/web-service)) | Schreibweisenprüfung der Anschrift | API-Key | Preise nicht verifiziert | optional |
| Google Maps Routes API | Pendelzeit für den Muss-Filter | API-Key; 10.000 Events/Monat kostenlos, danach 2–30 USD je 1.000 ([woosmap](https://www.woosmap.com/blog/google-maps-api-pricing-breakdown), mittlere Konfidenz) | siehe Zugriff | optional (v1); MVP nutzt Ort/Umkreis der BA-API |
| OSRM/OpenRouteService self-hosted | – | eigener Server | Serverkosten | vermeiden (Aufwand) |
| Insolvenz-Radar ([Funktionen](https://insolvenz-radar.de/funktionen/)) | Insolvenzmonitoring | API, kostenpflichtig | Preis nicht recherchiert | optional; sonst manuelle Stichprobe |
| Apollo.io, Hunter.io, Lusha ([Apollo](https://www.apollo.io/)) | – | – | – | vermeiden (DSGVO, Kapitel 16.8) |
| Crunchbase ([crunchbase.com](https://www.crunchbase.com/)), Dealroom ([dealroom.co](https://dealroom.co/)) | – | Enterprise-Preise | – | vermeiden |
| Startbase ([startbase.de](https://www.startbase.de/)), wlw ([wlw.de](https://www.wlw.de/)), gehalt.de ([gehalt.de](https://www.gehalt.de/)) | punktueller Kontext, Gehaltsreferenz | manuell/Web Search | 0 EUR | optional |

### 17.5 (D) Browser-Automation

Grundsatz aus Kapitel 15.5 und 16.8: Der Klick auf „Absenden“ und die DSGVO-Einwilligung bleiben beim Menschen; Plattform-Schnellbewerbungen werden nie automatisiert abgesendet.

| Werkzeug | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| Playwright, Python-API ([playwright.dev](https://playwright.dev/python/docs/api/class-browsertype)) | Portal-Co-Pilot: `launchPersistentContext` mit echtem Chrome-Profil, sichtbar, auf dem Rechner des Nutzers; `page.setInputFiles()` für den Lebenslauf-Upload; Screenshot der Bestätigungsseite | `pip install playwright`, Chromium/Chrome lokal | 0 EUR, Apache-2.0 | Kern (v2) |
| Playwright MCP ([microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp)) | Erkundung von Formularen in der Entwicklung; Einzelabruf JS-lastiger Karriereseiten im Scout | siehe 17.2.4; Accessibility-Snapshots statt Screenshots | 0 EUR | optional (v2) |
| browser-use ([GitHub](https://github.com/browser-use/browser-use)) | LLM-gesteuerter Fallback für unbekannte Firmenformulare ohne Selektoren | `pip install browser-use`, Anthropic-Modelle direkt | Bibliothek MIT, 0 EUR; Cloud 0,01 USD je Task plus 20 % auf Tokens plus Browserzeit ([Pricing](https://browser-use.com/pricing), mittlere Konfidenz) | optional (v2), nach Einzelprüfung |
| Stagehand ([GitHub](https://github.com/browserbase/stagehand)) | act/observe/extract für iframe-lastige Widgets | TS/Python/Go, lokal mit Chromium | MIT, 0 EUR; Browserbase optional kostenpflichtig | optional |
| Skyvern ([GitHub](https://github.com/Skyvern-AI/skyvern)) | robuste Formularautomation mit Vision | Docker Compose self-hosted oder Cloud | AGPL-3.0; Cloud Hobby 29 USD/Monat, Pro 149 USD/Monat plus 0,05 USD je Schritt (mittlere Konfidenz) | optional (v2), nur nach Lizenz- und Nutzenprüfung |
| Claude Browser-Use / Computer-Use-Toolsets ([Browser Use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/browser-use-tool), [Computer Use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool)) | – | client-seitig auszuführen; nicht in Managed Agents; 1.000–1.800 Token je Screenshot | Tokenpreise | vermeiden (teurer und weniger deterministisch als Playwright) |
| Browserbase gehosteter Endpunkt | – | `https://mcp.browserbase.com/mcp` | nutzungsabhängig | vermeiden (Repo archiviert; Bot-Erkennung wird nicht umgangen) |
| Simplify Copilot ([simplify.jobs](https://simplify.jobs/copilot)), LazyApply ([lazyapply.com](https://www.lazyapply.com/)), Teal ([tealhq.com](https://www.tealhq.com/)) | – | Chrome-Erweiterungen, Blackbox, Daten bei Dritten | kostenlos bzw. Abo | vermeiden als Kernkomponente; Teal privat unproblematisch |
| Captcha-Löser (2captcha u. ä.), gepatchte Browser gegen Bot-Erkennung | – | – | – | vermeiden (Umgehung technischer Sperren, Kapitel 16.1) |

### 17.6 (E) Dokumente und Test-Parsing

Toolchain-Entscheidung in Kapitel 13.2: WeasyPrint für PDF, python-docx für DOCX, kein Modell im Renderpfad; Test-Parsing in Kapitel 12.6.

| Werkzeug | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| WeasyPrint ([weasyprint.com](https://weasyprint.com/), [Doku](https://doc.courtbouillon.org/weasyprint/stable/common_use_cases.html)) | Primär-Renderer HTML/CSS→PDF, DIN-5008-Positionierung in mm | `pip install weasyprint` plus Systembibliotheken | BSD-artig, 0 EUR | Kern (MVP); PDF/A und PDF/UA experimentell, nicht bewerben |
| Jinja2 | Vorlagen `templates/de/sachlich/*.html` | pip | 0 EUR | Kern (MVP) |
| python-docx ([Doku](https://python-docx.readthedocs.io/)) | DOCX-Lebenslauf nativ aus derselben JSON-Struktur | pip | MIT, 0 EUR | Kern (MVP) |
| pypdf, pdfplumber | Mappe zusammenführen, Lesezeichen, Metadaten; Textextraktion, Zeichenattribute (Weißtext-Scan) | pip | 0 EUR | Kern (MVP) |
| Poppler-Utilities (`pdftotext`, `pdffonts`, `pdftoppm`) | Setzer-QA: Textextraktion, Schrifteinbettung, Seitenvorschau | apt | 0 EUR | Kern (MVP) |
| LibreOffice headless (`soffice`) | DOCX→PDF nur zur Prüfung der Seitenzahl | apt | 0 EUR | Kern (MVP, Prüfpfad) |
| pandoc ([pandoc.org](https://pandoc.org/)) | DOCX→Text für den Text-Diff der DOCX-Prüfung | apt | GPL, 0 EUR | Kern (MVP, Prüfpfad) |
| Tesseract mit `tesseract-ocr-deu` | einmalige OCR der Zeugnisse beim Onboarding | apt; Claude-Sandbox liefert nur Englisch | 0 EUR | Kern (Phase 0) |
| pdf2image, Pillow, reportlab (oder Ghostscript) | Zeugnisse komprimieren (Graustufen, 150 dpi) | pip/apt | 0 EUR | Kern (Phase 0) |
| qpdf | PDF-Reparatur, Linearisierung | apt | 0 EUR | optional |
| Carlito, Liberation Sans | eingebettete Schriften (metrisch kompatibel zu Calibri/Arial) | Dateien in `assets/fonts/` | frei lizenziert | Kern (MVP) |
| Typst ([GitHub](https://github.com/typst/typst)) | Zweitrenderer für den Lebenslauf | Single-Binary, `typst compile`; kein DIN-5008-Template vorhanden, Universe-Pakete pinnen ([modern-cv](https://typst.app/universe/package/modern-cv/), [brilliant-cv](https://typst.app/universe/package/brilliant-cv/)) | Apache-2.0, 0 EUR | optional (v1-Evaluation, V-16) |
| RenderCV ([GitHub](https://github.com/rendercv/rendercv)) | Referenz für Datenmodell und Typografie | pip, YAML→PDF, Python 3.12+ | MIT | Referenz |
| LaTeX moderncv ([CTAN](https://ctan.org/pkg/moderncv)), Tectonic ([GitHub](https://github.com/tectonic-typesetting/tectonic)) | Fallback für konservative Branchen | TeX Live bzw. Single-Binary | 0 EUR | optional |
| JSON Resume ([jsonresume.org](https://jsonresume.org/)) | Schema-Basis für `lebenslauf.yaml` und Dossier | offenes Schema | 0 EUR | Referenz |
| Reactive Resume ([GitHub](https://github.com/amruthpillai/reactive-resume)) | Layout-Inspiration | Docker, Web-App | MIT | Referenz, nicht integrieren |
| Puppeteer ([pptr.dev](https://pptr.dev/)), Paged.js ([GitHub](https://github.com/pagedjs/pagedjs/)) | – | Node | 0 EUR | vermeiden (Browser-Overhead ohne Nutzen) |
| docxtemplater ([Pricing](https://docxtemplater.com/pricing/)), Carbone ([Pricing](https://carbone.io/pricing.html)) | – | npm bzw. Cloud/self-hosted | 1.250–9.000 EUR/Jahr bzw. ab 29 EUR/Monat (unbestätigt) | vermeiden |
| Anthropic docx-/pdf-Skill ([docx](https://raw.githubusercontent.com/anthropics/skills/main/skills/docx/SKILL.md), [pdf](https://raw.githubusercontent.com/anthropics/skills/main/skills/pdf/SKILL.md)) | Werkzeugkasten in interaktiven Sitzungen (Vorlagen prüfen, Mappen zusammenführen) | Plugin oder `container.skills` | source-available; Code-Execution-Kosten | optional (siehe 17.1) |
| Apache Tika 4.0.0, `tika-server` ([GitHub](https://github.com/apache/tika)) | Test-Parsing Stufe 2: generische Textextraktion, Reihenfolge, Encoding | Java, self-hosted REST | Apache-2.0, 0 EUR | Kern (MVP) |
| OpenResume-Parser ([GitHub](https://github.com/xitanggg/open-resume)) | Test-Parsing Stufe 2: Feldererkennung (nur PDF) | npm oder Docker, `localhost:3000` | AGPL-3.0, 0 EUR; nur betreiben, nicht kopieren | Kern (MVP) |
| pyresparser ([GitHub](https://github.com/OmkarPathak/pyresparser)) | dritte Parser-Meinung | pip, spaCy/NLTK, primär Englisch | 0 EUR; 33 offene Issues, Wartung unklar | optional (Default aus, Kapitel 12.11) |
| Eden AI ([Übersicht](https://www.edenai.co/post/best-resume-parser-apis)) | kommerzielle Parser-Zweitmeinung (Affinda, SenseLoaf u. a.) | REST | ab 0,04–0,10 USD je Datei oder Seite (mittlere Konfidenz) | optional |
| Affinda ([G2](https://www.g2.com/products/resume-parser-by-affinda/pricing)) | – | REST | ab ca. 800 USD/Monat, kein Free-Tier | vermeiden |
| Jobscan ([Preise](https://pitchmeai.com/blog/jobscan-pricing-plans)), Resume Worded ([Site](https://resumeworded.com/)) | – | Web | Jobscan 5 Scans/Monat gratis, Premium $49,95/Monat; Resume Worded $49/Monat | vermeiden (US-fokussiert, kostenpflichtig) |
| Textkernel ([Parser](https://www.textkernel.com/de/produkte-loesungen/parser/)) | Referenz: Parser hinter Personio, softgarden, d.vinci | Enterprise | nicht öffentlich | vermeiden für eigenen Einsatz; als Referenz wichtig |

### 17.7 (F) E-Mail

Entscheidungen in Kapitel 15: Entwurfsmodus im MVP, SMTP-Versand in v1, Konto nach Nutzerwahl.

| Werkzeug | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| iCloud Mail SMTP/IMAP ([Apple](https://support.apple.com/en-us/102198)) | Entwurf per IMAP-APPEND, später SMTP-Versand | `smtp.mail.me.com:587` STARTTLS, `imap.mail.me.com:993`; App-spezifisches Passwort, 2FA Pflicht; wird bei Apple-ID-Passwortänderung widerrufen | 0 EUR; 1.000 Nachrichten und 1.000 Empfänger je Tag, 500 je Nachricht | Kern (MVP), wenn iCloud das Konto ist |
| Gmail API Drafts ([Drafts](https://developers.google.com/workspace/gmail/api/guides/drafts), [Scopes](https://developers.google.com/workspace/gmail/api/auth/scopes), [Quota](https://developers.google.com/workspace/gmail/api/reference/quota)) | Entwurf per `users.drafts.create` | OAuth 2.0, Scope `gmail.compose` oder `gmail.modify` (beide „Restricted“); Testing-Modus mit einem Testnutzer, Refresh-Token läuft nach 7 Tagen ab ([OAuth Testing](https://support.google.com/cloud/answer/15549945?hl=en)) | 0 EUR; 500 Mails je Tag privat | Kern (MVP), wenn Gmail das Konto ist |
| Microsoft Graph Mail ([Doku](https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0)) | Outlook/Microsoft 365 | OAuth 2.0, Azure-App-Registrierung | 0 EUR | optional, nur bei bestehendem Konto |
| `imaplib`, `smtplib`, `email` (Python-Standardbibliothek) | IMAP-APPEND im Entwurfsmodus, `Message-ID` per `email.utils.make_msgid` | stdlib | 0 EUR | Kern (MVP) |
| imap_tools ([GitHub](https://github.com/ikvk/imap_tools)) | Tracker-Daemon mit IMAP IDLE; Job-Alert-Leser | pip, Python 3.8+, keine Abhängigkeiten | Apache-2.0, 0 EUR | Kern (v1); im MVP für den Alert-Leser einsetzbar |
| aiosmtplib ([Doku](https://aiosmtplib.readthedocs.io/en/latest/usage.html)) | SMTP-Versand im Versandfenster | pip | 0 EUR | Kern (v1) |
| icalendar ([PyPI](https://pypi.org/project/icalendar)) | ICS-Einladungen strukturiert parsen | pip | 0 EUR | Kern (v1) |
| yagmail ([GitHub](https://github.com/kootenpv/yagmail)) | – | Gmail-only | 0 EUR | vermeiden |
| Nodemailer ([GitHub](https://github.com/nodemailer/nodemailer)), ImapFlow ([GitHub](https://github.com/postalsys/imapflow)) | – | Node | 0 EUR | vermeiden (Python-Stack) |
| Composio Agent Mail ([composio.dev](https://composio.dev/toolkits/agent_mail)), Pipedream MCP, Zapier MCP ([zapier.com/mcp](https://zapier.com/mcp)) | – | OAuth über Drittanbieter | Free Tiers, Preise nicht belegt | vermeiden (Drittzugriff auf das Postfach) |
| Fastmail ([Preise](https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates)), mailbox.org ([Preise](https://mailbox.org/en/news/new-price-plans-available-mailboxorg/)), iCloud+ Custom Domain ([Apple](https://support.apple.com/en-us/102540)) | eigene Bewerbungsdomain | IMAP/SMTP wie Basiskonto, DNS-Setup, Warm-up | Fastmail ca. 6 $/Monat, mailbox.org ca. 3 €/Monat (beide unbestätigt) | optional (v1, V-16) |
| Proton Mail ([Preise](https://proton.me/mail/pricing)), Google Workspace | – | Proton nur über Bridge-App | ca. 47 $/Jahr bzw. 8,40 $/Nutzer/Monat (unbestätigt) | vermeiden |

### 17.8 (G) Daten, Embeddings, Speicher

| Werkzeug | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| SQLite (WAL, `foreign_keys=ON`) | einzige Datenbank in MVP und v1; Schema in Kapitel 7.3; `event_log` append-only per Trigger | Python-Standardbibliothek, `sqlite3`-CLI für `.backup` | 0 EUR | Kern (MVP) |
| Git, zwei private Repositories | `bewerbungsagent` (Code) und `bewerbungen-data` (Profil, Bewerbungen); Commit je Statuswechsel | git | 0 EUR | Kern (MVP) |
| pydantic | Nachprüfung der Structured Outputs (Kennziffer, PLZ, E-Mail) | pip | 0 EUR | Kern (MVP) |
| datasketch ([GitHub](https://github.com/ekzhu/datasketch)) | MinHashLSH für Dedup (`num_perm=128`, Schwelle 0,75–0,85) | pip, MIT | 0 EUR | Kern (MVP) |
| bm25s ([GitHub](https://github.com/xhluca/bm25s)) mit PyStemmer | BM25-Vorauswahl mit deutschem Stemmer | pip, MIT | 0 EUR | Kern (MVP) |
| rank_bm25 ([GitHub](https://github.com/dorianbrown/rank_bm25)) | Ersatz, falls bm25s Probleme macht | pip | 0 EUR | optional |
| Jaro-Winkler (z. B. über recordlinkage oder eigene Implementierung) | Blocking-Stufe des Dedup | pip | 0 EUR | Kern (MVP) |
| recordlinkage ([GitHub](https://github.com/J535D165/recordlinkage)) | feldbasierter Vergleich als Ergänzung | pip, BSD-3 | 0 EUR | optional |
| dedupe ([GitHub](https://github.com/dedupeio/dedupe)) | – | braucht gelabelte Trainingspaare | 0 EUR | vermeiden (Trainingsaufwand) |
| sentence-transformers ([GitHub](https://github.com/UKPLab/sentence-transformers)) | lädt Embedding- und Cross-Encoder-Modelle lokal | pip, Apache-2.0 | 0 EUR | Kern (v1) |
| BGE-M3 ([FlagEmbedding](https://github.com/FlagOpen/FlagEmbedding)) | Dense-Embeddings, 100+ Sprachen, 8.192 Token | Hugging Face, MIT | 0 EUR; braucht mehr RAM als der CPX22 (Kapitel 7.7.3) | Kern (v1), Default; Alternative API-Embedding |
| LanceDB ([GitHub](https://github.com/lancedb/lancedb)) | eingebettete Vektorablage neben SQLite | pip, Apache-2.0, kein Server | 0 EUR | Kern (v1) |
| rerankers ([GitHub](https://github.com/AnswerDotAI/rerankers)) | austauschbare Reranker-Schicht | pip, Apache-2.0 | 0 EUR | Kern (v1) |
| Cohere Rerank 3.5 ([OpenRouter](https://openrouter.ai/cohere/rerank-v3.5)) | Reranking Top-30/50 auf Top-20 | API-Key | 0,001 USD je Suche (mittlere Konfidenz) | Kern (v1), Default-Reranker |
| Voyage AI rerank-2.5, voyage-4 ([Pricing](https://docs.voyageai.com/docs/pricing)) | Alternative mit Freikontingent | API-Key, SDK `voyageai` | rerank-2.5 0,05 USD je Mio. Token, voyage-4 0,06 USD je Mio., 200 Mio. Token gratis (unbestätigt) | optional |
| Jina Embeddings/Reranker ([jina.ai](https://jina.ai/reranker/)) | – | API oder Open-Weights unter CC-BY-NC 4.0 | ab 0,018 USD je Mio. Token (mittlere Konfidenz) | optional; Self-Hosting wegen Lizenz vermeiden |
| OpenAI text-embedding-3-large ([Pricing](https://platform.openai.com/docs/pricing)), Cohere Embed v4 | – | API | 0,13 USD je Mio. Token bzw. 0,12 USD (niedrige Konfidenz) | vermeiden (kein belegter Vorteil für Deutsch) |
| pgvector ([GitHub](https://github.com/pgvector/pgvector)) | Vektorsuche in Postgres | Extension, HNSW/IVFFlat | 0 EUR plus Postgres | Kern (v2) nur bei Mehrnutzerbetrieb |
| Qdrant ([GitHub](https://github.com/qdrant/qdrant)) | – | Docker | 0 EUR self-hosted | vermeiden (zweite Infrastruktur ohne Bedarf) |
| spaCy `de_core_news_lg` ([Release](https://github.com/explosion/spacy-models/releases/tag/de_core_news_lg-3.8.0)) | deutsche NER als zweite Prüfinstanz für Ansprechpartner | `spacy download`, 541 MB, MIT | 0 EUR | optional (v1) |
| `deutschland` ([GitHub](https://github.com/bundesAPI/deutschland)) | Sammel-Client für Bundesanzeiger, Handelsregister, Jobsuche | pip, Apache-2.0 | 0 EUR | Kern (MVP) |
| MTEB-Leaderboard ([Hugging Face](https://huggingface.co/spaces/mteb/leaderboard)) | Modellvergleich mit deutschen Tasks vor V-11 | Web | 0 EUR | Referenz |
| Hetzner Object Storage ([Produkt](https://www.hetzner.com/storage/object-storage/)) | verschlüsseltes Offsite-Backup | S3-kompatibel | 4,99 €/Monat inkl. 1 TB (unbestätigt) | optional (v1, V-15) |
| Managed Agents Memory Stores | Profil- und Firmenhistorie bei Migration | siehe 17.2.5 | im Preis enthalten | optional (v2) |

### 17.9 (H) UI und Benachrichtigung

| Werkzeug | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| FastAPI ([GitHub](https://github.com/fastapi/fastapi)) | Review-Cockpit-Backend, Telegram-Webhook | pip, MIT; nur über SSH-Tunnel erreichbar | 0 EUR | Kern (MVP) |
| htmx ([GitHub](https://github.com/bigskysoftware/htmx)) | Teil-Updates der Detailseite ohne SPA | Version 2.0.10; Lizenz vor Einsatz in der LICENSE-Datei prüfen (P0-22) | 0 EUR | Kern (MVP) |
| Jinja2, `difflib` (stdlib) | serverseitiges Rendering; Wort-Diff Master vs. Variante | pip / stdlib | 0 EUR | Kern (MVP) |
| python-telegram-bot ([GitHub](https://github.com/python-telegram-bot/python-telegram-bot)) | Push je Stelle, Tagesdigest, vier Aktionen als Inline-Buttons, Rückfragen | Bot-Token von BotFather; LGPL-3/GPL-Anteile beachten | 0 EUR | Kern (MVP) |
| E-Mail-Digest über das Bewerbungspostfach | Rückfallkanal bei Bot-Ausfall | Bote-Code | 0 EUR | Kern (MVP) |
| NiceGUI ([GitHub](https://github.com/zauberzeug/nicegui)), Reflex ([GitHub](https://github.com/reflex-dev/reflex)) | fertige Python-UI-Bausteine | pip; MIT bzw. Apache-2.0 | 0 EUR | optional (v1, falls HTMX nicht reicht) |
| Streamlit ([GitHub](https://github.com/streamlit/streamlit)), Gradio ([GitHub](https://github.com/gradio-app/gradio)) | – | pip | 0 EUR | vermeiden (Rerun-Modell, ML-Demo-Fokus) |
| Notion SDK ([GitHub](https://github.com/makenotion/notion-sdk-js)) | Kanban-/Kalender-Spiegel per Sync-Job | API-Version 2025-09-03 ab SDK v5 | Freemium; Automationen planabhängig, unbestätigt | optional (nach v1) |
| GitHub Issues/PRs | Diff- und Kommentar-Workflow im Daten-Repo | GitHub Free | 0 EUR | optional (v1, für Git-affine Nutzung) |
| Claude Artifacts | Cockpit-Experiment ohne Hosting | Claude-Produkt | im Abo | Experiment, kein Produktionsfundament |
| Obsidian Dataview ([GitHub](https://github.com/blacksmithgu/obsidian-dataview)) | Statusübersicht aus Markdown-Frontmatter | Obsidian, Plugin MIT | 0 EUR | optional |
| Airtable, Google Sheets | – | proprietär | Freemium | vermeiden (kein Diff, keine PDF-Vorschau) |
| Slack Block Kit, Discord, WhatsApp Business API | – | OAuth-App bzw. Meta-Verifizierung | Free Tier bzw. Kosten je Konversation | vermeiden (Mehraufwand ohne Nutzen; WhatsApp unverhältnismäßig) |
| grammY ([GitHub](https://github.com/grammyjs/grammY)) | – | Node | 0 EUR | vermeiden (Python-Stack) |

### 17.10 (I) Betrieb: Hosting, Secrets, Observability, Backup

| Werkzeug | Zweck bei uns | Zugriff | Kosten | Bewertung |
|---|---|---|---|---|
| Hetzner Cloud CPX22 ([Preisanpassung](https://docs.hetzner.com/de/general/infrastructure-and-availability/price-adjustment/), [Northflank](https://northflank.com/blog/hetzner-cloud-server-price-increases)) | Dauerserver in Falkenstein/Nürnberg, Ubuntu 24.04, Nutzer `agent`, nur SSH | Cloud-Konsole; 2 vCPU, 4 GB RAM, 80 GB NVMe | ca. 19,49–19,99 €/Monat nach der Erhöhung vom 15.6.2026 (vor Bestellung prüfen) | Kern (MVP) |
| systemd-Timer und -Services | Tageslauf, Versandlauf, Nachlauf, Wartung, Tracker-Daemon; `Persistent=true`, Zeitzone im Timer | Units unter `deploy/systemd/` | 0 EUR | Kern (MVP) |
| cron (cronie, `CRON_TZ`) | Alternative zu systemd | crontab | 0 EUR | optional |
| Fly.io ([fly.io](https://fly.io/)) | Zero-Ops-Alternative | Git-Deploy | shared-cpu-1x/1 GB ca. 5,70–5,92 $/Monat (unbestätigt) | optional |
| Mac mit Desktop-Scheduled-Tasks und Keychain | Entwicklung, Portal-Co-Pilot (residentielle IP) | lokal | 0 EUR | optional (Entwicklung, v2-Co-Pilot) |
| sops + age ([sops](https://github.com/getsops/sops)) | `config/secrets.enc.yaml`; `sops exec-env` injiziert Umgebungsvariablen | CLI, MPL-2.0; privater age-Schlüssel nur auf dem Server (0400) und im Passwortmanager | 0 EUR | Kern (MVP) |
| macOS Keychain (`security`, [ss64](https://ss64.com/mac/security-password.html)) | Secrets auf dem Mac des Nutzers für den Co-Piloten | systemeigen | 0 EUR | optional (v2, lokal) |
| 1Password CLI ([Doku](https://developer.1password.com/docs/cli/secrets-scripts)) | Secret-Referenzen `op://…`, Service-Accounts | Abo nötig | Business 7,99 $/Nutzer/Monat (mittlere Konfidenz) | optional, nur bei bestehendem Abo |
| Infisical ([GitHub](https://github.com/Infisical/infisical)) | Secret-Web-UI | Docker Compose self-hosted, MIT außer `ee/` | 0 EUR self-hosted; Cloud Pro 18 $/Identität/Monat (mittlere Konfidenz) | optional (mehrere Umgebungen) |
| Doppler ([doppler.com](https://www.doppler.com/)) | – | Cloud-only | Free bis 5 Identitäten | vermeiden |
| Managed Agents Vaults | Credentials ohne Klartext im Kontext | siehe 17.2.5 | im Preis | optional (v2 bei Migration) |
| `event_log` (SQLite) | Observability und Kostenwahrheit im MVP | eigener Code | 0 EUR | Kern (MVP) |
| Arize Phoenix ([GitHub](https://github.com/Arize-ai/phoenix)) | Tracing der Subagent-Läufe, OpenTelemetry, `openinference-instrumentation-anthropic` | `pip install arize-phoenix && phoenix serve`, SQLite-Backend | Elastic License 2.0, 0 EUR | Kern (v1, V-14) |
| Langfuse ([Self-Hosting](https://langfuse.com/self-hosting)) | – | Docker Compose mit Postgres, ClickHouse, Redis, MinIO; 4+ CPU, 16 GiB RAM, ca. 100 GiB | self-hosted 0 EUR; Cloud Hobby $0, Core $29/Monat, Pro $199/Monat (mittlere Konfidenz) | vermeiden (zweiter Server nur für Tracing) |
| Anthropic Console: Ausgabenlimit, Usage-and-Cost-API | dritte Stufe der Kostenkontrolle (Kapitel 7.7.7) | Console | 0 EUR | Kern (MVP) |
| `sqlite3 .backup`, verschlüsselte Kopie an zweiten Ort | nächtliches Backup, wöchentlich offsite | Wartungslauf | 0 EUR; Object Storage siehe 17.8 | Kern (MVP); offsite v1 |
| Docker (`deploy/Dockerfile`) | optionales Image mit Playwright- und WeasyPrint-Systempaketen | docker | 0 EUR | optional |
| n8n Community Edition ([GitHub](https://github.com/n8n-io/n8n)) | Glue-Schicht für Webhooks | Docker, Sustainable-Use-Lizenz | 0 EUR self-hosted; Cloud ab 20 €/Monat (mittlere Konfidenz) | vermeiden im MVP; allenfalls optionale Glue-Schicht |
| Temporal ([GitHub](https://github.com/temporalio/temporal)), Trigger.dev ([GitHub](https://github.com/triggerdotdev/trigger.dev)), Inngest ([GitHub](https://github.com/inngest/inngest)), LangGraph ([GitHub](https://github.com/langchain-ai/langgraph)), CrewAI ([GitHub](https://github.com/crewAIInc/crewAI)), APScheduler ([GitHub](https://github.com/agronholm/apscheduler)) | – | – | frei bis ca. 75 $/Monat (niedrige Konfidenz) | vermeiden (ein Lauf am Tag rechtfertigt keine Workflow-Engine; Agent SDK bringt Loop, Hooks, Permissions mit) |

### 17.11 (J) Entwicklungswerkzeuge und CLIs

| CLI / Werkzeug | Zweck bei uns | Install | Kosten | Bewertung |
|---|---|---|---|---|
| `claude` (Claude Code CLI) | Bauen, Testen, Onboarding-Sitzungen; `claude -p` für Golden-Tests; `--bare` für reproduzierbare CI-Läufe | Installation laut Claude-Code-Doku; für den Tageslauf vom Agent SDK gebündelt, kein separates Node-Setup nötig | Abo für interaktive Arbeit; Tageslauf über API-Key | Kern (MVP) |
| `git` | zwei Repos, Commit je Statuswechsel, Historie der Vorlagen | apt | 0 EUR | Kern (MVP) |
| Python 3.12, `uv` (oder `pip`) | Umgebung und Abhängigkeiten aus `pyproject.toml`; `uvx` für Phoenix und Werkzeuge | `.python-version`, `.venv` | 0 EUR | Kern (MVP); Poetry nicht vorgesehen |
| `pytest` | Unit-, Golden-, Sicherheits- und Ende-zu-Ende-Tests (Kapitel 19.4) | pip | 0 EUR | Kern (MVP) |
| `sops`, `age` | Secrets ver- und entschlüsseln, `exec-env` | Binaries | 0 EUR | Kern (MVP) |
| `sqlite3` | Migrationen prüfen, `.backup` | apt | 0 EUR | Kern (MVP) |
| `pdftotext`, `pdffonts`, `pdftoppm`, `soffice --headless`, `pandoc`, `tesseract` | Setzer-QA und Onboarding-OCR (17.6) | apt | 0 EUR | Kern (MVP) |
| Java-Laufzeit für `tika-server` | Test-Parsing | apt | 0 EUR | Kern (MVP) |
| Node.js, `npx` | Playwright MCP (v2), OpenResume (npm) | apt/nvm | 0 EUR | Kern (MVP für OpenResume; v2 für Playwright MCP) |
| `typst` | Zweitrenderer-Evaluation | Single-Binary | 0 EUR | optional (v1) |
| `docker`, `docker compose` | optionales Image; Crawl4AI, Infisical, Skyvern nur bei Bedarf | apt | 0 EUR | optional |
| `systemctl`, `journalctl` | Timer, Logs | Ubuntu | 0 EUR | Kern (MVP) |
| `/plugin` in Claude Code | document-skills, offizielle Marketplace | Claude Code | 0 EUR | optional |

### 17.12 Was bewusst fehlt

Damit niemand die Prüfung wiederholt: kein Scraping der Publikumsbörsen und keine Umgehung von Login, Captcha oder Ratenlimits (Kapitel 16.8); keine Kontaktanreicherung (Apollo.io, Hunter.io, Lusha); keine Chrome-Erweiterungen Dritter als Kernkomponente; keine E-Mail über Composio, Pipedream oder Zapier; kein archivierter MCP-Server; keine Workflow-Engine (Temporal, Trigger.dev, Inngest) und kein zweites Agenten-Framework (LangGraph, CrewAI) neben dem Agent SDK; keine Vektordatenbank im MVP; kein Langfuse; kein KI-Detektor als Gate (Kapitel 11.2); keine Anthropic-Dokumentskills im Renderpfad; kein Consumer-Abo-Token im Tageslauf; keine Claude-Code-Routine mit Schreib-Connector.

### 17.13 Minimal-Set für den MVP

Die zwölf Werkzeuge, ohne die der Durchstich aus Kapitel 19.4 nicht läuft. Reine Python-Abhängigkeiten (datasketch, bm25s, pydantic, Jinja2, pypdf, pdfplumber) stehen in `pyproject.toml` und zählen hier nicht als eigene Werkzeuge; Hosting (Hetzner CPX22, Ubuntu, systemd) und das E-Mail-Konto sind Betriebsvoraussetzungen.

| Nr. | Werkzeug | Rolle im MVP |
|---|---|---|
| 1 | Claude Code (CLI, Skills, Subagents, Hooks, Settings) | baut den Code; liefert die Definitionen, die das SDK lädt |
| 2 | Claude Agent SDK Python 0.2.152 | Rechercheur, Autor, Kritiker mit `canUseTool`, Hooks, Structured Output |
| 3 | `anthropic`-Python-SDK (Messages, Batch, Caching, Structured Outputs) | Extraktion, Judge, Fakten-Check, Klassifikation |
| 4 | Commercial-API-Key mit Fable 5.1, Opus 5, Sonnet 5, Haiku 4.5 | Modellzugang mit Ausgabenlimit; 30-Tage-Speicherung für Fable 5.1 aktiviert |
| 5 | Web Search und Web Fetch (Server-Tools) | Firmenrecherche mit `allowed_domains` und `max_uses` |
| 6 | BA-Jobsuche-API (plus Personio-, Greenhouse-, Lever-Feeds, Adzuna, Arbeitnow, Job-Alert-Mails als HTTP/IMAP-Adapter) | Quellen des Scouts, alle Grün |
| 7 | SQLite | Zustand, Status-Pipeline, `event_log` |
| 8 | Git, zwei private Repositories | Code getrennt von Profil und Bewerbungen |
| 9 | sops + age | Secrets als Umgebungsvariablen, nie im Kontext |
| 10 | WeasyPrint mit python-docx und Poppler-Utilities | Setzer und Setzer-QA |
| 11 | Apache Tika 4.0.0 und OpenResume-Parser | Test-Parsing des ATS-Prüfers |
| 12 | FastAPI + htmx + python-telegram-bot | Review-Cockpit und Push; Entwurf im Postfach über die Python-Standardbibliothek (`imaplib`) |

### 17.14 Default-Annahmen und offene Fragen

Bis du anders entscheidest (Kapitel 22): Kritiker auf Opus 5, Tracker-Klassifikation auf Haiku 4.5 mit Eskalation; keine bezahlten Recherche-APIs außer SerpAPI ab v1; Embeddings in v1 lokal mit BGE-M3, sofern der Server dafür vergrößert wird, sonst per API; Entgeltatlas erst in v1; keine Notion-Spiegelung; Playwright direkt statt Playwright MCP für den Co-Piloten; Managed Agents nur als v1-Test für den Rechercheur; Anthropic-Dokumentskills nur interaktiv.

**Quellen dieses Kapitels:**

- Anthropic: Pricing – https://platform.claude.com/docs/en/about-claude/pricing
- Anthropic: Prompt caching – https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Anthropic: Batch processing – https://platform.claude.com/docs/en/build-with-claude/batch-processing
- Anthropic: Structured outputs – https://platform.claude.com/docs/en/build-with-claude/structured-outputs
- Anthropic: Effort – https://platform.claude.com/docs/en/build-with-claude/effort
- Anthropic: API and data retention – https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
- Anthropic: Develop tests – https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
- Anthropic: Web search tool – https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool
- Anthropic: Web fetch tool – https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool
- Anthropic: Code execution tool – https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool
- Anthropic: Browser use tool – https://platform.claude.com/docs/en/agents-and-tools/tool-use/browser-use-tool
- Anthropic: Computer use tool – https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool
- Anthropic: Skills guide (container.skills) – https://platform.claude.com/docs/en/build-with-claude/skills-guide
- Anthropic: Files API – https://platform.claude.com/docs/en/build-with-claude/files
- Anthropic: PDF support – https://platform.claude.com/docs/en/build-with-claude/pdf-support
- Anthropic: Rate limits – https://platform.claude.com/docs/en/api/rate-limits
- Anthropic: Usage Policy – https://www.anthropic.com/aup
- Anthropic Managed Agents: Overview – https://platform.claude.com/docs/en/managed-agents/overview
- Anthropic Managed Agents: Scheduled deployments – https://platform.claude.com/docs/en/managed-agents/scheduled-deployments
- Anthropic Managed Agents: Budgets – https://platform.claude.com/docs/en/managed-agents/budgets
- Anthropic Managed Agents: Vaults – https://platform.claude.com/docs/en/managed-agents/vaults
- Anthropic Managed Agents: Memory – https://platform.claude.com/docs/en/managed-agents/memory
- Anthropic Managed Agents: Webhooks – https://platform.claude.com/docs/en/managed-agents/webhooks
- Anthropic Managed Agents: Cloud sandboxes reference – https://platform.claude.com/docs/en/managed-agents/cloud-sandboxes-reference
- Claude Code: Skills – https://code.claude.com/docs/en/skills
- Claude Code: Subagents – https://code.claude.com/docs/en/sub-agents
- Claude Code: Hooks – https://code.claude.com/docs/en/hooks
- Claude Code: Memory – https://code.claude.com/docs/en/memory
- Claude Code: MCP – https://code.claude.com/docs/en/mcp
- Claude Code: Plugins – https://code.claude.com/docs/en/plugins
- Claude Code: Security – https://code.claude.com/docs/en/security
- Claude Code: Headless – https://code.claude.com/docs/en/headless
- Claude Code: Scheduled tasks – https://code.claude.com/docs/en/scheduled-tasks
- Claude Code: Routines – https://code.claude.com/docs/en/routines
- Claude Code: Desktop scheduled tasks – https://code.claude.com/docs/en/desktop-scheduled-tasks
- Claude Agent SDK (PyPI) – https://pypi.org/project/claude-agent-sdk/
- Claude Agent SDK: Python – https://code.claude.com/docs/en/agent-sdk/python
- Claude Agent SDK: Custom tools – https://code.claude.com/docs/en/agent-sdk/custom-tools
- Claude Agent SDK: Subagents – https://code.claude.com/docs/en/agent-sdk/subagents
- Claude Agent SDK: Skills – https://code.claude.com/docs/en/agent-sdk/skills
- Claude Agent SDK: Permissions – https://code.claude.com/docs/en/agent-sdk/permissions
- Claude Agent SDK: Structured outputs – https://code.claude.com/docs/en/agent-sdk/structured-outputs
- Claude Agent SDK: Cost tracking – https://code.claude.com/docs/en/agent-sdk/cost-tracking
- Claude Agent SDK: Session storage – https://code.claude.com/docs/en/agent-sdk/session-storage
- Claude Agent SDK: Hosting – https://code.claude.com/docs/en/agent-sdk/hosting
- Claude Cowork – https://claude.com/product/cowork
- anthropics/skills – https://github.com/anthropics/skills
- anthropics/skills: docx SKILL.md – https://raw.githubusercontent.com/anthropics/skills/main/skills/docx/SKILL.md
- anthropics/skills: pdf SKILL.md – https://raw.githubusercontent.com/anthropics/skills/main/skills/pdf/SKILL.md
- anthropics/skills: doc-coauthoring – https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md
- anthropics/claude-plugins-official – https://github.com/anthropics/claude-plugins-official
- Playwright MCP – https://github.com/microsoft/playwright-mcp
- Playwright MCP README – https://raw.githubusercontent.com/microsoft/playwright-mcp/main/README.md
- Playwright Python: BrowserType – https://playwright.dev/python/docs/api/class-browsertype
- Firecrawl MCP – https://github.com/firecrawl/firecrawl-mcp-server
- Firecrawl – https://www.firecrawl.dev
- Firecrawl-Preise (scrapegraphai) – https://scrapegraphai.com/blog/firecrawl-pricing
- Exa MCP – https://github.com/exa-labs/exa-mcp-server
- Exa – https://exa.ai/
- Tavily MCP – https://github.com/tavily-ai/tavily-mcp
- Tavily – https://tavily.com/
- Brave Search MCP – https://github.com/brave/brave-search-mcp-server
- Brave Search API – https://brave.com/search/api/
- Brave: Free-Tier-Ende (implicator.ai) – https://www.implicator.ai/brave-drops-free-search-api-tier-puts-all-developers-on-metered-billing/
- Brave-Preise (agentdeals) – https://agentdeals.dev/vendor/brave-search-api
- Adzuna MCP – https://github.com/folathecoder/adzuna-job-search-mcp
- Google Workspace MCP (taylorwilsdon) – https://github.com/taylorwilsdon/google_workspace_mcp
- GongRzhe/Gmail-MCP-Server – https://github.com/GongRzhe/Gmail-MCP-Server
- Google: Gmail MCP-Server konfigurieren – https://developers.google.com/workspace/gmail/api/guides/configure-mcp-server
- Notion MCP – https://github.com/makenotion/notion-mcp-server
- Notion SDK – https://github.com/makenotion/notion-sdk-js
- Slack MCP (korotovsky) – https://github.com/korotovsky/slack-mcp-server
- Telegram MCP (chigwell) – https://github.com/chigwell/telegram-mcp
- Postgres MCP Pro – https://github.com/crystaldba/postgres-mcp
- Apify Actors MCP – https://github.com/apify/actors-mcp-server
- Browserbase MCP – https://github.com/browserbase/mcp-server-browserbase
- modelcontextprotocol/servers – https://github.com/modelcontextprotocol/servers
- neonwatty/job-apply-plugin – https://github.com/neonwatty/job-apply-plugin
- bundesAPI/jobsuche-api – https://github.com/bundesAPI/jobsuche-api
- bundesAPI/jobsuche-api openapi.yaml – https://raw.githubusercontent.com/bundesAPI/jobsuche-api/main/openapi.yaml
- bundesAPI/jobsuche-api Issues – https://github.com/bundesAPI/jobsuche-api/issues?q=is%3Aissue
- bundesAPI/entgeltatlas-api – https://github.com/bundesAPI/entgeltatlas-api
- bundesAPI/handelsregister – https://github.com/bundesAPI/handelsregister
- bundesAPI/deutschland – https://github.com/bundesAPI/deutschland
- Destatis: KldB 2010 – https://www.destatis.de/DE/Methoden/Klassifikationen/Berufe/klassifikation-berufe-kldb-2010.html
- ESCO – https://esco.ec.europa.eu/en/use-esco/download
- Personio: Jobs via XML – https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML
- Greenhouse Job Board API – https://developers.greenhouse.io/job-board.html
- Lever Postings API – https://github.com/lever/postings-api
- Recruitee Careers Site API – https://docs.recruitee.com/reference/intro-to-careers-site-api
- SmartRecruiters Posting API – https://developers.smartrecruiters.com/docs/posting-api
- Workday Source Guide (Community) – https://github.com/Francis1998/agentic-career-search/blob/main/docs/guides/WORKDAY_SOURCE_GUIDE.md
- Teamtailor Docs – https://docs.teamtailor.com/
- JOIN – https://join.com
- Google: JobPosting structured data – https://developers.google.com/search/docs/appearance/structured-data/job-posting
- Adzuna Developer Portal – https://developer.adzuna.com/
- Arbeitnow API – https://arbeitnow.com/api/job-board-api
- SerpAPI Google Jobs API – https://serpapi.com/google-jobs-api
- SerpAPI Pricing – https://serpapi.com/pricing
- SerpAPI-Preise (costbench) – https://costbench.com/software/web-scraping/serpapi/
- DataForSEO Google Jobs – https://dataforseo.com/pricing/serp/google-jobs-serp-api
- JSearch Pricing – https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch/pricing
- Jooble API – https://jooble.org/api/about
- Google Cloud Talent Solution – https://docs.cloud.google.com/talent-solution/job-search/v3/docs/basics
- StepStone Nutzungsbedingungen – https://www.stepstone.de/ueber-stepstone/nutzungsbedingungen-2022-03/
- Indeed Legal – https://www.indeed.com/legal
- LinkedIn: Verbotene Software – https://www.linkedin.com/help/linkedin/answer/a1341387
- Monster Terms of Use – https://www.monster.com/inside/terms-of-use
- Apify Indeed Scraper – https://apify.com/misceres/indeed-scraper
- Apify LinkedIn Jobs Scraper – https://apify.com/bebity/linkedin-jobs-scraper
- Apify StepStone Scraper – https://apify.com/jupri/stepstone-scraper
- python-jobspy (GitHub) – https://github.com/speedyapply/JobSpy
- python-jobspy (PyPI) – https://pypi.org/project/python-jobspy/
- Bright Data Jobs Scraper – https://brightdata.com/products/web-scraper/jobs-scraper
- ScraperAPI Pricing – https://www.scraperapi.com/pricing/
- softgarden Career Websites API – https://dev.softgarden.de/career-websites-api/jobs-api/
- § 5 DDG – https://www.gesetze-im-internet.de/ddg/__5.html
- handelsregister.de – https://www.handelsregister.de/
- Northdata – https://www.northdata.de/
- OpenCorporates – https://opencorporates.com/
- Unternehmensregister – https://www.unternehmensregister.de/
- Kununu – https://www.kununu.com/
- Glassdoor – https://www.glassdoor.de/
- Jina Reader – https://jina.ai/reader/
- Crawl4AI (GitHub) – https://github.com/unclecode/crawl4ai
- Crawl4AI (PyPI) – https://pypi.org/project/crawl4ai/
- Perplexity – https://www.perplexity.ai/
- webappanalyzer – https://github.com/enthec/webappanalyzer
- webappanalyzer categories.json – https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json
- BuiltWith – https://builtwith.com/
- Wappalyzer – https://www.wappalyzer.com/
- Nominatim API – https://nominatim.org/release-docs/latest/api/Overview/
- Google Places API – https://developers.google.com/maps/documentation/places/web-service
- Google-Maps-Preise (woosmap) – https://www.woosmap.com/blog/google-maps-api-pricing-breakdown
- Insolvenz-Radar – https://insolvenz-radar.de/funktionen/
- Apollo.io – https://www.apollo.io/
- Crunchbase – https://www.crunchbase.com/
- Dealroom – https://dealroom.co/
- Startbase – https://www.startbase.de/
- wlw – https://www.wlw.de/
- gehalt.de – https://www.gehalt.de/
- browser-use (GitHub) – https://github.com/browser-use/browser-use
- browser-use Pricing – https://browser-use.com/pricing
- Stagehand – https://github.com/browserbase/stagehand
- Skyvern – https://github.com/Skyvern-AI/skyvern
- Simplify Copilot – https://simplify.jobs/copilot
- LazyApply – https://www.lazyapply.com/
- Teal – https://www.tealhq.com/
- WeasyPrint – https://weasyprint.com/
- WeasyPrint-Doku: Common use cases – https://doc.courtbouillon.org/weasyprint/stable/common_use_cases.html
- python-docx – https://python-docx.readthedocs.io/
- pandoc – https://pandoc.org/
- Typst – https://github.com/typst/typst
- Typst Universe: modern-cv – https://typst.app/universe/package/modern-cv/
- Typst Universe: brilliant-cv – https://typst.app/universe/package/brilliant-cv/
- RenderCV – https://github.com/rendercv/rendercv
- moderncv (CTAN) – https://ctan.org/pkg/moderncv
- Tectonic – https://github.com/tectonic-typesetting/tectonic
- JSON Resume – https://jsonresume.org/
- Reactive Resume – https://github.com/amruthpillai/reactive-resume
- Puppeteer – https://pptr.dev/
- Paged.js – https://github.com/pagedjs/pagedjs/
- docxtemplater Pricing – https://docxtemplater.com/pricing/
- Carbone Pricing – https://carbone.io/pricing.html
- Apache Tika – https://github.com/apache/tika
- Apache Tika CHANGES.txt – https://raw.githubusercontent.com/apache/tika/main/CHANGES.txt
- OpenResume – https://github.com/xitanggg/open-resume
- pyresparser – https://github.com/OmkarPathak/pyresparser
- Eden AI: Resume Parser APIs – https://www.edenai.co/post/best-resume-parser-apis
- Affinda-Preise (G2) – https://www.g2.com/products/resume-parser-by-affinda/pricing
- Jobscan-Preise (pitchmeai) – https://pitchmeai.com/blog/jobscan-pricing-plans
- Resume Worded – https://resumeworded.com/
- Textkernel Parser – https://www.textkernel.com/de/produkte-loesungen/parser/
- Apple: iCloud Mail Limits – https://support.apple.com/en-us/102198
- Apple: iCloud+ Custom Email Domain – https://support.apple.com/en-us/102540
- Gmail API: Drafts – https://developers.google.com/workspace/gmail/api/guides/drafts
- Gmail API: Scopes – https://developers.google.com/workspace/gmail/api/auth/scopes
- Gmail API: Quota – https://developers.google.com/workspace/gmail/api/reference/quota
- Google Cloud: OAuth Testing – https://support.google.com/cloud/answer/15549945?hl=en
- Microsoft Graph: sendMail – https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0
- imap_tools – https://github.com/ikvk/imap_tools
- aiosmtplib – https://aiosmtplib.readthedocs.io/en/latest/usage.html
- icalendar – https://pypi.org/project/icalendar
- yagmail – https://github.com/kootenpv/yagmail
- Nodemailer – https://github.com/nodemailer/nodemailer
- ImapFlow – https://github.com/postalsys/imapflow
- Composio Agent Mail – https://composio.dev/toolkits/agent_mail
- Zapier MCP – https://zapier.com/mcp
- Fastmail Pricing – https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates
- mailbox.org Preise – https://mailbox.org/en/news/new-price-plans-available-mailboxorg/
- Proton Mail Pricing – https://proton.me/mail/pricing
- datasketch – https://github.com/ekzhu/datasketch
- bm25s – https://github.com/xhluca/bm25s
- rank_bm25 – https://github.com/dorianbrown/rank_bm25
- recordlinkage – https://github.com/J535D165/recordlinkage
- dedupe – https://github.com/dedupeio/dedupe
- sentence-transformers – https://github.com/UKPLab/sentence-transformers
- FlagEmbedding (BGE-M3) – https://github.com/FlagOpen/FlagEmbedding
- LanceDB – https://github.com/lancedb/lancedb
- rerankers – https://github.com/AnswerDotAI/rerankers
- Cohere Rerank 3.5 (OpenRouter) – https://openrouter.ai/cohere/rerank-v3.5
- Voyage AI Pricing – https://docs.voyageai.com/docs/pricing
- Jina Reranker – https://jina.ai/reranker/
- OpenAI Pricing – https://platform.openai.com/docs/pricing
- pgvector – https://github.com/pgvector/pgvector
- Qdrant – https://github.com/qdrant/qdrant
- spaCy de_core_news_lg 3.8.0 – https://github.com/explosion/spacy-models/releases/tag/de_core_news_lg-3.8.0
- MTEB Leaderboard – https://huggingface.co/spaces/mteb/leaderboard
- Hetzner Object Storage – https://www.hetzner.com/storage/object-storage/
- FastAPI – https://github.com/fastapi/fastapi
- htmx – https://github.com/bigskysoftware/htmx
- python-telegram-bot – https://github.com/python-telegram-bot/python-telegram-bot
- NiceGUI – https://github.com/zauberzeug/nicegui
- Reflex – https://github.com/reflex-dev/reflex
- Streamlit – https://github.com/streamlit/streamlit
- Gradio – https://github.com/gradio-app/gradio
- Obsidian Dataview – https://github.com/blacksmithgu/obsidian-dataview
- grammY – https://github.com/grammyjs/grammY
- Hetzner: Preisanpassung – https://docs.hetzner.com/de/general/infrastructure-and-availability/price-adjustment/
- Northflank: Hetzner price increases – https://northflank.com/blog/hetzner-cloud-server-price-increases
- Fly.io – https://fly.io/
- sops – https://github.com/getsops/sops
- macOS security CLI (ss64) – https://ss64.com/mac/security-password.html
- 1Password CLI: Secrets in scripts – https://developer.1password.com/docs/cli/secrets-scripts
- Infisical – https://github.com/Infisical/infisical
- Doppler – https://www.doppler.com/
- Arize Phoenix – https://github.com/Arize-ai/phoenix
- Langfuse Self-Hosting – https://langfuse.com/self-hosting
- n8n – https://github.com/n8n-io/n8n
- Temporal – https://github.com/temporalio/temporal
- Trigger.dev – https://github.com/triggerdotdev/trigger.dev
- Inngest – https://github.com/inngest/inngest
- LangGraph – https://github.com/langchain-ai/langgraph
- CrewAI – https://github.com/crewAIInc/crewAI
- APScheduler – https://github.com/agronholm/apscheduler
