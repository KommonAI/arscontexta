# Conversation Summary: Tutorial Skill, Operational Use, and the Vault-Project Gap

## Context
This conversation explored the arscontexta knowledge management system, starting from the tutorial skill and drilling into a fundamental architectural question about how vault knowledge should relate to project work.

## What We Covered

### 1. The Tutorial Skill (`/tutorial`)
An interactive 5-step onboarding walkthrough with three tracks (researcher, manager, personal). Each step follows WHY → DO → SEE and creates real content in the vault:
1. **Capture** — create a note from a genuine thought
2. **Discover** — create a second note, find connections
3. **Process** — extract atomic insights from raw material (demonstrates `/reduce`)
4. **Maintain** — run health checks on tutorial notes
5. **Reflect** — review the graph you built, handoff to productive use

State persists in `ops/tutorial-state.yaml` so the tutorial survives across sessions.

### 2. Operational Use of the Graph
After processing raw material, the graph is used through:
- **`/reflect`** — discover connections, update MOCs, add wiki-links
- **`/ask`** — synthesize answers across notes (not just retrieval)
- **`/next`** — surface the single most valuable action based on vault state
- **`/health`** — diagnose vault health across 8 categories
- **`/graph`** — structural analysis (bridges, clusters, hubs, triangles)

The operational cycle: `/reduce → /reflect → /health → /next → (repeat)`

### 3. What the Graph Actually Produces (Beyond Maintenance)
The graph is an instrument, not the product. It enables:
- **Assembly over creation** — write papers, PRDs, memos by composing accumulated claims rather than starting from scratch
- **Query-driven decision support** — `/ask` synthesizes across notes you can't hold in working memory
- **Impact analysis** — when an assumption changes, trace every decision that depends on it
- **Pattern detection at scale** — `/reflect` surfaces reasoning no single note contains
- **Domain-specific outputs**: literature reviews that auto-flag stale claims, architectural memory replacing tribal knowledge, consistency-verified worldbuilding, mood-trigger correlations across journal entries

Core principle: "Notes are skills — curated knowledge injected into context when relevant." Adding a note adds a capability.

### 4. The Vault-Project Relationship (Key Unresolved Question)

**The tension:** If knowledge compounds in the vault, shouldn't the agent always have access to it — even during project work? Should projects be nested inside the vault?

**Why nesting doesn't work:**
- **Context is a budget.** Loading vault identity, MOCs, goals, and skills into a project session wastes tokens that should be spent on project logic.
- **Skill budgets are hard ceilings.** Claude Code gives ~2% of context for skill descriptions. Vault skills + project tooling would blow the budget.
- **Skills are vault-bound.** `/reduce`, `/reflect`, `/ask` read `ops/derivation-manifest.md` for vocabulary transforms. They expect to run inside the vault.
- **Fresh context per task is a feature.** The system enforces session isolation to protect attention quality.

**The intended (inferred) workflow:**
- Knowledge work happens in the vault repo
- Project work happens in project repos
- You context-switch between them manually
- Insights from project work get captured back into the vault

**The gap:** The system provides no documented bridge pattern between vault and project work:
- No `arscontexta --export [query]` to inject vault knowledge into project context
- No cross-repo skill invocation
- No mechanism for vault learning to surface during project work
- No explicit tradeoff analysis documenting *why* separation is the right choice
- The theoretical foundations exist (identity constitution, attention management, session isolation) but the deliberate architectural position on vault-project relationship is never stated

This is arguably the system's biggest missing piece: a deliberate, documented position on the vault-project relationship rather than leaving users to infer it from architectural constraints.

## Open Questions for Continued Discussion
1. Should the system document an explicit vault-project relationship model?
2. Is there a lightweight bridge pattern that doesn't blow the context budget? (e.g., exporting a focused subset of vault claims relevant to a specific project)
3. Should `/setup` include a decision point: "vault-as-home-base vs vault-as-background-knowledge"?
4. How should the operational learning loop span repos — when project work surfaces an insight, what's the friction-minimized path back to the vault?
