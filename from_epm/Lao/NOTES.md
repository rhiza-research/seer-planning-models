# Lao -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Lao`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Lao.xlsx` (152 generators, 7 zones: North/Central1/Central2/South
are Lao itself, TH_N/TH_S/CM are Thai/Cambodian interconnection zones).

- **Generator data** comes from the `Generator Data` tab (human-readable Zone/Type/fuel1 as
  text), not `GenData` (the same data GAMS actually reads, but zone/fuel/tech as opaque
  indices) -- avoids needing to decode `ZoneData`/`FuelTechnologies` index tables.
- Same **`BuildLimitperYear`/`MaxTotalBuild` duplicates `Capacity`** raw-data artifact as every
  processed_gams region: 132 of 135 Existing/Committed plants have it. Dropped both so they're
  correctly treated as non-expandable (existing).
- `DUMMY_TH_N`/`DUMMY_TH_C`/`DUMMY_CM` (1000 MW each, at the Thai/Cambodian zones) are real
  modeled import-proxy generators (`ZoneData` flags all 7 zones "Include in model = YES"), but
  their raw `Type="STO HY"` is wrong -- overridden to `tech=fuel="Import"`, matching the same
  convention already used for real import generators in Eswatini/Tanzania.
- `fuel1="Bio"` -> `"Biomass"`, matching the vocabulary every other region uses (no
  standardization mapping exists for `"Bio"`).
- **Storage and capex trajectories are disabled** for this model (`Settings1`:
  `includeStorage=0`, `Captraj=0`), and both sheets reference placeholder plant/generator names
  that don't exist in `Generator Data` -- written as empty rather than wired up.
- **Fuel prices** (`Fuel Prices` tab) are keyed by Country, not Zone (`ZoneData`'s
  `MapCountry`/"MAP ZONES TO COUNTRIES" gives the zone -> country mapping); each country's price
  is broadcast to every zone in it.
- **Demand** uses the "detailed" formulation (`Settings1.altDemand=0`): the `Demand` tab gives
  real MW values per (zone, season, day, year, hour) directly, rather than a separate
  peak/energy forecast + normalized profile (`Demand_Forecast`/`DemandProfile` are both marked
  "IGNORED" in the workbook). Confirmed the per-hour shape is exactly proportional across years,
  so (like gams) the derived `pDemandProfile.csv` is year-invariant: `peak = max(MW)` and
  `energy = sum(MW x weight)` per zone/year, `profile = MW / peak`. `weight` (from the
  `Duration` tab -- how many real hours each representative hour stands for, confirmed
  year-invariant too) is carried as an extra column on `pDemandProfile.csv`; the seer pipeline
  doesn't consume it yet, but it's there for future use rather than being silently discarded.

