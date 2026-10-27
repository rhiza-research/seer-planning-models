# Chad -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Chad`). Paths below refer to the seer repo.


**Finalized** -- seer dataset in `data/seer/final/epm_open/Chad` (results in
`data/seer/results/Chad`, 2022-2026: pyomo $503.7M vs PyPSA $504.4M; annualized capex in
`data/seer/results/Chad/annualized`; records in `data/reports/country_report.pdf`). No hand
edits -- regenerating reproduces it.

Source: `data/raw_epm_open/Chad.xlsx` (26 generators + 4 real storage plants, 2 zones: `Chad`
itself, `Cameroon` interconnection-only). Forecast demand (`Q1..Q4`; only Q1/Q2 used, 3 day
types each). All 12 of 12 Existing/Committed plants have the duplicate-of-capacity artifact.
Capex disabled.
- **Regenerated 2026-10-19** with the current pipeline: fuel prices now converted from $/litre
  (FuelOil $0.42, Diesel $0.61 in 2020) and $/MWh (import $130) with `FuelTechnologies`' table
  -- matches the workbook's `FuelPrices` sheet (FuelOil $12.4, Diesel $17.9/MMBtu). Marginal
  costs: HFO ~$99-149/MWh, diesel ~$154-244/MWh, `Cameroon_Import` $130/MWh. `Duration`
  weights sum to 8760 h. Ramp rates all raw 0 -> 1.
- **Manual change -- dropped zone `Cameroon`** (`drop_zones`): `ZoneData` "Include in model:
  NO", no generators or demand, `interconMode=0`. Its 100 MW `TransferLimit` from 2028 is
  already represented by the Chad-zone `Cameroon_Import` generator (Committed, 100 MW, 2028,
  $130/MWh). No other trade modeled.
- **Manual change -- dropped disabled candidates** (`drop_generators`): `Djermaya`, `Farcha3`,
  `HFO_CGGC`, `Sarr` (HFO) and `Quadran_international` (PV) have Capacity/BuildLimitperYear/
  MaxTotalBuild all 0 -- EPM couldn't build them, but they came out expandable with no limit.
- **Manual change -- dropped 0 MW existing rows** (`drop_generators`): `Farcha_2`, `MBH` and
  their HFO conversions `Farcha_2HFODiesel`, `MBH_HFODiesel` (inert, just clutter).
- Existing system: 89 MW of diesel/HFO in 2020-2021 (below the 105/112 MW peak -- start
  solves in 2022), then the diesel units retire in 2022 and their HFO conversions
  (`*_HFODiesel`, 131 MW) start. These conversions are `Existing` with a Capex of $1.8M/MW --
  kept in `generator_costs.csv` but never used (existing units aren't charged capex).
- Left as-is: `Generic_HFO` has BuildLimitperYear 0 (blank -> no annual limit) but a 500 MW
  total; `Generic_PV_5`'s UnitSize (300 MW) exceeds its 200 MW limits (only matters with
  discrete additions); `Generic_Storage_4` has no annual limit. Storage MWh limits are the MW
  limits x 4 h (the Storage sheet's MWh / Capacity), not the Storage sheet's MWh.
- Not read: `PPAprice` (VPOWER/Aggreko/NRC $38-66.5/MWh, PV_Djermaya/AMEA $110-116/MWh) and
  `PPAfix` columns; VOM ($10/MWh existing, $5.67 candidates) not included. Not modeled (on in
  `Settings1`): 15% planning reserve margin, min-gen, ramp and spinning reserve constraints.
- Results (2022-2026): all oil, plus ~40 MW of new HFO and ~32 MW of batteries for peaks; no
  PV built. Both models charge the full capex in the build year by default (`jobs.solve_country`
  doesn't annualize), so 5 years can't recover a PV build. Annualized, PV at $0.9M/MW and CF
  ~0.2 is ~$59/MWh vs ~$115/MWh HFO: pyomo with `annualize_capex=True` (2022-2026) builds
  278 MW of PV by 2026 and ~14 MW less oil; PyPSA (now also supports `--annualize-capex`)
  263 MW, objective $372.0M vs $373.9M (re-solved after the PyPSA marginal-cost fix below). PV stops at what midday demand absorbs (~95% of its
  output used), not at build limits. Pyomo cycles the batteries freely (no efficiency), PyPSA
  barely uses them.
- **Manual change -- `settings.csv` gets `voll,500`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $500/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
