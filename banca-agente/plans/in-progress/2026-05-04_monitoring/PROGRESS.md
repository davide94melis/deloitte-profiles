# Progresso Implementazione Modulo Monitoraggio (BR V6)

Data creazione: `2026-05-05`
Ultimo aggiornamento: `2026-05-19 09:30` (audit cross-branch by Davide)

## Riepilogo

| Metrica | Valore |
|---|---|
| Task totali | 31 |
| Completate | 23 |
| In corso | 4 |
| Da iniziare | 4 |
| Bloccate | 0 |
| Progresso complessivo | 84% |

## Stato Task

| ID | Attività | Owner | Progresso | Stato | Branch | Note |
|---|---|---|---:|---|---|---|
| T-001 | Enum + Entità monitoring core | Alexios | 100% | Completata | feature/monitoring-structures | 3 enum + 2 entità + 1 migration SQL |
| T-002 | Entità Spread + MailingList + MonitoringDocument | Ahmad | 100% | Completata | feature/monitoring-entities-spread-mailing | 4 entità + 1 migration SQL |
| T-003 | DTO + Repository + Vista SQL estesa | Alexios | 100% | Completata | feature/monitoring-dto-repository-view | 7 DTO, 6 repository, 1 view entity, SQL view estesa. Build OK |
| T-004 | Modulo Angular monitoring + scaffold | Georgios | 100% | Completata | feature/monitoring-fe-scaffold | Scaffold FE completo: routing, models, enum, services, sidebar |
| T-005 | i18n labels + componenti condivisi monitoring | Adham | 100% | Completata | feature/monitoring | i18n IT/EN + semaforo 6 colori su branch principale FE |
| T-006 | Macchina stati 4 livelli + generatore ID + scadenziere | Alexios | 100% | Completata | feature/monitoring-state-machine | 3 service: MonitoringStateService, MonitoringIdGeneratorService, MonitoringScheduleGeneratorService. Build OK, pushed. |
| T-007 | Service + Controller pratiche monitoring | Davide | 100% | Completata | feature/monitoring-practice-service | Service, controller, repository, filter DTO, Excel export, createFromPostClosing trigger, terminatePractice con stato CONCLUSA, 6 PSM states monitoring. Build OK. |
| T-008 | Service + Controller covenant + eventi | Ahmad | 100% | Completata | feature/monitoring-covenant-events-service | 3 file: CovenantController, CovenantService, EventService. 372 righe. |
| T-009 | Dashboard monitoring BE endpoints | Ahmad | 100% | Completata | feature/monitoring-dashboard-be | 4 file: DashboardController, DashboardService, DashboardSummaryDTO, DashboardEventDTO |
| T-010 | Dashboard monitoring FE | Carmine | 95% | Da revieware | feature/monitoring-dashboard-fe | Audit 2026-05-19: tutti i requisiti implementati (donut 6 segmenti, polling 60s, 3 tabelle 13/13/12 col). Autore reale: Georgios. Mancano test unitari. |
| T-011 | Tabella pratiche monitoring FE | Georgios | 100% | Completata | feature/monitoring-practices-table-fe | 4 file FE + 2 i18n. Build OK. |
| T-012 | Dettaglio pratica ISP — header + tab scaffold | Adham | 100% | Completata | feature/monitoring-practice-detail-isp | Header, 3 tab, covenant cards, 3 accordion, 31 test. Build OK. |
| T-013 | Scadenziere unificato + dettaglio covenant singolo | Georgios | 100% | Completata | feature/monitoring-schedule-covenant-detail | BE: schedule endpoint + DTO esteso. FE: schedule-tab + covenant-detail dialog. Build OK. |
| T-014 | Upload Compliance Certificate + storico documenti | Alexios | 100% | Completata | feature/monitoring-upload-cc-documents | BE: service+controller+migration. FE: upload+history components, i18n. Build OK. |
| T-015 | Spread BE — service + controller (2 tabelle distinte) | Ahmad | 100% | Completata | feature/monitoring-spread-be | SpreadService (7 methods), SpreadController (7 endpoints), 14 unit test. |
| T-016 | Spread FE — 2 tabelle distinte (ISP + Deloitte) | Georgios | 100% | Completata | feature/monitoring-spread-fe | SpreadTabComponent: 2 tabelle, inline edit, CRUD, validazione, role-gating, 20 unit test. |
| T-017 | Flusso Deloitte — conferma/rifiuta + assegnazione CC manuale | Alexios | 85% | In corso | feature/monitoring-deloitte-flow | Audit 2026-05-19: role-gating + CC assignment OK. Wizard 4 step invece di 6 (UX divergente da spec). Mancano test cc-assignment.spec.ts e document-history.spec.ts. |
| T-018 | Flusso Deloitte — GenAI + modifica dati covenant | Davide | 0% | Da iniziare | — | Bloccata da T-017 |
| T-019 | Eccezioni ISP + gestione Deloitte | Adham | 50% | In corso | feature/monitoring-exceptions | Audit 2026-05-19: BE 95% (endpoint+service+7 test OK, serve rebase su feature/monitoring); FE 30% (dialog standalone esiste con 33 test, ma NON wired: module non importato in practice-detail, dialog mai renderizzato, action column inerte, monitoring.service.requestException usa JSON invece di multipart → 415/400 a runtime). BLOCKER per T-MERGE-FINAL. |
| T-020 | Upload COVNO/DB Obblighi + card dashboard | Alexios | 85% | Da revieware | feature/monitoring-covno-upload | Audit 2026-05-19: BE+FE implementati. CovnoUploadService (parseXlsx POI, mergeData priorità AUTO<COVNO<MANUAL_DELOITTE per A-011), CovnoController endpoints, entity CovnoUploadHistory, migration 005, dashboard card + dialog upload + storico, i18n IT/EN 22 chiavi. Autore: Alexios (myzonalexios). Mancano test BE (CovnoUploadServiceTest) e FE (dashboard spec aggiornato). |
| T-021 | Pagine complete Scaduti + In Scadenza | Adham | 0% | Da iniziare | — | Bloccata da T-010 e T-MERGE-008 |
| T-022 | Mailing List completo | Georgios | 100% | Completata | feature/monitoring-mailing-list | BE: MailingListService, Controller, AdForm entity+migration, 31 test. FE: main page+detail, i18n. Build OK. |
| T-023 | Email templates + scheduled jobs | Davide | 100% | Completata | feature/monitoring-notifications-jobs | 7 enum email, MonitoringSchedulerService, 4 @Scheduled jobs, 7 template HTML (EM). Build OK. |
| T-024 | Integration testing + UAT + polish | Davide | 0% | Da iniziare | — | Bloccata da T-MERGE-FINAL |
| T-MERGE-001-002 | Merge entità fondazioni in feature/monitoring | Alexios | 100% | Completata | feature/monitoring-dto-repository-view | Mergiati T-001 e T-002 nel branch T-003 |
| T-MERGE-003 | Merge fondazioni BE | Alexios | 100% | Completata | feature/monitoring | git pull origin feature/monitoring-dto-repository-view in feature/monitoring. Build OK. |
| T-MERGE-004 | Merge FE scaffold | Georgios | 100% | Completata | feature/monitoring | Fast-forward merge, build OK. Pushed 2026-05-06 |
| T-MERGE-006 | Merge core BE parziale (macchina stati + pratiche) | Davide | 100% | Completata | feature/monitoring | Fast-forward merge T-006 + T-007 in feature/monitoring. Build OK. T-008 sbloccata. |
| T-MERGE-008 | Merge core BE | Davide | 100% | Completata | feature/monitoring | Fast-forward merge T-008 in feature/monitoring. Build SUCCESS. |
| T-MERGE-FINAL | Merge finale pre-integrazione | Davide | 0% | Da iniziare | — | Bloccata da T-017..T-023. Sblocca T-024. |
| T-MERGE-W2 | Merge checkpoint Wave 2 | Davide | 100% | Completata | feature/monitoring | Mergiati 3 branch BE + 5 branch FE. Conflitti i18n e practice-detail risolti. Build BE+FE: SUCCESS. |
## Log Attività

