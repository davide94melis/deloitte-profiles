# ANALISI GAP BR v28 vs IMPLEMENTAZIONE

**Data originale:** 2026-04-22
**Data aggiornamento:** 2026-05-06
**Scope:** 11 feature (escluso Monitoraggio)
**Repository analizzati:** ba-back-end, ba-web, document-manager, email-manager
**Documentazione di riferimento:** BR v28 (v.27 aggiornato 16/04/2026), Mockup PostClosing v7, Tracciati GenAI v2

---

## 1. DOMANDA GARANZIA SACE - Pipeline GenAI CdF + Termsheet (BR v28 sez. 3.1.2-3.1.8)

**Cosa richiede il BR:** La piattaforma riceve 2 file dalla GenAI (estrazione da Contratto di Finanziamento e da Termsheet), li fonde in un unico file dando **priorità ai valori del CdF** (se vuoto o "Non Indicato", usa il Termsheet). La vista "Dettaglio estrazione dati GenAI | Censimento Domanda Garanzia SACE" mostra 19 campi fissi con colonne Documento di riferimento, Articolo, Campo estratto, Valore, Azioni.

Dopo validazione GenAI, il pulsante "Conferma dati GenAI e chiudi" apre una dialog: "Vuoi confermare la correttezza dei dati estratti e avviare la produzione del Report?" con Conferma/Annulla.

A seguito del caricamento del report, si abilita il pulsante "Modifica report caricati" per consentire all'utente Deloitte di modificare il report. A destra dello slot documentale, una freccia a tendina apre un box con anteprima tabellare del report e pulsante download.

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Workflow SACE (stati PSM) | **Implementato** | `SACE_DOC_INCOMPLETA → SACE_DOC_COMPLETA → SACE_VERIFICA_IN_CORSO → SACE_FASE_COMPLETATA` |
| Caricamento CdF-Draft + Termsheet | **Implementato** | `sace-phase.component` + `sace-document-section.service` |
| Conferma/Rifiuto documenti | **Implementato** | `sace-consultant-document-section.service` |
| Chiamata GenAI per SACE | **NON implementato** | `AIMetadataProcessService` ha solo `engagementAiPreClosingProcess` (use case `AGENCYDESK_ADFORM`). Nessun use case SACE |
| Logica fusione 2 file GenAI | **NON implementato** | Non esiste codice di merge con regola di priorità CdF > Termsheet |
| 19 campi fissi estrazione SACE | **NON implementato** | `GenAiFormKeyEnum` ha solo campi AD Form |
| Vista GenAI con tabella SACE | **NON implementato** | `genai-extraction.component` gestisce solo AD Form |
| Generazione automatica report SACE | **NON implementato** | `BOOKING_SACE_GUARANTEE_REPORT` esiste nel BE enum ma il report è solo caricato manualmente |
| Upload manuale report SACE | **Implementato** | `SaceReportUploadDialogComponent` funziona |
| Pulsante "Modifica report caricati" | **NON implementato** | Nessuna logica per riapertura modifica post-upload |
| Anteprima report con dropdown | **NON implementato** | Nessun componente preview tabellare |
| Dialog "Conferma dati GenAI e chiudi" | **NON implementato** | Nessuna dialog di conferma GenAI |

**GAP CRITICO**: L'intera pipeline GenAI per la fase SACE non è implementata. Attualmente il report viene solo caricato manualmente.

---

## 2. BOOKING - Pop-up Banca Lender + Interdipendenza AD Form (BR v28 sez. 3.1.14-3.1.15)

**Cosa richiede il BR:** Pulsante "+Aggiungi nuova Banca Lender" che apre un pop-up con 3 campi (Nome Banca, Indirizzo Banca, NDG Banca). Alla conferma, la piattaforma crea automaticamente 2 slot: (1) NDG nella sezione "Anagrafica controparti" già compilato, (2) slot AD Form nella sezione "AD Form Aggiuntivi" per upload documento. Massimo 15 nuovi lender. Per gli AD Form aggiuntivi del Booking va fatta una chiamata GenAI separata (document type `AD_FORM_BOOKING`).

