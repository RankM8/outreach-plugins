# Changelog

## 2026-10-04 – outreach 0.3.1

**outreach** (Copy-SOPs nach dem Testfeld korrigiert)
- `outreach-copy`: Betreff nur in Step 1, keine Ankündigung oder Ergebnisansage („Website für …“), Standard „kurze Frage“ (Buchungssystem als Kern: „Frage zur Terminbuchung“). Firmenname im Betreff nur über `{{ai.firma}}`, nie über das rohe `{{lead.company}}`; ohne `firma` „kurze Frage“. Follow-ups (Step 2-5) haben keinen Betreff und laufen im Thread. Betreff-Beispiele in `SKILL.md`, `copy-lehre.md`, `marketing-offer.md` und `beispiel-blueprint.md` angeglichen (Karte C hatte zwei verschiedene Betreffs).
- `outreach-copy`: Opener. Einstiegsrahmen wie „ich hab mir … angeschaut“ sind erlaubt (Widerspruch zu „nie über den Absender“ aufgelöst); eine bloße Feststellung („du bietest X an“) ist kein Aufhänger, dann gilt der Fallback; Gründungsjahr mit Ereignis und Teamgröße sind keine verbotenen Selbstangaben-Zahlen.
- `outreach-campaign`: Variable `firma` kürzt die Rechtsform ohne Wortrest, macht aus einer Domain die Marke, übernimmt einen neueren Namen aus Impressum oder Recherche, nimmt bei langen Namen den Markenkern und schreibt Versalien normal (Abkürzungen bis 4 Buchstaben bleiben). Neu: abgeleitete oder geklonte Kampagnen gegen das neue Offer prüfen (kein Offer-Rest im Prompt).
- `outreach-campaign` → Qualifizierung: Ausschlussgründe nur, was ein Agent öffentlich prüfen kann (nie Kontaktsperre oder Bestandskunde); Konzern (fremde Muttergesellschaft, Franchise, Börse) und Größenschwelle als getrennte Punkte; Nachbarbetriebe der Branche ausdrücklich ausschließen; kein Ausschlussgrund schließt den Bedarf aus, den das Angebot löst; „Selbstzahlerleistung erkennbar“ ist ein Fit-Kriterium; keine Adress- oder Pipeline-Anweisungen im `additional_prompt`.
- Hinweis: Leere Betreffs in Step 2-5 setzen voraus, dass der ListM8-Server sie beim Blueprint-Import annimmt.

## 2026-10-04 – outreach 0.3.0

**outreach** (Abo-Lauf vereinfacht: ein Agent je Lead für alle Stufen)
- Neuer Plugin-Agent `outreach:lead-agent` (Sonnet, alle Werkzeuge): qualifiziert, hört bei `not_qualified` auf, recherchiert (vorhandene Recherche wird genutzt) und schreibt die Mail-Variablen – `get_lead_data` wird je Lead nur einmal gelesen, die Mail entsteht mit dem, was der Agent selbst über den Lead gelernt hat. Lädt den Copy-Skill selbst; nennt der Auftrag einen Server, nutzt er nur dessen Werkzeuge.
- Entfernt: die Agenten `qualifier`, `researcher`, `writer` und die schlanken Mod-Agenten (`*-schlank`). Ihre Werkzeug-Einschränkung führte zu Abbrüchen (große `get_lead_data`-Antwort als Datei ohne Read, Werkzeugnamen je nach Servername).
- `outreach-pipeline`: Abschnitt „Abo-Lauf“ – Leads wählen, Fortschritt anmelden, je Lead ein `lead-agent` (höchstens 10 gleichzeitig), Bericht aus den Antwortzeilen statt erneuter Lead-Liste. `outreach`: leitet „Abo-Lauf“, „im Abo“, „ohne Server“ an `outreach-pipeline --abo` weiter.
- `outreach-qualify`, `outreach-research`, `outreach-generate`: einzelne Stufe im Abo über `lead-agent` mit „nur <Stufe>“.
- Band: ein Abo-Lauf ist EINE Zeile („5 Leads · 3 fertig · 1 aussortiert · Qual 5/5 · Rech 4/4 · Mail 3/4 · im Abo“); eine schon vorhandene Recherche zählt als erledigt, sobald die Mail des Leads gespeichert ist.

## 2026-10-04 – outreach 0.2.1

**outreach**
- `outreach-pipeline`: Abo-Lauf ist direkt aufrufbar (`/outreach-pipeline 80 --abo`, „im Abo“, „ohne Server“) und steht in der Aufruf-Tabelle; ohne Angabe fragt der Skill, ob Server- oder Abo-Lauf.

