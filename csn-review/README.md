# csn-review

An AI reviewer for developer blog posts. It scores a draft on three questions:

- **Proof:** did the authors actually do the thing they write about, and can a reader check it?
- **Originality:** what does the reader get here that the tool's docs don't give?
- **Author Authority:** would a skeptical reader trust these authors on this topic?

## Why

Generic technical content is cheap now. Anyone can produce a clean, correct-sounding
post about a tool they never ran. What readers still can't get elsewhere is the
first-hand version: the real repo, the commit that fixed it, the dead end that cost
a day, the number measured on real CI.

First-hand is not enough, though. A post can be fully real and verified and still
teach nothing: the authors followed the library's docs, it worked on the first try,
and they wrote it up. Such a post proves experience but gives the reader nothing they
could not read in the docs. Proof and Originality are scored separately so that one
cannot hide the other.

The three scales roughly follow how Google describes quality: Proof covers Experience
and Trust, Originality covers "original information … beyond the obvious", and
Author Authority covers the people behind the content.

## Why not just ask the model for a score

Ask an LLM to rate an article from 1 to 10 and it will answer 7 or 8 almost every
time. The skill avoids that in five ways:

1. **Ledgers before scores.** Each scale first collects facts: the evidence behind
   every claim (opened and checked), where each key takeaway already exists in the
   docs (with a URL), and what independent sources confirm about each author.
   Only then does it score.
2. **Criteria with described levels.** Each criterion is scored 0–3, and every
   level is described by something you can see ("at least one rejected alternative,
   with the reason"), not by adjectives ("good", "strong").
3. **A formula.** Each scale is a weighted sum, mapped to 1–10.
4. **Caps.** Some problems limit the score no matter how good the rest is. No
   verified evidence at all → at most 4. An invented statistic → at most 3. Nothing
   the docs don't already say → Originality at most 3.
5. **Independent scorers.** If the agent can run subagents, each scale is scored by
   its own subagent that never sees the other scores. A convincing post can't make
   the reviewer generous about its originality.

## What it checks

### Proof

| Criterion                  | Weight | In short                                              |
|----------------------------|--------|-------------------------------------------------------|
| First-hand experience      | 3      | A real project, with details only a practitioner knows |
| Verifiable evidence        | 3      | Repos, commits, PRs, screenshots — opened and checked |
| Failures and trade-offs    | 2      | Dead ends, real rejected options, and what they cost   |
| Technical depth            | 2      | Explains why, not only what; knows the limits          |
| Reader takeaway            | 1      | Something a reader can apply in their own repo         |
| Trust and voice            | 1      | No hype, no invented numbers, a clear verdict          |

### Originality

| Criterion        | Weight | In short                                                       |
|------------------|--------|----------------------------------------------------------------|
| New knowledge    | 3      | Takeaways that are in neither the docs nor the usual articles  |
| Real alternatives| 2      | Compared with what a reader would actually pick, ideally measured |
| Point of view    | 1      | A non-obvious verdict, argued from the authors' results        |

### Author Authority

Scored only when the authors are known: a byline, an author page, a CMS author
record, or names from the user. For bare text it is `n/a` and does not count.

| Criterion                | Weight | In short                                           |
|--------------------------|--------|----------------------------------------------------|
| Identity                 | 2      | A real, named person with matching external profiles |
| Experience in this topic | 3      | Public work in the topic of this post, not in general |
| Recognition by others    | 2      | Merged PRs to known projects, talks, citations     |
| Publications in topic    | 1      | A body of work on this topic                       |

An anonymous byline caps Authority at 2.

## The final score

```
final = mean of the scored scales
final ≤ min(Proof, Originality) + 2
final ≤ Proof's hard caps (contradicted central claim, unsourced statistic,
        not first-hand, no verified evidence)
```

The "+ 2" rule stops one strong scale from carrying a failed one: a verified,
honest post that only repeats the docs can't get past Revise. Author Authority is
left out of it, because editing the article can't change who wrote it.

A contradicted side detail (a wrong line count, a wrong claim about a tool the post
is not about) does not cap the score. It lowers Verifiable evidence by one level
and is listed first among the fixes.

| Score  | Verdict        |
|--------|----------------|
| 9–10   | Flagship       |
| 7–8.9  | Publish        |
| 5–6.9  | Revise         |
| 3–4.9  | Rework         |
| 1–2.9  | Not a post yet |

A solid, publishable first-hand post is a 7. Most first drafts land at 4–6.

## What you get back

- The final score, the verdict, the three scale scores, and any caps.
- Must-fix facts: every factual error any scale found, checked against the repo or
  the official docs.
- "Why this post?": one sentence on what the reader gets here and nowhere else, and
  angles to try if the answer is "nothing".
- The top three fixes, ordered by how much they would raise the final score.
- For each scale: a table with each criterion's level, the quote or ledger row that
  justifies it, and what would raise it by one, plus the ledger itself.

Send a revised draft and the reviewer scores it again, showing what moved and why.
If two rounds don't raise the score, it tells you the problem is missing evidence,
experience, or new knowledge, not wording.

The reviewer does not rewrite your article. For edits, use a copy-editing skill.

## Install

Copy or symlink the `csn-review` folder into your project's skills directory:

```bash
ln -s /path/to/skills/csn-review .claude/skills/csn-review
```

Optional, but recommended:

- **`gh` CLI, authenticated.** The reviewer uses it to open repos, commits, PRs, and
  author profiles. Without it, it falls back to the public GitHub API (60 requests
  per hour, public repos only).
- **Web fetch or search.** Needed to check takeaways against the docs and to look up
  authors. Without it, Originality and Authority are less reliable.
- **Subagents.** If your agent supports them, the three scales run in parallel and
  independently. Without them, the reviewer scores them one after another.

## Use

Ask in plain words:

- "Review this draft before we publish" (paste the text or give a file path)
- "Is this worth a post?"
- "Score this article, here's the repo: https://github.com/..."

Include links to the repo, commits, or PRs, and say who wrote the post. If the draft
has no evidence, the reviewer asks for it once before scoring.

## Files

- `SKILL.md`: the workflow, the final score, the verdict, and the output format.
- `references/proof.md`, `references/originality.md`, `references/authority.md`:
  one scale each, self-contained so a subagent can follow it alone.
- `evals/evals.json`: test cases.

## Testing the skill

`evals/evals.json` holds test cases: drafts with known problems and the score
range the reviewer must land in. Run them after every change to a rubric, for
example with the `skill-creator` skill.

To calibrate the reviewer to your team, score a few articles by hand and add them
as cases with an expected range ("Final score is between 6.5 and 8.5"). If the
reviewer keeps disagreeing with you on one criterion, sharpen that criterion's
level descriptions in its `references/` file.

## Limits

- It can't check private repos it has no access to, or read every screenshot. Such
  evidence counts as offered but unverified.
- Originality is only as good as the docs search. It checks the official docs of the
  tool the post is about and a few common sources, not the whole internet.
- Authority uses only professional facts from profiles it can tie to the author;
  a search hit with the same name is ignored.
- Authority depends on what is public. LinkedIn and similar sites often block
  fetches; facts from them count as offered, not verified.
- Scores vary slightly between runs. Treat a difference under one point as noise.
- Three subagents cost roughly three times one review.
- It judges evidence, novelty, and authorship, not whether the topic will find an
  audience. For search demand, use a content strategy or SEO skill.
