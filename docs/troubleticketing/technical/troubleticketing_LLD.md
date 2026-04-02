# LLD — Trouble Ticketing UAT

**Stream**: troubleticketing  
**Documento funzionale di riferimento**: `docs/troubleticketing/functional/Troubleticketing.md`  
**Data**: 2026-04-02

---

## 1. Architettura del processo

```
1.  [DB] Le tabelle in STAGING_PAAS persistono: segnalazioni (INFND_UAT_ISSUE_H),
        allegati (INFND_UAT_ATTACH_L), commenti (INFND_UAT_CMT_L), storico stati
        (INFND_UAT_HIST_L), liste di valori configurabili (INFND_UAT_LOV) e
        anagrafica casi d'uso con owner per il re-test (INFND_UAT_USECASE).

2.  [DB] Il package INFND_UAT_MGR_PKG centralizza la logica applicativa:
        inserimento e aggiornamento segnalazioni, transizioni di stato con
        validazioni (precondizioni workflow, campi obbligatori condizionali),
        lookup owner use case per assegnazione automatica re-test, gestione
        commenti e allegati.

3.  [DB] Il trigger INFND_UAT_ISSUE_TRG si attiva su UPDATE di status e
        inserisce automaticamente un record in INFND_UAT_HIST_L come meccanismo
        di audit complementare alla logica della procedura TRANSITION_ISSUE.

4.  [DB → ORDS] Il modulo ORDS uat_api_v1 (base path /uat/api/v1/) espone:
        - GET con SELECT dirette: lista/dettaglio issues, commenti, metadati
          allegati, LOV, use case;
        - POST con handler PL/SQL (INFND_UAT_MGR_PKG): creazione issue,
          aggiornamento campi, transizioni stato, commenti, upload allegati;
        - GET con SELECT BLOB: download contenuto allegato.

5.  [ORDS → VBCS] La pagina IssueList chiama GET /issues con query params
        (filtri multi-valore, paginazione server-side, sorting) e
        GET /lov/:type per popolare i Select. Naviga a IssueCreateEdit
        o IssueDetail.

6.  [ORDS → VBCS] La pagina IssueCreateEdit popola le LOV via GET /lov/:type
        e GET /usecases (filtrato per domainStream), poi invia la segnalazione
        via POST /issues con status=DRAFT (Salva bozza) o status=NEW (Invia).

7.  [ORDS → VBCS] La pagina IssueDetail carica dettaglio, commenti e allegati
        tramite GET dedicati. Le transizioni di stato avvengono via
        POST /issues/:issueId/transition: per SEND_TO_RETEST la procedura
        esegue il lookup owner_user_name su INFND_UAT_USECASE e lo valorizza
        come retest_assigned_to, con fallback su created_by.
```

---

## 2. Database

**Schema**: `STAGING_PAAS`

> **Assunzione A1**: lo schema `STAGING_PAAS` è usato come schema di riferimento, in linea con la sua destinazione a estensioni applicative custom. Il funzionale menziona uno schema `UAT_BUGS` non presente nelle naming conventions — vedi Punti aperti P1.

---

### Tabelle

