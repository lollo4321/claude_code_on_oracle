# Constraints

## Regole sui dati

- Ogni tabella deve includere le WHO COLUMNS (audit standard)
- Ogni tabella deve includere una colonna ID che consente il collegamento gerarchico
- Salvare sempre i codici nelle tabelle, mai le descrizioni
- I dati grezzi non transitano mai direttamente nelle strutture definitive: passano sempre per staging

---

## Regole architetturali

- VBCS non accede mai direttamente alle tabelle ATP: sempre e solo tramite moduli ORDS
- Ogni schema ATP ha responsabilità distinta: non usare uno schema fuori dal suo perimetro
- I moduli ORDS espongono esclusivamente POST (procedure PL/SQL) o GET (select): nessun altro metodo

---

## Regole operative

- Ogni operazione sui dati deve essere tracciabile
- Creare package PL/SQL per raggruppare procedure e funzioni relative allo stesso modulo applicativo
- Le funzioni e procedure riutilizzabili tra più applicazioni vanno centralizzate in INT_ADMIN
- Definire indici su colonne ID (univoci) e su colonne chiave di processo/applicazione
- Definire sequenze per le colonne ID

---

## Divieti espliciti

- NON scrivere direttamente nelle strutture definitive: passare sempre per staging
- NON usare APEX per nuove UI: il look&feel standard è Redwood su VBCS
- NON esporre servizi ORDS su grandi volumi di dati: usare una vista con condizioni WHERE
- NON duplicare funzioni/procedure tra schemi: centralizzare in INT_ADMIN

---

## Priorità in caso di conflitto

- Se **performance** è in conflitto con **integrità del dato** → privilegia integrità
- Se **semplicità implementativa** è in conflitto con **tracciabilità** → privilegia tracciabilità
- Se **accesso diretto ATP** è in conflitto con **mediazione ORDS** → privilegia sempre ORDS
