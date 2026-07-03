# Backups – Konzepte, Verfahren und Strategien

Ein strukturierter Überblick über Datensicherungsziele, Sicherungsarten, Backup-Ebenen, Betriebsmodi und Strategien.

---

## 1. Datensicherungsziele & Datenkompression

### 1.1 Welche Medien/Datenträger/Geräte dienen als Backup-Ziel?

Ein **Backup-Ziel** (engl. *backup target*) ist der Speicherort, auf den die Sicherung geschrieben wird. Die Wahl hängt von Datenmenge, Wiederherstellungsgeschwindigkeit, Kosten, Haltbarkeit und Schutzbedarf ab.

| Medium | Typische Kapazität | Kosten/GB | Geschwindigkeit | Haltbarkeit | Typischer Einsatz |
|---|---|---|---|---|---|
| **Externe HDD (USB)** | 1–20 TB | niedrig | mittel | mittel (3–5 Jahre) | Privates Backup, kleine Firmen |
| **Externe SSD (USB)** | 0.5–4 TB | mittel | hoch | begrenzte Schreibzyklen | Schnelle mobile Sicherung |
| **NAS** (Network Attached Storage) | mehrere TB–PB | mittel | mittel–hoch | gut (RAID) | Zentrale Sicherung im Netzwerk |
| **Band / LTO Tape** | 18 TB (LTO-9) | sehr niedrig | langsam (seriell) | sehr gut (15–30 Jahre) | Langzeitarchiv, Offline-Kopie |
| **Optische Medien** (Blu-ray, M-DISC) | 25–128 GB | mittel | langsam | sehr gut (M-DISC bis 100 Jahre) | Archiv, WORM-Charakter |
| **Cloud-Speicher** (z. B. S3, Backblaze) | quasi unbegrenzt | laufende Kosten | netzabhängig | gut (redundant) | Offsite-Kopie, Desaster-Recovery |
| **Zweite interne Platte** | wie Hauptplatte | niedrig | hoch | schlecht (gleiches Gerät) | Nur bedingt Backup (kein Schutz bei Diebstahl/Blitz) |

> **Merksatz:** Eine Kopie auf demselben Gerät (oder in derselben Wohnung) ist streng genommen kein vollwertiges Backup – siehe die **3-2-1-Regel** in Kapitel 5.

**Faustregel zur Auswahl:**

- **Häufige, schnelle Restores** → SSD/HDD/NAS (schneller Direktzugriff)
- **Langzeitarchiv & Air-Gap** → Band (LTO) oder optische Medien (physisch trennbar)
- **Schutz vor Feuer/Diebstahl/Ransomware** → Cloud oder externes, ausgelagertes Medium

---

### 1.2 Kompressionsverfahren

Backups werden fast immer **verlustfrei (lossless)** komprimiert – bei einer Sicherung darf **kein Bit** verloren gehen. (Verlustbehaftete Verfahren wie JPEG oder MP3 haben im Backup nichts zu suchen.)

Der zentrale Trade-off lautet immer: **Kompressionsrate ↔ CPU-Aufwand/Geschwindigkeit**.

| Verfahren / Algorithmus | Rate | Geschwindigkeit | Typische Nutzung |
|---|---|---|---|
| **LZ4** | gering | sehr schnell | Live-Systeme, wo Tempo zählt |
| **DEFLATE / gzip / ZIP** | mittel | schnell | Universeller Standard |
| **Zstandard (zstd)** | mittel–hoch | schnell (skalierbar) | Moderner Allrounder |
| **bzip2** | hoch | langsam | Wenn Platz wichtiger als Zeit |
| **LZMA / 7-Zip (xz)** | sehr hoch | sehr langsam | Maximale Kompression, Archive |

**Verwandtes Konzept – Deduplizierung:** Statt Daten nur zu komprimieren, werden identische Datenblöcke **nur einmal** gespeichert und ansonsten referenziert. Besonders wirksam, wenn viele ähnliche Systeme oder Versionen gesichert werden (z. B. 50 fast identische VMs).