Funzionalità "Cestino" per eliminare slot creati (disabilitata dopo "Conferma Documentazione"). Il pulsante "Conferma Documentazione" blocca la sezione: l'utente ISP non può più sovrascrivere/sostituire documenti. Eventuali modifiche successive solo forzando da pannello Admin.

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Pulsante "Aggiungi Lender" | **Parzialmente implementato** | `addLender()` aggiunge solo un campo NDG, senza pop-up con 3 campi |
| Struttura dati Lender | **NON implementato** | `Practice.bookingLenderNdgs` è `List<String>` (solo NDG). Mancano Nome e Indirizzo Banca |
| Pop-up con 3 campi | **NON implementato** | Nessuna modale con Nome Banca, Indirizzo Banca, NDG Banca |
| Limite max 15 lender | **NON implementato** | Nessuna validazione limite |
| Auto-creazione slot AD Form | **NON implementato** | Nessuna logica automatica |
| Sezione "AD Form Aggiuntivi" | **NON implementato** | Non esiste questa sezione nel FE |
| Chiamata GenAI per AD Form Booking | **Parzialmente preparato** | Il BE ha `resolveSourcePhaseId` che per `BOOKING_FEE_LETTERS_REPORT` punta al PRE_CLOSING |
| NDG auto-generati da Pre-closing | **Implementato** | `loadLenderDefinitions()` legge dati GenAI pre-closing |
| Funzionalità Cestino | **NON implementato** | Nessuna cancellazione slot |
| Pulsante "Conferma Documentazione" | **NON implementato** | Nessun lock della sezione post-conferma |

**GAP SIGNIFICATIVO**: Struttura dati incompleta (`List<String>` invece di entity strutturata), pop-up mancante, sezione AD Form Aggiuntivi assente, gestione slot cestino e lock post-conferma non implementati.

---

## 3. BOOKING - Indipendenza dei report (BR v28 sez. 3.1.19)

