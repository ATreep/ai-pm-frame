# init-pm

## Overview

Initialize an AI Product Manager (AI-PM) workspace for a specific product. Creates a concise PM role skill and all necessary workspace scaffolding for multi-agent product debates and PRD generation.

## Prerequisites

**STOP** if the user has not provided product materials and the workspace has no source code to analyze. You need at least one input source.

Two input modes are supported. Use whichever is available, or combine both.

### Mode A — Document Input

The user provides external materials directly. At least one of:
- Product Requirement Document (PRD)
- Product advertisements or marketing copy
- Official website content / product docs
- User stories / feature specs
- Competitive analysis docs

### Mode B — Source Code Analysis

The user points to an existing codebase (or runs `/init-pm` inside one). Claude reads the project source to infer product context.

**Requires the One Workflow plugin** (`/plugin install one-workflow@one-workflow`). If not installed, prompt the user to install it first.

Required signals (at least one must be present):
- `README.md` or equivalent project documentation
- `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, or other manifest with a project description
- A recognizable source tree (`src/`, `app/`, `lib/`, etc.) with meaningful code
- Config files that reveal product intent (routes, schemas, feature flags, i18n)

### Combined Mode

When both documents and source code are available, use documents as the primary source and source code to fill gaps, validate claims, and surface undocumented features or constraints.

## Steps

### 1. Validate Input

Determine which input mode applies:

**If the user provided documents** — proceed with Mode A.

**If no documents were provided but a source tree exists** — proceed with Mode B. First check that the One Workflow plugin is installed. If not, tell the user:

```
Source-based initialization requires the One Workflow plugin. Install it first:

/plugin marketplace add ATreep/one-workflow
/plugin install one-workflow@one-workflow
```

Then scan the workspace for:
1. `README.md` or similar documentation files
2. Manifest files (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, etc.)
3. Source directory structure (`src/`, `app/`, `lib/`)
4. Route definitions, schema files, API endpoints, feature flags, config files

**If neither documents nor usable source code exist**, respond:

```
I need product context to create an AI-PM role. Please provide at least one of:

Documents:
- PRD (Product Requirement Document)
- Ads / marketing copy
- Official website or product documentation
- User stories or feature specifications
- Competitive analysis

Or point me to an existing codebase with:
- A README or project docs
- A manifest file (package.json, Cargo.toml, pyproject.toml, go.mod, etc.)
- A recognizable source tree
```

Then **STOP** and wait for the user.

### 2. Generate Spec from Source (Mode B Only)

When source code is the input, generate detailed project documentation using the One Workflow plugin:

1. Invoke the `one-workflow:new-docs` skill to analyze the codebase and generate comprehensive spec documents.
2. Redirect all output to the `spec/` folder (not the default `docs/`). After the skill runs, move or rename `docs/` to `spec/` if the skill wrote to `docs/`.
3. The spec should cover: architecture, modules, runtime, data model, integrations, operations, and implementation guide — one focused file per topic.

Expected spec structure:
```
spec/
├── README.md               # Navigation index
├── architecture.md          # System structure, boundaries, flow
├── modules.md               # Module catalog with purpose and ownership
├── runtime.md               # Setup, scripts, execution model, env/config
├── data-model.md            # Storage schemas, entities, relationships
├── integrations.md          # Third-party APIs/services and contracts
├── operations.md            # Deploy, health checks, failure paths
├── implementation-guide.md  # End-to-end rebuild blueprint
└── modules/                 # One file per significant module
    └── <module-name>.md
```

**For Mode A (documents only):** Skip this step. No spec generation is needed when the user provides documents directly.

### 3. Create Product Manager Role Skill

Read all provided materials (and spec if generated), then generate a **concise** `pm-role.md` in the current workspace root.

The PM role should be **short and focused** — around 100-200 lines. A long role file is a burden on the LLM context window and dilutes the PM's effectiveness. The role captures product knowledge at a high level; deep technical details belong in the spec.

If spec documents were generated in step 2, include this note at the top of the PM role (after frontmatter):

> **Project Spec Available:** Detailed architecture, module, and technical documentation is available in the `spec/` folder. Reference `spec/README.md` for navigation. Use spec documents when you need deep technical context for decisions.

The role skill must include:

#### Product Foundation
- **Product vision** — 1-2 sentence summary of what the product is and who it serves
- **Target audience** — personas, segments, key user needs
- **Core features** — prioritized list with brief descriptions
- **Competitive landscape** — key differentiators, market positioning
- **Product highlights** — what the product does well, unique strengths
- **Known shortcomings** — gaps, limitations, areas for improvement

#### Architecture Summary (High-Level Only)
- **System overview** — monolith/microservices/serverless, main tech stack
- **Key modules** — list of major modules with one-line descriptions (no deep dive)
- **Integration points** — external services and APIs the product depends on
- **Technical constraints** — framework, language, deployment targets, performance budgets

> **Note:** If the product was initialized from source code, detailed architecture documentation is in the `spec/` folder. The PM role provides the high-level picture; use spec for module-level detail.

#### PM Decision Framework
- **Prioritization approach** — how this PM ranks work (impact vs. effort, user value, strategic alignment)
- **Quality bar** — what "good enough" means for this product
- **Communication style** — how this PM writes PRDs, gives feedback, handles disagreements

#### Constraints & Risks
- Technical, legal, timeline, or budget constraints
- Key risks and known trade-offs

Structure as a system-prompt-ready role file:
- Start with `# AI Product Manager — <Product Name>`
- Use imperative, persona-driven language ("You are a PM who...")
- **Target 100-200 lines** — concise enough to fit comfortably in context, detailed enough to guide decisions
- Use frontmatter:
  ```yaml
  ---
  name: pm-role-<kebab-case-product-name>
  description: AI Product Manager persona for <Product Name>. Use when orchestrating product debates, prioritizing features, or making trade-off decisions.
  ---
  ```

