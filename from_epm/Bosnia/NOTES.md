# Bosnia -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Bosnia`). Paths below refer to the seer repo.


**Finalized** -- seer dataset in `data/seer/final/epm_open/Bosnia` (results in
`data/seer/results/Bosnia`, records in `data/reports/country_report.pdf`).

**2026-10-19:** `generator_constants.csv`/`storage_constants.csv` converted in place to the
start_year + life schema (no `retirement_year`; see the checklist). Re-solved: results unchanged.

Source: `data/raw_epm_open/Bosnia.xlsx` (191 generators + 5 real storage plants, 4 zones: `BiH`
itself, `Croatia`/`Serbia`/`Montenegro` as pure interconnection zones with no generators).

- No `MapCountry`/"MAP ZONES TO COUNTRIES" block in `ZoneData` (falls back to `zone==country`,
  same as Georgia).
- **`REProfile`'s first column holds real generator names** (`"HPP_JajceII"`,
  `"Wind_Trebinje"`, `"PV_Generic"`, ...), not zone codes -- confirmed these match
  `Generator Data`'s `Plants` values directly. This workbook has no `REgenProfile` sheet at
  all, so its per-plant hydro/wind/solar data ended up filed under `REProfile` instead of
  where every other region puts generator-specific data. Without accounting for this, every
  row is silently treated as a (nonexistent) zone+tech generic profile, matching nothing --
  `renewable_profiles.csv` came out completely empty (0 rows) until caught. Fixed with a new
  `re_profile_is_generator_specific` flag that routes `REProfile` through the same
  generator-specific processing as `REgenProfile`, with zone looked up from `Generator Data`
  by generator name, same as any real `REgenProfile` sheet. 143 of 146 renewable generators
  now correctly match a profile (the 3 gaps are a genuine incomplete-time-coverage row in the
  raw data, and two plant-name spelling/diacritic mismatches between sheets -- the latter two
  now fixed, see below).
- **Manual change -- plant-name spelling fixes**: `Generator Data` has `"HPP_Ugar_Usce "`
  (trailing space) and `"HPP_Rogacica"`; `REProfile` has `"HPP_Ugar_Usce"` and
  `"HPP_Rogačica"`. Same plants -- a new per-region `generator_renames` setting in
  `scripts/convert_epm_open.py` maps both sheets to the clean ASCII spelling
  (`HPP_Ugar_Usce`, `HPP_Rogacica`), so both now match their own profiles.
- **`HPP_JajceII`'s profile is missing exactly one representative day** (Feb `Weekday`; every
  other month/day-type is present in `REProfile`), so the whole profile is dropped for
  incomplete time coverage and the plant (30 MW existing ROR, ~2.7% of existing ROR capacity)
  would end up non-RE with no profile. **Manual change**: filled the missing Feb `Weekday` day
  with a copy of JajceII's own Feb `Weekend` row (`profile_fills` in
  `scripts/convert_epm_open.py`) -- hydro output tracks season, not day-type (its Weekday and
  Weekend values are within a few % in every other month). All 146 PV/wind/ROR generators now
  have a profile.
- **General fix, not Bosnia-specific**: `<name>_expansion` build options split off from an
  existing plant (`_split_existing_and_expansion` in `inputs/epm2seer/generator_params.py`)
  previously got no generator-specific profile (lookup was by their own `_expansion` name), so
  every PV/wind/ROR expansion option came out `isre=False` with no profile (118 here). They now
  resolve to their parent plant's profile (and so `isre`). Every region's seer output needs
  regenerating to pick this up.
