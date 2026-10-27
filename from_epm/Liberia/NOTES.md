# Liberia -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Liberia`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Liberia.xlsx` (22 generators + 1 real storage plant). Shares WAPP's
regional zone list (`Benin`, `Burkina`, `IvoryCoast`, ..., `Export_Liberia`) but only lists
Liberia's own ~22 generators -- the other zones are interconnection-only here, unlike
Mauritania below. `CLSG` (the Liberia-Cote d'Ivoire-Guinea-Sierra Leone interconnector) has
`fuel1="Import"` but a wrong raw `Type="STO HY"` -- same DUMMY-generator artifact as Lao's
Thai/Cambodian import proxies. `GT` renamed to `OCGT`. All 5 of 5 Existing/Committed plants have
the duplicate-of-capacity artifact. Capex disabled.

**Finalized** (2026-10-20) -- seer dataset in `data/seer/final/epm_open/Liberia` (results in
`data/seer/results/Liberia`, 2030-2034, and `.../annualized`; records in
`data/reports/country_report.pdf`, with a Liberia-only lost-load-share plot). One zone, no lines,
no trade. Everything is in `REGIONS["Liberia"]` except one hand edit (below) -- **re-apply it
after regenerating**.
- **Manual change -- `settings.csv` gets `voll,500`** (hand edit in the seer folder,
  2026-10-20): EPM's `Settings1` VOLL, $500/MWh.
- **Regenerated 2026-10-20** with the current pipeline; validates. Years 2016-2042, 12 months x
  2 days (peak / off-peak), `Duration` weights 3.80 / 26.61 per hour -> 8760 h. Load factor of
  peak x profile 0.67. EPM's own `Energy` is inconsistent with its peak before ~2024 (load
  factor > 1, e.g. 2020: 750 GWh on a 50 MW peak), so peak x profile is 18-61% below it there;
  from 2029 it's within 6% (2030: -3%, 2034: +5%). Left as-is.
- **Manual change -- dropped every zone but Liberia** (`drop_zones`, 2026-10-20). The 13 other
  WAPP zones have no generators and no demand (`interconMode=0`). `Export_Liberia` is an export
  sink: a flat 100 MW load (200 MW from 2040) with no generators, joined by a 100 MW line (a
  200 MW line option in `ZoneData` isn't read) -- seer models no exports, so it and its line
  are dropped.
- **Manual change -- VOM in marginal cost** (`include_vom`): `CLSG` imports (27 MW) are priced
  only through VOM ($120/MWh) and were free without it. Also adds hydro/PV $1.12, Bushrod
  $6.5, diesel $10, GT $4.2.
- Fuel prices are $/MMBtu and match the `FuelPrices` sheet (HFO, diesel, LNG; 2016-2017 hold
  2018's prices). Existing/committed: `Existing_gen_lib` diesel 22.6 MW (2010-2030), Mount
  Coffee hydro 88 MW (2018-), Bushrod 38 MW (LNG, `ICE` -> tech `CCGT`, 2023-), `CLSG` imports
  27 MW (2022-), Saint Paul 2 hydro 154 MW (2030-). Build options: 14 hydro (1,026 MW, 2026-2044
  start, $1.1-9.8M/MW), PV (`Respite_PV` 20 MW from 2024, `PV_CAND_LIB` 5 GW from 2030, 50 MW/yr
  each, $0.97M/MW), battery (`Sto_Liberia` 2 h, 100 MWh/yr up to the Storage sheet's 10 GWh,
  $250k/MWh, life 15) and `GT_Liberia` (500 MW HFO, only from 2042).
- All hydro (existing and candidates) shares one `GenAvailability` shape: 0.19-0.32 Jan-May,
  0.71-0.99 Jun-Nov, 0.6 Dec (mean ~0.61).
- **Results (2030-2034, VOLL $500)**: pyomo and PyPSA agree exactly upfront ($717.2M; capital
  $143.7M, marginal $155.2M) and build 44 MW hydro (MountCoffee EXT) + 97 MW PV. But **1,010 GWh
  lost load** (71 GWh in 2030 rising to 310 GWh in 2034, ~13% of demand; up to 205 MW), all in
  the dry season (Dec-May). It's an energy shortage, not a peak one: Jan-Apr average demand is
  192 MW in 2030 (253 MW in 2034) against ~65 MW of thermal + imports and ~55-75 MW of hydro at
  its dry-season availability. The only firm build option (`GT_Liberia`) starts in 2042, and PV
  and the battery are capped at 50 MW / 100 MWh a year. Annualized (pyomo $489.3M, PyPSA $502.6M
  -- +2.7%, the battery's round-trip losses, which pyomo ignores): builds hydro (Cestos, Loffa,
  Small Sites, MountCoffee EXT, part of Tiboto / Saint Paul 2 EXT), PV and the battery at their
  annual limits (250 MW / ~450 MWh by 2034), and still sheds 295-313 GWh, Jan-Apr only.
  Dispatch: hydro ~5.6 TWh, Bushrod 0.8 TWh (dry season only), imports 0.5 TWh, PV 0.5 TWh.
  The lost load is accepted as EPM's own shortfall (user's call, 2026-10-20): 4.9% of demand in
  2030 rising to 16.3% in 2034 upfront, 1.4-4.5% annualized.
