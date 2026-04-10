# process-book

Transform a raw book text file into a multi-lens semantic-zoom exploration — a set of
interactive HTML pages that let you dive deep into any book from multiple angles.

## Motivation

Reading a book linearly is just one way to experience it. What if you could explore a
novel through its network of characters? Trace the emotional arc across chapters? See the
geographic journey on a map? Navigate the philosophical arguments as a tree?

This spell takes a plain text file of any book and generates a self-contained folder of
interactive HTML pages — each one a different "lens" on the same text. A landing page ties
them all together. The result is portable: zip it, host it, email it — it just works.

## What you get

```
./<book-id>/
  index.html              <- landing page with book intro and lens cards
  lenses/
    narrative/index.html   <- chronological narrative structure
    <lens-id>/index.html   <- one page per selected lens
    ...
```

Each lens supports **semantic zoom** — start with a high-level overview and drill down
through 3-4 layers of increasing detail, all the way to actual quotes and close analysis.

The assistant reads the entire book, suggests 5-8 lenses tailored to that specific book
(a detective novel gets completely different lenses than a philosophy text), and lets you
pick which ones to build. Lenses are built in parallel for speed.

## Usage

```
/process-book path/to/book.txt
```

## Examples of generated lenses

- **Narrative Structure** — chronological reading with chapter-by-chapter zoom
- **Character Constellation** — force graph of character relationships
- **Emotional Landscape** — sentiment arc with drillable peaks and valleys
- **Geographic Journey** — map-based exploration of locations in the text
- **Thematic Architecture** — hierarchical view of interwoven themes
- **Philosophical Arguments** — tree of claims, evidence, and counterpoints

## Install

```bash
mkdir -p ~/.claude/skills/process-book && \
  curl -sL https://raw.githubusercontent.com/anvaka/spell-book/main/spells/process-book/spell.md \
  -o ~/.claude/skills/process-book/SKILL.md
```
