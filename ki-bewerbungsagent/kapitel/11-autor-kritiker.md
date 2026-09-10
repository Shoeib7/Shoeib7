## 11. Modul Autor und Kritiker: Anschreiben-Engine, Lebenslauf-Tailoring, Anti-Generik, Rubrik, Qualitätsschleife

Autor und Kritiker sind das Herzstück des Bewerbungsagenten. Alles davor (Scout, Matcher, Rechercheur) sammelt Material; alles danach (ATS-Prüfer, Setzer, Bote) verpackt und transportiert. Hier entsteht der Text, den ein Mensch liest und nach dem er entscheidet. Der Autor schreibt Anschreiben und passt den Lebenslauf an; der Kritiker prüft gegen Rubrik, Story-Bank, Stellenanzeige und Stimmprofil und schickt zurück, was nicht trägt. Beide arbeiten in der Status-Pipeline zwischen „recherchiert“ und „geprüft“; „bereit zur Freigabe“ vergibt erst der ATS-Prüfer nach dem Test-Parsing (Kapitel 12).

Der Leitsatz gilt hier wörtlich: Eine Bewerbung, die nach dieser Person für diese Stelle klingt, schlägt hundert austauschbare. Das Modul ist deshalb nicht auf Tempo optimiert, sondern auf zwei Fragen: Ist jede Aussage belegt? Würde der Nutzer das so sagen?

### 11.1 Aufgabe, Schnittstellen, Leitsätze

**Eingaben** (alle als strukturierte Dateien, nie als Freitext-Prompt zusammengeklebt):

- Die normalisierte Stellenanzeige aus Kapitel 9 mit extrahierten Muss- und Kann-Kriterien, Sprache, Anredeform der Anzeige, Kennziffer und Bewerbungsweg.
- Das Rechercheur-Dossier aus Kapitel 10: Firmenname mit Rechtsform, Anschrift, Ansprechpartner mit Anredefeldern, Kultur- und Nachrichtenfakten, ATS-Typ – jedes Feld mit Konfidenz (hoch, mittel, niedrig) und Quelle.
- Das Kandidatenprofil aus Kapitel 8: Master-Lebenslauf, Story-Bank, Stimmprofil, Präferenzen, Standardantworten.
- Der Regelkatalog `ats_rules.yaml` aus Kapitel 4 (R01 bis R03, R09 als Schreibregeln; V01, V02, V08 als harte Prüfpunkte).
- Das Ergebnis des Format-Routers (11.3): welches Dokument in welcher Sprache und Länge.

**Ausgaben:**

- Das Bewerbungsdossier (JSON, Schema in Kapitel 13): Anschreiben in Absätzen, angepasster Lebenslauf in Abschnitten, Betreff, Anrede, Sprache, Dokumentliste.
- Der Kritikbericht (JSON, 11.9), das Fakten-Check-Protokoll (11.8, Schritt 5) und das Tailoring-Log (11.7) für das Review-Cockpit (Kapitel 14).
- Rückfragen an dich, wenn etwas nicht belegbar oder nicht entscheidbar ist; die Stelle bekommt dann den Status „Rückfrage offen“.

**Drei Leitsätze, die über allem stehen:**

