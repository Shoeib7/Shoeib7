## 4. Der Gegner: ATS und KI-Screening in Deutschland

Ein ATS (Applicant Tracking System, auf Deutsch Bewerbermanagementsystem) ist die Software, in der eine Bewerbung landet, sobald sie über ein Karriereportal oder per E-Mail eingeht. Es speichert die Unterlagen, liest den Lebenslauf maschinell aus (Parsing), zeigt Bewerber in einer Pipeline und hilft Recruitern beim Sortieren. „Gegner“ ist bewusst überspitzt: In Deutschland entscheidet das ATS 2026 fast nie selbst über eine Bewerbung. Es entscheidet aber darüber, in welcher Form ein Mensch sie zu sehen bekommt – als sauber ausgelesenes Profil mit den passenden Stichworten oder als Datensalat mit fehlender Telefonnummer. Drei Fragen stehen im Mittelpunkt: Welche Systeme sind relevant? Was tun sie wirklich? Welche Regeln folgen daraus? Die Regeln setzen der ATS-Prüfer (Kapitel 12) und der Setzer (Kapitel 13) um; die Erkennung des ATS pro Firma übernimmt der Rechercheur (Kapitel 10).

### 4.1 Der Markt: welche ATS in Deutschland zählen

Der deutsche ATS-Markt ist fragmentiert. Laut der DGFP-Benchmarkstudie „Recruiting-Strukturen 2025“ (3. Auflage, mit HTW Leipzig und Wollmilchsau, über 700 Befragte) kommen nur vier Anbieter auf einen Marktanteil über 5 %: SAP SuccessFactors, softgarden, Personio und rexx systems ([DGFP](https://www.dgfp.de/aktuell/recruiting-strukturen-2025-recruiting-wird-strukturierter-datengetriebener-und-technologischer)). Diese Aussage ist **unbestätigt**: Der unabhängige Faktenprüfer konnte die Studie nicht einsehen, und die Recherche zitiert sie nur aus Zweitquellen. Sie bleibt trotzdem die beste verfügbare Orientierung und dient hier als Arbeitshypothese, die vor der Architekturentscheidung in Kapitel 7 direkt gegen den Studientext zu prüfen ist (siehe 4.9).

Dazu kommen internationale Systeme, die in Deutschland vor allem bei Konzernen und Tech-Arbeitgebern stehen (Workday, Greenhouse, Lever, SmartRecruiters, Teamtailor, iCIMS), und ein langer Schwanz deutscher KMU-Systeme ohne belegten Marktanteil (d.vinci, onlyfy one als Fusion aus XING E-Recruiting und Prescreen, JOIN, Recruitee und weitere).

Wie viel KI dabei im Spiel ist, bleibt moderat: Nach der Bitkom New Work Studie 2025 nutzen 34 % der deutschen Unternehmen KI irgendwo im HR-Kontext, aber nur 11 % recruiting-spezifisch (EU-Schnitt 22 %); automatisiertes CV-Screening setzen rund 31 % der KI-nutzenden Unternehmen ein ([yena.ai](https://www.yena.ai/de/blog/ki-im-recruiting-studie-dach-2026); Sekundärquelle, nicht unabhängig geprüft). Für die Planung heißt das: Der typische deutsche Arbeitgeber liest Bewerbungen 2026 noch mit Menschenaugen – nach einem maschinellen Parsing.

| Anbieter | Typische Arbeitgeber | CV-Parser | KI-Ranking (Stand 09/2026) | Erkennbar per URL | Priorität |
|---|---|---|---|---|---|
| Personio | KMU, Mittelstand, Start-ups | Textkernel (Opt-in) | nein, nur Knockout-Fragen | hoch | 1 |
| SAP SuccessFactors | Konzerne | nicht belegt | Joule, lizenzabhängig | niedrig | 1 |
| softgarden | Mittelstand | Textkernel (Add-on) | nicht belegt | mittel (unbestätigt) | 1 |
| rexx systems | Mittelstand | nicht belegt | nicht belegt | kein Muster | 1 |
| Workday | internationale Konzerne | eigen (NLP) | HiredScore A–D | hoch | 2 |
| Greenhouse, Lever | Tech, Start-ups | nicht belegt | nein; Knockout tarifabhängig | hoch | 2 |
| SmartRecruiters | international | nicht belegt | SmartAssistant 1–5 Sterne | hoch | 2 |
| Teamtailor | Start-ups | nicht belegt | Co-Pilot (GPT) | mittel | 3 |
| Recruitee, d.vinci | KMU | Textkernel | nicht belegt | Recruitee hoch, d.vinci kein Muster | 3 |
| onlyfy one, JOIN | KMU | nicht belegt | nicht belegt | kein Muster | 3 |

**Entscheidung:** Erkennungs- und Formatlogik zuerst für Personio, SAP SuccessFactors, softgarden und rexx systems auslegen, dann Workday, Greenhouse, Lever und SmartRecruiters, dann der Rest. **Begründung:** Die vier deutschen Anbieter sind die einzigen mit (wenn auch unbestätigt) belegtem Marktanteil über 5 %; die internationalen Systeme sind technisch am besten erkennbar und dokumentiert, also billig zusätzlich abzudecken. **Alternative:** Reihenfolge nach der tatsächlichen Verteilung in den vom Scout gefundenen Stellen – die bessere Ordnung, sobald der Tracker nach vier Wochen Betrieb echte Zahlen liefert (Kapitel 15). Bis dahin gilt die Marktreihenfolge.

### 4.2 Parsing: was mit dem Lebenslauf wirklich passiert

Parsing ist der erste und meist einzige vollautomatische Schritt. Der Parser holt den Text aus der Datei, erkennt Abschnitte anhand ihrer Überschriften, extrahiert Entitäten (Name, Kontaktdaten, Arbeitgeber, Positionen, Zeiträume, Abschlüsse, Kenntnisse) und schreibt sie in ein strukturiertes Bewerberprofil. Recruiter sehen anschließend dieses Profil – oft bevor sie das Original-PDF öffnen.

Das Bemerkenswerte am deutschen Markt: Hinter vielen Oberflächen steckt derselbe Parser. Textkernel – seit der Übernahme von Sovren am 30.11.2021 und der Markenfusion am 14.12.2023 auch Träger der lange am weitesten verbreiteten Parsing-Engine der Branche ([Textkernel/Sovren](https://www.textkernel.com/sovren/)) – ist Sub-Dienstleister für Personio (Opt-in; bei Aktivierung wird jede eingehende Bewerbung mit Lebenslauf automatisch geparst und ins Profil übernommen) ([Personio Support](https://support.personio.de/hc/en-us/articles/360010193018-CV-parsing-for-candidate-profiles)), für softgarden (Add-on „Textkernel“, kostenfrei über den Marketplace aktivierbar; die Genauigkeit variiert laut Anbieter je nach Formatierung, Sprache und Aufbau des Lebenslaufs) ([softgarden Support](https://support.softgarden.de/de/articles/680780-cv-parsing)), für d.vinci und Recruitee ([d.vinci](https://www.dvinci.de/bms/cv-parsing-leichter-bewerben-dank-intelligenter-recruiting-technologie/)) sowie über die Sovren-Engine für iCIMS. Textkernel selbst nennt über 95 % Genauigkeit bei den wichtigsten Feldern, 29 Sprachen, rund 2 Mrd. Dokumente pro Jahr und zunehmenden LLM-Einsatz (Herstellerangaben; [Textkernel Parser](https://www.textkernel.com/de/produkte-loesungen/parser/), [Developer-Doku](https://developer.textkernel.com/Parser/master/)).

Daraus folgt dreierlei:

1. Ein Lebenslauf, den ein Textkernel-artiger Parser sauber liest, ist für einen großen Teil der deutschen Zielfirmen richtig formatiert. Formatoptimierung pro ATS ist überflüssig; Differenzierung lohnt sich nur bei der Inhaltsstrategie (4.3).
2. Textkernel ist nicht selbst testbar (Zugang nur über den Vertrieb, Enterprise-Preise). Der ATS-Prüfer braucht Stellvertreter-Parser. Die Recherche empfiehlt eine kostenlose Zwei-Parser-Gegenprobe: Apache Tika (Apache-2.0-Lizenz; Version 4.0.0 seit 18.8.2026 final, mit Markdown-Ausgabe als Standard und einem Vision-Language-Model-Parser – die in der Erstrecherche genannte 3.3.0 ist überholt) ([Tika CHANGES](https://raw.githubusercontent.com/apache/tika/main/CHANGES.txt), [Tika-Repo](https://github.com/apache/tika)) und den OpenResume-Parser (AGPL-3.0, rund 8.900 GitHub-Stars, PDF.js-basiert, nur PDF, lokal per npm oder Docker) ([OpenResume](https://github.com/xitanggg/open-resume)). Kostenpflichtige Dienste – Jobscan (kostenlos 5 Scans/Monat, Premium 49,95 $/Monat) ([Jobscan-Preise](https://pitchmeai.com/blog/jobscan-pricing-plans)), Affinda (kein Self-Service-Free-Tier, ab etwa 800 $/Monat) ([G2](https://www.g2.com/products/resume-parser-by-affinda/pricing)), Eden AI als Parser-Aggregator ab 0,04 $/Seite ([Eden AI](https://www.edenai.co/post/best-resume-parser-apis)) – sind für den MVP unnötig und auf den englischsprachigen Markt ausgelegt. Details in Kapitel 12 und 17.
3. Der Parser liest den Lebenslauf, nicht das Anschreiben. Die Parsing-Dokumentationen von Personio und softgarden beziehen sich ausschließlich auf den CV; ob LLM-Screener auch Anschreiben auswerten, ist in der Recherche nicht dokumentiert. Der Keyword-Abgleich konzentriert sich deshalb auf den Lebenslauf; das Anschreiben wird für Menschen geschrieben (Kapitel 11).

Was das Parsing bricht – und wie gut das belegt ist:

| Formatmerkmal | Wirkung im Parser | Belegstärke |
|---|---|---|
| PDF ohne Text-Layer (Scan, Bildexport, Design-Tool-Export) | kein oder unbrauchbarer Text | mittel: Ratgeberkonsens, keine Herstellerdoku |
| Textbasiertes PDF aus Word, Docs oder Generator | wird von Workday, Greenhouse, Lever zuverlässig gelesen | mittel: Ratgeberkonsens |
| Tabellen für Kerninhalte | Zellen in falscher Reihenfolge | mittel: mehrere unabhängige Blogs |
| Mehrspalten-Layout | linke und rechte Spalte vermischen sich | mittel: wie oben |
| Kontaktdaten in Kopf-/Fußzeile | werden oft ignoriert oder verworfen | mittel: wie oben |
| Nicht-Standard-Überschriften („Mein Weg“) | Abschnitt wird nicht zugeordnet | abgeleitet aus der Funktionsweise; softgarden-Hinweis zur Formatabhängigkeit |
| DOCX statt PDF | gilt als sicherste Variante für ältere, kleinere Systeme | niedrig: Ratgeberkonsens |

Quellen der Tabelle: [resumemate](https://www.resumemate.io/blog/pdf-vs-docx-for-resumes-in-2025-what-recruiters-ats-really-prefer/), [jobwizard](https://jobwizard.ai/blog/how-to-optimize-your-resume-for-ats-systems-in-2026-872842), [atsresumeai](https://www.atsresumeai.com/blog/ats-resume-formatting-guide). Die Belegstärke ist dünn: Fast alles stammt aus SEO-Karriereblogs ohne eigene Testmethodik. Deshalb gelten diese Regeln im Projekt als „plausibel, empirisch zu prüfen“ – der ATS-Prüfer testet die eigenen Vorlagen mit Tika und OpenResume, bevor eine Regel als belegt gilt (Kapitel 12). Da ein einspaltiger, tabellenfreier Lebenslauf nichts kostet, ist die konservative Wahl trotzdem richtig.

### 4.3 Ranking und Screening: System für System

Nach dem Parsing kommt – manchmal – eine Bewertung. Hier unterscheiden sich die Systeme deutlich.

**Personio.** Entgegen verbreiteter Annahme hat Personio Stand 2026 kein natives semantisches Ranking von Bewerbungen gegen die Stellenbeschreibung. Die automatische Vorselektion läuft über regelbasierte Knockout-Fragen im Bewerbungsformular ([Personio Community: AI im Recruiting](https://community.personio.de/recruiting-2/ai-im-personio-recruiting-bereich-13294), [trusted.de](https://trusted.de/personio)); eine automatische Absage bei Nichterfüllung frei definierter Muss-Kriterien aus der Anzeige gibt es nicht – Kunden fragten danach, ohne dass Personio konkrete Pläne nannte ([Personio Community: automatische Absage](https://community.personio.de/recruiting-2/automatische-absage-6551)). Das kann sich ändern: Im April 2026 hat Personio das Münchner Recruiting-KI-Start-up aurio übernommen (Sourcing, Screening-Automatisierung, Skills Intelligence); erste Funktionen wurden für „später im Jahr“ angekündigt und waren im September 2026 nicht ausgerollt ([Munich Startup](https://www.munich-startup.de/en/news/personio-acquires-recruiting-ai-startup-aurio), [Personio-Pressemitteilung](https://www.personio.com/about-personio/press/personio-profitability-acquisition-aurio/); vom Faktenprüfer nicht gegengeprüft). Personio nennt 16.000 Kunden, 9.000 davon mit Recruiting-Funktion (Herstellerangabe). Konsequenz: Für Personio-Ziele zählt heute das korrekte Beantworten der Knockout-Fragen mehr als jede Keyword-Dichte; die Architektur darf sich aber nicht auf dieses Verhalten verlassen und muss ein späteres KI-Ranking ohne Umbau aufnehmen.

**SAP SuccessFactors.** Die KI-Schicht Joule (separate AI-Units-Lizenz, neue „Candidate Workbench“) bietet Skills-Extraktion aus Lebensläufen, Candidate-Job-Matching und Ranking über einen Knowledge Graph statt reinem Keyword-Abgleich; 2025 kam ein „Suggest Skills“-Button hinzu, der Kandidaten weitere passende Skills vorschlägt ([SAP Community](https://community.sap.com/t5/human-capital-management-blog-posts-by-sap/successfactors-ai-ai-based-skill-matching-in-recruiting-set-up-and-hints/ba-p/13984884), [LeverX](https://leverx.com/newsroom/ai-recruiting-in-sap-successfactors)). Ob ein Arbeitgeber die Lizenz hat, ist von außen nicht erkennbar. Konsequenz: Skills so benennen, wie sie in einer Taxonomie stehen (gängiger Name des Werkzeugs, der Methode, des Zertifikats), nicht nur so, wie die Anzeige sie zufällig formuliert – und beides, wenn es abweicht.

**Workday (HiredScore).** HiredScore vergibt ein A-D-Grading für die Passung und findet Altbewerber, CRM-Kontakte und interne Mitarbeiter wieder. Die Herstellerangaben von 54 % mehr Recruiter-Kapazität und 57 % weniger Screening-Zeit sind Marketingzahlen ohne unabhängige Prüfung ([Workday](https://www.workday.com/en-us/products/talent-management/ai-recruiting.html), [HiredScore-Datenblatt](https://www.workday.com/content/dam/web/en-us/documents/datasheets/hiredscore-ai-recruiting.pdf)). Workday befüllt zudem Formularfelder aus dem geparsten Lebenslauf vor; jedes Feld ist vor dem Absenden gegen das Kandidatenprofil zu prüfen (Regel für den Boten, Kapitel 15).

**SmartRecruiters (SmartAssistant).** Match Score von 1 bis 5 Sternen auf Basis einer Skills-Taxonomie von rund 14.000 Skills; nur recruiterseitig sichtbar und laut dem AI-Whitepaper des Anbieters ohne automatisierte Ablehnungs- oder Einstellungsentscheidung, nur zur Priorisierung ([SmartRecruiters](https://www.smartrecruiters.com/recruiting-software/ai-recruiting-technology/), [atsverification](https://atsverification.com/ats/smartrecruiters/)).

**Teamtailor (Co-Pilot).** Der am besten dokumentierte LLM-Screener eines in Europa verbreiteten ATS: Co-Pilot nutzt OpenAI-GPT-Modelle über die Enterprise-API mit Zero Data Retention und prüft Kandidaten gegen frei definierte Kriterien. Pro Kriterium gibt es Haken, Kreuz oder „unknown“ plus schriftliche Begründung; Sortierung nach Anzahl erfüllter Kriterien, manuelles Override, automatisches Re-Screening bei Datenänderung ([Co-Pilot Screening](https://support.teamtailor.com/en/articles/10209597-co-pilot-candidate-screening), [Co-Pilot Überblick](https://support.teamtailor.com/en/articles/8403166-co-pilot-ai-features-overview)). Konsequenz: Ein LLM sucht Textbelege für Kriterien. Ein Kriterium, das nur implizit erfüllt ist („fünf Jahre Führungserfahrung“, verteilt über drei Stationen ohne Summe), bekommt „unknown“. Explizit machen, was explizit gefragt ist.

**Greenhouse und Lever.** Greenhouse lehnt nicht algorithmisch nach Score ab; jede Ablehnung ist eine Recruiter-Entscheidung. Automatisches Ablehnen oder Weiterleiten über Knockout-Fragen (Visastatus, Zertifikate, Umzugsbereitschaft) gibt es nur in den Plus-/Pro-Tarifen – viele Arbeitgeber nutzen es also gar nicht ([Jobscan zu Greenhouse](https://www.jobscan.co/blog/greenhouse-ats-what-job-seekers-need-to-know/), [notchresume](https://notchresume.com/resources/greenhouse-job-application.html)). Für Lever wurde kein KI-Ranking dokumentiert.

**softgarden, rexx systems, d.vinci, onlyfy one, JOIN.** Für keinen dieser Anbieter fand die Recherche eine dokumentierte KI-Ranking-Schicht; belegt ist nur das Textkernel-Parsing bei softgarden und d.vinci. Bis zum Gegenbeweis gilt: Parsing plus Mensch.

| System | Score | Automatische Ablehnung | Worauf das System reagiert |
|---|---|---|---|
| Personio | keiner (09/2026) | nur Knockout-Fragen | Formularantworten, Parsing-Qualität |
| SAP SuccessFactors + Joule | Ranking, lizenzabhängig | nicht belegt | Skills-Bezeichnungen, Rollentitel |
| Workday + HiredScore | A–D | nicht belegt | Titel, Skills, Erfahrung; Autofill-Felder |
| SmartRecruiters | 1–5 Sterne | nein (laut Anbieter) | Skills-Taxonomie |
| Teamtailor Co-Pilot | erfüllte Kriterien | nein, Override möglich | explizite Belege je Kriterium |
| Greenhouse | keiner | Knockout nur Plus/Pro | Formularantworten |
| softgarden, rexx, d.vinci, onlyfy, JOIN | nicht belegt | nicht belegt | Parsing-Qualität |

Drei Gemeinsamkeiten:

1. Die einzige echte Automatik-Absage ist die Knockout-Frage. Sie ist deterministisch, prüft harte Fakten (Arbeitserlaubnis, Sprache, Verfügbarkeit, Gehalt, Standort) und liegt vor jedem Text. Sie gehört als Pass/Fail-Prüfung in den Matcher, bevor der Autor eine Zeile schreibt (Kapitel 9).
2. Scores priorisieren, sie entscheiden nicht. Ein gutes Ranking bringt die Bewerbung früher vor ein Menschenauge – bei hunderten Bewerbungen ein realer Vorteil, aber kein Ersatz für Passung.
3. Alle dokumentierten KI-Schichten arbeiten mit Skills-Taxonomien oder Kriterienlisten, also semantisch. Sie belohnen klare, belegte Aussagen im Vokabular der Stelle, nicht Wiederholung.

### 4.4 Mythen und Fakten

1. **Mythos: „75 % aller Bewerbungen werden vom ATS automatisch abgelehnt.“** Die Zahl stammt aus einer 2012 verbreiteten Verkaufsbehauptung der Firma Preptel, die 2013 unterging; Methodik und Stichprobe wurden nie veröffentlicht ([The Interview Guys](https://blog.theinterviewguys.com/ats-resume-rejection-myth/)). In einer Enhancv-Befragung bestätigen 92 % der Recruiter, dass ihr ATS nicht automatisch nach Formatierung, Design oder Match-Score ablehnt; 68 % hörten die Zahl zuerst von Bewerbern in sozialen Medien, 20 % von Karrierecoaches und Resume-Services ([HR.com](https://www.hr.com/en/app/blog/2026/04/ats-rejection-myth-debunked-92-of-recruiters-confi_mntajhyq.html)). Der Hauptgrund für übersehene Bewerbungen ist die Menge: Gefragte Stellen ziehen binnen Tagen hunderte bis tausende Bewerbungen. **Konsequenz:** Kein „ATS-Hack“ ersetzt Passung und Sichtbarkeit; sauberes Parsing ist Pflicht, nicht Wunderwaffe.

2. **Mythos: „PDF ist Gift für ATS.“** Moderne Systeme lesen textbasierte PDFs zuverlässig; das Problem sind bildbasierte PDFs und Exporte aus Design-Tools ohne Text-Layer ([resumemate](https://www.resumemate.io/blog/pdf-vs-docx-for-resumes-in-2025-what-recruiters-ats-really-prefer/); Ratgeberkonsens). **Konsequenz:** PDF aus einem Generator mit echtem Text-Layer ist Standard, DOCX wird zusätzlich vorgehalten (Kapitel 13).

3. **Mythos: „Mehr Keywords, besserer Score.“** Keine der dokumentierten KI-Schichten zählt Wörter; Joule, SmartAssistant und Co-Pilot gleichen Skills und Kriterien semantisch ab (4.3). Wiederholung ohne Beleg erzeugt beim Co-Pilot ein „unknown“ und beim Menschen den Eindruck eines generischen Textes. **Konsequenz:** Jeder Begriff aus der Anzeige, der aufgenommen wird, muss im Kandidatenprofil belegt sein und einmal an der richtigen Stelle stehen.

4. **Mythos: „Weiße Schrift und versteckte Anweisungen an die KI helfen.“** Laut Greenhouse-Report „AI in Hiring 2025“ gaben 41 % von 1.200 befragten US-Bewerbern an, versteckten Text probiert zu haben; tatsächlich enthielten im ersten Halbjahr 2025 nur rund 1 % der eingereichten Lebensläufe Weißtext. 65 % der Hiring Manager berichten, KI-gestützte Täuschungsversuche erkannt zu haben, 22 % davon konkret versteckte Prompt-Injections ([The Interview Guys](https://blog.theinterviewguys.com/job-seekers-are-hiding-secret-text-in-their-resumes/)). Eine Studie vom Juni 2026 soll zeigen, dass der Ranking-Vorteil kollabiert, sobald viele Kandidaten es gleichzeitig versuchen (unbestätigt; die Quelle konnte vom Faktenprüfer nicht verifiziert werden). **Konsequenz:** Der Bewerbungsagent setzt weder Weißtext noch Prompt-Injection noch unsichtbare Keywords ein – Entdeckungsrisiko, sinkende Wirksamkeit, Reputationsschaden und Widerspruch zum Leitsatz „echte Person statt Bot“. Der Kritiker prüft Entwürfe aktiv darauf (Kapitel 11), der ATS-Prüfer scannt die fertige Datei auf unsichtbaren Text (Kapitel 12).

5. **Mythos: „Kreatives Layout zeigt Persönlichkeit.“** Zweispaltige Designs, Tabellen, Icons und Kompetenzbalken sind für den Parser Rauschen oder Bild (4.2). Persönlichkeit gehört in den Inhalt und ins Anschreiben, nicht ins Raster.

6. **Fakt: KI-Screener haben nachgewiesene Verzerrungen.** Mehrere Studien 2024–2026 zeigen demografische Verzerrungen bei LLM-basiertem Resume-Screening nach Name, Ethnie und Geschlecht ([FAIRE, arXiv 2504.01420](https://arxiv.org/pdf/2504.01420), [PeerJ Computer Science](https://peerj.com/articles/cs-3628/)). Das liegt außerhalb der Kontrolle des Kandidaten. **Konsequenz:** Nicht optimierbar, aber ein Grund mehr, AGG-sensible Angaben (Foto, Geburtsdatum, Familienstand) nicht ungefragt zu liefern (Kapitel 5 und 16) und Rückmeldequoten im Tracker pro ATS-Typ auszuwerten (Kapitel 15).

7. **Fakt: Kandidaten-KI ist normal geworden und wird erkannt.** 43,2 % der Bewerber nutzten laut softgarden-Umfrage (Mai bis Juli 2025, n = 6.929) KI für das Anschreiben, eine Verdreifachung seit 2023 ([ingenieur.de](https://www.ingenieur.de/karriere/bewerbung/ki-und-karriere-wie-kuenstliche-intelligenz-den-bewerbungsprozess-praegt/)). Zugleich geben 65 % der Hiring Manager an, KI-Täuschung zu erkennen (Punkt 4). **Konsequenz:** Der Unterschied liegt nicht in „KI ja oder nein“, sondern in generisch versus persönlich. Dafür gibt es Stimmprofil und Story-Bank (Kapitel 8) und die Anti-Generik-Regeln (Kapitel 11).

### 4.5 EU AI Act: Stand September 2026 und was er für Kandidaten bedeutet

KI-Systeme für Einstellung und Auswahl – Anzeigen ausspielen, Bewerbungen sichten und filtern, Bewerber bewerten – sind nach Anhang III Nr. 4 Buchst. a der KI-Verordnung (AI Act) Hochrisiko-Systeme ([anwalt.de](https://www.anwalt.de/rechtstipps/ki-im-bewerbungsverfahren-ab-2026-was-ai-act-dsgvo-und-betrvg-verlangen-277758.html)).

Der Zeitplan hat sich 2026 verschoben. Der „Digital Omnibus“ (Verordnung (EU) 2026/1744, Amtsblatt 24.7.2026, in Kraft seit 27.7.2026) verlegt die Hochrisiko-Pflichten für eigenständige Systeme nach Anhang III – also auch Recruiting-KI – vom 2.8.2026 auf den 2.12.2027; für in Produkte eingebettete Systeme nach Anhang I gilt der 2.8.2028 ([Gibson Dunn](https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/), [KPMG](https://kpmg.com/at/de/insights/2026/07/digital-omnibus-on-ai.html), [FGS](https://www.fgs.de/news-and-insights/blog/detail/eu-ai-act-was-ab-dem-2-august-2026-gilt-und-was-verschoben-wurde), [TÜV Rheinland](https://consulting.tuv.com/aktuelles/ki-im-fokus/digital-omnibus-ki-verordnung-fristen); vom Faktenprüfer bestätigt). Unverändert seit 2.8.2026 gelten die Transparenzpflichten des Art. 50, die Pflichten für General-Purpose-AI-Modelle (seit August 2025) und die Verbote nach Art. 5 (seit Februar 2025) ([Jones Walker](https://www.joneswalker.com/en/insights/blogs/ai-law-blog/yes-august-2-still-matters-the-eu-approved-a-high-risk-ai-delay-but-most-trans.html?id=102nbon)).

Für Arbeitgeber gilt zusätzlich Art. 26 Abs. 7 KI-VO: Wer ein Hochrisiko-KI-System am Arbeitsplatz einsetzt, muss vorab die Arbeitnehmervertretung und die betroffenen Beschäftigten informieren; das ergänzt die Mitbestimmung nach § 87 Abs. 1 Nr. 6 BetrVG, ersetzt sie aber nicht ([Bitkom-Leitfaden](https://www.bitkom.org/sites/main/files/2026-02/bitkom-leitfaden-kuenstliche-intelligenz-und-mitbestimmung.pdf), [anwalt.de](https://www.anwalt.de/rechtstipps/ki-im-bewerbungsverfahren-ab-2026-was-ai-act-dsgvo-und-betrvg-verlangen-277758.html)). Ob externe Bewerber daraus einen eigenen Auskunftsanspruch ableiten können, ist in der Recherche nicht eindeutig belegt.

Und der Kandidat selbst? Wer als natürliche Person ein KI-System ausschließlich für persönliche, nicht berufliche Zwecke nutzt, fällt nach Art. 2 Abs. 10 KI-VO nicht unter die Verordnung ([dejure](https://dejure.org/gesetze/KI-Verordnung/2.html), [ai-act-law.eu](https://ai-act-law.eu/de/artikel/2/)). Der Bewerbungsagent ist damit kein Betreiber im Sinne des AI Act. Ob Art. 50 eine Kennzeichnung KI-generierter Anschreiben verlangt, ist nach den gefundenen Quellen zweifelhaft, weil die Norm auf Inhalte zur Information der Öffentlichkeit zielt ([forum-institut](https://forum-institut.de/eu-ai-act-2-august-2026/ki-kennzeichnung-nach-artikel-50), [ai-act-law.eu Art. 50](https://ai-act-law.eu/de/artikel/50/)). Den vollständigen Compliance-Katalog enthält Kapitel 16.

Was das praktisch bedeutet:

1. Bis Dezember 2027 gibt es keine durchsetzbare Pflicht der Arbeitgeber, ihr Screening zu dokumentieren, menschlich zu überwachen oder Kandidaten zu erklären. Erwartungen an Transparenz sind bis dahin unrealistisch; danach steigt der Druck auf Arbeitgeber und ATS-Anbieter und damit die Wahrscheinlichkeit nachvollziehbar dokumentierter Scores.
2. Ein Anspruch des Kandidaten, „seinen Score zu sehen“, ergibt sich aus der Recherche nicht. Der Agent verspricht das nicht.
3. Der Agent kann recherchieren, ob ein Zielunternehmen öffentlich über KI im Bewerbungsprozess informiert (Karriereseite, Datenschutzhinweise des Bewerbungsformulars, Presse). Das ist ein Signal fürs Erwartungsmanagement, kein Blocker.
4. Die Verschiebung ändert nichts an den Regeln in 4.7: Parsing-Qualität, Knockout-Fragen und semantische Passung wirken unabhängig vom Rechtsrahmen.

**Entscheidung:** Der Rechercheur erhält ein optionales Feld `ki_screening_hinweis` (ja, nein, unbekannt; mit Fundstelle), das im MVP leer bleibt und ab v2 aus den Datenschutzhinweisen des Bewerbungsformulars extrahiert wird. **Begründung:** Für die einzelne Bewerbung geringer Nutzen, aber wertvoll für die Auswertung im Tracker (Rücklauf mit und ohne KI-Screening). **Alternative:** Ganz weglassen – siehe offene Frage in 4.9.

### 4.6 ATS-Erkennung pro Firma

Das ATS bestimmt drei Dinge: den Bewerbungsweg (E-Mail, Formular, Portal-Konto; Kapitel 15), die Wahrscheinlichkeit von Knockout-Fragen und Autofill (Kapitel 9 und 15) und die Formatannahmen (Kapitel 13). Der Rechercheur ermittelt es im Schritt „recherchiert“ aus der Bewerbungs-URL der Anzeige.

| ATS | URL-Muster | Konfidenz |
|---|---|---|
| Personio | `{firma}.jobs.personio.de` bzw. `.com` | hoch |
| Workday | `{firma}.wd{N}.myworkdayjobs.com/{board}` | hoch |
| Greenhouse | `boards.greenhouse.io/{firma}` | hoch |
| Lever | `jobs.lever.co/{firma}` | hoch |
| Recruitee | `{firma}.recruitee.com` | hoch |
| SmartRecruiters | `careers.smartrecruiters.com/{firma}` | hoch |
| Teamtailor | `{firma}.teamtailor.com`, `career.teamtailor.com` | mittel |
| softgarden | `{firma}.career.softgarden.de` | niedrig |
| SAP SuccessFactors | eigene Firmendomain oder `career{N}.sapsf.com` | niedrig (unbestätigt) |
| rexx systems | kein einheitliches Muster; teils `/jobs/` auf Firmendomain | – |
| d.vinci | White-Label auf Kundendomain, kein Muster | – |
| onlyfy one, JOIN | kein belegtes Muster | – |

Belege: Die Muster für Greenhouse, Lever, Recruitee, Personio, SmartRecruiters und Workday stammen aus der Dokumentation zweier Karriereseiten-Scraper ([Apify Greenhouse/Lever/Ashby](https://apify.com/webdata_labs/greenhouse-lever-ashby-jobs-scraper), [Apify Career Site Scraper](https://apify.com/santamaria-automations/career-site-jobs-scraper)). Das softgarden-Muster ist nur durch softgardens eigene Karriereseite belegt ([softgarden](https://softgarden.career.softgarden.de/en/)), nicht kundenübergreifend. Für rexx und d.vinci liegen Karriereseiten auf Firmendomains ([rexx Jobs](https://www.rexx-systems.com/jobs/), [d.vinci Whitepaper](https://www.dvinci.de/docs20/d.vinci_Whitepaper_Karrierewebsite.pdf)). Zu Teamtailor widersprechen sich zwei Recherchestränge (öffentliche Job-API mit oder ohne Token); das bleibt vor Nutzung an einer echten Teamtailor-Seite zu testen.

Fallback 1 ist HTML-Fingerprinting. Die Open-Source-Datenbank webappanalyzer (Community-Fork der 2023 privat gewordenen Wappalyzer-Fingerprints) führt die Kategorie 101 „Recruitment & staffing“ mit Fingerprints für Personio, Greenhouse, Lever, Recruitee, SmartRecruiters, Teamtailor und „onlyfy Application Manager“; Workday ist nur als CRM (Kategorie 53) erfasst; für softgarden, rexx systems, d.vinci, SAP SuccessFactors und JOIN gibt es keine Fingerprints ([webappanalyzer](https://github.com/enthec/webappanalyzer), [categories.json](https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json); vom Faktenprüfer verifiziert). Für die deutschen Top-Anbieter muss das Projekt eigene Fingerprints sammeln (Fußzeilen-Hinweise, Skript-URLs, Formularfeldnamen) – eine Aufgabe für Phase 0 anhand der ersten 50 gefundenen Anzeigen (Kapitel 19).

Fallback 2 ist eine LLM-Klassifikation mit Claude Haiku 4.5 (`claude-haiku-4-5`) aus Seitentitel, Fußzeile und Skript-Liste; das Ergebnis gilt als „vermutet“ mit Konfidenz höchstens 0,6. Fallback 3 ist der Wert „unbekannt“ mit konservativen Standardannahmen. Er ist ein gültiges Ergebnis, kein Fehler.

**Entscheidung:** Vierstufige Erkennung (URL-Regex, HTML-Fingerprint, LLM-Klassifikation, „unbekannt“), Regelwerk als versionierte YAML-Datei mit quartalsweiser Prüfung. **Begründung:** Regex deckt die internationalen Systeme fast kostenlos ab; Fingerprints liegen für sieben Anbieter fertig vor; die LLM-Stufe kostet pro Anzeige Bruchteile eines Cents (Kapitel 18) und fängt den langen Schwanz; „unbekannt“ verhindert falsche Sicherheit. **Alternative:** Nur Regex plus manuelle Zuordnung im Review-Cockpit – weniger Code, aber rexx und d.vinci blieben dauerhaft Handarbeit.

```yaml
# ats_detection_rules.yaml – Version 2026-09; Review jedes Quartal
- vendor: personio
  url_patterns: ['^https?://[a-z0-9-]+\.jobs\.personio\.(de|com)/']
  confidence: 0.95
  fingerprint: webappanalyzer:Personio
- vendor: workday
  url_patterns: ['^https?://[a-z0-9-]+\.wd\d+\.myworkdayjobs\.com/']
  confidence: 0.95
  fingerprint: null          # webappanalyzer führt Workday nur als CRM
- vendor: greenhouse
  url_patterns: ['^https?://boards\.greenhouse\.io/']
  confidence: 0.95
  fingerprint: webappanalyzer:Greenhouse
- vendor: lever
  url_patterns: ['^https?://jobs\.lever\.co/']
  confidence: 0.95
  fingerprint: webappanalyzer:Lever
- vendor: recruitee
  url_patterns: ['^https?://[a-z0-9-]+\.recruitee\.com/']
  confidence: 0.95
  fingerprint: webappanalyzer:Recruitee
- vendor: smartrecruiters
  url_patterns: ['^https?://careers\.smartrecruiters\.com/']
  confidence: 0.95
  fingerprint: webappanalyzer:SmartRecruiters
- vendor: teamtailor
  url_patterns: ['^https?://([a-z0-9-]+|career)\.teamtailor\.com/']
  confidence: 0.8
  fingerprint: webappanalyzer:Teamtailor
- vendor: softgarden
  url_patterns: ['^https?://[a-z0-9-]+\.career\.softgarden\.de/']
  confidence: 0.6              # nur durch softgardens eigene Seite belegt
  fingerprint: eigen:TODO      # in Phase 0 sammeln
- vendor: successfactors
  url_patterns: ['^https?://career\d*\.sapsf\.com/']
  confidence: 0.5              # unbestätigt
  fingerprint: eigen:TODO
- vendor: onlyfy
  url_patterns: []
  fingerprint: webappanalyzer:onlyfy Application Manager
- vendor: [rexx, dvinci, join]
  url_patterns: []
  fingerprint: eigen:TODO      # sonst LLM-Klassifikation oder "unbekannt"
```

Das Ergebnis landet im Datensatz der Stelle (Datenmodell in Kapitel 7):

```yaml
ats_profile:
  vendor: personio            # personio | successfactors | softgarden | rexx | workday |
                              # greenhouse | lever | smartrecruiters | teamtailor | recruitee |
                              # dvinci | onlyfy | join | sonstige | unbekannt
  detected_by: url_regex      # url_regex | html_fingerprint | llm_klassifikation | manuell
  confidence: 0.95
  parser_hint: textkernel     # textkernel | eigen | unbekannt
  ai_ranking: nein            # nein | optional | ja | unbekannt
  knockout_expected: true
  format_default: [pdf, docx]
  ki_screening_hinweis: unbekannt
  rules_version: "2026-09"
  checked_at: "2026-09-08"
```

### 4.7 Regelkatalog: was wirklich hilft, was Aberglaube ist

Der Katalog hat drei Klassen: Regeln, die nach Herstellerdokumentation oder Rechtslage wirken; Regeln, die als Ratgeberkonsens plausibel sind und nichts kosten; und Verbote. Jede Regel nennt ihre Belegstärke, damit spätere Tests (Kapitel 12) sie bestätigen oder streichen können.

**A. Was wirklich hilft**

| Nr. | Regel | Wirkt auf | Beleg |
|---|---|---|---|
| R01 | Exakte Fachbegriffe der Anzeige (Werkzeuge, Zertifikate, Rollentitel) verwenden – nur wenn im Kandidatenprofil belegt, je einmal an der passenden Stelle; keine ganzen Sätze der Anzeige übernehmen | Skills-Matching, LLM-Screener, Mensch | mittel (Herstellerdoku Joule, SmartAssistant, Co-Pilot); Urheberrecht an Anzeigentexten möglich ([urheberrecht.de](https://www.urheberrecht.de/kuenstliche-intelligenz/)) |
| R02 | Skills zusätzlich in Taxonomie-Schreibweise nennen (gängiger Produktname, Abkürzung und Langform je einmal) | Joule, SmartAssistant, HiredScore | mittel |
| R03 | Muss-Kriterien explizit und summiert beantworten (Jahre, Sprachniveau, Ort, Führungsspanne) | Co-Pilot, Knockout, Mensch | hoch (Teamtailor-Doku) |
| R04 | Standardüberschriften: Berufserfahrung, Ausbildung, Kenntnisse, Sprachen, Weiterbildung; bei englischer Anzeige Professional Experience, Education, Skills | Parser | abgeleitet plus Ratgeberkonsens |
| R05 | Einspaltig; keine Tabellen für Kerninhalte; Kontaktdaten im Fließtext, nicht in Kopf-/Fußzeile; keine Icons oder Balken als Informationsträger | Parser | mittel (Blogs, empirisch zu prüfen) |
| R06 | PDF mit Text-Layer aus dem Generator; DOCX aus derselben Quelle vorhalten | Parser | mittel |
| R07 | Knockout-Fragen vollständig und wahrheitsgemäß aus den Standardantworten des Kandidatenprofils beantworten; Unbekanntes wird „Rückfrage offen“ | Personio, Greenhouse Plus/Pro | hoch |
| R08 | Vorbefüllte Formularfelder (Workday) vor dem Absenden gegen das Kandidatenprofil prüfen | Workday | abgeleitet |
| R09 | Nichts behaupten, was der Master-Lebenslauf nicht hergibt – Täuschung über einstellungsrelevante Tatsachen erlaubt die Anfechtung des Arbeitsvertrags nach § 123 BGB, auch Jahre später | alle | hoch ([anwalt24](https://www.anwalt24.de/fachartikel/arbeit-und-betrieb/46090), [Kliemt](https://kliemt.blog/2016/10/05/nur-schoenfaerberei-luege-im-lebenslauf-und-drastische-spaetfolgen/)) |
| R10 | Sprache der Unterlagen gleich Sprache der Anzeige; einheitliches Datumsformat (MM/JJJJ), „bis heute“ statt offener Enden | Parser, LLM-Screener | abgeleitet |

**B. Aberglaube oder verboten**

| Nr. | Praxis | Warum nicht |
|---|---|---|
| V01 | Keyword-Stuffing (Begriffe mehrfach, Keyword-Blöcke, Fußnoten voller Skills) | kein System zählt Wörter; wirkt generisch (4.3, 4.4) |
| V02 | Weißtext, Schriftgröße 1, versteckte Prompt-Anweisungen | Täuschung; wird erkannt; Wirkung sinkt mit Verbreitung (4.4) |
| V03 | „ATS-Score 100 %“ als Ziel externer Scan-Tools | US-/Englisch-fokussierte Heuristiken, keine Abbildung deutscher ATS (4.2) |
| V04 | Bild-PDF, Scan, Export aus Design-Tools ohne Text-Layer | kein Text für den Parser |
| V05 | Zweispaltiges Layout, Tabellen, Kompetenzbalken, Icons | Parsing-Reihenfolge bricht; Grafik ist kein Text |
| V06 | Kontaktdaten nur in der Kopfzeile | wird oft verworfen |
| V07 | Anschreiben mit Keywords vollstopfen | Parser liest das Anschreiben nicht; Menschen schon |
| V08 | Qualifikationen „aufrunden“, Titel anpassen, Lücken kaschieren | § 123 BGB; Leitsatz des Projekts |

**C. ATS-spezifische Zusatzregeln**

| ATS | Zusatzregel |
|---|---|
| Personio | Knockout-Fragen vor dem Schreiben prüfen (R07); Keyword-Dichte ist irrelevant; Verhalten nach aurio-Rollout beobachten |
| SAP SuccessFactors, Workday, SmartRecruiters | R01 und R02 strikt; Skills als eigener Abschnitt mit Taxonomie-Namen |
| Teamtailor | R03 strikt: jedes Kriterium der Anzeige bekommt einen expliziten Beleg im Lebenslauf |
| Greenhouse, Lever | R07, falls Formular Knockout-Fragen zeigt; sonst Standard |
| softgarden, rexx, d.vinci, onlyfy, JOIN | Standard (Parsing plus Mensch); bei unbekanntem Parser PDF und DOCX anbieten |
| unbekannt | konservativer Standard: alle A-Regeln, beide Formate erzeugen |

Maschinenlesbar für ATS-Prüfer und Kritiker (Auszug; vollständige Datei in Kapitel 12):

```yaml
# ats_rules.yaml – Version 2026-09
- id: R01
  regel: exakte Fachbegriffe der Anzeige, belegt im Kandidatenprofil, je einmal
  beleg: mittel
  gilt_fuer: [alle]
  prueft: ATS-Prüfer        # Abgleich Anzeige <-> Lebenslauf (Kapitel 12)
  schreibt: Autor           # Kapitel 11
- id: R05
  regel: einspaltig, keine Tabellen, Kontaktdaten im Fließtext
  beleg: mittel
  gilt_fuer: [alle]
  prueft: ATS-Prüfer        # Test-Parsing mit Tika und OpenResume
  schreibt: Setzer          # Vorlagen in Kapitel 13
- id: R07
  regel: Knockout-Fragen wahrheitsgemäß aus Standardantworten; sonst Rückfrage offen
  beleg: hoch
  gilt_fuer: [personio, greenhouse]
  prueft: Matcher           # Pass/Fail vor Status "ausgewählt" (Kapitel 9)
  schreibt: Bote            # Formular-Vorbefüllung (Kapitel 15)
- id: V02
  regel: kein Weißtext, keine versteckten Anweisungen, keine unsichtbaren Keywords
  beleg: hoch
  gilt_fuer: [alle]
  prueft: [Kritiker, ATS-Prüfer]
  status_bei_verstoss: geprüft -> zurück an Autor
```

### 4.8 Was die anderen Kapitel hieraus übernehmen

- Kapitel 7 (Architektur): `ats_profile` im Datenmodell der Stelle; Regelwerke `ats_detection_rules.yaml` und `ats_rules.yaml` als versionierte Konfiguration mit Prüfdatum.
- Kapitel 9 (Matcher): Knockout-Prüfung als Pass/Fail vor „ausgewählt“; Muss-Kriterien der Anzeige strukturiert extrahieren.
- Kapitel 10 (Rechercheur): vierstufige ATS-Erkennung; optionales Feld `ki_screening_hinweis`.
- Kapitel 11 (Autor, Kritiker): R01 bis R03 und R09 als Schreibregeln; V01, V02, V08 als harte Prüfpunkte.
- Kapitel 12 (ATS-Prüfer): Keyword-Abgleich nur gegen den Lebenslauf; Zwei-Parser-Gegenprobe mit Apache Tika 4.0.0 und OpenResume; Scan auf unsichtbaren Text; empirische Bestätigung der A-Regeln.
- Kapitel 13 (Setzer): einspaltige Vorlagen mit Standardüberschriften; PDF mit Text-Layer und DOCX aus derselben Quelle.
- Kapitel 15 (Bote, Tracker): Autofill-Kontrolle; Auswertung der Rückmeldungen nach ATS-Typ.
- Kapitel 16 (Recht): AI Act, § 123 BGB, Urheberrecht an Anzeigentexten im Compliance-Katalog.
- Kapitel 20 (Risiken): Regelveraltung durch schnelle Produktänderungen der Anbieter (Beispiel aurio), Bias in Arbeitgeber-KI außerhalb unserer Kontrolle, dünne Beleglage der Formatregeln.

### 4.9 Default-Annahmen und offene Fragen (für Kapitel 22)

1. **DGFP-Marktanteile:** Default ist die Priorisierung Personio, SAP SuccessFactors, softgarden, rexx. Bitte den Studientext der DGFP „Recruiting-Strukturen 2025“ vor Phase 0 direkt einsehen und die Reihenfolge bestätigen oder korrigieren.
2. **Formate bei unbekanntem ATS:** Default ist, dass der Setzer immer PDF und DOCX erzeugt und der Bote PDF anhängt, DOCX nur auf ausdrückliche Anforderung des Formulars. Alternativ nur PDF.
3. **KI-Screening-Recherche:** Default ist das leere Feld `ki_screening_hinweis` im MVP und die Befüllung ab v2. Alternativ ganz streichen, wenn dir das Signal nichts wert ist.
4. **Budget für Scan-Tools:** Default ist strikt kostenlos (Apache Tika, OpenResume). Jobscan oder Eden AI nur, falls du einen externen Vergleichsmaßstab wünschst.
5. **Sprache der Bewerbungen:** Default ist Deutsch mit englischen Unterlagen bei englischer Anzeige (R10). Falls du dich ausschließlich in einer Sprache bewirbst, vereinfacht das Vorlagen und Regelkatalog.

**Quellen dieses Kapitels:**

- DGFP: Recruiting-Strukturen 2025 – https://www.dgfp.de/aktuell/recruiting-strukturen-2025-recruiting-wird-strukturierter-datengetriebener-und-technologischer
- yena.ai: KI im Recruiting, Studie DACH 2026 – https://www.yena.ai/de/blog/ki-im-recruiting-studie-dach-2026
- Textkernel: Sovren – https://www.textkernel.com/sovren/
- Textkernel: Parser (Produktseite) – https://www.textkernel.com/de/produkte-loesungen/parser/
- Textkernel: Developer-Dokumentation Parser – https://developer.textkernel.com/Parser/master/
- Personio Support: CV parsing for candidate profiles – https://support.personio.de/hc/en-us/articles/360010193018-CV-parsing-for-candidate-profiles
- softgarden Support: CV-Parsing – https://support.softgarden.de/de/articles/680780-cv-parsing
- d.vinci: CV-Parsing – https://www.dvinci.de/bms/cv-parsing-leichter-bewerben-dank-intelligenter-recruiting-technologie/
- Apache Tika: CHANGES.txt – https://raw.githubusercontent.com/apache/tika/main/CHANGES.txt
- Apache Tika: Repository – https://github.com/apache/tika
- OpenResume: Repository – https://github.com/xitanggg/open-resume
- Jobscan-Preise (pitchmeai) – https://pitchmeai.com/blog/jobscan-pricing-plans
- Affinda-Preise (G2) – https://www.g2.com/products/resume-parser-by-affinda/pricing
- Eden AI: Best Resume Parser APIs – https://www.edenai.co/post/best-resume-parser-apis
- resumemate: PDF vs. DOCX – https://www.resumemate.io/blog/pdf-vs-docx-for-resumes-in-2025-what-recruiters-ats-really-prefer/
- jobwizard: ATS-Optimierung 2026 – https://jobwizard.ai/blog/how-to-optimize-your-resume-for-ats-systems-in-2026-872842
- atsresumeai: ATS Resume Formatting Guide – https://www.atsresumeai.com/blog/ats-resume-formatting-guide
- Personio Community: AI im Personio Recruiting-Bereich – https://community.personio.de/recruiting-2/ai-im-personio-recruiting-bereich-13294
- trusted.de: Personio – https://trusted.de/personio
- Personio Community: Automatische Absage – https://community.personio.de/recruiting-2/automatische-absage-6551
- Munich Startup: Personio acquires aurio – https://www.munich-startup.de/en/news/personio-acquires-recruiting-ai-startup-aurio
- Personio Pressemitteilung: Profitability, acquisition aurio – https://www.personio.com/about-personio/press/personio-profitability-acquisition-aurio/
- SAP Community: AI-based skill matching in Recruiting – https://community.sap.com/t5/human-capital-management-blog-posts-by-sap/successfactors-ai-ai-based-skill-matching-in-recruiting-set-up-and-hints/ba-p/13984884
- LeverX: AI Recruiting in SAP SuccessFactors – https://leverx.com/newsroom/ai-recruiting-in-sap-successfactors
- Workday: AI Recruiting – https://www.workday.com/en-us/products/talent-management/ai-recruiting.html
- Workday: HiredScore-Datenblatt – https://www.workday.com/content/dam/web/en-us/documents/datasheets/hiredscore-ai-recruiting.pdf
- SmartRecruiters: AI Recruiting Technology – https://www.smartrecruiters.com/recruiting-software/ai-recruiting-technology/
- atsverification: SmartRecruiters – https://atsverification.com/ats/smartrecruiters/
- Teamtailor Support: Co-Pilot Candidate Screening – https://support.teamtailor.com/en/articles/10209597-co-pilot-candidate-screening
- Teamtailor Support: Co-Pilot AI Features Overview – https://support.teamtailor.com/en/articles/8403166-co-pilot-ai-features-overview
- Jobscan: Greenhouse ATS – https://www.jobscan.co/blog/greenhouse-ats-what-job-seekers-need-to-know/
- notchresume: Greenhouse Job Application – https://notchresume.com/resources/greenhouse-job-application.html
- The Interview Guys: ATS Resume Rejection Myth – https://blog.theinterviewguys.com/ats-resume-rejection-myth/
- HR.com: ATS Rejection Myth Debunked – https://www.hr.com/en/app/blog/2026/04/ats-rejection-myth-debunked-92-of-recruiters-confi_mntajhyq.html
- The Interview Guys: Job Seekers Are Hiding Secret Text – https://blog.theinterviewguys.com/job-seekers-are-hiding-secret-text-in-their-resumes/
- FAIRE (arXiv 2504.01420) – https://arxiv.org/pdf/2504.01420
- PeerJ Computer Science: LLM Resume Screening Bias – https://peerj.com/articles/cs-3628/
- ingenieur.de: KI und Karriere – https://www.ingenieur.de/karriere/bewerbung/ki-und-karriere-wie-kuenstliche-intelligenz-den-bewerbungsprozess-praegt/
- anwalt.de: KI im Bewerbungsverfahren ab 2026 – https://www.anwalt.de/rechtstipps/ki-im-bewerbungsverfahren-ab-2026-was-ai-act-dsgvo-und-betrvg-verlangen-277758.html
- Gibson Dunn: EU AI Act Omnibus – https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/
- KPMG: Digital Omnibus on AI – https://kpmg.com/at/de/insights/2026/07/digital-omnibus-on-ai.html
- FGS: EU AI Act – was ab 2.8.2026 gilt – https://www.fgs.de/news-and-insights/blog/detail/eu-ai-act-was-ab-dem-2-august-2026-gilt-und-was-verschoben-wurde
- TÜV Rheinland: Digital Omnibus KI-Verordnung Fristen – https://consulting.tuv.com/aktuelles/ki-im-fokus/digital-omnibus-ki-verordnung-fristen
- Jones Walker: Yes, August 2 Still Matters – https://www.joneswalker.com/en/insights/blogs/ai-law-blog/yes-august-2-still-matters-the-eu-approved-a-high-risk-ai-delay-but-most-trans.html?id=102nbon
- Bitkom: Leitfaden KI und Mitbestimmung – https://www.bitkom.org/sites/main/files/2026-02/bitkom-leitfaden-kuenstliche-intelligenz-und-mitbestimmung.pdf
- dejure: Art. 2 KI-Verordnung – https://dejure.org/gesetze/KI-Verordnung/2.html
- ai-act-law.eu: Artikel 2 – https://ai-act-law.eu/de/artikel/2/
- forum-institut: KI-Kennzeichnung nach Artikel 50 – https://forum-institut.de/eu-ai-act-2-august-2026/ki-kennzeichnung-nach-artikel-50
- ai-act-law.eu: Artikel 50 – https://ai-act-law.eu/de/artikel/50/
- Apify: Greenhouse/Lever/Ashby Jobs Scraper – https://apify.com/webdata_labs/greenhouse-lever-ashby-jobs-scraper
- Apify: Career Site Jobs Scraper – https://apify.com/santamaria-automations/career-site-jobs-scraper
- softgarden Karriereseite – https://softgarden.career.softgarden.de/en/
- rexx systems: Jobs – https://www.rexx-systems.com/jobs/
- d.vinci: Whitepaper Karrierewebsite – https://www.dvinci.de/docs20/d.vinci_Whitepaper_Karrierewebsite.pdf
- webappanalyzer: Repository – https://github.com/enthec/webappanalyzer
- webappanalyzer: categories.json – https://raw.githubusercontent.com/enthec/webappanalyzer/main/src/categories.json
- urheberrecht.de: Künstliche Intelligenz – https://www.urheberrecht.de/kuenstliche-intelligenz/
- anwalt24: Lüge im Lebenslauf – https://www.anwalt24.de/fachartikel/arbeit-und-betrieb/46090
- Kliemt: Lüge im Lebenslauf und drastische Spätfolgen – https://kliemt.blog/2016/10/05/nur-schoenfaerberei-luege-im-lebenslauf-und-drastische-spaetfolgen/
