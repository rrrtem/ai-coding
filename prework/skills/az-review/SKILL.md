---
name: az-review
description: "Automated multi-pass code review with parallel validation. Use when asked to review code, review changes, review a PR, review the diff, do a code review, check my changes, or '/review'. Triggers: 'review', 'code review', 'review PR', 'review diff', 'check changes', 'ревью', 'проверь код'."
---

# Code Review — az-review

Deep multi-perspective parallel code review with validation layer. All review agents run as **opus** for maximum analytical depth.

**Philosophy:** Find the ROOT CAUSE, not the symptom. If a null check is missing at line 50, the real bug might be a broken contract at line 20 or a missing upstream validation. Report where the fix should actually go.

Only **HIGH SIGNAL** issues — objective bugs, logic errors, architectural violations, broken flows, and clear rule violations. No subjective concerns, no nitpicks.

Do not use AskUserQuestion. Complete the entire review autonomously.

---

## Step 1: Gather Context

Run in parallel:

### 1a. Find CLAUDE.md / AGENTS.md files

Launch a **haiku** agent:
- Get list of changed files (paths only, not contents) using workspace diff with stat option
- Find CLAUDE.md and AGENTS.md files in: repo root, and every directory containing a modified file (walk up to root)
- Return the list of found config file paths

### 1b. Get PR context

If this workspace has an associated PR:
```bash
gh pr view --json title,body -q '.title + "\n\n" + .body'
```
If no PR exists, skip. This is optional context.

---

## Step 2: Summarize Changes

Launch an **opus** agent to view the full diff and return a structured summary: files changed, nature of changes, areas affected, data flow across changed files, and which user-facing flows are touched.

---

## Step 3: Parallel Deep Review

Launch **6 opus agents in parallel**. Each examines the diff independently with maximum analytical depth. Provide each agent with: PR title/description (from Step 1b), CLAUDE.md/AGENTS.md contents (from Step 1a), change summary (from Step 2).

**CRITICAL: All agents must NOT post comments. Return issues only. Only the main agent posts comments.**

**ROOT CAUSE PRINCIPLE (applies to ALL agents):** Do not just flag the symptom. Trace the issue to its origin. If a variable is null at line 50, the bug might be a missing assignment at line 20 or a broken API contract in another file. Always read surrounding code, imports, and callers/callees to find where the fix should actually go. Report both the symptom location AND the root cause location.

### Agent 1 — CLAUDE.md & Convention Compliance (opus)

Full audit of changes against ALL discovered CLAUDE.md/AGENTS.md rules. Check both:
- **Structural rules:** API handler wrappers, Zod schemas, logger usage, config access, query helpers, soft-delete conventions
- **Workflow rules:** migration conventions, naming, testing requirements, documentation requirements, dependency pinning

Only consider config files scoped to directories containing changed files (same directory or parent). Quote the exact rule being violated with its section heading.

### Agent 2 — Bug Detection & Root Cause Tracing (opus)

Scan the diff AND surrounding code for bugs, always tracing to root cause:
- **Null/undefined:** missing null checks, optional chaining on required paths, unguarded access after async calls
- **Off-by-one & boundaries:** loop bounds, array indexing, pagination, slice/substring
- **Error handling:** swallowed errors, missing catch blocks, error responses that leak internals, unhandled promise rejections
- **Race conditions:** concurrent state mutations, missing locks, stale closures, unguarded shared state
- **Type mismatches:** implicit coercions, wrong generic params, `any` leaking through boundaries

**Key instruction:** When you find a symptom (e.g., potential crash at line X), read the code that produces the value — follow imports, function definitions, database queries. Report where the actual fix belongs, not just where it crashes.

### Agent 3 — Logic & Data Flow Analysis (opus)

Follow data through the ENTIRE changed flow end-to-end:
- **State mutations:** does state change correctly at every step? Are there impossible states?
- **Async flow:** are awaits in the right places? Can promises resolve in unexpected order?
- **Error propagation:** if step 2 of 5 fails, does the error reach the user correctly? Are there partial updates left behind?
- **Edge cases in conditionals:** what happens with empty arrays, zero values, empty strings, undefined optionals?
- **Data shape assumptions:** does the code assume a structure that the source (DB, API, props) doesn't guarantee?

Read the full function/component, not just the diff lines. Understand the flow before flagging.

### Agent 4 — Architecture & System Design (opus)

Evaluate changes in context of the system architecture. Read related files (imports, shared modules, similar patterns in the codebase) to assess:
- **Layer violations:** does UI code reach into DB directly? Does an API route contain business logic that belongs in a shared module?
- **Broken abstractions:** does the change bypass an existing abstraction instead of extending it? Does it duplicate logic that already exists elsewhere?
- **Coupling:** does the change create tight coupling between modules that should be independent?
- **Consistency:** does the pattern used here match how similar things are done in the rest of the codebase? (Check 2-3 similar files for comparison)
- **Missing error boundaries:** if this component/route fails, does the failure propagate safely or crash the parent?
- **Scalability concerns:** N+1 queries, unbounded loops, missing pagination, large payloads without streaming

### Agent 5 — Business Logic & UX Flow (opus)

