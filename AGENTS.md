# openlore: LLM Agent Instructions

## Repository authority and context

This file is the canonical project instruction entry point for all coding agents.
`CLAUDE.md` is a compatibility entry point that imports this file. Keep shared
project rules here and task-specific reference material in the linked documents.

When starting unfamiliar project work, read `openspec/specs/overview/spec.md`.
Read `.openlore/analysis/CODEBASE.md` if it exists and is relevant; it is a generated,
git-ignored digest that a fresh clone may lack. Inspect source before relying on
cached details. Read `CONTRIBUTING.md` for development setup and required checks.

## Code discovery and OpenLore

Read a named file directly before expanding discovery. Use native tools for file
reads and literal searches. Use `orient` for an unfamiliar module or a task spanning
modules when structural context is needed; reuse that context while the relevant
source is unchanged. Already-read files need no repeated orientation.

OpenLore must be registered as an MCP server (`openlore mcp`). If it or its index is
unavailable or stale, use source search and disclose material coverage limits.
Refresh an index only when a structural query needs it; read-only audits must not
initialize an index. A freshness (Epistemic Lease) note means re-orient if relying
on cached cross-module structure. Treat it as information, not a failure.

For choosing tools to trace callers, assess changes, verify claims, work with specs
or memories, or coordinate work across repos, read
[`docs/agents/openlore-tools.md`](docs/agents/openlore-tools.md).
Tool results can be incomplete: cite their limits, distinguish test reachability
from verified behavior, and inspect source when a claim remains unverifiable.

The default MCP preset is `substrate`: navigation, spec-workflow composites, and
read tools `recall`, `verify_claim`, and `blast_radius`. It excludes the write tools
`remember` and `record_decision`. `navigation` is the lean navigation alternative;
`memory`, `verify`, `federation`, `coordination`, `minimal` (`--minimal`), and `full`
(`--all-tools`) expose other selections. Check the active tool list rather than
assuming this checkout has been wired with any particular preset.

For contributor setup, use the repository build and
`node dist/cli/index.js install --preset full` as documented in `CONTRIBUTING.md`.
When `record_decision` is unavailable, the architectural-decision workflow below
can use `openlore decisions record --title "..." --rationale "..."` instead.

## MCP and governance authoring

Navigation and governance share one structural graph, anchored-fact store, and
freshness lease. When adding an MCP tool:

- Declare its output class (`conclusion` or `explicit-topology`) in
  `src/core/services/mcp-handlers/tool-contract.ts`. Conclusion tools compute the
  answer on the server instead of making the agent traverse a graph.
- Declare one capability family in `TOOL_CAPABILITY_FAMILY`: `navigate`, `change`,
  `remember`, `verify`, `coordinate`, or `federate`. Keep tools with distinct
  conclusions distinct and cross-reference adjacent tools in the same family.
- Run the companion `tool-contract.test.ts` checks along with affected checks.

When adding a governance finding, register its stable code, source default class,
and description in `FINDING_CODE_REGISTRY` in
`src/core/services/mcp-handlers/enforcement-policy.ts`. Emit `GovernanceFinding`
with `code`, `severity`, `source`, `subject`, and `message`. The source owns intrinsic
severity; `enforcement.policy` owns the enforcement class. Most findings default
to advisory; preserve any explicitly stricter source default.

## Chat replies

Use concise, clear replies in the user's language. For English replies, follow
ASD-STE100 Simplified Technical English where practical; explain technical ideas
at an accessible level and lead with the result.

## Agent Role

When reverse-engineering OpenSpec specifications, act as a "code archaeologist": document what the code actually does, grounded in source evidence.

## Core Principles

1. **Archaeology over Creativity**: Extract truth from code, don't invent features
2. **Evidence-based**: Every requirement traces back to actual implementation
3. **OpenSpec-native**: Output follows OpenSpec conventions exactly
4. **Never stop early**: Do not say "Task completed", "Done", or "Finished" without having re-read the original request and verified every part of it is addressed. If any file change, test, or wiring step remains, keep working.

## OpenSpec reverse-engineering workflow

### Step 1: Codebase Survey

Analyze the project to understand its structure:

**Detect Project Type:**
- `package.json` → Node.js/TypeScript
- `pyproject.toml` / `setup.py` → Python
- `go.mod` → Go
- `Cargo.toml` → Rust
- `pom.xml` / `build.gradle` → Java

**Find High-Value Files (prioritize these):**
- Schema/model files (entities, types, interfaces)
- Service files (business logic)
- Route/controller files (API surface)
- Config files (settings, environment)
- Entry points (main, index, app)

**Identify Domains:**
- Directory structure patterns (src/users/, src/orders/)
- File naming conventions (user-service, order-controller)
- Import clusters (files that heavily import each other)

**Detect Tech Stack:**
- Frameworks: Express, NestJS, FastAPI, Django, etc.
- Databases: PostgreSQL, MongoDB, Redis, etc.
- Auth: JWT, OAuth, session-based, etc.

### Step 2: Deep Analysis

For each domain, extract:

**Entities:**
- Data structures and their properties
- Type definitions and interfaces
- Relationships between entities

**Behaviors:**
- Operations and mutations
- Business rules and validations
- Side effects (emails, payments, notifications)

**API Surface:**
- HTTP endpoints and methods
- Request/response shapes
- Authentication requirements

### Step 3: Generate Specifications

Create this directory structure:

```
openspec/
├── config.yaml
└── specs/
    ├── overview/spec.md
    ├── {domain-1}/spec.md
    ├── {domain-2}/spec.md
    └── architecture/spec.md
```

**Spec File Template:**

```markdown
# {Domain} Specification

> Generated by openlore on {date}
> Source files: {list of analyzed files}

## Purpose

{2-3 sentences describing this domain}

## Requirements

### Requirement: {RequirementName}

The system SHALL {behavior description}.

Use RFC 2119 keywords:
- **SHALL/MUST**: Required behavior
- **SHOULD**: Recommended behavior
- **MAY**: Optional behavior

#### Scenario: {ScenarioName}
- **GIVEN** {precondition}
- **WHEN** {action}
- **THEN** {expected outcome}

## Technical Notes

- **Implementation**: `{file paths}`
- **Dependencies**: {related domains}
```

**Formatting Rules:**
1. Requirements use RFC 2119 keywords (SHALL, MUST, SHOULD, MAY)
2. Scenarios use `####` heading level
3. Scenarios use Given/When/Then with bold labels
4. No delta markers — these are baseline specs

### Step 4: Create/Update Config

**config.yaml format:**

```yaml
schema: spec-driven
context: |
  {Brief project description}

  Tech stack: {detected technologies}
  Architecture: {detected pattern}

openlore:
  generatedAt: "{timestamp}"
  domains:
    - {domain-1}
    - {domain-2}
```

### Step 5: Drift Detection

When specs already exist and code has changed, check for **spec drift** — divergence between the codebase and its specifications.

**When to Check for Drift:**
- Before committing code (pre-commit hook via `openlore drift --install-hook`)
- When reviewing a branch or PR
- When asked to validate that specs are up to date

**Drift Detection Process:**

1. **Identify what changed** — Compare the current branch against the base (main/master) using git:
   - Which source files were added, modified, deleted, or renamed?
   - Filter out noise: test files, generated files, lock files, static assets, CI configs

2. **Map changes to specs** — For each changed file, determine which spec domain covers it:
   - Check `> Source files:` header in each `spec.md`
   - Check `**Implementation**:` references in Technical Notes
   - Infer from directory structure (file in `src/auth/` → auth domain)

3. **Detect four categories of drift:**

   | Category | Meaning | Severity |
   |----------|---------|----------|
   | **Gap** | Code changed but its spec was not updated | error (large changes on key files), warning (moderate), info (small) |
   | **Stale** | Spec references a deleted or renamed file | error (deleted), warning (renamed) |
   | **Uncovered** | New source file has no matching spec domain | warning (key files), info (utility files) |
   | **Orphaned Spec** | Spec declares source files that no longer exist on disk | warning |

4. **Report issues** with actionable suggestions:
   - Which file changed and by how much (+/- lines)
   - Which spec domain is affected
   - What the engineer should do to resolve it

