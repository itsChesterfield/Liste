# To-Do-Liste-Ohne-GUI – Einsteiger-Dokumentation

Dieses Dokument erklärt dir alles, was du wissen musst, um das Projekt nach längerer Pause wieder
zu verstehen und weiterzuprogrammieren.

---

## Übersicht: Was ist dieses Programm?

Eine **terminalbasierte To-Do-Liste in Java**, ohne grafische Oberfläche.  
Aufgaben werden mit Priorität (1–3) und automatischem Erstellungsdatum/-uhrzeit gespeichert.

Das Langzeitziel laut README:  
> Die App soll entscheiden, welche Aufgabe heute die „**eat the frog**"-Aufgabe ist – also die
> wichtigste/unangenehmste Aufgabe, die man zuerst erledigen sollte – und alle weiteren nach
> Wichtigkeit und Deadline sortieren.

**Dieses Feature ist noch nicht implementiert.** Aktuell kann man Aufgaben anlegen und ausgeben.
Die Sortierung ist halbfertig.

---

## Dateistruktur

```
To-Do-Liste-Ohne-GUI-/
├── ToDo.java          # Datenmodell – eine einzelne Aufgabe
├── Knoten.java        # Listenknoten – erweitert ToDo, hat left/right-Zeiger
├── List.java          # Die eigentliche verkettete Liste mit allen Operationen
├── KnotenTest.java    # JUnit-5-Tests für Knoten und kleine Listenoperationen
├── ListTest.java      # JUnit-5-Tests für List-Validierungen
└── README.md          # Kurze Projektbeschreibung (sehr knapp)
```

---

## Klassen im Detail

### `ToDo.java` – das Datenmodell

Repräsentiert **eine einzige Aufgabe**. Implementiert `Comparable<ToDo>` für Prioritätsvergleiche.

| Feld | Typ | Bedeutung |
|---|---|---|
| `value` | `String` | Aufgabenbeschreibung |
| `priority` | `int` | Priorität, erlaubt: 1 (niedrig) bis 3 (hoch) |
| `date` | `LocalDate` | Erstellungsdatum – wird automatisch gesetzt |
| `time` | `LocalTime` | Erstellungsuhrzeit – Sekunden/Nanosekunden werden auf 0 gesetzt |
| `counter` | `int` | Laufende Positionsnummer in der Liste |

**Wichtige Methoden:**
- `setValue(String)` – wirft `NullPointerException` bei null, `IllegalArgumentException` bei leerem String
- `setPriority(int)` – wirft `IllegalArgumentException` wenn nicht 1–3
- `compareTo(ToDo)` – vergleicht nach Priorität: gibt -1, 0 oder 1 zurück
- `toString()` – Ausgabe: `"counter: value (prio:X) | Erstellt am: YYYY-MM-DD | Uhrzeit: HH:MM"`

**Offenes TODO** (Zeile 20):
```
// TODO: Datum und Uhrzeit noch weiter anpassen, bis es schön aussieht.
```
Die Formatierung könnte noch schöner werden, z. B. mit `DateTimeFormatter`.

---

### `Knoten.java` – der Listenknoten

Erbt von `ToDo` und fügt die Zeiger für die **doppelt verkettete Liste** hinzu.

| Feld | Typ | Bedeutung |
|---|---|---|
| `left` | `Knoten` | Zeiger auf den vorherigen Knoten |
| `right` | `Knoten` | Zeiger auf den nächsten Knoten |
| `todo` | `ToDo` | **NICHT BENUTZT / FEHLER** – siehe unten |

**Konstruktoren:**
- `Knoten(String value, int priority)` – einfacher Konstruktor, delegiert an `ToDo`
- `Knoten(String value, int priority, Knoten left, Knoten right)` – mit Zeigerinitialisierung

**Bekannte Bugs in dieser Klasse:**

1. **`todo`-Feld ist überflüssig und nicht richtig initialisiert.**  
   `Knoten` erbt bereits alle Felder von `ToDo`, ein eigenes `todo`-Objekt ist redundant.  
   In Zeile 19 steht `this.todo = todo;`, aber der Parameter `todo` existiert im Konstruktor
   gar nicht – das kompiliert entweder nicht oder referenziert etwas Falsches.

2. **`knotenAusgabe()`-Methode ist kaputt.**  
   Sie ruft `todo.toString()` auf, aber `todo` ist nie korrekt initialisiert. Diese Methode
   funktioniert nicht. Stattdessen funktioniert `toString()`, das von `ToDo` geerbt wird.

