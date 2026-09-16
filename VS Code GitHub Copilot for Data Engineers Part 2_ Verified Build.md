# VS Code GitHub Copilot for Data Engineers — Part 2: Build-Ready Site Content, Verified & Corrected

## TL;DR
- Part 1's architecture largely holds, but six specifics were wrong or stale and are corrected here: prompt/agent `tools:` use **alias names** (`execute, read, edit, search, agent, web, todo`) and `#namespace` reference names — not `run_in_terminal`/`readfile`; `model:` values carry a `(copilot)` suffix and prompt files accept a **single** model; instruction files live at workspace-root `.github/instructions/`; the Databricks `npx` MCP example was fictional; chat participants and slash commands still exist; and **Apache Iceberg 1.11.0 (May 19, 2026)** plus **Apache Spark 4.1** are current.
- This report delivers every Part-2 deliverable: a verification/corrections table, full lesson pages for every VS Code Copilot feature, a complete Claude Markdown reference with a cross-tool mapping, instruction-writing guidance for Codex/GPT/Gemini/Claude, an M365 Copilot Chat deep dive, a complete `iceberg-copilot-lab` repo with full file contents, and Astro/Starlight front-end component specs.
- All examples target PySpark, Spark SQL, and Apache Iceberg maintenance/branching for data engineers; Preview/Experimental/Deprecated items are labelled throughout.

