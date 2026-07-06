# Workshop: Customizing GitHub Copilot CLI with Agents + Skills

**Duration:** ~60 minutes • **Format:** hands-on lab • **Language:** your choice (Python, C#, or JavaScript)

By the end of this workshop you will be able to:

- Explain the difference between an **agent** and a **skill** in Copilot CLI.
- Create a custom agent that gives Copilot a specialist persona.
- Create a custom skill that gives Copilot a reusable playbook, and describe it so it triggers automatically.
- Combine an agent and a skill on one task and see how they compose.

> **Who is this for?** Developers who have installed GitHub Copilot CLI and used it at least once, but have not customized it yet. No AI/ML background is assumed.

---

## Prerequisites (before the clock starts)

- **GitHub Copilot CLI** installed and signed in. Verify with:
  ```bash
  copilot --version
  ```
  If this errors, install per the official docs (`gh extension install github/copilot-cli` or the standalone installer) and sign in before you begin.
- A terminal you are comfortable in (bash, zsh, or PowerShell — all commands below work in each, unless noted).
- One of: Python 3.10+, .NET 8+, or Node.js 18+. You will pick **one** in the Setup section.

---

## Section 1 — Setup (5 min)

### 1.1 Clone the workshop repo

```bash
git clone https://github.com/syash-git/agents-and-skills-workshop.git
cd agents-and-skills-workshop
```

You'll be working directly in this folder. The two folders Copilot looks in for custom agents and skills live right here alongside the starter files:

```
.github/agents/
.github/skills/
```

If they don't already exist in the clone, create them:

```bash
mkdir -p .github/agents
mkdir -p .github/skills
```

> **Jargon check.** *Agent* = a persona Copilot adopts (e.g., "act as a code reviewer"). *Skill* = a reusable playbook Copilot pulls in automatically when your prompt matches it (e.g., "run this quality checklist"). We will build one of each in this workshop.

### 1.2 Pick your language and paste the starter file

The **only** language-specific thing in this workshop is a small starter file we will ask Copilot to review. All three versions are equivalent: same domain (a tiny "book collection" module), same three deliberate issues:

1. No input validation on the title.
2. An overly broad `catch` / `except` that hides bugs.
3. A search function that ignores case inconsistently and has no early return.

**Pick one language and paste the matching file. Skip the other two.**

<details>
<summary><b>🐍 Python — save as <code>books.py</code></b></summary>

```python
books = []

def add_book(title, author):
    books.append({"title": title, "author": author})
    return True

def find_book(query):
    results = []
    try:
        for b in books:
            if query in b["title"]:
                results.append(b)
            if query.lower() in b["author"]:
                results.append(b)
    except:
        pass
    return results

def remove_book(title):
    for b in books:
        if b["title"] == title:
            books.remove(b)
    return True
```

</details>

<details>
<summary><b>🟦 C# — save as <code>Books.cs</code></b></summary>

```csharp
using System.Collections.Generic;

public class Book
{
    public string Title;
    public string Author;
}

public class BookCollection
{
    public List<Book> Books = new List<Book>();

    public bool AddBook(string title, string author)
    {
        Books.Add(new Book { Title = title, Author = author });
        return true;
    }

    public List<Book> FindBook(string query)
    {
        var results = new List<Book>();
        try
        {
            foreach (var b in Books)
            {
                if (b.Title.Contains(query)) results.Add(b);
                if (b.Author.ToLower().Contains(query.ToLower())) results.Add(b);
            }
        }
        catch { }
        return results;
    }

    public bool RemoveBook(string title)
    {
        foreach (var b in Books)
        {
            if (b.Title == title) Books.Remove(b);
        }
        return true;
    }
}
```

</details>

<details>
<summary><b>🟨 JavaScript — save as <code>books.js</code></b></summary>

```javascript
const books = [];

function addBook(title, author) {
  books.push({ title, author });
  return true;
}

function findBook(query) {
  const results = [];
  try {
    for (const b of books) {
      if (b.title.includes(query)) results.push(b);
      if (b.author.toLowerCase().includes(query)) results.push(b);
    }
  } catch (e) {}
  return results;
}

function removeBook(title) {
  for (const b of books) {
    if (b.title === title) books.splice(books.indexOf(b), 1);
  }
  return true;
}

module.exports = { addBook, findBook, removeBook };
```

</details>

You now have the workshop repo cloned, `.github/agents/` and `.github/skills/` folders ready, and one starter file. That's the whole setup.

---

## Section 2 — Concepts: Agents vs Skills (10 min)

### 2.1 The analogy: a specialist + a checklist

Imagine you're hiring help for a code review.

- You could hire a **generalist** who reviews code however they see fit. That's default Copilot.
- You could hire a **specialist reviewer** — someone whose job title tells Copilot *how to think*. That's an **agent**.
- You could hand that reviewer your team's **checklist** — a written playbook of exactly what to look for. That's a **skill**.

Now imagine giving the specialist your checklist. Same person, sharper output. That's what this workshop builds.

### 2.2 The mental model

| Thing | What it is | When Copilot uses it | Where it lives |
|---|---|---|---|
| **Agent** | A persona / role Copilot adopts | When you explicitly select or invoke it | `.github/agents/<name>.md` |
| **Skill** | A reusable playbook / instructions | **Automatically**, when your prompt matches the skill's `description` | `.github/skills/<name>/SKILL.md` |
| **Both together** | Persona + playbook | The agent runs; the skill's playbook is pulled in on top | Both folders |

Two key takeaways:

1. **You pick the agent. Copilot picks the skill.** Agents are opt-in per session; skills auto-trigger based on how well your prompt matches their `description`.
2. **They compose.** An agent is *who* Copilot is; a skill is *what procedure* it follows. Nothing stops both from applying at once.

### 2.3 Where the files go

Everything in this workshop lives in your project's `.github/` folder, which means:

- Anyone who clones your project gets the same agents and skills.
- You could also put the same files in your user-level Copilot config (`~/.copilot/agents/` and `~/.copilot/skills/`) to have them available in **every** project. We'll stay project-local today for clarity.

---

## Section 3 — Lab A: Build a `code-reviewer` agent (15 min)

### 3.1 What you're building

A persona that tells Copilot: *"You are a senior engineer doing a code review. Focus on real correctness issues. Be direct, not fluffy."* When we invoke this agent later, Copilot's tone and priorities will shift compared to the default.

### 3.2 Create the agent file

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

### 3.3 Try it out

First, get a **baseline** with the default agent so you have something to compare against. In the project folder:

```bash
copilot
```

Then at the prompt (adjust the filename to whichever starter you picked):

```
Review books.py and tell me what you think.
```

Read the response and note two or three things about it — probably a mix of style comments, general advice, and some correctness notes.

Now exit (`/exit`) and start a new session, this time selecting your custom agent:

```bash
copilot
```

At the prompt, use the `/agents` command to see the list, then pick `code-reviewer`. Ask the same question:

```
Review books.py and tell me what you think.
```

### 3.4 What to look for

Regardless of which language you chose, the `code-reviewer` response should feel noticeably different from the baseline:

- Findings are **grouped by severity** (Bugs / Risks / Improvements) instead of a flat list.
- The bare `catch`/`except` block is called out as a **Bug** or **Risk**, not brushed past.
- The missing input validation on `title` and the case-insensitivity inconsistency in the search function are flagged specifically.
- The tone is more direct — fewer hedges like "you might want to consider".

If the output is still generic, double-check that you actually selected `code-reviewer` in `/agents` (Copilot shows the active agent in the prompt).

---

## Section 4 — Lab B: Build a `code-checklist` skill (15 min)

### 4.1 What you're building

A **skill** — a written checklist Copilot will pull in automatically whenever your prompt looks like a code-quality request. Unlike an agent, you won't select it manually; a well-written `description` field does the matching.

### 4.2 Create the skill file

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

### 4.3 Try it out

Start a new Copilot session (this time **without** selecting a custom agent — we want to see the skill acting on its own):

```bash
copilot
```

Ask a request that clearly matches the skill's `description`:

```
Do a code quality check on books.py.
```

### 4.4 What to look for

- Copilot's response should be **structured exactly like the checklist**: five numbered sections in order, ending with a one-line **Summary**.
- Categories with no issues should say "no issues found" — not be omitted.
- You can confirm the skill loaded by running `/skills` inside the session; `code-checklist` should be listed as available, and Copilot will typically mention which skills it used.

If the response ignores the checklist structure, your prompt probably didn't match the `description` closely enough. Try phrasing more like the description ("code review", "quality check", "pre-merge review") and rerun.

---

## Section 5 — Lab C: Combine the agent and the skill (10 min)

### 5.1 The whole point of this workshop

So far you have:

- An **agent** (`code-reviewer`) that shapes *how Copilot thinks* — persona, tone, severity grouping.
- A **skill** (`code-checklist`) that gives Copilot *a specific procedure to follow* — the five-category checklist.

Now let's run both at once.

### 5.2 Try it

Start a new session and select the agent:

```bash
copilot
```

Use `/agents` and pick `code-reviewer`. Then ask:

```
Do a code review of books.py against our team quality checklist.
```

Notice the prompt: it says both "code review" (the agent's job) and "team quality checklist" (matches the skill's `description`). This gives Copilot every signal it needs.

### 5.3 What to look for

You should see the two customizations **compose**:

- The **structure** comes from the skill: five numbered categories in the fixed order, followed by a Summary line.
- The **voice** comes from the agent: direct, specific, no fluff, concrete suggested fixes.
- Findings inside each category are labeled with the agent's severity vocabulary (**Bug / Risk / Improvement**) even though the skill didn't define those.

That blend — a persona following a playbook — is exactly what agents + skills are for. Neither one alone would give you both.

### 5.4 Quick discussion prompts

Take two minutes to think about (or discuss with the person next to you):

1. If you kept just one of the two, which would you keep for daily code reviews? Why?
2. What would you have to write into a single monolithic prompt every time to get the same effect without agents or skills?
3. Whose review of your PRs at work could this replace — or, more realistically, augment?

---

## Section 6 — Wrap-up (5 min)

### 6.1 Key takeaways

- **Agent = persona.** You opt in with `/agents`. Changes *how* Copilot responds.
- **Skill = playbook.** Auto-triggers via its `description`. Changes *what procedure* Copilot follows.
- **They compose.** Selecting an agent doesn't disable skills — matching skills still load on top.
- The `description` field is the most important line in a skill file. Vague descriptions don't trigger.
- Project-local (`.github/` inside the repo) means your team gets the same behavior. User-level (`~/.copilot/`) means you get it in every project.

### 6.2 When do I reach for which?

| Situation | Reach for |
|---|---|
| I want Copilot to sound like a specific role (reviewer, tutor, ops engineer) | Agent |
| I want Copilot to follow the exact same procedure every time a certain kind of task comes up | Skill |
| A whole team should get the same behavior without thinking about it | Skill (auto-triggers) + commit both files to the repo |
| I want to swap between very different working modes in one session | Multiple agents |
| I want one canonical checklist that many prompts can pull in | One skill, precise `description` |

### 6.3 Next steps (pick one)

- **Share with your team.** Commit `.github/agents/` and `.github/skills/` in a real project. Everyone who clones gets your customizations for free.
- **Write a second skill.** A `commit-message` skill (matches "write a commit message" prompts) or a `pr-description` skill are great next ones. Keep the `description` specific.
- **Add MCP.** Once you're comfortable with agents and skills, look into MCP servers to give Copilot access to external tools (issue trackers, databases, docs) inside the same customization model.

You now have a working `code-reviewer` agent and a `code-checklist` skill in the workshop repo. Copy them into a real repo whenever you're ready.
