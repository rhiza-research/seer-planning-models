# Djibouti -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Djibouti`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Djibouti.xlsx` (37 generators, 2 zones: `Djibouti` itself,
`Ethiopia` interconnection-only; storage disabled). `GT` renamed to `OCGT` (`Geothermal`
already resolves bare, no fix needed). Seasons are named (`cold`/`hot`), day-types are named
(`week`/`weekend`) -- both needed a `season_map`/`day_map` (the first two regions to need a
`day_map` at all, alongside Sudan). Capex disabled (ignore-banner mislabeled as "ENERGY
EFFICIENCY TARGETS", a copy-paste error seen again in several regions below -- the ignore
meaning still holds). **30 of 37 generators are missing `StYr` entirely** (only `RetrYr` given,
no `Life`) -- `life` can't be derived for most of this region's fleet; a genuine, substantial
source-data gap, not fixed here.

**Finalized** -- seer dataset in `data/seer/final/epm_open/Djibouti` (results in
`data/seer/results/Djibouti`, 2022-2026, and `.../annualized`; records in
`data/reports/country_report.pdf`). No hand edits -- regenerating reproduces it.
- **Regenerated 2026-10-19** with the current pipeline. Fuel prices are already $/MMBtu in
  `Fuel Prices` and match the `FuelPrices` sheet (HFO $12.8, diesel $17.4, gas $7.7, import
  $20.6/MMBtu in 2018; import x 3.412 = ~$70/MWh). `Duration` weights sum to 8760 h. The 25
  existing units with no `StYr` get 2010 (warned) -- harmless, since only `RetrYr` matters for
  their active window (MaraboutG01/G24 retire 2020, G13-16 2025); no `RetrYr` -> life 100.
  `Centrale_bifuel1-3` and `OCGT` have no `Life` -> 30 (warned).
- **Manual change -- dropped zone `Ethiopia`** (`drop_zones`): `ZoneData` "Include in model:
  NO", no generators or demand, `interconMode=0`. Imports are the Djibouti-zone
  `Interco_Ethiopia` (60 MW, existing) and `Interco_Ethiopia2` (150 MW, `StYr` 2035 -- after
  the 2030 horizon, although `TransferLimit` already raises the line to 210 MW in 2024; kept
  as in Generator Data).
- **Manual change -- VOM in marginal cost** (`include_vom`): the PPA candidates `Geoth`
  (15 MW), `Wind_on` (60 MW) and `Akuo` (30 MW PV) have a 1e-11 placeholder Capex and are
  priced only through VOM ($85/$75/$65 per MWh, matching the sheet's "Expec. @" notes) -- they
  came out free to build and run. Also adds VOM to thermal ($6 HFO/diesel, $10 CCGT, $26 OCGT).
  As PPAs they're dispatched like thermal: wind at $75 runs after the ~$70-73 imports and CCGT,
  so it's mostly held to the residual in the low-demand cold season.
- Left as-is: 6 candidates (`CCGT`, `OCGT`, `Centrale_bifuel3`, `Solar_2`, `Wind_2`,
  `Interco_Ethiopia2`) start in 2035, after the 2018-2030 horizon -- never available. Gas
  (CCGT/OCGT, $7.7-11.5/MMBtu) is EPM's assumption; Djibouti has no gas supply today. Not
  modeled (on in `Settings1`): 10% planning reserve margin, fuel limits, min-gen.
- **General fix found here -- PyPSA build options' marginal cost**: each build-option vintage
  was given its build year's marginal cost for its whole life (pyomo and existing units use
  each year's). With rising fuel prices, new thermal looked too cheap in PyPSA -- Djibouti's
  objective was 1.3% low ($393.4M vs $398.6M) with a different build. Now year-varying: the
  models agree exactly ($398.6M upfront, $358.1M annualized). All finalized countries re-solved.
- **General fix found here -- pyomo output with no storage**: `build_output_tables` crashed
  on a dataset with no storage (empty `storage_capacity` had no columns). Fixed.
- **Manual change -- `settings.csv` gets `voll,500`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $500/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
