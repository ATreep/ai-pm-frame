# init-pm

## Overview
Initialize an AI Product Manager (AI-PM) workspace for a specific product. This command creates the product manager role skill and all necessary workspace scaffolding so the user can run multi-agent product debates and generate PRDs.

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

The user points to an existing codebase (or runs `/init-pm` inside one). Claude reads the project source to infer product context. Useful when no formal docs exist yet.

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

**If no documents were provided but a source tree exists** — proceed with Mode B. Scan the workspace for:
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

### 2. Create Product Manager Role Skill

#### Mode A — From Documents

Read all provided materials, then generate a comprehensive `pm-role.md` in the **current workspace root**.

#### Mode B — From Source Code

Analyze the project source to extract product context:

1. **Read documentation files** — `README.md`, `CHANGELOG.md`, `docs/`, `CONTRIBUTING.md`
2. **Read manifest files** — extract name, description, dependencies, scripts
3. **Scan directory structure** — infer features from folder names, module organization, route files
4. **Read key source files** — look for API routes, schema definitions, feature modules, i18n keys, config constants
5. **Inspect test files** — test names and structure reveal intended user flows and feature boundaries

From this analysis, infer:
- What the product does and who it serves
- Core features and their priority (based on code complexity, test coverage, dependency weight)
- Technical constraints (framework, language, deployment targets)
- Business model signals (auth flows, payment modules, admin panels, analytics integrations)

#### Both Modes

Generate the same `pm-role.md` output regardless of input mode. When source code is the only input, note in the role file that the PM persona is code-inferred and may benefit from refinement once formal docs exist.

The role skill must include:
- **Product vision** — 1-2 sentence summary of what the product is and who it serves
- **Target audience** — personas, segments, key user needs
- **Core features** — prioritized list with brief descriptions
- **Business goals** — KPIs, success metrics, monetization model
- **Competitive landscape** — key differentiators, market positioning
- **Constraints & risks** — technical, legal, timeline, budget
- **Decision-making principles** — how this PM prioritizes trade-offs (speed vs. quality, user delight vs. revenue, etc.)
- **Tone & style** — how this PM communicates (e.g., data-driven, user-empathetic, direct, diplomatic)

Structure as a system-prompt-ready role file:
- Start with `# AI Product Manager — <Product Name>`
- Use imperative, persona-driven language ("You are a PM who...")
- Keep under 800 lines, 400-600 typical
- Use frontmatter:
  ```yaml
  ---
  name: pm-role-<kebab-case-product-name>
  description: AI Product Manager persona for <Product Name>. Use when orchestrating product debates, prioritizing features, or making trade-off decisions.
  ---
  ```

### 3. Create Workspace Files

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

### 4. Install Skills (Project-Local)

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

### 5. Teach the User

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
| Ignoring source code when docs are thin | Use Mode B to fill gaps from the codebase — routes, schemas, and tests reveal real product intent |
| Taking source-inferred features at face value | Cross-check: dead code, prototypes, and abandoned features exist. Prioritize signals with tests and active usage |
| Forgetting to make `claude-pm.sh` executable | Run `chmod +x claude-pm.sh` after writing |
| Putting files in the wrong directory | All files (`pm-role.md`, `.claude/settings.json`, `claude-pm.sh`, `debate-materials/`, `prd-outputs/`) go in the workspace root |
| Installing skills at user level | Skills go to `.claude/skills/<name>/SKILL.md` in the project root, **never** to `~/.claude/skills/` |
