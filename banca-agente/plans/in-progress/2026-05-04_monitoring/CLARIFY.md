# Review Documentazione BR Agency Desk Monitoraggio V6

Data review: `2026-04-28`

Documentazione analizzata:
- `BR - Agency Desk Monitoraggio V6.docx` (convertito in MD)
- `202604_Macchina Stati Monitoring_v1.xlsx` (convertito in MD)
- `202604_Deck Mockup_Monitoraggio_v10.pptx` (convertito in MD)

Codebase verificati:
- BE (Ba-back-end) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-back-end`
- FE (Ba-web) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-web`
- DM (Ba-document-manager) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-document-manager`
- EM (Ba-email-manager) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-email-manager`

## Esito sintetico

La documentazione descrive un modulo complesso e articolato (monitoraggio covenant con macchina a stati multi-livello, flussi ISP/Deloitte, spread, mailing list, dashboard). La qualita' complessiva e' **media**: la struttura e' chiara e i flussi principali sono ben descritti, ma sono presenti contraddizioni significative sul formato ID (tre formati diversi), ambiguita' sulla macchina stati (versione "old" vs corrente senza indicazione di quale sia autorevole), sezioni incomplete (placeholder interni non rimossi), e disallineamenti sistematici tra BR, mockup e codice esistente.

Problemi trovati: **21 totali (5 bloccanti, 16 non bloccanti)**

---

## Parte 1 -- Per il team funzionale

Questa sezione elenca i punti che richiedono chiarimento o correzione nella documentazione.

### Problemi bloccanti

Questi impediscono una pianificazione affidabile.

#### 1. Formato ID Evento di Monitoraggio contraddittorio nel BR

- **Categoria**: Incoerenza
- **Bloccante**: Si → **RISOLTO**
- **Dove**: BR V6, Sezione 1 (righe 184-282)
- **Problema**: Il BR utilizza **due formati ID diversi** per gli eventi di monitoraggio nella stessa sezione:
  - Nella tabella esempio (riga 184-260): `P1C2M1`, `P1C2M2`, ecc. (P=Pratica, C=Covenant, M=Monitoraggio)
  - Nella definizione strutturata (riga 270-282): `M1C1E1`, `M1C1E2` (M=Monitoraggio pratica, C=Covenant, E=Evento)
  
  Inoltre, la sezione 8.5.3 (riga 1639) usa `P3C2M4` e la sezione 8.6 (riga 1668) usa il prefisso `P` per le pratiche invece di `M`.
- **Impatto**: Senza un formato ID univoco e coerente, non e' possibile progettare il modello dati, le API, e la UI. Ogni riferimento a ID nel sistema sarebbe ambiguo.
- **Domanda per il funzionale**: Qual e' il formato ID definitivo? Dalla definizione strutturata sembra essere `M{n}C{n}E{n}` (es. M1C1E1). Confermate che la tabella esempio a riga 184 (P1C2M1) e' un refuso da correggere?
- **Risposta del funzionale**: Il formato ID Evento di Monitoraggio da utilizzare e' M{n}C{n}E{n} (es. M1C1E1).
- **Data risposta**: 2026-04-30

#### 2. Macchina stati: versione "old" vs corrente -- quale e' autorevole?

- **Categoria**: Ambiguita'
- **Bloccante**: Si → **RISOLTO**
- **Dove**: Macchina Stati Monitoring v1.xlsx, sheet "Overview Stati (old)" e sheet "Pratica di Monitoraggio (old)" vs sheet "Pratica di Monitoraggio" corrente
- **Problema**: Il file della macchina stati contiene sia tabelle marcate "(old)" sia tabelle senza marcatura. Le differenze sono sostanziali:
  - **Versione "old"** per Grey (Pratica Conclusa): si attiva con "Data di scadenza operazione < Today" **E** assenza di eventi scaduti/fuori soglia **E** azione manuale "Concludi pratica anticipatamente" -- cioe' include sia la chiusura automatica per data che quella manuale
  - **Versione corrente** per Grey: si attiva **solo** con l'azione manuale "Concludi pratica anticipatamente"
  - Il BR (sezione 2.3, riga 353) dice: "oltre questa data la pratica viene automaticamente chiusa" -- che e' coerente con la versione **old**, non con quella corrente.
- **Impatto**: La logica di chiusura automatica delle pratiche e' un requisito architetturale fondamentale (scheduler, batch job, notifiche). Implementare la versione sbagliata richiede un rifacimento completo.
- **Domanda per il funzionale**: La chiusura della pratica per raggiungimento della Termination Date e' automatica (come descritto nel BR e nella versione "old" della macchina stati) o richiede sempre un'azione manuale dell'utente (come nella versione corrente della macchina stati)?
- **Risposta del funzionale**: La chiusura di pratica per raggiungimento della Termination Date e' automatica come descritto nella versione old. Le condizioni che generano la chiusura automatica della pratica sono: "Data di scadenza operazione < Today" e "Non vi sono eventi di monitoraggio che presentano status Monitoraggio Scaduto / Monitoraggio Fuori Soglia".
- **Data risposta**: 2026-04-30
- **Risposta del funzionale (aggiornamento)**: Le condizioni per la chiusura automatica sono state precisate: (1) "Data di scadenza operazione < Today", (2) "Data di scadenza piu' lontana tra gli eventi di monitoraggio < Today", (3) "Non vi sono eventi di monitoraggio che presentano status Monitoraggio Scaduto / Monitoraggio Fuori Soglia / Monitoraggio in Scadenza".
- **Data aggiornamento**: 2026-05-05