| Nome | Schema | Descrizione |
|------|--------|-------------|
| `INFND_UAT_ISSUE_H` | STAGING_PAAS | Tabella principale delle segnalazioni UAT. Contiene: contesto (solution_type, application_module, environment, domain_stream, use_case_code), classificazione (record_type, severity, priority, frequency, impact_flags), descrizioni CLOB (title, expected_behavior, actual_behavior, steps_to_reproduce, notes), dati workflow (status, assignee, resolution_notes, fix_version, fix_environment, retest_assigned_to, retest_outcome, retest_notes, closure_reason, closure_evidence_link), campi di chiusura/risoluzione (resolved_on, resolved_by, closed_on, closed_by), auto-referenza per duplicati (duplicate_of FK a issue_id), WHO columns (created_by, created_on, updated_by, updated_on) e context_payload CLOB per dettagli variabili SaaS/PaaS futuri. |
| `INFND_UAT_ATTACH_L` | STAGING_PAAS | Allegati delle segnalazioni. Contiene metadati (file_name, mime_type, file_size, description), contenuto BLOB (file_content), WHO columns upload (uploaded_on, uploaded_by). FK su issue_id verso INFND_UAT_ISSUE_H con ON DELETE CASCADE. |
| `INFND_UAT_CMT_L` | STAGING_PAAS | Commenti delle segnalazioni. Gestisce timeline commenti con visibilità PUBLIC/INTERNAL, autore (comment_by) e timestamp (comment_on). FK su issue_id con ON DELETE CASCADE. |
| `INFND_UAT_HIST_L` | STAGING_PAAS | Storico dei cambi di stato (audit workflow). Registra old_status, new_status, changed_by, changed_on, change_note per ogni transizione. Alimentato dal trigger INFND_UAT_ISSUE_TRG e dalla procedura TRANSITION_ISSUE. FK su issue_id con ON DELETE CASCADE. |
| `INFND_UAT_LOV` | STAGING_PAAS | Configurazione centralizzata di tutte le liste di valori. Discriminata per lov_type con chiave composta (lov_type, lov_code). Valori: sort_order per ordinamento logico, enabled_yn per disabilitare voci senza cancellazione. LOV_TYPE previsti: STATUS, SEVERITY, PRIORITY, RECORD_TYPE, DOMAIN_STREAM, ENVIRONMENT, COMPONENT, CLOSURE_REASON, FREQUENCY. |
| `INFND_UAT_USECASE` | STAGING_PAAS | Anagrafica casi d'uso con owner per assegnazione re-test. Colonne chiave: use_case_code (PK), use_case_title, domain_stream (CONTABILITA/PROCUREMENT), owner_user_name (username per retest_assigned_to), owner_email (NOT NULL, per notifiche), enabled_yn, WHO columns. |

---

### Package

| Nome | Schema | Descrizione |
|------|--------|-------------|
| `INFND_UAT_MGR_PKG` | STAGING_PAAS | Package principale per la gestione delle segnalazioni UAT. Punto di ingresso unico per tutti gli handler ORDS di tipo POST. Centralizza: inserimento nuove segnalazioni con validazioni obbligatorie, aggiornamento campi non-workflow, transizioni di stato con precondizioni e validazioni per azione, lookup owner use case per SEND_TO_RETEST, inserimento commenti e upload allegati. |

---

### Procedure / Funzioni

| Nome | Package | Descrizione | Input principali | Output |
|------|---------|-------------|-----------------|--------|
| `INS_ISSUE` | `INFND_UAT_MGR_PKG` | Inserisce una nuova segnalazione. Valida i campi obbligatori (solution_type, application_module, environment, domain_stream, record_type, severity, title, expected_behavior, actual_behavior, steps_to_reproduce). Imposta status = DRAFT o NEW in base al parametro p_status. Valorizza created_by e created_on. | p_solution_type, p_app_module, p_environment, p_domain_stream, p_use_case_code, p_page_url, p_tenant_pod, p_app_version, p_record_type, p_severity, p_priority, p_frequency, p_impact_flags, p_title, p_expected_behavior, p_actual_behavior, p_error_message, p_prerequisites, p_steps_to_reproduce, p_reproducible_yn, p_notes, p_status, p_created_by | p_issue_id NUMBER, p_status VARCHAR2, p_result VARCHAR2 |
| `UPD_ISSUE` | `INFND_UAT_MGR_PKG` | Aggiorna i campi non-workflow di una segnalazione (title, expected_behavior, actual_behavior, error_message, prerequisites, steps_to_reproduce, reproducible_yn, notes, page_url, tenant_pod, app_version, impact_flags, component, assignee, target_release). Non gestisce transizioni di stato. Aggiorna updated_on e updated_by. | p_issue_id, campi modificabili, p_updated_by | p_result VARCHAR2 |
| `TRANSITION_ISSUE` | `INFND_UAT_MGR_PKG` | Gestisce le transizioni di stato del workflow. Verifica la precondizione (stato corrente), applica l'azione richiesta con le validazioni specifiche, aggiorna i campi di stato e inserisce il record storico in INFND_UAT_HIST_L. Azioni supportate: PROPOSE_SOLUTION (IN_PROGRESS→SOLUTION_PROPOSED), SEND_TO_RETEST con lookup owner use case (SOLUTION_PROPOSED→RETEST), CONFIRM_RESOLUTION (RETEST→RESOLVED), REOPEN_FAIL (RETEST→IN_PROGRESS), CLOSE (RESOLVED→CLOSED). | p_issue_id, p_action, p_user, p_resolution_notes, p_fix_version, p_fix_environment, p_retest_instructions, p_retest_assigned_to, p_retest_outcome, p_retest_notes, p_closure_reason, p_closure_evidence_link | p_new_status VARCHAR2, p_retest_assigned_to VARCHAR2, p_result VARCHAR2 |
| `INS_COMMENT` | `INFND_UAT_MGR_PKG` | Inserisce un commento su una segnalazione. Gestisce visibilità PUBLIC/INTERNAL. Valorizza comment_on (SYSTIMESTAMP) e comment_by. | p_issue_id, p_comment_text, p_visibility, p_comment_by | p_comment_id NUMBER, p_result VARCHAR2 |
| `INS_ATTACHMENT` | `INFND_UAT_MGR_PKG` | Inserisce metadati e contenuto BLOB di un allegato. Valorizza uploaded_on (SYSTIMESTAMP) e uploaded_by. | p_issue_id, p_file_name, p_mime_type, p_file_size, p_file_content BLOB, p_description, p_uploaded_by | p_attachment_id NUMBER, p_result VARCHAR2 |

