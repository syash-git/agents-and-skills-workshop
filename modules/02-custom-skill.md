# Module 2 — Build a `code-checklist` skill (15 min)

> **Prereq:** [Module 1](./01-custom-agent.md) is not strictly required — this module stands on its own — but the "compose them together" module coming next assumes you have both.

## 1. What you're building

A **skill** — a written checklist Copilot will pull in automatically whenever your prompt looks like a code-quality request. Unlike an agent, you won't select it manually; a well-written `description` field does the matching.

## 2. Create the skill file

Skills live in a folder named after the skill, containing a `SKILL.md`. Create both:

```bash
mkdir -p .github/skills/code-checklist
```

Then save the following as `.github/skills/code-checklist/SKILL.md`:

```markdown
---
name: code-checklist
description: Team code quality checklist. Use whenever the user asks for a code review, quality check, or pre-merge review of source code in any language.
---

# Code Quality Checklist

When invoked, walk through **every** item below for the code in question and report findings. Do not skip categories even if nothing is wrong — say "no issues found" for that category so the reader knows it was checked.

## 1. Input validation

- Are inputs checked for null / empty / wrong type before use?
- Are numeric ranges and string lengths validated where they matter?

## 2. Error handling

- Are exceptions / errors caught **specifically**, not with a catch-all that hides bugs?
- Are error paths tested or at least logged?
- Are resources (files, connections) released on the error path?

## 3. Correctness

- Do loops terminate under all conditions?
- Are collections mutated safely while being iterated?
- Are comparisons (equality, case-sensitivity) consistent?

## 4. Readability & maintainability

- Are names descriptive (a new teammate could guess what they mean)?
- Are functions short enough to fit on one screen?
- Are magic numbers / strings named as constants?

## 5. Testability

- Could this be unit-tested without heavy mocking?
- Are side effects (I/O, global state) isolated from pure logic?

## Output format

Return a single Markdown report with one section per category above, in the same order. End with a one-line **Summary** stating overall assessment (`Good`, `Needs work`, or `Blocking issues`).
```

> **Why the `description` matters.** Skills auto-trigger. Copilot compares your prompt to each skill's `description` and loads matching skills into context. A vague description like "helps with code" won't trigger reliably; a specific one like the above will. This is the most important part of a skill file.

## 3. Try it out

Start a fresh conversation (this time **without** a custom agent — we want to see the skill acting on its own). From your existing session, run `/new` to clear history, then `/agent` and pick the default agent so `code-reviewer` isn't active.

Ask a request that clearly matches the skill's `description`:

```
Do a code quality check on books.py.
```

## 4. What to look for

- Copilot's response should be **structured exactly like the checklist**: five numbered sections in order, ending with a one-line **Summary**.
- Categories with no issues should say "no issues found" — not be omitted.
- You can confirm the skill loaded by running `/skills` inside the session; `code-checklist` should be listed as available, and Copilot will typically mention which skills it used.

If the response ignores the checklist structure, your prompt probably didn't match the `description` closely enough. Try phrasing more like the description ("code review", "quality check", "pre-merge review") and rerun.

---

**Next module:** [03 — Combine the agent and the skill →](./03-combine-agent-and-skill.md)
