# Namibia -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Namibia`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Namibia.xlsx` (23 generators + 7 real storage plants, 3 zones:
`Windhoek`/`Aus`/`Kokerboom`, all real with generators -- no pure interconnection zones here).
Detailed demand (`Demand`+`Duration`, `q1..q4` seasons). Capex disabled (same mislabeled
banner as Djibouti/Sierra Leone).
- One hybrid solar+storage project is split into two unrelated generator rows with no link
  between them in the data (`New_Solar_wStorage_Nam`, `Type="PVwSTO"`, 10 MW;
  `New_Storage_wPV_Nam`, `Type="STOPV"`/`fuel="Battery"`, 1 MW) -- renamed to plain `PV`/
  `STORAGE` and modeled as two ordinary, independent generators (user-confirmed; seer's schema
  has no co-location concept to capture the physical link anyway).
- Three distinct named import sources (`ESKOMimport` = South Africa's Eskom, `ZESCOImport` =
  Zambia's ZESCO, `ZPCImport` = Zimbabwe's ZPC) -- folded to the same generic `"Import"`
  category as every other region (user-confirmed; still distinguishable by generator
  name/zone). This needed a small code generalization: `import_fuel_value` now accepts a list,
  not just one string.
- A handful of storage plants (`FP_Battery_Nam`, `FP_Storage_wPV_Nam`, `FP_Storage_wWind`) have
  no matching `Generator Data` row at all -- excluded correctly, same as any region's unused
  template/placeholder storage entries.

**Finalized** (2026-10-22) -- seer dataset in `data/seer/final/epm_open/Namibia` (results in
`data/seer/results/Namibia`, 2026-2030, and `.../annualized`; records in
`data/reports/country_report.pdf`). Three zones (only `Windhoek`
has demand), two lines plus two expandable corridors, no trade with neighbours beyond the
import contracts. Changes in `REGIONS["Namibia"]` plus **three processed-csv edits and four
seer-folder hand edits (below) -- regenerating overwrites them, re-apply all seven** (the
processed edits after `convert_epm_open.py`, before `jobs.epm_to_seer`).
- **Regenerated 2026-10-21** with the current pipeline; validates. Years 2020-2040. Fuel prices
  now converted with `FuelTechnologies` (coal $/ton -> $3.6/MMBtu, ~$36/MWh at Van Eck) and
  match the `FuelPrices` sheet. `includeVOM=0`, so VOM is left out.
- **General fix -- `GenAvailability` header**: the workbook labels only `q1`/`q2` in the header
  row (D/E hold the q3/q4 values unlabeled, H-J stray notes), which crashed the converter. The
  header is now padded, and `availability_seasons` names the season columns (Namibia only).
