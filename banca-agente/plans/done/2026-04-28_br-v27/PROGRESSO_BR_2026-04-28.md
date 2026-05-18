# Progresso Implementazione BR v28 — Risoluzione GAP

Data creazione: `2026-04-28`
Ultimo aggiornamento: `2026-05-06 15:30`

## Riepilogo

| Metrica | Valore |
|---|---|
| Task totali | 16 |
| Completate | 16 |
| In corso | 0 |
| Da iniziare | 0 |
| Bloccate | 0 |
| Progresso complessivo | 100% |

## Stato Task

| ID | Attività | Owner | Progresso | Stato | Branch | Note |
|---|---|---|---:|---|---|---|
| S-000 | Stream 0 — Foundation (Enums, Modelli, Migrazioni) | Ahmad | 100% | Completata | feature/gap-0-foundation | 9 enum nuovi, 4 enum aggiornati, BookingLender entity/DTO, Practice aggiornata, migration SQL |
| S-01A | Stream 1A — SACE GenAI Pipeline (BE) | Davide | 100% | Completata | feature/gap-1a-sace-genai-be | Pipeline completa, confirm GenAI, replace report, 6 test, commit e71578e |
| S-01B | Stream 1B — SACE GenAI Vista (FE) | Alexios | 100% | Completata | feature/gap-1b-sace-genai-fe | SaceGenaiExtractionComponent (19 campi), SaceGenAiService, integrazione sace-phase, report preview/replace, 3 stati GenAI FE |
| S-02A | Stream 2A — Booking Lender API (BE) | Ahmad | 100% | Completata | feature/gap-2a-booking-lender-be | Repository, Mapper, DTO, Service CRUD, Controller endpoints, 6 unit test |
| S-02B | Stream 2B — Booking Lender Pop-up (FE) | Carmine | 100% | Completata | feature/gap-2b-booking-lender-fe | Add lender dialog, booking-phase refactor, i18n, 1 commit |
| S-03A | Stream 3A — Post-Closing Pipeline (BE) | Davide | 100% | Completata | feature/gap-3a-post-closing-be | Pipeline GenAI 5 CSV, 3 report builder, CRUD, data scadenza, mutua esclusione, 23 test unitari |
| S-03B | Stream 3B — Post-Closing Viste GenAI (FE) | Carmine | 100% | Completata | feature/gap-3b-post-closing-fe | Dipende da S-000, S-03A. Carmine ha fatto anche la parte di Adham. Riserva risolta con S-03B-fix |
| S-04A | Stream 4A — Report Mapping + Booking GenAI (BE) | Ahmad | 100% | Completata | feature/gap-4a-report-mapping-be | GenAI pipeline, mailing list integration, fix test compilazione e regressione |
| S-04B | Stream 4B — Report Integration (FE) | Georgios | 100% | Completata | feature/gap-4b-report-integration-fe | PsmStateEnum booking GenAI, GenAI extraction configurabile, booking directive + component GenAI, commit 337f6ee |
| S-05A | Stream 5A — Pipeline Garanzie (BE) | Ahmad | 100% | Completata | feature/gap-5a-garanzie-be | Pipeline GenAI garanzia, FormatPgaSheetBuilder, merge obblighi, 5 test, commit 8b42935 |
| S-05B | Stream 5B — Pipeline Garanzie (FE) | Adham | 100% | Completata | feature/gap-5b-garanzie-fe | FormatPgaExtractionComponent, AssuranceGenaiContainerComponent, mutua esclusione AssuranceComponent, InformativeDutyExtraction con categorie garanzia. 3 commit, build OK |
| S-06A | Stream 6A — Nome Deal & Termination Date (BE) | Ahmad | 100% | Completata | feature/gap-6a-nome-deal-be | 2 DTO, 2 PATCH endpoint, 4 comment endpoint, 6 test unitari |
| S-06B | Stream 6B — Nome Deal & Termination Date (FE) | Georgios | 100% | Completata | feature/gap-6b-nome-deal-fe | ServiceDossierModel, PracticeService (CRUD + comments), environment, PostPhaseComponent (2 sezioni sopra accordion CdF), i18n. 8 file, +71 righe. Build OK |
| S-03B-fix | Fix Dropdown Obblighi e Covenant Numerici FE | Carmine | 100% | Completata | feature/gap-3b-fix-dropdown | Allineare INFORMATIVE_DUTY_TYPE_OPTIONS (37 voci BR) e COVENANT_TYPE_NUMERIC_OPTIONS (53 voci BR) in post-closing-dropdown.const.ts |
| S-07A | Rimozione dealId da creazione pratica | Ahmad | 100% | Completata | feature/gap-7a-remove-dealid-creation | Rimosso dealId FormControl+HTML dalla form FE. BE già opzionale. Build Angular OK. Nessuna regressione. |
| S-06B-fix | Verifica ownership Nome Deal / Termination Date | Georgios | 100% | Completata | feature/gap-6b-fix-ownership | FE: editableRole su NdgContractConfigurationModel, canEdit getter in ndg-contract-section, dealName→Owner, terminationDate→Consultant. BE: role-check in PracticeFacade (OWNER per dealName, CONSULTANT per terminationDate). Build FE OK |

