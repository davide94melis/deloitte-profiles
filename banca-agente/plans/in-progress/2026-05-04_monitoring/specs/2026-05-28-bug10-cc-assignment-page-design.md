# Design Spec — Bug #10: Refactor Compliance Certificate Assignment to Page-Based UX (BR 4.2.3)

**Data**: 2026-05-28
**Autore sessione**: Davide Melis + AI pair
**Riferimenti BR**: `requirements/BR_Monitoraggio_V6.md` sez. 4.2.3 (righe 652-744) + sez. 4.2.4 (righe 745-748)
**Stima**: 5-6h spalmate su 5 commit (BE + FE)
**Branch**: `feature/monitoring-integration-uat`
**Repo BE**: `C:/Users/davmelis/Documents/Github/ba-back-end` @ `c4efb8e`
**Repo FE**: `C:/Users/davmelis/Documents/Github/ba-web` @ `1d27047`

---

## 1. Contesto e problema

### Cosa c'è oggi (non BR-conforme)

- **`CcAssignmentComponent`** (`practice-detail/cc-assignment/`) — wizard 4-step dentro dialog PrimeNG. Logica: 1 covenant per volta, fa `assignEventValue` + `confirmDocument` in cascata. **NON copre**: niente Aggiungi Covenant multipli, niente scadenziere filtered ±15gg, niente toggle Modifica/Conferma, niente Concludi separato.
- **`MonitoringGenAiExtractionComponent`** (`practice-detail/monitoring-genai-extraction/`) — dialog GenAI key-value form. **NON copre**: niente N selettori, niente preview documento, niente bad flow modal con motivo.
- **State machine documento**: `PATCH /documents/{id}/confirm` salta `IN_ATTESA_CONFERMA → COMPLETATO` direttamente. **BR richiede stato intermedio** `DA_ASSEGNARE` (post-validate, pre-Concludi).
- **Schedule filter**: `GET /practices/{id}/schedule` non supporta filtro per range data ±15gg né per stati eventi.

### Cosa vuole il BR (sez 4.2.3-4.2.4)

- **Page dedicata** con preview documento, box selezione mode (GenAI/Manuale), N selettori covenant, scadenziere filtered inline, toggle Conferma/Modifica per slot, "Concludi Assegnazione" finale.
- **Bad flow** GenAI con modal motivo rifiuto + notifica ISP.
- **View mode** (BR 4.2.4) per documenti già `COMPLETATO`.

---

## 2. Scope decisioni chiave (consolidate brainstorming)

| Decisione | Scelta |
|---|---|
| Routing | Sub-route di practice-detail: `monitoring/practice/:id/cc/:documentId[?mode=view]` |
| Scope BE | BE esteso: nuovo endpoint `complete-assignment` + estensione `/schedule` con filtri |
| Component split | Moderato: 1 page + 3 sub (slot, schedule-table, reject-dialog) |
| Persist strategy | Batch al `Concludi` (no persist per Conferma di singolo slot) |
| State management | RxJS + service + destroy$ + local fields (no NgRx, no signals) |
| View mode N-covenants | Scope cut: supporto solo 1 covenant assignment legacy (multi-covenant = bug separato) |
| Eliminazione vecchi componenti | Commit dedicato post-smoke verify (rollback safety) |

---

## 3. Architettura e routing

```
monitoring/practice/:id              MonitoringPracticeDetailComponent (esistente)
├── covenant/:covenantId/edit        MonitoringCovenantEditComponent (esistente)
└── cc/:documentId                   MonitoringCcAssignmentPageComponent (NUOVO)
                                     QueryParam opzionale: ?mode=view (BR 4.2.4)
```

**Module structure** (dentro `MonitoringPracticeDetailModule`):
```
practice-detail/cc-assignment-page/
├── monitoring-cc-assignment-page.component.{ts,html,scss,spec.ts}
└── components/
    ├── monitoring-cc-covenant-slot.component.{ts,html,scss,spec.ts}
    ├── monitoring-cc-schedule-table.component.{ts,html,scss,spec.ts}
    └── monitoring-cc-reject-dialog.component.{ts,html,scss,spec.ts}
```

**Guards**: nessun guard custom, eredita auth role da `MonitoringPracticeDetailComponent`. Aggiungere `CanDeactivateGuard` standard per warning su modifiche non salvate.

