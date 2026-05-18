# PIANO DI SVILUPPO - Risoluzione GAP BR v28

**Data originale:** 2026-04-22
**Data aggiornamento:** 2026-05-06
**Riferimento:** GAP_ANALYSIS_BR_v27.md (aggiornato 2026-04-27)
**Branch base:** `feature/sprint-wave-3-forbice`
**Tooling:** Ogni sviluppatore lavora tramite Claude Code

---

## TEAM

| Nome | Area | Responsabilità |
|------|------|---------------|
| **Daniele** | BE | Code review, merge approval. Non scrive codice |
| **Davide** | BE | Logica complessa: pipeline GenAI, data fusion, report generation, mutua esclusione |
| **Ahmad** | BE | Enum, entity, DTO, migration SQL, endpoint CRUD, pipeline garanzie |
| **Alexios** | FE | Componenti complessi: vista GenAI SACE, classificazione documenti |
| **Carmine** | FE | Componenti complessi: modale booking, vista covenant numerici/descrittivi |
| **Adham** | FE | Sezioni UI: obblighi informativi, Format PGA, accordion post-closing |
| **Georgios** | FE | Integrazione report, wiring servizi, Nome Deal/Termination Date, document viewer |

---

## STANDARD DI DOCUMENTAZIONE

Ogni agente Claude Code che lavora su un qualsiasi stream DEVE produrre documentazione del codice come parte integrante del deliverable. Non è un task separato: la documentazione viene scritta contestualmente al codice.

### Backend (Java/Spring Boot)

**Javadoc obbligatorio su:**
- Ogni classe nuova: descrizione scopo, responsabilità, relazione con altre classi
- Ogni metodo pubblico: descrizione comportamento, `@param`, `@return`, `@throws`
- Ogni enum value nuovo: commento inline che spiega il significato nel contesto di business

**Documentazione API REST:**
- Ogni nuovo endpoint: commento Javadoc sul metodo controller con path, metodo HTTP, descrizione, codici di risposta

**Migration SQL:**
- Header con: scopo della migration, data, stream di riferimento, eventuali dipendenze da altre migration

### Frontend (Angular/TypeScript)

**TSDoc obbligatorio su:**
- Ogni componente nuovo: descrizione scopo, inputs/outputs, comportamento
- Ogni servizio nuovo: descrizione scopo, metodi principali
- Ogni interface/type nuovo: descrizione campi, vincoli, relazione con API BE

### Regola generale

- La documentazione deve spiegare il **perché** e il **contesto di business**, non il **cosa**
- Ogni riferimento al BR deve citare la sezione: es. `@see BR v28 sez. 3.1.26`
- Ogni riferimento al mockup deve citare la slide: es. `Mockup PostClosing v7 slide 6`

---

## GRAFO DELLE DIPENDENZE

```
STREAM 0  (Foundation) ── Ahmad
   │
   ├─────────────────────────┐──────────────────────────────┐──────────────────┐
   ▼                         ▼                              ▼                  ▼
STREAM 1A (SACE BE)    STREAM 2A (Booking BE)    STREAM 3A (Post-Closing BE)  STREAM 6A
   Davide                 Ahmad                     Davide                   (Nome Deal BE)
   │                         │                         │                       Ahmad
   ▼                         ├──────────┐              ├───────────────┐          │
STREAM 1B (SACE FE)         ▼          ▼              ▼               ▼          ▼
   Alexios            STREAM 2B   STREAM 4A     STREAM 3B        STREAM 5A   STREAM 6B
                    (Booking FE)  (Report BE)  (Covenant+Obblighi FE) (Garanzie BE) (Nome Deal FE)
                      Carmine       Ahmad        Carmine+Adham      Ahmad       Georgios
                                      │                                │
                                      ▼                                ▼
                                  STREAM 4B                       STREAM 5B
                                  (Report FE)                    (Garanzie FE)
                                   Georgios                        Adham
```

**Regola:** ogni stream si apre da `feature/sprint-wave-3-forbice` DOPO che le sue dipendenze sono state mergiate.

---

## STREAM 0 — Foundation (Enums, Modelli, Migrazioni)

**Branch:** `feature/gap-0-foundation`
**Assegnato a:** Ahmad
**Dipendenze:** nessuna
**Review:** Daniele
**Stima:** 3-4 giorni (ampliato per nuovi enum dropdown post-closing)
**Priorità merge:** PRIMO — blocca tutti gli altri stream

### Cosa fare

**1. Enum BE**

`AIUseCase.java` — aggiungere:
```java
AGENCYDESK_SACE("SACE"),
AGENCYDESK_POST_CLOSING("POST_CLOSING")
```

`GenAiFormKeyEnum.java` — aggiungere tutti i campi SACE (19), Covenant Numerici (14), Covenant Descrittivi (6), Obblighi Informativi (7).

`GenAiDocumentType.java` (NUOVO) — enum con tutti i document type dei tracciati:
```java
AD_FORM_PRE_CLOSING, AD_FORM_BOOKING, TERMSHEET_SACE, CDF_SACE, CDF_POST_CLOSING,
GARANZIA_IPOTECA_RE, GARANZIA_IPOTECA_NON_RE, GARANZIA_IPOTECA_NAVI_AEREI,
GARANZIA_CESSIONE_CREDITI_PGA, GARANZIA_PEGNO_CC_TERZI, GARANZIA_APENDICE_VINCOLO,
GARANZIA_PEGNO_AZIONI, GARANZIA_PEGNO_CC, GARANZIA_PEGNO_AZIONI_TERZI,
GARANZIA_PEGNO_QUOTE, GARANZIA_FIDE_PAT, GARANZIA_PRIVILEGIO_SPEC,
GARANZIA_CESSIONE_CREDITI_GSE, GARANZIA_PEGNO_MERCI_PGA,
GARANZIA_PEGNO_ATTIVITA_IMM_PGA, GARANZIA_PEGNO_CREDITI_PGA
```