### 2026-05-05
- Progresso file creato
- T-001 completata (Alexios): creati 3 enum (MonitoringPracticeStateEnum, MonitoringEventStateEnum, MonitoringDocumentStateEnum), 2 entità JPA (MonitoringCovenant, MonitoringEvent), 1 migration SQL (002_monitoring_foundation.sql). Build: SUCCESS. Branch: feature/monitoring-structures
- T-002 completata (Ahmad): create 4 entità (SpreadTable, SpreadHistory, MailingListContact, MonitoringDocument), 1 migration SQL (003_monitoring_spread_mailing.sql). Branch: feature/monitoring-entities-spread-mailing
- T-004 completata (Georgios): modulo Angular monitoring scaffold con routing (admin + consultant + back-office/ISP), 4 sotto-pagine, 8 modelli, 4 enum, 3 servizi, sidebar menu item + i18n IT/EN. Build OK. Branch: feature/monitoring-fe-scaffold
- T-005 completata (Adham): chiavi i18n monitoring in it-IT.json e en-GB.json, componente semaforo con 6 colori distinti (OK→verde, InScadenza→giallo, DaAttenzionare→arancione, Scaduto→rosso, FuoriSoglia→rosso scuro, Conclusa→grigio). Branch: feature/monitoring (diretto)

