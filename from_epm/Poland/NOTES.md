# Poland -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Poland`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Poland.xlsx` (147 generators + 1 real storage plant, 6 zones: `PSE`
is Poland itself (its TSO's code used as the zone name); Slovakia/Germany/Lithuania/Sweden/Czech
are interconnection-only zones).

- `Type="GT"`/`"Onshore"` have no tech synonym at all (only `"OCGT"`/`"Wind"` are recognized)
  -- renamed. `Type="INT"` (a single 0 MW placeholder import generator) -- renamed to
  `"Import"`.
- **`Type="CHP"` (69 plants, the largest single group) and `Type="WtoE"` (3 plants) had no
  PyPSA-aligned tech at all** -- unlike every other rename in this file, these aren't spelling
  variants of an existing category, they're genuinely missing ones. Checked PyPSA's own
  technology-data cost database directly (`outputs/costs_2020.csv`) and found real entries for
  them (`central coal/gas/solid biomass CHP`, `waste CHP`) -- added those as new synonym rows to
  `seer/inputs/utils/data/pypsa_tech_names.csv` (compound `CHP-Coal`/`CHP-Gas`/`CHP-Biomass`/
  `WtoE-MunicipalWaste` keys) and `pypsa_fuel_names.csv` (`MunicipalWaste` -> `solid biomass`,
  `LNGCCS` -> `gas`), rather than folding them into an unrelated existing category (they have
  materially different economics: CHP dispatch is partly driven by heat demand, not just power
  price -- not modeled by seer yet, but at least now correctly classified for when it is).
- No `BuildLimitperYear`/`MaxTotalBuild`-duplicates-`Capacity` artifact here (0 of 135
  Existing/Committed plants), like Maldives.
- **Season labels are inconsistent within the same workbook**: `DemandProfile`/`REProfile`/
  `TransferLimit` use `Winter`/`Summer`, but `REgenProfile` (2 rows, one of them the same kind
  of unrelated template leftover seen in other regions) uses `Q1`/`Q2` instead for the same
  two-season split -- both map onto the same `m1`/`m2` time grid.
- Demand uses the "less detailed" (forecast) formulation, like Maldives/Iraq.
- **Capex trajectories are enabled and real**, covering the full modeled horizon (2020-2050,
  matching `Demand_Forecast` exactly) -- no truncation needed here, unlike Iraq.

**Finalized** (2026-10-27) -- seer dataset in `data/seer/final/epm_open/Poland` (results in
`data/seer/results/Poland`, 2026-2030; records in `data/reports/country_report.pdf`). One zone (PSE), 2020-2050, Winter/Summer x 9 day types
(`Duration` -> 8760 h). Changes in `REGIONS["Poland"]` plus **two seer-folder hand edits
(below) -- regenerating overwrites them, re-apply both**.
- `drop_zones` Slovakia/Germany/Lithuania/Sweden/Czech: ZoneData "Include in model: NO", no
  generators or demand, and `allowExports=0` means EPM ignores `TradePrices` -- no trade in EPM
  either. Their `TransferLimit` lines (300-1690 MW) go with them.
- `availability_profile_types` `["STO HY", "ROR"]`: the two ROR plants (988 MW) had no profile
  (REProfile's `Water` rows, CF 0.75, match no plant Type) -- unlimited free generation. They take
  their GenAvailability (0.19/0.20, real Polish ROR ~0.2), which is also EPM's seasonal energy
  cap on them.
- `drop_generators`: `HFGT` (hydrogen OCGT -- its fuel only comes from EPM's electrolyzer module,
  `H2 Data`, not modeled; it would burn free fuel) and `Generic_ROR` (Capacity,
  BuildLimitperYear and MaxTotalBuild all blank -- disabled).
- **Manual change -- `settings.csv` gets `voll,2500`** (Settings1 VOLL).
- **Manual change -- thermal derates, utilization-based** (seer hand edit,
  `generator_constants.csv`; decided on review): `GenAvailability` here reads as a utilization
  (energy) limit -- EPM's calibration to historical output, applied as a seasonal energy cap --
  not an outage rate, so capacity is set so that capacity x a typical capacity factor = EPM's
  limit: factor = min(1, a / cf), a = Duration-weighted Winter/Summer mean (4368/4392 h), only
  where a < 0.8. Typical cf: hard coal 0.60, lignite 0.83, coal CHP 0.68, gas CHP 0.50, CCGT
  0.55, OCGT 0.10, biomass/waste 0.70. Lignite and coal CHP started at 0.75/0.50, but the model
  then ran them at 0.91/0.85, 21%/68% over EPM's energy limit -- set halfway (user's choice).
  Existing thermal 37.3 -> 26.6 GW (hard coal 18.1 -> 12.0 GW, lignite 8.8 -> 6.0, coal CHP
  5.0 -> 3.2; gas CHP and the CCGT unchanged; Stalowa Wola ~0).
  No build option is derated (each one's a >= its cf; `Generic_OCGT` 0.2 vs 0.10). The plain
  fixed derate (capacity x a, Iraq's rule) was tried first: 17.9 GW, 840 GWh shed in 2026 even
  with every thermal option at its annual cap. Reproduce: the factors above, applied to
  `capacity` of existing units with tech/fuel in that list.
- Left as-is: StYr 1900 on 14 CHP units (EPM's "long ago"; RetrYr gives the right window);
  build options' `UnitSize` 50,000 MW (only matters with modular expansion); peak x profile is
  5% above EPM's `Energy`; STO HY plants are Poland's pumped-storage stations, modeled by EPM as
  hydro with availability 0.04-0.09 (kept); battery 3 h, $400/kWh, 4.5 GWh/yr, 150 GWh total.
  Fuel prices are already $/MMBtu (match `FuelPrices`); `includeVOM=0` and no plant is priced
  only through VOM.
- Not modeled: 9% planning reserve, fuel limits, min-gen, ramps, spinning reserve, CHP heat
  demand, H2 production (all on in `Settings1`); 50% max import share.
- **Results (2026-2030, VOLL $2,500, utilization-based derate)**: pyomo and PyPSA identical
  (no storage built): $20.87B upfront, $19.50B annualized. 2026 generation ~178 TWh: coal 124
  (hard coal 70, lignite 48 + others) + coal CHP 24, wind 15.5, PV 9.2 (incl. 3.2 GW
  `Cumulative_PV_2021_in`), gas CHP 3.6, hydro/ROR 2.6. Against EPM's energy limits (2026):
  hard coal 70 vs 63 TWh, lignite 48 vs 44, coal CHP 24 vs 19, gas CHP 3.6 vs 17.3, CCGT 0.1 vs
  6.1 -- the energy taken off lignite/coal CHP went mostly to hard coal, not gas (coal stays
  ~$20-35/MWh vs gas $47-67). More coal, less gas than the real system (~100 / ~17 TWh in
  2023). Upfront builds 672 MW of lignite in 2026 and sheds 6-45 GWh/yr (up to 3.5 GW, winter
  peaks); annualized builds 1 GW/yr of lignite (5 GW by 2030) and 5.8 GW of PV from 2028, and
  sheds 4.5 GWh in 2026, none from 2029.
