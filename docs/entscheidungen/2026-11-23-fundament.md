# Fundament — die Entscheidungen vom 23.11.2026

> **Was das hier ist:** Die Festlegungen, auf denen der Neuaufbau steht, jede mit
> ihrer Begründung. Keine Spezifikation — die entsteht daraus. Der Zweck dieses
> Dokuments ist, dass in einem halben Jahr noch nachvollziehbar ist, *warum*
> etwas so ist, und nicht nur *dass* es so ist.
>
> **Stand:** Fundament entschieden. Die Funktionsliste ist noch nicht priorisiert.

---

## Ausgangslage

Es gibt ein Vorgängerprojekt, das drei Dinge bewiesen hat:

1. **Der Automationsunterbau trägt** — Zustandsmodell, Lichtprofil, Ausnahmen
   über Kalender, Durchsagen mit Kategorien, Betriebsstundenzählung. Das lief.
2. **Die Oberfläche war das eigentliche Vorhaben** und blieb unfertig.
3. **Mehrere Bereiche waren monatelang blockiert**, weil Geräte fehlten. Energie,
   Klima und Sicherheit standen im Dashboard als beschriftete Lücken.

Der Neuaufbau ändert bewusst nur eine Sache grundlegend — siehe Entscheidung 7 —
und zieht aus dem Rest die Lehren.

---

## 1. Home Assistant läuft in einer VM, nicht in Docker

**Entschieden:** HAOS in UTM (die frei verteilte Variante, nicht die aus dem App
Store), Image `generic-aarch64`, QEMU-Backend mit UEFI, **Bridged** ins Heimnetz.

**Warum nicht Docker auf macOS:** Zwei harte Gründe.

Der HA-Container hat **keinen Supervisor** und damit keine Add-ons — kein
Matter-Server, kein Samba, kein Mosquitto. Alles einzeln als Container
nachzubauen ist Arbeit ohne Gegenwert.

Und das Netz: Docker läuft auf macOS selbst in einer Linux-VM; auch die
„Host networking"-Option routet durch deren NAT. Eine Hue-Bridge ließe sich
notfalls per IP eintragen, **Sonos nicht** — die Boxen schicken UPnP-Ereignisse
aktiv an die HA-Adresse zurück, und hinter NAT kommen die nie an. Dasselbe gilt
für mDNS-Discovery, die HomeKit-Bridge, Matter-Commissioning und Thread.

**Was eine Mac-VM nicht löst:** USB-Durchreichung für Zigbee- oder Thread-Sticks
ist auf Apple Silicon unzuverlässig. Ein Einplatinenrechner bleibt das Ziel —
dann läuft das Haus auch, wenn der Laptop zugeklappt wird, und ein altes Gerät
hängt nicht dauerhaft am Netzteil.

**Wenn ein Laptop übergangsweise die Basis ist**, wird Autostart Pflicht statt
Kür: Dienst beim Login starten, automatischen Login aktivieren, Ruhezustand im
Netzbetrieb abschalten. Und das Bridging gehört ans Kabel — über WLAN ist es der
häufigste Stolperstein, weil manche Access Points nur eine MAC-Adresse pro
Verbindung durchlassen und die Frames der VM still verwerfen.

## 2. Speicher: kurzer Recorder, eigenes Archiv ohne Verfall

**Entschieden:** HAs Recorder bleibt kurz (3–7 Tage) und ist **nicht** das
Archiv. Das eigene Backend hängt sich an HAs WebSocket-Ereignisstrom, ist der
**einzige** Schreiber ins Archiv, und das Archiv kennt kein Purge.

**Warum — gerechnet, weil „wie viel Speicher braucht das" die Eingangsfrage war.**
Annahmen: 120 numerische Reihen, 80 Zustandsreihen, rund 8.000 Ereignisse am Tag.

| Variante | pro Tag | pro Jahr |
|---|---|---|
| HA-Recorder (SQLite), Rohdaten, alles | ~37 MB | ~13 GB |
| Eigenes Archiv, 1-Minuten-Raster, komprimiert | ~0,4 MB | ~125 MB |
| Eigenes Archiv, 10-Sekunden-Raster | ~2 MB | ~750 MB |
| Ereignis-Logbuch (8.000/Tag, JSONB) | ~0,5 MB | ~175 MB |
| Stunden- und Tages-Rollups | — | < 20 MB |

