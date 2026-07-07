# Module 4 — Use the GitHub MCP server (10 min)

> **Prereq:** Ideally you've done [Modules 1–3](./01-custom-agent.md) so you can see how MCP composes with a custom agent and skill. If you're jumping straight here, that's fine too — this module works standalone against the starter file from [Section 1](../README.md#section-1--setup-5-min).

## 1. What you're building

Agents changed *how* Copilot thinks; skills changed *what procedure* it follows. Now let's change *what Copilot can reach*. In this module you'll turn on the built-in **GitHub MCP server** in Copilot CLI, ask it to browse this workshop repo's issues, and drive an end-to-end fix: find an issue → edit code → open a PR — all from inside the same terminal session.

## 2. Enable the GitHub MCP server

Exit any running Copilot session, then start a new one with all GitHub MCP tools turned on:

```bash
copilot --enable-all-github-mcp-tools
```

Confirm the server is loaded and the full toolset is available:

```
/mcp show github-mcp-server
```

You should see the server listed with roughly 100 tools (e.g., `list_issues`, `create_pull_request`, `get_file_contents`, `search_code`, `create_issue`).

> **Heads up.** MCP tool calls run **with your GitHub permissions**. Copilot will ask you to approve each call the first time. Read what it's about to do before clicking through — especially for write operations like `create_pull_request` or `create_issue`.

## 3. Find an issue to fix

This repo has a handful of open issues that map directly to the deliberate bugs in the `books` starter file you picked in Section 1 of the main README. Ask Copilot to browse them:

```
List the open issues on this repository and summarize them.
```

Notice which tool Copilot calls (it should be `list_issues`). Then let Copilot triage for you:

```
Which of these issues would be the easiest to fix? Pick one for me.
```

## 4. Fix it and open a PR

Now ask Copilot to do the whole loop — code change, commit, push, PR — in one prompt. Adjust the filename to whichever starter you're working in (`books.py` / `Books.cs` / `books.js`):

```
Fix that issue. Follow these steps:
1. Check out a new local branch for the fix.
2. Make the changes to books.py (adjust for your language).
3. Run the code (or a quick manual check) to confirm the fix works.
4. Commit and push the branch.
5. Open a pull request that references the issue in its body (e.g., "Closes #N").
```

> ⚠️ **Review every write.** Copilot will call `create_pull_request` (and possibly `create_or_update_file`) via MCP. Read the diff summary and the PR title/body it proposes before approving.

## 5. What to look for

- Copilot narrates its **MCP tool calls** as it goes (`list_issues`, `get_file_contents`, `create_pull_request`, etc.). That's the MCP client at work — no glue code required from you.
- The PR appears on GitHub, linked to the issue via `Closes #N`.
- If you also select the `code-reviewer` agent from Module 1 **before** asking for the fix, Copilot's PR body will follow that persona — direct, severity-grouped, concrete. That's the agent, skill, and MCP customizations *all* composing on one task.

## 6. Take it further

- **Fix another issue.** Run `/new` for a fresh session and pick a different one from the list.
- **Try `search_code`.** Ask *"Where else in this repo do we swallow exceptions with a bare `except`?"* and watch Copilot use GitHub's code search across the whole repo.
- **Combine with the `code-checklist` skill.** After opening the PR, ask *"Do a code quality check on the diff in my open PR."* The skill's five-section format will apply to the change set the MCP server fetches back.

---

**Back to:** [Workshop README](../README.md) • [Module 3 — Combine agent + skill](./03-combine-agent-and-skill.md)
