# Contesto del progetto

## Scopo

Questo file descrive il contesto generale del progetto.

Usalo per:
- comprendere il tipo di sistema
- identificare i layer e le responsabilità
- inquadrare correttamente i requisiti

---

## Overview

Applicazione enterprise per la gestione di processi applicativi.

Il sistema gestisce:
- acquisizione dati
- elaborazione
- esposizione tramite API
- interazione utente tramite interfaccia web

---

## Scope

### In scope
- gestione dati applicativi
- logiche di processo
- esposizione servizi
- interfacce utente

### Out of scope
- sistemi esterni non controllati dal progetto
- integrazioni non esplicitamente definite
- logiche non documentate nelle specifiche funzionali

---

## Stack tecnologico

- Database → Oracle Autonomous Database (ATP)
- API → ORDS (REST)
- UI → VBCS

---

## Layer logici

Il sistema è organizzato in tre layer principali:

### Data layer
Responsabile di:
- persistenza dati
- logiche PL/SQL
- integrità e vincoli

### Service layer
Responsabile di:
- esposizione API REST
- orchestrazione delle operazioni
- gestione input/output

### Presentation layer
Responsabile di:
- interfaccia utente
- interazioni
- invocazione servizi

---

## Attori e sistemi esterni

### Utenti
- utenti applicativi che interagiscono tramite UI

### Sistemi esterni
- sistemi che consumano o forniscono dati tramite API (se presenti)

---

## Tipologie di flussi

### Flussi utente
- interazione UI → API → database → risposta UI

### Flussi di servizio
- chiamate API → elaborazione → accesso dati → risposta

---

## Note

- Il comportamento funzionale è definito nelle specifiche funzionali dello stream
- Le scelte implementative sono descritte nella documentazione tecnica
- Se un requisito non è chiaro, esplicitare le assunzioni