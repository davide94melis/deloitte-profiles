Business requirements Monitoraggio

Contents

[REQUISITI UTENTE - AGENCY DESK MONITORAGGIO 2](#_Toc227849747)

[Piattaforma Banca Agente - Gestione Covenant 2](#_Toc227849748)

[1. CREAZIONE PRATICA MONITORAGGIO 2](#_Toc227849749)

[2. TABELLA PRATICHE MONITORAGGIO 6](#_Toc227849750)

[2.1 Descrizione Generale e Accesso 6](#_Toc227849751)

[2.2 Sezione Filtri e Ricerca Avanzata 6](#_Toc227849752)

[2.3 Tabella Pratiche Monitoraggio - Vista Consolidata 7](#_Toc227849753)

[3. FLUSSO ISP (UTENTE BANCA) 8](#_Toc227849754)

[3.1 Descrizione Generale del Flusso ISP 8](#_Toc227849755)

[3.2 Dettaglio Pratica - Sezione Covenant (ISP) 9](#_Toc227849756)

[3.3 Dettaglio Covenant Singolo (ISP) 13](#_Toc227849757)

[3.4 Richiesta di Eccezione al Monitoraggio (ISP) 14](#_Toc227849758)

[3.5 Sezione Spread (ISP) 15](#_Toc227849759)

[3.6 Sezione Scadenziere (ISP) 16](#_Toc227849760)

[4. FLUSSO DELOITTE (UTENTE DELOITTE) 17](#_Toc227849761)

[4.1 Descrizione Generale del Flusso Deloitte 17](#_Toc227849762)

[4.2 Dettaglio Pratica - Sezione Covenant (Deloitte) 17](#_Toc227849763)

[5. SCADENZIERE UNIFICATO 23](#_Toc227849764)

[5.0 Descrizione Generale e Obiettivi 23](#_Toc227849765)

[5.1 Pagina Scadenziere - Vista Consolidata 23](#_Toc227849766)

[5.2 Tabella Scadenziere 24](#_Toc227849767)

[6. AGGIORNAMENTO MAILING LIST 25](#_Toc227849768)

[6.1 Descrizione Generale e Obiettivi 25](#_Toc227849769)

[6.2 Pagina Ricerca Pratica - Selezione della Pratica 25](#_Toc227849770)

[6.3 Pagina Aggiornamento Mailing List - Gestione Contatti 27](#_Toc227849771)

[7. SEZIONE SPREAD - GESTIONE SPREAD FINANZIARI 28](#_Toc227849772)

[7.1 Descrizione Generale e Obiettivi 28](#_Toc227849773)

[7.2 Visibilità della Sezione Spread 28](#_Toc227849774)

[7.3 Creazione della Tabella Spread (Deloitte) 28](#_Toc227849775)

[8. DASHBOARD MONITORAGGIO (ISP) 31](#_Toc227849776)

[8.1 Descrizione Generale e Obiettivi 31](#_Toc227849777)

[8.2 Sezione Monitoraggio - Grafico a Ciambella (Donut Chart) 32](#_Toc227849778)

[8.3 Sezione COVNO - Informazioni di Sincronizzazione 32](#_Toc227849779)

[8.4 Tabella Monitoraggi Scaduti 33](#_Toc227849780)

[8.4.1 Pagina Monitoraggi Scaduti - Descrizione Generale e Accesso 34](#_Toc227849781)

[8.4.2 Pagina Monitoraggi Scaduti - Sezione Filtri e Ricerca Avanzata 35](#_Toc227849782)

[8.4.3 Pagina Monitoraggi Scaduti - Tabella Monitoraggi Scaduti Completa 36](#_Toc227849783)

[8.5 Tabella Monitoraggi in Scadenza 37](#_Toc227849784)

[8.5.1 Pagina Monitoraggi in Scadenza - Descrizione Generale e Accesso 37](#_Toc227849785)

[8.5.2 Pagina Monitoraggi in Scadenza - Sezione Filtri e Ricerca Avanzata 38](#_Toc227849786)

[8.5.3 Pagina Monitoraggi in Scadenza - Tabella Monitoraggi in Scadenza Completa 39](#_Toc227849787)

[8.6 Tabella Pratiche Monitoraggio Recenti 40](#_Toc227849788)

# REQUISITI UTENTE - AGENCY DESK MONITORAGGIO

## Piattaforma Banca Agente - Gestione Covenant

**Data ultimo aggiornamento :** 22/04/2026

## 1. CREAZIONE PRATICA MONITORAGGIO

In seguito alla conclusione della fase Censimento Post-closing per una specifica pratica, la **piattaforma apre automaticamente una nuova pratica di Monitoraggio** (collegata alla pratica di censimento attraverso il medesimo tracking ID).

La nuova pratica di Monitoraggio confluirà nella Tabella Pratiche raggiungibile cliccando sul pulsante “Pratiche Monitoraggio” dal menu laterale e sarà alimentata inizialmente dalle informazioni derivanti dai Report di censimento dei Covenant prodotti nella fase Post-closing.

Accedendo al dettaglio Pratica, gli Utenti ISP e Deloitte potranno infatti visualizzare le informazioni inerenti covenant, spread, scadenziere della specifica operazione raccolte nella fase di Post-closing e riportate per la nuova pratica di Monitoraggio

NB: Gli utenti assegnati alla lavorazione della pratica (Utente ISP e Utente Deloitte) rimarranno i medesimi della pratica di censimento collegata

Ciascuna pratica di Monitoraggio sarà creata a partire dalle informazioni riferite ai Covenant Finanziari Numerici (estratti dalla Gen AI e verificati dal consulente Deloitte).

Infatti, ciascun covenant numerico dovrà essere preso in considerazione dalla Piattaforma per creare uno scadenziere di monitoraggio che sarà poi organizzato per andare a creare le diverse sezioni/tabelle all’interno della pratica di Monitoraggio.

Pertanto, di seguito si riportano le informazioni salvate dalla piattaforma e le modalità di creazione dello scadenziere relativamente a:

* Covenant Finanziari Numerici (riferiti alla medesima pratica di censimento individuabile dal Tracking ID)

Di seguito sono riportati i campi necessari per la creazione degli Eventi di Monitoraggio, basati sulle informazioni già censite in piattaforma e relative ai covenant numerici estratti e verificati.

In particolare, viene fornito il tracciato di output dei covenant numerici, ovvero la fonte da cui recuperare i dati per la costruzione della tabella, insieme all’indicazione puntuale della logica di riconduzione per ciascun dato.

Tracciato output Post-Closing per Covenant Numerici: ![](data:image/x-emf;base64...)

Colonne della tabella:

* + **Tipologia Covenant**: Campo da recuperare nella riga del tracciato output nominata “Tipologia Covenant” formato alfanumerico.
  + **Data di Riferimento Prima Rilevazione**: Campo da recuperare nella riga del tracciato output nominata “Data Riferimento Documento Prima Rilevazione”, formato data.
  + **Scadenza di monitoraggio**: Campo da recuperare nella riga del tracciato output nominata “Data di scadenza”, formato data.
  + **Periodicità Monitoraggio**: Campo da recuperare nella riga del tracciato output nominata “Periodicità di Monitoraggio”, formato alfanumerico.
  + **Valore Covenant Fisso (segno)**: Campo da recuperare nella riga del tracciato output nominata “Valore Covenant Fisso (segno)”, formato alfanumerico (carattere speciale).
  + **Valore Covenant Fisso (valore effettivo)**: Campo da recuperare nella riga del tracciato output nominata “Valore Covenant Fisso (valore effettivo)”, formato alfanumerico.
  + **Covenant Variabile (Si/No)**: Campo da recuperare nella riga del tracciato output nominata “Covenant variabile (se sì, inserire Piano Target)”, valore binario (SI/NO).

La Piattaforma pertanto utilizzerà tali informazioni per creare uno scadenziere per ciascun covenant finanziario a seconda della periodicità e della Termination Date (quest’ultima estratta durante la fase Post Closing inerente il campo “Scadenza del deal più lontata tra le facility” della pratica di censimento collegata), di seguito un esempio:

|  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| **Tipologia Covenant** | **Data di Riferimento Prima Rilevazione** | **Scadenza Monitoraggio** | **Periodicità di Monitoraggio** | **Valore Covenant Fisso (segno)** | **Valore Covenant Fisso (valore effettivo)** | **Covenant Variabile (Si/No)** |
| Loan to Value | 31/03/2026 | 30/06/2026 | Annuale | < | 5% | NO |

La Termination date inserita manualmente dall’utente nella fase precedente per questo esempio risulterà essere il 31/07/2030

Pertanto, la Piattaforma dovrà creare per tale Covenant i seguenti eventi di monitoraggio (caratterizzati da un proprio ID Univoco

|  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **ID Evento di Monitoraggio** | **Tipologia Covenant** | **Note** | **Data di Riferimento Prima Rilevazione** | **Scadenza Monitoraggio** | **Periodicità di Monitoraggio** | **Valore Covenant Fisso (segno)** | **Valore Covenant Fisso (valore effettivo)** | **Covenant Variabile (Si/No)** |
| P1C2M1 | Loan to Value |  | 31/03/2026 | 30/06/2026 | Annuale | < | 5% | NO |
| P1C2M2 | Loan to Value |  | 31/03/2027 | 30/06/2027 | Annuale | < | 5% | NO |
| P1C2M3 | Loan to Value |  | 31/03/2028 | 30/06/2028 | Annuale | < | 5% | NO |
| P1C2M4 | Loan to Value |  | 31/03/2029 | 30/06/2029 | Annuale | < | 5% | NO |
| P1C2M5 | Loan to Value |  | 31/03/2030 | 30/06/2030 | Annuale | < | 5% | NO |

Essendo la periodicità annuale, ciascun evento di monitoraggio sarà caratterizzato dalle medesime informazioni ad eccezioni della data di riferimento e della data di scadenza che aumenteranno progressivamente per ciascun evento di monitoraggio (es. 12 mesi essendo la periodicità annuale). La periodicità potrà essere mensile, trimestrale, semestrale e annuale e pertanto ciascun evento di monitoraggio dovrà aumentare progressivamente rispettivamente di 1 mese, 3 mesi, 6 mesi e 12 mesi. L’ultimo record di monitoraggio dovrà essere registrato fino a quando la data di riferimento del record non sia successiva alla Termination date, quando accade quest’ultimo record non dovrà essere registrato e non dovranno essere creati ulteriori eventi di monitoraggio per tale Covenant numerico.

Tale requisito dovrà essere applicato anche per tutti gli altri Covenant Finanziari previsti per la pratica di censimento.

Pertanto, la Piattaforma assocerà i seguenti ID per le pratiche di Monitoraggio:

* ID Pratica di Monitoraggio 🡪 Strutturato con “M” e a fianco un ID numerico che sia progressivo e identifichi ciascuna delle pratiche di monitoraggio create
* ID Covenant (da associare a ciascun Covenant relativo alla Pratica di Monitoraggio) 🡪 Strutturato con [ID Pratica di Monitoraggio] e accostato da “C” e a fianco un ID numerico che sia progressivo per ciacscun covenant di riferimento della pratica (rispetto a quelli estratti in fase di censimento).
* ID Evento di monitoraggio Covenant 🡪 Strutturato con [ID Pratica di Monitoraggio] + [ID Covenant] + accostare “E” e a fianco un ID numerico progessivo per ciascun evento di monitoraggio del Covenant

Di seguito un esempio:

* + ID Pratica di Monitoraggio (creata dopo completamento Post Closing Fase di Censimento): M1
  + ID Covenant (se nella pratica di censimento erano stati estratti 3 Covenant si avranno 3 ID differenti per identificarli):
    - M1C1; M1C2; M1C3
  + ID Evento di Monitoraggio (per uno dei covenant si dovranno verificare 2 eventi di monitoraggio):
    - M1C1E1; M1C1E2

In caso di covenant variabile, il report estratto provvederà una serie di soglie (valore effettivo) differenti rispetto a un dato periodo di riferimento.

Sulla base delle logiche sopra descritte saranno strutturate le diverse pratiche di monitoraggio, di seguito si riporta il dettaglio.

## 2. TABELLA PRATICHE MONITORAGGIO

### 2.1 Descrizione Generale e Accesso

La pagina “Pratiche Monitoraggio” (accessibile dal menu laterale sinistro) rappresenta una vista consolidata e completa di tutte le pratiche in gestione nella piattaforma. A differenza della dashboard che mostra solo un riepilogo, questa pagina fornisce una lista esaustiva di tutte le pratiche con i loro stati di monitoraggio, permettendo agli utenti di avere una visione d’insieme e di navigare verso i dettagli specifici di ogni pratica.

**Accesso:** Menu Laterale > “Pratiche Monitoraggio” oppure dal link “Vedi tutte” sopra alla tabella “Pratiche Monitoraggio Recenti” nella Dashboard

### 2.2 Sezione Filtri e Ricerca Avanzata

#### Descrizione Funzionale

La pagina “**Pratiche Monitoraggio**” inizia con una sezione dedicata ai filtri e alla ricerca, che consente agli utenti di restringere l’elenco delle pratiche secondo vari criteri.

#### Campi di Filtro Disponibili

**Filtro Utente Banca:** Un campo di ricerca che consente di cercare per email dell’utente Banca di riferimento

**Filtro Tracking ID:** Un campo di ricerca per cercare per il Tracking ID/ID Contratto della pratica

**Filtro ID Monitoraggio Pratica:** Un campo di ricerca per cercare l’ID della pratica di Monitoraggio

**Filtro Nome Deal:** un campo di ricerca per cercare il nome del Deal di riferimento

**Filtro Semaforo:** Un dropdown che consente di selezionare uno o più stati di monitoraggio: OK (verde) - Pratiche con monitoraggi completati; In Scadenza (giallo) - Pratiche con monitoraggi in scadenza; Da attenzionare (Arancione) - Valore covenant da attenzionare; Scaduto (rosso) - Pratiche con monitoraggi scaduti; Fuori Soglia (rosso scuro) - Pratiche con monitoraggi Fuori Soglia; Conclusa (Grigio) – Pratiche concluse

#### Pulsanti di Azione

**Pulsante “Applica filtri”:** Esegue la ricerca con i filtri selezionati e aggiorna la tabella sottostante.

**Pulsante “Resetta”:** Cancella tutti i filtri e mostra nuovamente tutte le pratiche.

**Pulsante “Scarica Report Pratiche”:** Esporta le informazioni di tutte le pratiche visualizzate, sulla base dei filtri applicati

Il report conterrà le medesime colonne presenti nella tabella in Piattaforma e una riga per ogni pratica monitorata

### 2.3 Tabella Pratiche Monitoraggio - Vista Consolidata

#### Descrizione Funzionale

La tabella principale della pagina “Pratiche Monitoraggio” fornisce una vista consolidata di tutte le pratiche, con informazioni chiave su ciascuna pratica e il suo stato di monitoraggio. Questa tabella è il cuore della pagina e consente agli utenti di navigare verso i dettagli specifici di ogni pratica.

#### Colonne e Dati

La tabella contiene le seguenti colonne:

1. **ID Monitoraggio Pratica:** L’identificativo univoco della pratica che assume un valore progressivo univoco a partire dalla prima pratica di monitoraggio creata (il formato sarà M1, mantenendo “M” fisso e progressivamente aumentando la parte numerica dell’ID)
2. **Tracking ID:** ID univoco dei sistemi ISP per riconoscere l’operazione di riferimento, dato recuperato dal medesimo campo riferito alla relativa pratica di censimento
3. **Nome Deal:** Il nome della pratica/finanziamento (es. “Banca Nuova S.p.A”), dato recuperato dal medesimo campo riferito alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
4. **Utente Banca:** L’email dell’utente ISP responsabile (es. “Marioverdi@isp-it”), dato recuperato dal medesimo campo riferito all’utente Banca associato alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
5. **Utente Deloitte:** L’email dell’utente Deloitte assegnata alla pratica di monitoraggio (es. mverdi@deloitte.it) e viene ereditato rispetto a quello già assegnato alla pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
6. **Tipologia Pratica:** La tipologia della pratica (es. “Banca Agente”, “SACE Agent”), dato recuperato dal medesimo campo riferito alla tipologia pratica relativa alla pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
7. **#Monitoraggi Scaduti:** Il numero eventi di monitoraggio della pratica che presentano lo stato “Monitoraggio Scaduto”
8. **# Monitoraggi in Scadenza:** Il numero eventi di monitoraggio della pratica che presentano lo stato “Monitoraggio in Scadenza”
9. **# Documenti in Lavorazione:** Il numero di documenti caricati dall’Utente Banca che non hanno ancora concluso l’iter di lavorazione dell’Utente Deloitte (**Stato documento [Vista ISP]:** In attesa di conferma, Verifica Dati in Corso oppure

**Stato documento [Vista Deloitte]:** In attesa di conferma;Verifica Dati in corso; Estrazione GenAI in corso; Estrazione GenAI da validare)

1. **Semaforo:** Un indicatore visivo dello stato complessivo della pratica, rappresentato da un punto colorato:
   1. Verde (●) = Monitoraggio OK
   2. Giallo (●) = Monitoraggio In scadenza
   3. Giallo (●) = Monitoraggio da attenzionare
   4. Rosso (●) = Monitoraggio Scaduto
   5. Rosso (●) = Monitoraggio fuori soglia
   6. Grigio (●) = Monitoraggio concluso
2. **Data creazione Pratica:** La data in cui la pratica è stata creata nel sistema (es. “15/03/2015”)
3. **Data Fine monitoraggio:** La data in cui il periodo di monitoraggio termina (es. “22/11/2019”), corrispondente alla Termination date determinata nella fase di Post-Closing, oltre questa data la pratica viene automaticamente chiusa e il suo stato diventa “Pratica Conclusa”
4. **Azione:** colonna di azione contestuale in cui è possibile chiudere anticipatamente la chiusura pratica di monitoraggio (segue il medesimo flusso per richiedere un’eccezione a un monitoraggio singolo, solo che dopo aver richiesto la chiusura anticipata la pratica verrà considerata conclusa e non più interagibile)

#### Comportamento e Interattività

**Ordinamento:** Tutte le colonne sono ordinabili. Cliccando sull’intestazione di una colonna, la tabella si ordina per quella colonna in ordine crescente (A-Z) o decrescente (Z-A).

**Righe per Pagina:** La tabella mostra 20 righe per pagina

**Paginazione:** Controlli di paginazione “<< Pagina X >>” consentono di navigare tra le pagine.

**Riga Selezionabile:** Cliccando su una riga della tabella, l’utente naviga al dettaglio della pratica dove può visualizzare i dettagli completi dei covenant, spread e scadenziere

**Evidenziazione Riga:** Al passaggio del mouse su una riga, la riga è evidenziata con un colore di sfondo leggermente più scuro per migliorare l’usabilità.

## 3. FLUSSO ISP (UTENTE BANCA)

### 3.1 Descrizione Generale del Flusso ISP

Il Flusso ISP rappresenta il percorso completo dell’Utente Banca all’interno della piattaforma Agency Desk. L’utente ISP è il responsabile operativo della pratica e ha il compito di:

1. **Caricare i Compliance Certificate** per i Covenant: Documenti che certificano il rispetto delle obbligazioni finanziarie
2. **Richiedere Eccezioni ai Monitoraggi:** Quando non è possibile completare un monitoraggio entro la scadenza, l’utente ISP può richiedere una deroga
3. **Visualizzare lo Stato dei Monitoraggi:** Monitorare l’avanzamento dei monitoraggi e identificare situazioni critiche

Per compiere tali azioni l’utente ISP può navigare, all’interno del Dettaglio Pratica, tra le seguenti 3 sezioni: **Covenant, Spread, Scadenziere**

In alto, nella schermata di dettaglio pratica, ma indipendentemente dalla sezione selezionata, saranno presenti le seguenti informazioni in un header:

* ID pratica di Monitoraggio
* NDG Cliente
* Tipologia Pratica
* ID Contratto
* Utente Banca
* Stato Pratica

### 3.2 Dettaglio Pratica - Sezione Covenant (ISP)

#### 3.2.1 Overview Covenant - Riepilogo dello Stato

Quando l’utente ISP accede al dettaglio di una pratica e seleziona la scheda “Covenant”, visualizza una sezione denominata “Overview Covenant” che fornisce un riepilogo completo dello stato dei covenant per quella pratica.

**Titolo e Sottotitolo:** La sezione è intitolata “Overview Covenant”

**Card Totale Covenant Monitorati:** Una card posta in alto a sinistra mostra il numero totale di covenant che devono essere monitorati per questa pratica (es. “5”).

**Card Overview Scadenze della Pratica:** alla destra della card del totale è presente una card che fornisce un riepilogo dello stato dei covenant suddiviso in tre categorie: - **Ok (verde):** Numero di covenant che presentano lo stato “Monitoraggio OK” - **In Scadenza (giallo):** Numero di covenant presentano lo stato “Monitoraggio in Scadenza” o “Monitoraggio da attenzionare” - **KO (rosso):** Numero di covenant che presentano lo stato “Monitoraggio Scaduto” o “Monitoraggio Fuori Soglia”

Ogni categoria è accompagnata da un indicatore colorato (punto colorato) per una rapida identificazione visiva.

**Card Overview Avanzamento Sezione:** Una card che mostra informazioni sulla gestione della sezione Covenant:

- **SEZIONE:** Mostra “Covenant”

- **IN CARICO A:** Mostra a chi è assegnato il compito di completare i covenant. Può essere “Utente Banca” (ISP) se il monitoraggio è in attesa di documentazione (tutti i documenti caricati dall’Utente Banca presentano lo stato “Completo” e sono ancora presenti degli eventi di monitoraggi futuri da monitorare e vi è almeno un evento di monitoraggio con stato “in scadenza” o “scaduto”), o “Utente Deloitte” se è presente almeno un documento caricato in lavorazione (non in stato “Completo”)

- **STATO LAVORAZIONE:** Mostra lo stato del lavoro di verifica della documentazione caricata dall’Utente Banca:

- “In attesa Documentazione” = L’utente ISP deve caricare i documenti (tutti i documenti caricati dall’Utente Banca presentano lo stato “Completo” e sono ancora presenti degli eventi di monitoraggi futuri da monitorare e vi è almeno un evento di monitoraggio con stato “in scadenza” o “scaduto”)

- In attesa di conferma = Il documento è stato caricato e deve essere confermato da Utente Deloitte (in caso di più documenti caricati prevedere di visualizzare lo stato meno avanzato)

- Verifica dati in corso = Deloitte ha confermato il documento e di conseguenza avvia il processo di estrazione GenAI/Gestione Manuale (Per la vista Deloitte, solo in caso di integrazione con la GEN AI, saranno previsti anche i sottostati “Estrazione GenAI in corso” e “Estrazione GenAI da validare”)

Completato = L’utente Deloitte ha validato l’estrazione GenAI/ ha caricato manualmente il report

#### 3.2.2 Caricamento Compliance Certificate - Gestione Documenti

La sezione “Caricamento Compliance Certificate” consente all’utente ISP di caricare i documenti che certificano il rispetto dei covenant.

La sezione mostra:

- **Data Ultimo Documento caricato:** La data in cui è stato caricato l’ultimo documento (es. “16/03/2025”)

- **# Documenti Caricati:** Il numero totale di documenti caricati per questa sezione (es. “5”)

- **# Documenti in lavorazione:** Il numero totale di documenti caricati per questa sezione che sono al momento in fase di verifica da parte di Deloitte (che non assumono lo stato “Completato”)

- **Link “Vedi Tutti”:** Un link che apre la sezione storico dei documenti, dove l’utente può visualizzare tutti i documenti caricati in precedenza

**Area di Upload:** La sezione include un’area di upload con le seguenti caratteristiche: - **Area Drag & Drop:** Un’area con bordo tratteggiato che consente all’utente di trascinare file direttamente - **Testo Descrittivo:** “Trascina qui il tuo file” con un’icona di upload - **Pulsante “Scegli File”:** Un pulsante che apre un file picker per selezionare file dal computer

**Validazione File:** - **Formati Accettati:** .pdf, .xlsx, .xls, .doc, .docx, .msg, .p7m - **Dimensione Massima:** 3 MB per file - **Upload Multiplo:** L’utente può caricare più file contemporaneamente (in caso si creeranno più slot documentali all’interno della sezione)

#### 3.2.3 Pagina Documenti Covenant - Gestione Storico Compliance Certificate

Quando l’utente ISP clicca su “Vedi Tutti” nella sezione “Caricamento Compliance Certificate” della pagina Covenant, accede a una pagina dedicata alla gestione completa dei compliance certificate caricati per quella pratica. Questa pagina fornisce una vista storica di tutti i documenti caricati nel tempo, permettendo all’utente di tracciare l’evoluzione della documentazione e di accedere facilmente ai documenti precedenti.

**Titolo Pagina:** “Overview Covenant | Pratica XXX” (es. “Overview Covenant | Pratica 002”)

**Pulsante Torna alla Dashboard:** Un pulsante “Torna alla dashboard” che consente di tornare alla pratica principale

**Sezione Compliance Certificate - Documenti In Lavorazione**

Questa sezione mostra i compliance certificate che sono stati caricati di recente e sono ancora in fase di lavorazione (non ancora verificati e approvati da Deloitte).

**Struttura di ogni Slot Documento:**

- **Nome File:** Il nome del documento caricato (es. “[Nome File] 6”)

- **Data Documento Caricato:** La data in cui il documento è stato caricato nel sistema (es. “21/03/2026”)

- **Status:** es. Un badge giallo che mostra “In Attesa di Conferma” “Verifica dati in corso” o “Completato”, indicando che il documento è in attesa di conferma da parte di Deloitte, in lavorazione o che i covenant sono stati lavorati

- **Giorni Giacenza Stato**: Un contatore che misura da quanti giorni il documento caricato è nel determinato stato

- **Pulsante “Visualizza”** che apre la vista di dettaglio del singolo documento con la sua relativa assegnazione

- **Icone di Azione:** - Icona di download per scaricare il documento, icona commento per inserire un commento al documento

**Comportamento:** I documenti in questa sezione sono ordinati per data di caricamento (più recenti in alto). Se non ci sono documenti in lavorazione, la sezione mostra un messaggio “Nessun documento in lavorazione”.

**Sezione Compliance Certificate - Documenti Lavorati**

Questa sezione mostra i compliance certificate che sono stati verificati e approvati da Deloitte, la composizione delle righe che mostrano i documenti è identica alla sezione sopra, con la sola eccezione dello status che risulterà “Completato” e della presenza di un pulsante “Visualizza” che permetterà di scaricare il report di assegnazione del compliance certificate ai rispettivi monitoraggi

#### 3.2.4 Pagina Documenti Covenant – Vista di dettaglio tramite pulsante visualizza

L’utente banca che clicca sul pulsante visualizza atterrerà su una sezione di dettaglio dove potrà vedere i dettagli dell’assegnazione di un compliance certificate.

La schermata è così composta:

Viewer del documento sulla sinistra

Pulsante “Indietro” che permette di tornare alla vista dei documenti

Indicatori di tipologia Covenant, Valore Monitoraggio e Data di Riferimento per ogni covenant assegnato da Deloitte per quel documento

#### 3.2.5 Tabella Monitoraggi Scaduti/Fuori Soglia (Covenant)

Sotto la sezione di upload, è presente una tabella che mostra i covenant il cui monitoraggio è scaduto o fuori soglia

**Colonne della Tabella:**

1. **ID Covenant:** L’identificativo del covenant, composto da ID pratica + ID Covenant (E.g. il secondo Covenant della terza Pratica sarà “M3C2”)

2. **Tipologia:** Il tipo di covenant (es. “Leverage Ratio”, “Gearing Ratio”)

3 **Note:** il campo note contenente eventuali indicazioni sul covenant / indicazione della sua localizzazione sul documento

4. **ID Evento Monitoraggio:** L’identificativo dell’evento di monitoraggio (es. “M1C1E1”)

5. **Data di riferimento:** La data a cui di riferimento dell’evento di monitoraggio (es. “31/12/2024”)

6. **Scadenza monitoraggi:** La data entro cui il monitoraggio doveva essere completato (es. “31/12/2025”)

7. **Giorni al prossimo monitoraggio:** Il numero di giorni di ritardo (es. “-45” indica 45 giorni di ritardo)

8. **Valore Monitoraggi:** Il valore attuale del covenant. Il valore è assente, è si visualizzerà un’icona “!” in rosso

9. **Segno Valore Limite:** Il segno di confronto (es. “<” per “minore di”, “>” per “maggiore di”)

10. **Valore limite:** Il valore limite del covenant (es. “40”, “70”)

11. **Status Monitoraggio:** Lo stato del monitoraggio, visualizzato con un punto rosso (●) accanto a “Monitoraggio Scaduto/Fuori Soglia)

12. **Azione:** Un link “Salta Monitoraggio/Ignora Soglia” che consente all’utente di richiedere un’eccezione

**Comportamento:** - Le colonne sono ordinabili per “Giorni al prossimo monitoraggio” e “Scadenza monitoraggi” - Se non ci sono monitoraggi scaduti, la tabella mostra un messaggio “Nessun monitoraggio scaduto”

#### 3.2.6 Tabella Monitoraggi in Scadenza (Covenant)

Accanto alla tabella dei monitoraggi scaduti, è presente una tabella che mostra i covenant il cui monitoraggio sta per scadere (“In Scadenza”).

**Colonne della Tabella:** Le stesse della tabella “Monitoraggi Scaduti”

**Indicatore Stato:** Un punto giallo (●) accanto a “Monitoraggio In scadenza”

**Comportamento:** - Le colonne sono ordinabili, con ordinamento predefinito per “Giorni al prossimo monitoraggi” in ordine crescente (i monitoraggi che scadono prima in alto) - Se non ci sono monitoraggi in scadenza, la tabella mostra un messaggio “Nessun monitoraggio in scadenza”

#### 3.2.7 Tabella “Tutti i Covenant” (Covenant)

Sotto alle tabelle dei monitoraggi scaduti e dei monitoraggi in scadenza è presente una terza tabella denominata “Tutti i Covenant”

**Colonne della tabella:**

1. **ID Covenant:** L’identificativo del covenant, composto da ID pratica + ID Covenant (E.g. il secondo Covenant della terza Pratica sarà “M3C2”)
2. **Tipologia:** Il tipo di covenant (es. “Alpha”, “Leverage Ratio”, “Gearing Ratio”)
3. **Note:** il campo note contenente eventuali indicazioni sul covenant / indicazione della sua localizzazione sul documento
4. **# Tot. Monitoraggi:** Corrispondente alla somma degli eventi di monitoraggio del Covenant di riferimento
5. **# Monitoraggi da svolgere**: Corrispondente alla somma degli eventi di Monitoraggio la cui data di scadenza risulta antecedente al giorno corrente
6. **# Monitoraggi ok**: Corrispendente alla somma degli eventi di Monitoraggio che presentano lo stato “OK”
7. **# Monitoraggi Ko**: Corrispendente alla somma degli eventi di Monitoraggio che presentano lo stato “Fuori Soglia” o “Scaduto”
8. **Segno Valore Limite:** Il segno di confronto (es. “<” per “minore di”, “>” per “maggiore di”)
9. **Valore limite:** Il valore limite del covenant (es. “40”, “70”)
10. **Status Monitoraggio:** Lo stato del monitoraggio che dovrà seguire le seguenti logiche:
11. Green – Monitoraggio Ok – tutti i monitoraggio risultano in linea alle regole estratte dal CdF (se il documento raccolto dopo la scadenza il check sullo stato viene fatto solo su valore soglia)
12. Mettiamo quello inserito nella macchina stati
    * Yellow – Monitoraggio in Scadenza – almeno un monitoraggio risulta Yellow - Data Scadenza Monitoraggio < 7gg rispetto al Today
    * Yellow – Valore soglia da attenzionare – Quando almeno un monitoraggio ha il valore per il covenant numerico ha un valore che si discosta di un 10% dalla soglia di rottura del covenant oppure Quando il valore per il covenant numerico ha un valore che si discosta di un 30% dalla soglia di rottura del covenant ed è differito del 40% rispetto alla rilevazione precedente (avvicinandosi al valore soglia);
    * Red – Fuori Soglia - Valore di almeno un monitoraggio non rispetta le regole estratte dal CdF (se il documento raccolto dopo la scadenza il check sullo stato viene fatto solo su valore soglia)
    * Red – Monitoraggio Scaduto – Almeno un monitoraggio ha Data Scadenza Monitoraggio > Today
    * Gli status Red hanno la priorità nel determinare lo status monitoraggio del covenant rispetto agli status Yellow
13. **Prossimo Monitoraggio:** Un link “Salta Monitoraggio” che consente all’utente di richiedere un’eccezione

### 3.3 Dettaglio Covenant Singolo (ISP)

#### 3.3.1 Pagina Dettaglio Covenant - Analisi Approfondita

Quando l’utente ISP clicca su un covenant specifico, che sia dalla tabella dei monitoraggi scaduti, in scadenza, o dalla tabella “Tutti i Covenant”, accede a una pagina dedicata al dettaglio di quel covenant. Questa pagina fornisce un’analisi approfondita dello stato del covenant e della cronologia dei monitoraggi.

**Titolo Pagina:** “Dettaglio Pratica XXX | Covenant YYY” (es. “Dettaglio Pratica M2 | Covenant C4”)

**Pulsante Torna Dashboard:** Un pulsante “Torna alla dashboard” che consente di tornare alla pratica principale

**Sezione Stato Avanzamento:** Una card che mostra: - **Tipologia Covenant:** Il tipo di covenant (es. “Loan to Value”) - **Valore limite covenant:** Il valore limite (es. “< 70”) - **Covenant Variabile:** Se il covenant è variabile (SI/NO) – **Note** - **Monitoraggi da svolgere:** Il numero di monitoraggi ancora da completare (es. “5/8”)

**Indicatori Circolari/Semaforo:** Tre indicatori circolari colorati che mostrano il conteggio dei monitoraggi per stato: - Verde (●) con numero di eventi di monitoraggi OK - Giallo (●) con numero di eventi di monitoraggi in scadenza/da attenzionare - Rosso (●) con numero di eventi di monitoraggi scaduti/fuori soglia

**Sezione Scadenziere Monitoraggi:** Una tabella completa che mostra tutti gli eventi di monitoraggio del covenant, con colonne:

1. **ID Evento di Monitoraggio**
2. **Data di riferimento**
3. **Scadenza monitoraggio**
4. **Giorni al prossimo monitoraggio** [calcolata come differenza tra il giorno corrente e la “Scadenza Monitoraggio”]
5. **Data avvenuto monitoraggio** [Data in cui Deloitte preme il pulsante “Concludi Assegnazione” per l’evento di moitoraggio]
6. **Segno Valore Limite**
7. **Valore limite**
8. **Valore nel momento del monitoraggio** [Valore estratto dal Compliance Certificate e associato all’evento di monitoraggio confermando l’operazione mediante il pulsante “Concludi Assegnazione”]
9. **Status** (con indicatore colorato):
   * **Green – Monitoraggio Ok** - Valore Covenant in linea alle regole estratte dal CdF (se il documento raccolto dopo la scadenza il check sullo stato viene fatto solo su valore soglia)
   * **Yellow – Monitoraggio in Scadenza** - Data Scadenza Monitoraggio < 7gg rispetto al Today
   * **Yellow – Valore soglia da attenzionare** - Quando il valore per il covenant numerico ha un valore che si discosta di un 10% dalla soglia di rottura del covenant oppure Quando il valore per il covenant numerico ha un valore che si discosta di un 30% dalla soglia di rottura del covenant ed è differito del 40% rispetto alla rilevazione precedente (avvicinandosi al valore soglia);
   * **Red – Fuori Soglia** - Valore Covenant non rispetta le regole estratte dal CdF (se il documento raccolto dopo la scadenza il check sullo stato viene fatto solo su valore soglia)
   * **Red – Monitoraggio Scaduto** - Data Scadenza Monitoraggio > Today
10. **Compliance certificate** (con link “Download”)
11. **Azioni** (“Salta Monitoraggio”, “Ignora Soglia” o “Visualizza Eccezione” se il monitoraggio è già stato gestito)
    1. L’azione “Salta Monitoraggio sarà presente solo per monitoraggi senza un compliance certificate e con data di avvenuto monitoraggio assente e data scadenza monitoraggio nel passato
    2. L’azione “Ignora Soglia” sarà presente solo per monitoraggi passati la cui soglia è stata sforata
    3. L’azione “Visualizza Eccezione” sarà presente solo per monitoraggi (ora verdi) che hanno registrato un’eccezione gestita con il flusso dettagliato nel capitolo successivo

### 3.4 Richiesta di Eccezione al Monitoraggio (ISP)

#### 3.4.1 Richiesta Eccezione - Processo di Deroga

Quando l’utente ISP clicca sulla frase di azione “Salta Monitoraggio” o “Ignora Soglia” si apre un modal intitolato “Richiesta di Eccezione al Monitoraggio”. Questo modal consente all’utente di richiedere una deroga al monitoraggio quando non è possibile completarlo entro la scadenza.

**Titolo e Descrizione:** Il modal è intitolato “Richiesta di Eccezione al Monitoraggio” e include una descrizione: “Descrivi le motivazioni a supporto della richiesta di saltare il monitoraggio/ignorare la soglia”

**Campo Testo Motivazione:** Un’area di testo libera dove l’utente può inserire la motivazione della richiesta di eccezione. Il campo ha i seguenti vincoli: - **Minimo:** 2 caratteri - **Massimo:** 2000 caratteri - **Contatore Caratteri:** Sotto il campo è visualizzato un contatore “X/2000 caratteri” che aggiorna in tempo reale mentre l’utente digita

**Sezione Upload Documentazione:** Una sezione intitolata “Carica eventuale documentazione a supporto” che consente all’utente di caricare file di supporto per la richiesta di eccezione (caricamento opzionale).

**Area Upload:** Un’area drag & drop per caricare file, con le seguenti caratteristiche: - **Formati Accettati:** .pdf, .xlsx, .xls, .doc, .docx, .msg, .p7m - **Dimensione Massima:** 3 MB per file - **Upload Multiplo:** L’utente può caricare più file

**Pulsanti di Azione:** - **Pulsante “Annulla”:** Chiude il modal senza salvare la richiesta - **Pulsante “Invia”:** Sottomette la richiesta di eccezione. Il pulsante è disabilitato se il campo testo Motivazione è vuoto o contiene meno di 2 caratteri (non è necessario caricare un documento per inviare)

**Notifiche:** - **Notifica di Successo:** Dopo l’invio, mostra “Eccezione gestita con successo” **Notifica di Errore:** Se l’invio fallisce, mostra un messaggio di errore specifico

**Logica di Invio:** 1. L’utente inserisce la motivazione e carica eventuali documenti di supporto 2. Clicca “Invia” 3. Il sistema valida: - La motivazione non è vuota -I file (se caricati) hanno formato e dimensione corretti 4. Se la validazione è OK: - La richiesta di eccezione viene salvata con successo, l’azione contestuale su ogni riga di quel monitoraggio presente in piattaforma diventa “Visualizza Eccezione”.

N.B. La **richiesta** di **eccezione** da parte dell’Utente ISP, se confermata, **genera un cambio di stato** per l’evento di monitoraggio, il quale passerà allo status “green” (cfr. Allegato “Macchina stati”)

### 3.5 Sezione Spread (ISP)

#### 3.5.1 Pagina Spread

Quando l’utente ISP accede alla scheda “Spread”, dopo aver consolidato la tabella, sarà possibile visualizzare la tabella e lo storico delle variazioni dello spread.

**Titolo Pagina:** “Dettaglio Pratica XXX | Spread” (es. “Dettaglio Pratica 002 | Spread”)

**Sezione Tabella Spread:**

Una sezione che mostra la tabella di spread creata da Deloitte.

**Tabella Spread:**

La tabella contiene le seguenti colonne:

**Esempio di Tabella Spread:**

| Cluster | Sottostante | Valore Sottostante | Margine |
| --- | --- | --- | --- |
| 1 | Leverage Ratio | < 0,5 | 2.20% |
| 2 | Leverage Ratio | tra 0,5 e 1 | 2.30% |
| 3 | Leverage Ratio | tra 1 e 1,5 | 2.40% |
| 4 | Leverage Ratio | tra 1,5 e 2 | 2.60% |
| 5 | Leverage Ratio | > 2 | 2.80% |

**Comportamento:** - La tabella è di sola lettura per ISP

**Evidenziazione:** Il Cluster correntemente attivo è evidenziato sulla tabella sia nella vista ISP che Deloitte

**Nota Importante:** “Questa tabella è stata creata da Deloitte e rimane fissa. Le variazioni di spread sono tracciate nella sezione ‘Storico Spread’ sottostante.”

#### 3.5.2 Sezione Storico Spread

Una sezione che mostra la cronologia di tutte le variazioni di spread nel tempo.

**Titolo Sezione:** “Storico Spread”

**Tabella Storico Spread:**

Una tabella che mostra tutte le variazioni di spread registrate nel sistema.

**Colonne della Tabella:**

1. **Data Variazione:** La data in cui la variazione di spread è stata registrata (es. “04/03/2026”)
2. **Documento:** Il documento in cui è presente la causa della variazione del covenant che ha determinato una modifica dello spread (legato al covenant sottostante)
3. **Azione:** Scarica, che permette di scaricare e visualizzare il documento di riferimento del monitoraggio del covenant (sia che abbia causato una variazione o meno)
4. **Cluster**: il Cluster della tabella spread di riferimento per quella data variazione

**Comportamento:**

* Le variazioni sono ordinate per data (più recenti in primo luogo)
* Se non ci sono variazioni registrate, la tabella mostra un messaggio “Nessuna variazione di spread registrata”
* La tabella mostra un massimo di 10 variazioni per pagina con controlli di paginazione
* Sia ISP che Deloitte possono visualizzare lo storico

**Logica di Registrazione delle Variazioni:**

Le variazioni di spread sono registrate automaticamente quando: 1. Un monitoraggio di covenant è completato e il valore del covenant cambia 2. Il nuovo valore del covenant corrisponde a un range diverso nella tabella di spread 3. Lo spread applicato cambia di conseguenza 4. La variazione è tracciata nel sistema con data, causa e documento di supporto

### 3.6 Sezione Scadenziere (ISP)

#### 3.6.1 Pagina Scadenziere - Cronologia Consolidata

La sezione “Scadenziere” fornisce una vista consolidata di tutti gli eventi di monitoraggio dei covenant della pratica, ordinati cronologicamente.

**Titolo Pagina:** “Scadenziere Pratica XXX”

- **Filtro Tipologia:** Un dropdown che consente di filtrare per tipologia specifica (es. Gearing Ratio, Leverage Ratio)

- **Pulsante “Applica filtri”:** Esegue la ricerca con i filtri selezionati

**Tabella Scadenziere:** Una tabella che mostra tutti gli eventi di monitoraggio, con colonne:

1. **Tipologia:** Il tipo specifico di covenant (E.g. Leverage Ratio)
2. **ID Evento di Monitoraggio**
3. **Data di riferimento**
4. **Scadenza monitoraggio**
5. **Giorni al prossimo monitoraggio**
6. **Data avvenuto monitoraggio**
7. **Status** (con indicatore colorato)
8. **Compliance Certificate** (con link “Download”), se presente
9. **Azioni** (link “Salta Monitoraggio”/”Ignora soglia”)

**Ordinamento e Paginazione:** Le colonne sono ordinabili per Data di riferimento, Scadenza monitoraggio, Giorni al prossimo monitoraggio. La tabella mostra 20 righe per pagina

## 4. FLUSSO DELOITTE (UTENTE DELOITTE)

### 4.1 Descrizione Generale del Flusso Deloitte

Il Flusso Deloitte rappresenta il percorso dell’Utente Deloitte all’interno della piattaforma Agency Desk. L’utente Deloitte è il verificatore e approvatore dei monitoraggi e ha il compito di:

1. **Verificare i Compliance Certificate** caricati da ISP per i Covenant
2. **Approvare o Rifiutare le Richieste di Eccezione** presentate da ISP
3. **Assegnare i Documenti ai Covenant** per collegare i documenti caricati ai monitoraggi specifici
4. **Modificare i Dati dei Covenant** quando necessario
5. **Generare Report di Monitoraggio** per analisi e reporting

### 4.2 Dettaglio Pratica - Sezione Covenant (Deloitte)

#### 4.2.1 Overview Covenant (Deloitte)

Quando l’utente Deloitte accede al dettaglio di una pratica e seleziona la scheda “Covenant”, visualizza una sezione “Overview Covenant” che fornisce un riepilogo completo dello stato dei covenant per quella pratica, con una prospettiva di verifica e approvazione.

**Titolo e Sottotitolo:** “Overview Covenant” con sottotitolo della pratica (es. “Pratica 002”)

**Card Totale Covenant Monitorati:** Una card che mostra il numero totale di covenant che devono essere monitorati per questa pratica (es. “5”)

**Card Overview Scadenze della Pratica:** alla destra della card del totale è presente una card che fornisce un riepilogo dello stato dei covenant suddiviso in tre categorie: - **Ok (verde):** Numero di covenant che presentano lo stato “Monitoraggio OK” - **In Scadenza (giallo):** Numero di covenant presentano lo stato “Monitoraggio in Scadenza” o “Monitoraggio da attenzionare” - **KO (rosso):** Numero di covenant che presentano lo stato “Monitoraggio Scaduto” o “Monitoraggio Fuori Soglia

**Card Overview Avanzamento Sezione:**

- **SEZIONE:** Mostra “Covenant”

- **IN CARICO A:** Mostra a chi è assegnato il compito di completare i covenant. Può essere “Utente Banca” (ISP) se il monitoraggio è in attesa di documentazione (tutti i documenti caricati dall’Utente Banca presentano lo stato “Completo” e sono ancora presenti degli eventi di monitoraggi futuri da monitorare e vi è almeno un evento di monitoraggio con stato “in scadenza” o “scaduto”), o “Utente Deloitte” se è presente almeno un documento caricato in lavorazione (non in stato “Completo”)

- **STATO LAVORAZIONE:** Mostra lo stato del lavoro di verifica della documentazione caricata dall’Utente Banca:

- “In attesa Documentazione” = L’utente ISP deve caricare i documenti (tutti i documenti caricati dall’Utente Banca presentano lo stato “Completo” e sono ancora presenti degli eventi di monitoraggi futuri da monitorare e evi è almeno un evento di monitoraggio con stato “in scadenza” o “scaduto”)

- In attesa di conferma = Il documento è stato caricato e deve essere confermato da Utente Deloitte (in caso di più documenti caricati prevedere di visualizzare lo stato meno avanzato)

- Verifica dati in corso = Deloitte ha confermato il documento e di conseguenza avvia il processo di estrazione GenAI/Gestione Manuale (Per la vista Deloitte, solo in caso di integrazione con la GEN AI, saranno previsti anche i sottostati “Estrazione GenAI in corso” e “Estrazione GenAI da validare”)

Completato = L’utente Deloitte ha validato l’estrazione GenAI/ ha caricato manualmente il report

**Differenze rispetto alla Vista ISP:** - La vista Deloitte mostra uno stato di lavorazione più granulare con l’utilizzo dei sottostati “Verifica dati in corso – Estrazione GenAI in corso” “Verifica dati in corso – Estrazione GenAI da validare”. Inoltre, Deloitte ha accesso a funzionalità di verifica e approvazione non disponibili a ISP

#### 4.2.2 Compliance Certificate - Processo Assegnazione

La sezione “Caricamento Compliance Certificate” consente all’utente Deloitte di visualizzare e verificare i Compliance Certificate caricati da ISP per i covenant.

La sezione mostra:

- **Data Ultimo Documento caricato:** La data in cui è stato caricato l’ultimo documento (es. “16/03/2025”)

- **# Documenti Caricati:** Il numero totale di documenti caricati per questa sezione (es. “5”)

- **# Documenti in lavorazione:** Il numero totale di documenti caricati per questa sezione che sono al momento in fase di verifica da parte di Deloitte (che non assumono lo stato “Completato”)

- **Link “Vedi Tutti”:** Un link che apre la sezione storico dei documenti, dove l’utente può visualizzare tutti i documenti caricati in precedenza

**Area di Upload:** La sezione include un’area di upload con le seguenti caratteristiche: - **Area Drag & Drop:** Un’area con bordo tratteggiato che consente all’utente di trascinare file direttamente - **Testo Descrittivo:** “Trascina qui il tuo file” con un’icona di upload - **Pulsante “Scegli File”:** Un pulsante che apre un file picker per selezionare file dal computer

**Validazione File:** - **Formati Accettati:** .pdf, .xlsx, .xls, .doc, .docx, .msg, .p7m - **Dimensione Massima:** 3 MB per file - **Upload Multiplo:** L’utente può caricare più file contemporaneamente (in caso si creeranno più slot documentali all’interno della sezione)

**Sezione Compliance Certificate:**

Dopo aver premuto il pulsante “Accedi alla sezione” l’Utente Deloitte potrà visualizzare i documenti caricati da ISP, comprensivi dei Compliance certificate già verificati e quelli in corso di lavorazione Deloitte.

Ciascun slot documentale è caratterizzato da:

1. **Nome File:** Il nome del documento caricato (es. “Compliance\_2025.pdf”)

2. **Data Caricamento:** La data in cui il documento è stato caricato (es. “16/03/2025”);

3. **Stato Verifica:** Lo stato attuale della verifica, visualizzato con un indicatore colorato (vd. Macchina stati) e i pulsanti “Conferma” / ”Rifiuta”; Pulsante “Visuallizza Assegnazione” qualora sia stato già generato il Report relativo al Compliance Certificate; Pulsante “Download” e pulsante “Commento”

Se viene cliccato il pulsante “Rifiuta” Deloitte può specificare il motivo del rifiuto mediante la funzionalità commento. Una notifica viene inviata a ISP per informarlo del rifiuto e del motivo - ISP può ricaricare una versione corretta del documento

**Comportamento:** - I documenti sono ordinati per data di caricamento (più recenti in alto) - Se non ci sono documenti in attesa di lavorazione la tabella mostra un messaggio “Nessun documento in attesa di verifica” .

#### 4.2.3 Assegnazione Compliance Certificate - Processo Manuale

Quando un documento è confermarlo, Deloitte deve assegnarlo al covenant specifico. Questo processo collega il documento a uno o più eventi di monitoraggio del covenant.

Quando l’utente Deloitte clicca su “Conferma” per un documento, compare il pulsante “Assegna Monitoraggi” che abilita il processo di assegnazione dell’Utente Deloitte.

L’Utente Deloitte, premendo il pulsante “Assegna Monitoraggi” viene indirizzato alla schermata di Assegnazione composta da:

* Preview del documento di Compliance Certificate
* Box di selezione tra “**Assegnazione con GenAI”** e “**Assegnazione manuale**”
* Pulsante “**Indietro**” che se premuto fa tornare alla visualizzazione della sezione dei documenti della pratica

Nel caso di “Assegnazione manuale” l’utente visualizzerà due box:

* “Aggiungi Covenant” che permette di avviare la procedura di assegnazione
* “Annulla” permette la gestione del bad flow interrompendo subito l’assegnazione senza collegare alcun covenant

Dopo aver cliccato su “Aggiungi Covenant” comparirà una riga per il covenant 1, con a fianco un’icona cestino per eliminarla

Sotto compariranno tre box:

* **Seleziona Tipologia Covenant -** contenente un drop down con all’interno tutti i covenant presenti sulla pratica (Quindi un subset dell’elenco completo di tipologie).Ogni covenant sarà nominato seguendo questa logica **“[ID del covenant] – [Tipologia del Covenant]”.**
* **Valore Monitoraggio** – in cui inserire il valore numerico del monitoraggio
* **Data di Riferimento –** la data di riferimento del monitoraggio (con formato “Data”)

E due pulsanti “**Associa Covenant**” (disinibito fino a quando non vengono compilati i 3 box sopra) e “**Annulla**”, in particolare:

* **“Annulla”** Resetta i filtri ma non cancella la riga del covenant
* **“Associa Covenant”** apre uno scadenziere sotto al pulsante composto delle seguenti colonne:

1. ID Evento di Monitoraggio (es. P2C3M4)

2. Data di riferimento (gg/mm/aaaa);

3. Scadenza Monitoraggio (es. gg/mm/aaaa);

4. Giorni al prossimo monitoraggio (es. 4)

5. Data avvenuto monitoraggio (gg/mm/aaaa)

6. Valore al momento del monitoraggio;

7. Status (es. Monitoraggio scaduto / in scadenza / ok)

8. Compliance Certificate (qui sarà presente link per scaricare il documento)

9. Azioni (Assegna / Rimuovi)

Gli eventi di monitoraggio inclusi nello scadenziere visualizzato sono riferiti ai covenant riferito alla Tipologia Covenant selezionato e aventi le seguenti caratteristiche:

* Eventi di monitoraggio che risultano essere in stato “Monitoraggio Scaduto”
* Eventi di monitoraggio che risultano essere in stato “Fuori Soglia”
* Eventi di monitoraggio che risultano essere in stato “Monitoraggio In Scadenza”
* Eventi di monitoraggio che presentano la data di riferimento uguale a quella inserita nel campo “Data di Riferimento” sopra
* Eventi di monitoraggio che presentano la data di riferimento che si discosta tra +/- 15 giorni dalla data inserita nel campo “Data di Riferimento” sopra

Dopo aver selezionato “Assegna” su una delle righe, comparirà sotto il pulsante “**Conferma Assegnazione**”

Una volta premuto il pulsante “**Conferma Assegnazione**”, comparirà in alto di fianco al pulsante “**Annulla**” il pulsante “**Concludi Assegnazione**”, che se premuto riporterà l’utente deloitte alla schermata precedente concludendo l’assegnazione e modificando lo status in “Completato” e il pulsante sulla riga del documento in “**Visualizza Assegnazione**”. Pertanto, per quegli eventi di monitoraggio *Assegnati,* questi assumeranno lo stato “Monitoraggio OK” andando ad aggiornare tutte le sezioni della piattaforma coinvolte (ad esempio, se era l’unico monitoraggio per quel Covenant che rendeva lo stato del Covenant “Scaduto” / “ In scadenza” ecc, si aggiornerà lo stato di conseguenza).

Una volta premuto il pulsante “**Conferma Assegnazione**” sullo scadenziere rimarrà visibile solo il monitoraggio a cui è stato assegnato il compliance certificate, l’input della colonna azione diventerà “Modifica Assegnazione” e se cliccato renderà nuovamente selezionabili i selettori sopra, espanderà lo scadenziere e permetterà di riassegnare e riconfermare l’assegnazione.

L’utente Deloitte potrà poi Concludere l’assegnazione come dettagliato sopra, oppure aggiungere altri covenant con il pulsante dedicato.

Affinchè sia possibile cliccare “Concludi Assegnazione” tutti i Covenant Aggiunti devono essere stati Confermati, altrimenti il pulsante non sarà cliccabile.

#### 4.2.3 Validazione Compliance Certificate - Processo GenAI

Quando un documento è validato, Deloitte deve assegnarlo al covenant specifico. Questo processo collega il documento al monitoraggio del covenant.

Quando l’utente Deloitte clicca su “Valida” per un documento, compare il pulsante “Assegna Monitoraggi” che abilita il processo di assegnazione dell’Utente Deloitte.

L’Utente Deloitte viene indirizzato alla schermata di Assegnazione composta da:

* Preview del documento di Compliance Certificate
* Box di selezione tra “**Assegnazione con GenAI”** e “**Assegnazione manuale**”
* Pulsante per gestione bad flow “Ritorna a documentazione incompleta” che modifica la validazione del documento considerandolo come rifiutato
  + Quando cliccato, si apre un modal che consente a Deloitte di specificare il motivo del rifiuto - Deloitte deve inserire un motivo obbligatorio (es. “Documento incompleto”, “Formato non corretto”, “Firma mancante”) - Deloitte può aggiungere note dettagliate sul rifiuto (facoltative) - Una notifica viene inviata a ISP per informarlo del rifiuto e del motivo - ISP può ricaricare una versione corretta del documento

Nel caso di “Assegnazione con GenAi” l’utente visualizzerà uno spinner mentre l’estrazione GenAI avrà luogo.

Dopo il completamento dell’operazione compariranno N Selettori di tipologia covenant, sotto ad ognuno sarà presente lo scadenziere di riferimento con la proposta di assegnazione fatta dalla GenAI, e ancora sotto saranno presenti due Pulsanti: “**Modifica Dati**” e “**Conferma Assegnazione**”

“**Modifica Dati**” permetterà di interagire con la tabella inserendo i dati puntuali, modificando l’assegnazione (o anche il covenant di riferimento, in caso di errore) e gestendo l’inserimento con le medesime modalità dell’assegnazione manuale.

“**Conferma Assegnazione**”permette di confermare l’assegnazione (indipendentemente dal fatto che sia stato modificato quanto estratto da GenAI o meno) e proseguire con la procedura.

Come nel workflow manuale, il pulsante Conferma Assegnazione, una volta cliccato, diventerà “**Modifica Assegnazione**”, e se ricliccato consentirà nuovamente di modificare i dati.

Dopo aver confermato l’assegnazione, comparirà in basso un divider orizzontale, con sotto il testo “L’assegnazione è confermata, vuoi concludere la procedura di assegnazione o aggiungere un altro covenant?”, e due pulsanti “Aggiungi Covenant #n” e “Concludi assegnazione”

Il primo pulsante aggiunge un nuovo selettore a lista per gestire un altro covenant con le medesime modalità, il secondo Chiude la procedura di assegnazione, modifica lo stato del documento in “Report Generato” e riporta l’utente deloitte alla schermata di gestione dei compliance certificate

#### 4.2.4 Modifica Assegnazione Compliance Certificate

Dalla sezione compliance certificate, l’utente deloitte può cliccare sul bottone “Visualizza Assegnazione” di qualsiasi documento già gestito: l’utente accede alla schermata di assegnazione, dove visualizzerà tutti i covenant, i rispettivi selettori e la riga dello scadenziere su cui è stata effettuata l’assegnazione: premendo “Modifica Assegnazione” sulla colonna azione. L’utente potrà quindi andare a modificare qualsiasi assegnazione (e relativo Covenant, anche rimuovendolo) con il medesimo flusso descritto in precedenza, per poi chiudere nuovamente la procedura di assegnazione dopo aver confermato l’assegnazione di tutti i singoli covenant.

#### 4.2.5 Modifica Dati Covenant - Aggiornamento Parametri

Deloitte ha la possibilità di modificare i dati degli eventi di monitoraggio dei covenant quando necessario. Questa funzionalità consente di aggiornare i parametri dei covenant (es. valore limite, periodicità) in caso di cambiamenti contrattuali.

**Accesso alla Modifica:**

Un pulsante “**Modifica Dati Covenant**” nella sezione di dettaglio del singolo Covenant consente a Deloitte di accedere alla pagina di modifica dei dati dei covenant. Se il pulsante viene premuto l’utente entra in modalità modifica, e può interagire come dettagliato sotto: il pulsante viene inoltre sostituito da due pulsanti “**Salva**” e “**Annulla**” e compare anche il pulsante “**Aggiungi Monitoraggio**”

**Campi Modificabili:** 1. **Tipologia Covenant:** Il tipo di covenant (es. “Leverage Ratio”) - Dropdown 2. **Valore Limite:** Il valore limite del covenant (es. “< 70”) - Campo testo 3. **Covenant Variabile:** Se il covenant è variabile (SI/NO) - Dropdown

**Valori Monitoraggi Modificabili:** l’utente Deloitte può inoltre modificare, dopo aver cliccato su “Modifica dati Covenant” i valori di qualsiasi evento di monitoraggio futuro, modificandone le scadenze, i valori soglia, le date di riferimento.

**Aggiungi Monitoraggio:** l’utente Deloitte può inoltre aggiungere una riga per un monitoraggio futuro affinchè si collochi correttamente nello scadenziere deve inserire necessariamente: Data di Riferimento, Scadenza, Segno Valore Limite e Valore Limite

**Rimozione manuale di un monitoraggio:** è possibile in caso di necessità rimuovere manualmente un evento di monitoraggio eliminando il valore “Data di riferimento” e salvando le modifiche

**Pulsante “Salva”:** - Quando cliccato, le modifiche sono salvate nel sistema lo stato del covenant potrebbe cambiare se i parametri sono stati modificati

**Pulsante “Annulla”:** - Quando cliccato, le modifiche non sono salvate e l’utente torna alla pagina precedente

**Validazione Dati:** - **Valore Limite:** Deve essere in formato valido (es. “< 70”, “> 50”)

- **Data Scadenza:** Deve essere una data valida

#### 4.2.6 Gestione Eccezioni (Covenant - Deloitte)

Quando ISP presenta una richiesta di eccezione per un covenant, Deloitte può prenderne visione, cliccando sull’azione “Visualizza Eccezione” dove presente. Dopo aver cliccato si aprirà una finestra modale da cui Deloitte potrà esaminare la motivazione inserita da ISP e scaricare eventuali documenti a supporto, nonché annullare la richiesta di eccezione con un pulsante dedicato “Annulla Eccezione” da utilizzare solo su esplicita richiesta di ISP per gestire il bad flow in cui un’eccezione è stata richiesta per il monitoraggio errato

## 5. SCADENZIERE UNIFICATO

### 5.0 Descrizione Generale e Obiettivi

La sezione “Scadenziere” rappresenta un modulo della piattaforma Agency Desk che fornisce una vista consolidata e cronologica di tutti gli eventi di monitoraggio dei covenant di una pratica specifica. Lo scadenziere consente agli utenti di visualizzare in un’unica tabella tutti i monitoraggi ordinati per data, facilitando la pianificazione e il tracciamento delle scadenze.

L’obiettivo principale della sezione “Scadenziere” è consentire agli utenti di:

1. **Visualizzare Cronologia Consolidata:** Vedere tutti gli eventi di monitoraggio (covenant) in un’unica vista ordinata per data
2. **Filtrare per Tipologia:** Filtrare gli eventi per tipologia specifica (es. Leverage Ratio, Bilancio, etc.)
3. **Tracciare Scadenze:** Monitorare le scadenze imminenti e i monitoraggi completati
4. **Accedere ai Dettagli:** Visualizzare informazioni complete su ogni evento di monitoraggio

**Accesso:** Scheda “Scadenziere” nel dettaglio della pratica (dopo Covenant e Spread)

### 5.1 Pagina Scadenziere - Vista Consolidata

#### 5.1.1 Descrizione Generale

Quando l’utente accede alla scheda “Scadenziere”, visualizza una pagina dedicata alla cronologia consolidata di tutti gli eventi di monitoraggio per la pratica.

**Titolo Pagina:** “Scadenziere Pratica XXX” (es. “Scadenziere Pratica 002”)

**Pulsante Torna Indietro:** Un pulsante “Torna alla pratica” che consente di tornare al dettaglio della pratica principale

#### 5.1.2 Sezione Filtri

La sezione di filtri consente agli utenti di restringere l’elenco degli eventi di monitoraggio secondo vari criteri.

**Campi di Filtro Disponibili:**

**Filtro Tipologia:** Un dropdown che consente di filtrare per tipologia specifica: - Tutti (mostra tutte le tipologie) - Leverage Ratio - Gearing Ratio - Interest Coverage Ratio - Debt Service Coverage Ratio - Bilancio Annuale Certificato - Bilancio Semestrale - Report Mensile - Altro

**Pulsante “Applica Filtri”:** - Quando cliccato, la tabella è aggiornata per mostrare solo gli eventi che corrispondono ai filtri selezionati

**Pulsante “Resetta”:** - Quando cliccato, tutti i filtri sono cancellati e la tabella mostra tutti gli eventi

### 5.2 Tabella Scadenziere

#### 5.2.1 Descrizione della Tabella

La tabella principale mostra tutti gli eventi di monitoraggio dei covenant della pratica, ordinati cronologicamente.

**Colonne della Tabella:**

1. **Tipologia:** Il tipo specifico dell’evento (es. “Leverage Ratio”, “Gearing Ratio”)
2. **ID Evento di Monitoraggio:** L’identificativo dell’evento di monitoraggio: ID costruito da ID Pratica di monitoraggio + ID Covenant + ID Evento (Esempio il quarto monitoraggio del secondo Covenant della Terza pratica sarà M3C2E4)
3. **Data di Riferimento:** La data a cui si riferisce il monitoraggio (es. “14/02/2025”)
4. **Scadenza Monitoraggio:** La data entro cui il monitoraggio deve essere completato (es. “14/03/2025”)
5. **Giorni al Prossimo Monitoraggio:** Il numero di giorni rimanenti fino alla scadenza (es. “10”) o il numero di giorni di ritardo (es. “-5”) rispetto alla data corrente
6. **Data Avvenuto Monitoraggio:** La data in cui il monitoraggio è stato completato (es. “11/03/2025”) o vuoto se non ancora completato
7. **Status:** Lo stato attuale dell’evento, visualizzato con un indicatore colorato:
   * Verde (●) = “Monitoraggio OK”
   * Giallo (●) = “Monitoraggio In Scadenza”
   * Giallo (●) = “Monitoraggio da attenzionare”
   * Rosso (●) = “Monitoraggio Scaduto”
   * Rosso (●) = “Monitoraggio Fuori Soglia”
8. **Documento di Monitoraggio:** Link per scaricare il documento associato all’evento (es. “Download”) o vuoto se non disponibile
9. **Azioni:** Pulsanti di azione:
   * **Visualizza Eccezione:** Consente di visualizzare le motivazioni dell’eccezione del monitoraggio se censite da ISP

#### 5.2.2 Comportamento della Tabella

**Ordinamento:** La tabella è ordinata per “Scadenza Monitoraggio” in ordine crescente (scadenze più prossime in alto)

**Righe per Pagina:** La tabella mostra 20 righe per pagina per impostazione predefinita

**Paginazione:** Controlli di paginazione “<< Pagina X >>” consentono di navigare tra le pagine

**Evidenziazione Riga:** Al passaggio del mouse su una riga, la riga è evidenziata con un colore di sfondo leggermente più scuro

**Monitoraggio Evidenziato:** Viene evidenziato il prossimo monitoraggio

**Viste differenti:** la sola differenza è che nella colonna azioni l’utente Deloitte vedrà soltanto “Visualizza Eccezione” per quei monitoraggi già gestiti, ma non avrà a disposizione le azioni “Salta Monitoraggio” e “Ignora Soglia”

#### 5.2.3 Esempio di Dati nella Tabella

| Tipologia | ID Evento | Data Riferimento | Scadenza | Giorni Prossimo Monitoraggio | Data Avvenuto | Status | Documento | Azioni |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Leverage Ratio | M2C4E1 | 14/02/2025 | 14/03/2025 | - | 11/03/2025 | ● OK | Download | - |
| Leverage Ratio | M2C4E 2 | 14/05/2025 | 14/06/2025 | -5 | 11/06/2025 | ● Scaduto | Download | . |
| Leverage Ratio | M2C4E 3 | 14/08/2025 | 14/09/2025 | - | 11/09/2025 | ● OK | Download | Visualizza Eccezione |
| Leverage Ratio | M2C4E 4 | 14/11/2025 | 14/12/2025 | - | 11/12/2025 | ● OK | Download | - |
| Leverage Ratio | M2C4E 5 | 14/02/2025 | 14/03/2025 | 8 | - | ● In Scadenza | - | - |
| Bilancio Annuale | M2C5E 1 | 14/02/2025 | 14/03/2025 | - | 11/03/2025 | ● OK | Download | - |
| Bilancio Annuale | M2C5E 2 | 14/05/2025 | 14/06/2025 | -12 | 11/06/2025 | ● Scaduto | Download | - |
| Bilancio Annuale | M2C5E 3 | 14/08/2025 | 14/09/2025 | - | 11/09/2025 | ● OK | Download | - |

## 6. AGGIORNAMENTO MAILING LIST

### 6.1 Descrizione Generale e Obiettivi

La sezione “Aggiornamento Mailing List” rappresenta un modulo dedicato della piattaforma Agency Desk che consente agli utenti di gestire e aggiornare le liste di contatti associate alle pratiche di monitoraggio. Questa funzionalità è essenziale per garantire che le comunicazioni relative ai monitoraggi, alle scadenze e alle eccezioni raggiungano i destinatari corretti e aggiornati.

L’obiettivo principale della sezione “Aggiornamento Mailing List” è consentire agli utenti di:

1. **Ricercare Pratiche:** Trovare rapidamente una pratica specifica per la quale aggiornare la mailing list
2. **Visualizzare Contatti Attuali:** Vedere i contatti attuali associati alla pratica
3. **Aggiungere Nuovi Contatti:** Aggiungere nuovi indirizzi email alla mailing list
4. **Rimuovere Contatti:** Rimuovere contatti obsoleti o non più necessari
5. **Modificare Contatti:** Aggiornare gli indirizzi email o i dettagli dei contatti esistenti

**Accesso:** Menu Laterale > “Aggiornamento Mailing List”

### 6.2 Pagina Ricerca Pratica - Selezione della Pratica

#### 6.2.1 Descrizione Generale

La pagina “Aggiornamento Mailing List” inizia con una sezione dedicata alla ricerca e selezione della pratica per la quale aggiornare la mailing list. Questa pagina consente agli utenti di trovare rapidamente la pratica desiderata tra tutte le pratiche disponibili nel sistema.

**Titolo Pagina:** “Aggiornamento Mailing List”

**Sottotitolo:** “Benvenuto nella sezione «Aggiornamento Mailing List»

Da questa schermata puoi aggiornare i dati dei contatti di qualsiasi pratica caricata in piattaforma”

#### 6.2.2 Sezione Richiedi Aggiornamento Mailing List

La sezione individua pratica consente agli utenti di trovare una pratica specifica utilizzando vari criteri di filtro.

**Campi disponibili:**

**Filtro Nome Deal:** Un campo di ricerca che consente di cercare la pratica per nome Deal

**Filtro ID Censimento Pratica:** Un campo di ricerca con autocomplete per cercare per ID Censimento (es. “Prat001”, “Prat002”).

**Filtro Tracking ID:** Un campo di ricerca con autocomplete per cercare per ID di tracciamento della pratica (es. “001”, “002”, “003”).

**Filtro ID Monitoraggio Pratica:** Un campo di ricerca con autocomplete per cercare per ID della pratica (es. “Prat001”, “Prat002”).

#### 6.2.3 Tabella Pratiche per Aggiornamento Mailing List

La tabella mostra i risultati della ricerca con tutte le pratiche disponibili per l’aggiornamento della mailing list.

**Colonne della Tabella:**

1. **ID Monitoraggio Pratica:** L’identificativo univoco della pratica: Lettera M seguita da numero ordinale da 1 a 999 (e.g. la terza pratica aperta sarà “P3”)
2. **ID Censimento Pratica:** L’identificativo univoco del censimento della pratica
3. **Tracking ID:** Il numero di tracciamento (es. “001”)
4. **Nome Deal:** Il nome della pratica/finanziamento (es. “Banca Nuova S.p.A”)
5. **Utente Banca:** L’email dell’utente ISP responsabile (es. “Marioverdi@isp-it”)
6. **Utente Deloitte:** l’email dell’Utente Deloitte a cui è assegnata la pratica
7. **Tipologia Pratica :** la tipologia della pratica (es. “Banca Agente”)
8. **Data Creazione Pratica:** la data in cui la pratica è stata creata sulla piattaforma
9. **Data Fine Monitoraggio:** la data oltre cui la pratica verrà considerata chiusa corrispondente alla Termination date
10. **Ultimo Aggiornamento Mailing list:** La data dell’ultimo aggiornamento della mailing list (es. “15/03/2026”)
11. **Azione:** Un pulsante “Richiedi Aggiornamento Mailing List” che attiva il popup di aggiornamento mailing list, in cui è possibile caricare un AD form su cui effettuare l’estrazione

**Comportamento e Interattività:**

**Ordinamento:** Tutte le colonne sono ordinabili. Cliccando sull’intestazione di una colonna, la tabella si ordina per quella colonna in ordine crescente (A-Z) o decrescente (Z-A).

**Righe per Pagina:** La tabella mostra 10 righe per pagina per impostazione predefinita, ma gli utenti possono configurare questo valore a 25, 50 o 100 righe per pagina.

**Paginazione:** Controlli di paginazione “<< Pagina X >>” consentono di navigare tra le pagine.

**Evidenziazione Riga:** Al passaggio del mouse su una riga, la riga è evidenziata con un colore di sfondo leggermente più scuro per migliorare l’usabilità.

### 6.3 Pagina Aggiornamento Mailing List - Gestione Contatti

#### 6.3.1 Descrizione Generale

Quando l’utente clicca su “Aggiorna Mailing List” per una pratica specifica, si apre un modal di caricamento dell’AD Form aggiornato, una volta caricato l’AD Form la pratica viene spostata nella sezione “Pratiche per cui è stato richiesto aggiornamento mailing List” posta sotto alla sezione “Richiedi aggiornamento Mailing List”

#### 6.3.2 Sezione Pratiche per cui è stato richiesto aggiornamento mailing List

Una sezione che mostra le informazioni principali della pratica per la quale si sta aggiornando la mailing list che conterrà le stesse colonne della tabella sopracitata, con aggiunta la seguente colonna (a sinistra prima della colonna “**Azione**”)

* **Stato aggiornamento** – Lo stato del processo di aggiornamento della mailing List (Documentazione Incompleta, Documentazione Completa, Documentazione Confermata, Verifica Dati in corso, Report Generato)

In questa sezione, inoltre, la colonna azione conterrà quanto descritto sotto:

* **Azione** - “Visualizza Richiesta aggiornamento” cliccando “Visualizza Richiesta Aggiornamento” si aprirà un pop up da cui l’utente ISP potrà visualizzare il box di avanzamento del documento caricato oppure modificare il file caricato, fino a che lo User Deloitte non lo avrà confermato (dallo stato “Doc Confermata” in poi il pulsante “Annulla richiesta” Sarà inibito, Inoltre, una volta che lo status è passato in “Report Generato” la colonna presenterà l’azione descritta sotto
* **Azione –** Visualizza Report – Visualizza e scarica il report della mailing list aggiornata

#### 6.3.3 Vista Deloitte

Quando l’utente Deloitte accede alla sezione, visualizzerà la tabella descritta sopra per la sezione “**Pratiche per cui è stato richiesto aggiornamento Mailing List**”, la cui azione disponibile sarà “**Aggiorna**”, cliccando aggiorna l’utente deloitte verrà indirizzato alla schermata contatti attuali. Nella schermata sarà presente una barra contenente la documentazione caricata (AD Form) assieme a un tag di status (In lavorazione), a un contatore di giorni in giacenza Stato, a due pulsanti “Conferma” e “Rifiuta”, a un’icona di download e a una di commento.

Se l’AD Form viene rifiutato, la richiesta di aggiornamento tornerà in status “Documentazione Incompleta”

Se l’AD Form viene accettato, lo stato passera a “Documentazione Confermata” comparirà sotto alla riga documento dell’AD form un pulsante “Estrai con GenAI”

Le funzionalità, gli stati e le modalità di estrazione GEN AI rimangono invariate rispetto a quanto fatto nella fase di AD Form

In fondo alla schermata sarà presente, inoltre, il pulsante “Conferma e Aggiorna” che conclude l’aggiornamento e passa lo status della richiesta (che resterà visibile sia da ISP che da Deloitte nella sezione “**Pratiche per cui è stato richiesto aggiornamento mailing List”** in “Fase Completata”, modificando contestualmente anche l’azione disponibile in “Scarica Report” e permettendo di scaricare la mailing list aggiornata

## 7. SEZIONE SPREAD - GESTIONE SPREAD FINANZIARI

### 7.1 Descrizione Generale e Obiettivi

La sezione “Spread” rappresenta un modulo specializzato della piattaforma Agency Desk che consente la gestione e il monitoraggio degli spread finanziari associati ai covenant di una pratica. Lo spread è il margine di interesse applicato al tasso base del finanziamento e può variare in base al rispetto dei covenant finanziari.

L’obiettivo principale della sezione “Spread” è consentire agli utenti di:

1. **Visualizzare la Tabella Spread (ISP):** ISP può visualizzare la tabella di spread configurata da Deloitte per comprendere come i covenant influenzano il tasso di interesse
2. **Creare la Tabella Spread (Deloitte):** Deloitte può decidere se includere la sezione Spread per una pratica e creare la tabella di spread da zero
3. **Customizzare la Tabella Spread (Deloitte):** Deloitte può personalizzare completamente la tabella inserendo i dati finanziari desiderati e collegandola ai covenant di riferimento
4. **Tracciare le Variazioni (Entrambi):** Sia ISP che Deloitte possono visualizzare lo storico delle variazioni di spread nel tempo
5. **Analizzare l’Impatto (Entrambi):** Comprendere come le variazioni nei covenant influenzano gli spread applicati

**Accesso:** Scheda “Spread” nel dettaglio della pratica (a destra della sezione “Covenant”)

### 7.2 Visibilità della Sezione Spread

La sezione Spread risulta visibile ad ISP solo dopo che l’Utente Deloitte ha consolidato la tabella

### 7.3 Creazione della Tabella Spread (Deloitte)

#### 7.3.1 Sezione Spread

Quando l’utente Deloitte atterra sulla sezione “Spread" Visualizzerà i seguenti elementi

#### 7.3.2 Sezione Informazioni Pratica

Una sezione che mostra le informazioni principali della pratica per la quale si sta creando la tabella di spread.

**Informazioni Visualizzate:**

- **Nome Deal:** Il nome della pratica (es. “Banca Nuova S.p.A”)

- **ID Pratica Monitoraggio:** L’identificativo univoco (es. M1)

– **Tipologia Pratica:** la tipologia della pratica (es. “Banca Agente”)

- **Utente Banca:** L’email dell’utente ISP responsabile (es. “Marioverdi@isp-it”)

- **Stato Pratica:** Lo stato della pratica

#### 7.3.3 Disclaimer

**Disclaimer di verifica della sezione** Sopra ai pulsanti delle sezioni pratica sarà presente un disclaimer in arancione chiaro, con simbolo di warning contenente il testo “La tabella dello Spread per la pratica non è stata verificata”, tale disclaimer permarrà finchè non sarà stata censita la tabella riferita allo Spread (tramite GEN AI o Manualmente)

#### 7.3.4 Sezione Tabella Spread (Inserimento Manuale)

Titolo: Dettaglio Pratica XXXX | Spread (con XXX compilato con l’ID Pratica di Monitoraggio)

Saranno presenti i seguenti elementi:

Sezione contenente il documento del contratto di finanziamento del Deal recuperato dalla pratica di censimento collegata (check su Tracking ID), con un pulsante “Visualizza” e pulsanti di azione “Download” e “Commenta”.

**Pulsante “Crea Tabella”:** che accede alla pipeline di lavorazione manuale descritta in seguito per la creazione/modifica della tabella

**Pulsante “Verifica e salva”:** Che conferma e salva quanto estratto da GenAI/Modificato/creato dall’utente Deloitte confermandolo in via definitiva come tabella di riferimento

**Pulsante “Nessuna tabella da inserire”:** Conferma che non è necessario seguire lo spread per questa pratica e disabilita la sezione di riferimento

#### 7.3.5 Creazione tabella

Nel caso in cui l’Utente Deloitte premesse il pulsante “Nessuna tabella da inserire”, ciò significa che all’interno del contratto di finanziamento non sono presenti tabelle che indicano una variazione dello spread (o nel tempo o per intervalli definiti rispetto al valore di un covenant) e pertanto verrà rimosso il disclaimer e mostrato il messaggio “Assenza Tabella Spread”

Tuttavia, nel caso l’utente selezionasse “Crea Tabella” l’utente verrà portato alla schermata di creazione tabella.

**TABELLA COVENANT**

Premendo il pulsante **Tabella Covenant**, al di sotto della sezione contenente il documento del Contratto di Finanziamento saranno presenti 3 pulsanti e una tabella con le seguenti caratteristiche:

- Pulsante “Aggiungi Riga”, che se premuto aggiungerà una riga alla tabella sottostante

- Pulsante “Valida tabella”, disinibito fintanto che tutti i campi della tabella sottostante siano stati compilati

- Pulsante “Modifica Tipologia”, che permette di tornare alla schermata precedente, in cui sarà possibile selezionare Nessuna tabella da inserire” che se premuto farà visualizzare quanto previsto dal paragrafo precedente)

- Tabella strutturata dalle seguenti colonne:

* **ID**, che sarà compilato con un numero progressivo a seconda del numero di righe censite (a partire dall’”1”)
* **Covenant**, da compilare per ciascuna riga censita con una selezione di un menu a tendina tutti i covenant presenti sulla pratica (Quindi un subset dell’elenco completo di tipologie).Ogni covenant sarà nominato seguendo questa logica **“[ID del covenant] – [Tipologia del Covenant]”.**
* **Valore Limite inferiore**, da compilare per ciascuna riga censita inserendo un valore numerico (comprendente anche i caratteri come “,”)
* **Valore Limite superiore**, da compilare per ciascuna riga censita inserendo un valore numerico (comprendente anche i caratteri come “,”)
* **Valore Margine (%)**, da compilare per ciascuna riga censita inserendo un valore numerico (comprendente anche i caratteri come “,”)
* **Azione**, che permetterà, una volta compilati i vari campi della riga di andarli a modificare (se premuto appariranno i classici pulsanti “Conferma” e “Annulla” come nelle tabelle di validazione delle informazioni dei covenant estratte dalla GEN AI)
* **Icona “Cestino”** che permetterà di eliminare la riga (anche se completamente compilata)

Una volta che l’Utente Deloitte abbia censito tutti i record, premerà sul pulsante “Valida tabella”, il Disclaimer in alto scomparirà e la Piattaforma mostrerà la tabella con i vari record con sfondo grigio (senza l’icona del cestino e senza la colonna “Azioni”).

A valle di ciò, al di sotto della tabella apparirà una scritta “Storico Spread” sotto cui sarà valorizzata una tabella per monitorare l’andamento dello spread per la quale verrà aggiunta una riga ogni qual volta venisse monitorato il covenant corrispondente al covenant censito.

La tabella che si creerà sarà caratterizzata dalle seguenti colonne:

* Data Riferimento 🡪 Nel quale verrà indicata la data di riferimento del valore controllato. La tabella sarà ordinata di default secondo questa data, in ordine decrescente.
* Data avvenuto monitoraggio 🡪 Nel quale verrà indicata la data in cui è stato monitorato il covenant corrispondente
* Valore Monitorato 🡪 Nel quale verrà indicato il valore del covenant monitorato relativo al covenant corrispondente
* Cluster di Riferimento 🡪 Sulla base del valore del covenant monitorato, la piattaforma dovrà verificare in quale degli ID censiti nella tabella sopra il Valore Monitorato ricada e riportare l’ID corrispondente
* Valore Margine (%) 🡪 Nel quale verrà indicato il Valore Margine (%) della tabella sopra relativo all’ID censito nel campo “Cluster di Riferimento”

Pertanto, ogni qual volta venisse monitorato il covenant di riferimento la tabella verrà arricchita con una nuova riga (che si posizionerà in cima alla lista) e sarà evidenziata. I precedenti record invece scaleranno verso il basso e saranno evidenziati in grigio chiaro.

Al censimento di una nuova riga verrà inviata apposita notifica all’Utente Banca di riferimento.

#### 7.3.6 Sezione Spread - Vista ISP

La vista ISP risulta essere analoga alla vista Deloitte a valle dell’azione di validazione della tabella (o clic sul pulsante “Nessuna tabella da inserire”) e seguirà le medesime logiche descritte nel paragrafo precedente. Fintanto che Deloitte non proceda a censire la tabella o confermare che questa non sia presente, l’Utente Banca vedrà al di sotto del Titolo il seguente messaggio “Verifica Spread Pratica”

## 8. DASHBOARD MONITORAGGIO (ISP)

### 8.1 Descrizione Generale e Obiettivi

La Dashboard Monitoraggio rappresenta il punto di ingresso principale del workflow di monitoraggio della Piattaforma Agency Desk. È concepita come una pagina di riepilogo strategico che fornisce agli utenti (sia ISP che Deloitte) una **visione aggregata** e immediata dello **stato di tutte le pratiche di monitoraggio attive**

L’obiettivo principale della dashboard è consentire agli utenti di **identificare** rapidamente le **situazioni critiche** (**monitoraggi scaduti**), le **situazioni di attenzione** (**monitoraggi in scadenza**), e avere una panoramica complessiva della salute del portafoglio di pratiche in gestione. La dashboard deve essere intuitiva, visivamente chiara e permettere una navigazione rapida verso i dettagli specifici delle pratiche.

### 8.2 Sezione Monitoraggio - Grafico a Ciambella (Donut Chart)

#### Descrizione Funzionale

La sezione “Monitoraggio” della dashboard, raggiungibile attraverso il selettore posto in alto a destra (Censimento / Monitoraggio) presenta in alto a sinistra un **grafico a ciambella** (donut chart) che rappresenta visivamente la distribuzione dello stato di tutti i monitoraggi nel sistema. Questo grafico fornisce una visione d’insieme immediata della salute complessiva del portafoglio, permettendo agli utenti di comprendere a colpo d’occhio quante pratiche si trovano in ciascuno stato.

Il grafico è suddiviso in quattro segmenti colorati, ognuno rappresentante una categoria di stato:

**Segmento Verde (OK):** Rappresenta le pratiche in cui tutti i monitoraggi sono stati completati e sono conformi alle scadenze. Questo è lo stato desiderato e indica che non siano richieste azioni immediate. Le pratiche sono Ok solo quando tutti i monitoraggi di tutti i covenant hanno status Verde (OK)

**Segmento Giallo (In Scadenza/Da attenzionare):** Rappresenta le pratiche che presentano monitoraggi in scadenza (7 giorni dalla data di scadenza). In tale segmento rientrano anche le pratiche aventi almeno 1 covenant numerico con valore nell’intorno del 10% del valore soglia di rottura del covenant e/o un valore che si discosta di un 30% dal valore soglia di rottura del covenant ed è differito del 40% rispetto alla rilevazione precedente

**Segmento Rosso (KO/Scaduto):** Rappresenta le pratiche che presentano almeno un monitoraggio scaduti e/o, nel caso dei covenant, sono stati monitorati ma il valore monitorato non rispetta il valore limite estratto dal contratto di finanziamento

**Segmento Grigio (Concluse):** Rappresenta le pratiche il cui periodo di monitoraggio è terminato e non richiedono ulteriori azioni.

Al centro del donut chart è visualizzato il numero totale di monitoraggi (es. “119 Totale”), che rappresenta la somma di tutti i segmenti.

#### Requisiti Tecnici e Comportamentali

I **dati del donut chart devono aggiornarsi in tempo reale** quando cambiano gli stati dei monitoraggi nel sistema. Se un utente carica un documento che completa un monitoraggio, il segmento verde deve aumentare e il segmento giallo o rosso deve diminuire di conseguenza. Questo aggiornamento deve avvenire senza richiedere un refresh manuale della pagina.

### 8.3 Sezione COVNO - Informazioni di Sincronizzazione

#### Descrizione Funzionale

Nella dashboard sono presenti due card informative che mostrano lo stato di sincronizzazione con il database esterni: “COVNO”. Questi database sono fonti esterne che contengono rispettivamente gli obblighi informativi e i covenant con i loro parametri di riferimento.

Ogni card visualizza:

- **Titolo della card:** “COVNO”

- **Data ultimo aggiornamento:** Mostra quando i dati sono stati sincronizzati per l’ultima volta, nel formato DD/MM/YYYY (es. “10/03/2026”)

- **Pulsante “Aggiorna”:** Consente all’utente di procedere al caricamento manuale dei file DB obblighi e COVN0

- **Pulsante “Storico”:** Consente di visualizzare la cronologia di tutti gli aggiornamenti precedenti, con date e orari di caricamento

#### Comportamento e Logica

Attraverso l’utilizzo del pulsante “**Aggiorna**”, la piattaforma mette a disposizione dell’utente Deloitte/ISP un pop-up per il caricamento dei file di scarico proveniente dai sistemi ISP. Tali file rappresentano la fonte di alimentazione della sezione Monitoraggio delle pratiche in piattaforma.

Nel dettaglio, la piattaforma dovrà raccogliere i dati riportati nei file caricati e associarli a ciascuna pratica e ai relativi obblighi e covenant aggiornandone i valori (es. scadenza, periodicità,…). La chiave per il collegamento tra file e pratiche in piattaforma è rappresentata dal Tracking ID.

Gli step previsti per l’aggiornamento dei valori di Monitoraggio di obblighi e covenant è il seguente:

1. Caricamento file provenienti da sistemi ISP (DB Obblighi e COVN0)
2. Verifica della piattaforma, tramite Tracking ID, delle pratiche che necessitano di un aggiornamento dei dati
3. Aggiornamento dati di monitoraggio per le pratiche risultanti dal controllo automatico del punto 2)

### 8.4 Tabella Monitoraggi Scaduti

#### Descrizione Funzionale

Nella dashboard è presente una tabella dedicata ai “**Monitoraggi Scaduti**”, che rappresenta la situazione più critica: **singoli obblighi informativi o covenant** (contenuti nelle differenti pratiche di monitoraggio) che **hanno** **superato la loro data di scadenza** senza essere stati monitorati o quei c**ovenant che sono stati monitorati ma presentano un valore monitorato non in linea** rispetto al valore target estratto dal Contratto di finanziamento. Questa tabella è posizionata subito sotto al grafico a donut per attirare l’attenzione degli utenti su situazioni che richiedono azione immediata.

La tabella include una descrizione introduttiva: “*Nella tabella seguente hai visibilità dei monitoraggi delle pratiche che risultano scadute/fuori soglia in piattaforma. Per visualizzare l’elenco completo puoi consultare la pagina dedicata*”, seguita da un link “Vedi tutte” che naviga alla pagina completa dei monitoraggi scaduti.

#### Colonne e Dati

La tabella contiene le seguenti colonne, in questo ordine:

1. **ID Pratica di Monitoraggio:** L’identificativo univoco della pratica, per esempio “M1” come definito nelle sezioni precedenti
2. **Tracking ID:** ID univoco dei sistemi ISP per riconoscere l’operazione di riferimento, dato recuperato dal medesimo campo riferito alla relativa pratica di censimento
3. **Nome Deal:** Il nome della pratica/finanziamento (es. “Banca Nuova S.p.A”), dato recuperato dal medesimo campo riferito alla relativa pratica di censimento (individuabile tramite Tracking ID)
4. **Utente Banca:** L’email dell’utente ISP responsabile (es. “Marioverdi@isp-it”), dato recuperato dal medesimo campo riferito all’utente Banca associato alla relativa pratica di censimento (individuabile tramite Tracking ID)
5. **Utente Deloitte:** L’email dell’utente Deloitte a cui è stata assegnata alla pratica madre di monitoraggio (es. mverdi@deloitte.it), assegnato appositamente tramite la funzionalità presente nel pannello ADMIN (nel caso un utente Deloitte non fosse assegnato alla pratica, il primo utente Deloitte che accede alla pratica verrà assegnato alla stessa)
6. **Tipologia Pratica:** La tipologia della pratica (es. “Banca Agente”, “SACE Agent”), dato recuperato dal medesimo campo riferito alla tipologia pratica relativa alla pratica di censimento (individuabile tramite Tracking ID)
7. **Tipologia:** Il tipo specifico di covenant (es. “Leverage Ratio”, “Gearing Ratio”)
8. **ID Evento di Monitoraggio:** L’identificativo dell’evento di monitoraggio: ID costruito da ID Pratica di monitoraggio + ID Covenant + ID Evento (Esempio il quarto monitoraggio del secondo Covenant della Terza pratica sarà M3C2E4)
9. **Data di riferimento:** La data a cui si riferisce il monitoraggio (es. “14/02/2025”)
10. **Scadenza Monitoraggio:** La data entro cui il monitoraggio doveva essere completato (es. “14/03/2025”)
11. **Giorni al prossimo monitoraggio:** Il numero di giorni di ritardo rispetto alla scadenza, indicato in negativo (es. “-10” giorni) in quanto riferito a monitoraggi scaduti (nel caso di covenant fuori soglia tale campo sarà visualizzato con un “-“)
12. **Stato Evento di Monitoraggi:** Lo stato attuale, visualizzato con un punto colorato rosso (●) accanto a “Monitoraggio Scaduto” (la label sarà “Monitoraggio fuori soglia” in caso di covenant con monitoraggio non in linea rispetto al valore target estratto dal contratto di finanziamento)
13. **Azione:** Un link “Salta monitoraggio” / “Ignora Soglia” che consente all’utente di richiedere un’eccezione (funzionalità descritta successivamente)

**NB**. Le pratiche ereditano il flag “Watchlist/Dati sensibili" selezionato durante l’apertura della pratica nella fase di censimento. Tale aspetto deve essere mantenuto non solo per la presente tabella ma per tutte quelle previste nella fase di monitoraggio

#### Comportamento e Interattività

La tabella nella dashboard mostra un massimo di 5 righe per mantenere la pagina leggibile. Se ci sono più di 5 monitoraggi scaduti, è presente un controllo di paginazione “<< Pagina 1 >>” che consente di navigare tra le pagine.

Le colonne sono ordinabili: cliccando sull’intestazione di una colonna, la tabella si ordina per quella colonna. L’ordinamento predefinito è per “Giorni di ritardo” in ordine decrescente (i monitoraggi più scaduti in alto).

### 8.4.1 Pagina Monitoraggi Scaduti - Descrizione Generale e Accesso

La pagina “**Monitoraggi Scaduti**” rappresenta la vista completa e dettagliata di tutti i monitoraggi che hanno superato la loro data di scadenza senza essere stati completati o, in caso di covenant, presentano un valore del covenant monitorato non rispettante la soglia del valore limite estratto dal contratto di finanziamento. A differenza della dashboard che mostra solo i primi 5 monitoraggi scaduti, questa pagina fornisce l’elenco esaustivo di tutti i monitoraggi critici (visualizzabili gli ultimi 20 monitoraggi ordinati per scadenza, per vedere i restanti si potrà scorrere con i pulsanti posizionati al fondo della tabella con il controllo di paginazione “<< Pagina 1 >>” che consente di navigare tra le pagine), permettendo agli utenti di avere una visione completa della situazione e di gestire efficacemente le priorità.

È presente, in alto a sinistra sopra i filtri, un **contatore** che quantifica i monitoraggi scaduti totali.

Questa pagina è particolarmente utile per:

* Identificare tutte le situazioni critiche che richiedono azione immediata
* Filtrare i monitoraggi scaduti per vari criteri (utente, tipologia pratica , tipologia covenant)
* Ordinare i monitoraggi per gravità (giorni di ritardo) - Esportare i dati per analisi esterne o reporting
* Gestire le eccezioni

**Accesso:** Cliccando su “Vedi tutte” nella sezione “Monitoraggi Scaduti” della Dashboard Monitoraggio

### 8.4.2 Pagina Monitoraggi Scaduti - Sezione Filtri e Ricerca Avanzata

#### Descrizione Funzionale

La pagina “Monitoraggi Scaduti” inizia con una sezione dedicata ai filtri e alla ricerca, che consente agli utenti di restringere l’elenco dei monitoraggi scaduti secondo vari criteri

#### Campi di Filtro Disponibili

**Filtro Utente Banca:** Un campo di ricerca che consente di cercare per email dell’utente ISP Di riferimento

**Filtro Tipologia Pratica:** Un dropdown che consente di selezionare una delle due tipologie di pratiche: Banca Agente; Banca Agente e SACE Agent

**Filtro Tracking ID:** Un campo di ricerca per cercare per ID univoco censito dall’utente ISP in fase di censimento.

**Filtro Nome Deal:** un campo di ricerca per cercare il nome del Deal di riferimento

**Filtro ID Evento di Monitoraggio:** Un campo di ricerca per cercare per ID del singolo evento di monitoraggio

**Filtro Giorni al prossimo Monitoraggio (Range):** Due campi numerici che consentono di specificare un intervallo di giorni al prossimo monitoraggio (in negativo trattandosi di monitoraggi scaduti): - “Giorni al prossimo monitoraggio a partire da” (es. “-5” per mostrare solo monitoraggi scaduti da più di 5 giorni) - “Giorni al prossimo monitoraggio fino a” (es. “-20” per mostrare solo monitoraggi scaduti da non più di 20 giorni)

#### Pulsanti di Azione

**Pulsante “Applica filtri”:** Esegue la ricerca con i filtri selezionati e aggiorna la tabella sottostante

**Pulsante “Resetta”:** Cancella tutti i filtri e mostra nuovamente tutti i monitoraggi scaduti

**Pulsante “Scarica Report Pratiche”:** Esporta tutti i record della tabella in un file excel formattato esattamente come la tabella

**Pulsante “Scarica Report Monitoraggi”**: Esporta il report contenente tutti i monitoraggi, formattato come nel documento allegato: ![](data:image/x-emf;base64...)

**Pulsante “Indietro”:** Torna alla Dashboard Monitoraggio

### 8.4.3 Pagina Monitoraggi Scaduti - Tabella Monitoraggi Scaduti Completa

#### Descrizione Funzionale

La tabella principale della pagina “Monitoraggi Scaduti” fornisce una vista completa di tutti i monitoraggi che hanno superato la scadenza. A differenza della tabella nella dashboard che mostra solo 5 righe, questa tabella mostra tutti i monitoraggi scaduti (visualizzabili gli ultimi 20 monitoraggi ordinati per scadenza, per vedere i restanti si potrà scorrere con i pulsanti posizionati al fondo della tabella con il controllo di paginazione “<< Pagina 1 >>” che consente di navigare tra le pagine) con la possibilità di ordinare, filtrare e scaricare l’intera tabella.

#### Colonne e Dati

La tabella contiene le seguenti colonne (risultano le medesime delle colonne presenti nella Dashboard di monitoraggio della Tabella “Monitoraggi Scaduti”):

1. **ID Pratica di Monitoraggio:** L’identificativo univoco della pratica: Lettera M seguita da numero ordinale (e.g. la terza pratica aperta sarà “M3”)
2. **Tracking ID:** ID univoco dei sistemi ISP per riconoscere l’operazione di riferimento, dato recuperato dal medesimo campo riferito alla relativa pratica di censimento
3. **Nome Deal:** Il nome della pratica/finanziamento (es. “Banca Nuova S.p.A”), dato recuperato dal medesimo campo riferito alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
4. **Utente Banca:** L’email dell’utente ISP responsabile (es. “Marioverdi@isp-it”), dato recuperato dal medesimo campo riferito all’utente Banca associato alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
5. **Utente Deloitte:** L’email dell’utente Deloitte a cui è stata assegnata alla pratica madre di monitoraggio (es. mverdi@deloitte.it), assegnato appositamente tramite la funzionalità presente nel pannello ADMIN (nel caso un utente Deloitte non fosse assegnato alla pratica, il primo utente Deloitte che accede alla pratica verrà assegnato alla stessa)
6. **Tipologia Pratica:** La tipologia della pratica (es. “Banca Agente”, “SACE Agent”), dato recuperato dal medesimo campo riferito alla tipologia pratica relativa alla pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
7. **Tipologia:** Il tipo specifico di covenant o (es. “Leverage Ratio”, “Gearing Ratio”)
8. **ID Evento di Monitoraggio:** L’identificativo dell’evento di monitoraggio: ID costruito da ID Pratica di monitoraggio + ID Covenant + ID Evento (Esempio il quarto monitoraggio del secondo Covenant della Terza pratica sarà M3C2E4)
9. **Data di riferimento:** La data a cui si riferisce il monitoraggio (es. “14/02/2025”)
10. **Scadenza Monitoraggio:** La data entro cui il monitoraggio doveva essere completato (es. “14/03/2025”)
11. **Giorni al prossimo monitoraggio:** Il numero di giorni di ritardo rispetto alla scadenza, indicato in negativo (es. “-10” giorni) in quanto riferito a monitoraggi scaduti (nel caso di covenant fuori soglia tale campo sarà visualizzato con un “-“)
12. **Stato Evento di Monitoraggi:** Lo stato attuale, visualizzato con un punto colorato rosso (●) accanto a “Monitoraggio Scaduto” (la label sarà “Monitoraggio fuori soglia” in caso di covenant con monitoraggio non in linea rispetto al valore target estratto dal contratto di finanziamento)
13. **Azione:** Un link “Salta monitoraggio” / “Ignora Soglia” che consente all’utente di richiedere un’eccezione

#### Comportamento e Interattività

**Ordinamento:** Tutte le colonne sono ordinabili. Cliccando sull’intestazione di una colonna, la tabella si ordina per quella colonna in ordine crescente (A-Z) o decrescente (Z-A). L’ordinamento predefinito è per “Giorni al prossimo monitoraggio” in ordine crescente (i monitoraggi più scaduti in alto).

**Righe per Pagina:** La tabella mostra 20 righe per pagina.

**Paginazione:** Controlli di paginazione “<< Pagina X >>” consentono di navigare tra le pagine.

**Evidenziazione Riga:** Al passaggio del mouse su una riga, la riga è evidenziata con un colore di sfondo leggermente più scuro per migliorare l’usabilità.

**Colore di Sfondo Riga:** Le righe con “Giorni di ritardo” > 30 giorni hanno uno sfondo rosso chiaro per evidenziare le situazioni più critiche.

**Riga Selezionabile:** Cliccando su una riga della tabella, l’utente naviga al dettaglio della pratica, dove può visualizzare i dettagli completi del monitoraggio scaduto.

### 8.5 Tabella Monitoraggi in Scadenza

#### Descrizione Funzionale

Sotto alla tabella dei monitoraggi scaduti, la dashboard presenta una tabella dedicata ai “Monitoraggi in Scadenza”, che rappresenta situazioni che richiedono attenzione (in scadenza entro una settimana dalla data attuale) ma non sono ancora critiche.

La tabella include una descrizione introduttiva: “Nella tabella seguente hai visibilità dei monitoraggi delle pratiche che risultano in scadenza in piattaforma. Per visualizzare l’elenco completo puoi consultare la pagina dedicata”, seguita da un link “Vedi tutte” che naviga alla pagina completa dei monitoraggi in scadenza.

#### Colonne e Dati

La tabella contiene le stesse colonne della tabella “Monitoraggi In Scadenza”

#### Comportamento e Interattività

La tabella nella dashboard mostra un massimo di 5 righe. Se ci sono più di 5 monitoraggi in scadenza, è presente un controllo di paginazione.

Le colonne sono ordinabili, con ordinamento predefinito per “Giorni al prossimo monitoraggi” in ordine crescente (i monitoraggi che scadono prima in alto).

Ogni riga include un link “Salta monitoraggio” che consente all’utente di richiedere un’eccezione, proprio come nella tabella dei monitoraggi scaduti.

### 8.5.1 Pagina Monitoraggi in Scadenza - Descrizione Generale e Accesso

La pagina “**Monitoraggi in Scadenza”** rappresenta la vista completa e dettagliata di tutti i monitoraggi che stanno per scadere (a partire da 7 giorni dalla data di scadenza). A differenza della dashboard che mostra solo i primi 5 monitoraggi in scadenza, questa pagina fornisce l’elenco esaustivo di tutti i monitoraggi che richiedono attenzione (visualizzabili gli ultimi 20 monitoraggi ordinati per scadenza, per vedere i restanti si potrà scorrere con i pulsanti posizionati al fondo della tabella con il controllo di paginazione “<< Pagina 1 >>” che consente di navigare tra le pagine), permettendo agli utenti di pianificare efficacemente le azioni necessarie per completarli entro le scadenze.

È presente, in alto a sinistra sopra i filtri, un **contatore** che quantifica i monitoraggi in scadenza totali

Questa pagina è particolarmente utile per:

* Identificare tutti i monitoraggi che stanno per scadere
* Pianificare le azioni necessarie per completare i monitoraggi entro le scadenze
* Filtrare i monitoraggi in scadenza per vari criteri (utente, tipologia pratiche, tipologia covenant)
* Ordinare i monitoraggi per urgenza (giorni al prossimo monitoraggio)
* Esportare i dati per analisi esterne o reporting - Richiedere eccezioni

**Accesso:** Cliccando su “Vedi tutte” nella sezione “Monitoraggi in Scadenza” della Dashboard Monitoraggio, oppure dal menu laterale sinistro (se disponibile).

### 8.5.2 Pagina Monitoraggi in Scadenza - Sezione Filtri e Ricerca Avanzata

#### Descrizione Funzionale

La pagina “Monitoraggi in Scadenza” inizia con una sezione dedicata ai filtri e alla ricerca, che consente agli utenti di restringere l’elenco dei monitoraggi in scadenza secondo vari criteri. Questa sezione è essenziale per gestire un numero potenzialmente grande di monitoraggi che richiedono attenzione.

#### Campi di Filtro Disponibili

**Filtro Utente Banca:** Un campo di ricerca che consente di cercare per email dell’utente ISP Di riferimento

**Filtro Tipologia Pratica:** Un dropdown che consente di selezionare una delle due tipologie di pratiche: Banca Agente; Banca Agente e SACE Agent

**Filtro Tracking ID:** Un campo di ricerca per cercare per ID univoco censito dall’utente ISP in fase di censimento.

**Filtro Nome Deal:** un campo di ricerca per cercare il nome del Deal di riferimento

**Filtro ID Evento di Monitoraggio:** Un campo di ricerca per cercare per ID del singolo evento di monitoraggio

**Filtro Giorni al prossimo Monitoraggio (Range):** Due campi numerici che consentono di specificare un intervallo di giorni al prossimo monitoraggio: - “Giorni al prossimo monitoraggio a partire da” (es. “5” per mostrare solo monitoraggi che avverranno almeno tra cinque giorni) - “Giorni al prossimo monitoraggio fino a” (es. “30” per mostrare solo monitoraggi che avverranno entro 30 giorni)

#### Pulsanti di Azione

**Pulsante “Applica filtri”:** Esegue la ricerca con i filtri selezionati e aggiorna la tabella sottostante.

**Pulsante “Resetta”:** Cancella tutti i filtri e mostra nuovamente tutti i monitoraggi scaduti.

**Pulsante “Scarica Report”:** Esporta tutti i record della tabella in un file excel formattato esattamente come la tabella

**Pulsante “Indietro”:** Torna alla Dashboard Monitoraggio.

### 8.5.3 Pagina Monitoraggi in Scadenza - Tabella Monitoraggi in Scadenza Completa

#### Descrizione Funzionale

La tabella principale della pagina “Monitoraggi in Scadenza” fornisce una vista completa di tutti i monitoraggi che stanno per scadere. A differenza della tabella nella dashboard che mostra solo 5 righe, questa tabella mostra tutti i monitoraggi in scadenza (visualizzabili gli ultimi 20 monitoraggi ordinati per scadenza, per vedere i restanti si potrà scorrere con i pulsanti posizionati al fondo della tabella con il controllo di paginazione “<< Pagina 1 >>” che consente di navigare tra le pagine) con la possibilità di ordinare, filtrare e scaricarli.

#### Colonne e Dati

La tabella contiene le seguenti colonne, in questo ordine:

1. **ID Pratica di Monitoraggio:** L’identificativo univoco della pratica: Lettera M seguita da numero ordinale da 1 a 999 (e.g. la terza pratica aperta sarà “M3”)
2. **Tracking ID:** ID univoco dei sistemi ISP per riconoscere l’operazione di riferimento, dato recuperato dal medesimo campo riferito alla relativa pratica di censimento
3. **Nome Deal:** Il nome della pratica/finanziamento (es. “Banca Nuova S.p.A”), dato recuperato dal medesimo campo riferito alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
4. **Utente Banca:** L’email dell’utente ISP responsabile (es. “Marioverdi@isp-it”), dato recuperato dal medesimo campo riferito all’utente Banca associato alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
5. **Utente Deloitte:** L’email dell’utente Deloitte a cui è stata assegnata alla pratica madre di monitoraggio (es. mverdi@deloitte.it), assegnato appositamente tramite la funzionalità presente nel pannello ADMIN (nel caso un utente Deloitte non fosse assegnato alla pratica, il primo utente Deloitte che accede alla pratica verrà assegnato alla stessa)
6. **Tipologia Pratica:** La tipologia della pratica (es. “Banca Agente”, “SACE Agent”), dato recuperato dal medesimo campo riferito alla tipologia pratica relativa alla pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
7. **Tipologia:** Il tipo specifico di covenant (es. “Leverage Ratio”, “Gearing Ratio”)
8. **ID Evento di Monitoraggio:** L’identificativo dell’evento di monitoraggio: ID costruito da ID Pratica di monitoraggio + ID Covenant + ID Evento (Esempio il quarto monitoraggio del secondo Covenant della Terza pratica sarà P3C2M4)
9. **Data di riferimento:** La data di riferimento a cui si riferisce il monitoraggio (es. “14/02/2025”)
10. **Scadenza Monitoraggio:** La data entro cui il monitoraggio deve essere completato (es. “14/03/2025”)
11. **Giorni al prossimo monitoraggio:** Il numero di giorni al prossimo monitoraggio (da calcolare come “*Today* – data di Scadenza”)
12. **Stato Evento di Monitoraggi:** Lo stato attuale, visualizzato con un punto colorato rosso (●) accanto a “Monitoraggio in scadenza”
13. **Azione:** Un link “Salta monitoraggio” che consente all’utente di richiedere un’eccezione (la funzionalità è descritta nel dettaglio nella sezione dedicata)

#### Comportamento e Interattività

**Ordinamento:** Tutte le colonne sono ordinabili. Cliccando sull’intestazione di una colonna, la tabella si ordina per quella colonna in ordine crescente (A-Z) o decrescente (Z-A). L’ordinamento predefinito è per “Giorni al prossimo monitoraggio” in ordine crescente (i monitoraggi che scadono prima in alto).

**Righe per Pagina:** La tabella mostra 20 righe per pagina

**Paginazione:** Controlli di paginazione “<< Pagina X >>” consentono di navigare tra le pagine.

**Evidenziazione Riga:** Al passaggio del mouse su una riga, la riga è evidenziata con un colore di sfondo leggermente più scuro per migliorare l’usabilità.

**Colore di Sfondo Riga:** - Nessun colore di sfondo

**Riga Selezionabile:** Cliccando su una riga della tabella, l’utente naviga al dettaglio della pratica, dove può visualizzare i dettagli completi del monitoraggio in scadenza.

### 8.6 Tabella Pratiche Monitoraggio Recenti

#### Descrizione Funzionale

Sotto alle due viste riferite ai Monitoraggi Scaduti e Monitoraggi in Scadenza sarà presente una terza tabella denominata “**Pratiche di monitoraggio Recenti”,** che conterrà una selezione delle ultime dieci Pratiche aperte, come nella tabella sopra sarà presente un link “vedi tutte” sopra all’angolo in alto a destra della tabella.

La tabella contiene le seguenti colonne, in questo ordine:

1. **ID Pratica di Monitoraggio:** L’identificativo univoco della pratica: Lettera P seguita da numero ordinale da 1 a 999 (e.g. la terza pratica aperta sarà “P3”)
2. **Tracking ID:** ID univoco dei sistemi ISP per riconoscere l’operazione di riferimento, dato recuperato dal medesimo campo riferito alla relativa pratica di censimento
3. **Nome Deal:** Il nome della pratica/finanziamento (es. “Banca Nuova S.p.A”), dato recuperato dal medesimo campo riferito alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
4. **Utente Banca:** L’email dell’utente ISP responsabile (es. “Marioverdi@isp-it”), dato recuperato dal medesimo campo riferito all’utente Banca associato alla relativa pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
5. **Utente Deloitte:** L’email dell’utente Deloitte a cui è stata assegnata alla pratica madre di monitoraggio (es. mverdi@deloitte.it), assegnato appositamente tramite la funzionalità presente nel pannello ADMIN (nel caso un utente Deloitte non fosse assegnato alla pratica, il primo utente Deloitte che accede alla pratica verrà assegnato alla stessa)
6. **Tipologia Pratica:** La tipologia della pratica (es. “Banca Agente”, “SACE Agent”), dato recuperato dal medesimo campo riferito alla tipologia pratica relativa alla pratica di censimento (individuabile tramite Tracking ID/ID Contratto)
7. **#Monitoraggi Scaduti:** Il numero di monitoraggi scaduti per questa pratica (es. “1”, “0”)
8. **# Monitoraggi in Scadenza:** Il numero di monitoraggi in scadenza per questa pratica (es. “0”, “1”)
9. **Stato Pratica Monitoraggio:** Un indicatore visivo dello stato complessivo della pratica, rappresentato da un punto colorato:
   1. Verde (●) = Monitoraggio OK
   2. Giallo (●) = Monitoraggio In scadenza
   3. Giallo (●) = Monitoraggio da attenzionare
   4. Rosso (●) = Monitoraggio Scaduto
   5. Rosso (●) = Monitoraggio Fuori soglia
   6. Grigio (●) = Pratica Conclusa
10. **Data creazione Pratica:** La data in cui la pratica è stata creata nel sistema (es. “15/03/2015”)
11. **Data Fine monitoraggio:** La data in cui il periodo di monitoraggio termina (es. “22/11/2019”), corrispondente alla Termination date dell’operazione estratta dal Contratto di Finanziamento (all’interno della fase di Booking)
12. **Azione:** colonna di azione contestuale in cui è possibile chiudere anticipatamente la pratica di monitoraggio (segue l’identico flusso per richiedere un’eccezione a un monitoraggio singolo, solo che dopo aver richiesto la chiusura anticipata la pratica verrà considerata conclusa e non più interagibile)

**\*Estinzione anticipata** 🡪 Attraverso l’apposito pulsante l’Utente ISP (o alternativamente l’Utente Deloitte) può segnalare l’avvenuta chiusura anticipata del finanziamento e della conseguente pratica di monitoraggio. All’utente verrà richiesto, attraverso un apposito pop-up indicazione della data di estinzione anticipata (= data fine monitoraggio nella tabella pratiche) e conferma dell’avvenuta chiusura dell’operazione.

Tale pratica confluirà quindi tra le “Pratiche concluse” e i dati di monitoraggio non dovranno essere più aggiornati. In caso di errore da parte dell’utente, sarà possibile forzare la riapertura della pratica che tornerà allo stato originale per poter essere ancora lavorato.

Cliccando sul link “Vedi tutte” l’utente atterrerà sulla vista “Tabella Pratiche Monitoraggio” come se avesse cliccato sulla sezione relativa della spalla a sinistra