**Das Ergebnis widerspricht der Erwartung: Speicher ist nicht die Grenze.**
„Lieber ein Datensatz mehr als weniger" ist bezahlbar. Teuer ist allein, dass
HAs Recorder eine komplette SQLite-Zeile pro Zustandsänderung schreibt,
unkomprimiert und dreifach indiziert. Das kostet keinen Platz, sondern
**Schreiblast und Abfragezeit** — genau das, was SD-Karten zerstört und
Oberflächen zäh macht.

**Datenbank: PostgreSQL mit TimescaleDB.** Der Anspruch lautet „wenig Speicher,
aber auch wenig Rechenarbeit zum Auslesen", und Continuous Aggregates sind genau
die zweite Hälfte davon: Stunden-, Tages- und Monatswerte werden automatisch
mitgeführt, die Jahresansicht liest 365 Zeilen statt Millionen. Eine reine
Zeitreihendatenbank könnte das nur über Umwege und ist für ein Ereignis-Logbuch
mit Fremdschlüsseln schwach.

**Nebenbei:** Für dieses Backend ist Docker richtig. Die NAT-Falle aus
Entscheidung 1 betrifft nur Home Assistant selbst; ein eigener Dienst muss nur HA
erreichen und Tablets bedienen, und beides geht über veröffentlichte Ports.

## 3. Geschrieben wird nur bei Änderung — plus eine Lebenszeichen-Spur

**Entschieden:** Die Werte-Tabelle enthält **nur Änderungen gegenüber dem
Vorwert**, kein Füllmaterial:

```
20.11.2026 10:00 UTC  Fenster zu
21.11.2026 11:00 UTC  Fenster auf
21.11.2026 11:30 UTC  Fenster zu
```

Drei Zeilen sagen alles über zwei Tage. Zeitstempel in **UTC**, Anzeige lokal —
sonst hat die Jahresgrafik im Oktober eine Stunde doppelt und im März eine zu
wenig.

**Die Lücke, die das lässt:** Ein stabiler Wert und ein toter Sensor sehen
identisch aus. Keine Zeile seit sechs Stunden heißt entweder „konstant 21,3°"
oder „Batterie leer".

**Deshalb daneben eine zweite, winzige Spur:** alle 15 Minuten „Sensor lebt, Wert
unverändert", eigene Tabelle, 90 Tage Aufbewahrung. Das deckelt sich bei rund
**3,5 MB, dauerhaft** — und Verstummen fällt binnen einer Viertelstunde auf,
statt beim nächsten Rückblick.

## 4. Zeitgewichtete Mittelwerte, von der ersten Aggregation an

**Entschieden:** Jede Verdichtung rechnet zeitgewichtet (`time_weight()`), Lücken
werden mit dem letzten bekannten Wert gefüllt (`locf()`).

**Warum das keine Feinheit ist:** Bei Änderungs-Logging bedeutet „wenig Zeilen"
nicht „wenig Zeit", sondern das Gegenteil. Ein ruhiger Tag erzeugt kaum Zeilen,
und die wenigen, die es gibt, sind genau die ungewöhnlichen Momente. Ein
ungewichteter Mittelwert zählt jede Zeile gleich und gibt damit systematisch den
Ausreißern das meiste Gewicht. **Je sparsamer geschrieben wird, desto falscher
wird er.**

Beispiel: 23 Stunden 21,0°, eine halbe Stunde 18,0° (Fenster auf), dann wieder
21,0°. Ein schlichtes `AVG` über die drei Zeilen sagt **20,0°**. Zeitgewichtet
sind es **20,94°** — fast ein Grad Unterschied wegen einer halben Stunde.

Die Rohdaten bleiben davon unberührt und Verdichtungen lassen sich jederzeit neu
rechnen. Der Schaden wäre kein Datenverlust, sondern ein Jahr lang auf Zahlen
geschaut zu haben, die nicht stimmen. Das reicht als Grund, es sofort richtig zu
machen.

## 5. Wetter wird mitgeschrieben — Messung und Vorhersage getrennt

**Entschieden:** Archiviert werden Außentemperatur, Windgeschwindigkeit,
Windrichtung, Bewölkung, Außenfeuchte und Niederschlag. Zusammen mit den
Vorhersage-Schnappschüssen kostet das **unter 10 MB im Jahr**.

