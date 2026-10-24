# Keeping HM as HM — two additions that sit outside it

Notes for the Edge City Goa community-fund experiment. Both additions below leave the Habermas Machine's generation and selection exactly as published. Take either, both, or neither.

## The constraint, and why it's right

The value of the Goa run is that it tests the Habermas Machine on a real allocation decision. That only holds if the HM arms are recognizably the published system. Change how HM generates candidates or how it picks among them and the result is about a new system, not about HM, and the comparison to Tessler et al. is lost. So the rule for anything we add should be: HM's generator and HM's selection rule stay as published, and anything else lives beside them.

The suggestions below were written with that rule in mind. Neither touches HM's internals. One adds an item to what participants see; the other concerns a step the published HM does not define at all.

## Where the published HM ends and the Goa design begins

Worth being precise about, because it decides what counts as "changing HM." The published HM (Tessler et al., *Science* 2024) generates candidate statements, scores each with a personalised reward model, picks **one** by Schulze, shows it, takes critiques, and regenerates. It never shortlists, and it never asks participants to rank several candidates.

| Step | Published HM | Goa design | Whose choice |
| --- | --- | --- | --- |
| Generate candidates | LLM, from everyone's opinions | LLM, 6–8 plans from proposals + context | HM's (keep) |
| Predict each person's view | Reward model predicts agreement | MoralMod utilities, or Hermes rankings | HM's in spirit (keep) |
| Select | Schulze picks **one** | Nash or Schulze picks **four** | Goa's addition |
| Show and rank | One statement, critique | Four plans, participants rank, Schulze over real rankings | Goa's addition |
| Revise | Regenerate once | Regenerate once | HM's (keep) |

Two things follow. The shortlist-of-four is not an HM component, so how the four are chosen is open without affecting fidelity; keeping Schulze for the final pick over participants' real rankings is what preserves the paper's rule. And the MoralMod arm (Nash welfare over estimated utilities) is already NYU's own method rather than the paper's, so the fidelity concern applies to the Hermes arm more than to MoralMod.

## Addition 1: a fifth plan on the slate, computed rather than generated

HM produces its four plans exactly as designed. A fifth plan is added beside them, computed from the same frozen proposals and the same participant inputs by the Method of Equal Shares (Peters, Pierczyński, Skowron, NeurIPS 2021): each participant gets an equal notional share of the budget, proposals are funded when their supporters' shares can cover them, and the cost is split in proportion to support. It is proportional by construction and comes with a receipt per person ("your share went 43% to A, 57% to B"). Participants rank five instead of four; Schulze over their real rankings picks the winner as now.

What it needs: a per-person support level for each proposal (0–3 is enough), extracted from the same transcripts or agent conversations by one fixed prompt; proposers count as supporters of their own proposals, as the rules already say. The explanation shown to participants is written the same way as the other four, so the comparison is between plans, not between write-ups.

What it gives: a direct head-to-head on real human rankings in every group, between a transparent proportional plan and HM's sampled plans, with HM untouched. If the fifth plan never wins, that is a clean and useful result. If it does, the paper has a baseline reviewers in computational social choice will expect anyway. The private step-4 allocations also let the team check how well the 0–3 extraction recovered what people wanted, and the same check applies to MoralMod's utilities and Hermes's rankings.

Cost to the design: one more row on the ranking screen and one line in the pre-registration. Nothing else moves.

## Addition 2 (optional): a multi-winner shortlist, Schulze kept for the final pick

This one is only worth considering if the team agrees the shortlist is a Goa step rather than an HM step. If not, skip it; Addition 1 stands on its own.

The issue it addresses is the one the "distinct plans" prompt change is already aimed at. Taking the top four from a single-winner ordering (Nash product, or Schulze) returns the winner and its three nearest variants. The prompt change tries to make the candidate pool more varied so that those four differ; a multi-winner rule makes the shortlist itself representative, whatever the pool looks like. Chamberlin–Courant picks the four plans that maximise how much each person likes their favourite plan on the slate, and runs on the same predicted rankings or utilities the arm already computes. STV with four seats is the other standard choice. In both cases the final winner is still chosen by Schulze over participants' real rankings, so the paper's rule is preserved where it matters.

With six to eight candidates and four slots the rules will often agree, which is why this is optional. They diverge exactly when the generated plans are near-duplicates, which is the case the prompt change is trying to prevent, so the two approaches are complements: the prompt improves the pool, the rule improves the pick. If the team prefers to leave selection alone, the diversity measure below will show whether the prompt alone was enough.

## Free regardless: a validity check and a diversity measure

Two small things that help whatever is decided above.

**A plan validator.** LLM-written plans get the arithmetic wrong often enough that a post-generation check is worth having: every funded proposal at or above its minimum (or zero), the full budget spent, nothing above the ideal if that is the rule. `cns_allocate.py --check` does this for any plan from any source and either flags or rejects. One protocol question it surfaces: the rules say "at least its minimum" with no cap, so a plan may legally put the whole budget into one proposal unless the ideal is treated as an upper bound. Worth settling before the generator is finalised.

**A slate-diversity measure.** The mean pairwise L1 distance among the four plans shown. It costs nothing to log, it tells the team whether the "distinct plans" prompt did what it was meant to, and it is the number that makes a fifth-plan result interpretable (a challenger that wins only when the other four are near-copies is a finding about the shortlist, not about the challenger).

## Code and references

**Code** (MIT, this repository):

- [`cns_allocate.py`](cns_allocate.py) — takes a proposals CSV (id, minimum, ideal) and a support matrix CSV (one row per person, 0–3 per proposal) and returns four named plans: Equal Shares (with a documented completion rule so the full budget is spent), Nash welfare, utilitarian, maximin. Prints per-person utilities, pairwise L1 distances between plans, Equal Shares receipts, and validity. `--check plan.json` validates any plan against the funding rules. Details: [ALLOCATOR.md](ALLOCATOR.md).
- Equal Shares is also available in [pabutools](https://pypi.org/project/pabutools/), maintained by the rule's authors and tested on real city participatory-budgeting data.

**References**

- Tessler, Bakker, Jarrett et al. *AI can help humans find common ground in democratic deliberation* (Science, 2024): the Habermas Machine.
- Peters, Pierczyński, Skowron. [Proportional Participatory Budgeting with Additive Utilities](https://arxiv.org/abs/2008.13276v2) (NeurIPS 2021): the Method of Equal Shares. Practitioner site: [equalshares.net](https://equalshares.net/).
- Fish, Gölz, Parkes, Procaccia et al. [Generative Social Choice](https://www.arxiv.org/abs/2309.01291) (JACM 2026), and Boehmer, Fish, Procaccia, [Generative Social Choice: The Next Generation](https://mlanthology.org/icml/2025/boehmer2025icml-generative) (ICML 2025): representation guarantees for LLM-generated slates.
- [Finding Common Ground in a Sea of Alternatives](https://arxiv.org/pdf/2603.16751) (2026): on selection rules inside Habermas-Machine-style systems and the limits set by generation.
- Chamberlin and Courant, *Representative Deliberations and Representative Decisions* (APSR, 1983): the multi-winner rule named above.
