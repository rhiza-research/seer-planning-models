# Ukraine -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Ukraine`). Paths below refer to the seer repo.

Source: `data/raw_epm_open/Ukraine.xlsx` (197 generators; detailed demand, Q1-Q3 x 9 days;
capex trajectories on). **Finalized** (2026-10-29) -- seer dataset in
`data/seer/final/epm_open/Ukraine` (results in `data/seer/results/Ukraine`, 2026-2030, and
`.../annualized`; records in `data/reports/country_report.pdf`). 8 zones (Z01-Z03, Z05-Z09),
2020-2050, 3 seasons x 9 day types (648 h, `Duration` weights sum to 8760). 51.5 GW existing
after derates (nuclear 13.8 GW, coal 19.5, gas-fired steam/CCGT 4.4, gas CHP 2.2, coal CHP 0.6,
hydro 4.7, PV 5.0, wind 1.2, biomass 0.1) + 1.5 GW of pumped hydro, against a 26-28 GW peak
(sum of zone peaks). 82 generator build options (per zone: coal, CCGT/OCGT incl. CCS from 2035,
nuclear from 2032, PV, rooftop PV, wind, offshore wind, biomass, hydro) plus `Kaniv_PSPS` and a
10 h battery per zone. Changes in `REGIONS` plus **five seer-folder hand edits (below) --
regenerating overwrites them, re-apply all with `data/processed_epm_open/Ukraine/hand_edits.py`**
(run from the seer repo root after regenerating, then move the folder to `final/`).
- Fuel Prices: the key column is `Country` and year headers are text -- both handled (general
  fix); `BIOGAS` has no unit and no price -- skipped. Coal grades $1.9-2.2/MMBtu (mc $19-29/MWh),
  gas $4.7 (gas steam $56, CCGT build option $30), nuclear $4.
- Zones: Donbas plants are `z01` and `REProfile` spells zones lowercase (`z02`..`z09`,
  `z010`) -- `zone_renames` to `Z01`..`Z10`. Dropped the neighbouring-country zones
  (z11-z22: Russia, Belarus, Moldova, Poland, ENTSO-E, Hungary, Slovakia, Romania, Austria --
  EPM external trade zones, no generators; trade at `TradePrices` isn't modeled) and
  `Z00`/`Z04`/`Z10` (no generators, no demand), with their lines. EPM has `allowExports=1`.
- Lines: kept as EPM has them, including `Z06`-`Z09` at 0 MW until 2022 and 9,999 MW from 2023
  (Z06 is the Burshtyn island, joined to the main grid after ENTSO-E synchronization).
- `CT` -> `OCGT`, `Nuk` -> `Nuclear`, `HY` -> `Water` with GenAvailability profiles (as Burkina).
  Hydro averages ~0.2 (8 TWh/yr from 4.7 GW).
- **Manual change -- `RooftopPV_Z7` dropped** (`drop_generators` in `REGIONS`): it sits in zone
  Z08, which has no `ROOFTOP_PV` `REProfile` row, so EPM gives it zero output; here it came out
  `isre=False` -- free, unlimited, dispatchable capacity (as Morocco's offshore wind).
- **Manual change -- PyPSA-aligned labels** (seer hand edit, `generator_constants.csv`, label
  only): coal grades `Anthracite`/`G/DG42`/`G/DG52`/`G/DG52_CCS` -> fuel `coal`; `Gas_CCS` ->
  `gas`; coal CHP (`CHP`) -> `central coal CHP`; the gas-fired steam units typed `Coal`/`Gas`
  (Vuhlehirska, Zaporizka gas units, 4 GW) -> `CCGT` (the tables' `ST-Gas` convention);
  `ROOFTOP_PV` -> `solar-rooftop`/`solar`. Pumped hydro (`*_PSPS`, `storage_constants.csv`) ->
  `PHS`.
- **Manual change -- utilization-based thermal derate** (seer hand edit, `generator_constants.csv`;
  user's call, as Poland): GenAvailability on 3.9 GW of CHP is heat-led (0.4-0.66 in Q1, ~0-0.4
  in Q3), Trypilska's gas units and the biomass plants are 0.35 -- running-hours targets, not
  outages. Capacity x min(1, a / typical CF), a = Duration-weighted Q1-Q3 mean, typical CF
  coal CHP 0.68, gas CHP 0.50, CCGT 0.55, biomass 0.70: 20 units, 4,629 -> 3,321 MW (coal CHP
  x0.40-0.64, gas CHP x0.55-0.99, Trypilska gas x0.64, biomass x0.50). The nuclear
  GenAvailability (0.61-0.88) is on 0 MW plant-level rows (`Khmelnitska_NPP`,
  `Pivdenno_Ukrainska_NPP`); the unit rows that carry the capacity have none, so nuclear is
  not derated.
- **Manual change -- pumped hydro energy read as kWh** (seer hand edit, `storage_constants.csv`;
  user's call): the `Storage` sheet's `Capacity` is labeled MWh and EPM reads it as MWh, but the
  values look like kWh -- as MWh, Kievska (235.5 MW) has 8,739 h and Tashlyk 3,311 h. Divided by
  1,000: Dnistrovska 972 MW x 42.7 h, Kievska 8.7 h, Tashlyk 3.3 h, `Kaniv_PSPS` build option
  2.7 h (2,700 MWh, $300k/MWh). The generic `Storage_Z*` batteries (100,000 MWh at 10,000 MW =
  10 h, $400k/MWh) are consistent as MWh and left as-is. Efficiency 0.72 (PSPS) / 0.9.
- **Manual change -- per-year demand profile** (seer hand edit, `demand_profile.csv` gets a
  `year` column; user's call): the converter takes one profile per zone from the first year
  ("year-invariant", true for Lao), but Ukraine's load factor rises -- peak x that profile fell
  short of EPM's `Energy` by 3.8% in 2026, 7.8% in 2030, 15% in 2040. Now each year's own
  `Demand` shape / that year's peak: energy matches EPM in every year (164.7 TWh 2026, 176.0
  2030, 201.0 2040, 240.0 2050). `seer/viz` (report) reads per-year profiles since 2026-10-28.
- **Manual change -- `settings.csv` gets `voll,5000`** (Settings1 VOLL). **Re-apply after regenerating.**
- Not modeled: 15% planning reserve margin, `MaxImports`, trade with the neighbouring zones,
  CHP heat obligations (EPM's heat-led availability is only reflected in the derate).
- **Results (2026-2030, VOLL $5,000)**: pyomo and PyPSA identical -- $4,530.6M, upfront and
  annualized alike: nothing is built (51.5 GW existing vs a 26-28 GW peak) and there's no lost
  load. Generation 851.5 TWh over 5 years: nuclear 606 (13.8 GW at ~100%: its units have no
  GenAvailability), coal 126, PV 39, hydro 38, coal CHP 24, wind 19; gas (CHP, CCGT) none --
  everything above ~$22/MWh (gas CHP $22.5, most coal) sits idle behind nuclear and the
  cheapest coal (Zaporizka, Vuhlehirska, Kryvorizka, Slovianska). Pumped hydro cycles daily (losses
  ~0.24 TWh/yr). Solves: pyomo ~60 s, PyPSA ~80 s.
