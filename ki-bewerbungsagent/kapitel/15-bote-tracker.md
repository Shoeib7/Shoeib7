## 15. Modul Bote und Tracker: E-Mail-Anbindung, Sendeprotokoll, Portale, Antworten, Nachfassen

Der **Bote** ist die einzige Komponente des Systems mit direkter Außenwirkung: Er legt E-Mail-Entwürfe an bzw. versendet sie nach Freigabe, und er füllt Portal-Formulare vor. Der **Tracker** übernimmt danach die Nachverfolgung: Statuspipeline, Klassifikation eingehender Antworten, Nachfassen, Kalender, Statistik. Beide Rollen laufen auf der SQLite-Datenbank und dem Freigabe-Mechanismus, die in Kapitel 7 als Architekturentscheidung festgelegt sind (kein `bypassPermissions`, ein eigenes `canUseTool`-Gate blockiert jeden `send_email`-Aufruf bis zur expliziten Nutzerfreigabe). Dieses Kapitel beschreibt, wie Bote und Tracker diese Vorgabe konkret umsetzen. Die Freigabe-Oberfläche selbst (Checklisten, Diff, Buttons) ist Kapitel 14; die rechtlichen Grenzen (was der Bote nie automatisch tun darf) sind in Kapitel 16 gesammelt.

### 15.1 E-Mail-Strategie: MVP „Entwurf im eigenen Postfach“, v1 „Versand nach Freigabe“

**Entscheidung:** Der MVP versendet keine E-Mail automatisch. Der Bote erzeugt eine fertig formatierte E-Mail und legt sie als Entwurf im echten Postfach des Nutzers ab – bei IMAP-Konten (z. B. iCloud) per `IMAP APPEND` in den Ordner „Entwürfe“, bei Gmail per Drafts-API (`users.drafts.create`). Der Nutzer öffnet seine gewohnte Mail-App und klickt selbst auf Senden. Erst in v1, nach Vertrauensaufbau, kommt ein echter Versand-Schritt hinzu: Der Bote sendet per SMTP (`aiosmtplib`), aber nur nachdem im Review-Cockpit (Kapitel 14) je Bewerbung oder je Tages-Batch explizit „freigegeben“ gesetzt wurde – das `canUseTool`-Gate aus Kapitel 7 bleibt auch dann bestehen, es öffnet sich nur für die konkret freigegebenen Datensätze.

