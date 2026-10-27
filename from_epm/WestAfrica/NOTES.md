# WestAfrica -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Mauritania`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Mauritania.xlsx` (538 generators + 17 real storage plants, 16
zones). **555 raw generators, but only ~36 are in Mauritania's own zones**
(`Mauritania_grid`/`Mauritania_Nema`) -- the rest are spread across every other WAPP country
(Nigeria: 122, IvoryCoast: 48, Guinea: 47, Burkina: 46, Senegal: 44, Ghana: 35, Mali: 35,
Benin: 34, Niger: 34, plus Togo/SierraLeone/Gambia/GuineaBissau/Liberia). Confirmed real
(plausible zone-specific capacities, not a copy-paste artifact like Rwanda's SAPP mixup) --
kept as a legitimate full-WAPP-interconnection model from Mauritania's planning perspective
(user-confirmed), same scope as WAPP's own gams data.
- `GT` renamed to `OCGT`. `fuel1="Biomas"` (one 's') has no fuel synonym, unlike `"Biomass"` --
  folded so the tech-side `"ST-Biomass"` compound resolves too (same reasoning as Morocco's
  `CoalCCS`).
- `fuel1="Rooftop"` (`Type="PV"`, 14 country-level rooftop-solar candidates) -- same real
  distinct-profile situation as Iraq: `REProfile` has genuine separate `"Rooftop"` data here,
  not folded into plain `"PV"`.
- **Inconsistent season/day spelling across sheets in the same workbook**: `REProfile` uses
  `"Dry"/"dry"/"wet"` (mixed case) and `"peak"/"offpeak"`; `DemandProfile` uses `"dry"/"wet"`
  and `"peak "/"off peak  "` (stray whitespace). This exposed a **general robustness gap** --
  `season_map`/`day_map` lookups are now stripped of leading/trailing whitespace before
  matching (`map_season`/`map_day` helpers), not just for Mauritania.
- Capex trajectories are enabled and real (212 rows, though many plant names are unmatched
  template leftovers, harmlessly ignored); years 2018-2040 vs. demand's 2016-2040 -- `y.csv`
  truncates to the overlap automatically, same as Iraq.
- `pFuelPrice.csv` has no `Biomass` entry at all (only `Coal`/`Diesel`/`Gas`/`LNG`/`HFO`) --
  `Aboisso(Biokala)` (zone `IvoryCoast`, part of the full-WAPP scope described above) has no
  real biomass price in Mauritania's own data. Since this is explicitly the same WAPP
  interconnection model WAPP's own gams dataset represents, and WAPP's own `pFuelPrice.csv`
  prices `IvoryCoast` biomass directly, `marginal_cost` for `Aboisso(Biokala)` and
  `Aboisso(Biokala)_expansion` in `data/seer/from_epm_open/Mauritania/generator_costs.csv` was
  computed by hand using WAPP's own `IvoryCoast` biomass price (an exact same-country match,
  not a regional average), multiplied by the generator's own `HeatRate`. Re-running
  `scripts/convert_epm_open.py`/`jobs.epm_to_seer` for Mauritania will **not** reproduce this --
  it only lives in the committed output CSV.

