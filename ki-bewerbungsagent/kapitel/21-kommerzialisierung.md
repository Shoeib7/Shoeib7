## 21. Später: Kommerzialisierung (kurz)

Dieses Kapitel ist ein Vorausblick, keine Entscheidung. Der Leitsatz aus Kapitel 1 gilt unverändert: Das System muss zuerst für eine Person zuverlässig funktionieren, bevor eine Öffnung für andere Nutzer überhaupt sinnvoll geprüft werden kann. Die Marktanalyse in Kapitel 3 zeigt eine echte Lücke (Pflicht-Freigabe, belegte Recherche, Versand über das eigene Postfach), aber keines der dortigen Preis- oder Bewertungssignale ersetzt eigene Erfahrung aus dem Eigenbetrieb. **Default-Annahme dieses Kapitels:** Kommerzialisierung bleibt eine Option, wird aber erst nach stabilem Betrieb in Phase v1/v2 (Kapitel 19) geprüft – bis dahin wird nichts architektonisch verbaut, was eine spätere Öffnung unmöglich machen würde (siehe 21.3).

### 21.1 Zielgruppen und Kanäle

| Zielgruppe | Kanal | Charakter |
|---|---|---|
| Berufstätige im Jobwechsel | B2C-Selfservice (Abo/Pay-per-Bewerbung) | Direktester Kanal, entspricht dem Ursprungsfall |
| Berufseinsteiger, Studierende | Career Services (Hochschul-Karrierezentren) | B2B2C, Seat-Lizenzen an Institutionen |
| Arbeitsuchende/Arbeitslose | AVGS über Jobcenter/Arbeitsagentur | B2B2C, setzt AZAV-Zulassung voraus (siehe 21.4) |
| Bewerbungscoaches, Karriereberater | White-Label-Lizenz pro Coach | B2B2C, Coach bleibt Ansprechpartner seiner Klienten |

Der AVGS-Kanal (Aktivierungs- und Vermittlungsgutschein) wird in der Recherche nur als allgemein bekannter, nicht live verifizierter Hintergrund benannt: Zulassung erfordert eine AZAV-Zertifizierung durch eine fachkundige Stelle – Kosten, Dauer und genaue Voraussetzungen sind **unbestätigt** und nicht Teil dieser Recherche. Für ein Einzelentwickler-Projekt ist dieser Aufwand vor einer ernsthaften B2B-Absicht klar unverhältnismäßig.

**Entscheidung:** Von den vier Kanälen zuerst B2C-Selfservice und Coach-Lizenzierung prüfen, AVGS/Career-Services zurückstellen. **Begründung:** Beide ersten Kanäle benötigen keine Drittzulassung und lassen sich mit demselben Produkt bedienen, das für den Eigenbetrieb entsteht. **Alternative:** Direkt auf AVGS zielen, weil öffentlich finanziert – verworfen wegen der unklaren AZAV-Hürde und weil sie den MVP-Zeitplan (Kapitel 19) sprengen würde.

### 21.2 Preismodelle

Der deutsche Markt spannt laut Kapitel 3 einen Preisrahmen zwischen Billig-Generator und Premium-Hybrid auf:

| Anker | Preis | Modell |
|---|---|---|
| erfolgo.de | 9,95 € Flatrate | reiner KI-Generator, dokumentierte Qualitätsmängel |
| Jobscan | 29,98–49,95 USD/Monat | Abo, reine Analyse ohne Versand |
| LazyApply | 99–999 USD/Jahr | Volumen-Abo, Trustpilot 2,1/5 |
| Bewerbung-Schreiber.com | 99–199 € pro Anschreiben | Mensch+KI-Hybrid, individuell |

**Entscheidung:** Falls kommerzialisiert wird, Positionierung als Abo mit begrenztem Kontingent hochwertiger Bewerbungen pro Monat (nicht Pay-per-Volume), preislich näher am Hybrid-Anker als am Billig-Generator. **Begründung:** Ein Preismodell nach Bewerbungsvolumen (wie LazyApply) setzt einen Anreiz zu mehr statt besseren Bewerbungen und widerspricht dem Leitsatz „Qualität statt Masse". **Alternative:** Reines Pay-per-Bewerbung ohne Abo – bleibt als Einstiegsoption für Coaches/Career Services denkbar, da dort Volumen planbarer ist als bei Einzelpersonen.