**Trigger navigazione**: `MonitoringDocumentHistoryComponent` (oggi ha Conferma/Reject/StartGenAi/ModifyGenAi) aggiunge:
- Bottone "Assegna Monitoraggi" (visibile se `status === DA_ASSEGNARE`) → `router.navigate(['./cc', documentId], {relativeTo: route})`
- Bottone "Visualizza Assegnazione" (visibile se `status === COMPLETATO`) → stesso navigate + queryParam `?mode=view`

Il bottone "Assegna Monitoraggi" in `practice-detail.component.html:191-200` viene **rimosso** insieme al `@ViewChild ccAssignment` in `practice-detail.component.ts`.

---

## 4. Component breakdown

### 4.1 `MonitoringCcAssignmentPageComponent` (page entry)

**Route params**: `:id` (practiceId), `:documentId` + queryParam `?mode=view`.

**State locale**:
```typescript
documentInfo: MonitoringDocumentModel | null = null;
practiceCovenants: MonitoringCovenantModel[] = [];
mode: 'CHOOSE' | 'GENAI' | 'MANUAL' | 'VIEW' = 'CHOOSE';
slots: CcSlotState[] = [];
extractionLoading = false;
concludeInFlight = false;
rejectModalOpen = false;
private readonly destroy$ = new Subject<void>();

get canConcludi(): boolean {
  return this.slots.length > 0 && this.slots.every(s => s.confirmed);
}
get showRejectButton(): boolean {
  return this.mode === 'GENAI'
      && this.documentInfo?.status === MonitoringDocumentStateEnum.GENAI_DA_VALIDARE;
}
```

**Layout HTML (BR 4.2.3 righe 660-662)**:
```
[Header: titolo + breadcrumb]
[Preview documento CC]   <object data="..../download" type="application/pdf">
[Box selezione mode: GenAI | Manuale]
[Bottone "Indietro"]                                        → router.back()
[Bottone "Ritorna a documentazione incompleta"] (GenAI only)→ open RejectDialog
---
[Slots area, rendering per mode]
  CHOOSE  → vuoto
  MANUAL  → CcCovenantSlot[] + bottone "Aggiungi Covenant"
  GENAI   → spinner extraction → N CcCovenantSlot prefilled
  VIEW    → CcCovenantSlot[] readonly
---
[Divider + "Aggiungi Covenant #n" + "Concludi Assegnazione"]
  Concludi enabled solo se canConcludi (BR 716)
```

### 4.2 `MonitoringCcCovenantSlotComponent`

**@Input**: `mode: 'manual'|'genai'|'view'`, `practiceCovenants: MonitoringCovenantModel[]`, `slotState: CcSlotState`, `slotIndex: number`.
**@Output**: `confirm`, `modify`, `remove`, `stateChange`.

**Phase state machine (BR-driven)**:
| Phase | Trigger | UI |
|---|---|---|
| `EMPTY` | Aggiungi Covenant click | Solo riga + icona cestino (BR 669) |
| `FILLING` | Slot creato in manual | 3 box (tipologia, valore, data) + Associa/Annulla (BR 671-679) |
| `SCHEDULE_VISIBLE` | Post-Associa | Scadenziere filtered 9 colonne (BR 680-698) |
| `EVENT_SELECTED` | Click Assegna su riga | Compare "Conferma Assegnazione" (BR 708) |
| `CONFIRMED` | Click Conferma | Scadenziere collapsed a 1 riga, action "Modifica Assegnazione" (BR 711-712) |
| `GENAI_PREFILLED` | Mode GenAI iniziale | Selettori prefilled + "Modifica Dati" + "Conferma Assegnazione" (BR 733) |

### 4.3 `MonitoringCcScheduleTableComponent`

**@Input**: `events: MonitoringEventModel[]`, `assignedEventId: number | null`, `readonly: boolean`.
**@Output**: `assign(eventId: number)`, `unassign()`.

**9 colonne** (BR 682-698): ID Evento, Data Riferimento, Scadenza, Giorni al prossimo, Data avvenuto, Valore, Status, CC Link Download, Azioni (Assegna/Rimuovi).

Il filtro è server-side (vedi sez. 5). Il component renderizza ciò che riceve.

### 4.4 `MonitoringCcRejectDialogComponent`

**@Input**: `open: boolean`, `documentId: number`.
**@Output**: `rejected(reason: string, notes?: string)`, `cancelled()`.

