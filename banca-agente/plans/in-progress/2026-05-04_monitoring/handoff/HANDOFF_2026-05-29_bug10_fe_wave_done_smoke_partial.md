# Handoff BUG #10 — FE WAVE 2 DONE + SMOKE PARTIAL — 2026-05-29 (sessione 2)

Sessione AI chiusa dopo Wave 2 FE completa + smoke E2E parziale. **2 commit BE + 8 commit FE pronti in locale, NON pushati.** Smoke Manual ✅, GenAI parzialmente ✅ (verifica finale richiesta), Reject ⏳ da testare, View ✅ fix applicato (da ri-verificare). Prossima sessione = **completare smoke (Reject + View re-verify) + push BE+FE + chiudere bug #10**.

## Branch state

- BE `feature/monitoring-integration-uat` HEAD = **`5f22718`** (era `c83a7c0` — 2 nuovi commit, NON pushati)
  - `7231bfb` fix(monitoring): allow GenAI extraction to start from DA_ASSEGNARE state (BR 4.2.3)
  - `5f22718` fix(monitoring): widen mp_monitoring_document.status column to VARCHAR(50) for DA_ASSEGNARE compat
- FE `feature/monitoring-integration-uat` HEAD = **`08f8025`** (era `1d27047` — 8 nuovi commit, NON pushati)
  - `9281812` feat(monitoring): add CC assignment page-based UX (bug #10 BR 4.2.3) — 26 file, +2346/-19
  - `a957ab5` feat(monitoring): wire CC assignment page in document-history + remove old dialog triggers — 5 file, +99/-252
  - `73d397e` chore(monitoring): remove deprecated CcAssignmentComponent + MonitoringGenAiExtractionComponent — 10 file, -1786
  - `19d9763` test(monitoring): fix 31 pre-existing failures (HttpClient injection, status falsy fallback, thead column count)
  - `ec833ba` fix(monitoring): sanitize preview PDF URL via DomSanitizer (NG0904 resource URL XSS guard) — sostituito poi da 829bbb4
  - `829bbb4` fix(monitoring): use shared PdfViewerWrap for CC preview (download blob via service, iframe + Blob URL)
  - `1450e37` fix(monitoring): VIEW mode loads assigned event + GenAI prefills covenantId + Modifica Dati -> FILLING (BR 4.2.4 + BR 735)
  - `08f8025` fix(monitoring): show 'Aggiungi Covenant' also in GenAI mode (BR 4.2.3 riga 741-742)
- Profilo `deloitte-profiles` HEAD = aggiornato con plan Wave 2 + questo handoff

## Cosa è stato fatto questa sessione

### BE Wave 1.5 (mini-fix necessari per BR-conformità)

1. **`7231bfb`** — `MonitoringDocumentService.markGenAiInProgress` ora accetta anche `DA_ASSEGNARE` come stato di partenza (oltre `IN_ATTESA_CONFERMA`/`VERIFICA_IN_CORSO`). Necessario perché BR 4.2.3 dice che GenAI extraction parte dalla page CC mode CHOOSE quando doc è già in `DA_ASSEGNARE`. +1 metodo + 3 nuovi test `MarkGenAiInProgress` nested.
2. **`5f22718`** — nuovo file SQL `010_monitoring_da_assegnare_status.sql` con `ALTER TABLE mp_monitoring_document MODIFY COLUMN status VARCHAR(50)`. Necessario perché il DB locale di Davide aveva colonna ENUM ristretta (non includeva `DA_ASSEGNARE`) → "Data truncated for column 'status' at row 1". **Davide ha già eseguito SQL sul suo DB locale manualmente.** Il file SQL serve per future fresh install.

### FE Wave 2 (4 commit funzionali + 4 commit bugfix UAT)

1. **`9281812`** — nuovo modulo `cc-assignment-page/` con 4 component (Page + Slot + ScheduleTable + RejectDialog). State machine slot `EMPTY → FILLING → SCHEDULE_VISIBLE → EVENT_SELECTED → CONFIRMED` + `GENAI_PREFILLED`. Sub-route `monitoring/practice/:id/cc/:documentId[?mode=view]`. Service `monitoring.service.ts`: rename `confirmDocument → validateDocument`, nuovo `completeAssignment(id, payload)`, `getSchedule` esteso con `compactParams`. Enum `MonitoringDocumentStateEnum` + `DA_ASSEGNARE`. Model `monitoring-cc-assignment.model.ts` con `CcSlotState` + `CcCompleteAssignmentRequest`. i18n keys `monitoring.cc.*` in it-IT.json + en-GB.json. 60 nuovi test FE (8+9+14+22+altri). Wave1 BE caller rename: `cc-assignment.component.ts` + `document-history.component.ts` puntano a `validateDocument` (per tenere tsc green).
2. **`a957ab5`** — `document-history.component`: wire-up bottoni "Assegna Monitoraggi" (DA_ASSEGNARE/GENAI_DA_VALIDARE → naviga `./cc/:documentId`) + "Visualizza Assegnazione" (COMPLETATO → `./cc/:documentId?mode=view`). Rimosso state GenAI dialog (`showGenAiDialog`, handler `onStartGenAi`/`onModifyGenAi`/`onGenAiApplied`/`waitForGenAiCompletion`/`handleGenAiError`). Mapping enum severity/label esteso con `DA_ASSEGNARE`. `practice-detail.component`: rimosso bottone "Assegna Monitoraggi" + `<app-cc-assignment>` host + `@ViewChild` + handler.
3. **`73d397e`** — DELETE cartelle `cc-assignment/` (4 file) + `monitoring-genai-extraction/` (5 file). Rimosso `CcAssignmentComponent` da declarations + `MonitoringGenAiExtractionModule` da imports nel `practice-detail.module.ts`.
4. **`19d9763`** — Fix 31 pre-existing FE test failures (NON causati da Wave 2):
   - `exception-dialog.component.spec.ts`: aggiunto `HttpClientTestingModule` import (MonitoringService injection HttpClient mancante).
   - `covenant-detail.component.spec.ts` + `schedule-tab.component.spec.ts`: test "status falsy default to OK" → component restituisce `'—'` stringa, aggiornato assertion.
   - `practices.component.spec.ts`: 14 columns (era 13) + `practiceId` hidden in thead.
   - `dashboard.component.spec.ts`: recentThead 13 columns (era 12) + `practiceId` hidden.
5. **`ec833ba`** — Fix Angular NG0904 (UNSAFE_VALUE_IN_RESOURCE_URL) su `<object data="..." type="application/pdf">` aggiungendo `DomSanitizer.bypassSecurityTrustResourceUrl`. *(Sostituito da 829bbb4 — preview non funzionava con `<object>` per CORS auth.)*
6. **`829bbb4`** — Sostituito `<object data>` con `<app-pdf-viewer-wrap [src]="pdfSource">` shared component. Page component scarica blob via `downloadDocument()`, converte in `Uint8Array`, lo passa al PdfViewerWrap che crea Blob URL + iframe sanitizzato. Pattern allineato a `genai-extraction.component`. Aggiunto `PdfViewerWrapModule` agli imports del module.
7. **`1450e37`** — Fix 2 bug UAT smoke:
   - **View Mode**: `buildViewSlot` ora cerca `covenantId` dal `doc.covenantCode` match con `practiceCovenants`, poi chiama `getEventsByCovenant` per caricare l'evento assegnato e popolare `scheduleEvents` + `monitoringValue` + `referenceDate` nello slot (BR 4.2.4).
   - **GenAI**: `buildSlotsFromExtraction` mappa `COVENANT_NUM_TIPOLOGIA` extracted al covenant esistente (`practiceCovenants.find(c => c.covenantType === extractedType)`) per popolare `selectedCovenantId`. `MonitoringCcCovenantSlot.onModificaDati` cambia phase da `SCHEDULE_VISIBLE` → `FILLING` (BR 735: "stesso UX manual" → 3 box editabili).
8. **`08f8025`** — Bottone "Aggiungi Covenant" ora visibile anche in mode GENAI (BR 4.2.3 riga 741-742). Era visibile solo in mode MANUAL.

### Test result post-fix
- BE: 97/97 PASS (3 nuovi `MarkGenAiInProgress`)
- FE: tutti i nuovi spec verdi (~60+ test), 585 monitoring + altri verdi
- 0 fail post-Wave 2

## Smoke E2E stato

Eseguito con utente Deloitte CONSULTANT `davide94x@gmail.com` su `localhost:4200` (BE 8091).

| Flusso | Status | Note |
|---|---|---|
| **A — Manual** | ✅ FULL OK | Valida → DA_ASSEGNARE → Assegna Monitoraggi → page CC → Manuale → compila 3 box → Associa Covenant → scadenziere → Assegna → Conferma → Concludi → doc COMPLETATO |
| **B — GenAI** | ⚠️ PARZIALE | Avviato OK fino a GENAI_PREFILLED. Modifica Dati → FILLING ora corretto (post `1450e37`). Aggiungi Covenant ora visibile (post `08f8025`). **Da ri-verificare flusso completo (Modifica Dati → Concludi)** |
| **C — Reject** | ⏳ NON TESTATO | Non ancora eseguito. Doc GENAI_DA_VALIDARE → header "Ritorna a documentazione incompleta" → modal → "Conferma Rifiuto" → status torna IN_ATTESA_CONFERMA |
| **D — View** | ⚠️ FIX APPLICATO | Bug iniziale: scadenziere vuoto. Fix `1450e37` carica evento assegnato. **Da ri-verificare** che ora scadenziere mostri 1 riga reale con monitoringValue + referenceDate |

## Bug trovati e fixati durante smoke

1. **NG0904 UNSAFE_VALUE_IN_RESOURCE_URL** (page CC preview) — Angular bloccava `<object data>` su URL non sanitizzato. Fix: usato shared `PdfViewerWrap` con Blob URL + iframe sanitizzato.
2. **Preview PDF vuota** anche con DomSanitizer — `<object>` non si caricava (probabilmente CORS auth bypass). Fix: pattern blob download standard.
3. **MySQL "Data truncated for column 'status'"** — colonna ENUM ristretta non includeva `DA_ASSEGNARE`. Fix: ALTER TABLE → VARCHAR(50) + migration SQL nel BE.
4. **BE markGenAiInProgress non accettava DA_ASSEGNARE** — necessario per startGenAiExtraction da page CC mode CHOOSE (BR 731). Fix: esteso whitelist stati.
5. **View Mode scadenziere vuoto** — buildViewSlot non caricava event. Fix: `getEventsByCovenant` + filter by eventId.
6. **GenAI tipologia vuota** — buildSlotsFromExtraction non mappava `COVENANT_NUM_TIPOLOGIA`. Fix: match `practiceCovenants` by covenantType.
7. **GenAI Modifica Dati → SCHEDULE_VISIBLE** errato (mostrava riepilogo non editabile invece di 3 box). Fix: passa a FILLING (BR 735).
8. **GenAI "Aggiungi Covenant" mancante** — era visibile solo in MANUAL. Fix: mostrato anche in GENAI (BR 741-742).

## Cosa fare nella prossima sessione

### Prio 1 — completare smoke E2E

1. **Re-verify View Mode (D)**: doc COMPLETATO → 👁 Visualizza Assegnazione → scadenziere deve mostrare 1 riga con valore + data dell'evento assegnato (no più "Nessun evento corrisponde ai filtri").
2. **Re-verify GenAI Flow completo (B)**:
   - Doc IN_ATTESA → Valida → DA_ASSEGNARE → Assegna Monitoraggi → page CC
   - Click "Assegnazione con GenAI" → spinner extraction → slot GENAI_PREFILLED con tipologia, valore, data prefilled (incluso tipologia ora se mock fornisce covenantType matching)
   - Click "Modifica Dati" → slot va a FILLING con 3 box editabili (tipologia/valore/data popolati ma modificabili)
   - Click "Associa Covenant" → scadenziere → Assegna riga → Conferma Assegnazione
   - (Opzionale) Click "Aggiungi Covenant" → slot #2 in FILLING → ripeti
   - Click "Concludi Assegnazione" → success + navigate back → doc COMPLETATO
3. **Test Reject Flow (C)**: doc GENAI_DA_VALIDARE → page CC mode GENAI → click 🚩 "Ritorna a documentazione incompleta" → modal → seleziona motivo + note → "Conferma Rifiuto" → status doc torna IN_ATTESA_CONFERMA con rejectReason "INCOMPLETE: <note>".

### Prio 2 — chiudere bug #10 + push

4. **Push BE + FE** con conferma utente. Comandi:
```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git push
cd C:/Users/davmelis/Documents/Github/ba-web && git push
```
5. **Update bug list**: bug #10 → ✅ FIXATO con commit FE `9281812..08f8025` + BE `7231bfb`+`5f22718`. Aggiorna `bug list cumulativa 18 totali`: 5 fixati (era 4 + bug 10).
6. **Update PLAN.md T-024**: avanzare da 85% post-Wave-BE a ~95% post-FE-Wave2.

### Prio 3 — eventuali polish / bonus

7. **GenAI Conferma Assegnazione diretta** (no Modifica Dati): attualmente in GENAI_PREFILLED con `assignedEventId=null`, click Conferma Assegnazione → CONFIRMED ma slot filtered out in `onConcludi` (`assignedEventId == null`). Soluzione: o mostrare anche scadenziere prefilled in GENAI_PREFILLED (richiede mock GenAI multi-block), o disabilitare Conferma Assegnazione finché eventId è null. Decisione: rimanderemo a bug separato per multi-block GenAI mock.

## Comandi rapidi per ripartire

```bash
# Verifica branch + HEAD
cd C:/Users/davmelis/Documents/Github/ba-back-end && git log --oneline -5
# Atteso BE HEAD: 5f22718

cd C:/Users/davmelis/Documents/Github/ba-web && git log --oneline -9
# Atteso FE HEAD: 08f8025

# Sanity test BE
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='Monitoring*' -q
# Atteso: 97+ PASS

# Sanity test FE (subset)
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/cc-assignment-page/**/*.spec.ts'
# Atteso: 62 SUCCESS

# Riavvia BE se serve (se non già up)
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw spring-boot:run -DskipTests -Dspring-boot.run.profiles=local

# Riavvia FE se serve
cd C:/Users/davmelis/Documents/Github/ba-web && npm run start
```

## Bug list aggiornata — **18 totali, 4 fixati + bug #10 quasi closed**

⚠️ Bug #10 = WAVE 2 FE OK ma smoke incompleto. Quando smoke verde (Reject + View re-verify) → push → bug #10 ufficialmente closed.

| # | Severity | Status |
|---|---|---|
| 16 | CRITICAL | ✅ FIXATO commit FE `1d27047` |
| 18 | HIGH | ✅ FIXATO commit BE `9ef5ede` |
| 2 | HIGH | ✅ FIXATO commit BE `c4efb8e` |
| **10** | **HIGH** | 🟡 **WAVE 2 FE DONE** — BE done (`21d5636`+`c83a7c0`+`7231bfb`+`5f22718`), FE done (`9281812`+`a957ab5`+`73d397e`+`19d9763`+`829bbb4`+`1450e37`+`08f8025`), smoke parziale (Manual ✅, GenAI ⚠️ re-verify, Reject ⏳, View ⚠️ re-verify), **NON pushato** |
| 1, 3, 5, 6, 8, 9, 11, 12, 15, 17 | MEDIUM | APERTI (10) |
| 4, 14 | LOW | APERTI (2) |
| 7 | LOW | NON da fixare (by-design) |

## Riferimenti

- **Spec design**: `plans/in-progress/2026-05-04_monitoring/specs/2026-05-28-bug10-cc-assignment-page-design.md`
- **Plan Wave 1 BE**: `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-28-bug10-cc-assignment-page-plan.md`
- **Plan Wave 2 FE**: `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-29-bug10-fe-wave2-plan.md`
- **Handoff predecessor**: `handoff/HANDOFF_2026-05-29_bug10_be_wave_done.md`
- **BR**: `requirements/BR_Monitoraggio_V6.md` sez 4.2.3 (652-744) + sez 4.2.4 (745-748)
- **PLAN.md**: T-024 al ~95% post-FE-Wave2 (era 80% pre-Wave1, 85% post-Wave1)

## Utenti DB locale (invariato)

- `davide94x@gmail.com` → Deloitte CONSULTANT (URL `/consultant/*`)
- `davide94melis@gmail.com` → ISP OWNER (banca) (URL `/user/*`)
- Login TOTP via Google/Microsoft Authenticator

## DB locale fix manuale già applicato

Davide ha già eseguito sul suo DB locale `ba-be-local`:
```sql
ALTER TABLE mp_monitoring_document MODIFY COLUMN status VARCHAR(50);
```
Il file SQL `010_monitoring_da_assegnare_status.sql` è committato in BE (5f22718) per future fresh installs / dev/uat env.
