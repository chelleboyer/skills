---
name: second-brain-fix
description: Work through the findings from a second-brain audit in batches, correcting stale facts and converting locations onto the state/event schema, driven by the second-brain-audit.md file the audit left behind. Use after running second-brain-audit, when someone wants to fix the rest of their notes rather than one page, when they ask how to take the audit forward, when they have a list of contradicted or unsupported claims to work through, or when a notes folder needs converting in bulk rather than one page at a time.
argument-hint: "[path-to-second-brain-audit.md-or-notes-folder]"
arguments: [ledger]
---

# Second Brain Fix

The audit found what is wrong and fixed one location so the shape was visible. This works
through the rest, in batches, from the file the audit left behind.

## The argument

The path this run was invoked with is `$ledger`, and it is optional:

```
/second-brain-fix                                   nothing passed
/second-brain-fix ~/notes                           a folder
/second-brain-fix ~/notes/second-brain-audit.md     the file itself
```

Resolve it before anything else:

| What you were given | Do |
|---|---|
| a path to a **file** | that is the ledger. The notes folder is its parent, unless the ledger names a different one |
| a path to a **folder** | the ledger is `second-brain-audit.md` inside it |
| nothing, so the line above still reads `\$ledger` | look in the current folder, then one level down |
| **a path that does not exist** | say so and stop. Do not fall back to searching, or a typo silently works on the wrong folder |
| **more than one match** when searching | list them with their newest `## Log` dates and ask which |
| **no match** when searching | stop. Say to run `/second-brain-audit` first, since there is nothing to work from |

Never invent a queue from scratch when the ledger is missing. Auditing and fixing in one
pass is how a bulk write happens against findings nobody read.

**The ledger is the input.** `second-brain-audit.md` is the queue and the record:
one keyed line per location in `## Current State`, one dated line per run in `## Log`.
Everything below is driven by that file, and a run that does not update it is a run nobody
can pick up from.

## This skill writes in bulk. Make it recoverable first.

Before touching a single file:

- **Notes in git?** `git checkout -b second-brain-fix`. Commit after every batch.
- **Not in git?** Copy the whole folder somewhere else and say you did.

Do not skip this and do not offer to skip it. The audit fixed one page behind a diff the user
approved. This one changes many, and "undo" has to mean something.

## The three piles are three different jobs

| Pile | What the finding contains | What to do |
|---|---|---|
| **Contradicted** | the stale claim AND the newer evidence | fix in batch, no questions |
| **Unsupported** | the claim, and nothing backing it up | never guess. One batch of questions for the user |
| **Locations not yet converted** | a page that has no `## Current State` / `## Log` | convert in batch, verbatim |

The middle row is where bulk fixing goes wrong. An unsupported claim has no answer in the
notes by definition, so an agent told to "correct these" will invent a value or quietly delete
the line. Both are worse than the stale claim.

## How much to take on

Ask how large the notes are, or count the markdown files, and scale the ambition:

| Size | Do this |
|---|---|
| **Under ~50 files** | everything in one session. Two or three batches |
| **~50 to 500** | the always-loaded surface first, then one folder per batch, committing between |
| **500+** | the always-loaded surface, plus only the pages with a trail (several entries about one subject over time). Everything else stays as it is, permanently |

Bigger notes do not mean convert more. They mean convert a smaller fraction and lean harder on
the write path, because at that size nobody is ever going to hand-tend the archive.

## Step 1: build the batch list

Read `second-brain-audit.md`. Take the `## Current State` entries that are not marked as
fixed, plus any pile the audit reported but did not enumerate. Group them into batches of
roughly 5 to 15 locations, keeping a folder together where you can.

Show the user the batch list and the order before writing anything. Ordering is always:

1. Whatever loads every session.
2. The pages the agent gets wrong most often.
3. Pages with a trail.
4. Nothing else.

## Step 2: the contradicted pile, in batch

For each finding, the newer evidence is already named. So:

- Replace the stale line in `## Current State`.
- Move the superseded line, **verbatim**, to `## Log`.
- Date the new line with the date of the evidence, not today.