Dialog PrimeNG (BR 729):
- Dropdown motivo (obbligatorio, hardcoded i18n: "Documento incompleto", "Formato non corretto", "Firma mancante", "Altro")
- Textarea note (facoltativa)
- Bottoni "Conferma Rifiuto" + "Annulla"

### 4.5 Type interno `CcSlotState`

```typescript
interface CcSlotState {
  uid: string;                                  // uuid locale per *ngFor track
  index: number;                                // 1-based "Covenant #n"
  phase: 'EMPTY' | 'FILLING' | 'SCHEDULE_VISIBLE' | 'EVENT_SELECTED' | 'CONFIRMED' | 'GENAI_PREFILLED';
  selectedCovenantId: number | null;
  monitoringValue: number | null;
  referenceDate: string | null;                 // ISO yyyy-MM-dd
  scheduleEvents: MonitoringEventModel[];
  assignedEventId: number | null;
  confirmed: boolean;
}
```

---

## 5. Backend — state machine, endpoint, query

### 5.1 State machine documento (BR-compliant)

```
IN_ATTESA_CONFERMA ──validate──> DA_ASSEGNARE ──complete-assignment──> COMPLETATO
IN_ATTESA_CONFERMA ──genai──> GENAI_IN_CORSO ──> GENAI_DA_VALIDARE ──apply──> COMPLETATO
IN_ATTESA_CONFERMA ──reject──> IN_ATTESA_CONFERMA (con reject_reason)
GENAI_DA_VALIDARE  ──reject──> IN_ATTESA_CONFERMA (BR 728 bad flow GenAI)
```

### 5.2 Enum `MonitoringDocumentStateEnum.java`

```java
IN_ATTESA_CONFERMA("In Attesa Conferma"),
VERIFICA_IN_CORSO("Verifica in Corso"),
GENAI_IN_CORSO("GenAI in Corso"),
GENAI_DA_VALIDARE("GenAI da Validare"),
DA_ASSEGNARE("Da Assegnare"),     // NUOVO
COMPLETATO("Completato");
```

### 5.3 Service `MonitoringDocumentService.java`

- `validateDocument(Long id)` — rename di `confirmDocument`. Setta `DA_ASSEGNARE`. Valida state == `IN_ATTESA_CONFERMA`.
- `completeAssignment(Long id)` — NUOVO. Valida state ∈ {`DA_ASSEGNARE`, `GENAI_DA_VALIDARE`}. Setta `COMPLETATO`.
- `rejectDocument(Long id, String reason)` — accetta anche state `GENAI_DA_VALIDARE` (BR 728 bad flow).

### 5.4 Controller `MonitoringDocumentController.java`

- **Rename**: `PATCH /documents/{id}/confirm` → `PATCH /documents/{id}/validate`. No alias retrocompat (FE è unico consumer, aggiornato in stessa sessione).
- **NUOVO**: `POST /documents/{id}/complete-assignment` body `MonitoringCcCompleteAssignmentRequestDTO`.

### 5.5 DTO `MonitoringCcCompleteAssignmentRequestDTO.java` (NUOVO, record)

```java
public record MonitoringCcCompleteAssignmentRequestDTO(
    @NotEmpty List<CovenantAssignment> assignments
) {
    public record CovenantAssignment(
        @NotNull Long covenantId,
        @NotNull Long eventId,
        @NotNull BigDecimal monitoringValue,
        @NotNull LocalDate actualDate,
        @NotNull LocalDate referenceDate
    ) {}
}
```

### 5.6 Logic `completeAssignment` (atomic transactional)

```
@Transactional
completeAssignment(documentId, request):
  for each assignment a in request.assignments:
    eventService.updateValue(a.eventId, a.monitoringValue, a.actualDate)   // bug #18 fix triggers recalc
    if a.referenceDate != event.referenceDate:
      eventService.updateDates(a.eventId, a.referenceDate, null)            // null = no change expirationDate
  documentService.completeAssignment(documentId)                            // status → COMPLETATO
  return updated DocumentDTO
```

### 5.7 Schedule extension `findScheduleByPractice`

**Controller**:
```java
@GetMapping("/practices/{practiceId}/schedule")
ApiResponse<Page<MonitoringEventDTO>> findScheduleByPractice(
    @PathVariable Long practiceId,
    @RequestParam(required = false) String covenantType,
    @RequestParam(required = false) LocalDate referenceDate,
    @RequestParam(defaultValue = "15") Integer rangeDays,
    @RequestParam(required = false) List<MonitoringEventStateEnum> statuses,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size)
```

