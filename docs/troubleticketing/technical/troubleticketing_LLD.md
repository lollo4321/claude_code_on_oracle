# LLD — Trouble Ticketing UAT

**Stream**: troubleticketing  
**Documento funzionale di riferimento**: `docs/troubleticketing/functional/Troubleticketing.md`  
**Data**: 2026-04-01

---

## 1. Architettura del processo

Flusso complessivo dal backend al frontend:

```
1.  [DB] Le tabelle INFND_UAT_ISSUE_H, INFND_UAT_ATTACH_L, INFND_UAT_CMT_L,
        INFND_UAT_HIST_L, INFND_UAT_LOV, INFND_UAT_USECASE in STAGING_PAAS
        persistono segnalazioni, allegati, commenti, storico stati, liste
        di valori configurabili e casi d'uso con i rispettivi owner.

2.  [DB] Il package INFND_UAT_MGR_PKG centralizza la logica applicativa:
        inserimento e aggiornamento segnalazioni, transizioni di stato
        con validazioni (precondizioni workflow, campi obbligatori condizionali),
        lookup owner use case per assegnazione re-test, gestione commenti
        e allegati.

3.  [DB] Il trigger INFND_UAT_ISSUE_TRG intercetta ogni UPDATE di status
        su INFND_UAT_ISSUE_H e inserisce automaticamente un record in
        INFND_UAT_HIST_L come meccanismo di audit complementare.

4.  [DB → ORDS] Il modulo ORDS uat_api_v1 (base path /uat/api/v1/) espone:
        - GET diretti su SELECT per issues (lista + dettaglio), commenti,
          allegati, LOV e use case;
        - POST su procedure PL/SQL (INFND_UAT_MGR_PKG) per creazione,
          aggiornamento, transizioni di stato, commenti e upload allegati.

5.  [ORDS → VBCS] La pagina IssueList chiama GET /issues con query params
        (filtri, paginazione, sorting) e GET /lov/:type per popolare i
        Select. Naviga a IssueCreateEdit o IssueDetail.

6.  [ORDS → VBCS] La pagina IssueCreateEdit carica le LOV via GET /lov/:type
        e GET /usecases, poi invia via POST /issues con status=DRAFT
        (Salva bozza) o status=NEW (Invia).

7.  [ORDS → VBCS] La pagina IssueDetail carica il dettaglio via GET /issues/:issueId,
        commenti e allegati tramite GET dedicati, e gestisce le transizioni
        di stato via POST /issues/:issueId/transition in base a ruolo
        corrente e stato della segnalazione.
```

---

## 2. Database

**Schema di riferimento**: `STAGING_PAAS`  
_(vedi Sezione 5 — Assunzioni e punti aperti, punto 1 e 2)_

---

### Tabelle

| Nome | Schema | Descrizione |
|------|--------|-------------|
| `INFND_UAT_ISSUE_H` | STAGING_PAAS | Tabella principale delle segnalazioni UAT. Contiene: contesto (solution_type, application_module, environment, domain_stream, use_case_code), classificazione (record_type, severity, priority, frequency, impact_flags), descrizioni (title, expected_behavior, actual_behavior, steps_to_reproduce), dati workflow (status, resolution_notes, fix_version, retest_assigned_to, retest_outcome, closure_reason), audit WHO columns (created_by, created_on, updated_by, updated_on) e campo context_payload CLOB per dettagli variabili futuri. |
| `INFND_UAT_ATTACH_L` | STAGING_PAAS | Allegati delle segnalazioni. Contiene metadati file (file_name, mime_type, file_size, description) e contenuto BLOB (file_content). Figlio di INFND_UAT_ISSUE_H tramite FK su issue_id (ON DELETE CASCADE). |
| `INFND_UAT_CMT_L` | STAGING_PAAS | Commenti delle segnalazioni. Gestisce la timeline commenti con visibilità PUBLIC/INTERNAL, autore e timestamp. Figlio di INFND_UAT_ISSUE_H tramite FK su issue_id (ON DELETE CASCADE). |
| `INFND_UAT_HIST_L` | STAGING_PAAS | Storico dei cambi di stato. Registra old_status, new_status, changed_by, changed_on e note per ogni transizione di stato. Alimentato sia dal trigger INFND_UAT_ISSUE_TRG che dalla procedura TRANSITION_ISSUE. Figlio di INFND_UAT_ISSUE_H tramite FK su issue_id (ON DELETE CASCADE). |
| `INFND_UAT_LOV` | STAGING_PAAS | Tabella di configurazione delle liste di valori. Discriminata per lov_type (STATUS, SEVERITY, PRIORITY, RECORD_TYPE, DOMAIN_STREAM, ENVIRONMENT, COMPONENT, CLOSURE_REASON). Evita hard-code in VBCS e permette modifiche senza redeploy. |
| `INFND_UAT_USECASE` | STAGING_PAAS | Anagrafica dei casi d'uso con owner (owner_user, owner_email). Usata per l'assegnazione automatica del tester al passaggio in RETEST: se use_case_code è valorizzato sul ticket, retest_assigned_to viene derivato da owner_user; in assenza di match, fallback su created_by. |

