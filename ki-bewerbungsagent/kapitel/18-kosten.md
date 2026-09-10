## 18. Kosten: pro Bewerbung, pro Monat; Infrastruktur; Datenquellen; Sparhebel

Für den Einzelnutzer-Fall (10 Bewerbungen/Tag, 22 Arbeitstage/Monat = 220 Bewerbungen/Monat) sind die Betriebskosten in jedem realistischen Szenario unkritisch: Sie liegen zwischen rund 260 € und rund 1.150 € im Monat, je nach Modellwahl und Gründlichkeit. Modell-Tokens dominieren die Rechnung, nicht Infrastruktur oder Datenquellen. Dieses Kapitel legt das Rechenmodell offen (Annahmen, Formel, Zahlen), rechnet drei Szenarien durch und benennt die wirksamsten Sparhebel.

### 18.1 Annahmen und Rechenweg

**Modellpreise** (bestätigt, Stand September 2026, pro 1 Mio. Token Input/Output; Rollen- und Modellzuordnung je Pipeline-Schritt siehe Kapitel 7.6 und 17.2.1):

| Modell | Input | Output | Cache-Read | Cache-Write (5 Min / 1 Std.) |
|---|---|---|---|---|
| Claude Fable 5.1 | $10/MTok | $50/MTok | 0,025× Input | 1,25× / 2× Input |
| Claude Opus 5 | $5/MTok | $25/MTok | 0,1× Input | 1,25× / 2× Input |
| Claude Sonnet 5 | $2/MTok | $10/MTok | 0,1× Input | 1,25× / 2× Input |
| Claude Haiku 4.5 | $1/MTok | $5/MTok | 0,1× Input | 1,25× / 2× Input |

