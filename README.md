# edge-city-nyu

Notes and code offered to the Edge City × NYU community-fund allocation experiment (Goa, October 2026).

- [NOTES.md](NOTES.md) — *Keeping HM as HM: two additions that sit outside it*
- [ALLOCATOR.md](ALLOCATOR.md) — `cns_allocate.py`: Method of Equal Shares, Nash, utilitarian, maximin; a validator for any plan
- `examples/` — synthetic proposals and a 0–3 support matrix

```
pip install numpy scipy
python3 cns_allocate.py --demo
python3 cns_allocate.py --proposals examples/proposals.csv --support examples/support.csv --budget 300000
python3 cns_allocate.py --proposals examples/proposals.csv --budget 300000 --check some_plan.json
```

Part of [The Cybernetic Society](https://github.com/The-Cybernetic-Society) and [Cybernetic Sutra](https://www.cyberneticsutra.com). MIT.
