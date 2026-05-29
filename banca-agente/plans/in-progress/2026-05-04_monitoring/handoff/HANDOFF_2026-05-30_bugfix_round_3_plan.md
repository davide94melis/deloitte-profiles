# Handoff BUG-FIX ROUND 3 — Pianificazione fedele al BR — preparato 2026-05-29

Sessione successiva = **chiusura cluster bug rimanenti monitoring (13 aperti)** secondo i riferimenti BR esatti. Bug #10 e' stato chiuso end-to-end nella sessione precedente (vedi `HANDOFF_2026-05-29_bug10_CLOSED_smoke_full.md`). Branch BE+FE `feature/monitoring-integration-uat` allineati con remote.

Tutti i fix sotto sono ancorati a sezioni precise del BR `requirements/BR_Monitoraggio_V6.md` per evitare deviazioni e mantenere semantica fedele agli stakeholder.

---

## Stato di partenza

### Branch
- BE `feature/monitoring-integration-uat` HEAD `5f22718` (allineato origin, pushed)
- FE `feature/monitoring-integration-uat` HEAD `08f8025` (allineato origin, pushed)
- DB locale `ba-be-local`: 8 docs M1 (`mp_monitoring_document` id 3..10), pratica 8288 `MONITORING_FUORI_SOGLIA`, M1C1 LOAN_TO_VALUE con 6 eventi

