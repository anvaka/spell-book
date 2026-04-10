---
name: process-book
description: Transform a raw book text file into a multi-lens semantic-zoom exploration — a fresh, fully self-contained folder of interactive HTML pages, one per lens, plus an LLM-generated landing page. Use when the user wants to "process a book", "explore a book through lenses", or "build a semantic zoom for a book". Invoked with /process-book <path-to-book.txt>. Drop this skill on any machine and it works — no host project, no shared library, no external state.
allowed-tools: Bash, Write, Edit, Read, Glob, Agent
---

# /process-book — Multi-lens semantic zoom for any book

Transform a raw book text file into a self-contained, multi-lens exploration. Each lens becomes its own interactive HTML page; a landing page is generated fresh by the LLM (not from a template) and ties them together.

**Portability is the whole point.** This skill makes ZERO assumptions about a host project. No `books.json`, no parent `index.html`, no shared library, no registry, no relative paths to anything outside the output folder. You can clone this skill onto a fresh laptop, point it at a `.txt` file, and get a complete, portable artifact you can zip up and host anywhere.

## Input

`$ARGUMENTS` is the path to a text file containing a book. If empty, ask the user for the path.

## Output layout

Everything is written under the **current working directory** in a single fresh folder named after the book:

```
./<book-id>/
  index.html                  <- landing page, generated from scratch by an LLM each run
  lenses/
    <lens-id>/index.html      <- one self-contained page per lens
    ...
```

That folder is the entire artifact. It contains no references to anything outside itself. Move it, zip it, host it on GitHub Pages — it just works.

## Step 1: Analyze the book (reader agent)

**Do NOT read the book yourself.** Spawn a single reader agent via the Agent tool to keep the main conversation context lean.

Reader agent prompt:

---

You are a literary analyst. Read the entire book and suggest exploration lenses.

**Book file path:** `{file_path}`

Read the whole file using the Read tool with `limit: 50000`, continuing with `offset` increments until you hit EOF. Then output a JSON block with this exact structure (and nothing else):

```json
{
  "title": "...",
  "author": "...",
  "language": "...",
  "summary": "One paragraph summary of the book",
  "lenses": [
    {
      "id": "kebab-case-id",
      "name": "Short Name (2-4 words)",
      "description": "One sentence: what this lens reveals",
      "visualization": "What interactive approach to use (map, force graph, timeline, zoom-tree, etc.) and why",
      "depthLevels": "Describe 3–4 layers of progressively deeper detail specific to this lens, from high-level overview down to close reading",
      "rationale": "Why this lens is particularly revealing for THIS book"
    }
  ]
}
```

### Rules for lenses:
- Suggest 5-8 lenses
- Every lens MUST support **progressive exploration** — at least three layers of depth, from high-level overview down to close reading of the actual text. The form is wide open — any interactive approach that lets users drill into what interests them
- Lenses must be **specific to THIS book** — a detective novel gets completely different lenses than philosophy or history
- Always include one **"Narrative Structure"** lens (id: `narrative`) for chronological reading
- The other lenses should reveal something non-obvious and interesting
- Think creatively — what visualization would genuinely serve each lens best?

Output ONLY the JSON block, nothing else.

---

## Step 2: Present lenses to the user

Parse the reader agent's JSON. Show the user a numbered list with each lens's name, description, and proposed visualization. Ask:

> "Which lenses would you like me to build? (e.g. `1,3,5` or `all`)"

**Wait for the user's response before proceeding.**

## Step 3: Set up the output directory

1. Generate a URL-safe `book-id` from the title: kebab-case, lowercase, max 40 chars, ASCII only — transliterate non-Latin characters if needed.
2. Resolve the absolute path of `./<book-id>/` (under the current working directory). Call this `<out>`.
3. If `<out>` already exists, ask the user whether to overwrite, pick a different ID, or abort.
4. Create `<out>/lenses/`.

## Step 4: Build each lens (parallel builder agents)

For each selected lens, spawn a builder agent via the Agent tool. **Spawn all builder agents in a single message** so they run in parallel. Run them in the background.

Each builder agent writes one file: `<out>/lenses/<lens-id>/index.html`. From there, the back link to the landing page is always exactly `../../index.html` — both files live in the same self-contained `<out>` folder.

Builder agent prompt template (fill in the `{variables}`):

---

You are creating an interactive web page that lets someone explore a book through one specific lens.

**Book file path:** `{file_path}`
**Book title:** "{title}"
**Book author:** "{author}"
**Book language:** {language}

**Lens id:** `{lens_id}`
**Lens name:** "{lens_name}"
**Lens description:** {lens_description}
**Suggested visualization approach:** {visualization_approach}
**Depth levels:** {depth_levels}