**Repository JPQL** (BR 700-706):
```jpql
SELECT e FROM MonitoringEvent e
WHERE e.covenant.practice.id = :practiceId
  AND (:covenantType IS NULL OR e.covenant.covenantType = :covenantType)
  AND (
    (:statuses IS NULL OR e.status IN :statuses)
    OR (:referenceDate IS NOT NULL
        AND e.referenceDate BETWEEN :startDate AND :endDate)
  )
ORDER BY e.expirationDate ASC
```

con `startDate = referenceDate.minusDays(rangeDays)`, `endDate = referenceDate.plusDays(rangeDays)`.

### 5.8 GenAI flow tweak `MonitoringGenAiService.java`

Linea 139: cambia `documentService.confirmDocument(documentId)` → `documentService.completeAssignment(documentId)` (semantica corretta — `applyExtraction` chiude direttamente perché GenAI flow non passa per `DA_ASSEGNARE`).

### 5.9 Callers FE da aggiornare

| Caller | Change |
|---|---|
| `monitoring.service.ts:165` `confirmDocument()` | rename + cambio path a `/validate` |
| `monitoring.service.ts` | nuovo `completeAssignment(documentId, assignments)` |
| `monitoring.service.ts` | extend `getSchedule()` con 3 filtri aggiuntivi |
| `MonitoringDocumentHistoryComponent.onConfirm` | chiama `validateDocument`; render bottone "Assegna Monitoraggi" se status `DA_ASSEGNARE` |

### 5.10 Backward-compat

Documenti già `COMPLETATO` legacy → consultabili in `?mode=view` con scope cut (1 covenant). Nuovi documenti seguono `validate → DA_ASSEGNARE → complete-assignment`.

---

## 6. Data flow FE (sequence per scenario BR)

### 6.1 Bootstrap page (tutti i mode)

```
1. Route activated → ngOnInit
2. forkJoin([getDocumentById(documentId), getCovenantsByPractice(practiceId)])
3. Determine mode:
   - queryParam mode==='view' + status===COMPLETATO → VIEW + load existing assignments (scope cut)
   - status===GENAI_DA_VALIDARE → GENAI + getGenAiResult(documentId, wait=true)
   - status===DA_ASSEGNARE → CHOOSE
   - altro → redirect back con errore
4. Preview render: <object data="${apiBaseUrl}/monitoring/documents/${documentId}/download" type="application/pdf">
   Fallback: link "Scarica documento" se preview KO
```

### 6.2 Flow Manual (BR 4.2.3 righe 652-716)

```
1. Click "Assegnazione Manuale" → mode=MANUAL + render "Aggiungi Covenant"
2. Click "Aggiungi Covenant" → push slot {phase:'FILLING'}
3. User compila tipologia + valore + dataRif
4. Click "Associa Covenant" (enabled se 3 box compilati)
   → getSchedule(practiceId, {covenantType, referenceDate, rangeDays:15, statuses:[SCADUTO,FUORI_SOGLIA,IN_SCADENZA]})
   → slot.phase='SCHEDULE_VISIBLE' + render CcScheduleTable inline
5. Click "Annulla" (BR 679) → slot.phase='FILLING' + 3 box vuoti (slot non eliminato)
6. Click "Assegna" su riga → slot.assignedEventId + phase='EVENT_SELECTED'
7. Click "Conferma Assegnazione" → slot.phase='CONFIRMED' + collapse scheduleEvents a [assignedEvent]
8. Click "Modifica Assegnazione" → slot.phase='SCHEDULE_VISIBLE' + restore filtered list
9. Click "Aggiungi Covenant #n" → ripete da step 2
10. Click "Concludi Assegnazione" (enabled se canConcludi)
    → completeAssignment(documentId, {assignments: slots.map(s => ({
        covenantId: s.selectedCovenantId,
        eventId: s.assignedEventId,
        monitoringValue: s.monitoringValue,
        actualDate: s.referenceDate,    // BR 675 prevede solo "Data di Riferimento" come input utente;
                                        // actualDate persiste sullo stesso valore per semantica "monitoraggio effettuato in data X"
        referenceDate: s.referenceDate
      }))})
    → success: status dialog + router.navigate(['../../'], {relativeTo: route})
```

### 6.3 Flow GenAI (BR 4.2.3 righe 718-744)

