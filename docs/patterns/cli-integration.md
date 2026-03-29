# CLI Integration Pattern

Wrap any command-line tool as a MARVIN skill for deeper integration than raw CLI access provides.

## Why CLI-First?

MARVIN connects to external tools through three tiers:

1. **CLI tools** (preferred) - Purpose-built CLIs are maintained by the tool vendor, handle their own auth, and work reliably. Examples: `gws` (Google Workspace), `gh` (GitHub), `npx skills` (agent skills).
2. **MCP servers** - For tools without CLIs. Work well but add a dependency layer.
3. **Custom scripts** - Last resort for tools with neither CLI nor MCP support.

**Why prefer CLIs over MCP?**
- Auth is handled by the CLI itself (no OAuth token juggling)
- Maintained by the vendor (updates, bug fixes, new features)
- Work the same way whether called by you or by MARVIN
- No server process to manage
- Better error messages and documentation

## The Pattern: Skill-as-CLI-Wrapper

A bare CLI gives MARVIN access to a tool. A skill-wrapped CLI gives MARVIN *understanding* of the tool: when to use it, how to interpret output, what the priority rules are.

### Anatomy of a CLI Wrapper Skill

```yaml
---
name: tool-name
description: What this skill does with the CLI
---
```

**Section 1: Configuration**
How to set up the CLI (install, auth, config directories).

**Section 2: Command Reference**
Key commands, common flags, and usage patterns. Not an exhaustive manual, just what MARVIN needs.

**Section 3: Triage / Priority Rules**
How to interpret and route output. For example, email triage priorities, calendar event classification, issue severity mapping.

**Section 4: Workflow Patterns**
Common multi-step workflows. For example: "check email, categorize by priority, surface P1 items."

**Section 5: Account-Specific Config** (if applicable)
Environment variables, config directories, or flags for switching between accounts.

### Example: Google Workspace via gws CLI

Here's a simplified example wrapping the `gws` CLI for email triage:

```yaml
---
name: gws-email
description: Email triage and calendar management via gws CLI
---
```

**Configuration:**
```bash
# Install
pip install google-workspace-cli  # or however gws installs

# Authenticate
GOOGLE_WORKSPACE_CLI_CONFIG_DIR=~/.config/gws gws auth login -s gmail,calendar

# Verify
gws gmail list --max-results 5
```

**Command Reference:**
```bash
# Email
gws gmail list [--max-results N] [--query "search"]
gws gmail read <message-id>
gws gmail +reply <message-id> --body "response"
gws gmail send --to <email> --subject "..." --body "..."

# Calendar
gws calendar list [--days N]
gws calendar create --title "..." --start "..." --end "..."
```

**Triage Rules:**
| Sender Pattern | Priority | Action |
|---------------|----------|--------|
| Your manager | P1 | Surface immediately |
| Direct reports | P1 | Surface immediately |
| External partners | P2 | Review today |
| Newsletters | Skip | Don't surface |
| Automated notifications | FYI | One-line summary |

**Workflow: Morning Email Triage**
1. Run `gws gmail list --max-results 20`
2. Categorize each by triage rules above
3. Surface P1 items with one-line summaries
4. Group P2 items as "Review Today" section
5. Skip everything else

### Example: GitHub via gh CLI

```yaml
---
name: github-ops
description: GitHub operations via gh CLI
---
```

**Command Reference:**
```bash
# PRs
gh pr list [--state open]
gh pr view <number>
gh pr create --title "..." --body "..."
gh pr merge <number> [--merge|--squash|--rebase]

# Issues
gh issue list [--label "bug"]
gh issue create --title "..." --body "..."

# Search
gh search repos <query>
gh search code <query>
```

**Triage Rules:**
| Event | Priority | Action |
|-------|----------|--------|
| PR review requested | P1 | Surface immediately |
| CI failure on your PR | P1 | Surface with error summary |
| Issue assigned to you | P2 | Add to daily briefing |
| Dependabot alerts | FYI | Batch weekly |

## Building Your Own CLI Wrapper

1. **Install the CLI** and verify it works manually
2. **Create a skill file** in `.claude/skills/your-tool.md`
3. **Document key commands** (just the ones MARVIN needs, not everything)
4. **Define triage rules** (how should MARVIN prioritize output?)
5. **Add workflow patterns** (common multi-step sequences)
6. **Symlink** to `~/.claude/skills/` for Claude Code auto-discovery
7. **Add routing rules** to CLAUDE.md if the skill should auto-activate

## Auth Patterns

Different CLIs handle auth differently:

| Pattern | Example | How It Works |
|---------|---------|-------------|
| OAuth browser flow | `gws auth login` | Opens browser, stores tokens locally |
| Token-based | `gh auth login` | Paste token or browser flow |
| API key env var | `OPENAI_API_KEY=...` | Set in `.env`, loaded at startup |
| Config file | `~/.config/tool/config.json` | CLI reads from standard location |

Store API keys in `.env` (never in source). For OAuth-based CLIs, auth once and the CLI manages token refresh.

## When to Use MCP Instead

CLIs aren't always the best choice:

- **No CLI exists** for the tool
- **Real-time streaming** is needed (MCP excels at this)
- **Complex multi-step auth** that a CLI doesn't handle well
- **The MCP server is vendor-maintained** and more reliable than a third-party CLI

In these cases, use MCP. The skill wrapper pattern still applies: create a skill that documents when and how to use the MCP tools, with triage rules and workflows.

## Tips

- Start with 3-5 key commands, not the full CLI manual
- Triage rules are what make the skill valuable, not just command docs
- Test workflows manually before codifying them
- Keep the skill focused on one tool per file
- Update the skill when the CLI gets new features you need