**Drift Detection CLI:**
```bash
openlore drift                    # Check drift against main branch
openlore drift --base develop     # Compare against develop branch
openlore drift --json             # JSON output for CI pipelines
openlore drift --fail-on error    # Only fail on error-level issues
openlore drift --install-hook     # Install as git pre-commit hook
openlore drift --uninstall-hook   # Remove the hook
```

**Key Rule**: Drift detection is static analysis by default (no LLM required). It compares git changes against spec file mappings. Use `--use-llm` only when deeper semantic analysis is needed.

## OpenSpec output checklist

Before completing, verify:

- [ ] `openspec/specs/overview/spec.md` — System summary
- [ ] `openspec/specs/{domain}/spec.md` — One per domain
- [ ] `openspec/specs/architecture/spec.md` — System structure
- [ ] All requirements use RFC 2119 keywords
- [ ] All scenarios use Given/When/Then format
- [ ] `openspec/config.yaml` exists with context
- [ ] No spec drift — run `openlore drift` to verify specs match code

## Example Session

**User**: "Analyze this codebase and generate OpenSpec specs"

**Agent Actions**:
1. Survey directory structure, package.json, etc.
2. Identify 3-6 core domains from code organization
3. Read high-value files in each domain
4. Extract entities, behaviors, and APIs
5. Generate spec.md for each domain
6. Create overview and architecture specs
7. Write/update config.yaml

**Agent Response**:
```
Created OpenSpec specifications:

openspec/
├── config.yaml
└── specs/
    ├── overview/spec.md      (system summary)
    ├── users/spec.md         (5 requirements, 8 scenarios)
    ├── orders/spec.md        (7 requirements, 12 scenarios)
    ├── payments/spec.md      (4 requirements, 6 scenarios)
    └── architecture/spec.md  (system structure)

Next steps:
- Run `openspec validate --all` to verify structure
- Run `openlore drift --install-hook` to catch future drift
- Review generated specs for accuracy
- Refine requirements and add edge cases
```

## Usage

**ChatGPT/Claude Web:**
Copy this entire document into the system prompt or paste at conversation start.

**API Integration:**
Include as the system message in your API calls.

**Claude Code:**
See `.claude/skills/openlore.md` for the skill version.

**OpenSpec Native:**
See `skills/openspec-skill.md` for OpenSpec skill format.

---

## Mistral Vibe (local CLI, no API key)

