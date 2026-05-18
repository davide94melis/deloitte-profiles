# Handoff Analisi BR Monitoraggio V6

Data: 2026-05-04
Sessione: br-analyzer fase 3 completata, fase 4 da generare

## Stato lavoro

- [x] Fase 1 — Raccolta input
- [x] Fase 2 — Conversione documentazione (in `br-docs-converted/`)
- [x] Fase 3 — Analisi gap (completata, risultati in questo file)
- [ ] Fase 4 — Generazione output (GAP_REPORT_BR.md + PIANO_IMPLEMENTAZIONE_BR.md)

## Input raccolti

### Repository

| Nome | Sigla | Path |
|------|-------|------|
| Ba-back-end | BE | `C:\Users\davmelis\Documents\Github\ba-back-end` |
| Ba-web | FE | `C:\Users\davmelis\Documents\Github\ba-web` |
| Ba-document-manager | DM | `C:\Users\davmelis\Documents\Github\ba-document-manager` |
| Ba-email-manager | EM | `C:\Users\davmelis\Documents\Github\ba-email-manager` |

### Team

| Nome | Area | Seniority | Claude Code | Note |
|------|------|-----------|-------------|------|
| Davide | BE | Senior | Si | Non disponibile settimana 1 |
| Carmine | FE | Senior | Si | Non disponibile settimana 1 |
| Alexios | Fullstack | Mid | Si | |
| Adham | FE | Junior | No | Task ultra-dettagliate |
| Georgios | FE | Junior | Si | |
| Ahmad | BE | Junior | No | Task ultra-dettagliate |

### Perimetro

Tutto il modulo Monitoraggio. ESCLUSO tutto ciò che è nel piano forbice (`plans/in-progress/2026-05-03_forbice/`):
- Stream 0-6: SACE GenAI, Booking Lender, Post-Closing Pipeline, Report Mapping, Garanzie, Nome Deal/Termination Date

### Branch base attuale

`feature/monitoring` (da git status)

## Review BR (da REVIEW_BR.md)

br-clarify eseguito il 2026-04-30. Tutti i 5 bloccanti risolti:

1. **B1** Formato ID = `M{n}C{n}E{n}` (es. M1C1E1). Occorrenze P1C2M1 sono refusi.
2. **B2** Chiusura pratica automatica (versione "old"): "Data scadenza < Today" AND assenza eventi Scaduto/Fuori Soglia
3. **B3** Status Covenant singolo segue logiche macchina stati, influenzato dai singoli Eventi
4. **B4** Semaforo 4 colori: Verde=OK, Giallo=InScadenza+DaAttenzionare, Rosso=Scaduto+FuoriSoglia, Grigio=Conclusa
5. **B5** Utente Deloitte ereditato da censimento, senza auto-assegnazione

16 assunzioni adottate (A-001 → A-016, nessuna risposta dal funzionale):
- A-001: Formula = Scadenza - Today (positivo=rimanenti, negativo=ritardo)
- A-002: "IN CARICO A" = "Utente Banca" quando nessun doc in lavorazione e nessun evento critico
- A-003: Dashboard "In Scadenza" stesse colonne di "Scaduti"
- A-004: Typo corretto "chiudere anticipatamente la pratica"
- A-005: Label = "Termina Pratica di Monitoraggio"
- A-006: Termine = "Valore Monitoraggio" (dal BR)
- A-007: Label = "IN CARICO A" (con spazi)
- A-008: Colonna "ID Covenant" = M{n}C{n}, colonna separata "ID Evento" = M{n}C{n}E{n}
- A-009: ID pratica = "M{n}" senza padding zero
- A-010: Seconda colonna "Note" nel mockup è errore, ignorata
- A-011: Upload COVN0/DB Obblighi in XLSX, aggiornamento incrementale per Tracking ID, file vince su automatici ma non su manuali Deloitte
- A-012: "10% dalla soglia" = valore entro 90%-100% soglia. Due condizioni in OR.
- A-013: Evento con eccezione: conteggiato OK, data avvenuto = data eccezione, valore = "N/A"
- A-014: Storico Spread definitivo = tabella sez. 7.3.5 (Data Rif, Data Avvenuto, Valore, Cluster, Margine)
- A-015: Mailing list per notifiche automatiche: scadenza (7gg prima), scaduto, eccezione, variazione spread. Tutti in TO.
- A-016: "Tempo reale" = polling FE ogni 60 secondi

