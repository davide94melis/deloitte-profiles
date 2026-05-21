# Deloitte Profiles

Repository centralizzato dei profili progetto e degli artefatti dei Business Requirement (BR) per l'ecosistema SDLC Skills. Ogni progetto mantiene una coppia di file in `constitution/` — `CONST.json` (principi e standard di archetipo) e `PROFILE.json` (dettagli specifici del progetto: tech stack, convenzioni, design system, dominio, agenti custom). Le SDLC skills (`sdlc-analyzer`, `sdlc-executor`, `sdlc-reviewer`, `sdlc-updater`, `sdlc-debug`, `sdlc-estimator`, `sdlc-clarify`, `sdlc-progress-report`) leggono questi profili per generare output contestualizzato senza configurazione manuale ripetuta.

Tutti gli artefatti BR (piani di implementazione, report di gap, progressi, stime, bug report, screenshot) sono centralizzati qui, non piu' nelle repo del codice. Le repo applicative restano focalizzate sul solo codice sorgente.

## Modalita' Operative (da maggio 2026)

Le skill SDLC supportano **due modalita'** mutuamente esclusive per progetto, discriminate via auto-detect dal file `.br-local.json` presente nella repo applicativa:

| Modalita' | Trigger `.br-local.json` | Storage profilo + plan | Stati plan |
|---|---|---|---|
| **Legacy** (questa repo) | `{"profiles_repo": "...", "profilo": "..."}` | `deloitte-profiles/<progetto>/` | `todo/`, `in-progress/`, `done/` |
| **Standalone** (repo dedicata) | `{"project_repo": "...", "project_name": "..."}` | `<project_repo>/` con `dataset/` Solaria-side | `draft/`, `todo/`, `in-progress/`, `done/` |

- **Nuovi progetti**: adottano la **modalita' standalone** (raccomandato). Una repo Git per progetto (es. `banca-agente`), con cartella `dataset/` popolata Solaria-side (branding, glossario, attori, perimetro), plus la nuova area `plans/draft/` dove Solaria authora l'AFU prima dell'handoff al team tecnico via GitHub Git Trees API.
- **Progetti esistenti** (es. `banca-agente` in questa repo): continuano in **modalita' legacy** senza modifiche. La transizione e' graduale: nessun obbligo di migrazione, le skill restano duali e auto-detect dal `.br-local.json` di ciascuna repo applicativa.

L'orchestrazione del flusso e' descritta in `claude-flow/docs/Fasi-New-way-of-working.md` (2 fasi composite: Fase 1 Pre-Coding [1a/b/c] + Fase 2 Coding & Test [2a/b/c]). I contratti d'interscambio Solaria↔Claude Code (schema `afu-manifest.json` v2 esteso, formato Excel bug con colonna `origine`, struttura playbook test) sono documentati in `claude-flow/docs/SOLARIA_SDLC_INTEGRATION.md`.

## Directory Structure

```
deloitte-profiles/
├── profile-schema.json          # JSON Schema for profile validation
├── README.md
├── <nome-progetto>/
│   ├── constitution/
│   │   ├── CONST.json           # principi/standard di archetipo (OWASP, WCAG, test coverage, ecc.)
│   │   └── PROFILE.json         # dettagli specifici progetto (tech stack, dominio, design system)
│   ├── agents/                  # agenti custom .md per questo progetto (opzionale)
│   ├── references/              # mockup, screenshot, style guide, codice gold-standard
│   └── plans/                   # ciclo di vita dei Business Requirement
│       ├── todo/                # BR in attesa di lavorazione
│       │   └── <data>_<nome-br>/
│       │       ├── requirements/   # documentazione BR convertita in markdown
│       │       ├── CLARIFY.md
│       │       ├── CLARIFY.docx
│       │       ├── PLAN.md
│       │       ├── TASKS.md
│       │       └── ESTIMATE.md / .xlsx
│       ├── in-progress/         # BR in lavorazione
│       │   └── <data>_<nome-br>/
│       │       ├── (tutti i file da todo +)
│       │       ├── PROGRESS.md
│       │       ├── BUG_REPORT.md
│       │       ├── PROGRESS.xlsx
│       │       └── screenshots/
│       └── done/                # BR completati (archivio storico)
│           └── <data>_<nome-br>/
└── ...
```

Ogni progetto ha la propria directory. Il nome della directory e' tipicamente lo slug del progetto (minuscolo, trattini).

### Ciclo di vita di un BR

Un BR si muove tra le cartelle `plans/` in base al suo stato:

1. **`todo/`** -- BR appena ricevuto. Contiene la documentazione funzionale convertita in markdown sotto `requirements/`, l'eventuale review qualita' (`CLARIFY.md/.docx`), il gap report (`PLAN.md`), il piano di implementazione (`TASKS.md`) e la stima (`ESTIMATE.md` / `.xlsx`).
2. **`in-progress/`** -- BR in lavorazione. Contiene tutti i file di `todo/` piu' il file di progresso (`PROGRESS.md`), gli eventuali bug raccolti (`BUG_REPORT.md`), l'Excel di avanzamento (`PROGRESS.xlsx`) e gli screenshot prodotti durante il debug.
3. **`done/`** -- BR completati e validati. Archivio storico utile per consultazione e riuso di pattern.