```
1. Mode=GENAI auto da status GENAI_DA_VALIDARE
2. getGenAiResult(documentId, wait=true) → DeferredResult 30s
3. Switch response.status:
   - PENDING/IN_PROGRESS → spinner (BR 731)
   - READY_FOR_VALIDATION → buildSlotsFromExtraction(response.extractedFields)
   - FAILED → banner + bottone "Riprova GenAI" → startGenAiExtraction
4. Per ogni "blocco" estratto → push slot {phase:'GENAI_PREFILLED', ...prefilled}
5. Click "Modifica Dati" → slot.phase='SCHEDULE_VISIBLE' (BR 735, stesso UX manual)
6. Click "Conferma Assegnazione" → phase='CONFIRMED' (BR 737)
7. Click "Modifica Assegnazione" → torna SCHEDULE_VISIBLE (BR 739)
8. Click "Concludi" (BR 743)
   → applyGenAiExtraction(documentId, {covenantUpdates, eventUpdates})
   → success: router.navigate back
```

### 6.4 Bad flow Reject (BR 728)

```
1. Solo mode=GENAI: render bottone "Ritorna a documentazione incompleta"
2. Click → CcRejectDialog.open=true
3. User: dropdown motivo + textarea note → "Conferma Rifiuto"
   → rejectDocument(documentId, `${reason}: ${notes}`)
   → success: status dialog "Notifica inviata a ISP" + router.navigate back
4. "Annulla" → dialog close
```

### 6.5 View mode (BR 4.2.4, scope cut)

```
1. queryParam ?mode=view + status===COMPLETATO
2. Reverse-lookup assignment via documentInfo.eventId (legacy 1 covenant)
3. Build slots con phase='CONFIRMED' + scheduleEvents=[assignedEvent]
4. Click "Modifica Assegnazione" → riapre flusso normale
5. Concludi → re-invoca completeAssignment (idempotente)
```

**Out of scope** (bug separato): N-covenants assignment lookup → richiede nuova tabella `mp_monitoring_document_assignment` o array `eventIds` in `MonitoringDocument`. Documentato come limitazione corrente.

---

## 7. State management + error handling

### 7.1 State management

- **No NgRx, no signals, no global BehaviorSubject** — coerente con pattern modulo (`practice-detail.component.ts:64,178`, `expired-events.component.ts:25`).
- **Local fields** + `destroy$` Subject + `takeUntil(destroy$)`.
- **Immutability slot[]**: ogni `stateChange` rimpiazza slot via `[...slots.slice(0,i), updated, ...slots.slice(i+1)]`.
- **RouteReuseStrategy default**: page reloaded ad ogni navigation, no cache.

### 7.2 Error handling

| Categoria | Esempio | Gestione |
|---|---|---|
| Network 5xx | BE down, GenAI timeout | `statusDialogService.setStatus({status:'KO',title:'Errore',message:i18n('shared.networkError'),closable:true})`. No retry auto. |
| Validation 400 | completeAssignment con stato doc errato | `err.error.errorMessage` in dialog. Slot state preservato. |
| GenAI FAILED | extraction status=FAILED | Banner inline + bottone "Riprova GenAI" → startGenAiExtraction + reload |
| Schedule vuoto | nessun evento matcha filtri | Empty state: i18n("monitoring.cc.schedule.empty") + suggerimento modifica data |
| Concorrenza | doc già processato da altro utente | 400 → dialog "Documento già processato, ricarica" + router.navigate back |
| PDF preview fail | binario corrotto o non-PDF | Fallback link "Scarica documento" + warning text |
| Loss of work guard | navigate via via con slot in stato dirty | `CanDeactivateGuard` con `confirm("Hai modifiche non salvate. Uscire?")` |

**Pattern RxJS standard**:
```typescript
this.monitoringService.completeAssignment(docId, payload)
  .pipe(takeUntil(this.destroy$), finalize(() => this.concludeInFlight = false))
  .subscribe({
    next: () => this.onCompleteSuccess(),
    error: (err) => this.handleCompleteError(err)
  });
```

**No silent swallow**: ogni `error:` branch chiama almeno `statusDialogService.setStatus()`.

### 7.3 Loading states

| Trigger | UI |
|---|---|
| Bootstrap forkJoin | `SpinnerComponent.showLoader()` + `hideLoader()` su `finalize` |
| GenAI extraction in corso | Inline spinner (BR 731) "Estrazione GenAI in corso..." |
| Click "Associa Covenant" | Spinner inline sotto bottone |
| Click "Concludi Assegnazione" | `concludeInFlight=true` → bottoni disabled + spinner |
| Reject submit | Spinner sul bottone "Conferma Rifiuto" |