**Warum das die Bedingung für jede Prognose ist:** Ohne Außentemperatur misst man
Einzelfälle — „an diesem Januartag fiel der Raum in zehn Minuten um 3°". Das sagt
nichts über heute. Mit ihr wird daraus ein Modell, weil die Abkühlung
proportional zur Differenz innen/außen läuft. Dann gibt es **eine** Kennzahl pro
Raum, die bei jedem Wetter anwendbar ist, auch bei einer Kombination, die so noch
nie vorkam. Das ist der Unterschied zwischen Erfahrungswert und Prognose.

**Wind ist der zweitwichtigste Faktor**, weil er den Luftwechsel am offenen
Fenster vervielfacht. Windrichtung entscheidet, ob ein bestimmtes Fenster
überhaupt im Wind steht.

**Ein eigener Außenfühler schlägt den Wetterdienst deutlich.** Der Dienst liefert
die Region, ein Fühler die Fassade — bei Februarsonne liegen die gern 8°
auseinander.

**Zwei Dinge gehören nicht ins Archiv:**

- **Sonnenstand nicht.** Höhe und Azimut sind eine reine Funktion aus Zeit und
  Koordinaten, für jeden vergangenen Zeitpunkt exakt nachrechenbar. Eine Spalte
  dafür wäre gespeicherte Redundanz.
- **Vorhersagen nicht in die Messwert-Tabelle.** Ein Messwert hat einen
  Zeitstempel, eine Vorhersage zwei: wann sie gemacht wurde und für wann sie
  gilt. Gemischt lässt sich hinterher weder sagen, wie warm es wirklich war, noch
  was man erwartet hatte. Also eine eigene Tabelle mit stündlichem Schnappschuss
  der nächsten 24–48 Stunden — die braucht das Vorheizen ohnehin, weil der
  Startzeitpunkt davon abhängt, was draußen passieren *wird*.

**Zwei Modell-Eingänge sind keine Wetterdaten und fehlen sonst:**

Der **Heizungszustand** muss mitlaufen. Sonst verwechselt das Modell „Fenster
auf, Heizung aus" mit „Fenster auf, Heizung heizt dagegen" — zwei völlig
verschiedene Abkühlkurven — und lernt den Mittelwert aus beiden, der für keinen
der beiden Fälle stimmt.

Und der **Fensterzustand ist nicht binär.** Gekippt und ganz offen unterscheiden
sich im Luftwechsel um Faktor drei bis fünf. Die meisten Kontaktsensoren melden
nur „auf" — dann lernt das Modell wieder einen Mittelwert, der nie eintritt.

**Für die Hardware-Empfehlung folgt daraus etwas Unerwartetes:** Für die Prognose
zählt die **Auflösung** des Fühlers mehr als die Schreibrate. Ein Sensor mit
0,5°-Stufen meldet bei einer Abkühlung von 0,3°/Minute nur alle 100 Sekunden
einen neuen Wert; die gerechnete Rate springt dann zwischen 0 und 6°/h, je
nachdem, wo die Stufe gerade fiel. Ein 0,1°-Fühler liefert im selben Moment alle
20 Sekunden einen Wert — daraus wird eine brauchbare Gerade. Es sind deshalb
**zwei** Kaufkriterien, nicht eins: feine Auflösung **und** ein kurzes minimales
Melde-Intervall. Manche Sensoren lösen fein auf und funken trotzdem nur alle zehn
Minuten.

## 6. „Nicht hartcoded" heißt: ziehen statt pflegen, und Rollen statt IDs

**Entschieden:** Räume und Geräte kommen aus Home Assistant und werden nicht
nebenher gepflegt. Automationen entstehen in der eigenen Oberfläche und lassen
sich exportieren und importieren.

**Was der Export dafür braucht, sonst trägt er nicht:** Eine hier gebaute
Automation zeigt auf konkrete Entitäten — `light.wohnzimmer_stehlampe`. In einem
anderen Haushalt heißt dasselbe Licht anders oder existiert gar nicht. Ein Export
mit Entity-IDs ist drüben wertlos.

Der Export liegt deshalb eine Ebene höher: **„Licht, Rolle *Hauptbeleuchtung*, in
einem Raum vom Typ *Wohnzimmer*"** statt einer konkreten ID. Beim Import wird das
einmal den echten Geräten zugeordnet — halbautomatisch, weil Home Assistant Räume
und Gerätetypen ohnehin mitliefert — und der Rest fällt an seinen Platz.

