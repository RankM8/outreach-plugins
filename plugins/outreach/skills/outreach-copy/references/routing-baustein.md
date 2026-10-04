# Routing-Follow-up – „bei dir richtig oder eher bei X oder Y?“

Ein Follow-up, das den Empfänger fragt, ob eine andere, namentlich genannte Person der bessere
Ansprechpartner ist. Wirkt, weil es eine leichte Antwort erlaubt („schreib Anna“) und zeigt, dass
recherchiert wurde. Erprobt in einer Agentur-Kampagne mit über 400 generierten Leads.

## Wann einsetzen

- Zielgruppen mit Teams und verteilter Zuständigkeit: Agenturen, Praxen mit Praxismanagement,
  Kanzleien, Finanz-/Maklerbüros, Bau- und Immobilienfirmen.
- Besonders bei Sammelpostfächern (info@, team@, kontakt@), wo unklar ist, wer liest.
- Bei Solo-Betrieben fällt der Baustein von selbst auf eine normale, weiche Nachfrage zurück.

## Wo in der Sequenz

Als **Step 4** an Stelle von „Dringlichkeit“, wenn der Nutzer keinen echten Zeit- oder
Kapazitätsgrund hat (passt zur CTA-Stufe „kurz abklären“). Bei einer vom Nutzer gewünschten
kürzeren Sequenz auch als Step 3. Body des Steps: `{{ai.hallo}}` + Leerzeile + `{{ai.routing}}`.

## Drei Teile, die zusammengehören

1. **Recherche-Ziel** (in `researchAgentConfig.researchGoals`/`additionalPrompt`): höchstens zwei
   andere, aktuell bei der Firma beschäftigte Personen mit plausibler Verantwortung für das Thema
   des Angebots, je mit Name, Rolle und Quellen-URL; Empfänger selbst ausschließen. Bei Solo-Betrieb
   oder eindeutig zuständigem Empfänger „Routing nicht sinnvoll“ vermerken. Namen nie aus E-Mail-
   Adressen raten; `secondary_emails` sind kein Beleg für Personen.
2. **AI-Variable `routing`** (nach `hallo` und `intro`), Prompt siehe unten.
3. **Step-Body** wie oben, ohne Signatur-Dopplung.

## Prompt der Variable `routing` (Vorlage)

Angebotsbezogene Stellen in `<…>` anpassen; Du-Form gezeigt, Sie-Form analog.

```text
Erzeuge ausschließlich den vollständigen Text des Routing-Follow-ups, OHNE Anrede und Signatur,
maximal 55 Wörter. Nutze hallo aus demselben Ergebnis und die Routing-Belege der Recherche.
Ton: locker wie „not sure if I should be speaking to you or X or Y, let me know“, keine förmliche
Zuständigkeitsabfrage.
Fall A, persönliche Anrede und belegte andere Personen: 'ich bin mir nicht sicher, ob ich damit
bei dir richtig bin oder eher bei <Name 1> oder <Name 2>. Gib mir gern kurz Bescheid.'
Bei Teamanrede: 'sagt mir gern kurz Bescheid, falls ich mich damit besser bei <Name 1> oder
<Name 2> melden sollte.' Bei nur einer Person die zweite Alternative weglassen.
Fall B, keine belegten Alternativen und Zuständigkeit unklar: persönlich exakt 'sag mir gern kurz
Bescheid, falls ich mich damit besser bei jemand anderem im Team melden sollte.', bei Teamanrede
exakt 'sagt mir gern kurz Bescheid, falls ich mich damit besser bei jemand anderem im Team melden
sollte.'
Fall C, Solo-Betrieb oder eindeutig zuständiger Empfänger: keine künstliche Unsicherheit; stattdessen
eine weiche Nachfrage zum Angebot, persönlich '<möchtest du dir … ansehen? Ich schicke dir gern …>',
im Team '<möchtet ihr euch … ansehen? Ich schicke euch gern …>'.
Namen nur aus belegter aktueller Firmenzugehörigkeit mit passender Rolle; nie den Empfänger selbst.
Kein leerer Wert, keine Platzhalter, kein erneuter Pitch, keine Frist, keine Gedankenstriche.
```

## Erfahrungswerte

In der Agentur-Kampagne verteilten sich die generierten Werte auf etwa 20 % mit Namen (Fall A),
20 % Team-Fallback (Fall B) und 60 % Solo/zuständig (Fall C). Wer mehr Fall A will, braucht eine
Recherche, die Team- und Über-uns-Seiten gezielt liest; die Quote ist eine gute Kennzahl für die
Recherche-Qualität.