`PsmStateCodeEnum.java` — aggiungere 6 nuovi stati GenAI (SACE + Post-Closing).

**[AGGIORNATO 2026-04-27] Nuovi enum dropdown per Post-Closing:**

`BookingEntityEnum.java` (NUOVO):
```java
ISP_MIL, ISP_FF, ISP_HK, ISP_IRL, ISP_LND, ISP_MD, ISP_NY, ISP_SN, ISP_TK, ISP_FRA
```

`CovenantTypeNumericEnum.java` (NUOVO) — ~50 voci: ICR, PFN_MOL, DSCR, ADSCR, LOAN_TO_VALUE, etc. (lista completa in BR v28 pagg. 1500-1553)

`CovenantTypeDescriptiveEnum.java` (NUOVO) — 10 voci: CHANGE_OF_CONTROL, LIMITAZIONE_OPERAZIONI_STRAORDINARIE, LIMITAZIONE_CESSIONE_ASSETS, LIMITAZIONE_DISTRIBUZIONE_DIVIDENDI, NEGATIVE_PLEDGE, OWNERSHIP_CLAUSE, PARI_PASSU, LIMITE_NUOVO_INDEBITAMENTO, CASH_SWEEP, CASH_TRAP

`CovenantMonitoringDocumentEnum.java` (NUOVO) — 7 voci: BILANCIO_CONSOLIDATO, BILANCIO_INDIVIDUALE, SITUAZIONE_CONTABILE, VISURA_CAMERALE, ESTRATTO_CONTO, NON_PREVISTO, ALTRO

`MonitoringPeriodicityEnum.java` (NUOVO) — 5 voci: ANNUALE, SEMESTRALE, TRIMESTRALE, MENSILE, ALTRO

`CovenantSignEnum.java` (NUOVO) — 5 voci: GT, GTE, EQ, LT, LTE

`InformativeDutyTypeEnum.java` (NUOVO) — ~35 voci (lista completa in BR v28 pagg. 1366-1410)

`PostClosingCommentFieldEnum.java` — aggiungere: `NOME_DEAL`, `TERMINATION_DATE`

**2. Entity/DTO BE**

`BookingLender.java` (NUOVO) — Entity con bankName, bankAddress, ndg, autogenerated, practice.

`BookingLenderDTO.java` (NUOVO)

Aggiornare `Practice.java`: sostituire `List<String> bookingLenderNdgs` con `List<BookingLender> bookingLenders`.

**[AGGIORNATO 2026-04-27]** Aggiungere a `Practice.java`:
```java
private String dealName;      // Nome Deal per post-closing
private LocalDate terminationDate;  // Termination Date per post-closing
```

Aggiornare `PracticeDTO.java` con i nuovi campi.

**3. Migration SQL**

- Tabella `mp_booking_lender` (come da piano originale)
- Nuovi stati PSM per GenAI SACE e Post-Closing
- **[AGGIORNATO 2026-04-27]** Colonne `deal_name` e `termination_date` su `mp_practice`

**4. Enum FE**

Aggiungere i nuovi stati PSM. Creare gli enum dropdown per i componenti post-closing.

---

## STREAM 1A — SACE GenAI Pipeline (Backend)

**Branch:** `feature/gap-1a-sace-genai-be`
**Assegnato a:** Davide
**Dipendenze:** Stream 0 merged
**Review:** Daniele
**Stima:** 3-4 giorni

### Cosa fare

Come da piano originale (invariato). Sintesi:
1. `engagementAiSaceTermsheetProcess()` e `engagementAiSaceCdfProcess()` — 2 chiamate GenAI separate
2. `SaceDataFusionService` — fusione CdF + Termsheet con priorità CdF
3. Callback SACE con tracking completamento parziale
4. Report SACE (`BOOKING_SACE_GUARANTEE_REPORT`) — generazione automatica

**[AGGIORNATO 2026-04-27]** Aggiungere:
- Logica "Modifica report caricati": endpoint per sostituire un report già caricato
- Dialog "Conferma dati GenAI e chiudi": endpoint di validazione con transizione stato

---

## STREAM 1B — SACE GenAI Vista (Frontend)

**Branch:** `feature/gap-1b-sace-genai-fe`
**Assegnato a:** Alexios
**Dipendenze:** Stream 0 merged, Stream 1A merged
**Review:** Daniele, Carmine (peer)
**Stima:** 3-4 giorni

### Cosa fare

Come da piano originale (invariato). Sintesi:
1. Componente `sace-genai-extraction/` con tabella 19 campi
2. Integrazione in `sace-phase.component.ts`
3. Service FE per endpoint SACE

**[AGGIORNATO 2026-04-27]** Aggiungere:
- Pulsante "Modifica report caricati" dopo upload report
- Anteprima report con dropdown arrow → box con preview tabellare + download
- Dialog "Conferma dati GenAI e chiudi" con messaggio e Conferma/Annulla

---

## STREAM 2A — Booking Lender API (Backend)

**Branch:** `feature/gap-2a-booking-lender-be`
**Assegnato a:** Ahmad
**Dipendenze:** Stream 0 merged
**Review:** Daniele
**Stima:** 2-3 giorni

### Cosa fare

Come da piano originale (invariato). Sintesi:
1. `BookingLenderRepository`, `BookingLenderMapper`
2. Service booking con entity `BookingLender` al posto di `List<String>`
3. Endpoint AD Form aggiuntivi con trigger GenAI

**[AGGIORNATO 2026-04-27]** Aggiungere:
- Validazione max 15 lender manuali
- Endpoint delete lender (funzionalità "Cestino") — solo prima di "Conferma Documentazione"
- Logica lock sezione dopo "Conferma Documentazione": impedire sovrascrittura/sostituzione documenti

---

## STREAM 2B — Booking Lender Pop-up (Frontend)

**Branch:** `feature/gap-2b-booking-lender-fe`
**Assegnato a:** Carmine
**Dipendenze:** Stream 0 merged, Stream 2A merged
**Review:** Alexios (peer)
**Stima:** 3-4 giorni

