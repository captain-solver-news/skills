# Proof

Did the authors actually do the thing they write about, and can a reader check it?

Score the article alone. Do not judge whether the topic is new (that is Originality)
or who the authors are (that is Author Authority).

## Step 1 — Evidence ledger (before any score)

For every claim about what the authors built, ran, measured, or broke, and every claim
about how a tool works, add a row:

| # | Claim (quote) | Central? | Evidence | Status |

A claim is **central** if the post's result rests on it: what the authors built, the
main numbers, the outcome, and, in a post that reviews a tool, its verdict on the tool.
Side details (how many lines a snippet has, a date, a claim about a tool the post is
not about) are not central.

Status:
- `verified`: you opened it and it supports the claim.
- `offered`: evidence exists but you could not check it (private repo, unreadable image).
- `unsupported`: no evidence, or the link is dead.
- `contradicted`: the evidence says otherwise.

Check GitHub with `gh repo view`, `gh api repos/{owner}/{repo}/commits/{sha}`, and
`gh pr view`. If `gh` is not installed or not authenticated, use the public API
(`curl -s https://api.github.com/repos/{owner}/{repo}/commits/{sha}`,
`.../pulls/{number}`) or fetch the page. Unauthenticated API calls are limited to
60 per hour, so check the central claims first. A missing tool is not missing
evidence: if you cannot check a link at all, mark it `offered`, not `unsupported`.

Confirm that:
- the repo exists and is not an empty template;
- the cited commits exist and touch the files the post describes;
- commit dates fit the post's timeline;
- code snippets in the post match the repo.

Check claims about how a tool works ("the editor has no code block", "the API can't
do X") against the tool's official docs. If the docs say otherwise, mark the claim
`contradicted` and quote the docs with the URL.

## Step 2 — Score each criterion 0–3

Quote the passage or ledger row behind each level. Torn between two levels → pick the lower.

### C1. First-hand experience (weight 3)
- 0: Could be written from docs. No specific project.
- 1: Names a project, but the story is generic.
- 2: Specific project, timeline, decisions, and at least two details only a practitioner
  would know (an exact error, a config gotcha, a surprising number).
- 3: As 2, and the reader can follow what was tried first, second, and why.

A practitioner detail counts only if a reader would meet it in their own project too.
Local facts (our palette, our file names, how many languages we enabled) prove the
project exists but are not practitioner knowledge.

### C2. Verifiable evidence (weight 3)
- 0: No links, screenshots, or numbers.
- 1: Evidence is mostly `offered` or `unsupported`, or links point to docs and homepages.
- 2: Key claims link to the real repo, commits, PRs, or screenshots; most are `verified`.
- 3: Every central claim is `verified`; numbers state how they were measured.

A `contradicted` side detail lowers C2 by one level (once, however many there are)
and goes first in the must-fix list. It does not trigger the cap.

### C3. Failures and trade-offs (weight 2)
- 0: Everything worked. No alternatives.
- 1: A problem mentioned in passing, with no cost or resolution.
- 2: At least one real dead end or rejected alternative, with the reason.
- 3: Dead ends with their cost (hours, money, reverts) and a "we'd do it differently" verdict.

An alternative counts only if a competent reader would seriously consider it (a
competing library, a built-in feature, a different architecture). A straw man such as
"we could have written our own parser" does not. "It worked on the first try" caps C3
at 1. A "what we would still improve" list describes limits: credit it in C4, not here.

### C4. Technical depth and correctness (weight 2)
- 0: Errors, or code with no explanation.
- 1: Correct but explains what, not why.
- 2: Explains why for the key decisions; code is correct and minimal.
- 3: As 2, plus limits, edge cases, and when not to use the approach.

### C5. Reader takeaway (weight 1)
- 0: Nothing to apply.
- 1: General lessons.
- 2: A concrete technique, config, or checklist.
- 3: Reproducible in the reader's own repo tonight (a repo, branch, or gist to start from).

### C6. Trust and voice (weight 1)
- 0: Unsourced stats or hype ("game-changer", "unlock the power of", "in today's
  fast-paced world").
- 1: Some hype or hedging; no clear verdict.
- 2: Plain voice and a clear verdict.
- 3: As 2, and the limits of the authors' experience are disclosed.

## Step 3 — Compute

```
points = Σ(level × weight), max 36
proof  = 1 + 9 × points / 36, rounded to one decimal
```

Then apply caps. The lowest cap wins:

| Condition                                                        | Max | Also caps the final score |
|------------------------------------------------------------------|-----|---------------------------|
| A `contradicted` central claim, or an unsourced statistic        | 3   | yes                       |
| C1 = 0 (not first-hand)                                          | 3   | yes                       |
| No `verified` evidence at all                                    | 4   | yes                       |
| Code in the post does not match the linked repo                  | 5   | no                        |
| Authoritative tone on a topic new to the authors, no disclosure  | 5   | no                        |

## Report

```
Proof: X.X / 10
Caps: <none | condition → max>

| Criterion | Level | Weight | Why (quote / ledger #) | +1 would need |

Evidence ledger: <table from Step 1>

Must-fix facts: <every `contradicted` row, with what the evidence says, or none>
```
