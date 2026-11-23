# 🏠 SmartHomeHub

> Eine komplett eigene Oberfläche **und** ein eigener Automationskern über Home Assistant — gebaut für Touch-Displays, Statistik und Haushalte, die nicht meiner sind.

![Status](https://img.shields.io/badge/Status-In%20Planung-orange)
![Home Assistant](https://img.shields.io/badge/Home%20Assistant-41BDF5?logo=homeassistant&logoColor=white)
![Vue](https://img.shields.io/badge/Vue-3-4FC08D?logo=vuedotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![TimescaleDB](https://img.shields.io/badge/TimescaleDB-FDB515?logo=postgresql&logoColor=black)

## 💡 Idee

Home Assistant ist gut darin, mit Geräten zu reden, und schlecht darin, so auszusehen und sich so zu verhalten, wie man es gern hätte. Dieses Projekt übernimmt deshalb alles oberhalb der Geräte selbst: Oberfläche, Automationen, Archiv. Ein Projekt für den Nerd in mir.

## 🧱 Aufbau

| | |
|---|---|
| **Home Assistant** | Gerätetreiber. Spricht mit Hue, Sonos, Matter, Thread — und sonst nichts. |
| **Dieses Projekt** | Zustand, Automationskern, Langzeit-Archiv, Oberflächen. |
| **Die Welt darunter** | Gibt es zweimal: die echte über HAs WebSocket, oder die simulierte. Gleiche Schnittstelle, austauschbar. |

## ✨ Geplante Features

### 🖐️ Touch-UI
- **3D-Grundriss**: Lampen, Fenster und Rollläden antippen und direkt steuern
- Schnelle Aktionen an festen Stellen im Haus
- Modi für Privat, Besuch und Handwerker — als Prop an der Komponente, nicht als zweiter Anzeigepfad
- Sperrbildschirm mit Alarmzustand, entsperrt beim Heimkommen
- Optimiert für iPad und andere Wand-Tablets

### 🌡️ Klima & Energie
- **Fenster-Prognose**: Wie viel Grad verliert der Raum in 5, 10, 15 Minuten? Als Badge über dem Verlauf
- Wunschtemperatur einstellen, Erinnerung per Push oder Durchsage beim Erreichen
- Dasselbe für Luftqualitätswerte
- Heizung abgesenkt, solange ein Fenster offen steht
- **Vorheizen nach Kalender und Vorhersage** — langsam und sparsam statt mit voller Kraft
- Stromverbrauch der erfassten Geräte

### 📊 Admin-UI mit Statistik
- Rückblick wochen-, monats- und jahresweise
- Wie lange hat das Licht gebrannt? Wie oft war dieses Fenster offen?
- Temperaturverlauf im Raum, während ein Fenster offen war
- Wartungshinweise, Systemzustand, Laufzeit und Störungen
- **Automationen bauen, exportieren und importieren**

### 🚨 Sicherheit & Anwesenheit
- Alarmanlage mit Fenster- und Türsensoren
- Ankommen erkennen: Handy nähert sich **und danach** löst der Türsensor aus
- **Anwesenheitssimulation aus echten Betriebsstunden** statt aus einer Zeitschaltuhr
- Briefkasten-Sensor, Paketankündigungen

### 🔊 Durchsagen & Medien
- Lautsprecher-Durchsagen mit KI-Unterstützung
- Willkommens- und Verabschiedungsdurchsage
- Musik in mehreren Räumen, Folgefunktion über Präsenzmelder
- Wecker, mit Szenen verknüpfbar

### 📅 Integrationen
- Kalender als Grundlage für Vorheizen und Vorbereiten
- Fernseher: eigene Startseiten, die eine Szene auslösen können
- KI-Anbindung für Auswertung und Sprachbefehle

## 🧪 Simulator zuerst

Gebaut wird gegen einen Simulator, **bevor** Hardware gekauft wird — Zeit, Wetter, Sensoren und Szenen sind frei einstellbar, aufgezeichnete Nächte lassen sich erneut abspielen.

Und er kann **gemein** sein: Verzögerung, `unavailable`, `unknown` nach Neustart, hängender Wert, Gerät antwortet gar nicht. Ein Simulator, in dem alles sofort funktioniert, bringt Code bei, der genau das annimmt.

## 🔌 Nicht auf eine Wohnung festgelegt

Räume und Geräte werden aus Home Assistant **gelesen**, nicht hier gepflegt. Automationen beschreiben **Rollen** statt Entity-IDs — „Licht, Rolle *Hauptbeleuchtung*, in einem Raum vom Typ *Wohnzimmer*" — damit ein Export in einem fremden Haushalt noch etwas bedeutet.

Dieses Repo enthält deshalb **keine** Wohnungsdaten: keine Entity-IDs, keine Adressen, keine Netzwerkangaben, keine Grundrisse. Das gehört in ein privates Gegenstück.

## 📐 Warum es so ist

[`docs/entscheidungen/2026-11-23-fundament.md`](docs/entscheidungen/2026-11-23-fundament.md) — zwölf Festlegungen mit Begründung: Unterbau, Speicherformat, Logging, Ausfallverhalten, Modularität, Plugin-Grenzen.

Wer eine davon umwerfen will, ändert das Dokument mit. Offene Punkte und Reihenfolge stehen in [`TODO.md`](TODO.md).

## 🗺️ Roadmap

- [x] Fundament entschieden
- [ ] HAOS in einer VM, Bridged, mit Autostart
- [ ] Archiv-Schema: Werte, Ereignisse, Lebenszeichen, Vorhersagen
- [ ] Welt-Schnittstelle mit zwei Implementierungen — echt und simuliert
- [ ] Simulator mit Quälknöpfen
- [ ] Automationskern und Editor in der Oberfläche
- [ ] Touch-UI mit Schnellaktionen
- [ ] 3D-Grundriss
- [ ] Admin-UI mit Statistik
- [ ] Fenster-Prognose
- [ ] Alarmanlage und Sensoren
- [ ] KI-Durchsagen und Willkommensroutine

## 📄 Lizenz

Noch offen.