**Begründung:** Der Entwurf-Modus erfüllt zwei Anforderungen gleichzeitig, ohne dass zusätzliche Versandlogik gebaut werden muss: Es gibt keinen Bot-artigen automatisierten Versand-Endpunkt, und die Freigabe durch den Menschen ist baulich erzwungen, weil der letzte Klick beim Nutzer liegt [Empfehlung](https://developers.google.com/workspace/gmail/api/guides/drafts). Zusätzlich vermeidet er jedes Risiko, dass ein Fehler im Rechercheur oder Autor (z. B. falsche Firmenadresse) ungeprüft versendet wird.

**Alternative(n):** Direkter Versand ab Tag 1 mit einem separaten Freigabe-Klick pro Bewerbung – technisch möglich, aber unnötiges Risiko, solange die anderen Module (Kapitel 9–13) noch nicht im Alltag erprobt sind.

### 15.2 Konto-Optionen und Empfehlung

| Weg | Zugriff | Limit/Tag | Aufwand | Eignung |
|---|---|---|---|---|
| iCloud (SMTP/IMAP) | App-spezifisches Passwort (2FA Pflicht), `smtp.mail.me.com:587` (STARTTLS), `imap.mail.me.com:993` (SSL/TLS) | 1.000 Nachrichten, max. 500 Empfänger/Nachricht [Apple](https://support.apple.com/en-us/102198) | niedrig – sofort mit bestehendem Konto | MVP/v1, wenn Nutzer iCloud verwendet |
| Gmail API (Drafts) | OAuth 2.0, Scope `gmail.compose` oder `gmail.modify` (beide „Restricted“, nicht „Sensitive“) | 500 E-Mails/Tag privat [Google Workspace](https://developers.google.com/workspace/gmail/api/reference/quota) | mittel – Google-Cloud-Projekt, Testing-Modus mit 100 Nutzern ohne Verifizierung, aber Refresh-Token läuft alle 7 Tage ab [Google](https://support.google.com/cloud/answer/15549945?hl=en) | MVP, wenn Nutzer Gmail verwendet |
| Microsoft Graph (Mail.Send) | OAuth 2.0, eigene Azure-App-Registrierung | Exchange-Throttling pro App+Mailbox [Microsoft](https://learn.microsoft.com/en-us/graph/throttling-limits) | hoch – Azure-Setup nötig | nur falls Outlook/Microsoft 365 bereits genutzt |
| Eigene Domain (Fastmail, mailbox.org, iCloud+ Custom Domain) | SMTP/IMAP wie das Basiskonto, eigener Domainname | wie Basiskonto | niedrig-mittel, DNS-Setup nötig | v1-Upgrade für Seriosität, MVP nicht nötig |

Für iCloud liefert Apple keine Sende-API, nur klassisches SMTP/IMAP [Mailmeteor](https://mailmeteor.com/smtp/icloud-smtp-settings). Für Gmail ist die technisch korrekte Scope-Einordnung wichtig: `gmail.send` allein ist „Sensitive“, erlaubt aber nur Senden, keine Entwürfe; der empfohlene Draft-Weg braucht `gmail.compose` oder `gmail.modify`, beide „Restricted“ [Google](https://developers.google.com/workspace/gmail/api/auth/scopes). Im OAuth-Testing-Modus (Einzelnutzer als Testperson) ist dafür keine Google-Verifizierung nötig – das gilt nur bei einer späteren Veröffentlichung „In Production“.

**Entscheidung:** Welches Konto primär genutzt wird, ist eine Nutzerentscheidung und wird nicht vorweggenommen (siehe Offene Fragen). Die Implementierung unterstützt beide Kernwege (IMAP-Append für beliebige IMAP-Konten inkl. iCloud, Drafts-API speziell für Gmail) über eine gemeinsame Bote-Schnittstelle, sodass der Wechsel später keine Architekturänderung erfordert.

**Begründung:** Beide Wege sind für ca. 10 Bewerbungen/Tag technisch beliebig ausreichend dimensioniert; die Entscheidung hängt vom bereits vorhandenen Konto des Nutzers ab, nicht von technischen Limits.

**Alternative(n):** Eine neue, eigene Bewerbungsdomain (z. B. `bewerbung@vorname-nachname.de`) über Fastmail (Individual-Plan, laut Anbieter ca. 6 $/Monat, bis 100 Domains) oder mailbox.org (Standard-Plan, laut Anbieter ca. 3 €/Monat) [Fastmail](https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates) [mailbox.org](https://mailbox.org/en/news/new-price-plans-available-mailboxorg/) – seriöser wirkend, aber ein v1-Upgrade, kein MVP-Blocker, weil SPF/DKIM/DMARC beim bestehenden Konto bereits serverseitig korrekt gesetzt sind [Woodpecker](https://woodpecker.co/blog/spf-dkim/).

Von Managed-E-Mail-Konnektoren wie Composio, Pipedream oder Zapier MCP wird für den Versand abgeraten: Sie verlangen vollen OAuth-Zugriff auf das Postfach über einen zusätzlichen Drittanbieter – für einen Einzelnutzer mit einem Konto ist der direkte Weg (Gmail-API/SMTP) schlanker und ohne zusätzlichen Datenfluss.

### 15.3 Sendeprotokoll (v1)

Sobald v1 echten Versand erlaubt, protokolliert der Bote jeden Versandversuch unveränderlich. Feste Regeln, unabhängig vom gewählten Konto:

- **Zeitfenster mit Jitter:** Kernfenster Dienstag–Donnerstag, ca. 7:00–9:30 Uhr (optional zusätzlich 14:00–16:00 Uhr), mit zufälligem Versatz von ±15–40 Minuten je E-Mail. Das ist HR-Ratgeber-Konsens, keine belastbare Studie – als Heuristik gegen ein erkennbares Cron-Muster nutzen, nicht als Fakt zitieren [arwa.de](https://arwa.de/de/blog/wann-sollte-man-eine-bewerbung-abschicken).
- **Tageslimit:** 10 E-Mails/Tag (Default aus `config/zeitplan.yaml`, konfigurierbar, hart im Boten geprüft; Kapitel 7.5), unabhängig von den Provider-Limits selbst überwacht (weit unter jeder Missbrauchsschwelle bei iCloud oder Gmail). Zum Start des echten Versands in v1 (Woche 5) gilt zunächst ein reduziertes Tageslimit von 3, danach greift der Default von 10 (Kapitel 19.5).
- **Format:** Plaintext oder schlichtes HTML, kein Tracking-Pixel, kein Link-Tracking, keine Lesebestätigung – Tracking-Pixel senken nachweislich die Zustellbarkeit und wirken unseriös [Instantly](https://instantly.ai/blog/email-tracking-and-deliverability-why-tracking-pixels-can-hurt-your-inbox-placement/).
- **Betreffkonvention:** `Bewerbung als [Position] – [Referenznummer, falls vorhanden]`. Kein Marketing-Ton, keine Emojis.
- **Signatur:** Name, Postanschrift, Telefon, ggf. LinkedIn/Portfolio-Link – Foto, Geburtsdatum und Familienstand gehören nicht in die Signatur (AGG-Risiko, Details Kapitel 16).
- **Anhang:** genau eine kombinierte PDF-Datei (Anschreiben, Lebenslauf, ggf. Zeugnisse), erzeugt vom Setzer (Kapitel 13); Zielgröße 1–3 MB, hartes Limit 5 MB.

Der Bote führt dafür keine eigene Tabelle: Eine dritte Ablage neben `application` und `event_log` (Kapitel 7.3) würde nur ein zusätzliches, konkurrierendes Statusvokabular schaffen. Stattdessen schreibt er Versandtatsachen in die dort bereits vorhandenen Spalten der Tabelle `application` (`sent_at`, `sent_to`, `message_id`, `sent_account`, `send_window_start`) und protokolliert jeden einzelnen Versuch – auch Entwürfe und Fehlschläge – als eigenen, unveränderlichen Eintrag im `event_log`:

```json
{
  "actor": "bote",
  "action": "draft | send | error",
  "entity_type": "application",
  "payload": {
    "empfaenger_domain": "beispielfirma.de",
    "betreff_hash": "sha256-hex …",
    "anhang_groesse_kb": 1820,
    "fehlermeldung": null
  }
}
```

`empfaenger_domain` (nur die Domain, keine volle Adresse), `betreff_hash` (SHA-256 statt Klartext) und `anhang_groesse_kb` brauchen keine eigenen Tabellenspalten, weil `event_log.payload` laut Kapitel 7.3 bereits ein geprüftes JSON-Feld ist. Diese Kombination aus `application`-Spalten und `event_log`-Einträgen ist die Grundlage für das Tageslimit, für Statistiken (15.7) und für den Audit-Nachweis „wann wurde was an wen mit welchem Status verschickt“.

### 15.4 Sicherheit der Zugangsdaten

App-spezifische Passwörter und OAuth-Tokens gehören nie in eine Klartext-`.env`-Datei oder ins Git-Repo. **Entscheidung:** Auf dem VPS ist sops + age der verbindliche Weg (Kapitel 7.7.6): Die Zugangsdaten des Boten liegen verschlüsselt in `config/secrets.enc.yaml` und werden zur Laufzeit per `sops exec-env` als Umgebungsvariablen in den Prozess injiziert, nie in eine Klartextdatei und nie in den Modellkontext geschrieben. macOS-Keychain oder 1Password CLI kommen für den Boten selbst nicht zum Einsatz; sie sind ausschließlich für den lokal auf dem Rechner des Nutzers laufenden Portal-Co-Piloten in v2 vorgesehen (Kapitel 15.5), sobald dort pro Arbeitgeber getrennte Zugangsdaten außerhalb des VPS verwaltet werden müssen.

Zwei betriebliche Besonderheiten muss der Bote aktiv behandeln, statt sie zu ignorieren:

1. Apple widerruft App-spezifische Passwörter automatisch und ohne Vorwarnung bei jeder Änderung des Apple-ID-Passworts. Der Bote muss einen Auth-Fehler (SMTP/IMAP 535) erkennen und den Nutzer aktiv zur Neu-Generierung auffordern – kein stiller Dauer-Retry.
2. Im Gmail-OAuth-Testing-Modus läuft der Refresh-Token nach 7 Tagen ab; ohne erneuten Browser-Consent bricht der Zugriff ab. Der Orchestrator (Kapitel 7) sollte dafür einen wöchentlichen Reauth-Reminder auslösen, statt den Tageslauf stillschweigend fehlschlagen zu lassen.

### 15.5 Portale: Co-Pilot-Modus statt Vollautomatisierung

Der deutsche Bewerbungsmarkt hat zwei technisch unterschiedliche Formular-Welten: unternehmenseigene ATS-Karriereseiten (Personio, Softgarden, JOIN, SAP SuccessFactors, Workday) und Plattform-Schnellbewerbungen (LinkedIn Easy Apply, Indeed Apply, StepStone Schnellbewerbung, XING Sofort bewerben). Die Nutzungsbedingungen der drei großen Plattformen verbieten automatisierten Zugriff und automatisiertes Absenden explizit – LinkedIn im User Agreement [LinkedIn](https://www.linkedin.com/help/linkedin/answer/a1341387), StepStone in seinen AGB [StepStone](https://www.stepstone.de/ueber-stepstone/nutzungsbedingungen-2022-03/), Indeed in seinen Nutzungsbedingungen [Indeed](https://www.indeed.com/legal). Eine Kontosperrung träfe dabei das echte, langfristige Profil des Nutzers, nicht ein Wegwerf-Konto – rechtliche Einordnung dazu in Kapitel 16.

**Entscheidung:** Für alle Portale gilt derselbe Grundmechanismus – ein Playwright-Skript mit persistentem Chrome-Profil (`launchPersistentContext`, kein Headless-Modus) läuft sichtbar auf dem Rechner des Nutzers (reale, residentielle IP statt Cloud-Server), füllt alle Felder inklusive Lebenslauf-Upload und Anschreiben aus einem Standardantworten-Profil vor, und der Nutzer prüft und klickt selbst auf „Absenden“. Der Automatisierungsgrad davor unterscheidet sich je Portaltyp:

| Portaltyp | Beispiele | Automatisierung vor dem Submit | Risiko |
|---|---|---|---|
| ATS ohne bekannte Bot-Abwehr | Personio, Softgarden, JOIN | vollständig, inkl. Batch mehrerer Bewerbungen | niedrig |
| ATS mit Konto pro Arbeitgeber | SAP SuccessFactors, Workday | Co-Pilot, eigener Zugangsdaten-Tresor pro Firma | mittel–hoch |
| Plattform-Schnellbewerbung | LinkedIn Easy Apply, Indeed Apply, StepStone-Schnellbewerbung, XING Sofort bewerben | nur Co-Pilot, geringe Frequenz | hoch |
| Unbekannte Firmen-Karriereseite | individuelle Eigenentwicklungen | LLM-Fallback (z. B. browser-use) oder manuell markieren | variabel |

**Begründung:** Bei ATS-Formularen ohne explizites Automatisierungsverbot und ohne aktive Bot-Abwehr ist ein höherer Automatisierungsgrad vertretbar, weil kein ToS-Bruch vorliegt. SAP SuccessFactors und Workday erzwingen pro Arbeitgeber-Instanz ein eigenes Bewerberkonto [jobwizard.ai](https://jobwizard.ai/blog/why-does-workday-keep-asking-me-to-make-a-new-account-for-every-company) – das erfordert einen Zugangsdaten-Tresor pro Firma statt eines globalen Logins. Auf den Plattform-Schnellbewerbungen überwiegt das Risiko für das persönliche Netzwerk des Nutzers den Zeitgewinn deutlich, besonders da ohnehin nur ca. 10 hochwertige Bewerbungen/Tag angestrebt werden (Leitsatz Qualität statt Masse) und jede davon ohnehin vom Nutzer gegengelesen wird.

**Alternative(n):** Vollautomatisierung inkl. automatischem Klick auf „Absenden“ auch bei ATS-Formularen – technisch möglich, aber ohne Zusatznutzen, wenn ohnehin jede Bewerbung geprüft werden soll (Kapitel 14); bei Plattform-Schnellbewerbungen ausdrücklich nicht empfohlen.

Für den Datei-Upload (Lebenslauf-PDF) ist ein eigenes Playwright-Skript mit `page.setInputFiles()` der robustere Standardweg, weil er unabhängig von einer Tool-Schnittstelle funktioniert; der offizielle Playwright-MCP-Server bietet inzwischen ebenfalls ein `browser_file_upload`-Werkzeug, das für Navigation/Exploration während der Entwicklung praktisch bleibt [Playwright MCP README](https://raw.githubusercontent.com/microsoft/playwright-mcp/main/README.md). Die DSGVO-Einwilligungs-Checkbox im Formular füllt der Bote nicht automatisch aus – das finale Anklicken bleibt Teil des sichtbaren Human-Review, weil Einwilligung eine bewusste, aktive Handlung sein muss.

**Bestätigungen erfassen:** Nach jedem Absenden speichert der Bote automatisch einen Screenshot der Bestätigungsseite, extrahiert eine eventuelle Tracking-/Referenz-ID und verknüpft sie später mit der eintreffenden Eingangsbestätigungs-E-Mail. Dieses Nachweis-Paar (Screenshot + E-Mail) ist ein etabliertes Nachweisformat und wird auch von Jobcentern akzeptiert [bewerbungsflow.de](https://bewerbungsflow.de/blog/bewerbungsnachweise-fuers-jobcenter).

Ein Standardantworten-Profil deckt die immer wiederkehrenden Knockout-Fragen ab (Gehaltsvorstellung als Spanne + Zielwert, Kündigungsfrist, frühestes Eintrittsdatum, Arbeitserlaubnis, Umzugsbereitschaft, „Wie haben Sie von uns erfahren“) – diese Felder disqualifizieren Bewerber vor jeder menschlichen Prüfung, wenn sie falsch oder gar nicht beantwortet werden [quickcv.io](https://quickcv.io/blog/ats-knockout-questions). Das Profil ist Teil des Kandidatenprofils (Kapitel 8) und wird vom Boten nur gelesen, nicht neu erfunden.

### 15.6 Tracker: Statusmodell und Antwortklassifikation

Der Tracker überwacht die konfigurierten Postfächer per IMAP IDLE (Push statt Poll), providerunabhängig über Standard-IMAP [ikvk/imap_tools](https://github.com/ikvk/imap_tools). Eine eingehende Antwort wird über die Header `Message-ID`, `In-Reply-To` und `References` (RFC 2822) der ursprünglichen Bewerbung zugeordnet, wenn der Bote beim Versand eine eigene `Message-ID` gesetzt hat. Die Statuspipeline selbst (entdeckt → … → gesendet → Rückmeldung → Interview → Absage/Zusage/archiviert) ist in Kapitel 3 definiert; „Rückfrage offen“ ist dabei kein eigener Status, sondern ein Flag (`rueckfrage_offen`), das den aktuellen Status unverändert festhält, bis die Frage beantwortet ist (Kapitel 7.4). Der Tracker ist die Komponente, die eine eingehende E-Mail in einen dieser Statusübergänge übersetzt oder das Flag setzt.

Dafür klassifiziert ein LLM-Schritt jede neue Antwort strukturiert:

```json
{
  "kategorie": "absage | einladung | rueckfrage | autoresponder | unklar",
  "konfidenz": 0.0,
  "termin_erkannt": null,
  "vorgeschlagene_aktion": "status_setzen | rueckfrage_anlegen | nutzer_fragen"
}
```

| Kategorie | Beispiel-Merkmal | Status-Übergang | Folgeaktion |
|---|---|---|---|
| Absage | „... entschieden uns für ...“ | gesendet → Rückmeldung → Absage | Grund protokollieren, falls genannt; Archivierung erst im Wartungslauf (Kapitel 7.4, 7.5) |
| Einladung (mit Termin) | ICS-Anhang oder Datum im Text | gesendet → Rückmeldung → Interview | Termin in Tracker + optionaler ICS-Export |
| Einladung (ohne Termin) | Terminvorschlag gefordert | gesendet → Rückmeldung, Flag `rueckfrage_offen` | nach Terminbestätigung durch Nutzer → Interview |
| Rückfrage | z. B. Gehaltsangabe fehlt | gesendet → Rückmeldung, Flag `rueckfrage_offen` | Benachrichtigung (Kapitel 14) |
| Autoresponder | „Eingang bestätigt“ | bleibt gesendet | nur `event_log`-Eintrag |
| Unklar / niedrige Konfidenz | – | unverändert | Eskalation an Nutzer |

Bei Konfidenz unter dem Schwellenwert setzt der Tracker nie selbst einen Statusübergang; er legt dem Nutzer die Antwort zur Bestätigung im Cockpit vor (Kapitel 7.4, 14).

**Entscheidung:** Claude Haiku 4.5 klassifiziert im Standardfall; bei Konfidenz unter einem konfigurierbaren Schwellenwert (Default 0,7) eskaliert der Tracker an Claude Sonnet 5 für einen zweiten Versuch, bevor er dem Nutzer eine ungeklärte Antwort vorlegt.

**Begründung:** Klassifikation ist laut Styleguide (Kapitel 3) Massenarbeit, für die günstigere Modelle vorgesehen sind; bei zehn bis wenigen Dutzend Antworten pro Woche lohnt sich das teurere Modell nur für die unsicheren Fälle.

**Alternative(n):** Durchgängig Sonnet 5 verwenden – geringeres Fehlklassifikationsrisiko, aber unnötige Mehrkosten für den Großteil der eindeutigen Fälle (klare Absagen, klare Autoresponder).

ICS-Kalendereinladungen (MIME-Typ `text/calendar`, oft mit `method=REQUEST`) werden nicht dem LLM zur Freitext-Interpretation überlassen, sondern strukturiert mit der Bibliothek `icalendar` geparst (`SUMMARY`, `DTSTART`, `DTEND`, `LOCATION`) [PyPI icalendar](https://pypi.org/project/icalendar). Für MVP bleibt die Kalenderfunktion intern (ein Feld „fällig am“ in der Statustabelle plus optionaler `.ics`-Export, den der Nutzer manuell importiert); eine native Anbindung an Google/Outlook-Kalender ist ein v1-Ausbau, kein MVP-Bestandteil.

### 15.7 Nachfassen und Statistik

**Entscheidung:** Der Nachlauf (Kapitel 3: Antwortverarbeitung/Nachfassen) prüft täglich, welche Bewerbungen seit N Werktagen im Status „gesendet“ ohne Rückmeldung sind, und legt dafür einen Nachfass-Entwurf an – kurz, höflich, ohne Vorwurf, mit Bezug auf die ursprüngliche Bewerbung über den `References`-Header. Dieser Entwurf läuft durch denselben Freigabe-Gate wie die Erstbewerbung; es gibt kein automatisches zweites Nachfassen ohne erneute Freigabe.

**Begründung:** Ein Nachfassen ist inhaltlich heikler als eine Erstbewerbung (Tonfall entscheidet, ob es aufdringlich wirkt) und daher besonders prüfungswürdig; ein zweites automatisches Nachfassen würde dem Leitsatz „Qualität statt Masse“ widersprechen.

**Alternative(n):** Kein automatisches Nachfassen, nur eine Erinnerung an den Nutzer – reduziert Komfort, aber auch jedes Restrisiko eines unpassenden zweiten Kontakts; als Konfigurationsoption sinnvoll.

Statistiken (Antwortquote, Time-to-Response, Bewerbungen/Woche, Wirkung des Nachfassens) lassen sich bei diesem Volumen (ca. 10/Tag, ca. 300/Monat) mit einfachen SQL-Aggregationen auf derselben SQLite-Datenbank abbilden; ein separates BI-Tool ist unverhältnismäßig. Die Darstellung dieser Kennzahlen erfolgt im Review-Cockpit (Kapitel 14); der Tracker liefert nur die Daten.

### 15.8 Rechtliche Leitplanken (Kurzfassung)

Der Bote klickt nie selbst auf einen ToS-geschützten „Absenden“-Button einer Plattform (LinkedIn, Indeed, StepStone, XING) und umgeht nie eine technische Zugriffssperre. Die DSGVO-Einwilligung im Formular bleibt ein bewusster, sichtbarer Klick des Nutzers. Die vollständige rechtliche Einordnung (AGB-Risiken, § 202a StGB, Anthropic-Vertragswahl) steht in Kapitel 16.

### Offene Fragen an den Nutzer

- Welches E-Mail-Konto soll primär genutzt werden (bestehendes iCloud- oder Gmail-Konto, oder eine neue eigene Domain)? Default-Annahme: technologieoffen, beide Wege sind implementiert; der Nutzer wählt in der Konfiguration.
- Soll der MVP als reiner Entwurf-Modus starten, oder ist von Anfang an ein automatisierter Versand nach täglicher Freigabe gewünscht? Default-Annahme: Entwurf-Modus.
- Ist eine wöchentliche Browser-Reautorisierung im Gmail-OAuth-Testing-Modus akzeptabel, oder soll eine Google-Verifizierung beantragt werden? Default-Annahme: Testing-Modus akzeptieren.
- Soll LinkedIn/XING Easy-Apply überhaupt automatisiert werden (auch nur als Co-Pilot), oder grundsätzlich manuell bleiben? Default-Annahme: nur Co-Pilot, sehr geringe Frequenz.
- Sollen für SAP SuccessFactors/Workday automatisch neue Bewerberkonten pro Arbeitgeber angelegt werden, oder erfolgt die Kontoerstellung immer manuell? Default-Annahme: manuell in v1.
- Welches Nachfass-Intervall ist gewünscht (z. B. 7, 10 oder 14 Werktage)? Default-Annahme: 10 Werktage.

**Quellen dieses Kapitels:**
- [Apple: iCloud Mail – Sende- und Empfängerlimits](https://support.apple.com/en-us/102198)
- [Mailmeteor: iCloud SMTP-Einstellungen](https://mailmeteor.com/smtp/icloud-smtp-settings)
- [Google Workspace: Gmail-API-Quoten](https://developers.google.com/workspace/gmail/api/reference/quota)
- [Google Workspace: Gmail-API-Scopes](https://developers.google.com/workspace/gmail/api/auth/scopes)
- [Google Workspace: Gmail Drafts-Guide](https://developers.google.com/workspace/gmail/api/guides/drafts)
- [Google Cloud: OAuth-Testing-Modus](https://support.google.com/cloud/answer/15549945?hl=en)
- [Microsoft Learn: Graph-Throttling-Limits](https://learn.microsoft.com/en-us/graph/throttling-limits)
- [Fastmail: Preise und Pläne](https://www.fastmail.help/hc/en-us/articles/8033939068815-2024-pricing-and-plan-updates)
- [mailbox.org: Preispläne](https://mailbox.org/en/news/new-price-plans-available-mailboxorg/)
- [Woodpecker: SPF/DKIM einrichten](https://woodpecker.co/blog/spf-dkim/)
- [Instantly: Tracking-Pixel und Zustellbarkeit](https://instantly.ai/blog/email-tracking-and-deliverability-why-tracking-pixels-can-hurt-your-inbox-placement/)
- [arwa.de: Beste Sendezeit für Bewerbungen](https://arwa.de/de/blog/wann-sollte-man-eine-bewerbung-abschicken)
- [GitHub: ikvk/imap_tools (IMAP IDLE)](https://github.com/ikvk/imap_tools)
- [PyPI: icalendar](https://pypi.org/project/icalendar)
- [LinkedIn: Nutzungsbedingungen (Automatisierung verboten)](https://www.linkedin.com/help/linkedin/answer/a1341387)
- [StepStone: Nutzungsbedingungen](https://www.stepstone.de/ueber-stepstone/nutzungsbedingungen-2022-03/)
- [Indeed: Rechtliche Hinweise](https://www.indeed.com/legal)
- [jobwizard.ai: Workday-Konto pro Arbeitgeber](https://jobwizard.ai/blog/why-does-workday-keep-asking-me-to-make-a-new-account-for-every-company)
- [Playwright MCP: README (browser_file_upload)](https://raw.githubusercontent.com/microsoft/playwright-mcp/main/README.md)
- [quickcv.io: ATS-Knockout-Fragen](https://quickcv.io/blog/ats-knockout-questions)
- [bewerbungsflow.de: Bewerbungsnachweise fürs Jobcenter](https://bewerbungsflow.de/blog/bewerbungsnachweise-fuers-jobcenter)
