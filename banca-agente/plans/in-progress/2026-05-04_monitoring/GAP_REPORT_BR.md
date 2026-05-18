# Report Verifica BR Agency Desk Monitoraggio V6

Data verifica: `2026-05-04`

Branch verificato:
- BE: `feature/monitoring`
- FE: `feature/monitoring`
- DM: `develop` (nessun codice monitoring presente)
- EM: `develop` (nessun template monitoring presente)

Perimetro documentale verificato:
- BR: `BR - Agency Desk Monitoraggio V6.docx` (convertito in `br-docs-converted/BR_Monitoraggio_V6.md`)
- Mockup: `202604_Deck Mockup_Monitoraggio_v10.pptx` (convertito in `br-docs-converted/Mockup_Monitoraggio_v10.md`)
- Macchina stati: `202604_Macchina Stati Monitoring_v1.xlsx` (convertito in `br-docs-converted/Macchina_Stati_Monitoring_v1.md` e `BA_Monitoraggio_Macchina_Stati.md`)

Codebase verificati:
- BE (Ba-back-end): `C:\Users\davmelis\Documents\Github\ba-back-end`
- FE (Ba-web): `C:\Users\davmelis\Documents\Github\ba-web`
- DM (Ba-document-manager): `C:\Users\davmelis\Documents\Github\ba-document-manager`
- EM (Ba-email-manager): `C:\Users\davmelis\Documents\Github\ba-email-manager`

## Assunzioni da review

br-clarify round 1 eseguito il 2026-04-30: tutti i 5 bloccanti risolti.
br-clarify round 2 eseguito il 2026-05-05: 13 risposte non bloccanti ricevute, 3 bloccanti aggiornati.