---

### Trigger

| Nome | Tabella | Descrizione |
|------|---------|-------------|
| `INFND_UAT_ISSUE_TRG` | `INFND_UAT_ISSUE_H` | Trigger BEFORE UPDATE OF status. Quando NEW.status ≠ OLD.status, inserisce automaticamente un record in INFND_UAT_HIST_L con old_status, new_status, SYSTIMESTAMP e updated_by come changed_by. Audit complementare alla procedura TRANSITION_ISSUE. |

---

### Indici

| Nome | Tabella | Colonna | Note |
|------|---------|---------|------|
| `INFND_UAT_ISSUE_STATUS_IND` | `INFND_UAT_ISSUE_H` | status | Filtri lista |
| `INFND_UAT_ISSUE_SEVERITY_IND` | `INFND_UAT_ISSUE_H` | severity | Filtri lista |
| `INFND_UAT_ISSUE_STREAM_IND` | `INFND_UAT_ISSUE_H` | domain_stream | Filtri lista |
| `INFND_UAT_ISSUE_ASSIGNEE_IND` | `INFND_UAT_ISSUE_H` | assignee | Filtri lista |
| `INFND_UAT_ISSUE_CREATED_IND` | `INFND_UAT_ISSUE_H` | created_on | Sorting default |
| `INFND_UAT_ATTACH_ISSUE_IND` | `INFND_UAT_ATTACH_L` | issue_id | Join su issue |
| `INFND_UAT_CMT_ISSUE_IND` | `INFND_UAT_CMT_L` | issue_id | Join su issue |
| `INFND_UAT_HIST_ISSUE_IND` | `INFND_UAT_HIST_L` | issue_id | Join su issue |
| `INFND_UAT_HIST_CHANGED_IND` | `INFND_UAT_HIST_L` | changed_on | Ordinamento timeline |
| `INFND_UAT_USECASE_STREAM_IND` | `INFND_UAT_USECASE` | domain_stream | Filtro GET /usecases |
| `INFND_UAT_USECASE_OWNER_IND` | `INFND_UAT_USECASE` | owner_user_name | Lookup re-test |

---

### LOV — Valori definitivi (seed `INFND_UAT_LOV`)