Read the code as a **product engineer**. Understand what the feature is supposed to DO, then verify the implementation matches:
- **User flow completeness:** can the user reach a dead end? Is there missing feedback (no loading state, no error message, no success confirmation)?
- **State transitions:** are all valid transitions handled? What happens on back-navigation, refresh, or browser close mid-flow?
- **Data consistency:** if the user creates/edits/deletes something, is the UI updated everywhere that shows this data (lists, detail views, counts, caches)?
- **Optimistic updates:** if used, is there rollback on failure? Does the rollback restore the exact previous state?
- **Edge cases in UX:** empty states, very long text, special characters in input, rapid clicks, concurrent edits
- **Intent vs implementation:** based on the PR title/description and code comments, does the code actually achieve what it's supposed to?

### Agent 6 — Security & Data Integrity (opus)

Deep security analysis of introduced code:
- **Injection:** SQL injection (especially raw queries), XSS (unescaped user content in HTML), command injection, path traversal
- **Auth & access control:** missing auth checks, broken authorization (user A accessing user B's data), privilege escalation
- **Data exposure:** secrets in logs or responses, excessive data in API responses, PII leaks
- **Data integrity:** partial writes without transactions, missing foreign key checks, orphaned records on delete
- **Input validation:** missing or insufficient validation at system boundaries (API routes, form submissions, webhook handlers)
- **Unsafe patterns:** `eval()`, `dangerouslySetInnerHTML` without sanitization, `new Function()`, unvalidated redirects

### Issue Format

Each agent returns issues as:
```
ISSUE: [description — what is wrong]
ROOT_CAUSE: [where the actual fix should go and why — trace from symptom to origin]
FILE: [path where the fix belongs]
SYMPTOM_FILE: [path where the problem manifests, if different from FILE]
LINE: [line number in diff]
REASON: [bug | logic-error | architecture | business-logic | ux-flow | claude-md-violation | security | data-integrity]
SEVERITY: [critical — breaks functionality | major — significant impact | moderate — edge case or degraded experience]
EVIDENCE: [quoted code or quoted rule]
```

---

## Step 4: Validate Issues

For each issue from Step 3, launch a **parallel opus validation subagent**. Every validator must:

1. **Read the actual code** (not just the diff) — open the file, read surrounding context (50+ lines around the issue)
2. **Trace the root cause** — if the issue claims a root cause in another file, open that file too and verify
3. **Check if already handled** — the issue might be addressed by code the reviewer didn't see (try-catch upstream, validation middleware, type guard in caller)
4. **Confirm severity** — is this really critical/major/moderate? Would a senior engineer flag this in a real review?

Each validator returns `VALID` or `FALSE_POSITIVE` with detailed reasoning.

### False Positive Criteria (exclude these)

- Pre-existing issues (code not in the diff)
- Correct code that superficially looks wrong
- Pedantic nitpicks a senior engineer would not flag
- Issues a linter would catch (do NOT run the linter to verify)
- General code quality concerns not backed by a CLAUDE.md/AGENTS.md rule
- Issues silenced by lint-ignore comments in the code
- Subjective concerns or "suggestions"
- Style preferences not explicitly required by CLAUDE.md
- Potential issues that "might" be problems
- Anything requiring interpretation or judgment calls

**If you are not certain an issue is real, do not flag it.** False positives erode trust and waste reviewer time.

---

## Step 5: Report

### 5a. Post Inline Comments

For each validated issue, post **ONE comment per unique issue** using `mcp__conductor__DiffComment`.

Comment format:
```
**[CATEGORY]** Issue description

Evidence: [quoted code or CLAUDE.md rule with link]
```

Categories: `BUG`, `LOGIC`, `ARCHITECTURE`, `BUSINESS-LOGIC`, `UX-FLOW`, `CLAUDE.MD`, `SECURITY`, `DATA-INTEGRITY`

Cite and link each issue — if referring to a CLAUDE.md rule, include a link to the file. For root cause issues, show both the symptom and the root cause location.

### 5b. Summary

Write a numbered list of all validated issues:

```
### #1 [Short title] — [SEVERITY]

**Category:** [CATEGORY]
**File:** [path]:[line]
**Root cause:** [where the fix belongs and why, if different from symptom location]

[Description — what is wrong and what should be done]

### #2 ...
```

If zero issues found, state that the review found no issues.

---

## Environment Detection

### Conductor (preferred)

- `mcp__conductor__GetWorkspaceDiff` → use for diffs (pass `stat: true` for file list, `file: "path"` for specific file, no params for full diff)
- `mcp__conductor__DiffComment` → use for inline comments
- Subagents available via Agent tool → use parallel agents

### Standalone Claude Code / Codex (fallback)

If Conductor MCP tools are unavailable:

```bash
# Get merge base
MERGE_BASE=$(git merge-base origin/main HEAD)

# Committed changes on this branch
git diff $MERGE_BASE HEAD

# Uncommitted changes
git diff HEAD

# Stat (file list)
git diff $MERGE_BASE HEAD --stat
```

For comments without DiffComment: output to console only (or use `gh pr review` if a PR exists).

If subagents are unavailable: run all review passes sequentially yourself.

Use `gh` CLI for GitHub interactions. Do not use web fetch.

No need to mention which environment path was used in the report.

---

## Step 6: Next Actions

### If issues were found

1. Create a fix plan — for each validated issue, describe what needs to change and where
2. Offer to fix them immediately
3. After fixes are applied, re-run the review from Step 1 to verify

### If no issues (clean review)

Check if a PR already exists for the current branch:
```bash
gh pr view --json url 2>/dev/null
```

If no PR exists, invoke `/az-pr` to create one.