## 7. Der Automationskern wandert aus Home Assistant heraus

**Das ist die eine wirkliche Architekturänderung gegenüber dem Vorgängerprojekt.**

**Entschieden:** Automationen sind nicht mehr HA-YAML, sondern ein eigenes
Format, in der eigenen Oberfläche angelegt und von **einer** Engine ausgeführt.

| | |
|---|---|
| **Home Assistant** | Gerätetreiber — spricht mit Hue, Sonos, Matter, Thread |
| **Eigenes Backend** | hält Zustand, führt Automationen aus, schreibt ins Archiv |
| **Die Welt darunter** | zweimal vorhanden: echt über HAs WebSocket, oder simuliert. Gleiche Schnittstelle. |

**Warum:** Im Vorgängerprojekt gab es einen Simulator mit eigener Engine neben
HAs Engine — zwei Auswertungen derselben Regeln, die über die Zeit auseinander
laufen. Schlimmer noch: HA-Automationen benutzen Jinja2-Vorlagen, und die in
JavaScript exakt nachzubauen ist nicht seriös möglich. Der YAML-Importeur des
Simulators teilte deshalb jede Automation in `ausführbar`, `teilweise` und
`nur_anzeige` ein, und **komplexe Automationen wurden nur angezeigt, nicht
ausgeführt**. Damit konnte der Simulator ausgerechnet die kniffligen Regeln nicht
testen — also die, bei denen ein Fehler wahrscheinlich ist.

Sind die Regeln ein eigenes Format, verschwindet das Problem: Es gibt nur noch
eine Engine, und der Simulator führt nicht eine Nachbildung aus, sondern das
Original.

## 8. Simulator zuerst — und er muss gemein sein können

**Entschieden:** Gebaut wird gegen den Simulator, bevor Hardware gekauft wird.

**Warum:** Im Vorgängerprojekt waren Energie, Rollläden, Heizung und Alarm
monatelang mit „braucht Geräte" blockiert. Simulator zuerst dreht das um: fertig
bauen, dann gezielt kaufen, was die fertige Funktion wirklich braucht. Und es
erlaubt Entwickeln ohne Zugriff auf das Haus.

**Die Bedingung, sonst kehrt sich der Nutzen um:** Ein Simulator, in dem alles
sofort und korrekt funktioniert, bringt Code bei, der davon ausgeht, dass alles
sofort und korrekt funktioniert. Echte Geräte antworten vier Sekunden später,
melden `unavailable`, hängen auf einem alten Wert fest oder schicken Zustände in
der falschen Reihenfolge.

**Der teuerste Fehler des Vorgängerprojekts war genau so einer** und ist als
„der einzige Fehler in diesem Projekt, der jemandem schadet statt nur zu nerven"
protokolliert: Nach einem Neustart steht ein Template-Sensor kurz auf `unknown`.
Die Bedingung lautete „Anzeige reduzieren, wenn der Handwerkermodus **an** ist" —
und zeigte in diesem Fenster die **volle** Ansicht mit Anwesenheits- und
Terminangaben, während ein Handwerker davorstand. Ein netter Simulator hätte das
nie erzeugt. Ein gemeiner beim ersten Durchlauf.

Deshalb braucht der Simulator Knöpfe zum Quälen: Verzögerung, `unavailable`,
`unknown` nach Neustart, hängender Wert, Gerät antwortet gar nicht.

**Weiter entschieden:**

- **Eigene Logs, gleiches Schema.** Getrennte Datenbank, damit simulierte
  Zustandsänderungen die echte Statistik nicht verseuchen — aber dieselbe
  Struktur, damit die Auswertungslogik mit denselben Abfragen läuft und gleich
  mitgetestet wird.
- **Aufgezeichnetes erneut abspielen.** Eine echte Nacht durch eine geänderte
  Automation laufen lassen und sehen, was diesmal anders greift — Regressionstest
  fürs Haus, und die Daten liegen ohnehin da. Dafür muss jedes Ereignis seine
  **Ursache** mittragen: Benutzer, Automation mit Kennung, oder Szene.
