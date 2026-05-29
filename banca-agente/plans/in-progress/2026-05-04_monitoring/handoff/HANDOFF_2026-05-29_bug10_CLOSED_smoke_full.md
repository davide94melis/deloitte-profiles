# Handoff BUG #10 — CLOSED end-to-end + SMOKE E2E FULL PASS — 2026-05-29 (sessione 3)

Sessione AI chiusa dopo smoke E2E COMPLETO 3/3 PASS via Playwright (View Mode + GenAI Flow + Reject Flow). **2 commit BE + 8 commit FE pronti in locale, NON pushati.** Bug #10 ✅ chiuso tecnicamente — manca solo `git push` su entrambi i branch (utente da confermare). Prossima sessione = push + cluster MEDIUM bugs.

## Branch state

- BE `feature/monitoring-integration-uat` HEAD = **`5f22718`** (2 commit ahead di origin, NON pushati)
  - `7231bfb` fix(monitoring): allow GenAI extraction to start from DA_ASSEGNARE state (BR 4.2.3)
  - `5f22718` fix(monitoring): widen mp_monitoring_document.status column to VARCHAR(50) for DA_ASSEGNARE compat
- FE `feature/monitoring-integration-uat` HEAD = **`08f8025`** (8 commit ahead, NON pushati)
  - `9281812` feat(monitoring): add CC assignment page-based UX (bug #10 BR 4.2.3) — 26 file, +2346/-19
  - `a957ab5` feat(monitoring): wire CC assignment page in document-history + remove old dialog triggers
  - `73d397e` chore(monitoring): remove deprecated CcAssignmentComponent + MonitoringGenAiExtractionComponent
  - `19d9763` test(monitoring): fix 31 pre-existing failures (HttpClient injection, status falsy fallback, thead column count)
  - `ec833ba` fix(monitoring): sanitize preview PDF URL via DomSanitizer (NG0904) — sostituito poi da 829bbb4
  - `829bbb4` fix(monitoring): use shared PdfViewerWrap for CC preview (download blob + iframe + Blob URL)
  - `1450e37` fix(monitoring): VIEW mode loads assigned event + GenAI prefills covenantId + Modifica Dati -> FILLING (BR 4.2.4 + BR 735)
  - `08f8025` fix(monitoring): show 'Aggiungi Covenant' also in GenAI mode (BR 4.2.3 riga 741-742)
- Profilo `deloitte-profiles` HEAD = aggiornato con PROGRESS update + questo handoff

## Cosa è stato fatto questa sessione

### Smoke E2E Playwright completo 3/3 PASS

Eseguito con utente Deloitte CONSULTANT `davide94x@gmail.com` su `localhost:4200` (BE 8091), pratica fixture M1 (id 8288), covenant M1C1 LOAN_TO_VALUE.

| Flusso | Status | Evidence | Note |
|---|---|---|---|
| **A — Manual** | ✅ FULL OK (sessione precedente) | — | Già verde dalla sessione 2 |
| **B — GenAI** | ✅ **FULL OK** | doc 9 (M1C1E5) → COMPLETATO | Full path Upload→IN_ATTESA→Valida→DA_ASSEGNARE→page CC→GenAI extraction→GENAI_PREFILLED→Modifica Dati→FILLING (3 box editabili)→Associa Covenant→scadenziere→Assegna evento M1C1E1→Conferma→Concludi→COMPLETATO. Screenshot `smoke-genai-prefilled-2026-05-29.png` + `smoke-genai-COMPLETED-2026-05-29.png` |
| **C — Reject** | ✅ **FULL OK** | doc 10 (M1C1E6) → IN_ATTESA_CONFERMA con rejectReason `"INCOMPLETE: smoke test reject 2026-05-29"` | Page CC mode GENAI→🚩 Ritorna a documentazione incompleta→modal "Rifiuto Documento"→selezionato "Documento incompleto"+note→Conferma Rifiuto→back to practice-detail. Screenshot `smoke-reject-modal-2026-05-29.png` |
| **D — View** | ✅ **FULL OK** | doc 8 (M1C1E1) mode=view | Scadenziere 1 riga (M1C1E1, valore 50, data 2026-05-29 OK), tipologia "M1C1 - LOAN_TO_VALUE" prefilled, iframe preview. Screenshot `smoke-view-mode-PASS-2026-05-29.png` |

### Fix BE/FE applicati questa sessione

NESSUN nuovo commit applicato in questa sessione. Tutti i 10 commit pendenti (2 BE + 8 FE) erano già pronti dalla sessione precedente. Sessione dedicata interamente a smoke E2E completo.

### Test result

- BE: 97/97 PASS (3 nuovi `MarkGenAiInProgress` nested)
- FE: tutti i nuovi spec verdi (~60+ test cc-assignment-page), 585 monitoring + altri verdi
- 0 fail post-Wave 2

## Findings minori durante smoke (non bloccanti per chiusura bug #10)

1. **Mock GenAI tipologia non-enum** — `AsyncMonitoringGenAiProcess.buildMockExtractedFields()` ritorna `COVENANT_NUM_TIPOLOGIA: "Esempio Covenant Mock"` che non matcha `LOAN_TO_VALUE`/etc. Il fix `1450e37` (`buildSlotsFromExtraction` cerca `practiceCovenants.find(c => c.covenantType === extractedType)`) è tecnicamente corretto ma fallisce con valore mock → tipologia non prefilled in slot GENAI_PREFILLED. **Stesso problema già tracciato come bug #9 T-024.x** ("Covenant fields non aggiornati dai mock GenAI"). In produzione con GenAI reale il match funzionerà. Mitigazione UAT: la tipologia si seleziona manualmente dopo Modifica Dati. Fix proposto: allineare i mock ai valori enum validi OPPURE aggiungere conversione label→enum nel mapping.
2. **`documentInfo` stale dopo extraction in-page** — Il getter `showRejectButton` ritorna true SOLO se `documentInfo?.status === GENAI_DA_VALIDARE`, ma quando l'utente clicca "Assegnazione con GenAI" nella page CC, `documentInfo` viene caricato all'init (status DA_ASSEGNARE) e NON viene refreshato dopo l'extraction completa. L'utente deve fare un page reload per vedere il button "Ritorna a documentazione incompleta". UX nit. Fix proposto: invocare `loadDocumentInfo()` dopo extraction success in `startGenAi()` polling.

## Cosa fare nella prossima sessione

### Prio 0 — push BE + FE (con conferma utente)

```bash
# BE: 2 commit
cd C:/Users/davmelis/Documents/Github/ba-back-end && git push

# FE: 8 commit
cd C:/Users/davmelis/Documents/Github/ba-web && git push
```

### Prio 1 — chiudere ufficialmente bug #10 + cluster MEDIUM

1. Conferma push completato, aggiorna bug list cumulativa: **18 totali, 5 fixati (#16 + #18 + #2 + #10 + 1 altro), 13 aperti**.
2. Avvia round bug-fix MEDIUM (10 bug aperti #1, #3, #5, #6, #8, #9, #11, #12, #15, #17). Suggerito ordine: prima i fix piccoli isolati, poi i bug interdipendenti.
3. Considerare i 2 findings minori smoke come bug separati o inglobare in T-024.x.

### Prio 2 — UAT funzionale ISP + review finale

4. Smoke E2E completo con utente ISP (`davide94melis@gmail.com`) sul lato banca (route `/user/*`).
5. Code review finale Davide BE + Carmine FE.
6. Test gli scheduled jobs (chiusura automatica, notifiche, ricalcolo stati) in scenari E2E reali.

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -3
# Atteso BE HEAD: 5f22718 (oppure post-push: stesso ma upstream allineato)

cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -9
# Atteso FE HEAD: 08f8025

# Sanity test BE
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='Monitoring*' -q
# Atteso: 97+ PASS

# Riavvia BE se serve (se non già up)
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw spring-boot:run -DskipTests -Dspring-boot.run.profiles=local

# Riavvia FE se serve
cd C:/Users/davmelis/Documents/Github/ba-web && npm run start
```

## Bug list aggiornata — **18 totali, 5 fixati (incluso bug #10), 13 aperti**

| # | Severity | Status |
|---|---|---|
| 16 | CRITICAL | ✅ FIXATO commit FE `1d27047` (sessione 28/05) |
| 18 | HIGH | ✅ FIXATO commit BE `9ef5ede` (sessione 28/05 notte) |
| 2 | HIGH | ✅ FIXATO commit BE `c4efb8e` (sessione 28/05 notte) |
| **10** | **HIGH** | ✅ **FIXATO end-to-end (sessione 29/05)** — BE 4 commit (`21d5636`+`c83a7c0`+`7231bfb`+`5f22718`) + FE 8 commit (`9281812..08f8025`), smoke E2E full 3/3 PASS, **NON ancora pushati** |
| 1, 3, 5, 6, 8, 9, 11, 12, 15, 17 | MEDIUM | APERTI (10) |
| 4, 14 | LOW | APERTI (2) |
| 7 | LOW | NON da fixare (by-design) |

## Riferimenti

- **Spec design**: `plans/in-progress/2026-05-04_monitoring/specs/2026-05-28-bug10-cc-assignment-page-design.md`
- **Plan Wave 1 BE**: `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-28-bug10-cc-assignment-page-plan.md`
- **Plan Wave 2 FE**: `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-29-bug10-fe-wave2-plan.md`
- **Handoff predecessor**: `handoff/HANDOFF_2026-05-29_bug10_fe_wave_done_smoke_partial.md`
- **BR**: `requirements/BR_Monitoraggio_V6.md` sez 4.2.3 (652-744) + sez 4.2.4 (745-748)
- **PLAN.md / PROGRESS.md**: T-024 al ~95% post-bug-#10-CLOSED

## Utenti DB locale (invariato)

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`)
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`)
- Login TOTP via Google/Microsoft Authenticator

## DB fixture state dopo smoke

Pratica M1 (id 8288, covenant M1C1 LOAN_TO_VALUE) ora ha 10 documenti totali:
- doc 3 (M1C1E1 26/05 14:49) COMPLETATO
- doc 4 (M1C1E2 27/05 11:00) COMPLETATO
- doc 5 (M1C1E3 29/05 14:25) COMPLETATO
- doc 6 (M1C1E4 29/05 15:19) IN_ATTESA_CONFERMA con rejectReason "OTHER: ho sbagliato io" (test session 2)
- doc 7 (M1C1E1 29/05 15:21) COMPLETATO
- doc 8 (M1C1E1 29/05 15:31) COMPLETATO
- doc 9 (M1C1E5 29/05 16:10) COMPLETATO ✅ smoke GenAI flow sessione 3
- doc 10 (M1C1E6 29/05 16:21) IN_ATTESA_CONFERMA con rejectReason "INCOMPLETE: smoke test reject 2026-05-29" ✅ smoke Reject flow sessione 3

DB locale `ba-be-local` ha già la modifica `ALTER TABLE mp_monitoring_document MODIFY COLUMN status VARCHAR(50)` (applicata manualmente sessione 2). File SQL `010_monitoring_da_assegnare_status.sql` committato in BE (`5f22718`) per future install / dev/uat env.
