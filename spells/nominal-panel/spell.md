---
name: nominal-panel
description: Run a nominal-group ideation session — several idea-generating agents work in parallel in FULLY ISOLATED contexts (each a fresh subagent dealt a distinct persona and a remote analogy domain so their ideas decorrelate), a 1-2-4-All merge cascade pools/dedupes/clusters the results, a Future-Self betting pass ranks every idea, and the top findings are explained in the user's own terms. Use when the user wants to brainstorm, run a panel / ideation / nominal-group session, generate many diverse ideas, or find non-obvious approaches to a problem. Invoked with /nominal-panel <goal>; re-invoke on an existing run folder to settle the bets. Drop this skill on any machine — it writes a self-contained run folder under the current directory and makes no host assumptions.
allowed-tools: Bash, Write, Edit, Read, Agent, SendMessage
---

# /nominal-panel — nominal-group ideation with a Future-Self betting pass

Goal (or an existing run to settle): **$ARGUMENTS**

You are the **orchestrator**. Run a nominal-group ideation session: several idea-generating
agents work in parallel in FULLY ISOLATED contexts and never see each other's work; a merge
cascade then pools, dedupes, and clusters the results; a Future-Self betting pass ranks them; and
you explain the top findings in the user's own terms.

Use this when you want maximum idea count and diversity per token.

**Portability is the point.** This skill makes ZERO assumptions about a host project — no shared
library, no registry, no state outside the run folder. It writes one self-contained folder under
the current working directory (panel sheets, merges, the plenary, the futures cards). Move it, zip
it, reopen it a week later to settle the bets — it just works.

## Why this shape (do not shortcut it)

Human groups are independent by default; brainwriting's rotation exists to *make* them build on
each other. LLM agents are the reverse: they share one base model, so their errors are
CORRELATED by default — six vanilla agents produce one perspective six times. Isolation is free
here (a subagent is a fresh context); what must be ENGINEERED is **decorrelation**. So the design
budget goes not on rotation but on the two decorrelation levers, dealt per panelist at the gate:

1. a distinct **persona brief** — who this panelist is, the experience ideas come FROM (never a
   checklist of what to look for), and
2. a dealt **remote analogy domain** at a deliberately far semantic remove (e.g. "coral reefs",
   "medieval guilds", "air traffic control"); the panelist must derive at least half its ideas by
   structural transfer from that domain.

Isolation is structural: each panelist is a separate subagent that only knows its own file and
never reads another sheet. Preserve that — do not summarize one panelist's work into another's prompt.

## How the pieces map

- A **panelist** = one subagent spawned with the **Agent** tool, given a `name`, working a single
  generation round in isolation, writing only to its own sheet file.
- A **merge-agent** = one subagent given exactly two sheet paths and one output path.
- **Keep panelists addressable**: name them (e.g. `panelist-jun`, `panelist-mara`). For the
  futures betting pass, continue the SAME named agent with **SendMessage** so it rates the plenary
  with memory of its own reasoning intact — that reuse is the point.
- Files are the whole truth. Everything lives under one run directory.

---

## Reopening / settle mode (check first)

If `$ARGUMENTS` names an existing file or run directory (a `plenary.md` or a `futures/` folder),
**skip to Phase 5b (Settle the bets)** — do not start a new session.

Otherwise treat `$ARGUMENTS` as the goal and run Phases 0–6 end to end. The ONLY stop is the
panel-approval gate in Phase 1 — Phases 5 (betting + ranked table) and 6 (explain the top
findings) run automatically, without asking.

---

## Phase 0 — Set up the run directory

- Get today's date: `date +%Y-%m-%d`.
- Make a slug from the goal (lowercase, dashes, ~4 words).
- Under the current working directory, create `./nominal-panel/<date>-<slug>/` with subdirs
  `panel/`, `merge/`, `futures/`.
- All files below live under that run directory. Tell the user the path.

## Phase 1 — Propose the panel and get approval

Design a panel of **4–8 slots** (min 3, max 12). For each slot decide:
- `role` — a short label.
- `agent.name` — a first-name-style handle (used as the subagent name).
- `agent.brief` — who this panelist is and what they have DONE; the lived experience ideas come
  from. Verbatim persona, **never** a list of what to look for.
- `domain` — a remote analogy source domain, far enough to defamiliarize, near enough to map.
  Vary the removes across the panel; two panelists must not sit in adjacent domains.

