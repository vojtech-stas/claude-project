# 2026-09-08 — ADR corpus rethink (topic entries become truth, ADRs become archive)

**Context.** A grill was owed on whether `decisions/` is still the right shape for this project's decision layer. The operator's stated instinct, recorded three weeks earlier, was to *"name decisions by the problem they solve, not by sequential ID … so they form a queryable knowledge base with full history in git"*. A prior decision had already retired the reading-order work with the sharper observation that *"the ADR mechanism itself is in question"*. So the rethink was not a new idea — it was an unlanded preference finally being grilled, and the job was to measure what the current mechanism actually does before proposing anything.

## Measured grounding

Three defects in the ADR → rules machinery, each verified against the files rather than inferred:

| # | Defect | Measurement |
|---|---|---|
| 1 | **Rule prose does not come from the ADRs.** All 92 rule texts are a Python dict inside the generator (`tools/gen_rules.py:279`); no ADR carries the prose the generator emits. Editing an ADR's decision text does not change the rule a session reads, and nothing compares the two. | 92 hardcoded rule statements; 0 ADRs carrying `rule_statements` |
| 2 | **Supersession lives in prose, not data.** The index narrates supersessions; the frontmatter field that the generator's `_is_superseded()` reads is almost entirely unpopulated. | 1 file carries `status: superseded` against 40 of 83 index rows mentioning one |
| 3 | **Half the corpus feeds nothing.** ADRs with no frontmatter cannot contribute a rule, a scope, or a supersession. | 39 of 81 ADRs (48%) have no frontmatter at all |

And one asymmetry that decided the shape of the answer: **the ADR number is a fully load-bearing handle; the rule id is not.** Counted over tracked files only (`git ls-files`, after a filesystem walk gave an 8× inflated figure by reading stale worktree copies):

- **2,003** `ADR-NNNN` citations outside `decisions/`, spanning **all 81** distinct ADRs.
- **450** rule-id citations, of which 97 sit inside the generator itself and 55 inside a single ADR — leaving rule ids barely used as handles anywhere else.

Any proposal that invalidates ADR numbers therefore breaks 2,003 references; one that retires rule ids as *external* handles breaks almost nothing.

## Prior-art sweep

The sweep was run at the operator's request before deciding granularity, and it changed two of the answers.

- **Four vendors independently converged on the same three fields for rule loading** — an always/glob/on-request selector, a path-glob, and a description used to decide relevance. Four independent designs reaching the same shape is the strongest signal in the sweep, and it is what the loading decision adopts wholesale.
- **The industry-standard decision-record format does *not* solve supersession**: it folds "superseded by …" into a free-text status string. This repo's separate `supersedes` / `superseded_by` fields are *better* than that standard — merely unpopulated. Adopting a known format would have made defect 2 worse, so the sweep's main effect was to stop a wrong move.
- **No tool was found that generates agent rules *from* a decision corpus.** The closest concatenates hand-authored prose and has no drift check. On the generation half this project is ahead of the field; what it lacks is loading semantics — which is exactly the half the four vendors solved.
- Off-the-shelf pieces that do apply, and are worth reusing rather than reinventing: a frontmatter-schema linter for defect 3, an existing decision-record lint rule for the supersession check, and the regenerate-and-diff CI pattern for defect 1 — the last one only once a real generation function exists.

## Queue triage (the grill's required pre-step)

187 open issues, 0 open PRs at grill time. Every item was sorted into exactly two buckets:

| Bucket | Count | Shape |
|---|---|---|
| **just-do** — no design decision needed | ~178 | 85 root-cause captures already naming symptom, cause and proposed fix; ~35 refactor/dead-code; ~30 guard-and-check gaps; ~25 doc-currency and citation items |
| **needs-decision** — a genuine design or policy fork | 8 | the forks below |

Six operator-owed items settled by the 2026-09-07 close-out were excluded from the fork set rather than re-asked. One item initially triaged as a fork was corrected to just-do on reading it: it carries its own exact fix, and its operator-owed half was already applied out of band.

## Decisions

**Q1 — Framing.** The registered topic is *"ADR-format rethink: a topic-entry memory format instead of ADRs?"*. The operator confirmed the framing and said to proceed with the measured version above rather than re-open the premise.

**Q2 — What shape does the decision layer take?**

- [x] **Topic entries become the truth; the ADRs become a frozen archive** — **chosen**. It fixes all three defects at once (the entry carries the prose, so defect 1 has nowhere to live; the entry carries real supersession fields; every entry has frontmatter by construction) while leaving all 2,003 ADR citations valid.
- [ ] Keep ADRs and populate their frontmatter properly — rejected: it fixes defect 3 and half of defect 2, and leaves defect 1 exactly as it is.
- [ ] Replace ADRs outright, rewriting citations — rejected: 2,003 references for no gain the archive doesn't already give.