---

### Package

| Nome | Schema | Descrizione |
|------|--------|-------------|
| `INFND_UAT_MGR_PKG` | STAGING_PAAS | Package principale per la gestione delle segnalazioni UAT. Centralizza inserimento, aggiornamento, transizioni di stato con validazioni, lookup owner use case, gestione commenti e upload allegati. È il punto di ingresso per tutti gli handler ORDS di tipo POST/PL/SQL. |

---

### Procedure / Funzioni

| Nome | Package | Descrizione | Input | Output |
|------|---------|-------------|-------|--------|
| `INS_ISSUE` | `INFND_UAT_MGR_PKG` | Inserisce una nuova segnalazione. Valida i campi obbligatori (solution_type, application_module, environment, domain_stream, record_type, severity, title, expected_behavior, actual_behavior, steps_to_reproduce). Imposta status = DRAFT o NEW in base all'azione richiesta. Valorizza created_by e created_on. | p_solution_type, p_app_module, p_environment, p_domain_stream, p_use_case_code, p_page_url, p_tenant_pod, p_app_version, p_record_type, p_severity, p_priority, p_frequency, p_impact_flags, p_title, p_expected_behavior, p_actual_behavior, p_error_message, p_prerequisites, p_steps_to_reproduce, p_reproducible_yn, p_notes, p_status, p_created_by | p_issue_id NUMBER, p_status VARCHAR2, p_result VARCHAR2 |
| `UPD_ISSUE` | `INFND_UAT_MGR_PKG` | Aggiorna i campi non-workflow di una segnalazione esistente (title, steps_to_reproduce, expected_behavior, actual_behavior, error_message, notes, etc.). Non gestisce transizioni di stato. Aggiorna updated_on e updated_by. | p_issue_id, p_title, p_expected_behavior, p_actual_behavior, p_error_message, p_prerequisites, p_steps_to_reproduce, p_reproducible_yn, p_notes, p_updated_by | p_result VARCHAR2 |
| `TRANSITION_ISSUE` | `INFND_UAT_MGR_PKG` | Gestisce le transizioni di stato del workflow. Valida lo stato corrente (precondizione) e applica l'azione richiesta: PROPOSE_SOLUTION (IN_PROGRESS → SOLUTION_PROPOSED), SEND_TO_RETEST (SOLUTION_PROPOSED → RETEST, con lookup owner use case), CONFIRM_RESOLUTION (RETEST → RESOLVED), REOPEN_FAIL (RETEST → IN_PROGRESS), CLOSE (RESOLVED → CLOSED). Per ogni transizione valida i campi obbligatori dell'azione, aggiorna i campi specifici e inserisce il record in INFND_UAT_HIST_L. | p_issue_id, p_action, p_user, p_resolution_notes, p_fix_version, p_fix_environment, p_retest_instructions, p_retest_assigned_to, p_retest_outcome, p_retest_notes, p_closure_reason, p_closure_evidence_link | p_new_status VARCHAR2, p_retest_assigned_to VARCHAR2, p_result VARCHAR2 |
| `INS_COMMENT` | `INFND_UAT_MGR_PKG` | Inserisce un commento su una segnalazione. Gestisce visibilità PUBLIC/INTERNAL. Valorizza comment_on e comment_by. | p_issue_id, p_comment_text, p_visibility, p_comment_by | p_comment_id NUMBER, p_result VARCHAR2 |
| `INS_ATTACHMENT` | `INFND_UAT_MGR_PKG` | Inserisce metadati e contenuto BLOB di un allegato associato a una segnalazione. Valorizza uploaded_on e uploaded_by. | p_issue_id, p_file_name, p_mime_type, p_file_size, p_file_content BLOB, p_description, p_uploaded_by | p_attachment_id NUMBER, p_result VARCHAR2 |

---

### Trigger

