# Arbeitsregeln für dieses Repo

## Was hier nie hineingehört

Dieses Repo soll jederzeit veröffentlicht werden können. **Keine** Entity-IDs,
Raumnamen einer konkreten Wohnung, Adressen, Netzwerkangaben, MAC-Adressen,
Grundrisse, Anwesenheits- oder Kalenderdaten, Namen von Bewohnern, Zugangsdaten.
Das gehört ins private Gegenstück.

Die `.gitignore` arbeitet als **Whitelist** — erst alles ignorieren, dann gezielt
freigeben. Beim Anlegen neuer Ordner muss die Freigabe bewusst ergänzt werden.
Das ist Absicht: Ein vergessener Ordner ist dann ausgeschlossen statt enthalten.

## Vor dem Bauen

Die Entscheidungen in `docs/entscheidungen/` sind verbindlich und tragen ihre
Begründung bei sich. Wer eine davon umwerfen will, ändert das Dokument mit und
schreibt dazu, was sich an der Begründung geändert hat.

Besonders leicht zu übersehen und besonders teuer nachzurüsten:

- **Kein `new Date()`** irgendwo im Code. Zeit kommt immer aus der Welt, sonst
  lässt sich nichts durchspielen.
- **Jeder Wert trägt seinen Zeitstempel** bis in die Oberfläche.
- **Unbekannt ist kein Wert.** Nie als Zahl rendern, nie als „aus" behandeln.
  Sicherheitsrelevante Zustände fallen auf die strenge Seite.
- **Verdichtungen rechnen zeitgewichtet**, nie mit schlichtem `AVG`.

## Commits

Commit-Nachrichten auf Deutsch, Betreffzeile sagt, was sich fachlich ändert,
nicht welche Datei angefasst wurde.
