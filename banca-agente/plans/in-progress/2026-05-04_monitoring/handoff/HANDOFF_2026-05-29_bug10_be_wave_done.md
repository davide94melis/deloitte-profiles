# Handoff BUG #10 — BE WAVE DONE → FE WAVE — 2026-05-29

Sessione AI chiusa dopo BE wave del bug #10 (refactor CC Assignment page-based UX, BR 4.2.3). **Backend completamente pronto**: state machine documento BR-compliant, nuovo endpoint atomico `complete-assignment`, filtri `/schedule` con range ±15gg. 2 commit BE pushati. Prossima sessione = **FE wave** (Commit 3, 4, 5 del plan): nuovi 4 componenti page-based, wire-up document-history, cleanup vecchi componenti.

## Branch state

- BE `feature/monitoring-integration-uat` HEAD = **`c83a7c0`** (era `c4efb8e` — 2 nuovi commit pushati)
  - `21d5636` feat(monitoring): add DA_ASSEGNARE state + complete-assignment endpoint + schedule filters (bug #10 BR 4.2.3)
  - `c83a7c0` refactor(monitoring): replace confirmDocument with validateDocument + completeAssignment per BR 4.2.3 state machine
- FE `feature/monitoring-integration-uat` HEAD = **`1d27047`** (invariato — FE wave da fare)
- Profilo `deloitte-profiles` HEAD = aggiornato con spec, plan e questo handoff

## Cosa è stato fatto BE (wave 1)

### Commit 1 — `21d5636` (non-breaking additions, 11 file, +487/-17)

**Enum**:
- `MonitoringDocumentStateEnum`: aggiunto `DA_ASSEGNARE("Da Assegnare")` come stato intermedio post-validate, pre-complete-assignment.

**Service `MonitoringDocumentService`**:
- Nuovo `validateDocument(Long)`: valida state `IN_ATTESA_CONFERMA`, setta `DA_ASSEGNARE`.
- Nuovo `completeAssignment(Long)`: valida state ∈ {`DA_ASSEGNARE`, `GENAI_DA_VALIDARE`}, setta `COMPLETATO`.
- Nuovo overload `completeAssignment(Long, MonitoringCcCompleteAssignmentRequestDTO)`: atomic batch — chiama `eventService.updateValue` + `eventService.updateDates` per ogni assignment, poi `completeAssignment(Long)`.
- Iniettato `MonitoringEventService` come constructor param.

**Controller `MonitoringDocumentController`**:
- Nuovo endpoint `POST /monitoring/documents/{documentId}/complete-assignment` body `MonitoringCcCompleteAssignmentRequestDTO`.

**DTO `MonitoringCcCompleteAssignmentRequestDTO`** (record nuovo):
```java
record MonitoringCcCompleteAssignmentRequestDTO(
  @NotEmpty @Valid List<CovenantAssignment> assignments
) {
  record CovenantAssignment(
    @NotNull Long covenantId,
    @NotNull Long eventId,
    @NotNull BigDecimal monitoringValue,
    @NotNull LocalDate actualDate,
    @NotNull LocalDate referenceDate
  ) {}
}
```

**Repository `MonitoringEventRepository`**:
- Nuova query JPQL `findScheduleFiltered(practiceId, covenantType, statuses, referenceDate, startDate, endDate, pageable)`: ritorna eventi del covenant con stati nel filter OR data di riferimento nel range `[startDate, endDate]`.

**Service `MonitoringEventService.findScheduleByPractice`** estesa con overload:
- `findScheduleByPractice(Long, String, Pageable)` — backward-compat, delega a nuovo overload con filtri null.
- `findScheduleByPractice(Long, String, LocalDate referenceDate, Integer rangeDays, List<MonitoringEventStateEnum> statuses, Pageable)` — computa `[refDate-rangeDays, refDate+rangeDays]`, normalizza statuses vuoti a null, chiama `findScheduleFiltered`.

**Controller `MonitoringCovenantController.findScheduleByPractice`** estesa con query params:
- `?covenantType=ICR&referenceDate=2026-05-28&rangeDays=15&statuses=SCADUTO,FUORI_SOGLIA,IN_SCADENZA&page=0&size=20`
- `rangeDays` default 15. Tutti gli altri filtri opzionali.

**Tests aggiunti**:
- `MonitoringDocumentServiceTest` (NEW, 8 tests): ValidateDocument (3), CompleteAssignment (4), CompleteAssignmentBatch (1).
- `MonitoringDocumentControllerTest` (NEW, 1 test): complete-assignment endpoint delegation.
- `MonitoringCovenantControllerTest` (NEW, 2 tests): schedule con filtri + default rangeDays.
- `MonitoringEventServiceTest`: nuovo nested `FindScheduleFiltered` (2 tests) + aggiornati 5 test esistenti `FindScheduleByPractice` per mock pattern nuovo (`findScheduleFiltered` invece di `findByCovenantPracticeId`).

### Commit 2 — `c83a7c0` (breaking switch + rename, 4 file, +13/-24)

- `MonitoringDocumentService`: rimosso `confirmDocument(Long)`. Adesso il flusso passa per `validateDocument` + `completeAssignment`.
- `MonitoringDocumentController`: rename `PATCH /documents/{id}/confirm` → `PATCH /documents/{id}/validate`. Metodo + path coerenti.
- `MonitoringGenAiService.applyExtraction` riga 139: chiama `documentService.completeAssignment(documentId)` invece di `confirmDocument(documentId)`. Semantica corretta (BR 743 finalize post-GenAI).
- `MonitoringGenAiServiceTest`: 11 occorrenze `confirmDocument` → `completeAssignment` (9 stub + 2 verify).

**Stato test BE post-wave**: 187/187 PASS (era 145 + 23 nuovi monitoring = 168, ma full suite contiene anche non-monitoring → 187 totali).

## State machine documento finale (BR 4.2.3)

```
IN_ATTESA_CONFERMA ──PATCH /validate──> DA_ASSEGNARE ──POST /complete-assignment──> COMPLETATO
IN_ATTESA_CONFERMA ──genai/extract──> GENAI_IN_CORSO ──async──> GENAI_DA_VALIDARE ──POST /genai/apply──> COMPLETATO
IN_ATTESA_CONFERMA ──PATCH /reject──> IN_ATTESA_CONFERMA (con reject_reason)
GENAI_DA_VALIDARE  ──PATCH /reject──> IN_ATTESA_CONFERMA (BR 728 bad flow)
```

⚠️ **BREAKING per il FE corrente**: il FE chiama ancora `PATCH /documents/{id}/confirm` che NON esiste più. Tutti i bottoni "Conferma" oggi falliranno. Il fix è in Commit 3 del plan (Wave 2 FE). **Non avviare ambiente locale** senza prima aver fatto Commit 3 FE.

## Cosa fare in Wave 2 (FE)

### Commit 3 FE — Nuovi componenti (~2-3h)

Vedi sezione III del plan: `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-28-bug10-cc-assignment-page-plan.md`.

**File da modificare**:
- `src/app/service/monitoring.service.ts`:
  - Rename `confirmDocument(documentId)` → `validateDocument(documentId)` (path da `/confirm` a `/validate`)
  - Nuovo `completeAssignment(documentId, payload)` → `POST /complete-assignment`
  - Estendere `getSchedule(practiceId, opts)` con 3 nuovi params opzionali (`referenceDate`, `rangeDays`, `statuses`)
- `src/app/shared/enum/monitoring-document-state.enum.ts`: aggiungere `DA_ASSEGNARE = 'DA_ASSEGNARE'`
- `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts`:
  - Declare 4 nuovi componenti
  - Aggiungere route `{ path: 'cc/:documentId', component: MonitoringCcAssignmentPageComponent }`
- `assets/i18n/it.json` + `assets/i18n/en.json`: chiavi nuove (lista completa in spec §7.4)

**File da creare**:
- `src/app/shared/model/monitoring/monitoring-cc-assignment.model.ts` — `CcSlotState` + `CcCompleteAssignmentRequest`
- 4 nuovi componenti in `practice-detail/cc-assignment-page/`:
  - `monitoring-cc-assignment-page.component.{ts,html,scss,spec.ts}`
  - `components/monitoring-cc-covenant-slot.component.{ts,html,scss,spec.ts}`
  - `components/monitoring-cc-schedule-table.component.{ts,html,scss,spec.ts}`
  - `components/monitoring-cc-reject-dialog.component.{ts,html,scss,spec.ts}`

**Pattern obbligatori**:
- RxJS + `destroy$` Subject + `takeUntil(destroy$)` (no NgRx, no signals)
- Immutability slot[] via spread `[...slots.slice(0,i), updated, ...slots.slice(i+1)]`
- `SpinnerComponent.showLoader/hideLoader` per loading states
- `statusDialogService.setStatus` per ogni error branch (no silent swallow)
- `@ViewChild` per dialog reject + slot list

**Commit message**: `feat(monitoring): add CC assignment page-based UX (bug #10 BR 4.2.3)`

### Commit 4 FE — Wire-up document-history + remove old trigger (~30-45min)

- `MonitoringDocumentHistoryComponent.onConfirm`: chiama `validateDocument` (status diventa `DA_ASSEGNARE` invece di `COMPLETATO`).
- `MonitoringDocumentHistoryComponent`: render bottone "Assegna Monitoraggi" se `status === DA_ASSEGNARE` → `router.navigate(['./cc', documentId])`.
- `MonitoringDocumentHistoryComponent`: render bottone "Visualizza Assegnazione" se `status === COMPLETATO` → `router.navigate(['./cc', documentId], {queryParams: {mode: 'view'}})`.
- `practice-detail.component.html`: RIMUOVERE bottone "Assegna Monitoraggi" (righe 191-200).
- `practice-detail.component.ts`: RIMUOVERE `@ViewChild ccAssignment` (riga 38), `openCcAssignment()` (riga 111-113), `onCcAssignmentComplete()` (riga 123).

**Smoke E2E manuale** (vedi spec §8.3): Playwright sull'ambiente locale, utente `davide94x@gmail.com` (Deloitte). 22 step elencati nello spec.

**Commit message**: `feat(monitoring): wire CC assignment page in document-history + remove old dialog triggers`

### Commit 5 FE — Cleanup vecchi componenti (~15min)

- DELETE `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment/` (cartella intera, 2 file).
- DELETE `src/app/module/agency-desk/monitoring/practice-detail/monitoring-genai-extraction/` (cartella intera, 3 file).
- `practice-detail.module.ts`: rimuovere declarations + imports rifs a `CcAssignmentComponent` + `MonitoringGenAiExtractionModule`.
- Verify `npx tsc --noEmit --skipLibCheck` green + `npm test` full suite green.

**Commit message**: `chore(monitoring): remove deprecated CcAssignmentComponent + MonitoringGenAiExtractionComponent (replaced by CC assignment page)`

## View mode N-covenants — scope cut

Come da spec §11, BR 4.2.4 multi-covenant assignment è **out of scope** per questo bug. Richiede nuova tabella `mp_monitoring_document_assignment` o array `eventIds` in `MonitoringDocument`. Per ora, mode=view supporta solo 1 covenant assignment legacy (= `document.event` esistente). Documentare come limitazione corrente + creare bug separato in lista.

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
# Atteso BE HEAD: c83a7c0

cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3
# Atteso FE HEAD: 1d27047 (invariato)

# Pull BE in caso di nuovi commit di team
cd C:/Users/davmelis/Documents/Github/ba-back-end && git pull

# Sanity check BE
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw compile -q
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='Monitoring*' -q

# Sanity check FE (tsc deve essere green PRE Commit 3, perché monitoring.service.ts non ancora aggiornato)
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck

# Avvio BE (dopo Commit 3 FE!)
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw spring-boot:run -DskipTests -Dspring-boot.run.profiles=local
curl -s http://localhost:8091/actuator/health

# Avvio FE
cd C:/Users/davmelis/Documents/Github/ba-web && npm run start
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200

# Test endpoint BE nuovo (verifica con curl che esistono)
curl -s -X PATCH http://localhost:8091/monitoring/documents/1/validate
# Atteso: 401/403 se non autenticato, MA NON 404. Conferma path esistente.

curl -s 'http://localhost:8091/monitoring/practices/8288/schedule?covenantType=ICR&referenceDate=2026-05-28&rangeDays=15&statuses=SCADUTO,FUORI_SOGLIA,IN_SCADENZA'
# Atteso: 401/403 se non autenticato, MA NON 404.
```

## Bug list aggiornata — **18 totali, 4 fixati cumulativi**

⚠️ Bug #10 NON ancora fixato — BE pronto, FE da fare in prossima sessione. Quando Wave 2 FE chiude → bug #10 closed.

| # | Severity | Status |
|---|---|---|
| 16 | CRITICAL | ✅ FIXATO commit FE `1d27047` |
| 18 | HIGH | ✅ FIXATO commit BE `9ef5ede` |
| 2 | HIGH | ✅ FIXATO commit BE `c4efb8e` |
| **10** | **HIGH** | 🔧 **IN CORSO** — BE done (`21d5636`+`c83a7c0`), FE da fare (Wave 2) |
| 1, 3, 5, 6, 8, 9, 11, 12, 15, 17 | MEDIUM | APERTI (10) |
| 4, 14 | LOW | APERTI (2) |
| 7 | LOW | NON da fixare (by-design) |

## Suggerimenti per Wave 2

### Strategia raccomandata sessione FE
1. **Inizia con bootstrap context**: read questo handoff + spec + plan + verifica branch state.
2. **Genera sub-plan FE dettagliato**: dato che il plan corrente ha solo outline FE, scrivi un nuovo plan-FE con codice completo dei 4 componenti (HTML + TS + spec). Pattern: TDD test-first per i componenti più isolati (slot, table, dialog) → page-component last (integra tutto).
3. **Componente per componente**: implementa nell'ordine `Reject Dialog` (più semplice) → `Schedule Table` → `Covenant Slot` → `Page` (più integrato). Commit incrementale per ognuno o tutto in Commit 3 unico (preferred per coerenza migration).
4. **Run dev server prima del Commit 4**: verifica visualmente che ogni componente renderizzi correttamente.
5. **Smoke E2E al Commit 4**: prima di togliere il vecchio trigger, testa manualmente il nuovo flow end-to-end.
6. **Commit 5 solo dopo smoke verde**: cleanup è rollback-safety se il nuovo flow ha problemi.

### Potential gotchas
- **`monitoring.service.ts` ha `compactParams()` helper** (commit `1d27047`): usalo per i nuovi metodi così `null|undefined|''` vengono filtrati prima del send. Mantieni il pattern.
- **GenAI mock single-block**: il mock `AsyncMonitoringGenAiProcess` restituisce single covenant. Per N-covenants servirà mock multi-block; per ora il smoke gira con N=1 (limite documentato in spec §11).
- **Document preview iframe**: il browser stamperà cookie session per il GET `/download`. Verifica che CORS + auth siano OK per `<object>` tag (vs `<iframe>`).
- **Route param resolution**: `:documentId` deve essere convertito a `Number` prima di passarlo a service (Angular lo passa come `string`).

### Test pattern FE consigliati (Karma + Jasmine)
- `TestBed.configureTestingModule` con `provideRouter([{path: 'cc/:documentId', component: ...}])` per testare navigation
- Mock `MonitoringService` con `jasmine.createSpyObj`
- `fixture.detectChanges()` dopo ogni state change per renderizzare
- `By.css` o `nativeElement.querySelector` per asserzioni HTML

## Utenti DB locale (invariato)

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`)
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`)
- Login TOTP via Google/Microsoft Authenticator

## Riferimenti

- **Spec design**: `plans/in-progress/2026-05-04_monitoring/specs/2026-05-28-bug10-cc-assignment-page-design.md` (commit `147efc0`)
- **Implementation plan**: `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-28-bug10-cc-assignment-page-plan.md` (commit `06d7146`) — sezione III contiene outline FE
- **Handoff predecessor**: `handoff/HANDOFF_2026-05-28_bugfix_round_2_done.md`
- **BR**: `requirements/BR_Monitoraggio_V6.md` sez 4.2.3 (652-744) + sez 4.2.4 (745-748)
- **Commit BE wave 1**: `21d5636` + `c83a7c0` su `origin/feature/monitoring-integration-uat`
- **PLAN.md**: T-024 al 85% post wave BE (era 80% pre-wave)