## Requisiti funzionali estratti dal BR

### 1. Creazione Pratica Monitoraggio
- Auto-creata dopo completamento Post-Closing, collegata via Tracking ID
- Alimentata da covenant numerici estratti in Post-Closing
- Genera scadenziere automatico basato su periodicità e Termination Date
- ID: M{n} (pratica), M{n}C{n} (covenant), M{n}C{n}E{n} (evento)
- Eventi creati progressivamente fino a Termination Date (1/3/6/12 mesi in base a periodicità)
- Utenti ISP e Deloitte ereditati dal censimento

### 2. Tabella Pratiche Monitoraggio
- Menu laterale > "Pratiche Monitoraggio"
- Filtri: Utente Banca, Tracking ID, ID Monitoraggio, Nome Deal, Semaforo (multi-select 6 stati)
- Colonne: ID, Tracking ID, Nome Deal, Utente Banca, Utente Deloitte, Tipologia, #Scaduti, #InScadenza, #DocInLavorazione, Semaforo, Data Creazione, Data Fine, Azione (Termina Pratica)
- Paginazione 20 righe, ordinamento, export Excel
- Click riga → dettaglio pratica

### 3. Flusso ISP (Utente Banca)
- Header pratica: ID, NDG, Tipologia, ID Contratto, Utente Banca, Stato
- 3 tab: Covenant, Spread, Scadenziere

#### 3.1 Sezione Covenant ISP
- Overview: Card totale covenant, Card scadenze (OK/InScadenza/KO), Card avanzamento (Sezione, IN CARICO A, STATO LAVORAZIONE)
- Upload Compliance Certificate: drag&drop, formati accettati, max 3MB
- Storico documenti: in lavorazione + lavorati, con status, giorni giacenza, visualizza/download/commento
- Vista dettaglio assegnazione: viewer doc sx, indicatori covenant dx
- Tabella Monitoraggi Scaduti/Fuori Soglia: 12 colonne + azione "Salta Monitoraggio"/"Ignora Soglia"
- Tabella Monitoraggi In Scadenza: stesse colonne
- Tabella "Tutti i Covenant": 13 colonne aggregate per covenant

#### 3.2 Dettaglio Covenant Singolo
- Stato avanzamento: tipologia, valore limite, variabile, note, monitoraggi da svolgere
- Semafori circolari: OK/InScadenza/Scaduti
- Scadenziere monitoraggi: 11 colonne con tutti gli eventi del covenant

#### 3.3 Eccezioni
- Modal "Richiesta di Eccezione": motivazione (2-2000 char) + upload opzionale
- Eccezione confermata → evento passa a Green
- Azione diventa "Visualizza Eccezione"

#### 3.4 Sezione Spread ISP
- Tabella Spread (read-only): Cluster, Sottostante, Valore Sottostante, Margine
- Cluster attivo evidenziato
- Storico Spread: Data Variazione, Documento, Azione (Scarica), Cluster

#### 3.5 Sezione Scadenziere ISP
- Filtro per tipologia covenant
- Tabella cronologica tutti gli eventi: 9 colonne

### 4. Flusso Deloitte
- Stesse overview + funzionalità aggiuntive:
  - Conferma/Rifiuta documenti
  - Assegnazione Compliance Certificate (manuale o GenAI)
  - Flusso manuale: Aggiungi Covenant → seleziona tipologia + valore + data → scadenziere filtrato → Assegna → Conferma → Concludi
  - Flusso GenAI: spinner → N selettori con proposta → Modifica Dati / Conferma
  - Modifica Dati Covenant: tipologia, valore limite, variabile, scadenze future, aggiungi/rimuovi monitoraggio
  - Gestione Eccezioni: visualizza motivazione, scarica documenti, "Annulla Eccezione"

### 5. Scadenziere Unificato
- Vista consolidata cronologica tutti gli eventi della pratica
- Filtro per tipologia
- 9 colonne, paginazione 20 righe

### 6. Mailing List
- Pagina ricerca pratica con filtri (Nome Deal, ID Censimento, Tracking ID, ID Monitoraggio)
- Tabella pratiche: 11 colonne + azione "Richiedi Aggiornamento"
- Upload AD Form → estrazione GenAI → report mailing list aggiornata
- Vista Deloitte: Conferma/Rifiuta AD Form, "Estrai con GenAI", "Conferma e Aggiorna"
- Stati: Doc Incompleta → Doc Completa → Doc Confermata → Verifica in corso → Report Generato