### Bug list cumulativa
**18 totali, 5 fixati cumulativi, 13 aperti** (+1 nuovo smoke finding da aggiungere come #19):

| Fixati | Aperti P2 MEDIUM | Aperti P3 LOW | By-design |
|---|---|---|---|
| #16 #18 #2 #10 + (#13 rimosso) | #1 #3 #5 #6 #8 #9 #11 #12 #15 #17 | #4 #14 | #7 |

### Nuovo bug smoke #19 (da aggiungere alla lista)
**MEDIUM** — `documentInfo` stale dopo GenAI extraction in-page → button "Ritorna a documentazione incompleta" appare solo dopo reload manuale.
- Componente: `monitoring-cc-assignment-page.component.ts:425` (polling `startGenAi`)
- Fix proposto: dopo `extraction.status === READY_FOR_VALIDATION` chiamare anche `this.loadDocumentInfo()` per refreshare `documentInfo.status` da `DA_ASSEGNARE` a `GENAI_DA_VALIDARE`
- BR ref: BR 4.2.3 (page-based UX) — non esplicitamente menzionato, ma il button "Ritorna a documentazione incompleta" deve essere disponibile post-extraction senza reload
- Test: smoke E2E ripetere flow Reject senza reload manuale → button visibile subito post-extraction

---

## Strategia: cluster ragionati per area + dipendenze

I 13 bug aperti si raggruppano naturalmente in **6 cluster** che condividono file/contesto. Ordine raccomandato basato su (a) impatto stakeholder, (b) dipendenze fra fix, (c) dimensione cluster, (d) ritmo di smoke.

### Cluster A — Storage & metadata documenti (BE, 1-2h) — P2
**Bug #1, #3** — semantica documenti
- **#1 file_reference=NULL document eccezione**: BR sez. 4.2.6 (riga 773) richiede `requestException` salvi documenti a supporto scaricabili. Attualmente `RequestExceptionService` salva metadati ma `file_reference` NULL nel DB → download fallisce.
- **#3 [object Object] "Caricato da"**: BR sez. 3.2.3 (riga 321-345) e 4.2.2 (riga 644) descrivono colonna "Caricato da" dello Storico Documenti → deve mostrare username/email. Il bug e' visivo: la cella mostra letterale `[object Object]` invece di una stringa, indicando binding errato (probabile `uploadedBy` come oggetto User invece che string email).

### Cluster B — Schedule generator (BE, 30min-1h) — P2
**Bug #6** — invariante BR sez. 1.1 (riga 142-149)
- **#6 PUT covenant non rigenera eventi se cambia `periodicity`**: BR specifica che lo scadenziere e' funzione di `periodicity` e `terminationDate` (riga 142, 149). Se Deloitte modifica `periodicity` (BR sez. 4.2.5 riga 757 "Modifica Dati Covenant"), gli eventi futuri devono essere rigenerati con il nuovo intervallo (1/3/6/12 mesi).

### Cluster C — Enum & label alignment (BE+FE, 30min) — P2+P3
**Bug #4, #8, #9** — drift fra label e enum
- **#4 (LOW) `CovenantTypeNumericEnum` FE 10/58**: drift enum FE vs BE (T-024.x originale). Ratio per fix: l'utente Deloitte (BR sez. 4.2.5 riga 757) deve poter scegliere `Tipologia Covenant` dal dropdown coerente con la lista valida BE.
- **#8 `MonitoringPracticeExcelWriterService` enum prefix**: export Excel mostra valori enum raw (`MONITORING_OK`) invece di label leggibili. BR sez. 8.4 (riga 1128+) descrive export tabelle pubbliche → label tradotte.
- **#9 Mock GenAI label vs enum**: `AsyncMonitoringGenAiProcess.buildMockExtractedFields()` restituisce `COVENANT_NUM_TIPOLOGIA: "Esempio Covenant Mock"` non-enum. Il fix `1450e37` (Wave 2 FE bug #10) si appoggia al match enum → fallisce con mock. **Conferma smoke sessione precedente: in produzione con GenAI reale funziona; in DEV/UAT serve allineare il mock** ai valori validi `MonitoringGenAiKeyEnum`/`MonitoringCovenantTypeEnum`.

### Cluster D — Mailing list UX (FE, 1h) — P2+P3
**Bug #11, #12, #14, #15** — feedback UX upload + popup ISP
- **#11 (MEDIUM) Upload AD Form senza spinner**: BR sez. 6.3 (riga 935-962) descrive il flusso upload AD Form. Atteso pattern `SpinnerComponent.showLoader/hideLoader` come nei flussi STEP 7 GenAI. Fix: wrap call upload in `mailing-list-detail.component.ts`.
- **#12 (MEDIUM) File upload validation senza messaggio testuale**: BR non specifica formato dei messaggi di errore, ma BR 6.3 richiede UX usabile. PrimeNG `p-fileupload` mostra solo 2 icone X rosse → user perde feedback motivo. Fix: hook `onError`/`onValidationError` → toast.
- **#14 (LOW) Bottone "Conferma e Aggiorna" visibile in `REPORT_GENERATO`**: BR sez. 6.3 (riga 962) esplicita "una volta che lo status è passato in 'Report Generato' la colonna presenterà l'azione 'Scarica Report'". Il bottone "Conferma e Aggiorna" non deve essere cliccabile/visibile per stati gia conclusi.
- **#15 (MEDIUM) Popup ISP "Visualizza Richiesta Aggiornamento" senza Chiudi**: BR sez. 6.3 (riga 949) descrive il popup. La UX best-practice + le altre modali del modulo hanno sempre bottone Chiudi/X. Fix: `[closable]="true"` + bottone "Chiudi" neutrale.

### Cluster E — Covenant detail UX (FE, 30min) — P2
**Bug #5** — campo "Variabile" SI/NO
- **#5 Field "Variabile" input text vs dropdown SI/NO**: BR sez. 1.1 (riga 127) definisce "Covenant Variabile (Si/No)" come campo binario. BR sez. 4.2.5 (riga 757) conferma "Covenant Variabile (SI/NO) - **Dropdown**". Fix: sostituire input text con `<p-dropdown>` opzioni `{label:'Si',value:true},{label:'No',value:false}`. File: covenant-detail.component (modalita' edit).

### Cluster F — Expired/Expiring action cell (FE, 1-1.5h) — P2
**Bug #17** — colonna Azione clickable
- **#17 Cella "Azione" stringa raw "UPDATE"**: BR sez. 8.4.3 (riga 1231) + 8.5 (riga 1335) richiedono link "Salta monitoraggio" / "Ignora Soglia" per ISP, "Visualizza Eccezione" per Deloitte (BR sez. 8.4 riga 850). Attualmente cella mostra stringa raw → no interazione. Pattern di riferimento esistente: `buildActionCell` in `practice-detail.component.ts` (T-019). Fix: refactor `mapEventToRow` in `expired-events.component.ts` + `expiring-events.component.ts` con `DynamicComponentModel<ButtonComponent>` role-gated + wire `MonitoringExceptionDialogModule` via `@ViewChild`.

### Cluster G — Smoke finding nuovo (FE, 15min) — P2 NEW
**Bug #19** — `documentInfo` refresh post GenAI extraction
- Vedi dettaglio sopra "Nuovo bug smoke #19".

### By-design (non fixare)
- **#7** — `MailingListAdForm.state VERIFICA_IN_CORSO` mai persistito (placeholder GenAI). Riesaminare quando GenAI reale sostituisce mock.

---

## Dettaglio per cluster con riferimenti BR + plan

### Cluster A — Documenti & metadata

#### Bug #1 — `file_reference=NULL` document eccezione
**Severity**: MEDIUM | **BR**: sez. 4.2.6 (riga 773-775), sez. 3.4 (Gestione Eccezioni ISP riga ~480)

**Comportamento atteso BR**: ISP/Deloitte deve poter scaricare documenti allegati alla richiesta di eccezione (es. supporting docs).

**Root cause stimata**: `MonitoringExceptionService.requestException(...)` salva i metadati `MonitoringEventException` ma NON persiste il binario o il riferimento allo storage. Il record `MonitoringDocument` collegato ha `file_reference=NULL`. La gap T-024.x era nota dall'STEP 6 smoke (handoff 2026-05-24).

**Plan fix**:
1. Read `MonitoringExceptionService.requestException(Long eventId, MultipartFile file, String reason)` in BE
2. Read pattern di reference da `MonitoringDocumentService.uploadDocument` (gia funzionante per CC: usa `documentManagerService.storeFile(file, "PRACTICE", practiceId)` o simile)
3. Applicare stesso pattern in `requestException`: prima `storeFile()` → ottieni `fileReference` → poi crea `MonitoringEventException` con `fileReference` settato
4. Wire endpoint `GET /monitoring/exceptions/{id}/download` se non esistente
5. Smoke: ISP carica eccezione con allegato PDF → Deloitte vede "Visualizza Eccezione" → button Download → file scaricato OK

**File impattati**:
- BE: `MonitoringExceptionService.java`, `MonitoringExceptionController.java`, possibilmente migration SQL
- FE: `monitoring-exception-dialog.component.{ts,html}` (verifica button download presente)

**Test**:
- Unit `MonitoringExceptionServiceTest.requestExceptionStoresFile`
- E2E manual: upload allegato → download dal dettaglio eccezione

---

#### Bug #3 — `[object Object]` colonna "Caricato da"
**Severity**: MEDIUM | **BR**: sez. 3.2.3 (riga 341), sez. 4.2.2 (riga 642-644)

**Comportamento atteso BR**: Storico Documenti mostra colonna "Caricato da" come username/email leggibile dell'uploader.

**Root cause stimata**: il backend probabilmente serializza `User` come oggetto JSON `{id, username, email, ...}` invece che come stringa. Il template Angular fa `{{ doc.uploadedBy }}` → toString su Object → `[object Object]`. Possibile che Alexios abbia gia refattorizzato nel branch `feature/monitoring-bugfixes` (da verificare).

**Plan fix**:
1. Verifica `git log --oneline --all -- 'src/**/MonitoringDocumentDTO*'` per check refactor Alexios
2. Read DTO `MonitoringDocumentDTO.java` BE → campo `uploadedBy` deve essere `String` (email/username) NON `UserDTO`
3. Read `MonitoringDocumentService.toDto(...)` → estrarre `entity.getUploadedBy().getUsername()` se entity ha `User` ref
4. FE: template binding `{{ doc.uploadedBy }}` (no cambio se DTO ora stringa)
5. Test sample API: `GET /monitoring/practices/{id}/documents` → payload `uploadedBy` = email string

**File impattati**:
- BE: `MonitoringDocumentDTO.java`, `MonitoringDocumentService.toDto`
- FE: nessun cambio se DTO stringa (eventualmente fix template se ancora obj)

**Test**:
- Unit `MonitoringDocumentServiceTest.toDtoReturnsUsernameString`
- Smoke browser: storico documenti M1 colonna "Caricato da" mostra `davide94x@gmail.com` (non `[object Object]`)

---

### Cluster B — Schedule generator

#### Bug #6 — PUT covenant non rigenera eventi al cambio `periodicity`
**Severity**: MEDIUM | **BR**: sez. 1.1 (riga 142, 149), sez. 4.2.5 (riga 757-759)

**Comportamento atteso BR**: "ciascun evento di monitoraggio dovrà aumentare progressivamente rispettivamente di 1 mese, 3 mesi, 6 mesi e 12 mesi" (riga 149). Se la `periodicity` cambia (BR 4.2.5 "Modifica Dati Covenant" → "valore limite, **periodicità**"), gli eventi futuri devono essere rigenerati per riflettere la nuova cadenza fino a `terminationDate`.

**Root cause stimata**: `MonitoringCovenantService.update(...)` aggiorna i campi del covenant ma non chiama `MonitoringScheduleGeneratorService.regenerateEvents(covenantId)` quando `periodicity` cambia. Gli eventi esistenti restano con la vecchia cadenza.

**Plan fix**:
1. Read `MonitoringCovenantService.update(Long id, CovenantUpdateDTO patch)` BE
2. Aggiungere check: se `patch.periodicity != null && !patch.periodicity.equals(entity.getPeriodicity())` → invoca `monitoringScheduleGeneratorService.regenerateEvents(entity.getId())`
3. Read `regenerateEvents`: deve cancellare eventi futuri NON ancora monitorati (no documento, no eccezione, `actualDate=null`) e ricreare nuovi eventi con `entity.getPeriodicity()` e `entity.getTerminationDate()`
4. Attenzione: NON toccare eventi gia monitorati (`actualDate != null` o `hasException=1` o documenti COMPLETATI collegati)
5. Test: covenant con periodicity TRIMESTRALE → 4 eventi futuri → Deloitte cambia a SEMESTRALE → check API → 2 eventi futuri ricalcolati

**File impattati**:
- BE: `MonitoringCovenantService.java`, `MonitoringScheduleGeneratorService.java` (verifica method `regenerateEvents` esista; se no, crearlo)

**Test**:
- Unit `MonitoringCovenantServiceTest.updatePeriodicityRegeneratesFutureEvents`
- Unit `MonitoringScheduleGeneratorServiceTest.regenerateEventsPreservesMonitoredOnes`
- Smoke: M1 covenant M1C1 (TRIMESTRALE) → PUT con SEMESTRALE → vedere eventi futuri ricalcolati

---

### Cluster C — Enum & labels

#### Bug #4 (LOW) — `CovenantTypeNumericEnum` FE 10/58 drift
**Severity**: LOW | **BR**: sez. 4.2.5 (riga 757) "Tipologia Covenant - **Dropdown**"

**Comportamento atteso BR**: dropdown Tipologia Covenant deve mostrare l'intero set di valori validi del BE (58 tipologie covenant numeriche).

**Root cause**: enum FE hardcoded subset di 10/58 (decisione T-024 pragmatica per smoke). Drift mantenibilita.

**Plan fix**:
1. Opzione A (consigliata BE-driven): nuovo endpoint `GET /monitoring/covenant-types` ritorna `List<TypologicalDTO>` con `code` + `label`. FE service `getCovenantTypes(): Observable<TypologicalDTO[]>`. Component carica all'init e popola dropdown.
2. Opzione B (sync FE): copia manuale dei 58 valori dall'enum BE in `CovenantTypeNumericEnum.ts`. Veloce ma fragile.

**Decision**: Opzione A. Costo: ~30min totali (1 endpoint BE + 1 service FE + dropdown refactor).

**File impattati**:
- BE: `TypologicalDTO.java` esistente, `MonitoringCovenantController.java`, `MonitoringCovenantTypeEnum.java`
- FE: `monitoring.service.ts`, `covenant-detail.component.ts` (rimuove import enum, usa service)

**Test**: dropdown popolato a runtime con N=58.

---

#### Bug #8 — `MonitoringPracticeExcelWriterService` enum prefix
**Severity**: MEDIUM | **BR**: sez. 2.5/8.4 (export Excel)

**Comportamento atteso**: export Excel colonne semaforo/stato devono mostrare label leggibili ("OK", "Fuori Soglia", "In Scadenza") non valori enum raw (`MONITORING_OK`).

**Root cause**: il writer probabilmente fa `event.getStatus().name()` invece di `translationService.translate(event.getStatus().getI18nKey())` o `MonitoringPracticeStateEnum.toLabel()`.

**Plan fix**:
1. Read `MonitoringPracticeExcelWriterService.java`
2. Trova chiamate `.name()` su enum monitoring → sostituire con label getter (i18n key + lookup) oppure mapper esplicito (switch statement che produce label IT/EN)
3. Decision: per export Excel statico (no contesto i18n runtime), preferire **mapper esplicito** in italiano (default lingua BR)
4. Test: export di una pratica con monitoraggi misti → file XLSX scaricato → colonna Stato mostra "Fuori Soglia" non "MONITORING_FUORI_SOGLIA"

**File impattati**:
- BE: `MonitoringPracticeExcelWriterService.java`, eventuale `MonitoringLabelMapper.java` nuovo

**Test**: unit `MonitoringPracticeExcelWriterServiceTest.writeRowsConvertsEnumToHumanLabel`

---

#### Bug #9 — Mock GenAI label non-enum
**Severity**: MEDIUM | **BR**: sez. 4.2.3 (riga 722-744)

**Comportamento atteso**: il flusso GenAI deve compilare i covenant in modo coerente (BR riga 728-732 "il modello fornira' un elenco di N covenant identificabili dalla tipologia"). Il mock deve simulare un output realistico per permettere UAT.

**Root cause** (gia analizzato in smoke sessione 3): `AsyncMonitoringGenAiProcess.buildMockExtractedFields()` ritorna stringhe libere come `"Esempio Covenant Mock"`, `"<="`, `"Estratto automaticamente in mock mode"`. Il merge BE (`MonitoringGenAiService.mergeCovenantPatch`) e il match FE (`buildSlotsFromExtraction` di cc-assignment-page) richiedono valori validi dei rispettivi enum.

**Plan fix**:
1. Read `AsyncMonitoringGenAiProcess.buildMockExtractedFields()` in BE
2. Allineare i valori mock agli enum BE:
   - `COVENANT_NUM_TIPOLOGIA` → valore valido di `MonitoringCovenantTypeEnum` (es. `"LOAN_TO_VALUE"`)
   - `COVENANT_NUM_VALORE_SEGNO` → `"LTE"` non `"<="` (enum `CovenantSignEnum`)
   - `COVENANT_NUM_PERIODICITA` → valore valido di `MonitoringPeriodicityEnum` (es. `"TRIMESTRALE"`)
   - `COVENANT_NUM_VARIABILE` → `"true"`/`"false"` o `"SI"`/`"NO"` allineato a parser BE
3. Aggiornare test mock `AsyncMonitoringGenAiProcessTest` se esiste
4. Smoke E2E ripetere flow GenAI sessione 3 → tipologia ora prefilled correttamente nello slot GENAI_PREFILLED senza dover selezionare manualmente

**File impattati**:
- BE: `AsyncMonitoringGenAiProcess.java`, `MonitoringGenAiKeyEnum.java` (verifica)

**Test**: unit allineato + smoke E2E (no UI changes richiesti, il fix `1450e37` lavorera correttamente)

---

### Cluster D — Mailing list UX

#### Bug #11 — Upload AD Form senza spinner
**Severity**: MEDIUM | **BR**: sez. 6.3 (riga 935-954)

**Comportamento atteso**: durante upload AD Form l'utente deve avere feedback visivo (spinner/progress) come negli altri flussi (es. GenAI extraction STEP 7).

**Root cause**: nessun wrap della Observable upload con `StatusDialogService`/`SpinnerService`.

**Plan fix**:
1. Read `mailing-list-detail.component.ts` metodo `onUploadAdForm(file)`
2. Pattern da copiare: `practice-detail.component.ts:onStartGenAi` → `this.spinnerService.showLoader('Estrazione GenAI in corso...')` prima della call, `hideLoader()` in `finalize()`
3. Applicare stesso pattern con messaggio `'Caricamento AD Form in corso...'`

**File**: `mailing-list-detail.component.ts`

**Test**: smoke browser → upload AD Form → spinner visibile durante request → spinner sparisce a fine

---

#### Bug #12 — Validation upload senza toast esplicativo
**Severity**: MEDIUM

**Comportamento atteso**: se l'upload fallisce per dimensione/formato, mostrare toast con messaggio leggibile.

**Plan fix**:
1. Read `<p-fileUpload>` config in `mailing-list-detail.component.html`
2. Aggiungere handler `(onError)="onUploadError($event)"` + `(invalidFileTypeMessageDetail)="..."` (PrimeNG built-in)
3. Component method `onUploadError(event)` → `statusDialogService.setStatus({severity:'error', message: 'File non valido: <reason>'})`
4. I18n keys nuove `monitoring.mailing.upload.error.{tooBig|invalidType|generic}`

**File**: `mailing-list-detail.component.{ts,html}`, `assets/i18n/it-IT.json`, `en-GB.json`

**Test**: smoke browser → upload file `.txt` (non PDF/XLSX) → toast "Tipo file non valido"; upload file 50MB (oltre limite) → toast "File troppo grande"

---

#### Bug #14 (LOW) — Bottone "Conferma e Aggiorna" in `REPORT_GENERATO`
**Severity**: LOW | **BR**: sez. 6.3 (riga 962)

**Comportamento atteso BR**: "una volta che lo status è passato in 'Report Generato' la colonna presenterà l'azione 'Scarica Report'". Implicitamente, il bottone "Conferma e Aggiorna" non deve essere disponibile in stato `REPORT_GENERATO`.

**Plan fix**:
1. Read `mailing-list-detail.component.html` template area bottone "Conferma e Aggiorna"
2. Wrap con `*ngIf="adForm?.state !== MailingListAdFormStateEnum.REPORT_GENERATO"`
3. Quando in REPORT_GENERATO, mostrare invece bottone "Scarica Report" (probabilmente gia presente?) → verifica

**File**: `mailing-list-detail.component.{ts,html}`

**Test**: smoke browser fixture mailing list M1 con AD Form in REPORT_GENERATO → bottone "Conferma e Aggiorna" non visibile, "Scarica Report" visibile

---

#### Bug #15 — Popup ISP "Visualizza Richiesta Aggiornamento" senza Chiudi
**Severity**: MEDIUM | **BR**: sez. 6.3 (riga 949)

**Comportamento atteso**: il popup deve essere chiudibile (X o bottone Chiudi). UX standard.

**Plan fix**:
1. Read modal/dialog "Visualizza Richiesta Aggiornamento" (probabilmente in `mailing-list` module ISP)
2. Aggiungere `[closable]="true"` al p-dialog
3. Inoltre aggiungere bottone "Chiudi" nel footer tra i due esistenti come fallback accessibilita

**File**: dialog component ISP

**Test**: ISP `/user/monitoring/mailing-list` → tabella "Pratiche per cui e' stato richiesto aggiornamento" → "Visualizza Richiesta" → modal con X + Chiudi

---

### Cluster E — Covenant detail UX

#### Bug #5 — Field "Variabile" SI/NO dropdown
**Severity**: MEDIUM | **BR**: sez. 1.1 (riga 127), sez. 4.2.5 (riga 757)

**Comportamento atteso BR riga 757**: `Covenant Variabile (SI/NO) - Dropdown`. Attualmente input text.

**Plan fix**:
1. Read `covenant-detail.component.html` modalita edit (cerca `formControlName="variable"` o simile)
2. Sostituire `<input type="text">` con `<p-dropdown [options]="variableOptions" formControlName="variable">`
3. `variableOptions = [{label: 'Si', value: true}, {label: 'No', value: false}]` definito in component
4. I18n eventualmente nuove chiavi `monitoring.covenant.variable.{yes|no}` se necessario

**File**: `covenant-detail.component.{ts,html}`

**Test**: covenant edit mode → dropdown Variabile con Si/No

---

### Cluster F — Expired/Expiring action cell

#### Bug #17 — Cella "Azione" stringa raw "UPDATE"
**Severity**: MEDIUM | **BR**: sez. 8.4.3 (riga 1231), sez. 8.5 (riga 1335), sez. 8.4 vista Deloitte (riga 850)

**Comportamento atteso BR**:
- ISP: link "Salta Monitoraggio" o "Ignora Soglia" a seconda dello stato (riga 463-465):
  - "Salta Monitoraggio" se monitoraggio senza CC, `actualDate=null`, `expirationDate < today`
  - "Ignora Soglia" se monitoraggio passato con soglia sforata
- Deloitte: link "Visualizza Eccezione" SOLO se monitoraggio gia gestito (riga 850)

**Root cause**: `mapEventToRow` in `expired-events.component.ts` + `expiring-events.component.ts` mette stringa raw "UPDATE" come placeholder.

**Plan fix**:
1. Read pattern `buildActionCell` in `practice-detail.component.ts` (T-019, schedule tab) → usa `DynamicComponentModel<ButtonComponent>` con `inputs` callback
2. Refactor `mapEventToRow` in expired + expiring component → produrre stessa `DynamicComponentModel` invece di stringa raw
3. Importare `MonitoringExceptionDialogModule` nei due moduli + `@ViewChild('exceptionDialog') exceptionDialog: MonitoringExceptionDialogComponent`
4. Callback button: invocare `this.exceptionDialog.openForEvent(event, mode)` dove `mode` e' `'skip'|'ignore'|'view'` derivato da role + event state
5. Role-gating: `authService.isUserConsultant() ? 'view' : (event.hasException ? 'view' : conditionalMode(event))`

**File**:
- FE: `expired-events.component.ts`, `expiring-events.component.ts`, rispettivi `.html` (aggiunta `<app-monitoring-exception-dialog #exceptionDialog>`), rispettivi `.module.ts` (import `MonitoringExceptionDialogModule`)
- Pattern reference: `practice-detail.component.ts` metodi `buildActionCell` + `onSkipMonitoring`/`onIgnoreThreshold`/`onViewException`

**Test**:
- Unit aggiornati per cella Azione = button (non string)
- Smoke E2E:
  - ISP `/user/monitoring/expired-events` → cliccare "Salta Monitoraggio" su M1C1E4 → modal eccezione → invia → riga aggiornata
  - Deloitte `/consultant/monitoring/expired-events` → riga con `hasException=1` → "Visualizza Eccezione" → modal read-only

---

### Cluster G — Smoke finding #19

#### Bug #19 (NEW) — `documentInfo` stale dopo GenAI extraction in-page
**Severity**: MEDIUM | **Origine**: smoke E2E sessione 3 (2026-05-29)

**Comportamento osservato**: dopo click "Assegnazione con GenAI" nella page CC, l'extraction completa lato BE (`documentInfo.status` passa `DA_ASSEGNARE → GENAI_DA_VALIDARE`), ma il componente `MonitoringCcAssignmentPageComponent` carica `documentInfo` solo all'init. Il getter `showRejectButton` (richiede `documentInfo?.status === GENAI_DA_VALIDARE`) ritorna false → button "Ritorna a documentazione incompleta" non visibile finche l'utente non fa reload manuale.

**Plan fix**:
1. Read `monitoring-cc-assignment-page.component.ts:425` (polling `startGenAi` → `getGenAiResult`)
2. Dopo `extraction.status === MonitoringGenAiExtractionStatusEnum.READY_FOR_VALIDATION`, chiamare anche `this.loadDocumentInfo()` (metodo che probabilmente gia esiste — verificare)
3. Se `loadDocumentInfo` non esiste come metodo separato, estrarre dal bootstrap iniziale
4. Smoke: ripetere flow Reject sessione 3 senza reload → button pi-flag visibile immediatamente post-extraction

**File**: `monitoring-cc-assignment-page.component.ts`

**Test**:
- Unit: mock extraction READY → expect `loadDocumentInfo` chiamato
- Smoke E2E: doc DA_ASSEGNARE → page CC → GenAI → no reload → click 🚩 → modal rifiuto

---

## Ordine di esecuzione raccomandato (1 sessione 4-6h)

```
1. [30min] Cluster C - Enum/labels (bug #9 mock GenAI prima, sblocca smoke pulito; poi #8 Excel; poi #4 endpoint covenant-types)
2. [15min] Cluster G - Bug #19 documentInfo refresh (one-liner)
3. [30min] Cluster E - Bug #5 Variabile dropdown
4. [1h]    Cluster D - Mailing list UX (bug #11, #12, #14, #15 — tutti FE mailing-list-detail)
5. [1-1.5h] Cluster F - Bug #17 cella Azione expired/expiring (refactor con riferimento T-019)
6. [30min-1h] Cluster B - Bug #6 PUT covenant regenerate events
7. [1-2h] Cluster A - Bug #1 storage eccezione + bug #3 Caricato da
```

**Ragionamento**:
- Cluster C prima perche' bug #9 sblocca smoke realistico per testare gli altri fix
- Cluster G fast win (1 line)
- Cluster D batch UX nello stesso file (mailing-list-detail)
- Cluster F refactor medio ma isolato
- Cluster A in coda perche' richiede possibile cambio schema/storage (rischio piu' alto)

