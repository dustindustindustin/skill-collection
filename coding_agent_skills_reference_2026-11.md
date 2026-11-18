# Coding Agent Skills & Plugins — A Lean Cross-Agent Reference

**Research date:** 2026-11-18  
**For:** OpenCode, Claude Code, and Codex  
**Primary goal:** Better, verifiably correct code with minimal unnecessary context; dependable progress through long sessions, including local models.  
**Starting point:** Superpowers already works well in Claude Code. Preserve that investment rather than rebuilding its strongest workflows.

> **Decision in one sentence:** Keep Superpowers as the central workflow, create two small cross-agent skills (**Impact Scout** and **Checkpoint Loop**), evaluate **GitNexus** for code intelligence and **RTK** for output compression, and keep everything else opt-in until it demonstrates measurable benefits.

## Legend and selection policy

Every item below has at least one **recommendation tag**:

- **`DEFINITELY USE`** — Worth keeping in the baseline setup or implementing now. Some entries remain *conditionally invoked*, not permanently loaded.
- **`LARGE MODELS ONLY`** — Reserve for strong frontier coding models or sufficiently capable local models; often involves nuanced orchestration, multiple agents, or large design tradeoffs. This describes the *recommended operating mode*, not a hard technical restriction.
- **`SMALL MODELS ONLY`** — A specialized, constrained workflow designed to make smaller local models more reliable. Larger models can also use it, but usually do not need the extra scaffolding.
- **`REFERENCE ONLY`** — Read, borrow a technique, or write a smaller tailored skill; don't install the complete project by default.
- **`TRIAL / CONDITIONAL`** — Test on the applicable project or feature before adopting. It is not required for every repository.

`DEFINITELY USE` does **not** mean blindly activate every tool on every prompt. It means keep it available, with narrow triggers, or make the practice part of the baseline. Likewise, avoid treating the model-size tags as compatibility guarantees: agent performance varies by model and harness.

## Contents

