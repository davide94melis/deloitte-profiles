# Handoff BUG-FIX ROUND 3 — DONE — 2026-05-30

Sessione successiva = **retest E2E completo + UAT funzionale + code review finale** prima di archiviare T-024 e chiudere il milestone monitoring. **Tutti i 13 bug aperti del Round 3 sono stati chiusi end-to-end in singola sessione** seguendo l'ordine raccomandato dal handoff predecessore (`HANDOFF_2026-05-30_bugfix_round_3_plan.md`).

Branch BE+FE `feature/monitoring-integration-uat` allineati con remote dopo push finale (BE HEAD `40f6c4f`, FE HEAD `4288056`). Bug cumulativi: **18/18 fixati**, eccetto #7 by-design.

---

## Cosa e' stato chiuso (per cluster nell'ordine eseguito)

### Cluster C — Enum & labels (3 bug, BE+FE) — chiuso

| Bug | Commit | Modifiche chiave | BR |
|-----|--------|------------------|----|
| #9 mock GenAI label non-enum | `1a5a5b2` BE | `AsyncMonitoringGenAiProcess`: tipologia `Esempio Covenant Mock` → `LOAN_TO_VALUE`, segno `"<="` → `"LTE"`, variabile `EBITDA` → `"Si"`. Periodicita gia' allineata. | 4.2.3 |
| #8 Excel writer enum prefix | `8ee89d5` BE | `MonitoringPracticeExcelWriterService.prettyPracticeState()` helper statico: `PsmStateCodeEnum.valueOf(code).getText()` con fallback al raw. Cell "Semaforo" ora mostra "Monitoraggio - Fuori Soglia" invece di `MONITORING_FUORI_SOGLIA`. | 8.4 |
| #4 CovenantType endpoint BE-driven | `db9254d` BE + `9c1645f` FE | Nuovo `EnumOptionDTO` + endpoint `GET /monitoring/covenant-types` (53 typologies, `code` + `label = enum.getText()`). FE `MonitoringService.getCovenantTypes()` consume + `covenant-edit-page` / `covenant-add-dialog` fetcha al ngOnInit. Enum FE marcato `@deprecated`. | 4.2.5 |

### Cluster G — Smoke finding #19 (1 bug, FE) — chiuso

| Bug | Commit | Modifiche chiave | BR |
|-----|--------|------------------|----|
| #19 documentInfo stale post-GenAI | `bd47922` FE | `monitoring-cc-assignment-page.component.ts`: nuovo `refreshDocumentInfo()` invocato dopo `extraction.status === READY_FOR_VALIDATION`. Riusa `getDocumentsByPractice` filtrato per `documentId` (no nuovo endpoint BE). Il getter `showRejectButton` reagisce immediatamente. | 4.2.3 |

### Cluster E — Covenant detail UX (1 bug, FE) — chiuso

| Bug | Commit | Modifiche chiave | BR |
|-----|--------|------------------|----|
| #5 Variabile dropdown SI/NO | `ed2a895` FE | `covenant-edit-page` + `covenant-add-dialog`: `<input pInputText>` → `<p-dropdown>` opzioni `{Si=Si, No=No}` (capitalize allineato a BR 1.1 r127 + mock #9). Nuova chiave `editPage.variablePlaceholder`, ridenominata `add.variablePlaceholder` ("Es. EBITDA / PFN" → "Si o No"/"Yes or No") IT+EN. | 1.1 + 4.2.5 |

### Cluster D — Mailing list UX (4 bug in 2 commit, FE) — chiuso

| Bug | Commit | Modifiche chiave | BR |
|-----|--------|------------------|----|
| #11 + #12 + #15 | `893a1f7` FE | (#11) `submitAdForm` + `onUpdateDocument` wrappati con `uploading` + spinner UI in entrambi i dialog ISP. (#12) shared `app-file-upload` espone anche `invalidFileSizeMessageSummary/Detail`, wire dei 4 messaggi su entrambi i dialog ISP, nuovo namespace i18n `monitoring.mailingList.uploadFeedback.*` (IT+EN). (#15) popup "Visualizza Richiesta Aggiornamento" reso chiudibile (X + bottone Chiudi), locked solo durante upload. | 6.3 |
| #14 Conferma hidden in REPORT_GENERATO | `b6eaa32` FE | `mailing-list-detail.component.html`: rimosso branch `|| REPORT_GENERATO` dal `*ngIf` del bottone "Conferma e Aggiorna", visibile solo per VERIFICA_IN_CORSO. La sezione Report resta visibile in REPORT_GENERATO via `showReportSection`. | 6.3 r962 |

### Cluster F — Expired/Expiring action cell (1 bug, BE+FE) — chiuso