## Key Findings
- **Modes/agents:** VS Code exposes built-in agent roles **Ask, Agent, Plan**, chosen from the agents dropdown in the Chat view. Edit's multi-file diffing is folded into Agent/Copilot Edits; "Edit mode" is no longer a first-class separate role. **Autopilot is an agent mode**, not a permission level.
- **Permissions:** Permission levels are **Default Approvals, Assisted permissions, Bypass Approvals**, plus **Autopilot** as an agent mode. Verified settings: `chat.permissions.default`, `chat.tools.terminal.autoApprove`, `chat.tools.autoApprove`, `chat.assistedPermissions.enabled`, `chat.agent.sandbox.enabled`.
- **Tool aliases (GitHub docs):** `execute, read, edit, search, agent, web, todo`. `web` and `todo` are not applicable to the cloud agent; `todo` is supported in VS Code. [github](https://docs.github.com/en/copilot/reference/custom-agents-configuration) MCP wildcard is `<server>/*`; enable all with `["*"]`, disable all with `[]`.
- **Hooks (Preview):** configured in `.github/hooks/*.json` (plus Claude `.claude/settings.json`); events are `SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, PreCompact, SubagentStart, SubagentStop, Stop`; entries take `type:"command"`, `command`, `windows/linux/osx`, `cwd`, `env`, `timeout` (default 30s). VS Code currently ignores `matcher`.
- **Claude files:** `CLAUDE.local.md` is **officially deprecated** in favor of `@import`. VS Code reads `AGENTS.md` (`chat.useAgentsMdFile`), `CLAUDE.md` (`chat.useClaudeMdFile`), and Claude skills (`chat.useClaudeSkills`).
- **Data stack:** Apache Iceberg 1.11.0 (May 19, 2026) is current stable; per Apache Spark's official release notes, "Apache Spark 4.1.0 is the second release in the 4.x series… this release addressed over 1,800 Jira tickets with contributions from more than 230 individuals," [Apache Spark](https://spark.apache.org/releases/spark-release-4.1.0.html) adding Spark Declarative Pipelines and Structured Streaming Real-Time Mode. Use `iceberg-spark-runtime-3.5_2.12` or `iceberg-spark-runtime-4.0_2.13`/`4.1` coordinates matching your Spark line.
- **Billing:** Per GitHub Enterprise Cloud Docs, "Copilot Business at $19 USD per user per month, includes 1,900 AI credits per user… Copilot Enterprise at $39 USD per user per month, includes 3,900 AI credits per user (GitHub Enterprise Cloud only)." [GitHub](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/billing/organizations-and-enterprises) 1 credit = $0.01.

## Details

---
## SECTION 1 — Verification & Corrections Table

| # | Part 1 claim / suspected error | Status | Correct value (source + date) |
|---|---|---|---|
| 1 | Built-in roles Agent/Ask/Plan; Edit folded into Agent | **Confirmed** | Ask, Agent, Plan in the agents dropdown (code.visualstudio.com custom-agents & agent-harnesses, 9/9/2026). Edit's multi-file diffing is part of Agent/Copilot Edits. |
| 2 | Plan saves to `/memories/session/plan.md` | **Unverifiable** | Official docs describe Plan generating a Markdown plan opened in an editor tab; the exact path was not confirmed. |
| 3 | "Default / Bypass / Autopilot (Preview)" | **Corrected** | Levels: **Default Approvals, Assisted permissions, Bypass Approvals**; **Autopilot is an agent mode** (approvals & security docs, 2026). |
| 4 | Settings `chat.autopilot.enabled` / `.advanced.enabled` | **Corrected/Unverifiable** | Verified: `chat.permissions.default`, `chat.assistedPermissions.enabled`, `chat.tools.autoApprove`, `chat.tools.terminal.autoApprove`, `chat.agent.sandbox.enabled`. `chat.autopilot.*` not found. |
| 5 | Subagents via `runSubagent`; model ≤ main cost tier | **Confirmed** | `runSubagent` confirmed (docs/agents/run/subagents, 9/9/2026): "The requested model cannot exceed the cost tier of the main model. If you request a more expensive model, the subagent doesn't run and reports which models are available." [Visual Studio Code](https://code.visualstudio.com/docs/agents/run/subagents) Nested off by default (`chat.subagents.allowInvocationsFromSubagents`). |
| 6 | `chat.useAgentsMdFile`, `chat.useNestedAgentsMdFiles` (exp), `chat.useClaudeMdFile` | **Confirmed** | All three confirmed; also `chat.useClaudeSkills` reads `~/.claude/skills/*/` and `${workspaceRoot}/.claude/skills/*/`. [GitHub](https://github.com/microsoft/vscode-docs/issues/9151) Nested AGENTS.md is experimental/off by default. |
| 7 | Instruction locations incl. `.github/instructions/` | **Confirmed** | `.github/copilot-instructions.md` + `.github/instructions/*.instructions.md` with `applyTo` glob (custom-instructions, 2026). |
| 8 | `jobs/.github/instructions/pyspark.instructions.md` | **Corrected** | Instruction files are discovered at **workspace-root** `.github/instructions/`. Use root `.github/instructions/` with `applyTo:"jobs/**/*.py"`. |
| 9 | SKILL.md frontmatter | **Confirmed w/ correction** | VS Code accepts `name, description, argument-hint, user-invocable, disable-model-invocation, context: fork` (experimental). The portable Agent Skills spec (claude.ai/API) accepts only six keys (`allowed-tools, compatibility, description, license, metadata, name`) and errors on `argument-hint`. [Claude Code Docs](https://code.claude.com/docs/en/skills) |
| 10 | `*.agent.md` frontmatter + handoffs | **Confirmed** | `description, tools, model, handoffs(label,agent,prompt,send,model), name, user-invocable, disable-model-invocation`. `argument-hint`/`handoffs` ignored by GitHub.com cloud agent. [github](https://docs.github.com/en/copilot/reference/custom-agents-configuration) |
| 11 | `*.prompt.md` frontmatter | **Confirmed** | `description, name, argument-hint, agent(ask/agent/plan/custom), model, tools`. If `tools` set, default agent = agent. |
| 12 | Tool ids `run_in_terminal`, `search/codebase`, `readfile` | **Corrected** | Aliases: `execute, read, edit, search, agent, web, todo` (docs.github.com custom-agents-configuration). `#`-refs use namespaced form (`#web/fetch`, `#tool:web/fetch`, `search`, `edit`, `todos`, `execute`). `readfile`/`run_in_terminal` invalid. |
| 13 | `model:` format; list supported? | **Corrected** | Values carry a `(copilot)` suffix, e.g. `GPT-5.2 (copilot)` (custom-agents handoff example). Custom agents may accept a list; **prompt files use a single model**. |
| 14 | `.vscode/mcp.json` `npx databricks-mcp-server@latest` | **Corrected** | Fictional package. Real: Databricks managed MCP servers, official Snowflake/Snowflake Cortex MCP, dbt MCP, BigQuery, DuckDB/MotherDuck, PostgreSQL, GitHub MCP. Schema: top-level `servers`, `type: stdio|http`, `inputs` for secrets. |
| 15 | `@workspace/@terminal/@vscode` & slash commands still exist? | **Confirmed** | `@github, @terminal, @vscode, @workspace` still documented. `/explain /fix /tests /new /doc` exist; scaffolding `/create-instruction /create-prompt /create-agent /create-skill /create-hook` confirmed. |
| 16 | Content exclusion not in agent/cloud/CLI | **Confirmed** | Verbatim (docs.github.com): "GitHub Copilot CLI, Copilot coding agent, and Agent mode in Copilot Chat in IDEs, do not support content exclusion." [github](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) (Edit mode also excluded.) |
| 17 | CLAUDE.local.md deprecated vs supported | **Resolved: Deprecated** | Anthropic memory docs: "Previously CLAUDE.local.md served a similar purpose, but is now deprecated in favor of imports since they work better across multiple git worktrees." [anthropic](https://docs.anthropic.com/en/docs/claude-code/memory) Still loads; replace with `@~/.claude/…`. |
| 18 | Iceberg 1.10.0 vs 1.11.0 | **Resolved: 1.11.0** | 1.11.0 released May 19, 2026 [Apache Iceberg](https://iceberg.apache.org/releases/) (iceberg.apache.org/releases; Google Open Source blog Jun 5, 2026); 1.10.2 was May 18, 2026. [Apache Iceberg](https://iceberg.apache.org/releases/) |
| 19 | Enterprise BYOK GA vs preview | **Partial/Unverifiable** | BYOK documented (docs.github.com concepts/models/bring-your-own-key; VS Code language-models). Exact GA-vs-preview per plan not definitively confirmed — check current plan docs. |
| 20 | Iceberg Spark procedures & branch/tag DDL | **Confirmed** | `rewrite_data_files, expire_snapshots (older_than/retain_last), remove_orphan_files (dry_run), rewrite_manifests` (spark-procedures). `CREATE BRANCH … RETAIN n DAYS WITH SNAPSHOT RETENTION`, `CREATE TAG … AS OF VERSION … RETAIN … DAYS`, `VERSION AS OF`/`TIMESTAMP AS OF` (spark-ddl). |
| 21 | Latest Spark | **Resolved** | Spark 4.1.0 GA (second 4.x release); 4.1.3 (Jul 15, 2026); 4.2.0 (Jul 14, 2026); [Apache Spark](https://spark.apache.org/releases/spark-release-4.1.1.html) 3.5.x LTS security fixes to Nov 2027. [End of Life Date](https://endoflife.date/apache-spark) Iceberg 1.11 modernizes the Spark 4.1 connector (MERGE INTO … WITH SCHEMA EVOLUTION). [Google Open Source](https://opensource.googleblog.com/2026/05/announcing-apache-iceberg-1110.html) |

---
## SECTION 2 — Build-Ready Lesson Pages (VS Code GitHub Copilot)

Each page follows the required 8-part template. Shortcuts shown **Win/Linux → macOS**.

### Ask mode
1. **What** — Conversational Q&A in the Chat view; returns explanations/snippets without editing files.
2. **Why** — Fast, low-risk, quota-efficient; understand code before changing it.
3. **When/where; not** — Use for "explain this window function," "why is my Spark job spilling." Not for multi-file edits or terminal execution (escalate to Agent).
4. **How** — Chat view `Ctrl+Alt+I` → `⌃⌘I`; pick **Ask**. Inline chat `Ctrl+I` → `⌘I`. Command Palette (`Ctrl+Shift+P`/`⇧⌘P`) → "Chat: Focus on Chat View". No settings required.
5. **Limits** — Won't modify files; inline chat is always Ask-style (no agent inline).
6. **DE walkthrough** — Prompt: *"Explain what `MERGE INTO` does in this Spark SQL file and whether it rewrites whole partitions in Iceberg."* Returns prose + copy-on-write vs merge-on-read notes; may cite:
```sql
MERGE INTO prod.db.customers t
USING staging.updates s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```
7. **Quiz** — (a) Does Ask edit files? *No.* (b) Inline chat on macOS? *⌘I.* (c) Invoke a custom agent inline? *No.* (d) Cheaper than Agent for one question? *Yes.*
8. **Storyboard** — Panel slides in (200ms ease-out) → dropdown highlights "Ask" → streaming token answer → hover reveals Copy.

### Plan mode
1. **What** — Built-in role producing a structured Markdown implementation plan; no code edits.
2. **Why** — De-risks large refactors; review sequencing (schema evolution → backfill → compaction) first.
3. **When/where; not** — Before multi-step migrations. Not for a one-line fix.
4. **How** — Dropdown → **Plan**; opens a plan in an editor tab. Built-in "Plan" agent has a handoff button to implementation.
5. **Limits** — Doesn't execute; save path unverified.
6. **DE walkthrough** — Prompt: *"Plan a migration of our Hive `events` table to Iceberg partitioned by days(event_ts), incl. backfill and validation."* Output: create Iceberg table → `CREATE TABLE … USING iceberg PARTITIONED BY (days(event_ts))` → audit branch → backfill INSERT → validate via `snapshots`/`files` metadata tables → cutover.
7. **Quiz** — (a) Does Plan edit code? *No.* (b) How to continue? *Handoff button.* (c) Where does it appear? *Editor tab.*
8. **Storyboard** — Dropdown→Plan → "Generating plan…" spinner → plan.md renders with animated checklist → pulsing "Start Implementation".

### Agent mode (permissions, sandboxing, checkpoints, todos, subagents, summarization, max requests)
1. **What** — Autonomous mode: chooses files, multi-file edits, runs terminal/tests, iterates to completion.
2. **Why** — Tasks needing 3+ files: building a PySpark job, wiring tests, fixing a pipeline.
3. **When/where; not** — Complex multi-step, tool-using work (incl. MCP). Not for trivial single edits.
4. **How** — Dropdown → **Agent** (default `chat.agent.enabled: true`). One prompt = one premium request; follow-up edits/commands are free.
   - **Permissions** — per-session picker: Default Approvals (respects settings); Assisted permissions (LLM judge; `chat.assistedPermissions.enabled`); [Visual Studio Code](https://code.visualstudio.com/docs/agents/run/approvals) Bypass Approvals (auto-approve tools, still stops for blocking questions); [Visual Studio Code](https://code.visualstudio.com/learn/foundations/approvals-autonomy-and-context-budget) **Autopilot** agent mode (auto-approves, auto-retries, resolves blocking questions). [Visual Studio Code](https://code.visualstudio.com/learn/foundations/approvals-autonomy-and-context-budget) Default via `chat.permissions.default`. [Visual Studio Code](https://code.visualstudio.com/docs/agents/run/approvals)
   - **Terminal auto-approve** — regex allow/deny in `chat.tools.terminal.autoApprove`:
```json
{
  "chat.tools.terminal.autoApprove": {
    "/^git\\s+(status|diff|log|show)\\b/": true,
    "/^pytest\\b/": true,
    "/^spark-submit\\b/": true,
    "/^rm\\s+-rf\\b/": false,
    "/^aws\\s+s3\\s+rb\\b/": false
  }
}
```
   All subcommands in a compound command must match an approved rule. [Visual Studio Code](https://code.visualstudio.com/docs/agents/security)
   - **Sandboxing** — `chat.agent.sandbox.enabled` (OS-level; macOS/Linux only, not Windows). [Rune Hub](https://rune.codes/hub/vscode/how-to-add-and-configure-mcp-servers-in-vs-code-with-mcp-json) Independent of level; even Bypass/Autopilot stay restricted. [Visual Studio Code](https://code.visualstudio.com/docs/agents/run/approvals) Enterprise policy `ChatAgentSandboxEnabled`.
   - **Checkpoints/restore** — review edits in a diff editor; keep or undo pending edits. [Visual Studio Code](https://code.visualstudio.com/docs/agents/run/security)
   - **Todo list** — `todo`/`todos` tool tracks a structured task list (VS Code supported; cloud agent not).
   - **Subagents** — `#runSubagent`; subagent model ≤ main cost tier; nested off by default.
   - **Context summarization / max requests** — long sessions summarize automatically; watch the context-budget indicator.
5. **Limits** — Content exclusion does NOT apply. Autopilot reduces review of intermediate steps. [Visual Studio Code](https://code.visualstudio.com/docs/agents/run/approvals) Enterprise policies `ChatToolsAutoApprove`, `ChatToolsEligibleForAutoApproval`, `ChatToolsTerminalEnableAutoApprove` [Visual Studio Code](https://code.visualstudio.com/docs/agents/run/security) can lock down.
6. **DE walkthrough** — Prompt: *"Add `jobs/ingest_orders.py` that reads new parquet from `s3://raw/orders/`, dedupes on `order_id` keeping latest `updated_at`, and MERGEs into `prod.sales.orders`. Add a pytest with a local SparkSession. Run the tests."* Agent scaffolds file, writes MERGE, creates test, runs `pytest`, fixes failures, re-runs green. Core:
```python
from pyspark.sql import SparkSession, functions as F, Window

def upsert_orders(spark: SparkSession, src_path: str, table: str = "prod.sales.orders") -> None:
    df = spark.read.parquet(src_path)
    w = Window.partitionBy("order_id").orderBy(F.col("updated_at").desc())
    latest = df.withColumn("_rn", F.row_number().over(w)).filter("_rn = 1").drop("_rn")
    latest.createOrReplaceTempView("orders_updates")
    spark.sql(f"""
        MERGE INTO {table} t
        USING orders_updates s ON t.order_id = s.order_id
        WHEN MATCHED AND s.updated_at > t.updated_at THEN UPDATE SET *
        WHEN NOT MATCHED THEN INSERT *
    """)
```
7. **Quiz** — (a) Is Autopilot a permission level? *No, an agent mode.* (b) Which OS lacks sandboxing? *Windows.* (c) Premium requests per agent prompt? *One (follow-ups free).* (d) Subagent pricier than main? *No.* (e) Content exclusion protects files in agent mode? *No.*
8. **Storyboard** — Agent loop Plan→Edit→Run→Observe→(loop)→Done with a credit meter and a draggable permissions dial (Default→Assisted→Bypass→Autopilot).

### Inline completions (ghost text)
What: grey suggested code as you type. Why: fast boilerplate (DataFrame transforms, imports). When/not: everywhere while typing; not architecture. How: automatic; accept `Tab`, dismiss `Esc`, cycle `Alt+]`/`Alt+[`. Unlimited on paid plans; separate from `copilot-instructions.md`. Limits: model chosen server-side; [Copilot](https://www.githubcopilot.dev/blogs/which-ai-model-powers-github-copilot) not steerable by chat instructions. DE: type `df = spark.read.format("iceberg").` → accept `.load("prod.sales.orders")`. Quiz: accept key? *Tab.* Repo instructions steer ghost text? *No.* Storyboard: ghost text fades in; `Tab` animates acceptance.

### Next edit suggestions (NES)
What: predicts your next edit location/change. Why: cascading refactors (rename a column across a job). When/not: repetitive edits; not novel logic. How: automatic; `Tab` to jump/accept; unlimited on paid plans. Limits: heuristic. DE: rename `ts`→`event_ts`; NES proposes the matching `withColumn` change. Quiz: trigger? *A recent edit.* Storyboard: arrow animates to the predicted line.

### Inline chat
What: chat in the editor at cursor. Why: quick localized refactor/explain. When/not: single-file; not multi-file. How: `Ctrl+I` → `⌘I`; always Ask-style. Limits: no agent inline. DE: select a UDF, `⌘I`, "vectorize with pandas_udf." Quiz: shortcut? *`Ctrl+I`/`⌘I`.* Storyboard: inline input expands under selection.

### Smart actions (explain, fix, generate tests/docs, commit messages, PR descriptions, rename)
What: right-click **Copilot** actions + Source Control sparkle. Why: one-click routine tasks. When/not: routine; not deep design. How: right-click selection → Copilot → Explain/Fix/Generate Tests/Generate Docs; commit-message sparkle in Source Control input; rename suggestions on `F2` (code.visualstudio.com/docs/editing/copilot-smart-actions). Limits: quality varies. DE: "Generate Docs" on `upsert_orders` → NumPy-style docstring; commit sparkle → `feat(ingest): idempotent MERGE upsert for orders`. Quiz: commit button location? *Source Control input.* Storyboard: sparkle pulses; docstring types in.

### Copilot code review in VS Code
What: AI review of changes/PRs in-editor. Why: catch issues pre-PR. When/not: pre-commit; not a human-review substitute. How: Source Control → Copilot Code Review, or on GitHub PRs. Reads `copilot-instructions.md` from the PR **base branch** (not feature branch). Limits: base-branch quirk; misses runtime behavior. DE: flags a missing `.cache()` before two actions, or an Iceberg write lacking a partition filter. Quiz: which branch's instructions? *Base.* Storyboard: review comments slide into the diff gutter.

### Chat context & #-mentions
What: add context via `#` (files, folders, selection, `#codebase`, terminal output, problems, images). Why: ground answers. When/not: always ground; avoid over-stuffing (budget). How: type `#` → `#file`, `#folder`, `#codebase`, `#terminalLastCommand`, `#problems`, drag images. Tools referenced as `#web/fetch`, `#search`, `#edit`, `#todos`, `#runSubagent`. Context items (info) vs tools (capabilities). Limits: large context degrades quality. DE: `#file:jobs/ingest_orders.py #problems fix the failing type check`. Quiz: `#codebase` — context or tool? *Context.* Storyboard: chips animate into the prompt box.

### Chat participants & slash commands
What: `@github, @terminal, @vscode, @workspace`; `/explain /fix /tests /new /doc` + scaffolding `/create-instruction /create-prompt /create-agent /create-skill /create-hook`. Why: routing + shortcuts. When/not: `@terminal` for shell help; custom agents are picked from the dropdown, NOT via `@`. How: type `@` or `/`. Limits: available commands depend on surface/agent. DE: `@terminal how do I pass --packages for iceberg-spark-runtime to spark-submit?` Quiz: invoke custom agents with `@`? *No — dropdown.* Storyboard: `@`/`/` menus cascade.

### Jupyter notebooks with Copilot
What: agent/ask support in `.ipynb`. Why: exploratory PySpark. When/not: notebooks; not production jobs. How: open notebook; Chat can add/run cells (docs/agents/guides/notebooks-with-ai). Limits: kernel state matters. DE: "profile nulls per column and plot top 10." Quiz: can agent run cells? *Yes.* Storyboard: cells append and execute with output fade-in.

### Copilot in the terminal
What: command help/explain; agent terminal execution with approvals. Why: shell fluency. When/not: command construction; approve destructive ops manually. How: `@terminal` in chat; agent runs via `execute` tool with `chat.tools.terminal.autoApprove`. Limits: sandbox not on Windows. DE: build a `spark-submit` line with `--conf spark.sql.catalog.prod=org.apache.iceberg.spark.SparkCatalog`. Quiz: auto-approve `pytest`? *Add regex rule = true.* Storyboard: approval dialog with allow/deny toggle.

### Agent Sessions view / Agents Window
What: manage multiple/parallel sessions (local, remote, cloud); discovers Copilot CLI, Claude Code, Codex sessions. Why: run/track background work. When/not: parallel tasks; not tiny edits. How: Agents Window; Manage Sessions / Session History (docs/agents/run/sessions). Limits: harness-dependent tools. DE: one session compacts tables while another writes tests. Quiz: shows Codex sessions? *Yes.* Storyboard: session cards with live status chips.

### Copilot CLI
What: terminal agent (`copilot`) with sandboxing, AGENTS.md, MCP, skills, custom agents. Why: headless/CI, remote hosts. When/not: automation; not GUI review. How: install per docs.github.com CLI quickstart; `copilot -p "…"`. Reads instructions/skills/MCP like the IDE. `todo` alias not available in CLI `-p`/SDK (VS Code only). Limits: content exclusion not supported. DE: `copilot -p "run table maintenance on prod.sales.orders"`. Quiz: does CLI honor `todo`? *No.* Storyboard: terminal typing animation.

### Copilot coding agent (cloud) delegation from VS Code
What: delegate a task to run on GitHub infra; returns a PR. Why: offload long tasks. When/not: well-scoped issues; not interactive debugging. How: assign issue/task to Copilot; custom agents via agent profiles (`argument-hint`/`handoffs` ignored in cloud). Limits: no content exclusion; `web`/`todo` aliases not applicable. DE: "compact all Iceberg tables and open a PR." Quiz: what does it return? *A pull request.* Storyboard: task→cloud→PR badge.

### Third-party agents (Claude, Codex) in VS Code
What: choose Claude or Codex harness alongside Copilot. Why: model/agent choice. When/not: when a provider agent fits; mind data-retention terms. How: agent harness picker (docs/agents/concepts/agent-harnesses). `allowDangerouslySkipPermissions` (Claude) only in sandboxes. [Visual Studio Code](https://code.visualstudio.com/docs/agents/security) Limits: provider-specific tools. DE: run Codex for a large refactor in a worktree. Quiz: skip-permissions safe outside a sandbox? *No.* Storyboard: harness selector morph.

### Custom instructions (`copilot-instructions.md` + `*.instructions.md`)
What: repo-wide + glob-scoped steering. Why: consistent PySpark/SQL conventions. When/not: durable rules; not one-off asks. How: `.github/copilot-instructions.md` (auto; `github.copilot.chat.codeGeneration.useInstructionFiles: true`); `.github/instructions/*.instructions.md` with `applyTo` (`chat.includeApplyingInstructions: true`). Multiple matching files **stack (union)**. `/create-instruction` scaffolds. Limits: no `applyTo` = inert until manually attached; [Medium](https://nivedv.medium.com/before-the-agent-starts-how-github-copilots-customization-layer-actually-works-a7db689c795e) doesn't affect ghost text. DE example:
```markdown
---
applyTo: "jobs/**/*.py"
---
- Target PySpark on Spark 4.1 and Iceberg 1.11.
- Always write to catalog `prod`; never hardcode S3 paths — use table names.
- Use MERGE INTO for upserts; never full-overwrite partitioned Iceberg tables.
- Add type hints; no bare `except`.
```
Quiz: do overlapping `applyTo` files override? *No — union.* Storyboard: file-tree explorer showing which instructions attach to a selected file.

### AGENTS.md
What: single always-on instruction file read by many agents. Why: one source of truth across Copilot/Codex/Gemini. When/not: cross-tool rules. How: root `AGENTS.md`; `chat.useAgentsMdFile: true`; nested via `chat.useNestedAgentsMdFiles` (experimental). Limits: off by default. DE: see Section 4. Quiz: enable setting? *`chat.useAgentsMdFile`.* Storyboard: one file glowing, arrows to Copilot/Codex/Gemini.

### Prompt files (`*.prompt.md`)
What: reusable slash-command prompts. Why: standardize "prepare a PR," "scaffold a job." When/not: repeated tasks; not always-on rules. How: `.github/prompts/*.prompt.md`; `/create-prompt` or Command Palette "Chat: New Prompt File". Frontmatter `description, name, argument-hint, agent, model, tools`. Limits: single `model`. DE example:
```markdown
---
description: Add Iceberg table maintenance for a given table
argument-hint: <catalog.db.table>
agent: agent
model: Claude Sonnet 4.6 (copilot)
tools: ['edit', 'search', 'execute']
---
Create a PySpark maintenance job for ${input:table} that calls rewrite_data_files,
expire_snapshots (retain_last 5), remove_orphan_files (dry_run first), and rewrite_manifests.
```
Quiz: pin a model? *Yes, one.* Storyboard: `/` menu shows prompt with argument hint.

### Custom agents with handoffs
What: personas with fixed tools/model + handoff buttons. Why: planner→implementer→reviewer workflows. When/not: role specialization. How: `.github/agents/*.agent.md` or user profile; "Chat: New Custom Agent"; pick from dropdown (not `@`). Limits: malformed YAML silently skipped. DE example:
```markdown
---
name: Iceberg Planner
description: Plans Iceberg migrations and maintenance, no edits.
tools: ['search', 'web']
model: GPT-5.4 (copilot)
handoffs:
  - label: Implement
    agent: iceberg-implementer
    prompt: Implement the plan above.
    send: false
    model: Claude Sonnet 4.6 (copilot)
---
You are a senior data engineer. Produce a numbered plan only.
```
Quiz: how invoked? *Dropdown.* Storyboard: handoff button chains agent cards.

### Agent Skills
What: progressively-loaded task workflows (`SKILL.md` + bundled scripts). Why: encode repeatable procedures cheaply (only description loaded until matched). When/not: fuzzy, reusable tasks. How: `.github/skills/<name>/SKILL.md`, `.claude/skills/`, `.agents/skills/`, user dirs; `/create-skill`. Discovery: name+description → body loads on match. Invoke via `/`. Limits: portable spec allows only 6 frontmatter keys; [Claude Code Docs](https://code.claude.com/docs/en/skills) VS Code adds `argument-hint`, `user-invocable`, `disable-model-invocation`, `context: fork` (experimental). DE: `iceberg-maintenance` skill with `scripts/compact.py`. Quiz: what loads first? *name + description.* Storyboard: context-cost slider comparing skill (lazy) vs instruction (always-on).

### MCP servers & tool sets
What: external tools via Model Context Protocol. Why: let the agent query warehouses/catalogs. When/not: governed data access; review before trusting. How: `.vscode/mcp.json` (workspace) or user; `MCP: Add Server`, `MCP: List Servers`. [datamcp](https://datamcp.app/blog/vscode-mcp-setup-guide) Top-level `servers`; `type: stdio|http`; `inputs` for secrets; `<server>/*` wildcard in `tools`; macOS/Linux `sandboxEnabled`. [Rune Hub](https://rune.codes/hub/vscode/how-to-add-and-configure-mcp-servers-in-vs-code-with-mcp-json) Limits: MCP runs arbitrary code; `type` required. [ContextBolt](https://contextbolt.com/blog/vscode-mcp-setup/) DE example (real servers):
```json
{
  "inputs": [
    { "type": "promptString", "id": "pg-conn", "description": "Postgres connection string", "password": true }
  ],
  "servers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:pg-conn}" }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": { "PG_CONN": "${input:pg-conn}" }
    },
    "dbt": { "type": "stdio", "command": "uvx", "args": ["dbt-mcp"] }
  }
}
```
For Databricks/Snowflake use their managed/official MCP endpoints (HTTP + OAuth) rather than an npx shim. Per Dataworkers' "10 Best MCP Servers for Data Engineering 2026," the leading data-engineering servers are "Data Workers… Snowflake Cortex MCP, Databricks MCP, BigQuery MCP, dbt MCP, Atlan MCP, DataHub MCP, Monte Carlo MCP, Airbyte MCP, and Linear MCP." [Dataworkers](https://dataworkers.io/resources/best-mcp-servers-data-engineering-2026/) Quiz: top-level key? *`servers`.* Storyboard: server list with connect/disconnect states.

### Hooks (Preview)
What: shell commands triggered on agent lifecycle events. Why: enforce formatting/validation/guardrails deterministically. When/not: policy enforcement; not model steering. How: `.github/hooks/*.json` (also `.claude/settings.json`, `~/.copilot/hooks`, agent frontmatter `hooks:` with `chat.useCustomAgentHooks`). Events: `SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, PreCompact, SubagentStart, SubagentStop, Stop`. Entry fields: `type:"command"`, `command`, `windows/linux/osx`, `cwd`, `env`, `timeout` (default 30s). `matcher` currently ignored. `/create-hook` scaffolds. Limits: Preview; runs on all tool invocations regardless of matcher. DE example (`.github/hooks/format.json`):
```json
{ "hooks": { "PostToolUse": [ { "type": "command", "command": "black jobs/ && sqlfluff fix sql/ --dialect sparksql" } ] } }
```
Quiz: workspace hooks dir? *`.github/hooks/`.* Storyboard: timeline of events firing hooks.

### BYOK (Bring Your Own Key)
What: use your own model provider key. Why: access models/regions or cost control. When/not: when policy allows; check plan GA status. How: model picker → Add Models → provider key (docs/models/bring-your-own-key; VS Code language-models). Limits: some models retain data (e.g., Claude Fable 5 retains prompts/outputs for safety classifiers). DE: add a large-context model to review a big migration. Quiz: which model has special retention? *Claude Fable 5.* Storyboard: masked key entry → model appears in picker.

### Model picker & Auto
What: choose model per session or let **Auto** route by task/availability; thinking-effort selectable in the picker (e.g., "Claude Sonnet 4.6 · High"). [Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/language-models) Why: match cost/quality. When/not: reasoning → premium; boilerplate → mini. How: model picker in chat (deprecated settings `github.copilot.chat.anthropic.thinking.effort`, `…responsesApiReasoningEffort` — use the picker). [Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/language-models) Limits: availability varies by plan. DE: Auto for mixed sessions. Quiz: set thinking effort now? *In the model picker.* Storyboard: picker with effort dial.

### Chat diagnostics / "Configure Instructions" & auto-generating workspace instructions
What: `Chat: Configure Instructions`, "what instruction files are active?", auto-generate `copilot-instructions.md`. Why: debug ignored rules. When/not: when instructions don't apply. How: Command Palette → Chat: Configure Instructions; check "Used references"; auto-generate scans the repo. Verify `chat.includeApplyingInstructions`, `chat.includeReferencedInstructions`, `chat.useAgentsMdFile`. [Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/custom-instructions) Limits: model may still not follow. DE: generate a starter Spark `copilot-instructions.md`, then trim. Quiz: command to list active instructions? *Chat: Configure Instructions.* Storyboard: diagnostic panel listing loaded files with checkmarks.

---
## SECTION 3 — Complete Claude-Related Markdown Reference

**Loading & precedence:** memory files load at session start; higher-in-hierarchy files load first as a foundation. Order: **Enterprise policy → Project `./CLAUDE.md` → User `~/.claude/CLAUDE.md`**, with `CLAUDE.local.md` as a (deprecated) local layer. Auto memory (v2.1.59+, on by default) writes `user/project/local`-typed notes. `@path` imports pull in additional files.

**Locations:**
| File | Location | Purpose |
|---|---|---|
| Enterprise policy CLAUDE.md | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`; Linux `/etc/claude-code/CLAUDE.md`; Windows `C:\ProgramData\ClaudeCode\CLAUDE.md` | Org-wide, cannot be excluded |
| Project CLAUDE.md | `./CLAUDE.md` | Team-shared, in VCS |
| User CLAUDE.md | `~/.claude/CLAUDE.md` | Personal, all projects |
| CLAUDE.local.md | `./CLAUDE.local.md` | **Deprecated** — use `@~/.claude/…` imports |
| Rules | `.claude/rules/*.md` (optional `paths`) | Path-scoped rules |
| Commands | `.claude/commands/*.md` (legacy; prefer skills) | Slash commands |
| Subagents | `.claude/agents/*.md` | Named subagents |
| Skills | `.claude/skills/<name>/SKILL.md` | Portable skills |
| Settings/hooks | `.claude/settings.json`, `.claude/settings.local.json` | Hooks, config |

**CLAUDE.md (project):**
```markdown
# Data Engineering — Iceberg Lakehouse
- Stack: PySpark 4.1, Spark SQL, Apache Iceberg 1.11, REST catalog `prod`.
- Never overwrite partitioned Iceberg tables; use MERGE INTO.
- Run `pytest -q` before proposing a commit.
- Maintenance jobs must call expire_snapshots with retain_last >= 5.
See @docs/architecture.md and import personal prefs: @~/.claude/de-prefs.md
```
**User CLAUDE.md (`~/.claude/CLAUDE.md`):**
```markdown
- Prefer NumPy-style docstrings.
- Explain Spark physical plans when I ask "why slow".
```
**Enterprise policy CLAUDE.md:** `- All data access via catalog prod; no raw credentials in code.`
**Nested (`jobs/CLAUDE.md`):** `- Files here are Spark batch jobs; entrypoint is main(spark).`
**CLAUDE.local.md (deprecated) replacement:** put personal notes in `~/.claude/de-prefs.md` and import via `@~/.claude/de-prefs.md`.

**.claude/rules/sql.md:**
```markdown
---
paths: ["sql/**/*.sql"]
---
- Use Spark SQL dialect. Qualify tables as prod.db.table.
- No SELECT *; list columns.
```
**.claude/commands/maintain.md:**
```markdown
---
description: Run Iceberg maintenance on a table
argument-hint: [catalog.db.table]
allowed-tools: Bash(spark-sql:*), Read
model: haiku
---
Run maintenance for $ARGUMENTS: rewrite_data_files, then expire_snapshots RETAIN_LAST 5,
then remove_orphan_files (dry_run true first). Positional: table=$1.
```
(`$ARGUMENTS` = full string; `$1/$2` positional.)

**.claude/agents/reviewer.md:**
```markdown
---
name: iceberg-reviewer
description: Reviews Spark/Iceberg PRs for partition and MERGE correctness
tools: Read, Grep, Glob
model: claude-opus-4-6
---
Review diffs for: full-overwrite anti-patterns, missing partition filters, unhandled schema evolution.
```
**.claude/skills/iceberg-maintenance/SKILL.md (spec-compliant):**
```markdown
---
name: iceberg-maintenance
description: Compact, expire snapshots, and remove orphan files on an Iceberg table.
allowed-tools: Bash, Read
---
Steps: 1) rewrite_data_files 2) expire_snapshots retain_last 5 3) remove_orphan_files dry_run 4) rewrite_manifests.
```

**Output styles / hooks / plugins / auto memory / imports / /init / /memory:**
- **Hooks:** `.claude/settings.json` with `PreToolUse`/`PostToolUse` matchers + `command`; a `PreToolUse` hook can *block* actions (instructions cannot).
- **Plugins:** manifest + marketplaces; plugin skills appear alongside local skills.
- **Auto memory:** `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` to disable; `--add-dir` + env to load CLAUDE.md from extra dirs.
- **@import:** `@path/to/file` (relative/absolute; home-dir imports for personal notes). [gemini-cli](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html)
- **/init** scaffolds a CLAUDE.md; **/memory** lists every CLAUDE.md/CLAUDE.local.md/rules file active.

**Cross-tool mapping:**
| Concept | Claude Code | VS Code Copilot | OpenAI Codex | Gemini CLI |
|---|---|---|---|---|
| Always-on project rules | `CLAUDE.md` | `.github/copilot-instructions.md` / `AGENTS.md` | `AGENTS.md` | `GEMINI.md` (or `AGENTS.md` via `context.fileName`) |
| User/global rules | `~/.claude/CLAUDE.md` | `~/.copilot/instructions` / user AGENTS.md | `~/.codex/AGENTS.md` | `~/.gemini/GEMINI.md` |
| Path-scoped rules | `.claude/rules/*.md` | `.github/instructions/*.instructions.md` (`applyTo`) | nested `AGENTS.md` | nested `GEMINI.md` |
| Slash command | `.claude/commands/*.md` | `.github/prompts/*.prompt.md` | custom prompts | `.gemini/commands/*.toml` |
| Skill | `.claude/skills/<n>/SKILL.md` | `.github/skills/<n>/SKILL.md` | Skills (`SKILL.md`) | extensions |
| Subagent | `.claude/agents/*.md` | `.github/agents/*.agent.md` | `.codex/agents/*.toml` | (extensions) |
| Config | `.claude/settings.json` | VS Code settings + `.vscode/mcp.json` | `~/.codex/config.toml` | `.gemini/settings.json` |
| Override | — | — | `AGENTS.override.md` | — |

**Which Claude files VS Code Copilot actually reads:** `AGENTS.md` (`chat.useAgentsMdFile`), `CLAUDE.md` (`chat.useClaudeMdFile`), and Claude **skills** at `~/.claude/skills/*/` and `${workspaceRoot}/.claude/skills/*/` (`chat.useClaudeSkills`). [GitHub](https://github.com/microsoft/vscode-docs/issues/9151) It does not natively read `.claude/rules` or `.claude/commands` as Copilot instructions.

---
## SECTION 4 — Writing Instruction Files for Codex, GPT, Gemini, Claude

**Universal dos/don'ts:** be specific and concise; give concrete code examples; reference (link) rather than duplicate; scope with globs/paths; keep files short (context is a budget); avoid vague "write good code."

**OpenAI Codex / GPT-5.x:** `AGENTS.md` is the source of truth; Codex concatenates `~/.codex/AGENTS.md` + every `AGENTS.md` from git root down to cwd (~32 KiB cap; closest = highest precedence). [CodeGateway](https://www.codegateway.dev/en/blog/agents-md-playbook-2026) `AGENTS.override.md` overrides per directory; custom filenames via `~/.codex/config.toml` `project_doc_fallback_filenames`/`project_doc_max_bytes`. Custom agents: `.codex/agents/*.toml` / `~/.codex/agents/*.toml` with `name`, `description`, `developer_instructions`. Skills: `SKILL.md` referenced via `[[skills.config]]`. Sandbox (OS) and approval policy (CLI) are independent dials. **Bad:** `Be a helpful Spark expert.` **Good:**
```markdown
# AGENTS.md
- PySpark 4.1 + Iceberg 1.11, catalog `prod`.
- Upserts: MERGE INTO only. Never INSERT OVERWRITE a partitioned table.
- Before finishing: run `pytest -q` and `sqlfluff lint sql/ --dialect sparksql`.
```

**Anthropic Claude:** short, imperative CLAUDE.md; use `@imports` for personal notes; use `PreToolUse` hooks to *enforce* (instructions are context, not enforcement); pick cheap models (Haiku) for boilerplate commands. **Bad:** `Follow best practices.` **Good:** `- Every maintenance job must call expire_snapshots with retain_last >= 5; block commits that don't (PreToolUse hook).`

**Google Gemini (Gemini CLI):** `GEMINI.md` hierarchy (global `~/.gemini/GEMINI.md` → project root → per-dir), concatenated every prompt; [Localskills](https://localskills.sh/blog/gemini-md-guide) rename/consolidate via `settings.json` `context.fileName` (set to `"AGENTS.md"` to share one file); [Localskills](https://localskills.sh/blog/gemini-md-guide) `@file.md` imports; [gemini-cli](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html) `/memory show|refresh|add`; [gemini-cli](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html) custom commands in `.gemini/commands/*.toml` (`description`, `prompt` with `{{args}}`); [Phil Schmid](https://www.philschmid.de/gemini-cli-cheatsheet) extensions via `gemini-extension.json` (`mcpServers`, `contextFileName`, `excludeTools`). [Phil Schmid](https://www.philschmid.de/gemini-cli-cheatsheet) **Gemini Code Assist in VS Code:** agent mode + rules/context files. **Bad:** `Help with data.` **Good:**
```markdown
# GEMINI.md
- Spark SQL dialect; Iceberg tables under prod.*.
- When asked to optimize, suggest rewrite_data_files sort/zorder, not manual repartition.
```

**Model-specific instructions inside VS Code Copilot:** pin a custom agent to a model with tailored body instructions (e.g., an "Iceberg Planner" on `GPT-5.4 (copilot)` and an "Implementer" on `Claude Sonnet 4.6 (copilot)`), and use `applyTo` instructions per path. There is no per-model instruction file; use custom agents + prompt files' `model:`.

**Final multi-tool repo layout (full contents):**

`AGENTS.md` (source of truth):
```markdown
# Iceberg Lakehouse — Agent Instructions
Stack: PySpark 4.1, Spark SQL, Apache Iceberg 1.11, REST catalog `prod`, MinIO/S3.
Rules:
- Upserts via MERGE INTO; never overwrite partitioned Iceberg tables.
- Reference tables by name (prod.db.table); never hardcode S3 URIs.
- Maintenance: rewrite_data_files, expire_snapshots (retain_last>=5), remove_orphan_files (dry_run first), rewrite_manifests.
- Tests: pytest with a local SparkSession fixture; run `pytest -q` before commits.
- SQL: no SELECT *; qualify columns; Spark SQL dialect.
```
`CLAUDE.md`: `See @AGENTS.md for all project rules. Personal prefs: @~/.claude/de-prefs.md`
`GEMINI.md`: `See @AGENTS.md for all project rules.` (+ `.gemini/settings.json`: `{ "context": { "fileName": ["GEMINI.md", "AGENTS.md"] } }`)
`.github/copilot-instructions.md`: `Follow AGENTS.md. PySpark 4.1 + Iceberg 1.11; MERGE INTO for upserts; run pytest -q before commits.`

---
## SECTION 5 — Microsoft 365 Copilot Chat Deep Dive (secondary)

**Where it runs:** Teams, Outlook, Word/Excel/PowerPoint, Edge sidebar, the Microsoft 365 Copilot app, and the web. **Free vs licensed:** the entitled Microsoft 365 Copilot license unlocks work-grounding (emails/files/meetings via Microsoft Graph), Researcher/Analyst, Pages, and Agent Builder; the free/metered "Copilot Chat" tier is web-grounded and limited on work data.

**Feature set (2026):** **Pages** (collaborative canvas), **Notebooks**, **file upload** (CSV/Excel + docs; analysis via **Analyst**), **Researcher**, **Analyst**, **Agent Builder** and **Copilot Studio** agents, **custom instructions/memory**, and model choice. Governance via **Agent 365** (GA) and Copilot Studio. GA of the reasoning agents began May 30, 2025 (announced June 2, 2025): per the Microsoft 365 Blog, "we're excited to announce the general availability of Researcher and Analyst… Now, these powerful agents are available to everyone with a Microsoft 365 Copilot license." [Microsoft](https://www.microsoft.com/en-us/microsoft-365/blog/2025/06/02/researcher-and-analyst-are-now-generally-available-in-microsoft-365-copilot/) Researcher, per Microsoft, "combines OpenAI's deep research model with Microsoft 365 Copilot's advanced orchestration and deep search capabilities," [Microsoft](https://www.microsoft.com/en-us/microsoft-365/blog/2025/06/02/researcher-and-analyst-are-now-generally-available-in-microsoft-365-copilot/) and Analyst is "built on OpenAI's o3-mini reasoning model and optimized to do advanced data analysis… Can run Python to tackle your most complex data queries—and you can view the code it's running in real time." [Ohio State University](https://it.osu.edu/news/2025/07/22/new-microsoft-365-copilot-agents-available-research-and-analysis)

**8–10 DE scenarios (exact prompts):**
1. Iceberg table design doc: *"Draft a design doc for an Iceberg `orders` table partitioned by days(event_ts): schema, partition spec, retention, compaction cadence."*
2. CSV data-quality profile (Analyst): upload `orders_sample.csv` → *"Profile null rates, cardinality, and outliers per column; flag columns unsuitable as a merge key."*
3. Spark incident summary (Teams): *"Summarize this channel thread into root cause, impact, and action items for the OOM in the nightly compaction job."*
4. Snapshot-expiry runbook: *"Write a runbook for expiring Iceberg snapshots older than 7 days across 40 tables, incl. rollback."*
5. Migration status update: *"From these three status emails, draft a one-page Hive→Iceberg migration update for leadership."*
6. Excel model review: *"Analyze this capacity Excel and forecast storage growth if we retain 5 vs 10 snapshots."*
7. Meeting prep (Researcher): *"Research Iceberg v3 deletion vectors and produce a 5-slide overview."*
8. Ticket triage: *"Cluster these 200 data-quality tickets by table and suggest owners."*
9. Policy doc: *"Draft a data-retention policy aligning Iceberg snapshot expiry with GDPR."*
10. Onboarding Page: *"Create a Page summarizing our lakehouse architecture for new data engineers."*

**What it CANNOT do vs GitHub Copilot in VS Code:** it does not edit your repo, run PySpark/pytest, run `spark-submit`, execute terminal commands against your cluster, use `.vscode/mcp.json`/agent tools, or open PRs. Its Analyst Python sandbox analyzes uploaded data, not your production Spark jobs.

**"Which Copilot should I use?" (all current):**
- **GitHub Copilot in VS Code** — write/run/refactor code, agent mode, tests, PRs. *Primary.*
- **Microsoft 365 Copilot Chat** — docs, email, meetings, CSV/Excel analysis, runbooks. *Secondary.*
- **Copilot in Microsoft Fabric** — notebooks/pipelines/Data Warehouse/DAX inside Fabric.
- **Copilot in Azure** — Azure resource management, KQL, infra guidance.
- **Copilot in SSMS 22** — T-SQL authoring with a model picker (GPT-5.x, Claude Sonnet/Opus, Gemini). [Microsoft Learn](https://learn.microsoft.com/en-us/ssms/github-copilot/ai-models)
- **Consumer Microsoft Copilot** — general web assistant; no work-data grounding.

---
## SECTION 6 — Hands-On Lab Repository: `iceberg-copilot-lab`

**File tree:**
```
iceberg-copilot-lab/
├─ AGENTS.md  ├─ CLAUDE.md  ├─ GEMINI.md  ├─ README.md
├─ docker-compose.yml  ├─ requirements.txt  ├─ .vscode/mcp.json
├─ .github/
│  ├─ copilot-instructions.md
│  ├─ instructions/pyspark.instructions.md
│  ├─ instructions/sql.instructions.md
│  ├─ prompts/maintain-table.prompt.md
│  ├─ agents/iceberg-planner.agent.md
│  ├─ skills/iceberg-maintenance/SKILL.md
│  ├─ skills/iceberg-maintenance/scripts/compact.py
│  └─ hooks/format.json
├─ jobs/ingest_orders.py  ├─ jobs/maintenance.py
├─ sql/create_orders.sql  ├─ sql/upsert_orders.sql
└─ tests/conftest.py  └─ tests/test_ingest_orders.py
```

**docker-compose.yml** (verified maintained images — Apache REST fixture + MinIO + Spark):
```yaml
services:
  spark-iceberg:
    image: tabulario/spark-iceberg
    container_name: spark-iceberg
    depends_on: [rest, minio]
    volumes:
      - ./warehouse:/home/iceberg/warehouse
      - ./notebooks:/home/iceberg/notebooks/notebooks
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
    ports: ["8888:8888", "8080:8080", "10000:10000", "10001:10001"]
  rest:
    image: apache/iceberg-rest-fixture
    container_name: iceberg-rest
    ports: ["8181:8181"]
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
      - CATALOG_WAREHOUSE=s3://warehouse/
      - CATALOG_IO__IMPL=org.apache.iceberg.aws.s3.S3FileIO
      - CATALOG_S3_ENDPOINT=http://minio:9000
  minio:
    image: minio/minio
    container_name: minio
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=minio
    ports: ["9001:9001", "9000:9000"]
    command: ["server", "/data", "--console-address", ":9001"]
```

**requirements.txt:** `pyspark==4.1.0`  `pytest>=8`  `pandas`

**sql/create_orders.sql:**
```sql
CREATE TABLE IF NOT EXISTS prod.sales.orders (
  order_id bigint, customer_id bigint, amount decimal(12,2),
  status string, event_ts timestamp, updated_at timestamp
) USING iceberg
PARTITIONED BY (days(event_ts));
```
**sql/upsert_orders.sql:**
```sql
MERGE INTO prod.sales.orders t
USING orders_updates s ON t.order_id = s.order_id
WHEN MATCHED AND s.updated_at > t.updated_at THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```
**jobs/ingest_orders.py:**
```python
from pyspark.sql import SparkSession, functions as F, Window

def build_spark(app: str = "ingest_orders") -> SparkSession:
    return (SparkSession.builder.appName(app)
            .config("spark.sql.catalog.prod", "org.apache.iceberg.spark.SparkCatalog")
            .config("spark.sql.catalog.prod.type", "rest")
            .config("spark.sql.catalog.prod.uri", "http://rest:8181")
            .getOrCreate())

def upsert_orders(spark: SparkSession, src_path: str, table: str = "prod.sales.orders") -> None:
    df = spark.read.parquet(src_path)
    w = Window.partitionBy("order_id").orderBy(F.col("updated_at").desc())
    latest = df.withColumn("_rn", F.row_number().over(w)).filter("_rn = 1").drop("_rn")
    latest.createOrReplaceTempView("orders_updates")
    spark.sql(f"""
        MERGE INTO {table} t
        USING orders_updates s ON t.order_id = s.order_id
        WHEN MATCHED AND s.updated_at > t.updated_at THEN UPDATE SET *
        WHEN NOT MATCHED THEN INSERT *
    """)

if __name__ == "__main__":
    spark = build_spark(); upsert_orders(spark, "s3a://raw/orders/"); spark.stop()
```
**jobs/maintenance.py:**
```python
from pyspark.sql import SparkSession

def maintain(spark: SparkSession, table: str = "prod.sales.orders") -> None:
    spark.sql(f"CALL prod.system.rewrite_data_files(table => '{table}')")
    spark.sql(f"CALL prod.system.rewrite_manifests('{table}')")
    spark.sql(f"CALL prod.system.expire_snapshots(table => '{table}', retain_last => 5)")
    spark.sql(f"CALL prod.system.remove_orphan_files(table => '{table}', dry_run => true)")
```
**tests/conftest.py:**
```python
import pytest
from pyspark.sql import SparkSession

@pytest.fixture(scope="session")
def spark():
    s = (SparkSession.builder.master("local[2]").appName("tests")
         .config("spark.sql.catalog.local", "org.apache.iceberg.spark.SparkCatalog")
         .config("spark.sql.catalog.local.type", "hadoop")
         .config("spark.sql.catalog.local.warehouse", "/tmp/wh")
         .getOrCreate())
    yield s
    s.stop()
```
**tests/test_ingest_orders.py:**
```python
from pyspark.sql import Row
from jobs.ingest_orders import upsert_orders

def test_dedup_keeps_latest(spark):
    spark.sql("CREATE TABLE IF NOT EXISTS local.db.orders (order_id bigint, updated_at timestamp) USING iceberg")
    df = spark.createDataFrame([Row(order_id=1, updated_at="2026-01-01 00:00:00"),
                                Row(order_id=1, updated_at="2026-02-01 00:00:00")])
    df.write.mode("overwrite").parquet("/tmp/orders_src")
    upsert_orders(spark, "/tmp/orders_src", "local.db.orders")
    rows = spark.sql("SELECT count(*) c FROM local.db.orders WHERE order_id = 1").collect()
    assert rows[0]["c"] == 1
```
**.github/copilot-instructions.md:** `Follow AGENTS.md. PySpark 4.1 + Iceberg 1.11; MERGE INTO for upserts; qualify tables prod.*; run pytest -q before commits.`
**.github/instructions/pyspark.instructions.md:**
```markdown
---
applyTo: "jobs/**/*.py"
---
- Build SparkSession via build_spark(); catalog `prod` is a REST catalog.
- Dedup with row_number() window before MERGE.
- Type hints on all functions; no bare except.
```
**.github/instructions/sql.instructions.md:**
```markdown
---
applyTo: "sql/**/*.sql"
---
- Spark SQL dialect. Iceberg DDL uses USING iceberg + PARTITIONED BY transforms.
- No SELECT *. Use MERGE INTO for upserts.
```
**.github/prompts/maintain-table.prompt.md:**
```markdown
---
description: Generate/run Iceberg maintenance for a table
argument-hint: <catalog.db.table>
agent: agent
model: Claude Sonnet 4.6 (copilot)
tools: ['edit', 'search', 'execute']
---
For ${input:table}: add a maintenance call sequence (rewrite_data_files, rewrite_manifests,
expire_snapshots retain_last 5, remove_orphan_files dry_run true) in jobs/maintenance.py and run pytest.
```
**.github/agents/iceberg-planner.agent.md:**
```markdown
---
name: Iceberg Planner
description: Plans Iceberg migrations/maintenance; no edits.
tools: ['search', 'web']
model: GPT-5.4 (copilot)
handoffs:
  - label: Implement
    agent: agent
    prompt: Implement the plan above.
    send: false
---
Produce a numbered plan only. Consider partition evolution, backfill, and validation via metadata tables.
```
**.github/skills/iceberg-maintenance/SKILL.md:**
```markdown
---
name: iceberg-maintenance
description: Compact, expire snapshots, and remove orphan files on an Iceberg table.
allowed-tools: Bash, Read
---
Run scripts/compact.py <table>. Steps: rewrite_data_files → expire_snapshots retain_last 5 → remove_orphan_files dry_run → rewrite_manifests.
```
**.github/skills/iceberg-maintenance/scripts/compact.py:**
```python
import sys
from jobs.maintenance import maintain
from jobs.ingest_orders import build_spark

if __name__ == "__main__":
    table = sys.argv[1] if len(sys.argv) > 1 else "prod.sales.orders"
    maintain(build_spark("maintenance"), table)
```
**.github/hooks/format.json:**
```json
{ "hooks": { "PostToolUse": [ { "type": "command", "command": "black jobs/ tests/" } ] } }
```
**.vscode/mcp.json:** github + postgres + dbt (see Section 2 example). **AGENTS.md / CLAUDE.md / GEMINI.md:** as in Section 4.

**Guided lab exercises (prompt → expected → verification):**
1. *Ask:* "Explain days() partitioning" → prose → learner restates.
2. *Instructions:* add `pyspark.instructions.md` → agent uses `build_spark()` → check "Used references".
3. *Agent build ingest:* Lesson-Agent prompt → `ingest_orders.py` + test → `pytest -q` passes.
4. *MERGE:* "make upsert idempotent" → dedup window added → `test_dedup_keeps_latest` passes.
5. *Maintenance prompt file:* `/maintain-table prod.sales.orders` → `maintenance.py` → run; `SELECT count(*) FROM prod.sales.orders.snapshots` ≤ 5 retained.
6. *Skill:* `/iceberg-maintenance` → runs compact.py → `SELECT count(*) FROM prod.sales.orders.files` decreases.
7. *Branch/tag:* "create audit branch, write, validate, fast-forward" → `ALTER TABLE … CREATE BRANCH audit RETAIN 7 DAYS` → `SELECT * FROM prod.sales.orders.refs`.
8. *Schema evolution:* "add column `channel`" → `ALTER TABLE … ADD COLUMN` → describe table.
9. *Time travel:* "read table AS OF yesterday" → `VERSION AS OF`/`TIMESTAMP AS OF` → row counts differ.
10. *MCP:* connect postgres MCP → "compare row counts source vs Iceberg" → numbers match.
11. *Hooks:* enable format hook → edit a job → `black` reformats automatically.
12. *Code review:* run Copilot review → flags a full-overwrite → fix and re-review clean.

---
## SECTION 7 — Front-End Build Specs (Astro + Starlight)

Target current Astro + Starlight; React islands via `@astrojs/react`; Motion (Framer Motion), GSAP, Lottie, Shiki (Starlight built-in), Mermaid, React Flow. All components honor `prefers-reduced-motion`, are keyboard-navigable, and expose ARIA roles.

### CopilotChatReplay
- **Props:** `script: Turn[]`, `speed?: number`, `autoplay?: boolean`.
- **JSON:** `{ "turns": [ { "role": "user"|"assistant", "text": "…", "code?": {"lang":"python","content":"…"}, "delayMs": 400 } ] }`
- **States:** idle, playing, paused, done. **Timing:** 18ms/char typewriter; 400ms inter-turn; reduced-motion renders instantly. **a11y:** `role="log"` `aria-live="polite"`; play/pause; focusable transcript.
```tsx
export default function CopilotChatReplay({ script, speed = 18 }: {script: Turn[]; speed?: number}) {
  const reduce = usePrefersReducedMotion();
  return <div role="log" aria-live="polite">{/* typewriter unless reduce */}</div>;
}
```

### RoleSwitcher (Ask/Plan/Agent)
Props: `roles:{id,label,desc}[]`, `active`. `role="radiogroup"`, arrow-key nav, animated underline (200ms), capability matrix per role.

### AgentLoop
Props: `nodes`, `edges`, `creditMeter`. React Flow diagram Plan→Edit→Run→Observe→Done; draggable permissions dial (Default→Assisted→Bypass→Autopilot); GSAP pulses the active node; text-list fallback.

### FileTreeExplorer
Props: `tree`, `onSelect`, `instructionMap`. Clicking a file highlights which `.instructions.md`/`AGENTS.md` attach (unioned). `role="tree"`, keyboard nav.

### ContextCostSlider (instructions vs skills)
Props: `min,max,value`. Shows always-on instruction cost vs lazy skill loading; live token estimate. `role="slider"`, `aria-valuenow`.

### WhichFileWizard
Props: `questions[]`. Branching wizard → outputs `.instructions.md` / prompt file / custom agent / skill / AGENTS.md. JSON decision tree; keyboard-first.

### CreditCalculator
Props: `plans`, `multipliers`. 1 credit = $0.01. Inputs: requests/day, model multiplier. Notes Business $19/mo (1,900 credits), Enterprise $39/mo (3,900 credits). Live output; `aria-live`.

### ClaudeFileHierarchyVisualizer
Props: `layers` (enterprise→project→user→local). Animated precedence stack; per-layer tooltip; marks CLAUDE.local.md deprecated.

### CopilotComparisonFlipCards (VS Code vs M365)
Props: `pairs[]`. CSS 3D flip (reduced-motion → fade). Front = capability, back = which Copilot wins. Button-activatable.

### Quiz
Props: `questions[]` with `answerIndex`, `explanation`. States: unanswered/correct/incorrect. `role="group"`, `aria-describedby` feedback.

### TabbedCodeBlocks
Props: `tabs:{label,lang,code}[]`. Shiki-highlighted; copy button (`aria-label="Copy code"`); tablist keyboard nav; toast on copy.

**Graphics/image asset list (alt text):**
- `agent-loop.svg` — "Diagram of the Copilot agent loop: plan, edit, run, observe, repeat, done."
- `permissions-dial.svg` — "Four-position permission dial: Default Approvals, Assisted, Bypass, Autopilot."
- `instruction-precedence.svg` — "Stack showing instruction precedence from enterprise policy down to local files."
- `iceberg-maintenance.svg` — "Four Iceberg maintenance operations and the table layer each affects."
- `mcp-topology.svg` — "VS Code connecting via MCP to GitHub, Postgres, and dbt servers."
- `annotated-chat.png` — "Annotated VS Code Chat view: agents dropdown, model picker, permissions picker."
- `claude-hierarchy.svg` — "Claude memory file hierarchy with CLAUDE.local.md marked deprecated."

**Brand/trademark guidance:** use each vendor's official brand assets and follow their guidelines — GitHub Logos and Usage; Microsoft Brand/Trademark Guidelines; Anthropic/Claude brand guidelines; OpenAI Brand guidelines; Google Brand Resource Center / Gemini brand. Do not imply endorsement, do not alter logos, maintain clear space, and prefer text names where a logo licence is unclear. Link to each vendor's official brand page rather than hotlinking logo files.

## Recommendations
1. **Ship Section 1 corrections first** — update Part 1's tool names, the `(copilot)` model suffix, instruction paths, and the fictional MCP server before any build work; these are the highest-impact fixes.
2. **Adopt `AGENTS.md` as the single source of truth**, with thin `CLAUDE.md`/`GEMINI.md`/`copilot-instructions.md` referencing it; enable `chat.useAgentsMdFile`. Benchmark: a new engineer's first agent session obeys repo conventions with zero manual context.
3. **Default the team to Default Approvals + a terminal allow-list; reserve Autopilot for sandboxed worktrees.** Loosen when CI is green and sandbox is enabled (macOS/Linux). Tighten if any destructive command reaches an unexpected approval prompt.
4. **Pin Spark 4.1 + Iceberg 1.11 coordinates** in the lab; if you must stay on Spark 3.5 (LTS to Nov 2027), swap runtime JARs accordingly.
5. **Build interactive components in priority order:** CopilotChatReplay → RoleSwitcher → AgentLoop → FileTreeExplorer → CreditCalculator.
6. **Re-verify Preview/Experimental items quarterly** (hooks, nested AGENTS.md, Autopilot, `context: fork`, BYOK GA status) against release notes.

## Caveats
- Items marked Unverifiable in Section 1 (Plan save path; `chat.autopilot.*` settings; exact BYOK GA-vs-preview per plan) could not be confirmed in official docs as of Sept 16, 2026 — treat as tentative.
- The full VS Code `#`-namespaced tool reference strings (`web/fetch`, `search/codebase`, `execute/runInTerminal`) are used illustratively; confirm exact spellings on the VS Code "Use Tools" page before publishing verbatim.
- Model picker names reflect 2026 snapshots (SSMS/VS Code docs) and drift frequently — always defer to the live model picker.
- Preview/Experimental features (hooks, Autopilot advanced behaviors, nested AGENTS.md, skill `context: fork`) may change or be removed.
- Third-party model retention differs (e.g., Claude Fable 5 retains prompts/outputs for safety classifiers); review vendor Service Specific Terms before enabling.
- The Analyst reasoning-model attribution (OpenAI o3-mini) reflects Microsoft's 2025 GA announcement; Microsoft may have since updated the underlying model — verify in current M365 documentation.