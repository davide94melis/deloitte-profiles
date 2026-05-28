# Handoff STEP 9 DONE → STEP 10 Expired/Expiring Events — 2026-05-28

Sessione AI chiusa dopo STEP 9 (Mailing list) PASS completo con coverage 100% PR #29 Georgios + 4 sub-step extra (Visualizza Report, CRUD contatti, popup ISP Aggiorna Documento, Annulla Richiesta). No nuovi commit BE/FE in questa sessione — solo smoke E2E + bug list update + documentazione. Nuova sessione consigliata per STEP 10 (ultimo step del piano monitoring).

## Branch state

- BE `feature/monitoring-integration-uat` HEAD = **`0de244a`** (invariato dalla sessione precedente — PR #29 gia mergiato)
- FE `feature/monitoring-integration-uat` HEAD = **`3f27c1b`** (invariato — PR #29 gia mergiato)
- Profilo `deloitte-profiles` HEAD = nuovo commit `<da fare>` con PROGRESS.md STEP 8 + STEP 9 + questo handoff

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
| 7 | GenAI Deloitte (extract → modify → apply, refactor DeferredResult) | DONE |
| 8 | COVNO upload + storico | DONE |
| 9 | Mailing list (CRUD + lifecycle completo + reject) | **DONE** — 100% coverage PR #29 |
| 10 | Expired/Expiring events pages | **PENDING — STARTING POINT NUOVA SESSIONE** |

## STEP 9 — Cosa e stato fatto in questa sessione (riepilogo)

### Sanity check post-merge PR #29
- BE `./mvnw clean compile -DskipTests -q`: PASS exit 0
- FE `npx tsc --noEmit --skipLibCheck`: PASS exit 0
- FE `MailingListPracticeFilterModel` allineato 1:1 a BE `MailingListPracticeFilterDTO` (dealName/censimentoId/trackingId/monitoringId + pageable)
- DB pre-smoke: cancellato record `mp_mailing_list_ad_form` residuo M1 (1 record DOC_INCOMPLETA da fixture pregressa) per partire pulito

### State machine end-to-end attraversata
```
∅ → DOC_COMPLETA → DOC_CONFERMATA → VERIFICA_IN_CORSO → REPORT_GENERATO ✓ (happy path)
∅ → DOC_COMPLETA → DOC_INCOMPLETA ✓ (reject path via "Annulla Richiesta" ISP popup,
                                       stesso endpoint del "Rifiuta" Deloitte detail)
```

### Nuovi endpoint Georgios PR #29 verificati al 100%
- `POST /monitoring/mailing-list/practices` — search dedicato no piu N+1. Tutti 4 filtri testati.
- `POST /practices/{id}/ad-form/conclude` — idempotente, transizione `VERIFICA_IN_CORSO|REPORT_GENERATO → REPORT_GENERATO`. Nav `../` post-success OK.
- `MonitoringPracticeSummaryDTO.adFormState/FileName/UploadDate` inline — badge severity renderizzati correttamente in lista.
- File upload accept estesi (rifiuto `.txt` verificato, vedi bug #12). maxFileSize 3MB non testato (skip — comportamento ereditato componente PrimeNG identico).

### CRUD contatti mailing list
- Create: 2 contatti aggiunti dal detail Deloitte
- Update: role di un contatto modificato + verificato in DB
- Delete: 1 contatto eliminato + verificato in DB
- Tutto via UI con toast + (per delete) confirm dialog. OK.

### Sub-step extra (4 nice-to-have completati)
1. **Visualizza Report** (azione lista su `REPORT_GENERATO`) → naviga al detail con sezione extra "Report Contatti Estratti" (placeholder output GenAI, oggi mostra gli stessi contatti CRUD).
2. **CRUD contatti** (vedi sopra).
3. **Popup ISP "Aggiorna Documento"** → sostituzione file: nuovo record `ad_form` id+1 `DOC_COMPLETA`, il `findFirstByPracticeIdOrderByUploadDateDesc` prende il piu recente come visible/attivo.
4. **Popup ISP "Annulla Richiesta"** → confirm dialog → `rejectAdForm` → `DOC_INCOMPLETA`. Riga resta in sezione 2 con badge Danger "Documentazione Incompleta".

## Bug noti residui aggiornati (15 totali)

### Bug rimasti da sessioni pregresse (10)
1. **file_reference=NULL document eccezione** — gap T-024.x, storage binario non implementato per requestException
2. **mp.deal_id/contract_key NULL su M1** — `createFromPostClosing` non copia. CONFERMATO ancora aperto in STEP 9 (ID Censimento/Tracking ID vuoti nella lista mailing list)
3. **[object Object] colonna "Caricato da"** — probabilmente risolto da Alexios refactor (da riverificare)
4. **CovenantTypeNumericEnum FE 10/58** — drift
5. **Field "Variabile" input text vs dropdown SI/NO**
6. **PUT covenant non rigenera eventi se cambia periodicity**
7. **MailingListAdForm.state mai persistito VERIFICA_IN_CORSO** — CONFERMATO non risolto strutturalmente: `extractAdForm` placeholder setta VERIFICA_IN_CORSO poi sovrascrive immediatamente a REPORT_GENERATO nello stesso TX (visibilita microsecondi). Il nuovo `concludeAdForm` Georgios accetta entrambi gli stati, quindi e gia pronto per quando GenAI vera sara async. By-design placeholder, non blocker.
8. **MonitoringPracticeExcelWriterService enum prefix bug**
9. **Mock GenAI label vs enum BE** — Covenant fields non aggiornati da applyExtraction
10. **BR 4.2.3 page-based UX** — refactor dialog Modifica Dati → pagina dedicata + N selettori + scadenziere inline (~2-3h)

### Bug NUOVI rilevati STEP 9 (5, tutti UX non bloccanti)
11. **Upload AD Form senza feedback visivo (no spinner/progress)** — MEDIUM. Fix: wrappare la call con `SpinnerComponent.showLoader()/hideLoader()` come pattern STEP 7.
12. **Validation formato/size file upload mostra solo 2 icone X rosse senza messaggio testuale** — MEDIUM. Fix: hook su `onError`/`onValidationError` PrimeNG → toast tipo "Formato non supportato. Accettati: PDF, DOC, DOCX, XLSX, XLS, MSG, P7M" oppure "File troppo grande (max 3MB)".
13. ~~Stage progress evidenzia tutti gli step inclusi quelli mai attraversati~~ — **NON e bug**: comportamento atteso (mostra l'intera state machine possibile, non il path reale). RIMOSSO da bug list.
14. **Bottone "Conferma e Aggiorna" still visible nel detail mailing-list Deloitte anche in REPORT_GENERATO** — LOW (idempotente, no rottura). Fix: gate `*ngIf="adForm.state !== REPORT_GENERATO"` o cambia label in "Scarica Report".
15. **Popup ISP "Visualizza Richiesta Aggiornamento" senza bottone Chiudi ne X** — MEDIUM. User intrappolato, deve fare "Aggiorna Documento" o "Annulla Richiesta". Fix: aggiungere bottone "Chiudi" neutrale tra i due esistenti, oppure `[closable]="true"`.

## Fixture DB locale M1 post-smoke (id 8288, MONITORING_OK, termination_date=2027-12-31)

```
Pratica M1 (id 8288): MONITORING_OK
  Covenant M1C1 (id 2): LOAN_TO_VALUE, LTE 70, EBITDA, TRIMESTRALE
    Event M1C1E1 (id 1): OK (con eccezione), actual_date=2026-05-26
      Document (id 3): COMPLETATO, file_reference=NULL
    Event M1C1E2 (id 2): FUORI_SOGLIA (GenAI applied), value=150000, actual=2026-05-27
      Document (id 4): COMPLETATO (GenAI applied)
    Event M1C1E3 (id 3): OK (COVNO), value=50, actual=2026-11-23
    Event M1C1E4 (id 4): FUORI_SOGLIA (COVNO), value=80, actual=2027-02-23
    Event M1C1E5..E6: NESSUNO_STATO (futuri)

Mailing list AD Form M1 (post STEP 9 smoke):
  id=3 DOC_COMPLETA (storicizzato, override da id=4 piu recente)
  id=4 DOC_INCOMPLETA (visible/attivo via findFirstOrderByUploadDateDesc)
  → UI mostra badge "Documentazione Incompleta" Danger

Mailing list contacts M1: 0 (cancellati durante test delete)
```

### Note importanti su fixture
- Per ripartire pulito su STEP 10 NON serve toccare ad_form/contacts (STEP 10 e su events scaduti/expiring, indipendente da mailing list).
- Se vuoi rifare un round STEP 9 da capo: `DELETE FROM mp_mailing_list_contact WHERE practice_id=8288; DELETE FROM mp_mailing_list_ad_form WHERE practice_id=8288;`

## Utenti DB locale

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`)
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`)

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3

# Pull eventuali nuovi commit Georgios/Alexios pre-STEP 10
cd C:/Users/davmelis/Documents/Github/ba-back-end && git pull
cd C:/Users/davmelis/Documents/Github/ba-web && git pull

# Sanity check post-pull (se ci sono nuovi commit)
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw clean compile -DskipTests -q
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck

# BE+FE health
curl -s http://localhost:8091/actuator/health
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200

# Stato eventi M1 per STEP 10 (filtraggio scaduti/expiring)
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT id, code, status, reference_date, expiration_date, actual_date, value, has_exception
   FROM mp_monitoring_event WHERE covenant_id=2 ORDER BY reference_date;"
```

## STEP 10 — Action item nuova sessione

1. **Legge questo handoff** + ultimo blocco PROGRESS.md (data 2026-05-28)
2. **Pull eventuali nuovi commit Georgios/Alexios** su `feature/monitoring-integration-uat` BE+FE (verificare PR aperte/mergiate dopo 2026-05-27)
3. **Sanity check post-pull**: `mvn compile` BE + `tsc --noEmit` FE
4. **Restart BE+FE** (utente)
5. **Identifica nel BR le sez. relative a Eventi Scaduti / In Scadenza**:
   - Probabili pagine dedicate (sidebar/dashboard?) — verificare se sono nuove pagine o sezioni di pagine esistenti
   - Check route FE: probabilmente sotto `/{role}/monitoring/expired-events` o `/{role}/monitoring/expiring-events` (da grep)
   - Identificare endpoint BE relativi (probabilmente filtri sul `mp_monitoring_event` o `vw_monitoring_practice_summary`)
6. **Procedi con STEP 10 smoke E2E** secondo flusso BR:
   - Test fixture: M1C1E1 (OK con eccezione 2026-05-26) + M1C1E2 (FUORI_SOGLIA 2026-05-27) + M1C1E4 (FUORI_SOGLIA 2027-02-23 COVNO) — gli eventi scaduti esistono gia
   - Verificare aggregazioni cross-pratica nelle pagine dedicate
   - Verificare action per "skip" / "ignora soglia" / "richiedi eccezione" se previste anche da queste viste

## Riferimenti

- Handoff predecessore: `handoff/HANDOFF_2026-05-27_step8_done_step9_mailing.md`
- Plan completo: `PLAN.md` (STEP 10 = ultimo step del piano monitoring)
- BR sezione di riferimento per STEP 10: **da identificare nel BR** (probabili sez. 5.x "Dashboard Monitoring" o sez. 8.x "Operativita continua")