## 2026-10-04 – outreach 0.2.0

**outreach** (Abo-Läufe: Leads im Claude- bzw. ChatGPT-Abo statt auf dem Server verarbeiten)
- Neue Plugin-Agenten `qualifier`, `researcher`, `writer` (Sonnet, ein Lead pro Agent) – laufen überall, auch in Cowork.
- In Claude Code meldet der Mod zusätzlich schlanke Varianten `qualifier-schlank`, `researcher-schlank`, `writer-schlank` an: nur die Outreach-Werkzeuge des verbundenen Servers (unter dem Namen, den der Kunde vergeben hat), Sonnet. Gemessen: weniger als halb so viel Kontext je Lead wie ein allgemeiner Agent.
- `outreach-qualify`, `outreach-research`, `outreach-generate`: Abschnitt „Abo-Lauf“ – ein Lead pro Agent (keine Sammel-Agenten, sonst Verwechslungen), Agent-Wahl schlank → Plugin-Agent → `general-purpose` mit Sonnet, höchstens 10 gleichzeitig, Anmeldung im Band über `outreach_progress`. Generate: der Orchestrator hängt die Copy-Regeln an, Subagents laden den Skill nicht selbst.
- `outreach-pipeline` (Manuell-Modus): Kette je Lead (Qualifizierung → Recherche → Mail) als Workflow in Claude Code, sonst Phasen nacheinander.
- Band: Abo-Phasen heißen „im Abo“; als nicht qualifiziert beurteilte Leads gelten in Recherche und Mail als aussortiert („4 fertig · 1 aussortiert“, grün); alle drei Phasen eines Laufs bleiben sichtbar; „Öffnen“ führt in die App des Servers, der geschrieben hat (vorher teils falsche Domain); Verbrauch am Wochen-Kontingent des Abos seit Phasenstart („+2,4 % Woche“).

## 2026-10-04 – outreach 0.1.9

**outreach**
- `outreach-copy`: Musterbeispiele in Skill und `marketing-offer.md` auf die Du-Form umgestellt (vorher „für euch erstellt“, „euch das morgen zusende“ – Agenten übernahmen daraus die Ihr-Form).
- `outreach-copy`: Das erste Wort nach `{{ai.hallo}}` schreibt man in jedem Step klein („du hast sicher …“).

## 2026-10-04 – outreach 0.1.8

**outreach**
- `outreach-copy` (Ansprache): `du` an den Ansprechpartner ist ausdrücklich der Standard, auch für Praxen, Finanz und generische Adressen. `team` und `sie` nur auf ausdrücklichen Wunsch – nicht, weil eine alte Vorlage oder Bestandskampagne so schreibt. Mischform „Hallo Max,“ vor „euch“ als Fehler benannt.

## 2026-10-04 – outreach 0.1.7

**outreach**
- Neuer Baustein Routing-Follow-up („bin ich bei dir richtig oder eher bei Anna oder Tom?“): `outreach-copy/references/routing-baustein.md` mit Einsatzfällen, Platz in der Sequenz (Step 4 statt „Dringlichkeit“), Recherche-Ziel, Prompt-Vorlage der Variable `routing` (Fälle mit Namen, Team, Solo) und Erfahrungswerten. `outreach-copy` und `outreach-campaign` verweisen darauf und bieten ihn bei Zielgruppen mit Teams an.

## 2026-10-04 – outreach 0.1.6, datenbeschaffung 0.1.2

**outreach**
- `outreach-import`: Job-Ergebnis richtig beschrieben – `duplicates`, `internalDuplicates`, `do_not_contact_hits` und `errors` sind Listen, berichtet wird ihre Anzahl; `consolidated` erklärt. Import meldet keinen Prozentwert, nur den Status.
- `outreach-import`: realistische Paketgröße im Chat (etwa 100–300 Leads je Aufruf, jeder Lead kostet Kontext); bei mehreren Tausend Zeilen den CSV-Import der App empfehlen und hier nur vorbereiten.

**datenbeschaffung**
- Übergabe an Outreach (`outreach-uebergabe.md`, `listm8-mcp.md`, `listen-qualitaet`): dieselbe Korrektur der Job-Ergebnis-Felder.

## 2026-10-04 – outreach 0.1.5

**outreach** (Live-Ansicht in Claude Code)
- Import-Ergebnis zählt Duplikate wieder: Der Server liefert sie als Liste der betroffenen Zeilen, nicht als Zahl; vorher fiel die Angabe weg.

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
