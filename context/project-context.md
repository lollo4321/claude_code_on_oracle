# Project Context

## Natura del sistema

- **Paradigma dominante:**
  Sistema orientato a flussi di dati strutturati. Non è CRUD puro: i dati transitano per fasi intermedie prima della persistenza definitiva.

- **Fasi del dato:**
  Acquisizione → Staging → Trasformazione/elaborazione (PL/SQL) → Persistenza definitiva → Esposizione (ORDS → VBCS)

- **Separazioni architetturali critiche:**
  - Dati grezzi (ETL_SOURCE) vs dati elaborati (STAGING_PAAS) vs dati consolidati
  - Ogni schema ATP ha responsabilità distinta e non deve essere usato fuori dal suo perimetro
  - Le UI VBCS non accedono mai direttamente alle tabelle: passano sempre tramite ORDS

---

## Processi critici

- **Elaborazione dati contabili e amministrativi:** i dati transitano per staging, vengono aggregati/normalizzati tramite PL/SQL e persistiti nelle strutture definitive. Integrità e tracciabilità di ogni operazione sono non negoziabili.
- **Estensioni SaaS tramite VBCS:** le UI custom estendono Oracle Fusion per funzionalità non coperte dallo standard. Le chiamate VBCS verso ATP avvengono esclusivamente tramite moduli ORDS (POST per procedure PL/SQL, GET per select).

---

## Vocabolario del dominio

- `Staging`: area intermedia di transito dati prima della persistenza definitiva
- `ORDS`: Oracle REST Data Services — espone procedure PL/SQL e SELECT dell'ATP come endpoint REST
- `Module ORDS`: oggetto che espone un endpoint REST sull'ATP; le chiamate VBCS usano solo POST (procedure) o GET (select)
- `WHO COLUMNS`: colonne di audit standard obbligatorie in ogni tabella
- `VBCS`: Visual Builder Cloud Service — piattaforma UI per le estensioni PaaS

---

## Perimetro decisionale

**Il sistema fa:**
- Staging, trasformazione e consolidamento di dati contabili e amministrativi su ATP
- Esposizione di servizi REST tramite ORDS verso VBCS
- UI custom su VBCS per funzionalità non coperte dallo standard Oracle Fusion

**Il sistema NON fa:**
- Le UI VBCS non accedono direttamente alle tabelle ATP — sempre e solo tramite ORDS
- VBCS non usa APEX — il look&feel standard è Redwood
- Su grandi volumi di dati non si usano servizi ORDS: si preferisce una vista con condizioni di WHERE