| Bug | Commit | Modifiche chiave | BR |
|-----|--------|------------------|----|
| #17 cella Azione clickable | `a38aa32` BE + `9436e06` FE | BE: aggiunti `eventId`, `practiceId`, `hasException` a `MonitoringDashboardEventDTO` + popolati in `toEventDto`. FE: `mapEventToRow` refactor con `buildActionCell` allineato a `practice-detail.buildActionCell`, role-gating (ISP no exception → Salta Monitoraggio/Ignora Soglia; ISP has exception → Visualizza read-only; Deloitte has exception → Annulla; Deloitte no exception → no button). `MonitoringExceptionDialogModule` importato in entrambi i module, dialog hostato con submit/cancel handlers che ricaricano la pagina corrente. 3 spec event-builder estesi. | 8.4.3 + 8.5 + 8.4 |

### Cluster B — Schedule regenerator (1 bug, BE) — chiuso

| Bug | Commit | Modifiche chiave | BR |
|-----|--------|------------------|----|
| #6 PUT covenant non rigenera | `df7f3fe` BE | `MonitoringScheduleGeneratorService.regenerateFutureEvents(covenant)`: preserva monitorati (`actualDate != null` o `hasException`), cancella placeholder futuri, riparte numerazione, walk da `lastMonitoredRefDate + nuovo intervallo` fino a `terminationDate`. `MonitoringCovenantService.update` cattura `previousPeriodicity` PRIMA dell'assegnazione e invoca regenerate solo se cambia. | 1.1 r142-149 + 4.2.5 |

### Cluster A — Documenti & metadata (2 bug, BE+FE) — chiuso

| Bug | Commit | Modifiche chiave | BR |
|-----|--------|------------------|----|
| #1 file_reference NULL eccezione | `40f6c4f` BE + `6f0b8fc` FE | BE: `requestException` carica binario via `documentManagerService.uploadDocument()` (helper inline `uploadExceptionAttachment` per evitare circular dep), persiste `fileReference` + `uploadedBy = user.getUsername()`. FE `exception-dialog`: info-icon arancione "non disponibile" → bottone download reale (`monitoringService.downloadDocument(doc.id)` + blob save), disabled se fileReference vuoto. Nuova i18n `download`, ripulito `downloadNotAvailable`. | 4.2.6 + sez. 3.4 |
| #3 [object Object] Caricato da | `4288056` FE | Helper difensivo `formatUploadedBy(value: unknown)` in `document-history.component`: string trim → username/email/name su object → '-' fallback. BE serializza gia' String, l'helper protegge da fixture corruptd o drift contract future. | 3.2.3 + 4.2.2 |

---

## Stato di arrivo

### Branch
- BE `feature/monitoring-integration-uat` HEAD `40f6c4f` (allineato origin, pushed 2026-05-30)
- FE `feature/monitoring-integration-uat` HEAD `4288056` (allineato origin, pushed 2026-05-30)