```
Ohne Dedup:   [BlockA][BlockA][BlockA][BlockB]   -> 4 Blöcke gespeichert
Mit Dedup:    [BlockA]---+----+----  [BlockB]     -> 2 Blöcke + Verweise
                         + Verweis + Verweis
```

---

### 1.3 Warum ist Kompression bei einem Image-Backup besonders wichtig?

Ein **Image-Backup** sichert eine ganze Partition/Festplatte **blockweise** – also die komplette Datenträgerstruktur inklusive Bootloader, Partitionstabelle und Dateisystemmetadaten (mehr dazu in Kapitel 3).

Das Problem: Ein Image erfasst standardmäßig auch **ungenutzte / leere Bereiche** der Platte. Eine 500-GB-Platte ergibt ohne Kompression ein 500-GB-Image – selbst wenn nur 150 GB tatsächlich belegt sind.

Hier wirkt Kompression **doppelt stark**:

1. **Leere Blöcke sind meist mit Nullen (`0x00`) gefüllt** und lassen sich extrem gut komprimieren (Rate teils >100:1).
2. **Nutzdaten** werden zusätzlich normal komprimiert.

```
500-GB-Festplatte (belegt: 150 GB, leer: 350 GB)

Ohne Kompression:   ############################  500 GB
Mit Kompression:    ######......................  ~90 GB
                    ^ Nutzdaten   ^ leere Blöcke -> fast nichts
```

**Analogie (Umzug):** Ein Image ohne Kompression ist wie ein Umzugskarton, in den man ordentlich Luft mit einpackt – man zahlt für das Volumen der leeren Blöcke mit. Kompression saugt diese „Luft" heraus, bevor gelagert oder transportiert wird.

**Konkreter Nutzen:**
- Spart massiv **Speicherplatz** auf dem Backup-Ziel
- Verkürzt die **Übertragungszeit** (wichtig bei Cloud/Netzwerk)
- Senkt die **Kosten** (weniger belegter Speicher, weniger Bandbreite)

> **Hinweis:** Viele Image-Tools kombinieren Kompression mit „Nur belegte Blöcke sichern" (skip empty / used-blocks-only), sodass leere Bereiche gar nicht erst gelesen werden.

---

## 2. Vollsicherung, Differenzielle & Inkrementelle Sicherung

### 2.1 Die drei Methoden im Überblick

| Methode | Was wird gesichert? | Basis / Bezugspunkt |
|---|---|---|
| **Vollsicherung** (Full) | **Alle** Daten, komplett | – (unabhängig) |
| **Differenzielle** Sicherung | Alle Änderungen seit dem **letzten Vollbackup** | Letztes **Full** |
| **Inkrementelle** Sicherung | Alle Änderungen seit dem **letzten Backup** (egal welcher Art) | Letztes **beliebiges** Backup |

Der entscheidende Unterschied liegt im **Bezugspunkt**: Differenziell schaut immer bis zum letzten Voll zurück, inkrementell nur bis zum unmittelbar vorherigen Backup.

---

### 2.2 Grafik: Ablauf über eine Woche

Angenommen, am **Sonntag** wird ein Vollbackup gemacht, danach täglich eine Sicherung.

```
                Mo        Di        Mi        Do        Fr
             +------------------------------------------------+
VOLL         | [==F==]  [==F==]  [==F==]  [==F==]  [==F==]     |  jeden Tag alles
             |  gross    gross    gross    gross    gross      |
             +------------------------------------------------+

             +------------------------------------------------+
DIFFERENZIELL| So=[==F==]                                      |
             | [d So>Mo] [d So>Di] [d So>Mi] [d So>Do] [d So>Fr]| waechst
             |   klein     ##        ###       ####      #####  |  taeglich
             +------------------------------------------------+
                 +-- jede Sicherung bezieht sich auf das FULL (So) --+

             +------------------------------------------------+
INKREMENTELL | So=[==F==]                                      |
             | [d So>Mo][d Mo>Di][d Di>Mi][d Mi>Do][d Do>Fr]  |
             |   klein    klein    klein    klein    klein     |  bleibt klein
             +------------------------------------------------+
                 +-- jede Sicherung bezieht sich auf den VORTAG --+

Legende:  [==F==] = Vollbackup   d = Änderungen (Delta)   # = Datenmenge
```

