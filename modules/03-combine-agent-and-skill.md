# Module 3 — Combine the agent and the skill (10 min)

> **Prereq:** You've completed [Module 1](./01-custom-agent.md) (the `code-reviewer` agent) **and** [Module 2](./02-custom-skill.md) (the `code-checklist` skill). Both files should be committed under `.github/agents/` and `.github/skills/` respectively.

## 1. The whole point of this workshop

So far you have:

- An **agent** (`code-reviewer`) that shapes *how Copilot thinks* — persona, tone, severity grouping.
- A **skill** (`code-checklist`) that gives Copilot *a specific procedure to follow* — the five-category checklist.

Now let's run both at once.

## 2. Try it

Switch to the agent with `/agent code-reviewer` (or run `/agent` and pick it from the list). Then ask:

```
Do a code review of books.py against our team quality checklist.
```

Notice the prompt: it says both "code review" (the agent's job) and "team quality checklist" (matches the skill's `description`). This gives Copilot every signal it needs.

## 3. What to look for

You should see the two customizations **compose**:

- The **structure** comes from the skill: five numbered categories in the fixed order, followed by a Summary line.
- The **voice** comes from the agent: direct, specific, no fluff, concrete suggested fixes.
- Findings inside each category are labeled with the agent's severity vocabulary (**Bug / Risk / Improvement**) even though the skill didn't define those.

That blend — a persona following a playbook — is exactly what agents + skills are for. Neither one alone would give you both.

## 4. Quick discussion prompts

Take two minutes to think about (or discuss with the person next to you):

1. If you kept just one of the two, which would you keep for daily code reviews? Why?
2. What would you have to write into a single monolithic prompt every time to get the same effect without agents or skills?
3. Whose review of your PRs at work could this replace — or, more realistically, augment?

---

**Next module:** [04 — Use the GitHub MCP server →](./04-mcp-server.md)
