# Stima BR — monitoring

Data stima: 2026-05-13
Modalita': dettagliata (dal piano di implementazione, precisione ±10-15%)
Deadline target: 2026-05-29

## Team

| Dev | Seniority | Area | Disponibilita' | Claude Code |
|---|---|---|---|---|
| Davide | Senior | BE | 100% | Si |
| Carmine | Senior | FE | 100% | Si |
| Alexios | Mid | BE+FE | 100% | Si |
| Georgios | Junior | FE | 100% | Si |
| Ahmad | Junior | BE | 100% | Si |
| Adham | Junior | FE | 100% | Si |

## Stato attuale

| Metrica | Valore |
|---|---|
| Task totali piano | 31 |
| Task completate | 16 (52%) |
| Task rimanenti | 15 |
| Effort rimanente (piano) | 46.0 gg |

## Ottimizzazioni applicate

Le seguenti riassegnazioni sono state simulate per ridurre il critical path:
- T-012 (Dettaglio pratica ISP): Adham (Junior) → **Carmine** (Senior FE)
- T-016 (Spread FE): Georgios (Junior) → **Carmine** (Senior FE)
- T-017 (Flusso Deloitte base): Alexios (Mid) → **Davide** (Senior BE)

## Scenario Selezionato: Realistico

| Metrica | Valore |
|---|---|
| Effort totale | 60.58 giorni/persona |
| Durata con team attuale | 20 giorni lavorativi |
| Data inizio | 2026-05-13 |
| Data fine stimata | 2026-06-17 |
| Deadline | 2026-05-29 |
| Delta | +14 giorni lavorativi (FUORI DEADLINE) |
| Utilizzo team medio | 39% |

### Bottleneck

1. **CRITICO — Catena seriale Davide (GL13-26)**: Da T-MERGE-W2 in poi, solo Davide lavora. 14 giorni lavorativi dove il team di 6 si riduce a 1 persona.
2. **T-018 dominio_nuovo (GenAI)**: 6 giorni seriali con moltiplicatore 1.3x. Rischio di ulteriori rallentamenti.
3. **T-015 Ahmad Junior (6gg)**: Bottleneck iniziale che ritarda T-016 e tutta la catena T-MERGE-W2.
4. **Team idle 10+ giorni**: Carmine, Alexios, Georgios, Ahmad, Adham completamente fermi dalla meta' del progetto.

### Causa radice

