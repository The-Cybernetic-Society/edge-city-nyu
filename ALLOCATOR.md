# cns_allocate

Compute funding plans from a frozen proposal list and a per-person support matrix,
under the Community Fund Allocation rules (0 or ≥ minimum per proposal; whole
budget spent). One file, numpy + scipy, MIT.

Four rules, each producing one named plan:

| rule | what it optimises | property |
|---|---|---|
| `mes` | Method of Equal Shares | proportional: a group that agrees and holds enough equal shares gets its project |
| `nash` | product of per-person utilities | the Nash welfare plan (what the MoralMod arm's Nash rule targets, without a candidate pool) |
| `utilitarian` | sum of utilities | most total benefit |
| `maximin` | the lowest utility | best for the worst-off person |

Per-person utility for a plan: `u_i = Σ_j support_ij · amount_j / ideal_j`.

## Run

```
pip install numpy scipy
python3 cns_allocate.py --demo                                   # synthetic data
python3 cns_allocate.py --proposals proposals.csv --support support.csv --budget 300000 --out plans.json
python3 cns_allocate.py --proposals proposals.csv --budget 300000 --check some_plan.json   # validate any plan
```

## Inputs

`proposals.csv`
```
id,name,minimum,ideal
water,Drinking-water filter for the hall,60000,90000
```
`minimum` may be blank (any positive amount allowed). Amounts are capped at `ideal`
unless you pass `--no-cap` (the protocol leaves this open; decide it).

`support.csv` — one row per person, one column per proposal id, values 0–3
(0 = no support, 3 = strong). Proposers should be given 3 for their own proposal.
```
person,water,kids,scooter
p01,3,1,0
```
Where the matrix comes from is up to you: a fixed extraction prompt over agent
conversations or the discussion transcript, or a direct ballot. Publish the prompt.

## Output

A table of the four plans, each person's utility under each, pairwise L1 distance
between plans (how different the slate is), validity, and for `mes` a receipt per
person showing where their equal share went. `--out` writes it all as JSON.

## Equal Shares: completion rule (pre-register this)

MES normally leaves money unspent; the protocol requires the full budget. The
steps are fixed and logged:

1. Run MES with each proposal's cost = its minimum. This decides **what** is funded
   and charges supporters in proportion to their support.
2. Top up funded proposals toward their ideals in proportion to total support,
   charging each proposal's supporters in proportion to their remaining shares.
3. If money is still left, bring in unfunded proposals most-supported first, at
   their minimum, and repeat step 2.
4. If money is still left and `--no-cap` is not set, the plan is reported INVALID
   rather than silently exceeding ideals.

## Scale

`nash`, `utilitarian`, `maximin` enumerate funded subsets (fine to ~14 proposals;
~1 s for 8). Beyond that, replace the enumeration with a MILP (scipy `milp` or
OR-Tools CP-SAT) using a binary "funded" variable per proposal with
`minimum·y ≤ x ≤ ideal·y`. `mes` has no such limit.

## What this is for

One or more of these plans can be added to the pool the shortlist rule chooses from,
or can replace an LLM plan generator outright: four plans, each labelled by what it
optimises, straight into the ranking step. `--check` validates any plan from any
source against the funding rules.
