# Sierra Leone -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Sierra Leone`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Sierra Leone.xlsx` (17 generators, single zone `Sierra_Leone`;
storage disabled). Forecast demand, `season`/`day` as plain integers (`1`/`2`) like Sudan --
needed both a `season_map` and `day_map`. `KARPOWERSHIP_IC_IPP` (a real floating power-ship
asset, 50 MW, Existing) has `Type=fuel1="IPP"` -- neither is a real tech or fuel
("Independent Power Producer" is an ownership/procurement label, mistakenly used here).
Classified as `ICE`/`HFO` (Karpowership's typical engine/fuel type -- an unconfirmed but
reasonable guess for this specific vessel, user-confirmed, consistent with this region's other
`ICE`/`HFO` generators). `Type="IMP"` here (not `"INT"` like other regions) for its import
generator -- the override matches on `fuel1`, not the original `Type` spelling, so no extra
`tech_renames` entry was needed.

**Finalized** (2026-10-26) -- seer dataset in `data/seer/final/epm_open/Sierra Leone`
(results in `data/seer/results/Sierra Leone`, 2026-2030, re-solved -- the old results predated
the model fixes; records in `data/reports/country_report.pdf`). One zone, 2019-2030, 2 seasons
x 2 day types (`Duration` 109/42/152/62 -> 8760 h). 331 MW existing incl. committed (243 MW
hydro, 27 MW CLSG import, 55 MW oil after derates, 6 MW PV), 7 build options, no storage
(`includeStorage` 0), no lines. Changes in `REGIONS["Sierra Leone"]` plus **four seer-folder
hand edits (below) -- regenerating overwrites them, re-apply all**.
- **Regenerated 2026-10-26** with the current pipeline; validates.
- Hydro availability now comes through (the integer-`GenAvailability`-header fix found on
  Sudan): Bumbuna-I 0.30/0.94, Bumbuna-II and Bumbuna-I NEW 0.65/0.52, Bekongor-III 0.10/0.57.
- **Manual change -- `include_vom`**: `KARPOWERSHIP_IC_IPP` ($170/MWh) and `CLSG_IMP`/`CLSG
  NEW_IMP` ($120/MWh) have HeatRate 0 and are priced only through VOM -- all were free without
  it. Adds $10/MWh to HFO/diesel, $2/MWh to gas, $2.77/MWh to hydro.
- **Manual change -- `drop_generators: ["CAND_HY"]`**: Capacity/BuildLimitperYear/
  MaxTotalBuild all 0 (disabled).
- `CLSG NEW_IMP` stays at 0 MW, as EPM has it (user's choice): `Committed` 2025 with Capacity 0
  but a 223 MW build limit; seer imports can't be expandable. A first pass set it to 223 MW --
  then imports undercut all oil (none ran) and nothing was built; reverted.
- **Manual change -- demand scaled x1.160** (seer hand edit, `demand_profile.csv`, user's
  choice): `DemandProfile` peaks at 0.705, not 1.0, so peak x profile was 14% below EPM's
  `Energy` (334 vs 387 GWh in 2019). Scaled so energy matches EPM in every year (energy/peak is
  the same every year); the shape is unchanged and the modeled peak is 82% of EPM's `Peak`
  (59 vs 72.6 MW in 2019; 214 vs 262 MW in 2030).
- **Manual change -- thermal derates** (seer hand edit, `generator_constants.csv`/
  `generator_costs.csv`): Blackhall Road 15 -> 10.5 MW and Kingtom 9 -> 6.3 MW (0.7; Kingtom
  retired 2008 anyway); Karpowership 50 -> 38.27 MW (1.0 dry / 0.6 wet, Duration-weighted
  0.766). Build options `CAND_IC_Gas/HFO/Diesel` (0.7, as Namibia's `GCoal_Nam_CO`): unit size
  50 -> 35 MW, build limits 1,000 -> 700 MW, capex / 0.7 (gas $2.14M/MW, HFO $2.18M, diesel
  $1.71M).
- **Manual change -- `zones.csv` country `Sierra Leone`** (seer hand edit; `MapCountry` gave
  `Sierra_Leone`).
- **Manual change -- `settings.csv` gets `voll,500`** (seer hand edit): `Settings1` VOLL, $500/MWh.
- PV/wind `GenAvailability` (0.17-0.19 / 0.05-0.10) is a capacity factor, already in the
  `REProfile` profiles (solar mean 0.15, wind 0.06) -- not applied again.
- Build options: CAND_PV ($1.2M/MW), WB_PV 30 MW ($1.0M), CAND_WIN ($1.38M), Bekongor-III
  hydro 160 MW ($4.06M), and the three derated engines above. Diesel price drops 2029 -> 2030
  ($26.6 -> $20.6/MMBtu) -- as in the workbook.
- Not modeled: 10% planning reserve, 50% max import share (`MaxImports`), fuel and min-gen
  constraints (`Settings1`); no fuel limits, trade prices or transfer limits are filled in.
- **Results (2026-2030, VOLL $500)**: pyomo and PyPSA both $245.5M upfront, $185.8M
  annualized. Generation 5.62 TWh: hydro 4.1, gas engines 0.66, CLSG import 0.48 (27 MW, near
  its cap 2026-27), oil 0.34 (Karpowership/Blackhall, almost all 2026-27). 2026-27 are tight
  (Bumbuna-I only, 15-47 MW of hydro) until Bumbuna-II and Bumbuna-I NEW (193 MW) arrive in
  2028; hydro carries most of the load after. Upfront builds 27.9 MW of gas engines in 2026
  and sheds 41.7 GWh in 2026-27 (dry-season evenings and nights, up to 35 MW) -- at $500/MWh
  cheaper than another engine charged in full for two short years. Annualized builds 56.9 MW
  instead and sheds only 0.9 GWh (2027 dry evenings). No PV: the hydro surplus from 2028
  leaves it little to displace.
- Cosmetic: `CAND_IC_Gas` (Type `ICE`, gas engines) comes out tech `CCGT` -- labeled CCGT in
  the report; costs and dispatch unaffected. Bumbuna-I NEW (50 MW) alongside Bumbuna-I is
  Bumbuna phase II's extension (with Yiben = Bumbuna-II, 143 MW), not a duplicate.