**Commit policy**: 1 commit atomico per bug con messaggio `fix(monitoring): <one-line> (bug #N BR sez X.Y.Z)`. NO Co-Author, NO --no-verify.

---

## Workflow per ogni bug (rigoroso)

1. **Read** del file impattato + contesto (component+service+test+i18n+BR section)
2. **Plan minimal** (no over-engineering, fedele al BR)
3. **Fix** + scrivere/aggiornare unit test
4. **Compile**: `mvn compile` BE (se BE) / `npx tsc --noEmit` FE (se FE)
5. **Unit test**: `./mvnw test -Dtest='Monitoring*'` BE / `npx ng test --watch=false --include='<spec>'` FE
6. **Smoke manuale** browser (Playwright MCP se UX-critical)
7. **Commit atomico** con messaggio descrittivo
8. **Update bug list** PROGRESS.md con `Bug #N CLOSED commit <hash>`

A fine sessione: 1 commit `deloitte-profiles` con PROGRESS + nuovo HANDOFF + push BE+FE con conferma utente.

---

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD allineati
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3 && git status --short
# Atteso BE HEAD: 5f22718 (allineato origin)

cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3 && git status --short
# Atteso FE HEAD: 08f8025 (allineato origin)

# Pull eventuali nuovi commit upstream
cd C:/Users/davmelis/Documents/Github/ba-back-end && git pull
cd C:/Users/davmelis/Documents/Github/ba-web && git pull