## Log Attivita

### 2026-04-28
- Sessione avviata da Ahmad. File spostati in `plans/in-progress/`. Progresso creato.
- **S-000 completato**: 9 enum nuovi, 4 enum aggiornati, BookingLender entity/DTO, Practice.java aggiornata (bookingLenders, dealName, terminationDate), PracticeDTO aggiornata, migration SQL 001_gap_0_foundation.sql. Commit su branch `feature/gap-0-foundation`.

### 2026-05-04
- **S-01A avviato** da Davide. Branch `feature/gap-1a-sace-genai-be` creato da `feature/sprint-wave-3-forbice`.
- **S-01A pipeline implementata** (NO COMMIT ancora): AIMetadataProcessService (3 metodi SACE), AsyncAIMetadataProcess (callback con tracking parziale TS/CdF), AIProcessController (3 endpoint), CsvImportService (importCsvFileSace), CsvDataRecordRepository (2 metodi), SaceDataFusionService+Impl (fusione CdF>TS), SaceGuaranteeReportSheetBuilder (19 campi), GeneratedReportExcelTemplateFactory (case SACE), GeneratedReportExcelService (mapping SACE).
- **DA FARE**: endpoint "Conferma dati GenAI e chiudi" (SACE_GENAI_COMPLETED → SACE_GENAI_VALIDATED), endpoint "Modifica report caricati", test unitari, verifica compilazione, commit.
- **COMPLETATI** (sessione 2): endpoint confirmSaceGenAi (PracticeFacade + PracticeController), endpoint replaceReport (GeneratedReportFacade + GeneratedReportController), completeSaceGuaranteeReport (GeneratedReportFacade), 6 test unitari (4 SaceDataFusionServiceImplTest + 2 GeneratedReportFacadeTest), compilazione OK con Java 21, test tutti verdi. Manca solo commit.
- Sessione avviata da Ahmad. Inizio lavoro su **S-02A — Booking Lender API (BE)**. Branch `feature/gap-2a-booking-lender-be` creato.
- **S-02A completato**: BookingLenderRepository (3 query), BookingLenderMapper (MapStruct), AddBookingLenderRequestDTO (3 campi validati), PracticeFacade (addBookingLender, deleteBookingLender, listBookingLenders con max 15 e state check), PracticeController (GET/POST/DELETE endpoints), 6 unit test. - Inizio lavoro su **S-04A — Report Mapping + Booking GenAI (BE)**. Branch `feature/gap-4a-report-mapping-be` creato. Merge di S-02A nel branch.
- **S-04A implementazione completata**: Migration SQL (002_gap_4a_booking_genai.sql), PsmStateCodeEnum (BOOKING_GENAI_IN_PROGRESS/COMPLETED), CsvDataRecord + repository (campo source per coesistenza dati multi-fase), CsvImportService (import per source senza cancellare dati pre-closing), AIMetadataProcessService (engagementAiBookingAdFormProcess + processCsvFileBookingAi), AsyncAIMetadataProcess (processBookingAiMetadataProcess), AIProcessController (endpoint engagement-process-booking + end-process-booking). 2 file test creati (5 test totali).
- **S-04A completato**: fix compilazione test e regressione test esistenti (commit 5cad20d).
- **Merge in feature/sprint-wave-3-forbice**: Stream 1A (Davide), Stream 2A + 4A (Ahmad). Conflitti risolti su 6 file (tutti additivi).
- Sessione avviata da Alexios. Inizio lavoro su **S-01B — SACE GenAI Vista (FE)**. Branch `feature/gap-1b-sace-genai-fe` creato.
- **S-01B completato**: PsmStateEnum (3 nuovi stati SACE GenAI), SaceGenAiService (engagement, confirm, CRUD, replaceReport), SaceGenaiExtractionComponent (tabella 19 campi con PDF viewer, edit/add/delete inline, dialog conferma), sace-phase integrazione (avvio GenAI, progress indicator, vista estrazione, anteprima report, modifica report caricati), environment endpoints aggiornati, CsvDataRecordDTO e SaceGenAIRowModel. Commit 7464d57.
- **S-01B fix**: corretti 2 import path errati in sace-phase.component.ts (SpinnerComponent e SaceGenaiExtractionComponent). Build Angular verificata OK. Commit dd2621a.
- **S-02B completato** da Carmine: AddBookingLenderDialog (3 campi), refactor booking-phase component, model BookingLender, i18n. Mergiato in ba-web.
- **Merge in ba-web feature/sprint-wave-3-forbice**: Stream 1B (Alexios) e Stream 2B (Carmine). Nessun conflitto.
- **S-04B avviato** da Georgios. Branch `feature/gap-4b-report-integration-fe` creato da `feature/sprint-wave-3-forbice`.
- **S-04B completato**: PsmStateEnum (BOOKING_GENAI_IN_PROGRESS/COMPLETED + stateOrder renumber), environment (engageAIProcessBooking), PracticeService (engageAIProcessBooking), GenAIExtractionService (fetchBookingAdFormView con ?source=BOOKING), GenaiExtractionComponent (reso configurabile: editableState/validatedState/fetchViewFn/dialogTitleSuffix con default pre-closing), BookingPhaseDirective (azioni startGenAI + viewGenAIExtraction), BookingPhaseComponent (sezione stato GenAI con spinner/check). 9 file, 144 righe. Commit 337f6ee. 
- Sessione avviata da Ahmad. Inizio lavoro su **S-06A — Nome Deal & Termination Date (BE)**. Branch `feature/gap-6a-nome-deal-be` creato da `feature/sprint-wave-3-forbice`.
- **S-06A completato**: UpdatePostClosingDealNameRequestDTO (String dealName @NotBlank), UpdatePostClosingTerminationDateRequestDTO (LocalDate terminationDate @NotNull), PracticeFacade (updatePostClosingDealName con trim+validazione, updatePostClosingTerminationDate con null check), PostClosingPracticeDataController (PATCH deal-name, PATCH termination-date, POST/GET deal-name/comments con NOME_DEAL, POST/GET termination-date/comments con TERMINATION_DATE). PostClosingFieldCommentService non modificato (NOME_DEAL e TERMINATION_DATE gestiti correttamente dal branch !isBookingField). 6 unit test tutti verdi, 52 test totali 0 regressions.
- **S-06B avviato** da Georgios. Branch `feature/gap-6b-nome-deal-fe` creato da `feature/sprint-wave-3-forbice`.
- **S-06B completato**: ServiceDossierModel (dealName, terminationDate), environment (updateDealName, updateTerminationDate), PracticeService (updateDealName, updateTerminationDate, updatePracticeField switch, fieldToApiPath), PostPhaseComponent (dealSections array con Nome Deal + Termination Date sopra accordion CdF, fieldUpdateMessageKeys, onEdit esteso), i18n IT (9 chiavi) + EN (2 chiavi). 8 file modificati, +71 righe. Build Angular OK (0 errori).