Lo spostamento tra cartelle e' gestito dalle skill (`sdlc-analyzer` crea in `todo/`, `sdlc-executor` sposta in `in-progress/` alla prima esecuzione, la chiusura del BR sposta in `done/`).

## Constitution Files

Ogni progetto ha due file nella sua folder `constitution/`:

### `CONST.json` — principi e standard di archetipo

Contiene le regole non-negoziabili e gli standard di qualità ripetibili tra progetti simili. Mai modificato in automatico dalle skill — è policy stabile gestita dall'utente.

| Section | Content | Required |
|---|---|---|
| `inviolable_principles` | Security (OWASP), accessibility (WCAG), responsiveness, data privacy (GDPR) | Almeno uno |
| `quality_standards` | Test coverage minima, error handling, logging, performance budget | No |
| `code_style` | Max function/file lines, nesting depth, no magic numbers | No |
| `git_workflow` | Branch pattern, commit convention | No |
| `architectural_patterns` | Layered separation, API response envelope, AAA test pattern, input validation | No |

Validato contro `const-schema.json`. Template di default in `claude-flow/skills/sdlc-profile-setup/_const-template.json`.

### `PROFILE.json` — dettagli specifici del progetto

Contiene tutto ciò che è unico del progetto. Auto-aggiornato da `sdlc-analyzer` quando rileva nuove convenzioni dal codice.

| Section | Content | Required |
|---|---|---|
| `project` | Name, client, description | Yes |
| `tech_stack` | Backend, frontend, repositories (multi-repo con sigle), infrastructure, integrations | Yes |
| `conventions` | Package structure, layers, base entity, API prefix, test framework, branch/commit specifici, naming | No |
| `design_system` | Palette, typography, spacing, components, reference files | No |
| `domain` | Glossary, business rules, entity state machines | No |
| `custom_agents` | Paths to custom agent `.md` files | No |

Validato contro `profile-schema.json`.

## Usage

### Initial Setup

Use the `sdlc-profile-setup` skill to scaffold a new project profile interactively:

```
/sdlc-profile-setup
```

Esegue auto-detect sul codebase, fa domande guidate su dominio e design system, e scrive `constitution/CONST.json` e `constitution/PROFILE.json` insieme alla struttura di cartelle (`agents/`, `references/`, `plans/{todo,in-progress,done}/`) nel progetto.

### Local Configuration

Ogni repository di progetto contiene un file `.br-local.json` che punta al suo profilo:

```json
{
  "profilo": "project-alpha",
  "profiles_repo": "C:/Users/davmelis/Documents/MyGitHub/deloitte-profiles"
}
```

- `profilo` -- nome della directory di progetto in questo repo
- `profiles_repo` -- path assoluto a questo repository sulla macchina dello sviluppatore

Le SDLC skills risolvono **entrambi** i file:
- `<profiles_repo>/<profilo>/constitution/CONST.json`
- `<profiles_repo>/<profilo>/constitution/PROFILE.json`

Entrambi devono esistere per il funzionamento. Se manca uno dei due, le skill segnalano errore e suggeriscono `python claude-flow/scripts/migrate-profile-split.py --apply` (per profili in formato legacy) oppure `/sdlc-profile-setup` (per nuovi progetti). Gli artefatti BR vengono cercati sotto `<profiles_repo>/<profilo>/plans/`.

### Auto-Sync

Le SDLC skills eseguono `git pull` sui profili prima di leggere. Non e' necessaria alcuna sincronizzazione manuale finche' i profili vengono committati e pushati dopo le modifiche.

### Auto-Maintenance

`sdlc-analyzer` aggiorna automaticamente **solo `PROFILE.json`** quando rileva nuove convenzioni, dipendenze o termini di dominio durante la gap analysis. `CONST.json` è policy stabile e va modificato manualmente dall'utente.

Le modifiche a PROFILE.json vengono committate in questo repo. Quando una skill rileva che il codice viola un principio CONST, lo segnala come finding nel `PLAN.md` sotto la sezione "Violazioni principi CONST rilevate", NON modifica CONST.json.

## Custom Agents

Place project-specific agent `.md` files in the project's `agents/` directory and list their relative paths in the `custom_agents` array:

```json
{
  "custom_agents": [
    "agents/domain-expert.md",
    "agents/legacy-migrator.md"
  ]
}
```

These agents are loaded by SDLC skills when working on the project, providing specialized behaviors (e.g., domain validation rules, migration patterns).

## Reference Files

Store design references, mockups, exported tokens, or style guides in the `references/` directory. Point to them from `design_system.reference_files`:

```json
{
  "design_system": {
    "reference_files": [
      "references/palette.png",
      "references/typography-guide.pdf"
    ]
  }
}
```

## Schema Validation

I file di costituzione sono validati contro JSON Schema Draft 2020-12:
- `CONST.json` → `const-schema.json`
- `PROFILE.json` → `profile-schema.json`

Le SDLC skills validano al caricamento e segnalano gli errori con il path del campo invalido prima di procedere.