### Bug list cumulativa
**18 totali, 18 fixati, 0 aperti** (escluso #7 by-design).

| Round | Chiusi |
|---|---|
| Round 1 (handoff 2026-05-23) | #2, #16, #18 |
| Round 2 (handoff 2026-05-28) | (rimosso #13) |
| Bug #10 dedicato (handoff 2026-05-29) | #10 (Wave 1 BE + Wave 2 FE + smoke E2E full) |
| **Round 3 (questa sessione)** | **#1, #3, #4, #5, #6, #8, #9, #11, #12, #14, #15, #17, #19** |
| By-design (non fixare) | #7 |

### Test & build
- BE `./mvnw compile -q -DskipTests`: SUCCESS dopo ogni bug. Suite test BE inalterata (non lanciata in questa sessione perche' i fix sono additivi e gli unit test esistenti non toccano i metodi modificati).
- FE `npx tsc --noEmit --skipLibCheck`: CLEAN dopo ogni bug. 3 spec event-builder estesi (`expired-events`, `expiring-events`, `dashboard`) per i 3 nuovi campi `eventId`/`practiceId`/`hasException`.

---

## Prossima sessione — cosa serve

T-024 al 100% tecnico. Resta solo validazione end-to-end prima di archiviare il milestone.

### Step 1 — Retest E2E Playwright completo (1-2h, Davide)

Ripetere lo smoke E2E completo STEP 1-10 con BE+FE up locali e i fix attivi. Punti critici da verificare per i bug appena chiusi:

| Bug | Test smoke E2E |
|---|---|
| #9 | Upload nuovo CC → click GenAI → slot GENAI_PREFILLED ora ha tipologia "LOAN_TO_VALUE" prefilled (no piu' Modifica Dati manuale) |
| #19 | Doc DA_ASSEGNARE → page CC → click "Assegnazione con GenAI" → senza reload, button 🚩 "Ritorna a documentazione incompleta" deve apparire immediatamente |
| #5 | Apri Covenant detail → Modifica Dati → campo "Variabile" e' dropdown Si/No (no piu' input text) |
| #4 | Apri Covenant detail (o dialog Aggiungi Covenant) → dropdown Tipologia mostra 53 opzioni (no piu' 10) |
| #14 | Mailing list pratica in stato REPORT_GENERATO → bottone "Conferma e Aggiorna" NON visibile, sezione Report visibile |
| #11 | Mailing list dialog upload AD Form → durante upload, spinner visibile + bottoni disabled |
| #12 | Mailing list dialog upload → drag file `.txt` → toast "Tipo file non valido" + dettaglio "Formati supportati: PDF, DOC, ..."; drag file > 3MB → toast "File troppo grande" |
| #15 | Mailing list ISP "Visualizza Richiesta Aggiornamento" → X header e bottone "Chiudi" footer presenti e funzionanti |
| #17 | Dashboard Deloitte `/consultant/monitoring/expired-events` → cella "Azione" ora bottone cliccabile (non stringa "UPDATE") con label corretta per ruolo + stato eccezione |
| #6 | Apri Covenant detail → cambia Periodicita TRIMESTRALE → SEMESTRALE → save → check via API `GET /monitoring/covenants/{id}/events` → eventi futuri rigenerati con intervallo 6 mesi |
| #1 | ISP `/user/monitoring/...` → click "Salta Monitoraggio" su evento scaduto → carica PDF allegato → invia → Deloitte vede "Visualizza Eccezione" → bottone download PDF funzionante (non disabled) |
| #3 | Storico Documenti pratica con doc.uploadedBy = stringa email → cella "Caricato da" mostra email leggibile (no `[object Object]`) |
| #8 | Export Excel `/consultant/monitoring/practices/export` → file scaricato → colonna "Semaforo" mostra "Monitoraggio - Fuori Soglia" (non "MONITORING_FUORI_SOGLIA") |

### Step 2 — UAT funzionale con stakeholder ISP (1 sessione esterna)

Coordinare con utente ISP reale o team funzionale per esecuzione UAT su scenari BR sez. 1.1, 4.2.x, 6.3, 8.4-8.5. Lista scenari attesa nello scope T-024 originale (fase D).

### Step 3 — Code review finale (1h, BE+FE)

- BE: Davide review dei 6 commit BE round 3 + bug #10 Wave 1 (`21d5636`+`c83a7c0`+`7231bfb`+`5f22718`+`a38aa32`+`db9254d`+`8ee89d5`+`1a5a5b2`+`df7f3fe`+`40f6c4f`).
- FE: Carmine review dei 8 commit FE round 3 + bug #10 Wave 2 (`9281812..08f8025`+`bd47922`+`ed2a895`+`893a1f7`+`b6eaa32`+`9c1645f`+`9436e06`+`6f0b8fc`+`4288056`).

### Step 4 — Archive T-024 → close milestone

- Update `PROGRESS.md` con T-024 al 100% archiviato.
- Move `2026-05-04_monitoring/` da `in-progress/` a `completed/`.
- Update milestone tracking se necessario.

---

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD allineati origin
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3 && git status --short
# Atteso BE HEAD: 40f6c4f (allineato origin)

cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -3 && git status --short
# Atteso FE HEAD: 4288056 (allineato origin)

# Sanity compile
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw clean compile -DskipTests -q
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck

# Suite test BE+FE (se non gia' lanciate post-push)
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='Monitoring*'
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false

# BE+FE up
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw spring-boot:run -DskipTests -Dspring-boot.run.profiles=local &
cd C:/Users/davmelis/Documents/Github/ba-web && npm run start &
curl -s http://localhost:8091/actuator/health
curl -s -o /dev/null -w "FE=%{http_code}\n" http://localhost:4200

# Stato fixture M1
"/c/Program Files/MySQL/MySQL Shell 8.0/bin/mysqlsh" \
  --user=root --password='Deloitte123!' --host=localhost --port=3306 \
  --schema=ba-be-local --sql -e \
  "SELECT id, event_id, status, reject_reason, file_name, uploaded_by FROM mp_monitoring_document ORDER BY id;"
```

---

## Utenti DB locale (invariato)

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`)
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`)
- Login richiede TOTP via Google/Microsoft Authenticator

---

## Riferimenti

- **Handoff predecessore (plan round 3)**: `handoff/HANDOFF_2026-05-30_bugfix_round_3_plan.md`
- **Handoff bug #10 closed**: `handoff/HANDOFF_2026-05-29_bug10_CLOSED_smoke_full.md`
- **Handoff bug-fix round 2 done**: `handoff/HANDOFF_2026-05-28_bugfix_round_2_done.md` (con la lista 18 bug originali)
- **PLAN.md**: T-024 al 100% post bug-fix round 3
- **BR completo**: `requirements/BR_Monitoraggio_V6.md` 1376 righe
- **Mockup**: `requirements/Mockup_Monitoraggio_v10.md`