### 2026-05-05
- Sessione avviata da Carmine. Inizio lavoro su **S-03B — Post-Closing Viste GenAI (FE)**. Carmine fa anche la parte di Adham (Obblighi Informativi). Branch `feature/gap-3b-post-closing-fe` creato da `feature/sprint-wave-3-forbice`.
- **S-03B foundation completata**: PostClosingGenAiService (CRUD + startGenAi + confirm), PostClosingGenAiRecordDTO/CreateRequest/UpdateRequest, CovenantNumericRow/CovenantDescriptiveRow/InformativeDutyRow, 9 costanti dropdown, PsmStateEnum (3 stati GenAI post-closing), environment endpoints (4 nuovi), FinancingAgreementModel.reportPathChoice. Commit 94f5651.
- **S-03B 3 componenti estrazione completati**: CovenantNumericExtractionComponent (14 colonne, 6 dropdown), CovenantDescriptiveExtractionComponent (6 colonne, 3 dropdown), InformativeDutyExtractionComponent (7 colonne, 2 dropdown, tab Periodici/Evento). Commit 0bcdd30.
- **S-03B container + integrazione completati**: PostClosingGenaiContainerComponent (3 tab), FinancingAgreementComponent aggiornato con mutua esclusione (startGenAi, viewGenAiExtraction, modifyUploadedReports), i18n IT+EN. Build Angular OK (0 errori TS). Commit 4fb5378, fix 6dcbf72.
- Sessione avviata da Ahmad. Inizio lavoro su **S-05A — Pipeline Garanzie (BE)**. Branch `feature/gap-5a-garanzie-be` creato da `feature/sprint-wave-3-forbice`.
- **S-05A completato**: AIUseCase.AGENCYDESK_GUARANTEE, GenAiFormKeyEnum (FORMAT_PGA_CAMPO/VALORE), engagementAiGuaranteeProcess (upload S3 + chiamata GenAI per singola garanzia), processCsvFileGuaranteeAi (3 CSV: obblighi periodici/evento + Format PGA con prefisso GAR_{id}_), processGuaranteeMetadataProcess (callback asincrono con update sectionState), PostClosingGenAIController TODO completati (CdF + Assurance con resolveDocumentType), FormatPgaSheetBuilder (2 colonne, multi-sheet per Ipoteca RE/non-RE/Navi-Aerei), CENS_ASSURANCES registrato in factory e service, merge obblighi informativi CdF + garanzie (isGuaranteeObbligoCategory). 1 file creato, 8 modificati, 5 test nuovi, 75 test totali tutti verdi. Commit 8b42935.
- Sessione avviata da Adham. Inizio lavoro su **S-05B — Pipeline Garanzie (FE)**. Branch `feature/gap-5b-garanzie-fe` creato da `feature/sprint-wave-3-forbice`.
- **S-05B completato**: AssuranceModel.reportPathChoice aggiunto, FormatPgaRow model, 3 helper funzioni categorie garanzia (guaranteeObblPeriodicCategory, guaranteeObblEventCategory, guaranteeFormatPgaCategory), FormatPgaExtractionComponent (tabella 2 colonne + CRUD), AssuranceGenaiContainerComponent (2 tab: Obblighi Informativi + Format PGA), InformativeDutyExtractionComponent esteso per categorie garanzia (GAR_{id}_OBBL_PERIOD/EVENT_), AssuranceComponent aggiornato con mutua esclusione (startGenAi, viewGenAiExtraction, modifyUploadedReports). 10 file (3 nuovi, 5 modificati, 2 creati SCSS/HTML). Build Angular OK (0 errori TS). 3 commit.
- **TUTTE LE TASK COMPLETATE (13/13, 100%)**. Report, piano e progresso da spostare in `plans/done/`.

