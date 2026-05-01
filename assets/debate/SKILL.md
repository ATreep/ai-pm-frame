Create an **agent team** to run a high-friction, structured product/design/policy debate using the user-provided materials.

---

## Objective

Simulate a realistic, adversarial, multi-stakeholder debate that exposes real trade-offs, conflicts, risks, and implementation implications, then converge on a practical conclusion.

## Core Principle

This skill is valid only when the debate is executed as a **live, turn-by-turn multi-agent process** with:

1. **One canonical main-thread transcript shown to the user**
2. **One canonical markdown output file saved on disk**
3. **One committed turn at a time**

Do not simulate the debate inside one monolithic answer. Do not compress rounds. Do not summarize instead of relaying turns.

---

## Platform Truths (Read First)

- **Only `@main` controls the user-visible main thread.** Teammate agents do not directly publish canonical debate turns to the user.
- **Teammates communicate completed turns to `@main` via `SendMessage`.**
- **`@main` is the only writer of the transcript file.** Teammates must never write the output file.
- Because of tool/event timing, "real-time" means: **the moment `@main` receives a teammate turn, `@main` must relay it before any other visible content in its next visible output and append it to the transcript before advancing the debate.**

Any instruction that treats teammate DMs as equivalent to main-thread output is invalid. The main thread is the source of truth.

---

## Execution Mechanism (Mandatory)

You **must** use the **Agent Team** mechanism (`TeamCreate` + `Agent` tools) to run the debate with strict procedural adherence.

1. Create a dedicated team for the debate.
2. Spawn one teammate agent per generated role, including the facilitator/moderator.
3. Every teammate uses the same model family as `@main` unless the user explicitly says otherwise.
4. `@main` orchestrates every turn in strict sequence with `SendMessage`.
5. **Exactly one teammate may speak at a time.** No parallel speaking within a round.
6. `@main` must not request the next turn until the current turn is both:
   - received from the speaking teammate, and
   - committed by `@main` to the main-thread transcript and output file.
7. Do not terminate before the minimum round count unless the user explicitly asks to stop early.

---

## Pre-Debate Preparation

### Relative Resource

You must read **all** relevant files in `debate-materials/` before starting the debate. No need to read files outside of `debate-materials/`.

### Debate Report

Save the full debate transcript and the moderator's final report to:

```
debate-outputs/debate_output_<debate-theme>_<YYMMDDhhmm>.md
```

- Use an English filename; the file contents may be Chinese.
- Before the rounds, summarize the given materials and clearly state the debate topic.
- Introduce each agent role and its stance before the discussion begins.
- End with a moderator conclusion that evaluates the arguments and recommends practical next steps.
- Do NOT care about other files except the one that you just created in `debate-outputs/`.

---

## Canonical Order and Commit Protocol

### Canonical turn order

The debate has a fixed canonical order:

- Round 1: Role 1 -> Role 2 -> Role 3 -> ... -> Facilitator
- Round 2: Role 1 -> Role 2 -> Role 3 -> ... -> Facilitator
- Continue until the required minimum round count is completed

`@main` must define the role order before Round 1 and never silently change it later.

### Turn lifecycle

A turn is complete only after all four steps succeed:

1. `@main` sends a turn request to the next scheduled teammate.
2. The teammate replies with one completed turn via `SendMessage`.
3. `@main` immediately relays that exact turn text to the user in the main thread.
4. `@main` appends that exact turn text to the markdown transcript file.

Only after step 4 may `@main` trigger the next speaker.

### Non-negotiable restrictions

- Do **not** ask multiple teammates for the same round in parallel.
- Do **not** batch several teammate turns into one main-thread relay.
- Do **not** hold turns in memory waiting for a full round to finish.
- Do **not** rewrite a teammate turn into an abstract before relaying it.
- Do **not** write the final report before every prior turn is committed.

---

## Dynamic Role Generation (Required)

Based on the user input, generate **at least 5 distinct roles** that match the domain and stakeholders in the material.

### Role-generation requirements

1. **Role source is strictly the user's material**: roles must be inferred from the actual people, institutions, incentives, and constraints named or implied in the source.
2. **Required one product role and one commercial role**: include one product-development role and one commercialization/operations role in addition to domain roles.
3. Create roles with genuinely different incentives and constraints.
4. Each role must include:
   - **Role name**
   - **Perspective / mandate**
   - **Primary success metric**
   - **Non-negotiable constraints**
   - **Default stance**
5. Spawn one teammate agent per role.
6. Include one facilitation role that enforces structure and synthesizes outcomes.

---

## Debate Behavior: Adversarial but Purposeful

All non-facilitator roles must:

1. Hold a clear and stable stance
2. Directly challenge named opposing claims
3. Use sharp rebuttals instead of polite agreement loops
4. Expose contradictions, weak assumptions, and hidden costs
5. Defend their position with evidence, logic, and stakeholder incentives
6. Escalate from surface disagreement to hard trade-off analysis

The debate should feel like a real high-stakes review meeting, not a collaborative brainstorm.

---

## Procedural Rigor (Critical Mandate)

**The debate process structure is mandatory.**

- **Strict turn sequencing**: every round includes every role in the predefined order.
- **Minimum round completion**: run at least **10 full rounds**.
- **Progressive depth**: early rounds surface positions, middle rounds expose contradictions, late rounds force decision criteria, metrics, failure modes, and implementation consequences.
- **Quality enforcement**: every turn must pass the turn-quality standard.
- **Real tension**: make conflicts explicit; do not suppress them to fake consensus.

---

## Debate Depth and Round Control