| Nome | Tabella | Descrizione |
|------|---------|-------------|
| `INFND_UAT_ISSUE_TRG` | `INFND_UAT_ISSUE_H` | Trigger BEFORE UPDATE OF status. Se NEW.status è diverso da OLD.status, inserisce automaticamente un record in INFND_UAT_HIST_L con old_status, new_status e updated_by come changed_by. Funziona come meccanismo di audit complementare alla logica della procedura TRANSITION_ISSUE. |

---

### Indici

| Nome | Tabella | Colonna/e | Tipo |
|------|---------|-----------|------|
| `INFND_UAT_ISSUE_STATUS_IND` | `INFND_UAT_ISSUE_H` | status | Standard |
| `INFND_UAT_ISSUE_SEVERITY_IND` | `INFND_UAT_ISSUE_H` | severity | Standard |
| `INFND_UAT_ISSUE_STREAM_IND` | `INFND_UAT_ISSUE_H` | domain_stream | Standard |
| `INFND_UAT_ISSUE_ASSIGNEE_IND` | `INFND_UAT_ISSUE_H` | assignee | Standard |
| `INFND_UAT_ISSUE_CREATED_IND` | `INFND_UAT_ISSUE_H` | created_on | Standard |
| `INFND_UAT_ATTACH_ISSUE_IND` | `INFND_UAT_ATTACH_L` | issue_id | Standard |
| `INFND_UAT_CMT_ISSUE_IND` | `INFND_UAT_CMT_L` | issue_id | Standard |
| `INFND_UAT_HIST_ISSUE_IND` | `INFND_UAT_HIST_L` | issue_id | Standard |
| `INFND_UAT_HIST_CHANGED_IND` | `INFND_UAT_HIST_L` | changed_on | Standard |
| `INFND_UAT_USECASE_STREAM_IND` | `INFND_UAT_USECASE` | domain_stream | Standard |
| `INFND_UAT_USECASE_OWNER_IND` | `INFND_UAT_USECASE` | owner_user | Standard |

---

## 3. ORDS

### Moduli

| Nome modulo | Base path | Schema |
|-------------|-----------|--------|
| `uat_api_v1` | `/uat/api/v1/` | STAGING_PAAS |

---

### Endpoint