**Wiederherstellung (Restore) am Freitag:**

```
DIFFERENZIELL:   [==F== So]  +  [d So>Fr]              -> 2 Teile nötig
INKREMENTELL:    [==F== So]  +  [Mo][Di][Mi][Do][Fr]   -> ganze Kette nötig
```

**Analogie (Tagebuch):**
- **Voll** = Jeden Tag das ganze Buch neu abschreiben.
- **Differenziell** = „Alles notieren, was sich seit dem letzten *kompletten* Abschreiben geändert hat." Der Zettel wird täglich dicker.
- **Inkrementell** = „Nur notieren, was sich seit *gestern* geändert hat." Jeder Zettel bleibt dünn – aber man braucht am Ende alle Zettel der Reihe nach.

---

### 2.3 Wird ein Vollbackup benötigt?

**Ja – in beiden Fällen zwingend.**

Sowohl die differenzielle als auch die inkrementelle Sicherung sind **relative** Verfahren: Sie speichern nur *Änderungen* und brauchen deshalb einen **vollständigen Ausgangspunkt (Basis-Full)**, auf den sie sich beziehen. Ohne dieses Vollbackup fehlt der „Grundzustand", und die Änderungen allein ergäben kein wiederherstellbares Gesamtbild.

> **Achtung:** Deshalb beginnt **jede** Backup-Kette mit einem Vollbackup. Fällt das Full weg oder ist es beschädigt, sind auch die abhängigen Sicherungen wertlos.

---

### 2.4 Vor- und Nachteile

| Kriterium | Vollsicherung | Differenziell | Inkrementell |
|---|---|---|---|
| **Backup-Dauer** | lang | mittel (wächst) | kurz |
| **Speicherbedarf** | hoch | mittel (wächst) | gering |
| **Restore-Geschwindigkeit** | schnell (1 Teil) | mittel (Full + 1 Diff) | langsam (Full + ganze Kette) |
| **Restore-Komplexität** | einfach | einfach (2 Teile) | komplex (Kette) |
| **Ausfallrisiko der Kette** | keins | gering (nur Full + 1 Diff) | hoch: fehlt **ein** Glied, ist alles danach unbrauchbar |
| **Netzwerk-/CPU-Last** | hoch | mittel | gering |

**Kurzfazit:**
- **Vollsicherung** – teuer bei Speicher/Zeit, aber am robustesten und schnellsten wiederherstellbar.
- **Differenziell** – guter Kompromiss: einfacher Restore (nur zwei Teile), aber die Sicherungen wachsen bis zum nächsten Full.
- **Inkrementell** – am sparsamsten bei Zeit/Speicher, aber Restore braucht die **lückenlose Kette** – ein einziges defektes Glied bricht die Wiederherstellung.

> **In der Praxis** kombiniert man die Verfahren, z. B.: **wöchentlich Voll + täglich inkrementell**, oder **monatlich Voll + wöchentlich differenziell**. So balanciert man Speicher, Zeit und Restore-Sicherheit.

---

## 3. Block-Level vs. File-Level Backup

Der Unterschied liegt in der **Ebene**, auf der das Backup ansetzt: **oberhalb** des Dateisystems (Dateien) oder **unterhalb** (rohe Datenblöcke).

```
   +--------------------------------------+
   |  Anwendung / Benutzer                |
   +--------------------------------------+
   |  DATEISYSTEM   (Dateien, Ordner)     | <-- File-Level setzt HIER an
   |   dokument.pdf, foto.jpg, /home/...   |    "kopiere diese Dateien"
   +--------------------------------------+
   |  BLÖCKE / SEKTOREN (rohe Daten)      | <-- Block-Level setzt HIER an
   |   0x00 0x1A 0xFF ... Block 0,1,2,3    |    "kopiere diese Blöcke"
   +--------------------------------------+
   |  Physische Festplatte / SSD          |
   +--------------------------------------+
```

### 3.1 Begriffe & Unterschiede

