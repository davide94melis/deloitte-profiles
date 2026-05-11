# Deloitte Profiles

Centralized project profiles for the BR Skills ecosystem. Each project maintains a `profile.json` that captures its tech stack, conventions, design system, domain knowledge, and custom agents. The BR skills (`br-analyzer`, `br-executor`, `br-reviewer`, `br-updater`, `br-debug`) read these profiles to generate context-aware output without repeated manual configuration.

## Directory Structure

```
deloitte-profiles/
├── profile-schema.json          # JSON Schema for profile validation
├── README.md
├── project-alpha/
│   ├── profile.json             # Project profile (validated against schema)
│   ├── agents/                  # Optional: custom agent .md files
│   │   └── domain-expert.md
│   └── references/              # Optional: design refs, mockups, style guides
│       └── palette.png
├── project-beta/
│   ├── profile.json
│   └── agents/
│       └── legacy-migrator.md
└── ...
```

Each project gets its own directory. The directory name is typically the project slug (lowercase, hyphens).

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

Use the `br-profile-setup` skill to scaffold a new project profile interactively:

```
/br-profile-setup
```

This walks through the tech stack, conventions, and domain details, then writes the `profile.json` and creates the project directory in this repo.

### Local Configuration

Each project repository contains a `.br-local.json` file that points to its profile:

```json
{
  "profilo": "project-alpha",
  "profiles_repo": "C:/Users/davmelis/Documents/MyGitHub/deloitte-profiles"
}
```

- `profilo` -- the directory name inside this repo
- `profiles_repo` -- absolute path to this repository on the developer's machine

### Auto-Sync

BR skills pull the latest profiles via `git pull` before reading. No manual sync is needed as long as profiles are committed and pushed after changes.

### Auto-Maintenance

`br-analyzer` updates the profile automatically when it detects new conventions, dependencies, or domain terms during gap analysis. Changes are committed back to this repo.

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

These agents are loaded by BR skills when working on the project, providing specialized behaviors (e.g., domain validation rules, migration patterns).

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

All `profile.json` files are validated against `profile-schema.json` (JSON Schema Draft 2020-12). BR skills validate on load and report errors before proceeding.
