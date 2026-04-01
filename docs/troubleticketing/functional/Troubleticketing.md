**Specifica funzionale & tecnica -- Pagina VBCS "Rilevazione
Bug/Incident UAT" (PaaS + SaaS)**

**0. Obiettivo**

Realizzare in **Oracle VBCS** una soluzione (set di pagine) per la
**raccolta e gestione delle segnalazioni UAT** (bug/incident/migliorie),
**deployabile una sola volta** e utilizzabile sia per:

-   **SaaS** (es. Oracle Fusion Applications)

-   **PaaS** (app custom, incluse schermate VBCS o altri front-end)

La soluzione deve supportare:

-   Inserimento segnalazione strutturata (contesto + riproducibilità +
    evidenze)

-   Allegati (screenshot/log)

-   Triage e assegnazione

-   **Ciclo di vita con "Soluzione proposta", "In re-test", "Risolto",
    "Chiuso"**

-   Notifiche agli attori coinvolti

-   Export e ricerca per facilitare UAT

**1) Layout "pixel‑ready" (VBCS UI) --- Pagina 1: Issue List (UAT)**

**Obiettivo**: vista elenco con filtri rapidi, ricerca, ordinamento,
paging e CTA per creare segnalazioni.

**1.1 Struttura a griglia**

-   **Container** (max‑width 1280px, padding 16/24 px)

-   **Row 1 --- Header**

    -   Col(8): **Titolo pagina**: "Segnalazioni UAT"

    -   Col(4), right‑aligned: **Pulsanti**: Nuova segnalazione
        (primary), Export CSV (secondary)

-   **Row 2 --- Filtri**

    -   Card "Filtri" (elevation 1, padding 12px):

        -   Col(3): **Domain/Stream** → Select Single (LOV: CONTABILITA,
            PROCUREMENT)

        -   Col(2): **Environment** → Select Single (LOV: SVI, CALL)

        -   Col(2): **Solution Type** → Select Single (SaaS, PaaS)

        -   Col(2): **Status** → Multi‑Select (LOV status)

        -   Col(2): **Severity** → Multi‑Select (BLOCKER/HIGH/...)

        -   Col(1): **Reset** → Button ghost

        -   Sotto‑riga: Col(12): **Ricerca testo** → Input Text (q) con
            icona search (placeholder: "Cerca in titolo...")

-   **Row 3 --- Tab/Quick views (opzionale)**

    -   Tabs: "Tutte", "Le mie", "Da re‑testare", "Alte priorità"

-   **Row 4 --- Tabella**

    -   **Table** (stretch, 8--12 righe per pagina; server‑side paging)

        -   Colonne:

            -   **ID** (link a dettaglio)

            -   **Titolo** (max 1 riga + tooltip su hover)

            -   **Stream** (badge: Contabilità/Procurement)

            -   **Env** (SVI/CALL)

            -   **Type** (Bug/Incident/...)

            -   **Severity** (chip colori:
                Blocker=Red/High=Orange/Medium=Gold/Low=Grey)

            -   **Status** (pill color by stato)

            -   **Assignee** (avatar+nome se disponibile)

            -   **Updated** (relative time + tooltip data/ora)

            -   **Azioni riga**: "Apri", "Cambia stato" (solo Triage),
                "Duplica"

    -   Footer: Paginazione (server‑side), PageSize (20/50/100)

-   **Sticky footer** (mobile): Nuova segnalazione (FAB)

**1.2 Colori e tipografia (stile Redwood)**

-   **Titoli**: H1/H2 Redwood

-   **Badge/Pill**: palette Redwood; Severity più visibili
    (rosso/arancio/giallo/grigio)

-   **Stato**:

    -   NEW: info/blue

    -   TRIAGE: indigo

    -   IN_PROGRESS: teal

    -   SOLUTION_PROPOSED: amber

    -   RETEST: purple

    -   RESOLVED: green

    -   CLOSED: neutral/grey

**1.3 Binding e azioni**

-   Filtri → bind ai query‑params di GET /issues (status=...,
    severity=..., domainStream=..., environment=..., solutionType=...,
    q=...).

-   Sorting → icone colonna (mappate su sort=field:dir)

-   Paginazione → page, pageSize

-   Bottoni:

    -   Nuova segnalazione → naviga a Page 2 (Create)

    -   Export CSV → chiama endpoint export (opzionale) o usa i dati
        correnti della tabella e genera CSV client‑side.

**2) Layout "pixel‑ready" (VBCS UI) --- Pagina 2: Issue Create/Edit
(Segnala)**

**Obiettivo**: form chiaro in 4 card, con validazioni e helper text.

**2.1 Struttura a griglia**

-   **Header**: Titolo "Nuova segnalazione UAT", breadcrumb ("Home /
    Segnalazioni / Nuova")

-   **Card A --- Contesto (obbligatoria)**

    -   **Solution Type**: Radio (SaaS, PaaS) *(required)*

    -   **Application/Module**: Select (LOV interna progetto)
        *(required)*

    -   **Environment**: Select (SVI, CALL) *(required)*

    -   **Domain/Stream**: Select (CONTABILITA, PROCUREMENT)
        *(required)*

    -   **Use Case**: Combo (lookup GET /usecases?domainStream=...) ---
        mostra useCaseTitle ma salva useCaseCode

    -   **URL/Pagina**: Input Text (validazione URL, opzionale)

    -   **Tenant/Pod** (se SaaS visibile)

    -   **App Version** (se PaaS visibile)

-   **Card B --- Descrizione (obbligatoria)**

    -   **Titolo**: Text (max 120) *(required)*

    -   **Record Type**: Select (Bug, Incident, Miglioria, Chiarimento,
        Dato, Accesso) *(required)*

    -   **Severity**: Select (BLOCKER/HIGH/MEDIUM/LOW) *(required)*

    -   **Priority**: Select (P1/P2/P3) *(opzionale ma consigliata)*

    -   **Frequenza**: Select (Sempre/Spesso/Raramente/Prima volta)

    -   **Impatti**: Checkbox group (Processo bloccato / Dati /
        Performance / UX / Compliance)

    -   **Comportamento atteso**: TextArea *(required)*

    -   **Comportamento osservato**: TextArea *(required)*

    -   **Messaggio errore**: TextArea *(opzionale)*

-   **Card C --- Riproducibilità (obbligatoria)**

    -   **Prerequisiti**: TextArea

    -   **Passi per riprodurre**: TextArea *(required)*

    -   **Riproducibile**: Switch (Sì/No); se No mostra **Note
        aggiuntive** (TextArea obbligatoria)

-   **Card D --- Evidenze**

    -   **Allegati**: File Picker (multi) con elenco file (nome/size)

    -   **Riferimenti transazione**: input(s) testuali (es. numero
        documento)

-   **Footer azioni**

    -   Salva bozza (status=Draft)

    -   Invia (status=NEW + notifica)

    -   Annulla (torna a lista)

**2.2 Validazioni UI (prima del submit)**

-   Required su campi indicati

-   Warning se severity ∈ {BLOCKER,HIGH} e **nessun allegato**

-   Se reproducible = No → **Note aggiuntive obbligatorie**

-   Se recordType = Dato → richiedi almeno un **riferimento
    transazione**

**3) Layout "pixel‑ready" (VBCS UI) --- Pagina 3: Issue Detail**

**Obiettivo**: leggere i dati, allegare evidenze, commentare e pilotare
**workflow** (propose solution → re‑test → resolved → close).

**3.1 Struttura a schede**

-   **Header**:

    -   H1: #ID -- Titolo

    -   Sub: chips di **Status**, **Severity**, **Env**, **Stream**,
        **SolutionType**

    -   Azioni contestuali (in alto a destra, condizionate da
        ruolo+stato):

        -   Proponi soluzione (da IN_PROGRESS)

        -   Invia in re‑test (da SOLUTION_PROPOSED)

        -   Conferma risoluzione (PASS) (da RETEST)

        -   Esito negativo / Riapri (FAIL) (da RETEST)

        -   Chiudi (da RESOLVED)

-   **Tabs**:

    1.  **Dettaglio** (read‑only + alcuni campi editabili da
        Triage/Lead)

    2.  **Allegati** (lista + upload)

    3.  **Commenti** (timeline con chi/quando; internal/public)

    4.  **Workflow & Audit** (card con campi soluzione, re‑test,
        closure + storico stati)

**3.2 Contenuti tab "Workflow & Audit"**

-   **Card: Soluzione proposta** (visibile da IN_PROGRESS e quando
    SOLUTION_PROPOSED)

    -   **Resolution Notes** *(required quando proponi soluzione)*

    -   **Fix Version** *(obbligatorio se PaaS)*

    -   **Fix Environment** (SVI/CALL)

    -   **CTA**: Proponi soluzione

-   **Card: Re‑test**

    -   **Istruzioni re‑test** (textarea)

    -   **Tester assegnato**: read‑only (owner use case)

    -   **CTA**: Invia in re‑test

    -   Se status=RETEST visualizza:

        -   **Esito**: radio (PASS/FAIL)

        -   **Note esito** *(required se FAIL)*

        -   CTA: Conferma risoluzione (PASS) / Esito negativo / Riapri
            (FAIL)

-   **Card: Chiusura**

    -   **Closure Reason** (LOV) *(required per close)*

    -   **Evidence link** (opzionale)

    -   CTA: Chiudi

-   **Card: Storico stati (timeline)**

    -   Lista: data/ora -- old → new -- changedBy -- note

**4) Wireframe testuale (ASCII) --- Issue Create/Edit**

Plain Text

\[Header\] Nuova segnalazione UAT \[Salva bozza\] \[Invia\] \[Annulla\]

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\| Card A: Contesto \[ \] \|

\| ( ) SaaS ( ) PaaS Application/Module: \[\...\...\...\...\...\.....\]
\|

\| Environment: \[SVI ▼\] Domain/Stream: \[CONTABILITA ▼\] \|

\| Use case: \[UC_ACC_001 ▼\] URL/Pagina:
\[https://\...\...\...\...\...\....\] \|

\| (SaaS) Tenant/Pod: \[\....\] (PaaS) App Version: \[\....\] \|

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\| Card B: Descrizione \* \|

\| Titolo:
\[\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\]
\|

\| Type: \[Incident ▼\] Severity: \[High ▼\] Priority: \[P1 ▼\] Freq:
\[Sempre ▼\] \|

\| Impatti: \[x\] Processo bloccato \[ \] Dati \[ \] Performance \[ \]
UX \[ \] Compliance \|

\| Comportamento atteso:
\[\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\.....\]
\|

\| Comportamento osservato:
\[\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\.....\]
\|

\| Messaggio errore:
\[\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\.....\]
\|

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\| Card C: Riproducibilità \* \|

\| Prerequisiti:
\[\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\]
\|

\| Passi per riprodurre:
\[\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\....\]
\|

\| Riproducibile: (x) Sì ( ) No (se No) Note aggiuntive:
\[\...\...\...\.....\] \|

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\| Card D: Evidenze \* \|

\| \[ + Add files \] \|

\| Riferimenti transazione:
\[\...\...\...\...\...\...\...\...\...\...\...\...\...\...\...\.....\]
\|

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

Footer: \[Salva bozza\] \[Invia\] \[Annulla\]

Mostra più linee

**5) Mappatura componenti VBCS (per costruire rapido)**

-   **Containers & Layout**:

    -   *oj-flex* / *oj-panel*: per righe/colonne responsive

    -   *oj-tab-bar* (opzionale) o *oj-tabs* per quick views

-   **Form**:

    -   *oj-form-layout* per form auto‑responsive

    -   *oj-input-text*, *oj-text-area*, *oj-select-single*,
        *oj-select-many*, *oj-switch*, *oj-radioset*, *oj-file-picker*

-   **Tabella**:

    -   *oj-table* (DataProvider REST) con columns + slot per templating
        chip/badge

-   **Feedback**:

    -   *oj-messages*, *oj-message-toast* per esiti/validazioni

-   **Bottoni**:

    -   *oj-button* (variants: primary/outlined/ghost)

-   **Decorazioni**:

    -   *oj-avatar* (Assignee), chips/pills con *oj-badge* / CSS utility
        Redwood

**6) Azioni & binding (Action Chains)**

**6.1 Issue List**

-   **On Filter Change** → Aggiorna DataProvider table con query‑params
    (status, severity, domainStream, environment, solutionType, q, page,
    pageSize, sort).

-   **On Sort** → aggiorna sort e ricarica.

-   **On Page Change** → aggiorna page/pageSize.

-   **Nuova segnalazione** → Navigate (Page 2).

**6.2 Issue Create/Edit**

-   **Invia** → POST /issues

    -   On success: toast "Segnalazione inviata (#id)" → Navigate to
        **Detail**

    -   On error: *oj-messages*

**6.3 Issue Detail --- Workflow**

-   **Proponi soluzione** → POST /issues/{id}/transition con
    action=PROPOSE_SOLUTION + payload (resolutionNotes, fixVersion,
    fixEnvironment)

-   **Invia in re‑test** → action=SEND_TO_RETEST + payload
    (retestInstructions)

-   **Conferma risoluzione** → action=CONFIRM_RESOLUTION +
    {retestOutcome: PASS, retestNotes}

-   **Riapri (FAIL)** → action=REOPEN_FAIL + {retestOutcome: FAIL,
    retestNotes}

-   **Chiudi** → action=CLOSE + {closureReason, closureEvidenceLink}

**7) Regole di visibilità/abilitazione (UI)**

-   **Campi SaaS/PaaS**: mostra Tenant/Pod se SaaS, mostra App Version
    se PaaS.

-   **Workflow buttons** (role+status):

    -   IN_PROGRESS → Proponi soluzione (Assignee/Lead)

    -   SOLUTION_PROPOSED → Invia in re‑test (Triage)

    -   RETEST → Conferma risoluzione e Esito negativo / Riapri
        (Tester/Triage)

    -   RESOLVED → Chiudi (Triage/PMO)

-   **Campi obbligatori condizionali**: enforced sia UI che server
    (package in DB).

**8) Accessibilità e UX micro‑dettagli**

-   **Label chiare + helper text** (es. "Se Blocker/High, allega
    screenshot/log")

-   **Keyboard friendly** (tab order, tasti invio/spazio su
    radio/switch)

-   **Live regions** per toast di stato (Announce)

-   **Contrasto** per chip severity/ status (palette Redwood compliant)

**9) KPI e quick views (facoltativo, ma utile)**

-   **Tabs rapidi** in lista con filtri pre‑impostati:

    -   "Le mie" → createdBy=currentClient (se vuoi usarlo)

    -   "Da re‑testare" → status=RETEST&retestAssignedTo=me

    -   "Alte priorità" → severity=BLOCKER,HIGH&status in open

-   **Badge count** (lazy): chiamate leggere per contare per tab.

**10) Tutto in 1 sprint: piano build consigliato**

1.  **Service Connections** (REST)

2.  **List** (tabella + filtri + sort/paging)

3.  **Create/Edit** (4 card + validazioni)

4.  **Detail** (tabs + commenti + allegati + workflow)

5.  **Permessi UI** (show/hide azioni)

6.  **Test end‑to‑end** con un caso per stream:

    -   CONTABILITA in SVI (flow completo fino a CLOSED)

    -   PROCUREMENT in CALL (fail → reopen)

**1. Ambito e principi di progettazione**

**1.1 Requisiti chiave**

-   **Pagina unica** (o set coerente di pagine) per segnalazioni UAT

-   Dati minimi comuni a SaaS e PaaS, con possibilità di **dettagli
    variabili** per dominio/soluzione

-   Tracciamento audit: chi/quando ha creato/modificato, e log cambi
    stato

-   Semplicità per utenti UAT (tester/key user) e capacità di governance
    per triage

**1.2 Approccio "riusabile" SaaS/PaaS**

Due opzioni implementative (consigliata A come baseline, B come
evoluzione):

-   **A. Campi condizionali**: mostra/nasconde campi in base a
    solutionType (SaaS/PaaS) e domainStream.

-   **B. Payload JSON**: campo context_payload per memorizzare dettagli
    aggiuntivi variabili (consigliato come evoluzione, ma si può
    predisporre subito nel DB).

**2. UX / Mock-up (wireframe funzionale)**

**2.1 Pagine previste**

1.  **Issue List (UAT)**

2.  **Issue Create/Edit (Segnala bug/incident)**

3.  **Issue Detail (dettaglio + allegati + commenti + azioni workflow)**

**2.2 Pagina 1 -- Issue List (UAT)**

**Funzioni**

-   Lista segnalazioni con colonne principali: ID, Titolo, Stream,
    Soluzione, Severità, Stato, Assegnatario, Data creazione, Ultimo
    aggiornamento.

-   Filtri: environment, domainStream, solutionType, status, severity,
    assignee, testo libero (titolo/descrizione).

-   Viste rapide:

    -   "Le mie segnalazioni"

    -   "Da re-testare" (status=In re-test AND retestAssignedTo=utente)

    -   "Aging \> X giorni" (opzionale)

-   Azioni: Apri dettaglio, Export CSV, (Triage) cambio
    stato/assegnazione rapidi.

**2.3 Pagina 2 -- Issue Create/Edit (Segnala bug/incident)**

**Sezioni (card)**

**A) Contesto (obbligatoria)**

-   Tipo soluzione (radio **SaaS / PaaS**) -- obbligatorio

-   Applicazione/Modulo (LOV) -- obbligatorio

-   Ambiente (LOV, es. UAT1/UAT2/...) -- obbligatorio

-   Domain/Stream (LOV) -- obbligatorio

-   URL/Pagina (testo)

-   Tenant/Pod (opzionale -- soprattutto per SaaS)

-   ID Test Case / Script UAT (testo) + Step UAT (testo multiline)

**B) Descrizione bug (obbligatoria)**

-   Titolo breve (max 120) -- obbligatorio

-   Record type: Bug / **Incident** / Miglioria / Chiarimento / Dato /
    Accesso (LOV)

-   Severità (Blocker/High/Medium/Low) -- obbligatorio

-   Priorità (P1/P2/P3) -- opzionale ma consigliata per triage

-   Frequenza (Sempre/Spesso/Raramente/Prima volta)

-   Impatti (checkbox: processo bloccato, errore dati, performance, UX,
    compliance)

-   Comportamento atteso (textarea) -- obbligatorio

-   Comportamento osservato (textarea) -- obbligatorio

-   Messaggio errore (textarea) -- opzionale

**C) Riproducibilità (obbligatoria)**

-   Prerequisiti (textarea)

-   Passi per riprodurre (textarea) -- obbligatorio

-   Riproducibile (toggle sì/no)

    -   se "No" → obbligo Note aggiuntive

**D) Evidenze (consigliata)**

-   Allegati multipli (screenshot/log/export)

-   Riferimenti transazione/documento (testo; facoltativo o dinamico per
    stream)

**Azioni**

-   Salva bozza (status=Bozza)

-   Invia (status=Nuovo) + notifica triage

-   Invia e nuova (per tester intensivi)

**2.4 Pagina 3 -- Issue Detail (dettaglio)**

**Contenuti**

-   Dati della segnalazione (read-only o edit parziale in base a
    permessi)

-   Allegati

-   Commenti (con distinzione internal/public se previsto)

-   Pannello gestione (visibile a triage/lead): stato, assegnatario,
    componente, release target

**Azioni workflow contestuali** (in alto a destra o in card "Azioni")

-   Proponi soluzione (Assignee/Lead, da In lavorazione → Soluzione
    proposta)

-   Invia in re-test (Triage o Assignee, Soluzione proposta → In
    re-test)

