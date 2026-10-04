---
name: outreach-pipeline
description: Use when user says "outreach:pipeline", "mcp:pipeline", "kompletter mcp durchlauf", "leads komplett verarbeiten", "lead lauf starten", "qualify research generate", "alles in einem lauf mcp", "full pipeline", "abo-lauf", "im abo", "lauf im abo", "mit subagents", "ohne server", "ohne openrouter", "lokal durchlaufen lassen", or triggers /mcp:pipeline. Server-Lauf (start_lead_run, kostet OpenRouter) oder Abo-Lauf (Subagents im Claude- bzw. ChatGPT-Abo, keine Server-Kosten).
---

# MCP Pipeline — serverseitiger Lead-Lauf (start_lead_run)

> **Live-Ansicht (Claude Code mit Plugin `outreach`):** Ergebnisse von `list_leads`, `import_leads`/`get_job_status` und Lead-Runs erscheinen dort als Karte bzw. im Band über dem Prompt. Dann die Liste **nicht noch einmal als Tabelle** wiederholen – nur kurz zusammenfassen, was der Nutzer wissen oder entscheiden muss. In anderen Umgebungen (Claude-Chat, ChatGPT, Codex) wie gewohnt als kurze Liste ausgeben.

Dieser Skill startet und überwacht die komplette Lead-Verarbeitung einer Kampagne als **einen serverseitigen Lauf**: das MCP-Tool `start_lead_run` verkettet Qualifizierung, Research und E-Mail-Variablen pro Lead mit den kampagneneigenen AI-Agents — abgerechnet über den OpenRouter-Account des Users (der Lauf kostet echtes Geld). Dein Client orchestriert nicht mehr selbst; er startet, pollt und berichtet.

```
ping + list_campaigns -> Konfiguration prüfen (export_campaign_blueprint) -> list_lead_runs(active_only=true)
    -> start_lead_run(stages=[...]) -> get_lead_run_status (poll bis is_terminal) -> Report
(danach manuell: /outreach-verify — Review & Approve)
```

## Aufruf

| Eingabe | Verhalten |
|---------|-----------|
| `/outreach-pipeline 80` | Voller Lauf für Kampagne 80 (alle 3 Stufen) |
| `/outreach-pipeline 80 --bis research` | Nur `stages: ["qualification","research"]` |
| `/outreach-pipeline` | Kampagne via list_campaigns wählen |
| `/outreach-pipeline 80 --abo` (oder „als Abo-Lauf“, „im Abo“, „ohne Server“) | Abo-Lauf: Qualifizierung → Recherche → Mail je Lead mit einem Agenten im Abo des Nutzers, siehe „Abo-Lauf“; keine OpenRouter-Kosten, belastet das Abo-Kontingent |

**Welcher Lauf?** Nennt der Nutzer weder Server noch Abo, kurz fragen: Server-Lauf (läuft im Hintergrund, kostet OpenRouter-Guthaben) oder Abo-Lauf (läuft in dieser Sitzung, nutzt das Claude- bzw. ChatGPT-Abo). Bei Abo-Lauf zusätzlich die Lead-Anzahl erfragen.

**Vorab abfragen:** Stufen (Default: alle 3 — Teilmengen und Lücken erlaubt, z.B. nur `["email"]`; bereits erfüllte Stufen werden pro Lead übersprungen) und optional ein `budget_usd` (bei größeren Läufen empfehlen). Review/Approve gehört bewusst NICHT in die Pipeline (Vier-Augen-Prinzip via /outreach-verify).

## Ablaufregeln

