Questo file descrive la struttura del repository e dove devono essere collocati codice e documentazione.

# Struttura del repository

```
repo/
├── context/
├── agents/
├── docs/
│   ├── common/
│   │   └── db_tables_registry.md
│   └── {stream}/
│       ├── functional/
│       └── technical/
│           ├── {stream}_assumptions.md
│           ├── {stream}_install_guide.md
│           ├── db/
│           ├── ords/
│           └── vbcs/
└── src/
    └── {stream}/
        ├── db/
        ├── ords/
        └── vbcs/
```

## `context/` e `agents/`

Cartelle del framework di sviluppo.
Non devono essere modificate salvo esplicita evoluzione del framework.

## `docs/`

- `common/` — documentazione trasversale condivisa tra più stream
  - `db_tables_registry.md` — registro centralizzato di tutte le tabelle del database
- `{stream}/functional/` — source of truth per logica funzionale
- `{stream}/technical/` — documentazione tecnica dello stream
  - `{stream}_assumptions.md` — assunzioni tecniche adottate durante lo sviluppo
  - `{stream}_install_guide.md` — guida di installazione e rilascio
  - `db/` — documentazione tabelle e PL/SQL
  - `ords/` — documentazione moduli ed endpoint ORDS
  - `vbcs/` — documentazione pagine, variabili, action chain e integrazioni VBCS

## `src/`

- `{stream}/db/` — DDL, package PL/SQL, grants
- `{stream}/ords/` — definizioni moduli ORDS ed endpoint
- `{stream}/vbcs/` — pagine, file html/css/json, action chain

## Regole

- Il codice va sempre in `src/`, la documentazione in `docs/`
- Ogni artefatto deve essere collocato nello stream e layer corretto
- Non creare nuove cartelle se esiste già una struttura coerente
- Prima di creare un nuovo file, verifica se esiste già un file da aggiornare
- Documentazione e implementazione devono rimanere allineate