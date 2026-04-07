---
name: harden
description: |
  Audit a software project for hardening — security, AI gaps, test coverage, code quality, and decoupling. Use when user wants to harden a project, audit for vulnerabilities, check test coverage, or separate private data from code.
license: MIT
compatibility: marvin
metadata:
  marvin-category: work
  user-invocable: true
  slash-command: /harden
  model: default
  proactive: false
---

# Project Hardening Audit

Perform a systematic hardening audit of this project. Work through each phase below, exploring the codebase to find real issues — not hypothetical ones. For each finding, explain the risk and suggest a concrete fix.

## When to Use

- When user types `/harden`
- When user wants to audit a project for security, test coverage, or code quality
- Before making a private repo public
- After a major feature or refactor to check for regressions
- When onboarding to an unfamiliar codebase to assess its health

## Process

### Step 1: Context Gathering (Phase 0)

Before scanning, ask these questions to calibrate the audit. The user can answer inline or say "skip" to assume worst-case (strictest ratings).

1. **Visibility** — Is this repo private, or could it go public?
2. **Access** — Just you, a team, or open source?
3. **Deployment** — Local only, server, or cloud?
4. **Compliance** — Any regulatory or legal requirements? (government forms, PII rules, HIPAA, etc.)
5. **Known issues** — Areas you already know are problematic? (prioritize or skip)

Use the answers to calibrate severity ratings throughout the audit. For example:
- "Private repo, solo access" — PII in git is medium, not critical
- "Going public" — PII in git is critical
- "Government forms" — output validation is high priority

If the user skips, assume: public visibility, shared access, compliance required.

**After calibration, state assumptions:**
Based on the answers, tell the user what this audit covers and what it doesn't. For example:
- "Covering: source code, config files, git history, dependencies, test coverage"
- "Not covering: infrastructure, CI/CD pipeline, database security, runtime monitoring"
- "Does this match your expectations, or should I adjust?"

Tailor the scope assumptions to the project. A web app with a Dockerfile gets different assumptions than a CLI tool. Let the user confirm or adjust before proceeding.

### Step 2: Audit Scopes

Work through these one at a time. After each scope, summarize findings and ask if the user wants to go deeper or move to the next scope.

#### Scope 1: Security
- Input validation and output sanitization (including stderr/log leakage)
- Secrets in code, config, and git history (not just current files — check committed history)
- Permission and access control
- Dependency hygiene (pinning, unused deps expanding attack surface, known CVEs)
- File system access patterns (path traversal, unsafe reads/writes)

#### Scope 2: AI-Specific Gaps
- Prompt injection risks (user input flowing into prompts unsanitized)
- Data exposure through AI context (secrets, PII, or sensitive content visible to the model)
- Access control on AI interfaces (who can invoke the AI, rate limits, cost controls)
- Output validation (does the system verify AI outputs before acting on them?)
- Fragile model assumptions (hardcoded model names, unpinned versions, deterministic output expectations)

#### Scope 3: Test Coverage
- Map existing test coverage (what has tests, what doesn't)
- Flag high-risk untested code (touches files, network, secrets, user input, money)
- Verify security and AI findings from Scopes 1-2 have test coverage
- Identify missing error path tests (what happens when things fail?)
- Suggest highest-value tests to add first, prioritized by risk

#### Scope 4: Code Quality
- Linter and formatter configuration (is one configured? If not, recommend one for the language/framework)
- Error handling (silent failures, bare `except`, swallowed exceptions)
- Dead code and stale config (unused functions, abandoned imports, orphaned files from refactors)
- Edge cases (empty inputs, nulls, unexpected types, boundary values)
- Consistency (naming, patterns, structure across similar components)
- Network and retry behavior (connection limits, timeout handling, backoff)
- Hardcoded values that should be configurable (paths, filenames, magic numbers)

#### Scope 5: Decoupling & Data Separation
- Tightly coupled components that should be independent (shared state, circular dependencies, components modifying same files)
- Private or sensitive data committed to the repo (PII, personal content, customer data)
- `.gitignore` coverage for sensitive and generated files
- Environment-specific config committed as if universal
- Data that should live outside the repo (personal content, user-specific state, local config)
- Clear boundaries between framework code and user data

### Step 3: Checkpoint Questions

During each scope, if you encounter something ambiguous — **ask before rating severity**. Only ask questions the codebase can't answer.

Examples:
- "This file has what looks like a real SSN — is this test data or production?"
- "This API key is in a committed file — is this a throwaway/dev key or production?"
- "Two skills modify the same file — is this intentional or a gap?"

**If the codebase can answer it, don't ask. If only the user knows, ask.**

Do NOT turn the audit into an interview. The skill's strength is autonomous exploration. Reserve questions for decisions that change severity or skip/prioritize a finding.

### Step 4: Scorecard

After completing all scopes, present a scorecard with letter grades per scope.

**Grading formula:**
- Points per finding: Critical = 4, High = 3, Medium = 2, Low = 1
- **Any critical finding in a scope = D minimum for that scope (cannot grade above D until critical is resolved)**

| Grade | Points |
|-------|--------|
| A | 0 |
| B | 1-4 |
| C | 5-9 |
| D | 10-14 |
| F | 15+ |

**Overall grade:** Average of scope grades (A=4, B=3, C=2, D=1, F=0), rounded.

### Step 5: Batch Plan

Propose batches for fixing the findings.

**Batching rules:**
1. **Dependencies first** — if finding X blocks finding Y, X goes in an earlier batch
2. **Logical grouping** — findings that touch the same files or fix related problems go together
3. **Severity within batches** — higher severity batches come first when no dependency constraint

### Step 6: Issue Creation (Optional)

After presenting the batch plan, ask: **"Ready to create GitHub issues? I'll file them in batch order."**

Only create issues after the user reviews and approves the batches.

## Output Format

### Scorecard

```
| Scope | Grade | Findings |
|-------|-------|----------|
| Security | C | 1 critical, 2 high, 1 medium |
| AI | B | 2 high |
| Tests | D | 3 high |
| Code Quality | B | 1 medium, 2 low |
| Decoupling | C | 2 high |

Overall: C
```

### Batch Plan

```
### Batch 1 — [description] ([count] issues)
Resolves: [finding numbers]
Dependency: [what must be done first, or "None — do this first"]
Effort: [Low / Medium / High]
```

## Rules

- Explore the codebase yourself before asking questions. Read files, check configs, scan for patterns.
- Only flag real issues you find in the code — not theoretical risks.
- Prioritize findings by severity: critical > high > medium > low, calibrated by Phase 0 context.
- The scorecard and batch plan are mandatory outputs. Do not skip them.

---

*Skill created: 2026-04-06*
