# Changelog

## 2026-10-04 – outreach 0.1.4

**outreach** (Live-Ansicht in Claude Code)
- Import im Chat als eine Zeile ohne Fortschrittsbalken („Import gestartet · 500 Leads · Fortschritt über dem Prompt“); mehrere auf einmal gestartete Importe werden zu einer Zeile zusammengefasst. Der alte Balken blieb bei 0 %, weil der Import keinen Prozentwert meldet.
- Der Fortschritt steht jetzt im Band über dem Prompt: Status „wartet“/„läuft“ (Prozent nur, wenn der Server einen meldet), danach Ergebnis mit importierten Leads und Duplikaten oder der Fehlergrund; bei Abschluss ein Hinweis. Das Band fragt `get_job_status` alle 10 s, nur solange ein Import läuft.
- `get_job_status` zu anderen Job-Arten (z. B. Recherche) wird nicht als Import gezeigt.

## 2026-10-04 – outreach 0.1.3

**outreach**
- `outreach-copy` (Anrede): Du-Form grüßt mit „Hallo Vorname,“ der Person, der die Versandadresse gehört; bei generischen Adressen (info@, kontakt@ …) mit dem Vornamen des Entscheiders aus der Recherche; ohne klar erkennbare Person nur „Hallo,“ (nie „Hallo zusammen,“ vor du-Text). Sie-Form analog, Fallback „Guten Tag,“.
- `outreach-copy` (Opener): Lob oder anerkennende Beobachtung, getragen von einem konkreten Detail. Bewertungsanker ist eine Paraphrase des Lobs, Anzahl und Sterne nie allein und nie als erste Worte; Stellenanzeige als Erkenntnis über den Betrieb statt „ihr sucht“; zweiter Satz konkret und ohne Vorwegnahme des festen Texts; Fallback ohne unbelegte Behauptung; keine Zuschreibung fremder Leistungen an die angeschriebene Person.
- Beispiele ohne „finde ich spannend“ und Zahl-zuerst-Bewertungsopener; die Aussage „AI-Personalisierung verdoppelt bis verdreifacht die Reply Rate“ ist als Erfahrungswert mit Empfehlung zur Kontrollgruppe formuliert.
- `outreach-copy` (Anrede): Bei mehreren gleichrangigen Inhabern oder Geschäftsführern gilt der empfohlene Ansprechpartner, sonst die erstgenannte Person im Impressum.
- `outreach-copy` (Opener): Ohne Kundenlob sind Projekt/Referenz/Spezialisierung mit einem Detail, Firmengeschichte, Auszeichnungen (ohne Altersgrenze) sowie Lage und Ausstattung erlaubt; ohne konkreten Aufhänger steht der Fallback-Satz statt einer Leistungsliste; Substantive bleiben am Anfang groß.
- `outreach-campaign` (Research-Fokus), `outreach-generate` und `outreach-verify` an die neuen Regeln angepasst.

## 2026-10-04 – datenbeschaffung 0.1.1, outreach 0.1.2

**datenbeschaffung**
- Neu: `datenbeschaffung-setup` (Outreach-Verbindung, Apify-Zugang mit kostenlosem Selbsttest, optional Outscraper, alte Kopien aufräumen) und `datenbeschaffung-update`.

**outreach**
- `outreach-setup` verweist nach der Installation der Datenbeschaffung auf `datenbeschaffung-setup`.

## 0.1.1 – 2026-10-04

**outreach**
- Skills wiederholen Lead-Listen nicht mehr als Tabelle, wenn die Live-Ansicht in Claude Code sie schon als Karte zeigt.

## 0.1.0 – 2026-10-03

Erste Fassung als Marketplace `outreach-plugins` (vorher Skills-Repo `listm8-skills`).

**outreach**
- Plugin mit allen Outreach-Skills, MCP-Verbindung `akquise` (`outreach.akquise.de`) und
  Live-Ansicht für Claude Code (Lead-Vorschau, Band mit laufenden Lead-Runs, `/outreach-runs`).
- Neu: `outreach` (Einstieg), `outreach-setup` (Einrichtung, Aufräumen alter Kopien),
  `outreach-update`.
- `outreach-copy`: Cold-Mail-Copy-Regeln aus der Cold-Mailing-SOP; `outreach-launch`:
  Versand-Vorbereitung und Optimierung.
- Skills passen zum MCP mit 32 Tools; kein Scraping über den MCP, kein Probelauf.

**datenbeschaffung**
- Plugin mit dem Einstieg `datenbeschaffung`, allen Beschaffungswegen (Apify/Outscraper im
  eigenen Konto), Anreicherung, Listen-Qualität und den geteilten Referenzen.