### 2026-05-06
- Aggregazione cross-branch: verificati 4 branch remoti (monitoring-structures, monitoring-entities-spread-mailing, monitoring-fe-scaffold, monitoring). T-001 e T-002 verificate su BE repo. T-004 e T-005 verificate da PROGRESSO + conferma TL.
- Aggiunti 4 merge checkpoint espliciti al piano: T-MERGE-001-002, T-MERGE-006, T-MERGE-W2, T-MERGE-FINAL. Motivo: task con dipendenze su branch separati non potevano partire (T-003 bloccata, T-008 bloccata, T-017 bloccata).
- T-MERGE-004 completata (Georgios): fast-forward merge di feature/monitoring-fe-scaffold in feature/monitoring (FE). 50 file, +1080 righe, nessun conflitto. Build production: SUCCESS. Pushed su origin/feature/monitoring. Wave 0 al 71% (5/7 task completate).
- **T-MERGE-001-002 completata (Alexios)**: mergiati branch T-001 (feature/monitoring-structures) e T-002 (origin/feature/monitoring-entities-spread-mailing) nel branch feature/monitoring-dto-repository-view. Conflitto risolto su MonitoringDocumentStateEnum (kept T-001 version with text descriptions).
- **T-003 completata (Alexios)**: Branch `feature/monitoring-dto-repository-view` creato da `feature/monitoring`, mergiati T-001 e T-002 nel branch. Creati: 7 DTO (MonitoringCovenantDTO, MonitoringEventDTO, SpreadTableDTO, SpreadHistoryDTO, MailingListContactDTO, MonitoringDocumentDTO, MonitoringPracticeSummaryDTO), 6 repository con JpaSpecificationExecutor (MonitoringCovenantRepository, MonitoringEventRepository, SpreadTableRepository, SpreadHistoryRepository, MailingListContactRepository, MonitoringDocumentRepository), 1 view entity (MonitoringPracticeSummaryView), SQL view estesa con deal_name, user names, expired/expiring/docs counts. Build: SUCCESS.
- Wave 0: 7/31 task completate (23%). Prossimo step: T-MERGE-003 (merge fondazioni BE) sbloccata.

### 2026-05-07
- **T-MERGE-003 completata (Alexios)**: `git pull origin feature/monitoring-dto-repository-view` su feature/monitoring. T-001 + T-002 + T-003 ora in feature/monitoring. T-006 e T-007 sbloccate.
- Wave 0 completata. 8/31 task completate (26%). Prossimo step: T-006 (macchina stati, Alexios) e T-007 (pratiche service, Davide) ora sbloccate.
- **T-006 completata (Alexios)**: Branch `feature/monitoring-state-machine`. Creati 3 service: MonitoringStateService, MonitoringIdGeneratorService, MonitoringScheduleGeneratorService. Build: SUCCESS.
- **T-007 completata (Davide)**: 5 file + createFromPostClosing + terminatePractice + trigger auto-creazione + 6 PSM states. Build: SUCCESS.
- **T-MERGE-006 completata (Davide)**: fast-forward merge T-006 + T-007 in feature/monitoring. Build: SUCCESS. T-008 sbloccata.

### 2026-05-19
- **Audit cross-branch (Davide, BE Senior)**: verifica stato reale di 4 task marcate come "Da iniziare" o "In corso" nel PROGRESS ma con commit presenti sui branch remoti. Risultati:
  - **T-010** (Dashboard FE) — passata da 0% Da iniziare a **95% Da revieware**. Tutti i requisiti del piano implementati (donut chart 6 segmenti con palette esatta, polling 60s pattern corretto, 3 tabelle con conteggio colonne 13/13/12). Autore reale del commit: Georgios (non Carmine). Manca solo test unitario (`dashboard.component.spec.ts`).
  - **T-017** (Deloitte flow) — riassesata da 90% In corso a **85% In corso**. Implementazione funzionalmente corretta (role-gating + CC assignment dialog + FormGroup reattivo) ma wizard a 4 step invece dei 6 step espliciti previsti dal piano (Step 4/5/6 collassati in unico "Assegna"). Mancano test (`cc-assignment.component.spec.ts`, `document-history.component.spec.ts`).
  - **T-019** (Eccezioni) — passata da 0% Da iniziare a **50% In corso**. BE 95% (7 test passanti, branch BE behind di feature/monitoring → serve rebase). FE 30%: il dialog `exception-dialog.component` esiste con 33 test passanti ma è completamente DISCONNESSO dal flusso utente — `MonitoringExceptionDialogModule` non importato in `MonitoringPracticeDetailModule`, selettore mai usato in template parent, action column ha label statica senza click handler, e `monitoring.service.requestException` invia JSON invece di multipart (415/400 a runtime). **BLOCKER per T-MERGE-FINAL**.
  - **T-020** (Covno upload) — passata da 0% Da iniziare a **85% Da revieware**. Implementazione BE (CovnoUploadService con parseXlsx Apache POI, mergeData con priorità A-011, CovnoUploadHistory entity, migration 005, MonitoringDataSourceEnum) + FE (card dashboard COVNO, dialog upload, dialog storico, i18n IT/EN 22 chiavi) completa. Autore: Alexios. Mancano test BE (CovnoUploadServiceTest) e FE.
- **Disallineamento PROGRESS rilevato**: alcuni sviluppatori (Adham, Alexios, Carmine→Georgios) hanno pushato codice senza aggiornare il PROGRESS.md. Procedura "Aggiornamento del progresso" della skill sdlc-executor da rinforzare.
- **Conseguenza per T-MERGE-FINAL**: bloccata da T-019 (FE wiring mancante). T-MERGE-FINAL può partire solo dopo fix integrazione FE eccezioni.
- **Sblocchi**: T-018 (GenAI) tecnicamente sbloccata (T-017 sostanzialmente completa, gap solo su test/UX).
