# Iraq -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Iraq`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Iraq.xlsx` (128 generators + 1 storage plant, single zone "IRAQ").

- `Type="GT"` (gas turbine) has no tech synonym at all (only `"OCGT"` is recognized) -- renamed
  to `"OCGT"`. `Type="INT"` (import) -- renamed to `"Import"`, same DUMMY-generator pattern as
  Lao's Thai/Cambodian import proxies.
- `fuel1="CRUDE"` has no fuel synonym -- folded into `"FuelOil"` (closest recognized oil
  product). `fuel1="Rooftop"` (`Type="PV"`) is a genuinely distinct tech, not a fuel -- and
  unlike Maldives, `REProfile` here *does* have real, separate rooftop-specific profile data.
  Overridden to `Type="Rooftop"`/`fuel1="PV"` so it resolves to seer's own `solar-rooftop`
  (synonym `"Rooftop"`) instead of being merged into `solar-utility` -- an earlier pass of this
  conversion folded it into plain `PV`, which (a) lost the real distinction and (b) caused a
  "conflicting duplicate profile" drop between the two techs' profiles that silently cost every
  Iraqi solar generator its renewable-profile match entirely.
- Same `BuildLimitperYear`/`MaxTotalBuild`-duplicates-`Capacity` artifact as Lao: 117 of 117
  Existing/Committed plants have it.
- Storage is enabled and real (same convention as Maldives: storage plants also appear as
  ordinary `Generator Data` rows with `Type="STORAGE"`).
- **Capex trajectories are enabled and real** (`Settings1.Captraj=1`, unlike Lao/Maldives) --
  keyed directly by real generator names, melted straight to `pCapexTrajectoriesCustom.csv`.
  They only cover 2018-2040, while `Demand_Forecast` starts a year earlier at 2017 (a
  historical/base year, not a real investment year) -- `y.csv` is restricted to the
  intersection so `check_capex_trajectory` doesn't fail on the extra year.
- Demand uses the "less detailed" (forecast) formulation, like Maldives.

**Finalized** (2026-10-19) -- seer dataset in `data/seer/final/epm_open/Iraq` (results in
`data/seer/results/Iraq`, 2025-2029, and `.../annualized`; records in
`data/reports/country_report.pdf`). Contains two hand edits (below: LEB profile rows, capacity
derate) -- regenerating overwrites them, re-apply both. All 23 years (2018-2040) stay in the
dataset; 2025-2029 is only the solve window. Nothing is dropped: all 128 `Generator Data`
plants (117 existing/committed, 11 build options) and the one battery come through.
- **Regenerated 2026-10-19** with the current pipeline. No warnings for missing `StYr`/`Life`.
  Fuel prices are already $/MMBtu and match the `FuelPrices` sheet (2021: gas $6.4, crude
  $11.8, HFO $10.2, diesel $22.1, uranium $0.37). `Duration` weights sum to 8760 h (4
  quarters x 3 days). 117 existing/committed units, 53.8 GW in all (20.2 GW in 2018, 42.1 GW
  in 2025 as the committed CCGTs come online).
- **Manual change -- dropped the `WI_NewWind_Rate1_LEB` rows from `pVREgenProfile.csv`** (hand
  edit in `data/processed_epm_open/Iraq/`, **re-apply after `convert_epm_open.py`**, then re-run
  `jobs.epm_to_seer`): the Lebanon template-leftover row is t1..t72, while every Iraqi profile
  is t1..t24. `convert_season_day_hour` takes the step count per file, so this one row put the
  whole file in Senegal's "3 days packed in t1..t72" mode and every hydro plant's
  GenAvailability profile came out on days 11/21/31 -- failed validation (times not in
  `times.csv`). The row was "harmlessly dropped" before that Senegal logic existed.
- **Manual change -- VOM in marginal cost** (`include_vom`): `Imports_Iran` (200 MW, HeatRate 0)
  is priced only through VOM ($83/MWh) and came out free. Also adds the real thermal VOMs ($4
  CCGT/ST, $17 OCGT, $7 diesel ICE, $5 hydro, $2.2 nuclear, $3 CSP).
