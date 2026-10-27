# Egypt -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Egypt`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Egypt.xlsx` (84 generators + 1 real storage plant, single zone
"EGYPT").

- `Type="GT"` has no tech synonym at all (only `"OCGT"` is recognized) -- renamed.
- 7 of 74 Existing/Committed plants have the `BuildLimitperYear`/`MaxTotalBuild`-duplicates-
  `Capacity` artifact.
- Storage is enabled and real (same convention as Maldives/Iraq/Poland/Morocco).
- Demand uses the "less detailed" (forecast) formulation, `Q1..Q4` seasons. Capex trajectories
  disabled.

**Finalized** -- seer dataset in `data/seer/final/epm_open/Egypt` (results in
`data/seer/results/Egypt`, 2022-2026: pyomo $46,625M vs PyPSA $46,633M; annualized capex in
`data/seer/results/Egypt/annualized`, $43.5B; records in `data/reports/country_report.pdf`).
**Has a hand edit** (`demand_profile.csv`, below) -- regenerating overwrites it, re-apply it.
- **Regenerated 2026-10-19** with the current pipeline. Fuel prices are already $/MMBtu in
  `Fuel Prices` and match the `FuelPrices` sheet (2022: gas $5, LNG $8.9, fuel oil $11.8, coal
  $4.3, uranium $0.37/MMBtu; prices start 2019, 2018 held at 2019). Marginal costs (2022): CCGT
  $29-56/MWh, gas OCGT $47-80, oil OCGT $111-186, oil steam $110, coal $38-41, nuclear $3.9.
  Capex (build options): PV $0.97M, wind $1.49M, CCGT $1.22M, OCGT $0.81M, coal $2.9M, HFO
  steam $2.5M, nuclear $5.5M, CSP $4.4M, hydro $2.0M/MW, battery $200k/MWh. `Duration`
  weights sum to 8760 h. Existing + committed fleet 62 GW (38.9 GW in 2018).
- **Manual change -- demand profile reshaped (hand edit of `demand_profile.csv`)**: peak x the
  workbook's `DemandProfile` gave only ~60% of EPM's own `Energy` forecast (2018: 113 vs
  186 TWh; load factor 0.43 vs 0.71). The profile is 0.23-0.5 outside Q2 -- implausibly peaky
  for Egypt. Reshaped to `p' = 1 - k(1 - p)`, k = 0.486: the peak hour stays at 1.0 and the
  weighted load factor becomes 0.724, the mean of EPM's energy/(peak x 8760) over 2018-2035
  (0.713 rising to 0.736), so energy is within +-1.6% of EPM's every year (2022: 269.8 vs
  267.3 TWh). Flattens the seasonal swing (min 0.23 -> 0.63). Original values:
  `p = 1 - (1 - p')/0.486`. The workbook's Q2 (not Q3) being the peak season is left as-is.
- **Manual change -- ROR availability** (`availability_profile_types: ["STO HY", "ROR"]`):
  `HYD_17_EGY` (32 MW, Type `ROR`) has no REProfile and came out as unlimited free generation;
  now gets its GenAvailability 0.55 like the `STO HY` plants. (The REProfile `Water` profile
  is also a flat 0.55, so hydro is the same either way.) Wind, PV and CSP use their REProfiles.
- Single zone, as in the workbook (`ZoneData`, `MapCountry`, `Topology` list only EGYPT; no
  zones dropped). `TransferLimit` has lines to Jordan, Libya, Palestine, Saudi Arabia and
  Sudan, but those aren't zones anywhere in the workbook, so `jobs.epm_to_seer` drops the rows
  (warned). `Settings1` has `interconMode=1`/`allowExports=1` and there are `TradePrices` and
  a 10% `ImportShare`, but with no neighbor zones defined EPM likely didn't trade either
  (unverified). Price-based trade isn't modeled anyway (known open item). No import generators in
  `Generator Data` either (`Import` fuel is "Included: NO") -- trade deliberately left out.
- Left as-is: steam-turbine gas plants (`ST_*`, fuel Gas) come out tech `CCGT` (the global
  `ST-Gas` synonym; label only, heat rates are per plant). Build options have placeholder
  limits (6,000 MW/yr, 100,000 MW total; hydro 1,000 MW). EPM retires the Aswan High Dam
  (2,100 MW) and Aswan I (280 MW) after 2025 (`RetrYr` 2025) -- kept. Committed wind is
  4,010 MW from 2020 (well above what Egypt has actually built). GenAvailability thermal
  derates (0.55 on `CC_Dami_93`, `GT_SoutI_89`, `ST_Tebe_10`) are ignored, as everywhere.
  Plant names contain a literal "…" from the workbook. VOM not included. Not modeled (on in
  `Settings1`): min-gen and ramp constraints.
- Results (2022-2026): no lost load; CCGT ~86% of generation, coal ~4%, hydro ~4%, wind ~5%,
  PV ~1%. Upfront capex builds only 2.3 GW of OCGT (2026) and 2,500 MWh of batteries (the
  500 MWh/yr limit), which cycle for the peak. Annualized, both models build 18 GW of wind
  (2024-2026) and 6 GW of PV (2026) -- both at the placeholder 6,000 MW/yr limit, so the
  limit, not cost, sets the RE build -- plus the 1 GW hydro option (its total limit) and
  5.5 GW of CCGT, and no batteries. Models agree to within 0.02% on objective and dispatch.
- **Manual change -- `settings.csv` gets `voll,500`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $500/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
