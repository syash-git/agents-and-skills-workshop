# Module 1 — Build a `code-reviewer` agent (15 min)

> **Prereq:** You've completed [Section 1 (Setup)](../README.md#section-1--setup-5-min) and skimmed [Section 2 (Concepts)](../README.md#section-2--concepts-agents-vs-skills-10-min) in the main README. In particular, you have `.github/agents/` in place and a starter file (`books.py` / `Books.cs` / `books.js`) at the repo root.

## 1. What you're building

A persona that tells Copilot: *"You are a senior engineer doing a code review. Focus on real correctness issues. Be direct, not fluffy."* When we invoke this agent later, Copilot's tone and priorities will shift compared to the default.

## 2. Create the agent file

Save the following as `.github/agents/code-reviewer.md`:

```markdown
---
name: code-reviewer
description: A senior engineer who reviews code for correctness, safety, and readability. Direct, specific, no fluff.
---

# Code Reviewer

You are acting as a senior software engineer performing a code review.

## How you review

- Read the code carefully before commenting.
- Focus on issues that would matter in production: correctness bugs, unsafe error handling, missing input validation, race conditions, and code that will be hard for a teammate to change safely.
- Ignore purely stylistic preferences unless they hurt readability.
- Prefer concrete suggestions ("replace X with Y because Z") over vague ones ("consider improving this").

## How you respond

- Group findings by severity: **Bugs** (definitely wrong), **Risks** (probably wrong or fragile), **Improvements** (nice to have).
- For each finding, show the offending snippet and a suggested fix.
- If the code looks fine, say so — do not invent issues to seem thorough.
```

> **Jargon check.** The block between the `---` lines is **YAML frontmatter**: structured metadata Copilot reads to register the agent. The `name` is how you refer to it; the `description` is what Copilot shows in menus.

## 3. Try it out

First, get a **baseline** with the default agent so you have something to compare against. In the project folder:

```bash
copilot
```

Then at the prompt (adjust the filename to whichever starter you picked):

```
Review books.py and tell me what you think.
```

Read the response and note two or three things about it — probably a mix of style comments, general advice, and some correctness notes.

Now switch to your custom agent without leaving the session. Run the `/agent` command to browse and pick `code-reviewer` (or type `/agent code-reviewer` to select it directly). Then ask the same question:

```
Review books.py and tell me what you think.
```

## 4. What to look for

Regardless of which language you chose, the `code-reviewer` response should feel noticeably different from the baseline:

- Findings are **grouped by severity** (Bugs / Risks / Improvements) instead of a flat list.
- The bare `catch`/`except` block is called out as a **Bug** or **Risk**, not brushed past.
- The missing input validation on `title` and the case-insensitivity inconsistency in the search function are flagged specifically.
- The tone is more direct — fewer hedges like "you might want to consider".

If the output is still generic, double-check that you actually selected `code-reviewer` with `/agent` (Copilot shows the active agent in the prompt).

---

**Next module:** [02 — Build a `code-checklist` skill →](./02-custom-skill.md)