### 7. Spread
- Deloitte crea tabella spread (manuale)
- Disclaimer "tabella non verificata" finché non censita
- Tabella Covenant: ID, Covenant (dropdown), Valore Limite Inf, Valore Limite Sup, Margine (%), Azione, Cestino
- "Nessuna tabella da inserire" → disabilita sezione
- Storico Spread automatico: Data Rif, Data Avvenuto, Valore Monitorato, Cluster, Margine (%)
- Notifica ISP a ogni nuova riga storico
- ISP vede solo dopo validazione Deloitte

### 8. Dashboard Monitoraggio
- Selettore Censimento/Monitoraggio in alto a destra
- Donut chart: 4 segmenti (OK/InScadenza/Scaduto/Concluse), totale al centro, polling 60s
- 2 card COVNO: data ultimo aggiornamento, pulsante Aggiorna (upload XLSX), Storico
- Tabella Monitoraggi Scaduti (dashboard): 13 colonne, max 5 righe, link "Vedi tutte"
- Pagina completa Scaduti: stesse colonne, filtri avanzati, export Excel, 20 righe/pagina
- Tabella Monitoraggi In Scadenza (dashboard): stesse colonne, max 5 righe
- Pagina completa In Scadenza: filtri, export, 20 righe/pagina
- Tabella Pratiche Recenti: ultime 10, 12 colonne, "Vedi tutte" → Tabella Pratiche

## Macchina a stati (da BA_Monitoraggio_Macchina_Stati.xlsx — versione aggiornata)

### Livello Pratica di Monitoraggio
| Stato | Colore | Logica |
|-------|--------|--------|
| Monitoraggio OK | Green | Assenza eventi InScadenza/Scaduto/FuoriSoglia/DaAttenzionare E eventi futuri presenti |
| Monitoraggio in Scadenza | Yellow | Assenza eventi Scaduto/FuoriSoglia + almeno 1 evento InScadenza |
| Monitoraggio da attenzionare | Yellow | Assenza Scaduto/FuoriSoglia/InScadenza + almeno 1 evento DaAttenzionare |
| Monitoraggio Scaduto | Red | Assenza FuoriSoglia + almeno 1 evento Scaduto |
| Monitoraggio Fuori Soglia | Red | Almeno 1 evento FuoriSoglia |
| Monitoraggio Concluso | Grey | Data scadenza < Today AND assenza Scaduto/FuoriSoglia |

### Livello Covenant (stessa logica della pratica, applicata ai suoi eventi)

### Livello Documento
| Stato | Sottostato | Logica | Owner |
|-------|-----------|--------|-------|
| In attesa di conferma | - | Documento caricato, da confermare Deloitte | Deloitte (Manuale) |
| Verifica dati in corso | Estrazione GenAI In corso | Deloitte ha avviato GenAI | Deloitte (GenAI) |
| Verifica dati in corso | Estrazione GenAI da Validare | GenAI terminata, da verificare | Deloitte (GenAI) |
| Completato | - | Deloitte ha validato/caricato manualmente | - |

### Livello Evento di Monitoraggio
| Stato | Colore | Logica |
|-------|--------|--------|
| Monitoraggio OK | Green | Valore in linea alle regole CdF |
| Monitoraggio in Scadenza | Yellow | Data Scadenza < 7gg da Today |
| Monitoraggio da attenzionare | Yellow | Valore si discosta 10% da soglia, OPPURE discosta 30% + differito 40% da precedente |
| Fuori Soglia | Red | Valore non rispetta regole CdF |
| Monitoraggio Scaduto | Red | Data Scadenza > Today (passata) |
| Nessuno Stato | - | Non in prossimità della scadenza |

## Disallineamenti col codice (da REVIEW_BR.md, verificati)