### Cosa fare

Come da piano originale (invariato). Sintesi:
1. Dialog con 3 campi (Nome Banca, Indirizzo, NDG)
2. Refactor `booking-phase.component.ts`
3. Sezione "AD Form Aggiuntivi"

**[AGGIORNATO 2026-04-27]** Aggiungere:
- Limite visuale 15 lender (contatore + disabilitazione pulsante)
- Icona "Cestino" per ogni slot creato — disabilitata dopo "Conferma Documentazione"
- Pulsante "Conferma Documentazione" con logica lock completa

---

## STREAM 3A — Post-Closing Pipeline Completa (Backend)

**Branch:** `feature/gap-3a-post-closing-be`
**Assegnato a:** Davide
**Dipendenze:** Stream 0 merged
**Review:** Daniele
**Stima:** 7-8 giorni (ampliato da 5-6 per mutua esclusione + 4 viste GenAI + obblighi informativi)

### Cosa fare

**1. Post-Closing AI Process CdF** — come da piano originale:
- `engagementAiPostClosingCdfProcess()` con document type `CDF_POST_CLOSING`
- Processing 5 CSV (obblighi periodici, obblighi evento, covenant numerici, covenant numerici variabili opzionale, covenant descrittivi)

**2. Endpoint manuale "Data di scadenza"** — come da piano originale:
- PUT `/api/agency-desk/practices/{practiceCode}/covenant/{recordId}/expiration-date`