1. **Nichts erfinden.** Jede Zahl, jeder Titel, jedes Werkzeug, jede Zeitangabe im Anschreiben oder im angepassten Lebenslauf verweist auf einen Eintrag der Story-Bank oder des Master-Lebenslaufs. Was keinen Eintrag hat, fliegt raus oder wird zur Rückfrage. Der rechtliche Hintergrund ist § 123 BGB: Eine arglistige Täuschung über einstellungsrelevante Tatsachen erlaubt die Anfechtung des Arbeitsvertrags, auch Jahre später ([anwalt24](https://www.anwalt24.de/fachartikel/arbeit-und-betrieb/46090), [Kliemt](https://kliemt.blog/2016/10/05/nur-schoenfaerberei-luege-im-lebenslauf-und-drastische-spaetfolgen/)).
2. **Für Menschen schreiben.** Die CV-Parser der relevanten ATS lesen den Lebenslauf, nicht das Anschreiben (Kapitel 4). Das Anschreiben ist deshalb frei von jeder Keyword-Taktik (V07) und ganz auf den lesenden Recruiter ausgerichtet. Keyword-Spiegelung findet ausschließlich im Lebenslauf statt, und nur belegt (R01).
3. **Die Stimme gehört dem Nutzer, nicht dem Modell.** Das Stimmprofil (Kapitel 8) liefert Regeln, Beispiele und Tabus. Der Autor schreibt innerhalb dieser Grenzen; der Kritiker prüft, ob der Text sie einhält. Die Forschung zeigt, dass Modelle die Stimme von „everyday authors“ mit wenigen Textproben nur unvollständig treffen ([ACL Findings 2025](https://arxiv.org/pdf/2509.14543), [Jemama 2025](https://arxiv.org/abs/2509.24930)); die Lücke wird deshalb über explizite Stilregeln geschlossen, nicht über reine Imitation versprochen.

| Der Autor darf | Der Autor darf nicht |
|---|---|
| Story-Bank-Einträge auswählen, kürzen, in eigene Worte fassen | Erfolge, Zahlen, Zeiträume, Titel, Abschlüsse, Zertifikate, Arbeitgeber hinzufügen |
| Lebenslauf-Bullets umordnen, betonen, weglassen (mit Log) | Bullets neu erfinden oder Verantwortungsumfang vergrößern |
| Fachbegriffe der Anzeige verwenden, wenn im Profil belegt (R01, R02) | Sätze der Anzeige übernehmen (Urheberrecht, Kapitel 4) oder unbelegte Begriffe einstreuen |
| Firmenfakten mit Konfidenz hoch oder mittel nennen | Fakten mit Konfidenz niedrig verwenden oder Firmenwissen aus dem Modell ergänzen |
| Gehalt und Eintrittstermin aus Profilfeldern einsetzen, wenn die Anzeige fragt | Gehalt schätzen oder unaufgefordert nennen |
| Lücken mit `[UNBELEGT: …]` markieren und die Stelle in „Rückfrage offen“ setzen | Lücken mit plausiblem Text füllen |

### 11.2 Warum Generik das Risiko ist, nicht KI

Die Ausgangslage 2025/2026: 61 Prozent der Bewerber haben laut StepStone-Studie 2025 KI für ihr Anschreiben genutzt; 69 Prozent der Recruiter empfinden Unterlagen seitdem als weniger auf die Stelle zugeschnitten, 73 Prozent als weniger authentisch, 75 Prozent stören sich an übertriebenen Qualifikationsdarstellungen ([StepStone](https://www.stepstone.de/e-recruiting/hr-wissen/recruiting/stepstone-studie-2025-ki-und-jobsuche), [persoblogger](https://persoblogger.de/2025/05/29/studie-zwei-von-drei-bewerbungen-mit-hilfe-von-ki-erstellt-recruiterinnen-fehlt-individualitaet); die Prozentwerte konnten vom Faktenprüfer nicht unabhängig geprüft werden). Der StepStone Hiring Trends Index Q1/2025 nennt 81 Prozent Recruiter, die einen Qualitätsrückgang beobachten ([StepStone](https://www.stepstone.de/e-recruiting/hr-wissen/recruiting/bewerberqualitaet-steigern), nicht unabhängig geprüft).

Entscheidend für die Architektur ist die Lücke zwischen Anspruch und Fähigkeit: 67 bis 74 Prozent der Recruiter glauben, KI-Texte zu erkennen; in einem Blindtest mit 1.000 Recruitern (ResumeBuilder, 2023) scheiterten 82 Prozent daran, alle KI-Briefe zu identifizieren, und die tatsächliche Trefferquote liegt laut anderen Erhebungen bei rund 52 Prozent ([Cover Letter Copilot](https://coverlettercopilot.ai/blog/recruiters-human-vs-ai-cover-letters)). Was Recruiter wirklich abstraft, ist fehlende Personalisierung: 62 Prozent lehnen KI-Unterlagen ohne erkennbare persönliche Details ab, 78 Prozent suchen aktiv nach solchen Details als Passungssignal ([JobCannon](https://jobcannon.io/blog/ai-resume-statistics-2026)); 49 Prozent der Hiring Manager in einer Resume.io-Umfrage (Januar 2025, 3.000 Befragte) sortieren als KI-generiert erkannte Lebensläufe automatisch aus ([JobCannon/Resume.io](https://jobcannon.io/research/stats/resumeio-49-reject)). Alle diese Zahlen stammen aus Anbieter- oder Ratgeberquellen und sind als Größenordnung, nicht als Messwert zu lesen.

Die Konsequenz: Zielmetrik der Qualitätsschleife ist „klingt nach dieser Person und dieser Stelle“, nicht „besteht einen KI-Detektor“. Detektoren sind ungeeignet als Freigabekriterium: GPTZero zeigt in Vergleichsstudien zwar eine niedrige Falsch-Positiv-Rate (0,24 Prozent in einer 3.000-Dokumente-Studie, Originality.ai 4,79 Prozent) ([GradPilot](https://gradpilot.com/news/ai-detector-false-positive-rates-compared)), alle Detektoren sind aber gegenüber Nicht-Muttersprachlern und formellem Stil massiv verzerrt – Falsch-Positiv-Raten von 23 Prozent gegenüber 4 Prozent bei Muttersprachlern, in einer weiteren Studie 61,3 gegenüber 5,1 Prozent ([WasItAIGenerated](https://www.wasitaigenerated.com/research/ai-detection-hiring-recruitment), [GPTOne](https://gptone.me/blog/ai-detector-for-recruiters-screen-ai-written-resumes-2026)). Recruiter setzen sie in der Praxis selten aktiv ein ([Textora](https://www.textora.org/blog/how-recruiters-detect-ai-cover-letters), Konfidenz niedrig).

**Entscheidung:** Kein KI-Detektor in der Qualitätsschleife, weder als Gate noch als Score. **Begründung:** Falsche Zielmetrik, Bias-Risiko, keine Praxisrelevanz beim Empfänger. **Alternative:** GPTZero als nicht blockierender Hinweis im Review-Cockpit (v2, nur wenn du es ausdrücklich willst).

Was stattdessen zählt, sind die belegten Erkennungsmerkmale generischer Texte: polierte Sprache ohne individuelle Brüche, Standardfloskeln („sehr motivierter Teamplayer“), identische Formulierungen und Satzrhythmen über mehrere Bewerbungen hinweg ([20 Minuten](https://www.20min.ch/story/kuenstliche-intelligenz-merken-recruiter-wenn-bewerbungen-ki-generiert-sind-103298940)); die Konstruktion „nicht nur X, sondern Y“, die in rund 6 Prozent aller ChatGPT-Nachrichten vorkommt, Dreierlisten (Tricolon) und mathematisch gleichmäßige Absatzrhythmen ([Decrypt](https://decrypt.co/348923/5-biggest-tells-something-written-ai)); niedrige Burstiness, also gleichförmige Satzlängen ([QuillBot](https://quillbot.com/blog/ai-writing-tools/burstiness-and-perplexity/)). Genau diese Merkmale prüft der Kritiker deterministisch (11.5) und per Rubrik (11.9).

### 11.3 Wann kein Anschreiben, und welches Format sonst

Das Anschreiben ist in Deutschland nicht mehr Standard: Laut einer Taledo-Umfrage ist es bei 64 Prozent der Unternehmen optional, laut StepStone-Analyse 2025 verlangen es noch 45 Prozent, während rund 60 Prozent der Bewerber es weiterhin nutzen ([Home of Jobs](https://homeofjobs.de/blog/bewerbung-2025-diese-trends-solltest-du-jetzt-kennen/), [CV Score](https://cvscore.net/de/blog/braucht-man-2026-noch-ein-anschreiben/); nicht unabhängig geprüft). Die Otto Group verzichtet komplett darauf ([Otto-Presse](https://www.otto.de/unternehmen/en/press/du-kannst-dich-ohne-anschreiben-bewerben-otto-setzt-neue-ma%C3%9Fst%C3%A4be-im-recruitment)), die Deutsche Bahn nur bei Azubi- und dualen Studienplätzen ([Talention](https://www.talention.de/blog/die-deutsche-bahn-kein-anschreiben-fuer-mehr-bewerbungen)). Die Kulturanalyse dazu steht in Kapitel 5; hier folgt nur die Produktkonsequenz: Der Format-Router entscheidet pro Stelle, bevor der Autor ein Wort schreibt.

| Signal (Anzeige, Portal, Rechercheur) | Format | Länge | Entwürfe |
|---|---|---|---|
| Anzeige nennt „Anschreiben“, „vollständige Unterlagen“, „Motivation“; Bewerbungsweg E-Mail | `voll` | 250–350 Wörter, 1 Seite | 3 |
| Portal mit Freitextfeld „Nachricht“ oder „Anschreiben (optional)“; Zeichenlimit unter 1.500 | `kurz` | 120–180 Wörter | 2 |
| Firma laut Rechercheur ohne Anschreiben (Otto-Typ, „One-Click“, nur CV-Upload) | `keins` | E-Mail-Begleittext 3–5 Sätze, falls E-Mail | 1 |
| Keine Stellenanzeige, Zielfirma aus Präferenzen (Kapitel 9) | `initiativ` | 250–350 Wörter, 1 Seite | 3 |
| Anzeige verlangt zusätzlich ein Motivationsschreiben | `voll` + `motivation` | Motivationsschreiben bis 1 Seite | 2 (v2) |
| Anzeige auf Englisch oder Konzernsprache Englisch | `voll-en` / `kurz-en` | bis 400 Wörter, 1 Seite | 3 |
| Kein Signal in beide Richtungen | Default-Annahme, siehe unten | | |

Kurzbewerbung, Kurzprofil und Motivationsschreiben sind drei verschiedene Dinge: Die Kurzbewerbung kombiniert ein kurzes Anschreiben mit dem Lebenslauf auf 1–2 Seiten; das Kurzprofil ist ein Zusatzblatt zum vollständigen Lebenslauf; das Motivationsschreiben ein ausführliches Zusatzdokument zu Beweggründen und Zielen ([bewerbung.net](https://bewerbung.net/kurzbewerbung), [business-on.de](https://www.business-on.de/unterschied-anschreiben-und-motivationsschreiben.html)). Im MVP erzeugt der Autor `voll`, `kurz`, `keins` und `initiativ`; Motivationsschreiben und Kurzprofil-Blatt sind v2 (Kapitel 19).

**Entscheidung:** Bei fehlendem Signal erzeugt der Agent ein `voll`-Anschreiben, wenn der Bewerbungsweg E-Mail ist, und ein `kurz`-Anschreiben, wenn der Weg ein Portal mit Upload-Feldern ist. **Begründung:** Per E-Mail fehlt ohne Anschreiben der Rahmen; im Portal liest der Recruiter zuerst den Lebenslauf, ein knapper Text stört nicht und kostet wenig. **Alternative:** Immer `voll` (Qualitätsanspruch, aber Aufwand ohne Nachfrage) oder immer `keins` bei fehlender Anforderung (15 Prozent Abbruchquote bei Pflicht-Anschreiben zeigen, dass Firmen es zunehmend nicht erwarten; Zahl unbestätigt). Deine Präferenz gehört in Kapitel 22.

### 11.4 Schreibregeln: was ein gutes deutsches Anschreiben 2026 auszeichnet

Die Regeln sind so formuliert, dass sie im Kritiker prüfbar sind. Der Autor bekommt sie in verkürzter Form (11.10), damit der Schreib-Prompt nicht zum Regelwerk wird.

1. **Länge und Form:** 250 bis 350 Wörter, eine DIN-A4-Seite, vier Absätze; Personalverantwortliche entscheiden in 30 bis 60 Sekunden, ob sie weiterlesen ([skill-sprinters](https://skill-sprinters.de/blog/karriere/anschreiben-2026-aufbau/), [ulmato](https://www.ulmato.de/anschreiben/)). Die entscheidende Aussage steht in den ersten zwei Sätzen.
2. **Einstieg ohne Floskel:** Der erste Satz enthält entweder einen belegten Erfolg mit Zahl aus der Story-Bank oder einen recherchierten Firmenbezug – nie „Hiermit bewerbe ich mich“ oder „Mit großem Interesse habe ich gelesen“ ([ulmato](https://www.ulmato.de/anschreiben/)).
3. **Nachprüfbarer Firmenbezug:** Mindestens ein, besser zwei Details aus dem Rechercheur-Dossier (Produkt, Projekt, Standort, Nachricht, Teamstruktur), die nicht aus der Selbstbeschreibung der Anzeige stammen und die der Empfänger als „hat sich informiert“ erkennt. Nur Fakten mit Konfidenz hoch oder mittel (Kapitel 10). Pauschales Lob („Ihr renommiertes Unternehmen“) zählt nicht.
4. **Zwei bis drei belegte Erfolge mit Zahl:** Erfolge statt Aufgaben („Drei Jahre Verantwortung für 65 Mitarbeiter“ statt „langjährige Erfahrung im Personalwesen“) ([ulmato](https://www.ulmato.de/anschreiben/)). Jede Zahl trägt eine Story-Bank-ID im Fakten-Check.
5. **Muss-Kriterien explizit beantworten:** Jedes Muss-Kriterium der Anzeige, das der Kandidat erfüllt, bekommt einen Satz oder Halbsatz mit Beleg (R03). Nicht erfüllte Muss-Kriterien werden nicht kaschiert; der Matcher hat sie vorher bewertet (Kapitel 9), und der Autor umgeht sie ehrlich („Statt X bringe ich Y mit“) oder gar nicht.
6. **Das Anschreiben erklärt Warum und Wie, der Lebenslauf das Was.** Keine Nacherzählung der Stationen. Ein Erfolg aus dem Lebenslauf darf im Anschreiben vertieft werden (Kontext, Vorgehen, Ergebnis), nicht wiederholt.
7. **Keine Selbstattribute, nur Belege:** Kein „teamfähig, belastbar, kommunikationsstark“. Wer teamfähig ist, beschreibt ein Team-Ergebnis mit Zahl.
8. **Anrede:** Standard ist „Sie“, auch wenn die Anzeige duzt ([bewerbung.com](https://bewerbung.com/du-in-stellenanzeigen/), [peopleatventure](https://www.peopleatventure.de/bewerbung-anrede)). „Du“ nur, wenn die Anzeige durchgehend duzt, die Firmenkultur laut Rechercheur eindeutig Du-geprägt ist (Startup, explizite Aufforderung) und deine Profil-Präferenz es erlaubt; dann professionell, ohne Kumpelton ([Karriereakademie](https://www.karriereakademie.de/duzen-stellenanzeige)). Die namentliche Anrede folgt der Rückfrage-Kaskade aus Kapitel 10.4 (Name mit Konfidenz ≥ 0,7 → namentlich; Name sicher, Anrede unsicher → „Guten Tag [Vorname] [Nachname]“; kein brauchbarer Name → „Sehr geehrtes Recruiting-Team [Firma]“; gar kein Kandidat → „Sehr geehrte Damen und Herren“, Kapitel 13); ein geratener Name oder Titel ist ein K.-o.-Fehler.
9. **Schluss ohne Konjunktiv:** „Ich freue mich auf das Gespräch“ statt „würde mich freuen“; Gehaltsvorstellung als Bruttojahresgehalt oder Spanne und Eintrittstermin nur, wenn die Anzeige explizit danach fragt, immer aus dem Profilfeld, nie geschätzt ([bewerbung.net](https://bewerbung.net/gehaltsvorstellung-bewerbung), [JobTeaser](https://www.jobteaser.com/de/advices/gehaltsvorstellung-in-der-bewerbung-formulieren-so-geht-s), [Karrierebibel](https://karrierebibel.de/bewerbung-eintrittstermin-nennen-sofort/)). Fehlt das Profilfeld, entsteht eine Rückfrage, kein Platzhalter.
10. **Rhythmus wie ein Mensch:** Satzlängen variieren (kurze Sätze neben langen), keine Dreierlisten aus parallelen Adjektiven, keine „nicht nur …, sondern auch“-Konstruktion, kein „Es geht nicht um X, sondern um Y“ ([Decrypt](https://decrypt.co/348923/5-biggest-tells-something-written-ai), [QuillBot](https://quillbot.com/blog/ai-writing-tools/burstiness-and-perplexity/)). Absätze dürfen unterschiedlich lang sein.
11. **Fachbegriffe ja, Anzeigensätze nein:** Werkzeuge, Zertifikate und Rollentitel in der Schreibweise der Anzeige, sofern belegt (R01); keine übernommenen Satzteile (Urheberrecht an Anzeigentexten, Kapitel 4). Der Kritiker prüft n-Gramm-Überlappung (11.5).
12. **Lesbarkeit:** Zielkorridor Wiener Sachtextformel Schulstufe 8 bis 10; die Formel ist selbst implementierbar, ebenso Flesch-Deutsch (FRE = 180 − ASL − 58,5 × ASW) und LIX ([fleschindex.de](https://fleschindex.de/lesbarkeitsindex), [fair-text](https://fair-text.com/lesbarkeitsindex-textanalyse-tool/)). Die Stufe 8–10 ist eine Projektannahme („verständlich, nicht simpel“) und wird gegen deine eigenen Textproben kalibriert (Kapitel 8).
13. **Ehrliche Wechselmotive:** Warum du wechselst, steht nur so im Text, wie es im Kandidatenprofil hinterlegt ist. Nichts über den aktuellen Arbeitgeber, was du nicht selbst freigegeben hast.
14. **Betreff mit Substanz:** exakter Stellentitel aus der Anzeige, Kennziffer, gegebenenfalls Standort; kein zweites „Bewerbung“ im Text. Die Formatierung nach DIN 5008 übernimmt der Setzer (Kapitel 13).
15. **Eine Stimme pro Bewerbung, verschiedene Stimmen über Bewerbungen hinweg:** Der Kritiker vergleicht neue Entwürfe mit den letzten zehn freigegebenen Anschreiben und meldet wiederkehrende Einstiegssätze oder Absatzmuster – identische Formulierungen über mehrere Bewerbungen sind ein belegtes Erkennungsmerkmal ([20 Minuten](https://www.20min.ch/story/kuenstliche-intelligenz-merken-recruiter-wenn-bewerbungen-ki-generiert-sind-103298940)).

### 11.5 Anti-Generik-Katalog

Der Katalog arbeitet zweistufig: eine deterministische Liste (Regex, läuft vor jedem Kritiker-Aufruf, kostet keine Tokens) und ein Rubrik-Kriterium im Kritiker für die Generik, die keine Wortliste fängt. Jeder Eintrag hat eine Schwere: `hart` blockiert den Entwurf (zurück an den Autor), `weich` senkt die Rubrik-Note und erzeugt einen Hinweis. Der Katalog ist ein Startbestand aus den zitierten Ratgeber- und Studienquellen ([ulmato](https://www.ulmato.de/anschreiben/), [20 Minuten](https://www.20min.ch/story/kuenstliche-intelligenz-merken-recruiter-wenn-bewerbungen-ki-generiert-sind-103298940), [Decrypt](https://decrypt.co/348923/5-biggest-tells-something-written-ai)) plus redaktioneller Ergänzung; du erweiterst ihn im Onboarding um deine eigenen Tabus (Kapitel 8) und im Betrieb über das Review-Cockpit („diese Wendung nie wieder“, Kapitel 14).

**Deutsch (Auszug: die 10 wirkungsvollsten harten Einträge)**

| Nr. | Floskel oder Muster | Schwere | Ersatzstrategie |
|---|---|---|---|
| D01 | „Hiermit bewerbe ich mich …“ | hart | Einstieg mit Erfolg oder Firmenbezug (Regel 2) |
| D02 | „Mit großem Interesse habe ich Ihre Stellenanzeige gelesen“ | hart | streichen; Interesse zeigt sich am Detail |
| D04 | „auf der Suche nach einer neuen Herausforderung“ | hart | Wechselmotiv aus dem Profil |
| D06 | „Teamplayer“, „teamfähig“ (ohne Beleg) | hart | Team-Ergebnis mit Zahl |
| D07 | „belastbar“ | hart | Situation mit Last und Ergebnis |
| D10 | „kommunikationsstark“ | hart | Beispiel: Präsentation, Verhandlung, Zahl |
| D17 | „Ich bringe alles mit, was Sie suchen“ | hart | Muss-Kriterien einzeln belegen |
| D19 | „nicht nur …, sondern auch …“ | hart | eine Aussage pro Satz |
| D30 | „Ihr renommiertes Unternehmen“, „Marktführer“ (pauschal) | hart | recherchiertes Detail |
| D33 | „Über eine Einladung … würde ich mich sehr freuen“ | hart | Indikativ (Regel 9) |

Vollständige Liste: 38 deutsche Einträge (D01–D38, harte und weiche) und, für englische Bewerbungen (11.11), 36 englische Einträge (E01–E36) nach demselben Muster – inklusive der weichen Treffer D27 und D35, auf die die Beispiele in 11.8 und 11.9 verweisen. Sie lebt ausschließlich maschinenlesbar in `config/anti_generik.yaml`, Version 2026-09, damit Kritiker und Review-Cockpit dieselbe Datei nutzen und der Schreib-Prompt des Autors sie nie ausgeschrieben sieht (11.10).

```yaml
# config/anti_generik.yaml – Version 2026-09; Regex case-insensitive, Wortgrenzen beachten
- id: D01
  sprache: de
  muster: '\bhiermit bewerbe ich mich\b'
  schwere: hart
  ersatz: Einstieg mit belegtem Erfolg oder recherchiertem Firmenbezug
- id: D19
  sprache: de
  muster: '\bnicht nur\b[^.]{3,80}\bsondern( auch)?\b'
  schwere: hart
  ersatz: eine Aussage pro Satz
- id: D37
  sprache: de
  typ: struktur           # kein Regex, Prüfung per Parser
  regel: Aufzählung von genau drei parallelen Adjektiven/Substantiven mit "und"
  schwere: weich
  max_vorkommen: 0
- id: D38
  sprache: de
  typ: metrik
  regel: Anteil Sätze mit Satzanfang "Ich" <= 0.40
  schwere: weich
- id: E24
  sprache: en
  muster: '\bnot only\b[^.]{3,80}\bbut( also)?\b'
  schwere: hart
  ersatz: one claim per sentence
- id: NUTZER-001          # aus dem Stimmprofil, Kapitel 8
  sprache: de
  muster: '\bspannend\w*\b'
  schwere: hart
  ersatz: vom Nutzer im Onboarding als Tabu gesetzt
```

Zusätzliche deterministische Prüfungen, die vor dem Kritiker laufen (alle in Python ohne Modellaufruf, Schwellen sind Projektannahmen und werden gegen deine Textproben aus Kapitel 8 kalibriert):

| Prüfung | Regel | Schwere |
|---|---|---|
| Wortzahl | im Korridor des Formats (11.3) | hart |
| Satzlängen-Varianz | Standardabweichung der Satzlänge mindestens 70 Prozent der Standardabweichung deiner eigenen Textproben; mindestens ein Satz unter 9 und einer über 20 Wörtern | weich |
| Lesbarkeit | Wiener Sachtextformel Stufe 8–10 | weich |
| Anzeigen-Kopie | keine identische Wortfolge von 8 oder mehr Wörtern zwischen Anschreiben und Anzeige | hart |
| Wiederholung | kein Einstiegssatz und kein Absatz mit mehr als 60 Prozent Wortüberlappung zu den letzten zehn freigegebenen Anschreiben | weich |
| Gedankenstriche | höchstens zwei Halbgeviertstriche als Satzeinschub | weich |
| Anrede und Name | Anrede, Name, Titel identisch mit Rechercheur-Feldern; Firmenname mit Rechtsform identisch | hart |
| Verbotene Inhalte | kein Weißtext, keine versteckten Anweisungen (V02), keine Zahl ohne Story-Bank-ID | hart |

### 11.6 Drei Struktur-Templates als Skelette

Die Skelette legen Reihenfolge, Zweck und Wortbudget jedes Absatzes fest; sie enthalten keine vorformulierten Sätze, denn vorformulierte Sätze wären Generik. Platzhalter in geschweiften Klammern verweisen auf Datenfelder: `{S-…}` Story-Bank-Eintrag, `{F-…}` Firmenfakt aus dem Rechercheur-Dossier, `{M-…}` Muss-Kriterium der Anzeige, `{P.…}` Profilfeld. Kopf, Anschrift, Datum und Grußformel setzt der Setzer nach DIN 5008 (Kapitel 13); das Skelett beginnt beim Betreff.

**Template A: klassisch (`voll`, 250–350 Wörter, vier Absätze)**

```text
BETREFF   {Stellentitel exakt wie Anzeige}, {Kennziffer}; optional {Standort}

ANREDE    {Anredefeld aus Kapitel 10, Fallback-Stufe 1–4}

ABSATZ 1  Einstieg (40–60 Wörter)
          Variante „Leistung“: Erfolg {S-a} mit Zahl, dann Brücke zur Stelle.
          Variante „Firmenbezug“: Fakt {F-1}, warum er den Kandidaten betrifft,
          dann Stellenbezug.
          Variante „Motiv“: konkretes Wechselmotiv {P.wechselmotiv} + Stelle.
          Kein Standardsatz; Stellentitel einmal nennen.

ABSATZ 2  Mehrwert (90–120 Wörter)
          Zwei Erfolge {S-a}, {S-b} mit Zahl, je einem Muss-Kriterium
          {M-1}, {M-2} zugeordnet. Ein Satz zu Werkzeugen/Methoden in
          Anzeigen-Schreibweise (R01), nur belegte. Ergebnis vor Aufgabe.

ABSATZ 3  Warum dieses Unternehmen (50–80 Wörter)
          Fakt {F-2} (Produkt, Projekt, Team, Nachricht) und was der
          Kandidat daran konkret mitgestalten will; bei Wechsel: nur das
          im Profil hinterlegte Motiv. Kein pauschales Lob.

ABSATZ 4  Schluss (30–50 Wörter)
          Eintrittstermin {P.eintritt} und Gehalt {P.gehalt} nur, wenn die
          Anzeige fragt. Indikativ. Ein konkreter Gesprächsanlass
          („… gern zeige ich Ihnen am Beispiel {S-a}, wie …“).

GRUSS     {Grußformel laut Stimmprofil}, Name
```

**Template B: kurz (`kurz`, 120–180 Wörter, drei Absätze; Portal-Freitextfeld, optionales Anschreiben, E-Mail-Begleittext in Langform)**

```text
BETREFF   nur bei E-Mail: {Stellentitel}, {Kennziffer}

ANREDE    wie Template A

ABSATZ 1  Hook (25–40 Wörter)
          Ein Erfolg {S-a} mit Zahl oder ein Firmenfakt {F-1}, direkt mit
          dem Stellentitel verbunden.

ABSATZ 2  Kern (60–90 Wörter)
          Die zwei wichtigsten Muss-Kriterien {M-1}, {M-2} mit je einem
          Beleg; ein Satz Werkzeuge in Anzeigen-Schreibweise.

ABSATZ 3  Schluss (20–40 Wörter)
          Verfügbarkeit nur wenn gefragt; Gesprächsanlass; Indikativ.

GRUSS     wie Template A
```

Für `keins` mit Bewerbungsweg E-Mail erzeugt der Autor aus Template B einen Begleittext von drei bis fünf Sätzen (Absatz 1 in einem Satz, Absatz 2 in zwei bis drei Sätzen, Absatz 3 in einem Satz); der Bote übernimmt ihn als Mailtext (Kapitel 15).

**Template C: Initiativbewerbung (`initiativ`, 250–350 Wörter, vier Absätze)**

Initiativbewerbungen erzielen laut Ratgeberquellen Erfolgsquoten von 20 bis 33 Prozent, wenn sie individuell und recherchiert sind, nicht als Massenmail ([Robert Half](https://www.roberthalf.com/de/de/insights/bewerbungs-tipps/initiativbewerbung-erster-schritt-zum-traumjob-oder-eher-vergebene-liebesmueh), [Karrierebibel](https://karrierebibel.de/initiativbewerbung/); Konfidenz niedrig, keine Primärstudie). Der Einstieg beginnt „mitten im Geschehen“ mit dem Unternehmen beim Namen; Betreffvarianten sind „Initiativbewerbung als …“ oder „Interesse an einer Mitarbeit im Bereich …“ ([Karrierebibel](https://karrierebibel.de/initiativbewerbung/)). Voraussetzung: ein namentlich bekannter Ansprechpartner mit Konfidenz hoch (Kapitel 10); sonst „Rückfrage offen“, keine Initiativbewerbung an „Sehr geehrte Damen und Herren“.

```text
BETREFF   Initiativbewerbung als {Zielrolle aus P.zielrollen} im Bereich {Bereich}

ANREDE    namentlich, Konfidenz hoch (Pflicht)

ABSATZ 1  Anlass (40–60 Wörter)
          Konkreter, datierter Anlass {F-1}: Nachricht, Produktstart,
          Standort-Eröffnung, Vortrag, Gespräch. Warum der Anlass den
          Kandidaten betrifft. Firma beim Namen.

ABSATZ 2  Problem und Beitrag (90–120 Wörter)
          Welches Problem oder Vorhaben die Firma laut {F-1}, {F-2} hat;
          zwei Erfolge {S-a}, {S-b} mit Zahl, die genau dazu passen.

ABSATZ 3  Vorschlag (50–80 Wörter)
          Konkreter Einsatzbereich oder Rollenvorschlag; was in den ersten
          Monaten realistisch wäre (nur aus Story-Bank ableitbar, keine
          Versprechen); Bezug auf Teamstruktur {F-3}, falls belegt.

ABSATZ 4  Nächster Schritt (30–50 Wörter)
          Gesprächsangebot mit konkretem Thema; Verfügbarkeit;
          Indikativ; kein Gehalt.

GRUSS     wie Template A
```

**Englische Variante (`voll-en`, `kurz-en`):** gleiche Absatzlogik, aber bis 400 Wörter, direktere Selbstdarstellung, Betreffzeile nur in der britischen Form, amerikanische Cover Letter ohne Betreff ([Lebenslaufdesigns](https://lebenslaufdesigns.de/anschreiben-englisch), [Elinora](https://elinora.net/cover-letter-deutsch)); Details in 11.11.

### 11.7 Lebenslauf-Tailoring ohne Erfindung

Der Master-Lebenslauf (Kapitel 8) ist die vollständige Wahrheit: alle Stationen, alle Bullets, alle Kenntnisse mit Niveau, jede Zahl mit Story-Bank-Verweis. Tailoring ist eine Projektion daraus, nie eine Erweiterung. Der Recruiter sieht den Lebenslauf zuerst und in Sekunden; die Eye-Tracking-Studie von The Ladders (2018, US-Kontext) nennt 7,4 Sekunden und ein F-Muster mit Blick auf aktuellen Titel, Firma und Daten ([The Ladders](https://www.theladders.com/career-advice/you-only-get-6-seconds-of-fame-make-it-count)); deutsche Quellen streuen zwischen 6 und 43 Sekunden (Kapitel 5). Was oben und links steht, zählt.

**Erlaubte Operationen** (jede erzeugt einen Eintrag im Tailoring-Log):

| Operation | Was passiert | Grenze |
|---|---|---|
| Kurzprofil | 3–4 Zeilen unter dem Namen, neu formuliert je Stelle: Rolle, Jahre, zwei Kernkompetenzen, ein Erfolg mit Zahl | jede Aussage mit Story-Bank- oder Master-ID; keine Adjektive ohne Beleg |
| Kernkompetenzen-Block | 6–10 Begriffe direkt unter dem Kurzprofil, in Anzeigen-Schreibweise (R01) plus Taxonomie-Schreibweise, wo abweichend (R02) | nur Begriffe mit Niveau im Master; keine Begriffe, die der Kandidat nicht kann |
| Betonen | pro Station die 3–5 relevantesten Bullets aus dem Master-Pool auswählen; irrelevante ausblenden | Bullets werden gewählt, nicht geschrieben; jede Station behält mindestens einen Bullet |
| Umordnen | Reihenfolge der Bullets innerhalb einer Station; Reihenfolge der Kenntnis-Kategorien; Position des Weiterbildungsblocks | Chronologie der Stationen bleibt; keine Station verschwindet |
| Sprache spiegeln | Werkzeug- und Methodennamen exakt wie in der Anzeige, wenn im Master als Synonym hinterlegt („MS Excel“ ↔ „Excel“, „PM“ ↔ „Projektmanagement“) | Synonym-Mapping kommt aus dem Profil, nicht aus dem Modell |
| Kürzen | ältere oder fachfremde Stationen auf Titel, Firma, Zeitraum und einen Bullet reduzieren | Zeiträume bleiben vollständig; Lücken bleiben sichtbar |
| Umformulieren | Bullet-Wortlaut glätten (Verb voran, Ergebnis nach vorn) | Inhalt, Zahl, Umfang unverändert |

**Verbotene Operationen** (V08, R09; der Kritiker prüft sie als hartes Gate):

- Neue oder geänderte Jobtitel, Zeiträume, Abschlüsse, Noten, Zertifikate, Arbeitgebernamen, Teamgrößen, Budgets, Prozentzahlen.
- Verantwortung aufrunden („Leitung“ statt „Mitarbeit“), Teilzeit verschweigen, Lücken schließen oder verschieben.
- Kenntnisse hochstufen („fließend“ statt „gut“) oder Werkzeuge nennen, die nur „gehört“ sind.
- Keyword-Blöcke, Weißtext, unsichtbare Begriffe (V01, V02).

**Grauzonen mit Entscheidung:**

| Fall | Regel | Woher die Freigabe kommt |
|---|---|---|
| Interner Jobtitel ist unverständlich („Specialist II“) | Originaltitel bleibt; marktüblicher Titel in Klammern nur, wenn im Profil als `titel_alias` hinterlegt | einmalig im Onboarding (Kapitel 8) |
| Lücke im Lebenslauf | wird mit dem im Profil hinterlegten Wortlaut benannt („Elternzeit“, „Weiterbildung“, „Bewerbungsphase“) | Profilfeld; fehlt es: Rückfrage |
| Zahl in der Story-Bank ist „ca.“ | „rund“ oder „etwa“ im Text, nie eine präzisere Zahl | Story-Bank-Eintrag |
| Anzeige verlangt Begriff, der im Profil fehlt | ATS-Prüfer meldet ihn (Kapitel 12); Autor darf ihn nicht ergänzen; Rückfrage „Hast du Erfahrung mit X? Wenn ja, wo?“ | deine Antwort erweitert den Master, nicht nur diese Bewerbung |
| Sprachniveau | GER-Stufe plus umgangssprachlicher Begriff, wie im Master („Englisch (C1, verhandlungssicher)“) ([Karrierebibel](https://karrierebibel.de/sprachkenntnisse-lebenslauf/)) | Master |

Wo genau die rechtliche Grenze zwischen zulässiger Betonung und Täuschung im Einzelfall verläuft, konnte die Recherche nicht mit einer belastbaren Quelle klären; Kapitel 16 nimmt den Punkt als juristisch zu prüfende Leitplanke auf. Bis dahin gilt die strengere Auslegung: Im Zweifel bleibt der Master-Wortlaut.

**Entscheidung:** Jede Bewerbung erzeugt ein Tailoring-Log als Diff gegen den Master, das im Review-Cockpit neben dem Text angezeigt wird. **Begründung:** Du prüfst in Sekunden, was verändert wurde, statt den ganzen Lebenslauf zu lesen; das Log ist zugleich der Nachweis, dass nichts erfunden wurde. **Alternative:** Nur der fertige Lebenslauf ohne Diff (schneller zu bauen, aber die Prüfung wird zur Lesearbeit).

```json
{
  "stelle_id": "2026-09-08-0142",
  "master_version": "2026-08-30",
  "operationen": [
    {"op": "kurzprofil", "neu": "Projektleiterin Logistik, 8 Jahre, SAP EWM und Lean, zuletzt Durchlaufzeit um 18 % gesenkt",
     "belege": ["MASTER.kopf", "S-004", "S-011"]},
    {"op": "kernkompetenzen", "begriffe": ["SAP EWM", "Lean Management", "Kanban", "Lieferantensteuerung"],
     "belege": ["MASTER.kenntnisse.sap_ewm", "MASTER.kenntnisse.lean", "MASTER.kenntnisse.kanban", "S-007"],
     "anzeige_begriffe": ["M-2", "M-3"]},
    {"op": "betonen", "station": "ST-3", "sichtbar": ["B-3-2", "B-3-5", "B-3-1"], "ausgeblendet": ["B-3-3", "B-3-4"],
     "grund": "M-2 verlangt Lagerprozesse; B-3-3/B-3-4 betreffen Vertrieb"},
    {"op": "spiegeln", "station": "ST-3", "bullet": "B-3-2", "von": "Excel", "nach": "MS Excel",
     "beleg": "MASTER.synonyme.excel"},
    {"op": "kuerzen", "station": "ST-6", "auf": "titel_firma_zeitraum_1bullet", "grund": "älter als 10 Jahre, fachfremd"}
  ],
  "nicht_erlaubt_angefragt": [
    {"anzeige_begriff": "Six Sigma Green Belt", "status": "nicht im Master", "aktion": "Rückfrage offen"}
  ]
}
```

### 11.8 Der Prozess: vom Briefing bis zur Übergabe

Die Schleife folgt dem Muster „Entwurf → Prüfung gegen Kriterien → Überarbeitung“, das Anthropic als häufigstes Prompt-Chaining-Muster beschreibt, weil jeder Schritt als eigener Aufruf protokollierbar und abzweigbar ist ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)), und dem dreistufigen Vorgehen des Anthropic-Skills `doc-coauthoring`: Kontext sammeln, Optionen erzeugen und kuratieren, Entwurf, dann Leser-Test mit einem frischen Kontext ohne Wissen über den Prompt ([doc-coauthoring SKILL.md](https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md)).

| Schritt | Komponente | Modell, effort | Ausgabe | Abbruch/Rückfrage |
|---|---|---|---|---|
| 0 Vorbedingungen | Orchestrator | – | Status „recherchiert“, Knockout bestanden (R07), Format entschieden | fehlende Voraussetzung → zurück an Rechercheur |
| 1 Briefing | Autor (Vorstufe) | Sonnet 5, medium | `briefing.json` | Muss-Kriterium ohne Story → Hinweis an Matcher |
| 2 Entwürfe | Autor | Fable 5.1, high | 2–3 Anschreiben-Varianten; getrennt: Lebenslauf-Tailoring + Log | `[UNBELEGT]`-Marker → Rückfrage offen |
| 3 Kritik | Kritiker | deterministische Prüfungen, dann Opus 5, high | Kritikbericht (JSON) je Variante | hartes Gate → zurück an Autor |
| 4 Überarbeitung | Autor | Fable 5.1, medium | Version n+1 der besten Variante | nach 2 Schleifen ohne Schwelle → Review-Cockpit „Kritiker unzufrieden“ |
| 5 Fakten-Check | Kritiker | Sonnet 5, medium, Structured Output | Claim-Liste mit Belegstatus | unbelegte Aussage → streichen oder Rückfrage |
| 6 Stimm-Check und Leser-Test | Kritiker (frischer Kontext) | Opus 5, medium | Stimm-Score, Sätze mit Abweichung, Recruiter-Eindruck | Stimm-Score unter Schwelle → Überarbeitung nur dieser Sätze |
| 7 Konsistenz-Check | Kritiker | deterministisch + Haiku 4.5, low | Abgleich Anzeige/Lebenslauf/Anschreiben | Widerspruch → zurück an Autor oder Rückfrage |
| 8 Übergabe | Orchestrator | – | Dossier, Status „geschrieben“ → „geprüft“ | ATS-Prüfer (Kapitel 12), Setzer (13), Cockpit (14); ein Kommentar aus dem Cockpit startet die Schleife ab Schritt 4 neu |

**Schritt 1, Briefing.** Aus den Eingaben entsteht eine kompakte Datei, die alles enthält, was der Autor braucht, und nichts, was ihn ablenkt: Anzeigen-Extrakt (Muss/Kann, Sprache, Anrede, Ton), Firmenfakten mit Konfidenz und Quelle, die drei bis fünf passendsten Story-Bank-Einträge (Sonnet 5 bewertet jede Story gegen jedes Muss-Kriterium auf einer 0–3-Skala, Structured Output; die STAR-Struktur der Story-Bank mit Situation, Aufgabe, Aktion, Ergebnis mit Zahl und Learning stammt aus Kapitel 8 ([CareerScribe](https://blog.careerscribeai.com/star-story-bank-template/))), Stimmprofil-Auszug (Regeln, Tabus, drei bis fünf Textproben), Format und Constraints (Wortkorridor, Anrede, Gehaltsblock ja/nein, Sprache).

```json
{
  "stelle_id": "2026-09-08-0142",
  "format": "voll",
  "sprache": "de",
  "anrede": {"form": "Sie", "text": "Sehr geehrte Frau Dr. Yilmaz", "konfidenz": "hoch"},
  "anzeige": {
    "titel": "Projektleiter Intralogistik (m/w/d)", "kennziffer": "LOG-2026-117",
    "muss": [{"id": "M-1", "text": "mind. 5 Jahre Projektleitung"}, {"id": "M-2", "text": "SAP EWM"}, {"id": "M-3", "text": "Lean-Methoden"}],
    "kann": [{"id": "K-1", "text": "Englisch verhandlungssicher"}]
  },
  "firmenfakten": [
    {"id": "F-1", "text": "Neues Logistikzentrum in Kassel, Inbetriebnahme Q1/2027", "konfidenz": "hoch", "quelle": "{URL der Pressemitteilung aus dem Rechercheur-Dossier}"},
    {"id": "F-2", "text": "Team Intralogistik 14 Personen, Leitung Dr. Yilmaz", "konfidenz": "mittel", "quelle": "{URL der Teamseite aus dem Rechercheur-Dossier}"}
  ],
  "stories": [
    {"id": "S-004", "relevanz": {"M-1": 3, "M-2": 3, "M-3": 2}, "kurz": "Migration auf SAP EWM an 2 Standorten, 9 Monate, Durchlaufzeit -18 %"},
    {"id": "S-011", "relevanz": {"M-3": 3}, "kurz": "Kanban-Einführung Wareneingang, Bestände -22 % in 6 Monaten"},
    {"id": "S-007", "relevanz": {"M-1": 2}, "kurz": "Lieferantenumstellung, 40 Lieferanten, ohne Lieferausfall"}
  ],
  "stimmprofil": {"regeln": ["kurze Hauptsätze", "keine Superlative", "Zahlen als Ziffern"], "tabus": ["spannend", "Leidenschaft"],
                  "proben": ["…", "…", "…"], "metriken": {"satzlaenge_mittel": 14.2, "satzlaenge_sd": 7.1}},
  "constraints": {"woerter_min": 250, "woerter_max": 350, "gehalt": false, "eintritt": true, "eintritt_wert": "01.01.2027"}
}
```

**Schritt 2, Entwürfe.** Ein Aufruf erzeugt alle Varianten. **Entscheidung:** Drei Varianten in einem Aufruf mit vorgegebenen Einstiegstypen (Leistung, Firmenbezug, Motiv), nicht drei getrennte Aufrufe. **Begründung:** Der stabile Prefix (Stimmprofil, Regeln, Master) wird einmal gelesen; das Modell sorgt selbst für Verschiedenheit der Varianten, was bei getrennten Aufrufen nicht garantiert ist; Kosten sinken um etwa zwei Drittel des Inputs. **Alternative:** Getrennte Aufrufe bei Bedarf an echter Unabhängigkeit (z. B. für Evaluationen, Best-of-N-Verifikation nach [Anthropic-Leitfaden](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)). Das Lebenslauf-Tailoring ist ein eigener Aufruf (Fable 5.1, effort medium), weil es eine andere Aufgabe ist: Auswahl und Umordnung unter strengen Regeln, kein freies Schreiben.

**Schritt 3, Kritik.** Erst die deterministischen Prüfungen aus 11.5 (kostenlos), dann der Rubrik-Aufruf. Der Kritiker bewertet jede Variante einzeln gegen die Rubrik, nie paarweise: Bei paarweisen Vergleichen kann allein die Reihenfolge der Präsentation die Bewertung um mehr als 10 Prozentpunkte verschieben ([arXiv 2602.02219](https://arxiv.org/html/2602.02219v2), [Adaline](https://www.adaline.ai/blog/llm-as-a-judge-reliability-bias)). Die Rubrik hat Zahlenanker statt Adjektive, wie Anthropic es für LLM-Grading empfiehlt, und das bewertende Modell ist ein anderes als das schreibende ([Develop tests](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests), [Define success](https://platform.claude.com/docs/en/test-and-evaluate/define-success)). Der Kritiker wählt die beste Variante als Basis und liefert Überarbeitungsanweisungen; er darf auch Absätze aus zwei Varianten kombinieren, wenn er das im Bericht begründet.

**Schritt 4, Überarbeitung.** Der Autor erhält die gewählte Variante, den Kritikbericht und die Anweisung, nur die genannten Punkte zu ändern. Höchstens zwei Schleifen. Danach entscheidet der Orchestrator: Liegt der Gesamtwert über der Freigabeschwelle, weiter; sonst landet der Fall mit dem Bericht im Review-Cockpit, und du entscheidest, ob du selbst formulierst, die Stelle verwirfst oder eine dritte Schleife freigibst.

**Schritt 5, Fakten-Check gegen die Story-Bank.** Das Muster kommt aus dem Anthropic-Leitfaden zur Halluzinationsvermeidung: Behauptungen extrahieren, für jede ein wörtliches Zitat aus den bereitgestellten Dokumenten suchen, ohne Beleg zurückziehen und die Lücke markieren; externes Wissen ausdrücklich ausschließen ([Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)). Der Prüfer bekommt das Anschreiben, den angepassten Lebenslauf, die Story-Bank und den Master, aber nicht das Briefing und nicht den Schreib-Prompt.

```json
{
  "stelle_id": "2026-09-08-0142",
  "variante": "B",
  "claims": [
    {"nr": 1, "zitat": "Durchlaufzeit im Wareneingang um 18 Prozent gesenkt", "typ": "zahl",
     "beleg": {"quelle": "S-004", "zitat_quelle": "Durchlaufzeit von 41 auf 33,6 Stunden (−18 %)"}, "status": "belegt"},
    {"nr": 2, "zitat": "an zwei Standorten", "typ": "umfang",
     "beleg": {"quelle": "S-004", "zitat_quelle": "Standorte Kassel und Fulda"}, "status": "belegt"},
    {"nr": 3, "zitat": "Inbetriebnahme Ihres Logistikzentrums Anfang 2027", "typ": "firmenfakt",
     "beleg": {"quelle": "F-1", "konfidenz": "hoch"}, "status": "belegt"},
    {"nr": 4, "zitat": "Lieferanten ohne einen einzigen Lieferausfall umgestellt", "typ": "umfang",
     "beleg": {"quelle": "S-007", "zitat_quelle": "ohne Lieferausfall"}, "status": "belegt"},
    {"nr": 5, "zitat": "Six-Sigma-Erfahrung", "typ": "kenntnis",
     "beleg": null, "status": "unbelegt", "aktion": "streichen; Rückfrage an Nutzer (Anzeige verlangt Green Belt)"}
  ],
  "zusammenfassung": {"belegt": 4, "teilbelegt": 0, "unbelegt": 1, "gate": "nicht bestanden"}
}
```

Eine unbelegte Aussage ist ein hartes Gate. Ob sie automatisch gestrichen oder dir als Rückfrage vorgelegt wird, hängt vom Typ ab: Zahlen und Umfänge werden gestrichen und gemeldet; Kenntnisse und Erfahrungen, die die Anzeige verlangt, werden zur Rückfrage, weil deine Antwort den Master dauerhaft verbessert.

**Schritt 6, Stimm-Check und Leser-Test.** Zwei getrennte Aufrufe mit jeweils frischem Kontext, nach dem Reader-Testing-Muster des `doc-coauthoring`-Skills: Der Prüfer kennt weder Prompt noch Briefing, nur das, was auch ein Außenstehender hätte. Der Stimm-Check bekommt das Stimmprofil (Regeln, Tabus, Textproben, Metriken) und das Anschreiben und beantwortet satzweise „Würde diese Person das so sagen?“ mit Begründung; zusätzlich vergleicht deterministischer Code die Satzlängen-Statistik und die Ich-Quote mit deinen Textproben. Der Leser-Test bekommt nur Anzeige und Anschreiben und antwortet als Recruiter in fünf Fragen: Was ist die konkrete Leistung? Warum diese Firma? Welche zwei Muss-Kriterien sind belegt? Klingt der Text nach Textbaustein? Würde ich nach 30 Sekunden weiterlesen? Antworten, die „weiß nicht“ oder „Textbaustein“ enthalten, gehen als Befund in die Überarbeitung.

**Schritt 7, Konsistenz-Check.** Deterministisch, wo möglich; Haiku 4.5 nur für die semantischen Vergleiche.

| Prüfung | Anzeige | Lebenslauf | Anschreiben |
|---|---|---|---|
| Stellentitel und Kennziffer | Quelle | Kurzprofil darf abweichen (eigene Rolle) | Betreff identisch mit Anzeige |
| Firmenname, Rechtsform, Ansprechpartner | Rechercheur | – | Anschrift und Anrede identisch mit Rechercheur-Feldern |
| Zahlen und Erfolge | – | Bullet mit Story-ID | dieselbe Zahl, dieselbe Story-ID; nichts im Anschreiben, was im Lebenslauf fehlt |
| Werkzeuge, Zertifikate | Muss/Kann | Kernkompetenzen und Kenntnisse | nur Begriffe, die im Lebenslauf stehen |
| Sprache, Anrede | Sprache der Anzeige | gleiche Sprache; Standardüberschriften | gleiche Sprache; Du/Sie durchgehend |
| Termine, Gehalt | fragt / fragt nicht | – | nur wenn gefragt; Werte identisch mit Profilfeldern |
| Datum, Version | – | Dossier-Version | dieselbe Version; Datum setzt der Setzer |

**Kosten pro Bewerbung, Größenordnung.** Autor und Kritiker sind zwei von fünf Schritten der Pipeline (Rechercheur, Autor, Kritiker, ATS-Prüfer, Setzer, Kapitel 10–13); ihr Anteil lässt sich deshalb nicht als eigener Gesamtpreis je Bewerbung lesen. Im „empfohlen“-Szenario aus Kapitel 18 entfallen auf Autor und Kritiker zusammen rund 2,00 US-Dollar der insgesamt rund 3,71 US-Dollar (≈ 3,45 €) pro Bewerbung: rund 1,74 US-Dollar auf den Autor (Briefing, zwei Entwürfe, Überarbeitung) und rund 0,26 US-Dollar auf den Kritiker (Rubrik, Fakten-Check, Stimm- und Leser-Test, Konsistenz-Check). Prompt Caching senkt den Input-Anteil dieser Schleife zusätzlich: Cache-Lesen kostet bei Fable 5.1 0,025-fach, Mindestlänge des Prefix 512 Tokens ([Prompt Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)). Verbindlich ist die Rechnung in Kapitel 18; die hier genannte Größenordnung ist nur eine Plausibilitätsprobe.

### 11.9 Die Rubrik

Sieben Kriterien, Skala 1 bis 5 mit definierten Ankern, gewichtete Summe. Zwei Kriterien tragen zusätzlich harte Gates: Ein Gate-Verstoß schickt den Entwurf unabhängig vom Gesamtwert zurück. Die Anker nennen Zahlen, keine Adjektive, damit zwei Kritiker-Läufe dasselbe messen.

| Kriterium | Gewicht | Anker 1 | Anker 3 | Anker 5 | Gate / Schwelle |
|---|---|---|---|---|---|
| K1 Belegdichte | 20 % | kein Erfolg mit Zahl | 1 Erfolg mit Zahl, 1 ohne | 2–3 Erfolge, jeder mit Zahl und Story-ID | mind. 3 |
| K2 Stellen- und Firmenbezug | 20 % | nur Anzeigen-Selbstbeschreibung wiederholt | 1 recherchiertes Detail, Muss-Kriterien teilweise | 2 Details (Konfidenz ≥ mittel), alle erfüllten Muss-Kriterien belegt | mind. 3 |
| K3 Stimme | 15 % | Stimmprofil-Regeln verletzt, Tabus enthalten | Regeln eingehalten, 2–3 Sätze „klingt nicht nach mir“ | alle Sätze im Profil, Metriken im Korridor | Tabu-Treffer = Gate |
| K4 Floskel- und Musterfreiheit | 15 % | ≥ 2 harte Treffer | 0 harte, 2–3 weiche Treffer | 0 harte, ≤ 1 weicher Treffer, Satzlängen-Varianz im Korridor | harter Treffer = Gate |
| K5 Struktur und Lesefluss | 10 % | Standardeinstieg, Lebenslauf nacherzählt, > 350 Wörter | Skelett eingehalten, Absätze gleichförmig | Einstieg konkret, 4 Absätze mit Zweck, Wortkorridor, Lesbarkeit Stufe 8–10 | mind. 3 |
| K6 Faktentreue | 10 % | ≥ 1 unbelegte Zahl oder Kenntnis | alles belegt, 1 Rundung unsauber | jede Aussage mit Beleg-ID, Rundungen wie in der Story-Bank | unbelegt = Gate |
| K7 Formales | 10 % | Anrede/Name falsch, Gehalt ungefragt, Konjunktiv-Schluss | 1 Formfehler | Anrede korrekt, Betreff vollständig, Du/Sie durchgehend, Schluss im Indikativ, Gehalt/Eintritt regelkonform | Anrede-Fehler = Gate |

Schwellen: gewichteter Gesamtwert mindestens 4,0 und jedes Kriterium mindestens 3 → „geprüft“; 3,3 bis 3,9 → eine Überarbeitung mit gezielten Anweisungen; unter 3,3 → neue Entwürfe mit anderem Einstiegstyp. Die Schwellen sind Startwerte; nach den ersten 20 Bewerbungen kalibrierst du sie im Review-Cockpit anhand deiner Freigaben und Ablehnungen (Feedback-Schleife, Kapitel 14). Der Kritiker liefert außerdem eine Selbsteinschätzung seiner Sicherheit je Kriterium, damit unsichere Bewertungen im Cockpit markiert werden. Ein weicher Floskeltreffer (Schwere „weich“, 11.5) senkt nur die Note des betroffenen Kriteriums; ist die Freigabeschwelle trotzdem erreicht, erscheint er im Kritikbericht als nicht blockierender Hinweis, den der Autor bei der nächsten Überarbeitung aufgreifen kann, aber nicht muss. Ein harter Treffer bleibt in jedem Fall ein Gate (11.5).

**Beispiel-Kritikausgabe** (Structured Output, Schema in `kritik.schema.json`): Der Entwurf erreicht die Freigabeschwelle (Gesamt 4,25, kein Kriterium unter 3, keine Gates verletzt); `entscheidung` lautet deshalb „geprueft“. Der weiche Treffer D27 unter K4 und die Einträge unter `anweisungen` sind hier nicht blockierende Hinweise für die nächste Bewerbung, keine Bedingung für diese Freigabe.

```json
{
  "stelle_id": "2026-09-08-0142",
  "variante": "B",
  "runde": 1,
  "modell": "claude-opus-5",
  "deterministisch": {
    "woerter": 296, "satzlaenge_sd": 6.4, "satzlaenge_sd_nutzer": 7.1, "wiener_stufe": 9.1,
    "anzeige_kopie_8gramm": 0, "floskeln_hart": [], "floskeln_weich": ["D27"], "ich_quote": 0.31
  },
  "kriterien": {
    "K1": {"wert": 4, "sicherheit": 0.9, "befund": "Zwei Erfolge mit Zahl (S-004, S-011); dritter Erfolg (S-007) ohne Zahl."},
    "K2": {"wert": 5, "sicherheit": 0.8, "befund": "F-1 und F-2 korrekt eingebunden; M-1 bis M-3 jeweils mit Beleg."},
    "K3": {"wert": 3, "sicherheit": 0.7, "befund": "Satz 7 und Satz 12 formeller als die Textproben; Tabus nicht enthalten."},
    "K4": {"wert": 4, "sicherheit": 0.9, "befund": "Ein weicher Treffer D27 in Absatz 2."},
    "K5": {"wert": 4, "sicherheit": 0.8, "befund": "Einstieg mit Firmenbezug; Absatz 3 mit 96 Wörtern zu lang gegenüber Absatz 4 mit 28."},
    "K6": {"wert": 5, "sicherheit": 0.95, "befund": "Fakten-Check: 5 Claims, alle belegt (nach Streichung von Six Sigma in Runde 0)."},
    "K7": {"wert": 5, "sicherheit": 0.95, "befund": "Anrede identisch mit Rechercheur; Eintritt genannt, Gehalt nicht (Anzeige fragt nicht)."}
  },
  "gates": {"tabu": "ok", "floskel_hart": "ok", "unbelegt": "ok", "anrede": "ok", "anzeige_kopie": "ok"},
  "gesamt": 4.25,
  "entscheidung": "geprueft",
  "anweisungen": [
    {"absatz": 2, "satz": 7, "problem": "„maßgeblich beigetragen“ ohne Zahl (D27)", "vorschlag": "Ergebnis aus S-007 beziffern oder Satz streichen"},
    {"absatz": 2, "satz": 7, "problem": "Stimme: Nominalstil, Textproben nutzen Verben", "vorschlag": "Verb voran: „Ich habe 40 Lieferanten … umgestellt“"},
    {"absatz": 3, "problem": "96 Wörter, Absatz 4 nur 28", "vorschlag": "Absatz 3 auf ca. 70 Wörter kürzen, Absatz 4 um konkreten Gesprächsanlass ergänzen"},
    {"absatz": 4, "satz": 12, "problem": "„stehe ich Ihnen gerne zur Verfügung“ (D35)", "vorschlag": "streichen"}
  ],
  "leser_test": {
    "leistung_erkannt": "SAP-EWM-Migration, Durchlaufzeit −18 %",
    "warum_firma": "neues Logistikzentrum Kassel 2027",
    "muss_belegt": ["M-1", "M-2"],
    "textbaustein_eindruck": "nein",
    "weiterlesen": "ja"
  },
  "wiederholung": {"aehnlichster_einstieg": "2026-09-02-0087", "ueberlappung": 0.22, "hinweis": null}
}
```

### 11.10 Prompting für Fable 5.1 (Autor) und Opus 5 (Kritiker)

Die Modellwahl folgt dem Styleguide: Fable 5.1 für das Schreiben, weil es die Wunschbasis für qualitätskritische Schritte ist; Opus 5 als Kritiker, weil ein anderes Modell bewerten soll als das schreibende und Opus 5 nicht unter die 30-Tage-Speicherpflicht der „Covered Models“ fällt ([Data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)). **Alternative:** Opus 5 als Autor, falls du die 30-Tage-Speicherung bei Anthropic für deine Bewerbungstexte nicht willst (Kapitel 16); dann bewertet Fable 5.1 oder Sonnet 5.

**Grundsätze, die aus der Anthropic-Dokumentation folgen:**

1. **Ziel und Constraints statt Schrittfolge.** Für Opus 5 und Fable 5.1 senken rigide Schritt-für-Schritt-Vorgaben die Qualität, weil die Modelle Sequenzierung und Formatierung selbst beherrschen; aggressive Formulierungen („CRITICAL: You MUST …“) führen zu Übertriggern und sollen durch normale Sprache ersetzt werden ([Prompting Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5), [Charles Jones](https://charlesjones.dev/blog/claude-opus-5-context-engineering-what-to-delete)). Der Schreib-Prompt nennt deshalb Rolle, Material, Ziel, wenige Leitplanken und das Format – nicht die 38 Floskeln, nicht die 15 Schreibregeln. Die vollständigen Regeln gehören dem Kritiker und dem deterministischen Filter.
2. **Struktur mit XML-Tags, Beispiele aus dem Stimmprofil.** Getrennte Tags für Anzeige, Firmenfakten, Stories, Stimmprofil, Regeln und Format; drei bis fünf Beispiele, relevant und verschieden genug, damit das Modell keine unbeabsichtigten Muster übernimmt, in `<example>`-Tags ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). Die Beispiele sind deine echten Textproben (Kapitel 8), keine Muster-Anschreiben – Muster-Anschreiben würden die Generik einschleusen, die wir vermeiden wollen.
3. **Länge explizit vorgeben.** `effort` steuert die Denktiefe, nicht die sichtbare Textlänge; für harte Wortgrenzen braucht der Prompt eine ausdrückliche Längenanweisung ([Effort](https://platform.claude.com/docs/en/build-with-claude/effort), [Prompting Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5)). Der deterministische Wortzähler bleibt trotzdem das Gate.
4. **Manierierte Prosa benennen.** Fable 5.1 schreibt mit weniger Stockphrasen als Vorgänger, neigt aber zu dichteren, längeren Sätzen; Anthropic empfiehlt eine Anweisung, die das Anti-Muster definiert („mannered prose“: Metapher und Schmuck statt direkter Aussage), wahlweise kurz: „Please remove all mannered prose“ ([Prompting Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)). Eine deutsche Fassung steht im Prompt unten.
5. **Quellen nicht abschreiben.** Fable 5.1 reproduziert beim Zusammenfassen häufiger Passagen der Quelle ohne Kennzeichnung ([Prompting Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)). Für uns heißt das: Die Anzeige ist Quelle, und der Prompt sagt ausdrücklich „Fachbegriffe übernehmen, Sätze nicht“; die 8-Gramm-Prüfung fängt den Rest.
6. **Unsicherheit erlauben.** Das Modell darf und soll `[UNBELEGT: …]` schreiben, statt eine Lücke zu füllen ([Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)).
7. **Keine Anti-Formatierungsregeln, keine Sparsamkeitsregeln für Zwischenmeldungen.** Fable 5.1 formatiert von sich aus zurückhaltend; alte Regeln gegen Fettdruck und Listen sind zu entfernen ([Prompting Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)). Für das Anschreiben gilt ohnehin: reiner Fließtext, keine Listen.
8. **Der Kritiker soll alles melden.** Opus 5 folgt Anweisungen wie „nur schwere Befunde“ wörtlich und meldet dann weniger; besser alles melden lassen und in der Rubrik gewichten ([Prompting Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5)).
9. **Effort-Stufen sweepen.** Fable 5.1 startet bei `high`; `medium` entspricht etwa Fable 5 bei geringeren Kosten, und bei `low` ist Fable 5.1 laut Anthropic bei den Kosten pro Aufgabe oft konkurrenzfähig mit Opus- und Sonnet-Modellen und schneidet dabei besser ab; bei `xhigh` und `max` vor langen Texten Platz in `max_tokens` lassen ([Prompting Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)). Die Stufen in der Prozesstabelle (11.8) sind Startwerte; nach 20 Bewerbungen den Sweep mit deinen Freigabedaten wiederholen.
10. **Refusals abfangen.** Fable 5.1 kann `stop_reason: "refusal"` liefern ([Prompting Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)); der Orchestrator wiederholt den Aufruf dann mit Opus 5 und protokolliert den Fall (Kapitel 7).
11. **Stabiler Prefix für Caching.** System-Prompt, Regeln, Stimmprofil und Master-Lebenslauf bilden den gecachten Prefix; nur Briefing und Anzeige wechseln. Der Prefix muss byteidentisch bleiben, sonst gibt es keinen Cache-Treffer ([Prompt Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)).

**System-Prompt des Autors (Skelett, deutsch):**

```text
Du schreibst Bewerbungsanschreiben für eine Person, deren Unterlagen und
Schreibproben du unten findest. Du schreibst in ihrer Stimme, nicht in deiner.

<ziel>
Ein Anschreiben, das ein Recruiter in 30 Sekunden als „hat sich mit uns
beschäftigt und kann belegen, was er sagt“ erkennt.
</ziel>

<material>
Nutze ausschließlich die Fakten in <stories>, <master> und <firmenfakten>.
Wenn dir eine Angabe fehlt, schreibe [UNBELEGT: was fehlt] statt sie zu ergänzen.
Fachbegriffe aus <anzeige> darfst du übernehmen, wenn sie in <master> belegt sind;
Sätze oder Satzteile der Anzeige übernimmst du nicht.
</material>

<stimme>
Halte dich an die Regeln und Tabus in <stimmprofil>. Die <examples> sind echte
Texte dieser Person; triff ihren Tonfall, Satzbau und ihre Wortwahl.
Vermeide manierierte Prosa: Metaphern und Schmuck statt direkter Aussage.
Wenn eine wörtliche Formulierung verfügbar ist, nimm sie.
Ein Anschreiben klingt nach einem Menschen, wenn Satzlängen wechseln und
nicht jeder Satz mit „Ich“ beginnt.
</stimme>

<format>
Schreibe {anzahl} Varianten mit den Einstiegstypen {einstiege}.
Jede Variante: {woerter_min}–{woerter_max} Wörter, vier Absätze nach <skelett>,
Anrede „{anrede}“, reiner Fließtext ohne Listen.
Gib jede Variante in <variante id="A" einstieg="leistung">…</variante> aus,
Absätze durch Leerzeilen getrennt. Nach den Varianten nichts weiter.
</format>
```

Die User-Nachricht enthält dann die Tags `<anzeige>`, `<firmenfakten>` (nur Konfidenz hoch und mittel), `<stories>`, `<master>` (Auszug), `<stimmprofil>` mit `<examples>`, `<skelett>` und `<constraints>` – in dieser Reihenfolge, stabile Teile zuerst.

**System-Prompt des Kritikers (Skelett):**

```text
Du bist eine erfahrene Recruiterin in {branche} in Deutschland und prüfst ein
Bewerbungsanschreiben für die Rolle in <anzeige>. Du bewertest jede Variante
einzeln gegen <rubrik>; du vergleichst Varianten nicht miteinander.

Melde jeden Befund, auch kleine; die Gewichtung übernimmt die Rubrik.
Für jedes Kriterium: Wert 1–5 nach den Ankern, ein Satz Befund mit Absatz- und
Satznummer, deine Sicherheit 0–1. Prüfe jede Zahl und Kenntnis gegen <stories>
und <master>; was dort fehlt, ist unbelegt. Prüfe jeden Satz gegen <stimmprofil>.

Gib ausschließlich JSON nach <schema> aus.
```

Für Subagents in Claude Code liegen Rolle, Modell, Effort und Werkzeugrechte im Frontmatter ([Subagents](https://code.claude.com/docs/en/sub-agents)); Skills für wiederholbare Abläufe tragen `model`, `effort` und `allowed-tools` in der `SKILL.md` ([Skills](https://code.claude.com/docs/en/skills)). Beispiel für den Autor:

```markdown
---
name: autor
description: Schreibt Anschreiben-Varianten und passt den Lebenslauf aus dem Master an. Nutzen, wenn eine Stelle den Status "recherchiert" hat und das Briefing vorliegt.
model: claude-fable-5-1
effort: high
tools: Read, Write
skills: anschreiben, lebenslauf-tailoring
---
Du arbeitest nur mit den Dateien im Ordner der Stelle: briefing.json, stories.json,
master.json, stimmprofil.json. Du liest keine anderen Bewerbungen und keine Webseiten.
Ausgabe: entwuerfe.md (Varianten in <variante>-Tags) und tailoring_log.json.
```

Der Kritiker bekommt `model: claude-opus-5`, `effort: high`, `tools: Read` und keinen Schreibzugriff auf die Entwürfe; er schreibt nur `kritik.json`. Die Trennung nach dem Tool-Minimalprinzip ist die technische Form der Rollentrennung: Der Kritiker kann nichts „reparieren“, nur melden. Im Agent SDK liefert `output_format` mit JSON-Schema das validierte `structured_output`-Feld und wiederholt bei Schemafehlern automatisch ([Structured outputs](https://code.claude.com/docs/en/agent-sdk/structured-outputs)); auf der Messages API dient dafür Structured Outputs ([Structured outputs API](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)). Der Fließtext der Entwürfe bleibt außerhalb von JSON, weil Schema-Zwang die Prosa steifer macht.

**Was nicht in den Prompt gehört:** Die vollständige Floskelliste (D01–D38); „Sei kreativ/authentisch/individuell“ (leere Anweisungen); Muster-Anschreiben; Firmenfakten mit Konfidenz niedrig; frühere Anschreiben des Nutzers an andere Firmen (Wiederholungsrisiko; der Kritiker vergleicht sie, der Autor sieht sie nicht); die Rubrik selbst (der Autor soll schreiben, nicht auf Punkte optimieren).

### 11.11 Englische Bewerbungen

Auslöser: Anzeige auf Englisch, Konzernsprache Englisch laut Rechercheur, ausdrückliche Anforderung. Sprache der Unterlagen ist die Sprache der Anzeige (R10, Kapitel 4). Englische Cover Letter sind keine Übersetzung: direktere Selbstdarstellung, Kürze und persönlicher Ton statt formaler Struktur, bis 400 Wörter auf einer Seite; die britische Form hat eine Betreffzeile, die amerikanische nicht ([Lebenslaufdesigns](https://lebenslaufdesigns.de/anschreiben-englisch), [Elinora](https://elinora.net/cover-letter-deutsch)). Für den Lebenslauf gilt: kein Foto, kein Geburtsdatum, kein Familienstand, Struktur „Professional Experience / Education / Skills“, ein bis zwei Seiten; Foto und Alter gelten im englischen CV als Ausschlussgrund ([cvlotse](https://cvlotse.de/ratgeber/deutscher-vs-englischer-lebenslauf), [Indeed](https://at.indeed.com/karriere-guide/bewerbung/lebenslauf-englisch)). Die Vorlage `international-en` liefert der Setzer (Kapitel 13).

Regeln für Autor und Kritiker im englischen Modus:

- Eigener Skelett-Satz (11.6, englische Variante) und der englische Anti-Generik-Katalog (E01–E36); der deutsche Katalog ist abgeschaltet.
- Stimmprofil: Deine englischen Textproben, sofern vorhanden; sonst die übersetzten Stilregeln plus die ausdrückliche Anweisung, direkter zu formulieren als im Deutschen und Selbstlob mit Beleg zu koppeln. Ohne englische Textproben ist die Stimm-Treue geringer; der Kritiker markiert das im Bericht.
- Anrede: namentlich („Dear Dr Yilmaz“), Fallback „Dear Hiring Team at {Firma}“; nie „To whom it may concern“ (E03).
- Muss-Kriterien, Story-IDs, Fakten-Check und Konsistenz-Check identisch; der Leser-Test läuft mit einer englischsprachigen Recruiter-Rolle in Deutschland.
- Mischfälle: Deutsche Anzeige mit „English CV welcome“ → deutsches Anschreiben, deutscher Lebenslauf, Hinweis im Cockpit; englische Anzeige einer deutschen Firma mit deutschem Ansprechpartner → englisch, weil die Anzeige die Sprache vorgibt.

**Entscheidung:** Default ist die britische Form mit Betreffzeile und britischer Schreibweise. **Begründung:** Der Empfänger sitzt in Deutschland, wo die Betreffzeile dem gewohnten Briefbild entspricht; die DIN-Vorlage des Setzers trägt sie ohnehin. **Alternative:** Amerikanische Form ohne Betreff für US-Konzerne mit US-Recruiting-Team; als Profilschalter vorgesehen, Default in Kapitel 22 abzufragen.

### 11.12 Default-Annahmen und offene Fragen (für Kapitel 22)

1. **Format ohne Signal:** `voll` bei E-Mail, `kurz` bei Portal (11.3). Alternativ immer `voll`.
2. **Anrede:** „Sie“ als Default, „Du“ nur bei Du-Anzeige plus Du-Kultur plus deiner Freigabe. Willst du „Du“ grundsätzlich ausschließen oder grundsätzlich spiegeln?
3. **Rote Linien im Tailoring:** Titel-Alias, Lücken-Wortlaut und Synonym-Mapping werden einmalig im Onboarding festgelegt; danach keine Rückfragen mehr zu diesen Fällen. Alternativ jede Grauzone einzeln vorlegen.
4. **Unbelegte Aussagen:** Zahlen werden gestrichen und gemeldet, fehlende Kenntnisse werden zur Rückfrage. Alternativ alles zur Rückfrage (mehr Unterbrechungen, besserer Master).
5. **Autor-Modell:** Fable 5.1 (30-Tage-Speicherung bei Anthropic). Alternativ Opus 5 als Autor, wenn du keine Speicherung willst.
6. **Leser-Test:** eingeschaltet für `voll` und `initiativ`, ausgeschaltet für `kurz` (Kosten). Alternativ immer oder nie.
7. **Englisch:** britische Form, britische Schreibweise; englische Textproben erwünscht (wie viele kannst du liefern?).
8. **Motivationsschreiben:** nur auf ausdrückliche Anforderung der Anzeige, v2. Alternativ standardmäßig zusätzlich.
9. **Gehalt:** aus dem Profilfeld (Spanne oder „verhandelbar“); der Agent schlägt nie selbst eine Zahl vor. Alternativ Marktdaten als markierter Vorschlag (Kapitel 10 nennt Quellen).
10. **Wiederholungsprüfung:** Vergleich mit den letzten zehn freigegebenen Anschreiben. Alternativ alle, dann steigen Kosten und Prüfzeit.
11. **Rubrik-Schwellen:** 4,0 gesamt, 3 je Kriterium, zwei Überarbeitungsschleifen. Nach 20 Bewerbungen gemeinsam kalibrieren.

**Quellen dieses Kapitels:**

- § 123 BGB, Lüge im Lebenslauf (anwalt24) – https://www.anwalt24.de/fachartikel/arbeit-und-betrieb/46090
- Schönfärberei und Lüge im Lebenslauf (Kliemt) – https://kliemt.blog/2016/10/05/nur-schoenfaerberei-luege-im-lebenslauf-und-drastische-spaetfolgen/
- LLMs Still Struggle to Imitate the Implicit Writing Styles of Everyday Authors (ACL Findings 2025) – https://arxiv.org/pdf/2509.14543
- How Well Do LLMs Imitate Human Writing Style? (Jemama 2025) – https://arxiv.org/abs/2509.24930
- StepStone-Studie 2025: KI und Jobsuche – https://www.stepstone.de/e-recruiting/hr-wissen/recruiting/stepstone-studie-2025-ki-und-jobsuche
- persoblogger: Zwei von drei Bewerbungen mit KI erstellt – https://persoblogger.de/2025/05/29/studie-zwei-von-drei-bewerbungen-mit-hilfe-von-ki-erstellt-recruiterinnen-fehlt-individualitaet
- StepStone: Bewerberqualität steigern (Hiring Trends Index Q1/2025) – https://www.stepstone.de/e-recruiting/hr-wissen/recruiting/bewerberqualitaet-steigern
- Human vs AI Cover Letters: Recruiter Data (Cover Letter Copilot) – https://coverlettercopilot.ai/blog/recruiters-human-vs-ai-cover-letters
- AI Resume Statistics 2026 (JobCannon) – https://jobcannon.io/blog/ai-resume-statistics-2026
- Resume.io: 49 % lehnen KI-Lebensläufe ab (JobCannon) – https://jobcannon.io/research/stats/resumeio-49-reject
- AI Detector False Positive Rates Compared (GradPilot) – https://gradpilot.com/news/ai-detector-false-positive-rates-compared
- AI Detection in Hiring and Recruitment (WasItAIGenerated) – https://www.wasitaigenerated.com/research/ai-detection-hiring-recruitment
- AI Detector for Recruiters (GPTOne) – https://gptone.me/blog/ai-detector-for-recruiters-screen-ai-written-resumes-2026
- How Recruiters Detect AI Cover Letters (Textora) – https://www.textora.org/blog/how-recruiters-detect-ai-cover-letters
- KI-Bewerbungen: So erkennen Recruiter die Tricks (20 Minuten) – https://www.20min.ch/story/kuenstliche-intelligenz-merken-recruiter-wenn-bewerbungen-ki-generiert-sind-103298940
- The 5 Biggest Tells Something Was Written by AI (Decrypt) – https://decrypt.co/348923/5-biggest-tells-something-written-ai
- Burstiness and Perplexity (QuillBot) – https://quillbot.com/blog/ai-writing-tools/burstiness-and-perplexity/
- Bewerbung 2025: Trends (Home of Jobs) – https://homeofjobs.de/blog/bewerbung-2025-diese-trends-solltest-du-jetzt-kennen/
- Braucht man 2026 noch ein Anschreiben? (CV Score) – https://cvscore.net/de/blog/braucht-man-2026-noch-ein-anschreiben/
- Otto: Bewerben ohne Anschreiben (Pressemitteilung) – https://www.otto.de/unternehmen/en/press/du-kannst-dich-ohne-anschreiben-bewerben-otto-setzt-neue-ma%C3%9Fst%C3%A4be-im-recruitment
- Deutsche Bahn: kein Anschreiben (Talention) – https://www.talention.de/blog/die-deutsche-bahn-kein-anschreiben-fuer-mehr-bewerbungen
- Kurzbewerbung (bewerbung.net) – https://bewerbung.net/kurzbewerbung
- Unterschied Anschreiben und Motivationsschreiben (business-on.de) – https://www.business-on.de/unterschied-anschreiben-und-motivationsschreiben.html
- Anschreiben 2026: Aufbau (skill-sprinters) – https://skill-sprinters.de/blog/karriere/anschreiben-2026-aufbau/
- Bewerbungsschreiben 2026 (ulmato) – https://www.ulmato.de/anschreiben/
- Du in Stellenanzeigen (bewerbung.com) – https://bewerbung.com/du-in-stellenanzeigen/
- Bewerbung Anrede (peopleatventure) – https://www.peopleatventure.de/bewerbung-anrede
- Duzen in der Stellenanzeige (Karriereakademie) – https://www.karriereakademie.de/duzen-stellenanzeige
- Gehaltsvorstellung in der Bewerbung (bewerbung.net) – https://bewerbung.net/gehaltsvorstellung-bewerbung
- Gehaltsvorstellung formulieren (JobTeaser) – https://www.jobteaser.com/de/advices/gehaltsvorstellung-in-der-bewerbung-formulieren-so-geht-s
- Eintrittstermin nennen (Karrierebibel) – https://karrierebibel.de/bewerbung-eintrittstermin-nennen-sofort/
- Lesbarkeitsindex (fleschindex.de) – https://fleschindex.de/lesbarkeitsindex
- Lesbarkeitsindex-Tool (fair-text) – https://fair-text.com/lesbarkeitsindex-textanalyse-tool/
- Initiativbewerbung (Robert Half) – https://www.roberthalf.com/de/de/insights/bewerbungs-tipps/initiativbewerbung-erster-schritt-zum-traumjob-oder-eher-vergebene-liebesmueh
- Initiativbewerbung (Karrierebibel) – https://karrierebibel.de/initiativbewerbung/
- Anschreiben auf Englisch (Lebenslaufdesigns) – https://lebenslaufdesigns.de/anschreiben-englisch
- Cover Letter auf Deutsch (Elinora) – https://elinora.net/cover-letter-deutsch
- Deutscher vs. englischer Lebenslauf (cvlotse) – https://cvlotse.de/ratgeber/deutscher-vs-englischer-lebenslauf
- Lebenslauf Englisch (Indeed) – https://at.indeed.com/karriere-guide/bewerbung/lebenslauf-englisch
- Sprachkenntnisse im Lebenslauf (Karrierebibel) – https://karrierebibel.de/sprachkenntnisse-lebenslauf/
- You Only Get 6 Seconds of Fame (The Ladders) – https://www.theladders.com/career-advice/you-only-get-6-seconds-of-fame-make-it-count
- STAR Story Bank Template (CareerScribe) – https://blog.careerscribeai.com/star-story-bank-template/
- doc-coauthoring SKILL.md (anthropics/skills) – https://github.com/anthropics/skills/blob/main/skills/doc-coauthoring/SKILL.md
- Prompting best practices (Claude Platform Docs) – https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Prompting Claude Fable 5.1 (Claude Platform Docs) – https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1
- Prompting Claude Opus 5 (Claude Platform Docs) – https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5
- Claude Opus 5 Context Engineering: What to Delete (Charles Jones) – https://charlesjones.dev/blog/claude-opus-5-context-engineering-what-to-delete
- Effort (Claude Platform Docs) – https://platform.claude.com/docs/en/build-with-claude/effort
- Reduce hallucinations (Claude Platform Docs) – https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
- Develop tests (Claude Platform Docs) – https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
- Define success (Claude Platform Docs) – https://platform.claude.com/docs/en/test-and-evaluate/define-success
- LLM-as-a-Judge Position Bias (arXiv 2602.02219) – https://arxiv.org/html/2602.02219v2
- LLM-as-a-Judge Reliability and Bias (Adaline) – https://www.adaline.ai/blog/llm-as-a-judge-reliability-bias
- Prompt caching (Claude Platform Docs) – https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- API and data retention (Claude Platform Docs) – https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
- Structured outputs (Claude Platform Docs) – https://platform.claude.com/docs/en/build-with-claude/structured-outputs
- Structured outputs im Agent SDK (Claude Code Docs) – https://code.claude.com/docs/en/agent-sdk/structured-outputs
- Subagents (Claude Code Docs) – https://code.claude.com/docs/en/sub-agents
- Skills (Claude Code Docs) – https://code.claude.com/docs/en/skills