### 7.4 i18n keys (assets/i18n/it.json + en.json)

```
monitoring.cc.page.title                "Assegnazione Compliance Certificate"
monitoring.cc.mode.genai                "Assegnazione con GenAI"
monitoring.cc.mode.manual               "Assegnazione Manuale"
monitoring.cc.back                      "Indietro"
monitoring.cc.reject.button             "Ritorna a documentazione incompleta"
monitoring.cc.reject.modal.title        "Rifiuto Documento"
monitoring.cc.reject.modal.reasonLabel  "Motivo del rifiuto *"
monitoring.cc.reject.reasons.incomplete "Documento incompleto"
monitoring.cc.reject.reasons.format     "Formato non corretto"
monitoring.cc.reject.reasons.signature  "Firma mancante"
monitoring.cc.reject.reasons.other      "Altro"
monitoring.cc.reject.modal.notes        "Note (facoltative)"
monitoring.cc.reject.modal.confirm      "Conferma Rifiuto"
monitoring.cc.reject.successIsp         "Rifiuto inviato a ISP con notifica"
monitoring.cc.addCovenant               "Aggiungi Covenant"
monitoring.cc.addCovenantN              "Aggiungi Covenant #{n}"
monitoring.cc.concludi                  "Concludi Assegnazione"
monitoring.cc.slot.typology             "Seleziona Tipologia Covenant"
monitoring.cc.slot.value                "Valore Monitoraggio"
monitoring.cc.slot.referenceDate        "Data di Riferimento"
monitoring.cc.slot.associaCovenant      "Associa Covenant"
monitoring.cc.slot.annulla              "Annulla"
monitoring.cc.slot.confermaAssegnazione "Conferma Assegnazione"
monitoring.cc.slot.modificaAssegnazione "Modifica Assegnazione"
monitoring.cc.slot.modificaDati         "Modifica Dati"
monitoring.cc.schedule.empty            "Nessun evento corrisponde ai filtri"
monitoring.cc.schedule.action.assegna   "Assegna"
monitoring.cc.schedule.action.rimuovi   "Rimuovi"
monitoring.cc.preview.unavailable       "Preview non disponibile, scarica il documento"
monitoring.cc.canDeactivate.confirm     "Hai modifiche non salvate. Uscire comunque?"
monitoring.cc.genai.extracting          "Estrazione GenAI in corso..."
monitoring.cc.genai.failed              "Estrazione GenAI fallita"
monitoring.cc.genai.retry               "Riprova Estrazione"
monitoring.cc.complete.success          "Assegnazione completata"
monitoring.cc.complete.error            "Errore durante il completamento dell'assegnazione"
```

---

## 8. Testing strategy

### 8.1 Unit test FE (Karma + Jasmine, pattern modulo)

| Spec | Casi minimi |
|---|---|
| `monitoring-cc-assignment-page.component.spec.ts` | bootstrap (forkJoin success/fail), mode detection per status, canConcludi con N slot, completeAssignment success+error, canDeactivate guard, navigate back, reject success |
| `monitoring-cc-covenant-slot.component.spec.ts` | phase transitions EMPTY→FILLING→SCHEDULE_VISIBLE→EVENT_SELECTED→CONFIRMED→back to SCHEDULE_VISIBLE on Modifica, 3-box validation, stateChange emit, GENAI_PREFILLED initial |
| `monitoring-cc-schedule-table.component.spec.ts` | render 9 colonne, action emit, readonly mode no-action, empty state |
| `monitoring-cc-reject-dialog.component.spec.ts` | dropdown obbligatorio, submit emit con motivo+notes, cancel emit |

Target coverage ≥80% per file.

### 8.2 Unit test BE (JUnit 5 + AssertJ + Mockito)

| Test class | Casi |
|---|---|
| `MonitoringDocumentServiceTest` | validateDocument IN_ATTESA→DA_ASSEGNARE, completeAssignment from DA_ASSEGNARE e da GENAI_DA_VALIDARE, reject da GENAI_DA_VALIDARE |
| `MonitoringEventServiceTest` | findScheduleByPractice con filtri combinati (statuses, referenceDate+rangeDays, covenantType) |
| `MonitoringDocumentControllerTest` | nuovo endpoint complete-assignment: body validation, atomic batch, transactional rollback |
| `MonitoringCovenantControllerTest` | schedule endpoint con nuovi query params + paging |
| `MonitoringGenAiServiceTest` | applyExtraction chiama completeAssignment (assert state finale) |