**Finalized** (2026-10-21) -- seer dataset in `data/seer/final/epm_open/WestAfrica` (results in
`data/seer/results/WestAfrica`, 2026-2030, and `.../annualized`; report section `WestAfrica`).
**Renamed `Mauritania` -> `WestAfrica`** (2026-10-21, user's call): Mauritania is ~1.5% of 2030
peak demand and 36 of 538 generators, so the dataset is really a West Africa model. The converter
still names it `Mauritania`, so regenerating writes `from_epm_open/Mauritania` -- rename it again. Regenerated with the current
pipeline; validates. All 16 zones kept: `interconMode=1`, `allowExports=0`, and every zone has
its own generators and demand -- it's EPM's full WAPP model, not external trade zones. Four hand
edits -- **re-apply them after regenerating** (the first two after `convert_epm_open.py`, before
`jobs.epm_to_seer`; the last two after `jobs.epm_to_seer`):
- **Manual change -- offshore wind tech** (`data/processed_epm_open/Mauritania/pGenDataInputCustom.csv`):
  the 12 `OFSHORE_WIND_*` candidates are `Type="WIND"`, `fuel1="Offshore"`, so they came out
  `onwind` and took each zone's onshore profile. `REProfile` has separate `Offshore` rows with
  roughly double the capacity factor (Mauritania 0.49 vs 0.31, Benin 0.27 vs 0.13). Their `tech`
  column was set to `Offshore`, so they come out `offwind` and get the offshore profile.
- **Manual change -- Senegal PV wet-season hour 19** (`.../pVREProfile.csv`): blank in the
  workbook (GAMS reads that as 0). One missing hour dropped Senegal's whole PV profile, leaving
  12 Senegal solar plants with no profile. Added as 0 for both day types.
- **Manual change -- `Aboisso(Biokala)` marginal cost** (`generator_costs.csv`): $81.27/MWh flat
  (WAPP's own IvoryCoast biomass price $6.02/MMBtu x HeatRate 13.5). Mauritania's `Fuel Prices`
  has no biomass, so without this the cost is NaN and validation fails. Recomputed 2026-10-20;
  the earlier patch was already lost.
- **Manual change -- `settings.csv` gets `voll,600`**: EPM's `Settings1` VOLL, $600/MWh.
- **Manual change -- ROR availability** (`availability_profile_types: ["STO HY", "ROR"]`): the 17
  ROR plants (Felou, Gouina, Kpong, Kaleta, ...) have seasonal `GenAvailability` (0.09-0.64).
  Without it they ran flat out.
- Checked, no change: `times.csv` weights 15/168 (peak/off-peak days) x 2 seasons -> 8760 h.
  Thermal `GenAvailability` is a flat 0.85 (no derate). Fuel prices are $/MMBtu. VOM is only
  $2-10 on thermal (no imports or PPAs priced through it), so no `include_vom`. Capex: PV $0.85M/MW,
  CCGT $0.94M, OCGT $0.84M, coal $2.5M, hydro ~$3.3M, battery $250k/MWh. Network
  `TransferLimit` step-ups come through as rows. Mauritania_Nema is islanded until its 80 MW line
  in 2028; Mauritania_grid-Senegal is 80 MW, then 200 MW from 2023. Peak x profile vs EPM
  `Energy` (2030): mostly within ±12%, but Mauritania_grid +16%, Mauritania_Nema +74%, Burkina
  +23%, Mali -22%, Liberia -70%. Left as-is (no demand reshapes).
- **Results (2026-2030, VOLL $600)**: upfront pyomo $29,732M vs PyPSA $29,760M (+0.1%); capital
  $5.1B, marginal $27.7B. WAPP-wide build is 4.4 GW CCGT, 0.6 GW OCGT, 0.3 GW hydro and 0.2 GW
  PV. In Mauritania that's 369 MW `CC_GAS_CAND_MAU`. Lost load is 245 GWh, mostly evening peaks
  (19-22 h) in 2029-2030 (Guinea 75, Niger 36, Mali 27, Nigeria 26 GWh). Annualized: pyomo
  $24,367M vs PyPSA $24,388M (+0.1%), Mauritania builds 299 MW CCGT + 176 MW PV + 500 MW wind
  (`WIND_CAND_MAU` at its 100 MW/yr limit), and lost load drops to 41 GWh. Mauritania_Nema sheds
  12 GWh in both runs in 2026-2027, up to 5 MW at night: it's islanded, with 24 MW of oil and no
  build options until the 2028 line. Dispatch is gas-led (CCGT ~81 TWh/yr, OCGT ~34, hydro ~20).
  Offshore wind isn't built in either run.