Dazu: Web-Suche (Server-Tool) $10 pro 1.000 Suchen zzgl. normaler Tokenkosten der Ergebnisse; fehlgeschlagene Suchen werden nicht berechnet. Web-Fetch ist **kostenlos** (nur Tokenkosten des abgerufenen Inhalts) – eine Korrektur des Faktenprüfers gegenüber der ursprünglichen Annahme, Web-Fetch koste ebenfalls $10/1.000. Die Batch-API gewährt 50 % Rabatt auf Input- und Output-Tokenpreise, gilt aber **nicht** für Web-Suche und nicht für Managed Agents [Anthropic-Preisliste](https://platform.claude.com/docs/en/about-claude/pricing), [Prompt Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching), [Batch-Processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing), [Web-Search-Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool).

Weitere Annahmen:
- **Wechselkurs:** 1 USD ≈ 0,93 EUR (Annahme, schwankt; vor Umsetzung aktuellen Kurs einsetzen).
- **Tokenizer-Korrektur:** Modelle ab Claude 4.7 (inkl. Sonnet 5, Opus 5, Fable 5.x) erzeugen laut Faktenprüfer für denselben Text ca. 30 % mehr Tokens als ältere Modelle – in allen folgenden Zahlen bereits eingerechnet.
- **Volumen:** 10 vollständig bearbeitete Bewerbungen/Tag, 22 Arbeitstage/Monat = 220 Bewerbungen/Monat. Zusätzlich scannt der Scout täglich mehr Anzeigen, als am Ende bearbeitet werden (Annahme: 100 gescannte Anzeigen/Tag, um daraus 10 auszuwählen – hängt von Zielrolle, Branche und Ort ab, siehe Kapitel 22).
- **Token- und Kostenschätzung pro Schritt ist eine eigene, nicht gemessene Architektur-Annahme** (kein externes Benchmark für vergleichbare Agenten-Pipelines auffindbar). **Entscheidung:** Vor dem Produktivbetrieb die tatsächlichen `response.usage`-Werte aus einem Mini-Prototyp (eine reale Bewerbung, Ende-zu-Ende) messen und dieses Kapitel damit kalibrieren. **Begründung:** Geschätzte Token liegen in der Praxis leicht daneben, besonders bei der Recherche-Tiefe. **Alternative:** ohne Nachmessen starten und das erste Monatsbudget als Puffer großzügig ansetzen.

Rechenweg pro Bewerbung:

```
Kosten pro Bewerbung =
    Σ über die Modellschritte der Pipeline (Kapitel 7.6):
        Rechercheur; Autor (Briefing/Entwürfe/Überarbeitung);
        Kritiker (Rubrik/Fakten-Check/Stimm-Leser-Test/Konsistenz-Check);
        ATS-Prüfer (Begriffsextraktion/Lesetest)
      → je Schritt: (Input-Token / 1.000.000 × Modellpreis-Input)
                  + (Output-Token / 1.000.000 × Modellpreis-Output)
                  + (Websuchen im Schritt × 0,01 USD)
    + Setzer: 0 USD (Code ohne Modell, Kapitel 13.1)

Kosten pro Monat (Bewerbungen)   = Kosten pro Bewerbung × 10 × 22
Kosten pro Monat (Scout+Matcher) = Kosten pro gescannter Anzeige × Anzeigen/Tag × 22
Gesamt pro Monat = Modellkosten (Bewerbungen) + Scout/Matcher + Hosting + E-Mail + Datenquellen
```

### 18.2 Kosten pro Bewerbung: drei Szenarien

Die Pipeline pro ausgewählter Bewerbung durchläuft Rechercheur, Autor, Kritiker, ATS-Prüfer und Setzer (Kapitel 10–13). Modell, effort und Rollentrennung je Teilschritt folgen der verbindlichen Zuordnung aus Kapitel 7.6: Fable 5.1 nur für die beiden qualitätskritischen Schritte Rechercheur und Autor, Opus 5 getrennt davon als Grader-Modell für den Kritiker (kein zusätzlicher Zweitgutachter), Sonnet 5 für die mittelschweren Einzelaufrufe (Autor-Briefing, Fakten-Check, ATS-Lesetest), Haiku 4.5 für mechanische Schritte. Der Setzer bleibt in allen drei Szenarien Code ohne Modellkosten (Kapitel 13.1). Scout und Matcher laufen separat als Massen-Scan über alle gescannten Anzeigen (18.3).

**Szenario „sparsam"** – Konfigurationsschalter `MODEL_TOP=claude-opus-5` (Kapitel 7.6): Rechercheur und Autor laufen auf dem günstigeren, ZDR-fähigen Opus 5 statt auf Fable 5.1, ein Entwurf, eine reduzierte Kritikrunde, 3 Websuchen, mechanische Schritte auf Haiku 4.5:

| Schritt | Modell (effort) | Input-Token | Output-Token | Websuchen | Kosten |
|---|---|---|---|---|---|
| Rechercheur | Opus 5 (high) | 55.000 | 9.000 | 3 | $0,530 |
| Autor: Briefing | Sonnet 5 (medium) | 3.000 | 1.200 | 0 | $0,018 |
| Autor: Entwurf (1) | Opus 5 (high) | 9.000 | 11.000 | 0 | $0,320 |
| Autor: Überarbeitung/Tailoring | Opus 5 (medium) | 3.000 | 2.500 | 0 | $0,078 |
| Kritiker: Rubrik | Opus 5 (high) | 7.000 | 1.800 | 0 | $0,080 |
| Kritiker: Fakten-Check | Sonnet 5 (medium) | 3.500 | 1.000 | 0 | $0,017 |
| Kritiker: Stimm-/Leser-Test | Opus 5 (medium) | 5.000 | 1.200 | 0 | $0,055 |
| Kritiker: Konsistenz-Check | Haiku 4.5 (low) | 2.500 | 300 | 0 | $0,004 |
| ATS-Prüfer: Begriffsextraktion | Haiku 4.5 | 6.000 | 1.500 | 0 | $0,014 |
| ATS-Prüfer: Lesetest | Sonnet 5 | 5.000 | 1.500 | 0 | $0,025 |
| **Summe** | | **99.000** | **31.000** | **3** | **$1,141 ≈ 1,06 €** |

`MODEL_TOP` schaltet Rechercheur und Autor auf dasselbe Modell wie den Kritiker um; die Trennung von Autor- und Grader-Modell (Kapitel 7.6) entfällt in diesem Szenario zugunsten von Kosten und Zero-Data-Retention.

**Szenario „empfohlen" (Standard)** – Rechercheur und Autor auf Fable 5.1 (Wunschbasis des Nutzers für die qualitätskritischen Schritte Recherche und Schreiben, Kapitel 7.6), Kritiker als vom Autor getrenntes Grader-Modell auf Opus 5, zwei Entwürfe, 6 Websuchen, mechanische Schritte auf Haiku 4.5:

| Schritt | Modell (effort) | Input-Token | Output-Token | Websuchen | Kosten |
|---|---|---|---|---|---|
| Rechercheur | Fable 5.1 (high) | 85.000 | 15.000 | 6 | $1,660 |
| Autor: Briefing | Sonnet 5 (medium) | 4.000 | 1.500 | 0 | $0,023 |
| Autor: Entwürfe (2) | Fable 5.1 (high) | 16.000 | 22.000 | 0 | $1,260 |
| Autor: Überarbeitung/Tailoring | Fable 5.1 (medium) | 6.000 | 8.000 | 0 | $0,460 |
| Kritiker: Rubrik | Opus 5 (high) | 12.000 | 3.000 | 0 | $0,135 |
| Kritiker: Fakten-Check | Sonnet 5 (medium) | 5.000 | 1.500 | 0 | $0,025 |
| Kritiker: Stimm-/Leser-Test | Opus 5 (medium) | 8.000 | 2.000 | 0 | $0,090 |
| Kritiker: Konsistenz-Check | Haiku 4.5 (low) | 3.000 | 400 | 0 | $0,005 |
| ATS-Prüfer: Begriffsextraktion | Haiku 4.5 | 8.000 | 2.500 | 0 | $0,021 |
| ATS-Prüfer: Lesetest | Sonnet 5 | 7.000 | 2.000 | 0 | $0,034 |
| **Summe** | | **154.000** | **57.900** | **6** | **$3,713 ≈ 3,45 €** |

**Szenario „maximal"** – wie „empfohlen", aber drei Entwürfe, mehrere Kritikrunden (Rubrik und Stimm-/Leser-Test je erneut), 8 Websuchen – die technische Obergrenze `max_uses` des Rechercheur-Subagents (Kapitel 7.6, 10.3):

| Schritt | Modell (effort) | Input-Token | Output-Token | Websuchen | Kosten |
|---|---|---|---|---|---|
| Rechercheur | Fable 5.1 (high) | 100.000 | 18.000 | 8 | $1,980 |
| Autor: Briefing | Sonnet 5 (medium) | 5.000 | 1.800 | 0 | $0,028 |
| Autor: Entwürfe (3) | Fable 5.1 (high) | 24.000 | 34.000 | 0 | $1,940 |
| Autor: Überarbeitung/Tailoring (mehrere Runden) | Fable 5.1 (medium) | 10.000 | 14.000 | 0 | $0,800 |
| Kritiker: Rubrik (mehrere Runden) | Opus 5 (high) | 20.000 | 5.000 | 0 | $0,225 |
| Kritiker: Fakten-Check | Sonnet 5 (medium) | 7.000 | 2.000 | 0 | $0,034 |
| Kritiker: Stimm-/Leser-Test | Opus 5 (medium) | 12.000 | 3.000 | 0 | $0,135 |
| Kritiker: Konsistenz-Check | Haiku 4.5 (low) | 4.000 | 500 | 0 | $0,007 |
| ATS-Prüfer: Begriffsextraktion | Haiku 4.5 | 10.000 | 3.000 | 0 | $0,025 |
| ATS-Prüfer: Lesetest | Sonnet 5 | 9.000 | 2.500 | 0 | $0,043 |
| **Summe** | | **201.000** | **83.800** | **8** | **$5,217 ≈ 4,85 €** |

Der Setzer trägt in allen drei Szenarien 0 USD Modellkosten bei (Kapitel 13.1); seine Rechenzeit ist Teil der Hosting-Kosten (18.5). Optional prüft Claude Haiku 4.5 die gerenderte PDF-Vorschau auf sichtbare Layoutfehler (Setzer-QA, Kapitel 13.10) – bei rund 1.500 Input-/50 Output-Token je geprüfter Seite und 220 Bewerbungen/Monat sind das zusätzliche ≈ 0,4 €/Monat, vernachlässigbar gegenüber den übrigen Posten.

Hochgerechnet auf 220 Bewerbungen/Monat (10 × 22 Tage): sparsam ≈ **233 €**, empfohlen ≈ **760 €**, maximal ≈ **1.067 €** – reine Modellkosten der Bewerbungs-Pipeline, ohne Massen-Scan und Infrastruktur.

### 18.3 Massen-Scan: Scout und Matcher

Scout (Extraktion je Anzeige) und Matcher (Scoring gegen das Kandidatenprofil) laufen über deutlich mehr Anzeigen, als am Ende bearbeitet werden – hier lohnt sich Haiku 4.5 kombiniert mit der Batch-API (nicht-interaktiver Nachtlauf, 50 % Rabatt). Annahme: ~3.000 Input-/400 Output-Token je Scout-Extraktion, ~2.000 Input-/300 Output-Token je Matcher-Bewertung (Kandidatenprofil größtenteils gecacht), 100 gescannte Anzeigen/Tag:

- Pro gescannter Anzeige: $0,00425.
- Pro Monat (100 Anzeigen/Tag × 22 Tage = 2.200 Anzeigen): $9,35 ≈ **8,70 €**.

Selbst bei dreimal so vielen gescannten Anzeigen (300/Tag, größerer Suchradius oder Branche mit viel Angebot) bleibt dieser Posten unter 30 €/Monat – der Massen-Scan ist wegen Haiku 4.5 + Batch nahezu vernachlässigbar gegenüber den 10 tatsächlich bearbeiteten Bewerbungen.

### 18.4 Websuche, Code-Execution, Datenquellen

Die Websuchkosten sind bereits in den Rechercheur-Zeilen aus 18.2 enthalten (3/6/8 Suchen × $0,01 × 220 Bewerbungen ≈ 6 € / 12 € / 16 € pro Monat); 8 Suchen je Bewerbung ist zugleich die technische Obergrenze, die `max_uses` im Rechercheur-Subagent erzwingt (Kapitel 7.6, 10.3). Code-Execution (für die Skills `docx`/`pdf`, siehe Kapitel 13 und 17) ist kostenlos, solange es zusammen mit Web-Search/Web-Fetch läuft (aktuelle Toolversionen) oder innerhalb der 1.550 Freistunden/Organisation/Monat bleibt – bei 10 Dokumentensätzen/Tag bei Weitem ausreichend [Code-Execution-Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool).

Datenquellen (Details und Vergleichsmatrix in Kapitel 6): Bundesagentur-für-Arbeit-Jobsuche-API und Adzuna sind kostenlos und sollten die Kernquellen sein; die öffentlichen ATS-Feeds (Personio, Greenhouse, Lever, Recruitee, SmartRecruiters) sind ebenfalls kostenlos und unauthentifiziert [BA-Jobsuche-API](https://github.com/bundesAPI/jobsuche-api), [Adzuna](https://developer.adzuna.com/). Bei 10 Bewerbungen/Tag reicht das voraussichtlich aus – **Kostenannahme: 0 €/Monat für Datenquellen im MVP.** Falls Kontingente nicht reichen (z. B. Google-for-Jobs-Zugriff für breitere Marktabdeckung), ist SerpAPI mit 250 kostenlosen Suchen/Monat, danach ab 25 $/Monat für 1.000 Suchen, die günstigste geprüfte Ergänzung [SerpAPI-Preise](https://serpapi.com/pricing); Firecrawl bietet für punktuelles Abrufen einzelner Karriereseiten einen Free-Tier mit 1.000 Credits/Monat, danach ab 16 $/Monat [Firecrawl-Preise](https://scrapegraphai.com/blog/firecrawl-pricing). Exa und Tavily wurden ebenfalls als günstige Ergänzungen recherchiert, ihre genauen 2026er-Preise konnten in dieser Recherche aber nicht zweifelsfrei an den Originalquellen bestätigt werden (unbestätigt) – vor Nutzung direkt bei den Anbietern prüfen. Bright Data und JSearch sind bei diesem Volumen überdimensioniert und teurer. Brave Search hat den kostenlosen Tier im Februar 2026 abgeschafft und rechnet seitdem guthabenbasiert ab ($5 je 1.000 Anfragen, Kreditkarte Pflicht); jeder Plan enthält aber $5 Gratis-Guthaben pro Monat (≈ 1.000 Suchen) – für das hier angenommene Volumen (18.1) kann dieses Monatsguthaben bereits ausreichen [Brave: Free-Tier-Ende](https://www.implicator.ai/brave-drops-free-search-api-tier-puts-all-developers-on-metered-billing/), [Brave-Preise](https://agentdeals.dev/vendor/brave-search-api).

### 18.5 Infrastruktur: Hosting und E-Mail

**Entscheidung:** Hetzner-VPS CPX22 (2 vCPU/4 GB/80 GB) als Standard-Hosting für den Orchestrator (Cron/systemd-Timer, Agent-SDK-Prozess, siehe Kapitel 7), rund 20 €/Monat (19,49–19,99 € je nach Quelle; vor Bestellung im Hetzner-Konfigurator prüfen; Stand September 2026, nach der vierten Preisanpassung des Jahres am 15.6.2026) [Hetzner-Preisanpassung](https://docs.hetzner.com/general/infrastructure-and-availability/price-adjustment/), [Hetzner-Preisvergleich](https://northflank.com/blog/hetzner-cloud-server-price-increases). Ob die früher günstigeren CX23/CAX11-Linien für Neubestellungen noch verfügbar sind, ließ sich in der Recherche nicht belegen (unbestätigt). **Begründung:** bestes Preis-Kontrolle-Verhältnis für einen dauerhaft laufenden Ein-Personen-Server in der EU. **Alternative:** Anthropic Managed Agents (Scheduled Deployments) statt eigenem Server – $0,08 pro aktiver Session-Stunde, keine Grundgebühr; bei realistisch 30–60 aktiven Minuten/Tag ergibt das nur rund 1–2 $/Monat, ist aber Beta und bringt eine neue, sich noch ändernde API mit sich [Managed-Agents-Budgets](https://platform.claude.com/docs/en/managed-agents/budgets). Details zur Architekturentscheidung in Kapitel 7.

E-Mail-Versand über das bestehende private Konto des Kandidaten (iCloud, Gmail oder Outlook) kostet 0 €/Monat und reicht für 10 Bewerbungen/Tag bei Weitem (Details Kapitel 15). Eine eigene Bewerbungsdomain für einen professionelleren Auftritt ist optional: mailbox.org Standard ab ca. 3 €/Monat (deutscher, DSGVO-fokussierter Anbieter) oder Fastmail Individual ab 5–6 $/Monat [mailbox.org-Preise](https://mailbox.org/en/news/new-price-plans-available-mailboxorg/), [Fastmail-Preise](https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates). **Default-Annahme:** bestehendes Konto nutzen, 0 €/Monat; offene Frage an dich in Kapitel 22.

### 18.6 Gesamtkosten pro Monat: Zusammenfassung

| Szenario | Modelle (Bewerbungen) | Scout/Matcher | Hosting | E-Mail/Daten | Gesamt/Monat |
|---|---|---|---|---|---|
| sparsam | 233 € | 9 € | 20 € | 0 € | **≈ 262 €** |
| empfohlen | 760 € | 9 € | 20 € | 0 € | **≈ 789 €** |
| maximal | 1.067 € | 35 € | 20 € | 29 € | **≈ 1.151 €** |

Die Maximal-Spalte enthält zusätzlich eine eigene Domain (~6 €) und einen SerpAPI-Einstiegstarif (~23 €) sowie ein höheres Scan-Volumen bei Scout/Matcher, weil das Szenario bewusst keine Sparhebel zieht. **Entscheidung:** „empfohlen" (Fable 5.1 für Rechercheur und Autor, Opus 5 als vom Autor getrenntes Grader-Modell für den Kritiker, Sonnet 5 für die mittelschweren Schritte Briefing/Fakten-Check/ATS-Lesetest, Haiku 4.5 für mechanische Schritte) als Standard-Betriebsmodus. **Begründung:** Diese Zuordnung folgt der verbindlichen Modellwahl aus Kapitel 7.6 – Fable 5.1 ist die Wunschbasis des Nutzers für die qualitätskritischen Schritte Recherche und Schreiben, der Kritiker bewertet als eigenständiges, vom Autor getrenntes Modell statt als zusätzlicher Zweitgutachter. Rund 789 €/Monat sind für ein Einzelprojekt unkritisch, und die Modellwahl erhält die Qualitätssicherung an der entscheidenden Stelle (Anti-Generik-Check vor Versand, Kapitel 11). **Alternative:** „sparsam" per Konfigurationsschalter `MODEL_TOP=claude-opus-5` (Rechercheur und Autor laufen dann auf dem günstigeren, ZDR-fähigen Opus 5 statt auf Fable 5.1 – zugleich die richtige Wahl ohne 30-Tage-Datenspeicherung bei Anthropic, Kapitel 7.6/16.2) bei explizitem Kostenlimit; „maximal" bei mehr Entwürfen und Kritikrunden, wenn du die höheren Kosten akzeptierst.

**Daraus abgeleitet: Default-Budget je Tageslauf (Teil 2).** Die Pro-Bewerbung-Kosten aus 18.2 ergeben für einen vollständigen Tageslauf von zehn Bewerbungen (Teil 2, Schreiben und Setzen): im Szenario „empfohlen" 10 × 3,713 USD ≈ **37 USD**, im Szenario „maximal" 10 × 5,217 USD ≈ **52 USD**. **Entscheidung:** Das technisch durchgesetzte Default-Budget für Teil 2 ist **45 USD je Tageslauf** – es deckt „empfohlen" mit spürbarem Puffer (für Ausreißer wie zusätzliche Kritikrunden oder etwas höheren Tokenverbrauch), ohne einen vollen „maximal"-Lauf ungebremst durchzulassen. **Begründung:** Ein fester Dollar-Deckel je Lauf ist die harte, technische Leitplane, die verhindert, dass ein einzelner Tageslauf durch Endlosschleifen oder zu viele Websuchen aus dem Ruder läuft, unabhängig vom Wechselkurs oder von Tagesschwankungen einzelner Bewerbungen. **Alternative:** Wer dauerhaft im Szenario „maximal" arbeitet, hebt den Wert in `config/zeitplan.yaml` entsprechend an (z. B. auf 55–60 USD); dieser Deckel gilt identisch für die systemd-Unit/Crontab und die Kostenkontrolle in Kapitel 7.7.7 sowie für die Referenzen in Kapitel 19 und 22. Getrennt davon bleibt das kleinere Budget von 5 USD für den ersten echten Versandlauf in Woche 5 (Kapitel 19) unverändert – das ist ein anderer, bewusst kleiner Testlauf, kein Widerspruch zum 45-USD-Default für den laufenden Betrieb.

### 18.7 Sparhebel

1. **Prompt-Caching**: Kandidatenprofil, Story-Bank, Stimmprofil und System-Prompt als stabilen Cache-Präfix (1-Stunden-TTL) vor die variablen Tagesdaten legen. Rechenbeispiel für einen 8.000-Token-Präfix, genutzt in drei Schritten × 10 Bewerbungen/Tag (30 Zugriffe) auf Sonnet 5: ohne Caching $0,48/Tag, mit Caching (1 Schreibvorgang + 29 Lesevorgänge) nur $0,078/Tag – 84 % Ersparnis auf diesem Anteil. Da der Cache-Präfix aber nur einen Teil des Gesamt-Inputs ausmacht (der variable Anteil aus Websuchergebnissen und Recherchefunden lässt sich kaum cachen), sinkt die Gesamtmonatsrechnung dadurch realistisch um niedrige zweistellige Prozentpunkte, nicht um 80–90 %. Bei Fable 5.1 ist der Cache-Read-Rabatt mit 0,025× sogar noch größer als bei den anderen Modellen.
2. **Batch-API** für den nicht-interaktiven Massen-Scan (Scout + Matcher): 50 % Rabatt, bereits in 18.3 eingerechnet. Gilt nicht für die interaktive Autor-/Kritiker-Schleife mit Freigabeschritt, da Batch-Ergebnisse asynchron (typisch unter 24 Stunden) zurückkommen.
3. **Modell-Mix statt Einheitsmodell**: Haiku 4.5 für mechanische Schritte (Konsistenz-Check, ATS-Begriffsextraktion, Massen-Scan), Sonnet 5 für die mittelschweren Einzelaufrufe (Autor-Briefing, Fakten-Check, ATS-Lesetest), Fable 5.1 nur für die beiden qualitätskritischen Schritte Rechercheur und Autor, Opus 5 getrennt davon für den Kritiker als Grader-Modell (Kapitel 7.6). Würde man im „empfohlen"-Szenario die beiden Opus-5-Schritte des Kritikers (Rubrik-Bewertung und Stimm-/Leser-Test) durch Sonnet 5 ersetzen, sänke die Monatsrechnung um rund 28 € – der Sparhebel ist real, kostet aber genau die vom Autor getrennte Qualitätssicherung, die der Leitsatz „Qualität vor Quantität" verlangt (Kapitel 1). Größere Wirkung hat der Konfigurationsschalter `MODEL_TOP=claude-opus-5` (Basis von „sparsam", 18.2): Er ersetzt Fable 5.1 bei Rechercheur und Autor durch das günstigere Opus 5 und senkt die Monatsrechnung allein dadurch um mehrere Hundert Euro, hebt dabei aber die Trennung zwischen Autor- und Grader-Modell auf.
4. **Weniger Entwürfe/Kritikrunden**: ein Entwurf statt zwei oder drei spart in der Autor-/Kritiker-Stufe rund ein Drittel bis die Hälfte der dortigen Kosten (Vergleich sparsam vs. empfohlen vs. maximal in 18.2).
5. **Wiederverwendung von Dossiers**: Recherchiert der Rechercheur eine Firma bereits für eine Bewerbung, kostet eine zweite Bewerbung an dieselbe Firma (andere Stelle) nur noch einen Aktualitätscheck statt einer vollständigen Neu-Recherche – spart auf diesen Fall bezogen bis zu 80–90 % der Rechercheur-Kosten. Kandidatenprofil, Story-Bank und Stimmprofil sind ohnehin über alle Bewerbungen stabil und Teil des Cache-Präfix (Hebel 1).
6. **Ghost-Job-Filter vor der teuren Recherche**: 18–38 % aller Online-Stellenanzeigen gelten laut mehreren 2025er-Quellen als Ghost Jobs [Ghost-Jobs-Studie](https://unternehmer.de/wirtschaft/625515-ghost-jobs-jede-dritte-stellenanzeige-betroffen). Ein Plausibilitäts-/Frische-Check im Matcher, bevor eine Anzeige den teuren Rechercheur- und Autor-Schritt durchläuft, spart entsprechend Token-Budget (Details Kapitel 9).
7. **Harte Budget-Deckel technisch erzwingen**: `max_budget_usd` im Agent SDK bzw. das Session-Budget (`max_list_cost`, in US-Cent) bei Managed Agents verhindern Kostenausreißer durch Endlosschleifen oder zu viele Websuchen pro Bewerbung [Managed-Agents-Budgets](https://platform.claude.com/docs/en/managed-agents/budgets). Konkret gesetzt auf 45 USD je Tageslauf (Teil 2), aus dem Szenario „empfohlen" abgeleitet und mit Puffer versehen (18.6). Details zur Umsetzung in Kapitel 7 und 19.

### 18.8 Einordnung: Vergleich mit Wettbewerbern und Coaching

| Angebot | Art | Preis | Einordnung |
|---|---|---|---|
| Bewerbungsagent (empfohlen) | Eigenbau, KI + Freigabe | ≈ 789 €/Monat (≈ 3,59 €/Bewerbung) | volle Recherche + Individualisierung, Mensch prüft jede Bewerbung |
| Jobscan | SaaS, ATS-Keyword-Scan | 49,95 $/Monat (29,98 $/Monat quartalsweise) | nur Abgleich, kein Schreiben, kein Versand |
| Teal+ | SaaS, Tracker | ≈ 29 $/Monat | kein Auto-Apply, reines Tracking |
| Kickresume | SaaS, CV-Builder mit KI | 24 $/Monat bzw. 96 $/Jahr | Dokument-Tool, keine Recherche/Versand |
| LazyApply | SaaS, Volumen-Auto-Apply | 99–999 $/Jahr | Massenversand ohne Individualisierung, 2,1/5 Trustpilot |
| Bewerbung-Schreiber.com | DE, Mensch+KI-Hybrid | 99–199 €/Anschreiben | pro Bewerbung teurer als der Agent für einen ganzen Monat |
| erfolgo.de | DE, reiner KI-Generator | 9,95 € Pauschale | günstig, aber mindestens ein dokumentierter Halluzinationsfall (Qualitätsrisiko, niedrige Beleglage) |

Quellen: [Jobscan](https://www.jobscan.co/), [Teal](https://www.tealhq.com/), [Kickresume](https://www.kickresume.com/), [LazyApply-Bewertung](https://www.loopcv.pro/directory/lazyapply/), [Bewerbung-Schreiber.com](https://bewerbung-schreiber.com/), [erfolgo.de](https://erfolgo.de/).

Der Bewerbungsagent kostet im „empfohlen"-Szenario pro Bewerbung (≈ 3,59 €) einen Bruchteil dessen, was ein deutscher Mensch+KI-Hybrid-Dienst für ein einzelnes Anschreiben verlangt (99–199 €) – bei geringerem Automatisierungsgrad im Versand (Human-in-the-Loop bleibt Pflicht, Kapitel 14) und ohne den Anspruch, menschliches Coaching vollständig zu ersetzen. Marktübliche Stundensätze deutscher Bewerbungscoaches (grob 80–200 €/Stunde bzw. 300–1.500 € für ein komplettes Bewerbungspaket) konnten in der Recherche nicht live verifiziert werden und sind **unbestätigt** – sie dienen nur als grobe Orientierung, dass bereits ein bis zwei Wochen reiner Modellkosten des Agentenbetriebs (7–14 Bewerbungen, ca. 24–48 €) günstiger sind als eine einzelne Coaching-Stunde, ohne dass die Qualität eins zu eins vergleichbar wäre.

### Offene Fragen

- **Monatliches Kostenlimit**: Gibt es eine harte Obergrenze, die der Agent (per `max_budget_usd`/Session-Budget) technisch durchsetzen soll? Default-Annahme: kein hartes Monatslimit; technisch durchgesetzt wird stattdessen das Default-Budget von 45 USD je Tageslauf (18.6, Kapitel 7.7.7), das „empfohlen" mit Puffer deckt. Das „empfohlen"-Szenario (~789 €/Monat) dient als Erwartungswert und Ausgangspunkt für die Monatsrechnung, nicht als hart durchgesetzter Deckel.
- **Scan-Volumen**: Wie viele Anzeigen/Tag realistisch gescannt werden, hängt von Zielrolle, Branche, Ort und Suchradius ab (unbekannt, siehe Kapitel 22). Default-Annahme dieses Kapitels: 100/Tag; der Kostenanteil bleibt auch bei deutlich mehr Anzeigen gering.
- **Eigene E-Mail-Domain**: bestehendes Konto (0 €) oder eigene Domain (3–6 €/Monat) für einen professionelleren Auftritt? Default-Annahme: bestehendes Konto.
- **Zahlungsbereitschaft für Datenquellen**, falls Freikontingente (BA-API, Adzuna, Firecrawl/SerpAPI-Free-Tier) nicht reichen. Default-Annahme: kostenlose Quellen zuerst ausreizen, erst bei nachgewiesenem Bedarf auf bezahlte Stufen wechseln.

**Quellen dieses Kapitels:**
- [Anthropic – Preisübersicht Modelle](https://platform.claude.com/docs/en/about-claude/pricing)
- [Anthropic – Prompt Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Anthropic – Batch Processing (Message Batches API)](https://platform.claude.com/docs/en/build-with-claude/batch-processing)
- [Anthropic – Web-Search-Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)
- [Anthropic – Code-Execution-Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool)
- [Anthropic – Managed Agents: Budgets](https://platform.claude.com/docs/en/managed-agents/budgets)
- [Bundesagentur für Arbeit Jobsuche API (bundesAPI)](https://github.com/bundesAPI/jobsuche-api)
- [Adzuna Developer](https://developer.adzuna.com/)
- [SerpAPI – Preise](https://serpapi.com/pricing)
- [Firecrawl-Preise (Drittquelle)](https://scrapegraphai.com/blog/firecrawl-pricing)
- [Hetzner – Preisanpassung Cloud](https://docs.hetzner.com/general/infrastructure-and-availability/price-adjustment/)
- [Hetzner-Preisvergleich (Northflank)](https://northflank.com/blog/hetzner-cloud-server-price-increases)
- [Brave: Free-Tier-Ende (implicator.ai)](https://www.implicator.ai/brave-drops-free-search-api-tier-puts-all-developers-on-metered-billing/)
- [Brave-Preise (agentdeals)](https://agentdeals.dev/vendor/brave-search-api)
- [mailbox.org – neue Preispläne](https://mailbox.org/en/news/new-price-plans-available-mailboxorg/)
- [Fastmail – Preise](https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates)
- [Ghost-Jobs-Studie (unternehmer.de)](https://unternehmer.de/wirtschaft/625515-ghost-jobs-jede-dritte-stellenanzeige-betroffen)
- [Jobscan](https://www.jobscan.co/)
- [Teal](https://www.tealhq.com/)
- [Kickresume](https://www.kickresume.com/)
- [LazyApply-Bewertung (LoopCV-Verzeichnis)](https://www.loopcv.pro/directory/lazyapply/)
- [Bewerbung-Schreiber.com](https://bewerbung-schreiber.com/)
- [erfolgo.de](https://erfolgo.de/)