| # | Concetto BR | Nel codice | File chiave | Da fare |
|---|-------------|-----------|-------------|---------|
| D-001 | Stati doc monitoraggio (In attesa conferma, Verifica in corso, Estrazione GenAI in corso/da validare, Completato) | `DocumentStateEnum`: TO_BE_UPLOAD, TO_BE_DOWNLOAD, TO_BE_VALIDATED, VALIDATED, UPLOADED, REJECTED | `enumeration/document/DocumentStateEnum.java` | Nuovo enum o estensione per monitoring |
| D-002 | Sottostati Deloitte GenAI a livello documento | `SectionStateEnum` a livello sezione | `enumeration/agency_desk/SectionStateEnum.java` | Nuovo enum a livello documento |
| D-003 | Periodicità: Annuale, Semestrale, Trimestrale, Mensile | `MonitoringPeriodicityEnum` include ALTRO | `enumeration/agency_desk/MonitoringPeriodicityEnum.java` | Verificare se mantenere ALTRO |
| D-004 | ID Pratica "M{n}" | Nessun generatore | Nessuno | Creare da zero |
| D-005 | Entità Covenant (Tipologia, Valore Limite, Segno, Periodicità, Variabile, Note) | Solo enum (CovenantTypeNumericEnum ~53 valori, CovenantSignEnum) | `enumeration/agency_desk/` | Creare entità JPA completa |
| D-006 | Entità Evento Monitoraggio (ID, Date, Valore, Status) | Nessuna | Nessuno | Creare da zero |
| D-007 | Macchina stati multi-livello (Pratica→Covenant→Evento→Documento) | PSM generica per censimento | `service/psm/PsmStateService.java` | 4 livelli di stato indipendenti |
| D-008 | Email monitoraggio (scadenza, eccezione, rifiuto, spread) | Nessun template monitoring | `enumeration/EmailTemplateEnum.java` | 6-10 nuovi template |
| D-009 | Spread: tabella + storico + collegamento covenant | Solo "spreadsheet" (fogli Excel) | Nessuno | Creare da zero |
| D-010 | Mailing list CRUD + AD Form + GenAI | Solo check booleano Booking | `service/psm/PsmStateService.java` | Modulo completo da zero |
| D-011 | FE: dashboard, tabelle, dettaglio, spread, scadenziere, mailing | Solo MonitoringComponent generico (step wizard) | `Ba-web` | Tutte le pagine da zero |
| D-012 | DM: compliance certificate, upload, validazione, stati monitoring | Nessun codice monitoring | `Ba-document-manager` | Da zero se gestione doc passa dal DM |

## Verifiche codebase aggiuntive (sessione corrente)

- **Vista SQL**: `vw_monitoring_practice_summary` esiste già — query su `mp_practice` con join a `mp_psm_state`, filtra `msi.id != 1` (non Census)
- **Nessuna entità Covenant nel domain**: confermato con Grep su `src/main/java/.../domain/` — solo enum esistono
- **28 file Java menzionano monitoring**: principalmente enum, DTO pratica, service pratica, PSM
- **PracticeMonitoringKey.java** esiste in `utils/` — da verificare
- **ScheduledService.java** menziona monitoring — potrebbe avere job batch esistenti

## Piano forbice (escluso dal perimetro)

Branch: `feature/sprint-wave-3-forbice`
Stream 0: Foundation (enum, entity BookingLender, migration) — Ahmad
Stream 1A/1B: SACE GenAI BE/FE — Davide/Alexios
Stream 2A/2B: Booking Lender BE/FE — Ahmad/Carmine
Stream 3A/3B: Post-Closing Pipeline BE/FE — Davide/Carmine+Adham
Stream 4A/4B: Report Mapping BE/FE — Ahmad/Georgios
Stream 5A/5B: Garanzie BE/FE — Ahmad/Adham
Stream 6A/6B: Nome Deal BE/FE — Ahmad/Georgios

## Istruzioni per prossima sessione

1. Leggere questo file HANDOFF_ANALYSIS.md
2. Leggere REVIEW_BR.md (plans/todo/2026-05-04_monitoring/REVIEW_BR.md)
3. Consultare i file in br-docs-converted/ per dettagli specifici se necessario
4. Generare GAP_REPORT_BR.md seguendo il template della skill br-analyzer (Fase 4.1)
5. Generare PIANO_IMPLEMENTAZIONE_BR.md seguendo il template della skill br-analyzer (Fase 4.2)
6. Ricordare: Davide e Carmine NON disponibili settimana 1
7. Ricordare: Ahmad e Adham SENZA Claude Code → task ultra-dettagliate
8. Ricordare: branch base è `feature/monitoring`
9. Il FE (Ba-web) NON è stato esplorato in questa sessione — esplorarlo per capire i pattern Angular esistenti
