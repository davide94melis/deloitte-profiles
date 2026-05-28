# Handoff BUG-FIX ROUND 2 DONE — 2026-05-28 (notte)

Sessione AI chiusa dopo bug-fix round 2. Chiusi **2 bug HIGH** end-to-end (#18 + #2). Scoperto che il bug #10 ha scope reale molto maggiore della stima (~4-6h invece di 2-3h) e va trattato in sessione dedicata. Restano **13 bug aperti** prima di chiudere T-024.

## Branch state

- BE `feature/monitoring-integration-uat` HEAD = **`c4efb8e`** (era `0de244a` — 2 nuovi commit)
  - `9ef5ede` fix(monitoring): recalculate practice.state after every event-affecting operation
  - `c4efb8e` fix(monitoring): map Tracking ID to booking_tracking_id per BR sez 8.4.3
- FE `feature/monitoring-integration-uat` HEAD = **`1d27047`** (invariato — nessuna modifica FE in sessione)
- Profilo `deloitte-profiles` HEAD = nuovo commit `<da fare>` con PROGRESS.md + questo handoff

## Bug list aggiornata — **18 totali, 4 fixati, 14 aperti** (-3 da handoff precedente)

### Bug fixati cumulativi (4, +2 in sessione corrente)

| # | Severity | Descrizione | Commit |
|---|---|---|---|
| 16 | CRITICAL | Query params undefined → 500 BE | FE `1d27047` (sessione precedente) |
| 13 | — | (rimosso, no bug) | — |
| **18** | **HIGH** | **Dashboard donut OK ma tabella Scaduti=2 — recalc state non aggiorna practice.state** | **BE `9ef5ede` (sessione corrente)** |
| **2** | **HIGH** | **Tracking ID vuoto + Nome Deal mostra tracking_id — mapper trackingId=dealId errato + dati legacy M1 corrotti** | **BE `c4efb8e` (sessione corrente)** |

### Bug aperti restanti (13)

#### Priorita 1 — HIGH (1 restante)

| # | Severity | Descrizione | Fix suggerito | Note |
|---|---|---|---|---|
| **10** | **HIGH** | **BR 4.2.3 page-based UX — refactor dialog Modifica Dati → pagina dedicata** | Vedere sezione dedicata sotto | **SCOPE REVISIONATO 4-6h** |

#### Priorita 2 — MEDIUM (10 restanti)

| # | Severity | Descrizione | Fix suggerito |
|---|---|---|---|
| 1 | MEDIUM | `file_reference=NULL` document eccezione (storage binario non implementato per requestException) | Implementare storage binario S3/DM per requestException multipart |
| 3 | MEDIUM | `[object Object]` colonna "Caricato da" | Riverificare dopo refactor Alexios `feature/monitoring-bugfixes` |
| 5 | MEDIUM | Field "Variabile" input text vs dropdown SI/NO | Cambiare a `<p-dropdown>` con `{label:'Si',value:true},{label:'No',value:false}` |
| 6 | MEDIUM | PUT covenant non rigenera eventi se cambia `periodicity` | Hook su `MonitoringCovenantService.update` → `MonitoringScheduleGeneratorService.regenerateEvents(covenantId)` |
| 8 | MEDIUM | `MonitoringPracticeExcelWriterService` enum prefix bug | Controllare e correggere `*.name()` vs label/translation |
| 9 | MEDIUM | Mock GenAI label vs enum BE — Covenant fields non aggiornati da `applyExtraction` | Allineare chiavi mock (`COVENANT_NUM_*`) a `MonitoringGenAiKeyEnum` BE |
| 11 | MEDIUM | Upload AD Form senza feedback visivo (no spinner/progress) | Wrappare la call con `SpinnerComponent.showLoader()/hideLoader()` (pattern STEP 7) |
| 12 | MEDIUM | Validation formato/size file upload mostra solo 2 icone X rosse senza messaggio testuale | Hook su `onError`/`onValidationError` PrimeNG → toast esplicativo |
| 15 | MEDIUM | Popup ISP "Visualizza Richiesta Aggiornamento" senza bottone Chiudi ne X | Aggiungere bottone "Chiudi" neutrale tra i due esistenti, oppure `[closable]="true"` |
| 17 | MEDIUM | Cella "Azione" expired/expiring mostra stringa raw `"UPDATE"` invece di link/button cliccabile | Refactor `mapEventToRow` in `expired-events`/`expiring-events.component.ts` per usare `DynamicComponentModel<ButtonComponent>` role-gated (pattern T-019 `buildActionCell`). Wire `MonitoringExceptionDialogModule` con `@ViewChild` |

#### Priorita 3 — LOW (2 restanti)

| # | Severity | Descrizione | Fix suggerito |
|---|---|---|---|
| 4 | LOW | `CovenantTypeNumericEnum` FE 10/58 — drift tra enum FE e BE | Allineare enum FE generato da BE (es. `quicktype` o copia manuale) |
| 14 | LOW | Bottone "Conferma e Aggiorna" visibile nel detail mailing-list Deloitte anche in `REPORT_GENERATO` | Gate `*ngIf="adForm.state !== REPORT_GENERATO"` o cambia label in "Scarica Report" |

#### Bug NON da fixare (by-design)

| # | Note |
|---|---|
| 7 | `MailingListAdForm.state VERIFICA_IN_CORSO` mai persistito (placeholder GenAI, by-design fino a GenAI vera async) |

## Bug #18 fix dettagli (commit BE `9ef5ede`)

### Root cause
`MonitoringStateService.recalculateAllStates(Long practiceId)` aggiornava solo `event.status` di tutti gli eventi del covenant, **mai** `mp_practice.state_id`. Tutte le entry points (updateValue, updateDates, requestException, cancelException, addEventToCovenant, deleteEvent, CovnoUploadService.uploadAndMerge, e indirettamente MonitoringGenAiService.applyExtraction via eventService) gia chiamavano `recalculateAllStates`, quindi non c'era da wire-up niente — la fix andava concentrata in `recalculateAllStates` stessa.

### Modifiche (`9ef5ede`)
- `MonitoringStateService.java`:
  - Inject `PracticeRepository` + `PsmStateService`
  - Esteso `recalculateAllStates()`: dopo il loop eventi, fetch practice, skip se `MONITORING_CONCLUSA` (terminale impostato manualmente da `terminatePractice`), poi `practice.setState(psmStateService.findByCode(toPsmStateCode(calculatePracticeState(practice))))`
  - Nuovo helper privato `toPsmStateCode(MonitoringPracticeStateEnum) → PsmStateCodeEnum` con switch exhaustive 6 casi (OK, IN_SCADENZA, DA_ATTENZIONARE, SCADUTO, FUORI_SOGLIA, CONCLUSA — `CONCLUSA` aggiunta per exhaustive anche se mai chiamata da `calculatePracticeState`)
- `MonitoringGenAiControllerTest.java` (fix pre-esistente test compile error):
  - Signature `getResult(Long)` → `getResult(Long, boolean)` con return `DeferredResult<...>`
  - Aggiunto `@Mock MonitoringGenAiWaiter`

### Verifica
- Main compile pulito, test compile pulito, 145/145 monitoring tests PASS
- Smoke E2E browser (Playwright) come ISP `davide94melis@gmail.com` → click "Salta Monitoraggio" su M1C1E2 → SQL conferma `mp_practice.state_id` M1 da `MONITORING_OK` a `MONITORING_FUORI_SOGLIA` (last_modified ora attuale, 18:17)
- Dashboard donut post-recalc: `OK=0, Fuori Soglia=1` (era `OK=1, Fuori Soglia=0`)
- Tabella Pratiche Recenti M1: `N. Scaduti=1` (E2 ora con eccezione, E4 resta), Semaforo "Fuori Soglia"
- Tabella Monitoraggi Scaduti: solo M1C1E4 (M1C1E2 sparito)

## Bug #2 fix dettagli (commit BE `c4efb8e`)

### Root cause analisi
BR sez 8.4.3 (riga 218) + tabella pratiche (riga 915) chiariscono semantica:
- **Tracking ID** = "ID univoco dei sistemi ISP per riconoscere l'operazione di riferimento" / "numero di tracciamento (es. 001)" → mappato a `mp_practice.booking_tracking_id`
- **ID Contratto** = colonna separata, mappato a `mp_practice.contract_key`
- **Nome Deal** = "Il nome della pratica/finanziamento (es. Banca Nuova S.p.A)" → mappato a `mp_practice.deal_name`
- **ID Censimento Pratica** = colonna mailing list, mappato al `code` della pratica censimento parent

Bug: i mapper in `MonitoringDashboardService.toEventDto:173` + `MonitoringDashboardService.toPracticeDto:197` + `MonitoringPracticeService.toDto:184` settavano `dto.trackingId = practice.dealId / view.dealId` (concetto separato, identificativo banca-side).

### Modifiche (`c4efb8e`)
- `vw_monitoring_practice_summary` (file `001_view_monitoring_practice_summary.sql`): aggiunto `mp.booking_tracking_id` come campo separato della view
- `MonitoringPracticeSummaryView.java` (entity): aggiunto field `bookingTrackingId` con `@Column(name="booking_tracking_id")`
- `MonitoringDashboardService.toEventDto`: `practice.getBookingTrackingId()` invece di `practice.getDealId()`
- `MonitoringDashboardService.toPracticeDto`: `view.getBookingTrackingId()` invece di `view.getDealId()`
- `MonitoringPracticeService.toDto`: `view.getBookingTrackingId()` invece di `view.getDealId()`
- **DB view live**: applicata manualmente via `CREATE OR REPLACE VIEW` (Spring SQL init non e' configurato in application-local.yml — il file SQL e' usato per setup iniziale, non per migration automatica)

### Data fix legacy M1 (NO commit, fixture locale)
M1 (id 8288) e parent C_8265 (id 8265) avevano dati corrotti da fixture manuale:
- Pre-fix: `deal_id=NULL, deal_name='1234' (=tracking ID errato), contract_key=NULL, booking_tracking_id='1234'`
- Post-fix SQL inline:
  ```sql
  UPDATE mp_practice SET deal_id='DEAL-M1-001', deal_name='Starhotels Finanziaria', contract_key='CONTRACT-M1-001' WHERE id IN (8265, 8288);
  ```

### Verifica
- Main compile pulito, 81/81 monitoring tests PASS
- Restart BE necessario per ricaricare entity con nuovo field
- Smoke browser dashboard (`/user/monitoring/dashboard`):
  - Tabella Monitoraggi Scaduti M1 riga: `[M1, 1234, Starhotels Finanziaria, M1C1, M1C1E4, ...]` ✓
  - Tabella Pratiche Recenti M1 riga: `[M1, 1234, Starhotels Finanziaria, davide94melis@gmail.com, davide94x@gmail.com, BANCA_AGENTE, 1, 0, 0, Fuori Soglia, 23/05/2026, ...]` ✓

## Bug #10 — SCOPE REALE REVISIONATO (PROSSIMA SESSIONE DEDICATA)

L'handoff predecessore stimava 2-3h. Lettura BR sez 4.2.3 (riga 652-744) ha rivelato un flusso molto piu' articolato:

### Requirements page-based UX (BR 4.2.3)

**Workflow GenAI** (riga 722-744):
1. Page dedicata con **preview del documento** Compliance Certificate
2. Box selezione "Assegnazione con GenAI" vs "Assegnazione manuale"
3. Bottone bad flow "Ritorna a documentazione incompleta" → apre modal motivazione rifiuto + notifica ISP
4. Caso GenAI: spinner durante estrazione
5. Post-estrazione: **N selettori multipli di tipologia covenant**, ognuno con scadenziere proposta GenAI
6. Pulsanti `Modifica Dati` + `Conferma Assegnazione`
7. `Conferma` → toggle a `Modifica Assegnazione`
8. Dopo conferma → divider + testo + `Aggiungi Covenant #n` + `Concludi Assegnazione`

**Workflow Manuale** (riga 652-716):
1. Bottone `Aggiungi Covenant`
2. Riga covenant con cestino per eliminarla
3. 3 box: dropdown "Tipologia Covenant" (formato `[ID] - [Tipologia]`), input "Valore Monitoraggio", date picker "Data di Riferimento"
4. Bottoni `Associa Covenant` (disabilitato fino a 3 box compilati) + `Annulla`
5. `Associa Covenant` apre **scadenziere filtered** con 9 colonne (ID Evento, Data Rif, Scadenza, Giorni, Data Avv, Valore, Status, Compliance Cert link, Azioni)
6. **Filtro eventi scadenziere**: stati `Scaduto|Fuori Soglia|In Scadenza` OR data riferimento = data input OR data riferimento ± 15 giorni dalla data input
7. Click `Assegna` su riga → appare `Conferma Assegnazione` sotto
8. Click `Conferma Assegnazione` → appare `Concludi Assegnazione` in alto
9. Click `Concludi` → torna a schermata documenti, status documento `Completato`, bottone diventa `Visualizza Assegnazione`
10. Post-conferma scadenziere mostra solo riga assegnata, azione diventa `Modifica Assegnazione` (riapre selettori)
11. Vincolo: `Concludi Assegnazione` cliccabile solo se TUTTI i covenant aggiunti sono confermati

### Componente esistente (insufficiente)
- `CcAssignmentComponent` (`practice-detail/cc-assignment/`) e' solo wizard 4-step in dialog form. NON copre niente del flusso BR sopra.
- `MonitoringGenAiExtractionComponent` (`practice-detail/monitoring-genai-extraction/`) e' il dialog GenAI corrente (form 2-colonne). Anche questo va sostituito dal page-based.

### Stima reale: 4-6h con tests
- 1-2h: scaffold nuovo route + nuova page component + preview documento + box selezione GenAI/Manuale
- 1-2h: implementazione flow GenAI (N selettori + scadenziere + Modifica Dati toggle)
- 1-2h: implementazione flow Manuale (Aggiungi Covenant → scadenziere filtered ± 15gg)
- 1h: bad flow modal + test + i18n + smoke

### Strategia consigliata prossima sessione
1. Sessione dedicata 4-6h (no interruzioni)
2. Iniziare da: lettura completa BR sez 4.2.3 + Mockup slide 17-24
3. Creare nuovo `MonitoringCcAssignmentPageComponent` standalone con route lazy `/{role}/monitoring/practice/{id}/cc/{documentId}`
4. Riusare logica `MonitoringGenAiService.applyExtraction` BE (gia chiama `eventService.updateValue` che fa recalc — bug #18 gia coperto)
5. Migrare cleanup `CcAssignmentComponent` + `MonitoringGenAiExtractionComponent` (eliminare dopo verify nuovo flow)

## Fixture DB locale M1 post-bug-fix round 2

```sql
-- Pratica M1 (id 8288): MONITORING_FUORI_SOGLIA (✓ fixato bug #18)
-- DEAL-M1-001 / Starhotels Finanziaria / CONTRACT-M1-001 / booking_tracking_id=1234 (✓ fixato bug #2)
SELECT id, code, deal_id, deal_name, contract_key, booking_tracking_id,
       (SELECT code FROM mp_psm_state WHERE id=p.state_id) AS state
FROM mp_practice p WHERE id=8288;
-- Output:
-- 8288 | M1 | DEAL-M1-001 | Starhotels Finanziaria | CONTRACT-M1-001 | 1234 | MONITORING_FUORI_SOGLIA

-- Eventi M1C1 (post-recalc bug #18 fix):
-- E1: OK (has_exception=1, era gia), exp=2026-05-28
-- E2: OK (has_exception=1, NUOVO da smoke "Salta Monitoraggio"), value=100000, exp=2026-08-27
-- E3: OK (COVNO), value=50, exp=2026-11-23
-- E4: FUORI_SOGLIA (COVNO, no exception), value=80, exp=2027-02-23
-- E5: NESSUNO_STATO, exp=2027-05-23
-- E6: NESSUNO_STATO, exp=2027-08-23
```

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
# Atteso BE HEAD: c4efb8e

cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3
# Atteso FE HEAD: 1d27047 (invariato)

# Sanity check compile
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw clean compile -DskipTests -q
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck

# Avvio BE (profile local obbligatorio)
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw spring-boot:run -DskipTests -Dspring-boot.run.profiles=local
# Wait health 200
curl -s http://localhost:8091/actuator/health

# Avvio FE
cd C:/Users/davmelis/Documents/Github/ba-web && npm run start
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200

# Stato fixture M1 post-bugfix round 2
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT p.id, p.code, p.deal_id, p.deal_name, p.contract_key, p.booking_tracking_id, ps.code AS state FROM mp_practice p JOIN mp_psm_state ps ON ps.id=p.state_id WHERE p.id IN (8265, 8288);"
```

## Utenti DB locale (invariato)

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`) — password sessione
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`) — password sessione
- Login richiede TOTP via Google/Microsoft Authenticator

## Prossima sessione — Action plan

### Opzione A — Sessione dedicata bug #10 (consigliata, 4-6h)
1. Read BR sez 4.2.3 completo + Mockup_Monitoraggio_v10 slide 17-24
2. Read `CcAssignmentComponent` + `MonitoringGenAiExtractionComponent` per migration logic
3. Scaffold nuovo `MonitoringCcAssignmentPageComponent` standalone con route lazy
4. Implementare flow GenAI (N selettori + scadenziere)
5. Implementare flow Manuale (Aggiungi Covenant + scadenziere filtered)
6. Bad flow modal rifiuto + i18n + smoke + commit FE

### Opzione B — Cluster bug MEDIUM piccoli (60-90min)
Bug MEDIUM piccoli da fare in serie con commit atomici:
- #5 dropdown Variabile SI/NO
- #6 PUT covenant regen events
- #8 Excel enum prefix
- #9 Mock GenAI label/enum
- #11 spinner upload AD Form
- #12 toast validation file upload
- #15 popup ISP "Chiudi"
- #17 cella Azione expired/expiring (refactor con DynamicComponentModel — medio)

Poi sessione dedicata bug #10.

### Opzione C — Sessione retest E2E + UAT
- Dopo aver chiuso almeno P1 #10 + cluster P2: retest completo Playwright STEP 1-10
- UAT funzionale con stakeholder (richiede dati realistici + scenario testing)
- Code review finale BE + FE (Davide BE / Carmine FE)

## Riferimenti

- Handoff predecessore: `handoff/HANDOFF_2026-05-28_step10_smoke_done_bugfix_round.md`
- Plan completo: `PLAN.md` (T-024 al 80%, manca bug-fix round + retest + UAT + code review)
- BR sez 4.2.3 (page-based UX bug #10): `requirements/BR_Monitoraggio_V6.md` riga 652-744
- BR sez 8.4.3 (semantica Tracking ID bug #2): `requirements/BR_Monitoraggio_V6.md` riga 218
- Commit BE bug #18: `9ef5ede` (MonitoringStateService.java + MonitoringGenAiControllerTest.java)
- Commit BE bug #2: `c4efb8e` (4 file: view SQL + entity + 2 service mapper)
- Componenti BE chiave bug #18: `MonitoringStateService.java`, `MonitoringEventService.java` (entry points), `CovnoUploadService.java`, `MonitoringGenAiService.java`
- Componenti BE chiave bug #2: `MonitoringPracticeSummaryView.java`, `MonitoringDashboardService.java`, `MonitoringPracticeService.java`, `001_view_monitoring_practice_summary.sql`
- Componenti FE da rifare bug #10: `CcAssignmentComponent`, `MonitoringGenAiExtractionComponent`