Pattern: `@ExtendWith(MockitoExtension.class)` + `@Mock` + `when().thenReturn()` + AssertJ `assertThat()`.

Regression: 145+ test monitoring esistenti devono restare GREEN dopo refactor.

### 8.3 Smoke E2E manuale (Playwright)

Eseguito a fine implementazione, prima del commit FE finale. Utente Deloitte CONSULTANT (`davide94x@gmail.com`), DB M1 fixture.

**Step Manual**:
1. Upload documento (via STEP 8 COVNO o mailing list).
2. Click "Conferma" doc → status `DA_ASSEGNARE`.
3. Click "Assegna Monitoraggi" → naviga `/cc/:documentId`.
4. Verify preview PDF.
5. "Assegnazione Manuale" → "Aggiungi Covenant".
6. Compila 3 box → "Associa Covenant".
7. Verify scadenziere mostra eventi filtered (status target OR ref date ±15gg).
8. "Assegna" su riga → "Conferma Assegnazione".
9. Verify scadenziere collapse + action "Modifica Assegnazione".
10. "Aggiungi Covenant #2" → ripeti.
11. "Concludi Assegnazione" → success dialog + navigate back + status `COMPLETATO`.
12. Verify dashboard donut M1 ricalcolato.

**Step GenAI**:
13. Doc → "Valida con GenAI" → wait extraction.
14. Verify spinner → N selettori prefilled.
15. "Modifica Dati" → SCHEDULE_VISIBLE → "Conferma Assegnazione" → "Modifica Assegnazione".
16. "Concludi Assegnazione" → success + navigate back.

**Step Bad Flow**:
17. Doc GENAI_DA_VALIDARE → "Ritorna a documentazione incompleta".
18. Motivo + note → "Conferma Rifiuto".
19. Verify status doc → `IN_ATTESA_CONFERMA` + reject_reason persistito.

**Step View (scope cut)**:
20. Doc COMPLETATO → "Visualizza Assegnazione" → `?mode=view`.
21. Verify readonly mode con singolo slot.
22. "Modifica Assegnazione" → riapre flusso → "Concludi" → success.

---

## 9. Migration order (5 commit, no breaking window)

```
COMMIT 1 — BE state machine + endpoints (build green, no FE break)
  - Add enum DA_ASSEGNARE
  - Add validateDocument() + completeAssignment() service methods (NON usati ancora)
  - Add MonitoringCcCompleteAssignmentRequestDTO
  - Add POST /documents/{id}/complete-assignment endpoint
  - Extend GET /practices/{id}/schedule con filtri (backward-compat default)
  - Add/update test → 145+ green
  - confirmDocument() RIMANE invariato (legacy ancora funziona)

COMMIT 2 — BE switch confirmDocument behavior + rename
  - Rename PATCH /confirm → PATCH /validate
  - confirmDocument body → validateDocument (DA_ASSEGNARE invece di COMPLETATO)
  - MonitoringGenAiService.applyExtraction switch a completeAssignment()
  - Update test esistenti per nuovo behavior
  - BREAKING per FE confirmDocument callers — risolto in commit 3

COMMIT 3 — FE service signature + new page component
  - monitoring.service.ts: rename + add completeAssignment + extend getSchedule
  - Add monitoring-cc-assignment-page module + 4 component + spec files
  - Add nuova route practice/:id/cc/:documentId
  - i18n keys (it+en)
  - tsc + unit test green

COMMIT 4 — FE wire-up document-history + remove old dialog trigger
  - MonitoringDocumentHistoryComponent.onConfirm → validateDocument + render "Assegna Monitoraggi" se DA_ASSEGNARE
  - MonitoringDocumentHistoryComponent → "Visualizza Assegnazione" se COMPLETATO
  - practice-detail.component.html: remove bottone "Assegna Monitoraggi" (righe 191-200)
  - practice-detail.component.ts: remove @ViewChild ccAssignment + handler
  - Smoke E2E manual

COMMIT 5 — Cleanup vecchi componenti (post-smoke verify)
  - Delete cc-assignment/ + monitoring-genai-extraction/
  - Remove module imports + spec references
  - tsc + npm test green
```