**File-Level Backup** (dateibasiert)
- Arbeitet **auf Dateisystem-Ebene**: kopiert einzelne **Dateien und Ordner**.
- „Versteht" die Dateistruktur → kann gezielt auswählen (z. B. nur `.docx`).
- Restore einzelner Dateien ist trivial.
- Dateisystem-agnostisch beim Zurückspielen (Datei bleibt Datei).

**Block-Level Backup** (blockbasiert)
- Arbeitet **unterhalb des Dateisystems**: kopiert die rohen **Datenblöcke / Sektoren**.
- „Sieht" keine Dateien, nur Blöcke → erfasst **alles** exakt so, wie es auf der Platte liegt (Bit-für-Bit), inkl. Bootloader, Partitionstabelle, Metadaten, sogar Fragmentierung.
- Ermöglicht **Changed Block Tracking (CBT)**: Nur geänderte Blöcke werden für inkrementelle Sicherungen erfasst → sehr effizient.
- Für den Zugriff auf einzelne Dateien muss das Image ggf. erst **gemountet** werden.

| Merkmal | File-Level | Block-Level |
|---|---|---|
| Ebene | Dateisystem (oben) | Blöcke/Sektoren (unten) |
| Einheit | Datei/Ordner | Block/Sektor |
| Granularität Restore | einzelne Dateien direkt | ganze Images (Datei nur nach Mounten) |
| Erfasst Systemzustand (Boot, MBR/GPT) | nein | ja |
| Effiziente Inkrementelle | dateibasiert (ganze Datei bei kleiner Änderung) | nur geänderte Blöcke (CBT) |
| Selektive Auswahl (nur bestimmte Ordner) | einfach | meist ganze Partition |
| Dateisystem-Kenntnis nötig | ja | nein (arbeitet darunter) |

> **Analogie (Haus):** File-Level ist wie **einzelne Möbel** aus einem Haus einpacken (gezielt, aber man muss wissen, was wo steht). Block-Level ist wie das **ganze Haus als 3D-Scan** aufnehmen – exakt jeder Ziegel an seinem Platz, inklusive Fundament und Verkabelung, aber um einen einzelnen Löffel herauszuholen, muss man den Scan erst „betreten" (mounten).

---

### 3.2 Image einer ganzen Festplatte → **Block-Level**

Wenn ein **1:1-Image** einer Festplatte erstellt werden soll (z. B. zum Klonen eines Systems oder für Bare-Metal-Recovery), ist **Block-Level** klar überlegen:

- Erfasst die Platte **exakt Bit-für-Bit**, inkl. **Bootloader, Partitionstabelle (MBR/GPT), Dateisystemmetadaten** und aller versteckten Bereiche.
- Das wiederhergestellte System ist **sofort bootfähig** und identisch zum Original.
- Über **CBT** lassen sich anschließend effiziente inkrementelle Images ziehen (nur geänderte Blöcke).
- Ein reines File-Level-Backup würde z. B. den Bootsektor und die exakte Partitionsstruktur **nicht** erfassen → System wäre nach Restore evtl. nicht startfähig.

---

### 3.3 Nur den Home-Folder sichern → **File-Level**

Wenn lediglich der **Home-Ordner** (`/home/benutzer`) gesichert werden soll, ist **File-Level** die bessere Wahl:

- Man will **gezielt bestimmte Dateien/Ordner** sichern, nicht die ganze Platte.
- **Selektive Wiederherstellung** einzelner Dateien ist direkt und einfach (kein Mounten eines Images nötig).
- Kein unnötiges Sichern von Systemblöcken, leeren Bereichen oder fremden Partitionen.
- Restore funktioniert unabhängig vom ursprünglichen Dateisystem/Layout.

> **Zusammengefasst:** Ganze Platte / Systemabbild → **Block-Level**. Einzelne Ordner/Dateien → **File-Level**.

---

## 4. Hot Backup vs. Cold Backup

Der Unterschied bezieht sich darauf, ob das System / die Anwendung **während** der Sicherung **läuft** oder **gestoppt** ist.