#### 3. Sezione 3.2.7 incompleta -- placeholder interno non rimosso

- **Categoria**: Gap funzionale
- **Bloccante**: Si → **RISOLTO**
- **Dove**: BR V6, Sezione 3.2.7, punto 10-12 (righe 538-545)
- **Problema**: I punti 10-12 della tabella "Tutti i Covenant" contengono un placeholder interno: il punto 11 dice letteralmente **"Mettiamo quello inserito nella macchina stati"** invece di definire la logica dello status. A seguire, i punti 11-12 iniziano a elencare le logiche di stato ma senza la struttura di lista numerata coerente col resto della tabella (i punti 10-13 dovrebbero essere colonne della tabella, non una sotto-lista di regole di stato).
- **Impatto**: La logica di stato del covenant a livello di tabella "Tutti i Covenant" non e' definita nel BR. La macchina stati la definisce a livello di Evento, non di Covenant aggregato.
- **Domanda per il funzionale**: Potete completare la sezione 3.2.7 punto 10 "Status Monitoraggio" con la definizione esatta delle regole? In particolare: lo stato del covenant nella tabella "Tutti i Covenant" segue la stessa logica di priorita' dello stato Pratica (Fuori Soglia > Scaduto > In Scadenza > Da attenzionare > OK) applicata ai suoi eventi?
- **Risposta del funzionale**: Lo Status Monitoraggio per i singoli Covenant segue le medesime logiche riportate nel file "202604_Macchina Stati Monitoring_v1.xlsx" -- Sheet "Pratica di Monitoraggio" in quanto influenzato dai singoli Eventi di Monitoraggio.
- **Data risposta**: 2026-04-30
- **Risposta del funzionale (aggiornamento)**: Lo Status Monitoraggio per i singoli Covenant segue le medesime logiche di priorita' dello stato pratica sulla base degli eventi di monitoraggio del covenant di riferimento. Il funzionale ha aggiornato la macchina a stati: `202604_Macchina Stati Monitoring_v2.xlsx` -- nuovo sheet "Singolo Covenant" con le logiche specifiche.
- **Data aggiornamento**: 2026-05-05

#### 4. Semaforo: 4 o 6 colori? Incoerenza tra filtri e tabella

- **Categoria**: Incoerenza
- **Bloccante**: Si → **RISOLTO**
- **Dove**: BR V6, Sezione 2.2 vs Sezione 2.3 (righe 311 vs 346-351)
- **Problema**: 
  - **Sezione 2.2** (Filtri): elenca 6 stati con colori distinti: OK (verde), In Scadenza (giallo), Da attenzionare (arancione), Scaduto (rosso), Fuori Soglia (rosso scuro), Conclusa (grigio)
  - **Sezione 2.3** (Tabella Semaforo): elenca solo 4 colori ma 6 stati, usando lo stesso colore per stati diversi: Giallo per "In scadenza" E "Da attenzionare", Rosso per "Scaduto" E "Fuori soglia"
  - Se il semaforo usa solo 4 colori, l'utente non puo' distinguere visivamente "In Scadenza" da "Da attenzionare", ne' "Scaduto" da "Fuori Soglia".
  - I mockup non chiariscono perche' usano emoji generiche (?) che non mostrano il colore effettivo.
- **Impatto**: L'implementazione UI e la logica di filtraggio dipendono da questa decisione. Se sono 6 colori, servono 6 classi CSS e 6 valori filtro. Se sono 4, servono label di testo diverse per distinguere stati con stesso colore.
- **Domanda per il funzionale**: Quanti colori distinti ha il semaforo? Se sono 4 (come nella tabella 2.3), come si distinguono visivamente "In Scadenza" da "Da attenzionare" e "Scaduto" da "Fuori Soglia"? Tramite label testuale, icona diversa, o il filtro ha semplicemente piu' opzioni del necessario?
- **Risposta del funzionale**: Il semaforo ha 4 colori distinti per 6 stati: Verde = OK, Giallo = In scadenza e Da attenzionare, Rosso = Scaduto e Fuori soglia, Grigio = Conclusa. Non vi e' una distinzione grafica per "In scadenza" vs "Da attenzionare" e per "Scaduto" vs "Fuori Soglia".
- **Data risposta**: 2026-04-30
- **Risposta del funzionale (correzione)**: Il Semaforo ha **6 colori distinti** per 6 stati: (1) Verde = Monitoraggio OK, (2) Giallo = Monitoraggio In scadenza, (3) Arancione = Monitoraggio Da attenzionare, (4) Rosso = Monitoraggio Scaduto, (5) Rosso scuro = Monitoraggio Fuori soglia, (6) Grigio = Monitoraggio Concluso. **Nota**: questa risposta corregge la precedente del 2026-04-30 che indicava 4 colori.
- **Data aggiornamento**: 2026-05-05