**3. Report Builder: Covenant Numerici** — `CovenantNumericSheetBuilder.java`:
- 14 colonne come da specifica BR v28 (vedi GAP #4)
- Recuperare Data di scadenza dalla entry manuale

**4. [AGGIORNATO 2026-04-27] Report Builder: Covenant Descrittivi** — `CovenantDescriptiveSheetBuilder.java`:
- 6 colonne: Booking Entity, Nome cliente, Tipologia Covenant, Articolo, Documento Monitoraggio, Soggetto diverso dal borrower
- Dati da CsvDataRecord category `COVENANT_DESCRIPTIVE`

**5. [AGGIORNATO 2026-04-27] Report Builder: Obblighi Informativi** — `ObblighiInformativiSheetBuilder.java`:
- 7 colonne: Tipologia Obbligo, Data di riferimento, Data di scadenza, Periodicità, Articolo, Nome società, Descrizione Obbligo
- Due sezioni: periodici + evento
- Predisporre merge multi-fonte (CdF + Garanzie da Stream 5A)

**6. [NUOVO 2026-04-27] Mutua Esclusione "Carica Report" / "Avvia GenAI"**

Nuova logica in `GeneratedReportFacade.java` o service dedicato:
- Per ogni accordion: tracciare lo stato del percorso scelto (MANUAL / GENAI / NOT_CHOSEN)
- Se `uploadManualReport()` viene chiamato: impostare percorso = MANUAL, impedire avvio GenAI
- Se `startGenAIExtraction()` viene chiamato: impostare percorso = GENAI, impedire upload manuale
- Differenziare slot obbligatori per tipo accordion:
  - CdF: 3 slot (Obblighi informativi, Covenant Numerici, Covenant Descrittivi)
  - Garanzia standard: 2 slot (Obblighi Informativi, Format PGA)
  - Garanzia SACE: 1 slot (Obblighi Informativi)
- "Modifica Report Caricati": endpoint che riapre la finestra di upload per documenti già caricati. Attivo fino a chiusura post-closing.

**7. [NUOVO 2026-04-27] CRUD CsvDataRecord per editing manuale**

Endpoint generici per le 4 viste GenAI post-closing:
- POST per "Aggiungi dato" (nuova riga manuale)
- PUT per "Modifica" (edit riga esistente)
- DELETE per "Cestino" (elimina riga)
- GET per recupero dati per vista (con filtro per category)

Questi endpoint servono TUTTE le viste: Obblighi, Covenant Num, Covenant Desc, Format PGA.

**File coinvolti:**
- `AIMetadataProcessService.java`, `AsyncAIMetadataProcess.java` (modifica)
- `CovenantNumericSheetBuilder.java`, `CovenantDescriptiveSheetBuilder.java`, `ObblighiInformativiSheetBuilder.java` (nuovi)
- `GeneratedReportFacade.java`, `GeneratedReportExcelService.java`, `GeneratedReportExcelTemplateFactory.java` (modifica)
- Controller post-closing (modifica — endpoint mutua esclusione, CRUD record, avvio GenAI)
- `CsvImportService.java` (modifica — supportare 5 CSV)

---

## STREAM 3B — Post-Closing Viste GenAI (Frontend)

**Branch:** `feature/gap-3b-post-closing-fe`
**Assegnato a:** Carmine (Covenant Num + Desc) + Adham (Obblighi Inf)
**Dipendenze:** Stream 0 merged, Stream 3A merged
**Review:** Alexios (peer)
**Stima:** 6-7 giorni (ampliato da 4-5: 3 viste complete con dropdown + mutua esclusione UI)

### Cosa fare

**1. Componente Covenant Numerici** (Carmine):
- Vista tabellare con 14 colonne da BR v28
- 6 dropdown: Booking Entity (10), Tipologia Covenant (~50), Doc Monitoraggio (7), Periodicità (5), Segno covenant (5), Covenant variabile (2)
- "Data di scadenza" come campo date manuale (non precompilato GenAI)
- Pulsante "Aggiungi dato" → nuova riga editabile con tutti i dropdown
- "Modifica" / "Cestino" per ogni riga

**2. Componente Covenant Descrittivi** (Carmine):
- Vista tabellare con 6 colonne
- 3 dropdown: Booking Entity (10), Tipologia Covenant (10), Doc Monitoraggio (6)
- Stessa struttura Aggiungi/Modifica/Cestino

**3. Componente Obblighi Informativi** (Adham):
- Vista tabellare con 7 colonne
- 2 dropdown: Tipologia Obbligo (~35), Periodicità (5)
- Due sezioni/tab: "Periodici" e "Evento"
- Stessa struttura Aggiungi/Modifica/Cestino

**4. Componente Document Viewer (pannello sinistro)** (condiviso):
- Separatore verticale con freccia espandibile
- Visualizzazione navigabile del documento sorgente
- Evidenziazione sezione di riferimento per i dati selezionati

**5. [NUOVO 2026-04-27] Mutua Esclusione UI**:
- Per ogni accordion: mostrare entrambi i pulsanti "Carica Report" / "Avvia Estrazione GenAI"
- Al click di uno: disabilitare l'altro, aggiornare stato
- Dopo upload completo tutti i report obbligatori: sostituire con "Modifica Report Caricati"
- "Modifica Report Caricati": apre stessa finestra upload, permette modifica/eliminazione

**6. Dialog "Conferma dati GenAI e chiudi"**:
- Per tutte le viste: pulsante che apre dialog con messaggio "Vuoi confermare la correttezza dei dati estratti e avviare la produzione del Report?"
- Conferma → chiude vista, passa accordion allo stato successivo

**7. Integrazione in post-closing phase component**:
- 3 tab nella vista GenAI CdF: Obblighi Informativi, Covenant Descrittivi, Covenant Numerici
- Progress indicator durante GenAI in corso

---

## STREAM 4A — Report Mapping + Booking GenAI (Backend)

**Branch:** `feature/gap-4a-report-mapping-be`
**Assegnato a:** Ahmad
**Dipendenze:** Stream 2A merged
**Review:** Daniele
**Stima:** 2-3 giorni

### Cosa fare

Come da piano originale (invariato):
1. `engagementAiBookingAdFormProcess()` con document type `AD_FORM_BOOKING`
2. Mailing List Integration: raccogliere dati pre-closing + booking

---

## STREAM 4B — Report Integration FE (Frontend)

**Branch:** `feature/gap-4b-report-integration-fe`
**Assegnato a:** Georgios
**Dipendenze:** Stream 2B merged
**Review:** Carmine (peer)
**Stima:** 2-3 giorni

### Cosa fare

Come da piano originale (invariato):
1. Vista estrazione GenAI per AD Form Booking
2. Download report aggiornati (Mailing List con dati booking)
3. Stato upload AD Form

---

## STREAM 5A — Pipeline Garanzie Post-Closing (Backend)

**Branch:** `feature/gap-5a-garanzie-be`
**Assegnato a:** Ahmad
**Dipendenze:** Stream 3A merged
**Review:** Daniele, Davide (peer)
**Stima:** 4-5 giorni (ampliato da 3-4 per Format PGA)

### Cosa fare

Come da piano originale per punti 1-5 (pipeline GenAI garanzie, CSV processing, merge obblighi informativi, callback).

**[AGGIORNATO 2026-04-27] Aggiungere:**

**6. Format PGA Builder** — `FormatPgaSheetBuilder.java` (NUOVO):
- 2 colonne: Campo estratto, Valore campo
- Per Ipoteca RE/non-RE/Navi-Aerei: Excel con 2 sheet (atto ipoteca + dati specifici)
- Per tutti gli altri tipi: 1 sheet standard
- Report code: `CENS_ASSURANCES` o nuovo code dedicato

**7. Endpoint per avvio GenAI per singola garanzia**:
- Usa la logica di mutua esclusione da Stream 3A
- Ogni accordion garanzia ha il suo percorso indipendente (manuale/GenAI)

---

## STREAM 5B — Pipeline Garanzie Post-Closing (Frontend)

**Branch:** `feature/gap-5b-garanzie-fe`
**Assegnato a:** Adham
**Dipendenze:** Stream 3B merged, Stream 5A merged
**Review:** Carmine (peer)
**Stima:** 3-4 giorni (ampliato da 2-3 per Format PGA view)

### Cosa fare

Come da piano originale per punti 1-4 (sezione upload garanzie, vista risultati, integrazione report).

**[AGGIORNATO 2026-04-27] Aggiungere:**

**5. Vista Format PGA** — componente per visualizzare/editare dati Format PGA:
- Tabella 2 colonne (Campo estratto, Valore campo) + Azioni
- Pulsante "Aggiungi dato" / "Modifica" / "Cestino"
- Applicare mutua esclusione (Carica Report / Avvia GenAI) per ogni accordion garanzia

**6. Differenziazione per tipo garanzia**:
- CdF: 3 viste (da Stream 3B)
- Garanzia standard: 2 viste (Obblighi Informativi + Format PGA)
- Garanzia SACE: 1 vista (Obblighi Informativi solo)

---

## STREAM 6A — Nome Deal & Termination Date (Backend)

**Branch:** `feature/gap-6a-nome-deal-be`
**Assegnato a:** Ahmad
**Dipendenze:** Stream 0 merged
**Review:** Daniele
**Stima:** 1-2 giorni
**Origine:** NUOVO (GAP #9)

### Cosa fare

**1. Endpoint CRUD Nome Deal**:
```
PATCH /api/agency-desk/post-closing/practices/{practiceId}/deal-name
Body: { "dealName": "Deal Rapallo" }
```

**2. Endpoint CRUD Termination Date**:
```
PATCH /api/agency-desk/post-closing/practices/{practiceId}/termination-date
Body: { "terminationDate": "2027-06-30" }
```

**3. Commenti per Nome Deal e Termination Date**:
- Aggiungere endpoint commenti in `PostClosingPracticeDataController` seguendo il pattern di NDG e ContractId
- Usare `PostClosingCommentFieldEnum.NOME_DEAL` e `TERMINATION_DATE`

**File coinvolti:**
- `PostClosingPracticeDataController.java` (modifica — 4 nuovi endpoint)
- `PracticeFacade.java` (modifica — nuovi metodi update)
- `Practice.java` (modifica — nuovi campi, da Stream 0)

---

## STREAM 6B — Nome Deal & Termination Date (Frontend)

**Branch:** `feature/gap-6b-nome-deal-fe`
**Assegnato a:** Georgios
**Dipendenze:** Stream 0 merged, Stream 6A merged
**Review:** Carmine (peer)
**Stima:** 1-2 giorni
**Origine:** NUOVO (GAP #9)

### Cosa fare

**1. Sezione Nome Deal nella vista post-closing**:
- Campo testo con stato: "Da caricare" → input → "Dato caricato"
- Pulsante "Carica nome Deal"
- Dopo caricamento: visualizzazione dato con possibilità commento

**2. Sezione Termination Date nella vista post-closing**:
- Campo data con stesso workflow di Nome Deal
- Posizionamento: sopra l'accordion CdF, accanto a Nome Deal

**3. Commenti**:
- Integrare i commenti per entrambi i campi usando i service esistenti

---

## TIMELINE

> **Nota:** Timeline estesa a 5 settimane per l'aggiunta di Stream 6A/6B, l'ampliamento
> di Stream 0, 3A, 3B, 5A, 5B, e la gestione di 4 viste GenAI post-closing.

```
SETTIMANA 1
═══════════════════════════════════════════════════════════════════

Lun-Mar (Day 1-2):
  Ahmad     → Stream 0 (Foundation + tutti i nuovi enum dropdown)  [CODICE + DOC]
  Alexios   → Prototipo componente SACE con dati mock              [CODICE + DOC]
  Carmine   → Prototipo modale booking con dati mock               [CODICE + DOC]
  Adham     → Analisi post-closing views, setup comp.               [ANALISI]
  Georgios  → Analisi UI report booking + Nome Deal                 [ANALISI]
  Davide    → Analisi AIMetadataProcessService pattern              [ANALISI]
  Daniele   → Review Stream 0 parziale                              [REVIEW]

Mer-Gio (Day 3-4):
  Ahmad     → Completa Stream 0 (enum dropdown numerosi)            [CODICE + DOC]
  Davide    → Stream 1A (SACE GenAI BE - 2 chiamate)               [CODICE + DOC]
  Alexios   → Continua SACE FE component (mock)                    [CODICE + DOC]
  Carmine   → Continua booking modale (mock)                        [CODICE + DOC]
  Adham     → Inizio obblighi informativi view (mock data)          [CODICE + DOC]
  Georgios  → Inizio Nome Deal/Termination Date (mock)              [CODICE + DOC]
  Daniele   → Review Stream 0 → MERGE                              [REVIEW]

Ven (Day 5):
  Ahmad     → Stream 2A (Booking Lender BE)                         [CODICE + DOC]
  Davide    → Continua Stream 1A                                    [CODICE + DOC]
  Ahmad     → Stream 6A (Nome Deal BE, piccolo, in parallelo)      [CODICE + DOC]
  Tutti FE  → Continuano prototipi con mock                         [CODICE + DOC]

SETTIMANA 2
═══════════════════════════════════════════════════════════════════

Lun-Mar (Day 6-7):
  Davide    → Completa Stream 1A → MERGE                           [CODICE + DOC]
  Ahmad     → Completa Stream 2A + 6A → MERGE                     [CODICE + DOC]
  Alexios   → Stream 1B: wire SACE FE a API reale                  [CODICE + DOC]
  Carmine   → Stream 2B: wire booking FE a API reale               [CODICE + DOC]
  Georgios  → Stream 6B: wire Nome Deal FE a API reale             [CODICE + DOC]
  Adham     → Continua obblighi informativi view (mock)             [CODICE + DOC]
  Daniele   → Review Stream 1A, 2A, 6A, merge                      [REVIEW]

Mer-Ven (Day 8-10):
  Davide    → Stream 3A (Post-Closing BE: pipeline + mutua escl.)  [CODICE + DOC]
  Ahmad     → Stream 4A (Report Mapping BE)                         [CODICE + DOC]
  Alexios   → Completa Stream 1B → MERGE                           [CODICE + DOC]
  Carmine   → Completa Stream 2B → MERGE                           [CODICE + DOC]
  Georgios  → Completa Stream 6B → MERGE, poi Stream 4B            [CODICE + DOC]
  Adham     → Continua obblighi informativi + inizio covenant views  [CODICE + DOC]
  Daniele   → Review 1B, 2B, 6B, merge                              [REVIEW]

SETTIMANA 3
═══════════════════════════════════════════════════════════════════

Lun-Mer (Day 11-13):
  Davide    → Continua Stream 3A (3 report builder + mutua escl.)  [CODICE + DOC]
  Ahmad     → Completa Stream 4A → MERGE                           [CODICE + DOC]
  Carmine   → Inizio Stream 3B: covenant num + desc con mock       [CODICE + DOC]
  Adham     → Continua Stream 3B: obblighi informativi con mock    [CODICE + DOC]
  Georgios  → Completa Stream 4B → MERGE                           [CODICE + DOC]
  Alexios   → Document viewer component (pannello sinistro)         [CODICE + DOC]
  Daniele   → Review 4A, 4B, merge                                  [REVIEW]

Gio-Ven (Day 14-15):
  Davide    → Completa Stream 3A → MERGE                           [CODICE + DOC]
  Ahmad     → Stream 5A (Garanzie BE + Format PGA)                 [CODICE + DOC]
  Carmine   → Stream 3B: wire covenant views a API reale           [CODICE + DOC]
  Adham     → Stream 3B: wire obblighi informativi a API reale     [CODICE + DOC]
  Georgios  → Supporto integrazione                                 [SUPPORTO]
  Alexios   → Supporto integrazione / classificazione mock          [SUPPORTO]
  Daniele   → Review 3A, merge                                      [REVIEW]

SETTIMANA 4
═══════════════════════════════════════════════════════════════════

Lun-Mer (Day 16-18):
  Ahmad     → Completa Stream 5A → MERGE                           [CODICE + DOC]
  Carmine   → Completa Stream 3B (covenant views) → MERGE          [CODICE + DOC]
  Adham     → Completa Stream 3B (obblighi views) → MERGE          [CODICE + DOC]
  Daniele   → Review 3B, 5A, merge                                  [REVIEW]
  Alexios   → Supporto integrazione                                 [SUPPORTO]
  Georgios  → Supporto integrazione                                 [SUPPORTO]

Gio-Ven (Day 19-20):
  Adham     → Stream 5B (Garanzie FE + Format PGA view)            [CODICE + DOC]
  TUTTI     → Integration testing end-to-end                        [TEST]
  Daniele   → Review ongoing                                        [REVIEW]

SETTIMANA 5
═══════════════════════════════════════════════════════════════════

Lun-Mar (Day 21-22):
  Adham     → Completa Stream 5B → MERGE                           [CODICE + DOC]
  TUTTI     → Integration testing + bug fix                         [TEST]
  Daniele   → Review 5B, regression check finale                    [REVIEW]

Mer-Ven (Day 23-25):
  TUTTI     → Bug fix, polish, edge case                            [TEST + FIX]
  Daniele   → Regression check finale completo                      [REVIEW]
```

---

## MERGE ORDER (sequenza esatta)

| # | Branch | Quando | Chi approva |
|---|--------|--------|-------------|
| 1 | `feature/gap-0-foundation` | Fine Day 4 | Daniele |
| 2 | `feature/gap-2a-booking-lender-be` | Fine Day 7 | Daniele |
| 3 | `feature/gap-1a-sace-genai-be` | Fine Day 7 | Daniele |
| 4 | `feature/gap-6a-nome-deal-be` | Fine Day 7 | Daniele |
| 5 | `feature/gap-1b-sace-genai-fe` | Fine Day 10 | Daniele + Carmine |
| 6 | `feature/gap-2b-booking-lender-fe` | Fine Day 10 | Daniele + Alexios |
| 7 | `feature/gap-6b-nome-deal-fe` | Fine Day 10 | Daniele + Carmine |
| 8 | `feature/gap-4a-report-mapping-be` | Fine Day 13 | Daniele |
| 9 | `feature/gap-4b-report-integration-fe` | Fine Day 13 | Daniele + Carmine |
| 10 | `feature/gap-3a-post-closing-be` | Fine Day 15 | Daniele |
| 11 | `feature/gap-5a-garanzie-be` | Fine Day 18 | Daniele + Davide |
| 12 | `feature/gap-3b-post-closing-fe` | Fine Day 18 | Daniele + Alexios |
| 13 | `feature/gap-5b-garanzie-fe` | Fine Day 22 | Daniele + Carmine |

---

## REGOLE OPERATIVE

1. **Ogni dev fa rebase da `feature/sprint-wave-3-forbice`** prima di aprire la PR
2. **Nessuno tocca file di altri stream** — coordinarsi via Daniele se necessario
3. **I FE possono iniziare con dati mock** — creare un file `mock-data.ts` temporaneo
4. **Commit atomici** — un commit per ogni unità logica, messaggi chiari
5. **Test unitari obbligatori** — ogni stream BE include test
6. **Daniele revisiona entro 24h**
7. **Documentazione è parte del codice** — la PR non è approvabile senza documentazione
8. **[AGGIORNATO 2026-04-27] Enum dropdown**: i valori dei dropdown devono corrispondere ESATTAMENTE alle voci del BR v28. Verificare maiuscolo/minuscolo e caratteri speciali

---

## CHECKLIST CODE REVIEW PER DANIELE

Per ogni PR, Daniele verifica:

- [ ] **Correttezza funzionale**: il codice implementa quanto descritto nello stream
- [ ] **Aderenza pattern**: il nuovo codice segue i pattern esistenti
- [ ] **Test**: test unitari presenti
- [ ] **Javadoc/TSDoc presente e di qualità**
- [ ] **Nessun file di altri stream toccato**
- [ ] **Migration SQL**: header presente, nessun conflitto
- [ ] **Nessuna regressione**
- [ ] **[AGGIORNATO 2026-04-27] Dropdown values**: i valori dei menù a tendina corrispondono esattamente a quelli del BR v28

---

## RISCHI E MITIGAZIONI

| Rischio | Impatto | Mitigazione |
|---------|---------|-------------|
| Stream 0 ritarda (più enum di prima) | Blocca tutto | Stima estesa a 3-4gg. Daniele revisiona daily |
| Conflitti merge su file condivisi | Medio | Merge order definito |
| API GenAI non disponibile in dev | Alto | Mock/stub del servizio AI |
| Schema CSV SACE non definito | Alto | Verificare con team AI prima di iniziare |
| Path SACE invertiti nei tracciati | Alto | Verificare col team GenAI PRIMA di iniziare |
| Discrepanza Pre-Closing document type | Medio | Verificare se `AD_FORM` → `AD_FORM_PRE_CLOSING` |
| 16 tipi garanzia = alto effort testing | Alto | Pattern template + testing parametrizzato |
| Davide sovraccarico (1A + 3A ampliato) | Alto | 3A ampliato a 7-8gg. Ahmad prende 5A e 6A |
| **[NUOVO] ~50 voci Tipologia Covenant Numerici** | **Medio** | **Copia esatta dal BR, NO interpretazione. Ahmad verifica** |
| **[NUOVO] ~35 voci Tipologia Obbligo** | **Medio** | **Copia esatta dal BR. Ahmad verifica** |
| **[NUOVO] 3B ampliato (3 viste FE)** | **Alto** | **Split Carmine (covenant) + Adham (obblighi) in parallelo** |
| **[NUOVO] Mutua esclusione complex** | **Alto** | **Davide implementa la logica core in 3A, poi FE la consuma** |
| **[NUOVO] Timeline 5 settimane** | **Medio** | **Settimana 5 è buffer per testing. Se garanzie non bloccanti, posticipabili** |
| Documentazione trascurata per fretta | Medio | Daniele blocca la PR senza documentazione |

---

## PUNTI DA VERIFICARE COL TEAM GENAI (prima di iniziare)

| # | Domanda | Impatta stream | Bloccante |
|---|---------|---------------|-----------|
| 1 | I path SACE nei tracciati sono invertiti? | Stream 1A | SI |
| 2 | La fase SACE richiede 2 chiamate GenAI separate o una sola? | Stream 1A | SI |
| 3 | Il Pre-Closing usa ancora `AD_FORM` o è stato aggiornato a `AD_FORM_PRE_CLOSING`? | Pre-closing | SI |
| 4 | Il Booking AD Form usa path unico o per-lender? | Stream 4A | MEDIO |
| 5 | Garanzie: una chiamata per tipo o una sola con tutti i documenti? | Stream 5A | SI |
| 6 | Output CSV `Output_covenant_numerici_variabili.csv`: struttura? Davvero opzionale? | Stream 3A | BASSO |
| 7 | **[NUOVO] Il Format PGA per garanzie: quale struttura esatta ha il CSV di output?** | Stream 5A | MEDIO |
| 8 | **[NUOVO] La classificazione automatica documenti: endpoint API esistente o da sviluppare?** | Stream 11 (futuro) | BASSO |

---

## Storico Aggiornamenti

### Aggiornamento 2026-04-27
- **Documenti aggiornati**: BR v28, Tracciati GenAI v2, Mockup PostClosing v7
- **Task aggiunte**: Stream 6A (Nome Deal BE, Ahmad, 1-2gg), Stream 6B (Nome Deal FE, Georgios, 1-2gg)
- **Task modificate**: Stream 0 (ampliato da 2-3 a 3-4gg per nuovi enum dropdown), Stream 1A/1B (aggiunti Modifica report + dialog conferma), Stream 2A/2B (aggiunti max 15 lender, cestino, lock), Stream 3A (ampliato da 5-6 a 7-8gg per mutua esclusione + 4 viste GenAI complete), Stream 3B (ampliato da 4-5 a 6-7gg per 3 viste con dropdown), Stream 5A (ampliato da 3-4 a 4-5gg per Format PGA), Stream 5B (ampliato da 2-3 a 3-4gg per Format PGA view)
- **Task annullate**: nessuna
- **Timeline**: estesa da 4 a 5 settimane
- **Motivazione**: BR v28 fornisce specifiche complete per il post-closing (campi con dropdown esatti, workflow classificazione, mutua esclusione report/GenAI, Nome Deal/Termination Date)

### Aggiornamento 2026-05-06 — Revisione post-implementazione
- **Documenti ri-analizzati**: BR v28 completo (docs/review/), tutti i mockup, tracciati GenAI v2 + vUpdate, esempi JSON AI
- **Task aggiunte**: S-03B-fix (fix dropdown FE, Carmine, 1gg), S-07A (rimozione dealId da creazione pratica, Ahmad, 0.5gg), S-06B-fix (verifica ownership Nome Deal/Termination Date, Georgios, 0.5gg)
- **Task modificate**: S-03B portata a "Completata con riserva" per discrepanza dropdown
- **Task annullate**: nessuna
- **Motivazione**: Ri-analisi completa post-implementazione, identificate 4 discrepanze tra codice e BR

---

## TASK CORRETTIVE (Aggiornamento 2026-05-06)

### S-03B-fix — Fix Dropdown Obblighi e Covenant Numerici FE

**Branch:** `feature/gap-3b-fix-dropdown`
**Assegnato a:** Carmine
**Dipendenze:** nessuna (fix su codice già completato)
**Stima:** 1 giorno

#### Cosa fare

**1. Allineare `INFORMATIVE_DUTY_TYPE_OPTIONS` al BR/enum BE**

File: `ba-web/src/app/shared/const/post-closing-dropdown.const.ts` linea 64

Sostituire le 34 voci attuali con le 37 voci esatte dall'enum BE `InformativeDutyTypeEnum.java`, che corrispondono al BR v28 sez. 3.1.26:

```typescript
export const INFORMATIVE_DUTY_TYPE_OPTIONS = [
  'Agency fee', 'Altro', 'Bilancio', 'Bilancio annuale', 'Bilancio annuale certificato',
  'Bilancio consolidato', 'Bilancio semestrale', 'Bilancio semestrale consolidato',
  'Budget', 'Business Plan', 'Compliance certificate',
  'Dati annuali', 'Dati annuali iva', 'Dati mensili', 'Dati semestrali', 'Dati trimestrali',
  'Dichiarazione annuale iva',
  'Esg - environmental', 'Esg - compliance certificate', 'Esg - governance', 'Esg - social',
  'Gar. c/c pegnato', 'Gar. ipoteche/perizie', 'Gar. Vincoli assicurativi',
  'Parametri finanziari', 'Perizia', 'Perizie',
  'Relazione semestrale', 'Relazione trimestrale',
  'Rendiconto', 'Rendiconto annuale certificato della gestione del fondo', 'Rendiconto del fondo',
  'Report', 'Report mensile *', 'Report trimestrale',
  'Ricognitivo garanzia', 'Semestrale', 'Vendite autorizzate',
] as const;
```

**2. Allineare `COVENANT_TYPE_NUMERIC_OPTIONS` al BR/enum BE**

Sostituire le ~50 voci generiche attuali con le 53 voci esatte dall'enum BE `CovenantTypeNumericEnum.java`, che corrispondono al BR v28 sez. 3.1.26:

```typescript
export const COVENANT_TYPE_NUMERIC_OPTIONS = [
  'ICR (Risultato Operativo/Oneri Finanziari)', 'PFN/MOL',
  '(PN - Immob. Imm.)/(Attivo + Immob. in leasing o impegni per canoni- Immob. Imm.)',
  'PFN/PN',
  '(P.N. + DEBITI VERSO SOCI PER FINANZIAMENTI INFRUTTIFERI) / (IMMOBILIZZAZIONI NETTE + RIMANENZE)',
  '(UTILE NETTO + RISERVE DISTR.)/SERVIZIO DEL DEBITO',
  'ADSCR', 'ANNUAL CAPEX LIMIT', 'ARPU Blended (Euro/month per customer)',
  'ATTIVO CIRCOLANTE/FATTURATO', 'ATTIVO CIRCOLANTE/PASSIVO A BT',
  'ATTIVO/DEBITI FINANZIARI', 'AUTOFINANZIAMENTO/FATTURATO',
  'CANALIZZAZIONE CANONI/ INCASSI', 'CAR - CAPITAL ADEGUACY RATIO',
  'CASH FLOW / INTERESSI PASSIVI', 'CASH FLOW / TOTAL DEBT', 'CASH FLOW / TOTAL DEBT SERVICE',
  'CET1', 'CLEAN UP CLEAN DOWN', 'COSTO INDUSTRIALE/RICAVI NETTI',
  'CRED.A BT+MAGAZZ/PASSIVITA A BT', 'CRED.A BT/PASSIVITA A BT',
  'DEBITI FINANZIARI NETTI / (P.N. + DEBITI VERSO SOCI PER FINANZIAMENTI INFRUTTIFERI)',
  'DEBITI FINANZIARI/MOL', 'DEBITI FINANZIARI/PN', 'DEBITI FINANZIARI/PN TANGIBILE',
  'DEBT : EQUITY', 'DEBT YIELD', 'DEBT/CAPITALIZATION',
  'DSCR', 'DYR', 'FATTURATO MIN.',
  'FFO / NET FINANCIAL CHARGES (Real Estate)', 'FFO/NET DEBT',
  'FIXED ASSET VALUE / GROSS DEBT', 'GUARANTEE COVERAGE',
  'LOAN TO COST', 'LOAN TO RAB', 'LOAN TO VALUE',
  'MAGAZZINO/FATTURATO', 'MOL/FATTURATO', 'ONERI FINANZIARI/FATTURATO',
  'PFN/CASH FLOW', 'PFN/FATTURATO', 'PFN/RAB',
  'PN / ATTIVO', 'PN TANGIBILE/ATTIVO TANGIBILE',
  'PN+FIN.SOCI/(IMMOBILIZZAZIONE-AZIONI PROPRIE)',
  'PN/(IMMOBILIZZAZIONI-AZIONI PROPRIE)', 'PN/(PN + DEBITO)',
  'PN/DEBITI FINANZIARI', 'TOTAL LONG TERM DEBT/TOTAL ASSETS',
] as const;
```

**3. Verificare che i componenti FE non facciano mapping/trasformazione dei valori dropdown**

Controllare che `CovenantNumericExtractionComponent` e `InformativeDutyExtractionComponent` usino i valori direttamente dalle costanti senza trasformazioni.

---

### S-07A — Rimozione dealId dalla creazione pratica

**Branch:** `feature/gap-7a-remove-dealid-creation`
**Assegnato a:** Ahmad
**Dipendenze:** nessuna
**Stima:** 0.5 giorni

#### Cosa fare

**1. FE — Rimuovere campo dealId dalla form di creazione**

File: `ba-web/src/app/shared/ui/base-element/practice-creation-form/practice-creation-form.component.ts`
- Rimuovere `dealId: new FormControl(null, Validators.required)` dalla FormGroup (linea 49)

File: `ba-web/src/app/shared/ui/base-element/practice-creation-form/practice-creation-form.component.html`
- Rimuovere il blocco `<div class="field col-12 mb-3">` con `<app-input name="dealId" ...>` (linee 12-20)

**2. BE — Rendere dealId opzionale in StartPracticeReqDTO**

File: `ba-back-end/src/main/java/it/deloitte/ba/backend/dto/agency_desk/StartPracticeReqDTO.java`
- Il campo `dealId` è già opzionale (nessuna annotation `@NotBlank`), quindi nessuna modifica BE necessaria.
- `PracticeService.java:388` fa `practice.setDealId(startPracticeReqDTO.getDealId())` — se null, setta null. OK.

**3. Verificare che la tabella pratiche e l'header pratica continuino a mostrare dealId/ID Contratto** (se presente)

Il dealId rimane nel model `Practice.java` e nel DTO, semplicemente non viene più settato alla creazione. Verrà popolato quando l'utente lo inserisce nella fase Post-Closing (come da BR).

**ATTENZIONE**: Verificare che non ci siano validazioni o logiche a valle che assumano che dealId sia sempre presente alla creazione.

---

### S-06B-fix — Verifica ownership Nome Deal / Termination Date

**Branch:** `feature/gap-6b-fix-ownership`
**Assegnato a:** Georgios
**Dipendenze:** nessuna
**Stima:** 0.5 giorni

#### Cosa fare

**1. Verificare nel FE che Nome Deal sia editabile SOLO dall'Utente ISP**

File: `ba-web/src/app/shared/ui/section-element/dossier/post-phase/post-phase.component.ts`
- Il campo "Nome Deal" deve essere visibile a entrambi gli utenti ma editabile solo da ISP
- Il campo "Termination Date" deve essere visibile a entrambi gli utenti ma editabile solo da Deloitte
- Usare il pattern di role-check già presente nel progetto (es. `isConsultant`, `isBank`)

**2. Aggiungere commenti per Nome Deal solo da Deloitte**

Il BR dice: "In caso la data [Termination Date] inserita risulti errata, l'utente ISP avrà la possibilità di lasciare un commento per segnalare l'anomalia."
Verificare che i commenti su Termination Date siano abilitati per ISP.

**3. BE — Verificare che gli endpoint PATCH rispettino i ruoli**

File: `ba-back-end/src/main/java/it/deloitte/ba/backend/controller/agency_desk/PostClosingPracticeDataController.java`
- PATCH deal-name → solo ruolo ISP/BANK
- PATCH termination-date → solo ruolo DELOITTE/CONSULTANT
