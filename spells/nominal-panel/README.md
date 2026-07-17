# nominal-panel

Run a nominal-group ideation session where several idea-generating agents work **in parallel, in
fully isolated contexts**, then a merge cascade pools their ideas and a Future-Self betting pass
ranks them — ending in a plain-language explanation of the strongest findings.

## Motivation

If you just ask one assistant to "brainstorm ideas", you get one perspective — fluent, plausible,
and narrow. Spawn six copies of it and you get the *same* perspective six times, because they all
share one base model: their errors are **correlated**.

Human brainstorming groups have the opposite problem. People are independent by default, so
techniques like brainwriting add *rotation* to make them build on each other.

This spell inverts that. Isolation is free for agents (a fresh subagent is a clean context), so it
spends its whole design budget on the thing that is actually scarce: **decorrelation**. Every
panelist is dealt

- a distinct **persona** — a lived experience the ideas come *from* (a patent examiner, a jazz
  bandleader, a field ecologist…), and
- a **remote analogy domain** at a deliberately far semantic remove (mycelial networks, air
  traffic control, metro maps…) that it must transfer at least half its ideas from.

Then no panelist ever sees another's sheet. You get genuinely different ideas — and when two far
personas independently reach the same idea, that convergence is a strong signal rather than an echo.

## What you get

A self-contained run folder under your current directory:

```
./nominal-panel/<date>-<slug>/
  panel/
    _panel.md              <- the approved panel (roles, personas, dealt domains)
    <n>-<role>.md          <- one raw idea sheet per panelist (never shared between them)
  merge/
    <a>-<b>.md             <- pairwise merges (1-2-4-All cascade)
    plenary.md             <- the pooled, deduped, clustered idea set with provenance
  futures/
    <name>.md              <- each panelist's dated 1–10 bets on every pooled idea
    _table-<date>.md       <- the ranked table, sorted by aggregate bet
```

The run is the artifact. Reopen it a week later (`/nominal-panel <that folder>`) and the same
panelists **settle** their bets — building a calibration record of which first-day enthusiasms to
discount.

## How it runs

1. **Propose a panel** — the assistant designs 4–8 personas, each with a dealt remote domain, and
   shows it to you to approve, edit, or re-deal. *(This is the only stop.)*
2. **Generate** — all panelists run at once, each in isolation, writing its own sheet to a hard
   quota (default 12 ideas) with at least half transferred from its dealt domain.
3. **Merge** — a 1-2-4-All cascade pools and clusters every idea without dropping any; near-
   duplicates are linked, not deleted.
4. **Future-Self betting** — each panelist rates all pooled ideas 1–10 (a prediction of how it'll
   rate them in a week), and a ranked table is printed, flagging where the panel disagrees.
5. **Explain the top findings** — in your own terms, grounded in your original question, translated
   out of the agents' internal jargon.

Steps 4 and 5 run automatically after the panel is approved.

## Usage

```
/nominal-panel <the goal or question to ideate on>
```

Re-run on an existing run folder to settle the bets:

```
/nominal-panel ./nominal-panel/2026-07-16-my-question
```

## Install

```bash
mkdir -p ~/.claude/skills/nominal-panel && \
  curl -sL https://raw.githubusercontent.com/anvaka/spell-book/main/spells/nominal-panel/spell.md \
  -o ~/.claude/skills/nominal-panel/SKILL.md
```

After installation, invoke with `/nominal-panel <goal>` in Claude Code.
