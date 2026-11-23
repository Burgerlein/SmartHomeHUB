# Was als Nächstes ansteht

## ❓ Offene Entscheidungen

Keine davon blockiert die Arbeit — gesammelt, damit sie am Stück beantwortet
werden können.

**O1 — Gekippt und offen unterscheiden?**
Kostet einen zweiten Magneten pro Fenster oder einen Sensor mit Lagesensor. Das
ist eine **Kaufentscheidung**, keine nachträgliche. Die Alternative ist,
beides bewusst zusammenzuwerfen und die Prognose entsprechend ungenauer zu
halten. Hintergrund: Entscheidung 5.

**O2 — Mehrere Räume, unterschiedliche Musik.**
Mit einem einzelnen Streaming-Konto, über Radio, oder von einem eigenen
Medienserver? Ein Konto erlaubt in der Regel nur einen gleichzeitigen Stream —
das ist die eigentliche Frage dahinter. Noch nicht besprochen.

**O3 — Matter oder Thread als Zielstandard.**
Muss festliegen, bevor eine Hardware-Empfehlungsseite entsteht, sonst empfiehlt
sie in zwei Richtungen.

**O4 — Was wird aus der bestehenden Vue-Oberfläche?**
Im Vorgängerprojekt liegen rund 45 Komponenten mit je einer eigenen Testdatei,
auf Props ausgelegt, dazu ein Attrappen- und Offline-System. Die
Architekturänderung aus Entscheidung 7 betrifft das Backend, nicht die
Komponenten. Übernehmen, überarbeiten oder wirklich neu schreiben?

**O5 — Reihenfolge der Funktionen.**
Die Liste unten ist lang und noch nicht priorisiert. Das ist der nächste
Schritt.

## 🧱 Fundament — bevor Funktionen gebaut werden

- [ ] HAOS-VM in UTM, Bridged, Autostart (Entscheidung 1)
- [ ] Postgres mit TimescaleDB, Schema für Werte, Ereignisse, Lebenszeichen,
      Vorhersagen (Entscheidungen 2–5)
- [ ] Die Welt-Schnittstelle: eine Fassade, zwei Implementierungen —
      echt und simuliert (Entscheidung 7)
- [ ] Einspeisbare Uhr. **Kein `new Date()` irgendwo im Code**, Zeit kommt
      immer aus der Welt. Nachträglich einzubauen ist Hölle, weil jede
      vergessene Stelle still falsch läuft.
- [ ] Simulator mit Quälknöpfen: Verzögerung, `unavailable`, `unknown` nach
      Neustart, hängender Wert, keine Antwort (Entscheidung 8)
- [ ] Lebenszeichen und die eine Notbetriebs-Automation in HA (Entscheidung 9)
- [ ] Die drei Anzeigezustände als Prop, Zeitstempel an jedem Wert
      (Entscheidung 10)

## 📋 Funktionen — gesammelt, nicht sortiert

**Klima und Energie**
- Fenster-Prognose: Grad-Verlust in 5/10/15 Minuten als Badge, Wunschtemperatur
  einstellbar, Erinnerung per Push oder Durchsage beim Erreichen
- Dieselbe Mechanik für Luftqualitätswerte
- Heizung aus bzw. abgesenkt bei offenem Fenster
- Vorheizen anhand von Kalender und Vorhersage, langsam statt mit voller Kraft
- Energieerfassung und -statistik

**Steuerung**
- Rollladensteuerung
- 3D-Grundriss mit antippbaren Elementen
- Musiksteuerung: mehrere Räume, unterschiedliche Quellen (siehe O2)
- Präsenzmelder: Musik-Folgefunktion, raumbezogene Szenen

**Anwesenheit und Sicherheit**
- Ankunftsbenachrichtigung
- Ankommen: Handy nähert sich **und danach** löst der Türsensor aus
- Touch-Dashboards in den Modi Handwerker, Besuch, Privat
- Sperrbildschirm mit Alarmzustand, entsperrt beim Heimkommen
- Anwesenheitssimulation aus echten Betriebsstunden

**Integrationen**
- Kalender
- Fernseher: eigene Startseiten, die einen lokalen Push auslösen und Szenen
  auslösen können; Szene startet ein Video (z. B. Kamin)
- Paketankündigungen
- Briefkasten-Sensor
- Wecker, mit Szenen verknüpfbar
- KI-Anbindung für Auswertung und Sprachbefehle

**Rund ums Projekt**
- Admin-Panel mit Statistik, Rückblick wochen-, monats-, jahresweise
- Automationen in der Oberfläche anlegen, exportieren, importieren
- Hardware-Empfehlungsseite (abhängig von O1 und O3)
- Netzwerk-Messungen wiederkehrend auswerten
