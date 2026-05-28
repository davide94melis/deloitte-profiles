# Handoff STEP 10 SMOKE DONE → BUG-FIX ROUND + RETEST E2E — 2026-05-28 (sera)

Sessione AI chiusa dopo STEP 10 smoke E2E PASS funzionale. Tutti gli STEP 1-10 ora smokati. 1 bug CRITICO #16 fixato in sessione (commit FE `1d27047`). Restano **17 bug aperti** da risolvere prima di chiudere T-024 e archiviare il piano. Prossima sessione = **bug-fix round** + **retest E2E completo Playwright** su tutto il modulo monitoring.

## Branch state

- BE `feature/monitoring-integration-uat` HEAD = **`0de244a`** (invariato dalla sessione precedente — PR #29 gia mergiato)
- FE `feature/monitoring-integration-uat` HEAD = **`1d27047`** (nuovo: fix #16 compactParams)
- Profilo `deloitte-profiles` HEAD = nuovo commit `<da fare>` con PROGRESS.md STEP 10 + questo handoff

## STEP smoke status — TUTTI DONE

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
| 7 | GenAI Deloitte (extract → modify → apply, refactor DeferredResult) | DONE |
| 8 | COVNO upload + storico | DONE |
| 9 | Mailing list (CRUD + lifecycle completo + reject) | DONE |
| 10 | Expired/Expiring events pages | **DONE smoke** (PASS funzionale + 1 fix CRITICO in sessione) |

## STEP 10 — Cosa e stato fatto in questa sessione

### Sanity check
- BE+FE HEAD invariati dalla sessione precedente (BE `0de244a`, FE `3f27c1b` pre-fix).
- `git fetch` su entrambi i repo: nessun nuovo commit su `feature/monitoring-integration-uat` (Georgios/Alexios non hanno pushato nulla dopo PR #29).
- BE+FE up e healthy (curl actuator 200 + FE root 200).
- DB locale M1 fixture invariato dal handoff precedente (id 8288, MONITORING_OK, 6 eventi M1C1E1-E6).

### Fix #16 in sessione (commit FE `1d27047`)
**Root cause**: `monitoring.service.ts` passava direttamente l'oggetto `MonitoringEventFilterModel` a `HttpClient`/`RestService` come params. Quando i campi opzionali (`practiceCode`, `trackingId`, `dealName`, `covenantType`, `expirationDateFrom`, `expirationDateTo`) sono `undefined`, Angular serializza come stringa letterale `"undefined"` nel query string. Il BE `MonitoringDashboardController.getExpiredEvents(@RequestParam(required=false) LocalDate expirationDateFrom, ...)` riceve `"undefined"` e fallisce il parsing LocalDate → 500 Internal Server Error → pagina vuota (`In totale ci sono 0 righe`).

**Impattati 4 endpoint** del service:
- `getExpiredEventsPage`
- `getExpiringEventsPage`
- `exportExpiredEventsExcel`
- `exportExpiringEventsExcel`

**Fix minimal**: helper privato `compactParams()` (8 righe) che filtra `undefined|null|''` prima del send:
```typescript
private compactParams(params: Record<string, any>): Record<string, any> {
  return Object.entries(params)
    .filter(([, value]) => value !== undefined && value !== null && value !== '')
    .reduce((acc, [key, value]) => ({ ...acc, [key]: value }), {});
}
```
**Post-fix**: query string pulita (`page=0&size=20&sortBy=expirationDate&sortDirection=ASC`), tabella popolata con 2 righe FUORI_SOGLIA. Verificato in browser per entrambi i ruoli.

### Coverage smoke E2E
**Deloitte `/consultant/monitoring/*`**:
- `expired-events`: 2 righe FUORI_SOGLIA (M1C1E2 + M1C1E4), 13 colonne BR sez. 8.4.3, 2 date filter + 4 column filter, server pagination 20 righe, filtro M99→0 / M1→2, export Excel `Report_expired_events.xlsx` (sheet `Monitoraggi_Scaduti`, 13 col, 2 righe) verificato via openpyxl.
- `expiring-events`: 0 righe (atteso, fixture default). Test logica IN_SCADENZA modificando temp E5 `exp=2026-06-03` → 1 riga visualizzata correttamente (`6` giorni al prossimo), rollback fatto.
- Link "Vedi tutte" dashboard → naviga alle pagine dedicate.

**ISP `/user/monitoring/*`**:
- `expired-events`: 2 righe identiche a Deloitte, sidebar simile, filtri identici.
- `expiring-events`: 0 righe (atteso).

### Logica BE `MonitoringStateService` verificata
- `calculateEventState`: se `event.isHasException()` → forzato `OK` (riga 58-60 di `MonitoringStateService.java`). Per questo M1C1E1 (exp=2026-05-28=today, `has_exception=1`) NON appare in expiring nonostante exp=today. By-design corretto.
- `calculateDateBasedState`: IN_SCADENZA se `expiration BETWEEN today AND today.plusDays(EXPIRATION_WARNING_DAYS=7)`.

## Bug list aggiornata — **18 totali, 1 fixato, 17 aperti**

### Bug da sessioni pregresse (10, 0 fixati)

| # | Severity | Descrizione | Fix suggerito |
|---|---|---|---|
| 1 | MEDIUM | `file_reference=NULL` document eccezione (gap T-024.x, storage binario non implementato per requestException) | Implementare storage binario S3/DM per requestException multipart |
| 2 | HIGH | `mp.deal_id`/`contract_key` NULL su M1; `practice.deal_name=NULL` → in lista mailing list e in expired/expiring le celle "ID Censimento"/"Tracking ID" sono vuote o swap-pate (cella "Nome Deal" mostra `"1234"` che e' il tracking_id) | `createFromPostClosing` deve copiare `deal_id`/`contract_key`/`deal_name` dalla pratica censimento collegata via Tracking ID |
| 3 | MEDIUM | `[object Object]` colonna "Caricato da" (probabilmente risolto da Alexios refactor) | Riverificare dopo refactor Alexios `feature/monitoring-bugfixes` |
| 4 | LOW | `CovenantTypeNumericEnum` FE 10/58 — drift tra enum FE e BE | Allineare enum FE generato da BE (es. `quicktype` o copia manuale) |
| 5 | MEDIUM | Field "Variabile" input text vs dropdown SI/NO | Cambiare a `<p-dropdown>` con `{label: 'Si', value: true}, {label: 'No', value: false}` |
| 6 | MEDIUM | PUT covenant non rigenera eventi se cambia `periodicity` | Hook su `MonitoringCovenantService.update` → `MonitoringScheduleGeneratorService.regenerateEvents(covenantId)` |
| 7 | LOW (by-design placeholder) | `MailingListAdForm.state` mai persistito `VERIFICA_IN_CORSO` (extract placeholder setta poi sovrascrive nello stesso TX) | No action (gia documentato), si risolvera quando GenAI vera sara async |
| 8 | MEDIUM | `MonitoringPracticeExcelWriterService` enum prefix bug | Controllare e correggere `*.name()` vs label/translation |
| 9 | MEDIUM | Mock GenAI label vs enum BE — Covenant fields non aggiornati da `applyExtraction` | Allineare chiavi mock (`COVENANT_NUM_*`) a `MonitoringGenAiKeyEnum` BE |
| 10 | HIGH | BR 4.2.3 page-based UX — refactor dialog Modifica Dati → pagina dedicata + N selettori + scadenziere inline (~2-3h) | Sostituire `MonitoringGenAiExtractionComponent` con `MonitoringCcAssignmentPageComponent` |

### Bug rilevati STEP 9 (5, 0 fixati)

| # | Severity | Descrizione | Fix suggerito |
|---|---|---|---|
| 11 | MEDIUM | Upload AD Form senza feedback visivo (no spinner/progress) | Wrappare la call con `SpinnerComponent.showLoader()/hideLoader()` come pattern STEP 7 |
| 12 | MEDIUM | Validation formato/size file upload mostra solo 2 icone X rosse senza messaggio testuale | Hook su `onError`/`onValidationError` PrimeNG → toast esplicativo |
| 14 | LOW | Bottone "Conferma e Aggiorna" visibile nel detail mailing-list Deloitte anche in `REPORT_GENERATO` | Gate `*ngIf="adForm.state !== REPORT_GENERATO"` o cambia label in "Scarica Report" |
| 15 | MEDIUM | Popup ISP "Visualizza Richiesta Aggiornamento" senza bottone Chiudi ne X | Aggiungere bottone "Chiudi" neutrale tra i due esistenti, oppure `[closable]="true"` |

(NB. il vecchio bug #13 e' stato rimosso nella sessione STEP 9 perche' non era un bug ma comportamento atteso del stage progress.)

### Bug nuovi STEP 10 (3, 1 fixato)

| # | Severity | Descrizione | Fix suggerito | Stato |
|---|---|---|---|---|
| 16 | CRITICAL | Query params `undefined` serializzati come stringa letterale → 500 BE su 4 endpoint Expired/Expiring/Export | Helper `compactParams()` privato | ✅ **FIXATO commit FE `1d27047`** |
| 17 | MEDIUM | Cella "Azione" mostra stringa raw `"UPDATE"` invece di link/button cliccabile. BR sez. 8.4.3 punto 13 vuole "Salta monitoraggio"/"Ignora Soglia" cliccabili per ISP, action diversa per Deloitte | Refactor `mapEventToRow` in `expired-events.component.ts` + `expiring-events.component.ts` per usare `DynamicComponentModel<ButtonComponent>` role-gated identico al `buildActionCell` di `practice-detail.component.ts`. Wire `MonitoringExceptionDialogModule` con `@ViewChild` come gia fatto in T-019. | APERTO |
| 18 | MEDIUM | Dashboard donut mostra pratica `OK` ma stessa pratica ha `N. Scaduti=2` in tabella sotto. `mp_practice.state_id` non ricalcolato post-`applyExtraction` GenAI o post-`CovnoUploadService.processCovno` | Chiamare `MonitoringStateService.recalculateAllStates(practiceId)` + ricalcolare stato pratica al termine di `MonitoringGenAiService.applyExtraction` e `CovnoUploadService.processCovno`. Verificare anche se manca call al termine di `requestException`/`rejectAdForm`/altri endpoint che modificano lo stato. | APERTO |

## Fixture DB locale M1 post-smoke STEP 10 (id 8288, MONITORING_OK, termination_date=2027-12-31)

```
Pratica M1 (id 8288): MONITORING_OK ← stato fuorviante, dovrebbe essere FUORI_SOGLIA (bug #18)
  Covenant M1C1 (id 2): LOAN_TO_VALUE, LTE 70, EBITDA, TRIMESTRALE
    Event M1C1E1 (id 1): OK (has_exception=1), actual=2026-05-26, exp=2026-05-28
    Event M1C1E2 (id 2): FUORI_SOGLIA (GenAI applied), value=100000, actual=2026-05-27, exp=2026-08-27
    Event M1C1E3 (id 3): OK (COVNO), value=50, actual=2026-11-23, exp=2026-11-23
    Event M1C1E4 (id 4): FUORI_SOGLIA (COVNO), value=80, actual=2027-02-23, exp=2027-02-23
    Event M1C1E5 (id 5): NESSUNO_STATO, exp=2027-05-23 (rollback dopo test IN_SCADENZA)
    Event M1C1E6 (id 6): NESSUNO_STATO, exp=2027-08-23

Mailing list AD Form M1: id=3 DOC_COMPLETA (storicizzato), id=4 DOC_INCOMPLETA (visible)
Mailing list contacts M1: 0
```

### Note importanti su fixture
- Per il bug-fix round NON serve toccare la fixture: i bug riguardano logica di stato, UX, mapping e dipendono dai dati gia presenti.
- Per testare nuovamente IN_SCADENZA: `UPDATE mp_monitoring_event SET expiration_date='<today+5gg>' WHERE id=5;` poi ricarica `/{role}/monitoring/expiring-events` poi `UPDATE mp_monitoring_event SET expiration_date='2027-05-23' WHERE id=5;`.

## Utenti DB locale

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`)
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`)
- Password (locale dev, entrambi gli account): nota su file profile / sessione
- Login richiede TOTP via Google/Microsoft Authenticator

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3

# Pull eventuali nuovi commit pre-bug-fix
cd C:/Users/davmelis/Documents/Github/ba-back-end && git pull
cd C:/Users/davmelis/Documents/Github/ba-web && git pull

# Sanity check
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw clean compile -DskipTests -q
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck

# BE+FE health
curl -s http://localhost:8091/actuator/health
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200

# Stato fixture M1 (post-rollback)
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT id, event_code, status, expiration_date, actual_date, monitoring_value, has_exception, data_source FROM mp_monitoring_event WHERE covenant_id=2 ORDER BY reference_date;"

# Stato pratica (per verificare bug #18)
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT p.id, p.code, p.deal_id, p.deal_name, p.contract_key, ps.code AS state FROM mp_practice p JOIN mp_psm_state ps ON ps.id=p.state_id WHERE p.id=8288;"
```

## BUG-FIX ROUND — Action plan prossima sessione

### Priorita 1 — HIGH (semantica errata visibile a stakeholder)
1. **Bug #2** — `createFromPostClosing` copia `deal_id`/`contract_key`/`deal_name`. Test: M1 lista mailing/expired/expiring deve mostrare "Nome Deal" e "Tracking ID" corretti.
2. **Bug #18** — `recalculateAllStates` post `applyExtraction` + post `processCovno` + post `requestException`/`rejectAdForm`. Test: dashboard donut M1 deve mostrare "Fuori Soglia" = 1.
3. **Bug #10** — Refactor dialog Modifica Dati → pagina dedicata (~2-3h). BR 4.2.3 + mockup slide 17-24.

### Priorita 2 — MEDIUM (UX/funzionale)
4. **Bug #17** — Cella Action wire in expired/expiring (refactor `mapEventToRow` con `DynamicComponentModel<ButtonComponent>` + role-gating + `MonitoringExceptionDialogModule`).
5. **Bug #1** — Storage binario document eccezione (S3 o DM).
6. **Bug #5** — Field "Variabile" dropdown SI/NO.
7. **Bug #6** — PUT covenant rigenera eventi se cambia periodicity (`MonitoringScheduleGeneratorService.regenerateEvents`).
8. **Bug #8** — `MonitoringPracticeExcelWriterService` enum prefix.
9. **Bug #9** — Mock GenAI label vs enum BE (`COVENANT_NUM_*` → `MonitoringGenAiKeyEnum`).
10. **Bug #11** — Upload AD Form spinner.
11. **Bug #12** — File upload validation toast esplicativo.
12. **Bug #15** — Popup ISP "Visualizza Richiesta" bottone Chiudi/X.
13. **Bug #3** — `[object Object]` colonna "Caricato da" (riverificare se ancora aperto post Alexios refactor).

### Priorita 3 — LOW
14. **Bug #4** — Enum FE 10/58 drift.
15. **Bug #14** — Bottone "Conferma e Aggiorna" visibile in `REPORT_GENERATO`.

### Bug NON da fixare (by-design)
- **#7** — `MailingListAdForm.state VERIFICA_IN_CORSO` mai persistito (placeholder GenAI, by-design fino a GenAI vera async).

### Workflow proposto per ogni bug
1. Read del file impattato + contesto (component+service+test).
2. Fix minimal + scrivere/aggiornare unit test.
3. `mvn compile` BE (se BE) / `npx tsc --noEmit` FE (se FE).
4. Smoke manuale nel browser (Playwright se necessario per UX-critical).
5. Commit atomico con messaggio descrittivo (no Co-Author per memoria utente).
6. Push.

## RETEST E2E COMPLETO (post bug-fix round)

Quando tutti i bug priorita 1+2 sono chiusi, eseguire **retest E2E completo via Playwright** su:
- **STEP 1-3**: Dashboard donut (verificare bug #18 fixato), 3 tabelle, polling 60s, COVNO card.
- **STEP 4**: Tabella pratiche monitoring (filtri, export, paginazione).
- **STEP 5-6**: Dettaglio pratica ISP (3 tab) + Aggiungi Covenant + Modifica Dati Covenant full-page + Eccezioni (request/skip/visualizza/annulla).
- **STEP 7**: GenAI Deloitte (extract → modify → apply). Verificare bug #9 (mock label) + bug #10 (page-based UX se gia fixato).
- **STEP 8**: COVNO upload + storico. Verificare bug #18 (ricalcolo stato pratica post-COVNO).
- **STEP 9**: Mailing list (CRUD + AD Form lifecycle + reject). Verificare bug #11/#12/#14/#15.
- **STEP 10**: Expired/Expiring events pages. Verificare bug #2 (Tracking ID/Nome Deal corretti) + bug #17 (Azione cliccabile con modal eccezione).

Output retest: aggiornare PROGRESS.md con esito + nuovo handoff finale per chiudere T-024 e archiviare il piano.

## Riferimenti

- Handoff predecessore: `handoff/HANDOFF_2026-05-28_step9_done_step10_events.md`
- Plan completo: `PLAN.md` (STEP 10 = ultimo step del piano monitoring)
- BR sez. 8.4 + 8.5: tabelle Monitoraggi Scaduti + In Scadenza (pagine dedicate)
- Commit fix #16: `1d27047` (FE) — `fix(monitoring): omit undefined filters in expired/expiring queries`
- Componenti FE STEP 10:
  - `src/app/module/agency-desk/monitoring/expired-events/expired-events.component.ts`
  - `src/app/module/agency-desk/monitoring/expiring-events/expiring-events.component.ts`
  - `src/app/service/monitoring.service.ts` (compactParams helper)
- Controller BE STEP 10:
  - `src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDashboardController.java` (endpoint `/monitoring/dashboard/expired`, `/expiring`, `/expired/export`, `/expiring/export`)
- Service BE STEP 10:
  - `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDashboardService.java` (`getExpiredEvents`, `getExpiringEvents`, `applyFilters`)
  - `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringStateService.java` (`calculateEventState`, `calculateDateBasedState`)