- **Die 3D-Ansicht ist zugleich die Bedienfläche des Simulators.** Ein Fenster
  anzuklicken, um es zu öffnen, ist dort die natürlichste Geste — und die
  Komponente verdient sich damit früh ihr Geld, statt am Ende als Deko
  dazuzukommen.

**Eine Warnung zur Prognose:** Kühlt der Simulator einen Raum mit demselben
Modell ab, mit dem die Prognose rechnet, bestätigt er sich selbst und beweist
nichts. Der Simulator testet die Mechanik — ob die Warnung kommt, ob der Badge
stimmt, ob die Durchsage erfolgt. Ob das **Modell** stimmt, entscheiden
ausschließlich echte aufgezeichnete Daten.

## 9. Ausfall: ein Wächter in Home Assistant, und sonst nichts dort

**Der Punkt, der die Lösung bestimmt:** Ein abgestürztes Backend kann sich nicht
selbst in einen sicheren Zustand bringen. Es ist weg. Der Notbetrieb muss dort
liegen, wo dann noch etwas läuft — in Home Assistant.

**Entschieden:** Das Backend schickt alle 30 Sekunden ein Lebenszeichen. In HA
liegt **genau eine** Automation: Kommt zwei Minuten lang keines, Notbetrieb.

**Notbetrieb heißt:**

- **Heizung auf Absenkung, nicht aus.** Ganz aus ist im Winter Frost- und
  Schimmelrisiko; abgesenkt spart und ist sicher. Damit ist der Fall „Backend
  weg, Fenster offen" ohne eine zweite Regel abgedeckt.
- Rollläden bleiben stehen. Licht bleibt, wie es ist — niemand soll im Dunkeln
  stehen; Schalter funktionieren weiter über HA.
- Alarm behält seinen Zustand, Durchsagen schweigen.
- Der Handwerkermodus fällt auf die **strenge** Seite. `unknown` zählt wie `an`.
- **Die Tablets gehen in den Sperrbildschirm** — ohne Backend ist ohnehin nichts
  steuerbar. Sie müssen das **selbst** merken, am abgebrochenen WebSocket.
  Darauf zu warten, dass das Backend „ich bin weg" meldet, kann nicht
  funktionieren.

**Eine Regel, nicht fünf.** Ein erster Entwurf sah eine Handvoll
Sicherheitsregeln in HA vor; zurückgenommen. Je weniger in der Schicht liegt, die
niemals ausfallen darf, desto eher fällt sie nicht aus.

**Beim Neustarten:**

- Automatisch neu starten, **aber mit Zähler**. Fünf Abstürze in zehn Minuten
  heißt: aufhören, im Notbetrieb bleiben, Nachricht raus. Sonst flattert das Haus
  im Minutentakt zwischen Normal- und Notbetrieb, und das ist schlimmer als ein
  ehrlicher Stillstand.
- **Neu einlesen statt fortsetzen.** Das Backend fragt HA nach dem aktuellen
  Zustand, statt seinen letzten anzunehmen. Verpasste Auslöser werden
  **verworfen**, nicht nachgeholt — sonst begrüßt das Haus um 23 Uhr eine
  Heimkehr von 22:40.

Nebenbei fällt eine Statistik fürs Admin-Panel ab, die ohnehin gewünscht war:
Laufzeit, Abstürze, längste Störung.

## 10. Die Oberfläche hat drei Zustände und lügt nie

**Entschieden:** Jeder angezeigte Wert ist in einem von drei Zuständen, und das
ist ein **Prop** an der Komponente, kein zweiter Anzeigepfad.

| Zustand | Was die Kachel zeigt |
|---|---|
| `unknown` | Skeleton. Nie eine Zahl. |
| `stale` | Wert **plus Alter** — „21,3° · Stand 16:42" |
| `live` | Wert, normal |

**Zwei Konsequenzen, die nur am Anfang billig sind:**

Jeder Wert trägt **von Beginn an seinen Zeitstempel** mit. „Wie alt ist das?"
nachträglich in jede Komponente einzubauen ist genau die Sorte Umbau, die man
dann doch nicht macht.

Und es gibt **zwei verschiedene Sperrbildschirme** — „niemand daheim, Alarm
scharf" und „Backend weg". Die müssen deutlich unterschiedlich aussehen, sonst
steht man davor und weiß nicht, ob das Haus bewacht oder kaputt ist.