- **General fix, not Bosnia-specific -- `Candidate` capacity**: a `Status=Candidate` plant's
  `Capacity` is the size of the project/unit that could be built, not installed capacity. It
  was previously kept as `Capacity`, so every candidate (all 163 here have `Capacity =
  BuildLimitperYear = MaxTotalBuild`) was split into a phantom "existing" plant plus a
  `_expansion` build option -- 82.6 GW of fake existing capacity against ~4.2 GW real.
  `scripts/convert_epm_open.py` now moves a candidate's `Capacity` to `UnitSize` (unless one
  is given; only used under modular expansion) and clears it, so each candidate is one pure
  build option. Where `MaxTotalBuild` is missing or 0, the candidate's `Capacity` (project
  size) becomes its `MaxTotalBuild`, so a single-project candidate can't be built without
  bound (e.g. Bosnia's 16 small ROR candidates with `BuildLimitperYear = MaxTotalBuild = 0`).
  Storage plants are exempt (their MW `Capacity` is needed for `duration_hours`, and
  `storage_params.py` already handles candidates correctly). `MaxTotalBuild` now flows through
  to the new schema column `total_build_limit` (enforced in both pyomo and PyPSA), and
  `annual_build_limit` is only set where `BuildLimitperYear` is explicitly nonzero (no longer
  back-filled from `MaxTotalBuild` or `inf`). Every region needs regenerating to pick this up.
- **General fix, not Bosnia-specific -- reservoir hydro (`Type="STO HY"`)**: these were
  labeled `PHS` and had no profile or availability limit at all -- an unlimited zero-cost
  generator (Bosnia: 1.5 GW running flat out all year, pushing coal to 0% of dispatch). Now:
  (1) relabeled `hydro` (bare `STO HY`, `STO HY-Water`, `STO HY-Hydro` in
  `inputs/epm2seer/data/pypsa_tech_names.csv`; `STO HY-Pumped` stays `PHS`; storage-sheet plants
  are still mapped to `PHS` by `storage_params.py`), and (2) given a generator-specific profile
  from the `GenAvailability` sheet (`_availability_profiles` in `scripts/convert_epm_open.py`):
  each season's availability factor held flat over every modeled hour of that season, unless
  the plant already has a real REProfile/REgenProfile profile. Bosnia's values checked: spring
  snowmelt peak (Mar 0.6-0.8), summer/autumn low (0.1-0.3), Visegrad ~0.39 average (real
  output ~1 TWh/yr from 315 MW); all 26 plants matched. Approximation: EPM applies availability
  as a seasonal energy (capacity-factor) limit, so hydro can't shift energy to peak hours here.
  `GenAvailability` also has (unused) monthly derates for thermal plants (e.g. coal 0.15-0.87).
- **General fix -- capex units**: EPM's generator `Capex` is total capital cost in $M/MW and
  storage `CapexMWh` is $/kWh (150-400 in every region); seer's capex is total $/MW (or $/MWh)
  incurred once in the build year. Converted (x1e6, x1e3) in `inputs/epm2seer/`; the
  generator-side `Capex` fallback for storage ($M/MW) is converted via `duration_hours`.
  Previously capex was ~1e6x too cheap relative to marginal cost ($/MWh). Burkina Faso has
  storage `CapexMWh` of 1 and 4 -- checked, see its section. Fixed O&M
  (`FOMperMW`) is still not carried into the schema at all.
- **General fix -- network rows**: `pTransferLimit` lists each interconnector once per direction
  (Bosnia: BiH->Croatia and Croatia->BiH, ...), and seer lines are undirected, so every line's
  capacity was effectively doubled. `inputs/epm2seer/network_params.py` now keeps one row per
  zone pair (warning if the two directions disagree); Bosnia goes from 6 rows to 3 (both
  directions were identical). `network.csv` also carries optional expansion columns
  (`expandable`, `capex`, `start_year`, `unit_size`, `total_build_limit`) from
  `pNewTransmission` where a region has one. Bosnia's own expansion table (the "ZONES:
  transmission" block in `ZoneData`) is all disabling placeholders (`EarliestEntry=2060`,
  `CapacityPerLine=1e6`, `CostPerLine=1e14`) and isn't read -- no line expansion here.
- **General fix -- ramp rates**: ramp-down defaulted to `-1` (now `1`, both rates are positive
  fractions of capacity per hour). Bosnia's raw ramp rates are mostly above 1 (5.0 for
  hydro/gas, 0.54-3.0 for coal, labeled "% of Gen") -- read as a fraction per hour (as EPM's own
  constraint uses it) and capped at 1, since anything >= 1 can't bind. Only slower coal units
  (0.36-0.9) end up with a real limit.
- **General fix -- plant-name whitespace**: plant names are normalized on every sheet they're
  read from (`clean_plant_name` in `scripts/convert_epm_open.py`: strip both ends, drop spaces
  around underscores, collapse runs of spaces), so they still match across sheets. Bosnia:
  `"PHP_Vrilo "` -> `PHP_Vrilo`, `"PHP_ Buk_Bijela"` -> `PHP_Buk_Bijela`, `"HPP_ Mokronoge"` ->
  `HPP_Mokronoge`, plus several trailing-space plants (`"HPP_Ugar_Usce "`, `"Wind_Kupres "`, ...)
  -- which also replaces the earlier per-plant `HPP_Ugar_Usce` rename.
- **General fix -- time weights and ordering**: `times.csv` now carries each representative
  day's `weight` from EPM's demand profile (Bosnia: 1 for each month's `Peakday`, 27-30 for its
  `Weekday`, summing to 8760 h); both models scale the weights to a full year. Previously every
  hour was weighted equally, overweighting peak days ~15x (Bosnia's annual demand came out
  11,469 GWh vs. the 11,042 GWh forecast; now 11,059). `demand_profile.csv` and
  `renewable_profiles.csv` are sorted month -> day -> hour. Storage now balances within each day
  in both models (otherwise it could charge on a weight-1 day and discharge on a weight-30 day,
  creating energy).
