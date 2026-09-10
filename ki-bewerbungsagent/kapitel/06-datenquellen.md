## 6. Datenquellen: Jobbörsen, Karriereseiten, APIs und Zugriffswege

Der Scout kann nur so gut sein wie seine Quellen. Für Deutschland gibt es keine einzige Schnittstelle, die alle relevanten Stellenanzeigen liefert: Die großen Publikumsbörsen (StepStone, Indeed, LinkedIn, XING/onlyfy, Monster, Glassdoor, Jobware, stellenanzeigen.de, meinestadt.de, Kimeta) bieten ausschließlich Schnittstellen für Arbeitgeber, um Anzeigen *hinein* zu schalten, aber keine öffentliche Lese-API für Dritte (Belege je Quelle in 6.2.3). Wer sie automatisiert ausliest, verstößt gegen ihre Nutzungsbedingungen. Auf der anderen Seite stehen drei Quellenklassen, die technisch sauber und vertraglich unbedenklich sind: die Jobsuche-API der Bundesagentur für Arbeit, die öffentlichen Feeds der ATS-Karriereseiten und wenige offizielle Aggregator-APIs. Dieses Kapitel ordnet alle Quellen in eine Matrix ein, trifft die Auswahl für MVP, v1 und v2 und legt die technischen Leitplanken für den Scout fest. Normalisierung, Deduplikation und Scoring folgen in Kapitel 9, der Compliance-Katalog in Kapitel 16, Kosten in Kapitel 18.

### 6.1 Bewertungsraster und Risikoampel

Jede Quelle wird nach sechs Kriterien bewertet: offizielle API (ja/nein), Zugriffsweg, ToS-/Sperr-Risiko, Kosten, Aktualität, Empfehlung. Das ToS-/Sperr-Risiko wird als Ampel geführt, die später auch in der Quellenkonfiguration steht:

| Stufe | Definition | Beispiele | Regel im Bewerbungsagenten |
|---|---|---|---|
| Grün | Vom Betreiber bewusst öffentlich bereitgestellte Schnittstelle oder Feed; keine Login-, Captcha- oder Ratenlimit-Umgehung | BA-API, Greenhouse/Lever/Personio-Feeds, Adzuna, Arbeitnow, JobPosting-JSON-LD auf Firmenseiten | autonom im Tageslauf |
| Gelb | Bezahlter Drittanbieter, der selbst scrapt und das Risiko vertraglich trägt; oder menschlich ausgelöster Einzelabruf | SerpAPI (Google for Jobs), JSearch, Apify-Actors, Einzelabruf einer Anzeige nach Nutzerklick | nur nach Freigabe des Nutzers in der Konfiguration, mit Volumendeckel |
| Rot | Eigenes Scraping von Börsen, deren Nutzungsbedingungen es verbieten; Umgehung technischer Schutzmaßnahmen; Fake-Accounts; Proxy-Rotation | LinkedIn, StepStone, Indeed, XING, Monster, Glassdoor direkt | nie, auch nicht mit Freigabe |

