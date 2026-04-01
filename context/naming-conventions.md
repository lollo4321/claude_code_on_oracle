# Naming Conventions

## Schema ATP

- `ETL_SOURCE`: dati grezzi in ingresso da elaborare tramite ODI
- `STAGING_PAAS`: estensioni applicative e dati post-elaborazione ETL
- `STAGING_SAAS`: dati provenienti da Oracle Fusion (anagrafiche, fbdi, lookup)
- `INT_ADMIN`: schema con privilegi DBA — viste, procedure, package e funzioni condivise tra più schemi
- `DW_OAC`: dati destinati a OAC, disaccoppiati dai dati transazionali

---

## Naming oggetti ATP

**Struttura:**
`[Modulo]_[Applicazione]_[Funzione]_[Gerarchia]`

| Blocco | Descrizione | Lunghezza |
|--------|-------------|-----------|
| Modulo | Prefisso dell'area funzionale | 4-5 char |
| Applicazione | Nome dell'applicazione di riferimento | 5-10 char |
| Funzione | Operazione (se presente) | 3 char |
| Gerarchia | Posizione nella struttura dati | 3 char |

**Valori ammessi — Modulo:**

| Prefisso | Area |
|----------|------|
| `INFND` | Oggetti common o provenienti dallo schema FND del SaaS |
| `ININT` | Integrazioni |
| `INAP` | Payables |
| `INPO` | Procurement |
| `INAR` | Ciclo attivo |
| `INGL` | Contabilità / General Ledger |
| `INCE` | Cash Management |

**Valori ammessi — Funzione:**
`INS`, `UPD`, `DEL` (se presente)

**Valori ammessi — Gerarchia:**

| Valore | Significato |
|--------|-------------|
| `H` / `HEADER` / `T` / `TESTATA` | Testata |
| `L` / `LINEE` | Linee |
| `DET` | Dettaglio |
| `V` | Vista |
| `STG` | Tabella di staging |
| `IN` | Input |
| `OUT` | Output |
| `IND` | Indice |
| `S` / `SEQ` | Sequenza |

---

## Suffissi per tipo oggetto

| Tipo | Sottotipo | Suffisso |
|------|-----------|---------|
| TABLE | Temporary | `_TMP` |
| VIEW | | `_V` |
| INDEX | Primary Key | `_PK` |
| | Unique Key | `_UK` |
| | Foreign Key | `_FK` |
| | Other | `_IND` |
| PACKAGE | | `_PKG` |
| SEQUENCE | | `_SEQ` oppure `_S` |
| TRIGGER | | `_TRG` |
| TYPE | Object Type | `_OT` |
| | Collection Type | `_CT` |
| | Type Body | `_BT` |
| MATERIALIZED VIEW | | `_MV` |
| MATERIALIZED VIEW LOG | | `_MVL` |
| DATABASE LINK | | `_DBL` |

---

## Naming moduli ORDS

<!-- Convenzione per il nome dei moduli e dei template esposti tramite ORDS -->
<!-- Da definire a livello di progetto -->

-
-

---

## Naming componenti VBCS

<!-- Convenzione per il nome di pagine, action chain, service connection -->
<!-- Da definire a livello di progetto -->

-
-
