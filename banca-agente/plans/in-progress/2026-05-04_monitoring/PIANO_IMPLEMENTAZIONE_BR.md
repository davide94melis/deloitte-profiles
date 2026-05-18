# Piano Implementazione Modulo Monitoraggio (BR V6)

Data: `2026-05-04`

Assunzioni:
- Perimetro: intero modulo Monitoraggio come descritto nel BR V6. ESCLUSO tutto ciò che è nel piano forbice (`plans/in-progress/2026-05-03_forbice/`)
- Branch base: `feature/monitoring` (da `develop`)
- Review BR completata con br-clarify round 1 (2026-04-30): 5 bloccanti risolti. Round 2 (2026-05-05): 13 risposte non bloccanti ricevute (12 confermano assunzioni, 1 rigettata A-014). 3 bloccanti aggiornati (B2 condizioni, B3 macchina stati v2, B4 semaforo 6 colori). Dettagli in REVIEW_BR.md e GAP_REPORT_BR.md
- Davide e Carmine NON disponibili settimana 1
- Ahmad e Adham SENZA Claude Code → task ultra-dettagliate con istruzioni passo-passo
- Team disponibile:
  - Davide — BE Senior, Claude Code
  - Carmine — FE Senior, Claude Code
  - Alexios — Fullstack Mid, Claude Code
  - Adham — FE Junior, NO Claude Code
  - Georgios — FE Junior, Claude Code
  - Ahmad — BE Junior, NO Claude Code

## Obiettivo

Implementare il modulo Monitoraggio completo: dashboard, tabella pratiche, dettaglio pratica (ISP + Deloitte), macchina a stati 4 livelli, spread, scadenziere, mailing list, eccezioni, upload COVNO, notifiche email, scheduled jobs. Massimizzare il parallelismo tra BE e FE, usando la settimana 1 (senza Senior) per le fondazioni.

## Strategia di esecuzione

1. **Wave 0 (Settimana 1)**: Fondazioni BE (entità, enum, migration, DTO, repository) + scaffold FE (modulo, routing, models, services). Alexios guida il BE, Georgios il FE, Ahmad e Adham su task guidate.
2. **Wave 1 (Settimana 2)**: Core BE (macchina stati, servizi, controller). Davide e Alexios in parallelo su BE. Carmine inizia dashboard FE.
3. **Wave 2 (Settimane 2-3)**: Feature in parallelo: dashboard, tabella pratiche, dettaglio ISP, scadenziere. Massimo parallelismo BE/FE.
4. **Wave 3 (Settimane 3-4)**: Spread, flusso Deloitte, eccezioni, upload COVNO.
5. **Wave 4 (Settimana 5)**: Mailing list, notifiche email, scheduled jobs.
6. **Wave 5 (Settimana 6)**: Integrazione, polish, UAT.

Punto di congelamento iniziale: le entità JPA + migration della Wave 0 bloccano tutto il BE. La scaffold FE blocca tutto il FE.

## Distribuzione team consigliata

- **Davide** (BE Senior): governance architetturale, macchina stati, review codice BE, sblocco tecnico. Non caricato di implementazione continua — il suo valore è nel design e review.
- **Carmine** (FE Senior): architettura FE monitoring, dashboard, dettaglio pratica, review codice FE. Stesso principio.
- **Alexios** (Fullstack Mid): fondazioni BE settimana 1, poi stream core BE + feature BE/FE. Riferimento tecnico quando i senior non sono disponibili.
- **Georgios** (FE Junior, Claude Code): scaffold FE settimana 1, poi pagine feature (tabella pratiche, scadenziere, spread).
- **Ahmad** (BE Junior, no Claude Code): entità semplici settimana 1, poi endpoint BE guidati con istruzioni passo-passo.
- **Adham** (FE Junior, no Claude Code): i18n + pagine semplici settimana 1, poi tabelle e form con istruzioni dettagliate.

## Definizione degli Stream

- `stream-fondazioni` — entità JPA, enum, migration SQL, DTO, repository, vista SQL
- `stream-fe-scaffold` — modulo Angular monitoring, routing, models TS, enum TS, services, i18n
- `stream-core-be` — macchina stati 4 livelli, generatore ID, generatore scadenziere, service + controller pratiche/covenant/eventi
- `stream-dashboard` — endpoint BE summary + pagina FE dashboard (donut, tabelle, polling, card COVNO)
- `stream-pratiche` — endpoint BE tabella pratiche + pagina FE (filtri, paginazione, export, azioni)
- `stream-dettaglio-isp` — pagina FE dettaglio pratica ISP (header, tab covenant, upload CC, storico docs, dettaglio covenant)
- `stream-scadenziere` — endpoint BE + tab FE scadenziere unificato
- `stream-spread` — entità + service BE + tab FE spread (ISP read-only + Deloitte editabile)
- `stream-deloitte` — flusso Deloitte (conferma/rifiuta, assegnazione CC manuale/GenAI, modifica covenant)
- `stream-eccezioni` — modal eccezione ISP, gestione Deloitte, cambio stato
- `stream-covno` — upload XLSX COVNO/DB Obblighi, parsing, merge incrementale
- `stream-mailing-list` — CRUD contatti, upload AD Form, GenAI, report, pagine FE
- `stream-notifiche` — email templates, scheduled jobs, integrazione Ba-email-manager
- `stream-integrazione` — test E2E, export Excel, terminazione manuale, polish, UAT

## Backlog operativo