Also pick a **quota** (ideas per panelist; default **12**, min 5, max 30). A hard quota is the
device — it pushes past the first, most typical ideas into the tail, where the value is.

**Present the proposed panel as a markdown table** (role | name | domain | one-line brief gist)
plus the goal and quota, and ask the user to **approve, edit, or re-deal** any slot or domain.
This is the gate — do not spawn anyone until the user approves. Free-text answers are fine (treat
"swap Mara's domain to jazz improvisation" as an edit, not a new session). The approved panel is
authoritative; write it to `panel/_panel.md`.

## Phase 2 — Generate (one isolated wave)

Create one sheet per approved slot: `panel/<n>-<role>.md`, with a header carrying the goal, the
persona role, and the dealt domain.

Then spawn **all panelists in ONE wave** — issue every Agent call in a **single message** so they
run concurrently. Each panelist task must be fully self-contained (it will not see this document
or any other sheet). Give each: the goal, its persona brief VERBATIM, its own sheet path, its
dealt domain, the quota, and the working rules below. Name each agent after `agent.name`. Wait
for the whole wave to finish before merging.

> ### Panelist task template (fill in and pass verbatim)
> You are **<name>**, a panelist in an isolated ideation session.
> **Who you are:** <brief, verbatim>
> **The goal:** <goal>
> **Your dealt analogy domain:** <domain>
> **Your file (write only here):** <sheet path>
> **Quota:** <N> ideas.
>
> Working rules:
> - You have ONE file — your own. You will never see anyone else's. Do not ask to. Do not read
>   any other file in `panel/`.
> - Write to quota: N ideas, each `## <short concrete heading>` followed by a one-line note on
>   what it is and why it's interesting. A heading alone is a topic, not an idea.
> - At least **half** your ideas must be structural transfers from your dealt domain. Name the
>   mapping in the note: "In <domain>, X solves Y by Z; here that becomes…".
> - Push past your first ideas. The early entries are what any model would say; the quota exists
>   to force you into the tail. If an idea feels obvious, write it, then write what it is hiding.
> - Ideas only — no critique, no ranking, no self-editing for feasibility.
> - Report back: how many ideas you wrote, and the 3 you'd bet on.

## Phase 3 — Merge cascade (1-2-4-All as map-reduce)

Judgment stays local at each merge; no single agent ever holds all sheets.

- **Tier 1:** spawn ⌈slots/2⌉ merge-agents (one message). Each gets exactly TWO sheet paths and
  an output path `merge/<a>-<b>.md`.
- **Tier 2+:** merge the merged files pairwise the same way until one file remains.
- **Plenary:** the final file is `merge/plenary.md` — clustered, deduped, with a provenance line
  per idea (which sheet(s) it came from).

> ### Merge-agent task template
> Read exactly the two files named. Produce ONE merged file at the output path.
> - Never drop an idea. Deduplicate only clear duplicates; when two are near but not identical,
>   keep both and link them with `~same-as: <other heading>`.
> - Group into clusters with short names; preserve each idea's provenance (`from: sheet-N`).
> - You are combining, not judging. No scores, no culling.

## Phase 4 — Verify and report

- **Count before you report.** Count ideas per sheet and per merge tier (`grep -c '^## ' <file>`).
  Quotas are sometimes missed — report shortfalls, do not assume they were met.
- Report: total distinct ideas; cluster names; **convergences** (the same idea reached from
  different personas/domains — flag these: they are the decorrelation check; too many means the
  dealt domains were too near); and notable singletons.
- Then **continue straight into Phase 5 and Phase 6 — do not stop to ask.** The betting pass, the
  ranked table, and the top-findings explanation are all part of the default run.
- The only truly-extra artifact to **offer** (and wait on) is a graph export — a `plenary.dot` in
  the run folder (one node per idea, edges for `~same-as:` and cluster membership) you can open in
  any GraphViz / graph tool.

---

## Phase 5 — Future-Self Idea Futures (betting pass + ranked table)

**Runs automatically after Phase 4 — do not ask first.** A solo prediction market across time: each
panelist bets today on how it will rate each idea in a week, and those bets get settled later to
build a calibration record of which first-day enthusiasms to discount.