#### 5. Assegnazione Utente Deloitte: ereditata dal censimento o auto-assegnata al primo accesso?

- **Categoria**: Incoerenza
- **Bloccante**: Si → **RISOLTO**
- **Dove**: BR V6, Sezione 1 (riga 105) vs Sezione 8.4.3 (riga 1453)
- **Problema**: 
  - Sezione 1: "Gli utenti assegnati alla lavorazione della pratica (Utente ISP e Utente Deloitte) rimarranno i medesimi della pratica di censimento collegata"
  - Sezione 8.4.3: "nel caso un utente Deloitte non fosse assegnato alla pratica, il primo utente Deloitte che accede alla pratica verra' assegnato alla stessa"
  - Se l'utente Deloitte viene sempre ereditato dalla pratica di censimento, il caso "non fosse assegnato" non dovrebbe mai verificarsi.
- **Impatto**: Logica di assegnazione pratica, permessi di accesso, e logica di "IN CARICO A" dipendono da questa definizione.
- **Domanda per il funzionale**: L'utente Deloitte e' sempre ereditato dalla pratica di censimento? O c'e' un caso in cui la pratica di monitoraggio viene creata senza Utente Deloitte e serve l'auto-assegnazione al primo accesso?
- **Risposta del funzionale**: Per quanto concerne la Wave 3/3bis l'utente Deloitte incaricato della lavorazione della pratica di monitoraggio e' il medesimo della corrispondente pratica di censimento, senza necessita' di una nuova assegnazione.
- **Data risposta**: 2026-04-30

---

### Problemi non bloccanti

#### 1. Formula "Giorni al prossimo monitoraggio" -- segno e direzione

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: BR V6, Sezione 3.3.1 (riga 567), Sezione 5.2 (riga 982), Sezione 8.5.3 (riga 1642)
- **Problema**: La formula e' descritta come "Today - Scadenza Monitoraggio" (sez. 3.3.1 e 8.5.3). Con questa formula:
  - Per monitoraggi **scaduti** (scadenza nel passato): il risultato e' **positivo** (es. Today=28/04 - Scadenza=01/03 = +58 giorni)
  - Per monitoraggi **in scadenza** (scadenza nel futuro): il risultato e' **negativo** (es. Today=28/04 - Scadenza=05/05 = -7 giorni)
  
  Ma gli **esempi nel BR mostrano il contrario**: monitoraggi scaduti con valori negativi ("-45", "-5") e monitoraggi in scadenza con valori positivi ("10", "6", "8"). La formula corretta per matchare gli esempi sarebbe **Scadenza - Today**.
- **Domanda per il funzionale**: La formula corretta e' "Scadenza - Today" (valori positivi = giorni rimanenti, negativi = giorni di ritardo)?
- **Risposta del funzionale**: Applichiamo la logica suggerita, invertendo la formula in **Giorni al prossimo monitoraggio = Data di scadenza - Today**.
- **Data risposta**: 2026-05-05

#### 2. "IN CARICO A" -- caso non coperto

- **Categoria**: Gap funzionale
- **Bloccante**: No
- **Dove**: BR V6, Sezione 3.2.1 (righe 406-417) e 4.2.1 (righe 756-769)
- **Problema**: La logica definisce "IN CARICO A" come:
  - "Utente Banca" se tutti i documenti sono "Completo" **E** ci sono eventi futuri **E** almeno un evento e' "in scadenza" o "scaduto"
  - "Utente Deloitte" se almeno un documento non e' "Completo"
  
  Ma non copre il caso: tutti i documenti sono "Completo" **E** nessun evento e' "in scadenza" o "scaduto" (tutti OK o tutti futuri lontani). A chi e' "IN CARICO A" in questo caso?
- **Domanda per il funzionale**: Quando tutti i documenti sono completi e nessun evento e' in scadenza/scaduto, il campo "IN CARICO A" deve mostrare "Utente Banca" (in attesa del prossimo evento) o rimanere vuoto?
- **Risposta del funzionale**: In tal caso, la label "IN CARICO A" deve mostrare "Utente Banca".
- **Data risposta**: 2026-05-05

#### 3. Sezione 8.5: auto-riferimento colonne

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: BR V6, Sezione 8.5 (riga 1566)
- **Problema**: La sezione dice "La tabella contiene le stesse colonne della tabella 'Monitoraggi In Scadenza'" -- ma sta descrivendo proprio la tabella "Monitoraggi in Scadenza". E' un auto-riferimento circolare, probabilmente un errore di copia-incolla dalla sezione 8.4 (Monitoraggi Scaduti).
- **Domanda per il funzionale**: Le colonne della tabella dashboard "Monitoraggi in Scadenza" sono identiche a quelle della tabella dashboard "Monitoraggi Scaduti" (sezione 8.4)?
- **Risposta del funzionale**: Refuso. Le colonne della dashboard "Monitoraggi in Scadenza" sono identiche a quelle di "Monitoraggi Scaduti" (sezione 8.4).
- **Data risposta**: 2026-05-05