1. **Round count**: minimum **10** rounds, maximum **30** rounds.
2. **No early stop**: do not stop before 10 rounds unless the user explicitly instructs it.
3. **Later rounds must deepen** into:
   - edge cases and boundary conditions
   - failure modes and unintended consequences
   - incentive conflicts and misalignments
   - feasibility, cost, timeline, and resource constraints
   - measurement, validation, and decision criteria
4. **No fake progress**: do not repeat earlier claims without adding a new constraint, implication, rebuttal, or evidence point.
5. **Force unresolved tensions into trade-off form** when they remain irreconcilable.

---

## Turn Request and Turn Payload

### Turn request from `@main`

When `@main` prompts a teammate, it must specify:

- current round number
- role name
- required rebuttal target(s)
- any newly introduced constraint for this round
- the exact response format

### Required teammate payload

Each teammate reply sent to `@main` must contain exactly one completed turn in this structure:

```md
TURN_ID: R[round]-[role-slug]
ROUND: [number]
ROLE: [role name]
Round [number] — [Role Name]
[120-220 words of argument + rebuttal]
```

Rules:
- One `SendMessage` = one turn.
- Do not send partial drafts.
- Do not send a summary plus a full version.
- Do not send multiple alternative turns.
- `TURN_ID`, `ROUND`, and `ROLE` are coordination metadata for `@main`; they are optional in the visible relay and optional in the canonical transcript block.
- The visible relay and the canonical transcript block must preserve the same argumentative body text as the teammate payload.

---

## Turn Quality Standard (Every Round)

Each turn **must** satisfy all five criteria:

1. A clear, position-specific claim
2. Evidence, reasoning, or precedent supporting the claim
3. Direct rebuttal to at least one named opposing claim from the same or prior round
4. One concrete implication grounded in incentives, cost, risk, timeline, or measurement
5. Logical coherence with the role's persona, constraints, and success metric

**Minimum turn length**: **120-220 words**.

If a turn fails the quality gate, `@main` must reject it, request regeneration from the same role, and must not advance the sequence.

---

## Main-Thread Relay Rules

### Before Round 1

`@main` must output in the main thread:

```md
## Debate Topic
[one concise statement of the debated claim]

## Material Summary
[brief summary of the provided materials]

## Generated Roles
- [Role Name]: [mandate, metric, constraints, stance]
- ...

## Canonical Debate Order
1. [Role 1]
2. [Role 2]
3. [Role 3]
- ...
```

### During the debate

Whenever `@main` receives a valid teammate turn, `@main` must relay it immediately in this exact visible format:

```md
Round [N] — [Role Name]
[full teammate turn text]
```

Restrictions:
- Do **not** wait for the round to finish.
- Do **not** combine multiple turns in one relay.
- Do **not** replace the original turn with an abstract.
- Do **not** skip visible relay just because the turn is already saved to file.

If delivery timing forces a slight delay, the next visible main-thread output must be the pending turn relay, not a summary or a later orchestration step.

---

## Transcript File Protocol (Mandatory)

The full debate transcript and final report must be saved to the output path required by the calling command.

### Single-writer rule

- `@main` is the **only** transcript writer.
- Teammates must never create, append, or overwrite the transcript file.

### When to write

`@main` must create the file before Round 1 with:
- debate topic
- source-material summary
- generated roles
- canonical role order

After that, `@main` must append each committed turn **immediately after relaying it**.

### Canonical file contents

The file must contain, in order:

1. Title
2. Debate topic
3. Date / timezone if required by caller
4. Material summary
5. Generated roles
6. Canonical debate order
7. Full verbatim turn-by-turn transcript for every round
8. Final report generated by the facilitator/moderator and appended by `@main`

### Canonical transcript block for each turn

Each appended turn block must use this exact structure:

```md
Round [N] — [Role Name]
[full teammate turn text]
```

Do not save only summaries. Do not save only the final conclusions. Do not reconstruct the transcript later from memory.

### Commit discipline

For each turn, the order is:
1. receive teammate message
2. relay to main thread
3. append same text to file
4. continue orchestration

Do not reorder these steps.

---

## Final Report (After Final Round Only)

After the last required round is committed, `@main` may request the facilitator/moderator's final report and must append that report to the same output file. `@main` remains the only file writer.

The final report must contain:

1. **Debated Claim**
2. **Stakeholder Map**
3. **Top Arguments Supporting Action**
4. **Top Arguments Opposing or Constraining Action**
5. **Key Conflicts and Trade-offs**
6. **Turning Points**
7. **Decision Assessment**
   - Validity: High / Medium / Low / Invalid
   - Confidence: High / Medium / Low
   - Rationale grounded in debate evidence
8. **Open Questions / Unknowns**
9. **Recommended Next Step**
10. **Decision Trigger Metrics**

The final report must be appended to the same output file after the transcript.

---

## Failure Modes to Avoid

These are violations, not style preferences:

- claiming teammate windows satisfy the main-thread realtime requirement
- waiting for a full round before relaying turns
- batching several turns for convenience
- running roles in parallel and reconstructing order afterward
- saving only a polished summary instead of the verbatim debate
- letting teammates write the transcript file
- writing the final report before all turns are committed
- silently changing role order mid-debate
- accepting low-quality turns to move faster

If any of these happen, the debate is procedurally invalid.

---

## Quality Guardrails

- **No role collapse**: roles must remain distinct in incentives, constraints, and tone.
- **No artificial consensus**: disagreement must remain explicit where real tensions persist.
- **No shallow summaries before required rounds are complete**.
- **No procedural shortcuts**: do not skip, compress, or batch turns for efficiency.
- **Conclusions must stay evidence-linked and decision-oriented**.

## Language

Both the agent names and the debate content are in **Simplified Chinese**.