**Rollback safety**: smoke fail al commit 4 → `git revert COMMIT 4` ripristina trigger vecchio (vecchi componenti ancora presenti, deletati solo in commit 5).

### File touch summary

| Repo | Tipo | Path | Action |
|---|---|---|---|
| BE | enum | `MonitoringDocumentStateEnum.java` | EDIT (+1 value) |
| BE | service | `MonitoringDocumentService.java` | EDIT (rename + new method) |
| BE | controller | `MonitoringDocumentController.java` | EDIT (rename + new endpoint) |
| BE | service | `MonitoringEventService.java` | EDIT (extend findScheduleByPractice) |
| BE | controller | `MonitoringCovenantController.java` | EDIT (extend /schedule params) |
| BE | service | `MonitoringGenAiService.java` | EDIT (line 139) |
| BE | dto | `MonitoringCcCompleteAssignmentRequestDTO.java` | NEW (record) |
| BE | repository | `MonitoringEventRepository.java` | EDIT (extend query) |
| BE | test | 5 test classes | EDIT/NEW |
| FE | service | `monitoring.service.ts` | EDIT (rename + new + extend) |
| FE | model | `monitoring-cc-assignment.model.ts` | NEW |
| FE | enum | `monitoring-document-state.enum.ts` | EDIT (+1 value) |
| FE | component | `monitoring-cc-assignment-page.component.{ts,html,scss,spec.ts}` | NEW (4 file) |
| FE | component | `monitoring-cc-covenant-slot.component.{ts,html,scss,spec.ts}` | NEW (4 file) |
| FE | component | `monitoring-cc-schedule-table.component.{ts,html,scss,spec.ts}` | NEW (4 file) |
| FE | component | `monitoring-cc-reject-dialog.component.{ts,html,scss,spec.ts}` | NEW (4 file) |
| FE | module | `practice-detail.module.ts` | EDIT (declare + route) |
| FE | component | `practice-detail.component.{html,ts}` | EDIT (remove old trigger) |
| FE | component | `document-history.component.{html,ts}` | EDIT (nav buttons) |
| FE | i18n | `assets/i18n/it.json` + `en.json` | EDIT |
| FE | DELETE | `cc-assignment/*` + `monitoring-genai-extraction/*` | DELETE (commit 5) |

---

## 10. Constraints e principi non negoziabili

- **Fedeltà BR**: ogni decisione UX/data-flow tracciata con riga BR. Niente "miglioramenti" che divergono.
- **No silent failure**: ogni error branch chiama `statusDialogService.setStatus()`.
- **Immutability**: slot[] aggiornato via spread, mai `slot.field = x` su array indexed.
- **Pattern modulo**: RxJS + destroy$ + service injection, no NgRx/signals.
- **No Co-Author / no auto-push** (memoria utente Davide Melis).
- **80% coverage target** per FE + BE.
- **Atomic transactional** per `completeAssignment` BE.
- **No breaking window** durante migration (5 commit ordinati, commit 5 dedicato a cleanup post-verify).

---

## 11. Open questions / out of scope

1. **View mode N-covenants** (BR 4.2.4 multi-covenant assignment): scope cut. Richiede nuova tabella `mp_monitoring_document_assignment` o array `eventIds`. → Bug separato.
2. **REPORT_GENERATO vs COMPLETATO**: BR 743 GenAI dice "Report Generato"; BR 710 manual dice "Completato". Decisione pragmatica: usiamo solo `COMPLETATO` per entrambi (FE può mostrare label diversa). Drift minore documentato.
3. **GenAI mock N-blocchi**: il mock attuale (`AsyncMonitoringGenAiProcess`) restituisce un singolo blocco di campi. Per N-covenants veri serve estendere il mock — questa sessione gestisce N>=1 ma il mock resta single-block (limite documentato, smoke gira con N=1).

---

## 12. Riferimenti

- **BR**: `requirements/BR_Monitoraggio_V6.md` sez. 4.2.3 (652-744), 4.2.4 (745-748)
- **Handoff predecessor**: `handoff/HANDOFF_2026-05-28_bugfix_round_2_done.md`
- **Bug list aggiornata**: 14 bug aperti, bug #10 HIGH = questo design
- **Commit base**: BE `c4efb8e`, FE `1d27047`
- **PLAN.md**: T-024 al 80%, manca bug-fix round + retest + UAT + code review