| LOV_TYPE | CODE | LABEL | SORT |
|----------|------|-------|------|
| DOMAIN_STREAM | CONTABILITA | Contabilità | 10 |
| DOMAIN_STREAM | PROCUREMENT | Procurement | 20 |
| ENVIRONMENT | SVI | SVI | 10 |
| ENVIRONMENT | CALL | CALL | 20 |
| STATUS | DRAFT | Bozza | 10 |
| STATUS | NEW | Nuovo | 20 |
| STATUS | TRIAGE | In triage | 30 |
| STATUS | IN_PROGRESS | In lavorazione | 40 |
| STATUS | SOLUTION_PROPOSED | Soluzione proposta | 50 |
| STATUS | RETEST | In re-test | 60 |
| STATUS | RESOLVED | Risolto | 70 |
| STATUS | CLOSED | Chiuso | 80 |
| STATUS | WAITING_INFO | In attesa info | 90 |
| STATUS | NOT_REPRODUCIBLE | Non riproducibile | 100 |
| STATUS | DUPLICATE | Duplicato | 110 |
| STATUS | REJECTED | Scartato | 120 |
| RECORD_TYPE | BUG | Bug | 10 |
| RECORD_TYPE | INCIDENT | Incident | 20 |
| RECORD_TYPE | IMPROVEMENT | Miglioria | 30 |
| RECORD_TYPE | QUESTION | Chiarimento | 40 |
| RECORD_TYPE | DATA | Dato | 50 |
| RECORD_TYPE | ACCESS | Accesso | 60 |
| SEVERITY | BLOCKER | Blocker | 10 |
| SEVERITY | HIGH | High | 20 |
| SEVERITY | MEDIUM | Medium | 30 |
| SEVERITY | LOW | Low | 40 |
| PRIORITY | P1 | P1 | 10 |
| PRIORITY | P2 | P2 | 20 |
| PRIORITY | P3 | P3 | 30 |
| FREQUENCY | ALWAYS | Sempre | 10 |
| FREQUENCY | OFTEN | Spesso | 20 |
| FREQUENCY | RARELY | Raramente | 30 |
| FREQUENCY | ONCE | Prima volta | 40 |
| COMPONENT | UI | UI | 10 |
| COMPONENT | INTEGRATION | Integration | 20 |
| COMPONENT | DATA | Data | 30 |
| COMPONENT | SECURITY | Security | 40 |
| COMPONENT | WORKFLOW | Workflow | 50 |
| COMPONENT | REPORTING | Reporting | 60 |
| CLOSURE_REASON | FIX_VERIFIED | Fix verificato | 10 |
| CLOSURE_REASON | WORKAROUND_ACCEPTED | Workaround accettato | 20 |
| CLOSURE_REASON | DUPLICATE | Duplicato | 30 |
| CLOSURE_REASON | NOT_REPRODUCIBLE | Non riproducibile | 40 |
| CLOSURE_REASON | OUT_OF_SCOPE | Fuori ambito UAT | 50 |

> **Nota**: il sorting di SEVERITY e PRIORITY usa il campo sort_order della LOV (BLOCKER=10, HIGH=20, …) tramite JOIN o CASE expression lato SQL, per ottenere ordinamento logico nella lista issue.

---

## 3. ORDS

### Moduli

| Nome modulo | Base path | Schema |
|-------------|-----------|--------|
| `uat_api_v1` | `/uat/api/v1/` | STAGING_PAAS |

---

### Endpoint