| | Hot Backup (online) | Cold Backup (offline) |
|---|---|---|
| System-Zustand | **läuft weiter**, Daten in Benutzung | **gestoppt** / heruntergefahren |
| Konsistenz | muss aktiv sichergestellt werden (Snapshots, Transaktionslogs, Quiescing) | garantiert, da keine Schreibvorgänge |
| Downtime | keine | ja (während der Sicherung nicht verfügbar) |
| Komplexität | höher | einfach |

### 4.1 Hot Backup – Sicherung im laufenden Betrieb

Die Daten werden gesichert, **während** das System aktiv ist und weiter geschrieben wird. Damit die Kopie nicht inkonsistent wird (halb geschriebene Transaktionen), sind **Konsistenzmechanismen** nötig – z. B. **Snapshots**, **Transaktionslogs** oder das kurzzeitige „Einfrieren" (Quiescing) der Anwendung.

**Praktische Einsatzzwecke:**
- **24/7-Datenbank eines Online-Shops:** Der Shop darf keine Sekunde offline sein. Über einen Snapshot + Transaktionslogs wird ein konsistenter Stand gesichert, ohne den Verkauf zu unterbrechen.
- **Produktiver Virtualisierungs-Host (VMs):** Laufende virtuelle Maschinen werden per Snapshot gesichert, während Nutzer weiterarbeiten.

### 4.2 Cold Backup – Sicherung im gestoppten Zustand

Das System oder der Dienst wird **heruntergefahren bzw. gestoppt**, dann gesichert. Da niemand schreibt, ist die Kopie **garantiert konsistent** – ohne Zusatzmechanismen. Der Preis ist eine **Ausfallzeit (Downtime)**.

**Praktische Einsatzzwecke:**
- **Nächtliche Sicherung einer kritischen Finanz-Datenbank** im Wartungsfenster: Die Datenbank wird kurz gestoppt, sauber gesichert und wieder gestartet – maximale Konsistenz für sensible Buchungsdaten.
- **Privat-PC / Image im ausgeschalteten Zustand:** Ein Systemabbild wird von einem Boot-Medium (Live-USB) erstellt, während das Betriebssystem nicht läuft – so ändert sich während der Sicherung nichts.

> **Merksatz:** *Hot* = Verfügbarkeit hat Vorrang (aber Konsistenz muss erarbeitet werden). *Cold* = Konsistenz hat Vorrang (aber man akzeptiert Downtime).

---

## 5. Datensicherungsstrategie

Eine Strategie beantwortet die Fragen: **Wie oft?**, **Wohin (wie viele Medien)?** und **Wie lange aufbewahren?**

### 5.1 Wie häufig soll man ein Backup machen?

Die Antwort hängt vom **RPO (Recovery Point Objective)** ab – also davon, **wie viele Daten man maximal verlieren darf**.

```
              letztes Backup            Ausfall
                   |                       |
   ----------------o-----------------------X------>  Zeit
                   +--------- RPO ----------+
                    max. tolerierbarer Datenverlust
```