### 4. Create Workspace Files

Create the following in the **current workspace root**:

#### `.claude/settings.json`
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "32000"
  },
  "skipDangerousModePermissionPrompt": true
}
```

#### `claude-pm.sh`
```sh
#!/bin/zsh
claude --dangerously-skip-permissions --system-prompt "$(cat ./pm-role.md)"
```

After writing, make it executable:
```bash
chmod +x claude-pm.sh
```

#### `debate-materials/` folder
Create an empty directory named `debate-materials`.

#### `prd-outputs/` folder
Create an empty directory named `prd-outputs`.

### 5. Install Skills (Project-Local)

Both the `debate` and `gen-prd` skills must be installed as **project-local skills**, not at the user level (`~/.claude/`).

For each skill:

1. Read the skill template from the plugin assets:
   ```
   assets/debate/SKILL.md
   assets/gen-prd/SKILL.md
   ```
   (These files are relative to the plugin root directory.)

2. Write the content to the project's local skills directory:
   ```
   .claude/skills/debate/SKILL.md
   .claude/skills/gen-prd/SKILL.md
   ```

3. Create the directory structure if it does not exist:
   ```bash
   mkdir -p .claude/skills/debate
   mkdir -p .claude/skills/gen-prd
   ```

**Do NOT** copy skills to `~/.claude/skills/` or any other user-level location. They belong exclusively in the project scope so they can be version-controlled and customized per product.

**Spec reference note:** If spec documents were generated in step 2, add the following note to the beginning of the installed `gen-prd/SKILL.md` (after the first heading):

> **Project Spec Available:** Detailed architecture, module, and technical documentation is available in the `spec/` folder. Reference `spec/README.md` for navigation. When generating the PRD, consult spec documents for accurate module descriptions, data models, and integration details.

### 6. Teach the User

After setup is complete, output exactly:

---

**Your AI-PM workspace is ready.**

Start the AI Product Manager by running:

```bash
./claude-pm.sh
```

**Available skills:**

| Skill | Purpose |
|-------|---------|
| `/debate` | Run a multi-agent adversarial debate to surface trade-offs and validate decisions |
| `/gen-prd` | Generate a comprehensive, production-grade PRD from product context |

Before starting a debate:
1. Copy any debate materials (research docs, competitor briefs, user interview notes, etc.) into the `debate-materials/` folder.
2. Start a debate session with the `/debate` command.

Generate a PRD at any time with `/gen-prd`. It reads from `pm-role.md`, debate outputs, materials, and source code.

Both skills are installed as project-local skills at `.claude/skills/`. They will not appear in your global `~/.claude/` folder.

[If spec was generated, add:]
> **Spec documents** are available in the `spec/` folder. These contain detailed architecture, module, and technical documentation generated from your source code. Reference them when you need deep context for product decisions.

Example workflow:
```bash
# 1. Start the AI-PM shell
./claude-pm.sh

# 2. Run a debate to validate a product decision
/debate Should we build a mobile app or a PWA?

# 3. Generate a PRD from the accumulated context
/gen-prd
```

---

## When NOT to Use

- Do not use if the user only wants a generic PM role without product context.
- Do not use if neither documents nor a meaningful source codebase are available.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Proceeding without product materials or source code | Stop and ask the user for docs or point them to a codebase |
| Writing a generic PM role | Derive specifics from the provided materials or source code; every section should reference the actual product |
| Writing a PM role that's too long | Keep to 100-200 lines; deep technical details go in spec/, not the role file |
| Skipping spec generation for source-based init | Use `one-workflow:new-docs` to generate spec documents when source code is provided |
| Forgetting to check One Workflow plugin | Verify the plugin is installed before attempting source-based initialization |
| Forgetting to make `claude-pm.sh` executable | Run `chmod +x claude-pm.sh` after writing |
| Putting files in the wrong directory | All files (`pm-role.md`, `.claude/settings.json`, `claude-pm.sh`, `debate-materials/`, `prd-outputs/`) go in the workspace root |
| Installing skills at user level | Skills go to `.claude/skills/<name>/SKILL.md` in the project root, **never** to `~/.claude/skills/` |
