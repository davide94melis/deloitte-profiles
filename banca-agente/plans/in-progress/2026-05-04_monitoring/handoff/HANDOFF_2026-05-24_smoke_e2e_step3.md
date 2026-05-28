# Handoff Smoke E2E T-024 — 2026-05-24

## Stato sessione

Smoke E2E completo del monitoring in corso, sospeso a **STEP 3/10** per context limit
della sessione AI. Tutto il lavoro fixato e committato. Branch BE+FE pushati.

## Branch

- BE: `feature/monitoring-integration-uat` HEAD = `a5ed8ff` (pushato origin)
- FE: `feature/monitoring-integration-uat` HEAD = `b396515` (pushato origin)
- Pratica fixture monitoring nel DB: `M1` (id 8288) in stato `MONITORING_OK`,
  generata da `C_8265` (id 8265) via trigger `createFromPostClosing`.

## Fix applicati in questa sessione

### FE (commit `ebfca35` + `b396515`)

1. **`ebfca35`** — fix StatesOptions: 16 stati nuovi a `PsmStateEnum`/`StatesOptions`
   (3 SACE_GENAI_*, 2 BOOKING_GENAI_*, 4 POST_CLOSING_GENAI_*, 7 MONITORING_*).
   Defense `?.` ai 7 punti di accesso. Risolve crash dashboard pratiche principale.
2. **`b396515`** — i18n IT+EN: 16 chiavi `psmState` mancanti per gli stati sopra +
   refactor filtro fase in `practice-filters.component.ts:109` da `startsWith(e.value)`
   a `StatesOptions[v]?.phase === e.value` (risolve bug filtro SACE).

### BE (commit `a5ed8ff`)

1. `createFromPostClosing`: aggiunto `setCreationDate(LocalDateTime.now())`.
2. 7 controller monitoring: rimosso prefisso `/api/v1/` da `@RequestMapping`
   (allineato al FE che chiama `/monitoring/...`).
3. View `vw_monitoring_practice_summary` + `vw_census_practice_summary`:
   discrimine cambiato da `service_item_id` a `mps.code LIKE/NOT LIKE 'MONITORING%'`.
4. `MonitoringDashboardService` + `MonitoringPracticeService`: strip del prefisso
   `MONITORING_` prima di `MonitoringPracticeStateEnum.valueOf()` (l'enum ha valori
   `OK/IN_SCADENZA/...` senza prefisso).

### DB locale

- Migration 008 PSM ENUM gia' applicata in sessioni precedenti.
- View `vw_monitoring_practice_summary` (e altre 3) gia' aggiornate manualmente
  via `mysqlsh --file=...` dal repo. **Da aggiornare in dev/UAT/prod** in
  contestualmente al deploy del commit `a5ed8ff` (BE non riapplica le view all'avvio).

## Checklist smoke E2E (stato corrente)

| # | Step | Stato |
|---|---|---|
| 1 | Verifica fix StatesOptions su dashboard pratiche | DONE |
| 2 | Generare 1 pratica monitoring fixture (M1 / id 8288) | DONE |
| 3 | Dashboard monitoring (donut + 3 tabelle) | IN CORSO — ultimo fix `a5ed8ff` da verificare con BE rebuild |
| 4 | Tabella pratiche monitoring + filtri + export | PENDING |
| 5 | Dettaglio pratica monitoring ISP (3 tab) | PENDING |
| 6 | Eccezioni (request/skip/ignore) role-gated | PENDING |
| 7 | GenAI Deloitte (extract -> modify -> apply) | PENDING |
| 8 | COVNO upload + storico | PENDING |
| 9 | Mailing list (CRUD + AD Form reject) | PENDING |
| 10 | Expired/expiring events pages | PENDING |

## Azione immediata per la prossima sessione

1. **Rebuild + riavvia BE** (IntelliJ stop+run del `BackEndApplication`) per applicare
   le modifiche di `a5ed8ff` su `MonitoringDashboardService` + `MonitoringPracticeService`
   + 7 controller monitoring.
2. **Hard refresh** della dashboard monitoring (Ctrl+F5) su
   `http://localhost:4200/user/monitoring/dashboard` con utente Deloitte/Consultant.
3. **Atteso**:
   - Totale: **1**
   - Donut: 1 segmento verde **OK** (#4CAF50)
   - Tabella "Pratiche Recenti": riga **M1**
   - Console pulita, no 500.
4. Se OK, marcare STEP 3 DONE e passare a STEP 4 (tabella pratiche monitoring
   completa: filtri, paginazione, export Excel).

## Fixture utili nel DB locale

- M1 (8288): pratica monitoring MONITORING_OK, NO covenant / NO event / NO document.
  Per testare STEP 5+ (dettaglio + eccezioni + GenAI) serve creare almeno 1
  covenant + qualche event sotto M1 (via UI o seed SQL).
- 22 pratiche `POST_CLOSING_UPLOADED` rimaste se servono altre fixture monitoring
  (es. per testare gli altri 5 stati semaforo).

## Bug noti residui (fuori scope STEP 3)

- **AGENCYDESK_STATES** include 9 stati GENAI nuovi (commit `ebfca35`) — verificare
  che il refactor non rompa `isAgencyDeskStateEqOrAfter`/`Before` su flussi pre/post.
- **POST_CLOSING_GENAI_*** mappati nel FE ma 0 pratiche nel DB li usano (defensive
  only).
- View DDL: nessun meccanismo (Flyway/Liquibase) li riapplica al deploy. Documentare
  in CONTRIBUTING.md o automatizzare a startup.

## Comandi rapidi per ripartire

```bash
# Verifica branch + commit
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3

# Verifica DB
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT practice_id, practice_code, practice_state FROM vw_monitoring_practice_summary;"
# Atteso: 1 riga -> M1, MONITORING_OK

# Verifica BE+FE
curl -s http://localhost:8091/actuator/health
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200
```