| Modulo | Template | Metodo | Procedura / Select | Input | Output | Descrizione |
|--------|----------|--------|-------------------|-------|--------|-------------|
| `uat_api_v1` | `lov/:type` | GET | SELECT da `INFND_UAT_LOV` | type (path param: STATUS, SEVERITY, PRIORITY, RECORD_TYPE, DOMAIN_STREAM, ENVIRONMENT, COMPONENT, CLOSURE_REASON) | `{data: [{code, label, sortOrder}]}` | Restituisce le voci attive di una LOV filtrata per lov_type. Usato da VBCS per popolare tutti i Select/Radio della UI. |
| `uat_api_v1` | `usecases` | GET | SELECT da `INFND_UAT_USECASE` | domainStream (query param), enabled (query param, default Y) | `{data: [{useCaseCode, useCaseTitle, domainStream, ownerUser, ownerEmail}]}` | Restituisce i casi d'uso filtrati per domainStream. Usato in IssueCreateEdit per popolare il campo Use Case. |
| `uat_api_v1` | `usecases/:useCaseCode` | GET | SELECT da `INFND_UAT_USECASE` | useCaseCode (path param) | `{data: {useCaseCode, useCaseTitle, ownerUser, ownerEmail}}` | Restituisce il dettaglio di un singolo caso d'uso. |
| `uat_api_v1` | `issues` | GET | SELECT da `INFND_UAT_ISSUE_H` | status, severity, domainStream, environment, solutionType, assignee, createdBy, q, page, pageSize, sort (query params, tutti opzionali) | `{data: [{issueId, title, domainStream, environment, solutionType, severity, status, assignee, createdOn, updatedOn}], meta: {page, pageSize, total}}` | Lista segnalazioni con filtri multipli, ricerca testuale su title (q), paginazione server-side e sorting dinamico. Usato dalla pagina IssueList. |
| `uat_api_v1` | `issues` | POST | `INFND_UAT_MGR_PKG.INS_ISSUE` | JSON body: solutionType, applicationModule, environment, domainStream, useCaseCode, recordType, severity, title, expectedBehavior, actualBehavior, stepsToReproduce, reproducible, status (DRAFT o NEW), + campi opzionali | `{data: {issueId, status}}` | Crea una nuova segnalazione. status=DRAFT per salvataggio bozza, status=NEW per invio ufficiale. |
| `uat_api_v1` | `issues/:issueId` | GET | SELECT da `INFND_UAT_ISSUE_H` | issueId (path param) | `{data: {oggetto completo con tutti i campi inclusi CLOB e campi workflow}}` | Restituisce il dettaglio completo di una segnalazione. Usato dalla pagina IssueDetail. |
| `uat_api_v1` | `issues/:issueId` | POST | `INFND_UAT_MGR_PKG.UPD_ISSUE` | issueId (path param), JSON body: campi modificabili (title, expectedBehavior, actualBehavior, stepsToReproduce, errorMessage, notes, etc.) | `{data: {issueId, updatedOn}}` | Aggiornamento parziale dei campi non-workflow. Implementato come POST (non PATCH) per coerenza con la convenzione di progetto — vedi punti aperti. |
| `uat_api_v1` | `issues/:issueId/transition` | POST | `INFND_UAT_MGR_PKG.TRANSITION_ISSUE` | issueId (path param), JSON body: `{action: "PROPOSE_SOLUTION" \| "SEND_TO_RETEST" \| "CONFIRM_RESOLUTION" \| "REOPEN_FAIL" \| "CLOSE", payload: {...}}` | `{data: {issueId, status, retestAssignedTo}}` | Centralizza tutte le transizioni di stato del workflow con validazioni. Per SEND_TO_RETEST: se retestAssignedTo è null, esegue lookup owner da INFND_UAT_USECASE. |
| `uat_api_v1` | `issues/:issueId/comments` | GET | SELECT da `INFND_UAT_CMT_L` | issueId (path param) | `{data: [{commentId, commentText, visibility, commentOn, commentBy}]}` | Restituisce i commenti di una segnalazione, ordinati per commentOn ASC. Usato nel tab Commenti di IssueDetail. |
| `uat_api_v1` | `issues/:issueId/comments` | POST | `INFND_UAT_MGR_PKG.INS_COMMENT` | issueId (path param), JSON body: `{commentText, visibility: "PUBLIC" \| "INTERNAL"}` | `{data: {commentId}}` | Aggiunge un commento a una segnalazione. |
| `uat_api_v1` | `issues/:issueId/attachments` | GET | SELECT da `INFND_UAT_ATTACH_L` | issueId (path param) | `{data: [{attachmentId, fileName, mimeType, fileSize, description, uploadedOn, uploadedBy}]}` | Restituisce la lista degli allegati (solo metadati, senza BLOB). Usato nel tab Allegati di IssueDetail. |
| `uat_api_v1` | `issues/:issueId/attachments` | POST | `INFND_UAT_MGR_PKG.INS_ATTACHMENT` | issueId (path param), multipart/form-data: file (BLOB), description (VARCHAR2) | `{data: {attachmentId}}` | Upload di un allegato (BLOB) associato a una segnalazione. |
| `uat_api_v1` | `attachments/:attachmentId/content` | GET | SELECT BLOB da `INFND_UAT_ATTACH_L` | attachmentId (path param) | Contenuto binario del file con Content-Type corretto (mime_type) | Download del contenuto di un allegato. Usato per visualizzare screenshot e scaricare log. |

---

## 4. VBCS

### Pagine

| Nome pagina | Descrizione | Endpoint ORDS chiamati |
|-------------|-------------|------------------------|
| `IssueList` | Vista elenco segnalazioni con filtri rapidi (status multi-select, severity multi-select, domainStream, environment, solutionType, ricerca testuale su title), tabs per quick views (Tutte / Le mie / Da re-testare / Alte priorità), tabella con paginazione server-side e sorting. Azioni: navigazione a creazione o dettaglio, Export CSV. | GET `/lov/:type` (STATUS, SEVERITY, DOMAIN_STREAM, ENVIRONMENT, SOLUTION_TYPE), GET `/issues` |
| `IssueCreateEdit` | Form strutturato in 4 card (Contesto, Descrizione, Riproducibilità, Evidenze) per creare o modificare una segnalazione. Gestisce: visibilità condizionale campi SaaS/PaaS (tenantPod vs appVersion), LOV dinamiche per Use Case filtrate per domainStream, validazioni obbligatorie e warning (allegato consigliato per severity BLOCKER/HIGH; noteAggiuntive obbligatorie se reproducible=No; riferimento transazione se recordType=Dato). | GET `/lov/:type` (DOMAIN_STREAM, ENVIRONMENT, RECORD_TYPE, SEVERITY, PRIORITY, FREQUENCY, APPLICATION_MODULE), GET `/usecases`, POST `/issues` |
| `IssueDetail` | Pagina di dettaglio a tabs. Header: ID, titolo, chips status/severity/env/stream, pulsanti workflow condizionati da ruolo+stato. Tab Dettaglio: campi read-only (edit parziale per Triage/Lead). Tab Allegati: lista + upload. Tab Commenti: timeline con badge PUBLIC/INTERNAL. Tab Workflow & Audit: card azioni (Proponi soluzione, Invia in re-test, Conferma risoluzione/Riapri, Chiudi) + storico stati. | GET `/issues/:issueId`, GET `/issues/:issueId/comments`, POST `/issues/:issueId/comments`, GET `/issues/:issueId/attachments`, POST `/issues/:issueId/attachments`, GET `/attachments/:attachmentId/content`, POST `/issues/:issueId/transition`, POST `/issues/:issueId` |