### 2026-05-06
- **Revisione post-implementazione** da documentazione BR aggiornata (docs/review/). Ri-analizzato intero BR v28 + mockup + tracciati GenAI.
- **4 discrepanze identificate**:
  1. Campo `dealId` obbligatorio nella form di creazione pratica (FE + BE) — non previsto dal BR 3.1.1
  2. Dropdown `INFORMATIVE_DUTY_TYPE_OPTIONS` nel FE con valori diversi dal BR (34 voci generiche vs 37 voci specifiche del BR)
  3. Dropdown `COVENANT_TYPE_NUMERIC_OPTIONS` nel FE con valori diversi dal BR (50 voci generiche vs 53 voci specifiche del BR)
  4. Ownership Nome Deal (ISP) vs Termination Date (Deloitte) non verificata nel FE/BE
- **3 task correttive aggiunte**: S-03B-fix (Carmine, 1gg), S-07A (Ahmad, 0.5gg), S-06B-fix (Georgios, 0.5gg)
- **S-03B** portata a stato "Completata con riserva" per discrepanza dropdown
- **Progresso aggiornato**: 12 completate + 1 con riserva + 3 da iniziare = 16 totali, 81%
- **S-03B-fix completato** da Carmine: creati enum TS InformativeDutyTypeEnum (38 valori) e CovenantTypeNumericEnum (53 valori) che specchiano i BE enum. Costanti dropdown ora usano Object.values(enum). Label testuali in it-IT.json (sezioni genAIExtraction.informativeDutyType e covenantTypeNumeric). Template aggiornati con customTranslate pipe per dropdown e read-only. Build OK. Commit 6eaa9d7.
- **S-03B** riportata a "Completata" (riserva risolta)
- **Progresso aggiornato**: 14 completate + 2 da iniziare = 16 totali, 88%
- Sessione avviata da Ahmad. Inizio lavoro su **S-07A — Rimozione dealId da creazione pratica**. Branch `feature/gap-7a-remove-dealid-creation` creato in ba-web e ba-back-end.
- **S-07A completato**: Rimosso `dealId: new FormControl(null, Validators.required)` da practice-creation-form.component.ts, rimosso blocco HTML input dealId da practice-creation-form.component.html. BE già opzionale (nessuna annotation su StartPracticeReqDTO.dealId, PracticeService gestisce null). Verificato: BankContactsSheetBuilder gestisce dealId null, header FE ha fallback `|| '-'`, model FE ha `dealId?: string`. Build Angular OK (0 errori TS). 2 file FE modificati.

