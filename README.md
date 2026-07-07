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

## Setup (5 min)

### Clone the workshop repo

```bash
git clone https://github.com/syash-git/agents-and-skills-workshop.git
cd agents-and-skills-workshop
```

You'll be working directly in this folder. Copilot looks for custom agents and skills in two folders that **do not exist yet** in the clone — you'll create them now:

```bash
mkdir -p .github/agents
mkdir -p .github/skills
```

### Pick your language and paste the starter file

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

## Concepts: Agents vs Skills (10 min)

### The analogy: a specialist + a checklist

Imagine you're hiring help for a code review.

- You could hire a **generalist** who reviews code however they see fit. That's default Copilot.
- You could hire a **specialist reviewer** — someone whose job title tells Copilot *how to think*. That's an **agent**.
- You could hand that reviewer your team's **checklist** — a written playbook of exactly what to look for. That's a **skill**.

Now imagine giving the specialist your checklist. Same person, sharper output. That's what this workshop builds.

### The mental model

| Thing | What it is | When Copilot uses it | Where it lives |
|---|---|---|---|
| **Agent** | A persona / role Copilot adopts | When you explicitly select or invoke it | `.github/agents/<name>.md` |
| **Skill** | A reusable playbook / instructions | **Automatically**, when your prompt matches the skill's `description` | `.github/skills/<name>/SKILL.md` |
| **Both together** | Persona + playbook | The agent runs; the skill's playbook is pulled in on top | Both folders |

Two key takeaways:

1. **You pick the agent. Copilot picks the skill.** Agents are opt-in per session; skills auto-trigger based on how well your prompt matches their `description`.
2. **They compose.** An agent is *who* Copilot is; a skill is *what procedure* it follows. Nothing stops both from applying at once.

### Where the files go

Everything in this workshop lives in your project's `.github/` folder, which means:

- Anyone who clones your project gets the same agents and skills.
- You could also put the same files in your user-level Copilot config (`~/.copilot/agents/` and `~/.copilot/skills/`) to have them available in **every** project. We'll stay project-local today for clarity.

---


---

## Workshop modules

The hands-on labs live in the [`modules/`](./modules) folder. Work through them in order for the full workshop, or jump straight to the topic you want.

| # | Module | Time | What you'll build |
|---|---|---|---|
| 1 | [Build a custom agent](./modules/01-custom-agent.md) | 15 min | A `code-reviewer` agent — a persona that changes *how* Copilot responds. |
| 2 | [Build a custom skill](./modules/02-custom-skill.md) | 15 min | A `code-checklist` skill — a playbook Copilot auto-triggers on quality prompts. |
| 3 | [Combine agent + skill](./modules/03-combine-agent-and-skill.md) | 10 min | Run both on one task and watch them compose. |
| 4 | [Use the GitHub MCP server](./modules/04-mcp-server.md) | 10 min | Reach real GitHub from Copilot CLI: list issues, fix one, open a PR. |

> **Recommended order:** 1 → 2 → 3 → 4. Modules 1 and 2 are independent; Module 3 needs both; Module 4 is standalone but is best after 1–3 so you can see MCP compose with an agent and a skill.
