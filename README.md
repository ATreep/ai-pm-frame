# ai-pm-frame

An AI-driven Product Manager for Claude Code that transforms product materials into actionable PRDs through multi-agent stakeholder debates.

## How It Works

The plugin provides one skill (`/init-pm`) that scaffolds an entire AI-PM workspace. Once initialized, two project-local skills become available inside that workspace:

| Skill | Scope | What It Does |
|-------|-------|-------------|
| `/init-pm` | Plugin-level | Analyzes documents or source code, generates a PM persona, and scaffolds the workspace |
| `/debate` | Project-local (installed by init) | Runs a live, turn-by-turn adversarial debate with 5+ AI stakeholder roles |
| `/gen-prd` | Project-local (installed by init) | Synthesizes all context into a production-grade PRD |

## Installation

```
/plugin marketplace add ATreep/ai-pm-frame
/plugin install ai-pm-frame@ai-pm-frame
```

Verify with `/plugin marketplace list`.

**Source code mode** requires an additional plugin:

```
/plugin marketplace add ATreep/one-workflow
/plugin install one-workflow@one-workflow
```

If you only provide documents, no extra plugins are needed.

## Quick Start

### 1. Initialize an Empty Project

Create an empty directory for your product workspace, open Claude Code there, and run:

```
/init-pm
```

Two input modes — use whichever is available, or combine both:

- **Documents:** PRD, ads, user stories, competitive analysis, product docs
- **Source code:** Run inside an existing codebase. Claude scans source, generates a detailed `spec/` folder, and derives the PM role

This creates:

```
your-project/
├── pm-role.md                  # Concise AI PM persona (100-200 lines)
├── claude-pm.sh                # Launch script
├── .claude/
│   ├── settings.json           # Agent team config
│   └── skills/
│       ├── debate/SKILL.md     # Debate skill (project-local)
│       └── gen-prd/SKILL.md    # PRD generation skill (project-local)
├── debate-materials/           # Drop debate inputs here
├── prd-outputs/                # Generated PRDs land here
└── spec/                       # (Source mode only) Architecture, modules, data model...
```

### 2. Add Debate Materials

Copy research docs, competitor briefs, user interview notes, or any relevant files into `debate-materials/`.

### 3. Start the AI-PM Shell

```bash
./claude-pm.sh
```

### 4. Run a Debate

```
/debate Should we build a mobile app or a PWA for the next quarter?
```

### 5. Generate a PRD

```
/gen-prd
```

The PRD reads from `pm-role.md`, `spec/`, debate outputs, materials, and source code to produce a structured document with requirements, user stories, data models, success metrics, and more.

## How the Debate Works

The debate is a real-time, sequential multi-agent process — not a summary generator. Only available inside a workspace initialized by `/init-pm`.

1. **Role generation** — Claude reads your materials and generates 5+ stakeholder roles with distinct incentives, metrics, and constraints. Always includes at least one product role and one commercial role.
2. **Sequential turns** — Each role speaks one at a time in a fixed canonical order. No parallel speaking, no batching.
3. **Adversarial by design** — Roles challenge named opposing claims with evidence, expose contradictions, and escalate into hard trade-off analysis.
4. **Progressive depth** — Early rounds surface positions, middle rounds expose contradictions, late rounds force decision criteria and implementation consequences.
5. **Verbatim transcript** — Every turn is relayed to you in real time and saved to a markdown file.
6. **Final report** — After 10+ rounds, the facilitator synthesizes a structured decision assessment with recommended next steps.

Output is saved to `debate-outputs/debate_output_<topic>_<YYMMDDhhmm>.md`.

## How PRD Generation Works

`/gen-prd` is a project-local skill installed by `/init-pm`. It synthesizes all available sources in priority order:

| Priority | Source | What It Provides |
|----------|--------|------------------|
| 1 | `pm-role.md` | Product vision, target audience, core features, business goals |
| 2 | `spec/` | Architecture, modules, data models, integrations |
| 3 | Source code | Implemented features, API surfaces, technical constraints |

The PRD includes 17 mandatory sections: executive summary, problem statement, personas, user stories with acceptance criteria, functional and non-functional requirements, data model, API design, release strategy, success metrics, risks, and more.

Every requirement must pass the "engineer test": can an engineer estimate and build this without a follow-up meeting?

Output is saved to `prd-outputs/prd_<product-name>_<YYMMDDhhmm>.md`.

## Project Structure

```
ai-pm-frame/
├── .claude-plugin/
│   ├── plugin.json             # Plugin metadata
│   └── marketplace.json        # Marketplace listing
├── assets/
│   ├── debate/SKILL.md         # Debate skill template (copied to project on init)
│   └── gen-prd/SKILL.md        # PRD generation skill template (copied on init)
├── skills/
│   └── init-pm/SKILL.md        # Workspace initialization skill
└── README.md
```

Skills in `assets/` are installed as **project-local** skills (`.claude/skills/<name>/SKILL.md`), not at the user level (`~/.claude/`). This keeps them version-controllable and customizable per product.

## Requirements

- Claude Code with agent team support (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`)
- Product context from one of: documents (PRD, ads, docs) or an existing codebase
- **One Workflow plugin** — only required for source code mode. Not needed for documents.

## License

MIT