Il problema non e' l'effort (il team ha 78 gg/persona di capacita' in 13 giorni lavorativi). La causa e' la **catena seriale di dipendenze** che attraversa un singolo dev (Davide):

```
T-015 → T-016 → T-MERGE-W2 → T-017 → T-018 → T-MERGE-FINAL → T-024
                     solo Davide da qui in poi (14 gg seriali)
```

### Allocazione Team (scenario realistico)

| Dev | Seniority | Area | Task assegnate | Giorni occupato | Giorni totali | Utilizzo |
|---|---|---|---|---|---|---|
| Davide | Senior | BE | T-023, T-MERGE-W2, T-017, T-018, T-MERGE-FINAL, T-024 | 19 | 26 | 73% |
| Carmine | Senior | FE | T-010, T-012, T-016 | 12 | 26 | 46% |
| Alexios | Mid | BE+FE | T-014, T-020 | 9 | 26 | 35% |
| Georgios | Junior | FE | T-022 | 9 | 26 | 35% |
| Ahmad | Junior | BE | T-015 | 6 | 26 | 23% |
| Adham | Junior | FE | T-019, T-021 | 9 | 26 | 35% |

### Timeline realistico (giorno per giorno)

| GL | Data | Davide | Carmine | Alexios | Georgios | Ahmad | Adham |
|---:|---|---|---|---|---|---|---|
| 1 | 13/05 | T-023 [1/5] | T-010 [1/4] | T-014 [1/4] | T-022 [1/9] | T-015 [1/6] | T-019 [1/5] |
| 2 | 14/05 | T-023 [2/5] | T-010 [2/4] | T-014 [2/4] | T-022 [2/9] | T-015 [2/6] | T-019 [2/5] |
| 3 | 15/05 | T-023 [3/5] | T-010 [3/4] | T-014 [3/4] | T-022 [3/9] | T-015 [3/6] | T-019 [3/5] |
| — | WE | — | — | — | — | — | — |
| 4 | 18/05 | T-023 [4/5] | T-010 DONE | T-014 DONE | T-022 [4/9] | T-015 [4/6] | T-019 [4/5] |
| 5 | 19/05 | T-023 DONE | T-012 [1/4] | T-020 [1/5] | T-022 [5/9] | T-015 [5/6] | T-019 DONE |
| 6 | 20/05 | idle | T-012 [2/4] | T-020 [2/5] | T-022 [6/9] | T-015 DONE | T-021 [1/4] |
| 7 | 21/05 | idle | T-012 [3/4] | T-020 [3/5] | T-022 [7/9] | idle | T-021 [2/4] |
| 8 | 22/05 | idle | T-012 DONE | T-020 [4/5] | T-022 [8/9] | idle | T-021 [3/4] |
| — | WE | — | — | — | — | — | — |
| 9 | 25/05 | idle | T-016 [1/4] | T-020 DONE | T-022 DONE | idle | T-021 DONE |
| 10 | 26/05 | idle | T-016 [2/4] | idle | idle | idle | idle |
| 11 | 27/05 | idle | T-016 [3/4] | idle | idle | idle | idle |
| 12 | 28/05 | idle | T-016 DONE | idle | idle | idle | idle |
| 13 | 29/05 | T-MERGE-W2 DONE | idle | idle | idle | idle | idle |
| — | WE | — | — | — | — | — | — |
| 14 | 01/06 | T-017 [1/3] | idle | idle | idle | idle | idle |
| 15 | 02/06 | T-017 [2/3] | idle | idle | idle | idle | idle |
| 16 | 03/06 | T-017 DONE | idle | idle | idle | idle | idle |
| 17 | 04/06 | T-018 [1/6] | idle | idle | idle | idle | idle |
| 18 | 05/06 | T-018 [2/6] | idle | idle | idle | idle | idle |
| — | WE | — | — | — | — | — | — |
| 19 | 08/06 | T-018 [3/6] | idle | idle | idle | idle | idle |
| 20 | 09/06 | T-018 [4/6] | idle | idle | idle | idle | idle |
| 21 | 10/06 | T-018 [5/6] | idle | idle | idle | idle | idle |
| 22 | 11/06 | T-018 DONE | idle | idle | idle | idle | idle |
| 23 | 12/06 | T-MERGE-FINAL DONE | idle | idle | idle | idle | idle |
| 24 | 15/06 | T-024 [1/3] | idle | idle | idle | idle | idle |
| 25 | 16/06 | T-024 [2/3] | idle | idle | idle | idle | idle |
| 26 | 17/06 | T-024 DONE | idle | idle | idle | idle | idle |

## Scenari a Confronto

| Metrica | Ottimistico | Realistico | Pessimistico |
|---|---|---|---|
| Effort totale | 47.4 gg/p | 60.6 gg/p | 79.1 gg/p |
| Durata | 16 gg lav | 20 gg lav | 27 gg lav |
| Data fine | 03/06/2026 | 17/06/2026 | 01/07/2026 |
| Dentro deadline? | FUORI (+3 gg) | FUORI (+14 gg) | FUORI (+23 gg) |
| Utilizzo team | 49% | 39% | 33% |

## Effort dettagliato per task e scenario

| ID | Nome | Owner finale | Sen. | Effort Ott. | Effort Real. | Effort Pess. |
|---|---|---|---|---:|---:|---:|
| T-010 | Dashboard monitoring FE | Carmine | Sr | 2.80 | 3.50 | 4.55 |
| T-012 | Dettaglio pratica ISP | Carmine | Sr | 3.20 | 4.00 | 5.20 |
| T-014 | Upload CC + storico docs | Alexios | Mid | 3.12 | 3.90 | 5.07 |
| T-015 | Spread BE | Ahmad | Jr | 4.32 | 5.40 | 7.02 |
| T-016 | Spread FE | Carmine | Sr | 3.20 | 4.00 | 5.20 |
| T-MERGE-W2 | Merge checkpoint Wave 2 | Davide | Sr | 0.80 | 1.00 | 1.30 |
| T-017 | Flusso Deloitte base | Davide | Sr | 2.40 | 3.00 | 3.90 |
| T-018 | GenAI + modifica covenant | Davide | Sr | 3.60 | 5.20 | 6.80 |
| T-019 | Eccezioni ISP | Adham | Jr | 3.60 | 4.50 | 5.85 |
| T-020 | Upload COVNO | Alexios | Mid | 3.51 | 4.68 | 6.24 |
| T-021 | Pagine Scaduti/InScadenza | Adham | Jr | 2.88 | 3.60 | 4.68 |
| T-022 | Mailing List completo | Georgios | Jr | 7.20 | 9.00 | 11.70 |
| T-023 | Email + scheduled jobs | Davide | Sr | 3.60 | 4.80 | 6.40 |
| T-MERGE-FINAL | Merge finale | Davide | Sr | 0.80 | 1.00 | 1.30 |
| T-024 | Integration + UAT | Davide | Sr | 2.40 | 3.00 | 3.90 |
| **TOTALE** | | | | **47.43** | **60.58** | **79.11** |

## Raccomandazioni per rientrare nella deadline

| # | Azione | Risparmio | Impatto |
|---|---|---|---|
| 1 | Parallelizzare T-017/T-018 (BE+FE separati) | 4-6 gg | Basso — richiede split task |
| 2 | Rilassare T-MERGE-W2 (T-017 parte senza merge completo) | 4-6 gg | Medio — rischio conflitti |
| 3 | Differire T-018 GenAI post-deadline | 4-7 gg | Basso — flusso manuale resta funzionante |
| 4 | Differire T-022 Mailing List + UAT ridotto | 4-5 gg | Medio |

### Combinazione suggerita (Opzione 2 + 3)

| Scenario | Senza tagli | Con tagli | Delta |
|---|---|---|---|
| Ottimistico | 03/06 | 28/05 DENTRO (-1 gg) | |
| Realistico | 17/06 | 05/06 FUORI (+5 gg) | |
| Pessimistico | 01/07 | 16/06 FUORI (+12 gg) | |

## Parametri Utilizzati

### Effort per complessita'
| Complessita' | Giorni/persona |
|---|---|
| Bassa | 0.5 |
| Media | 1.0 |
| Alta | 2.0 |
| Molto Alta | 3.5 |

Nota: in modalita' dettagliata, l'effort e' preso direttamente dal piano (non dai bucket di complessita').

### Moltiplicatori seniority
| Seniority | Moltiplicatore |
|---|---|
| Senior | 1.0x |
| Mid | 1.3x |
| Junior | 1.8x |

### Moltiplicatori rischio per scenario
| Scenario | Standard | Integrazione | Dominio nuovo |
|---|---|---|---|
| Ottimistico | 0.8x | 0.9x | 0.9x |
| Realistico | 1.0x | 1.2x | 1.3x |
| Pessimistico | 1.3x | 1.6x | 1.7x |

### Calibrazione storica
Fattore: 1.0x (default — nessun BR completato end-to-end)

## Storico di Riferimento

Nessun dato storico sufficiente. La directory plans/done/ contiene un solo BR (export-login) senza file di progresso.
Dati parziali dal BR monitoring in-progress (16/31 task completate in 4 giorni lavorativi) suggeriscono rapporto 0.66x sulle task fondazionali, ma il campione e' troppo piccolo per una calibrazione affidabile.