---

### Componenti principali

| Pagina | Componente | Tipo | Descrizione |
|--------|------------|------|-------------|
| `IssueList` | FiltriCard | `oj-form-layout` | Card filtri: domainStream (select-single), environment (select-single), solutionType (select-single), status (select-many), severity (select-many), pulsante Reset. |
| `IssueList` | RicercaTestuale | `oj-input-text` | Input con icona search per ricerca libera su title. Binding sul query param q di GET /issues. |
| `IssueList` | TabQuickViews | `oj-tab-bar` | Tabs con filtri pre-impostati: Tutte, Le mie (createdBy=me), Da re-testare (status=RETEST, retestAssignedTo=me), Alte priorità (severity=BLOCKER,HIGH). |
| `IssueList` | IssueTable | `oj-table` | Tabella con DataProvider REST. Colonne: ID (link a IssueDetail), Titolo, Stream (badge), Env, Type, Severity (chip colorato), Status (pill colorata), Assignee (avatar+nome), Updated (relative time). Paginazione server-side, sorting per colonna. |
| `IssueList` | PulsanteNuovaSegnalazione | `oj-button` (primary) | Naviga a IssueCreateEdit in modalità creazione. |
| `IssueList` | PulsanteExportCSV | `oj-button` (secondary) | Export CSV delle segnalazioni con i filtri correnti. |
| `IssueCreateEdit` | CardContesto | `oj-form-layout` | Card A: solutionType (oj-radioset), applicationModule (oj-select-single), environment (oj-select-single), domainStream (oj-select-single), useCaseCode (oj-select-single, LOV filtrata per domainStream), pageUrl (oj-input-text), tenantPod e appVersion (visibili condizionalmente su solutionType). |
| `IssueCreateEdit` | CardDescrizione | `oj-form-layout` | Card B: title (oj-input-text, max 120), recordType (oj-select-single), severity (oj-select-single), priority (oj-select-single), frequency (oj-select-single), impactFlags (oj-checkboxset), expectedBehavior/actualBehavior/errorMessage (oj-text-area). |
| `IssueCreateEdit` | CardRiproducibilita | `oj-form-layout` | Card C: prerequisites (oj-text-area), stepsToReproduce (oj-text-area, required), reproducible (oj-switch), noteAggiuntive (oj-text-area, visibile e obbligatoria se reproducible=No). |
| `IssueCreateEdit` | CardEvidenze | `oj-form-layout` | Card D: allegati multipli (oj-file-picker), lista file con nome/size, riferimenti transazione (oj-input-text). |
| `IssueCreateEdit` | FooterAzioni | `oj-button` group | Salva bozza → POST /issues con status=DRAFT. Invia → POST /issues con status=NEW. Annulla → torna a IssueList. |
| `IssueDetail` | HeaderDettaglio | `oj-panel` | H1: #ID — Titolo. Chips: status (pill colorata), severity, environment, domainStream, solutionType. Pulsanti workflow a destra, visibili condizionalmente per ruolo+stato. |
| `IssueDetail` | TabsDettaglio | `oj-tabs` | 4 tabs: Dettaglio, Allegati, Commenti, Workflow & Audit. |
| `IssueDetail` | CommentTimeline | `oj-list-view` | Lista commenti ordinata per commentOn ASC. Ogni item: autore, data/ora, badge PUBLIC/INTERNAL, testo. Form inline per aggiunta nuovo commento. |
| `IssueDetail` | AllegatiList | `oj-list-view` + `oj-file-picker` | Lista allegati con nome, size, tipo, link download (GET /attachments/:id/content). Upload nuovo allegato via POST. |
| `IssueDetail` | WorkflowCard | `oj-panel` | Card visibili per ruolo+stato: Proponi soluzione (resolutionNotes obbligatorio, fixVersion obbligatorio per PaaS, fixEnvironment), Invia in re-test (retestInstructions, retestAssignedTo con default owner use case), Re-test (retestOutcome radio PASS/FAIL, retestNotes obbligatorio se FAIL), Chiusura (closureReason obbligatorio). |
| `IssueDetail` | StatusHistory | `oj-list-view` | Storico stati: per ogni record mostra changed_on, old_status → new_status, changed_by, change_note. |

