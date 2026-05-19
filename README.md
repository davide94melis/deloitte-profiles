# Deloitte Profiles

Repository centralizzato dei profili progetto e degli artefatti dei Business Requirement (BR) per l'ecosistema SDLC Skills. Ogni progetto mantiene un `constitution/profile.json` che cattura il suo tech stack, le convenzioni, il design system, la conoscenza di dominio e gli agenti custom. Le SDLC skills (`sdlc-analyzer`, `sdlc-executor`, `sdlc-reviewer`, `sdlc-updater`, `sdlc-debug`, `sdlc-estimator`, `sdlc-clarify`, `sdlc-progress-report`) leggono questi profili per generare output contestualizzato senza configurazione manuale ripetuta.

Tutti gli artefatti BR (piani di implementazione, report di gap, progressi, stime, bug report, screenshot) sono centralizzati qui, non piu' nelle repo del codice. Le repo applicative restano focalizzate sul solo codice sorgente.

## Directory Structure

```
deloitte-profiles/
├── profile-schema.json          # JSON Schema for profile validation
├── README.md
├── <nome-progetto>/
│   ├── constitution/
│   │   └── profile.json         # configurazione progetto (tech stack, dominio, design system)
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
│       │       └── STIMA_BR.md / .xlsx
│       ├── in-progress/         # BR in lavorazione
│       │   └── <data>_<nome-br>/
│       │       ├── (tutti i file da todo +)
│       │       ├── PROGRESS.md
│       │       ├── BUG_REPORT_BR.md
│       │       ├── PROGRESS.xlsx
│       │       └── screenshots/
│       └── done/                # BR completati (archivio storico)
│           └── <data>_<nome-br>/
└── ...
```

Ogni progetto ha la propria directory. Il nome della directory e' tipicamente lo slug del progetto (minuscolo, trattini).

### Ciclo di vita di un BR

Un BR si muove tra le cartelle `plans/` in base al suo stato:

1. **`todo/`** -- BR appena ricevuto. Contiene la documentazione funzionale convertita in markdown sotto `requirements/`, l'eventuale review qualita' (`CLARIFY.md/.docx`), il gap report (`PLAN.md`), il piano di implementazione (`TASKS.md`) e la stima (`STIMA_BR.md` / `.xlsx`).
2. **`in-progress/`** -- BR in lavorazione. Contiene tutti i file di `todo/` piu' il file di progresso (`PROGRESS.md`), gli eventuali bug raccolti (`BUG_REPORT_BR.md`), l'Excel di avanzamento (`PROGRESS.xlsx`) e gli screenshot prodotti durante il debug.
3. **`done/`** -- BR completati e validati. Archivio storico utile per consultazione e riuso di pattern.

Lo spostamento tra cartelle e' gestito dalle skill (`sdlc-analyzer` crea in `todo/`, `sdlc-executor` sposta in `in-progress/` alla prima esecuzione, la chiusura del BR sposta in `done/`).

## Profile Sections

| Section | Content | Required |
|---------|---------|----------|
| `project` | Name, client, description | Yes |
| `tech_stack` | Backend and frontend: language, framework, database, ORM, build tool | Yes |
| `conventions` | Package structure, architectural layers, base entity, API prefix, test framework, naming, branch/commit conventions | No |
| `design_system` | Palette, typography, spacing, components, reference files | No |
| `domain` | Glossary, business rules, entity state machines | No |
| `custom_agents` | Paths to custom agent `.md` files for project-specific AI behaviors | No |

## Usage

### Initial Setup

Use the `sdlc-profile-setup` skill to scaffold a new project profile interactively:

```
/sdlc-profile-setup
```

Esegue auto-detect sul codebase, fa domande guidate su dominio e design system, e scrive `constitution/profile.json` insieme alla struttura di cartelle (`agents/`, `references/`, `plans/{todo,in-progress,done}/`) nel progetto.

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

Le SDLC skills risolvono il profilo come `<profiles_repo>/<profilo>/constitution/profile.json` e cercano gli artefatti BR sotto `<profiles_repo>/<profilo>/plans/`.

### Auto-Sync

Le SDLC skills eseguono `git pull` sui profili prima di leggere. Non e' necessaria alcuna sincronizzazione manuale finche' i profili vengono committati e pushati dopo le modifiche.

### Auto-Maintenance

`sdlc-analyzer` aggiorna il profilo automaticamente quando rileva nuove convenzioni, dipendenze o termini di dominio durante la gap analysis. Le modifiche vengono committate in questo repo.

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

Tutti i file `constitution/profile.json` sono validati contro `profile-schema.json` (JSON Schema Draft 2020-12). Le SDLC skills validano al caricamento e segnalano gli errori prima di procedere.