-   Conferma risoluzione (Tester, In re-test → Risolto)

-   Esito negativo / Riapri (Tester, In re-test → In lavorazione)

-   Chiudi incident/issue (Triage/PMO, Risolto → Chiuso)

-   Richiedi info (Triage/Assignee, qualsiasi stato → In attesa info)

**3. Workflow -- Stati, transizioni e regole**

**3.1 Stati (status LOV)**

1.  Bozza

2.  Nuovo

3.  In triage

4.  In lavorazione

5.  Soluzione proposta

6.  In re-test

7.  Risolto

8.  Chiuso

9.  In attesa info

10. Non riproducibile

11. Duplicato

12. Scartato

**3.2 Transizioni principali**

-   Nuovo → In triage (Triage)

-   In triage → In lavorazione (Triage/Lead)

-   In lavorazione → Soluzione proposta (Assignee/Lead)

-   Soluzione proposta → In re-test (Triage o Assignee)

-   In re-test → Risolto (Tester, esito PASS)

-   In re-test → In lavorazione (Tester, esito FAIL)

-   Risolto → Chiuso (Triage/PMO)

Transizioni di servizio:

-   Qualsiasi → In attesa info (Triage/Assignee)

-   In attesa info → (stato precedente o In triage) (Triage/Assignee)

**3.3 Ruoli e permessi (VBCS app roles)**

-   **UAT_Tester**: crea segnalazioni, vede proprie e lista (in base
    alla policy), aggiunge commenti, esegue re-test e registra esito.

-   **UAT_Triage**: vede tutte, cambia stato, assegna, invia in re-test,
    chiude.

-   **UAT_Lead/Assignee**: lavora sui ticket assegnati, propone
    soluzione, aggiorna note e fix version.

-   **UAT_Admin**: configura LOV, gestisce permessi, eventuali export
    completi.

Nota: in VBCS i permessi si implementano con **role-based security** +
condizioni su record (es. reporter=me) e UI rules (show/hide/readonly).

**4. Validazioni e business rules**

**4.1 Validazioni al submit (Invia)**

Obbligatori:

-   solutionType, applicationModule, environment, domainStream

-   title, recordType, severity

-   expectedBehavior, actualBehavior, stepsToReproduce

Condizionali:

-   reproducible = No → notes obbligatorio

-   severity in {Blocker, High} e nessun allegato → warning "allegato
    fortemente consigliato"

-   recordType = Dato → richiedere riferimento record/transazione (se
    previsto)

**4.2 Validazioni per cambio stato**

-   **In lavorazione → Soluzione proposta**

    -   resolutionNotes obbligatorio

    -   fixVersion obbligatorio (almeno per PaaS; per SaaS "best
        effort")

-   **Soluzione proposta → In re-test**

    -   retestAssignedTo obbligatorio (default = reporter)

    -   retestInstructions consigliato (warning se vuoto)

-   **In re-test → Risolto**

    -   retestOutcome = PASS obbligatorio

    -   retestNotes opzionale, allegato consigliato (warning)

-   **In re-test → In lavorazione (riapri)**

    -   retestOutcome = FAIL + retestNotes obbligatorio

-   **Risolto → Chiuso**

    -   closureReason obbligatorio

    -   closedOn/closedBy valorizzati automaticamente

**4.3 Regole UI (visibilità / editabilità)**

-   Campi resolutionNotes, fixVersion, fixEnvironment visibili/editabili
    quando status ≥ In lavorazione (solo lead/assignee)

-   Campi re-test (retestOutcome, retestNotes) visibili solo quando
    status = In re-test (tester/triage)

-   Card "Chiusura" visibile quando status = Risolto (triage/pmo)

**5. Notifiche (minimo indispensabile)**

Trigger consigliati:

1.  Quando status = **Nuovo** → notifica a gruppo Triage

2.  Quando status = **In re-test** → notifica a retestAssignedTo +
    reporter

3.  Quando status = **Risolto** → notifica a triage + assignee

4.  Quando status = **Chiuso** → notifica a reporter

Canali:

-   Email (baseline)

-   Teams webhook / integrazione (opzionale)

-   Commento automatico "System" in timeline (consigliato per audit)

**6. Data Model su Oracle ATP (backend per VBCS)**

**6.1 Scelte progettuali**

-   DB: **Oracle Autonomous Transaction Processing (ATP)**

-   Schema dedicato (es. UAT_BUGS)

-   Tabelle principali: UAT_ISSUE, UAT_ISSUE_ATTACHMENT,
    UAT_ISSUE_COMMENT, UAT_ISSUE_STATUS_HIST

-   Strategie chiavi: NUMBER GENERATED AS IDENTITY per PK

-   CLOB per descrizioni lunghe; BLOB per allegati (opzionale: in
    alternativa usare storage esterno, ma per UAT va bene BLOB)

-   Indici su: status, severity, domain_stream, assignee, created_on,
    ricerca testuale (opzionale: Oracle Text)

**6.2 DDL -- Tabelle**

**Nota**: Adeguare i nomi/schema secondo standard interni.\
Le LOV possono essere implementate come tabelle di configurazione
(consigliato) oppure hard-coded in VBCS. Qui propongo anche tabelle LOV.

**6.2.1 Tabella principale: UAT_ISSUE**

CREATE TABLE uat_issue (

  issue_id            NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY
KEY,

  \-- audit

  created_on          TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

  created_by          VARCHAR2(128) NOT NULL,

  updated_on          TIMESTAMP,

  updated_by          VARCHAR2(128),

  \-- contesto comune

  solution_type       VARCHAR2(10)  NOT NULL,  \-- \'SaaS\' / \'PaaS\'

  application_module  VARCHAR2(100) NOT NULL,

  environment         VARCHAR2(50)  NOT NULL,  \-- es. UAT1/UAT2

  domain_stream       VARCHAR2(100) NOT NULL,

  page_url            VARCHAR2(1000),

  tenant_pod          VARCHAR2(100),           \-- SaaS

  app_version         VARCHAR2(60),            \-- PaaS (build/deploy
version)

  test_case_id        VARCHAR2(80),

  uat_step_ref        VARCHAR2(200),

  \-- classificazione

  record_type         VARCHAR2(20)  NOT NULL,  \--
Bug/Incident/Miglioria/\...

  severity            VARCHAR2(10)  NOT NULL,  \--
Blocker/High/Medium/Low

  priority            VARCHAR2(5),             \-- P1/P2/P3

  frequency           VARCHAR2(20),            \-- Sempre/Spesso/\...

  impact_flags        VARCHAR2(200),           \-- CSV o JSON (es.
\"ProcessoBloccato,UX\")

  \-- descrizioni

  title               VARCHAR2(120) NOT NULL,

  expected_behavior   CLOB NOT NULL,

  actual_behavior     CLOB NOT NULL,

  error_message       CLOB,

  prerequisites       CLOB,

  steps_to_reproduce  CLOB NOT NULL,

  reproducible_yn     CHAR(1) DEFAULT \'Y\' CHECK (reproducible_yn IN
(\'Y\',\'N\')),

  notes              CLOB,

  \-- gestione

  status              VARCHAR2(30) NOT NULL,

  component           VARCHAR2(30),            \--
UI/Integration/Data/Security/\...

  assignee            VARCHAR2(128),           \-- username/idcs user

  target_release      VARCHAR2(60),

  duplicate_of        NUMBER,                 \-- issue_id

  \-- proposta soluzione / retest / chiusura

  resolution_notes    CLOB,

  fix_version         VARCHAR2(60),

  fix_environment     VARCHAR2(50),

  retest_assigned_to  VARCHAR2(128),

  retest_outcome      VARCHAR2(10),            \-- PASS/FAIL

  retest_notes        CLOB,

  resolved_on         TIMESTAMP,

  resolved_by         VARCHAR2(128),

  closure_reason      VARCHAR2(50),

  closed_on           TIMESTAMP,

  closed_by           VARCHAR2(128),

  \-- contesto estendibile (opzionale ma consigliato)

  context_payload     CLOB,

  CONSTRAINT fk_uat_issue_duplicate

    FOREIGN KEY (duplicate_of) REFERENCES uat_issue(issue_id)

);

CREATE INDEX idx_uat_issue_status       ON uat_issue(status);

CREATE INDEX idx_uat_issue_severity     ON uat_issue(severity);

CREATE INDEX idx_uat_issue_stream       ON uat_issue(domain_stream);

CREATE INDEX idx_uat_issue_assignee     ON uat_issue(assignee);

CREATE INDEX idx_uat_issue_created_on   ON uat_issue(created_on);

**6.2.2 Allegati: UAT_ISSUE_ATTACHMENT**

CREATE TABLE uat_issue_attachment (

  attachment_id      NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY
KEY,

  issue_id           NUMBER NOT NULL,

  file_name          VARCHAR2(255) NOT NULL,

  mime_type          VARCHAR2(100),

  file_size          NUMBER,

  file_content       BLOB,

  description        VARCHAR2(500),

  uploaded_on        TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

  uploaded_by        VARCHAR2(128) NOT NULL,

  CONSTRAINT fk_uat_attach_issue

    FOREIGN KEY (issue_id) REFERENCES uat_issue(issue_id) ON DELETE
CASCADE

);

CREATE INDEX idx_uat_attach_issue ON uat_issue_attachment(issue_id);

**6.2.3 Commenti: UAT_ISSUE_COMMENT**

CREATE TABLE uat_issue_comment (

  comment_id         NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY
KEY,

  issue_id           NUMBER NOT NULL,

  comment_text       CLOB NOT NULL,

  visibility         VARCHAR2(10) DEFAULT \'PUBLIC\' CHECK (visibility
IN (\'PUBLIC\',\'INTERNAL\')),

  comment_on         TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

  comment_by         VARCHAR2(128) NOT NULL,

  CONSTRAINT fk_uat_comment_issue

    FOREIGN KEY (issue_id) REFERENCES uat_issue(issue_id) ON DELETE
CASCADE

);

CREATE INDEX idx_uat_comment_issue ON uat_issue_comment(issue_id);

**6.2.4 Storico stati (audit workflow): UAT_ISSUE_STATUS_HIST**

CREATE TABLE uat_issue_status_hist (

  hist_id            NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY
KEY,

  issue_id           NUMBER NOT NULL,

  old_status         VARCHAR2(30),

  new_status         VARCHAR2(30) NOT NULL,

  changed_on         TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

  changed_by         VARCHAR2(128) NOT NULL,

  change_note        VARCHAR2(500),

  CONSTRAINT fk_uat_hist_issue

    FOREIGN KEY (issue_id) REFERENCES uat_issue(issue_id) ON DELETE
CASCADE

);

CREATE INDEX idx_uat_hist_issue ON uat_issue_status_hist(issue_id);

CREATE INDEX idx_uat_hist_changed_on ON
uat_issue_status_hist(changed_on);

**6.3 Tabelle LOV (consigliate)**

Se si vuole governance e modifiche senza redeploy, creare tabelle di
configurazione.

**6.3.1 Esempio: UAT_LOV (generica)**

CREATE TABLE uat_lov (

  lov_type           VARCHAR2(30) NOT NULL,

  lov_code           VARCHAR2(50) NOT NULL,

  lov_label          VARCHAR2(200) NOT NULL,

  sort_order         NUMBER DEFAULT 10,

  enabled_yn         CHAR(1) DEFAULT \'Y\' CHECK (enabled_yn IN
(\'Y\',\'N\')),

  CONSTRAINT pk_uat_lov PRIMARY KEY (lov_type, lov_code)

);

\-- esempi di lov_type: STATUS, SEVERITY, PRIORITY, RECORD_TYPE,
DOMAIN_STREAM, ENVIRONMENT, COMPONENT, CLOSURE_REASON

**6.4 Trigger/Procedure (opzionali ma consigliati)**

**6.4.1 Audit updated_on/updated_by + storico stato**

In VBCS si può gestire via event/action (preferred), ma lato DB si può
aggiungere robustezza.

Esempio trigger per storico stato:

CREATE OR REPLACE TRIGGER trg_uat_issue_status

BEFORE UPDATE OF status ON uat_issue

FOR EACH ROW

WHEN (OLD.status IS DISTINCT FROM NEW.status)

BEGIN

  INSERT INTO uat_issue_status_hist(issue_id, old_status, new_status,
changed_by, change_note)

  VALUES (:NEW.issue_id, :OLD.status, :NEW.status, NVL(:NEW.updated_by,
:NEW.created_by), NULL);

END;

/

\</span\>\<span
attribution=\"{&quot;name&quot;:&quot;Copilot&quot;,&quot;oid&quot;:&quot;E64C3D4F-5E12-4514-AD9B-893A6FAFD00C&quot;,&quot;id&quot;:&quot;E64C3D4F-5E12-4514-AD9B-893A6FAFD00C&quot;,&quot;userInfo&quot;:{&quot;name&quot;:&quot;Copilot&quot;,&quot;oid&quot;:&quot;E64C3D4F-5E12-4514-AD9B-893A6FAFD00C&quot;,&quot;id&quot;:&quot;E64C3D4F-5E12-4514-AD9B-893A6FAFD00C&quot;},&quot;timestamp&quot;:1773335700000,&quot;dataSource&quot;:0}\"\>

Nota: updated_by deve essere valorizzato dall'app (VBCS) per avere lo
username corretto.

**7. Integrazione VBCS ↔ ATP**

**7.1 Modalità consigliata**

-   VBCS usa **Business Objects** con **REST endpoints** generati
    automaticamente.

-   Il backend ATP può essere integrato in due modi:

    1.  **BO su VBCS** e storage su DB di VBCS (più rapido ma meno
        controllabile)

    2.  **ATP come DB** esponendo REST (ORDS) o via service connection e
        mapping\
        **Consigliato**: se l'organizzazione vuole governare dati e
        avere pieno controllo SQL.

**7.2 Mapping suggerito**

-   BO Issue ↔ tabella UAT_ISSUE

-   BO IssueAttachment ↔ tabella UAT_ISSUE_ATTACHMENT

-   BO IssueComment ↔ tabella UAT_ISSUE_COMMENT

-   (opz) BO IssueStatusHistory ↔ UAT_ISSUE_STATUS_HIST

Se si usa ORDS:

-   definire moduli REST /issues, /issues/{id}/attachments,
    /issues/{id}/comments, /issues/{id}/statusHistory.

-   gestire POST/PUT/PATCH per transizioni di stato con validazioni
    applicative.

**8. Regole operative (azioni workflow in VBCS)**

**8.1 Azioni e precondizioni**

**Azione: Proponi soluzione**

-   Visibile: Assignee/Lead

-   Precondizione: status = In lavorazione

-   Effetto:

    -   status = Soluzione proposta

    -   resolutionNotes obbligatorio

    -   fixVersion (obbligatorio per PaaS)

    -   Inserire commento "System" con note fix (opzionale)

**Azione: Invia in re-test**

-   Visibile: Triage (o Assignee se consentito)

-   Precondizione: status = Soluzione proposta

-   Effetto:

    -   status = In re-test

    -   retestAssignedTo valorizzato (default reporter o selezione)

    -   notifica al tester

**Azione: Conferma risoluzione**

-   Visibile: Tester assegnato (retestAssignedTo = me) o Triage

-   Precondizione: status = In re-test

-   Effetto:

    -   retestOutcome = PASS

    -   status = Risolto

    -   resolved_on/by valorizzati

    -   notifica triage + assignee

**Azione: Esito negativo / Riapri**

-   Visibile: Tester assegnato o triage

-   Precondizione: status = In re-test

-   Effetto:

    -   retestOutcome = FAIL

    -   retestNotes obbligatorio

    -   status = In lavorazione

    -   notifica assignee

**Azione: Chiudi**

-   Visibile: Triage/PMO

-   Precondizione: status = Risolto (o in casi particolari:
    Duplicato/Scartato/Non riproducibile)

-   Effetto:

    -   status = Chiuso

    -   closureReason obbligatorio

    -   closed_on/by valorizzati

    -   notifica reporter

**9. Non funzionali (NFR)**

-   Sicurezza: autenticazione SSO/IDCS; accesso basato su ruoli

-   Performance: lista paginata; indici su
    status/severity/stream/assignee/date

-   Data retention: definire durata conservazione (es. fine UAT + X
    mesi)

-   Privacy: banner "non allegare dati sensibili"; mascheramento
    eventuale

**10. MVP e milestone di implementazione**

**MVP (rapido)**

-   Tabelle ATP + ORDS (se scelto) o BO VBCS

-   Pagine: List / Create / Detail

-   Allegati + commenti

-   Stati base + transizioni principali (incl. re-test e chiusura)

-   Notifiche email base

**Evoluzioni**

-   Deduplica assistita

-   Dashboard KPI (aging, volume per stream, severity mix)

-   Integrazione con Jira/Azure DevOps/ServiceNow

-   Oracle Text per ricerca full-text avanzata

**11. Checklist per lo sviluppatore (pronta all'uso)**

-   Implementare data model ATP (DDL sopra) e credenziali/schema

-   Esporre API (ORDS) oppure mappare BO direttamente su ATP

-   Creare LOV (tabelle o VBCS LOV)

-   Implementare pagine VBCS: List, Create/Edit, Detail

-   Implementare regole UI (visibilità/editabilità)

-   Implementare validazioni (submit + cambio stato)

-   Implementare azioni workflow (bottoni contestuali)

-   Implementare notifiche (almeno 4 eventi)

-   Implementare storico stato (trigger DB o log in app)

-   Test end-to-end con 2 profili: Tester e Triage

**1) Data Model ATP (Backend)**

**1.1 Tabelle principali (come già definito) + aggiornamenti**

Confermiamo le tabelle:

-   UAT_ISSUE

-   UAT_ISSUE_ATTACHMENT

-   UAT_ISSUE_COMMENT

-   UAT_ISSUE_STATUS_HIST

-   UAT_LOV (configurazione liste)

**Aggiornamenti richiesti**:

-   Environment limitato a SVI, CALL (LOV)

-   Domain/Stream limitato a Contabilità, Procurement (LOV)

-   Aggiunta gestione **Use Case** e Owner (re-test assignment da
    tabella)

**1.1.1 Nuova tabella: UAT_USE_CASE (owner del caso d'uso)**

Questa tabella abilita la regola: *quando si manda in re-test, si
assegna automaticamente al Owner del caso d'uso associato.*

SQL

CREATE TABLE uat_use_case (

use_case_code VARCHAR2(50) PRIMARY KEY, \-- es. UC_ACC_001

use_case_title VARCHAR2(200) NOT NULL, \-- descrizione breve

domain_stream VARCHAR2(100) NOT NULL, \-- Contabilità / Procurement

owner_user VARCHAR2(128) NOT NULL, \-- username (IDCS / corporate)

owner_email VARCHAR2(256), \-- se utile per email notification

enabled_yn CHAR(1) DEFAULT \'Y\' CHECK (enabled_yn IN (\'Y\',\'N\')),

created_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

created_by VARCHAR2(128) NOT NULL,

updated_on TIMESTAMP,

updated_by VARCHAR2(128)

);

CREATE INDEX idx_uat_use_case_stream ON uat_use_case(domain_stream);

CREATE INDEX idx_uat_use_case_owner ON uat_use_case(owner_user);

Mostra più linee

**1.1.2 Modifica tabella UAT_ISSUE: collegamento al caso d'uso**

Aggiungiamo use_case_code e vincolo FK.

SQL

ALTER TABLE uat_issue ADD (

use_case_code VARCHAR2(50)

);

ALTER TABLE uat_issue ADD CONSTRAINT fk_uat_issue_use_case

FOREIGN KEY (use_case_code) REFERENCES uat_use_case(use_case_code);

Mostra più linee

Nota: use_case_code può sostituire o affiancare test_case_id. Se volete
mantenere entrambi, nessun problema: test_case_id resta "free text",
mentre use_case_code governa l'assegnazione re-test.

**1.2 Seed LOV (environment e domain/stream)**

Se usate UAT_LOV per tutte le liste, questi insert iniziali evitano
hard-code in VBCS.

SQL

\-- DOMAIN_STREAM

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'DOMAIN_STREAM\',\'CONTABILITA\',\'Contabilità\',10);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'DOMAIN_STREAM\',\'PROCUREMENT\',\'Procurement\',20);

\-- ENVIRONMENT

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'ENVIRONMENT\',\'SVI\',\'SVI\',10);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'ENVIRONMENT\',\'CALL\',\'CALL\',20);

\-- STATUS (estratto minimo; aggiungere tutti)

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'DRAFT\',\'Bozza\',10);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'NEW\',\'Nuovo\',20);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'TRIAGE\',\'In triage\',30);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'IN_PROGRESS\',\'In lavorazione\',40);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'SOLUTION_PROPOSED\',\'Soluzione proposta\',50);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'RETEST\',\'In re-test\',60);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'RESOLVED\',\'Risolto\',70);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'STATUS\',\'CLOSED\',\'Chiuso\',80);

Mostra più linee

**1.3 Regola re-test: assegnazione automatica "Owner caso d'uso"**

**Regola funzionale**:

-   Quando un ticket passa a RETEST, valorizzare retest_assigned_to
    usando:

    -   SELECT owner_user FROM uat_use_case WHERE use_case_code =
        :use_case_code AND enabled_yn=\'Y\'

-   Se use_case_code è null o owner non trovato → fallback:

    -   retest_assigned_to = created_by (reporter)

**Implementazione consigliata**

-   lato applicazione (VBCS action "Invia in re-test") **oppure**

-   lato DB tramite procedure ORDS (handler PL/SQL) per avere coerenza e
    audit

**2) Template API ORDS (Struttura risorse + payload)**

**2.1 Base URL e convenzioni**

-   **Base path**: /uat/api/v1

-   Formati: JSON UTF-8

-   Date: ISO-8601 (es. \"2026-03-12T18:06:59+01:00\")

-   Identità utente: header (es. X-User) oppure derivata da SSO/OAuth2
    (raccomandato).

**Standard response (success)**

JSON

{

\"data\": { },

\"meta\": { \"requestId\": \"\...\" }

}

Mostra più linee

**Standard response (error)**

JSON

{

\"error\": {

\"code\": \"VALIDATION_ERROR\",

\"message\": \"Missing required field: title\",

\"details\": \[

{ \"field\": \"title\", \"issue\": \"REQUIRED\" }

\]

},

\"meta\": { \"requestId\": \"\...\" }

}

Mostra più linee

**2.2 Risorsa: LOV (liste configurate)**

**GET /lov/{type}**

Esempio:

-   GET /uat/api/v1/lov/ENVIRONMENT

-   GET /uat/api/v1/lov/DOMAIN_STREAM

-   GET /uat/api/v1/lov/STATUS

**Response**

JSON

{

\"data\": \[

{ \"code\": \"SVI\", \"label\": \"SVI\", \"sortOrder\": 10 },

{ \"code\": \"CALL\", \"label\": \"CALL\", \"sortOrder\": 20 }

\]

}

Mostra più linee

**2.3 Risorsa: Use Case (owner per re-test)**

**GET /usecases?domainStream=CONTABILITA&enabled=Y**

**Response**

JSON

{

\"data\": \[

{

\"useCaseCode\": \"UC_ACC_001\",

\"useCaseTitle\": \"Registrazione prima nota\",

\"domainStream\": \"CONTABILITA\",

\"ownerUser\": \"mario.rossi\",

\"ownerEmail\": \"mario.rossi@ente.it\",

\"enabled\": \"Y\"

}

\]

}

Mostra più linee

**GET /usecases/{useCaseCode}**

**2.4 Risorsa: Issues (CRUD + ricerca)**

**GET /issues (ricerca con filtri)**

Query params (tutti opzionali):

-   status (es. NEW, RETEST)

-   severity (Blocker/High/Medium/Low)

-   domainStream (CONTABILITA/PROCUREMENT)

-   environment (SVI/CALL)

-   solutionType (SaaS/PaaS)

-   assignee

-   createdBy

-   q (search testo su title)

-   page, pageSize, sort (es. createdOn:desc)

**Response (lista)**

JSON

{

\"data\": \[

{

\"issueId\": 101,

\"title\": \"Errore validazione fattura split payment\",

\"domainStream\": \"CONTABILITA\",

\"environment\": \"SVI\",

\"solutionType\": \"SaaS\",

\"severity\": \"High\",

\"status\": \"TRIAGE\",

\"assignee\": \"dev.user1\",

\"createdOn\": \"2026-03-12T10:15:00+01:00\"

}

\],

\"meta\": { \"page\": 1, \"pageSize\": 20, \"total\": 53 }

}

Mostra più linee

**POST /issues (creazione)**

**Request (minimo)**

JSON

{

\"solutionType\": \"SaaS\",

\"applicationModule\": \"Fusion Payables\",

\"environment\": \"SVI\",

\"domainStream\": \"CONTABILITA\",

\"useCaseCode\": \"UC_ACC_001\",

\"pageUrl\": \"https://\...\",

\"recordType\": \"Incident\",

\"severity\": \"Blocker\",

\"priority\": \"P1\",

\"frequency\": \"Sempre\",

\"title\": \"Blocco in conferma fattura\",

\"expectedBehavior\": \"La fattura deve essere confermata
correttamente.\",

\"actualBehavior\": \"Errore in fase di conferma.\",

\"stepsToReproduce\": \"1) \... 2) \...\",

\"reproducible\": \"Y\",

\"errorMessage\": \"ORA-xxxx (se presente)\"

}

Mostra più linee

**Response**

JSON

{

\"data\": {

\"issueId\": 123,

\"status\": \"NEW\"

}

}

Mostra più linee

**GET /issues/{issueId}**

Restituisce il dettaglio completo (inclusi CLOB, campi retest/closure,
etc.)

**PATCH /issues/{issueId}**

Aggiornamento parziale campi non "di stato" (es. title, steps, notes).\
Per cambio stato **usare endpoint di transition** (vedi §2.6) così
centralizzi validazioni.

**2.5 Risorse: Commenti e Allegati**

**GET /issues/{issueId}/comments**

**POST /issues/{issueId}/comments**

**Request**

JSON

{

\"commentText\": \"Ho verificato: avviene solo con supplier X.\",

\"visibility\": \"PUBLIC\"

}

Mostra più linee

**GET /issues/{issueId}/attachments**

**POST /issues/{issueId}/attachments**

Per upload BLOB via ORDS si può:

-   usare multipart/form-data (consigliato)

-   oppure base64 in JSON (meno efficiente)

**Esempio (multipart)**

-   campi: file, description

**GET /attachments/{attachmentId}/content**

Restituisce il file con Content-Type corretto.

**2.6 Endpoint di workflow (transizioni di stato) --- consigliato**

**POST /issues/{issueId}/transition**

**Request (schema generico)**

JSON

{

\"action\": \"PROPOSE_SOLUTION\",

\"payload\": {

\"resolutionNotes\": \"Applicata correzione su validazione.\",

\"fixVersion\": \"1.0.12\",

\"fixEnvironment\": \"SVI\"

}

}

Mostra più linee

Azioni previste:

-   PROPOSE_SOLUTION (IN_PROGRESS → SOLUTION_PROPOSED)

-   SEND_TO_RETEST (SOLUTION_PROPOSED → RETEST)

-   CONFIRM_RESOLUTION (RETEST → RESOLVED)

-   REOPEN_FAIL (RETEST → IN_PROGRESS)

-   CLOSE (RESOLVED → CLOSED)

-   REQUEST_INFO (ANY → WAITING_INFO) *(opzionale)*

**SEND_TO_RETEST: assegnazione automatica Owner Use Case**

**Request**

JSON

{

\"action\": \"SEND_TO_RETEST\",

\"payload\": {

\"retestInstructions\": \"Ripetere step 1-3; verificare conferma.\",

\"retestAssignedTo\": null

}

}

Mostra più linee

**Logica**:

-   se retestAssignedTo nullo → lookup su UAT_USE_CASE.owner_user usando
    useCaseCode

-   fallback su created_by

**Response**

JSON

{

\"data\": {

\"issueId\": 123,

\"status\": \"RETEST\",

\"retestAssignedTo\": \"mario.rossi\"

}

}

