# ai-pm-frame

An AI-driven Product Manager framework for Claude Code. Transform product materials into structured multi-agent stakeholder debates that surface real trade-offs, conflicts, and actionable conclusions.

## What It Does

`ai-pm-frame` gives Claude Code three skills:

| Skill | Purpose |
|-------|---------|
| `/init-pm` | Scaffolds a product-specific AI-PM workspace from docs, research materials, or existing source code |
| `/debate` | Runs a live, turn-by-turn adversarial debate across 5+ generated stakeholder roles |
| `/gen-prd` | Generates a comprehensive, production-grade PRD from product context and source code |

The debate is not a summary generator. It is a real-time, sequential multi-agent process where each role defends its incentives, challenges opposing claims, and forces hard trade-off analysis — like a high-stakes product review meeting.

The PRD generator synthesizes all available context — PM role, debate outputs, materials, and source code — into a structured document that engineering teams can directly implement from.

## Installation

### 1. Add the Marketplace

```
/plugin marketplace add ATreep/ai-pm-frame
```

### 2. Install the Plugin

```
/plugin install ai-pm-frame@ai-pm-frame
```

### 3. Verify

```
/plugin marketplace list
```

You should see `ATreep-ai-pm-frame` in the list.

## Quick Start

### Step 1 — Initialize a Product Workspace

Open Claude Code in your project directory, then run:

```
/init-pm
```

`/init-pm` supports two input modes:

**Mode A — From documents:** Provide PRD, ads, docs, user stories, or competitive analysis. No extra plugins needed.

**Mode B — From source code:** Run `/init-pm` inside an existing codebase. Claude scans your source code, generates detailed spec documents (via the [One Workflow](https://github.com/ATreep/one-workflow) plugin), and creates a concise PM role.

> **Mode B requires the One Workflow plugin.** Install it before running `/init-pm` on a source codebase:
>
> ```
> /plugin marketplace add ATreep/one-workflow
> /plugin install one-workflow@one-workflow
> ```
>
> If you only provide documents (Mode A), no extra plugins are needed.

Both modes can be combined — documents are the primary source, source code fills gaps and validates claims.

This creates:

```
your-project/
├── pm-role.md                  # Concise AI PM persona derived from your materials
├── claude-pm.sh                # Launch script
├── .claude/
│   ├── settings.json           # Agent team config
│   └── skills/
│       ├── debate/
│       │   └── SKILL.md        # Debate skill (project-local)
│       └── gen-prd/
│           └── SKILL.md        # PRD generation skill (project-local)
├── debate-materials/           # Drop debate inputs here
├── prd-outputs/                # Generated PRDs land here
└── spec/                       # (Mode B only) Detailed project spec from source analysis
    ├── README.md               # Spec navigation index
    ├── architecture.md         # System structure and flow
    ├── modules.md              # Module catalog
    └── ...                     # Runtime, data model, integrations, operations
```

### Step 2 — Add Debate Materials

Copy research docs, competitor briefs, user interview notes, or any relevant files into `debate-materials/`.

### Step 3 — Start the AI-PM Shell

```bash
./claude-pm.sh
```

### Step 4 — Run a Debate

Inside the AI-PM session:

```
/debate Should we build a mobile app or a PWA for the next quarter?
```

### Step 5 — Generate a PRD

After running debates (or directly after init), generate a comprehensive PRD:

```
/gen-prd
```

The PRD reads from `pm-role.md`, `spec/` (if present), debate outputs, materials, and source code to produce a structured document with requirements, user stories, data models, success metrics, and more.

## How the Debate Works

The `/debate` skill orchestrates a structured adversarial discussion:

1. **Role generation** — Claude reads your materials and generates 5+ stakeholder roles with distinct incentives, metrics, and constraints.
2. **Sequential turns** — Each role speaks one at a time in a fixed canonical order. No parallel speaking, no batching.
3. **Adversarial by design** — Roles challenge named opposing claims with evidence, expose contradictions, and escalate into hard trade-off analysis.
4. **Progressive depth** — Early rounds surface positions, middle rounds expose contradictions, late rounds force decision criteria and implementation consequences.
5. **Verbatim transcript** — Every turn is relayed to you in real time and saved to a markdown file.
6. **Final report** — After 10+ rounds, the facilitator synthesizes a structured decision assessment with recommended next steps.

Output is saved to:

```
debate-outputs/debate_output_<topic>_<timestamp>.md
```

## Project Structure

```
ai-pm-frame/
├── .claude-plugin/
│   ├── plugin.json             # Plugin metadata
│   └── marketplace.json        # Marketplace listing
├── assets/
│   ├── debate/
│   │   └── SKILL.md            # Debate skill template (copied to project on init)
│   └── gen-prd/
│       └── SKILL.md            # PRD generation skill template (copied to project on init)
├── skills/
│   └── init-pm/
│       └── SKILL.md            # Workspace initialization skill
└── README.md
```

Skills in `assets/` are installed as **project-local** skills (`.claude/skills/<name>/SKILL.md`), not at the user level (`~/.claude/`). This keeps them version-controllable and customizable per product.

## Plugin Management

```bash
# Update to latest version
/plugin marketplace update ai-pm-frame

# Remove the marketplace
/plugin marketplace remove ai-pm-frame
```

## Requirements

- Claude Code with agent team support (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`)
- Product context from one of: documents (PRD, ads, docs) or an existing codebase (README, manifests, source tree)
- **[One Workflow](https://github.com/ATreep/one-workflow) plugin** — only required for Mode B (source-based init). Not needed if you only provide documents.

## License

MIT