#### 4. Sezione 2.3: typo "chiusura pratica"

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: BR V6, Sezione 2.3 (riga 354)
- **Problema**: Il testo dice "chiudere anticipatamente la chiusura pratica di monitoraggio" -- la parola "chiusura" (con typo) sembra un doppione. Dovrebbe essere "chiudere anticipatamente la pratica di monitoraggio".
- **Domanda per il funzionale**: Confermate che il testo corretto e' "chiudere anticipatamente la pratica di monitoraggio"?
- **Risposta del funzionale**: Confermato refuso.
- **Data risposta**: 2026-05-05

#### 5. Label azione "Termina Pratica" -- naming incoerente tra mockup

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: Mockup slide 1 vs slide 2, BR V6 sezione 2.3 e 8.6
- **Problema**: La stessa azione ha nomi diversi:
  - Mockup slide 1 (dashboard "Pratiche Recenti"): "Termina Pratica id Monitoraggio"
  - Mockup slide 2-3 (Tabella Pratiche): "Termina Pratica di Monitoraggio"
  - BR sezione 2.3: "chiudere anticipatamente la pratica di monitoraggio"
  - BR sezione 8.6: "colonna di azione contestuale in cui e' possibile chiudere anticipatamente la pratica"
- **Domanda per il funzionale**: Qual e' la label definitiva del pulsante? "Termina Pratica di Monitoraggio" sembra la piu' usata -- confermate?
- **Risposta del funzionale**: Confermato: "Termina Pratica di Monitoraggio".
- **Data risposta**: 2026-05-05

#### 6. "Valore Monitoraggio" vs "Importo Monitorato" -- terminologia

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: BR V6 (tutto il documento) vs Mockup slide 7 (righe 361-367)
- **Problema**: Il BR usa costantemente "Valore Monitoraggio" per indicare il valore numerico del covenant monitorato. Il mockup slide 7 (assegnazione compliance certificate) usa "Importo Monitorato" per lo stesso concetto.
- **Domanda per il funzionale**: Il termine corretto e' "Valore Monitoraggio" (come nel BR) o "Importo Monitorato" (come nel mockup)?
- **Risposta del funzionale**: Utilizziamo la dicitura "Valore Monitoraggio" come descritto nei BR.
- **Data risposta**: 2026-05-05

#### 7. Mockup "INCARICO A" vs BR "IN CARICO A"

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: Mockup slide 5 (riga 234) vs BR V6 sezione 3.2.1 e mockup slide 4 (riga 167)
- **Problema**: Il mockup slide 5 scrive "INCARICO A" (attaccato, senza spazio) mentre il BR e lo slide 4 scrivono "IN CARICO A". Probabilmente un typo nel mockup.
- **Domanda per il funzionale**: Confermate che la label corretta e' "IN CARICO A"?
- **Risposta del funzionale**: Utilizziamo la dicitura "IN CARICO A".
- **Data risposta**: 2026-05-05

