# Sao Tome and Principe -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Sao Tome and Principe`). Paths below refer to the seer repo.

Source: `data/raw_epm_open/Sao Tome and Principe.xlsx` (25 generators, 2 zones; forecast
demand, one season/day). **Finalized** (2026-10-28) -- seer dataset in
`data/seer/final/epm_open/Sao Tome and Principe` (results in `data/seer/results/Sao Tome and
Principe`, 2022-2026, and `.../annualized`; records in `data/reports/country_report.pdf`). Two islands, no lines between them (none in the workbook), 2020-2035,
one representative day weighted 365 (-> 8760 h). Existing: São Tomé 14.6 MW of diesel (`ICE`/`LCO`;
BoboForo1/2 are 0 MW in the workbook) + 1.5 MW `Contador` ROR (+2.5 MW `ContadorRehab`
committed 2021); Príncipe 1.65 MW of diesel -- against peaks of 30.5 / 2.5 MW in 2022 (25 /
2.3 MW in 2020; the first build options start in 2022, so 2020-21 can't be met). 11 generator build options
(3 diesel, 7 PV/floating PV/PV+storage, 1 ROR; the 2 DSM options dropped, below) plus two battery
options (`BESS-ST`/`BESS-P`, 4 h/8 h, $150k/$125k per MWh, 85%). Changes in `REGIONS` plus
**four seer-folder hand edits (below) -- regenerating overwrites them, re-apply all**.
- No lines: the islands are ~140 km apart with no link; `Topology` marks none and
  `TransferLimit` is empty. `ZoneData`'s "new transmission lines" block (400 MW / $81M and
  500 MW / $118M lines from 2023) is a Namibia-template leftover (Aus-Kokerboom /
  Kokerboom-Windhoek, with zone indices in `To`) -- not read by the converter, ignored.
- Fuel Prices are keyed by zone (SaoTome/Principe), not by the `SaoTome_Principe` country --
  the converter now handles zone-keyed prices (general fix). LCO $/ton -> ~$15/MMBtu -> diesel
  $184-198/MWh.
- `PVwSTO` -> `PV` (as in Burkina): `PVwStorageST`/`PVwStorageP` are plain PV (their storage
  isn't modeled -- EPM doesn't link any either: `Storage`'s `LinkedPlant` is blank). `REProfile`'s `Hydro` rows are the ROR plants' -- `re_profile_tech_renames`
  `Hydro` -> `ROR` (0.4-0.5, matching their GenAvailability 0.5).
- Diesel GenAvailability is 0.85 -- no derate (BoboForo's 0.2 is on 0 MW).
- Demand: peak x profile is 3.9% above EPM's `Energy` in every zone and year -- left as-is.
- EPM has `includeVOM` 0 -- `include_vom` not set.
- **Manual change -- `zones.csv` country `Sao Tome and Principe`** (seer hand edit; was the
  `MapCountry` placeholder `SaoTome_Principe`).
- **Manual change -- diesel engines labeled tech/fuel `oil`** (seer hand edit,
  `generator_constants.csv`, 11 rows: the 8 existing engines and `CandidateDieselST`,
  `CandidateDieselP`, `LCP_ICE`). The workbook types them `ICE` / `LCO` (a light oil), which the
  standard-name tables don't know (they map `ICE-Diesel`/`ICE-HFO` and `LFO` to `oil`), so they
  came out raw. PyPSA technology-data has one petroleum entry, `oil` (DEA diesel engine farm),
  as every other finalized country uses. Label only -- costs still come from the `LCO` price
  ($184-198/MWh).
- **Manual change -- DSM options dropped** (`drop_generators: ["DSM-ST", "DSM-P"]` in
  `REGIONS`; user's call, 2026-10-27): two 4 MW household demand-side-management options (Type
  `PVwSTO`, fuel `DSM-HH`, $1.75M/MW capex only, 5-year life). In EPM they're zero-cost
  "generators" on `REProfile`'s `DSM-HH` shape (matched by fuel): 0.9 at 18-20h, 0.64 at
  05h/17h/21h, 0.13 at night, 0 by day. `PVwSTO` -> `PV` turned them into solar with the solar
  profile; carrying the DSM shape instead would need `isre=True` on a non-renewable, so they're
  dropped. The `DSM-PB` rows (São Tomé; 0.9 by day) belong to no plant either.
- **Manual change -- `PVwStorageST`/`PVwStorageP` capital $2.0M -> $1.7M/MW** (seer hand edit,
  `generator_costs.csv`, every year; user's call, 2026-10-27): they're plain PV here, so they
  shouldn't pay for storage they don't get. $1.7M is `SolarST1`/`SolarP`'s plain-PV capex. The
  $0.3M/MW premium would buy ~2 MWh at `BESS-ST`'s $150k/MWh, i.e. ~2 h storage -- that
  storage now comes from the zones' own battery options (`BESS-ST` São Tomé, `BESS-P`
  Príncipe, 400 MWh each). (Their FOM, $36k vs $20k/MW-yr, isn't modeled anyway.)
- **Manual change -- `settings.csv` gets `voll,500`** (Settings1 VOLL). **Re-apply after regenerating.**
- Kept as EPM has them: `SolarST1` has BuildLimitperYear 80 / MaxTotalBuild 16, where its
  siblings have 16 / 80 (looks swapped); `FloatingSolarST2` is in zone Principe; unit sizes
  above the build limits (`HydroCandidate` 3.2 MW vs 2.5 MW total, `PVwStorageST` 80 MW vs 16
  MW/yr) don't bind, because builds are continuous (as for every country).
- Not modeled: 5% planning reserve margin, FOM (diesel $80k/MW-yr).
- **Results (2022-2026, VOLL $500; storage efficiency as a round trip on charging in both
  models, DSM dropped)**: pyomo and PyPSA identical -- $178.44M upfront, $153.60M annualized.
  Upfront: generation 1,181 GWh -- diesel 1,096, ROR 85; builds 12.2 MW of diesel on São Tomé
  and 1 MW on Príncipe, a 23.9 MWh battery on São Tomé, no PV, and sheds 3.3 GWh at the
  evening peaks in 2025-26 (no PV is built upfront, so the PV repricing doesn't matter here).
  Annualized: no lost load; PV 65.3 MW on São Tomé (`SolarST1`'s 16 MW plus `PVwStorageST`),
  5.2 MW on Príncipe; batteries 39 MWh on São Tomé, 6.7 on Príncipe; ROR `HydroCandidate`
  2.5 MW (2025). Generation: diesel 684 GWh, PV 400, ROR 106, storage losses 10. (With DSM
  the upfront run was $175.99M -- it built 5 MW of it for the evening peak; annualized, none.)