---

## 5. Assunzioni e punti aperti

### Assunzioni fatte

- **A1** — Le tabelle sono create nello schema `STAGING_PAAS`, schema standard per estensioni applicative custom. Il funzionale suggerisce uno schema dedicato `UAT_BUGS` non presente nelle naming conventions del progetto.
- **A2** — Il prefisso modulo utilizzato è `INFND` (oggetti common/cross-domain), in assenza di un prefisso dedicato nelle naming conventions per funzionalità UAT/tracking.
- **A3** — Le notifiche email (4 eventi: NEW, RETEST, RESOLVED, CLOSED) sono gestite lato VBCS tramite action chain + service connection. Non è prevista una coda di notifica DB nel MVP.
- **A4** — Gli allegati sono persistiti come BLOB direttamente su ATP. Soluzione accettabile per i volumi attesi in fase UAT.
- **A5** — L'endpoint di aggiornamento parziale è implementato come POST (non PATCH) per coerenza con la convenzione di progetto (POST per procedure PL/SQL, GET per select).
- **A6** — Il campo `context_payload` (CLOB JSON per dettagli variabili SaaS/PaaS) è predisposto nella tabella INFND_UAT_ISSUE_H ma non esposto nell'UI MVP. Evoluzione futura.
- **A7** — Le identità utente vengono passate agli endpoint ORDS tramite header (es. `X-User`). Il meccanismo esatto dipende dalla configurazione SSO/IDCS dell'ambiente.

### Punti aperti da risolvere prima dello sviluppo

- **P1** — **Schema dedicato vs STAGING_PAAS**: confermare se usare `STAGING_PAAS` o creare uno schema dedicato (es. `UAT_BUGS`). Impatta permessi DBA, configurazione ORDS e isolamento dati.
- **P2** — **Prefisso modulo naming**: `INFND` è il più prossimo disponibile, ma nessun prefisso nelle naming conventions copre esplicitamente funzionalità UAT/cross-domain. Valutare l'introduzione di un nuovo prefisso (es. `INUAT`) o confermare `INFND`.
- **P3** — **Naming ORDS e VBCS**: le convenzioni per nomi di moduli ORDS e componenti VBCS non sono ancora definite in `naming-conventions.md`. I nomi in questo LLD sono funzionali e devono essere allineati quando la convenzione sarà definita.
- **P4** — **Meccanismo identità utente in ORDS**: definire se lo username corrente è passato via header `X-User`, derivato dal token OAuth2/IDCS o da altro meccanismo. Impatta tutte le procedure che valorizzano created_by, updated_by, comment_by, uploaded_by.
- **P5** — **Notifiche — implementazione**: confermare se le 4 notifiche email (NEW, RETEST, RESOLVED, CLOSED) sono gestite lato VBCS (action chain + Oracle Integration Cloud / SMTP) o lato DB (tabella coda + Oracle job scheduler). La scelta impatta affidabilità e complessità.
- **P6** — **PATCH vs POST per aggiornamento parziale**: il funzionale specifica `PATCH /issues/:issueId`. ORDS supporta PATCH, ma la convenzione di progetto prevede POST per procedure. Chiarire se usare PATCH con handler PL/SQL o mantenere POST.
- **P7** — **Upload allegati multipart in VBCS**: verificare compatibilità tra `oj-file-picker` e l'endpoint ORDS `POST /issues/:issueId/attachments` con `multipart/form-data`. Se non supportato nativamente, valutare encoding base64 nel body JSON.
- **P8** — **Ricerca full-text (Oracle Text)**: il funzionale menziona Oracle Text come evoluzione per ricerca avanzata. L'MVP usa `LIKE` su title. Definire se predisporre l'indice CTX fin dall'inizio o aggiungerlo in seguito.