That last one matters more than it looks. A date on a current value is a claim that somebody
checked the value on that day. Stamping today's date on a line you corrected from a note
written in June makes a June fact look verified this morning.

Work the whole batch, then show one summary: how many lines replaced, in which files, and the
three or four that were least obvious. Do not show a diff per line, and do not ask per line.

## Step 3: convert the locations, in batch

Give each page the two sections. Adapt to the shape the audit found:

```markdown
## Current State
<!-- One entry per subject. Dated. REPLACED on update, never appended to. -->

- **Retainer** (2026-08-01): $3,200/mo, renewed through February 2027
- **Main contact** (2026-05-02): Curtis Ilo

## Log
<!-- Append-only. Never edit or delete an entry. -->

- (2026-04-30) Delivered and paid, $21,000
- (2026-06-15) Added reply drafting, retainer to $3,200/mo
```

One big file gets a `## Current State` block at the top and everything else beneath it, no new
files. Daily notes get one new file of current values and the journal untouched. Notes that are
not markdown do not get converted at all.

**This is sorting, not rewriting.** Every existing line lands in one of the two sections,
verbatim, at most with a date prepended. Improving the prose is how information disappears
without anyone noticing, and in a batch nobody is reading closely enough to catch it.

Two kinds of page to leave alone, and say so rather than converting them:

- **Reference checklists.** Packing lists, hospital-bag lists, standard operating steps. Few
  bolded keys, few dates, and the order is the content. Converting one passes every structural
  check and destroys the thing that made it useful.
- **Pages where the freshest status lives inside prose**, under a heading that owns the bullets
  below it. Dissolving that section leaves the newest status outside `## Current State` and the
  oldest inside it, which is the exact failure being removed, rebuilt one level up.

## Step 4: the unsupported pile, one pass of questions

Collect them all and ask once, as a numbered list. Not one at a time, and never silently.

For each, three outcomes:

- **Still true** goes into `## Current State` with the date the user gives, and say plainly that
  the evidence was missing, not just misfiled.
- **No longer true** gets replaced, old line verbatim to `## Log`.
- **Cannot tell** comes out of the always-loaded file entirely. A confident wrong answer costs
  more than a missing one.

If the user does not answer, leave every one of them exactly as it is. An unanswered question is
not permission.

## Step 5: update the ledger

After each batch, edit `second-brain-audit.md`:

- **Replace** each location's `## Current State` line with its new status. One line per
  location, always. Never a second line.
- **Append** one entry to `## Log`: the date, which batch, how many lines replaced, how many
  locations converted, and how many unsupported claims are still unanswered.

Then commit the batch. The ledger and the notes move together, so an interrupted run is
resumable by reading one file.

## Step 6: re-audit and compare

When the batches are done, run `/second-brain-audit` again and compare the contradicted count
to the last `## Log` line. That number is the only evidence any of this worked.

If the count did not move much, say so and say why rather than presenting the conversion as a
result. Restructuring cannot reach a fact nobody ever wrote down, and when the count stays flat
that is usually what happened. The fix then is the write path, not another batch.

## Rules that never bend

1. **Lose nothing.** Every line lands somewhere, verbatim. If a line cannot be placed, leave it
   where it is and report it.
2. **Never merge two subjects that look alike.** "Acme (May)" and "Acme Corp renewal" may be two
   real things. A duplicate entry is a cheap mistake; a wrong merge destroys information. Report
   near-misses at the end of the batch and let the user decide. Batch work merges by default
   because merging looks like tidying, so this rule needs holding on purpose.
3. **Never invent a value**, and never delete a claim to make a pile smaller.
4. **Never edit or delete a `## Log` entry.** Old and superseded is the point of that section.
5. **A date is a claim that the value was checked.** A line you only moved keeps its own date.
6. **Stop when the always-loaded surface has nothing contradicted.** An archive full of old
   pages is history, not rot. There is no version of this where every page gets converted.

## If the write path has not changed yet

Check whether `CLAUDE.md` (or `AGENTS.md`, or the system prompt) carries the state/event rule
from the audit's phase 7. If it does not, add it before starting, and say why: fixing four
hundred lines under a write path that can only append buys about a month.
