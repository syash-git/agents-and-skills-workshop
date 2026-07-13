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

## 6. Take it further — add more MCP servers

The **GitHub MCP server is built in** to Copilot CLI (that's what you just enabled with `--enable-all-github-mcp-tools`). But MCP is an open protocol, and you can plug in any other server — Azure DevOps, Jira, Postgres, Playwright, your own internal APIs, etc. — by declaring them in an `mcp.json` config file.

### Where to put `mcp.json`

Drop an `mcp.json` (or `.mcp.json`) into your project so it's picked up whenever you launch Copilot from that repo — and commit it so your team gets the same servers:

- **Project root:** `./.mcp.json` — simplest option, sits next to your `README.md`.
- **`.github/` folder:** `./.github/mcp.json` — keeps it grouped with your other GitHub/Copilot config (workflows, `copilot-instructions.md`, etc.) and out of the root listing.

### Example: Azure DevOps MCP server

Add the [Azure DevOps MCP server](https://github.com/microsoft/azure-devops-mcp) so Copilot can list work items, read pipelines, and open PRs in ADO the same way it does for GitHub:

```json
{
  "mcpServers": {
    "azure-devops": {
      "command": "npx",
      "args": [
        "-y",
        "@azure-devops/mcp",
        "your-ado-organization"
      ],
      "env": {
        "AZURE_DEVOPS_PAT": "${env:AZURE_DEVOPS_PAT}"
      }
    }
  }
}
```

Replace `your-ado-organization` with your ADO org name, then set the `AZURE_DEVOPS_PAT` environment variable to a [Personal Access Token](https://learn.microsoft.com/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate) with the scopes you need (e.g., Work Items: Read & Write, Code: Read & Write).

Start a new Copilot session and confirm the server loaded:

```
/mcp list
/mcp show azure-devops
```

Then try it out:

```
List my active work items in the "Contoso" project and pick one I can knock out in under an hour.
```

> ⚠️ **Never commit secrets.** Reference tokens via `${env:VAR_NAME}` in `mcp.json` and keep the actual PAT in your shell environment or a secret manager — never inline in the JSON.

### More ideas

- **Fix another issue.** Run `/new` for a fresh session and pick a different one from the GitHub issues list.
- **Try `search_code`.** Ask *"Where else in this repo do we swallow exceptions with a bare `except`?"* and watch Copilot use GitHub's code search across the whole repo.
- **Combine with the `code-checklist` skill.** After opening the PR, ask *"Do a code quality check on the diff in my open PR."* The skill's five-section format will apply to the change set the MCP server fetches back.

---

**Back to:** [Workshop README](../README.md) • [Module 3 — Combine agent + skill](./03-combine-agent-and-skill.md)