\`\`

Mostra più linee

**CONFIRM_RESOLUTION (PASS)**

JSON

{

\"action\": \"CONFIRM_RESOLUTION\",

\"payload\": {

\"retestOutcome\": \"PASS\",

\"retestNotes\": \"Ok in SVI con dati standard.\"

}

}

Mostra più linee

**REOPEN_FAIL (FAIL)**

JSON

{

\"action\": \"REOPEN_FAIL\",

\"payload\": {

\"retestOutcome\": \"FAIL\",

\"retestNotes\": \"Persistono errori con supplier Y; allego
screenshot.\"

}

}

Mostra più linee

**CLOSE**

JSON

{

\"action\": \"CLOSE\",

\"payload\": {

\"closureReason\": \"Fix verificato\",

\"closureEvidenceLink\": \"https://link-evidenza-opzionale\"

}

}

Mostra più linee

**2.7 Skeleton ORDS (opzionale, per far partire lo sviluppatore)**

Se lo sviluppatore vuole uno scheletro per ORDS, ecco l'impianto minimo
(da adattare):

SQL

BEGIN

ORDS.DEFINE_MODULE(

p_module_name =\> \'uat_api_v1\',

p_base_path =\> \'/uat/api/v1/\',

p_items_per_page =\> 25);

\-- Template: GET /lov/{type}

ORDS.DEFINE_TEMPLATE(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'lov/:type\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'lov/:type\',

p_method =\> \'GET\',

p_source_type =\> ORDS.SOURCE_TYPE_QUERY,

p_source =\> q\'\[

SELECT lov_code AS code, lov_label AS label, sort_order AS sortOrder

FROM uat_lov

WHERE lov_type = :type

AND enabled_yn = \'Y\'

ORDER BY sort_order

\]\');

COMMIT;

END;

/

Mostra più linee

Per /issues/{id}/transition conviene usare SOURCE_TYPE_PLSQL per fare
validazioni + update + storico stato in un unico punto.

**3) Modello Dati "BO" / Entità VBCS allineate ad ATP**

**3.1 Entità (logical BO)**

**3.1.1 Issue**

**Campi principali (mapping su UAT_ISSUE)**

-   issueId ↔ issue_id

-   createdOn, createdBy, updatedOn, updatedBy

-   solutionType (SaaS/PaaS)

-   applicationModule

-   environment (SVI/CALL)

-   domainStream (Contabilità/Procurement)

-   useCaseCode (FK a UseCase)

-   pageUrl, tenantPod, appVersion

-   recordType (Bug/Incident/...)

-   severity, priority, frequency, impactFlags

-   title

-   expectedBehavior, actualBehavior, errorMessage

-   prerequisites, stepsToReproduce, reproducible, notes

-   status

-   component, assignee, targetRelease, duplicateOf

**Campi workflow**

-   resolutionNotes, fixVersion, fixEnvironment

-   retestAssignedTo, retestInstructions, retestOutcome, retestNotes

-   resolvedOn, resolvedBy

-   closureReason, closedOn, closedBy, closureEvidenceLink

-   contextPayload (JSON/CLOB)

**3.1.2 UseCase**

**Campi (mapping UAT_USE_CASE)**

-   useCaseCode (PK)

-   useCaseTitle

-   domainStream

-   ownerUser

-   ownerEmail

-   enabled

**3.1.3 Attachment**

**Campi (UAT_ISSUE_ATTACHMENT)**

-   attachmentId (PK)

-   issueId (FK)

-   fileName, mimeType, fileSize, fileContent

-   description, uploadedOn, uploadedBy

**3.1.4 Comment**

**Campi (UAT_ISSUE_COMMENT)**

-   commentId (PK)

-   issueId (FK)

-   commentText, visibility

-   commentOn, commentBy

**3.1.5 StatusHistory (audit)**

**Campi (UAT_ISSUE_STATUS_HIST)**

-   histId

-   issueId

-   oldStatus, newStatus

-   changedOn, changedBy, changeNote

**3.2 Relazioni (cardinalità)**

-   Issue **1 --- N** Attachment

-   Issue **1 --- N** Comment

-   Issue **1 --- N** StatusHistory

-   UseCase **1 --- N** Issue (via useCaseCode)

-   Issue **N --- 1** Issue (duplicato) via duplicateOf *(opzionale)*

**3.3 Regole di business in VBCS (implementazione "build-ready")**

**3.3.1 Policy re-test (owner use case)**

Quando l'utente (Triage/Assignee) preme **"Invia in re-test"**:

1.  Se Issue.useCaseCode valorizzato → chiamare API GET /usecases/{code}

2.  Valorizzare Issue.retestAssignedTo = useCase.ownerUser

3.  Se non esiste owner/disabled → fallback createdBy

4.  Eseguire transition SEND_TO_RETEST

Questa logica può stare in VBCS (action chain) oppure in ORDS transition
handler.

**3.3.2 Visibilità campi e bottoni**

-   Bottoni in base a status + ruolo (Tester/Triage/Assignee)

-   Campi retest visibili solo in RETEST

-   Chiusura visibile solo in RESOLVED (Triage)

**3.3.3 Validazioni (riassunto)**

-   Submit: obbligatori contesto + descrizioni + steps

-   Transition:

    -   PROPOSE_SOLUTION: obbligo resolutionNotes; obbligo fixVersion
        (soprattutto PaaS)

    -   SEND_TO_RETEST: retestAssignedTo autocalcolato; warning se
        retestInstructions vuoto

    -   CONFIRM_RESOLUTION: outcome PASS

    -   REOPEN_FAIL: outcome FAIL + note obbligatorie

    -   CLOSE: closureReason obbligatorio

**4) Implementazione VBCS (consigliata) con ATP/ORDS**

**4.1 Service Connections**

Creare in VBCS una **Service Connection** verso ORDS:

-   Base URL: https://\<ords-host\>/uat/api/v1/

-   Autenticazione: (a seconda del setup) Basic/OAuth2; lato VBCS
    conviene OAuth2 se disponibile.

**4.2 Types / Data Providers**

-   Creare **Types** (Issue, UseCase, Attachment, Comment) coerenti con
    i payload ORDS

-   Configurare **Data Providers** per:

    -   lista issues (GET /issues)

    -   dettaglio issue (GET /issues/{id})

    -   lov (GET /lov/{type})

    -   use cases (GET /usecases)

    -   attachments/comments

**4.3 Action Chains (workflow)**

Creare Action Chain per ogni bottone workflow:

-   ProposeSolution → POST transition

-   SendToRetest → (GET usecase owner) + POST transition

-   ConfirmResolution → POST transition

-   ReopenFail → POST transition

-   Close → POST transition

**5) Notifiche (solo email)**

Implementazione (2 opzioni):

1.  **Lato ORDS/DB**: procedure che inviano email (se consentito dal
    contesto; in ATP si può integrare con servizi di mail)

2.  **Lato VBCS**: chiamata verso un servizio mail interno (più comune)
    oppure integrazione con meccanismi già presenti

**Eventi** (minimi):

-   NEW → mail a gruppo Triage

-   RETEST → mail a retestAssignedTo

-   RESOLVED → mail a Triage + Assignee

-   CLOSED → mail a Reporter

Nel payload di notifica includere: IssueId, titolo, status, link pagina
VBCS, domainStream, environment.

**6) Parametri già fissati (da configurare)**

-   DOMAIN_STREAM: Contabilità, Procurement

-   ENVIRONMENT: SVI, CALL

-   RETEST_OWNER_POLICY: lookup UAT_USE_CASE.owner_user (default)

-   Notifiche: email

**A) Mappatura LOV -- codici interni e label (definitiva)**

**A.1 Convenzioni**

-   I **codici** (LOV_CODE) sono **stabili** e usati in API/DB.

-   Le **label** (LOV_LABEL) sono "human readable" e usate in UI.

-   Tutti i codici sono **UPPERCASE**, senza spazi, con \_ se
    necessario.

-   Il DB salva i **codici**, non le label.

**A.2 Domain/Stream (LOV_TYPE = DOMAIN_STREAM)**

**Scelta confermata:** codici interni CONTABILITA / PROCUREMENT.

  -----------------------------------------------------------------------
  **CODE**                        **LABEL**                  **SORT**
  ------------------------------- -------------------------- ------------
  CONTABILITA                     Contabilità                10

  PROCUREMENT                     Procurement                20
  -----------------------------------------------------------------------

SQL seed:

SQL

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'DOMAIN_STREAM\',\'CONTABILITA\',\'Contabilità\',10);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'DOMAIN_STREAM\',\'PROCUREMENT\',\'Procurement\',20);

\`\`

Mostra più linee

**A.3 Environment (LOV_TYPE = ENVIRONMENT)**

  -----------------------------------------------------------------------
  **CODE**                **LABEL**                **SORT**
  ----------------------- ------------------------ ----------------------
  SVI                     SVI                      10

  CALL                    CALL                     20
  -----------------------------------------------------------------------

SQL seed:

SQL

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'ENVIRONMENT\',\'SVI\',\'SVI\',10);

INSERT INTO uat_lov(lov_type, lov_code, lov_label, sort_order) VALUES
(\'ENVIRONMENT\',\'CALL\',\'CALL\',20);

Mostra più linee

**A.4 Status (LOV_TYPE = STATUS)**

(riporto la mappatura completa consigliata; label italiane)

  -------------------------------------------------------------------------
  **CODE**                          **LABEL**                    **SORT**
  --------------------------------- ---------------------------- ----------
  DRAFT                             Bozza                        10

  NEW                               Nuovo                        20

  TRIAGE                            In triage                    30

  IN_PROGRESS                       In lavorazione               40

  SOLUTION_PROPOSED                 Soluzione proposta           50

  RETEST                            In re-test                   60

  RESOLVED                          Risolto                      70

  CLOSED                            Chiuso                       80

  WAITING_INFO                      In attesa info               90

  NOT_REPRODUCIBLE                  Non riproducibile            100

  DUPLICATE                         Duplicato                    110

  REJECTED                          Scartato                     120
  -------------------------------------------------------------------------

**A.5 Record Type (LOV_TYPE = RECORD_TYPE)**

  -----------------------------------------------------------------------
  **CODE**                         **LABEL**                 **SORT**
  -------------------------------- ------------------------- ------------
  BUG                              Bug                       10

  INCIDENT                         Incident                  20

  IMPROVEMENT                      Miglioria                 30

  QUESTION                         Chiarimento               40

  DATA                             Dato                      50

  ACCESS                           Accesso                   60
  -----------------------------------------------------------------------

**A.6 Severity (LOV_TYPE = SEVERITY)**

  -----------------------------------------------------------------------
  **CODE**                    **LABEL**                 **SORT**
  --------------------------- ------------------------- -----------------
  BLOCKER                     Blocker                   10

  HIGH                        High                      20

  MEDIUM                      Medium                    30

  LOW                         Low                       40
  -----------------------------------------------------------------------

**A.7 Priority (LOV_TYPE = PRIORITY)**

  -----------------------------------------------------------------------
  **CODE**                **LABEL**                **SORT**
  ----------------------- ------------------------ ----------------------
  P1                      P1                       10

  P2                      P2                       20

  P3                      P3                       30
  -----------------------------------------------------------------------

**A.8 Frequency (LOV_TYPE = FREQUENCY)**

  ------------------------------------------------------------------------
  **CODE**                **LABEL**                       **SORT**
  ----------------------- ------------------------------- ----------------
  ALWAYS                  Sempre                          10

  OFTEN                   Spesso                          20

  RARELY                  Raramente                       30

  ONCE                    Prima volta                     40
  ------------------------------------------------------------------------

**A.9 Component (LOV_TYPE = COMPONENT)**

  -----------------------------------------------------------------------
  **CODE**                        **LABEL**                 **SORT**
  ------------------------------- ------------------------- -------------
  UI                              UI                        10

  INTEGRATION                     Integration               20

  DATA                            Data                      30

  SECURITY                        Security                  40

  WORKFLOW                        Workflow                  50

  REPORTING                       Reporting                 60
  -----------------------------------------------------------------------

**A.10 Closure Reason (LOV_TYPE = CLOSURE_REASON)**

  --------------------------------------------------------------------------
  **CODE**                           **LABEL**                    **SORT**
  ---------------------------------- ---------------------------- ----------
  FIX_VERIFIED                       Fix verificato               10

  WORKAROUND_ACCEPTED                Workaround accettato         20

  DUPLICATE                          Duplicato                    30

  NOT_REPRODUCIBLE                   Non riproducibile            40

  OUT_OF_SCOPE                       Fuori ambito UAT             50
  --------------------------------------------------------------------------

**B) Update Owner Use Case: owner_user_name + owner_email (definitivo)**

**B.1 Tabella UAT_USE_CASE (aggiornata)**

-   owner_user_name: usato per valorizzare retest_assigned_to (username)

-   owner_email: usato per inviare email di notifica

-   (opzionale) entrambi obbligatori per coerenza operativa

DDL aggiornato:

SQL

CREATE TABLE uat_use_case (

use_case_code VARCHAR2(50) PRIMARY KEY,

use_case_title VARCHAR2(200) NOT NULL,

domain_stream VARCHAR2(100) NOT NULL, \-- CONTABILITA / PROCUREMENT

owner_user_name VARCHAR2(128) NOT NULL, \-- username

owner_email VARCHAR2(256) NOT NULL, \-- email

enabled_yn CHAR(1) DEFAULT \'Y\' CHECK (enabled_yn IN (\'Y\',\'N\')),

created_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

created_by VARCHAR2(128) NOT NULL,

updated_on TIMESTAMP,

updated_by VARCHAR2(128)

);

CREATE INDEX idx_uat_use_case_stream ON uat_use_case(domain_stream);

CREATE INDEX idx_uat_use_case_owner ON uat_use_case(owner_user_name);

Mostra più linee

**B.2 Regola di assegnazione re-test (definitiva)**

In transizione SEND_TO_RETEST:

1.  se use_case_code valorizzato e use case attivo →

    -   retest_assigned_to = owner_user_name

    -   notify_to = owner_email

2.  fallback se non trovato →

    -   retest_assigned_to = created_by

    -   notify_to = created_by_email *(se disponibile; altrimenti
        notifica a triage)*

Se created_by_email non è disponibile oggi, puoi gestire l'email del
reporter in due modi:

-   \(A\) ricavarla da identity provider / SSO

-   \(B\) aggiungere campo reporter_email su UAT_ISSUE (opzionale)

**C) Specifica dettagliata GET /issues (filtri, sorting, paginazione)**

**C.1 Endpoint**

GET /uat/api/v1/issues

**C.2 Paginazione (standard)**

Parametri:

-   page (int, default 1, min 1)

-   pageSize (int, default 20, min 1, max 100)

-   In alternativa (se preferito): offset + limit (non consigliato se
    già usate page/pageSize)

**Comportamento**

-   offset = (page - 1) \* pageSize

-   La response include meta.total, meta.page, meta.pageSize,
    meta.totalPages

**Response meta esempio**

JSON

\"meta\": {

\"page\": 2,

\"pageSize\": 20,

\"total\": 53,

\"totalPages\": 3,

\"sort\": \"createdOn:desc\"

}

Mostra più linee

**C.3 Sorting (ordinamento)**

Parametro:

-   sort (string)

    -   Formato: field:direction

    -   direction ∈ {asc, desc}

    -   Supporto multi-sort: sort=severity:desc,createdOn:desc

**Default**

-   sort=createdOn:desc

**Campi ordinabili (whitelist)**

-   createdOn, updatedOn

-   severity

-   status

-   domainStream

-   environment

-   assignee

-   priority

**Nota per severity/priority** Per avere ordering "logico" (BLOCKER \>
HIGH \> ...), usare:

-   una **CASE expression** lato SQL oppure

-   una colonna "rank" derivata o join su LOV con sort_order.

**C.4 Filtri (query params) -- dettagli e operatori**

**C.4.1 Filtri base (uguaglianza)**

-   status (code, es. RETEST)

-   severity (code, es. HIGH)

-   priority (code, es. P1)

-   recordType (code, es. INCIDENT)

-   domainStream (code, CONTABILITA\|PROCUREMENT)

-   environment (code, SVI\|CALL)

-   solutionType (SaaS\|PaaS)

-   assignee (username)

-   createdBy (username)

-   retestAssignedTo (username)

-   useCaseCode (es. UC_ACC_001)

**Esempio** GET
/issues?domainStream=CONTABILITA&environment=SVI&status=RETEST

**C.4.2 Filtri multipli (IN list)**

Supporto lista con separatore ,:

-   status=NEW,TRIAGE,IN_PROGRESS

-   severity=BLOCKER,HIGH

**Esempio** GET /issues?status=NEW,TRIAGE&severity=HIGH,BLOCKER

Nota: la semantica è IN (\...) (OR all'interno dello stesso filtro).

**C.4.3 Ricerca testuale (q)**

Parametro:

-   q cerca su:

    -   title

    -   (opzionale) actualBehavior, errorMessage

**Regole**

-   minimo 3 caratteri

-   matching LIKE case-insensitive (baseline)

-   evoluzione: Oracle Text

**Esempio** GET /issues?q=fattura

**C.4.4 Filtri temporali (range)**

Parametri:

-   createdFrom, createdTo (ISO-8601 oppure YYYY-MM-DD)

-   updatedFrom, updatedTo

**Esempio** GET /issues?createdFrom=2026-03-01&createdTo=2026-03-12

**C.4.5 Filtro "aging"**

Parametro:

-   ageDaysMin (int) → SYSDATE - created_on \>= ageDaysMin

**Esempio** GET /issues?status=IN_PROGRESS&ageDaysMin=10

**C.4.6 Vista "Da re-testare" per utente corrente**

Parametro:

-   myRetest=Y → server imposta retestAssignedTo = currentUser

**Esempio** GET /issues?myRetest=Y&status=RETEST

*(In alternativa VBCS invia esplicitamente
retestAssignedTo=\<username\>.)*

**C.5 Set di filtri consigliati "operativi" (view presets)**

-   **My Issues**: createdBy=currentUser

-   **My Retest Queue**: status=RETEST&retestAssignedTo=currentUser

-   **Triage Inbox**: status=NEW,TRIAGE

-   **High severity open**:
    severity=BLOCKER,HIGH&status=NEW,TRIAGE,IN_PROGRESS,SOLUTION_PROPOSED,RETEST

-   **Closed last week**: status=CLOSED&updatedFrom=\...

**C.6 Response payload (lista) -- campi restituiti**

Per performance, GET /issues in lista restituisce un subset (no CLOB, no
BLOB):

-   issueId, title, domainStream, environment, solutionType

-   recordType, severity, priority, status

-   assignee, retestAssignedTo

-   createdOn, updatedOn

-   (opzionale) useCaseCode

Il dettaglio completo si recupera con GET /issues/{id}.

**C.7 Error handling (validazione query)**

Esempi:

-   pageSize \> 100 → 400 INVALID_PAGINATION

-   sort su campo non whitelist → 400 INVALID_SORT_FIELD

-   status con code non valido → 400 INVALID_FILTER_VALUE

**D) Esempi completi di chiamate GET /issues**

**D.1 Triage -- nuove segnalazioni Contabilità in SVI**

GET
/issues?domainStream=CONTABILITA&environment=SVI&status=NEW,TRIAGE&sort=severity:desc,createdOn:desc&page=1&pageSize=20

**D.2 Tester -- coda personale re-test**

GET
/issues?status=RETEST&retestAssignedTo=mario.rossi&sort=updatedOn:desc

**D.3 Ricerca per testo "procurement" con filtro ambiente CALL**

GET /issues?q=procurement&environment=CALL&page=1&pageSize=50

**D.4 Aging oltre 7 giorni, in lavorazione o soluzione proposta**

GET
/issues?status=IN_PROGRESS,SOLUTION_PROPOSED&ageDaysMin=7&sort=createdOn:asc

**E) Aggiornamento payload ORDS per Use Case e assegnazione owner**

**E.1 GET /usecases response (includere user + email)**

JSON

{

\"data\": \[

{

\"useCaseCode\": \"UC_ACC_001\",

\"useCaseTitle\": \"Registrazione prima nota\",

\"domainStream\": \"CONTABILITA\",

\"ownerUserName\": \"mario.rossi\",

\"ownerEmail\": \"mario.rossi@ente.it\",

\"enabled\": \"Y\"

}

\]

}

\`\`

Mostra più linee

**E.2 Transition SEND_TO_RETEST response (esplicita owner)**

JSON

{

\"data\": {

\"issueId\": 123,

\"status\": \"RETEST\",

\"retestAssignedTo\": \"mario.rossi\",

\"notifiedEmail\": \"mario.rossi@ente.it\"

}

}

Mostra più linee

**F) Mini-update DB: domain_stream e environment come code
(allineamento)**

Assicurarsi che in UAT_ISSUE i campi siano coerenti e contengano
**code**:

-   domain_stream = CONTABILITA o PROCUREMENT

-   environment = SVI o CALL

Se la tabella è già popolata con label, prevedere una migrazione
(update) coerente.

ORDS:

-   GET /uat/api/v1/lov/{type}

-   GET /uat/api/v1/usecases e GET /uat/api/v1/usecases/{code}

-   GET /uat/api/v1/issues (filtri + sorting + paginazione + meta)

-   POST /uat/api/v1/issues (creazione)

-   GET /uat/api/v1/issues/{id} (dettaglio)

-   PATCH/PUT /uat/api/v1/issues/{id} (update "non stato")

-   POST /uat/api/v1/issues/{id}/transition (workflow: propose solution
    / send to retest / confirm pass / reopen fail / close)

-   GET/POST /uat/api/v1/issues/{id}/comments

-   GET/POST /uat/api/v1/issues/{id}/attachments + GET
    /uat/api/v1/attachments/{attId}/content

E con regole che abbiamo definito:

-   Domain codes: **CONTABILITA, PROCUREMENT**

-   Environment codes: **SVI, CALL**

-   Re-test assignment: **Owner del caso d'uso** da tabella
    (owner_user_name + owner_email)

-   Notifiche: **solo email**

**0) Prerequisiti minimi (pratici)**

1.  Hai uno **schema** su ATP (es. UAT_BUGS) su cui creare tabelle e
    package.

2.  ORDS è già **configurato** verso quello schema (anche via Autonomous
    ORDS/REST).

3.  Per email: uno di questi scenari (scegli il più semplice per te):

    -   **A)** APEX presente e SMTP configurato → useremo APEX_MAIL
        (consigliato su ATP).

    -   **B)** SMTP non pronto → useremo una **OUTBOX** (tabella) +
        invio manuale/aggiuntivo dopo.\
        (Io ti preparo entrambe: se APEX_MAIL funziona, invia;
        altrimenti accoda.)

**1) Script 01 --- Data Model ATP (tabelle + indici)**

Esegui tutto il blocco.

SQL

\-- =========================================================

\-- 01 - DATA MODEL: tabelle principali

\-- =========================================================

CREATE TABLE uat_issue (

issue_id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,

created_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

created_by VARCHAR2(128) NOT NULL,

updated_on TIMESTAMP,

updated_by VARCHAR2(128),

solution_type VARCHAR2(10) NOT NULL, \-- SaaS / PaaS

application_module VARCHAR2(100) NOT NULL,

environment VARCHAR2(50) NOT NULL, \-- SVI / CALL

domain_stream VARCHAR2(100) NOT NULL, \-- CONTABILITA / PROCUREMENT

use_case_code VARCHAR2(50),

page_url VARCHAR2(1000),

tenant_pod VARCHAR2(100),

app_version VARCHAR2(60),

test_case_id VARCHAR2(80),

uat_step_ref VARCHAR2(200),

record_type VARCHAR2(20) NOT NULL, \-- BUG/INCIDENT/\...

severity VARCHAR2(10) NOT NULL, \-- BLOCKER/HIGH/\...

priority VARCHAR2(5),

frequency VARCHAR2(20),

impact_flags VARCHAR2(200),

title VARCHAR2(120) NOT NULL,

expected_behavior CLOB NOT NULL,

actual_behavior CLOB NOT NULL,

error_message CLOB,

prerequisites CLOB,

steps_to_reproduce CLOB NOT NULL,

reproducible_yn CHAR(1) DEFAULT \'Y\' CHECK (reproducible_yn IN
(\'Y\',\'N\')),

notes CLOB,

status VARCHAR2(30) NOT NULL,

component VARCHAR2(30),

assignee VARCHAR2(128),

target_release VARCHAR2(60),

duplicate_of NUMBER,

resolution_notes CLOB,

fix_version VARCHAR2(60),

fix_environment VARCHAR2(50),

retest_assigned_to VARCHAR2(128),

retest_instructions CLOB,

retest_outcome VARCHAR2(10), \-- PASS/FAIL

retest_notes CLOB,

resolved_on TIMESTAMP,

resolved_by VARCHAR2(128),

closure_reason VARCHAR2(50),

closure_evidence_link VARCHAR2(1000),

closed_on TIMESTAMP,

closed_by VARCHAR2(128),

context_payload CLOB,

CONSTRAINT fk_uat_issue_duplicate

FOREIGN KEY (duplicate_of) REFERENCES uat_issue(issue_id)

);

CREATE INDEX idx_uat_issue_status ON uat_issue(status);

CREATE INDEX idx_uat_issue_severity ON uat_issue(severity);

CREATE INDEX idx_uat_issue_stream ON uat_issue(domain_stream);

CREATE INDEX idx_uat_issue_env ON uat_issue(environment);

CREATE INDEX idx_uat_issue_assignee ON uat_issue(assignee);

CREATE INDEX idx_uat_issue_created_on ON uat_issue(created_on);

CREATE INDEX idx_uat_issue_usecase ON uat_issue(use_case_code);

CREATE TABLE uat_issue_attachment (

attachment_id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,

issue_id NUMBER NOT NULL,

file_name VARCHAR2(255) NOT NULL,

mime_type VARCHAR2(100),

file_size NUMBER,

file_content BLOB,

description VARCHAR2(500),

uploaded_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

uploaded_by VARCHAR2(128) NOT NULL,

CONSTRAINT fk_uat_attach_issue

FOREIGN KEY (issue_id) REFERENCES uat_issue(issue_id) ON DELETE CASCADE

);

CREATE INDEX idx_uat_attach_issue ON uat_issue_attachment(issue_id);

CREATE TABLE uat_issue_comment (

comment_id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,

issue_id NUMBER NOT NULL,

comment_text CLOB NOT NULL,

visibility VARCHAR2(10) DEFAULT \'PUBLIC\' CHECK (visibility IN
(\'PUBLIC\',\'INTERNAL\')),

comment_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

comment_by VARCHAR2(128) NOT NULL,

CONSTRAINT fk_uat_comment_issue

FOREIGN KEY (issue_id) REFERENCES uat_issue(issue_id) ON DELETE CASCADE

);

CREATE INDEX idx_uat_comment_issue ON uat_issue_comment(issue_id);

CREATE TABLE uat_issue_status_hist (

hist_id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,

issue_id NUMBER NOT NULL,

old_status VARCHAR2(30),

new_status VARCHAR2(30) NOT NULL,

changed_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

changed_by VARCHAR2(128) NOT NULL,

change_note VARCHAR2(500),

CONSTRAINT fk_uat_hist_issue

FOREIGN KEY (issue_id) REFERENCES uat_issue(issue_id) ON DELETE CASCADE

);

CREATE INDEX idx_uat_hist_issue ON uat_issue_status_hist(issue_id);

CREATE INDEX idx_uat_hist_changed_on ON
uat_issue_status_hist(changed_on);

CREATE TABLE uat_lov (

lov_type VARCHAR2(30) NOT NULL,

lov_code VARCHAR2(50) NOT NULL,

lov_label VARCHAR2(200) NOT NULL,

sort_order NUMBER DEFAULT 10,

enabled_yn CHAR(1) DEFAULT \'Y\' CHECK (enabled_yn IN (\'Y\',\'N\')),

CONSTRAINT pk_uat_lov PRIMARY KEY (lov_type, lov_code)

);

CREATE TABLE uat_use_case (

use_case_code VARCHAR2(50) PRIMARY KEY,

use_case_title VARCHAR2(200) NOT NULL,

domain_stream VARCHAR2(100) NOT NULL, \-- CONTABILITA / PROCUREMENT

owner_user_name VARCHAR2(128) NOT NULL, \-- username

owner_email VARCHAR2(256) NOT NULL, \-- email

enabled_yn CHAR(1) DEFAULT \'Y\' CHECK (enabled_yn IN (\'Y\',\'N\')),

created_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

created_by VARCHAR2(128) NOT NULL,

updated_on TIMESTAMP,

updated_by VARCHAR2(128)

);

CREATE INDEX idx_uat_use_case_stream ON uat_use_case(domain_stream);

CREATE INDEX idx_uat_use_case_owner ON uat_use_case(owner_user_name);

ALTER TABLE uat_issue ADD CONSTRAINT fk_uat_issue_use_case

FOREIGN KEY (use_case_code) REFERENCES uat_use_case(use_case_code);

\-- OUTBOX email (fallback se APEX_MAIL non è configurato)

CREATE TABLE uat_email_outbox (

email_id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,

created_on TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,

status VARCHAR2(20) DEFAULT \'PENDING\', \-- PENDING/SENT/ERROR

to_email VARCHAR2(256) NOT NULL,

cc_email VARCHAR2(512),

subject VARCHAR2(300) NOT NULL,

body_html CLOB,

body_text CLOB,

last_error VARCHAR2(1000)

);

CREATE INDEX idx_uat_email_outbox_status ON uat_email_outbox(status);

Mostra più linee

**2) Script 02 --- Seed LOV (codici/label definitivi)**

Esegui tutto il blocco.

SQL

\-- =========================================================

\-- 02 - SEED LOV (codici stabili + label UI)

\-- =========================================================

\-- DOMAIN_STREAM

INSERT INTO uat_lov VALUES
(\'DOMAIN_STREAM\',\'CONTABILITA\',\'Contabilità\',10,\'Y\');

INSERT INTO uat_lov VALUES
(\'DOMAIN_STREAM\',\'PROCUREMENT\',\'Procurement\',20,\'Y\');

\-- ENVIRONMENT

INSERT INTO uat_lov VALUES (\'ENVIRONMENT\',\'SVI\',\'SVI\',10,\'Y\');

INSERT INTO uat_lov VALUES (\'ENVIRONMENT\',\'CALL\',\'CALL\',20,\'Y\');

\-- STATUS

INSERT INTO uat_lov VALUES (\'STATUS\',\'DRAFT\',\'Bozza\',10,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'NEW\',\'Nuovo\',20,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'TRIAGE\',\'In
triage\',30,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'IN_PROGRESS\',\'In
lavorazione\',40,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'SOLUTION_PROPOSED\',\'Soluzione
proposta\',50,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'RETEST\',\'In
re-test\',60,\'Y\');

INSERT INTO uat_lov VALUES
(\'STATUS\',\'RESOLVED\',\'Risolto\',70,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'CLOSED\',\'Chiuso\',80,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'WAITING_INFO\',\'In attesa
info\',90,\'Y\');

INSERT INTO uat_lov VALUES (\'STATUS\',\'NOT_REPRODUCIBLE\',\'Non
riproducibile\',100,\'Y\');

INSERT INTO uat_lov VALUES
(\'STATUS\',\'DUPLICATE\',\'Duplicato\',110,\'Y\');

INSERT INTO uat_lov VALUES
(\'STATUS\',\'REJECTED\',\'Scartato\',120,\'Y\');

\-- RECORD_TYPE

INSERT INTO uat_lov VALUES (\'RECORD_TYPE\',\'BUG\',\'Bug\',10,\'Y\');

INSERT INTO uat_lov VALUES
(\'RECORD_TYPE\',\'INCIDENT\',\'Incident\',20,\'Y\');

INSERT INTO uat_lov VALUES
(\'RECORD_TYPE\',\'IMPROVEMENT\',\'Miglioria\',30,\'Y\');

INSERT INTO uat_lov VALUES
(\'RECORD_TYPE\',\'QUESTION\',\'Chiarimento\',40,\'Y\');

INSERT INTO uat_lov VALUES (\'RECORD_TYPE\',\'DATA\',\'Dato\',50,\'Y\');

INSERT INTO uat_lov VALUES
(\'RECORD_TYPE\',\'ACCESS\',\'Accesso\',60,\'Y\');

\-- SEVERITY

INSERT INTO uat_lov VALUES
(\'SEVERITY\',\'BLOCKER\',\'Blocker\',10,\'Y\');

INSERT INTO uat_lov VALUES (\'SEVERITY\',\'HIGH\',\'High\',20,\'Y\');

INSERT INTO uat_lov VALUES
(\'SEVERITY\',\'MEDIUM\',\'Medium\',30,\'Y\');

INSERT INTO uat_lov VALUES (\'SEVERITY\',\'LOW\',\'Low\',40,\'Y\');

\-- PRIORITY

INSERT INTO uat_lov VALUES (\'PRIORITY\',\'P1\',\'P1\',10,\'Y\');

INSERT INTO uat_lov VALUES (\'PRIORITY\',\'P2\',\'P2\',20,\'Y\');

INSERT INTO uat_lov VALUES (\'PRIORITY\',\'P3\',\'P3\',30,\'Y\');

\-- FREQUENCY

INSERT INTO uat_lov VALUES
(\'FREQUENCY\',\'ALWAYS\',\'Sempre\',10,\'Y\');

INSERT INTO uat_lov VALUES
(\'FREQUENCY\',\'OFTEN\',\'Spesso\',20,\'Y\');

INSERT INTO uat_lov VALUES
(\'FREQUENCY\',\'RARELY\',\'Raramente\',30,\'Y\');

INSERT INTO uat_lov VALUES (\'FREQUENCY\',\'ONCE\',\'Prima
volta\',40,\'Y\');

\-- COMPONENT

INSERT INTO uat_lov VALUES (\'COMPONENT\',\'UI\',\'UI\',10,\'Y\');

INSERT INTO uat_lov VALUES
(\'COMPONENT\',\'INTEGRATION\',\'Integration\',20,\'Y\');

INSERT INTO uat_lov VALUES (\'COMPONENT\',\'DATA\',\'Data\',30,\'Y\');

INSERT INTO uat_lov VALUES
(\'COMPONENT\',\'SECURITY\',\'Security\',40,\'Y\');

INSERT INTO uat_lov VALUES
(\'COMPONENT\',\'WORKFLOW\',\'Workflow\',50,\'Y\');

INSERT INTO uat_lov VALUES
(\'COMPONENT\',\'REPORTING\',\'Reporting\',60,\'Y\');

\-- CLOSURE_REASON

INSERT INTO uat_lov VALUES (\'CLOSURE_REASON\',\'FIX_VERIFIED\',\'Fix
verificato\',10,\'Y\');

INSERT INTO uat_lov VALUES
(\'CLOSURE_REASON\',\'WORKAROUND_ACCEPTED\',\'Workaround
accettato\',20,\'Y\');

INSERT INTO uat_lov VALUES
(\'CLOSURE_REASON\',\'DUPLICATE\',\'Duplicato\',30,\'Y\');

INSERT INTO uat_lov VALUES
(\'CLOSURE_REASON\',\'NOT_REPRODUCIBLE\',\'Non
riproducibile\',40,\'Y\');

INSERT INTO uat_lov VALUES (\'CLOSURE_REASON\',\'OUT_OF_SCOPE\',\'Fuori
ambito UAT\',50,\'Y\');

COMMIT;

Mostra più linee

**3) Script 03 --- Package PL/SQL (logiche core: validazioni, ricerca,
workflow, email)**

Questo è il "cuore": ti evita di sviluppare logiche sparse.

**3.1 Package spec**

SQL

CREATE OR REPLACE PACKAGE uat_api_pkg AS

\-- Utility: JSON response

PROCEDURE write_json(p_clob CLOB);

\-- LOV

FUNCTION lov_list(p_type VARCHAR2) RETURN SYS_REFCURSOR;

\-- Use cases

FUNCTION usecase_list(p_domain_stream VARCHAR2 DEFAULT NULL, p_enabled
CHAR DEFAULT NULL) RETURN SYS_REFCURSOR;

FUNCTION usecase_get(p_code VARCHAR2) RETURN SYS_REFCURSOR;

\-- Issues CRUD

PROCEDURE issue_create(p_body CLOB, p_user_name VARCHAR2, p_issue_id OUT
NUMBER);

FUNCTION issue_get(p_issue_id NUMBER) RETURN SYS_REFCURSOR;

PROCEDURE issue_update(p_issue_id NUMBER, p_body CLOB, p_user_name
VARCHAR2);

\-- Search Issues (lista con meta)

FUNCTION issue_search_json(

p_page NUMBER,

p_page_size NUMBER,

p_sort VARCHAR2,

p_status VARCHAR2,

p_severity VARCHAR2,

p_priority VARCHAR2,

p_record_type VARCHAR2,

p_domain_stream VARCHAR2,

p_environment VARCHAR2,

p_solution_type VARCHAR2,

p_assignee VARCHAR2,

p_created_by VARCHAR2,

p_retest_assigned_to VARCHAR2,

p_use_case_code VARCHAR2,

p_q VARCHAR2,

p_created_from VARCHAR2,

p_created_to VARCHAR2,

p_updated_from VARCHAR2,

p_updated_to VARCHAR2,

p_age_days_min NUMBER

) RETURN CLOB;

\-- Comments

FUNCTION comments_list(p_issue_id NUMBER) RETURN SYS_REFCURSOR;

PROCEDURE comment_add(p_issue_id NUMBER, p_body CLOB, p_user_name
VARCHAR2);

\-- Attachments

FUNCTION attachments_list(p_issue_id NUMBER) RETURN SYS_REFCURSOR;

PROCEDURE attachment_add(p_issue_id NUMBER, p_file_name VARCHAR2, p_mime
VARCHAR2, p_size NUMBER,

p_blob BLOB, p_desc VARCHAR2, p_user_name VARCHAR2);

PROCEDURE attachment_get_content(p_attachment_id NUMBER);

\-- Workflow transition

FUNCTION transition_json(p_issue_id NUMBER, p_body CLOB, p_user_name
VARCHAR2) RETURN CLOB;

END uat_api_pkg;

/

Mostra più linee

**3.2 Package body (implementazione)**

È lungo ma completo. Incollalo ed eseguilo.

SQL

CREATE OR REPLACE PACKAGE BODY uat_api_pkg AS

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Helpers

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

PROCEDURE write_json(p_clob CLOB) IS

BEGIN

owa_util.mime_header(\'application/json; charset=utf-8\', FALSE);

htp.p(\'Cache-Control: no-store\');

owa_util.http_header_close;

htp.prn(p_clob);

END;

PROCEDURE raise_bad_request(p_code VARCHAR2, p_msg VARCHAR2) IS

BEGIN

raise_application_error(-20000, p_code \|\| \': \' \|\| p_msg);

END;

FUNCTION lov_exists(p_type VARCHAR2, p_code VARCHAR2) RETURN BOOLEAN IS

v_cnt NUMBER;

BEGIN

SELECT COUNT(\*) INTO v_cnt

FROM uat_lov

WHERE lov_type = p_type AND lov_code = p_code AND enabled_yn=\'Y\';

RETURN v_cnt \> 0;

END;

FUNCTION normalize_csv(p_csv VARCHAR2) RETURN VARCHAR2 IS

BEGIN

RETURN REGEXP_REPLACE(UPPER(TRIM(p_csv)), \'\\s+\', \'\');

END;

FUNCTION is_null_or_empty(p_str VARCHAR2) RETURN BOOLEAN IS

BEGIN

RETURN p_str IS NULL OR TRIM(p_str) IS NULL;

END;

FUNCTION get_json_string(p_obj JSON_OBJECT_T, p_key VARCHAR2) RETURN
VARCHAR2 IS

BEGIN

IF p_obj.has(p_key) THEN

RETURN p_obj.get_string(p_key);

END IF;

RETURN NULL;

EXCEPTION

WHEN OTHERS THEN

RETURN NULL;

END;

FUNCTION get_json_clob(p_obj JSON_OBJECT_T, p_key VARCHAR2) RETURN CLOB
IS

BEGIN

IF p_obj.has(p_key) THEN

RETURN p_obj.get_clob(p_key);

END IF;

RETURN NULL;

EXCEPTION

WHEN OTHERS THEN

RETURN NULL;

END;

PROCEDURE add_status_hist(p_issue_id NUMBER, p_old VARCHAR2, p_new
VARCHAR2, p_user VARCHAR2, p_note VARCHAR2) IS

BEGIN

INSERT INTO uat_issue_status_hist(issue_id, old_status, new_status,
changed_by, change_note)

VALUES (p_issue_id, p_old, p_new, p_user, p_note);

END;

PROCEDURE add_system_comment(p_issue_id NUMBER, p_text VARCHAR2, p_user
VARCHAR2) IS

BEGIN

INSERT INTO uat_issue_comment(issue_id, comment_text, visibility,
comment_by)

VALUES (p_issue_id, p_text, \'INTERNAL\', p_user);

END;

PROCEDURE queue_email(p_to VARCHAR2, p_subject VARCHAR2, p_body_html
CLOB, p_body_text CLOB DEFAULT NULL, p_cc VARCHAR2 DEFAULT NULL) IS

BEGIN

INSERT INTO uat_email_outbox(to_email, cc_email, subject, body_html,
body_text)

VALUES (p_to, p_cc, p_subject, p_body_html, p_body_text);

END;

PROCEDURE send_email_best_effort(p_to VARCHAR2, p_subject VARCHAR2,
p_body_html CLOB, p_body_text CLOB DEFAULT NULL, p_cc VARCHAR2 DEFAULT
NULL) IS

v_mail_id NUMBER;

BEGIN

\-- Tentativo APEX_MAIL. Se non configurato, ripiega su OUTBOX.

BEGIN

v_mail_id := apex_mail.send(

p_to =\> p_to,

p_cc =\> p_cc,

p_from =\> \'noreply@uat.local\', \-- \<\< personalizza

p_subj =\> p_subject,

p_body =\> NVL(p_body_text, \' \'),

p_body_html =\> p_body_html

);

apex_mail.push_queue;

EXCEPTION

WHEN OTHERS THEN

queue_email(p_to, p_subject, p_body_html, p_body_text, p_cc);

END;

END;

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- LOV / Use case

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

FUNCTION lov_list(p_type VARCHAR2) RETURN SYS_REFCURSOR IS

rc SYS_REFCURSOR;

BEGIN

OPEN rc FOR

SELECT lov_code AS code, lov_label AS label, sort_order AS sortOrder

FROM uat_lov

WHERE lov_type = UPPER(p_type) AND enabled_yn=\'Y\'

ORDER BY sort_order, lov_label;

RETURN rc;

END;

FUNCTION usecase_list(p_domain_stream VARCHAR2 DEFAULT NULL, p_enabled
CHAR DEFAULT NULL) RETURN SYS_REFCURSOR IS

rc SYS_REFCURSOR;

BEGIN

OPEN rc FOR

SELECT use_case_code AS useCaseCode,

use_case_title AS useCaseTitle,

domain_stream AS domainStream,

owner_user_name AS ownerUserName,

owner_email AS ownerEmail,

enabled_yn AS enabled

FROM uat_use_case

WHERE (p_domain_stream IS NULL OR domain_stream =
UPPER(p_domain_stream))

AND (p_enabled IS NULL OR enabled_yn = UPPER(p_enabled))

ORDER BY domain_stream, use_case_code;

RETURN rc;

END;

FUNCTION usecase_get(p_code VARCHAR2) RETURN SYS_REFCURSOR IS

rc SYS_REFCURSOR;

BEGIN

OPEN rc FOR

SELECT use_case_code AS useCaseCode,

use_case_title AS useCaseTitle,

domain_stream AS domainStream,

owner_user_name AS ownerUserName,

owner_email AS ownerEmail,

enabled_yn AS enabled

FROM uat_use_case

WHERE use_case_code = p_code;

RETURN rc;

END;

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Issue create / get / update

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

PROCEDURE issue_create(p_body CLOB, p_user_name VARCHAR2, p_issue_id OUT
NUMBER) IS

obj JSON_OBJECT_T;

v_solution_type VARCHAR2(10);

v_application_module VARCHAR2(100);

v_environment VARCHAR2(50);

v_domain_stream VARCHAR2(100);

v_use_case_code VARCHAR2(50);

v_record_type VARCHAR2(20);

v_severity VARCHAR2(10);

v_title VARCHAR2(120);

v_expected CLOB;

v_actual CLOB;

v_steps CLOB;

v_repro CHAR(1);

BEGIN

obj := JSON_OBJECT_T.parse(p_body);

v_solution_type := get_json_string(obj,\'solutionType\');

v_application_module := get_json_string(obj,\'applicationModule\');

v_environment := get_json_string(obj,\'environment\');

v_domain_stream := get_json_string(obj,\'domainStream\');

v_use_case_code := get_json_string(obj,\'useCaseCode\');

v_record_type := get_json_string(obj,\'recordType\');

v_severity := get_json_string(obj,\'severity\');

v_title := get_json_string(obj,\'title\');

v_expected := get_json_clob(obj,\'expectedBehavior\');

v_actual := get_json_clob(obj,\'actualBehavior\');

v_steps := get_json_clob(obj,\'stepsToReproduce\');

v_repro := NVL(UPPER(get_json_string(obj,\'reproducible\')), \'Y\');

\-- Validazioni minime

IF is_null_or_empty(v_solution_type) OR
is_null_or_empty(v_application_module)

OR is_null_or_empty(v_environment) OR is_null_or_empty(v_domain_stream)

OR is_null_or_empty(v_record_type) OR is_null_or_empty(v_severity)

OR is_null_or_empty(v_title)

OR v_expected IS NULL OR v_actual IS NULL OR v_steps IS NULL

THEN

raise_bad_request(\'VALIDATION_ERROR\', \'Campi obbligatori mancanti per
creazione issue\');

END IF;

IF NOT lov_exists(\'ENVIRONMENT\', UPPER(v_environment)) THEN

raise_bad_request(\'INVALID_LOV\', \'Environment non valido:
\'\|\|v_environment);

END IF;

IF NOT lov_exists(\'DOMAIN_STREAM\', UPPER(v_domain_stream)) THEN

raise_bad_request(\'INVALID_LOV\', \'Domain stream non valido:
\'\|\|v_domain_stream);

END IF;

IF NOT lov_exists(\'RECORD_TYPE\', UPPER(v_record_type)) THEN

raise_bad_request(\'INVALID_LOV\', \'Record type non valido:
\'\|\|v_record_type);

END IF;

IF NOT lov_exists(\'SEVERITY\', UPPER(v_severity)) THEN

raise_bad_request(\'INVALID_LOV\', \'Severity non valida:
\'\|\|v_severity);

END IF;

INSERT INTO uat_issue(

created_by, solution_type, application_module, environment,
domain_stream,

use_case_code, page_url, tenant_pod, app_version, test_case_id,
uat_step_ref,

record_type, severity, priority, frequency, impact_flags,

title, expected_behavior, actual_behavior, error_message,

prerequisites, steps_to_reproduce, reproducible_yn, notes,

status, component, assignee, target_release, context_payload

) VALUES (

p_user_name, v_solution_type, v_application_module,
UPPER(v_environment), UPPER(v_domain_stream),

v_use_case_code,

get_json_string(obj,\'pageUrl\'),

get_json_string(obj,\'tenantPod\'),

get_json_string(obj,\'appVersion\'),

get_json_string(obj,\'testCaseId\'),

get_json_string(obj,\'uatStepRef\'),

UPPER(v_record_type), UPPER(v_severity),

get_json_string(obj,\'priority\'),

get_json_string(obj,\'frequency\'),

get_json_string(obj,\'impactFlags\'),

v_title, v_expected, v_actual, get_json_clob(obj,\'errorMessage\'),

get_json_clob(obj,\'prerequisites\'), v_steps, CASE WHEN v_repro=\'N\'
THEN \'N\' ELSE \'Y\' END, get_json_clob(obj,\'notes\'),

\'NEW\',

get_json_string(obj,\'component\'),

get_json_string(obj,\'assignee\'),

get_json_string(obj,\'targetRelease\'),

get_json_clob(obj,\'contextPayload\')

)

RETURNING issue_id INTO p_issue_id;

add_status_hist(p_issue_id, NULL, \'NEW\', p_user_name, \'Issue
creata\');

END;

FUNCTION issue_get(p_issue_id NUMBER) RETURN SYS_REFCURSOR IS

rc SYS_REFCURSOR;

BEGIN

OPEN rc FOR

SELECT \*

FROM uat_issue

WHERE issue_id = p_issue_id;

RETURN rc;

END;

PROCEDURE issue_update(p_issue_id NUMBER, p_body CLOB, p_user_name
VARCHAR2) IS

obj JSON_OBJECT_T;

BEGIN

obj := JSON_OBJECT_T.parse(p_body);

UPDATE uat_issue

SET updated_on = SYSTIMESTAMP,

updated_by = p_user_name,

title = COALESCE(get_json_string(obj,\'title\'), title),

expected_behavior = COALESCE(get_json_clob(obj,\'expectedBehavior\'),
expected_behavior),

actual_behavior = COALESCE(get_json_clob(obj,\'actualBehavior\'),
actual_behavior),

error_message = COALESCE(get_json_clob(obj,\'errorMessage\'),
error_message),

prerequisites = COALESCE(get_json_clob(obj,\'prerequisites\'),
prerequisites),

steps_to_reproduce = COALESCE(get_json_clob(obj,\'stepsToReproduce\'),
steps_to_reproduce),

notes = COALESCE(get_json_clob(obj,\'notes\'), notes),

page_url = COALESCE(get_json_string(obj,\'pageUrl\'), page_url),

app_version = COALESCE(get_json_string(obj,\'appVersion\'),
app_version),

tenant_pod = COALESCE(get_json_string(obj,\'tenantPod\'), tenant_pod),

component = COALESCE(get_json_string(obj,\'component\'), component),

assignee = COALESCE(get_json_string(obj,\'assignee\'), assignee),

target_release = COALESCE(get_json_string(obj,\'targetRelease\'),
target_release)

WHERE issue_id = p_issue_id;

IF SQL%ROWCOUNT = 0 THEN

raise_bad_request(\'NOT_FOUND\', \'Issue non trovata: \'\|\|p_issue_id);

END IF;

END;

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Search issues: filtri + sorting + paginazione + meta

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

FUNCTION issue_search_json(

p_page NUMBER,

p_page_size NUMBER,

p_sort VARCHAR2,

p_status VARCHAR2,

p_severity VARCHAR2,

p_priority VARCHAR2,

p_record_type VARCHAR2,

p_domain_stream VARCHAR2,

p_environment VARCHAR2,

p_solution_type VARCHAR2,

p_assignee VARCHAR2,

p_created_by VARCHAR2,

p_retest_assigned_to VARCHAR2,

p_use_case_code VARCHAR2,

p_q VARCHAR2,

p_created_from VARCHAR2,

p_created_to VARCHAR2,

p_updated_from VARCHAR2,

p_updated_to VARCHAR2,

p_age_days_min NUMBER

) RETURN CLOB IS

v_page NUMBER := NVL(p_page,1);

v_ps NUMBER := NVL(p_page_size,20);

v_offset NUMBER;

v_sort VARCHAR2(200) := NVL(p_sort,\'createdOn:desc\');

v_where VARCHAR2(4000) := \' WHERE 1=1 \';

v_order VARCHAR2(4000) := \'\';

v_sql_count VARCHAR2(4000);

v_sql_data VARCHAR2(4000);

v_total NUMBER;

v_json CLOB;

\-- sort whitelist mapping

FUNCTION map_sort_field(p_field VARCHAR2) RETURN VARCHAR2 IS

BEGIN

CASE LOWER(p_field)

WHEN \'createdon\' THEN RETURN \'created_on\';

WHEN \'updatedon\' THEN RETURN \'updated_on\';

WHEN \'severity\' THEN

\-- ordering logico severity

RETURN \'CASE severity WHEN \'\'BLOCKER\'\' THEN 4 WHEN \'\'HIGH\'\'
THEN 3 WHEN \'\'MEDIUM\'\' THEN 2 WHEN \'\'LOW\'\' THEN 1 ELSE 0 END\';

WHEN \'priority\' THEN

RETURN \'CASE priority WHEN \'\'P1\'\' THEN 3 WHEN \'\'P2\'\' THEN 2
WHEN \'\'P3\'\' THEN 1 ELSE 0 END\';

WHEN \'status\' THEN RETURN \'status\';

WHEN \'domainstream\' THEN RETURN \'domain_stream\';

WHEN \'environment\' THEN RETURN \'environment\';

WHEN \'assignee\' THEN RETURN \'assignee\';

ELSE RETURN NULL;

END CASE;

END;

PROCEDURE add_in_filter(p_param VARCHAR2, p_col VARCHAR2) IS

BEGIN

IF NOT is_null_or_empty(p_param) THEN

\-- p_param può essere lista \"A,B,C\": trasformo in regexp per IN-safe
senza dynamic IN list

\-- uso REGEXP_LIKE su token separati da virgola:

v_where := v_where \|\| \' AND REGEXP_LIKE(\'\|\|p_col\|\|\',
\'\'(\^\|,)\'\'\|\|REPLACE(:\'\|\|p_col\|\|\', \'\',\'\',
\'\'\|\'\')\|\|\'\'(,\|\$)\'\') \';

END IF;

END;

\-- Nota: per semplicità e sicurezza, usiamo binds diversi in execute
immediate.

\-- In PL/SQL reale, conviene costruire WHERE con binds nominati e poi
usare DBMS_SQL.

BEGIN

IF v_page \< 1 THEN raise_bad_request(\'INVALID_PAGINATION\',\'page min
1\'); END IF;

IF v_ps \< 1 OR v_ps \> 100 THEN
raise_bad_request(\'INVALID_PAGINATION\',\'pageSize 1..100\'); END IF;

v_offset := (v_page-1)\*v_ps;

\-- filtri equality / IN list con semplice IN via regexp su valore
normalizzato

IF NOT is_null_or_empty(p_status) THEN

v_where := v_where \|\| \' AND status IN (SELECT REGEXP_SUBSTR(:status,
\'\'\[\^,\]+\'\', 1, LEVEL) FROM dual CONNECT BY REGEXP_SUBSTR(:status,
\'\'\[\^,\]+\'\', 1, LEVEL) IS NOT NULL) \';

END IF;

IF NOT is_null_or_empty(p_severity) THEN

v_where := v_where \|\| \' AND severity IN (SELECT
REGEXP_SUBSTR(:severity, \'\'\[\^,\]+\'\', 1, LEVEL) FROM dual CONNECT
BY REGEXP_SUBSTR(:severity, \'\'\[\^,\]+\'\', 1, LEVEL) IS NOT NULL) \';

END IF;

IF NOT is_null_or_empty(p_priority) THEN

v_where := v_where \|\| \' AND priority IN (SELECT
REGEXP_SUBSTR(:priority, \'\'\[\^,\]+\'\', 1, LEVEL) FROM dual CONNECT
BY REGEXP_SUBSTR(:priority, \'\'\[\^,\]+\'\', 1, LEVEL) IS NOT NULL) \';

END IF;

IF NOT is_null_or_empty(p_record_type) THEN

v_where := v_where \|\| \' AND record_type IN (SELECT
REGEXP_SUBSTR(:record_type, \'\'\[\^,\]+\'\', 1, LEVEL) FROM dual
CONNECT BY REGEXP_SUBSTR(:record_type, \'\'\[\^,\]+\'\', 1, LEVEL) IS
NOT NULL) \';

END IF;

IF NOT is_null_or_empty(p_domain_stream) THEN v_where := v_where \|\| \'
AND domain_stream = UPPER(:domain_stream) \'; END IF;

IF NOT is_null_or_empty(p_environment) THEN v_where := v_where \|\| \'
AND environment = UPPER(:environment) \'; END IF;

IF NOT is_null_or_empty(p_solution_type) THEN v_where := v_where \|\| \'
AND solution_type = :solution_type \'; END IF;

IF NOT is_null_or_empty(p_assignee) THEN v_where := v_where \|\| \' AND
assignee = :assignee \'; END IF;

IF NOT is_null_or_empty(p_created_by) THEN v_where := v_where \|\| \'
AND created_by = :created_by \'; END IF;

IF NOT is_null_or_empty(p_retest_assigned_to) THEN v_where := v_where
\|\| \' AND retest_assigned_to = :retest_assigned_to \'; END IF;

IF NOT is_null_or_empty(p_use_case_code) THEN v_where := v_where \|\| \'
AND use_case_code = :use_case_code \'; END IF;

IF NOT is_null_or_empty(p_q) THEN

v_where := v_where \|\| \' AND (UPPER(title) LIKE
\'\'%\'\'\|\|UPPER(:q)\|\|\'\'%\'\' ) \';

END IF;

IF NOT is_null_or_empty(p_created_from) THEN v_where := v_where \|\| \'
AND created_on \>= TO_TIMESTAMP(:created_from) \'; END IF;

IF NOT is_null_or_empty(p_created_to) THEN v_where := v_where \|\| \'
AND created_on \<= TO_TIMESTAMP(:created_to) \'; END IF;

IF NOT is_null_or_empty(p_updated_from) THEN v_where := v_where \|\| \'
AND updated_on \>= TO_TIMESTAMP(:updated_from) \'; END IF;

IF NOT is_null_or_empty(p_updated_to) THEN v_where := v_where \|\| \'
AND updated_on \<= TO_TIMESTAMP(:updated_to) \'; END IF;

IF p_age_days_min IS NOT NULL THEN

v_where := v_where \|\| \' AND (SYSTIMESTAMP - created_on) \>=
NUMTODSINTERVAL(:age_days_min,\'\'DAY\'\') \';

END IF;

\-- sorting parsing: \"field:dir,field2:dir2\"

DECLARE

v_part VARCHAR2(200);

v_field VARCHAR2(100);

v_dir VARCHAR2(10);

v_col VARCHAR2(400);

v_idx NUMBER := 1;

v_first BOOLEAN := TRUE;

BEGIN

LOOP

v_part := REGEXP_SUBSTR(v_sort, \'\[\^,\]+\', 1, v_idx);

EXIT WHEN v_part IS NULL;

v_field := REGEXP_SUBSTR(v_part, \'\^\[\^:\]+\');

v_dir := LOWER(NVL(REGEXP_SUBSTR(v_part, \':(asc\|desc)

, 1, 1, NULL, 1), \'desc\'));

v_col := map_sort_field(v_field);

IF v_col IS NULL THEN

raise_bad_request(\'INVALID_SORT_FIELD\',\'Sort non consentito:
\'\|\|v_field);

END IF;

IF v_first THEN

v_order := \' ORDER BY \';

v_first := FALSE;

ELSE

v_order := v_order \|\| \', \';

END IF;

v_order := v_order \|\| v_col \|\| \' \' \|\| CASE WHEN v_dir=\'asc\'
THEN \'ASC\' ELSE \'DESC\' END;

v_idx := v_idx + 1;

END LOOP;

IF v_order IS NULL OR v_order = \'\' THEN

v_order := \' ORDER BY created_on DESC \';

END IF;

END;

v_sql_count := \'SELECT COUNT(\*) FROM uat_issue \' \|\| v_where;

v_sql_data := \'SELECT issue_id, title, domain_stream, environment,
solution_type, record_type, severity, priority, status, assignee,
retest_assigned_to, use_case_code, created_on, updated_on

FROM uat_issue \' \|\| v_where \|\| v_order \|\|

\' OFFSET :offset ROWS FETCH NEXT :fetch ROWS ONLY\';

\-- COUNT

EXECUTE IMMEDIATE v_sql_count INTO v_total

USING p_status, p_severity, p_priority, p_record_type,

p_domain_stream, p_environment, p_solution_type, p_assignee,
p_created_by,

p_retest_assigned_to, p_use_case_code, p_q, p_created_from,
p_created_to, p_updated_from, p_updated_to, p_age_days_min;

\-- DATA -\> JSON

DECLARE

c SYS_REFCURSOR;

v_arr JSON_ARRAY_T := JSON_ARRAY_T();

v_obj JSON_OBJECT_T;

v_issue_id NUMBER;

v_title VARCHAR2(120);

v_ds VARCHAR2(100);

v_env VARCHAR2(50);

v_sol VARCHAR2(10);

v_rt VARCHAR2(20);

v_sev VARCHAR2(10);

v_pri VARCHAR2(5);

v_st VARCHAR2(30);

v_ass VARCHAR2(128);

v_retest VARCHAR2(128);

v_uc VARCHAR2(50);

v_co TIMESTAMP;

v_uo TIMESTAMP;

v_meta JSON_OBJECT_T := JSON_OBJECT_T();

v_root JSON_OBJECT_T := JSON_OBJECT_T();

v_total_pages NUMBER;

BEGIN

OPEN c FOR v_sql_data

USING p_status, p_severity, p_priority, p_record_type,

p_domain_stream, p_environment, p_solution_type, p_assignee,
p_created_by,

p_retest_assigned_to, p_use_case_code, p_q, p_created_from,
p_created_to, p_updated_from, p_updated_to, p_age_days_min,

v_offset, v_ps;

LOOP

FETCH c INTO v_issue_id, v_title, v_ds, v_env, v_sol, v_rt, v_sev,
v_pri, v_st, v_ass, v_retest, v_uc, v_co, v_uo;

EXIT WHEN c%NOTFOUND;

v_obj := JSON_OBJECT_T();

v_obj.put(\'issueId\', v_issue_id);

v_obj.put(\'title\', v_title);

v_obj.put(\'domainStream\', v_ds);

v_obj.put(\'environment\', v_env);

v_obj.put(\'solutionType\', v_sol);

v_obj.put(\'recordType\', v_rt);

v_obj.put(\'severity\', v_sev);

IF v_pri IS NOT NULL THEN v_obj.put(\'priority\', v_pri); END IF;

v_obj.put(\'status\', v_st);

IF v_ass IS NOT NULL THEN v_obj.put(\'assignee\', v_ass); END IF;

IF v_retest IS NOT NULL THEN v_obj.put(\'retestAssignedTo\', v_retest);
END IF;

IF v_uc IS NOT NULL THEN v_obj.put(\'useCaseCode\', v_uc); END IF;

v_obj.put(\'createdOn\', TO_CHAR(v_co,
\'YYYY-MM-DD\"T\"HH24:MI:SS.FF3TZH:TZM\'));

IF v_uo IS NOT NULL THEN v_obj.put(\'updatedOn\', TO_CHAR(v_uo,
\'YYYY-MM-DD\"T\"HH24:MI:SS.FF3TZH:TZM\')); END IF;

v_arr.append(v_obj);

END LOOP;

CLOSE c;

v_total_pages := CEIL(v_total / v_ps);

v_meta.put(\'page\', v_page);

v_meta.put(\'pageSize\', v_ps);

v_meta.put(\'total\', v_total);

v_meta.put(\'totalPages\', v_total_pages);

v_meta.put(\'sort\', v_sort);

v_root.put(\'data\', v_arr);

v_root.put(\'meta\', v_meta);

v_json := v_root.to_clob();

END;

RETURN v_json;

END;

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Comments

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

FUNCTION comments_list(p_issue_id NUMBER) RETURN SYS_REFCURSOR IS

rc SYS_REFCURSOR;

BEGIN

OPEN rc FOR

SELECT comment_id AS commentId, issue_id AS issueId,

comment_text AS commentText, visibility,

comment_on AS commentOn, comment_by AS commentBy

FROM uat_issue_comment

WHERE issue_id = p_issue_id

ORDER BY comment_on DESC;

RETURN rc;

END;

PROCEDURE comment_add(p_issue_id NUMBER, p_body CLOB, p_user_name
VARCHAR2) IS

obj JSON_OBJECT_T;

v_text CLOB;

v_vis VARCHAR2(10);

BEGIN

obj := JSON_OBJECT_T.parse(p_body);

v_text := get_json_clob(obj,\'commentText\');

v_vis := NVL(UPPER(get_json_string(obj,\'visibility\')), \'PUBLIC\');

IF v_text IS NULL THEN
raise_bad_request(\'VALIDATION_ERROR\',\'commentText obbligatorio\');
END IF;

IF v_vis NOT IN (\'PUBLIC\',\'INTERNAL\') THEN
raise_bad_request(\'VALIDATION_ERROR\',\'visibility non valida\'); END
IF;

INSERT INTO uat_issue_comment(issue_id, comment_text, visibility,
comment_by)

VALUES (p_issue_id, v_text, v_vis, p_user_name);

END;

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Attachments

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

FUNCTION attachments_list(p_issue_id NUMBER) RETURN SYS_REFCURSOR IS

rc SYS_REFCURSOR;

BEGIN

OPEN rc FOR

SELECT attachment_id AS attachmentId, issue_id AS issueId,

file_name AS fileName, mime_type AS mimeType, file_size AS fileSize,

description, uploaded_on AS uploadedOn, uploaded_by AS uploadedBy

FROM uat_issue_attachment

WHERE issue_id = p_issue_id

ORDER BY uploaded_on DESC;

RETURN rc;

END;

PROCEDURE attachment_add(p_issue_id NUMBER, p_file_name VARCHAR2, p_mime
VARCHAR2, p_size NUMBER,

p_blob BLOB, p_desc VARCHAR2, p_user_name VARCHAR2) IS

BEGIN

INSERT INTO uat_issue_attachment(issue_id, file_name, mime_type,
file_size, file_content, description, uploaded_by)

VALUES (p_issue_id, p_file_name, p_mime, p_size, p_blob, p_desc,
p_user_name);

END;

PROCEDURE attachment_get_content(p_attachment_id NUMBER) IS

v_blob BLOB;

v_mime VARCHAR2(100);

v_name VARCHAR2(255);

BEGIN

SELECT file_content, mime_type, file_name

INTO v_blob, v_mime, v_name

FROM uat_issue_attachment

WHERE attachment_id = p_attachment_id;

owa_util.mime_header(NVL(v_mime,\'application/octet-stream\'), FALSE);

htp.p(\'Content-Disposition: inline;
filename=\"\'\|\|REPLACE(v_name,\'\"\',\'\')\|\|\'\"\');

owa_util.http_header_close;

wpg_docload.download_file(v_blob);

END;

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Transition (workflow)

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

FUNCTION transition_json(p_issue_id NUMBER, p_body CLOB, p_user_name
VARCHAR2) RETURN CLOB IS

obj JSON_OBJECT_T;

v_action VARCHAR2(50);

payload JSON_OBJECT_T;

v_old_status VARCHAR2(30);

v_new_status VARCHAR2(30);

v_solution_type VARCHAR2(10);

v_use_case_code VARCHAR2(50);

v_owner_user VARCHAR2(128);

v_owner_email VARCHAR2(256);

v_subject VARCHAR2(300);

v_html CLOB;

BEGIN

obj := JSON_OBJECT_T.parse(p_body);

v_action := UPPER(get_json_string(obj,\'action\'));

IF v_action IS NULL THEN raise_bad_request(\'VALIDATION_ERROR\',\'action
obbligatoria\'); END IF;

IF obj.has(\'payload\') THEN

payload := obj.get_object(\'payload\');

ELSE

payload := JSON_OBJECT_T();

END IF;

SELECT status, solution_type, use_case_code

INTO v_old_status, v_solution_type, v_use_case_code

FROM uat_issue

WHERE issue_id = p_issue_id

FOR UPDATE;

\-- Determina transizione

IF v_action = \'PROPOSE_SOLUTION\' THEN

IF v_old_status \<\> \'IN_PROGRESS\' THEN
raise_bad_request(\'INVALID_TRANSITION\',\'Da \'\|\|v_old_status\|\|\'
non puoi PROPOSE_SOLUTION\'); END IF;

IF get_json_clob(payload,\'resolutionNotes\') IS NULL THEN
raise_bad_request(\'VALIDATION_ERROR\',\'resolutionNotes
obbligatorio\'); END IF;

IF v_solution_type = \'PaaS\' AND
is_null_or_empty(get_json_string(payload,\'fixVersion\')) THEN

raise_bad_request(\'VALIDATION_ERROR\',\'fixVersion obbligatorio per
PaaS\');

END IF;

v_new_status := \'SOLUTION_PROPOSED\';

UPDATE uat_issue

SET status = v_new_status,

resolution_notes = get_json_clob(payload,\'resolutionNotes\'),

fix_version = COALESCE(get_json_string(payload,\'fixVersion\'),
fix_version),

fix_environment = COALESCE(get_json_string(payload,\'fixEnvironment\'),
fix_environment),

updated_on = SYSTIMESTAMP,

updated_by = p_user_name

WHERE issue_id = p_issue_id;

add_status_hist(p_issue_id, v_old_status, v_new_status, p_user_name,
\'Soluzione proposta\');

add_system_comment(p_issue_id, \'System: soluzione proposta.
FixVersion=\'\|\|COALESCE(get_json_string(payload,\'fixVersion\'),\'n/a\'),
p_user_name);

ELSIF v_action = \'SEND_TO_RETEST\' THEN

IF v_old_status \<\> \'SOLUTION_PROPOSED\' THEN
raise_bad_request(\'INVALID_TRANSITION\',\'Da \'\|\|v_old_status\|\|\'
non puoi SEND_TO_RETEST\'); END IF;

\-- lookup Owner Use Case (username+email)

IF v_use_case_code IS NOT NULL THEN

BEGIN

SELECT owner_user_name, owner_email INTO v_owner_user, v_owner_email

FROM uat_use_case

WHERE use_case_code = v_use_case_code AND enabled_yn=\'Y\';

EXCEPTION

WHEN NO_DATA_FOUND THEN

v_owner_user := NULL;

v_owner_email := NULL;

END;

END IF;

v_new_status := \'RETEST\';

UPDATE uat_issue

SET status = v_new_status,

retest_assigned_to = COALESCE(v_owner_user, created_by),

retest_instructions =
COALESCE(get_json_clob(payload,\'retestInstructions\'),
retest_instructions),

updated_on = SYSTIMESTAMP,

updated_by = p_user_name

WHERE issue_id = p_issue_id;

add_status_hist(p_issue_id, v_old_status, v_new_status, p_user_name,
\'Inviato in re-test\');

\-- Notifica email a owner_email (se presente) altrimenti nulla (o
triage)

IF v_owner_email IS NOT NULL THEN

v_subject := \'\[UAT\] Issue #\'\|\|p_issue_id\|\|\' - In re-test\';

v_html := \'\<p\>Issue \<b\>#\'\|\|p_issue_id\|\|\'\</b\> è stata
inviata in \<b\>re-test\</b\>.\</p\>\';

send_email_best_effort(v_owner_email, v_subject, v_html);

END IF;

ELSIF v_action = \'CONFIRM_RESOLUTION\' THEN

IF v_old_status \<\> \'RETEST\' THEN
raise_bad_request(\'INVALID_TRANSITION\',\'Da \'\|\|v_old_status\|\|\'
non puoi CONFIRM_RESOLUTION\'); END IF;

IF UPPER(get_json_string(payload,\'retestOutcome\')) \<\> \'PASS\' THEN

raise_bad_request(\'VALIDATION_ERROR\',\'retestOutcome deve essere
PASS\');

END IF;

v_new_status := \'RESOLVED\';

UPDATE uat_issue

SET status = v_new_status,

retest_outcome = \'PASS\',

retest_notes = COALESCE(get_json_clob(payload,\'retestNotes\'),
retest_notes),

resolved_on = SYSTIMESTAMP,

resolved_by = p_user_name,

updated_on = SYSTIMESTAMP,

updated_by = p_user_name

WHERE issue_id = p_issue_id;

add_status_hist(p_issue_id, v_old_status, v_new_status, p_user_name,
\'Risolto (PASS)\');

add_system_comment(p_issue_id, \'System: re-test PASS.
\'\|\|SUBSTR(COALESCE(get_json_string(payload,\'retestNotes\'),\'\'),1,300),
p_user_name);

ELSIF v_action = \'REOPEN_FAIL\' THEN

IF v_old_status \<\> \'RETEST\' THEN
raise_bad_request(\'INVALID_TRANSITION\',\'Da \'\|\|v_old_status\|\|\'
non puoi REOPEN_FAIL\'); END IF;

IF UPPER(get_json_string(payload,\'retestOutcome\')) \<\> \'FAIL\' THEN

raise_bad_request(\'VALIDATION_ERROR\',\'retestOutcome deve essere
FAIL\');

END IF;

IF get_json_clob(payload,\'retestNotes\') IS NULL THEN

raise_bad_request(\'VALIDATION_ERROR\',\'retestNotes obbligatorio in
caso FAIL\');

END IF;

v_new_status := \'IN_PROGRESS\';

UPDATE uat_issue

SET status = v_new_status,

retest_outcome = \'FAIL\',

retest_notes = get_json_clob(payload,\'retestNotes\'),

updated_on = SYSTIMESTAMP,

updated_by = p_user_name

WHERE issue_id = p_issue_id;

add_status_hist(p_issue_id, v_old_status, v_new_status, p_user_name,
\'Riaperto (FAIL)\');

add_system_comment(p_issue_id, \'System: re-test FAIL.
\'\|\|SUBSTR(get_json_string(payload,\'retestNotes\'),1,300),
p_user_name);

ELSIF v_action = \'CLOSE\' THEN

IF v_old_status \<\> \'RESOLVED\' THEN
raise_bad_request(\'INVALID_TRANSITION\',\'Da \'\|\|v_old_status\|\|\'
non puoi CLOSE\'); END IF;

IF is_null_or_empty(get_json_string(payload,\'closureReason\')) THEN

raise_bad_request(\'VALIDATION_ERROR\',\'closureReason obbligatorio\');

END IF;

v_new_status := \'CLOSED\';

UPDATE uat_issue

SET status = v_new_status,

closure_reason = get_json_string(payload,\'closureReason\'),

closure_evidence_link =
COALESCE(get_json_string(payload,\'closureEvidenceLink\'),
closure_evidence_link),

closed_on = SYSTIMESTAMP,

closed_by = p_user_name,

updated_on = SYSTIMESTAMP,

updated_by = p_user_name

WHERE issue_id = p_issue_id;

add_status_hist(p_issue_id, v_old_status, v_new_status, p_user_name,
\'Chiuso\');

ELSE

raise_bad_request(\'INVALID_ACTION\', \'Azione non supportata:
\'\|\|v_action);

END IF;

RETURN JSON_OBJECT(\'data\' VALUE JSON_OBJECT(\'issueId\' VALUE
p_issue_id, \'status\' VALUE v_new_status)).to_clob();

END;

END uat_api_pkg;

/

Mostra più linee

✅ Risultato: ora hai tutte le logiche principali su DB (validazione +
workflow + query + meta + assegnazione owner + email best-effort).

**4) Script 04 --- ORDS: definizione endpoint
(module/templates/handlers)**

Qui ti do i template ORDS **già collegati al package**.

Nota: i dettagli di ORDS.DEFINE\_\* possono variare leggermente per
versione, ma questa struttura è "standard".

SQL

BEGIN

ORDS.ENABLE_SCHEMA(

p_enabled =\> TRUE,

p_schema =\> USER,

p_url_mapping_type =\> \'BASE_PATH\',

p_url_mapping_pattern =\> \'uat\',

p_auto_rest_auth =\> FALSE

);

ORDS.DEFINE_MODULE(

p_module_name =\> \'uat_api_v1\',

p_base_path =\> \'/api/v1/\',

p_items_per_page =\> 25

);

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- LOV: GET /uat/api/v1/lov/{type}

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

ORDS.DEFINE_TEMPLATE(\'uat_api_v1\', \'lov/:type\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'lov/:type\',

p_method =\> \'GET\',

p_source_type =\> ORDS.SOURCE_TYPE_QUERY,

p_source =\> q\'\[

SELECT lov_code AS code, lov_label AS label, sort_order AS sortOrder

FROM uat_lov

WHERE lov_type = UPPER(:type) AND enabled_yn=\'Y\'

ORDER BY sort_order, lov_label

\]\'

);

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Use cases

\-- GET /usecases

\-- GET /usecases/{code}

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

ORDS.DEFINE_TEMPLATE(\'uat_api_v1\', \'usecases\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'usecases\',

p_method =\> \'GET\',

p_source_type =\> ORDS.SOURCE_TYPE_QUERY,

p_source =\> q\'\[

SELECT use_case_code AS \"useCaseCode\",

use_case_title AS \"useCaseTitle\",

domain_stream AS \"domainStream\",

owner_user_name AS \"ownerUserName\",

owner_email AS \"ownerEmail\",

enabled_yn AS \"enabled\"

FROM uat_use_case

WHERE (:domainStream IS NULL OR domain_stream = UPPER(:domainStream))

AND (:enabled IS NULL OR enabled_yn = UPPER(:enabled))

ORDER BY domain_stream, use_case_code

\]\'

);

ORDS.DEFINE_TEMPLATE(\'uat_api_v1\', \'usecases/:code\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'usecases/:code\',

p_method =\> \'GET\',

p_source_type =\> ORDS.SOURCE_TYPE_QUERY,

p_source =\> q\'\[

SELECT use_case_code AS \"useCaseCode\",

use_case_title AS \"useCaseTitle\",

domain_stream AS \"domainStream\",

owner_user_name AS \"ownerUserName\",

owner_email AS \"ownerEmail\",

enabled_yn AS \"enabled\"

FROM uat_use_case

WHERE use_case_code = :code

\]\'

);

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Issues list: GET /issues (custom JSON with meta)

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

ORDS.DEFINE_TEMPLATE(\'uat_api_v1\', \'issues\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'issues\',

p_method =\> \'GET\',

p_source_type =\> ORDS.SOURCE_TYPE_PLSQL,

p_source =\> q\'\[

DECLARE

v_json CLOB;

BEGIN

v_json := uat_api_pkg.issue_search_json(

:page, :pageSize, :sort,

:status, :severity, :priority, :recordType,

:domainStream, :environment, :solutionType,

:assignee, :createdBy, :retestAssignedTo, :useCaseCode,

:q, :createdFrom, :createdTo, :updatedFrom, :updatedTo,

:ageDaysMin

);

uat_api_pkg.write_json(v_json);

END;

\]\'

);

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Issues: POST /issues

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'issues\',

p_method =\> \'POST\',

p_source_type =\> ORDS.SOURCE_TYPE_PLSQL,

p_source =\> q\'\[

DECLARE

v_issue_id NUMBER;

BEGIN

uat_api_pkg.issue_create(:body, NVL(:userName,\'anonymous\'),
v_issue_id);

uat_api_pkg.write_json(

JSON_OBJECT(\'data\' VALUE JSON_OBJECT(\'issueId\' VALUE v_issue_id,
\'status\' VALUE \'NEW\')).to_clob()

);

END;

\]\'

);

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Issue detail: GET /issues/{id}

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

ORDS.DEFINE_TEMPLATE(\'uat_api_v1\', \'issues/:id\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'issues/:id\',

p_method =\> \'GET\',

p_source_type =\> ORDS.SOURCE_TYPE_QUERY,

p_source =\> q\'\[

SELECT \*

FROM uat_issue

WHERE issue_id = TO_NUMBER(:id)

\]\'

);

\-- Update (PUT/PATCH): /issues/{id}

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'issues/:id\',

p_method =\> \'PATCH\',

p_source_type =\> ORDS.SOURCE_TYPE_PLSQL,

p_source =\> q\'\[

BEGIN

uat_api_pkg.issue_update(TO_NUMBER(:id), :body,
NVL(:userName,\'anonymous\'));

uat_api_pkg.write_json(JSON_OBJECT(\'data\' VALUE
JSON_OBJECT(\'issueId\' VALUE TO_NUMBER(:id))).to_clob());

END;

\]\'

);

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Workflow: POST /issues/{id}/transition

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

ORDS.DEFINE_TEMPLATE(\'uat_api_v1\', \'issues/:id/transition\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'issues/:id/transition\',

p_method =\> \'POST\',

p_source_type =\> ORDS.SOURCE_TYPE_PLSQL,

p_source =\> q\'\[

DECLARE

v_json CLOB;

BEGIN

v_json := uat_api_pkg.transition_json(TO_NUMBER(:id), :body,
NVL(:userName,\'anonymous\'));

uat_api_pkg.write_json(v_json);

END;

\]\'

);

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-- Comments: GET/POST /issues/{id}/comments

\--
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

ORDS.DEFINE_TEMPLATE(\'uat_api_v1\', \'issues/:id/comments\');

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'issues/:id/comments\',

p_method =\> \'GET\',

p_source_type =\> ORDS.SOURCE_TYPE_QUERY,

p_source =\> q\'\[

SELECT comment_id AS \"commentId\",

issue_id AS \"issueId\",

comment_text AS \"commentText\",

visibility,

comment_on AS \"commentOn\",

comment_by AS \"commentBy\"

FROM uat_issue_comment

WHERE issue_id = TO_NUMBER(:id)

ORDER BY comment_on DESC

\]\'

);

ORDS.DEFINE_HANDLER(

p_module_name =\> \'uat_api_v1\',

p_pattern =\> \'issues/:id/comments\',

p_method =\> \'POST\',

p_source_type =\> ORDS.SOURCE_TYPE_PLSQL,

p_source =\> q\'\[

BEGIN

uat_api_pkg.comment_add(TO_NUMBER(:id), :body,
NVL(:userName,\'anonymous\'));

uat_api_pkg.write_json(JSON_OBJECT(\'data\' VALUE
JSON_OBJECT(\'issueId\' VALUE TO_NUMBER(:id))).to_clob());

END;

\]\'

);

COMMIT;

END;

/

Mostra più linee

Nota: ho lasciato la gestione :userName come parametro/query/header
"semplificato". In produzione è meglio derivarlo da SSO/OAuth, ma per
UAT va bene per partire.

**5) Come testare subito (senza dev)**

Esempi con curl (o Postman):

**5.1 Creazione issue**

Shell

curl -X POST
\"https://\<HOST\>/ords/uat/api/v1/issues?userName=giuseppe.fiorillo\"
\\

-H \"Content-Type: application/json\" \\

-d \'{

\"solutionType\":\"SaaS\",

\"applicationModule\":\"Fusion Payables\",

\"environment\":\"SVI\",

\"domainStream\":\"CONTABILITA\",

\"useCaseCode\":\"UC_ACC_001\",

\"recordType\":\"INCIDENT\",

\"severity\":\"HIGH\",

\"title\":\"Errore conferma fattura\",

\"expectedBehavior\":\"Conferma ok\",

\"actualBehavior\":\"Errore\...\",

\"stepsToReproduce\":\"1) \... 2) \...\"

}\'

Mostra più linee

**5.2 Lista con filtri + meta**

Shell

curl
\"https://\<HOST\>/ords/uat/api/v1/issues?domainStream=CONTABILITA&environment=SVI&status=NEW,TRIAGE&sort=severity:desc,createdOn:desc&page=1&pageSize=20\"

Mostra più linee

**5.3 Transition: invio in re-test (assegna owner use case)**

Shell

curl -X POST
\"https://\<HOST\>/ords/uat/api/v1/issues/123/transition?userName=triage.user\"
\\

-H \"Content-Type: application/json\" \\

-d \'{

\"action\":\"SEND_TO_RETEST\",

\"payload\":{

\"retestInstructions\":\"Ripetere step 1-3; verificare conferma.\"

}

}\'

Mostra più linee

**6) VBCS: come collegarlo senza sviluppatore (operativo)**

1.  In VBCS → **Service Connections** → **Define by Endpoint**

    -   Base URL: https://\<HOST\>/ords/uat/api/v1/

2.  Crea Data Providers per:

    -   LOV (/lov/{type})

    -   Use cases (/usecases)

    -   Issues list (/issues)

    -   Issue detail (/issues/{id})

    -   Transition (/issues/{id}/transition)

3.  Crea pagine:

    -   List: chiama /issues con filtri UI → bind a variables

    -   Form: POST /issues

    -   Detail: GET /issues/{id} + Comments + Attachments

4.  Bottoni workflow: Action Chain → POST transition

{{referencesBody}}

Qui sotto ti do un **runbook + script completi** (copiaincolla), con
**due varianti** in base alla versione ORDS:

-   ✅ **ORDS 24.3+** → usare i package **ORDS_SECURITY /
    ORDS_SECURITY_ADMIN** (consigliato perché i vecchi OAUTH/OAUTH_ADMIN
    sono deprecati)
    [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
    [\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

-   ⚠️ **ORDS ≤ 24.2** → usare **ORDS_METADATA.OAUTH_ADMIN /
    ORDS_METADATA.OAUTH** (legacy)

Nota importante (trasparente): con **OAuth2 Client Credentials**
l'identità "vista" da ORDS è quella del **client** (non dell'utente
finale). Questo è normale: nel client-credentials "non c'è un utente in
contesto".\
Per UAT spesso va benissimo; se un domani vuoi *identity propagation*
end-user, si entra in pattern più avanzati.
[\[blogs.oracle.com\]](https://blogs.oracle.com/vbcs/connect-visual-builder-to-rest-apis-secured-by-identity-domainidcs-oauth),
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

**1) Obiettivo di sicurezza (cosa otterrai)**

1.  Tutti gli endpoint /ords/uat/api/v1/\* diventano **protetti** da un
    privilegio ORDS.

2.  Viene creato un **OAuth Client** (confidential) per **grant type =
    client_credentials**.

3.  VBCS usa quel client per ottenere un **token** e chiamare le API con
    Authorization: Bearer \<token\>.
    [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
    [\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

**2) Script "di verifica" (capire quale package usare)**

Esegui in ATP (SQL Worksheet):

SQL

\-- Se ritorna 1, hai ORDS_SECURITY (nuovo modello, ORDS 24.3+)

SELECT COUNT(\*) AS has_ords_security

FROM all_objects

WHERE object_type=\'PACKAGE\'

AND object_name=\'ORDS_SECURITY\';

Mostra più linee

-   Se has_ords_security = 1 → vai a **Sezione 3 (nuovo)**

-   Se 0 → vai a **Sezione 4 (legacy)**

Perché: ORDS 24.3+ ha sostituito i package OAuth storici con
ORDS_SECURITY.\*
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
[\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

**3) ORDS 24.3+ (consigliato) --- Role + Privilege + OAuth client**

**3.1 Crea ROLE + PRIVILEGE + MAPPING URL**

Questo protegge le risorse e collega la protezione ai pattern URL.
[\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

SQL

BEGIN

\-- 1) Role

ORDS.CREATE_ROLE(

p_role_name =\> \'UAT_API_ROLE\'

);

\-- 2) Privilege (collegato al role)

ORDS.CREATE_PRIVILEGE(

p_name =\> \'UAT_API_PRIV\',

p_role_name =\> \'UAT_API_ROLE\',

p_label =\> \'UAT API Access\'

);

\-- 3) Mapping del privilegio a un pattern URL

\-- NB: pattern coerente con il tuo mapping schema \"uat\" e base path
\"/api/v1/\"

ORDS.CREATE_PRIVILEGE_MAPPING(

p_privilege_name =\> \'UAT_API_PRIV\',

p_pattern =\> \'/uat/api/v1/\*\'

);

\-- 4) (Opzionale ma consigliato) associa direttamente il privilegio al
module ORDS

ORDS.SET_MODULE_PRIVILEGE(

p_module_name =\> \'uat_api_v1\',

p_privilege_name =\> \'UAT_API_PRIV\'

);

COMMIT;

END;

/

Mostra più linee

**3.2 Registra OAuth Client (client_credentials) e ottieni Client
ID/Secret**

Esempio con **ORDS_SECURITY.register_client** (nuovo modello)
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
[\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

SQL

SET SERVEROUTPUT ON;

DECLARE

l_client_cred ords_types.t_client_credentials;

BEGIN

l_client_cred := ords_security.register_client(

p_name =\> \'vbcs_uat_client\',

p_grant_type =\> \'client_credentials\',

p_description =\> \'VBCS client for UAT Issue Tracker APIs\',

p_client_secret =\> NULL, \-- genera secret automaticamente

p_support_email =\> \'support@ente.it\',

p_privilege_names =\> \'UAT_API_PRIV\'

);

COMMIT;

dbms_output.put_line(\'CLIENT_ID : \' \|\|
l_client_cred.client_key.client_id);

dbms_output.put_line(\'CLIENT_SECRET : \' \|\|
l_client_cred.client_secret.secret);

END;

/

Mostra più linee

**3.3 Concedi il role al client (fondamentale)**

SQL

BEGIN

ords_security.grant_client_role(

p_client_name =\> \'vbcs_uat_client\',

p_role_name =\> \'UAT_API_ROLE\'

);

COMMIT;

END;

/

Mostra più linee

Client ↔ ruolo ↔ privilegio: è la catena che abilita l'accesso alle
risorse protette.
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
[\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

**4) ORDS ≤ 24.2 (legacy) --- alternativa se non hai ORDS_SECURITY**

Se sei su versioni precedenti, la logica è equivalente ma usando package
legacy (deprecati nelle versioni più recenti).
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
[\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

(Se ti serve davvero questa branch dimmelo e te la metto "chiavi in
mano"; per non appesantire ora, ti ho dato prima la versione
corretta/nuova.)

**5) Token endpoint + test end-to-end (curl/Postman)**

**5.1 Token URL (con il tuo mapping uat)**

In ORDS, l'endpoint standard per token è:

-   https://\<HOST\>/ords/uat/oauth/token
    [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
    [\[jmjcloud.com\]](https://jmjcloud.com/blog/ords-securing-services-using-oauth2-client-credentials/)

**5.2 Ottenere un token (client_credentials)**

Shell

curl -i -k \\

\--user \<CLIENT_ID\>:\<CLIENT_SECRET\> \\

\--data \"grant_type=client_credentials\" \\

https://\<HOST\>/ords/uat/oauth/token

Mostra più linee

Risposta attesa (200):

JSON

{\"access_token\":\"\...\",\"token_type\":\"bearer\",\"expires_in\":3600}

Mostra più linee

[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
[\[jmjcloud.com\]](https://jmjcloud.com/blog/ords-securing-services-using-oauth2-client-credentials/)

**5.3 Chiamare un endpoint protetto**

Shell

curl -i -k \\

-H \"Authorization: Bearer \<ACCESS_TOKEN\>\" \\

https://\<HOST\>/ords/uat/api/v1/issues?page=1&pageSize=20

Mostra più linee

Se chiami senza header Bearer → 401 Unauthorized (atteso).
[\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html),
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

**6) Aggiornamento "pulito" dei tuoi handler ORDS: niente userName in
query**

Con OAuth2 attivo, puoi evitare di passare username. Nel tuo handler
PL/SQL puoi usare l'utente "autenticato" che ORDS espone (tipicamente il
client). Per ORDS, il token viene associato al client e l'accesso è
controllato da ruoli/privilegi.
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
[\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

**6.1 Modifica rapida (nei handler ORDS)**

Dove avevi:

PL/SQL

uat_api_pkg.issue_create(:body, NVL(:userName,\'anonymous\'),
v_issue_id);

Mostra più linee

Sostituisci con:

PL/SQL

uat_api_pkg.issue_create(:body,
NVL(owa_util.get_cgi_env(\'REMOTE_USER\'),\'anonymous\'), v_issue_id);

Mostra più linee

Nota: con client_credentials REMOTE_USER corrisponde al client, quindi
created_by sarà "vbcs_uat_client" (o simile). È coerente con il modello
client-credentials (nessun utente umano in contesto).
[\[blogs.oracle.com\]](https://blogs.oracle.com/vbcs/connect-visual-builder-to-rest-apis-secured-by-identity-domainidcs-oauth),
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

Se vuoi comunque tracciare in DB il **tester umano**, lo puoi fare **in
modo controllato** facendo inserire in payload un campo
reportedByDisplayName "solo a fini audit UAT" (non per autorizzazione).
(Se vuoi te lo preparo pulito.)

**7) Configurazione VBCS (Service Connection) con OAuth2 Client
Credentials**

In VBCS, configura un Backend/Service con OAuth2 **Client Credentials**:
VBCS supporta questo grant type.
[\[blogs.oracle.com\]](https://blogs.oracle.com/vbcs/connect-visual-builder-to-rest-apis-secured-by-identity-domainidcs-oauth)

**Valori:**

-   **Token URL**: https://\<HOST\>/ords/uat/oauth/token
    [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
    [\[jmjcloud.com\]](https://jmjcloud.com/blog/ords-securing-services-using-oauth2-client-credentials/)

-   **Client ID / Client Secret**: quelli stampati dallo script
    register_client
    [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

-   **Base URL** service: https://\<HOST\>/ords/uat/api/v1/

Suggerimento pratico: prima prova con Postman/curl; quando funziona,
VBCS si configura in modo lineare.
[\[blogs.oracle.com\]](https://blogs.oracle.com/vbcs/connect-visual-builder-to-rest-apis-secured-by-identity-domainidcs-oauth),
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

**8) Ultimo check: HTTPS**

OAuth2 richiede normalmente HTTPS; per test locali si può allentare, ma
in ambienti reali tieni HTTPS.

**1) SCRIPT "ONE‑SHOT" ORDS SECURITY + OAUTH2 (client_credentials)**

**1.1 Parametri da impostare (solo 4 righe)**

Lo script funziona **anche se alcune cose esistono già** (gestione
errori "already exists").

-   **DB schema**: INT_ADMIN (ok)

-   **URL mapping pattern ORDS**: di solito coincide con lo schema in
    lowercase, es. int_admin → endpoint \.../ords/int_admin/\.... Se nel
    tuo ORDS avete scelto un pattern diverso, sostituiscilo qui.

-   **Nome modulo**: uat_api_v1 (come da quanto abbiamo definito)

-   **Base path modulo**: /api/v1/

Il token endpoint per ORDS, con mapping int_admin, diventa:\
https://\<HOST\>/ords/int_admin/oauth/token
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
[\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

**1.2 One‑shot (copia/incolla tutto in SQL Worksheet su ATP connesso
come INT_ADMIN)**

SQL

\-- =========================================================

\-- ONE-SHOT ORDS SECURITY + OAUTH2 (client_credentials)

\-- Schema DB: INT_ADMIN

\-- =========================================================

\-- Modifica SOLO questi parametri se necessario:

DECLARE

c_url_mapping_pattern CONSTANT VARCHAR2(128) := \'int_admin\'; \-- \<\<
se diverso nel tuo ORDS, cambia qui

c_module_name CONSTANT VARCHAR2(128) := \'uat_api_v1\';

c_api_path_pattern CONSTANT VARCHAR2(512) := \'/int_admin/api/v1/\*\';
\-- coerente con mapping sopra

c_role_name CONSTANT VARCHAR2(128) := \'UAT_API_ROLE\';

c_priv_name CONSTANT VARCHAR2(128) := \'UAT_API_PRIV\';

c_priv_label CONSTANT VARCHAR2(256) := \'UAT API Access\';

c_oauth_client_name CONSTANT VARCHAR2(128) := \'VBCS_UAT_CLIENT\';

v_has_ords_security NUMBER := 0;

v_has_oauth_pkg NUMBER := 0;

\-- output credenziali client

v_client_id VARCHAR2(4000);

v_client_secret VARCHAR2(4000);

PROCEDURE log(p_msg VARCHAR2) IS BEGIN dbms_output.put_line(p_msg); END;

PROCEDURE try_exec(p_sql CLOB) IS

BEGIN

EXECUTE IMMEDIATE p_sql;

EXCEPTION

WHEN OTHERS THEN

\-- ignora errori \"already exists\" e simili; logga solo

log(\'WARN: \' \|\| SQLERRM);

END;

BEGIN

dbms_output.enable(NULL);

\-- 0) Enable schema (idempotente). Se già abilitato, verrà
ignorato/ri-loggato.

BEGIN

ORDS.ENABLE_SCHEMA(

p_enabled =\> TRUE,

p_schema =\> USER,

p_url_mapping_type =\> \'BASE_PATH\',

p_url_mapping_pattern=\> c_url_mapping_pattern,

p_auto_rest_auth =\> FALSE

);

COMMIT;

log(\'OK: ORDS schema enabled/mapped: \' \|\| c_url_mapping_pattern);

EXCEPTION

WHEN OTHERS THEN

log(\'WARN: enable_schema -\> \' \|\| SQLERRM);

END;

\-- 1) Detect ORDS_SECURITY (ORDS 24.3+).
\[1\](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

SELECT COUNT(\*) INTO v_has_ords_security

FROM all_objects

WHERE object_type=\'PACKAGE\'

AND object_name=\'ORDS_SECURITY\';

\-- 2) Create ROLE + PRIVILEGE + MAPPING (comune: package ORDS).
\[2\](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)\[3\](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

BEGIN

ORDS.CREATE_ROLE(p_role_name =\> c_role_name);

COMMIT;

log(\'OK: role created: \' \|\| c_role_name);

EXCEPTION WHEN OTHERS THEN log(\'WARN: create_role -\> \'\|\|SQLERRM);
END;

BEGIN

ORDS.CREATE_PRIVILEGE(

p_name =\> c_priv_name,

p_role_name =\> c_role_name,

p_label =\> c_priv_label,

p_description =\> \'Privilege protecting UAT Issue Tracker APIs\'

);

COMMIT;

log(\'OK: privilege created: \' \|\| c_priv_name);

EXCEPTION WHEN OTHERS THEN log(\'WARN: create_privilege -\>
\'\|\|SQLERRM); END;

BEGIN

ORDS.CREATE_PRIVILEGE_MAPPING(

p_privilege_name =\> c_priv_name,

p_pattern =\> c_api_path_pattern

);

COMMIT;

log(\'OK: privilege mapping created: \' \|\| c_api_path_pattern);

EXCEPTION WHEN OTHERS THEN log(\'WARN: create_privilege_mapping -\>
\'\|\|SQLERRM); END;

\-- 3) (Opzionale) collega il privilegio al modulo ORDS (se il modulo
esiste)

BEGIN

ORDS.SET_MODULE_PRIVILEGE(

p_module_name =\> c_module_name,

p_privilege_name =\> c_priv_name

);

COMMIT;

log(\'OK: module privilege set on: \' \|\| c_module_name);

EXCEPTION WHEN OTHERS THEN log(\'WARN: set_module_privilege -\>
\'\|\|SQLERRM); END;

\-- 4) Create OAuth Client (client_credentials)

IF v_has_ords_security \> 0 THEN

\-- ORDS 24.3+ (ORDS_SECURITY)
\[1\](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

DECLARE

l_client_cred ords_types.t_client_credentials;

BEGIN

l_client_cred := ords_security.register_client(

p_name =\> c_oauth_client_name,

p_grant_type =\> \'client_credentials\',

p_description =\> \'VBCS client for UAT APIs\',

p_client_secret =\> NULL, \-- genera secret

p_support_email =\> \'support@ente.it\',

p_privilege_names =\> c_priv_name

);

COMMIT;

v_client_id := l_client_cred.client_key.client_id;

v_client_secret := l_client_cred.client_secret.secret;

log(\'OK: OAuth client created (ORDS_SECURITY): \' \|\|
c_oauth_client_name);

log(\'CLIENT_ID : \' \|\| v_client_id);

log(\'CLIENT_SECRET : \' \|\| v_client_secret);

\-- grant role

ords_security.grant_client_role(

p_client_name =\> c_oauth_client_name,

p_role_name =\> c_role_name

);

COMMIT;

log(\'OK: role granted to client: \' \|\| c_role_name);

EXCEPTION

WHEN OTHERS THEN

log(\'ERROR: ORDS_SECURITY client creation failed -\> \'\|\|SQLERRM);

RAISE;

END;

ELSE

\-- Legacy packages (oauth / ords_metadata.oauth_admin)
\[3\](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)\[2\](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html)

\-- Verifica disponibilità package OAUTH

SELECT COUNT(\*) INTO v_has_oauth_pkg

FROM all_objects

WHERE object_type=\'PACKAGE\'

AND object_name IN (\'OAUTH\');

IF v_has_oauth_pkg \> 0 THEN

\-- create client via oauth.create_client

BEGIN

EXECUTE IMMEDIATE q\'\[

BEGIN

oauth.create_client(

p_name =\> :1,

p_grant_type =\> \'client_credentials\',

p_description =\> \'VBCS client for UAT APIs\',

p_support_email =\> \'support@ente.it\',

p_privilege_names =\> :2

);

COMMIT;

END;

\]\' USING c_oauth_client_name, c_priv_name;

EXECUTE IMMEDIATE q\'\[

BEGIN

oauth.grant_client_role(

p_client_name =\> :1,

p_role_name =\> :2

);

COMMIT;

END;

\]\' USING c_oauth_client_name, c_role_name;

\-- recupero client_id (client_secret spesso non è più leggibile in
chiaro dopo la creazione)

EXECUTE IMMEDIATE q\'\[SELECT client_id FROM user_ords_clients WHERE
name = :1\]\' INTO v_client_id USING c_oauth_client_name;

log(\'OK: OAuth client created (legacy OAUTH): \' \|\|
c_oauth_client_name);

log(\'CLIENT_ID : \' \|\| v_client_id);

log(\'NOTE: il client_secret potrebbe essere visibile SOLO al momento
della creazione (dipende dalla versione).\');

EXCEPTION WHEN OTHERS THEN

log(\'ERROR: legacy OAUTH client creation failed -\> \'\|\|SQLERRM);

RAISE;

END;

ELSE

raise_application_error(-20001,\'Né ORDS_SECURITY né OAUTH risultano
disponibili. Verificare versione ORDS / grants.\');

END IF;

END IF;

log(\'DONE. Token URL atteso:
https://\<HOST\>/ords/\'\|\|c_url_mapping_pattern\|\|\'/oauth/token\');

END;

/

Mostra più linee

**1.3 Test rapido (curl)**

1.  **Token** (client_credentials) --- ORDS token endpoint standard
    [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements),
    [\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

Shell

curl -i -k \\

\--user \<CLIENT_ID\>:\<CLIENT_SECRET\> \\

\--data \"grant_type=client_credentials\" \\

https://\<HOST\>/ords/int_admin/oauth/token

Mostra più linee

1.  **Chiamata API protetta** (senza token → 401) e con token → 200
    [\[docs.oracle.com\]](https://docs.oracle.com/en/database/oracle/sql-developer-web/25.1/sdweb/securing-restful-services.html),
    [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

Shell

curl -i -k -H \"Authorization: Bearer \<ACCESS_TOKEN\>\" \\

https://\<HOST\>/ords/int_admin/api/v1/issues?page=1&pageSize=20

Mostra più linee

**2) "PACCHETTO VBCS" (installazione su OCI) --- Runbook completo**

Qui ti do il **pacchetto operativo**: non è un file zip (quello si
genera da VBCS con Export), ma è un **install kit** che ti permette di
fare tutto tramite console/UI VBCS senza sviluppatore.

**2.1 Prerequisiti OCI**

-   Istanza **Oracle Visual Builder / Visual Builder Studio** pronta su
    OCI (o in OIC).

-   ORDS raggiungibile da VBCS (rete, DNS, SSL). Se c'è un problema di
    **CORS o certificati**, VBCS consiglia "Always use proxy" oppure
    sistemare la chain SSL.
    [\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

**2.2 Configurazione Backend in VBCS (OAuth2 Client Credentials)**

VBCS supporta l'autenticazione **OAuth 2.0 Client Credentials** verso
ORDS (fixed credentials).
[\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html),
[\[blogs.oracle.com\]](https://blogs.oracle.com/vbcs/connect-visual-builder-to-rest-apis-secured-by-identity-domainidcs-oauth)

**Passi (UI VBCS):**

1.  **Services → Backends → New Backend**

2.  Tipo: **Custom**

3.  **Base URL**:\
    https://\<HOST\>/ords/int_admin/api/v1/

4.  **Authentication**: **OAuth 2.0 Client Credentials**
    [\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

5.  Inserisci:

    -   **Client ID** e **Client Secret** (output dello script one‑shot)

    -   **Token URL**: https://\<HOST\>/ords/int_admin/oauth/token
        [\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html),
        [\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

    -   **Scope**: vuoto (per ORDS fixed credentials in genere è blank)
        [\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

6.  **Connection Type**: se hai dubbi su CORS, imposta **Always use
    proxy** (evita configurazioni CORS lato ORDS).
    [\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

7.  Test connection.

**2.3 Service Connections (endpoint)**

Dato che i tuoi endpoint sono custom, il modo più semplice è:

-   **Define by Endpoint** (uno per operazione o uno per risorsa +
    metodi)

Crea service connections per:

-   GET /issues

-   POST /issues

-   GET /issues/{id}

-   PATCH /issues/{id}

-   POST /issues/{id}/transition

-   GET/POST /issues/{id}/comments

-   GET /lov/{type}

-   GET /usecases e GET /usecases/{code}

In alternativa, se vuoi "import massivo", puoi pubblicare un OpenAPI
spec e farla mappare da VBCS; ma per partire subito, "Define by
Endpoint" è più rapido e coerente con la doc Oracle su mapping endpoint
ORDS.
[\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

**2.4 Profili Applicativi (parametri ambiente)**

Per rendere il deploy pulito su OCI, crea un **Application Profile**
(es. UAT) con:

-   ORDS_BASE_URL = https://\<HOST\>/ords/int_admin/api/v1

-   ORDS_TOKEN_URL = https://\<HOST\>/ords/int_admin/oauth/token

Così puoi cambiare host/ambiente senza toccare le pagine.

**2.5 Pagine VBCS (no‑code build, blueprint)**

**A) Issue List**

-   Component: Table

-   Data: Service GET /issues

-   Filtri UI: status, severity, domainStream, environment (SVI/CALL),
    q, paging/sort

-   Link riga → Issue Detail

**B) Issue Create/Edit**

-   Form Layout con card/sezioni (contesto, descrizione, riproduzione,
    allegati)

-   Submit → POST /issues

-   Validazioni UI (minime) + validazioni server (già nel package DB)

**C) Issue Detail**

-   GET /issues/{id}

-   Tab "Commenti": GET/POST comments

-   Tab "Workflow": bottoni → transition (PROPOSE_SOLUTION,
    SEND_TO_RETEST, CONFIRM_RESOLUTION, REOPEN_FAIL, CLOSE)

Nota: VBCS + ORDS richiede che tu gestisca filter/sort/paging lato ORDS
(che abbiamo fatto), perché a differenza dei Business Objects nativi,
ORDS non fornisce automaticamente i "transforms".
[\[docs.oracle.com\]](https://docs.oracle.com/en/cloud/paas/application-integration/visual-developer/connect-ords-apis-using-fixed-credentials.html)

**2.6 Deploy su OCI (runbook)**

1.  In VBCS: **Deployments → New Deployment**

2.  Seleziona profilo UAT (variabili base URL / token URL)

3.  Scegli target (test/prod)

4.  Avvia deploy

5.  Esegui smoke test:

    -   apertura lista

    -   creazione issue

    -   transizione a re-test (assegnazione owner use case)

    -   chiusura

**3) Due aspetti importanti (per evitare problemi reali)**

**3.1 Identità utente finale**

Con **client_credentials** l'identità è quella del **client**, non
dell'utente umano (tester). È normale per questo grant.\
Se vuoi tracciare il tester:
[\[blogs.oracle.com\]](https://blogs.oracle.com/vbcs/connect-visual-builder-to-rest-apis-secured-by-identity-domainidcs-oauth),
[\[oracle-base.com\]](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-authentication-enhancements)

-   aggiungi nel form un campo reportedByName/reportedByEmail (solo
    audit) e salvalo su DB; **non usarlo mai per autorizzazioni**.

**3.2 SSL e certificati**

Se VBCS lamenta errori di certificato / chain incompleta, Oracle
raccomanda di:

-   assicurare la chain completa sul server ORDS o

-   usare un LB che presenti chain completa\
    Oppure usare "Always use proxy" lato VBCS.
