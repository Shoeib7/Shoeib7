## 22. Offene Fragen an dich

Dieses Kapitel sammelt jede Frage, die in Kapitel 1 bis 21 als „offene Frage an dich“ markiert wurde, sowie alle `open_questions_for_user` aus den 15 Recherche-Dateien. Dedupliziert, gruppiert, mit Antwortoptionen und der Default-Annahme, die der Plan bis zu deiner Antwort verwendet. Die Default-Annahmen sind identisch mit denen, die in den jeweiligen Kapiteln bereits als **Entscheidung** oder **Default-Annahme** stehen – dieses Kapitel widerspricht ihnen an keiner Stelle, sondern bündelt sie an einem Ort zum Durcharbeiten.

53 Fragen in acht Gruppen: Profil & Ziele, Quellen & Regionen, Sprache & Ton, Versand & Konten, Technik & Budget, Recht & Grenzen, Produktname, Roadmap & Tempo. Acht davon sind **Blocker vor Phase 0** (Kapitel 19.3, Arbeitspaket P0-01) – ohne Antwort auf diese acht kann Woche 0 nicht sauber starten, weil kein sinnvoller Platzhalter existiert oder weil die Architektur zwei grundverschiedene Wege vorsieht. Für alle übrigen Fragen gilt: Die genannte Default-Annahme ist bereits aktiv in der Architektur verankert; du kannst mit dem Bau beginnen, ohne sie einzeln zu bestätigen, und sie später über `config/` oder das Onboarding-Interview (Kapitel 8) ändern.

### 22.1 Die acht Blocker im Überblick

| # | Frage | Default, falls unbeantwortet | Kapitel |
|---|---|---|---|
| 1 | Zielrolle(n), Branche, Senioritätsstufe | keine Default-Annahme möglich | 1, 6, 9 |
| 8 | Zielregion(en), Pendeldistanz, Remote | keine Default-Annahme möglich | 6, 9 |
| 17 | Sprache der Bewerbungen | Deutsch, Englisch nur bei eindeutigem Signal | 4, 8, 11 |
| 22 | Primäres E-Mail-Konto | technologieoffen implementiert, iCloud als einfachster Start | 7, 15 |
| 30 | Commercial-API-Key oder Pro/Max-Abo | Commercial-API-Key | 7, 16 |
| 31 | Hosting: Hetzner-VPS, Mac lokal oder Managed Agents | Hetzner-VPS in Deutschland | 7 |
| 32 | Fable 5.1 (30-Tage-Speicherung) oder durchgängig Opus 5 (ZDR) | Fable 5.1 mit Offenlegung, Opus-5-Umschalter vorhanden | 7, 11, 16 |
| 33 | Tägliches/monatliches Kostenlimit | 45 USD je Tageslauf (Kapitel 18.2); Monatsdeckel offen, Orientierung rund 262–1.151 EUR/Monat je nach Szenario, rund 789 EUR im empfohlenen Szenario | 7, 18 |

Fragen 1 und 8 haben bewusst keine Default-Annahme: Der Plan enthält an keiner Stelle eine geratene Branche, einen geratenen Beruf oder einen geratenen Ort – jedes Beispiel im Dokument ist ein Platzhalter. Ohne diese zwei Antworten kann der Scout (Kapitel 6, 9) keine einzige Quelle sinnvoll konfigurieren.

### 22.2 Profil & Ziele

**1. [Blocker] Zielrolle(n)/Berufsfeld/Branche/Senioritätsstufe.** Welche konkreten Jobtitel, Branchen und Erfahrungsstufe soll der Agent suchen? Ohne Antwort bleibt jede Zahl in diesem Dokument (10 Bewerbungen/Tag, Gewichte im Scoring, Beispieltexte) ein Platzhalter. **Default-Annahme:** keine möglich (Kapitel 1, 6, 9).

**2. Bewerbungen pro Tageslauf.** Wie viele Bewerbungen soll der Agent realistisch pro Tag vorbereiten? **Default-Annahme:** 5–10 pro Tag (Kapitel 1).

**3. Story-Bank-Umfang und Pflege.** Wie viele Erfolgsgeschichten kannst du vor dem ersten Lauf liefern, wie oft willst du die Story-Bank danach aktiv erweitern? **Default-Annahme:** mindestens 8, Ziel 10 Einträge vor Produktivstart; Pflege nach Bedarf, kein festes Intervall (Kapitel 8.11).