- **Schema cleanups**: all floats rounded to 3 decimals on write; cost types renamed `capex` ->
  `capital`, `marginal_cost` -> `marginal` (and `network.csv`'s `capex` column -> `capital`).
- **General fix -- storage build limits**: storage `BuildLimitperYear`/`MaxTotalBuild`/`UnitSize`
  come from the plant's Generator Data row in MW, but were written as MWh -- now converted via
  `duration_hours` (Bosnia: `PHP_Buk_Bijela` 600 MW -> 3,600 MWh, matching the Storage sheet).
  Storage now also has `total_build_limit_mwh` (from `MaxTotalBuild`; a candidate without one
  gets its project size), enforced in both models, so single-project storage can't be rebuilt
  every year. Build limits are cleared for existing storage, and the annual limit is only kept
  where given.
- **Manual change -- yearly PV/wind build options collapsed**: the workbook has one generic
  build option per year per tech (`PV_2020`..`PV_2050`, `Wind_2020`..`Wind_2050`), each with
  that year's capex, identical profiles, 500 MW/yr and 1,000 MW total each -- an EPM workaround
  for declining capex (`PV_2020`..`PV_2023`/`Wind_2020`..`Wind_2023` are disabled, `StYr` 2050).
  Replaced by one option per tech, `PV_New` / `Wind_New` (`collapse_yearly_options` in
  `scripts/convert_epm_open.py`): start 2024, 500 MW/yr, no total limit, the shared profile,
  and a capital trajectory taking year y's cost from `PV_y`/`Wind_y` (PV $694k/MW in 2024 ->
  $376k/MW in 2040; wind $1.05M -> $789k), written to `pCapexTrajectoriesCustom.csv` (all other
  build options stay flat). Not exactly equivalent: the original could exceed 500 MW in a year
  by also building older (pricier) yearly options; the collapsed option caps each year at
  500 MW at that year's price. 191 -> 131 generators.
- **Manual change -- BiH-only model**: `Croatia`/`Serbia`/`Montenegro` are EPM external zones
  (`ZoneExternal` in `ZoneData`) with no generators or demand -- EPM trades with them at hourly
  `TradePrices` (interconnected mode, price-based exports; imports capped at 40% of demand,
  exports at 50%, via `ImportShare`/`ExportShare`). seer has no trade, so they were inert (their
  lines could never carry flow). Dropped (`drop_zones` in `scripts/convert_epm_open.py`), along
  with their lines: Bosnia is now a single-zone model with an empty `network.csv`. **Known gap:**
  no imports/exports, so the model is an isolated BiH system, unlike EPM's.
- **`season`/`day` here use text labels, not `"m1"`/`"d1"` style**: months are spelled out
  (`"Jan"`..`"Dec"`) and day-types are named (`"Peakday"`/`"Weekday"`/`"Weekend"`) --
  needed both a `season_map` and a `day_map`. `REProfile` has all 3 day-types but `Demand`
  itself only ever uses 2 (no `"Weekend"` row) -- this **initially made every single profile
  fail time-coverage and get dropped** (a profile's row count no longer matched `times.csv`'s
  actual size), until the fix below.
- **New general robustness fix, not Bosnia-specific**: `REProfile`/`REgenProfile` rows are now
  filtered to only the `(season, day)` combinations the demand-defining sheet (`Demand` or
  `DemandProfile`) actually uses, before being written out -- so a finer-grained profile sheet
  referencing times the model calendar never includes doesn't collapse every profile's
  time-coverage check.
- `Type="GT"` has no tech synonym -- renamed (`CCGT`/`GT`+`"GasZenica"` otherwise resolve fine,
  since bare `CCGT`/`OCGT` match regardless of fuel). `fuel1` is plant/mine-specific for coal
  (`"CoalTuzla"`, `"CoalGacko"`, ...) and gas (`"GasZenica"`) -- deliberately **not** folded to
  generic `"Coal"`/`"Gas"`, same reasoning as Sudan's Coke: `Fuel Prices` prices each mine/plant
  separately, so renaming would silently merge genuinely different fuel prices. `tech` already
  resolves cleanly regardless (`Type="COAL"`/`"CCGT"` match bare, independent of fuel), so this
  is a fuel-column cosmetic gap only. `"PumpedHydro"`/`"Batteries"` *are* pure spelling variants
  (no real distinct pricing behind them) -- folded to `"Pumped"`/`"Battery"` so the storage tech
  compound match (`"Storage-Pumped"`/`"Storage-Battery"`) resolves correctly.
- Only 3 of 33 Existing/Committed plants have the `BuildLimitperYear`/`MaxTotalBuild`-
  duplicates-`Capacity` artifact.
- Demand uses the "detailed" formulation (`Demand`+`Duration`), like Lao/Morocco/Georgia.
  Capex trajectories disabled.
- **Manual change -- `settings.csv` gets `voll,1500`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $1,500/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
