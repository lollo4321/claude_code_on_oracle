# SKILL: Architect — Produzione LLD

## Fase 1 — Lettura e comprensione

1. Leggi il documento funzionale integralmente
2. Leggi `project-context.md`
3. Leggi `naming-conventions.md`
4. Identifica i processi descritti nel funzionale e mappali sui layer del sistema (DB → ORDS → VBCS)
5. Identifica le ambiguità che impattano strutture dati, contratti API o logiche core
6. Se esistono ambiguità critiche, **fermati e fai le domande necessarie** prima di procedere — una domanda alla volta, in ordine di priorità
7. Documenta le assunzioni fatte per ambiguità a basso impatto

---

## Fase 2 — Produzione LLD

Produci il file `LLD.md` rispettando esattamente la struttura seguente.

---

### Sezione 1 — Architettura del processo

Descrivi il flusso complessivo dal backend al frontend.

- Rappresenta il flusso come sequenza numerata di step
- Per ogni step indica: componente coinvolto, azione, output verso il componente successivo
- Evidenzia i punti di confine tra layer (DB → ORDS, ORDS → VBCS)

Esempio di formato:
```
1. [DB] La procedura X elabora i dati in staging e li persiste nella tabella Y
2. [ORDS] Il modulo Z espone la procedura X come endpoint POST
3. [VBCS] La pagina W chiama l'endpoint POST e visualizza il risultato
```

---

### Sezione 2 — Database

Per ogni elemento DB elenca:

**Tabelle**
| Nome | Schema | Descrizione |
|------|--------|-------------|

**Package**
| Nome | Schema | Descrizione |
|------|--------|-------------|

**Procedure / Funzioni**
| Nome | Package | Descrizione | Input | Output |
|------|---------|-------------|-------|--------|

Regole:
- I nomi devono essere conformi a `naming-conventions.md`
- La descrizione deve spiegare cosa fa l'elemento e a cosa serve nel processo
- Non scrivere DDL o codice PL/SQL

---

### Sezione 3 — ORDS

Per ogni modulo elenca:

**Moduli**
| Nome modulo | Base path | Schema |
|-------------|-----------|--------|

**Endpoint**
| Modulo | Template | Metodo | Procedura/Select | Input | Output | Descrizione |
|--------|----------|--------|-----------------|-------|--------|-------------|

Regole:
- Metodo: solo POST (procedure PL/SQL) o GET (select)
- Input e Output in formato sintetico — solo nome e tipo dei parametri principali
- La descrizione deve chiarire cosa fa l'endpoint dal punto di vista del processo

---

### Sezione 4 — VBCS

Per ogni pagina elenca:

**Pagine**
| Nome pagina | Descrizione | Endpoint ORDS chiamati |
|-------------|-------------|----------------------|

**Componenti principali**
| Pagina | Componente | Tipo | Descrizione |
|--------|------------|------|-------------|

Regole:
- Gli endpoint ORDS referenziati devono corrispondere a quelli definiti nella Sezione 3
- Non descrivere dettagli di implementazione UI — solo struttura e responsabilità
- Se una pagina chiama più endpoint, elencali tutti

---

### Sezione 5 — Assunzioni e punti aperti

**Assunzioni fatte:**
- [assunzione 1]

**Punti aperti da risolvere prima dello sviluppo:**
- [punto aperto 1]

---

## Regole generali

- Non produrre codice in nessuna sezione
- Non aggiungere elementi non presenti nel funzionale
- Se un elemento non è determinabile dal funzionale, inseriscilo nei punti aperti
- I nomi di tutti gli elementi devono essere conformi a `naming-conventions.md` prima di essere scritti nell'LLD