**Cosa richiede il BR:** Le informazioni estratte dai nuovi AD Form inseriti vanno integrate per creare il report Censimento Mailing list. Quest'integrazione NON modifica gli output generati nelle precedenti fasi (modulo di garanzia SACE, censimento dati lender).

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Report Pre-closing non modificati dal Booking | **Implementato** | Liste report separate per fase |
| Report Booking Operazione | **Implementato** | `BOOKING_SACE_GUARANTEE_REPORT` caricato manualmente |
| Report Censimento Mailing List | **Parzialmente implementato** | `BOOKING_FEE_LETTERS_REPORT` generato da dati pre-closing |
| Integrazione dati AD Form Booking nel Mailing List | **NON implementato** | Sezione AD Form Aggiuntivi non esiste (dipende dal gap #2) |

**GAP BASSO**: L'indipendenza dei report precedenti è rispettata. Il gap è che il Censimento Mailing List non può integrare dati da AD Form del Booking perché quella sezione non esiste ancora.

---

## 4. POST-CLOSING - Covenant Numerici GenAI View (BR v28 sez. 3.1.26, pagg. 1482-1601)

**Cosa richiede il BR:** Pagina "Dettaglio estrazione dati GenAI | Covenant numerici" con 14 colonne + Azioni. Pulsante "Aggiungi dato" per righe manuali. Pannello sinistro con viewer documento navigabile. Pulsante "Modifica" e "Cestino" per ogni riga.

**Colonne complete con specifiche campi:**

| # | Colonna | Tipo | Dropdown / Valori | Obbligatorio |
|---|---------|------|------------------|-------------|
| 1 | Booking Entity | Dropdown | ISP MIL, ISP FF, ISP HK, ISP IRL, ISP LND, ISP MD, ISP NY, ISP SN, ISP TK, ISP FRA | SI |
| 2 | Nome cliente | Testo alfanumerico | - | SI |
| 3 | Tipologia Covenant | Dropdown | ~50 voci (ICR, PFN/MOL, DSCR, ADSCR, LOAN TO VALUE, etc.) | SI |
| 4 | Articolo | Testo alfanumerico | - | SI |
| 5 | Documento Monitoraggio Covenant | Dropdown | Bilancio consolidato, Bilancio individuale, Situazione Contabile, Visura Camerale, Estratto Conto, Non Previsto, Altro | SI |
| 6 | Soggetto diverso dal borrower (nome) | Testo alfanumerico | - | NO |
| 7 | Periodicità di Monitoraggio | Dropdown | Annuale, Semestrale, Trimestrale, Mensile, Altro | SI |
| 8 | Data Riferimento Documento Prima Rilevazione | Data | - | SI |
| 9 | Data di scadenza | Data | - | SI (MANUALE, non GenAI) |
| 10 | Data/e di Monitoraggio | Data | - | SI |
| 11 | Valore Covenant Fisso (segno) | Dropdown | >, >=, =, <, <= | SI |
| 12 | Valore Covenant Fisso (valore effettivo) | Numerico | - | SI |
| 13 | Covenant variabile (Si/No) | Dropdown | Si, No | SI |
| 14 | Note | Testo alfanumerico | - | NO |

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Report code CENS_NUMERIC_COVENANTS | **Definito** | Presente in `GeneratedReportCodeEnum` |
| Vista GenAI Covenant Numerici (FE) | **NON implementato** | `post-phase-mockup.component.ts` è un mockup statico |
| 14 colonne con dropdown | **NON implementato** | Nessun data model per campi covenant |
| Colonna "Data di scadenza" (BE) | **NON implementato** | Nessun campo `expirationDate` nel dominio |
| Colonna "Data monitoraggio" (BE) | **NON implementato** | Nessun campo monitoraggio covenant |
| Pulsante "Aggiungi dato" | **NON implementato** | Nessuna logica aggiunta righe manuali |
| Viewer documento (pannello sinistro) | **NON implementato** | Nessun componente document viewer |
| "Modifica" / "Cestino" per riga | **NON implementato** | Nessuna logica edit/delete per record GenAI |
| Upload/generazione report | **Implementato** | Upload manuale funziona per tutti i codici post-closing |

**GAP CRITICO**: Vista GenAI post-closing è solo mockup. I 14 campi con 6 dropdown menu non sono modellati in nessun layer.

---

## 5. OUTPUT GENAI - REPORT PIATTAFORMA (Tracciati GenAI v2)

**Cosa richiede il BR + Tracciati v2:** Mappatura completa degli output GenAI e report per fase:

| Fase | Chiamata GenAI | Report Piattaforma | Stato | Wave 3 |
|------|----------------|-------------------|-------|--------|
| Dom. Garanzia SACE | Termsheet → Output TS; CdF Bozza → Output CdF | Report Modulo di Garanzia | **NON impl.** (solo upload manuale) | Y |
| Pre-Closing | AD Form → Output AD Form | Report AD Form [Dati Lender] | **Implementato** (3 report) | Y |
| Booking | AD Form (aggiornamento) → Output AD Form | Report AD Form [Mailing List] | **Parziale** (dati solo pre-closing) | Y |
| Booking | - | Fee Letters/Agency Fee Letters | **Implementato** come doc upload | Y |
| ~~Booking~~ | ~~Comunicazioni TAEG/TEG~~ | ~~Report TAEG/TEG~~ | ~~NON impl.~~ | **N** |
| Post-Closing | CdF Signed → 5 CSV (obblighi+covenant) | Report Covenant Num/Desc + Obblighi Inf. | **Parziale** (codici report esistono, upload manuale funziona) | Y |
| Post-Closing | Garanzia → 2 CSV (obblighi) | Report Obblighi Inf. + Format PGA | **Parziale** (CENS_ASSURANCES esiste, upload manuale funziona) | Y |

**Aggiornamento v28:** Il tracciato GenAI v2 chiarisce che CDF_POST_CLOSING produce 5 CSV (incluso `Output_covenant_numerici_variabili.csv` opzionale). Ogni tipo garanzia produce 2 CSV (obblighi periodici + obblighi evento). Comunicazioni TAEG/TEG confermate fuori scope Wave 3.

**GAP MODERATO**: I codici report sono definiti ma la generazione automatica è limitata al pre-closing.

---

## 6. POST-CLOSING - Obblighi Informativi GenAI View (BR v28 sez. 3.1.26, pagg. 1358-1429)

**Cosa richiede il BR:** Pagina "Dettaglio estrazione dati GenAI | Obblighi informativi" con 7 colonne + Azioni. Stessa struttura UI dei Covenant: pannello sinistro document viewer, pulsante "Aggiungi dato", "Modifica"/"Cestino" per riga. Applicabile sia al CdF (accordion Contratto di Finanziamento) sia alle Garanzie (accordion Garanzia) con diversa origine dati.

**Colonne:**

| # | Colonna | Tipo | Dropdown / Valori | Obbligatorio |
|---|---------|------|------------------|-------------|
| 1 | Tipologia Obbligo | Dropdown | ~35 voci (Bilancio consolidato, Bilancio individuale, Bilancio separato, Situazione contabile consolidata, Relazione trimestrale/semestrale, Budget, Business plan, Compliance certificate, Estratto conto, Polizza assicurativa, Visura camerale, Visura ipotecaria, DURC, Comunicazione acquisizioni, Comunicazione modifiche organiche, etc.) | SI |
| 2 | Data di riferimento | Data | - | SI |
| 3 | Data di scadenza | Data | - | SI |
| 4 | Periodicità | Dropdown | Annuale, Semestrale, Trimestrale, Mensile, Altro | SI |
| 5 | Articolo | Testo alfanumerico | - | SI |
| 6 | Nome società | Testo alfanumerico | - | SI |
| 7 | Descrizione Obbligo | Testo alfanumerico | - | SI |

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Report code CENS_INFORMATIVE_DUTIES | **Definito** | Presente in `GeneratedReportCodeEnum` |
| Data model Obblighi Informativi | **NON implementato** | Nessuna entity/DTO con campi specifici |
| Vista GenAI Obblighi Informativi | **NON implementato** | Nessun componente FE |
| Dropdown Tipologia Obbligo (~35 voci) | **NON implementato** | Nessun enum con le voci |
| Dropdown Periodicità | **NON implementato** | Nessun enum |
| "Aggiungi dato" / "Modifica" / "Cestino" | **NON implementato** | Nessuna logica CRUD record |

**GAP CRITICO**: Feature completamente da sviluppare. Impatta sia CdF accordion sia Garanzie accordion nel post-closing.

---

## 7. POST-CLOSING - Covenant Descrittivi GenAI View (BR v28 sez. 3.1.26, pagg. 1430-1479)

**Cosa richiede il BR:** Pagina "Dettaglio estrazione dati GenAI | Covenant descrittivi" con 6 colonne + Azioni. Stessa struttura UI.

**Colonne:**

| # | Colonna | Tipo | Dropdown / Valori | Obbligatorio |
|---|---------|------|------------------|-------------|
| 1 | Booking Entity | Dropdown | ISP MIL, ISP FF, ISP HK, ISP IRL, ISP LND, ISP MD, ISP NY, ISP SN, ISP TK, ISP FRA | SI |
| 2 | Nome cliente | Testo alfanumerico | - | SI |
| 3 | Tipologia Covenant | Dropdown | CHANGE OF CONTROL, LIMITAZIONE AD OPERAZIONI STRAORDINARIE, LIMITAZIONE ALLA CESSIONE DI ASSETS, LIMITAZIONE ALLA DISTRIBUZIONE DIVIDENDI, NEGATIVE PLEDGE, OWNERSHIP CLAUSE, PARI PASSU, Limite all'assunzione di nuovo indebitamento, CASH SWEEP, CASH TRAP | SI |
| 4 | Articolo | Testo alfanumerico | - | SI |
| 5 | Documento Monitoraggio Covenant | Dropdown | Bilancio individuale, Bilancio consolidato, Centrale dei rischi, Visura camerale, Situazione contabile, Non previsto | SI |
| 6 | Soggetto diverso dal borrower (nome) | Testo alfanumerico | - | NO |

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Report code CENS_DESCRIPTIVE_COVENANTS | **Definito** | Presente in `GeneratedReportCodeEnum` |
| Data model Covenant Descrittivi | **NON implementato** | Nessuna entity con campi specifici |
| Vista GenAI Covenant Descrittivi | **NON implementato** | Nessun componente FE |
| Dropdown Tipologia Covenant (10 voci) | **NON implementato** | Nessun enum |
| Dropdown Documento Monitoraggio (6 voci) | **NON implementato** | Nessun enum |

**GAP CRITICO**: Feature completamente da sviluppare.

---

## 8. POST-CLOSING - Format PGA GenAI View per Garanzie (BR v28 sez. 3.1.26, pagg. 1603-1618)

**Cosa richiede il BR:** Pagina "Dettaglio estrazione dati GenAI | [Nome tipologia garanzia]" con 2 colonne + Azioni. Per le sole garanzie Ipoteca RE, Ipoteca non RE e Ipoteca Navi-Aerei il file Excel comprende 2 sheet.

**Colonne:**

| # | Colonna | Tipo | Obbligatorio |
|---|---------|------|-------------|
| 1 | Campo estratto | Testo alfanumerico | SI |
| 2 | Valore campo | Testo | SI |

**Report per tipo accordion:**
- Accordion CdF: 3 output (Obblighi informativi, Covenant numerici, Covenant descrittivi)
- Accordion Garanzie: 2 output (Obblighi informativi, Format PGA)
- Accordion Garanzia SACE: 1 output (Obblighi informativi solo)

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Report code CENS_ASSURANCES | **Definito** | Presente in `GeneratedReportCodeEnum` |
| Data model Format PGA | **NON implementato** | Nessun `formatPga` nel dominio |
| Vista GenAI Format PGA | **NON implementato** | Nessun componente FE |
| Logica multi-sheet per Ipoteca | **NON implementato** | Nessuna logica di generazione 2-sheet |

**GAP CRITICO**: Feature completamente da sviluppare. La struttura è semplice (2 colonne) ma la variabilità per tipo garanzia e la logica multi-sheet aggiungono complessità.

---

## 9. POST-CLOSING - Nome Deal & Termination Date (BR v28 sez. 3.1.21, pagg. 1133-1137)

**Cosa richiede il BR:** Due campi nella vista post-closing:
- **Nome Deal**: campo testo, ISP inserisce il valore, Deloitte valida tramite sistema commenti. Stato: "Da caricare" → "Dato caricato".
- **Termination Date**: campo data, stesso workflow di Nome Deal.

Entrambi visibili nella vista principale post-closing sopra l'accordion CdF.

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Campo Nome Deal nel BE | **NON implementato** | `Practice` ha `dealId` ma non `nomeDeal`/`dealName`. `CreditContactsSheetBuilder` usa "Nome Deal" come header colonna ma non esiste campo dati |
| Campo Termination Date nel BE | **NON implementato** | Nessun campo `terminationDate` in Practice |
| Endpoint CRUD per Nome Deal | **NON implementato** | `PostClosingPracticeDataController` ha NDG e ContractId ma non Nome Deal |
| Endpoint CRUD per Termination Date | **NON implementato** | Non esiste |
| Commenti su Nome Deal | **Parzialmente predisposto** | `PostClosingFieldCommentService` esiste con `PostClosingCommentFieldEnum` ma non include NOME_DEAL e TERMINATION_DATE |
| UI Nome Deal / Termination Date | **NON implementato** | Nessun componente FE |

**GAP SIGNIFICATIVO**: 2 campi da aggiungere a livello di entity, DTO, controller, e UI. La struttura dei commenti è già predisposta ma va estesa.

---

## 10. POST-CLOSING - Mutua Esclusione "Carica Report" / "Avvia Estrazione GenAI" (BR v28 sez. 3.1.25-3.1.26, pagg. 1334-1344)

**Cosa richiede il BR:** Per ogni accordion post-closing (CdF e Garanzie), quando la sezione raggiunge lo stato "Sezione Documentale validata", si abilitano 2 pulsanti:
- **"Carica Report"**: upload manuale dei report. Premendo, si inibisce "Avvia Estrazione GenAI".
- **"Avvia Estrazione GenAI"**: avvia pipeline GenAI. Premendo, si inibisce "Carica Report".

Il pulsante "Carica Report" rimane abilitato fino al completamento di tutti i report obbligatori, poi viene sostituito da **"Modifica Report Caricati"** che resta attivo fino alla chiusura del post-closing.

Per il CdF: 3 slot documentali (Obblighi informativi, Covenant Numerici, Covenant Descrittivi).
Per le Garanzie: 1 slot (Obblighi Informativi). Per Garanzia SACE: 1 slot (Obblighi Informativi solo).

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| SectionStateEnum (accordion states) | **Implementato** | SECTION_INCOMPLETE → SECTION_COMPLETE → AI_EXTRACTION_IN_PROGRESS → AI_TO_VERIFY_EXTRACTION → AI_TO_VALIDATE_REPORT → REPORT_GENERATED |
| Logica mutua esclusione BE | **NON implementato** | Nessuna logica che inibisca un path quando l'altro è attivato |
| Pulsante "Carica Report" | **Parzialmente implementato** | `GeneratedReportFacade.uploadManualReport()` funziona ma senza mutua esclusione |
| Pulsante "Avvia Estrazione GenAI" | **NON implementato** | Nessun endpoint per avviare GenAI post-closing da accordion |
| "Modifica Report Caricati" | **NON implementato** | Nessuna logica sostituzione pulsante post-completamento |
| Slot documentali per CdF (3) | **NON implementato** | Nessuna distinzione slot per tipo report |
| Slot documentali per Garanzie (1-2) | **NON implementato** | Nessuna logica differenziata per tipo garanzia |

**GAP CRITICO**: La macchina a stati dell'accordion esiste ma manca l'intera logica di scelta percorso (manuale vs GenAI), la mutua esclusione, e il pulsante "Modifica Report Caricati".

---

## 11. POST-CLOSING - Workflow Classificazione Documenti (BR v28 sez. 3.1.23-3.1.24, pagg. 1130-1320)

**Cosa richiede il BR:** Workflow di classificazione documenti nella fase post-closing:

1. **Upload documenti**: ISP carica N documenti che finiscono nella "Sezione documenti" (Waiting List)
2. **Classificazione manuale**: per ogni documento, Deloitte clicca "Classifica documento" → selezione tipo: Contratto di finanziamento / Garanzia / Perfezionamento. Per Garanzia: selezione tipo e sotto-tipo (16+6 combinazioni)
3. **Classificazione automatica GenAI**: pulsante "Classificazione automatica" avvia la GenAI per suggerire la classificazione. Per ogni documento compare un box con la classificazione suggerita e pulsante di conferma
4. **"Concludi caricamento documenti"**: chiude la fase di upload
5. **"Rinomina Documento"**: solo per Deloitte, permette di rinominare il file caricato
6. **Gestione Garanzie**: classificare come garanzia crea un accordion numerato (Garanzia 1, 2...). "Modifica numero garanzie previste" aggiorna il contatore. "Cambia tipologia garanzia" permette di riclassificare. Per Ipoteca: 4 slot documentali (Atto d'ipoteca, Polizza, Nota iscrizione, CdF)
7. **Perfezionamenti**: abbinamento perfezionamento-garanzia tramite menù a tendina

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Entity PostClosingWaitingDocument | **Implementato** | Con stati WAITING, CLASSIFIED, DELETED. Controller, Service, Repository, DTO tutti presenti |
| Upload documenti in Waiting List | **Implementato** | Service e controller funzionanti |
| Commenti su documenti | **Implementato** | `PostClosingWaitingDocumentComment` entity e service |
| Classificazione manuale CdF/Garanzia/Perfezionamento | **Parzialmente implementato** | L'infrastruttura c'è ma va verificata la completezza del workflow di selezione tipo |
| Classificazione automatica GenAI | **NON implementato** | Nessun pulsante "Classificazione automatica", nessuna logica GenAI per document classification |
| Gestione Accordion Garanzie | **Implementato** | `Assurance` entity con `AssuranceType`/`AssuranceSubType`, `SectionStateEnum` con stati accordion, `AssurancePerfection` entity |
| 16 tipi garanzia allineati | **Parzialmente implementato** | `AssuranceTypeEnum` ha 6 valori (GUARANTEES, IPOTECA, PEGNO_CC, PEGNO_MERCI_CREDITI, PEGNO_QUOTE_AZIONI, PRIVILEGIO_FIXED_CHARGE) vs 16 nel BR. I sotto-tipi sono gestiti via `AssuranceSubType` entity |
| "Rinomina Documento" | **NON verificato** | Da verificare nel FE |
| "Concludi caricamento documenti" | **NON verificato** | Da verificare nel FE |
| Modifica numero garanzie previste | **Implementato** | `UpdatePostClosingExpectedAssurancesRequestDTO` e endpoint |
| 4 slot per Ipoteca | **NON verificato** | Da verificare la logica di creazione slot differenziati per tipo |

**GAP SIGNIFICATIVO**: L'infrastruttura base esiste (waiting list, classificazione, garanzie). I gap principali sono: classificazione automatica GenAI, allineamento tipi garanzia (6 vs 16 nel BR), e completamento funzionalità FE.

---

## RIEPILOGO PRIORITA

| # | Feature | Severità GAP | Impatto | Origine |
|---|---------|-------------|---------|---------|
| 1 | SACE - Pipeline GenAI CdF+Termsheet | **CRITICO** | Intera pipeline GenAI SACE da sviluppare | v27 |
| 4 | Post-closing - Covenant Numerici (14 colonne) | **CRITICO** | Vista GenAI con 14 campi, 6 dropdown, non modellati | v27 (espanso v28) |
| 6 | Post-closing - Obblighi Informativi (7 colonne) | **CRITICO** | Vista GenAI con 7 campi, ~35 dropdown tipologia | NUOVO v28 |
| 7 | Post-closing - Covenant Descrittivi (6 colonne) | **CRITICO** | Vista GenAI con 6 campi, dropdown specifici | NUOVO v28 |
| 8 | Post-closing - Format PGA Garanzie | **CRITICO** | Vista GenAI per garanzie, logica multi-sheet | NUOVO v28 |
| 10 | Post-closing - Mutua Esclusione Report/GenAI | **CRITICO** | Logica di scelta percorso manuale vs automatico | NUOVO v28 |
| 2 | Booking - Pop-up Banca Lender + AD Form | **SIGNIFICATIVO** | Struttura dati incompleta, pop-up e AD Form aggiuntivi | v27 (espanso v28) |
| 9 | Post-closing - Nome Deal & Termination Date | **SIGNIFICATIVO** | 2 campi mancanti con workflow commenti | NUOVO v28 |
| 11 | Post-closing - Classificazione Documenti | **SIGNIFICATIVO** | Infrastruttura presente, manca GenAI classification | NUOVO v28 |
| 5 | Mappatura Output GenAI / Report | **MODERATO** | Codici report definiti, generazione auto limitata | v27 (aggiornato v28) |
| 3 | Booking - Indipendenza report | **BASSO** | Sostanzialmente rispettata | v27 |

---

## Storico Aggiornamenti

### Aggiornamento 2026-04-27
- **Documenti aggiornati**: BR v28 (v.27 aggiornato 16/04/2026), Tracciati GenAI v2, Mockup PostClosing v7
- **Requisiti nuovi**: 6 (features 6, 7, 8, 9, 10, 11)
- **Requisiti modificati**: 4 (features 1, 2, 4, 5 — dettagli aggiuntivi e specifiche campi complete)
- **Requisiti invariati**: 1 (feature 3)
- **Requisiti rimossi**: 0
- **Motivazione**: Il BR v28 fornisce specifiche molto più dettagliate per il post-closing (campi completi con dropdown, workflow classificazione, mutua esclusione report/GenAI) e aggiorna i tracciati GenAI con la struttura CSV v2

### Aggiornamento 2026-05-06 — Revisione post-implementazione
- **Documenti ri-analizzati**: BR v28 completo (docs/review/), Mockup Booking v7, Mockup PostClosing v7, Mockup SACE v2, Tracciati GenAI v2 + vUpdate, Esempi JSON interazione AI Wave 3
- **Requisiti nuovi**: 1 (feature 12 — rimozione dealId da creazione pratica)
- **Requisiti modificati**: 0
- **Requisiti rimossi**: 0
- **Discrepanze rilevate nell'implementazione**: 4
- **Motivazione**: Ri-analisi completa della documentazione BR dopo completamento di tutte le 13 task del piano. Identificate discrepanze tra codice implementato e requisiti BR: (1) campo dealId obbligatorio nella creazione pratica non previsto dal BR, (2) valori dropdown Tipologia Obbligo FE non allineati al BR/BE, (3) valori dropdown Tipologia Covenant Numerici FE non allineati al BR/BE, (4) ownership Nome Deal (ISP) vs Termination Date (Deloitte) da verificare nel FE

---

## 12. CREAZIONE PRATICA - Campo dealId non previsto dal BR (BR v28 sez. 3.1.1)

**Cosa richiede il BR:** Il popup "Nuova Pratica" prevede SOLO 2 campi:
1. **Tipologia Pratica** (Banca Agente / Banca Agente e SACE Agent) — dropdown
2. **Checkbox** "La pratica contiene informazioni privilegiate"

Nessun altro campo è previsto nella creazione pratica. Il BR specifica che "ID Contratto" è un'informazione "inserita da Utente ISP in fase Censimento Post-Closing" (colonna tabella pratiche).

**Stato implementativo:**

| Elemento | Stato | Dettaglio |
|----------|-------|-----------|
| Campo dealId nella form FE | **DISCREPANZA** | `practice-creation-form.component.ts:49` — `dealId: new FormControl(null, Validators.required)`. Il campo è OBBLIGATORIO nella creazione |
| Campo dealId nel DTO BE | **DISCREPANZA** | `StartPracticeReqDTO.java` ha `dealId`. `PracticeService.java:388` lo setta su Practice |
| Campo Tipologia Pratica | **Implementato** | `mandateType: new FormControl(null, Validators.required)` — presente e obbligatorio |
| Checkbox informazioni privilegiate | **Implementato** | `hasPrivilegedInformation: new FormControl(false)` — presente |

**GAP — DISCREPANZA**: Il campo `dealId` nella creazione pratica è pre-esistente (non introdotto dal piano gap) ma contradice il BR. Il BR prevede che l'ID Contratto sia inserito solo nella fase Post-Closing, non alla creazione.

---

## DISCREPANZE IMPLEMENTATIVE POST-COMPLETAMENTO

### D1. Dropdown "Tipologia Obbligo" FE non allineato al BR (impatta S-03B)

**BR v28 sez. 3.1.26**: 38 voci specifiche per il dropdown (Agency fee, Altro, Bilancio, Bilancio annuale, Bilancio annuale certificato, Bilancio consolidato, Bilancio semestrale, Bilancio semestrale consolidato, Budget, Business Plan, Compliance certificate, Dati annuali, Dati annuali iva, Dati mensili, Dati semestrali, Dati trimestrali, Dichiarazione annuale iva, Esg - environmental, Esg - compliance certificate, Esg - governance, Esg - social, Gar. c/c pegnato, Gar. ipoteche/perizie, Gar. Vincoli assicurativi, Parametri finanziari, Perizia, Perizie, Relazione semestrale, Relazione trimestrale, Rendiconto, Rendiconto annuale certificato della gestione del fondo, Rendiconto del fondo, Report, Report mensile *, Report trimestrale, Ricognitivo garanzia, Semestrale, Vendite autorizzate)

**BE enum** `InformativeDutyTypeEnum.java`: 37 voci — **ALLINEATO al BR**

**FE** `post-closing-dropdown.const.ts:64` `INFORMATIVE_DUTY_TYPE_OPTIONS`: 34 voci DIVERSE (Bilancio consolidato, Bilancio individuale, Bilancio separato, Situazione contabile consolidata, ...) — **NON ALLINEATO**

**Impatto**: Il componente `InformativeDutyExtractionComponent` mostra dropdown con valori errati rispetto al BR.

### D2. Dropdown "Tipologia Covenant Numerici" FE non allineato al BR (impatta S-03B)

**BR v28 sez. 3.1.26**: ~55 voci specifiche con formule finanziarie esatte (ICR (Risultato Operativo/Oneri Finanziari), PFN/MOL, (PN - Immob. Imm.)/(Attivo + Immob. in leasing o impegni per canoni- Immob. Imm.), PFN/PN, ...)

**BE enum** `CovenantTypeNumericEnum.java`: 53 voci — **ALLINEATO al BR**

**FE** `post-closing-dropdown.const.ts` `COVENANT_TYPE_NUMERIC_OPTIONS`: ~50 voci GENERICHE (ICR, PFN/MOL, DSCR, ADSCR, LOAN TO VALUE, PFN/EBITDA, PFN/EQUITY, GEARING, ...) — **NON ALLINEATO** (nomi semplificati, mancano formule complete, presenti voci non nel BR come PFN/EBITDA)

**Impatto**: Il componente `CovenantNumericExtractionComponent` mostra dropdown con valori errati rispetto al BR.

### D3. Nome Deal / Termination Date ownership (impatta S-06B)

**BR v28 sez. 3.1.23**: "Nome Deal" → inserito da Utente **ISP**. "Termination Date" → inserito da Utente **Deloitte**.
**BR v28 sez. 3.1.29**: Per transizione a "Documentazione completa": Nome Deal "ownership del caricamento utente ISP", Termination Date "ownership del caricamento utente Deloitte".

**Codice FE** `post-phase.component.ts`: Entrambi i campi sono nella stessa sezione `dealSections` senza distinzione di ruolo visibile. Da verificare se il componente nasconde/disabilita correttamente i campi in base al ruolo utente.