**Was du tun solltest:**  
Entweder das `todo`-Feld komplett löschen (empfohlen) oder den Design-Ansatz überdenken.
Die `knotenAusgabe()`-Methode entweder reparieren (sie soll `this.toString()` aufrufen)
oder löschen.

---

### `List.java` – die Hauptklasse

Verwaltet die **doppelt verkettete Liste** aller Aufgaben.

| Feld | Typ | Bedeutung |
|---|---|---|
| `top` | `Knoten` | Zeiger auf den ersten Knoten (zuletzt hinzugefügt) |
| `position` | `int` | Zählt, wie viele Knoten insgesamt hinzugefügt wurden |

**Methoden:**

| Methode | Status | Beschreibung |
|---|---|---|
| `add(String, int)` | ✅ fertig | Fügt neue Aufgabe am Anfang der Liste ein |
| `add(Knoten)` | ✅ fertig | Überladung: nimmt direkt einen Knoten |
| `ausgabe()` | ✅ fertig | Gibt alle Aufgaben aus (traversiert von `top` nach rechts) |
| `middleList(List)` | ✅ fertig | Gibt den mittleren Index der Liste zurück |
| `split(List)` | ⚠️ halbfertig | Teilt die Liste in der Mitte – für Merge Sort vorbereitet |
| `switching(Knoten, Knoten)` | ⚠️ vorhanden | Tauscht zwei Knoten aus – wird von Sortierung genutzt |
| `sortPositionPriority()` | ❌ unfertig/buggy | Soll nach Priorität sortieren – Logik ist fehlerhaft |
| `insert(String, int)` | ❌ leer | Methode existiert, aber kein Inhalt |
| `delete(Knoten)` | ❌ leer | Gibt nur den Knoten zurück, löscht nichts |

**Außerdem:** Der Import `java.sql.SQLOutput` ist vorhanden aber wird nirgendwo verwendet – kann
gelöscht werden.

---

## Wie die Datenstruktur funktioniert

```
top
 ↓
[Knoten A] ←→ [Knoten B] ←→ [Knoten C] ←→ null
```

- Neue Knoten werden **am Anfang** eingefügt (neueste Aufgabe = `top`)
- Jeder Knoten zeigt mit `right` auf den nächsten, mit `left` auf den vorherigen
- Es gibt **keine Persistenz** – beim Beenden des Programms gehen alle Daten verloren
- `ausgabe()` traversiert von `top` immer nach rechts (`.right`) bis `null`

---

## Was funktioniert bereits

- Aufgaben anlegen mit `list.add("Aufgabe", 2)`
- Erstellungsdatum und -uhrzeit werden automatisch erfasst
- Eingabevalidierung (null, leer, Priorität außerhalb 1–3) mit sinnvollen Exceptions
- Alle Aufgaben anzeigen mit `list.ausgabe()`
- Prioritätsvergleich per `compareTo()` (wird für Sortierung gebraucht)
- JUnit-Tests für Validierungen laufen durch

---

## Was noch fehlt (Reihenfolge nach Sinnhaftigkeit)

### 1. Merge Sort fertigstellen – das wichtigste offene Thema

Du hast dir das selbst in Kommentaren notiert. Der Plan:

```
// AKTUELL: split() teilt die Liste nur in zwei Hälften
// ZUKÜNFTIG: Soll jede Teilliste wieder splitten, bis jeder Knoten allein steht.
// Dann alle Knoten in der richtigen Reihenfolge verschmelzen → sortierte Liste.
//
// 1. merge() als eigene Methode schreiben (damit man nach verschiedenen Kriterien
//    sortieren kann, z.B. Priorität, Deadline, Alter)
// 2. Entweder rekursiv oder Werte in Array zwischenspeichern
```

**Empfehlung:** Rekursiver Merge Sort direkt auf der verketteten Liste, ohne Array.
Vorhandene Hilfsmethoden: `middleList()` und `split()` sind schon da.

### 2. `delete(Knoten)` implementieren

Zeiger des Vorgängers und Nachfolgers anpassen, dann Knoten loslassen.

```java
// Logik:
// knoten.left.right = knoten.right
// knoten.right.left = knoten.left
// Sonderfall: knoten ist top → top = knoten.right
// Sonderfall: knoten ist letzter → nur left.right = null
```

### 3. `insert(String, int position)` implementieren

Bis zur gewünschten Position traversieren, dann Zeiger neu setzen (wie bei `add`, aber an
beliebiger Stelle).

### 4. Deadline-Feld zu `ToDo` hinzufügen