| ID | Stream | Owner | Area | Branch | Priorità | Attività | Descrizione | Dipendenze | Effort |
|---|---|---|---|---|---:|---|---|---|---:|
| `T-001` | `stream-fondazioni` | `Alexios` | BE | `feature/monitoring-structures` | P0 | Enum + Entità monitoring core | Creare: (1) `MonitoringPracticeStateEnum` (OK, IN_SCADENZA, DA_ATTENZIONARE, SCADUTO, FUORI_SOGLIA, CONCLUSA) in `enumeration/agency_desk/`, (2) `MonitoringEventStateEnum` (stessi + NESSUNO_STATO), (3) `MonitoringDocumentStateEnum` (IN_ATTESA_CONFERMA, VERIFICA_IN_CORSO, GENAI_IN_CORSO, GENAI_DA_VALIDARE, COMPLETATO), (4) Entità `MonitoringCovenant.java` in `domain/agency_desk/` con campi: id (Long, auto), covenantCode (String, M{n}C{n}), covenantType (CovenantTypeNumericEnum), valueLimitUpper (BigDecimal), valueLimitLower (BigDecimal), sign (CovenantSignEnum), periodicity (MonitoringPeriodicityEnum), variable (String), notes (String), practice (@ManyToOne Practice). Seguire pattern `BookingLender.java`. (5) Entità `MonitoringEvent.java` con campi: id, eventCode (M{n}C{n}E{n}), referenceDate (LocalDate), expirationDate (LocalDate), monitoringValue (BigDecimal), status (MonitoringEventStateEnum), actualDate (LocalDate), notes (String), hasException (boolean), exceptionMotivation (String), covenant (@ManyToOne MonitoringCovenant). (6) Migration SQL `ddl/init/agency-desk/wave-3/002_monitoring_foundation.sql`: CREATE TABLE mp_monitoring_covenant e mp_monitoring_event con FK + inserimento stati PSM monitoring in mp_psm_state. | Nessuna | `3 gg` |
| `T-002` | `stream-fondazioni` | `Ahmad` | BE | `feature/monitoring-entities-spread-mailing` | P0 | Entità Spread + MailingList + MonitoringDocument | Creare: (1) `SpreadTable.java` in `domain/agency_desk/` con: id, covenant (String, tipo covenant), lowerLimit (BigDecimal), upperLimit (BigDecimal), margin (BigDecimal, percentuale), practice (@ManyToOne), validated (boolean, default false). (2) `SpreadHistory.java` con: id, referenceDate (LocalDate), actualDate (LocalDate), monitoredValue (BigDecimal), cluster (String), margin (BigDecimal), practice (@ManyToOne). (3) `MailingListContact.java` con: id, name (String), email (String), role (String), practice (@ManyToOne). (4) `MonitoringDocument.java` con: id, documentCode (String), status (MonitoringDocumentStateEnum), uploadDate (LocalDateTime), fileName (String), fileSize (Long), uploadedBy (String), event (@ManyToOne MonitoringEvent). (5) Migration SQL `003_monitoring_spread_mailing.sql`: CREATE TABLE mp_spread_table, mp_spread_history, mp_mailing_list_contact, mp_monitoring_document. **ISTRUZIONI PASSO-PASSO per Ahmad**: (a) Copia `BookingLender.java` come template, (b) cambia il nome classe e i campi, (c) aggiungi @Entity @Table(name="mp_...") @Data @NoArgsConstructor, (d) per ogni campo: @Column(name="snake_case") tipo nomeCampo, (e) per le FK: @ManyToOne(fetch=FetchType.LAZY) @JoinColumn(name="fk_name") Tipo riferimento. (f) Per la migration SQL: segui il formato di `001_gap_0_foundation.sql` con CREATE TABLE IF NOT EXISTS. | Nessuna | `2.5 gg` |
| `T-MERGE-001-002` | `stream-fondazioni` | `Alexios` | BE | — | P0 | Merge entità fondazioni in feature/monitoring | Merge branches `feature/monitoring-structures` (T-001) e `feature/monitoring-entities-spread-mailing` (T-002) in `feature/monitoring`. Verificare build: `mvn clean compile -DskipTests`. Verificare che le migration SQL 002 e 003 siano compatibili e in ordine. Risolvere eventuali conflitti. | T-001, T-002 | `0.5 gg` |
| `T-003` | `stream-fondazioni` | `Alexios` | BE | `feature/monitoring-dto-repository-view` | P0 | DTO + Repository + Vista SQL estesa | Creare: (1) DTO in `dto/agency_desk/`: MonitoringCovenantDTO, MonitoringEventDTO, SpreadTableDTO, SpreadHistoryDTO, MailingListContactDTO, MonitoringDocumentDTO, MonitoringPracticeSummaryDTO (con conteggi aggregati: countExpired, countExpiring, countDocsInProgress, semaphoreColor). Seguire pattern `BookingLenderDTO.java`. (2) Repository in `repository/agency_desk/`: MonitoringCovenantRepository, MonitoringEventRepository, SpreadTableRepository, SpreadHistoryRepository, MailingListContactRepository, MonitoringDocumentRepository. Tutti extends JpaRepository + JpaSpecificationExecutor. (3) Estendere `vw_monitoring_practice_summary` in `999_views/001_view_monitoring_practice_summary.sql`: aggiungere LEFT JOIN a mp_monitoring_event per conteggi (expired_count, expiring_count), LEFT JOIN mp_monitoring_document per docs_in_progress_count, aggiungere deal_name da mp_practice, aggiungere user info (bank_user_name, deloitte_user_name). | T-MERGE-001-002 | `2 gg` |
| `T-004` | `stream-fe-scaffold` | `Georgios` | FE | `feature/monitoring-fe-scaffold` | P0 | Modulo Angular monitoring + scaffold | Creare: (1) `module/agency-desk/monitoring/` con: monitoring.module.ts, monitoring-routing.module.ts, monitoring.component.ts/html/scss. (2) Routing lazy-loaded: aggiungere in `shared/const/routes/consultant-route.const.ts` e `admin-route.const.ts` path `monitoring` con loadChildren → MonitoringModule. (3) Sottopagine routing: dashboard, practices, practice/:id, mailing-list. (4) Models in `shared/model/monitoring/`: monitoring-practice.model.ts, monitoring-covenant.model.ts, monitoring-event.model.ts, spread.model.ts, mailing-list-contact.model.ts. (5) Enums in `shared/enum/`: monitoring-practice-state.enum.ts (OK, IN_SCADENZA, DA_ATTENZIONARE, SCADUTO, FUORI_SOGLIA, CONCLUSA), monitoring-semaphore.enum.ts (GREEN, YELLOW, ORANGE, RED, DARK_RED, GREY con mapping 1:1 colore→stato — 6 colori distinti da B4). (6) Service in `service/`: monitoring.service.ts (endpoints pratiche, covenant, eventi, dashboard), monitoring-spread.service.ts, monitoring-mailing-list.service.ts. Seguire pattern `PracticeService` + `RestService`. (7) Aggiungere voce menu sidebar "Pratiche Monitoraggio". | Nessuna | `3 gg` |
| `T-005` | `stream-fe-scaffold` | `Adham` | FE | `feature/monitoring-fe-i18n-semaphore` | P0 | i18n labels + componenti condivisi monitoring | [AGGIORNATO 2026-05-05: semaforo da 4 a 6 colori distinti (B4 corretto)] (1) Aggiungere in `assets/i18n/it-IT.json` e `en-GB.json` tutte le chiavi monitoring: monitoring.dashboard.title, monitoring.practices.title, monitoring.practice.detail.title, monitoring.covenant.*, monitoring.event.*, monitoring.spread.*, monitoring.mailingList.*, monitoring.status.ok/inScadenza/daAttenzionare/scaduto/fuoriSoglia/conclusa, monitoring.actions.terminatePractice/uploadCC/exception/skipMonitoring. (2) Creare `shared/ui/base-element/semaphore/` — componente semaforo con **6 colori distinti** (cerchio colorato + tooltip stato). Input: state (MonitoringPracticeStateEnum). Mapping: OK→verde(#4CAF50), IN_SCADENZA→giallo(#FFC107), DA_ATTENZIONARE→arancione(#FF9800), SCADUTO→rosso(#F44336), FUORI_SOGLIA→rosso scuro(#B71C1C), CONCLUSA→grigio(#9E9E9E). **ISTRUZIONI PASSO-PASSO per Adham**: (a) Per i18n: apri it-IT.json, trova la sezione appropriata, aggiungi chiavi nidificate sotto "monitoring". (b) Per il semafore: copia la struttura di un componente semplice (es. `badge/`), crea .ts con @Input() state, .html con un <span> con [ngClass] condizionale, .scss con classi .green/.yellow/.orange/.red/.dark-red/.grey e border-radius:50%. Ogni stato ha il suo colore unico. | Nessuna | `2.5 gg` |
| `T-MERGE-003` | `stream-fondazioni` | `Alexios` | BE | — | P0 | Merge fondazioni BE | Merge i seguenti branch in `feature/monitoring` nell'ordine indicato: (1) `feature/monitoring-structures` (T-001), (2) `feature/monitoring-entities-spread-mailing` (T-002), (3) `feature/monitoring-dto-repository-view` (T-003 — mergiare per ultimo, dipende da T-001 e T-002). Verificare che la build compili senza errori: `mvn clean compile -DskipTests`. Verificare che le migration SQL siano in ordine (002, 003 dopo 001). Risolvere eventuali conflitti. | T-003 | `0.5 gg` |
| `T-MERGE-004` | `stream-fe-scaffold` | `Georgios` | FE | — | P0 | Merge FE scaffold | Merge i seguenti branch in `feature/monitoring`: (1) `feature/monitoring-fe-scaffold` (T-004), (2) `feature/monitoring-fe-i18n-semaphore` (T-005). Verificare build FE: `ng build --configuration=production`. Verificare che il routing funzioni e il modulo si carichi correttamente. | T-004, T-005 | `0.5 gg` |
| `T-006` | `stream-core-be` | `Alexios` | BE | `feature/monitoring-state-machine` | P0 | Macchina stati 4 livelli + generatore ID + scadenziere | [AGGIORNATO 2026-05-05: condizioni chiusura B2 aggiornate, formule A-012 dettagliate, riferimento macchina stati v2] Creare: (1) `service/monitoring/MonitoringStateService.java`: metodo `calculateEventState(MonitoringEvent)` — calcola stato evento da: scadenza vs today (7gg=InScadenza, passata=Scaduto), valore vs soglia con formule precise da A-012: **Condizione A** (10%): segno `<`/`<=` → `(soglia - 10%*soglia) <= valore < soglia`; segno `>`/`>=` → `(soglia + 10%*soglia) >= valore > soglia`. **Condizione B** (30%+40%): segno `<`/`<=` → `(soglia - 30%*soglia) <= valore[t] < soglia AND valore[t] >= 140%*valore[t-1]`; segno `>`/`>=` → `(soglia + 30%*soglia) >= valore[t] > soglia AND valore[t] <= 60%*valore[t-1]`. A e B in OR → DaAttenzionare. Eccezione approvata=OK. Riferimento: `202604_Macchina Stati Monitoring_v2.xlsx` sheet "Singolo Covenant". Metodo `calculateCovenantState(MonitoringCovenant)` — aggrega: FuoriSoglia > Scaduto > InScadenza > DaAttenzionare > OK (stessa logica priorita' della pratica, confermato B3). Metodo `calculatePracticeState(Practice)` — stessa aggregazione su tutti i covenant. Metodo `recalculateAllStates(Long practiceId)` — ricalcola bottom-up evento→covenant→pratica. Metodo `canAutoClose(Practice)` — 3 condizioni (B2 aggiornato): (1) Data scadenza operazione < Today, (2) Data scadenza piu' lontana tra gli eventi < Today, (3) nessun evento con status Scaduto/FuoriSoglia/InScadenza. (2) `service/monitoring/MonitoringIdGeneratorService.java`: generatePracticeId() → "M{nextVal}", generateCovenantId(practiceCode, covenantIndex) → "M{n}C{n}", generateEventId(covenantCode, eventIndex) → "M{n}C{n}E{n}". Sequenziale progressivo per pratica/covenant/evento. (3) `service/monitoring/MonitoringScheduleGeneratorService.java`: generateEvents(covenant) → crea N eventi da data inizio fino a terminationDate in base a periodicità (MENSILE=1m, TRIMESTRALE=3m, SEMESTRALE=6m, ANNUALE=12m). Per ogni evento: referenceDate = data calcolo, expirationDate = data scadenza, status = NESSUNO_STATO. | T-MERGE-003 | `4.5 gg` |
| `T-007` | `stream-core-be` | `Davide` | BE | `feature/monitoring-practice-service` | P0 | Service + Controller pratiche monitoring | Creare: (1) `service/monitoring/MonitoringPracticeService.java`: createFromPostClosing(Long postClosingPracticeId) — crea pratica monitoring collegata via Tracking ID, copia utenti, genera ID M{n}. findAll(filters, pageable) — filtri: utenteBanca, trackingId, idMonitoraggio, nomeDeal, semaforo (multi-select). terminatePractice(Long practiceId) — cambio stato a CONCLUSA. (2) `controller/monitoring/MonitoringPracticeController.java`: GET /api/v1/monitoring/practices (paginato, filtrato), GET /api/v1/monitoring/practices/{id}, POST /api/v1/monitoring/practices/{id}/terminate, GET /api/v1/monitoring/practices/export (Excel). Seguire pattern `PracticeController.java`. (3) Trigger auto-creazione: in `PsmStateService.java` o `SystemEventListener.java`, aggiungere listener per transizione Post-Closing → completato che invoca createFromPostClosing(). | T-MERGE-003 | `3 gg` |
| `T-MERGE-006` | `stream-core-be` | `Davide` | BE | — | P0 | Merge core BE parziale (macchina stati + pratiche) | Merge branches `feature/monitoring-state-machine` (T-006) e `feature/monitoring-practice-service` (T-007) in `feature/monitoring`. Review approfondita del codice macchina stati prima del merge. Verificare build + test. Questo merge sblocca T-008 che dipende da MonitoringStateService. | T-006, T-007 | `0.5 gg` |
| `T-008` | `stream-core-be` | `Ahmad` | BE | `feature/monitoring-covenant-events-service` | P1 | Service + Controller covenant + eventi | Creare: (1) `service/monitoring/MonitoringCovenantService.java`: findByPractice(Long practiceId), create(MonitoringCovenantDTO), update(Long id, MonitoringCovenantDTO), delete(Long id). Per ogni create: invocare scheduleGenerator per creare eventi. (2) `service/monitoring/MonitoringEventService.java`: findByCovenant(Long covenantId), findExpiredByPractice(Long practiceId), findExpiringByPractice(Long practiceId), findAllByPractice(Long practiceId, Pageable). updateValue(Long eventId, BigDecimal value, LocalDate actualDate) — aggiorna valore e ricalcola stato. requestException(Long eventId, String motivation) — imposta hasException=true, stato=OK (A-013). (3) `controller/monitoring/MonitoringCovenantController.java`: CRUD su /api/v1/monitoring/practices/{practiceId}/covenants e /api/v1/monitoring/covenants/{id}/events. **ISTRUZIONI PASSO-PASSO per Ahmad**: (a) Crea il service con @Service @RequiredArgsConstructor, inietta il repository. (b) Per ogni metodo: chiama repository.findBy...(), converto con mapper, return DTO. (c) Per il controller: @RestController @RequestMapping, @GetMapping/@PostMapping, chiama service, return ResponseEntity. Segui il pattern di `PracticeController.java`. | T-MERGE-006 | `3 gg` |
| `T-MERGE-008` | `stream-core-be` | `Davide` | BE | — | P1 | Merge core BE | Merge i seguenti branch in `feature/monitoring` nell'ordine indicato: (1) `feature/monitoring-state-machine` (T-006), (2) `feature/monitoring-practice-service` (T-007), (3) `feature/monitoring-covenant-events-service` (T-008 — mergiare per ultimo, dipende da T-006). Verificare build + run test. Review codice macchina stati (T-006) prima del merge. | T-008 | `0.5 gg` |
| `T-009` | `stream-dashboard` | `Ahmad` | BE | `feature/monitoring-dashboard-be` | P1 | Dashboard monitoring BE endpoints | [AGGIORNATO 2026-05-05: donut 6 segmenti (B4 corretto)] Creare: (1) `service/monitoring/MonitoringDashboardService.java`: getSummary() — conta pratiche per stato (**6 segmenti donut**: OK, InScadenza, DaAttenzionare, Scaduto, FuoriSoglia, Conclusa), getExpiredEvents(Pageable) — eventi scaduti/fuori soglia paginati, getExpiringEvents(Pageable) — eventi in scadenza paginati, getRecentPractices(limit=10). (2) `controller/monitoring/MonitoringDashboardController.java`: GET /api/v1/monitoring/dashboard/summary, GET /api/v1/monitoring/dashboard/expired?page=0&size=5, GET /api/v1/monitoring/dashboard/expiring?page=0&size=5, GET /api/v1/monitoring/dashboard/recent?limit=10. (3) DTO: MonitoringDashboardSummaryDTO (countOk, countExpiring, countToWatch, countExpired, countOutOfThreshold, countClosed, total), MonitoringDashboardEventDTO (13 colonne: idPratica, trackingId, nomeDeal, idCovenant, idEvento, tipologia, note, dataScadenza, giorniAlProssimoMonitoraggio, valoreLimite, valoreMonitoraggio, statusMonitoraggio, azione). **ISTRUZIONI per Ahmad**: stesso pattern di T-008, crea service che usa repository con query custom (@Query JPQL), controller espone gli endpoint REST. | T-MERGE-008 | `2 gg` |
| `T-010` | `stream-dashboard` | `Carmine` | FE | `feature/monitoring-dashboard-fe` | P1 | Dashboard monitoring FE | [AGGIORNATO 2026-05-05: donut da 4 a 6 segmenti (B4 corretto)] Creare pagina dashboard in `module/agency-desk/monitoring/dashboard/`: (1) Selettore Censimento/Monitoraggio in alto a destra (toggle button o p-selectButton). (2) `app-doughnut` con **6 segmenti**: OK(verde #4CAF50), InScadenza(giallo #FFC107), DaAttenzionare(arancione #FF9800), Scaduto(rosso #F44336), FuoriSoglia(rosso scuro #B71C1C), Concluse(grigio #9E9E9E). Totale al centro. Dati da MonitoringService.getDashboardSummary(). (3) Polling: `interval(60000).pipe(switchMap(() => service.getDashboardSummary()))` con `takeUntil(destroy$)`. (4) Tabella "Monitoraggi Scaduti" (max 5 righe): usa `app-table` con 13 colonne (vedi T-009 DTO), `[pageRows]="5"` senza paginator, bottone "Vedi tutte" → link a pagina dedicata. (5) Tabella "Monitoraggi In Scadenza" (max 5 righe): stesse colonne di Scaduti (A-003 confermata). (6) Tabella "Pratiche Recenti" (ultime 10): 12 colonne (ID, Tracking ID, Nome Deal, Utente Banca, Utente Deloitte, Tipologia, #Scaduti, #InScadenza, #DocInLavorazione, Semaforo con `app-semaphore` a 6 colori, Data Creazione, Azione "Termina"). "Vedi tutte" → link a tabella pratiche. | T-MERGE-004, T-009 | `3.5 gg` |
| `T-011` | `stream-pratiche` | `Georgios` | FE | `feature/monitoring-practices-table-fe` | P1 | Tabella pratiche monitoring FE | Creare pagina in `module/agency-desk/monitoring/practices/`: (1) Filtri: Utente Banca (text), Tracking ID (text), ID Monitoraggio (text), Nome Deal (text), Semaforo (p-multiSelect con 6 stati). (2) `app-table` server-paginated con `[isServerPaginated]="true"` `[pageRows]="20"` `(onLazyLoad$)="loadPractices($event)"`. (3) 13 colonne: ID (M{n}), Tracking ID, Nome Deal, Utente Banca, Utente Deloitte, Tipologia, #Scaduti (badge rosso), #InScadenza (badge giallo), #DocInLavorazione, Semaforo (`app-semaphore`), Data Creazione, Data Fine, Azione ("Termina Pratica di Monitoraggio"). (4) Click riga → navigazione a `/monitoring/practice/{id}`. (5) Export Excel: bottone che chiama MonitoringService.exportPracticesExcel() e download blob. Seguire pattern di `PracticeService.exportPracticesExcel()`. | T-MERGE-004, T-MERGE-008 | `2.5 gg` |
| `T-012` | `stream-dettaglio-isp` | `Adham` | FE | `feature/monitoring-practice-detail-isp` | P1 | Dettaglio pratica ISP — header + tab scaffold | Creare pagina in `module/agency-desk/monitoring/practice-detail/`: (1) Header: ID Pratica, NDG, Tipologia, ID Contratto, Utente Banca, Stato (con `app-semaphore`). Dati da MonitoringService.getPractice(id). (2) 3 tab con PrimeNG TabView: "Covenant", "Spread", "Scadenziere". (3) Tab Covenant — overview cards: Card "Totale Covenant" (numero totale), Card "Scadenze" (3 sotto-card: OK/verde, InScadenza/giallo, Scaduto/rosso con conteggi), Card "Avanzamento" (Sezione, IN CARICO A, STATO LAVORAZIONE). (4) Sotto la overview: 3 tabelle comprimibili (p-accordion): "Monitoraggi Scaduti/Fuori Soglia" (12 col + azione Salta/Ignora), "Monitoraggi In Scadenza" (stesse colonne), "Tutti i Covenant" (13 col aggregate). **ISTRUZIONI per Adham**: (a) Crea component con `ng generate component monitoring/practice-detail`. (b) Nel .ts: inietta ActivatedRoute per il practiceId, MonitoringService per i dati, AuthService per il ruolo. (c) Nel .html: usa `<p-tabView>` con 3 `<p-tabPanel>`. (d) Per le card: usa flexbox con div + classe CSS. (e) Per le tabelle: usa `<app-table [thead]="..." [tbody]="..."`. (f) I dati thead sono array di `{field, header, sortable}`. | T-MERGE-004, T-MERGE-008 | `4 gg` |
| `T-013` | `stream-scadenziere` | `Georgios` | BE+FE | `feature/monitoring-schedule-covenant-detail` | P1 | Scadenziere unificato + dettaglio covenant singolo | BE: (1) Aggiungere a MonitoringEventService: findAllByPractice(practiceId, covenantType, Pageable) — tutti gli eventi della pratica ordinati cronologicamente, filtrabili per tipologia. (2) Endpoint: GET /api/v1/monitoring/practices/{id}/schedule?covenantType=&page=&size=20. FE: (1) Tab "Scadenziere" nella pagina dettaglio pratica: dropdown filtro tipologia covenant, `app-table` con 9 colonne (ID Evento, ID Covenant, Tipologia, Data Riferimento, Data Scadenza, Giorni al prossimo, Valore, Status con semaforo, Azione). Paginazione 20 righe. (2) Pagina/modal "Dettaglio Covenant Singolo": stato avanzamento (tipologia, valore limite, variabile, note, monitoraggi da svolgere), 3 semafori circolari (OK/InScadenza/Scaduti con conteggi), scadenziere monitoraggi del covenant (11 colonne con tutti gli eventi). Accessibile da click su riga tabella "Tutti i Covenant". | T-MERGE-004, T-MERGE-008 | `3 gg` |
| `T-014` | `stream-dettaglio-isp` | `Alexios` | BE+FE | `feature/monitoring-upload-cc-documents` | P1 | Upload Compliance Certificate + storico documenti | BE: (1) `service/monitoring/MonitoringDocumentService.java`: upload(practiceId, eventId, MultipartFile), getDocumentHistory(practiceId), confirmDocument(docId), rejectDocument(docId, reason). (2) `controller/monitoring/MonitoringDocumentController.java`: POST /api/v1/monitoring/documents/upload (multipart), GET /api/v1/monitoring/practices/{id}/documents, PATCH /api/v1/monitoring/documents/{id}/confirm, PATCH /api/v1/monitoring/documents/{id}/reject. (3) Integrazione con Ba-document-manager se necessario (upload S3), altrimenti storage locale seguendo pattern `BucketS3PracticeData`. FE: (1) Sezione upload CC nel tab Covenant: `app-file-upload` con accept=".pdf,.doc,.docx,.xlsx", maxFileSize=3MB, drag&drop. (2) Storico documenti: tabella con colonne Status (badge colorato), Giorni Giacenza (calcolato FE), Nome File, Data Upload, Azioni (Visualizza/Download/Commento). | T-MERGE-004, T-MERGE-008 | `3 gg` |
| `T-015` | `stream-spread` | `Ahmad` | BE | `feature/monitoring-spread-be` | P2 | Spread BE — service + controller (2 tabelle distinte) | [AGGIORNATO 2026-05-05: A-014 rigettata — "Tabella Spread" e "Tabella storico Spread" sono due oggetti distinti] Creare: (1) `service/monitoring/MonitoringSpreadService.java` con due servizi logici: **Tabella Spread (sez. 7.3.5, Deloitte)**: getSpreadTable(practiceId), createSpreadRow(SpreadTableDTO) — campi: Data Riferimento, Data avvenuto monitoraggio, Valore Monitorato, Cluster di Riferimento, Valore Margine (%), updateSpreadRow(id, SpreadTableDTO), deleteSpreadRow(id), validateSpread(practiceId) — flag validated=true. **Tabella storico Spread (sez. 3.5.2, ISP)**: getSpreadHistory(practiceId) — campi: Data Variazione, Documento (nome file), Azione (link download), Cluster, addSpreadHistoryEntry(SpreadHistoryDTO) — aggiunge riga + trigger notifica ISP. (2) `controller/monitoring/MonitoringSpreadController.java`: CRUD su /api/v1/monitoring/practices/{id}/spread (Tabella Spread Deloitte), GET /api/v1/monitoring/practices/{id}/spread/history (storico ISP), PATCH .../spread/validate, GET .../spread/history/{id}/download (download documento storico). **ISTRUZIONI per Ahmad**: (a) Service: @Service, inietta SpreadTableRepository e SpreadHistoryRepository. (b) Le due tabelle hanno DTO diversi perche' le colonne sono diverse: SpreadTableDTO per la tabella Deloitte, SpreadHistoryDTO per lo storico ISP. (c) validateSpread: trova tutte le righe della pratica, imposta validated=true, salva. (d) Controller: @RestController @RequestMapping("/api/v1/monitoring/practices/{practiceId}/spread"), segui pattern T-008. | T-MERGE-008 | `3 gg` |
| `T-016` | `stream-spread` | `Georgios` | FE | `feature/monitoring-spread-fe` | P2 | Spread FE — 2 tabelle distinte (ISP + Deloitte) | [AGGIORNATO 2026-05-05: A-014 rigettata — implementare due tabelle con strutture diverse] **Tabella storico Spread (sez. 3.5.2, vista ISP)**: (1) Tab "Spread" nella pagina dettaglio pratica ISP: tabella read-only con colonne: Data Variazione, Documento (nome file), Azione (bottone "Scarica" → download file), Cluster. Visibile solo se ci sono entries nello storico. **Tabella Spread (sez. 7.3.5, vista Deloitte)**: (2) Vista Deloitte (condizionata da RoleEnum.Consultant): tabella editabile con colonne: Data Riferimento, Data avvenuto monitoraggio, Valore Monitorato, Cluster di Riferimento, Valore Margine (%). Azioni: Aggiungi Riga (bottone), Modifica (inline edit), Elimina (p-confirmDialog). Bottone "Valida Tabella" → PATCH validate. Banner disclaimer "Tabella non verificata" prima della validazione. "Nessuna tabella da inserire" → checkbox che disabilita la sezione. **Condivisione**: ISP vede la Tabella Spread solo dopo validazione Deloitte (validated=true), con colonne read-only. ISP vede sempre la propria Tabella storico Spread. | T-MERGE-004, T-015 | `4 gg` |
| `T-MERGE-W2` | `stream-integrazione` | `Davide` | BE+FE | — | P1 | Merge checkpoint Wave 2 | Merge tutti i branch Wave 2 in `feature/monitoring` (BE e FE): `feature/monitoring-practices-table-fe` (T-011), `feature/monitoring-practice-detail-isp` (T-012), `feature/monitoring-schedule-covenant-detail` (T-013), `feature/monitoring-upload-cc-documents` (T-014), `feature/monitoring-spread-be` (T-015), `feature/monitoring-spread-fe` (T-016). Verificare build completa BE + FE. Risolvere conflitti cross-stream. Questo merge sblocca Wave 3 (T-017 dipende da T-014). | T-011, T-012, T-013, T-014, T-015, T-016 | `1 gg` |
| `T-017` | `stream-deloitte` | `Alexios` | BE+FE | `feature/monitoring-deloitte-flow` | P2 | Flusso Deloitte — conferma/rifiuta + assegnazione CC manuale | BE: (1) In MonitoringDocumentService: conferma/rifiuta documento (aggiorna MonitoringDocumentStateEnum). FE: (1) Nella pagina dettaglio pratica, se ruolo=Consultant: aggiungere bottoni "Conferma"/"Rifiuta" su ogni documento nel storico. (2) Assegnazione CC manuale: dialog con step: Passo 1 → seleziona covenant (dropdown da lista), Passo 2 → inserisci tipologia + valore + data, Passo 3 → scadenziere filtrato per il covenant selezionato, Passo 4 → "Assegna" → salva valori nel covenant, Passo 5 → "Conferma" → aggiorna stato documento, Passo 6 → "Concludi". (3) Form reattivo (FormGroup) con validazione. | T-MERGE-W2 | `3 gg` |
| `T-018` | `stream-deloitte` | `Davide` | BE+FE | `feature/monitoring-deloitte-genai` | P2 | Flusso Deloitte — GenAI + modifica dati covenant | BE: (1) `service/monitoring/MonitoringGenAiService.java`: extractFromComplianceCertificate(docId) → invoca servizio GenAI (seguire pattern `GenAiFormKeyEnum` e servizi AI esistenti in `service/ai/`), restituisce proposta di valori per i covenant. (2) Endpoint: POST /api/v1/monitoring/documents/{id}/genai-extract, GET /api/v1/monitoring/documents/{id}/genai-result. FE: (1) Flusso GenAI: bottone "Estrai con GenAI" → spinner (stato GENAI_IN_CORSO) → N selettori con proposta valore per ogni covenant → bottoni "Modifica Dati" / "Conferma" per ognuno. Seguire pattern `GenaiExtractionComponent` (`shared/ui/section-element/genai-extraction/`). (2) "Modifica Dati Covenant": dialog con form: tipologia (dropdown CovenantTypeNumericEnum), valore limite (number input), variabile (text), scadenze future (lista editable), aggiungi/rimuovi monitoraggio. | T-017 | `4 gg` |
| `T-019` | `stream-eccezioni` | `Adham` | FE | `feature/monitoring-exceptions` | P2 | Eccezioni ISP + gestione Deloitte | FE: (1) ISP — bottone "Salta Monitoraggio"/"Ignora Soglia" su ogni evento scaduto/in scadenza: apre modal PrimeNG Dialog "Richiesta di Eccezione" con: textarea motivazione (2-2000 char, validazione minLength/maxLength), upload opzionale (`app-file-upload`), bottoni Annulla/Invia. (2) Dopo conferma: evento passa a Green (A-013), azione diventa "Visualizza Eccezione" (apre modal read-only con motivazione + file). (3) Deloitte — sulla stessa vista: "Annulla Eccezione" (ripristina stato precedente). BE: [AGGIORNATO 2026-05-05: A-013 confermata con correzione — valore resta vuoto, non "N/A"] (1) In MonitoringEventService: requestException(eventId, motivation, file) → hasException=true, exceptionMotivation=motivation, status=OK, actualDate=now, monitoringValue=null (vuoto, non "N/A"). cancelException(eventId) → reset a stato calcolato. (2) Endpoint: POST /api/v1/monitoring/events/{id}/exception, DELETE /api/v1/monitoring/events/{id}/exception. **ISTRUZIONI per Adham**: (a) Per il modal: usa PrimeNG `<p-dialog [visible]="showExceptionDialog">`. (b) Dentro il dialog: `<textarea pInputTextarea [(ngModel)]="motivation" minlength="2" maxlength="2000">`. (c) Per l'upload nel dialog: includi `<app-file-upload>` con (onSelect)="onFileSelect($event)". (d) Al click "Invia": chiama monitoringService.requestException(eventId, {motivation, file}). | T-MERGE-004, T-MERGE-008 | `2.5 gg` |
| `T-020` | `stream-covno` | `Alexios` | BE+FE | `feature/monitoring-covno-upload` | P2 | Upload COVNO/DB Obblighi + card dashboard | BE: (1) `service/monitoring/CovnoUploadService.java`: parseXlsx(MultipartFile) → legge file XLSX, estrae righe per Tracking ID. mergeData(parsed, practiceId) → aggiornamento incrementale: file vince su dati automatici, NON vince su manuali Deloitte (A-011). saveUploadHistory(practiceId, fileName, userId, result). (2) `controller/monitoring/CovnoController.java`: POST /api/v1/monitoring/covno/upload (multipart), GET /api/v1/monitoring/covno/history. Per parsing XLSX: usare Apache POI (già nel progetto) o libreria esistente. FE: (1) Card COVNO nella dashboard: data ultimo aggiornamento, bottone "Aggiorna" → apre dialog con `app-file-upload` accept=".xlsx", "Storico" → lista upload precedenti. (2) Entità `CovnoUploadHistory.java`: id, uploadDate, fileName, uploadedBy, result (SUCCESS/PARTIAL/FAILED), recordsProcessed, recordsUpdated. | T-MERGE-008 | `3 gg` |
| `T-021` | `stream-dashboard` | `Adham` | FE | `feature/monitoring-expired-expiring-pages` | P2 | Pagine complete Scaduti + In Scadenza | Creare 2 pagine dedicate raggiungibili da "Vedi tutte" nella dashboard: (1) `monitoring/expired-events/`: tabella con 13 colonne (stesse della dashboard), filtri avanzati (ID Pratica, Tracking ID, Nome Deal, Tipologia covenant, range date), `app-table` server-paginated 20 righe, export Excel. (2) `monitoring/expiring-events/`: stesse colonne e filtri (A-003). Aggiungere route nel monitoring-routing.module.ts. **ISTRUZIONI per Adham**: (a) Copia la struttura della pagina tabella pratiche (T-011) come template. (b) Cambia i filtri e le colonne. (c) Per l'export: chiama service.exportExpiredEventsExcel() → httpClient.get con responseType:'blob', poi crea un link temporaneo con URL.createObjectURL(blob) e click() per scaricare. | T-010, T-MERGE-008 | `2 gg` |
| `T-022` | `stream-mailing-list` | `Georgios` | BE+FE | `feature/monitoring-mailing-list` | P2 | Mailing List completo | BE: (1) `service/monitoring/MailingListService.java`: CRUD contatti, uploadAdForm(practiceId, file) → integrazione GenAI per estrazione contatti, generateReport(practiceId). (2) `controller/monitoring/MailingListController.java`: CRUD /api/v1/monitoring/practices/{id}/mailing-list, POST .../ad-form/upload, POST .../ad-form/extract, POST .../ad-form/confirm. (3) Stati AD Form: DOC_INCOMPLETA → DOC_COMPLETA → DOC_CONFERMATA → VERIFICA_IN_CORSO → REPORT_GENERATO. FE: (1) Pagina ricerca pratica con filtri (Nome Deal, ID Censimento, Tracking ID, ID Monitoraggio). (2) Tabella pratiche mailing list (11 colonne + azione "Richiedi Aggiornamento"). (3) Upload AD Form: `app-file-upload`. (4) Vista Deloitte: bottoni Conferma/Rifiuta AD Form, "Estrai con GenAI", "Conferma e Aggiorna". (5) Report mailing list: tabella con contatti estratti. | T-MERGE-008, T-MERGE-004 | `5 gg` |
| `T-023` | `stream-notifiche` | `Davide` | BE+EM | `feature/monitoring-notifications-jobs` | P2 | Email templates + scheduled jobs | BE: (1) Aggiungere a `EmailTemplateEnum.java`: MONITORING_EXPIRATION_ALERT, MONITORING_EXPIRED, MONITORING_EXCEPTION_REQUESTED, MONITORING_EXCEPTION_HANDLED, MONITORING_SPREAD_VARIATION, MONITORING_DOCUMENT_REJECTED, MONITORING_PRACTICE_CLOSED. (2) `service/monitoring/MonitoringSchedulerService.java`: checkExpiringEvents() — trova eventi a 7gg dalla scadenza, invia notifica (A-015). checkExpiredEvents() — trova eventi scaduti oggi, invia notifica. autoClosePractices() — [AGGIORNATO 2026-05-05: B2 aggiornato con 3 condizioni] trova pratiche che soddisfano TUTTE e 3 le condizioni: (1) Data scadenza operazione < Today, (2) Data scadenza piu' lontana tra gli eventi di monitoraggio < Today, (3) nessun evento con status Scaduto/FuoriSoglia/InScadenza. Chiudi automaticamente impostando stato = CONCLUSA. recalculateStates() — ricalcola stati di tutti gli eventi attivi. (3) In `ScheduledService.java`: aggiungere 3 @Scheduled(cron="0 0 6 * * *") che invocano MonitoringSchedulerService. In `SchedulingJobNames.java` aggiungere costanti. (4) EM: creare template HTML in Ba-email-manager per ogni tipo (seguire pattern template esistenti). | T-MERGE-008 | `4 gg` |
| `T-MERGE-FINAL` | `stream-integrazione` | `Davide` | BE+FE | — | P2 | Merge finale pre-integrazione | Merge tutti i branch Wave 3+4 in `feature/monitoring`: `feature/monitoring-deloitte-flow` (T-017), `feature/monitoring-deloitte-genai` (T-018), `feature/monitoring-exceptions` (T-019), `feature/monitoring-covno-upload` (T-020), `feature/monitoring-expired-expiring-pages` (T-021), `feature/monitoring-mailing-list` (T-022), `feature/monitoring-notifications-jobs` (T-023). Verificare build completa BE + FE. Risolvere conflitti. Preparazione per integration testing e UAT. | T-017, T-018, T-019, T-020, T-021, T-022, T-023 | `1 gg` |
| `T-024` | `stream-integrazione` | `Davide` | BE+FE | `feature/monitoring-integration-uat` | P3 | Integration testing + UAT + polish | (1) Verificare flusso completo ISP: dashboard → tabella pratiche → dettaglio → tab covenant → upload CC → eccezione → spread → scadenziere. (2) Verificare flusso Deloitte: conferma/rifiuta → assegnazione CC → GenAI → modifica covenant → gestione eccezione → validazione spread. (3) Verificare scheduled jobs: chiusura automatica, notifiche, ricalcolo stati. (4) Export Excel su tutte le tabelle. (5) Azione "Termina Pratica di Monitoraggio" con dialog conferma. (6) Fix bug, polish UI, pulizia codice. (7) Review finale di Davide (BE) e Carmine (FE). | T-MERGE-FINAL | `3 gg` |

## Ordine di esecuzione

### Wave 0 — Fondazioni (Settimana 1, senza Davide e Carmine)

- `stream-fondazioni`: T-001 (Alexios, 3gg) + T-002 (Ahmad, 2.5gg, in parallelo) → **T-MERGE-001-002** (Alexios, 0.5gg, merge entità in feature/monitoring) → T-003 (Alexios, 2gg) → T-MERGE-003 (Alexios, 0.5gg)
- `stream-fe-scaffold`: T-004 (Georgios, 3gg) + T-005 (Adham, 2.5gg, in parallelo) → T-MERGE-004 (Georgios, 0.5gg)

### Wave 1 — Core BE + Dashboard FE (Settimana 2)

- `stream-core-be`: T-006 (Alexios, 4.5gg) + T-007 (Davide, 3gg, in parallelo) → **T-MERGE-006** (Davide, 0.5gg, merge macchina stati + pratiche) → T-008 (Ahmad, 3gg) → T-MERGE-008 (Davide, 0.5gg)
- `stream-dashboard`: T-009 (Ahmad, 2gg, dopo T-MERGE-008)
- `stream-dashboard`: T-010 (Carmine, 3.5gg, dopo T-009)

### Wave 2 — Features parallele (Settimane 2-3)

- `stream-pratiche`: T-011 (Georgios, 2.5gg)
- `stream-dettaglio-isp`: T-012 (Adham, 4gg) + T-014 (Alexios, 3gg)
- `stream-scadenziere`: T-013 (Georgios, 3gg, dopo T-011)
- `stream-spread`: T-015 (Ahmad, 3gg) → T-016 (Georgios, 4gg, dopo T-013)
- → **T-MERGE-W2** (Davide, 1gg, merge tutti i branch Wave 2)

### Wave 3 — Features avanzate (Settimane 3-4)

- `stream-deloitte`: T-017 (Alexios, 3gg, dopo T-MERGE-W2) → T-018 (Davide, 4gg)
- `stream-eccezioni`: T-019 (Adham, 2.5gg)
- `stream-covno`: T-020 (Alexios, 3gg, dopo T-017)
- `stream-dashboard`: T-021 (Adham, 2gg, dopo T-019)

### Wave 4 — Completamento (Settimana 5)

- `stream-mailing-list`: T-022 (Georgios, 5gg)
- `stream-notifiche`: T-023 (Davide, 4gg)
- → **T-MERGE-FINAL** (Davide, 1gg, merge tutti i branch Wave 3+4)

### Wave 5 — Integrazione e UAT (Settimana 6)

- `stream-integrazione`: T-024 (Davide + tutti, 3gg, dopo T-MERGE-FINAL)

## Dipendenze critiche

1. **T-001 + T-002 → T-MERGE-001-002 → T-003**: Le entità JPA devono essere mergiate in `feature/monitoring` prima che T-003 possa creare DTO/repository
2. **T-MERGE-003 → T-006, T-007**: Tutto il BE core dipende dalle fondazioni mergiate
3. **T-006 + T-007 → T-MERGE-006 → T-008**: La macchina stati e i servizi pratiche devono essere mergiati prima che T-008 (covenant) possa usare MonitoringStateService
4. **T-MERGE-004 → T-010, T-011, T-012, T-013**: Tutto il FE feature dipende dallo scaffold mergiato
5. **T-MERGE-008 → T-009**: Gli endpoint dashboard dipendono dai service core
6. **T-009 → T-010**: La dashboard FE dipende dagli endpoint BE
7. **T-011..T-016 → T-MERGE-W2 → T-017**: Il flusso Deloitte (T-017) dipende dal codice di T-014 (upload CC), reso disponibile tramite merge checkpoint Wave 2
8. **T-017 → T-018**: GenAI estende il flusso base Deloitte
9. **T-017..T-023 → T-MERGE-FINAL → T-024**: L'integration testing richiede che tutto il codice Wave 3+4 sia mergiato in `feature/monitoring`

## Piano per persona

### Davide (BE Senior — disponibile da settimana 2)
1. T-007 (Service+Controller pratiche, 3gg, settimana 2)
2. T-MERGE-006 (Merge macchina stati + pratiche + review, 0.5gg)
3. T-MERGE-008 (Merge core BE, 0.5gg)
4. T-018 (GenAI + modifica covenant, 4gg, settimane 3-4)
5. T-MERGE-W2 (Merge checkpoint Wave 2, 1gg)
6. T-023 (Email + scheduled jobs, 4gg, settimana 5)
7. T-MERGE-FINAL (Merge finale pre-integrazione, 1gg)
8. T-024 (Integration + UAT, 3gg, settimana 6)
- **Ruolo**: review tutto il codice BE (T-001, T-002, T-006, T-008), sblocco architetturale, governance macchina stati, responsabile di tutti i merge checkpoint

### Carmine (FE Senior — disponibile da settimana 2)
1. T-010 (Dashboard FE, 3gg, settimana 2)
2. Review T-004, T-005, T-011, T-012 (settimane 2-3)
3. T-024 (Integration + UAT, 3gg, settimana 6)
- **Ruolo**: review tutto il codice FE, architettura componenti, UX consistency

### Alexios (Fullstack Mid)
1. T-001 (Enum + entità core, 3gg, settimana 1)
2. T-MERGE-001-002 (Merge entità fondazioni, 0.5gg, dopo T-001+T-002)
3. T-003 (DTO + Repository + Vista SQL, 2gg, settimana 1)
4. T-MERGE-003 (Merge fondazioni completo, 0.5gg)
4. T-006 (Macchina stati + ID gen + scadenziere, 4gg, settimana 2)
5. T-014 (Upload CC + storico docs, 3gg, settimana 3)
6. T-017 (Flusso Deloitte base, 3gg, settimana 3-4)
7. T-020 (Upload COVNO, 3gg, settimana 4)

### Georgios (FE Junior, Claude Code)
1. T-004 (Module FE scaffold, 3gg, settimana 1)
2. T-MERGE-004 (Merge FE scaffold, 0.5gg)
3. T-011 (Tabella pratiche FE, 2.5gg, settimana 2)
4. T-013 (Scadenziere + dettaglio covenant, 3gg, settimana 3)
5. T-016 (Spread FE — 2 tabelle distinte, **4gg**, settimana 3-4) [+1gg da aggiornamento A-014]
6. T-022 (Mailing List, 5gg, settimana 5)

### Ahmad (BE Junior, no Claude Code — task ultra-dettagliate)
1. T-002 (Entità Spread + MailingList, 2.5gg, settimana 1)
2. T-008 (Service + Controller covenant, 3gg, settimana 2, con review Davide)
3. T-009 (Dashboard BE endpoints 6 segmenti, 2gg, settimana 2-3)
4. T-015 (Spread BE — 2 tabelle distinte, **3gg**, settimana 3) [+1gg da aggiornamento A-014]

### Adham (FE Junior, no Claude Code — task ultra-dettagliate)
1. T-005 (i18n + componente semaforo 6 colori, **2.5gg**, settimana 1) [+0.5gg da aggiornamento B4]
2. T-012 (Dettaglio pratica ISP, 4gg, settimane 2-3, con review Carmine)
3. T-019 (Eccezioni FE, 2.5gg, settimana 3-4)
4. T-021 (Pagine complete Scaduti/InScadenza, 2gg, settimana 4)

## Stima complessiva

### Effort (aggiornato 2026-05-06)
- BE: circa `26.5 gg/uomo` (fondazioni 7 + core 10.5 + spread/covno/mailing 8 + notifiche 4)
- FE: circa `27 gg/uomo` (scaffold 5.5 + dashboard 5.5 + pratiche/dettaglio 9 + spread/eccezioni/mailing 11)
- Merge checkpoint: `3 gg/uomo` (T-MERGE-001-002 0.5 + T-MERGE-006 0.5 + T-MERGE-W2 1 + T-MERGE-FINAL 1)
- Integrazione e UAT: circa `3 gg/uomo`
- **Totale: ~59.5 gg/uomo** (+3 gg per merge checkpoint rispetto a stima 2026-05-05)

### Durata calendario realistica
- **Scenario realistico (6 settimane)**: Settimana 1 fondazioni (senza senior), Settimana 2 core + primi FE, Settimane 3-4 feature parallele, Settimana 5 completamento, Settimana 6 UAT. Con 6 persone (4 disponibili settimana 1, 6 dalle settimane 2+), circa 6 settimane lavorative.
- **Scenario aggressivo (5 settimane)**: Comprime Wave 3+4, sovrappone mailing list e notifiche con features. Rischio: review insufficiente, debito tecnico.

## Rischi principali

1. **Macchina stati 4 livelli (T-006)**: Componente architetturale più complesso. Se il design non è corretto, impatta tutto. Mitigazione: Davide fa review approfondita prima del merge.
2. **GenAI extraction (T-018)**: Dipende da servizio GenAI esterno, potrebbe richiedere tuning. Mitigazione: flusso manuale funziona indipendentemente; GenAI è enhancement.
3. **Settimana 1 senza Senior**: Rischio di fondazioni non allineate con l'architettura. Mitigazione: Alexios è Mid e conosce il codebase; Davide fa review retroattiva settimana 2.
4. **Ahmad e Adham senza Claude Code**: Produttività inferiore, rischio errori. Mitigazione: task ultra-dettagliate con istruzioni passo-passo, review frequente.
5. **Perimetro molto ampio**: 31 task (inclusi merge checkpoint), ~59.5 gg/uomo. Rischio scope creep. Mitigazione: wave 4 (mailing list) e wave 5 (UAT) possono slittare senza impattare il core.
6. **Upload COVNO senza tracciato record**: Assunzione A-011 su formato XLSX. Se il formato reale è diverso, rework parsing.

## Raccomandazioni operative

1. **Branch strategy**: ogni task crea branch `feature/monitoring-{task-name}` da `feature/monitoring`. Merge dopo review.
2. **Review obbligatoria**: ogni PR di Ahmad/Adham deve essere reviewata da un senior prima del merge. PR di Alexios/Georgios reviewate a campione.
3. **Standup giornaliero**: 15 minuti per sbloccare dipendenze. Critico nelle settimane 2-3 quando il parallelismo è massimo.
4. **API contract first**: Davide definisce i DTO + endpoint (contratto API) all'inizio della settimana 2, prima che il FE inizi a consumarli. Questo sblocca il FE anche se il BE non è completato.
5. **Feature flag**: il modulo monitoring può essere nascosto dal menu fino al completamento. Aggiungere flag `monitoring.enabled=false` in application.properties.
6. **Test progressivi**: ogni task deve includere almeno test unitari per service/controller. Niente merge senza build verde.

## Deliverable minimi

1. Dashboard monitoring con donut **6 segmenti** (OK/InScadenza/DaAttenzionare/Scaduto/FuoriSoglia/Concluse), tabelle scaduti/in scadenza, pratiche recenti, polling 60s
2. Tabella pratiche monitoring con filtri, paginazione, export Excel
3. Dettaglio pratica ISP: header, tab covenant (overview + tabelle + dettaglio singolo), tab scadenziere, tab spread (read-only)
4. Upload compliance certificate con storico documenti
5. Flusso Deloitte: conferma/rifiuta, assegnazione CC (manuale + GenAI), modifica covenant
6. Eccezioni: richiesta ISP + gestione Deloitte
7. Spread: **due tabelle distinte** — "Tabella Spread" editabile Deloitte (sez. 7.3.5) + "Tabella storico Spread" read-only ISP (sez. 3.5.2) con download documenti
8. Macchina stati 4 livelli con calcolo automatico e propagazione bottom-up
9. Chiusura automatica pratiche (scheduled job giornaliero)
10. Notifiche email: scadenza, scaduto, eccezione, spread, rifiuto documento

## Storico Aggiornamenti

### Aggiornamento 2026-05-05 (br-clarify round 2)
- Fonte: risposte funzionale a 13 domande non bloccanti + 3 aggiornamenti ai bloccanti
- Task modificate: T-005 (+0.5gg), T-006 (+0.5gg), T-009, T-010 (+0.5gg), T-015 (+1gg), T-016 (+1gg), T-019, T-023
- Delta effort: +3.5 gg/uomo (da ~53 a ~56.5)
- Cambiamento principale: semaforo da 4 a 6 colori (B4), spread due tabelle distinte (A-014 rigettata), formule "Da attenzionare" dettagliate (A-012), condizioni chiusura automatica aggiornate (B2)

### Aggiornamento 2026-05-06 (aggiunta colonna Branch + correzione merge task)
- Aggiunta colonna **Branch** al backlog operativo con nome branch per ogni task
- Branch esistenti riconosciuti da remote: `feature/monitoring-structures` (T-001), `feature/monitoring-entities-spread-mailing` (T-002), `feature/monitoring-fe-scaffold` (T-004)
- Aggiornate descrizioni merge task (T-MERGE-003, T-MERGE-004, T-MERGE-008) con lista esplicita di tutti i branch da mergiare e ordine di merge

### Aggiornamento 2026-05-06 (aggiunta merge checkpoint espliciti)
- **Problema identificato**: task con dipendenze su codice in branch separati non potevano partire senza merge intermedi (T-003 bloccata da T-001+T-002 su branch diversi, T-008 bloccata da T-006 su branch separato, T-017 bloccata da T-014 non mergiata)
- **Soluzione**: aggiunti 4 merge checkpoint espliciti:
  - `T-MERGE-001-002` (Wave 0): merge entità fondazioni prima di T-003. Owner: Alexios, 0.5gg
  - `T-MERGE-006` (Wave 1): merge macchina stati + pratiche prima di T-008. Owner: Davide, 0.5gg
  - `T-MERGE-W2` (Wave 2): merge tutti i branch Wave 2 prima di Wave 3. Owner: Davide, 1gg
  - `T-MERGE-FINAL` (Wave 4): merge tutti i branch Wave 3+4 prima di UAT. Owner: Davide, 1gg
- Dipendenze aggiornate: T-003 (→T-MERGE-001-002), T-MERGE-003 (→T-003), T-008 (→T-MERGE-006), T-MERGE-008 (→T-008), T-017 (→T-MERGE-W2), T-018 (→T-017), T-024 (→T-MERGE-FINAL)
- Delta effort: +3 gg/uomo (da ~56.5 a ~59.5). Task totali: da 27 a 31
- Nessuna modifica al contenuto funzionale delle task esistenti
