# Persönlicher KI-Bewerbungsagent — Konzept und Masterplan

Ein Agent, der täglich den Stellenmarkt scannt, Unternehmen und Ansprechpartner wirklich recherchiert, maßgeschneiderte Anschreiben in der eigenen Stimme des Kandidaten schreibt, den Lebenslauf je Rolle anpasst, die Unterlagen als PDF und DOCX setzt, sie gegen Bewerbermanagementsysteme prüft und alles zur Freigabe vorlegt. Leitsatz: eine sorgfältige Bewerbung ist mehr wert als hundert generische.

## Inhalt dieses Ordners

| Datei / Ordner | Inhalt |
|---|---|
| `MASTERPLAN.md` | Das Gesamtdokument mit allen Kapiteln, Inhaltsverzeichnis und Quellenverzeichnis |
| `kapitel/` | Dieselben Kapitel als Einzeldateien (`NN-kurzname.md`) zum gezielten Weiterarbeiten |
| `recherche/` | Strukturierte Rechercheergebnisse je Thema als JSON, mit Quellen, Konfidenz und den Korrekturen der unabhängigen Faktenprüfung; `INDEX.md` gibt den Überblick |
| `STYLEGUIDE_UND_GLIEDERUNG.md` | Verbindliche Terminologie, Komponentennamen, Statuswerte und Kapitelgliederung |

## Aufbau des Masterplans

Kapitel 0 fasst alles auf einer Seite zusammen. Kapitel 1 bis 5 klären Ziele, Markt, Bewerbermanagementsysteme und die deutsche Bewerbungskultur. Kapitel 6 und 7 legen Datenquellen und Architektur fest. Kapitel 8 bis 15 beschreiben die einzelnen Module von Kandidatenprofil bis Versand und Nachverfolgung. Kapitel 16 bis 18 behandeln Recht, Werkzeuge und Kosten. Kapitel 19 bis 24 enthalten Roadmap, Risiken, Kommerzialisierung, offene Fragen, Glossar und Quellen.

## Wie dieses Dokument entstanden ist

Fünfzehn Themenfelder wurden mit Websuche recherchiert und in strukturierte JSON-Dateien überführt. Unabhängige Prüfer haben die entscheidungsrelevanten Behauptungen gegengeprüft; ihre Urteile stehen in `recherche/verify-*.json` und haben Vorrang vor den ursprünglichen Befunden. Auf dieser Grundlage haben Kapitelautoren die Einzelkapitel geschrieben, danach folgten Kritik- und Korrekturdurchgänge. Zahlen, die sich nicht bestätigen ließen, sind im Text als unbestätigt gekennzeichnet.

## Nächster Schritt

Kapitel 22 enthält den Fragenkatalog. Acht Fragen sind als Blocker markiert und sollten vor Phase 0 beantwortet werden. Kapitel 19 nennt die ersten zehn konkreten Schritte.