openlore supports the `mistral-vibe` provider, which routes generation through the local
[`vibe` CLI](https://github.com/withvibe/vibe) — a standalone binary that runs Mistral models
locally. No API key or network connection is required.

### Setup

```bash
# Install the vibe CLI (see https://github.com/withvibe/vibe for your platform)
# Then configure openlore to use it:
openlore generate --provider mistral-vibe

# Or set it in .openlore/config.json:
# { "generation": { "provider": "mistral-vibe" } }

# If vibe is not on PATH, point to it:
export MISTRAL_VIBE_CLI=/path/to/vibe
```

### Pre-computed artifacts

When generating specs through Vibe, run `openlore analyze` first if the analysis is missing or stale. It produces structured JSON artifacts that Devstral can
read directly instead of exploring the codebase from scratch:

| Artifact | What it contains |
|----------|-----------------|
| `.openlore/analysis/CODEBASE.md` | Architecture digest: entry points, critical hubs, god functions, spec domains, most-coupled files |
| `.openlore/analysis/env-inventory.json` | All env vars with `required` (no fallback) and `hasDefault` flags, source files |
| `.openlore/analysis/schema-inventory.json` | ORM tables and fields (Prisma, TypeORM, Drizzle, SQLAlchemy) |
| `.openlore/analysis/route-inventory.json` | HTTP routes with method, path, handler, framework |
| `.openlore/analysis/ui-inventory.json` | UI components with framework, props, source file |
| `.openlore/analysis/middleware-inventory.json` | Middleware entries with type (auth/cors/rate-limit/…) and framework |

### Injecting context into Vibe

Vibe does not auto-read project files, but it supports global system prompts via `~/.vibe/prompts/`.
To make CODEBASE.md available in every Vibe session on this project:

```bash
# After running openlore analyze:
cat .openlore/analysis/CODEBASE.md >> ~/.vibe/prompts/openlore.md
```

Or install the Vibe skill (creates `.vibe/skills/openlore.md` as a `/openlore` slash command):

```bash
openlore analyze --ai-configs
```

### Constraints and recommendations

| Property | Value |
|----------|-------|
| Context window | 128 000 tokens |
| **Max output** | **4 096 tokens** |
| API key required | No |
| Network required | No |

Because the output limit is 4 096 tokens (vs 16 000 for Claude), generation produces
**shorter specs per run**. To work within this constraint:

1. **One domain at a time** — run `openlore generate` with `--domain <name>` so each
   invocation focuses on a single domain and stays within the output budget.
2. **Use the analysis artifacts** — always run `openlore analyze` first; the digest files
   reduce how much the model needs to infer, leaving more of the output budget for spec content.
3. **Keep prompts concise** — the pipeline automatically trims context to fit the 128 K
   window, but fewer input tokens leave more room for output.
4. **Expect multi-pass generation** — large codebases may require multiple `generate` runs
   (one per domain) rather than a single all-domains run.

### Selecting a model / agent

```bash
# Use a specific Mistral agent registered in your vibe installation:
openlore generate --provider mistral-vibe --model mistral-large-latest
```

If `--model` is omitted, `vibe` uses its default agent.

<!-- openlore-decisions-instructions -->
## Architectural decisions

When making a significant design choice, call `record_decision` **before** writing the code.

Significant choices: data structure, library/dependency, API contract, auth strategy,
module boundary, database schema, caching approach, error handling pattern.

```
record_decision({
  title: "Use JWTs for stateless auth",
  rationale: "Avoids session store in infra",
  consequences: "Tokens can't be revoked early",
  affectedFiles: ["src/auth/middleware.ts"],
  supersedes: "a1b2c3d4"  // 8-char ID of prior decision being reversed
})
```

Decisions are consolidated in the background immediately after `record_decision` is called — the pre-commit gate reads the already-consolidated store and adds no LLM latency.

**Performance note**: if you skip `record_decision`, the gate detects unrecorded source changes at commit time and triggers a slow LLM extraction on the *next* commit (~10-30s). Calling `record_decision` proactively keeps every commit instant.

## When git commit is blocked by the decisions gate

If `git commit` fails and the output is JSON with `"gated": true`, do NOT retry silently.
Check the `reason` field and act accordingly:

**`reason: "verified"` — decisions await review:**
Present each decision to the user:
> "The commit is blocked — I found N architectural decision(s) to validate:
> 1. **[id]** Title — rationale
Do you approve? (yes/no)"
For each approval call `approve_decision`, for rejections call `reject_decision`.
Then run `openlore decisions --sync` and retry `git commit`.

**`reason: "approved_not_synced"` — decisions approved but not written to specs:**
Run `openlore decisions --sync` then retry `git commit`. Do not skip this step.

**`reason: "drafts_pending_consolidation"` — drafts were recorded but not yet consolidated:**
Present to the user:
> "N decision draft(s) were recorded but never consolidated. Run consolidation now? (~10-30s)"
If yes: run `openlore decisions --consolidate --gate` and handle the result.
If no: retry with `git commit --no-verify` to skip the gate.

**`reason: "no_decisions_recorded"` — source files staged but nothing recorded:**
Present to the user:
> "Source files are staged but no architectural decisions were recorded. Run fallback extraction to check for undocumented decisions? (~10-30s)"
If yes: run `openlore decisions --consolidate --gate` and handle the result.
If no: retry with `git commit --no-verify` to skip the gate.
<!-- end-openlore-decisions-instructions -->

## Agent skills

### Issue tracker

Issues live in `unununnnn/OpenLore-wk` GitHub Issues, via the `gh` CLI. Read `docs/agents/issue-tracker.md` before working with tickets or Wayfinder maps.

### Triage labels

Use the five default triage labels. Read `docs/agents/triage-labels.md` before classifying issues.

### Domain docs

Single-context, with ADRs in `openspec/decisions/`. Read `docs/agents/domain.md` when resolving domain terms or checking architectural decisions.