| Modulo | Template | Metodo | Tipo handler | Input | Output | Descrizione |
|--------|----------|--------|-------------|-------|--------|-------------|
| `uat_api_v1` | `lov/:type` | GET | SELECT su `INFND_UAT_LOV` | type (path param) | `{data:[{code,label,sortOrder}]}` | Voci attive di una LOV filtrate per lov_type. Usato da VBCS per tutti i Select/Radio. |
| `uat_api_v1` | `usecases` | GET | SELECT su `INFND_UAT_USECASE` | domainStream (query param), enabled (default Y) | `{data:[{useCaseCode,useCaseTitle,domainStream,ownerUserName,ownerEmail}]}` | Casi d'uso filtrati per domainStream. Usato in IssueCreateEdit per il campo Use Case. |
| `uat_api_v1` | `usecases/:useCaseCode` | GET | SELECT su `INFND_UAT_USECASE` | useCaseCode (path param) | `{data:{useCaseCode,useCaseTitle,ownerUserName,ownerEmail}}` | Dettaglio singolo caso d'uso. |
| `uat_api_v1` | `issues` | GET | SELECT su `INFND_UAT_ISSUE_H` | status, severity, priority, recordType, domainStream, environment, solutionType, assignee, createdBy, retestAssignedTo, useCaseCode, q (testo su title), page (default 1), pageSize (default 20, max 100), sort (es. createdOn:desc — multi-sort con virgola) | `{data:[{issueId,title,domainStream,environment,solutionType,severity,status,assignee,createdOn,updatedOn}], meta:{page,pageSize,total,totalPages,sort}}` | Lista segnalazioni con filtri (equality + IN list con separatore `,`), ricerca testuale su title via LIKE, paginazione server-side e sorting su whitelist (createdOn, updatedOn, severity, status, domainStream, environment, assignee, priority). Sorting severity/priority usa sort_order da LOV. |
| `uat_api_v1` | `issues` | POST | PL/SQL `INFND_UAT_MGR_PKG.INS_ISSUE` | JSON body: solutionType, applicationModule, environment, domainStream, useCaseCode, recordType, severity, title, expectedBehavior, actualBehavior, stepsToReproduce, reproducible (Y/N), status (DRAFT\|NEW), + campi opzionali | `{data:{issueId,status}}` | Crea nuova segnalazione. status=DRAFT per bozza, status=NEW per invio ufficiale. |
| `uat_api_v1` | `issues/:issueId` | GET | SELECT su `INFND_UAT_ISSUE_H` | issueId (path param) | `{data:{oggetto completo con tutti i campi, inclusi CLOB e campi workflow}}` | Dettaglio completo di una segnalazione. Usato da IssueDetail. |
| `uat_api_v1` | `issues/:issueId` | POST | PL/SQL `INFND_UAT_MGR_PKG.UPD_ISSUE` | issueId (path param), JSON body: campi non-workflow modificabili | `{data:{issueId,updatedOn}}` | Aggiornamento parziale campi non di stato. |
| `uat_api_v1` | `issues/:issueId/transition` | POST | PL/SQL `INFND_UAT_MGR_PKG.TRANSITION_ISSUE` | issueId (path param), JSON body: `{action, payload:{...}}` — azioni: PROPOSE_SOLUTION, SEND_TO_RETEST, CONFIRM_RESOLUTION, REOPEN_FAIL, CLOSE | `{data:{issueId,status,retestAssignedTo}}` | Centralizza tutte le transizioni di stato con validazioni. Per SEND_TO_RETEST: lookup owner_user_name da INFND_UAT_USECASE se retestAssignedTo è null, fallback su created_by. |
| `uat_api_v1` | `issues/:issueId/comments` | GET | SELECT su `INFND_UAT_CMT_L` | issueId (path param) | `{data:[{commentId,commentText,visibility,commentOn,commentBy}]}` | Commenti ordinati per commentOn ASC. Usato nel tab Commenti di IssueDetail. |
| `uat_api_v1` | `issues/:issueId/comments` | POST | PL/SQL `INFND_UAT_MGR_PKG.INS_COMMENT` | issueId (path param), JSON body: `{commentText, visibility}` | `{data:{commentId}}` | Aggiunge un commento. |
| `uat_api_v1` | `issues/:issueId/attachments` | GET | SELECT su `INFND_UAT_ATTACH_L` | issueId (path param) | `{data:[{attachmentId,fileName,mimeType,fileSize,description,uploadedOn,uploadedBy}]}` | Lista metadati allegati (senza BLOB). Usato nel tab Allegati di IssueDetail. |
| `uat_api_v1` | `issues/:issueId/attachments` | POST | PL/SQL `INFND_UAT_MGR_PKG.INS_ATTACHMENT` | issueId (path param), multipart/form-data: file (BLOB), description | `{data:{attachmentId}}` | Upload allegato BLOB. |
| `uat_api_v1` | `attachments/:attachmentId/content` | GET | SELECT BLOB su `INFND_UAT_ATTACH_L` | attachmentId (path param) | Binario con Content-Type = mime_type | Download contenuto allegato. Usato per visualizzare screenshot e scaricare log. |

