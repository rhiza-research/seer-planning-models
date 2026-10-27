# Gambia -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Gambia`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Gambia.xlsx` (42 generators, 3 zones: GBA/FFN/BSE; forecast demand,
Q1-Q4 x 2 day types).

**Finalized** (2026-10-26) -- seer dataset in `data/seer/final/epm_open/Gambia` (results in
`data/seer/results/Gambia`, 2026-2030; records in `data/reports/country_report.pdf`). Three zones (GBA = Greater Banjul, FFN = Farafenni,
BSE = Basse), 2019-2035, 4 quarters x 2 day types (`Duration` 65.2/26.1 per hour -> 8760 h).
232 MW existing incl. committed after derates (114 MW oil, 58 MW OMVG hydro imports from
Guinea, 30 MW Sambangalou hydro from 2025, 10 MW Senegal import, 20 MW PV), 12 build options,
one battery build option, lines GBA-FFN 10 MW and FFN-BSE 5 MW (flat, symmetric). Changes in
`REGIONS["Gambia"]` plus **four seer-folder hand edits (below) -- regenerating overwrites them,
re-apply all**.
- `GT` -> `OCGT`.
- Battery candidate is `Sto_GBA` in Generator Data but `Sto_GAM` in Storage --
  `generator_renames` matches them. 10 h duration ($250k/MWh, 1,000 MWh/yr, 10,000 MWh total)
  -- as in the `Storage` sheet.
- `OMVG_AKaleta`/`OMVG_BSouapiti` (Guinean hydro imported over OMVG) keep Type `STO HY`
  and their own fuel `Import_Kaleta_Souapiti` (priced in Fuel Prices, $30/MWh in 2026; heat
  rate 1.0) -- availability 0.3 (Q1-Q2) / 0.5 (Q3-Q4). `OMVG_Csambangalou` 0.9 flat.
- Demand: peak x profile matches EPM's `Energy` exactly in every zone and year.
- **Manual change -- `include_vom`** (in `REGIONS`): `Import_Senegal_FFN` has no HeatRate and is
  priced only through VOM ($100/MWh) -- free without it. Adds $10/MWh to the HFO engines ($6.5
  Brikama III), $4.2 to `GT_GBA`, and **$130/MWh to `OMVG_Csambangalou`** (hydro, but its
  VOM in the workbook -- kept as EPM has it).
- **Manual change -- zero-availability oil units at 0 MW** (seer hand edit,
  `generator_constants.csv`): `GenAvailability` is 0 in every quarter for KotuG1, KotuG3,
  KotuG6, BrikamaIG2, BrikamaIG6, BrikamaIIIG1, BrikamaIIIG2 and FarafenniG1 (46.6 MW) -- the
  fixed-derate rule with a factor of 0. The other oil units are 0.85-0.9 (left as-is).
- **Manual change -- `Nama_Solar_FFN`/`Nama_Solar_BSE` get GBA's solar profile** (seer hand
  edit, `generator_constants.csv`: `isre=True`, `Jambur_Solar`'s `profile_id`): `REProfile`
  has solar only for GBA, so they came out as free dispatchable 10.5 MW. Gambia is ~300 km
  across, so GBA's shape is a fair stand-in. `Nama_Solar_BSE` is in zone FFN in the workbook --
  kept.
- **Manual change -- `Import_Senegal_FFN` tech/fuel `import`** (seer hand edit; typed
  `ICE`/`HFO` in the workbook). Label only.
- **Manual change -- `zones.csv` country `Gambia`** (seer hand edit; ZoneData has no
  zone->country map, so it was the raw `GBA`/`FFN`/`BSE`).
- **Manual change -- `settings.csv` gets `voll,500`** (Settings1 VOLL). **Re-apply after regenerating.**
- PV `GenAvailability` (0.87-0.95) is not applied -- PV uses its `REProfile` profile.
- Not modeled: 15% planning reserve margin, 50% max import share, `GT_GBA`'s 0.3
  `MinLimitShare`, zonal spinning reserve (`Settings1`).
- **Results (2026-2030, VOLL $500)**: pyomo and PyPSA both $623.7M upfront; annualized $492.0M
  vs $494.5M (0.5%: pyomo builds a ~220 MWh battery, PyPSA none -- the known storage
  efficiency difference). Generation 6.2 TWh: oil 2.41, hydro 2.07, PV 1.22, imports 0.42,
  OCGT 0.11. Upfront builds 138 MW of PV (mostly 2026) and 26 MW of `GT_GBA`, and sheds 39.2
  GWh at the evening peaks (19-23h), mostly 2029-30 (up to 29-35 MW in GBA) -- the
  end-of-horizon effect. Annualized builds all 330 MW of PV and 38 MW of OCGT and sheds
  0.2-0.5 GWh. FFN/BSE share some of the shedding: BSE has 6.6 MW of local oil and a 5 MW
  line against an 8.5 MW 2030 peak, and no local build option.