#### 8. ID Covenant nella colonna "ID Covenant" -- incoerenza tra mockup

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: Mockup slide 4 vs slide 8
- **Problema**: La colonna "ID Covenant" nelle tabelle monitoraggi scaduti/in scadenza mostra formati diversi tra mockup:
  - Slide 4 (riga 196): "M2C1E1" nella colonna "ID Covenant" (ma questo e' un ID evento, non covenant)
  - Slide 8 (riga 434): "Prat002cov001" nella colonna "ID Covenant"
  - BR sezione 3.2.5: "L'identificativo del covenant, composto da ID pratica + ID Covenant (E.g. M3C2)"
  
  Il mockup mette l'ID evento nella colonna ID Covenant, mentre il BR dice che l'ID Covenant e' M{n}C{n} (senza la parte evento).
- **Domanda per il funzionale**: La colonna "ID Covenant" deve mostrare l'ID covenant (es. M3C2) come da BR, o l'ID evento (es. M3C2E4)?
- **Risposta del funzionale**: La colonna ID Covenant deve seguire il formato M{n}C{n} (senza la parte evento). La colonna ID Evento di Monitoraggio deve seguire invece il formato M{n}C{n}E{n}.
- **Data risposta**: 2026-05-05

#### 9. Dashboard mockup: ID pratica usa "Prat001" ma BR definisce "M1"

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: Mockup slide 1-3 vs BR V6 sezione 2.3
- **Problema**: Tutti i mockup della dashboard e della tabella pratiche usano "Prat001", "Prat002", ecc. come ID pratica, mentre il BR definisce chiaramente il formato come "M1", "M2", ecc. (lettera M + numero progressivo).
- **Domanda per il funzionale**: I mockup con "Prat001" rappresentano dati placeholder/esempio e il formato reale sara' "M1"? Oppure volete adottare un formato diverso (es. "PRAT001" con padding zero)?
- **Risposta del funzionale**: Il formato per ID Pratica e' M{n}.
- **Data risposta**: 2026-05-05

#### 10. Mockup "Monitoraggi Scaduti": colonna "Note" duplicata

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: Mockup slide 5 (righe 260-262)
- **Problema**: La tabella "Monitoraggi Scaduti" nel mockup slide 5 ha **due colonne "Note"**: una in posizione 3 (dopo Tipologia) e una in posizione 12 (dopo Status). Nella slide 4 la colonna "Note" compare solo una volta.
- **Domanda per il funzionale**: La seconda colonna "Note" nella slide 5 e' un errore di duplicazione nel mockup?
- **Risposta del funzionale**: Refuso nel mockup, errore di duplicazione della colonna "Note".
- **Data risposta**: 2026-05-05

#### 11. Sezione COVNO: processo di sincronizzazione non dettagliato

- **Categoria**: Gap funzionale
- **Bloccante**: No
- **Dove**: BR V6, Sezione 8.3 (righe 1409-1436)
- **Problema**: La sezione descrive il caricamento di file "DB Obblighi" e "COVN0" per l'aggiornamento dei dati di monitoraggio, ma non specifica:
  - Il formato dei file (CSV, XLSX, altro?)
  - La struttura/colonne attese nei file
  - Le regole di matching (oltre al Tracking ID)
  - La gestione dei conflitti (cosa succede se un valore nel file contraddice un valore inserito manualmente da Deloitte?)
  - Se l'aggiornamento e' incrementale o sostitutivo
- **Domanda per il funzionale**: Potete fornire un tracciato record dei file COVN0 e DB Obblighi con i campi, i formati e le regole di aggiornamento?
- **Risposta:** *(in attesa di risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 12. Soglia "Da attenzionare" -- condizione OR o AND tra le due regole?

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: BR V6, Sezione 3.2.7 (riga 542-543) e Macchina Stati (riga 24)
- **Problema**: Lo stato "Da attenzionare" si attiva quando:
  - (A) Il valore covenant si discosta del 10% dalla soglia di rottura, **oppure**
  - (B) Il valore covenant si discosta del 30% dalla soglia di rottura **ed** e' differito del 40% rispetto alla rilevazione precedente (avvicinandosi al valore soglia)
  
  Le due condizioni (A) e (B) sono in OR tra loro (basta una). Ma non e' chiaro:
  - "Discosta del 10%" significa che il valore e' entro il 10% della soglia (vicino alla soglia)?
  - "Avvicinandosi al valore soglia" -- come si determina la direzione? Se il segno e' "<", avvicinarsi significa che il valore sta crescendo?
- **Domanda per il funzionale**: Potete fornire un esempio numerico per entrambe le condizioni? Es. se il covenant e' "Leverage < 70", a quale valore scatta "Da attenzionare"?
- **Risposta del funzionale**: Esempio numerico fornito con formule precise:
  - **Condizione A (10% dalla soglia)**:
    - Segno `<` o `<=`: `(valore_soglia - 10% * valore_soglia) <= valore_covenant < valore_soglia`. Es. soglia <70: 63 <= 66 < 70 → Da attenzionare.
    - Segno `>` o `>=`: `(valore_soglia + 10% * valore_soglia) >= valore_covenant > valore_soglia`. Es. soglia >70: 77 >= 75 > 70 → Da attenzionare.
  - **Condizione B (30% dalla soglia + 40% variazione)**:
    - Segno `<` o `<=`: `(valore_soglia - 30% * valore_soglia) <= valore_covenant[t] < valore_soglia AND valore_covenant[t] >= 140% * valore_covenant[t-1]`. Es. soglia <70, valore[t]=50, valore[t-1]=30: limite 30%=49, limite 40% da t-1=42; 50>49 e 50>42 e 50<70 → Da attenzionare.
    - Segno `>` o `>=`: `(valore_soglia + 30% * valore_soglia) >= valore_covenant[t] > valore_soglia AND valore_covenant[t] <= 60% * valore_covenant[t-1]`. Es. soglia >70, valore[t]=80, valore[t-1]=140: limite 30%=91, limite 40% da t-1=84; 80<91 e 80<84 e 80>70 → Da attenzionare.
  - Le due condizioni A e B sono in OR.
- **Data risposta**: 2026-05-05

#### 13. Eccezione "Salta Monitoraggio" -- effetto sullo stato e sui conteggi

- **Categoria**: Gap funzionale
- **Bloccante**: No
- **Dove**: BR V6, Sezione 3.4.1 (riga 604) e Macchina Stati (riga 35)
- **Problema**: Il BR dice che l'eccezione "genera un cambio di stato per l'evento di monitoraggio, il quale passera' allo status 'green'". Ma non chiarisce:
  - Se l'evento saltato viene conteggiato in "# Monitoraggi OK" o escluso dai conteggi
  - Se "Data avvenuto monitoraggio" viene compilata con la data dell'eccezione o resta vuota
  - Se il "Valore Monitoraggio" resta vuoto o viene compilato con un valore speciale
- **Domanda per il funzionale**: Un evento con eccezione approvata viene conteggiato come "OK" nei conteggi? La data avvenuto monitoraggio viene compilata?
- **Risposta del funzionale**: In seguito all'azione "Salta Monitoraggio" l'evento viene conteggiato in # Monitoraggi OK, "Data avvenuto Monitoraggio" viene compilata con la data in cui viene svolta l'azione "Salta Monitoraggio" e "Valore Monitoraggio" resta vuoto.
- **Data risposta**: 2026-05-05

#### 14. Spread: storico variazioni -- manca il campo "Azione: Scarica"

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: BR V6, Sezione 3.5.2 (righe 690-694) vs Sezione 7.3.5 (righe 1360-1369)
- **Problema**: La sezione 3.5.2 (vista ISP) descrive lo Storico Spread con colonne: Data Variazione, Documento, Azione (Scarica), Cluster. La sezione 7.3.5 (creazione Deloitte) descrive una tabella diversa: Data Riferimento, Data avvenuto monitoraggio, Valore Monitorato, Cluster di Riferimento, Valore Margine (%). Le due tabelle hanno strutture completamente diverse per lo stesso concetto.
- **Domanda per il funzionale**: Lo "Storico Spread" ha una struttura unica per ISP e Deloitte? Se si, quale delle due versioni e' corretta?
- **Risposta del funzionale**: Gli oggetti identificati sono due oggetti differenti correttamente requisitati in maniera differente: l'oggetto requisitato "Tabella storico Spread" (sezione 3.5.2) ha una struttura definita e differente rispetto alla "Tabella Spread" (sezione 7.3.5). Sono entrambi corretti.
- **Data risposta**: 2026-05-05

#### 15. Mailing List: trigger notifiche non specificati

- **Categoria**: Gap funzionale
- **Bloccante**: No
- **Dove**: BR V6, Sezione 6 (tutto)
- **Problema**: La sezione descrive la gestione della mailing list (CRUD contatti, upload AD Form, estrazione GenAI) ma non specifica:
  - Quali eventi generano notifiche email ai contatti della mailing list
  - Se la mailing list si applica solo alla pratica di monitoraggio o anche al censimento
  - Se ci sono ruoli diversi nella mailing list (destinatari obbligatori, CC, ecc.)
  - La relazione tra mailing list e le notifiche automatiche di scadenza/eccezione
- **Domanda per il funzionale**: Quando e a chi vengono inviate le notifiche? La mailing list e' usata per le notifiche di scadenza monitoraggio, eccezioni, variazioni spread, o solo per comunicazioni manuali?
- **Risposta:** *(in attesa di risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 16. Dashboard donut chart: aggiornamento "tempo reale"

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: BR V6, Sezione 8.2 (riga 1407)
- **Problema**: Il BR richiede che "I dati del donut chart devono aggiornarsi in tempo reale quando cambiano gli stati dei monitoraggi nel sistema. Questo aggiornamento deve avvenire senza richiedere un refresh manuale della pagina." Questo implica una tecnologia push (WebSocket, SSE) che ha impatto architetturale significativo.
- **Domanda per il funzionale**: "Tempo reale" significa letteralmente push dal server (WebSocket) o e' sufficiente un polling periodico (es. ogni 30 secondi)?
- **Risposta:** *(in attesa di risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

---

## Parte 2 -- Per il team tecnico

Questa sezione contiene le assunzioni che il team tecnico adottera' in assenza di chiarimenti dal funzionale. Ogni assunzione e' legata a un problema della Parte 1.

### Assunzioni proposte

| # | Problema rif. | Assunzione proposta | Rischio se errata | Costo | Stato | Risposta funzionale |
|---|---|---|---|---|---|---|
| A-001 | NB-1 | Formula = Scadenza - Today. Positivo = giorni rimanenti, negativo = giorni di ritardo. | Tutti i valori mostrati in tabella hanno segno invertito | Basso | **Confermata** | "Invertiamo la formula in Data di scadenza - Today" |
| A-002 | NB-2 | Se nessun documento in lavorazione e nessun evento in scadenza/scaduto, "IN CARICO A" = "Utente Banca" | Label fuorviante per l'utente | Basso | **Confermata** | "Mostrare Utente Banca" |
| A-003 | NB-3 | Dashboard "Monitoraggi in Scadenza" ha le stesse colonne di "Monitoraggi Scaduti" (sez. 8.4) | Colonne diverse da aggiungere/rimuovere | Basso | **Confermata** | "Refuso, colonne identiche" |
| A-004 | NB-4 | Correggere in "chiudere anticipatamente la pratica di monitoraggio" | Nessun rischio | Basso | **Confermata** | "Confermato refuso" |
| A-005 | NB-5 | Label = "Termina Pratica di Monitoraggio" | Cambio label | Basso | **Confermata** | "Confermato" |
| A-006 | NB-6 | Usare "Valore Monitoraggio" (termine del BR) | Cambio label | Basso | **Confermata** | "Valore Monitoraggio come da BR" |
| A-007 | NB-7 | Label = "IN CARICO A" (con spazi) | Cambio label | Basso | **Confermata** | "IN CARICO A" |
| A-008 | NB-8 | Colonna "ID Covenant" mostra M{n}C{n}, colonna separata "ID Evento" mostra M{n}C{n}E{n} | Rework colonne tabella | Basso | **Confermata** | "M{n}C{n} per covenant, M{n}C{n}E{n} per evento" |
| A-009 | NB-9 | Formato ID pratica = "M{n}" (M1, M2, M3...) senza padding zero. "Prat001" nei mockup e' placeholder | Se serviva padding zero (M001), rework generazione ID e display | Basso | **Confermata** | "Formato M{n}" |
| A-010 | NB-10 | La seconda colonna "Note" e' un errore nel mockup, ignorarla | Manca una colonna richiesta | Basso | **Confermata** | "Refuso nel mockup" |
| A-011 | NB-11 | Upload manuale COVN0/DB Obblighi in formato XLSX. Aggiornamento incrementale per Tracking ID. Conflitti: il file vince sui dati automatici, non su quelli inseriti manualmente da Deloitte. | Formato/logica diversi da reimplementare | Alto | *In attesa* | -- |
| A-012 | NB-12 | "10% dalla soglia" = valore entro il 90%-100% della soglia (es. soglia 70 con segno "<": valore > 63 e < 70 attiva "Da attenzionare"). Le due condizioni sono in OR. | Calcolo soglia errato, stati monitoraggio sbagliati | Medio | **Confermata con dettaglio** | Formule precise fornite con esempi numerici per entrambi i segni e entrambe le condizioni |
| A-013 | NB-13 | Evento con eccezione: conteggiato come "OK", data avvenuto monitoraggio = data dell'eccezione, valore monitoraggio = "N/A" | Conteggi errati, report inconsistenti | Medio | **Confermata con correzione** | "Conteggiato OK, data = data azione, valore resta vuoto (non N/A)" |
| A-014 | NB-14 | La tabella Storico Spread della sezione 7.3.5 (con Data Riferimento, Valore Monitorato, Cluster, Margine) e' quella definitiva. La sezione 3.5.2 e' una versione precedente. | Tabella da rifare | Medio | **Rigettata** | "Sono due oggetti distinti entrambi corretti: Tabella storico Spread (sez. 3.5.2) e Tabella Spread (sez. 7.3.5)" |
| A-015 | NB-15 | La mailing list e' usata per notifiche automatiche di: scadenza monitoraggio (7 giorni prima), monitoraggio scaduto, eccezione richiesta/gestita, variazione spread. Tutti i contatti sono in TO (nessuna distinzione CC). | Template notifiche da rifare, logica destinatari diversa | Alto | *In attesa* | -- |
| A-016 | NB-16 | "Tempo reale" = polling dal frontend ogni 60 secondi, non WebSocket | Se servono WebSocket, impatto architetturale significativo | Medio | *In attesa* | -- |

### Disallineamenti col codice

| # | Concetto BR | Nel codice | File/Classe | Nota |
|---|---|---|---|---|
| D-001 | Stati documento monitoraggio: "In attesa di conferma", "Verifica dati in corso", "Estrazione GenAI in corso", "Estrazione GenAI da validare", "Completato" | `DocumentStateEnum`: TO_BE_UPLOAD, TO_BE_DOWNLOAD, TO_BE_VALIDATED, VALIDATED, UPLOADED, REJECTED | `Ba-back-end/.../enumeration/document/DocumentStateEnum.java` | Il BR definisce stati completamente diversi per i documenti di monitoraggio. L'enum esistente serve il flusso censimento. Servira' un nuovo enum o un'estensione. |
| D-002 | Stati documento monitoraggio (sottostati Deloitte): "Estrazione GenAI in corso", "Estrazione GenAI da validare" | `SectionStateEnum`: SECTION_COMPLETE_AI_EXTRACTION_IN_PROGRESS, SECTION_COMPLETE_AI_TO_VERIFY_EXTRACTION, SECTION_COMPLETE_AI_TO_VALIDATE_REPORT, SECTION_REPORT_GENERATED | `Ba-back-end/.../enumeration/agency_desk/SectionStateEnum.java` | La `SectionStateEnum` ha logica simile ma naming diverso e si applica a livello di sezione, non di singolo documento. Per il monitoraggio servono stati a livello documento. |
| D-003 | Periodicita' monitoraggio: Annuale, Semestrale, Trimestrale, Mensile | `MonitoringPeriodicityEnum`: ANNUALE, SEMESTRALE, TRIMESTRALE, MENSILE, **ALTRO** | `Ba-back-end/.../enumeration/agency_desk/MonitoringPeriodicityEnum.java` | Il BR non menziona il valore "ALTRO" che esiste nell'enum. Chiarire se va mantenuto per retrocompatibilita'. |
| D-004 | ID Pratica Monitoraggio: formato "M{n}" | Nessun generatore di ID monitoraggio | Nessun file dedicato | Non esiste logica di generazione ID per le pratiche di monitoraggio. La `vw_monitoring_practice_summary` usa `mp.code` come `practice_code` ma non e' chiaro il formato. |
| D-005 | Entita' Covenant (con campi: Tipologia, Valore Limite, Segno, Periodicita', Covenant Variabile, Note) | Nessuna entita' `Covenant` nel dominio | `Ba-back-end/.../domain/` | Esistono enum per tipi covenant (`CovenantTypeNumericEnum` con ~53 valori, `CovenantSignEnum`) ma nessuna entita' JPA. Tutto da creare. |
| D-006 | Entita' Evento di Monitoraggio (con campi: ID, Data Riferimento, Scadenza, Valore Monitoraggio, Status, Data Avvenuto Monitoraggio) | Nessuna entita' `MonitoringEvent` | `Ba-back-end/.../domain/` | Non esiste. Tutto da creare. |
| D-007 | Macchina a stati multi-livello: Pratica -> Covenant -> Evento -> Documento | Nessuna macchina a stati per monitoraggio | `Ba-back-end/.../service/psm/PsmStateService.java` | La PSM esistente gestisce stati pratica generici (censimento/booking/pre-closing/post-closing). Per il monitoraggio servono 4 livelli di stato indipendenti. |
| D-008 | Email monitoraggio: notifiche scadenza, eccezione, rifiuto documento, variazione spread | Nessun template email per monitoraggio | `Ba-back-end/.../enumeration/EmailTemplateEnum.java` | Servira' aggiungere 6-10 nuovi template email in `EmailTemplateEnum` e i corrispettivi template in Ba-email-manager. |
| D-009 | Spread: tabella spread per pratica, storico variazioni, collegamento a covenant | Nessuna logica spread | `Ba-back-end/` | La parola "spread" nei file BE si riferisce a spreadsheet (fogli Excel), non a spread finanziari. Tutto da creare. |
| D-010 | Mailing list per pratica monitoraggio: CRUD contatti, upload AD Form, estrazione GenAI | Solo `isBookingMailingListReportGenerated()` per il flusso Booking | `Ba-back-end/.../service/psm/PsmStateService.java` | Il concetto di mailing list esiste solo come check booleano nel flusso Booking. Per il monitoraggio serve un modulo completo. |
| D-011 | Frontend: dashboard monitoraggio, tabella pratiche, dettaglio pratica, dettaglio covenant, spread, scadenziere, mailing list | Solo `MonitoringComponent` generico (step in un wizard di dossier) | `Ba-web/.../dossier/step-type-monitoring/monitoring.component.ts` | Il FE ha solo un componente monitoring generico usato nel wizard del dossier censimento. Tutte le pagine del modulo monitoraggio sono da creare da zero. |
| D-012 | Document manager: gestione compliance certificate, upload, validazione, stati specifici monitoraggio | Nessun codice monitoring | `Ba-document-manager/` | Il DM non ha alcun codice relativo al monitoraggio. Se la gestione documenti monitoraggio passa dal DM, va implementato da zero. |

---

## Riepilogo per br-analyzer

Ultimo aggiornamento: 2026-05-05 (br-clarify, round 2)

### Bloccanti risolti

1. [B1] Formato ID Evento Monitoraggio → Il formato definitivo e' `M{n}C{n}E{n}` (es. M1C1E1). Le occorrenze di `P1C2M1` nel BR sono refusi.
2. [B2] Macchina stati old vs corrente → La chiusura e' automatica (versione "old"). Condizioni aggiornate: (1) "Data di scadenza operazione < Today", (2) "Data di scadenza piu' lontana tra gli eventi di monitoraggio < Today", (3) assenza di eventi con status "Monitoraggio Scaduto" / "Monitoraggio Fuori Soglia" / "Monitoraggio in Scadenza".
3. [B3] Sezione 3.2.7 incompleta → Lo Status Monitoraggio del singolo Covenant segue le logiche di priorita' dello stato pratica, basate sugli eventi. Macchina a stati aggiornata: `202604_Macchina Stati Monitoring_v2.xlsx` -- nuovo sheet "Singolo Covenant".
4. [B4] Semaforo → **6 colori distinti** (corretto rispetto alla risposta precedente): Verde=OK, Giallo=In scadenza, Arancione=Da attenzionare, Rosso=Scaduto, Rosso scuro=Fuori soglia, Grigio=Concluso. Servono 6 classi CSS e 6 valori filtro.
5. [B5] Assegnazione Utente Deloitte → Per Wave 3/3bis l'utente Deloitte e' ereditato dalla pratica di censimento, senza auto-assegnazione.

### Bloccanti ancora aperti

Nessun bloccante aperto. Tutti i bloccanti sono stati risolti.

### Stato assunzioni

Assunzioni confermate dal funzionale: A-001, A-002, A-003, A-004, A-005, A-006, A-007, A-008, A-009, A-010, A-012, A-013
Assunzioni adottate (nessuna risposta, si procede con l'assunzione proposta): A-011, A-015, A-016
Assunzioni rigettate (risposta diversa dall'assunzione):
- A-014: assunzione era "tabella sez. 7.3.5 definitiva, sez. 3.5.2 precedente" → il funzionale ha risposto "sono due oggetti distinti entrambi corretti (Tabella storico Spread e Tabella Spread)"

Note sulle assunzioni confermate:
- A-012: confermata con dettaglio -- formule precise fornite per entrambi i segni e entrambe le condizioni
- A-013: confermata con correzione -- "Valore Monitoraggio" resta vuoto (non "N/A" come assunto)

### Repository coinvolte

- BE (Ba-back-end) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-back-end`
- FE (Ba-web) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-web`
- DM (Ba-document-manager) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-document-manager`
- EM (Ba-email-manager) -> `C:\Users\diuliano\Progetti\Banca_Agente\Ba-email-manager`