0. **Umgebung und Konfiguration (Pflicht)**: `ping` und `list_campaigns` — Konto und Kampagne müssen zum Auftrag passen. Dann `export_campaign_blueprint(campaign_id)` lesen: Qualifizierungskriterien, Research-Vorgaben, E-Mail-Schritte und AI-Variablen müssen vorhanden und sinnvoll sein (Details: `/outreach-campaign`). Fehlt etwas, NICHT starten, sondern erst ergänzen. Ein Lauf auf unvollständigem Setup kostet Geld, und ein abgeschlossenes Qualifizierungsurteil ist pro Kampagne final: der Lauf überspringt es, `write_lead_details` lehnt Änderungen mit `verdict_final` ab.
1. **Vorprüfung (Pflicht)**: `list_lead_runs(campaign_id, active_only=true)`. Ist ein Lauf aktiv: NICHT starten — parallele Läufe über dieselben Leads blockieren sich und können Leads still überspringen. Stattdessen den aktiven Lauf verfolgen oder mit `cancel_lead_run` stoppen. Vorbedingungen des Servers: OpenRouter-Key + AI-Modell konfiguriert; die email-Stufe braucht eine E-Mail-Sequenz an der Kampagne.
2. **Lead-Auswahl**: `lead_ids` (1-2000) für bekannte Mengen ODER `select_by_filter=true` für „alles, was ansteht" (Filter wie `list_leads`). Urteile gelten **pro Kampagne**: Ein Lead, der nur in einer anderen Kampagne qualifiziert wurde, gilt hier als nie qualifiziert. FALLSTRICK: Startet der Lauf bei der Qualifizierung, `fit_level=""` und `research_status=""` setzen — sonst matchen die Defaults frisch hinzugefügte Leads nicht (`no_leads_matched`). Bei `matched_total > selected` sind nur die Top-2000 nach Score im Lauf — Folgelauf für den Rest. Bereits kontaktierte Leads blendet der Default-Filter aus (`contact_status`).
3. **Optionen**: `budget_usd` (0.01-10000; erreicht => Lauf endet als `budget_exhausted`, laufende Jobs laufen aus). `agent_key` nur auf explizite User-Nennung — ein unbekannter Schlüssel überspringt die Qualifizierungs- und Research-Stufe still pro Lead.
4. **Fehler beim Start** (Text `<code>: <Meldung>`): `run_not_startable` = fehlende Vorbedingung (OpenRouter-Key, AI-Modell, E-Mail-Sequenz, gesperrter KI-Anbieter) — an den User zurückgeben, keine Retry-Schleife. `no_leads_matched` = der Filter trifft nichts — Filter anpassen oder mit `list_leads` und denselben Parametern gegenprüfen. `validation_failed` = ungültige Parameter (Stufen, Budget, `lead_ids` UND `select_by_filter` zugleich). `limit_reached` = mehr als 2000 `lead_ids` (in mehrere Läufe splitten) ODER Plan-Limit/Paywall (HTTP 402, der Text nennt den Grund) — dann an den User, kein Retry. `campaign_not_found` = falsche Kampagne. Ein erschöpftes Plan-Kontingent kann sich auch erst nach dem Start zeigen: Der Lauf endet dann als `limit_exhausted` (nur im Status sichtbar).
5. **Polling**: `get_lead_run_status(lead_run_id)` alle 30-60 s. Fortschritt am completed-Zähler der LETZTEN Stufe in `stageProgress[]` gegen `leadTotal` messen (Fehler = Summe aller `stageProgress[].failed`) — NICHT an den Stufen-Totals, die wachsen während des Laufs. Stoppen bei `is_terminal: true`. CAVEAT: ein `completed` jünger als ~30 Minuten kann der Server wieder auf `running` zurückholen — vor dem Abschlussbericht nachprüfen.
6. **Abbruch**: `cancel_lead_run(lead_run_id)` — storniert noch nicht gestartete Jobs und Folgestufen, bereits laufende Jobs laufen aus (`running_jobs`; leichtes Überschießen möglich); idempotent (`already_terminal`). Endzustand via `get_lead_run_status` verifizieren.
7. **Während des Laufs**: `save_lead_variables`, `approve_lead_variables` und `reject_lead_variables` (E-Mail-Stufe) sowie `write_lead_details` (Qualifizierung/Research) sind mit `lead_run_active` gesperrt, ebenso `switch_primary_email` bei aktiver Research-Stufe (Rennschutz, kein Fehler).

## Terminal-Status deuten

| Status | Bedeutung | Nächster Schritt |
|--------|-----------|-------------------|
| `completed` | Alle Leads durch | Stichprobe mit /outreach-verify |
| `completed_with_failures` | Mind. ein Job endgültig gescheitert | Fehl-Leads berichten, Folgelauf anbieten |
| `budget_exhausted` | Budget erreicht, Rest storniert | Restmenge beziffern, höheres Budget anbieten |
| `limit_exhausted` | Plan-Limit mitten im Lauf | An den User (Plan/Limit) — kein Auto-Retry |
| `provider_exhausted` | KI-Anbieter (OpenRouter) lehnt das Konto ab | An den User — kein Retry |
| `cancelled` | Vom User gestoppt | Stand berichten |
| `failed` | Vorbereitung gescheitert, nichts verarbeitet | `statusReason` ausgeben, Vorbedingungen prüfen |