### 2026-05-06 (sessione pomeridiana)
- Sessione avviata da Georgios. Inizio lavoro su **S-06B-fix — Verifica ownership Nome Deal / Termination Date**. Branch `feature/gap-6b-fix-ownership` creato in ba-web e ba-back-end da `feature/sprint-wave-3-forbice`.
- **S-06B-fix completato**: FE: aggiunto campo `editableRole` a `NdgContractConfigurationModel`, getter `canEdit` in `NdgContractSectionComponent` (sostituisce hardcoded `RoleEnum.Owner`), `*ngIf="canEdit"` nel template (sostituisce `*appHasRole`), `dealName→RoleEnum.Owner` e `terminationDate→RoleEnum.Consultant` in `PostPhaseComponent`. BE: role-check in `PracticeFacade.updatePostClosingDealName` (solo OWNER) e `updatePostClosingTerminationDate` (solo CONSULTANT). Build Angular OK. 5 file modificati (4 FE + 1 BE).

### 2026-05-06 (merge task correttive)
- **Merge S-03B-fix** (Carmine): già su `feature/sprint-wave-3-forbice` (commit 6eaa9d7)
- **Merge S-07A** (Ahmad): `feature/gap-7a-remove-dealid-creation` mergiato in forbice (ba-web + ba-back-end)
- **Merge S-06B-fix** (Georgios): `feature/gap-6b-fix-ownership` mergiato in forbice (ba-web + ba-back-end)
- **Tutte le 16 task completate e mergiate (100%)**