### 21.3 Was sich technisch ändern muss

- **Mandantenfähigkeit:** Kandidatenprofil (Kapitel 8), Story-Bank und Stimmprofil müssen von einer festen Datei auf isolierte Datensätze pro Nutzer umgestellt werden; Orchestrator (Kapitel 7) braucht Mandanten-Kontingente statt eines einzelnen Tageslaufs.
- **DSGVO-Verantwortlichkeit:** Die Haushaltsausnahme (Kapitel 16.2) entfällt vollständig, sobald ein Dritter das System nutzt. Der Betreiber wird Verantwortlicher i. S. d. DSGVO: Rechtsgrundlage pro Kunde, Verzeichnis von Verarbeitungstätigkeiten, Löschkonzept, Prozess für Betroffenenrechte (Art. 15, 17 DSGVO).
- **AVV mit Anthropic:** Der für den Eigenbetrieb bereits empfohlene Commercial-API-Key mit AVV (Kapitel 16.2) wird zur zwingenden Voraussetzung, nicht mehr nur Empfehlung; zusätzlich muss der Betreiber Anthropic als Unterauftragsverarbeiter in der eigenen Datenschutzerklärung offenlegen. Die Retention-Logik aus Kapitel 16.2 (Claude Fable 5.1 als Covered Model mit 30-Tage-Speicherung, Claude Opus 5 als ZDR-fähige Alternative) muss dann pro Kunde einzeln zugestimmt oder standardmäßig auf ein ZDR-fähiges Modell umgestellt werden.
- **Hosting:** Der für einen Nutzer ausreichende Hetzner-Cron-Server (Kapitel 18) muss durch Mandanten-Isolation, Auth, Kontingent-/Kostenlimits pro Kunde und Abrechnungsanbindung ersetzt werden.

### 21.4 Rechtliche Voraussetzungen

Vor jeder Öffnung für zahlende Kunden: Gewerbeanmeldung, Impressumspflicht (§ 5 DDG), AGB und Widerrufsbelehrung für Verbraucherverträge, eine belastbare anwaltliche Prüfung der in Kapitel 16.10 offengelassenen Punkte (§ 7 UWG bei Akquise-Mails, aktuelle Anthropic-Consumer-Terms-Durchsetzung) sowie – nur bei AVGS-Ambition – eine gesonderte AZAV-Kostenrecherche. Diese Prüfungen sind für den in Kapitel 19 geplanten Einzelnutzer-Betrieb nicht erforderlich, aber Voraussetzung für jeden Schritt über Kapitel 21 hinaus.

### 21.5 Go/No-Go-Kriterien nach der Eigen-Nutzung

**Go**, wenn nach mehreren Monaten Eigenbetrieb (Kapitel 19) alle folgenden Punkte zutreffen: (1) reale Token-/Kostenwerte aus `response.usage` liegen vor und bestätigen die Schätzung aus Kapitel 18, (2) keine rechtlichen Zwischenfälle (Kontosperrung, Abmahnung), (3) die Qualitätsschleife (Kapitel 11) liefert nachweisbar bessere Rückmeldequoten als generische Vergleichswerte, (4) der Nutzer hat Zeit und Interesse, Support/Compliance für fremde Nutzer zu tragen.

**No-Go**, wenn: der Eigenbetrieb bereits die volle Aufmerksamkeit bindet, die Stimmprofil-Individualisierung (Kapitel 8) sich nicht ohne massiven manuellen Aufwand auf fremde Nutzer übertragen lässt, oder die DSGVO-Verantwortlichkeit (21.3) als Aufwand den erwarteten Nutzen übersteigt.

**Quellen dieses Kapitels:**
- LazyApply (Preismodell) – https://www.loopcv.pro/directory/lazyapply/
- Jobscan (Preismodell) – https://www.jobscan.co/
- Bewerbung-Schreiber.com – https://bewerbung-schreiber.com/
- erfolgo.de – https://erfolgo.de/
- erfolgo.de Trustpilot – https://ch.trustpilot.com/review/erfolgo.de
- Anthropic API and Data Retention – https://platform.claude.com/docs/en/manage-claude/api-and-data-retention
