# SmartHomeHUB

> Eine eigene Oberfläche und ein eigener Automationskern über Home Assistant —
> gebaut für Touch-Displays, Statistik und Haushalte, die nicht meiner sind.

**Status:** Fundament entschieden, Umsetzung noch nicht begonnen.

## Worum es geht

Home Assistant ist gut darin, mit Geräten zu reden, und schlecht darin, so
auszusehen und sich so zu verhalten, wie man es gern hätte. Dieses Projekt
übernimmt deshalb alles oberhalb der Geräte selbst:

- **Touch-Oberflächen** für Wandtablets, inklusive eines 3D-Grundrisses, in dem
  Lampen, Fenster und Rollläden direkt antippbar sind
- **Ein Admin-Panel** mit Statistik, Rückblicken und Systemzustand
- **Ein eigener Automationskern** mit Regeln, die man in der Oberfläche baut
  statt in YAML — und die sich exportieren und in einem anderen Haushalt wieder
  importieren lassen
- **Ein Langzeit-Archiv**, das nichts wegwirft, und Auswertungen darauf:
  Wie viel Grad verliert ein Raum beim Lüften? Wann sollte das Vorheizen
  beginnen, damit es pünktlich warm ist, ohne mit voller Kraft zu heizen?
- **Ein Simulator**, in dem sich das alles ohne ein einziges echtes Gerät bauen
  und testen lässt

## Wie es aufgeteilt ist

| | |
|---|---|
| **Home Assistant** | Gerätetreiber. Spricht mit Hue, Sonos, Matter, Thread — und sonst nichts. |
| **Dieses Projekt** | Zustand, Automationskern, Archiv, Oberflächen. |
| **Die Welt darunter** | Gibt es zweimal: die echte über HAs WebSocket, oder die simulierte. Gleiche Schnittstelle, austauschbar. |

## Nichts davon ist auf eine bestimmte Wohnung festgelegt

Räume und Geräte werden aus Home Assistant **gelesen**, nicht hier gepflegt.
Automationen beschreiben **Rollen** statt Entity-IDs („Licht, Rolle
Hauptbeleuchtung, in einem Raum vom Typ Wohnzimmer"), damit ein Export in einem
fremden Haushalt noch etwas bedeutet.

Dieses Repo enthält deshalb **keine** Wohnungsdaten: keine Entity-IDs, keine
Adressen, keine Netzwerkangaben, keine Grundrisse. Das gehört in ein privates
Gegenstück.

## Wo die Entscheidungen stehen

[`docs/entscheidungen/2026-11-23-fundament.md`](docs/entscheidungen/2026-11-23-fundament.md)
— zwölf Festlegungen mit Begründung: Unterbau, Speicherformat, Logging,
Ausfallverhalten, Modularität, Plugin-Grenzen. Wer wissen will, *warum* etwas so
ist, liest dort nach.

Offene Punkte und Reihenfolge: [`TODO.md`](TODO.md)

## Lizenz

Noch offen.
