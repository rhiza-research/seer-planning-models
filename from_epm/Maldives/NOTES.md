# Maldives -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Maldives`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Maldives.xlsx` (83 real generators + 53 storage, 92 zones --
distribution feeders, all in one country "Maldives").

- `Type="RTOPPV"` (rooftop solar, distinct from `PV`) -- seer *does* have a distinct
  `solar-rooftop` tech (synonym `"Rooftop"`/`"RPV"`), but this workbook's `REProfile` has no
  separate rooftop profile data at all (only generic `PV`) -- so `RTOPPV` is folded into plain
  `"PV"` anyway, to keep a real (approximate) generation profile rather than none. Contrast with
  Iraq below, which does have distinguishing profile data and keeps the distinction.
- **Storage is enabled and real** here (unlike Lao): storage plants also appear as ordinary
  `Generator Data` rows (`Type="STORAGE"`, `fuel1="Battery"`) -- exactly the same convention
  gams regions use, so `storage_params.py` resolves their zone/tech from those rows with no
  special-casing needed. The `Storage` tab itself only supplies `CapacityMWh`/`CapexMWh`/
  `Efficiency`/`Life`.
- No `BuildLimitperYear`/`MaxTotalBuild`-duplicates-`Capacity` artifact here (0 of 28
  Existing/Committed plants) -- unlike Lao/Iraq, this model's raw data doesn't have it.
- **Demand uses the "less detailed" (forecast) formulation** (`Settings1.altDemand=1`, unlike
  Lao): `Demand_Forecast` (peak/energy per zone/year) and `DemandProfile` (a separately
  normalized zone/season/day/hour shape, seasons `Q1..Q4`) map directly to
  `pDemandForecast.csv`/`pDemandProfile.csv` with no derivation needed -- the same shape gams
  already produces. 40 of 92 zones have no demand and no profile (bus-chain junction nodes),
  same as some gams regions' export-only zones.
- Capex trajectories disabled (`Captraj=0`), same as Lao.

**Finalized** (2026-10-21) -- seer dataset in `data/seer/final/epm_open/Maldives` (results in
`data/seer/results/Maldives`, 2021-2025, and `.../annualized` (pyomo only); records in
`data/reports/country_report.pdf`). Contains three hand edits
(below: lines, unit sizes, voll) -- **re-apply them after regenerating**.
- **Regenerated 2026-10-20** with the current pipeline; validates. Years 2021-2040, 4 quarters x
  3 days, `Duration` weights sum to 8760 h. The model is the Greater Male distribution grid:
  `Genzone` (all 27 existing diesels, 81 MW 2000-2045, + 0.67 MW `SolarPVJICA`) feeds three bus
  chains (F2/F7/F14, 8 MW segments, 1000 MW from `Genzone`) with DT transformer zones on them
  (0.2-1.4 MW lines) and ten single-zone feeders (F3..F16 S1). Demand 74 MW (sum of zonal peaks)
  in 2021 -> 146 MW in 2040; peak x profile matches EPM's `Energy` exactly (2021: 427 GWh).
  DT37/DT37A/DT88 have no demand (DT37A/DT88 lines are 0 MW both ways, DT37 and F14S21 have no
  line) -- inert, left in. `interconMode=1`, `allowExports=0`, no external zones.
- Diesel $20.4/MMBtu (2021) -> $34.2 (2040), matches the `FuelPrices` sheet; heat rate 9.37
  (existing) / 9.0 (`GenericDiesel`) -> marginal cost $191-320/MWh. VOM ($7/MWh diesel) not
  included (`includeVOM=0`). One PV profile shared by all solar (CF 0.18). Build options:
  `GenericDiesel` (100 MW, $1.2M/MW), `GenericPV` (5 MW, $1.2M/MW, from 2023), rooftop PV per
  DT (10 MW) and per F feeder (10 MW/yr, 20 MW total) at $3M/MW, and a 4 h battery per zone
  (20 MWh per DT, 40 per F feeder, 200 at `Genzone`; $250k/MWh, eff 0.85, life 15).
- **Manual change -- `drop_generators: ["GenericPV2"]`** (in `REGIONS`): a disabled candidate
  (Capacity / BuildLimitperYear / MaxTotalBuild all 0) that would have been unlimited PV.
- **Manual change -- `network.csv`: the ten `F*S1`-`Genzone` lines set to 8 MW** (hand edit).
  The two directions disagree (`F*S1 -> Genzone` 0, `Genzone -> F*S1` 8 MW) and the converter
  keeps the first row (0), which would strand ~50 MW of feeder load; 8 MW is the import
  direction (into the feeders).
- **Manual change -- `generator_constants.csv`: `unit_size` 0 -> blank** for the ten
  `RooftopS_F*S1` options (hand edit). EPM's `UnitSize` 0 is a blank; harmless unless modular
  expansion is on (needs a positive size).
- **Manual change -- `settings.csv` gets `voll,1000`** (hand edit): EPM's `Settings1` VOLL.
- Not modeled: planning reserve margin (`planning_reserve_constraints=1`, 10%), `LossFactor`,
  FOM.
- **Results (2021-2025, VOLL $1000)**: upfront, pyomo $420.03M vs PyPSA $420.12M (+0.02%). Only
  1.2-1.5 MW of DT batteries built (DT20A, DT50A); no diesel or PV. Lost load 0.01 GWh (2021) ->
  ~1 GWh (2025), all at DTs whose peak exceeds their transformer line (DT20A, DT50A, DT75,
  DT41A; a little on F10S1/F14) -- EPM's own grid limits (`pAllowHighTransfer=0`), not missing
  generation: existing diesel covers the system peak. Annualized, pyomo $416.29M builds ~96 MW
  PV (`GenericPV` 5 MW + ~91 MW rooftop, mostly the F feeders' 10 MW/yr caps) and 2.3 MW DT
  batteries; lost load <= 0.07 GWh/yr. PyPSA annualized not run: HiGHS dual simplex
  was still far from converged after ~20 h (1.1M rows after presolve), so it was dropped.
  Solve times (5 years): pyomo ~300 s, PyPSA ~710 s upfront.
