## Vorlesung 3 Resumé

SRAM/DRAM

C-Syntax
- Endians
  - lib stdint.h

## VL4
### HAL API
#### HAL_ADC
- example Selector in Projekterstellung in CUBE-IDE

Conversion via Polling
- ADC Start Conversion
- Warten auf ADC-Ready-Flag
- Abfragen der Value-Register

- ADC1->CR2 |= ADC_CR2_ADON; // ADC einschalten


#### C-Pointer
```c
int *a; // Zeiger auf Int
int *b = NULL; // Initialisierung auf NULL
int c = 42; // Variable c mit Wert 42
a = &c; // a zeigt auf c
*a = 33; // c ist über Zeiger auf 33 gesetzt
```

```c
typedef struct {
  char name[163];
  int a;
  int b*;
} data_t;

data_t data; 
data.a = 42; // initialisieren
data_t *ptr_data = &data;

// Zeiger dereferenzieren
ptr_data->a = 42; // (*ptr_data).a = 42
```


Pointer typen
- int -> int*
- void -> void*

Datenverfälschung durch falsche casts
```c
int x;
void *vptr = (void*)&x;
char *y = (char*) vptr; // casten von int* (4 Byte) auf char* (1 Byte) -> 4x so viele Elemente
*y = 42;
```


```c
int x;
void *vptr = (void*)&x;
int *y = (int*)vptr;
*y = 0xDEADFACE;
```

Verwendung von Voidpointern zum Arbeiten mit unbekannten Datentypen 
(Nich so schön)
```c
#include <stdint.h>
void licht_an(void* params) {
  uint_16* ptr_LEDs = (int*) params; // Annahme das ein Array von integern übergeben wird
  for (int i = 0; i < 1000; i++) {
    // anschalten(); // Setzen von RGB werten (zB)
    ptr_LEDs[i] = 0xFFFFFFFF; // RGBA
  }
  // some code
}
```
alternativ, übergeben von parametern als struct:
```c

#define NLAMPS
typedef Struct {
  uint32_t nlamps;
  uint32_t lamps[NLAMPS];
}

void licht_an(void* params) {
  data_t* ptr_data = (data_t*) params;
  for (int i = 0; i < ptr_data->nlamps; i++) {
    ptr_data->lamps[i] = 0xFFFFFFFF;
  }
}
```

Problem:
Größe des Voidpointer ist systemabhängig
```c
int a;
int size = sizeof(a); // 4

sizeof(void*); // ? - Architekturbreite, typischerweise 32 oder 64 bit, aber auch 16 oder 8 bit
```

Header File task.h
- kreieren von Tasks
- benutzt void* um Parameter zu übergeben

---

#### C und OOP
Objekttyp in C
```c
typedef struct {
  // some value
  // some function pointers 
} data_t;

data_t data;
```
##### OOP Prinzipien
- Vererbung
- Polymorphismus
- Kapselung
- Abstraktion

möglich zu implementieren in C, durch inherieren von structs mit entsprechenden Daten und Funktionszeigern
##### Designated Initializer
```c
typedef struct {
  bool on;
  int size;
} data_t;

// designated initializer
data_t data = {
  .on = true,
  .size = 42
};

// oder auch
data_t data_2 = {true, 31}; // keine Zuordnung, nur Reihenfolge
```


#### C-Schlüsselwörter

extern
- Globale Variablen
- Deklaration einer Variable oder Funktion mit nur einer Defintion im Projekt
  - Projektstruktur zb
    - modul1.h
    - modul1.c
    - modul2.h
    - modul2.c
      - ```c
        #include <globals.h>
        // modul2.c kann global_var verwenden
        ```
    - main.h
    - main.c
    - globals.h
      - ```c
        extern int global_var; // global verfügbar machen
        ```
    - globals.c
      - ```c
        #include <globals.h>
        int global_var;
        ```


volatile
- flüctige Speicherbereiche
- verhindert Optimierung von Compilern
- Wert der Variable ist flüchtig und unabhängig vom Code
- Addressraum ist nicht nur SRAM oder Flash, sondern auch Register für Peripherieoperationen
- shared variables beim multithreading
```c
volatile int a;
```

register (obsolet)
- HInweis auf stark genutzte Variablen
- Compiler kann diese Variablen in CPU-Register legen (typischerweise SRAM strukturen)

#### C-Literale
Englisch Literal -> Wörtlich
Schriftliche Darstellung der Werte von Datentypen im code
- 0x... -> Hexadezimal
- 0b... -> Binär
- 0... -> Oktal
- 0...f -> Float
- 0...l -> Long
- 0...u -> Unsigned
- 0...ul -> Unsigned Long
- 0...ull -> Unsigned Long Long

## Mikroprozessortechnik Grundlagen

### CPU
- Central Processing Unit
- Ausführung von Programmen durch befolgen von Arbeitsinstruktionen
- früher NAND/NOR Gates, heute Transistoren
- eingebettet mit Peripherie in einem Chip -> microcomputer
  
- Instruktionen / Daten
  - Binär speziell codierte Bitabfolgen
  - Instruktionen weisen CPU an, bestimmte Daten zu verarbeiten
- Bus
  - Sammlung von mehreren elektrischen Leitungen
  - z.B. Datenbus (Daten), Adressbus (Datenspeicheradressen), Steuerbus (Lesen/Schreiben)
  - senden von digitellem Signalen, an all die Komponenten die an den Bus angeschlossen sind
  - Busse sind bidirektional

#### Architekturen
- Harvard
  - getrennte Speicher für Daten und Instruktionen
  - getrennte Busse für Daten und Instruktionen
  - Programm kann sich nicht selbst verändern
  - Programm kann nicht selbstständig auf Daten zugreifen
  - gut bei Sicherheitsrelevanten Anwendungen
  - ESP32 ist modifizierte Havard Architektur
- von Neumann
  - gemeinsamer Speicher für Daten und Instruktionen (keine physikalische Trennung)
  - gemeinsamer Bus für Daten und Instruktionen

CPU Register
- Speicherbereiche in der CPU
- sehr schneller Zugriff
- genereller Operationsmodus
  - Endlosschleife (endloses durchlaufen von z.B einer while(true) Schleife)
- typische Instruktionen
  - arithmetische Operationen
  - logische Operationen
  - Speicherzugriffsoperationen
  - Sprungoperationen
  - Interruptoperationen
  - etc.
- Register sind sehr schnell, aber auch sehr teuer(?)
- Operationscode opcode
  - Instruktionen werden durch Opcodes codiert
  - z.B. ADD, SUB, MUL, DIV, AND, OR, XOR, NOT, LOAD, STORE, JUMP, etc.
  - stellen bitpattern dar, die operationen einzigartig identifizieren
- Assembler
  - Programmiersprache, die direkt in Maschinencode übersetzt werden kann
  - Assemblerbefehle sind 
    - direkt mit Opcodes verknüpft (Mnemonics)
    - sehr hardwarenah
    - plattformspezifisch

```asm
; beispiel
mov r0, #0x42
add r0, r0, #0x1	
```