# init-pm

## Overview
Initialize an AI Product Manager (AI-PM) workspace for a specific product. This command creates the product manager role skill and all necessary workspace scaffolding so the user can run multi-agent product debates.

## Prerequisites

**STOP** if the user has not provided product materials. You cannot proceed without them.

Required materials (at least one):
- Product Requirement Document (PRD)
- Product advertisements or marketing copy
- Official website content / product docs
- User stories / feature specs
- Competitive analysis docs

## Steps

### 1. Validate Input

If no product materials were provided, respond:

```
I need product materials to create an AI-PM role. Please provide at least one of:
- PRD (Product Requirement Document)
- Ads / marketing copy
- Official website or product documentation
- User stories or feature specifications
- Competitive analysis
```

Then **STOP** and wait for the user.

### 2. Create Product Manager Role Skill

Read all provided materials, then generate a comprehensive `pm-role.md` in the **current workspace root**.

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

### 4. Teach the User

After setup is complete, output exactly:

---

**Your AI-PM workspace is ready.**

Start the AI Product Manager by running:

```bash
./claude-pm.sh
```

Before starting a debate:
1. Copy any debate materials (research docs, competitor briefs, user interview notes, etc.) into the `debate-materials/` folder.
2. Start a debate session with the `/debate` command.

Example workflow:
```bash
# 1. Start the AI-PM shell
./claude-pm.sh

# 2. Inside the session, kick off a debate
/debate ...a topic to debate...
```

---

## When NOT to Use

- Do not use if the user only wants a generic PM role without product context.
- Do not use if the user has not provided any product materials.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Proceeding without product materials | Stop and ask the user for PRD, ads, docs, or website |
| Writing a generic PM role | Derive specifics from the provided materials; every section should reference the actual product |
| Forgetting to make `claude-pm.sh` executable | Run `chmod +x claude-pm.sh` after writing |
| Putting files in the wrong directory | All files (`pm-role.md`, `.claude/settings.json`, `claude-pm.sh`, `debate-materials/`) go in the workspace root |
