# Handoff Smoke E2E T-024 — 2026-05-26

Sessione AI chiusa per limite context (~80%). Riapertura su sessione pulita.

## Stato sessione

- **STEP 1-6 chiusi.** Confermati dall'utente durante smoke E2E.
- **STEP 7-10 pending**: GenAI Deloitte, COVNO upload, Mailing list, Expired/Expiring pages.

## Branch

- BE: `feature/monitoring-integration-uat` HEAD = `25ba16f` (pushato)
- FE: `feature/monitoring-integration-uat` HEAD = `c9ba5cd` (pushato)
- Profilo: `main` HEAD = `9436ab3` (pushato, PROGRESS.md aggiornato — dovra' essere riaggiornato post-questo-handoff)

## Fixture DB locale (M1)

```
Pratica M1 (id 8288): MONITORING_OK, termination_date=2027-12-31
  Covenant M1C1 (id 1): LOAN_TO_VALUE, LTE 70, EBITDA, TRIMESTRALE
    Event M1C1E1 (id 1): SCADUTO -> OK (con eccezione), motivation="Sto testando"
      Document (id 1): ok_Starhotels...pdf (155KB, file_reference=NULL)
    Event M1C1E2..E7: NESSUNO_STATO (futuri)
```

Per testare flusso Salta/Ignora ISP serve loggarsi come **utente banca** (es. davide94melis@gmail.com).

## STEP completati questa sessione (2026-05-25 + 2026-05-26)

### STEP 3 Dashboard Monitoraggio (3 fix critici)
- Crash JS `mapPracticeToRow undefined toString` — contratto DTO BE/FE allineato (BE commit `888e704`).
- 6 stati Monitoraggio nel widget SACE dashboard `/user` — migration 009 sposta i 6 stati a phase MONITORING (BE commit `888e704`, FE commit `bdc195a`).

### STEP 4 Tabella Pratiche Monitoraggio (9 fix critici)
- Sort camelCase native query, is_archived='NO', semaphoreStates senza prefix, export GET vs POST, MonitoringCovenantDTO mancanti 5 campi, Dashboard filtri ignorati, status label vs enum code, export dashboard 404, filtro Tracking ID `=` vs `LIKE`, IN clause SQL multi semaforo, sfalsatura colonne (BE commits `0df3952`, `b2332de`; FE commit `573f1d5`).

### STEP 5 Dettaglio Pratica
- Carica correttamente, no bug.

### STEP 5b UI Aggiungi Covenant (commit FE `453346c`)
- Dialog form Reactive con dropdown Tipologia/Segno/Periodicita' + Valore Limite + Variabile + Note + endpoint createCovenant FE + 3 enum FE subset 10/58 + i18n IT/EN 17 chiavi. Deviazione documentata vs mockup full-page.

### STEP 5b post-creazione (4 fix)
- Generator 0 eventi se termination_date passata — `resolveStartDate` parte da `creation_date` (BE commit `1c91c66`) + DB locale M1 termination_date=2027-12-31.
- Button Dettaglio Covenant non cliccabile (FE commit `df82634`).
- i18n `monitoring.documents.history.*` + `actions` mancanti (stesso commit).
- Dialog vs mockup full-page documentato come T-024.x.

### STEP 5c Modifica Dati Covenant full-page (BR sez. 4.2.5)
- 3 endpoint BE event CRUD: `POST /covenants/{id}/events`, `PUT /events/{id}`, `DELETE /events/{id}` (BE commit `25ba16f`).
- Pagina dedicata `/{role}/monitoring/practice/:id/covenant/:covenantId/edit` con form covenant + tabella scadenziere inline-editable + Aggiungi/Rimuovi/Modifica monitoring (FE commit `c3a90ac`).
- Fix routing relativo + semaforo NESSUNO_STATO schedule-tab (FE commit `8cfda71`).
- i18n `monitoring.documents.upload.dragDrop` mancante (FE commit `67aecf5`).

### STEP 6 Gestione Eccezioni (commit FE `f714585` + `c9ba5cd`)
- Flusso end-to-end testato: ISP "Salta Monitoraggio" → request exception con motivazione+file → DB updated → Deloitte "Visualizza Eccezione" mostra read-only → "Annulla Eccezione" ricalcola status.
- Aggiunta sezione "Documenti allegati" nel dialog Visualizza Eccezione (BR sez. 4.2.6).
- `getExceptionDocuments` rinominato da singolare a plurale + ritorna `List<MonitoringDocumentModel>` JSON (fix bug latente sweep STEP 4).
- Fix build pTooltip → title nativo.

## Bug noti residui per STEP 7+

1. **`file_reference=NULL` per document eccezione** — il BE `requestException` salva solo metadati (filename+size) NON il binario. Download fallirebbe. Da implementare: storage filesystem o BLOB DB + endpoint `GET /events/{id}/exception/documents/{docId}/download`. Gap T-024.x.
2. **`mp.deal_id`+`mp.contract_key` NULL su M1** — `createFromPostClosing` non copia questi campi dalla pratica padre. Da fixare in T-024.
3. **`[object Object]` nella colonna "Caricato da" della document-history** (visto in screenshot STEP 6) — probabile binding sbagliato `uploadedBy` come oggetto invece di stringa. Da investigare.
4. **CovenantTypeNumericEnum FE limitato a 10/58 tipologie** — da migrare a endpoint API dedicato per evitare drift.
5. **Field "Variabile" e' input text** invece di dropdown SI/NO come BR sez. 4.2.5 — dominio BE espone come String generica.
6. **PUT covenant non rigenera eventi** se cambia `periodicity` — chiamare `scheduleGeneratorService.regenerateEvents()`.
7. **`MailingListAdForm.state`** mai persistito in `VERIFICA_IN_CORSO` (overwritten subito a `REPORT_GENERATO`).
8. **`MonitoringPracticeExcelWriterService`** ha stesso bug `Enum::name` senza prefix MONITORING_ sui semaphoreStates filter.

## Azione immediata per nuova sessione

1. **Resume**: leggere questo HANDOFF + PROGRESS.md (sezione 2026-05-26).
2. **Verifica BE+FE running**: `curl -s http://localhost:8091/actuator/health` + `http://localhost:4200` raggiungibile.
3. **STEP 7 GenAI Deloitte** (BR sez. 4.2.4):
   - Logga come **Deloitte** su `/consultant/monitoring/practice/8288`.
   - Tab Covenant → upload "Compliance Certificate" su M1C1E2 (evento futuro).
   - Click "Estrai con GenAI" (icona `pi-bolt`) sull'evento appena caricato (status IN_ATTESA_CONFERMA o VERIFICA_IN_CORSO).
   - Atteso: stato passa a GENAI_IN_CORSO (mock async ~5s), poi GENAI_DA_VALIDARE.
   - Click "Modifica Dati" (icona `pi-pencil`) → dialog GenAI extraction mostra 9 chiavi mock `COVENANT_NUM_*` (vedi T-018 in PROGRESS).
   - Click "Conferma e Aggiorna" → applyExtraction PATCH covenant+event con i valori GenAI.
   - Verifica DB: covenant + event aggiornati con i valori mock.

## Checklist smoke E2E (stato corrente)

| # | Step | Stato |
|---|---|---|
| 1 | Verifica fix StatesOptions su dashboard pratiche | DONE |
| 2 | Generare 1 pratica monitoring fixture (M1 / id 8288) | DONE |
| 3 | Dashboard monitoring (donut + 3 tabelle) | DONE |
| 4 | Tabella pratiche monitoring + filtri + export | DONE |
| 5 | Dettaglio pratica monitoring ISP (3 tab) | DONE |
| 5b | Aggiungi Covenant + 7 eventi auto-generati | DONE |
| 5c | Modifica Dati Covenant full-page | DONE |
| 6 | Eccezioni (request/skip/visualizza/annulla) role-gated | DONE |
| 7 | GenAI Deloitte (extract -> modify -> apply) | PENDING — STARTING POINT NUOVA SESSIONE |
| 8 | COVNO upload + storico | PENDING |
| 9 | Mailing list (CRUD + AD Form reject) | PENDING |
| 10 | Expired/expiring events pages | PENDING |

## Comandi rapidi per ripartire

```bash
# Verifica branch + commit
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3

# Stato DB M1
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT event_code, status, has_exception, exception_motivation FROM mp_monitoring_event WHERE covenant_id=1;"

# BE+FE health
curl -s http://localhost:8091/actuator/health
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200
```