**4. Textproben fürs Stimmprofil.** Wie viele echte Textproben (alte Anschreiben, berufliche E-Mails, LinkedIn-/Xing-Posts) kannst du bereitstellen, und dürfen sie dauerhaft gespeichert werden? **Default-Annahme:** mindestens 5 Proben, dauerhafte Speicherung im Daten-Repository (Kapitel 8.11, 11.12).

**5. Ausländischer Bildungsabschluss.** Liegt bei dir ein ausländischer Abschluss vor, ist der Anabin-Status bekannt oder eine ZAB-Bewertung bereits beantragt? Die ZAB-Bearbeitung dauert Monate, deshalb steht diese Frage am Anfang, nicht erst bei der ersten passenden Stelle ([Anabin-Kurzanleitung](https://anabin.kmk.org/kurzanleitung/ich-moechte-feststellen-wie-mein-auslaendischer-hochschulabschluss-in-deutschland-bewertet-wird.html)). **Default-Annahme:** nein (Kapitel 5.12, 8.11).

**6. Beglaubigte Zeugnisübersetzung.** Liegen fremdsprachige Zeugnisse bereits als beglaubigte (vereidigte) Übersetzung vor ([mentorium.de](https://www.mentorium.de/zeugnisse-beglaubigt-uebersetzen/))? **Default-Annahme:** nein, bleibt offener Punkt in der Checkliste (Kapitel 5.4, 8.7).

**7. Initiativbewerbungen im Tageskontingent.** Sollen Initiativbewerbungen (ohne aktuelle Stellenanzeige) Teil des täglichen Kontingents sein oder ein separates, von dir aktiviertes Feature? **Default-Annahme:** separates Feature, nicht im Standardkontingent (Kapitel 5.12).

### 22.3 Quellen & Regionen

**8. [Blocker] Zielregion(en) und Pendeldistanz.** Wohnort, maximale Pendeldistanz bzw. Umkreis, und ob Remote-Stellen ohne Ortsbezug ebenfalls infrage kommen. Ohne diese Angabe kann der `umkreis`-Parameter der BA-Jobsuche-API (Kapitel 6.3) nicht sinnvoll gesetzt werden. **Default-Annahme:** keine möglich (Kapitel 6, 9).

**9. Watchlist von Wunscharbeitgebern.** Gibt es eine Liste von Firmen, auf die sich die direkten ATS-Feed-Integrationen (Personio, Greenhouse, Lever) konzentrieren sollen? **Default-Annahme:** du lieferst 20–50 Firmen, der Scout schlägt aus BA-Treffern weitere vor (Kapitel 6.9).

**10. Kostenpflichtige und scraping-basierte Quellen.** Dürfen „gelbe“ Quellen (SerpAPI für Google for Jobs) genutzt werden, und sollen „rote“ Scraper (Apify/JobSpy gegen StepStone, Indeed, LinkedIn) trotz ToS-Risiko eingesetzt werden, oder ausschließlich ToS-unbedenkliche Quellen? **Default-Annahme:** SerpAPI ab v1 ja, Apify/JSearch nein (Kapitel 6.9, [SerpAPI-Preise](https://serpapi.com/pricing)).

**11. Monatsbudget für Datenquellen.** Welches Budget ist für SerpAPI, Firecrawl, ggf. Northdata/OpenCorporates für Firmendaten akzeptabel? **Default-Annahme:** 0 EUR im MVP, bis ca. 41 USD in v1; ein laufendes Northdata-Abo wird nicht abgeschlossen, Websuche plus Impressum reichen zunächst (Kapitel 6.9, company_research.json).

**12. LinkedIn/XING-Einbeziehung.** Soll LinkedIn in irgendeiner Form einbezogen werden, und sei es nur manuell durch dich kuratiert, oder komplett ausgeschlossen? Die LinkedIn-Nutzervereinbarung verbietet automatisierten Zugriff ausdrücklich ([LinkedIn](https://www.linkedin.com/help/linkedin/answer/a1341387/verbotene-software-und-erweiterungen?lang=de-DE)). **Default-Annahme:** kein automatisierter Zugriff; nur ein einmaliger, manueller Konsistenz-Check zwischen Profil und Lebenslauf im Onboarding (Kapitel 3.3, 5.12).

**13. Personalvermittler-/Zeitarbeit-Anzeigen.** Sollen sie standardmäßig einbezogen, ausgeschlossen oder nur markiert werden? **Default-Annahme:** einbeziehen, mit Label „Personalvermittler (Verdacht)“ und Rückfrage bei mittlerer Konfidenz (Kapitel 9.7).

**14. Ansprechpartner-Recherche auf LinkedIn/XING.** Bist du bereit, diesen Schritt selbst manuell zu übernehmen (der Agent liefert dir einen vorbereiteten Link), oder soll stärker automatisiert werden trotz ToS-Risiko? **Default-Annahme:** du übernimmst es manuell (company_research.json).

**15. Rückfrage-Schwelle beim Ansprechpartner.** Soll bei niedriger Konfidenz immer eine Rückfrage ohne Default kommen, oder reicht eine Rückfrage mit Default, die sich von selbst auflöst? **Default-Annahme:** Nur eine unsichere oder widersprüchliche Anschrift löst eine Rückfrage ohne Default aus (kein Textbaustein möglich, Archivierung nach 5 Werktagen ohne Antwort); eine unsichere Ansprechperson/Anrede löst ebenfalls eine Rückfrage aus, aber mit Default, der bei Nichtantwort automatisch zu Beginn des nächsten Tageslaufs eingetragen und in der Review-Checkliste als „per Default beantwortet – bitte prüfen" markiert wird (Kapitel 10.4, 14.4).

**16. Abweichender Firmenstandort.** Wenn Stellenanzeige und Impressum/Hauptsitz voneinander abweichen: automatisch den Anzeige-Standort übernehmen, oder immer nachfragen? **Default-Annahme:** Anzeige-Standort übernehmen, bei echter Unsicherheit Rückfrage (company_research.json).

### 22.4 Sprache & Ton

**17. [Blocker] Sprache der Bewerbungen.** Deutsch, Englisch, oder situativ je nach Sprache der Stellenanzeige und Unternehmenssprache? **Default-Annahme:** Deutsch; Englisch nur bei eindeutig englischsprachiger Anzeige oder Konzernsprache Englisch (Kapitel 4.9, 11.11).

**18. Sie oder Du als Grundpräferenz.** Wenn die Stellenanzeige keine klare Präferenz zeigt: generell Sie oder generell Du? **Default-Annahme:** Sie (Kapitel 5.12, 11.12).

**19. Motivationsschreiben als Standard.** Soll zusätzlich zum Anschreiben standardmäßig ein Motivationsschreiben entstehen, auch wenn nicht explizit verlangt? **Default-Annahme:** nein, nur auf ausdrückliche Anforderung der Stellenanzeige (Kapitel 8.11, 11.12).

**20. Standardantwort bei Nachfrage zur KI-Nutzung.** Soll eine vorformulierte, ehrliche Antwort bereitliegen, falls ein Arbeitgeber direkt danach fragt? **Default-Annahme:** nein, du entscheidest situativ im Gespräch (Kapitel 16.6).

**21. Grauzonen beim Lebenslauf-Tailoring.** Sollen Titel-Alias, der Wortlaut für Lücken und Synonym-Mapping einmalig im Onboarding pauschal festgelegt werden, oder soll jede Grauzone einzeln zur Freigabe vorgelegt werden? **Default-Annahme:** einmalig im Onboarding festlegen, danach keine Einzelrückfragen mehr zu diesen Fällen (Kapitel 11.12).

### 22.5 Versand & Konten

**22. [Blocker] Primäres E-Mail-Konto.** Bestehendes iCloud-Konto, bestehendes Gmail-Konto, oder eine neue, eigene Domain? Zugangsdaten werden je nach Betriebsumgebung in macOS-Keychain oder 1Password CLI abgelegt, nie im Klartext ([1Password CLI](https://developer.1password.com/docs/cli/secrets-scripts), [macOS Keychain](https://ss64.com/mac/security-password.html)). **Default-Annahme:** technologieoffen implementiert; iCloud als einfachster Startpunkt, da bereits vorhanden (Kapitel 15.2).

**23. Zwei-Faktor-Authentifizierung auf der Apple-ID.** Ist sie bereits aktiviert (Voraussetzung für das App-spezifische Passwort, [Apple](https://support.apple.com/en-us/102198))? **Default-Annahme:** wird in Phase 0 geprüft, Aktivierung ist Voraussetzung für P0-05.

**24. Entwurf-Modus oder Auto-Versand ab Start.** Soll der MVP als reiner Entwurf-Modus starten (du klickst selbst auf Senden), oder ist von Anfang an automatisierter Versand nach täglicher Freigabe gewünscht? **Default-Annahme:** Entwurf-Modus (Kapitel 15.1, 15.8).

**25. Gmail-OAuth-Reautorisierung.** Ist eine wöchentliche Browser-Reautorisierung im Testing-Modus akzeptabel, oder soll eine vollständige Google-Verifizierung beantragt werden ([Gmail-Scopes](https://developers.google.com/workspace/gmail/api/auth/scopes), [OAuth-Testing-Modus](https://support.google.com/cloud/answer/15549945?hl=en))? **Default-Annahme:** Testing-Modus akzeptieren, nur relevant falls Gmail gewählt wird (Kapitel 15.8).

**26. LinkedIn/XING Easy-Apply.** Soll das automatisiert werden, auch nur als Co-Pilot (Playwright füllt aus, du klickst „Absenden“), oder grundsätzlich manuell bleiben? **Default-Annahme:** nur Co-Pilot, sehr geringe Frequenz (Kapitel 15.5, portals.json).

**27. Bewerberkonten bei SAP SuccessFactors/Workday.** Sollen sie automatisch pro Arbeitgeber angelegt werden, oder erfolgt die Kontoerstellung immer manuell? **Default-Annahme:** manuell in v1 (Kapitel 15.5, portals.json).

**28. Nachfass-Intervall.** Nach wie vielen Werktagen ohne Rückmeldung soll ein Nachfassen vorgeschlagen werden ([karrierebibel.de](https://karrierebibel.de/nachfassen-bewerbung/))? **Default-Annahme:** 10 Werktage, branchenunabhängig (Kapitel 2.13, 15.7).

**29. Digitale Signaturvorlage.** Hast du einen Scan deiner Unterschrift, der bei Bedarf eingefügt werden soll? **Default-Annahme:** keine, Unterschrift bleibt aus (Kapitel 5.12, 13.12).

### 22.6 Technik & Budget

**30. [Blocker] Commercial-API-Key oder persönliches Pro/Max-Abo.** Läuft der Automatisierungs-Server über einen separaten Anthropic-API-Key (planbare, nutzungsabhängige Abrechnung, eigener AVV) oder über dein persönliches Abo (Konkurrenz mit deiner eigenen interaktiven Nutzung, Consumer Terms schließen Drittwerkzeuge wie das Agent SDK aus)? Falls du bereits eine Anthropic-Organisation/einen Workspace hast, kann der Key dort erzeugt werden. **Default-Annahme:** Commercial-API-Key (Kapitel 7.11, 16.2, 16.8).

**31. [Blocker] Hosting.** Eigener Hetzner-VPS in Deutschland, dein Mac lokal, oder vollständig Anthropic-verwaltete Managed Agents (Beta/Research Preview)? **Default-Annahme:** Hetzner-VPS in Deutschland (Kapitel 7.7.3, 7.11).

**32. [Blocker] Modellwahl für die qualitätskritischen Schritte.** Claude Fable 5.1 (deine Wunschbasis für Rechercheur und Autor, aber 30-Tage-Datenspeicherung bei Anthropic für Covered Models, [API and data retention](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention)) oder durchgängig Claude Opus 5 (ZDR-fähig und mit $5/$25 pro 1 Mio. Token halb so teuer wie Fable 5.1 mit $10/$50)? Der Kritiker läuft unabhängig von dieser Wahl immer als eigenständiges, vom Autor getrenntes Grader-Modell auf Opus 5 – kein zusätzlicher Zweitgutachter obendrauf. **Default-Annahme:** Fable 5.1 mit Offenlegung und Zustimmung im Onboarding für Rechercheur und Autor; der Konfigurationsschalter `MODEL_TOP` schaltet bei Bedarf alle Fable-5.1-Schritte auf Opus 5 um (dann ohne 30-Tage-Speicherung); der Kritiker bleibt davon unberührt und läuft in jedem Fall auf Opus 5 (Kapitel 7.6, 7.11, 11.12, 16.2).

**33. [Blocker] Kostenlimit.** Wie hoch darf das tägliche bzw. monatliche Budget für Modell-Tokens, Websuche und Session-Laufzeit sein? **Default-Annahme:** 45 USD je Tageslauf (abgeleitet aus Kapitel 18.2: rund 37 USD für zehn Bewerbungen im empfohlenen Szenario, plus Puffer); eine feste Monatsobergrenze ist noch offen und sollte spätestens bei P0-02 (Ausgabenlimit in der Anthropic Console) konkret gesetzt werden – als Orientierung nennt Kapitel 18 für 220 Bewerbungen im Monat (inkl. Massen-Scan 9 EUR und Server 20 EUR) rund 262 EUR im sparsamen, rund 789 EUR im empfohlenen und rund 1.151 EUR im maximalen Szenario (Kapitel 7.11, 18).

**34. EU-Hosting der Bewerbungsdaten.** Sollen Kandidatenprofil und Bewerbungshistorie ausschließlich in der EU liegen? **Default-Annahme:** ja, konsistent mit der Hetzner-VPS-Wahl (Kapitel 7.7.3).

**35. Wochenend-Läufe.** Soll der Quellenabruf auch am Wochenende laufen, oder nur Montag bis Freitag? **Default-Annahme:** Quellenabruf täglich, Tagesauswahl und Schreibschleife nur Mo–Fr (Kapitel 2.13, 7.11).

**36. Länge des Undo-Fensters.** Wie viele Sekunden Verzögerung nach „Freigeben“ sind akzeptabel, bevor tatsächlich gesendet wird? **Default-Annahme:** 60 Sekunden (Kapitel 2.13, 7.11, 14.13).

**37. Benachrichtigungskanal.** Telegram-Push plus täglicher E-Mail-Digest, oder reicht der Digest allein? **Default-Annahme:** beides (Kapitel 2.13, 7.11).

**38. Optionale kostenpflichtige Zusatztools.** Sollen Cohere/Voyage-Embeddings, ein laufendes Northdata-Abo oder Vergleichstools wie Eden AI/Jobscan im vereinbarten Budget genutzt werden, oder ausschließlich kostenlose/selbstgehostete Alternativen (BGE-M3, Apache Tika, OpenResume)? **Default-Annahme:** strikt kostenlos/selbstgehostet als Ausgangspunkt, Cohere Rerank 3.5 als einzige bezahlte Ausnahme wegen vernachlässigbarer Kosten; alles andere nur mit ausdrücklicher Freigabe (Kapitel 4.9, 9.10, 12.11, company_research.json).

### 22.7 Recht & Grenzen

**39. Spätere Kommerzialisierung oder Mehrnutzer-Betrieb.** Ist das auch nur informell angedacht (z. B. Mitnutzung durch Familie/Freunde)? Das beendet die DSGVO-Haushaltsausnahme sofort und löst volle Verantwortlicheneigenschaft aus ([dr-datenschutz.de](https://www.dr-datenschutz.de/die-haushaltsausnahme-der-dsgvo/)). **Default-Annahme:** bleibt eine Option (Kapitel 21), aber MVP und v1 sind strikt Einzelnutzer-Systeme; nichts wird architektonisch verbaut, was eine spätere Öffnung unmöglich machen würde.

**40. AGG-sensible Angaben.** Sollen Foto, Geburtsdatum und Familienstand grundsätzlich weggelassen werden, oder branchenabhängig flexibel gehandhabt werden (in manchen konservativen Branchen wird ein Foto weiterhin implizit erwartet, [Haufe](https://www.haufe.de/id/beitrag/agg-die-merkmale-rasse-und-ethnische-herkunft-23-bewerbungsfoto-HI16209081.html))? **Default-Annahme:** grundsätzlich weglassen, nur auf explizite Nutzeranfrage mit Warnhinweis einfügen (Kapitel 5.3, 13.12, 16.4).

**41. Toleranz für rechtliches Restrisiko bei Datenzugriff.** Sollen StepStone, Indeed, LinkedIn, XING und Monster ausschließlich manuell im Browser bedient werden, oder ist bei sehr geringem Automatisierungsvolumen ein bewusstes Restrisiko (Account-Sperre, Unterlassungsschreiben, [§ 11 AÜG](https://www.gesetze-im-internet.de/a_g/__11.html) als Kontext für Vermittler-Erkennung) akzeptabel? **Default-Annahme:** ausschließlich ToS-unbedenkliche Quellen (BA-API, ATS-Feeds, Adzuna, Arbeitnow, SerpAPI); kein automatisiertes Scraping der großen Publikumsbörsen (Kapitel 6, 16.1).

**42. Löschfrist für Ansprechpartner-Kontaktdaten.** Wie lange dürfen recherchierte Namen und Kontaktdaten von Ansprechpartnern gespeichert bleiben? **Default-Annahme:** 12 Monate nach letzter Aktualisierung, bzw. sofortige Löschung nach Status „archiviert“ ohne geplante erneute Bewerbung (Kapitel 10.7).

### 22.8 Produktname

**43. Eigener Produktname.** Soll das System einen eigenen Namen bekommen, oder bleibt „der Bewerbungsagent“ auch im täglichen Gebrauch der Arbeitstitel? **Default-Annahme:** Arbeitstitel bleibt; eine Umbenennung ist jederzeit möglich, ohne die Architektur zu ändern (Kapitel 3, Styleguide Abschnitt 3).

**44. Markenrücksicht bei der Namenswahl.** Falls ein Name gewünscht ist: Soll er schon jetzt auf eine mögliche spätere Kommerzialisierung Rücksicht nehmen (Marken-/Domainverfügbarkeit prüfen), oder reicht ein rein privater Arbeitsname? **Default-Annahme:** rein privater Arbeitsname; eine Marken-/Domainprüfung erfolgt erst, falls Kapitel 21 tatsächlich aktiviert wird.

**45. Sichtbarkeit des Namens gegenüber Dritten.** Soll der Name in E-Mail-Signaturen, im Telegram-Bot-Anzeigenamen oder sonst gegenüber Empfängern auftauchen, oder komplett intern bleiben? **Default-Annahme:** intern bleiben; gegenüber Arbeitgebern tritt ausschließlich der Kandidat auf, nicht das Werkzeug (konsistent mit Kapitel 15, 16).

### 22.9 Roadmap & Tempo

Diese Gruppe bündelt die offenen Fragen aus der Roadmap (Kapitel 19.10), die kein anderes Kapitel als „offene Frage an dich" führt: Tempo, Umfang und Reihenfolge der ersten Monate. Keine davon ist ein Blocker vor Phase 0.

**46. Startzeitpunkt und Wochenkapazität für Phase 0.** Wann startet Phase 0 tatsächlich, und wie viel Zeit kannst du dir verbindlich pro Woche dafür nehmen? **Default-Annahme:** Start noch in dieser Woche; rund 3 Personentage (PT) je Woche im MVP, danach 1 bis 2 PT je Woche plus die tägliche Review-Routine aus Kapitel 2.13 (Kapitel 19.9, 19.10).

**47. Englischsprachige Zielrollen bereits im MVP.** Sind englischsprachige Zielrollen relevant genug, dass die Vorlage `international-en` (Kapitel 13.3) schon im MVP gebraucht wird? **Default-Annahme:** nein; `international-en` und `klassisch` folgen erst in v1, nach dem Test-Parsing der MVP-Vorlage `sachlich` (Kapitel 13.3, 19.5, 19.10).

**48. Versandstart in Woche 5.** Ist der Start des echten Versands in Woche 5 mit Tageslimit 3 und Budget 5 USD akzeptabel, oder soll der Entwurf-Modus länger laufen? **Default-Annahme:** Woche 5; danach gelten die Limits aus Kapitel 7.11 (Kapitel 15.1, 19.4, 19.10).

**49. Zuschnitt des Onboarding-Interviews.** Soll das Onboarding-Interview in zwei Sitzungen zu je 2 bis 3 Stunden laufen (P0-14 bis P0-16), oder lieber über mehrere kürzere Termine verteilt über die Woche? **Default-Annahme:** zwei Sitzungen (Kapitel 8, 19.9, 19.10).

**50. Watchlist-Zusage in Phase 0.** Kannst du die Watchlist mit 20 bis 50 Wunscharbeitgebern (Frage 9) bereits in Phase 0 liefern (P0-13), oder erst im laufenden Betrieb nachreichen? **Default-Annahme:** ja, in Phase 0; sonst startet der MVP zunächst nur mit BA-API, Adzuna, Arbeitnow und Alert-Mails (Kapitel 6.9, 19.9, 19.10).

**51. Managed-Agents-Test durchführen.** Soll der optionale Managed-Agents-Test für den Rechercheur (V-17) in v1 überhaupt stattfinden? **Default-Annahme:** optional, nur wenn Zeit bleibt; das Ergebnis fließt erst in eine spätere v2-Entscheidung ein (Kapitel 7.10, 19.5, 19.10).

**52. Reihenfolge der Portalfamilien im Co-Pilot.** Welche Portalfamilien sollen im Portal-Co-Pilot (W-02, v2) zuerst unterstützt werden? **Default-Annahme:** Personio, softgarden, JOIN zuerst; SAP SuccessFactors und Workday danach; Plattform-Schnellbewerbungen nur nach ausdrücklicher Entscheidung (Kapitel 15.5, 19.6, 19.10).

**53. Anwaltliche Prüfung vor v2 budgetieren.** Soll die anwaltliche Prüfung (W-12) schon vor v2 budgetiert werden, oder erst bei einer Go-Entscheidung? **Default-Annahme:** erst bei einer Go-Entscheidung nach Kapitel 21.5 (Kapitel 19.6, 19.10).

### 22.10 Wie mit dieser Liste weiterarbeiten

Kapitel 19.3 (Arbeitspaket P0-01) verlangt, die acht Blocker aus 22.1 vor Beginn von Phase 0 zu beantworten; für alle übrigen 45 Fragen gilt bis zu deiner Antwort die genannte Default-Annahme unverändert. Antworten trägst du am einfachsten direkt in `config/` bzw. in die entsprechenden Profildateien aus Kapitel 8 ein, sobald das Onboarding läuft – eine separate Antwortdatei ist nicht nötig, da jede Frage bereits auf die Stelle verweist, an der die Antwort technisch wirksam wird.

**Quellen dieses Kapitels:**

- Anabin-Kurzanleitung (KMK) – https://anabin.kmk.org/kurzanleitung/ich-moechte-feststellen-wie-mein-auslaendischer-hochschulabschluss-in-deutschland-bewertet-wird.html
- mentorium.de: Zeugnisse beglaubigt übersetzen – https://www.mentorium.de/zeugnisse-beglaubigt-uebersetzen/
- SerpAPI Pricing – https://serpapi.com/pricing
- LinkedIn: Verbotene Software und Erweiterungen – https://www.linkedin.com/help/linkedin/answer/a1341387/verbotene-software-und-erweiterungen?lang=de-DE
- Apple: iCloud Mail Limits (Support 102198) – https://support.apple.com/en-us/102198
- 1Password CLI: Secrets in scripts – https://developer.1password.com/docs/cli/secrets-scripts
- macOS security CLI – https://ss64.com/mac/security-password.html
- Gmail API: Scopes – https://developers.google.com/workspace/gmail/api/auth/scopes
- Google Cloud Support: OAuth-Testing-Modus – https://support.google.com/cloud/answer/15549945?hl=en
- karrierebibel.de: Nachfassen nach der Bewerbung – https://karrierebibel.de/nachfassen-bewerbung/
- Anthropic: API and data retention (Covered Models) – https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
- Haufe: AGG und Bewerbungsfoto – https://www.haufe.de/id/beitrag/agg-die-merkmale-rasse-und-ethnische-herkunft-23-bewerbungsfoto-HI16209081.html
- § 11 AÜG (gesetze-im-internet.de) – https://www.gesetze-im-internet.de/a_g/__11.html
- dr-datenschutz.de: Die Haushaltsausnahme der DSGVO – https://www.dr-datenschutz.de/die-haushaltsausnahme-der-dsgvo/
