# Task card: <title> (<date>, log ID <id>)

## Goal
What to produce, and what "done" means in one sentence.

## Frozen contract
- Input paths:
- Output paths:
- Function signatures / file formats that must not change:

## Red lines
- No full-file rewrites.
- No changes unrelated to this card.
- No upgrading packages to make an error go away.
- On a version or dependency problem: write it to QUESTIONS.md and stop.
- Do not touch: <folders or files>

## Negative control
How to prove the check really fails on bad input (e.g. shift by one, feed an empty file, remove the new rule).

## Acceptance command
```
<the exact command to run, and the expected result>
```

## Delivery
- Where: hub/07_mail/<topic>/<file>.md
- One page, three-line conclusion first, sources cited.
- If sub-agents were used: one line saying how many, for what, with what result.
