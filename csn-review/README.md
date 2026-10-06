# csn-review

An AI reviewer for developer blog posts. It scores a draft from 1 to 10 on one
question: **did the authors actually do the thing they write about, and can a reader check it?**

## Why

Generic technical content is cheap now. Anyone can produce a clean, correct-sounding
post about a tool they never ran. What readers still can't get elsewhere is the
first-hand version: the real repo, the commit that fixed it, the dead end that cost
a day, the number measured on real CI.

This skill gates publication on that. A well-written post with no evidence scores
low. A rough post with a real repo, verified commits, and honest failures scores high.

## Why not just ask the model for a score

Ask an LLM to rate an article from 1 to 10 and it will answer 7 or 8 almost every
time. The skill avoids that in four ways:

1. **Evidence before scores.** The reviewer first builds an evidence ledger: every
   claim, the link behind it, and whether it opened the link and confirmed it.
   Only then does it score.
2. **Six criteria with described levels.** Each criterion is scored 0–3, and every
   level is described by something you can see in the text ("at least one rejected
   alternative, with the reason"), not by adjectives ("good", "strong").
3. **A formula.** The final score is a weighted sum. First-hand experience and
   verifiable evidence carry the most weight.
4. **Caps.** Some problems limit the score no matter how good the rest is. No
   verified evidence at all → at most 4. An invented statistic → at most 3.

## What it checks

| Criterion                  | Weight | In short                                              |
|----------------------------|--------|-------------------------------------------------------|
| First-hand experience      | 3      | A real project, with details only a practitioner knows |
| Verifiable evidence        | 3      | Repos, commits, PRs, screenshots — opened and checked |
| Failures and trade-offs    | 2      | Dead ends, rejected options, and what they cost        |
| Technical depth            | 2      | Explains why, not only what; knows the limits          |
| Reader takeaway            | 1      | Something a reader can apply in their own repo         |
| Trust and voice            | 1      | No hype, no invented numbers, a clear verdict          |

| Score  | Verdict        |
|--------|----------------|
| 9–10   | Flagship       |
| 7–8.9  | Publish        |
| 5–6.9  | Revise         |
| 3–4.9  | Rework         |
| 1–2.9  | Not a post yet |

A solid, publishable first-hand post is a 7. Most first drafts land at 4–6.

## What you get back

- The score, the verdict, and any caps that were applied.
- A table with each criterion's level, the quote that justifies it, and what would
  raise it by one.
- The evidence ledger, so you can see which links were checked.
- The top three fixes, ordered by how much they would raise the score.

Send a revised draft and the reviewer scores it again, showing what moved and why.
If two rounds don't raise the score, it tells you the problem is missing evidence
or experience, not wording.

The reviewer does not rewrite your article. For edits, use a copy-editing skill.

## Install

Copy or symlink the `csn-review` folder into your project's skills directory:

```bash
ln -s /path/to/skills/csn-review .claude/skills/csn-review
```

Optional, but recommended:

- **`gh` CLI, authenticated.** The reviewer uses it to open repos, commits, and PRs.
  Without it, it falls back to the public GitHub API (60 requests per hour, public
  repos only).
- **`.agents/product-marketing.md`** in your project. If it exists, the reviewer
  uses its audience, voice rules, and banned phrases.

## Use

Ask in plain words:

- "Review this draft before we publish" (paste the text or give a file path)
- "Is this worth a post?"
- "Score this article, here's the repo: https://github.com/..."

Include links to the repo, commits, or PRs. If the draft has none, the reviewer
asks for them once before scoring.

## Testing the skill

`evals/evals.json` holds test cases: drafts with known problems and the score
range the reviewer must land in. Run them after every change to the rubric, for
example with the `skill-creator` skill.

To calibrate the reviewer to your team, score a few articles by hand and add them
as cases with an expected range ("Final score is between 6.5 and 8.5"). If the
reviewer keeps disagreeing with you on one criterion, sharpen that criterion's
level descriptions in `SKILL.md`.

## Limits

- It can't check private repos it has no access to, or read every screenshot. Such
  evidence counts as offered but unverified.
- Scores vary slightly between runs. Treat a difference under one point as noise.
- It judges evidence and honesty, not whether the topic will find an audience.