- **Manual change -- only seasons q1/q2 modeled** (`season_map`, and a general fix: Demand and
  Duration rows whose season isn't in `season_map` are dropped): `LoadDefinition` defines only
  q1 and q2, whose `Duration` sums to 185 + 180 = 365 days. The q3 Demand/TransferLimit and q3/q4
  Duration/REProfile rows are leftovers (all four summed to 584 days, 14,016 h). Now 2 seasons x
  3 days, 8760 h. Load factor ~0.67 (2020: 644 MW peak, 3.8 TWh).
- **Manual change -- `zones.csv` country `Namibia`** (seer hand edit; `MapCountry` gave each
  zone its own name).
- **Manual change -- `settings.csv` gets `voll,500`** (seer hand edit): `Settings1` VOLL, $500/MWh.
- **Manual change -- `Anixas_Nam_IC` derated to 15.75 MW** (seer hand edit of
  `generator_constants.csv`): the existing diesel ICE has `GenAvailability` 0.7 (22.5 x 0.7). The
  other existing/committed fossil is >= 0.8 (Van Eck 0.9, Kudu 0.9) -- left as-is.
- **Manual change -- `GCoal_Nam_CO` derated** (seer hand edit of `generator_constants.csv` and
  `generator_costs.csv`, 2026-10-22): greenfield coal option (1 GW, $2.6M/MW, ~$30/MWh) with
  `GenAvailability` 0.25/0.2 -- at nameplate it would be the cheapest firm baseload. As a build
  option it has no capacity to derate, so its build is scaled by the Duration-weighted
  availability 0.225 instead (as a capacity derate, like Anixas): `unit_size` 50 -> 11.25,
  `annual_build_limit` 500 -> 112.5, `total_build_limit` 1000 -> 225 MW, capital $2.6M/MW /
  0.225 = $11.56M/MW, so the plant's total cost is unchanged. It stays non-RE (no profile). Never built.
- **Manual change -- imports priced at their own tariffs** (`marginal_cost_from_tariff`, all
  seven import units): folding `ZESCOImport` and `ZPCImport` to `Import` gave two `Windhoek,Import`
  Fuel Prices rows, and ZPC took ZESCO's price ($56 vs $60/MWh). HeatRate is 3.41 throughout,
  so the tariffs are otherwise unchanged (2026: ESKOM $62.5, ZESCO $56.9, ZPC $61.8).
- **Manual change -- import contract renewals as fixed capacity** (processed edit of
  `pGenDataInputCustom.csv`: added `Capacity` rows ESKOM-2 200 / ZESCO-2 50 / ZPC-2 80 MW, removed
  their `Capex`/`BuildLimitperYear`/`MaxTotalBuild` rows, and moved their `StYr` a year later:
  ESKOM-2 2024, ZPC-2 2027, ZESCO-2 2032). Every existing import ends: ESKOM (200 MW) and
  ESKOM-PEAK (300 MW) in 2023, ZPC (80) in 2026, ZESCO (50) in 2031. The renewals are EPM
  candidates at $10k/MW, which the pipeline drops (imports are never build options). At that
  capex EPM would always take them, so they come in as existing. The workbook starts each
  renewal in its predecessor's `RetrYr`, which the pipeline counts as an active year, so both
  contracts would run in the handover year (ZPC + ZPC-2 = 160 MW in 2026) -- hence the shifted
  `StYr`, a straight handover. ESKOM-PEAK has no renewal. Not modeled: `MaxImports` (0.9) / `ImportShare` limits, and
  price-based trade (`allowExports=1`, `TradePrices`).
- **Manual change -- batteries 4 h** (processed edit of `pStorageDataInput.csv`): every real
  battery's `Storage` sheet `Capacity` is the 1,000,000 MWh placeholder (1000 h at Generator
  Data's 1000 MW). Set to 4,000 MWh (**assumed 4 h**, as Botswana): units 40 MWh
  (`New_Storage_wPV_Nam` 80), 4,000 MWh/yr, 40,000 MWh total. Capex is the `Storage` sheet's
  $/kWh ($250k/MWh 2022 options, $150k/MWh 2025, $200k/MWh with-PV); Generator Data's per-MW
  capex ($0.15-0.18M/MW) is not added. Life 8 (the `*2` options' Generator Data says 12).
- **Manual change -- line expansion** (processed `pNewTransmission.csv`, new, from `ZoneData`'s
  expansion block; `pAllowHighTransfer=1`): Aus-Kokerboom 400 MW/line, $81M, Kokerboom-Windhoek
  500 MW/line, $118M, up to 5 lines each, from 2023 (the workbook's CostPerLine is in $, the
  file takes $M). Existing lines 100 / 600 MW (flat).
- Existing/committed in 2020: 1,106 MW -- imports 630 MW, Ruacana hydro 300 MW (0.34/0.34),
  PV 125 MW, Van Eck coal 30 MW, Anixas 16 MW (derated), wind 5 MW. Build options: Baynes hydro
  (300 MW, 0.65, $2.3M/MW), Kudu CCGT (450 MW, $2.15M/MW), PV ($0.65-0.8M/MW), wind ($1.8M/MW,
  1 GW each at Aus/Kokerboom), CSP, HFO, biomass (40 MW), GCoal, 7 batteries. Left as-is: a
  one-hour dip in the q1 d1 demand profile (0.64 -> 0.33, weight 20 h); Kokerboom wind pinned at
  1.0 in places; CSP ~1.0 by day.
- **Results (2026-2030, VOLL $500)**: upfront pyomo $967.2M, PyPSA $987.1M (+2.1%), all battery
  losses (PyPSA with efficiency 1: $967.4M). Upfront builds ~230-300 MW PV, ~50 MW wind,
  ~400-520 MWh battery (PyPSA also 42 MW Baynes); lost load 2-11 GWh/yr (~40 evening-peak
  hours, up to 350 MW in 2030), economic at $500/MWh. Annualized $479.5M in both, with no lost
  load: Baynes 300 MW, wind 128-168 MW, PV 84-130 MW, Aus-Kokerboom expanded to 120-156 MW, no
  storage. Dispatch (upfront 2026): imports 1.7 TWh (~42%), PV 1.0 TWh, hydro 0.9 TWh, wind 0.3,
  coal 0.26; annualized: hydro 2.5 TWh, wind 0.6, PV 0.5, imports 0.4, coal 0.17. GCoal, Kudu,
  CSP, HFO and biomass are never built.