## Abschluss-Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Lead-Lauf abgeschlossen — Kampagne {campaign.name} ({status})
Stufen: {stages} | Leads: {completed}/{leadTotal} | Fehler: {failed}
Kosten: {spentUsd} USD{von budgetUsd USD}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Hinweis zu den Kosten: `spentUsd` ist der Betrag, an dem `budgetUsd` stoppt. Laufende Jobs können nach dem Stopp noch Kosten nachbuchen.

Hinweis zu den Zahlen: `not_qualified`-Leads und Leads mit blockierendem Kontaktstatus verlassen die Kette OHNE Job — sie erscheinen in keinem Stufen-Zähler; `completed` kann deshalb legitim unter `leadTotal` bleiben.

Danach: "Nächster Schritt: Stichprobe mit /outreach-verify (10 Leads) bei neuer Kampagne, nach Änderungen oder vor dem ersten Export; danach freigeben."

## Abo-Lauf (Subagents im Claude- bzw. ChatGPT-Abo)

Soll der Client selbst denken (eigenes Abo statt OpenRouter-Guthaben, kein OpenRouter-Key, gezielte Einzelfälle), bearbeitet je Lead EIN Agent alle Stufen: qualifizieren → bei `not_qualified` aufhören → recherchieren (eine vorhandene Recherche gilt kampagnenübergreifend und wird nur genutzt) → Mail-Variablen schreiben. So wird `get_lead_data` je Lead nur einmal gelesen, und die Mail entsteht mit dem, was der Agent selbst über den Lead gelernt hat.

1. **Vorprüfung** wie beim Server-Lauf (Regeln 0 und 1): Konto, Kampagne, Setup, kein aktiver Lauf.
2. **Leads wählen:** `list_leads(campaign_id, fit_level="", research_status="", campaign_status="processing", qualification_status="pending", limit=<Anzahl>)`. Anzahl vorher erfragen; bei mehr als 50 Leads auf den Server-Lauf hinweisen (läuft im Hintergrund, belastet das Abo nicht).
3. **Fortschritt anmelden** (wenn `outreach_progress` verfügbar): je einmal `outreach_progress(action="start", campaign_id, phase=…, total=<Anzahl>)` für `qualification`, `research` und `email`. Das Band zeigt den Lauf als eine Zeile; nicht qualifizierte Leads zählen als aussortiert, eine schon vorhandene Recherche als erledigt.
4. **Agents starten:** je Lead genau ein Agent, höchstens 10 gleichzeitig (`run_in_background: true`, eine Welle je Message-Block).
   - Claude Code und Cowork: `outreach:lead-agent` mit dem Auftrag „Kampagne <id>, Lead <id> (<Firma>)“; sind mehrere Outreach-Server verbunden, zusätzlich „Server: <name>“.
   - Ohne diesen Agenten: `general-purpose` mit `model: "sonnet"`; Auftrag = die Sub-Agent-Vorlagen aus `/outreach-qualify`, `/outreach-research` und `/outreach-generate` nacheinander, mit ersetzten Platzhaltern, den Abschnitten „Anrede und Ansprache“ und „Intro-Regeln“ aus `outreach-copy` und dem Hinweis, bei `not_qualified` nach der Qualifizierung aufzuhören.
   - Ohne Subagents (manche Clients): die Leads nacheinander mit denselben Schritten.
5. **Bericht:** aus den Antwortzeilen der Agents (`OK lead=… fit=… recherche=… mail=…` bzw. `FEHLER …`): qualifiziert / aussortiert / Mails gespeichert / Fehler, je mit Lead. Nicht erneut per `list_leads` auflisten. Danach immer alle drei angemeldeten Phasen mit `outreach_progress(action="end", campaign_id, phase=…)` schließen, auch wenn alles gezählt scheint: Leads ohne Schreibaufruf (Recherche vorhanden, Mail wegen Fremdadresse übersprungen, Fehler) hält das Band sonst als offen.
6. **Rennschutz:** Fällt ein Schreib-Tool mit `lead_run_active` aus, läuft parallel ein Server-Lauf – abwarten (`get_lead_run_status`), dann fortsetzen.

Danach: Stichprobe mit `/outreach-verify` (10 Leads) bei neuer Kampagne, nach Änderungen oder vor dem ersten Export; danach freigeben.

## Verwandt

- Review: `/outreach-verify` · Einzelne Stufen im Abo: `/outreach-qualify`, `/outreach-research`, `/outreach-generate`
- Vorbereitung: `/outreach-campaign`, `/outreach-import`, `/outreach-lists`
- Copy-Fragen (Sequenz, Betreffzeilen, Offer, Prompts von `hallo`/`intro`, Prüfung der Mails) laufen über den Skill `outreach-copy`.