---

## 4. VBCS

> **Assunzione A2**: le naming conventions per componenti VBCS non sono ancora definite in `naming-conventions.md`. I nomi usati in questa sezione sono descrittivi e dovranno essere allineati alla convenzione quando definita — vedi Punti aperti P3.

---

### Pagine

| Nome pagina | Descrizione | Endpoint ORDS chiamati |
|-------------|-------------|------------------------|
| `IssueList` | Vista elenco segnalazioni con filtri (status multi-select, severity multi-select, domainStream, environment, solutionType, ricerca testuale), tabs quick views (Tutte / Le mie / Da re-testare / Alte priorità), tabella con paginazione server-side e sorting per colonna. Azioni riga: Apri, Cambia stato rapido (Triage), Duplica. Bottoni: Nuova segnalazione (naviga a IssueCreateEdit), Export CSV. | GET `/lov/:type` (STATUS, SEVERITY, DOMAIN_STREAM, ENVIRONMENT), GET `/issues` |
| `IssueCreateEdit` | Form in 4 card (Contesto, Descrizione, Riproducibilità, Evidenze) per creare o modificare segnalazione. Visibilità condizionale campi SaaS/PaaS. LOV Use Case filtrate dinamicamente per domainStream selezionato. Validazioni obbligatorie al submit e warning condizionali (allegato per BLOCKER/HIGH; noteAggiuntive se reproducible=No; riferimento transazione se recordType=DATA). | GET `/lov/:type` (tutti i tipi), GET `/usecases?domainStream=`, POST `/issues` |
| `IssueDetail` | Pagina dettaglio a 4 tabs. Header: #ID — Titolo con chips status/severity/env/stream/solutionType e bottoni workflow condizionati da ruolo+stato. Tab **Dettaglio**: campi read-only (edit parziale per Triage/Lead via POST /issues/:id). Tab **Allegati**: lista + upload. Tab **Commenti**: timeline PUBLIC/INTERNAL con form aggiunta. Tab **Workflow & Audit**: card azioni workflow + storico stati. | GET `/issues/:issueId`, GET `/issues/:issueId/comments`, POST `/issues/:issueId/comments`, GET `/issues/:issueId/attachments`, POST `/issues/:issueId/attachments`, GET `/attachments/:attachmentId/content`, POST `/issues/:issueId/transition`, POST `/issues/:issueId` |

---

### Componenti principali

