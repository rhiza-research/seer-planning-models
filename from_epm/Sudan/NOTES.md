# Sudan -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Sudan`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Sudan.xlsx` (49 generators, single zone "Sudan"; storage disabled).

- **`Generator Data`'s own `Zone` column is `"SUDAN"` (uppercase), while every other sheet
  (`ZoneData`, `Demand_Forecast`, `DemandProfile`, `REProfile`, `Fuel Prices`) uses `"Sudan"`**
  -- an isolated casing inconsistency in that one column, not a real second zone. Left
  unfixed, every generator's zone fails to match `zcmap.csv` and the whole region comes out
  with 0 generators. Renamed via a `zone_renames` config (new -- the first region needing one).
- `season`/`day` in `DemandProfile`/`REProfile` are plain integers (`1`/`2`), not `"Q1"`/`"d1"`
  style labels -- needed both a `season_map` and (new) a `day_map` to relabel them; previously
  every region's raw `day` values were already `"d1"`-style, so no region had needed a
  `day_map` before this one.
- Stray leading/trailing whitespace on raw values (`fuel1="Diesel "`/`"LFO "`, `Fuel Prices`'
  own `Zone="Sudan "`) -- fixed generically (`.str.strip()` on `Generator Data`'s `Type`/
  `fuel1` and `Fuel Prices`' `Fuel`/`Zone`, applied to every region now, not just Sudan).
- `Type="GT"` has no tech synonym -- renamed. The sole Coke-fired unit (`Type="ST"`,
  `fuel1="Coke"`) is deliberately **not** folded to `"Coal"` like Morocco's `CoalCCS` fix --
  `Fuel Prices` genuinely prices Coke separately from Coal here (a real distinct commodity,
  confirmed by checking its own fuel list), so renaming would silently merge two different
  fuel prices. Left as raw `tech="ST"` (no `"ST-Coke"` compound exists) -- a cosmetic gap, not
  a functional one.
- `REProfile` has real Wind/Solar capacity-factor data, but **Sudan's `Generator Data` has no
  wind or solar generators at all** (confirmed directly -- 49 generators, all thermal/hydro) --
  so `renewable_profiles.csv` comes out empty, correctly (nothing to match), not a bug.
- Demand uses the "less detailed" (forecast) formulation. Capex trajectories disabled (the
  workbook's own "ignored" banner text is mislabeled as "ENERGY EFFICIENCY TARGETS" instead of
  capex -- a copy-paste error in the source file, but the ignore-flag meaning still holds).

**Finalized** (2026-10-26) -- seer dataset in `data/seer/final/epm_open/Sudan` (results in
`data/seer/results/Sudan`, 2026-2030; records in `data/reports/country_report.pdf`).
One zone, 2019-2030, 2 seasons x 2 day types (`Duration` 129/52/132/52 -> 8760 h). 3,479 MW
existing in 2019 (1,614 MW hydro, 1,126 MW oil, 410 MW CCGT, 220 MW OCGT, 110 MW coke), 3
build options (991 MW in all), no storage (`includeStorage` 0, `Storage` sheet empty), no lines.
Changes in `REGIONS["Sudan"]` plus **two seer-folder hand edits (below) -- regenerating
overwrites them, re-apply both**.
- **Regenerated 2026-10-26** with the current pipeline; validates.
- **General fix -- integer `GenAvailability` season headers** (`_availability_profiles`,
  `scripts/convert_epm_open.py`): the header row's `1`/`2` were read as floats and turned into
  the strings `"1.0"`/`"2.0"`, which never matched the integer `season_map` -- every hydro
  plant came out with no availability profile and ran unlimited at 1.9 GW for free. Integer
  headers are now kept as ints (and `_strip_if_str` only strips actual strings). Only Sudan and
  Sierra Leone (same 1/2 seasons) change; every other region's processed csvs are identical.
- Hydro availability (`GenAvailability`, flat in both seasons): Merowe, Jebel Aulia, Kajbar
  (build option) 0.85; Rumela, Burdana, Sennar 0.35.
- **Manual change -- thermal derate** (seer hand edit, `generator_constants.csv`): five fossil
  units have `GenAvailability` 0.35 in both seasons, applied as a fixed capacity derate:
  KILO-X DIT_ICE_HFO 260 -> 91 MW, KAREMA_ICE_HFO 10 -> 3.5, NYALA_ICE_Diesel 7.4 -> 2.59,
  KADUGLI_ICE_Diesel 2 -> 0.7, TENDALTI_ICE_HFO 2 -> 0.7. Everything else is 0.85 (left as-is).
- **Manual change -- `settings.csv` gets `voll,500`** (seer hand edit): `Settings1` VOLL, $500/MWh.
- `zones.csv` country is already `Sudan` (no edit needed).
- Retirements are EPM's own `RetrYr` (life = RetrYr - StYr + 1): Roseires (280 MW, 2011),
  Khasm El Girba (2004) and Sennar (2002) hydro are retired before the horizon, though they
  operate in reality; Mahmoud Sharif ST (380 MW) retires 2025, plus several small diesels.
  Kept as EPM has them.
- Workbook quirks kept as-is: `EL FAO_ICE_Coal` (build option) and `WADI HALFA_ICE_Diesel` both
  burn HFO per `fuel1`; `EL GAILI_ST_Coal` burns Coke ($2.76/MMBtu, comes out `coal`/`coal` but
  priced at Coke). Fuel prices are already $/MMBtu. VOM not included (every plant is priced
  through `Fuel Prices`).
- Build options: ARKYIAI coal 600 MW ($3.95M/MW), Kajbar hydro 360 MW ($2.0M/MW), El Fao HFO
  31 MW ($1.5M/MW), all from 2020, each limited to its project size.
- Demand: peak 4,250 MW (2019) -> 7,000 MW (2030); weighted load factor 0.611, peak x profile
  within 0.5% of EPM's `Energy` (22.7 vs 22.9 TWh in 2019).
- Not modeled: 10% planning reserve, spinning reserve, min-gen constraints (`Settings1`). No
  fuel limits, trade prices, transfer limits or import share are filled in.
- **Results (2026-2030, VOLL $500)**: pyomo and PyPSA both $23,708.0M upfront, $21,898.2M
  annualized. Both runs build every option in 2026 (600 MW coal, 360 MW hydro, 31 MW HFO) and
  still shed 35.0 TWh: 5.2 TWh (16% of demand) in 2026 rising to 8.9 TWh (24%) in 2030, in
  about half of all modeled hours -- all of season 1 and season 2's peaks. Available capacity
  is ~3.6 GW against a 6.0-7.0 GW peak, so this is the workbook's own shortfall (only 991 MW of
  build options), not a data problem. Generation 138.7 TWh: hydro 66.2, coal 30.3, oil 21.2,
  CCGT 15.4, OCGT 5.7. Dispatch is near-identical year to year: with all three options online
  from the window's first year, capacity is flat and demand growth all goes to lost load.
- The three candidates are the workbook's only build options (`Generator Data` and the
  GAMS-read `GenData` list the same 49 plants; `Storage`/`CSP` empty). A 2019-2030 pyomo solve
  builds all three in 2020 instead -- same capacity by 2026 ($23,792.6M). EPM's own run has the
  same options; its 10% planning reserve (not modeled here) can't be met either.