**Finalized** (2026-10-20) -- seer dataset in `data/seer/final/epm_open/Lao` (results in
`data/seer/results/Lao`, 2026-2030, and `.../annualized`; records in
`data/reports/country_report.pdf`). Four zones (North, Central1, Central2, South) joined by
three lines; no trade. Everything is in `REGIONS["Lao"]` except two hand edits (below) --
**re-apply them after regenerating**.
- **Manual change -- `settings.csv` gets `voll,2500`** (hand edit in the seer folder,
  2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"), $2,500/MWh.
  Both models now read `voll` from `settings.csv` (general model change; absent -> $1M/MWh, so
  countries without it are unchanged -- Iraq re-solved: same objective).
- **Manual change -- `zones.csv` country renamed `Country1` -> `Lao`** (hand edit in the seer
  folder, 2026-10-20). The workbook's `MapCountry` names it `Country1`. No effect on the solve.
- **Regenerated 2026-10-20** with the current pipeline; validates. Years 2020-2030 (the
  `Demand` tab's years); every zone has demand in every year. Profile is year-invariant, so
  peak x profile reproduces the `Demand` tab's energy exactly. The four zones have different MW
  but exactly the same normalized shape (the demand-profile plot now draws identical curves
  once, labeled with every zone -- a general plot change). Peak 2.5 GW in 2025, 4.1 GW in 2030.
  Fuel prices are already $/MMBtu and match the `FuelPrices` sheet (2020 coal $3.79, bio $1);
  `Fuel Prices` stops at 2025, so 2026-2030 hold 2025's prices.
- **`Duration` sums to 8772 h, not 8760** (Dry 213 x 24, Wet alternating 152/153 per hour) --
  EPM's own weights, kept.
- **Manual change -- dropped the Thai/Cambodian zones** `TH_N`/`TH_S`/`CM` (`drop_zones`; user's
  choice, 2026-10-20). Each is a flat 1000 MW load met by a 1000 MW `DUMMY_*` import proxy at
  $75/MWh VOM, so there is nothing spare for Lao to import (short of shedding that zone's load)
  -- they only value Lao's exports, and seer models no exports. Dropped with them: the three
  proxies (3 GW) and the lines Central1-TH_N (200 -> 800 MW), Central2-TH_S (120 MW) and
  South-CM (80 MW in, 880 MW out). Tested before dropping: keeping the zones with their load
  zeroed instead would give Lao $75/MWh imports up to the line limits that EPM doesn't have
  (2026-2030: 78 GWh of peak imports, no builds, objective $99M vs $418M).
- **Manual change -- VOM in marginal cost** (`include_vom`): Lao's own VOMs (hydro $0.65, coal
  $0.12, biomass $3.04). Without them hydro and PV are both free and the two models split their
  output arbitrarily (PV 541 vs 34 GWh in a test run).
- **Manual change -- dropped 37 generators** (`drop_generators`): 10 candidates with
  Capacity/BuildLimitperYear/MaxTotalBuild all 0 (came out expandable with no limit: Pak Lay,
  Xanakham, Phou Ngoy, Xekong 4/5, Bualapha Coal Thermal, Kaleum_140, Houay Ho (retirement),
  Wind_147, the two "connection" rows), and 27 existing/committed rows with 0 MW (EPM zeroes the
  export IPPs -- Hongsa, Nam Theun 1/2, Xayabury, Nam Ngum 2/3, Xekaman, Don Sahong, ... -- and
  some placeholder groups). Left: 105 existing/committed units, 5.6 GW (4.5 GW hydro -- 4.3 GW
  after the pre-2020 retirements --, 870 MW coal, 187 MW PV, 38 MW biomass), and 7 build
  options (555 MW: 200 MW coal, 230 MW hydro, 125 MW PV).
- **Retired before the horizon** (EPM's `RetrYr`): Nam Ngum 1 (155 MW, 2010), Nam Dong and
  Xelabam (2009). Kept as in the workbook.
- **Network steps up over time** (general pipeline change, 2026-10-19): `TransferLimit` rises
  (Central1-North 1304 -> 2104 MW from 2023, Central1-Central2 214 -> 776 from 2022;
  Central2-South flat 201 MW). It used to be averaged over 2018-2030; now each increase is its
  own `network.csv` row with a `start_year`, so every modeled year has EPM's limit.
- **Manual change -- Dry-season hydro availability raised halfway to 1** (`availability_raise:
  {"m1": 0.5}`, a' = a + 0.5(1 - a), applied to every `GenAvailability` profile; user's choice,
  2026-10-19). Lao is ~80% reservoir hydro by capacity, and seer caps it hourly at its seasonal
  `GenAvailability`, where EPM uses it as a seasonal energy budget (full capacity in any hour).
  At the Dry average of 0.39 it can't meet the dry-season evening peak (hours 18-21): with the
  cap at EPM's value the four-zone model sheds 1.64 TWh over 2026-2030 (up to 1.3 GW). Raising
  Wet instead changes nothing (the shortage is all in Dry). Dry mean is now 0.69 (0.56-0.89),
  the same as Wet's 0.69 -- so dry-season hydro energy can be up to ~78% above EPM's budget.
- Storage and capex trajectories off (`Settings1`), as before -- the two `Storage` sheet plants
  have no zone and are excluded. Existing units carry their workbook Capex (ignored for
  non-expandable units). Not modeled: trade (zones dropped above), min-gen
  (`mingen_constraints=1`, but every `MinLimitShare` is 0), line expansion
  (`pAllowHighTransfer=1`).
- **Results (2026-2030, VOLL $2,500)**: pyomo and PyPSA agree exactly. Upfront: $229.4M,
  nothing built, marginal $121.3M; 77.4 GWh lost load (2030 only, up to 247 MW in North plus
  98 MW in South at the evening peaks; $193.6M at VOLL) -- every build option would only come
  online in 2030, and with full capex charged in the horizon's last year, shedding a few peak
  hours at $2,500 is cheaper. Dispatch hydro 82.9 TWh, PV 1.6 TWh, coal 1.4 TWh, biomass
  0.1 TWh. Annualized ($127.8M both): no lost load; builds Bualapha coal (200 MW),
  Luangprabang (60 MW) and 8 MW of Pak Ngum in 2030. Coal runs only at the evening peaks (from
  2028); hydro follows the load the rest of the time. (At the old $1M VOLL the upfront run built
  the same 268 MW instead: $418.5M, no lost load.)