Es ist derselbe Fehler wie in Entscheidung 8, eine Ebene höher: Im Backend war
„unbekannt wird wie ein echter Wert behandelt" ein Sicherheitsproblem. In der
Oberfläche ist es ein Vertrauensproblem — eine Kachel, die 21,3° zeigt, obwohl
sie es seit zwei Stunden nicht weiß, lügt.

## 11. Plugins: Grenzen jetzt, eigene Repos später

**Entschieden:** Ein Repo mit echten Paketgrenzen (Workspaces). Jedes Feature ein
Paket, das mit dem Kern nur über eine definierte Schnittstelle redet.

**Warum nicht sofort getrennte Repos:** Der Aufwand pro Repo ist nicht der Code,
sondern die Pflege drumherum — eigene CI, Version, Changelog,
Abhängigkeits-Updates, README, Issues. Zehn Repos sind nicht zehnmal so viel
Code, aber ungefähr zehnmal so viel Hausmeisterei, und das für eine Person.

Dazu kommt der tiefere Grund: **Eine Plugin-Schnittstelle ist ein Vertrag, und
den kann man erst schreiben, wenn vier Features gebaut sind und man weiß, was sie
wirklich brauchen.** Wird er vorher definiert, formt sich jede Erweiterung nach
einer Vermutung.

**Die Wiederverwendbarkeit kommt von der Grenze, nicht vom Repo-Split.** Was
sauber abgegrenzt ist, lässt sich an dem Tag herauslösen, an dem jemand danach
fragt — mit Historie, in zehn Minuten. Zwölf Repos wieder zusammenzuführen ist
ein Wochenende.

**Trennlinie für das, was sofort nach draußen gehört** — *ergibt das ohne diesen
Hub Sinn?*

- **Ja → eigenes Repo, gern veröffentlicht.** Paketankündigungen,
  Briefkasten-Sensor, eine Fernseh-Brücke, Netzwerk-Messwerkzeuge. Dinge, die
  andere mit Home Assistant allein gebrauchen können.
- **Nein → Paket im Monorepo.** Fenster-Prognose, 3D-Grundriss,
  Automations-Editor, Admin-Panel. Die hängen am Datenmodell; ausgelagert wären
  sie nur umständlicher, ohne dass irgendwer etwas davon hat.

**Was das für das Plugin-Design festlegt:** Ein Feature, das sich nicht im
Simulator durchspielen lässt, kann niemand testen — weder man selbst noch ein
fremder Entwickler. Also **deklariert jedes Paket, welche Fähigkeiten es
braucht** („einen Temperaturfühler im Raum, einen Fensterkontakt"), statt sich
Entitäten selbst zu greifen. Dann kann der Simulator sie stellen — und dieselbe
Deklaration ist genau das, was der Automations-Export aus Entscheidung 6 ohnehin
benötigt. Eine Mechanik, drei Zwecke.

## 12. Öffentlich und privat werden getrennt gehalten

**Entschieden:** Dieses Repo enthält **nur** Code und Entscheidungen und ist
jederzeit veröffentlichungsfähig. Wohnungsdaten gehören in ein privates
Gegenstück.

**Was hier nie hineingehört:** Entity-IDs, Raumnamen einer konkreten Wohnung,
Adressen, Netzwerkangaben, MAC-Adressen, Grundrisse, Anwesenheits- und
Kalenderdaten, Namen von Bewohnern, Zugangsdaten.

**Durchgesetzt wird das mechanisch, nicht durch Disziplin:** Die `.gitignore`
arbeitet als **Whitelist** — erst wird alles ignoriert, dann werden einzelne
Pfade gezielt freigegeben. Ein Ordner, an den heute niemand denkt, ist damit
morgen automatisch ausgeschlossen statt automatisch drin.

**Die Lehre dahinter stammt aus dem Vorgängerprojekt:** Dort war die
Secret-Hygiene gut — echte Zugangsdaten lagen nie im Git. Trotzdem wuchs über
120 Commits eine vollständige Dokumentation von Wohnung, Netz und Anwesenheit in
die Historie, und die lässt sich nicht nachträglich entfernen, ohne die Historie
neu zu schreiben. **Ein Repo wird nicht durch einen Commit öffentlich, sondern
durch seine gesamte Vergangenheit.** Deshalb beginnt dieses hier bei null.
