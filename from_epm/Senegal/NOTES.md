# Senegal -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Senegal`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Senegal.xlsx` (25 generators, single zone `SEN`). Detailed demand
(`Q1..Q4`). No `CapexTrajectories` sheet at all. `GT` renamed to `OCGT`; `Type="PV"`/
`fuel1="Solar"` (generator side) and `REProfile`'s own `tech="Solar"` both independently
resolve to `solar-utility` bare, no rename needed despite the differing raw spelling.
**The "Storage" sheet here isn't battery/pumped storage data at all -- it's CSP thermal-storage
configuration** (`EnergyLimit`/`PowerLimit`/`CSP_SolarMultiple`/...), a different table
entirely under the same sheet name. Exposed a **general robustness gap**: storage-sheet reading
now checks for a `Battery` column before trying to use it, rather than assuming the standard
layout everywhere.
Each quarter is one `d1` day type with `t1..t72`: three 24-hour days packed together, weighted
by `Duration` (13.0 / 26.1 / 52.1 h per hour -> 8760 h). The pipeline now splits them into
three days per quarter (12 modeled days). The earlier "empty `renewable_profiles.csv`" (demand
`t1` missing) no longer happens: the Demand sheet's `T1` is read case-insensitively.

**Finalized** (2026-10-26) -- seer dataset in `data/seer/final/epm_open/Senegal`
(results in `data/seer/results/Senegal`, 2026-2030; records in `data/reports/country_report.pdf`).
One zone, 2017-2030, 1,348 MW existing (844 MW oil/diesel/coal, 235 MW PV, 150 MW wind, 120 MW
ROR hydro), 6 build options, no storage (`EnergyLimit`/`PowerLimit` all 0), no lines. Changes in
`REGIONS["Senegal"]` plus **two seer-folder hand edits (below) -- regenerating overwrites them,
re-apply both**.
- **Regenerated 2026-10-22** with the current pipeline; validates.
- **General fix -- `RetrYr` 0 is a blank** (`to_start_year_and_life`, `inputs/epm2seer/
  generator_params.py`): every existing unit here has `RetrYr` 0, which came out as "retires
  before it starts" (life 1) -- all 19 existing units offline in every modeled year. Now 0
  means no retirement year (life 100), like a blank. No other region has a `RetrYr` of 0.
- **Manual change -- ROR profile** (`re_profile_tech_renames: {"Water": "ROR"}`): `REProfile`'s
  `Water` row (flat 0.359, all quarters) is `HY_19_WA_SEN`'s (120 MW, Type `ROR`, Senegal's
  OMVS hydro share); unmatched, the plant had no profile and ran unlimited at 120 MW for free.
  Now 43 MW average, ~377 GWh/yr.
- **Manual change -- `zones.csv` country `Senegal`** (seer hand edit; `MapCountry` gave `SEN`).
- **Manual change -- `settings.csv` gets `voll,500`** (seer hand edit): `Settings1` VOLL, $500/MWh.
- `GenAvailability` is 0.9 for every plant (>= 0.8): no thermal derate.
- VOM not included: every fuel is priced in `Fuel Prices` ($/MMBtu, matches the computed
  `FuelPrices` sheet); VOM is $1-11/MWh on top. Gas $7.3/MMBtu (2025) for the CCGT/OCGT build
  options -- Senegal has no gas-fired plant yet, EPM's assumption.
- **General fix -- explicit `Inf` build limits come out blank** (`generator_params.py`,
  `storage_params.py`), like every finalized country's unlimited options; they were written as
  `inf` before (Senegal, Poland, several gams regions). Same solve (pyomo $2,539.1M).
- Build options have `Inf` build limits and placeholder sizes (10,000 MW CCGT/PV/CSP); capex
  CCGT $1.02M/MW, OCGT $0.87M, PV $1.0M, CSP $3.0M, coal $4.0M, HFO steam $1.9M.
- Not modeled: imports from the external zone `KAM` (`ZoneExternal`, `TradePrices` $27-32/MWh
  growing 3.2%/yr) -- no import capacity is given anywhere (`TransferLimit` holds only a stray
  Lao-template block, `Topology` is empty), so there is nothing to carry over; zonal spinning
  reserve; 15% system reserve (`planning_reserve_constraints` 0 anyway); CSP thermal storage
  (`includeCSP` 0 -- CSP is a plain solar-profile plant here, as in EPM).
- Demand: peak 761 MW (2017) -> 1,535 MW (2030), +5.5%/yr; weighted load factor 0.675, 4.5 TWh
  in 2017 (no `Energy` row to compare against -- detailed demand). The three packed days jump at
  their joins (e.g. a peak-day evening into the next day's midnight) -- as in the workbook.
- **Results (2026-2030, VOLL $500)**: pyomo and PyPSA both $2,539.1M upfront, $2,030.0M
  annualized; no lost load in either. Upfront builds 812 MW of CCGT in 2026 (gas at ~$50/MWh vs
  HFO $128+, coal $59) and nothing else; generation 40.8 TWh = demand: CCGT 31.9 TWh, coal 2.1,
  ROR 1.9, solar 1.8, oil 1.6, wind 1.5. Annualized builds CCGT year by year to 1,074 MW (2030),
  pushing coal (1.0 TWh) and oil (0.4 TWh) further out. No PV built in either: at $1M/MW and
  CF 0.18 it costs ~$75/MWh, above the CCGT's fuel cost. CSP never built.