Bloccanti risolti (risposte del funzionale, usate come fatti nell'analisi):
- [B1] Formato ID Evento Monitoraggio → Il formato definitivo è `M{n}C{n}E{n}` (es. M1C1E1). Le occorrenze di `P1C2M1` nel BR sono refusi.
- [B2] Chiusura pratica automatica → Versione "old" con 3 condizioni aggiornate (2026-05-05): (1) "Data di scadenza operazione < Today", (2) "Data di scadenza piu' lontana tra gli eventi di monitoraggio < Today", (3) assenza di eventi con status Scaduto / Fuori Soglia / **In Scadenza**.
- [B3] Status Covenant singolo → Segue le logiche di priorita' dello stato pratica, basate sugli eventi del covenant. Macchina a stati aggiornata: `202604_Macchina Stati Monitoring_v2.xlsx` — nuovo sheet "Singolo Covenant".
- [B4] Semaforo **6 colori** (corretto 2026-05-05, era 4): (1) Verde=OK, (2) Giallo=In scadenza, (3) Arancione=Da attenzionare, (4) Rosso=Scaduto, (5) Rosso scuro=Fuori soglia, (6) Grigio=Concluso.
- [B5] Utente Deloitte → Ereditato dal censimento, senza auto-assegnazione.

Bloccanti ancora aperti: nessuno.

Assunzioni — stato aggiornato al 2026-05-05:

Confermate dal funzionale:
- A-001: Formula = Scadenza - Today (positivo=rimanenti, negativo=ritardo) — **Confermata**
- A-002: "IN CARICO A" = "Utente Banca" quando nessun doc in lavorazione e nessun evento critico — **Confermata**
- A-003: Dashboard "In Scadenza" stesse colonne di "Scaduti" — **Confermata** (refuso)
- A-004: Typo corretto "chiudere anticipatamente la pratica" — **Confermata** (refuso)
- A-005: Label = "Termina Pratica di Monitoraggio" — **Confermata**
- A-006: Termine = "Valore Monitoraggio" (dal BR) — **Confermata**
- A-007: Label = "IN CARICO A" (con spazi) — **Confermata**
- A-008: Colonna "ID Covenant" = M{n}C{n}, colonna separata "ID Evento" = M{n}C{n}E{n} — **Confermata**
- A-009: ID pratica = "M{n}" senza padding zero — **Confermata**
- A-010: Seconda colonna "Note" nel mockup è errore, ignorata — **Confermata**
- A-012: "10% dalla soglia" con formule precise fornite. Segno <: (soglia-10%*soglia) <= valore < soglia. Segno >: (soglia+10%*soglia) >= valore > soglia. Condizione B (30%+40%) con formule per entrambi i segni. Le due condizioni in OR. — **Confermata con dettaglio**
- A-013: Evento con eccezione: conteggiato OK, data avvenuto = data eccezione, valore resta **vuoto** (non "N/A") — **Confermata con correzione**

Rigettate:
- A-014: ~~Storico Spread definitivo = tabella sez. 7.3.5~~ → **Rigettata**: "Tabella storico Spread" (sez. 3.5.2) e "Tabella Spread" (sez. 7.3.5) sono due oggetti distinti entrambi corretti. Servono ENTRAMBE le tabelle.

In attesa (nessuna risposta, si procede con l'assunzione proposta):
- A-011: Upload COVN0/DB Obblighi in XLSX, aggiornamento incrementale per Tracking ID, file vince su automatici ma non su manuali Deloitte
- A-015: Mailing list per notifiche automatiche: scadenza (7gg prima), scaduto, eccezione, variazione spread. Tutti in TO.
- A-016: "Tempo reale" = polling FE ogni 60 secondi

## Esito sintetico

Il modulo Monitoraggio è quasi interamente da costruire da zero. L'unico codice esistente è infrastrutturale: enum per periodicità e tipi covenant, una vista SQL di riepilogo pratiche, e un componente FE generico usato come step wizard nel censimento. Tutti i domini core (Covenant, Evento di Monitoraggio, Spread, Mailing List), la macchina a stati multi-livello (4 livelli), le pagine frontend (dashboard, tabella pratiche, dettaglio pratica ISP/Deloitte, spread, scadenziere, mailing list), i template email, e la logica di scheduling (chiusura automatica, notifiche scadenza) sono completamente mancanti.

Il perimetro è significativo: circa 8 domini nuovi, 15+ pagine FE, 20+ endpoint API, 4 livelli di macchina a stati, 6-10 template email, e integrazione GenAI.

## Matrice di verifica

### 1. Creazione Pratica Monitoraggio

| Requisito | BE | FE | DM | EM | Stato | Evidenze | Gap |
|---|---|---|---|---|---|---|---|
| Auto-creazione dopo Post-Closing | Non impl. | N/A | N/A | N/A | Mancante | `PsmStateService.java` gestisce transizioni ma nessun trigger per creazione pratica monitoring | Creare listener/service che alla chiusura del Post-Closing generi automaticamente la pratica di monitoraggio collegata via Tracking ID |
| Alimentazione covenant da Post-Closing | Non impl. | N/A | N/A | N/A | Mancante | Nessuna entità Covenant nel domain (`domain/agency_desk/` contiene solo `BookingLender`, `Assurance*`, `FinancingAgreement`, `PostClosing*`) | Creare entità `MonitoringCovenant` con i campi richiesti (tipologia, valore limite, segno, periodicità, variabile, note) e logica di import |
| Generazione scadenziere automatico | Non impl. | N/A | N/A | N/A | Mancante | Nessun generatore di scadenziere | Creare service che generi eventi progressivamente fino a Termination Date basandosi su periodicità (1/3/6/12 mesi) |
| ID pratica: M{n} | Non impl. | N/A | N/A | N/A | Mancante | `vw_monitoring_practice_summary` usa `mp.code` ma nessun generatore ID formato M{n}. `PracticeMonitoringKey.java` (`utils/`) contiene solo `limitDays` + `customerId` — non è un generatore ID | Creare generatore sequenziale formato "M{n}" |
| ID covenant: M{n}C{n} | Non impl. | N/A | N/A | N/A | Mancante | Nessuna logica | Creare generatore progressivo per covenant all'interno della pratica |
| ID evento: M{n}C{n}E{n} | Non impl. | N/A | N/A | N/A | Mancante | Nessuna logica | Creare generatore progressivo per evento all'interno del covenant |
| Utenti ereditati da censimento | Parziale | N/A | N/A | N/A | Parziale | `Practice.java` ha `consultant_id` e `user_id` (FK a utente). La pratica di monitoraggio può ereditarli dalla pratica di censimento collegata | Implementare copia automatica degli utenti dalla pratica di censimento alla pratica di monitoraggio |

### 2. Tabella Pratiche Monitoraggio

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Menu laterale "Pratiche Monitoraggio" | N/A | Non impl. | Mancante | FE: routing in `shared/const/routes/consultant-route.const.ts` e `admin-route.const.ts` — nessuna route monitoring. Sidebar definita nel layout component. | Aggiungere voce menu e route lazy-loaded per il modulo monitoring |
| Filtri (Utente Banca, Tracking ID, ID Monitoraggio, Nome Deal, Semaforo multi-select) | Non impl. | Non impl. | Mancante | BE: `PracticeSummaryRequestDTO.java` ha filtri generici per census. FE: `PracticeTableFilter` in `shared/model/practice-table-filter.model.ts` — solo per census | Creare DTO filtri specifico per monitoring + endpoint di filtraggio |
| Colonne tabella (ID, Tracking ID, Nome Deal, Utente Banca, Utente Deloitte, Tipologia, #Scaduti, #InScadenza, #DocInLavorazione, Semaforo, Data Creazione, Data Fine, Azione) | Parziale | Non impl. | Parziale | BE: `vw_monitoring_practice_summary` ha practice_id, practice_code, deal_id, contract_id, practice_state, creation_date — manca: deal_name, utente banca/deloitte nomi, conteggi scaduti/in scadenza, doc in lavorazione. FE: nessuna pagina | Vista SQL da estendere con colonne aggregate + nuova pagina FE con `app-table` server-paginated |
| Paginazione 20 righe, ordinamento, export Excel | N/A | Non impl. | Mancante | FE: pattern esistente in `TableComponent` (`shared/ui/base-element/table/table.component.ts`) con `pageRows=20`, `isServerPaginated`, `onLazyLoad$`. Export via `PracticeService.exportPracticesExcel()` | Seguire pattern `TableComponent` + creare endpoint export Excel |
| Click riga → dettaglio pratica | N/A | Non impl. | Mancante | FE: `TableComponent` supporta `onRowClick$` e `isRowClickable` | Routing a pagina dettaglio con practiceId |
| Azione "Termina Pratica di Monitoraggio" | Non impl. | Non impl. | Mancante | Nessuna logica di terminazione manuale | Endpoint + dialog conferma + transizione stato a Conclusa (Grey) |

### 3. Dettaglio Pratica — Flusso ISP

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Header pratica (ID, NDG, Tipologia, ID Contratto, Utente Banca, Stato) | Parziale | Non impl. | Parziale | BE: `Practice.java` ha code, clientNdg, contractKey, mandateType, stateId. FE: nessuna pagina dettaglio monitoring | Pagina FE con header. Servono API per i dati aggregati |
| Tab Covenant | Non impl. | Non impl. | Mancante | Nessuna entità, DTO, controller, o pagina | Intero flusso da creare |
| Overview covenant: card totale, card scadenze (OK/InScadenza/KO), card avanzamento (Sezione, IN CARICO A, STATO LAVORAZIONE) | Non impl. | Non impl. | Mancante | | Endpoint conteggi + componenti card FE |
| Upload Compliance Certificate (drag&drop, formati, max 3MB) | Non impl. | Parziale | Parziale | FE: `FileUploadComponent` (`shared/ui/base-element/file-upload/`) wrappa `p-fileUpload` PrimeNG con drag&drop, `maxFileSize`, `accept`. Pattern riusabile | Upload FE riusabile. Servono: endpoint BE upload, gestione documento DM, collegamento a pratica |
| Storico documenti (in lavorazione + lavorati, status, giorni giacenza, visualizza/download/commento) | Non impl. | Non impl. | Mancante | BE: `DocumentStateEnum` ha stati per census (TO_BE_UPLOAD, VALIDATED, etc.) — servono stati monitoring specifici | Nuovo enum stati documento monitoring + entità + tabella FE |
| Vista dettaglio assegnazione (viewer doc sx, indicatori covenant dx) | Non impl. | Non impl. | Mancante | FE: `PdfViewerWrapComponent` in `shared/ui/section-element/pdf-viewer-wrap/` esiste per visualizzazione documenti | Layout split-view con PDF viewer e indicatori |
| Tabella Monitoraggi Scaduti/Fuori Soglia (12 colonne + azione Salta/Ignora) | Non impl. | Non impl. | Mancante | | Endpoint filtrato per eventi con stato Scaduto/FuoriSoglia + tabella FE |
| Tabella Monitoraggi In Scadenza | Non impl. | Non impl. | Mancante | | Stesse colonne di Scaduti (A-003) |
| Tabella Tutti i Covenant (13 colonne aggregate per covenant) | Non impl. | Non impl. | Mancante | | Endpoint aggregazione per covenant + tabella FE |
| Dettaglio Covenant Singolo (stato avanzamento, semafori circolari, scadenziere) | Non impl. | Non impl. | Mancante | | Pagina/modal dettaglio con sub-tabella eventi |
| Tab Spread (tabella read-only, cluster attivo evidenziato, storico) | Non impl. | Non impl. | Mancante | Nessuna entità Spread nel domain | Intero modulo spread da creare |
| Tab Scadenziere (filtro tipologia, tabella cronologica 9 colonne) | Non impl. | Non impl. | Mancante | | Vista consolidata tutti gli eventi della pratica |

### 4. Flusso Deloitte

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Stessa overview ISP + funzionalità aggiuntive | Non impl. | Non impl. | Mancante | FE: differenziazione ruolo via `AuthService` e `RoleEnum` (CONSULTANT = Deloitte, OWNER/MEMBER = ISP) | Stessa base ISP + sezioni aggiuntive condizionate da ruolo |
| Conferma/Rifiuta documenti | Non impl. | Non impl. | Mancante | Pattern esistente nel dossier: `onApprove$`/`onReject$` in componenti fase | Endpoint + button FE |
| Assegnazione Compliance Certificate (manuale) | Non impl. | Non impl. | Mancante | | Flusso: Aggiungi Covenant → seleziona tipologia+valore+data → scadenziere filtrato → Assegna → Conferma → Concludi |
| Assegnazione Compliance Certificate (GenAI) | Non impl. | Non impl. | Mancante | BE: `GenAiFormKeyEnum.java` ha chiavi per extraction GenAI. FE: `GenaiExtractionComponent` in `shared/ui/section-element/genai-extraction/` esiste | Pattern GenAI riusabile: spinner → proposta → Modifica/Conferma. Integrare con servizio GenAI |
| Modifica Dati Covenant (tipologia, valore limite, variabile, scadenze, aggiungi/rimuovi monitoraggio) | Non impl. | Non impl. | Mancante | | Form reattivo con validazione |
| Gestione Eccezioni (visualizza motivazione, scarica documenti, "Annulla Eccezione") | Non impl. | Non impl. | Mancante | | Modal visualizzazione + endpoint annullamento |

### 5. Scadenziere Unificato

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Vista cronologica tutti gli eventi | Non impl. | Non impl. | Mancante | | Endpoint con tutti gli eventi ordinati + tabella FE 9 colonne |
| Filtro per tipologia | Non impl. | Non impl. | Mancante | | Dropdown filtro tipologia covenant |
| Paginazione 20 righe | N/A | Non impl. | Mancante | Pattern `TableComponent` riusabile | |

### 6. Mailing List

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Pagina ricerca pratica | Non impl. | Non impl. | Mancante | | Nuova pagina con filtri |
| Tabella pratiche (11 colonne + Richiedi Aggiornamento) | Non impl. | Non impl. | Mancante | | Tabella con azioni contestuali |
| Upload AD Form → GenAI → report | Non impl. | Non impl. | Mancante | BE: solo check booleano `isBookingMailingListReportGenerated()` in `PsmStateService.java` per flusso Booking | Modulo completo: upload, parsing GenAI, report generato |
| Vista Deloitte: Conferma/Rifiuta, Estrai GenAI, Conferma e Aggiorna | Non impl. | Non impl. | Mancante | | Flusso completo con stati (Doc Incompleta → Completa → Confermata → Verifica → Report) |
| CRUD contatti mailing list | Non impl. | Non impl. | Mancante | | Entità MailingListContact + CRUD API + form FE |

### 7. Spread

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Tabella spread Deloitte (manuale) | Non impl. | Non impl. | Mancante | BE: la parola "spread" nel codice si riferisce solo a spreadsheet (fogli Excel), non a spread finanziari | Entità SpreadTable + CRUD + tabella FE editabile |
| Disclaimer "tabella non verificata" | N/A | Non impl. | Mancante | | Banner condizionale FE |
| Tabella Spread Deloitte (sez. 7.3.5): Data Riferimento, Data avvenuto, Valore Monitorato, Cluster, Margine (%) | Non impl. | Non impl. | Mancante | | Entità SpreadTable con struttura sez. 7.3.5 + CRUD + tabella FE editabile |
| Tabella storico Spread ISP (sez. 3.5.2): Data Variazione, Documento, Azione (Scarica), Cluster | Non impl. | Non impl. | Mancante | | Entità SpreadHistory con struttura sez. 3.5.2 + tabella FE read-only + download documento |
| Notifica ISP nuova riga storico | Non impl. | N/A | Non impl. | Mancante | Nessun template email monitoring in `EmailTemplateEnum` | Template email + trigger su insert storico |
| ISP vede solo dopo validazione Deloitte | Non impl. | Non impl. | Mancante | | Flag `validated` su SpreadTable + filtro visibilità per ruolo |

### 8. Dashboard Monitoraggio

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Selettore Censimento/Monitoraggio | N/A | Non impl. | Mancante | FE: `DashboardComponent` (`shared/ui/base-element/dashboard/dashboard.component.ts`) ha filtro `DashboardMandateFilterEnum` ma solo per fasi census | Aggiungere toggle o tab "Monitoraggio" alla dashboard esistente |
| Donut chart 6 segmenti (OK/InScadenza/DaAttenzionare/Scaduto/FuoriSoglia/Concluse) + totale (B4 corretto) | Parziale | Non impl. | Parziale | BE: `DashboardService` con endpoint `/dashboard/summary`. FE: `DoughnutComponent` (`shared/ui/base-element/doughnut/`) wrappa Chart.js | Nuovo endpoint summary monitoring + riuso `DoughnutComponent` con 6 segmenti/colori |
| Polling 60 secondi | N/A | Non impl. | Mancante | FE: nessun polling esistente nella dashboard | `interval(60000)` RxJS sul componente |
| 2 card COVNO (data aggiornamento, upload XLSX, storico) | Non impl. | Non impl. | Mancante | | Card con stato ultimo upload + pulsante upload + storico |
| Tabella Monitoraggi Scaduti (dashboard, max 5 righe, "Vedi tutte") | Non impl. | Non impl. | Mancante | | Tabella limitata a 5 righe con link a pagina completa |
| Pagina completa Scaduti (filtri avanzati, export Excel, 20 righe/pagina) | Non impl. | Non impl. | Mancante | | Pagina dedicata con `app-table` server-paginated |
| Tabella Monitoraggi In Scadenza (dashboard + pagina completa) | Non impl. | Non impl. | Mancante | | Stesse colonne di Scaduti (A-003), stessa struttura |
| Tabella Pratiche Recenti (ultime 10, "Vedi tutte") | Non impl. | Non impl. | Mancante | | Tabella limitata con link a Tabella Pratiche |

### 9. Macchina a Stati

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Livello Pratica (6 stati: OK/InScadenza/DaAttenzionare/Scaduto/FuoriSoglia/Conclusa) | Non impl. | N/A | Mancante | BE: `PsmStateService.java` gestisce stati generici di pratica (census/booking/pre-closing/post-closing). Nessuno stato monitoring specifico. `PsmState.java` in `domain/practice/` è l'entità stati. | Nuovi stati in `mp_psm_state` per monitoring + logica di calcolo basata su aggregazione eventi |
| Livello Covenant (stessa logica della pratica) | Non impl. | N/A | Mancante | Nessuna entità Covenant con stato | Stato calcolato dinamicamente dagli eventi del covenant |
| Livello Evento (6 stati: OK/InScadenza/DaAttenzionare/FuoriSoglia/Scaduto/NessunoStato) | Non impl. | N/A | Mancante | | Stato calcolato da: data scadenza, valore monitoraggio, soglie |
| Livello Documento (3 stati + 2 sottostati GenAI) | Non impl. | N/A | Mancante | `DocumentStateEnum.java` ha stati census. `SectionStateEnum.java` ha sottostati GenAI a livello sezione | Nuovo enum `MonitoringDocumentStateEnum` con: IN_ATTESA_CONFERMA, VERIFICA_IN_CORSO (sottostati: GENAI_IN_CORSO, GENAI_DA_VALIDARE), COMPLETATO |
| Chiusura automatica (3 condizioni: Data scadenza op < Today AND Data scadenza piu' lontana eventi < Today AND assenza Scaduto/FuoriSoglia/InScadenza) | Non impl. | N/A | Mancante | `ScheduledService.java` ha scheduled jobs ma nessuno per monitoring | Scheduled job giornaliero per chiusura automatica pratiche con 3 condizioni (B2 aggiornato) |
| Semaforo 6 colori (B4 corretto) | Non impl. | Non impl. | Mancante | | Mapping stato → colore (Verde/Giallo/Arancione/Rosso/RossoScuro/Grigio) in FE con 6 classi CSS |

### 10. Email e Notifiche

| Requisito | BE | FE | DM | EM | Stato | Evidenze | Gap |
|---|---|---|---|---|---|---|---|
| Notifica scadenza (7gg prima) | Non impl. | N/A | N/A | Non impl. | Mancante | `EmailTemplateEnum.java` ha `PRACTICE_ALERT_BEFORE` / `PRACTICE_ALERT_AFTER` per census — nessun equivalente monitoring. `ScheduledService.java` non ha job monitoring | Nuovo template + scheduled job 7gg prima scadenza |
| Notifica monitoraggio scaduto | Non impl. | N/A | N/A | Non impl. | Mancante | | Nuovo template + trigger su cambio stato a Scaduto |
| Notifica eccezione richiesta/gestita | Non impl. | N/A | N/A | Non impl. | Mancante | | 2 template: richiesta + gestita |
| Notifica variazione spread | Non impl. | N/A | N/A | Non impl. | Mancante | | Template + trigger su insert storico spread |
| Notifica documento rifiutato | Non impl. | N/A | N/A | Non impl. | Mancante | | Template + trigger su rifiuto documento |

### 11. Upload COVNO / DB Obblighi

| Requisito | BE | FE | Stato | Evidenze | Gap |
|---|---|---|---|---|---|
| Upload XLSX | Non impl. | Non impl. | Mancante | BE: `SheetReaderComponent` nel FE (`shared/ui/base-element/sheet-reader/`) legge file Excel. Pattern riusabile | Upload + parsing XLSX |
| Aggiornamento incrementale per Tracking ID | Non impl. | N/A | Mancante | | Logica merge: file vince su automatici, non su manuali Deloitte (A-011) |
| Storico aggiornamenti | Non impl. | Non impl. | Mancante | | Tabella storico con data, utente, esito |

## Gap aperti reali

### 1. Dominio Covenant — Entità JPA completa da creare

Il BR definisce il Covenant come entità centrale del monitoraggio con: Tipologia (da `CovenantTypeNumericEnum`, ~53 valori), Valore Limite, Segno (da `CovenantSignEnum`: >, >=, =, <, <=), Periodicità (da `MonitoringPeriodicityEnum`: Annuale/Semestrale/Trimestrale/Mensile/Altro), Variabile, Note. Nel codice esistono solo gli enum ma nessuna entità JPA `MonitoringCovenant`.

**File coinvolti**:
- Da creare: `domain/agency_desk/MonitoringCovenant.java`
- Enum esistenti: `enumeration/agency_desk/CovenantTypeNumericEnum.java` (65 righe, ~53 valori), `CovenantSignEnum.java`, `MonitoringPeriodicityEnum.java`
- Pattern: seguire `BookingLender.java` (`domain/agency_desk/`) — @Entity, @Data, @Id @GeneratedValue(IDENTITY), @ManyToOne(LAZY) su Practice
- Migration: `ddl/init/agency-desk/wave-3/` (numerazione `002_monitoring_foundation.sql`)

### 2. Dominio Evento di Monitoraggio — Entità da creare da zero

Ogni covenant ha N eventi di monitoraggio generati automaticamente in base alla periodicità. Campi: ID Evento (M{n}C{n}E{n}), Data Riferimento, Data Scadenza, Valore Monitoraggio, Status (calcolato), Data Avvenuto Monitoraggio, Note. Nessuna traccia nel codice.

**File coinvolti**:
- Da creare: `domain/agency_desk/MonitoringEvent.java`
- Da creare: `domain/agency_desk/MonitoringEventStatus.java` (enum: OK, IN_SCADENZA, DA_ATTENZIONARE, FUORI_SOGLIA, SCADUTO, NESSUNO_STATO)
- Da creare: migration per tabella `mp_monitoring_event`

### 3. Macchina a Stati Multi-Livello — Architettura nuova

Il BR richiede 4 livelli di stato gerarchici (Pratica → Covenant → Evento → Documento) con propagazione bottom-up. La `PsmStateService.java` esistente gestisce transizioni lineari per le fasi del censimento — non supporta gerarchia multi-livello né calcolo aggregato. Necessaria architettura nuova.

**File coinvolti**:
- Esistente: `service/psm/PsmStateService.java` — da estendere o creare servizio parallelo
- Da creare: `service/monitoring/MonitoringStateService.java` — logica di calcolo stato per tutti e 4 i livelli
- Da creare: `enumeration/agency_desk/MonitoringPracticeStateEnum.java`, `MonitoringCovenantStateEnum.java`, `MonitoringEventStateEnum.java`, `MonitoringDocumentStateEnum.java`
- Logica: stato Evento calcolato da scadenza/valore/soglie → aggregato in stato Covenant → aggregato in stato Pratica. Priorità: FuoriSoglia > Scaduto > InScadenza > DaAttenzionare > OK > Conclusa.

### 4. Generazione Scadenziere — Logica batch nuova

Il sistema deve generare eventi progressivi fino alla Termination Date in base alla periodicità del covenant. Quando una pratica viene creata (auto da post-closing), per ogni covenant si generano N eventi con date scadenza calcolate. Nessuna logica simile nel codice.

**File coinvolti**:
- Da creare: `service/monitoring/MonitoringScheduleGeneratorService.java`
- Riferimento periodicità: `MonitoringPeriodicityEnum.java` → ANNUALE(12m), SEMESTRALE(6m), TRIMESTRALE(3m), MENSILE(1m)
- Dipende da: `Practice.termination_date` (campo aggiunto nella migration `001_gap_0_foundation.sql`)

### 5. Frontend Monitoring — Modulo intero da creare

Il FE ha solo `MonitoringComponent` (`shared/ui/section-element/dossier/step-type-monitoring/monitoring.component.ts`) — un componente generico per lo step "Monitoring" nel wizard del dossier (censimento). Non è un modulo standalone. Tutte le pagine del modulo monitoraggio (dashboard, tabella pratiche, dettaglio pratica, spread, scadenziere, mailing list) sono da creare da zero.

**Pattern FE da seguire**:
- Modulo: creare `module/agency-desk/monitoring/` con routing lazy-loaded
- Tabelle: riusare `app-table` (`shared/ui/base-element/table/`) con PrimeNG Table, server pagination, lazy loading
- Chart: riusare `app-doughnut` (`shared/ui/base-element/doughnut/`) con Chart.js
- Upload: riusare `app-file-upload` (`shared/ui/base-element/file-upload/`) con PrimeNG p-fileUpload
- Dialog: PrimeNG `DialogService` / `DynamicDialog` (pattern in `dossier.component.ts`)
- Servizi: `RestService` (`service/rest.service.ts`) wrapping HttpClient, risposte `BaseResponseModel<T>`
- i18n: aggiungere chiavi in `assets/i18n/it-IT.json` e `en-GB.json`
- Ruoli: `RoleEnum.Consultant` = Deloitte, `RoleEnum.Owner`/`Member` = ISP (banca)
- Routing: aggiungere in `shared/const/routes/consultant-route.const.ts` e `admin-route.const.ts`

### 6. Spread — Modulo da creare da zero (A-014 rigettata: due tabelle distinte)

Nessuna logica spread (finanziario) nel BE — la parola "spread" nel codice si riferisce solo a spreadsheet Excel. Il funzionale ha chiarito (2026-05-05) che esistono **due oggetti distinti**:
- **"Tabella Spread"** (sez. 7.3.5, Deloitte): tabella editabile con colonne Data Riferimento, Data avvenuto monitoraggio, Valore Monitorato, Cluster di Riferimento, Valore Margine (%). Entità `SpreadTable`.
- **"Tabella storico Spread"** (sez. 3.5.2, ISP): tabella read-only con colonne Data Variazione, Documento, Azione (Scarica), Cluster. Entità `SpreadHistory` con collegamento a documento scaricabile.

Servono: entrambe le entità, CRUD API per Tabella Spread (Deloitte), API read-only per storico (ISP), disclaimer "non verificata", notifica ISP, validazione Deloitte.

**File coinvolti**: tutti da creare in `domain/agency_desk/`, `dto/agency_desk/`, `repository/`, `service/monitoring/`, `controller/monitoring/`

### 7. Mailing List — Modulo completo da zero

Il codice ha solo `isBookingMailingListReportGenerated()` in `PsmStateService.java` — un check booleano per il flusso Booking. Per il monitoraggio serve: entità MailingListContact, CRUD contatti, upload AD Form, estrazione GenAI, report generato, stati documento (Doc Incompleta → Completa → Confermata → Verifica → Report), conferma/rifiuta Deloitte.

**File coinvolti**: tutti da creare

### 8. Email Monitoring — 6-10 template da creare

`EmailTemplateEnum.java` non ha alcun template per monitoring. Servono almeno:
1. `MONITORING_EXPIRATION_ALERT` — scadenza 7gg prima
2. `MONITORING_EXPIRED` — evento scaduto
3. `MONITORING_EXCEPTION_REQUESTED` — eccezione richiesta da ISP
4. `MONITORING_EXCEPTION_HANDLED` — eccezione gestita da Deloitte
5. `MONITORING_SPREAD_VARIATION` — variazione spread
6. `MONITORING_DOCUMENT_REJECTED` — documento rifiutato
7. `MONITORING_PRACTICE_CLOSED` — pratica chiusa automaticamente

**File coinvolti**:
- `enumeration/EmailTemplateEnum.java` — aggiungere valori
- `Ba-email-manager/` — creare template HTML per ogni tipo
- `scheduled/ScheduledService.java` — aggiungere job per scadenze monitoring

### 9. Upload COVNO / DB Obblighi — Parsing XLSX da creare

Il dashboard richiede upload di file XLSX (COVN0 e DB Obblighi) per aggiornare dati di monitoraggio. Logica: aggiornamento incrementale per Tracking ID, il file vince sui dati automatici ma non su quelli inseriti manualmente da Deloitte (A-011).

**File coinvolti**:
- FE: `SheetReaderComponent` (`shared/ui/base-element/sheet-reader/`) esiste come lettore Excel — riusabile
- BE: da creare parsing XLSX + logica merge + storico aggiornamenti

### 10. Scheduled Jobs Monitoring — Da creare

`ScheduledService.java` non ha job per il monitoring. Servono:
1. Job giornaliero chiusura automatica pratiche — 3 condizioni (B2 aggiornato 2026-05-05): (1) Data scadenza operazione < Today, (2) Data scadenza piu' lontana tra gli eventi < Today, (3) assenza di eventi con status Scaduto / Fuori Soglia / In Scadenza
2. Job giornaliero aggiornamento stati eventi (ricalcolo InScadenza per eventi a 7gg dalla scadenza)
3. Job notifica scadenze imminenti (7gg prima)

**File coinvolti**:
- `scheduled/ScheduledService.java` — aggiungere 3 metodi @Scheduled
- `scheduled/SchedulingJobNames.java` — aggiungere costanti nomi job
- Da creare: `service/monitoring/MonitoringSchedulerService.java`

### 11. Document Manager — Nessun codice monitoring

`Ba-document-manager` non ha alcun codice relativo al monitoraggio. Se la gestione documenti (compliance certificate, AD Form) passa dal DM, servono: endpoint upload/download, validazione formati, gestione stati documento specifici per monitoring.

### 12. Vista SQL Monitoring — Da estendere

`vw_monitoring_practice_summary` (`ddl/init/agency-desk/999_views/001_view_monitoring_practice_summary.sql`) esiste ma è minimale: solo practice_id, code, deal_id, contract_id, state, creation_date. Mancano: deal_name, utente banca/deloitte (nomi), conteggi aggregati (scaduti, in scadenza, doc in lavorazione), semaforo calcolato.

## Conclusione finale

**Coperto**: Infrastruttura base (enum periodicità, tipi covenant numerici, segni covenant, vista SQL minimale, campo termination_date su Practice).

**Mancante (12 gap aperti)**:
1. **Entità Covenant** — da creare con tutti i campi BR
2. **Entità Evento Monitoraggio** — da creare da zero
3. **Macchina a Stati 4 livelli** — architettura nuova (non estensione)
4. **Generazione scadenziere** — logica batch nuova
5. **Modulo FE intero** — ~15 pagine/componenti da zero
6. **Spread** — dominio + frontend da zero
7. **Mailing List** — modulo completo da zero
8. **Email template** — 6-10 template nuovi
9. **Upload COVNO** — parsing + merge da zero
10. **Scheduled jobs** — 3 job nuovi
11. **Document Manager** — integrazione monitoring da zero
12. **Vista SQL** — da estendere significativamente

**Da chiarire**: nessun bloccante aperto. 3 assunzioni ancora in attesa (A-011, A-015, A-016). 12 confermate, 1 rigettata (A-014).

**Complessità complessiva**: Alta. Stima 43-57 gg/uomo con team di 6 persone (+3.5 gg rispetto a stima originale per: semaforo 6 colori, due tabelle spread distinte).

## Storico Aggiornamenti

### Aggiornamento 2026-05-05
- Fonte: risposte funzionale (br-clarify round 2)
- Requisiti modificati: 4 (semaforo 6 colori, chiusura automatica 3 condizioni, spread due tabelle, formule "Da attenzionare")
- Requisiti con correzione minore: 2 (eccezione valore vuoto non N/A, macchina stati v2)
- Requisiti confermati senza impatto: 10 (A-001 → A-010)
- Assunzione rigettata: A-014 (spread due tabelle distinte, non una sola)
- Delta effort: +3.5 gg/uomo