| Pagina | Componente | Tipo | Descrizione |
|--------|------------|------|-------------|
| `IssueList` | FiltriCard | `oj-form-layout` | Filtri: domainStream (oj-select-single), environment (oj-select-single), solutionType (oj-select-single), status (oj-select-many), severity (oj-select-many), pulsante Reset. LOV caricate da GET /lov/:type. |
| `IssueList` | RicercaTestuale | `oj-input-text` | Ricerca libera su title, binding su query param q. |
| `IssueList` | TabsQuickViews | `oj-tab-bar` | Tutte / Le mie (createdBy=me) / Da re-testare (status=RETEST, retestAssignedTo=me) / Alte priorità (severity=BLOCKER,HIGH). |
| `IssueList` | IssueTable | `oj-table` | DataProvider REST su GET /issues. Colonne: ID (link), Titolo, Stream (badge), Env, Type, Severity (chip colorato per sort_order: BLOCKER=rosso, HIGH=arancio, MEDIUM=giallo, LOW=grigio), Status (pill colorata per stato), Assignee (avatar+nome), Updated (relative time). Paginazione server-side (page, pageSize 20/50/100). |
| `IssueList` | PulsanteNuovaSegnalazione | `oj-button` primary | Naviga a IssueCreateEdit in modalità creazione. |
| `IssueCreateEdit` | CardContesto | `oj-form-layout` | solutionType (oj-radioset: SaaS/PaaS), applicationModule (oj-select-single), environment (oj-select-single), domainStream (oj-select-single), useCaseCode (oj-select-single, ricariata on-change di domainStream via GET /usecases), pageUrl (oj-input-text), tenantPod (visibile se SaaS), appVersion (visibile se PaaS). |
| `IssueCreateEdit` | CardDescrizione | `oj-form-layout` | title (oj-input-text, max 120), recordType (oj-select-single), severity (oj-select-single), priority (oj-select-single), frequency (oj-select-single), impactFlags (oj-checkboxset), expectedBehavior / actualBehavior / errorMessage (oj-text-area). |
| `IssueCreateEdit` | CardRiproducibilita | `oj-form-layout` | prerequisites (oj-text-area), stepsToReproduce (oj-text-area, required), reproducible (oj-switch Sì/No), noteAggiuntive (oj-text-area, visibile e obbligatoria se reproducible=No). |
| `IssueCreateEdit` | CardEvidenze | `oj-form-layout` | allegati multipli (oj-file-picker), lista file con nome/size, riferimentiTransazione (oj-input-text, obbligatorio se recordType=DATA). |
| `IssueCreateEdit` | FooterAzioni | `oj-button` group | Salva bozza → POST /issues status=DRAFT. Invia → POST /issues status=NEW. Annulla → torna a IssueList. |
| `IssueDetail` | HeaderDettaglio | `oj-panel` | H1: #ID — Titolo. Chips status/severity/environment/domainStream/solutionType. Bottoni workflow a destra, visibilità condizionata da ruolo VBCS + status corrente. |
| `IssueDetail` | TabsDettaglio | `oj-tabs` | 4 tabs: Dettaglio, Allegati, Commenti, Workflow & Audit. |
| `IssueDetail` | AllegatiList | `oj-list-view` + `oj-file-picker` | Lista metadati allegati. Link download per ogni file via GET /attachments/:id/content. Upload nuovo allegato via POST. |
| `IssueDetail` | CommentTimeline | `oj-list-view` | Commenti ordinati ASC. Ogni item: commentBy, commentOn, badge PUBLIC/INTERNAL, testo. Form inline per nuovo commento con select visibilità. |
| `IssueDetail` | WorkflowCard | `oj-panel` | Card visibili per ruolo+stato. Proponi soluzione (Assignee/Lead, da IN_PROGRESS): resolutionNotes, fixVersion, fixEnvironment. Invia in re-test (Triage, da SOLUTION_PROPOSED): retestInstructions, retestAssignedTo (pre-valorizzato da owner use case). Re-test (Tester/Triage, da RETEST): retestOutcome radio PASS/FAIL, retestNotes (obbligatorio se FAIL). Chiusura (Triage/PMO, da RESOLVED): closureReason, closureEvidenceLink. |
| `IssueDetail` | StatusHistory | `oj-list-view` | Timeline storico stati: changed_on, old_status → new_status, changed_by, change_note. |

---

### Action Chains VBCS

| Action Chain | Pagina | Trigger | Endpoint chiamati | Descrizione |
|-------------|--------|---------|-------------------|-------------|
| `OnFilterChange` | IssueList | Cambio filtro/pagina/sort | GET `/issues` | Aggiorna DataProvider tabella con tutti i query params. |
| `NavigateToCreate` | IssueList | Click "Nuova segnalazione" | — | Naviga a IssueCreateEdit in modalità creazione. |
| `SubmitIssue` | IssueCreateEdit | Click "Invia" | POST `/issues` (status=NEW) | Valida campi, invia, toast conferma, naviga a IssueDetail. |
| `SaveDraft` | IssueCreateEdit | Click "Salva bozza" | POST `/issues` (status=DRAFT) | Salva bozza, toast conferma. |
| `LoadUseCases` | IssueCreateEdit | On-change domainStream | GET `/usecases` | Ricarica LOV Use Case filtrata per domainStream selezionato. |
| `ProposeSolution` | IssueDetail | Click "Proponi soluzione" | POST `/issues/:id/transition` (PROPOSE_SOLUTION) | Valida resolutionNotes e fixVersion (PaaS), esegue transizione, aggiorna stato in header. |
| `SendToRetest` | IssueDetail | Click "Invia in re-test" | GET `/usecases/:code`, POST `/issues/:id/transition` (SEND_TO_RETEST) | Recupera owner use case, pre-valorizza retestAssignedTo, esegue transizione. |
| `ConfirmResolution` | IssueDetail | Click "Conferma risoluzione" | POST `/issues/:id/transition` (CONFIRM_RESOLUTION) | Imposta retestOutcome=PASS, esegue transizione. |
| `ReopenFail` | IssueDetail | Click "Esito negativo / Riapri" | POST `/issues/:id/transition` (REOPEN_FAIL) | Valida retestNotes obbligatorio, imposta retestOutcome=FAIL, esegue transizione. |
| `CloseIssue` | IssueDetail | Click "Chiudi" | POST `/issues/:id/transition` (CLOSE) | Valida closureReason obbligatorio, esegue transizione, notifica reporter. |

