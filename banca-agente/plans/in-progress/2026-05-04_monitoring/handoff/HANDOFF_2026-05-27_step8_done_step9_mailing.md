# Handoff STEP 8 DONE → STEP 9 Mailing List — 2026-05-27

Sessione AI chiusa dopo STEP 8 (COVNO) PASS. Pull di nuovi commit Georgios PR #29 mailing list applicato. Nuova sessione consigliata per verificare quanto fatto + procedere con STEP 9.

## Branch state

- BE `feature/monitoring-integration-uat` HEAD = **`0de244a`** (pushato, PR #29 mergiato)
- FE `feature/monitoring-integration-uat` HEAD = **`3f27c1b`** (pushato, PR #29 mergiato)
- Profilo `deloitte-profiles` HEAD = `9436ab3` (PROGRESS.md aggiornato fino a STEP 7 refactor wait/spinner — STEP 8 da documentare)

## STEP smoke status

| # | Step | Stato |
|---|---|---|
| 1 | Verifica fix StatesOptions dashboard | DONE |
| 2 | Generare M1 fixture | DONE |
| 3 | Dashboard monitoring (donut + 3 tabelle) | DONE |
| 4 | Tabella pratiche monitoring + filtri + export | DONE |
| 5 | Dettaglio pratica ISP (3 tab) | DONE |
| 5b | Aggiungi Covenant + 7 eventi auto-generati | DONE |
| 5c | Modifica Dati Covenant full-page | DONE |
| 6 | Eccezioni (request/skip/visualizza/annulla) | DONE |
| 7 | GenAI Deloitte (extract → modify → apply) | **DONE** (refactor BR-aligned: DeferredResult + spinner globale) |
| 8 | COVNO upload + storico | **DONE** (XLSX `covno_test_M1.xlsx` 2 righe E3+E4, DB verificato) |
| 9 | Mailing list (CRUD + AD Form reject) | **PENDING — STARTING POINT NUOVA SESSIONE** |
| 10 | Expired/Expiring events pages | PENDING |

## Nuovi commit Georgios da analizzare prima di STEP 9

**Mergiati 2026-05-27** in PR #29 (BE+FE). Focused esattamente sulla mailing list.

### BE (`c5c06b2..0de244a` → 3 commit Georgios + 1 merge)
- `8b668ce` **feat(monitoring): add dedicated mailing list practice search endpoint**
- `af28153` **fix(monitoring): add mailing list practice search, conclude endpoint, and inline AD form status**
- `c6e8dae` test(monitoring): testing new endpoints
- `bb4cf97` Merge feature/monitoring-integration-uat → feature/monitoring-alignment-be
- `0de244a` Merge PR #29

**File modificati** (7 file, 232 ins):
- NEW `MailingListPracticeFilterDTO.java`
- MOD `MailingListController.java` (+23 — nuovi endpoint)
- MOD `MailingListService.java` (+17)
- MOD `MonitoringPracticeService.java` (+53)
- MOD `MonitoringPracticeSummaryDTO.java` (+4 — probabile inline AD form status)
- MOD `MonitoringPracticeSummaryViewRepository.java` (+44 — nuova query)
- MOD `MailingListServiceTest.java` (+72 — test nuovi endpoint)

### FE (`b7d3974..3f27c1b` → 3 commit Georgios + 1 merge)
- `f59e468` **fix(monitoring): file upload formats and size, filters**
- `7d243b3` **fix(monitoring): wire mailing list to dedicated endpoint, eliminate N+1 calls, fix UX issues**
- `0d59349` test(monitoring): align tests with changes
- `3f27c1b` Merge PR #29

**File modificati** (8 file, 90 ins / 71 del):
- MOD `monitoring-mailing-list.service.ts` (+12 — nuovo endpoint client)
- MOD `monitoring-practice.model.ts` (+3 — inline AD form status field)
- MOD `mailing-list.component.html` (refactor)
- MOD `mailing-list.component.ts` (-63/+63 — refactor sostanziale, no più N+1)
- MOD `mailing-list-detail.component.html` (+2)
- MOD `mailing-list-detail.component.ts` (+11)
- MOD test files

### Azioni preliminari STEP 9 (nuova sessione)

1. **Diff dei nuovi commit Georgios** per capire:
   - Nuovi endpoint BE esposti (search dedicato + conclude)
   - `MailingListPracticeFilterDTO` fields
   - Nuovo metodo `MonitoringPracticeSummaryDTO.adFormStatus`
   - Validazioni file upload FE (formats accettati + size max)
   - Filtri introdotti FE
   - "Conclude" endpoint — probabile relativo a "Concludi Procedura"
2. **Sanity check post-merge**: `mvn compile` BE + `npx tsc --noEmit` FE
3. **Verifica fixture DB M1**: `mp_mailing_list_ad_form` ha 1 record `DOC_INCOMPLETA`; `mp_mailing_list_contact` ha 0 contatti per M1
4. **Restart BE+FE**
5. **Smoke STEP 9** secondo flusso BR sez. 6.x

## STEP 7 refactor — Cosa è cambiato in questa sessione (riepilogo)

- **Race condition `@Async`** — fix con `TransactionSynchronizationManager` in `MonitoringGenAiService.startExtraction` (commit BE `e442a02`)
- **Refactor BR-aligned wait** (commit BE `c5c06b2`, FE `b7d3974`):
  - NEW `GenAiExtractionCompletedEvent` (record event)
  - NEW `MonitoringGenAiWaiter` (`@TransactionalEventListener(AFTER_COMMIT)` + `ConcurrentMap<documentId, List<DeferredResult>>`)
  - MOD `MonitoringGenAiController.getResult` → ritorna `DeferredResult<...>` con `?wait=true` query param + timeout 30s
  - MOD `AsyncMonitoringGenAiProcess` → publish event on success/failure
  - **FIX collaterale** `JwtTokenFilter.shouldNotFilterAsyncDispatch() = false` (default `OncePerRequestFilter` skippa async dispatch → AnonymousAuthenticationToken → AccessDenied al re-dispatch)
  - FE `document-history.component.ts` → spinner globale `SpinnerComponent` (pattern statico montato in `app.component.html:4`)
  - FE `monitoring.service.ts` → `getGenAiResult(documentId, wait?=false)` aggiunge query param

## STEP 8 COVNO — Risultati (da documentare in PROGRESS.md)

- XLSX fixture: `C:\Users\davmelis\Downloads\Monitoring e2e test\covno_test_M1.xlsx` (2 righe E3+E4)
- DB post-upload:
  - `mp_covno_upload_history` → 1 record SUCCESS, 2 processed, 2 updated, uploaded_by `davide94x@gmail.com`, practice_id `8288`
  - M1C1E3 → status `OK`, value `50`, actual_date `2026-11-23`, data_source `COVNO` ✅
  - M1C1E4 → status `FUORI_SOGLIA`, value `80`, actual_date `2027-02-23`, data_source `COVNO` ✅
- UI Tab Covenant riflette correttamente i conteggi (`Scaduti=2, In Scadenza=0, OK=2`)

## Fixture DB locale M1 (id 8288, MONITORING_OK, termination_date=2027-12-31)

```
Pratica M1 (id 8288): MONITORING_OK
  Covenant M1C1 (id 2): LOAN_TO_VALUE, LTE 70, EBITDA, TRIMESTRALE
    Event M1C1E1 (id 1): OK (con eccezione), actual_date=2026-05-26
      Document (id 3): COMPLETATO, file_reference=NULL
    Event M1C1E2 (id 2): FUORI_SOGLIA (GenAI applied), value=100000, actual=2026-05-27
      Document (id 4): COMPLETATO (GenAI applied)
    Event M1C1E3 (id 3): OK (COVNO), value=50, actual=2026-11-23
    Event M1C1E4 (id 4): FUORI_SOGLIA (COVNO), value=80, actual=2027-02-23
    Event M1C1E5..E6: NESSUNO_STATO (futuri)

Mailing list AD Form M1: 1 record DOC_INCOMPLETA, file_reference=NULL (pre-fix Georgios)
Mailing list contacts M1: 0
```

## Utenti DB locale

- `davide94x@gmail.com` → Deloitte CONSULTANT
- `davide94melis@gmail.com` → ISP OWNER (banca)

## Bug noti residui

1. **file_reference=NULL document eccezione** (#1) — gap T-024.x, storage binario non implementato per requestException
2. **mp.deal_id/contract_key NULL su M1** (#2) — createFromPostClosing non copia
3. **[object Object] colonna "Caricato da"** (#3) — probabilmente risolto da Alexios refactor
4. **CovenantTypeNumericEnum FE 10/58** (#4) — drift
5. **Field "Variabile" input text vs dropdown SI/NO** (#5)
6. **PUT covenant non rigenera eventi se cambia periodicity** (#6)
7. **MailingListAdForm.state mai persistito VERIFICA_IN_CORSO** (#7) — **da verificare se risolto da Georgios PR #27 + #29**
8. **MonitoringPracticeExcelWriterService enum prefix bug** (#8)
9. **Mock GenAI label vs enum BE** (#9) — Covenant fields non aggiornati da applyExtraction
10. **BR 4.2.3 page-based UX** (#10) — dialog Modifica Dati va sostituito con pagina dedicata + N selettori + scadenziere inline (refactor ~2-3h)

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3

# Diff dei nuovi commit Georgios PR #29 (focus mailing list)
cd C:/Users/davmelis/Documents/Github/ba-back-end && git diff c5c06b2..0de244a --stat
cd C:/Users/davmelis/Documents/Github/ba-web && git diff b7d3974..3f27c1b --stat

# Sanity check post-merge
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw clean compile -DskipTests -q
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck

# Stato DB mailing list M1
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT * FROM mp_mailing_list_ad_form WHERE practice_id=8288; \
   SELECT * FROM mp_mailing_list_contact WHERE practice_id=8288;"

# BE+FE health
curl -s http://localhost:8091/actuator/health
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200
```

## Action item nuova sessione

1. Legge questo handoff + ultimo blocco PROGRESS.md
2. Diff dei 6 commit Georgios PR #29 per capire la mailing list updated
3. Sanity check `mvn compile` + `tsc`
4. Restart BE+FE (utente)
5. Procedi con STEP 9 smoke E2E ISP+Deloitte come da flusso BR sez. 6.x