**Q3 — What is the granularity of an entry?**

- [x] **One entry per topic (~25–30 files), rule ids retained as internal handles** — **chosen**. This is the unit every converged format scopes at, and the citation count shows rule ids are not load-bearing externally, so keeping them internal costs nothing.
- [ ] One entry per rule (~92 files) — rejected: the convergent prior art scopes at topic level, and 92 files re-creates the fragmentation the rethink is meant to remove.
- [ ] One entry per area (~8 files) — rejected: too coarse for path-scoped loading to mean anything.

**Q4 — How does a session decide which entries to load?**

- [x] **Adopt the four-mode inclusion field** — always / glob+pattern / agent-requested+description / manual — **chosen**. Four vendors reached it independently, and it converts two existing open defects from bugs into settings.
- [ ] Keep the current always-loaded plus path-scoped split — rejected: it has no on-request mode, which is where most of the context budget is currently spent.

**Q5 — What happens to `decisions/` after the cut?**

- [x] **Freeze it at its current highest number; new decisions are entry supersessions** — **chosen**. An archive that keeps growing is not an archive, all existing citations stay valid, and the numbering-collision hazard closes because nothing new claims a number.
- [ ] Keep writing ADRs alongside entries — rejected: two sources of truth is the defect being fixed, restated.

**Q6 — How is supersession represented across the two layers?**

- [x] **The entry states current truth and lists the contributing decision ids in its provenance field; archive prose is left untouched** — **chosen**. Roughly 40 lineages are hand-resolved once, at migration, instead of a resolver being built to interpret prose forever.
- [ ] Build a resolver over the archive's prose — rejected: it makes the prose load-bearing permanently.

**Q7 — What does a downstream adopter receive?**

- [x] **Entries only; the archive stays here and is linked, never copied** — **chosen**. Nothing in the payload can collide with a decision the adopter makes. **This reverses a prior decision** and is recorded as a new dated item superseding it, not as an edit.
- [ ] Entries plus the full archive — rejected: it re-imports the numbering hazard the freeze exists to remove.
- [ ] Entries plus a distilled rationale digest — rejected: a third artifact no check can prove still matches either of the other two.

**Q8 — The generator's hardcoded rule-id baseline.**

- [x] **It becomes a per-repo generated manifest** — **chosen**, which unblocks the open defect where any downstream repo declaring its own rule fails a check owned upstream.
- [ ] Keep the constant and special-case downstream repos — rejected: the special case is the bug.

**Q9 — How does the migration run?**

- [x] **Mechanical first, with byte-identical generated output as the proof, then a separately-reviewed re-cut pass** — **chosen**. It separates "did I move it correctly" from "is this the right cut", so a reviewer can actually judge each.
- [ ] One hand-authored pass — rejected: nothing could prove no rule changed silently.

**Q10 — Does the rethink run before or after the queue drain?**

- [x] **Before** — **chosen**. Draining first means doing every ADR-touching item twice: once against the current shape, once against the new one.
- [ ] Drain first, rethink after — rejected on that duplication.

**Q11 — Drain-ledger retention.**

- [x] **Unbounded; the choice documented where the drain is specified** — **chosen**. A delete path sitting beside a parked run's resume state is a worse failure than a slow directory scan, and the scan cost is theoretical.
- [ ] Cap by count or age — rejected: the cap would have to know which runs are still resumable, and it cannot.

**Q12 — The deferred items that surfaced during the grill.**

- [x] **Close them at the tag; the written revisit trigger carries them** — **chosen**. This satisfies the fix-or-close-everything goal with no amendment, because a documented trigger is a real disposition rather than an open item.
- [ ] Hold them open past the tag — rejected: it contradicts the standing goal directly.

## Consequence recorded rather than asked

With entries carrying the prose and the generator reduced to concatenation, the hardcoded rule-statement dict has nothing left to hold and is deleted. At that point the existing regen-clean CI check becomes a genuine regenerate-and-diff gate over real inputs — so **defect 1 dissolves by construction**, and drift between an ADR's text and the rule a session reads stops being possible rather than becoming newly detectable. This was not put as a fork because no alternative exists once Q2 and Q3 are chosen.

**Outcome.** Twelve forks answered, none left open. Six existing open items are absorbed by these answers rather than needing their own design pass; three new root-cause captures were filed and promoted during the same session. The next step is the queue drain, entered with this record as its input and with the rethink program ahead of the rest of the queue per Q10 — not a build launched from this grill.