1. [Quick-choice matrix](#1-quick-choice-matrix)
2. [Lessons from the Reddit thread](#2-lessons-from-the-reddit-thread)
3. [Core workflow and skill authoring](#3-core-workflow-and-skill-authoring)
4. [Code intelligence and search](#4-code-intelligence-and-search)
5. [Context and token efficiency](#5-context-and-token-efficiency)
6. [Agent orchestration and long-running work](#6-agent-orchestration-and-long-running-work)
7. [Requirements, engineering quality, and documentation](#7-requirements-engineering-quality-and-documentation)
8. [Frontend, browser, and reverse engineering](#8-frontend-browser-and-reverse-engineering)
9. [Two custom skills ready to adapt](#9-two-custom-skills-ready-to-adapt)
10. [Cross-agent setup](#10-cross-agent-setup)
11. [Context-efficiency rules](#11-context-efficiency-rules)
12. [Adoption plan and measurable tests](#12-adoption-plan-and-measurable-tests)
13. [Links and further reading](#13-links-and-further-reading)

---

## 1. Quick-choice matrix

| Category | Resource | Recommendation | Where it belongs |
|---|---|---|---|
| Core coding workflow | [Superpowers](https://github.com/obra/superpowers) | **`DEFINITELY USE`** | Install separately in Claude Code, Codex, and OpenCode as needed |
| Code impact analysis | **Impact Scout** (custom; template below) | **`DEFINITELY USE`** | Project skill, invoked for meaningful changes |
| Long-session reliability | **Checkpoint Loop** (custom; template below) | **`DEFINITELY USE`** | Project skill, long sessions only |
| Structured repo graph | [GitNexus](https://github.com/abhigyanpatwari/GitNexus) | **`TRIAL / CONDITIONAL`** | Large/unfamiliar repositories; compare with native search |
| CLI output compression | [RTK](https://github.com/rtk-ai/rtk) | **`TRIAL / CONDITIONAL`** | Build-heavy and test-heavy sessions |
| Current API docs | [Context7](https://github.com/upstash/context7) | **`TRIAL / CONDITIONAL`** | Fast-moving frameworks and unfamiliar libraries |
| Local semantic search | [zvec-grep](https://github.com/zvec-ai/zvec-grep) | **`TRIAL / CONDITIONAL`** | Compare against GitNexus; do not install both by default |
| Alternative repo graph | [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) | **`REFERENCE ONLY`** | Alternative to GitNexus |
| Cross-agent config sync | [rulesync](https://github.com/dyoshikawa/rulesync) | **`TRIAL / CONDITIONAL`** | Useful once rules/skills become difficult to maintain |
| Requirements discovery | [Matt Pocock skills](https://github.com/mattpocock/skills) | **`REFERENCE ONLY`** | Borrow `grill-with-docs` and `writing-for-agents` concepts |
| Engineering disciplines | [Addy Osmani agent-skills](https://github.com/addyosmani/agent-skills) | **`REFERENCE ONLY`** | Borrow targeted review and quality concepts |
| Browser QA | [Agent Browser](https://github.com/vercel-labs/agent-browser) | **`TRIAL / CONDITIONAL`** | UI/web repositories only |
| UI design system | [UI/UX Pro Max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | **`TRIAL / CONDITIONAL`** | Design tasks, not backend work |
| UI refinement | [Taste Skill](https://github.com/jeettrench/taste-skill) | **`REFERENCE ONLY`** | Extract one design critique workflow first |
| Full engineering workflow | [gstack](https://github.com/garrytan/gstack) | **`REFERENCE ONLY`** | Borrow a review or QA workflow, not the full stack |
| Multi-agent orchestration | [OMO-Slim](https://github.com/alvinunreal/oh-my-opencode-slim) | **`LARGE MODELS ONLY`** | Optional OpenCode experiment with a strong coordinator |
| Cross-session memory | [Magic Context](https://github.com/skyiron/opencode-magic-context) | **`TRIAL / CONDITIONAL`** | OpenCode-only memory experiment |
| Automatic pruning | [Dynamic Context Pruning](https://github.com/Tarquinen/opencode-dynamic-context-pruning) | **`REFERENCE ONLY`** | Read pruning concepts; investigate its successor before install |
| Reverse engineering | [REA](https://github.com/morluto/rea) | **`TRIAL / CONDITIONAL`** | Recomp/reverse engineering experiments; validate architecture support |
| Local-model guardrails | **Bounded Task Contract** (suggested future skill) | **`SMALL MODELS ONLY`** | Local agents handling limited, testable work |

**Shortlist to implement first:** Superpowers + Impact Scout + Checkpoint Loop. Add a code indexer *only if it beats ordinary search on your project*. Add RTK *only if its output filters preserve diagnostic information you need*.

---

## 2. Lessons from the Reddit thread

**Source:** [r/opencode — What skills, tool sets, plugins do you find useful?](https://www.reddit.com/r/opencode/comments/1x78wqz/what_skills_tool_sets_plugins_do_you_find_useful/)

Participants describe reducing very large skill collections to Superpowers plus a few self-authored tools, finding some multi-agent frameworks expensive in token use, choosing skills on a per-project basis, and trying code intelligence / context optimization tools. One commenter replaces separate planning artifacts with their own issue tracker; others emphasize local search, memory, and sandboxing.

**Practical conclusions:**

1. **A smaller curated set is preferable to a broad install-everything pack.** The concern is not simply that *every* skill body is always loaded: most modern skill systems expose concise metadata and load the body on demand. The bigger risks are selection ambiguity, prompt collisions, and distracting tools.
2. **Superpowers is already a solid baseline.** Extend your habits around it instead of forking it immediately.
3. **A skill is worthwhile when it changes agent behavior at a known failure point.** A generic “write good code” skill rarely adds much; a reproducible handoff or impact map can.
4. **Use project-specific requirements and terminology.** Agents can spend fewer tokens re-deriving domain concepts if a concise glossary and repo-specific rules already exist.
5. **Instrument results.** Measure verified outcomes, not just pleasant-looking plans or self-reported token savings.

---

## 3. Core workflow and skill authoring

### 3.1 Superpowers — **`DEFINITELY USE`**

- **Link:** https://github.com/obra/superpowers
- **Why:** Already proven useful in your workflow; covers discovery/brainstorming, design, planning, debugging, TDD, and verification.
- **Use with:** All three agents. Particularly useful for new features, complicated fixes, ambiguous requirements, and changes needing review.
- **Don't duplicate:** Avoid custom skills that re-explain all of Superpowers' debugging, planning, and TDD steps.
- **Tailor without forking:** Put your repo's exact build commands, passing-test requirements, compatibility constraints, and review thresholds into short repo instructions, then add Impact Scout and Checkpoint Loop. Fork only after you can identify repeated friction that upstream configuration cannot solve.

**Official installation paths (check current README if menus move):**

- **Claude Code:** run `/plugin install superpowers@claude-plugins-official`. Alternative marketplace path: `/plugin marketplace add obra/superpowers-marketplace`, then `/plugin install superpowers@superpowers-marketplace`. You already use it here; avoid duplicate installs.
- **Codex app:** open **Plugins** in the sidebar, find **Superpowers** in Coding, and install. **Codex CLI:** run `/plugins`, find **Superpowers**, and install.
- **OpenCode:** follow the official maintained installer instructions from the project: https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md ; details: https://github.com/obra/superpowers/blob/main/docs/README.opencode.md . Install separately from Claude/Codex.
- **Verification:** In a fresh session, request a small new feature and check whether the appropriate planning workflow activates. Avoid assuming an already-running session sees new plugins without restart.

**Context recommendation:** Keep it installed and discoverable, but require proportional ceremony. Don't run the longest design ritual for trivial typos or obvious one-line changes. Strong models are best for difficult ambiguous design decisions; smaller models can follow a preapproved plan in constrained steps.

### 3.2 Codex `$skill-creator` and the Agent Skills format — **`DEFINITELY USE`**

- **Documentation:** https://learn.chatgpt.com/docs/build-skills
- **Why:** A reusable, agent-friendly format; helpful for drafting, checking trigger descriptions, and iterating on your skills.
- **Setup:** In Codex invoke `$skill-creator`; or author `SKILL.md` manually. Claude Code also supports filesystem-based skills and skill authoring. Keep instructions short, with resource files for rare details.
- **Important:** The older `openai/skills` examples repository is deprecated for distribution; prefer current Codex documentation for new setup and plugins.

### 3.3 rulesync — **`TRIAL / CONDITIONAL`**

- **Link:** https://github.com/dyoshikawa/rulesync
- **Why:** Can manage agent rules and related configuration across different coding tools from one source.
- **When:** You start maintaining several platform-specific instructions, MCP entries, and hooks across multiple repositories.
- **Setup:** Follow its README and generate integrations into a *test repo first*; review diffs before committing generated files. For only two custom skills, directory links/copies are simpler.

---

## 4. Code intelligence and search

### 4.1 GitNexus — **`TRIAL / CONDITIONAL`**, high priority

- **Link:** https://github.com/abhigyanpatwari/GitNexus
- **Why:** Local graph of symbols, call relationships, dependencies, and execution flows. It can answer “who calls this?” and “what could this change break?” without having an agent dump many full files into context.
- **Useful with:** Complicated C/C++ code paths, Python and TypeScript apps, architecture exploration, and Impact Scout.
- **Setup:** Requires an appropriate Node.js runtime (README currently specifies Node 22.18+ on 22.x or 24.11+). From a test repo:

```bash
npx gitnexus analyze --index-only
npx gitnexus setup
npx gitnexus status
```

- **Critical setup caveat:** A standard `gitnexus analyze` also creates or updates some agent integrations and `AGENTS.md`/`CLAUDE.md` context. Use `--index-only` first to avoid unexpected repo modifications, and carefully inspect any changes after enabling integrations.
- **License caveat:** The repository currently states **PolyForm Noncommercial 1.0.0** for OSS, with commercial licensing options. Review the license and your employer's policies *before workplace use*; do not assume it can be used commercially for free.
- **Accuracy caveat:** C/C++ support exists but may not resolve unusual macros, indirect calls, specialized generated code, or every construct. Validate findings against source and real build/test evidence. A graph is useful context, not proof.
- **Verdict:** Test it on one unfamiliar component of your recomp project before committing to it everywhere.

### 4.2 zvec-grep — **`TRIAL / CONDITIONAL`**

- **Link:** https://github.com/zvec-ai/zvec-grep
- **Why:** Local-first semantic and exact search across source, docs, configs, and notes.
- **When:** Natural-language queries are important (“where is deck state validated?”), and text search or GitNexus does not find relevant files efficiently.
- **Setup:** Follow the repo's current installation and index instructions. Benchmark 10 representative searches and compare retrieved file accuracy and latency against `rg`/built-in search.
- **Warning:** Work in progress; don't deploy this and another semantic indexer globally until you know you need both.

### 4.3 codebase-memory-mcp — **`REFERENCE ONLY`**

- **Link:** https://github.com/DeusData/codebase-memory-mcp
- **Why:** Persistent local code/documentation/infrastructure relationships, agent integrations, and change-aware test selection.
- **How to use:** Treat as a *GitNexus alternative*. Compare indexing success, MCP output size, multi-language coverage, and whether the relationship graph is helpful on your actual repo. If GitNexus is good enough, do not duplicate the indexer.

### 4.4 Keep basic search as the control case — **`DEFINITELY USE`**

- **Resource:** Built-in agent file search, `rg`, `git grep`, compiler diagnostics, ctags/LSP where available.
- **Why:** Exact symbol search and compiler/test output are cheap, repeatable, and often more reliable than semantic tools for straightforward tasks.
- **Operating rule:** Ask for the *smallest useful excerpt*, not repository-wide dumps. Use graph/semantic tools when specific relationship questions justify them.

---

## 5. Context and token efficiency

### 5.1 RTK (Rust Token Killer) — **`TRIAL / CONDITIONAL`**, high priority

- **Link:** https://github.com/rtk-ai/rtk
- **Official docs:** https://www.rtk-ai.app/
- **Why:** Filters verbose CLI output (Git, tests, builds, searches, etc.) and advertises significant output-token reductions on supported commands. Savings are workload-dependent, not guaranteed.
- **Setup:** Follow the project's current install instructions for Windows/native or WSL. Verify the installed binary with `rtk gain` (the project cautions that another `rtk` package exists), then enable the integration appropriate to your agent.
- **Use on:** Repeated large test suites, verbose builds, status inspection, and long grep output.
- **Do not use blindly on:** First-time compiler failures, nondeterministic test crashes, linker errors, stack traces, reverse-engineering diagnostics, or other cases where *the omitted lines may contain the answer*. Save raw logs to a file and re-read targeted spans.
- **Success criterion:** Lower emitted tokens without increased diagnosis time or missed regressions.

### 5.2 Context7 — **`TRIAL / CONDITIONAL`**

- **Link:** https://github.com/upstash/context7
- **Why:** Current and version-aware library documentation for unfamiliar frameworks and shifting APIs.
- **Quick start (current README):** `npx ctx7 setup` (Node.js 22.13+); choose skill/CLI or MCP integration, select the relevant agent, and authenticate if prompted. Manual MCP endpoint: `https://mcp.context7.com/mcp`.
- **When:** Next.js/React changes, new SDKs, library migrations, uncertain behavior that differs by version.
- **Avoid:** Triggering it for every trivial language feature or code style change. Retrieved docs also cost tokens; search only the required API and version.
- **Privacy:** It is a networked documentation service, so verify privacy/security posture before use on sensitive workplace context.

### 5.3 OpenCode Magic Context — **`TRIAL / CONDITIONAL`**

- **Link:** https://github.com/skyiron/opencode-magic-context
- **Why:** Cross-session memory and conversation history management for OpenCode.
- **When:** Long-running coding sessions lose critical decisions after compaction, and Checkpoint Loop alone is not enough.
- **Setup:** Follow the repository's OpenCode-specific plugin installation instructions. Test in an isolated project and verify what data it persists, retains, and recovers.
- **Risk:** Another memory layer can store stale or inaccurate summaries and complicate provenance. Never let stored summaries override the current repository or test results.

### 5.4 Dynamic Context Pruning (DCP) — **`REFERENCE ONLY`**

- **Link:** https://github.com/Tarquinen/opencode-dynamic-context-pruning
- **Why:** Illustrates methods to remove or compress low-value tool output and stale history.
- **How to use:** Read the approach and repository status; the maintainer has pointed users toward a newer successor. Research that current successor rather than installing an outdated plugin by default.
- **Risk:** Aggressive pruning can erase the exact error evidence an agent needs later.

---

## 6. Agent orchestration and long-running work

### 6.1 OMO-Slim — **`LARGE MODELS ONLY`** (as the coordinator), optional

- **Link:** https://github.com/alvinunreal/oh-my-opencode-slim
- **Why:** Lightweight relative to the larger Oh My OpenCode orchestration approach; offers agent delegation and model routing.
- **Where:** OpenCode experiments involving a strong planner/coordinator and smaller specialist models.
- **Setup:** Follow the current README to install/configure in a throwaway workspace. Start with **one coordinator plus one specialist**; record model routes, tool calls, token usage, failed delegations, and final test results.
- **Important interpretation of tag:** The project may technically run with many models; the `LARGE MODELS ONLY` recommendation means **keep complex coordination on your most reliable model**. Delegate narrow implementation, search, or test work to smaller models when proven safe.
- **Do not add if:** Superpowers plus the normal coding harness already handles the task; multilayer delegation can cost more than it saves.

### 6.2 Checkpoint Loop — **`DEFINITELY USE`**

- **Source:** Custom skill design in [section 9](#9-two-custom-skills-ready-to-adapt).
- **Why:** Explicit tested checkpoints, next steps, blockers, and failure evidence make a long-running session resumable after context compaction or a model/session change.
- **Trigger:** Multiple tasks, long runs, task queues, or a session expected to span compaction. Do *not* invoke for simple one-file edits.
- **Setup:** Create `.agents/skills/checkpoint-loop/SKILL.md`; link/copy for Claude Code as described below.
- **Boundary:** Does not itself grant permissions, schedule runs, sandbox code, or guarantee unattended execution. Use harness controls and isolated worktrees for those concerns.

### 6.3 Bounded Task Contract — **`SMALL MODELS ONLY`** (future custom skill)

- **Why:** Smaller local models can lose track of broad goals, over-edit, or claim completion too early. A narrow contract makes performance easier to assess.
- **Suggested 5-line contract:** Objective; allowed files; prohibited changes; exact validation command; required completion evidence.
- **Use:** Give the small model **one verified chunk at a time**, in a clean worktree. A larger coordinator can prepare tasks and review outputs.
- **Trigger:** Local agent repeatedly fails on open-ended assignments, loops, or edits unrelated files.
- **Implementation:** Write a short `SKILL.md` only if tests show this improves reliability. Otherwise a task template might be sufficient.

---

## 7. Requirements, engineering quality, and documentation

### 7.1 Matt Pocock's skills — **`REFERENCE ONLY`**

- **Link:** https://github.com/mattpocock/skills
- **Concepts to borrow:** `grill-me`, `grill-with-docs`, `grilling`, `writing-for-agents`, shared vocabulary, and concise ADRs.
- **What to build yourself:** A 10–15-question *maximum* architecture interrogation skill that starts only when requirements are truly ambiguous, and a lightweight `GLOSSARY.md` for project jargon.
- **Overlap warning:** Much of this covers similar ground to Superpowers brainstorming. Test a narrowly differentiated variant, not two parallel interview frameworks.

### 7.2 Addy Osmani's agent-skills — **`REFERENCE ONLY`**

- **Link:** https://github.com/addyosmani/agent-skills
- **Concepts to borrow:** Constraint-driven implementation, source-grounded coding, deliberate tests, independent review, and evidence-based claims.
- **What to build yourself:** Small, project-specific review checklists (e.g., for ABI compatibility, persistence migrations, API backward compatibility) that add *new constraints* beyond Superpowers.
- **Avoid:** Importing an entire collection that repeats generic software-engineering advice.

### 7.3 gstack — **`REFERENCE ONLY`**

- **Link:** https://github.com/garrytan/gstack
- **Why:** Opinionated engineering roles and workflows for design, QA, release, and code review.
- **How to use:** Read one targeted workflow when you need a second perspective; adapt the evidence checklist. Do not install a full parallel development lifecycle alongside Superpowers unless a controlled test shows an improvement.

### 7.4 Impact Scout — **`DEFINITELY USE`**

- **Source:** Custom skill design in [section 9](#9-two-custom-skills-ready-to-adapt).
- **Why:** Narrows the files and behavior at risk before invasive edits; discourages speculative full-repo reading.
- **Trigger:** Unfamiliar code, changes across modules, refactors with unknown callers, API/schema changes.
- **Doesn't replace:** Actual tests, compiler diagnostics, or independent review.

---

## 8. Frontend, browser, and reverse engineering

### 8.1 Agent Browser — **`TRIAL / CONDITIONAL`**

- **Link:** https://github.com/vercel-labs/agent-browser
- **Why:** Browser automation for checking whether the UI works, with compact accessibility snapshots, screenshots, and interactions.
- **Quick start (current README):**

```bash
npm install -g agent-browser
agent-browser install
```

- **Use with:** Frontend-heavy apps, regressions in workflows, responsive layout, navigation, and forms. Run against a local test environment, not production with real customer data.
- **Avoid:** Running browser automation for backend-only or pure compiler tasks.

### 8.2 UI/UX Pro Max — **`TRIAL / CONDITIONAL`**

- **Link:** https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
- **Why:** Searchable design ideas, styling systems, UX guidance, and framework-specific frontend advice.
- **Setup:** Follow the project's installation instructions; prefer a repo-local install that loads only during a design assignment.
- **Check:** Does its design critique actually outperform your existing instructions on the same mockup and requirements?

### 8.3 Taste Skill — **`REFERENCE ONLY`**

- **Current source:** https://github.com/jeettrench/taste-skill
- **Why:** Frontend refinement ideas for layout, typography, spacing, and motion.
- **How:** Borrow a concise UI critique checklist. Its current README includes `npx skills add https://github.com/jeettrench/taste-skill --skill "design-taste-frontend"` if you later decide to install just that skill.
- **Note:** An earlier link to `tasteskill/tasteskill` did not load at the time of verification; the repository README surfaced under the `jeettrench/taste-skill` install URL.

### 8.4 Reverse Engineer Anything (REA) — **`TRIAL / CONDITIONAL`**

- **Link:** https://github.com/morluto/rea
- **Why:** Agent-oriented analysis workflows and tools around behavioral and binary reverse engineering.
- **Where:** Your PS2 recompilation project, as a *contained experiment* rather than a prerequisite.
- **Setup:** Inspect supported tools/architectures and host requirements, then run on a small known sample to confirm MIPS/PS2 or relevant Ghidra-backed workflow support. Avoid promising one-click PS2 recompilation.
- **Safety and correctness:** Analyze only binaries you are authorized to use; isolate tools, preserve original dumps, compare addresses/symbols against verified ground truth, and don't let AI-generated pseudocode substitute for runtime tests.

---

## 9. Two custom skills ready to adapt

These are deliberately concise, instruction-only starting points. Keep the `SKILL.md` body short. Put rare procedures, long examples, and repo-specific command lists in files that the skill can read on demand. Start with **Impact Scout** and **Checkpoint Loop**; test with and without before modifying them.

### 9.1 `impact-scout/SKILL.md` — **`DEFINITELY USE`**

```markdown
---
name: impact-scout
description: Map dependencies and regression risks before changes spanning modules or unfamiliar code; skip isolated trivial edits.
---

# Impact Scout

1. Identify the smallest relevant set of entry points, callers, dependencies, and tests.
2. Use code-intelligence tools if useful; fall back to source search, LSP, and compiler evidence.
3. List behavior and interfaces that must remain unchanged.
4. Separate proven relationships from hypotheses; note index coverage gaps.
5. Return a compact impact map:
   - Key files and call paths
   - Intended change boundary
   - Regression risks and behavioral invariants
   - Exact validation commands
   - Remaining unknowns
6. Stop investigating when there is enough evidence to make the change safely.

Continue with the requested implementation workflow; do not duplicate full planning or debugging procedures.
```

**Project-specific overlay examples:**

- **PS2 recomp / C/C++:** ABI, memory layout, original behavior, callsites, asset formats, deterministic comparison tests.
- **Python backend:** schemas, migrations, API contracts, business rules, test fixtures.
- **Web app:** public APIs, component usage, browser behavior, accessibility, visual regressions.

**What makes it different from Superpowers:** Focuses specifically on a *bounded change-impact map*, not full requirements planning. Skip it when the affected scope is already obvious.

### 9.2 `checkpoint-loop/SKILL.md` — **`DEFINITELY USE`**

```markdown
---
name: checkpoint-loop
description: Execute and resume multi-step coding work through verified checkpoints; use for long sessions, task queues, and handoffs, not small edits.
---

# Checkpoint Loop

1. Inspect the task plan, Git status, and `.agent-state/active.md` if present.
2. Pick one unblocked task with a clear test or other verifiable outcome.
3. Make a bounded change; run the relevant checks.
4. Update `.agent-state/active.md` with:
   - Completed task + evidence
   - Current task + next action
   - Changed files and relevant commit/branch
   - Commands run + actual results
   - Blockers and failed approaches
5. If an approach fails twice without new evidence, change the investigation strategy or mark it blocked.
6. Continue to another independent task only when safe and authorized.
7. On handoff, leave a concise, reproducible checkpoint and honest unresolved items.

Never invent progress, discard test failures, bypass harness permissions, or overwrite unrelated changes.
```

**Example checkpoint file** (`.agent-state/active.md`):

```markdown
# Active checkpoint
- Goal: Preserve behavior while refactoring the parser
- Branch/worktree: feature/parser-refactor
- Verified complete: Parser header split; `ctest -R parser` passed (18/18)
- Current: Update error mapping in `src/parser/errors.cpp`
- Changed: `src/parser/errors.cpp`, `tests/parser_errors.cpp`
- Last command: `ctest --test-dir build -R parser --output-on-failure`
- Issue: Invalid-input test fails with status 4 instead of 3
- Tried: Manual enum swap (failed; reverted)
- Next: Trace conversion from parser status to external error code
- Do not alter: Existing public status values or binary fixture files
```

*The checkpoint example above is illustrative, not a report of actual code or tests.*

**Scope controls:** Use a Git worktree for autonomous/long-running work, predefine network and shell permissions in the actual coding harness, and never treat a skill as a sandbox or permission grant. Prefer explicit stop criteria over “keep going forever.” Add `.agent-state/` to `.gitignore` if checkpoints are local, or selectively commit/share it when cross-machine handoff is an intentional part of the workflow.

---

## 10. Cross-agent setup

### Recommended repo layout

```text
project-root/
├── AGENTS.md                  # Short canonical repo facts and verification commands
├── .agents/
│   └── skills/
│       ├── impact-scout/
│       │   └── SKILL.md
│       └── checkpoint-loop/
│           └── SKILL.md
├── .claude/
│   └── skills/
│       ├── impact-scout/      # symlink/copy of canonical skill
│       └── checkpoint-loop/   # symlink/copy of canonical skill
└── .agent-state/
    └── active.md              # optional per-session state
```

| Agent | Locations for these skills | Setup/activation notes |
|---|---|---|
| **Codex** | `.agents/skills/<name>/SKILL.md`; personal `~/.agents/skills/` | Native format. Use `$skill-creator` for iteration and invoke by name if auto-selection misses. |
| **OpenCode** | `.agents/skills/`, `.opencode/skills/`, or compatible `.claude/skills/` | Native skill discovery; prefer `.agents/skills/` to share with Codex. |
| **Claude Code** | `.claude/skills/<name>/SKILL.md`; personal `~/.claude/skills/` | Add symlinks or copies pointing to the canonical folders; verify discovery after installation. |

**Links:** [Codex skill guide](https://learn.chatgpt.com/docs/build-skills), [OpenCode skill docs](https://open-code.ai/en/docs/skills), [Claude Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview), [Claude Code skills](https://code.claude.com/docs/en/skills).

**Windows:** Git Bash/WSL symlinks and Windows symbolic links have different setup and privilege requirements. If linking creates friction, copying two directories is acceptable. Use one canonical source in version control and a simple sync step. Don't simultaneously leave duplicated skills in several discovery paths if that makes the same name appear twice.

**Keep `AGENTS.md` tiny:** What this repo is, commands to build/test, unusual invariants, and links to detailed docs. If Claude needs information from the canonical file, place a *brief pointer* in `CLAUDE.md`, rather than copying a long policy. Check the harness's own instruction precedence before assuming one global file affects all three.

**Minimal rules example:**

```markdown
# Repository instructions
- Goal: Preserve observable behavior while improving code quality.
- Before a nontrivial refactor, use impact-scout if dependencies are uncertain.
- For multi-session work, use checkpoint-loop and keep `.agent-state/active.md` accurate.
- Treat build/test output as evidence; never report passing tests unless run.
- Run relevant targeted checks first; run the full regression suite before declaring major changes complete.
- Preserve unrelated work and ask/stop before destructive changes not explicitly authorized.
```

---

## 11. Context-efficiency rules

1. **Default to progressive disclosure.** A skill's `name` and `description` should reveal exactly when it triggers; long instructions should remain in the body, not the always-visible rules.
2. **One skill, one distinct failure mode.** If it duplicates Superpowers, rewrite it or remove it.
3. **Keep execution evidence outside chat when possible.** Save complete build logs to files; bring only relevant errors into context. RTK output is a summary, not the original evidence.
4. **Prefer bounded investigation.** Determine the question, choose a search/tool, inspect a small set of files, then stop once the next safe action is clear.
5. **Keep task state short and factual.** A handoff checkpoint should contain results and exact next actions, not the entire conversation.
6. **Avoid multiple overlapping indexers or memory plugins.** One graph tool and one explicit checkpoint file are a reasonable starting point.
7. **Only retrieve docs on demand.** Context7 is valuable for unfamiliar or version-sensitive APIs, but not routine code edits.
8. **Give small models structured chunks.** If a smaller model needs a 3,000-token philosophical instruction to stay focused, redesign the assignment instead.
9. **Use subagents for genuinely independent work.** A second agent should produce a measurable artifact (review findings, independent research, tests), not merely repeat the lead agent's context.
10. **Maintain an escape hatch.** If a plugin prunes, filters, or compresses useful evidence, rerun the command unfiltered and inspect raw output.

### Possible compact evaluation targets (hypotheses, not universal thresholds)

- Skill description: ideally **one to three sentences**, including positive and negative triggers.
- Typical skill body: target **roughly 150–450 words** if the task can be expressed that simply; use references only when needed.
- Checkpoint: aim for **one screenful** of reproducible facts, not a running diary.
- A tool remains installed only if it measurably improves **successful, verified outcomes** or reduces costs *without hurting quality*.

---

## 12. Adoption plan and measurable tests

### Phase 0 — baseline (**`DEFINITELY USE`**)

- Keep your current Claude Code + Superpowers workflow stable.
- Pick five representative tasks: an easy bug fix, cross-module refactor, unfamiliar subsystem, ambiguous feature, and long task queue.
- Record initial results: correctness/test pass rate, wall-clock time, model usage, rework, raw output volume, and unattended progress/looping incidents.

### Phase 1 — custom skills (**`DEFINITELY USE`**)

- Add **Impact Scout** and **Checkpoint Loop** to one repo.
- Invoke them manually initially so you can inspect their impact, then test automatic triggers.
- Compare to baseline: Do they reduce missed callsites, rework, or resume failures? Do they create excessive procedural steps?

### Phase 2 — repo intelligence (**`TRIAL / CONDITIONAL`**)

- On a *personal* repo, use GitNexus in index-only mode first. Check language coverage and licensing before expanding scope.
- Compare Impact Scout using `rg`/LSP alone versus using GitNexus on the same type of problem.
- Pick GitNexus **or** codebase-memory-mcp **or** zvec-grep based on actual retrieval needs, rather than layering all three.

### Phase 3 — command-output optimization (**`TRIAL / CONDITIONAL`**)

- Enable RTK for selected noisy commands.
- Preserve full logs on disk and record whether any important compiler/test diagnostics disappear from the agent's view.
- Compare successful tasks per 100k tokens (if counts are available) and total rework, not just compressed output percentages.

### Phase 4 — specialized workflows

- **Web UI:** Agent Browser; UI/UX Pro Max when designing new screens.
- **Library-heavy work:** Context7 only on version-sensitive questions.
- **Recomp work:** Try REA on a bounded sample and measure reproducibility.
- **Small local models:** Trial a Bounded Task Contract and a strong coordinator with narrowly scoped workers.
- **Complex orchestration:** Only then consider OMO-Slim or persistent context plugins.

### Comparison worksheet

| Metric | Without addition | With addition | Pass/fail question |
|---|---:|---:|---|
| Correct implementation / tests | ___ | ___ | Did correctness improve or remain equal? |
| Relevant files identified | ___ | ___ | Were fewer important dependencies missed? |
| Tokens used per verified task | ___ | ___ | Was context use actually reduced? |
| Total time incl. rework | ___ | ___ | Was net time improved? |
| Wrong/unrelated edits | ___ | ___ | Did scope control improve? |
| Fresh-agent resume success | ___ | ___ | Could another session continue without reconstruction? |
| Looping or failed retries | ___ | ___ | Did repeated unproductive attempts decrease? |
| Tool/maintenance friction | ___ | ___ | Is the capability worth maintaining? |

**Decision rule:** Promote a tool to your default only after multiple real tasks demonstrate that it is useful enough to justify its setup, prompt exposure, complexity, and security footprint.

---

## 13. Links and further reading

### Original discussion and specifications

- [Original Reddit discussion](https://www.reddit.com/r/opencode/comments/1x78wqz/what_skills_tool_sets_plugins_do_you_find_useful/)
- [Superpowers upstream README](https://github.com/obra/superpowers)
- [Codex build-skills guidance](https://learn.chatgpt.com/docs/build-skills)
- [OpenCode skill documentation](https://open-code.ai/en/docs/skills)
- [Claude Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Claude Code skills](https://code.claude.com/docs/en/skills)

### Projects mentioned in this guide

- [GitNexus](https://github.com/abhigyanpatwari/GitNexus)
- [RTK](https://github.com/rtk-ai/rtk)
- [Matt Pocock skills](https://github.com/mattpocock/skills)
- [zvec-grep](https://github.com/zvec-ai/zvec-grep)
- [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)
- [Context7](https://github.com/upstash/context7)
- [Agent Browser](https://github.com/vercel-labs/agent-browser)
- [OMO-Slim](https://github.com/alvinunreal/oh-my-opencode-slim)
- [OpenCode Magic Context](https://github.com/skyiron/opencode-magic-context)
- [Dynamic Context Pruning](https://github.com/Tarquinen/opencode-dynamic-context-pruning)
- [Addy Osmani agent-skills](https://github.com/addyosmani/agent-skills)
- [UI/UX Pro Max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)
- [Taste Skill](https://github.com/jeettrench/taste-skill)
- [gstack](https://github.com/garrytan/gstack)
- [rulesync](https://github.com/dyoshikawa/rulesync)
- [REA](https://github.com/morluto/rea)

### Notes on currency and caution

- This is a **working research reference**, not a declaration that every tool was installed or benchmarked on your machine. Installation commands, compatibility, pricing, and licensing can change; always confirm the project's latest README/release docs.
- Tags are **recommendations for your use case**, not claims that a tool technically works only with large or small models.
- Third-party tools, plugins, MCP servers, and skills can read or execute within repositories and may have broader permission needs than a plain instruction file. Review their source, release provenance, network destinations, and license. Begin experiments in an isolated worktree and do not allow unreviewed third-party code to operate on workplace secrets.
- In particular, review GitNexus's **noncommercial OSS license** before any commercial deployment, and carefully inspect changes created by indexing/setup commands.

---

*Prepared as a reusable, editable single-file reference. Last reviewed: November 18, 2026.*