---

## 5. Assunzioni e punti aperti

### Assunzioni fatte

- **A1** — Schema `STAGING_PAAS` usato come standard per estensioni custom. Il funzionale suggerisce `UAT_BUGS` che non è nelle naming conventions.
- **A2** — Naming ORDS e VBCS non è definita in `naming-conventions.md`: i nomi usati sono descrittivi/funzionali, da allineare a convenzione definitiva.
- **A3** — Prefisso modulo `INFND` (common/cross-domain) usato per tutti gli oggetti DB, per assenza di un prefisso dedicato al dominio UAT/tracking.
- **A4** — Le notifiche email (eventi: NEW, RETEST, RESOLVED, CLOSED) sono gestite lato VBCS tramite action chain. Il payload di notifica include: issueId, title, status, link pagina VBCS, domainStream, environment.
- **A5** — Gli allegati sono persistiti come BLOB su ATP, soluzione adeguata per i volumi attesi in fase UAT.
- **A6** — L'aggiornamento parziale di una segnalazione è esposto come POST (non PATCH) per coerenza con la convenzione di progetto (POST per procedure PL/SQL).
- **A7** — Il campo `context_payload` (CLOB JSON) è predisposto nella tabella ma non esposto nell'UI MVP.
- **A8** — L'identità utente corrente è passata agli handler ORDS tramite header (es. `X-User`) o token OAuth2/SSO.

### Punti aperti da risolvere prima dello sviluppo

- **P1** — **Schema dedicato vs STAGING_PAAS**: confermare se usare `STAGING_PAAS` o creare uno schema dedicato (es. `UAT_BUGS`). Impatta permessi DBA, configurazione ORDS e isolamento dati.
- **P2** — **Prefisso naming DB**: `INFND` è il più prossimo disponibile ma non copre esplicitamente il dominio UAT. Valutare nuovo prefisso (es. `INUAT`) o confermare `INFND`.
- **P3** — **Naming ORDS e VBCS**: le convenzioni per nomi di moduli ORDS e componenti VBCS non sono definite in `naming-conventions.md`. Da definire prima dello sviluppo.
- **P4** — **Meccanismo identità utente**: definire come lo username corrente viene propagato alle procedure (header `X-User`, token OAuth2/IDCS, altro). Impatta tutti i campi who-columns.
- **P5** — **Email notifiche — implementazione**: confermare se le notifiche vengono inviate lato VBCS (action chain + servizio mail/OIC) o lato DB (procedura + job scheduler). Per il fallback email del reporter (quando use_case_code è null): definire se ricavare l'email da SSO/identity provider o aggiungere il campo `reporter_email` su `INFND_UAT_ISSUE_H`.
- **P6** — **Upload multipart in VBCS**: verificare la compatibilità tra `oj-file-picker` e l'endpoint POST multipart/form-data di ORDS. Se non supportato, valutare encoding base64 nel body JSON.
- **P7** — **Sorting severity/priority**: il documento indica CASE expression SQL o JOIN su LOV.sort_order per ordinamento logico. Definire l'approccio preferito prima di implementare GET /issues.
- **P8** — **Ricerca full-text Oracle Text**: l'MVP usa LIKE su title. Definire se predisporre l'indice CTX fin da subito o aggiungerlo come evoluzione.
