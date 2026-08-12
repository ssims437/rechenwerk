# Rechenwerk

Ein Computer aus NAND-Gattern. Von einem einzigen Grundbaustein bis zu einem laufenden
Programm — und man kann bis auf das einzelne Gatter hinunterschauen, während gerechnet wird.

### → [Öffnen](https://ssims437.github.io/rechenwerk/)

Eine einzelne HTML-Datei. Kein Build, keine Bibliothek, nichts verlässt den Browser.

![Volladdierer aus dreizehn NAND, live geschaltet](bilder/volladdierer.png)

Dreizehn NAND-Gatter, die gerade `1 + 2` an Bit 0 rechnen. Helle Leitung heißt 1, dunkle 0.
Die Werte stammen aus dem Befehl, der im selben Moment ausgeführt wird.

---

## Was daran Schaltung ist

Ehrlichkeit zuerst, weil die Behauptung sonst nichts wert ist.

**Aus NAND gebaut:**

| Teil | |
|---|---|
| Rechenwerk | acht Operationen: ADD SUB AND OR XOR NOT SHL SHR |
| Addierer | acht Volladdierer in Reihe, je dreizehn NAND |
| Operationsauswahl | Multiplexerbaum aus den drei Opcode-Bits |
| Steuersignale | „subtrahieren", „Übertrag gilt" — alles Gatterlogik |
| Befehlsdecoder | vier Bit hinein, sechzehn Leitungen heraus, genau eine aktiv |

**Nicht aus NAND:** Speicher und Register sind Felder. Ein Flipflop aus rückgekoppelten
NANDs wäre möglich, verlangt aber eine Simulation, die Schleifen bis zur Ruhe iteriert.
Das steht hier, statt es zu verschweigen.

## Der Beweis auf Knopfdruck

Auf der Seite steckt ein Knopf, der das Rechenwerk **erschöpfend** prüft: acht Operationen,
je alle 65 536 Eingangspaare, gegen eine unabhängige Referenz — Ergebnis, Übertrag und
Null-Flag. Keine Stichprobe.

```
524 288 Fälle geprüft, kein einziger falsch · 1,9 s
```

## Was es kostet

Ein einziger Rechenbefehl braucht **757 NAND-Auswertungen** allein im Rechenwerk. Der
Grund ist kein Fehler, sondern die Bauart: alle acht Operationen werden immer gerechnet,
der Multiplexer wählt erst danach aus. Genau so arbeitet Hardware — parallel und
verschwenderisch, dafür in einem Takt.

Vier mitgelieferte Programme, alle gegen die erwartete Ausgabe geprüft:

| Programm | Takte | NAND-Auswertungen | Ergebnis |
|---|---|---|---|
| Fibonacci | 146 | 29 420 | 0 1 1 2 3 5 8 13 21 34 |
| Multiplikation | 110 | 27 302 | 13 × 11 = 143 |
| Quadratzahlen | 1 294 | 329 470 | 1 4 9 … 225 |
| Herunterzählen | 42 | 11 610 | 10 9 8 … 0 |

## Die Maschine

8 Bit, ein Akkumulator, 256 Byte gemeinsamer Speicher für Programm und Daten, sechzehn
Befehle mit Vier-Bit-Opcode:

```
NOP  LDA a  STA a  LDI n  ADD a  SUB a  AND a  ORA a
XOR a  JMP a  JZ a  JC a  INC  DEC  OUT  HLT
```

Der Quelltext auf der Seite ist bearbeitbar. Marken mit Doppelpunkt, `;` leitet einen
Kommentar ein, `.BYTE` legt Daten ab.

```
; Multiplikation durch wiederholtes Addieren
LDI 13
STA 200
LDI 11
STA 201
LDI 0
STA 202
schleife:
LDA 201
JZ fertig
LDA 202
ADD 200
STA 202
LDA 201
DEC
STA 201
JMP schleife
fertig:
LDA 202
OUT
HLT
```

## Vier Ebenen zum Hineinschauen

1. **Ein NAND** — zwei Eingänge, ein Ausgang, die einzige Regel
2. **XOR aus vier NAND** — zwei Bits addieren, ohne Übertrag
3. **Volladdierer aus dreizehn NAND** — drei Bits zu Summe und Übertrag
4. **Acht in Reihe** — der Übertrag wandert von rechts nach links, daher Ripple-Carry

Alle vier zeigen die Werte des gerade ausgeführten Befehls, nicht ein Lehrbuchbeispiel.

## Was mich das gekostet hat

**Ich hatte an einer Stelle geschummelt.** Die erste Fassung wählte die Operation mit
`kandidaten[op]` aus — einem JavaScript-Arrayzugriff. Der Rechenpfad war aus Gattern, der
**Steuerpfad nicht**. Das ist der Unterschied zwischen „ein Rechenwerk aus NAND" und „ein
Rechenwerk, das NAND benutzt". Nachgebaut als Multiplexerbaum stieg der Aufwand von 297
auf 757 NAND-Auswertungen je Operation — und genau dieser Preis ist die Wahrheit über
solche Hardware.

**Ein Abbruchvergleich zeigte ins Leere.** Das Quadratzahlen-Programm rechnete korrekt und
lief trotzdem in die Taktgrenze, weil ich gegen eine Speicherzelle verglich, die nie
gesetzt wurde. Die Ausgabe war richtig, das Programm nur nie fertig — der unangenehmste
Fehlertyp, weil alles Sichtbare stimmt.

## Lizenz

[MIT](LICENSE)

Verwandt: [Redundanz](https://github.com/ssims437/redundanz) ·
[Reparatur](https://github.com/ssims437/reparatur) ·
[Würfel](https://github.com/ssims437/wuerfel) ·
[Plotterblätter](https://github.com/ssims437/plotterblaetter)