- **Manual change -- fixed capacity derate (hand edit of `generator_constants.csv` in the seer
  folder)**: the 90 existing fossil units with a `GenAvailability` below 0.8 have `capacity` =
  workbook Capacity x availability -- all 64 OCGTs (0.7), the older CCGTs (2 at 0.7, 3 at 0.61)
  and the steam/diesel `oil` plants (12 at 0.61, 9 at 0.65). Existing fossil 45.6 -> 39.2 GW.
  Units at 0.8-0.95 (the big committed CCGTs, some oil) and every build option (0.8-0.95) are
  left at nameplate. Without it CCGT ran flat out at its full 24.4 GW. Stricter than EPM, which
  (per the converter's `_availability_profiles` note) applies `GenAvailability` as a seasonal
  energy limit, not an hourly cap -- chosen because the derates are large. Reproduce: multiply
  `capacity` by the `GenAvailability` Q1 value for tech CCGT/OCGT/oil, not expandable, value <
  0.8. Hydro keeps its availability as a profile (no derate).
- **Demand not reshaped** (decided on review): peak x `DemandProfile` gives 21-54% *more* energy
  than EPM's `Energy` forecast (load factor 0.83 vs EPM's 0.54-0.68; 2025: 259 vs 206 TWh). An
  Egypt-style reshape (`p' = 1 - k(1 - p)`, k = 2.284) was tried and rejected -- it pushed
  off-peak days down to ~0.2 of peak. Kept as in the workbook.
- **Fuel prices blank 2018-2020** in `Fuel Prices` (the `FuelPrices` sheet computes 0, so EPM
  itself ran those years with free fuel). The pipeline back-fills each fuel's first real year
  (2021), so 2018-2020 thermal costs equal 2021's.
- **Capacity shortfall to 2024** -- EPM's own data: existing + committed is 21.3 GW against a
  30.3 GW peak in 2021 (14.9 GW derated), and no build option starts before 2026
  (CCGT/OCGT/HFO/crude/CSP/wind 2026, diesel 2027, PV/nuclear/hydro 2028). A 2021-only pyomo
  solve (before the derate) shed 9.3 TWh (~6% of demand, up to 9.3 GW, ~3,000 h). Hence the
  2025-2029 solve window (first year existing covers most of the peak). Real Iraq does have chronic shortages; EPM's own VOLL is $500/MWh (seer: $1M).
- Left as-is (EPM's assumptions): hydro `GenAvailability` is flat across quarters (0.18-0.52);
  nuclear and the new-hydro option have no `MaxTotalBuild` (no total limit); the battery
  option is 10 GW max power / 5 GWh (`Storage` sheet) -> 0.5 h duration, $200/kWh, life 20
  (`Storage` sheet; `Generator Data` says 10). Six thermal/nuclear options have no capex
  trajectory -> flat. Not modeled: trade (`interconMode=1`, lines to Jordan/KSA/Kuwait 0 MW
  and Syria 227 MW, but the neighbours aren't zones -- dropped by `jobs.epm_to_seer`),
  `GenAvailability` of 0.8 and above, 15% planning reserve, min-gen, spinning reserve, ramps (all on in
  `Settings1`), CSP storage (`CSP` sheet).
- **Results (2025-2029, with the derate)**: models agree -- upfront $152,545M pyomo vs
  $152,556M PyPSA (+0.01%), capex +0.3%, dispatch within 0.6%; annualized $147,363M both.
  **Lost load in 2025 only**: 90 GWh (0.04% of demand), up to 1.2 GW over 146 h, the same in
  both models -- derated firm capacity is 34.7 GW (excl. solar) against a 35.6 GW peak and
  nothing can be built before 2026. At seer's $1M/MWh VOLL that's ~$90B, ~60% of the objective.
  Upfront builds ~4.2 GW CCGT, ~3.9 GW OCGT, battery 400 MWh pyomo / 370 MWh PyPSA (storage
  losses). Annualized builds 10 GW PV and 8 GW wind (both at their annual build limits),
  ~7.7 GW CCGT, ~2.2 GW OCGT, 400 MW hydro.
- **General fix found here -- PyPSA load shedding**: PyPSA had no lost-load slack, so the 2025
  shortfall was infeasible there (pyomo sheds at $1M/MWh). Added one load-shedding generator
  per zone at the same VOLL, kept out of the output tables as in pyomo. No effect where nothing
  is shed (Djibouti re-solved: $398.63M, unchanged).
- **Manual change -- `settings.csv` gets `voll,500`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $500/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