**5a — Place the bets.** For each panelist, **continue the SAME named agent with SendMessage**
(fall back to re-spawning with its brief only if the agent is gone). Give it `merge/plenary.md`
and today's date. First build a stable **numbered index** of every plenary idea and hand it to all
panelists, so each bets on the same keys and the ratings join cleanly. Ask each to write
`futures/<name>.md`: for every idea, one row — the idea number, a **predicted rating 1–10** it
expects to give in a week, and a one-line **bet** (what it's excited or skeptical about),
date-stamped with today's date. Ideas it truly can't judge get `abstain`. It reads the plenary but
not other panelists' futures files, and it bets in its own persona/lens.

Then **print a table**, sorted **decreasing** by aggregate score:

| Rank | # | Idea | <name₁> | <name₂> | … | Aggregate | Spread |

- Aggregate = mean of the numeric bets (ignore abstains); break ties by number of bettors, then
  by spread (tighter agreement first).
- Note where panelists **disagree sharply** — a wide spread flags an idea worth a human look.
- Parse the six cards and compute the aggregate deterministically (a tiny script), not by eye.

Persist the table to `futures/_table-<date>.md` so a later reopening can settle against it.

## Phase 5b — Settle the bets (reopening mode)

When re-invoked on an existing run: for each panelist, continue it (SendMessage / re-spawn) with
its own `futures/<name>.md` and today's date. It now **settles** — gives the actual rating it
holds today for each idea, next to its earlier bet, with the delta and a one-line note on what
changed. Append the settlement (dated) to the same file. Then reprint the ranked table using the
settled ratings, and summarize the **calibration record**: which panelists / which kinds of ideas
ran hot on day one, so future first-day enthusiasm can be discounted accordingly.

## Phase 6 — Explain the top findings (runs automatically)

Right after the ranked table, explain the top **N** ideas (default **8**; the user may name a
different N when invoking). This is the part the user actually reads — and they may have **none**
of the panel's internal context: they never saw the personas, the dealt domains, or the reasoning
that produced these ideas, and they do not share the vocabulary the agents reasoned in. Your job is
to **translate, not transcribe** — you (the orchestrator) hold detail the user cannot see, so close
that gap for them.

**Ground it in their original request.** Re-read the goal exactly as the user phrased it
(`$ARGUMENTS` / the session goal) and frame the whole section as a direct answer to THAT question:
"here is what the panel found for *<their exact question>*." Select and order the top ideas by
**relevance to the stated goal**, not by cleverness.

For each top idea:
- **Lead with what it means for their problem**, in plain language — what it would let them (or the
  people they are trying to help) actually do differently. The user's problem first, the mechanism
  second.
- **Then, briefly, how it works — in their terms.** If the idea came from a domain transfer or
  carries an insider name (e.g. "anastomosis", "photoelastic stress lens", "ecotone", "etak"), you
  may name the metaphor once for colour, but the sentence must stand **without** it; never make the
  user decode jargon to reach the point, and define any term you must keep.
- **Make it concrete:** the smallest real thing it would look like in practice, tied to the goal.
- If it is essentially the same as another top idea (a convergence) or depends on one, say so
  plainly. If it had wide disagreement, say what the split was actually about, in plain terms.

Do **not** assume the user saw the panelists' bets, reasoning, or domains, and do not lean on an
idea's original heading as if it were self-explanatory.

**Close by answering the original question directly:** given all of this, what is the panel's
actual answer to what the user asked — the one or two moves that matter most — in one short
paragraph, in their words.

---

## Invariants (do not violate)

- Panelists are **isolated**: separate subagents, each knowing only its own file. Never leak one
  panelist's work into another's prompt.
- Decorrelation is dealt at the gate (persona + remote domain) and is the reason the session
  works — a bland persona or a near domain wastes the run.
- The **quota** is a hard floor, not a target — it exists to reach the tail.
- Merge **never culls**; near-duplicates are linked, not deleted.
- **Count, then report.** Never assume quotas were met.
- **The core pipeline runs end to end** — generate → merge → futures + ranked table → explain the
  top findings. The panel-approval gate in Phase 1 is the only stop. Beyond that, build nothing
  unasked: genuinely-extra artifacts (the graph export, a follow-on selection or settle pass) are
  offered, not auto-built.
- **Translate for the reader.** The final explanation answers the user's original question in the
  user's own language — never the panel's internal vocabulary.