**Output file (absolute path):** `{out}/lenses/{lens_id}/index.html`
**Back link to the book's landing page (relative from your output file):** `../../index.html`
  ^ This points to a sibling landing page that another agent is generating in the same self-contained folder. Do NOT link to anything outside this folder.

### Your task

1. Read the ENTIRE book text from `{file_path}` using the Read tool with `limit: 50000`. If the file is longer, continue reading with `offset` increments until EOF.
2. Create the directory if needed, then write the single self-contained `index.html` at the absolute output path above.

### Design principles

- **Semantic zoom is the core metaphor.** The user should start at a high-level overview and progressively drill down to specific details and ultimately to the actual text or close analysis. How you implement this zoom is up to you. All user-facing labels should describe what the user will see.
- **Fully self-contained** — a single HTML file with embedded CSS and JS. You may load libraries from CDNs (unpkg, cdnjs, jsdelivr). No references to any file outside `<out>/`.
- **Visually polished.** Use a warm, bookish aesthetic — serif fonts for content (e.g. `Georgia`, `Lora`, `Crimson Text`), clean sans-serif for UI.
- **Include the back link** to the landing page at the top of the page using the relative path provided above.
- **Write all content in the same language as the book.**
- **Be substantive.** Real analysis, real quotes from the text, real insights. This is not a summary — it's a tool for deep exploration.
- **Works offline** once loaded (except CDN libraries and any map tiles).

### What to create

Use your creative judgment based on the lens. Don't feel constrained — invent something new if it fits the book better.

### Output

Write a single file at `{out}/lenses/{lens_id}/index.html`. Create any needed directories first. Do not write any other files anywhere.

---

## Step 5: Generate the landing page (from scratch, not a template)

After all builder agents finish, spawn ONE more agent — the **landing-page agent**. This is what makes the skill independent of any host project: the landing page is **generated fresh by an LLM every run**, not copied from a template, not patched into a shared registry.

Only include lenses whose builder agent actually produced a file (verify with Glob first).

Landing-page agent prompt:

---

You are designing the landing page for a single-book semantic-zoom exploration. The landing page is the only entry point — there is no parent site, no library, no surrounding project. Everything must live inside one self-contained folder.

**Book title:** "{title}"
**Book author:** "{author}"
**Book language:** {language}
**Book summary:** {summary}

**Built lenses** (each lives at `./lenses/<id>/index.html` relative to your output file):
```json
{lenses_json}
```
(each entry has `id`, `name`, `description`, `visualization`, `depthLevels`, `rationale`)

**Output file (absolute path):** `{out}/index.html`

### Your task

Create a single self-contained `index.html` at the absolute output path above that serves as the landing page for this book's exploration. It must:

- Introduce the book (title, author, summary) with a warm, bookish aesthetic
- Present each lens as a card / tile / entry that links to `./lenses/<id>/index.html`
- Convey what each lens reveals and the kinds of exploration it enables
- Be **fully self-contained** — embedded CSS and JS only, optional CDN libs allowed. No references to any file outside this folder.
- Be written in the **same language as the book**
- Be visually distinct and creative — do NOT default to a generic "card grid". Think about what landing page would feel right for THIS specific book. A poetry collection deserves a different landing page than a treatise on economics.
- Work offline once loaded

### Output

Write a single file at `{out}/index.html`. Do not write any other files. Do not write a `meta.json`, `books.json`, or any registry — this exploration is standalone.

---

## Step 6: Verify and report

After the landing-page agent finishes:

1. List all generated files under `<out>/` using Glob.
2. Report to the user:
   - Book title and author
   - Number of lenses built and their names
   - Path to the landing page (`<out>/index.html`)
   - Suggested next step:
     > "Open `<out>/index.html` in a browser to explore your book."

## Guidelines

- **No host-project assumptions, ever.** Never reference `books.json`, a parent `index.html`, a `books/` directory, or any path outside `<out>/`. The output folder is the entire universe of the artifact.
- **Parallelism matters.** Builder agents must be spawned in a single message so they run concurrently. The landing-page agent runs *after* builders finish — it needs to know which lenses succeeded.
- **Context discipline.** The main agent should never read the full book itself — only the reader agent and builder agents do. This keeps the main conversation small enough to coordinate many parallel builders.
- **Failure handling.** If a builder agent fails or produces no file, omit that lens from the landing page and tell the user which lenses didn't build. Do not block the rest of the output on one failed lens.
- **Language fidelity.** Every page (lenses + landing) must be written in the book's language, not the user's interface language.
- **Portability test.** After generation, the user should be able to `mv ./<book-id> ~/Desktop/` and have it still work, or zip it and email it to someone. If anything you generate would break under that test, you've leaked a host-project assumption — fix it.