Die Ampel folgt der Rechtslage, wie sie Kapitel 16 ausführt: Screen Scraping frei zugänglicher Daten ist nach BGH I ZR 224/12 nicht per se wettbewerbswidrig ([BGH-Pressemitteilung 2014](https://www.bundesgerichtshof.de/SharedDocs/Pressemitteilungen/DE/2014/2014069.html)), Nutzungsbedingungen können es aber vertraglich wirksam verbieten (EuGH C-30/14, [Kanzlei.biz](https://www.kanzlei.biz/16-01-2015-eugh-c-30-14/)), und strafrechtlich relevant nach § 202a StGB wird es erst, wenn Zugangssicherungen wie Login, Captcha oder IP-Sperren überwunden werden ([dejure § 202a StGB](https://dejure.org/gesetze/StGB/202a.html)). Die TDM-Schranke des § 44b UrhG hilft in der Praxis nicht, weil sie unter dem Nutzungsvorbehalt der Betreiber steht ([gesetze-im-internet § 44b UrhG](https://www.gesetze-im-internet.de/urhg/__44b.html)). Für eine Einzelperson mit rund zehn Bewerbungen am Tag ist das Klagerisiko gering, das Sperrrisiko für das eigene LinkedIn- oder XING-Konto dagegen real; beides spricht dafür, Rot konsequent zu meiden.

### 6.2 Die Quellenmatrix

Die Spalte „API/Zugriff“ fasst zusammen, ob es eine offizielle Schnittstelle gibt und wie der Zugriff erfolgt. Alle Preise Stand 2026 aus der Recherche; Endpunkte, die der Faktenprüfer nicht live testen konnte, sind mit „(live prüfen)“ markiert.

#### 6.2.1 Breitenquellen und Aggregator-APIs

| Quelle | API/Zugriff | ToS-/Sperr-Risiko | Kosten | Aktualität | Empfehlung |
|---|---|---|---|---|---|
| Bundesagentur für Arbeit, Jobsuche-API ([bundesAPI](https://github.com/bundesAPI/jobsuche-api)) | inoffiziell, aber stabil dokumentiert (Community); REST GET, Header `X-API-Key: jobboerse-jobsuche` | Grün (staatliche, öffentliche Daten); Betriebsrisiko: kein SLA, Schema-Brüche v4→v6, 403/404-Issues ([Issues](https://github.com/bundesAPI/jobsuche-api/issues?q=is%3Aissue)) | 0 EUR | live pro Lauf; Filter `veroeffentlichtseit` in Tagen | **Kern (MVP)** |
| Google for Jobs via SerpAPI ([Google Jobs API](https://serpapi.com/google-jobs-api)) | keine Google-API; SerpAPI parst die SERP | Gelb: SerpAPI scrapt Google, Vertragspartner ist SerpAPI; SERP-Änderungen können Parsing brechen | Free 250 Suchen/Monat; 25 USD/1.000; 75 USD/5.000; 150 USD/15.000; 275 USD/30.000 ([costbench](https://costbench.com/software/web-scraping/serpapi/)) | indexabhängig, meist zeitnah | **v1** (Meta-Index über StepStone, Indeed, Firmenseiten) |
| DataForSEO Google Jobs SERP ([Pricing](https://dataforseo.com/pricing/serp/google-jobs-serp-api)) | wie SerpAPI, Pay-as-you-go | Gelb | nicht recherchiert | indexabhängig | optional (Preisalternative) |
| Adzuna API ([developer.adzuna.com](https://developer.adzuna.com/)) | offiziell; REST, App-ID + App-Key, 12 Länder inkl. DE | Grün | kostenloser Tarif; Rate Limits nicht öffentlich benannt | live | **Kern (MVP)**, Abdeckungstiefe DE ungeprüft |
| Arbeitnow API ([arbeitnow.com](https://arbeitnow.com/api/job-board-api)) | offiziell; REST ohne Auth | Grün | 0 EUR | live | **Kern (MVP)** für englischsprachige Tech-Rollen; sonst Nische |
| JSearch (RapidAPI) ([Pricing](https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch/pricing)) | Aggregator über LinkedIn/Indeed/ZipRecruiter/Glassdoor/Bayt | Gelb; DE-Abdeckung unbestätigt (niedrige Konfidenz) | Free 200 Requests; danach ca. 10–200 USD/Monat | unbekannt | optional (Cross-Check in v2) |
| Jooble REST API ([jooble.org/api](https://jooble.org/api/about)) | offiziell, Key per Formular, separater Key je Land (de.jooble.org); POST-only ([jobspipe](https://jobspipe.dev/blog/jooble-api)) | Grün | Free: 500 Requests **lebenslang** pro Key | live | vermeiden (Kontingent für Tagesbetrieb untauglich) |
| Google Cloud Talent Solution ([Doku](https://docs.cloud.google.com/talent-solution/job-search/v3/docs/basics)) | offiziell, aber Matching-Engine nur für eigene eingespeiste Jobs | – | nutzungsabhängig | – | vermeiden (falscher Anwendungsfall) |

#### 6.2.2 Karriereseiten-Feeds der ATS-Systeme (Watchlist-Quellen)

Diese Feeds liefern pro Arbeitgeber alle offenen Stellen, setzen aber voraus, dass Board-Token oder Subdomain der Firma bekannt sind. Sie eignen sich für eine gepflegte Watchlist von Wunscharbeitgebern, nicht für die marktweite Suche. Rechtlich sind sie Grün: Der ATS-Anbieter stellt sie absichtlich öffentlich bereit, damit Firmen ihre Stellen auf eigenen Seiten einbetten können.

| ATS | URL-Muster Karriereseite | Feed/Endpunkt | Auth | Belegstatus | Priorität |
|---|---|---|---|---|---|
| Personio | `{firma}.jobs.personio.de` | `GET /xml?language=de` (auch en/fr/es/nl/it/pt); Filter subcompany, department, office, employmentType ([Personio-Hilfe](https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML)) | keine | Doku eindeutig, live prüfen | P1 (DE-Mittelstand) |
| Greenhouse | `boards.greenhouse.io/{token}` | `GET boards-api.greenhouse.io/v1/boards/{token}/jobs?content=true` ([Greenhouse Docs](https://developers.greenhouse.io/job-board.html)) | keine | Doku eindeutig, live prüfen | P1 (Tech) |
| Lever | `jobs.lever.co/{site}` | `GET api.lever.co/v0/postings/{site}?mode=json`; EU-Instanz `api.eu.lever.co` ([Lever README](https://github.com/lever/postings-api)) | keine für GET | bestätigt | P1 (Tech) |
| Recruitee | `{firma}.recruitee.com` | `GET /api/offers`, 56 Felder je Angebot ([Recruitee Docs](https://docs.recruitee.com/reference/intro-to-careers-site-api)) | keine | Doku eindeutig, live prüfen | P2 |
| SmartRecruiters | `careers.smartrecruiters.com/{firma}` | `GET api.smartrecruiters.com/v1/companies/{id}/postings`; Parameter q, country, city, department ([SR Docs](https://developers.smartrecruiters.com/docs/posting-api)) | öffentlich, falls Kunde freigibt | Doku eindeutig, live prüfen | P2 |
| Workday | `{tenant}.wd{N}.myworkdayjobs.com/{site}` | `POST /wday/cxs/{tenant}/{site}/jobs` mit Body `{appliedFacets, limit, offset, searchText}`; GET liefert 400; Seitengröße hart 20 ([Community-Guide](https://github.com/Francis1998/agentic-career-search/blob/main/docs/guides/WORKDAY_SOURCE_GUIDE.md)) | keine | nur Community-belegt, inoffiziell | P2 (Konzerne) |
| Teamtailor | `{firma}.teamtailor.com` | öffentlicher JSON-Feed **unbestätigt**; ATS-Recherche behauptet Token-Pflicht (Widerspruch) | unklar | unverifiable | P3, erst nach Test |
| softgarden | `{firma}.career.softgarden.de` (nur ein Beleg, niedrige Konfidenz) | Frontend-API v3 braucht eine ClientID, die der Arbeitgeber erzeugt ([dev.softgarden.de](https://dev.softgarden.de/career-websites-api/jobs-api/)) → stattdessen JSON-LD/HTML der Karriereseite | – | belegt | P3 (generischer Adapter) |
| SAP SuccessFactors | eigene Domain oder `career{N}.sapsf.com` | kein öffentlicher Feed gefunden → JSON-LD/HTML | – | – | P3 (generischer Adapter) |
| JOIN | kein festes Muster | inoffiziell: Firmen-ID aus `__NEXT_DATA__`, dann interne Listings-API ([join.com](https://join.com)) | keine | inoffiziell | P3 |
| rexx, d.vinci, onlyfy one | kein einheitliches Muster belegt ([rexx Beispiel](https://www.rexx-systems.com/jobs/), [d.vinci Whitepaper](https://www.dvinci.de/docs20/d.vinci_Whitepaper_Karrierewebsite.pdf)) | JSON-LD/HTML der Karriereseite | – | – | P3 (generischer Adapter) |

Zwei Hinweise des Faktenprüfers sind für die Umsetzung verbindlich: Erstens konnten die Endpunkte von Greenhouse, Personio, Recruitee und SmartRecruiters in der Prüfsitzung nicht live aufgerufen werden (Egress-Sperre); sie decken sich mit den Anbieterdokumentationen, müssen aber in Phase 0 mit je einem echten Arbeitgeber getestet werden. Zweitens ist der Teamtailor-Feed intern widersprüchlich belegt; der Adapter wird erst gebaut, wenn ein Test an einer realen Teamtailor-Seite (Netzwerk-Tab, `/jobs.rss` oder JSON-Endpunkte) Klarheit bringt.

#### 6.2.3 Publikumsbörsen ohne Lese-API

| Quelle | API/Zugriff | ToS-/Sperr-Risiko | Kosten | Aktualität | Empfehlung |
|---|---|---|---|---|---|
| StepStone | keine Lese-API; JobFeed/Connect sind Arbeitgeber-Einspielkanäle ([StepStone Integrations](https://api.stepstone.com/article-categories/integrations/)) | Rot: AGB (Stand 27.04.2022) verbieten Scraping und Bots ([Nutzungsbedingungen](https://www.stepstone.de/ueber-stepstone/nutzungsbedingungen-2022-03/)) | – | – | vermeiden; indirekt über Google for Jobs und Job-Alert-Mails |
| Indeed | Publisher-API 2023 abgeschaltet; verbleibende APIs nur für Arbeitgeber/Partner ([rolesapi](https://rolesapi.com/blog/does-indeed-have-an-api/), [jobspipe](https://jobspipe.dev/blog/indeed-publisher-api)) | Rot: ToS verbieten „robots, spiders, scraper“ ([indeed.com/legal](https://www.indeed.com/legal)) | – | – | vermeiden; indirekt wie StepStone |
| LinkedIn Jobs | keine offene Jobsuche-API; nur Enterprise-Talent-Produkte | Rot: User Agreement § 8.2 verbietet Scraping und Bots ([LinkedIn Hilfe](https://www.linkedin.com/help/linkedin/answer/a1341387)); LinkedIn klagt aktiv, Proxycurl stellte 2025 den Betrieb ein ([nubela](https://nubela.co/blog/is-scraping-linkedin-legal-in-2026/)); hiQ endete 2022 mit Unterlassung und Löschpflicht ([privacyworld](https://www.privacyworld.blog/2022/12/linkedins-data-scraping-battle-with-hiq-labs-ends-with-proposed-judgment/)) | – | – | vermeiden; höchstes Konto-Risiko aller Quellen |
| XING Jobs / onlyfy | keine Lese-API; onlyfy ist Arbeitgeber-Tool ([onlyfy](https://onlyfy.com/de/)) | Rot (AGB-Volltext in der Recherche nicht verifizierbar, Scraping-Verbot wahrscheinlich) | – | – | vermeiden; Job-Alert-Mails |
| Monster.de | nur XML-Einspielung für Arbeitgeber ([Monster Integration](https://arbeitgeber.monster.de/produkte/personal-plattform-integration.aspx)) | Rot: Terms of Use verbieten Crawling ([monster.com](https://www.monster.com/inside/terms-of-use)) | – | – | vermeiden |
| Glassdoor | keine API; Cloudflare, Captchas, clientseitiges Rendering ([scrapeops](https://scrapeops.io/websites/glassdoor/)) | Rot | – | – | vermeiden (geringer Mehrwert für DE-Stellensuche) |
| Jobware, stellenanzeigen.de, meinestadt.de, Kimeta | nur Einspiel-Schnittstellen via ATS-Konnektoren ([d.vinci Schnittstelle](https://www.dvinci.de/standard-schnittstelle/), [meinestadt B2B](https://www.meinestadt.de/unternehmen/b2b/stellenmarkt/downloads), [kimeta](https://www.kimeta.de/)) | Rot/unklar; Kimeta ist selbst Aggregator (Duplikate) | – | – | vermeiden |
| Honeypot.io | Reverse-Plattform, keine API | – | – | – | optional als Profil-Kanal des Nutzers, keine Scan-Quelle |

Wichtig für die Erwartung: Diese Börsen sind nicht verloren. Praktisch alle großen Börsen und viele Karriereseiten zeichnen ihre Anzeigen mit schema.org/JobPosting aus, weil das Voraussetzung für die Sichtbarkeit in Google for Jobs ist ([Google JobPosting-Doku](https://developers.google.com/search/docs/appearance/structured-data/job-posting)). Google for Jobs ist damit faktisch ein Meta-Index, der über SerpAPI (Gelb) lesbar wird. Zusätzlich kann der Nutzer bei StepStone, Indeed, LinkedIn und XING selbst Job-Alerts per E-Mail abonnieren; diese Mails landen in seinem eigenen Postfach und dürfen dort vom Scout gelesen werden (siehe 6.5).

#### 6.2.4 Werkzeuge für Abruf, Rendering und Scraping

| Werkzeug | Zweck/Zugriff | ToS-/Sperr-Risiko | Kosten | Bewertung |
|---|---|---|---|---|
| Claude Web-Fetch-Tool ([Doku](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool)) | serverseitiger Abruf; nur URLs, die zuvor in Nutzer-Nachricht oder Tool-Ergebnis standen; kein JS-Rendering | Grün (nur mit Grün-Quellen nutzen) | keine Zusatzkosten | für Firmen-Karriereseiten ohne Feed in v1; Quellenliste muss als Tool-Ergebnis übergeben werden |
| Jina Reader ([r.jina.ai](https://jina.ai/reader/)) | HTML→Markdown, `GET r.jina.ai/{url}`; ohne Key ca. 20 Requests/Minute | Grün auf Grün-Zielen | kostenlos; mit Key 10 Mio. Gratis-Tokens, danach Pay-as-you-go | Kern der generischen Abrufschicht (v1) |
| Firecrawl ([firecrawl.dev](https://www.firecrawl.dev)) | Scrape/Crawl/Map/Extract, Markdown-Ausgabe; Stealth-Modus 5 Credits/Seite | Gelb bei Stealth (umgeht Bot-Abwehr → nicht nutzen) | Free 1.000 Credits/Monat; Hobby 16 USD/Monat (5.000 Credits); Standard 83 USD ([scrapegraphai](https://scrapegraphai.com/blog/firecrawl-pricing)) | Fallback für Karriereseiten ohne Feed (v1), ohne Stealth |
| Crawl4AI ([GitHub](https://github.com/unclecode/crawl4ai)) | Self-hosted Alternative zu Firecrawl, Playwright-basiert, Version 0.9.3 vom 31.8.2026 ([PyPI](https://pypi.org/project/crawl4ai/)) | Grün, aber „Undetected-Chrome“-Modus nicht nutzen | 0 EUR plus Betrieb | optional (v2), wenn Firecrawl-Kosten stören |
| Playwright MCP ([microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp)) | Browser-Automatisierung über Accessibility-Snapshots, `npx @playwright/mcp@latest` | Grün auf Grün-Zielen; Rot bei Login-Portalen | 0 EUR | Eskalationsstufe für einzelne JS-lastige Karriereseiten (v2); nicht für Massen-Scan |
| Apify-Actors (z. B. [Indeed](https://apify.com/misceres/indeed-scraper), [LinkedIn Jobs](https://apify.com/bebity/linkedin-jobs-scraper), [StepStone](https://apify.com/jupri/stepstone-scraper), [XING](https://apify.com/epctex/xing-scraper)) | Pay-per-Result-Scraper Dritter | Gelb (Risiko beim Actor-Betreiber; Qualität schwankt stark) | ca. 0,05–5 USD je 1.000 Ergebnisse je Actor | nur mit ausdrücklicher Freigabe, v2, Stichproben vor Einsatz |
| python-jobspy ([GitHub](https://github.com/speedyapply/JobSpy), [PyPI](https://pypi.org/project/python-jobspy/)) | Open-Source-Scraper für LinkedIn/Indeed/Glassdoor/Google/ZipRecruiter; MIT | Rot für LinkedIn/Indeed/Glassdoor (eigenes Scraping); LinkedIn blockt ab ca. Seite 10 je IP, „Proxies are a must“ | 0 EUR | vermeiden im Produktivbetrieb; letzte Version 1.1.82 vom 28.7.2025, seit 13 Monaten kein Release |
| Bright Data Jobs Scraper ([brightdata.com](https://brightdata.com/products/web-scraper/jobs-scraper)) | Enterprise-Scraper-API | Gelb | 0,75–1,50 USD/1.000 Records; Datasets ab 2,50 USD/1.000 mit 250 USD Mindestbestellung | vermeiden (überdimensioniert) |
| ScraperAPI ([Pricing](https://www.scraperapi.com/pricing/)) | Proxy-/Rendering-API gegen Bot-Abwehr | Rot (Zweck ist Umgehung) | Free 1.000 Credits; Hobby 49 USD/Monat | vermeiden |

Zu beachten: Cloudflare blockiert seit dem 1. Juli 2025 bekannte AI-Crawler für neue Domains standardmäßig und bietet Betreibern seitdem granulare Steuerung ([Cloudflare-Blog](https://blog.cloudflare.com/control-content-use-for-ai-training/)). Trifft der generische Abruf auf eine solche Sperre, ist das für den Bewerbungsagenten ein Stoppsignal, kein Anlass für Stealth-Modi.

### 6.3 Kernquelle 1: Bundesagentur für Arbeit

Die BA betreibt die größte deutsche Stellendatenbank. Eine offizielle API gibt es nicht; das Community-Projekt bundesAPI hat die Schnittstelle der Jobsuche-App dokumentiert ([openapi.yaml](https://raw.githubusercontent.com/bundesAPI/jobsuche-api/main/openapi.yaml), [Doku-Portal](https://jobsuche.api.bund.dev/)). Der Faktenprüfer hat Endpunkte, Header und Filterparameter anhand der OpenAPI-Spezifikation bestätigt; die Live-Erreichbarkeit wurde nicht getestet.

```http
GET https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v6/jobs
    ?was=<Zielrolle>&wo=<Ort>&umkreis=50
    &veroeffentlichtseit=1        # Tage seit Veröffentlichung (0–100)
    &angebotsart=1                # 1 Arbeit, 2 Selbstständigkeit, 4 Ausbildung, 34 Praktikum/Trainee
    &zeitarbeit=false             # Zeitarbeitsfirmen ausblenden (Default true)
    &befristung=2                 # 1 befristet, 2 unbefristet (optional)
    &arbeitszeit=vz;ho            # vz Vollzeit, tz Teilzeit, snw Schicht, ho Homeoffice, mj Minijob
    &size=50&page=1
X-API-Key: jobboerse-jobsuche

GET https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v4/jobdetails/{base64(refnr)}
X-API-Key: jobboerse-jobsuche
```

Weitere Parameter: `berufsfeld`, `arbeitgeber` (Filter nach Arbeitgebername), `pav` (private Arbeitsvermittler), `behinderung`. Das Flag `zeitarbeit` und der Parameter `pav` sind die einzigen strukturierten Signale zur Vermittler-Erkennung in irgendeiner Quelle; Kapitel 9 nutzt sie als Eingangsindiz.

**Betriebsrisiken:** Die offenen GitHub-Issues zeigen Schema-Änderungen (v4→v6, Issue #69, August 2026), zeitweise 403-Antworten (Issue #60), fehlende Hashes, die den Detailabruf brechen (Issue #59), und 404 auf Detail-Endpunkten (Issue #61). Der Adapter braucht deshalb: strikte Schema-Validierung mit Alarm statt stillem Weiterlaufen, Fallback von `/pc/v4/jobdetails` auf `/pc/v3/jobdetails`, und eine Tagesnotiz im Review-Cockpit, wenn die Trefferzahl gegenüber dem Vortag um mehr als 50 Prozent einbricht.

**Entscheidung:** Die BA-API ist die Primärquelle des MVP. **Begründung:** einzige kostenlose Quelle mit breiter Abdeckung deutscher Direktarbeitgeber und sauberem JSON-Zugriff; rechtlich der unbedenklichste Weg (öffentliche Daten einer Behörde, keine restriktiven AGB), wobei das Repository keine Lizenz- oder Nutzungsbedingungen der BA enthält und der Zugang jederzeit geändert werden kann ([bundesAPI README](https://github.com/bundesAPI/jobsuche-api)). **Alternative:** das PyPI-Paket `de-jobsuche` (Version 0.1.0 aus 2022, veraltet, deaktivierte TLS-Prüfung) wird nicht verwendet; der Adapter wird als eigener schlanker Client gegen v6 geschrieben ([PyPI de-jobsuche](https://pypi.org/project/de-jobsuche/)).

### 6.4 Kernquelle 2: Watchlist mit ATS-Feeds und ATS-Detektor

Der zweite Pfeiler ist eine vom Nutzer gepflegte Liste von Wunscharbeitgebern (Default-Annahme: 30 bis 100 Firmen; siehe offene Fragen). Für jede Firma ermittelt ein **ATS-Detektor** einmalig den Karriereseiten-Typ und speichert den passenden Adapter. Ablauf:

1. Karriereseiten-URL der Firma abrufen (Rechercheur liefert sie, Kapitel 10).
2. URL gegen die Muster aus Tabelle 6.2.2 prüfen (Regex-Set: `*.jobs.personio.de`, `boards.greenhouse.io/*`, `jobs.lever.co/*`, `*.recruitee.com`, `careers.smartrecruiters.com/*`, `*.wd\d+.myworkdayjobs.com`, `*.teamtailor.com`, `*.career.softgarden.de`).
3. Ohne Treffer: HTML-Fingerprinting mit der Kategorie 101 „Recruitment & staffing“ von webappanalyzer, die Fingerprints für Personio, Greenhouse, Lever, Recruitee, SmartRecruiters, Teamtailor und onlyfy enthält, aber nicht für softgarden, rexx, d.vinci, SAP SuccessFactors oder JOIN ([categories.json](https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json), [Repo](https://github.com/enthec/webappanalyzer)). Für diese deutschen Anbieter braucht der Detektor eigene URL-/HTML-Muster, die in Phase 0 an je drei Beispielseiten erhoben werden.
4. Weiter ohne Treffer: Seite auf eingebettetes `application/ld+json` mit `@type: JobPosting` prüfen; falls vorhanden, generischer JSON-LD-Adapter.
5. Letzter Fallback: Seite per Jina Reader oder Firecrawl in Markdown wandeln und mit Claude Haiku 4.5 (`claude-haiku-4-5`) in das Rohtreffer-Schema extrahieren; bei niedriger Extraktionskonfidenz Claude Sonnet 5 (`claude-sonnet-5`) als zweite Stufe.

Das Ergebnis der Erkennung (ATS-Typ, Feed-URL, Adapter, letzter erfolgreicher Abruf) wird pro Firma gespeichert und dient außerdem dem Rechercheur (ATS-Typ als Recherchefeld, Kapitel 10), dem ATS-Prüfer (Kapitel 12) und dem Boten (Formulartyp, Kapitel 15).

**Entscheidung:** Adapter für Personio, Greenhouse und Lever im MVP; Recruitee, SmartRecruiters und Workday in v1; Teamtailor, JOIN und der generische JSON-LD/Markdown-Adapter in v1 nach Live-Test; softgarden, SuccessFactors, rexx, d.vinci, onlyfy nur über den generischen Adapter. **Begründung:** Die drei MVP-Adapter sind dokumentiert, authentifizierungsfrei und decken Mittelstand (Personio) wie Tech-Arbeitgeber (Greenhouse, Lever) ab; Workday ist verbreitet, aber nur inoffiziell belegt und kann sich ohne Ankündigung ändern. Die Marktanteilsaussage der DGFP-Studie 2025 (nur SAP SuccessFactors, softgarden, Personio, rexx über 5 Prozent) konnte der Prüfer nicht verifizieren; sie wird als unbestätigte Priorisierungshilfe geführt und in Phase 0 direkt geprüft. **Alternative:** ein einziger LLM-basierter Universal-Scraper für alle Karriereseiten; verworfen, weil teurer, langsamer und schlechter prüfbar als deterministische Adapter.

### 6.5 Ergänzende Kanäle: Job-Alert-Mails, Adzuna, Arbeitnow, Google for Jobs

**Job-Alert-Mails des Nutzers.** Der Nutzer abonniert bei StepStone, Indeed, LinkedIn und XING selbst tägliche Job-Alerts an sein Bewerbungspostfach. Der Scout liest diese Mails über die in Kapitel 15 beschriebene Postfach-Anbindung und extrahiert Titel, Firma, Ort und Link. Das ist kein Scraping der Börse, sondern das Lesen eigener Post; die legal-Recherche führt „vom Nutzer selbst abonnierte Job-Alerts per E-Mail“ ausdrücklich als autonom zulässigen Kanal. Der Volltext der Anzeige wird jedoch nicht automatisch nachgeladen: Erst wenn der Nutzer den Treffer im Review-Cockpit auswählt, ruft der Scout genau diese eine Seite ab (menschlich ausgelöster Einzelabruf, Gelb), oder der Nutzer fügt den Text ein. Massenabrufe der Börsen bleiben damit ausgeschlossen. Diese Einordnung ist eine eigene Ableitung aus der Rechtslage in Kapitel 16, kein gerichtlich bestätigter Fall; sie steht in den offenen Fragen.

**Adzuna** liefert eine offizielle, kostenlose Such-API mit Deutschland-Abdeckung; konkrete Rate Limits nennt die Recherche nicht, sie sind in Phase 0 bei der Registrierung zu prüfen ([developer.adzuna.com](https://developer.adzuna.com/)). Ein fertiger MCP-Server existiert ([adzuna-job-search-mcp](https://github.com/folathecoder/adzuna-job-search-mcp)), für das Backend ist der direkte REST-Aufruf einfacher. **Arbeitnow** ist eine offene API ohne Auth mit Fokus auf englischsprachige Tech-, Data- und Remote-Rollen in Deutschland ([Arbeitnow-Blog](https://www.arbeitnow.com/blog/job-board-api)); ob sie für den Nutzer relevant ist, hängt von seinen Zielrollen ab (Kapitel 22).

**Google for Jobs über SerpAPI** kommt in v1 hinzu, sobald der MVP gezeigt hat, wie viele relevante Stellen die Grün-Quellen verfehlen. Der Free-Tier mit 250 Suchen im Monat (rund 8 pro Tag) dient am Ende des MVP genau dieser Abdeckungsmessung; für den Dauerbetrieb reichen 1.000 Suchen für 25 USD/Monat, also gut 30 Suchen pro Tageslauf ([SerpAPI Pricing](https://serpapi.com/pricing)). SerpAPI ist Gelb, weil es selbst Googles Ergebnisseiten parst und Google die SERP-Struktur jederzeit ändern kann; der Vertragspartner des Nutzers ist SerpAPI, nicht Google und nicht die Börsen.

### 6.6 Stufenstrategie

| Stufe | Quellen | Abrufschicht | Kosten Datenquellen | Ausstiegskriterium |
|---|---|---|---|---|
| MVP (Wochen 1–4) | BA-API; Watchlist-Adapter Personio/Greenhouse/Lever; Adzuna; Arbeitnow; Job-Alert-Mails (Metadaten) | deterministische Python-Adapter, kein Browser | 0 € (Kapitel 18.4) | 20 Werktage stabile Tagesläufe; Abdeckungsmessung mit SerpAPI-Free-Tier dokumentiert |
| v1 (Monat 2–3) | + SerpAPI 1.000 Suchen/Monat; + Recruitee, SmartRecruiters, Workday, Teamtailor (nach Test); + generischer JSON-LD-Adapter; + Jina/Firecrawl-Fallback ohne Stealth; Einzelabruf nach Nutzerklick | + Web-Fetch/Jina/Firecrawl, Haiku-4.5-Extraktion | siehe Kapitel 18.4/18.6 | Watchlist-Abdeckung ≥ 90 % der Firmen mit funktionierendem Adapter |
| v2 (Monat 4–6) | + Playwright MCP für einzelne JS-Seiten; + JOIN; optional Apify-Actors oder JSearch nur nach ausdrücklicher Freigabe mit Volumendeckel; Crawl4AI statt Firecrawl, falls Kosten | + Browser für Einzelseiten | siehe Kapitel 18.6 | siehe Kapitel 19 |

Die verbindlichen Euro-Gesamtkosten (inklusive Datenquellen) rechnet Kapitel 18 durch; dieses Kapitel nennt hier keine eigenen, davon abweichenden Monatsbeträge.

**Entscheidung:** Der MVP nutzt ausschließlich Grün-Quellen; Gelb-Quellen kommen erst in v1 nach dokumentierter Abdeckungslücke, Rot-Quellen nie. **Begründung:** Bei rund zehn Bewerbungen am Tag ist Breite weniger wert als Verlässlichkeit; jede Gelb-Quelle bringt Kosten, Parsing-Fragilität und ein Restrisiko, das nur der Nutzer selbst eingehen kann. **Alternative:** Frühstart mit SerpAPI und Apify für maximale Abdeckung ab Tag 1; verworfen, weil Dedup, Scoring und Kritiker zuerst an einer stabilen Quelle reifen sollen.

### 6.7 Technische Leitplanken für den Scout

Der Quellenabruf ist deterministischer Code, kein LLM-Tool-Aufruf. Claude-Modelle kommen erst bei der Extraktion unstrukturierter Seiten ins Spiel. Das hält Kosten und Fehlerbilder beherrschbar und verhindert, dass Inhalte einer Stellenanzeige (ungeprüfter Drittinhalt) Tool-Aufrufe auslösen; die Anthropic-Leitlinie zu indirekter Prompt Injection verlangt, solche Inhalte nur als gekennzeichnete Tool-Ergebnisse zu übergeben ([Anthropic Guardrails](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)). Weitere Regeln:

- **Selbstauferlegte Raten:** höchstens 1 Request pro Sekunde je Host; BA-API maximal 40 Suchanfragen plus 200 Detailabrufe pro Tageslauf; jeder Watchlist-Feed genau einmal pro Tageslauf; HTTP-Caching per ETag/Last-Modified, wo angeboten.
- **Identität:** ein ehrlicher User-Agent mit Kontakt-E-Mail des Nutzers; keine Proxys, keine rotierenden IPs, kein Headless-Tarnmodus, keine Login-Automatisierung zum Lesen.
- **robots.txt:** wird für generische Seitenabrufe (Jina, Firecrawl, Web Fetch, Playwright) beachtet; ein `Disallow` oder eine Bot-Sperre beendet den Abruf und erzeugt eine Notiz im Review-Cockpit. Für API-Endpunkte gilt die Doku des Anbieters.
- **Fehlerbehandlung:** 403/429 setzen die Quelle für 24 Stunden aus (exponentielles Backoff über mehrere Tage); Schema-Abweichungen werden protokolliert und dem Nutzer gemeldet, nicht stumm ignoriert.
- **Personenbezogene Daten:** Namen von Ansprechpartnern aus Anzeigen werden nur für die konkrete Bewerbung gespeichert und nach Abschluss gelöscht oder anonymisiert; das BGH-Urteil vom November 2024 lässt schon den Kontrollverlust über solche Daten als Schaden genügen ([Noerr](https://www.noerr.com/de/insights/bgh-urteil-zum-immateriellen-schadensersatz-der-dsgvo-wegen-scraping)). Details in Kapitel 16.
- **Web-Fetch-Besonderheit:** Das serverseitige Web-Fetch-Tool ruft nur URLs ab, die zuvor in einer Nutzer-Nachricht oder einem Tool-Ergebnis standen, nicht solche aus dem System-Prompt; Quellen- und Watchlist-URLs müssen dem Modell deshalb als Tool-Ergebnis übergeben werden.

Quellenkonfiguration (Auszug, YAML):

```yaml
quellen:
  - id: ba-jobsuche
    typ: api
    risiko: gruen
    stufe: mvp
    endpoint: https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v6/jobs
    auth: { header: X-API-Key, wert: jobboerse-jobsuche }
    abfragen:
      - { was: "<Zielrolle 1>", wo: "<Ort>", umkreis: 50, veroeffentlichtseit: 2, angebotsart: 1, zeitarbeit: false }
      - { was: "<Zielrolle 2>", wo: "<Ort>", umkreis: 50, veroeffentlichtseit: 2, angebotsart: 1, zeitarbeit: false }
    limits: { requests_pro_lauf: 40, details_pro_lauf: 200, pause_ms: 1000 }
  - id: watchlist
    typ: ats-feeds
    risiko: gruen
    stufe: mvp
    firmen_datei: watchlist.yaml       # firma, karriere_url, ats_typ, feed_url, adapter, letzter_erfolg
    adapter_aktiv: [personio, greenhouse, lever]
  - id: adzuna
    typ: api
    risiko: gruen
    stufe: mvp
    auth: { app_id: env:ADZUNA_APP_ID, app_key: env:ADZUNA_APP_KEY }
    land: de
  - id: job-alert-mails
    typ: postfach
    risiko: gruen
    stufe: mvp
    absender: [stepstone.de, indeed.com, linkedin.com, xing.com]
    volltext_nachladen: nur_nach_nutzerklick
  - id: serpapi-google-jobs
    typ: api
    risiko: gelb
    stufe: v1
    freigabe_nutzer: erforderlich
    limits: { suchen_pro_lauf: 30 }
```

Jeder Adapter liefert einen **Rohtreffer** mit denselben Pflichtfeldern: `quelle_id`, `quelle_url`, `externe_id` (z. B. BA-`refnr`, Greenhouse-`id`), `titel`, `arbeitgeber`, `ort`, `veroeffentlicht_am`, `beschreibung_roh`, `bewerbungs_url`, `abgerufen_am`, `inhalts_hash`. Das vollständige Datenmodell steht in Kapitel 7, die Normalisierung in Kapitel 9. Der Status jeder Stelle nach dem Abruf ist „entdeckt“.

### 6.8 Ausblick auf Dedup und Vermittler-Erkennung

Dieselbe Stelle taucht regelmäßig in BA-API, Google for Jobs, einer Job-Alert-Mail und dem ATS-Feed auf. Reines Fuzzy-Matching auf Titel und Firma übersieht laut Fachquelle rund 30 Prozent der Duplikate ([Medium: Fuzzy Matching](https://medium.com/@williamflaiz/why-fuzzy-matching-isnt-enough-and-what-actually-finds-your-hidden-duplicates-7ddfdc5c26de)); der Scout liefert deshalb kanonische IDs und `bewerbungs_url` mit, damit Kapitel 9 zuerst hart und erst dann unscharf abgleichen kann. Ob eine Anzeige von einem Personalvermittler stammt, ist aus dem Text nicht sicher ableitbar, weil § 11 AÜG nur eine schriftliche Information gegenüber Arbeitnehmer und Entleiher verlangt, keine Kennzeichnung im Inserat ([§ 11 AÜG](https://www.gesetze-im-internet.de/a_g/__11.html)). Der Scout gibt darum nur Indizien weiter: BA-Flags `zeitarbeit`/`pav`, Domain der Anzeige ungleich genannter Firma, Formulierungen wie „für unseren Kunden“. Die Entscheidung trifft der Matcher, im Zweifel per „Rückfrage offen“ an den Nutzer (Kapitel 9).

### 6.9 Offene Fragen mit Default-Annahmen

1. Dürfen Gelb-Quellen (SerpAPI, später Apify/JSearch) überhaupt genutzt werden? Default: SerpAPI ab v1 ja, Apify/JSearch nein.
2. Soll der Volltext einer Anzeige aus einer Job-Alert-Mail automatisch nachgeladen werden? Default: nein, nur nach Klick im Review-Cockpit.
3. Gibt es eine Watchlist von Wunscharbeitgebern und wie groß ist sie? Default: der Nutzer liefert 20 bis 50 Firmen, der Scout schlägt aus BA-Treffern weitere vor.
4. Welches Monatsbudget für Datenquellen ist akzeptabel? Default: 0 € im MVP; ab v1 siehe Kostenrahmen in Kapitel 18.4/18.6.
5. Sind englischsprachige Tech-Rollen relevant (Arbeitnow, Greenhouse/Lever-Schwerpunkt)? Default: ja, bis Zielrollen bekannt sind.
6. Sollen Ansprechpartner-Namen aus Anzeigen über die einzelne Bewerbung hinaus gespeichert werden? Default: nein, Löschung nach Abschluss.

**Quellen dieses Kapitels:**

- bundesAPI/jobsuche-api (GitHub): https://github.com/bundesAPI/jobsuche-api
- bundesAPI/jobsuche-api openapi.yaml: https://raw.githubusercontent.com/bundesAPI/jobsuche-api/main/openapi.yaml
- bundesAPI/jobsuche-api Issues: https://github.com/bundesAPI/jobsuche-api/issues?q=is%3Aissue
- Jobsuche-API Dokumentationsportal: https://jobsuche.api.bund.dev/
- de-jobsuche (PyPI): https://pypi.org/project/de-jobsuche/
- BGH-Pressemitteilung I ZR 224/12 (Screen Scraping): https://www.bundesgerichtshof.de/SharedDocs/Pressemitteilungen/DE/2014/2014069.html
- EuGH C-30/14 Ryanair/PR Aviation (Kanzlei.biz): https://www.kanzlei.biz/16-01-2015-eugh-c-30-14/
- § 202a StGB (dejure): https://dejure.org/gesetze/StGB/202a.html
- § 44b UrhG (gesetze-im-internet): https://www.gesetze-im-internet.de/urhg/__44b.html
- § 11 AÜG (gesetze-im-internet): https://www.gesetze-im-internet.de/a_g/__11.html
- Noerr: BGH-Urteil zum immateriellen Schadensersatz wegen Scraping: https://www.noerr.com/de/insights/bgh-urteil-zum-immateriellen-schadensersatz-der-dsgvo-wegen-scraping
- SerpAPI Google Jobs API: https://serpapi.com/google-jobs-api
- SerpAPI Pricing: https://serpapi.com/pricing
- SerpAPI-Preisübersicht (costbench): https://costbench.com/software/web-scraping/serpapi/
- DataForSEO Google Jobs SERP API: https://dataforseo.com/pricing/serp/google-jobs-serp-api
- Google JobPosting Structured Data: https://developers.google.com/search/docs/appearance/structured-data/job-posting
- Google Cloud Talent Solution: https://docs.cloud.google.com/talent-solution/job-search/v3/docs/basics
- Adzuna Developer Portal: https://developer.adzuna.com/
- Adzuna Job Search MCP: https://github.com/folathecoder/adzuna-job-search-mcp
- Arbeitnow Job Board API: https://arbeitnow.com/api/job-board-api
- Arbeitnow Blog: Job Board API: https://www.arbeitnow.com/blog/job-board-api
- JSearch (RapidAPI) Pricing: https://rapidapi.com/letscrape-6bRBa3QguO5/api/jsearch/pricing
- Jooble API: https://jooble.org/api/about
- Jooble API (jobspipe): https://jobspipe.dev/blog/jooble-api
- Personio: Jobs via XML integrieren: https://support.personio.de/hc/en-us/articles/207576365-Integrate-jobs-from-Personio-into-your-website-via-XML
- Greenhouse Job Board API: https://developers.greenhouse.io/job-board.html
- Lever Postings API (GitHub): https://github.com/lever/postings-api
- Recruitee Careers Site API: https://docs.recruitee.com/reference/intro-to-careers-site-api
- SmartRecruiters Posting API: https://developers.smartrecruiters.com/docs/posting-api
- Workday Source Guide (Community): https://github.com/Francis1998/agentic-career-search/blob/main/docs/guides/WORKDAY_SOURCE_GUIDE.md
- softgarden Career Websites API: https://dev.softgarden.de/career-websites-api/jobs-api/
- rexx systems Jobs (Beispiel): https://www.rexx-systems.com/jobs/
- d.vinci Whitepaper Karrierewebsite: https://www.dvinci.de/docs20/d.vinci_Whitepaper_Karrierewebsite.pdf
- JOIN: https://join.com
- webappanalyzer categories.json: https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json
- webappanalyzer (GitHub): https://github.com/enthec/webappanalyzer
- StepStone Integrations (JobFeed/Connect): https://api.stepstone.com/article-categories/integrations/
- StepStone Nutzungsbedingungen (Stand 2022): https://www.stepstone.de/ueber-stepstone/nutzungsbedingungen-2022-03/
- Indeed API (rolesapi): https://rolesapi.com/blog/does-indeed-have-an-api/
- Indeed Publisher API (jobspipe): https://jobspipe.dev/blog/indeed-publisher-api
- Indeed Legal/Terms: https://www.indeed.com/legal
- LinkedIn Hilfe: Verbotene Software und Erweiterungen: https://www.linkedin.com/help/linkedin/answer/a1341387
- Nubela: Is scraping LinkedIn legal in 2026: https://nubela.co/blog/is-scraping-linkedin-legal-in-2026/
- Privacy World: hiQ v. LinkedIn Judgment: https://www.privacyworld.blog/2022/12/linkedins-data-scraping-battle-with-hiq-labs-ends-with-proposed-judgment/
- onlyfy: https://onlyfy.com/de/
- Monster Plattform-Integration: https://arbeitgeber.monster.de/produkte/personal-plattform-integration.aspx
- Monster Terms of Use: https://www.monster.com/inside/terms-of-use
- ScrapeOps: Glassdoor: https://scrapeops.io/websites/glassdoor/
- d.vinci Standard-Schnittstelle: https://www.dvinci.de/standard-schnittstelle/
- meinestadt.de B2B Stellenmarkt: https://www.meinestadt.de/unternehmen/b2b/stellenmarkt/downloads
- Kimeta: https://www.kimeta.de/
- Claude Web Fetch Tool: https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool
- Anthropic: Mitigate jailbreaks and prompt injections: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks
- Jina Reader: https://jina.ai/reader/
- Firecrawl: https://www.firecrawl.dev
- Firecrawl Pricing (scrapegraphai): https://scrapegraphai.com/blog/firecrawl-pricing
- Crawl4AI (GitHub): https://github.com/unclecode/crawl4ai
- Crawl4AI (PyPI): https://pypi.org/project/crawl4ai/
- Playwright MCP: https://github.com/microsoft/playwright-mcp
- Apify Indeed Scraper: https://apify.com/misceres/indeed-scraper
- Apify LinkedIn Jobs Scraper: https://apify.com/bebity/linkedin-jobs-scraper
- Apify StepStone Scraper: https://apify.com/jupri/stepstone-scraper
- Apify XING Scraper: https://apify.com/epctex/xing-scraper
- python-jobspy (GitHub): https://github.com/speedyapply/JobSpy
- python-jobspy (PyPI): https://pypi.org/project/python-jobspy/
- Bright Data Jobs Scraper API: https://brightdata.com/products/web-scraper/jobs-scraper
- ScraperAPI Pricing: https://www.scraperapi.com/pricing/
- Cloudflare: Control content use for AI training: https://blog.cloudflare.com/control-content-use-for-ai-training/
- Medium: Why fuzzy matching isn't enough: https://medium.com/@williamflaiz/why-fuzzy-matching-isnt-enough-and-what-actually-finds-your-hidden-duplicates-7ddfdc5c26de