README sagt: Sortierung nach Deadline. Aktuell hat `ToDo` nur `date` (Erstellungsdatum),
aber kein Fälligkeitsdatum. Du bräuchtest:
```java
LocalDate deadline; // Fälligkeitsdatum
```
Mit `setPriority()`-Muster: Setter mit Validierung (Deadline darf nicht in der Vergangenheit
liegen).

### 5. „Eat the Frog"-Algorithmus

Das ist das Herzstück des Projekts laut README. Idee: Die Aufgabe mit der höchsten Priorität,
die am längsten in der Liste steht, ist die „Eat the Frog"-Aufgabe.  
Formel: `score = priority * 3 + tageInListe` (oder ähnliches) → höchster Score = Frosch.

### 6. Datenpersistenz

Ohne Speicherung verlierst du alle Aufgaben beim Programmende. Einfachste Lösung:
- Aufgaben in eine `.csv`-Datei schreiben (`value,priority,datum,uhrzeit`)
- Beim Start einlesen und Liste befüllen
- Java: `BufferedWriter`/`BufferedReader` oder `Files.write()`/`Files.readAllLines()`

### 7. Einfache CLI-Eingabe (User Interface)

Aktuell hat das Programm keine `main()`-Methode. Du bräuchtest eine Schleife:
```
> add "Aufgabe" 2
> list
> delete 3
> quit
```
Umsetzbar mit `Scanner(System.in)` in einer `while(true)`-Schleife.

### 8. Knoten-Klasse aufräumen

- `todo`-Feld und `knotenAusgabe()` löschen oder reparieren
- Ungenutzten Import `java.sql.SQLOutput` in `List.java` löschen

### 9. Tests verbessern

Aktuell nutzen die Tests oft nur `System.out.println` statt `assertEquals`/`assertTrue`.
Das bedeutet: Tests laufen immer grün, auch wenn die Ausgabe falsch ist.
Tipp: `assertEquals(erwartet, tatsächlich)` verwenden.

---

## Bugs die du kennen solltest

| Datei | Zeile | Bug | Auswirkung |
|---|---|---|---|
| `Knoten.java` | ~19 | `this.todo = todo` – `todo` existiert nicht im Konstruktor | Kompilierungsfehler oder falsches Verhalten |
| `Knoten.java` | `knotenAusgabe()` | Ruft `todo.toString()` auf nicht-initialisiertem Feld auf | `NullPointerException` zur Laufzeit |
| `List.java` | `sortPositionPriority()` | Zeigerverwaltung ist fehlerhaft, Sortierung funktioniert nicht korrekt | Falsche Reihenfolge oder `NullPointerException` |
| `List.java` | Import | `java.sql.SQLOutput` importiert aber nie benutzt | Unnötiger Import (kein funktionaler Bug) |

---

## Empfohlene Reihenfolge zum Weitermachen

1. **Knoten-Bug fixen** – damit die Klasse sauber ist, bevor du darauf aufbaust
2. **`delete()` implementieren** – grundlegende Listenfunktion, kurz
3. **Merge Sort** – das war dein nächstes großes Ziel laut TODO-Kommentaren
4. **`insert()` implementieren** – dann ist die Listenstruktur vollständig
5. **Deadline-Feld hinzufügen** – bereitet „Eat the Frog" vor
6. **„Eat the Frog"-Algorithmus** – Kernfeature laut README
7. **Datenpersistenz** – ohne das verlierst du jedes Mal alles
8. **CLI-Interface** – damit das Programm nutzbar wird
9. **Tests verbessern** – auf echte Assertions umstellen

---

## Schnelleinstieg: Beispiel-Nutzung (wie der Code aktuell verwendet wird)

```java
// Aus ListTest.java – so werden Aufgaben angelegt:
List list = new List();
list.add("Wischen", 1);
list.add("Staubsaugen", 1);
list.add("Semester Gebühren bezahlen", 2);
list.add("Studenten Jobs bewerben", 3);

// Alle ausgeben:
list.ausgabe();
// Ausgabe z.B.:
// 4: Studenten Jobs bewerben (prio:3) | Erstellt am: 2026-03-13 | Uhrzeit: 14:30
// 3: Semester Gebühren bezahlen (prio:2) | Erstellt am: 2026-03-13 | Uhrzeit: 14:30
// 2: Staubsaugen (prio:1) | Erstellt am: 2026-03-13 | Uhrzeit: 14:30
// 1: Wischen (prio:1) | Erstellt am: 2026-03-13 | Uhrzeit: 14:30
```

---

*Dokumentation generiert am 25.04.2026 auf Branch `claude/document-todo-program-zxAov`.*