# Sanity compile
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw clean compile -DskipTests -q
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck

# BE+FE up
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw spring-boot:run -DskipTests -Dspring-boot.run.profiles=local &
cd C:/Users/davmelis/Documents/Github/ba-web && npm run start &
curl -s http://localhost:8091/actuator/health
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200

# Stato fixture M1 + 10 documenti smoke sessione 3
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT id, event_id, status, reject_reason, file_name FROM mp_monitoring_document WHERE practice_id=8288 ORDER BY id;"
```

---

## Utenti DB locale (invariato)

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`)
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`)
- Login richiede TOTP via Google/Microsoft Authenticator

---

## Riferimenti

- **Handoff predecessore**: `handoff/HANDOFF_2026-05-29_bug10_CLOSED_smoke_full.md` (bug #10 chiuso end-to-end)
- **Bug list dettagliata**: `handoff/HANDOFF_2026-05-28_bugfix_round_2_done.md` (lista 18 bug originali con fix suggeriti)
- **PLAN.md**: T-024 al 95% post bug #10 closed
- **BR completo**: `requirements/BR_Monitoraggio_V6.md` 1376 righe
- **Mockup**: `requirements/Mockup_Monitoraggio_v10.md` (slide refs per UI/UX bug)
- **Riferimenti BR per cluster**:
  - Cluster A: 4.2.6 (eccezioni), 3.2.3 + 4.2.2 (storico documenti)
  - Cluster B: 1.1 (scadenziere), 4.2.5 (modifica covenant)
  - Cluster C: 4.2.5 (dropdown tipologia), 8.4 (export)
  - Cluster D: 6.3 (mailing list workflow + popup)
  - Cluster E: 1.1 + 4.2.5 (Covenant Variabile SI/NO dropdown)
  - Cluster F: 8.4.3 / 8.5 (cella Azione expired/expiring), 4.2.6 + 3.4 (modal eccezioni pattern)
  - Cluster G: 4.2.3 (page-based UX bug #10, follow-up smoke)

---

## Esito atteso sessione

- 13 bug aperti → 0 aperti (eccetto #7 by-design)
- 5+13 = 18 fixati cumulativi (su 18 totali, escluso #7 e #13 rimosso)
- T-024 → 100% post bug-fix round
- Prossima sessione (4): retest E2E Playwright completo STEP 1-10 + UAT funzionale ISP + code review finale → archive T-024 → close milestone monitoring