- Kleines RPO (z. B. „max. 1 Stunde Verlust") → sehr häufige Backups (stündlich, kontinuierlich).
- Großes RPO (z. B. „ein Tag ist okay") → tägliche Sicherung genügt.

**Faustregeln:**
- Wichtige, sich oft ändernde Daten → **täglich** oder häufiger.
- Weniger kritische Daten → **wöchentlich**.
- Merksatz: *„Ein Backup ist so viel wert wie die Arbeit, die man ohne es verlieren würde."* Man wähle das Intervall danach, wie viel Neuerstellung man verkraftet.

*(Verwandt: **RTO – Recovery Time Objective** = wie schnell muss man wieder lauffähig sein. Beeinflusst Medienwahl und Backup-Art, siehe Kapitel 2 & 4.)*

---

### 5.2 Auf wie vielen unterschiedlichen Datenträgern? → Die **3-2-1-Regel**

Die bewährteste Faustregel der Datensicherung:

```
        +----------------- 3-2-1-REGEL -----------------+
        |                                               |
        |   3   Kopien der Daten insgesamt              |
        |       (1 Original + 2 Backups)                |
        |                                               |
        |   2   verschiedene Medien-/Speichertypen      |
        |       (z. B. interne SSD + externe HDD)       |
        |                                               |
        |   1   Kopie ausgelagert (offsite / offline)   |
        |       (z. B. Cloud oder anderer Standort)     |
        |                                               |
        +-----------------------------------------------+
```

**Beispiel für einen privaten PC:**

| # | Kopie | Medium | Ort |
|---|---|---|---|
| 1 | Original | interne SSD | zu Hause |
| 2 | Backup A | externe HDD | zu Hause (anderer Datenträgertyp) |
| 3 | Backup B | Cloud-Speicher | ausgelagert / offsite |

**Warum das so wirkt:**
- **3 Kopien** → ein einzelner Defekt vernichtet nie alles.
- **2 Medientypen** → schützt vor systematischen Fehlern eines Medientyps.
- **1 offsite** → schützt vor Feuer, Diebstahl, Blitzschlag, Ransomware (physisch/logisch getrennt).

> **Weiterentwicklung – 3-2-1-1-0:** zusätzlich **1** Kopie *offline/air-gapped* (nicht erreichbar für Ransomware) und **0** Fehler bei der Wiederherstellungs-Prüfung. → **Restore regelmäßig testen!** Ein ungetestetes Backup ist nur eine Hoffnung.

---

### 5.3 Rotations- und Aufbewahrungsschemata

Bei mehreren Medien stellt sich die Frage, **welches Medium wann** (wieder-)verwendet wird, um sowohl aktuelle Stände als auch ältere Versionen vorzuhalten – bei begrenzter Medienzahl.

**Turm von Hanoi**
- Benannt nach dem gleichnamigen mathematischen Puzzle.
- Die Medien werden nach dem Muster der Hanoi-Scheiben rotiert: Manche Medien werden **häufig**, andere in **exponentiell größer werdenden Abständen** überschrieben.
- Ergebnis: Mit **wenigen Medien** deckt man einen **langen Zeitraum** ab – man hat sowohl sehr aktuelle als auch überraschend alte Wiederherstellungspunkte.
- Kompromiss: gute Retention (Rückgriffstiefe) bei sparsamer Medienzahl, dafür etwas komplexer im Handling.

```
Beispiel (5 Medien A-E, Hanoi-artige Frequenz):

Tag:   1  2  3  4  5  6  7  8  9 10 11 12 ...
Band:  A  B  A  C  A  B  A  D  A  B  A  E ...
       ^ A jeden 2. Tag   ^ B jeden 4.   ^ C jeden 8. ... (Abstände verdoppeln sich)
```

**GFS (Grandfather-Father-Son)** – oft gemeinsam genannt:
- **Son** = tägliche Backups, **Father** = wöchentliche, **Grandfather** = monatliche.
- Einfaches, weit verbreitetes Schema für gestaffelte Aufbewahrung (täglich/wöchentlich/monatlich).

> **Begriffe zum Vertiefen:** *Turm von Hanoi (Rotationsschema)*, *3-2-1-Regel*, *GFS-Schema*, *RPO/RTO*, *Air-Gap*, *Immutable Backups*.

---

## Zusammenfassung auf einen Blick

| Thema | Kernaussage |
|---|---|
| **Backup-Ziele** | Medium nach Kosten/Tempo/Haltbarkeit/Schutz wählen; niemals nur auf demselben Gerät. |
| **Kompression** | Verlustfrei; Trade-off Rate ↔ Tempo; bei Images essenziell (leere Blöcke komprimieren enorm). |
| **Voll/Diff/Inkr.** | Diff & Inkr. brauchen immer ein Basis-Full. Diff = einfacher Restore, Inkr. = sparsamer, aber Kettenrisiko. |
| **Block vs. File** | Block = ganze Platte/Image (bootfähig, CBT). File = einzelne Ordner/Dateien (granular, selektiv). |
| **Hot vs. Cold** | Hot = kein Downtime, braucht Konsistenzmechanismen. Cold = Downtime, aber garantiert konsistent. |
| **Strategie** | Frequenz nach RPO; **3-2-1-Regel**; rotieren (Hanoi/GFS); **Restores testen!** |
