# Burkina Faso -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Burkina Faso`). Paths below refer to the seer repo.


**Finalized** -- seer dataset in `data/seer/final/epm_open/Burkina Faso` (results in
`data/seer/results/Burkina Faso`, records in `data/reports/country_report.pdf`).

**2026-10-19:** `generator_constants.csv`/`storage_constants.csv` converted in place to the
start_year + life schema (no `retirement_year`; see the checklist). Re-solved: results unchanged.

Source: `data/raw_epm_open/Burkina Faso.xlsx` (40 generators + 5 real storage plants, single
zone `BUR`; a `REProfile` zone="SEN" block is unrelated template leftover, harmlessly
filtered). Forecast demand; both `REProfile` (`"dry"/"Dry"/"wet"`, `"peak"/"offpeak"`) and
`DemandProfile` (`"dry"/"wet"`, `"peak "/"off peak  "` -- stray whitespace, stripped
automatically) needed covering.
- `GT` renamed to `OCGT`. `HY` (hydro) has no tech synonym, but bare `"Water"` (its own fuel1
  value) is already registered -- renamed to match. `PVwSTO` (2 candidate solar+storage
  hybrids, 95/144 MW) modeled as plain `PV`, same precedent as Namibia's hybrid split.
- `Fuel Prices` is missing its literal header row entirely (lands straight on data after the
  usual skip) -- exposed a **general robustness gap**: the Scenario/Zone/Fuel/Unit columns are
  now renamed by position when the literal `"Scenario"` header text isn't found, not just for
  this region.
- Storage tab has 7 entries; 5 (`Sto_BUR_75/150/300`, `Stowapp1`, `stowapp2`) already match
  real `Generator Data` rows directly. The other 2 (`StoPV1_BUR`/`StoPV2_BUR`, 120/180 MWh,
  plausible scale for this system's ~300-400 MW peak) have no matching row -- investigated
  (not a copy-paste leftover; sizes check out against this region's own demand) and left
  dropped per user call, consistent with not inventing data the source doesn't have.
- **Manual change -- VOM in marginal cost** (`include_vom`): marginal cost is otherwise only
  heat rate x fuel price, and here the imports (`Interco_COT`/`Interco_Ghana`, VOM $120/$110
  per MWh) and PPA-style solar (capex 0, VOM $50-90/MWh, incl. candidates `WAPP_phase1/2`) are
  priced *only* through VOM -- they came out free. VOM is now added on top of fuel for this
  model (an `includeVOM` row in `pSettings.csv`, read by `inputs/epm2seer/generator_params.py`;
  off for every other region).
- **Manual change -- hydro availability profiles** (`availability_profile_types`): hydro is
  `Type="HY"` (renamed `"Water"`), not `"STO HY"`, so it missed the general `GenAvailability`
  fix and was unlimited zero-cost generation. All 10 hydro plants now get a generator-specific
  profile from `GenAvailability`'s dry/wet factors (0.08-0.36, e.g. BAGRE 0.22 dry / 0.33 wet).
- **Manual change -- dropped `Ouaga-Ouest_dropped`** (`drop_generators`): a disabled candidate
  (Capacity/BuildLimitperYear/MaxTotalBuild all 0) that came out as an expandable diesel plant
  with no build limit.
- **Storage `CapexMWh` of 1 and 4** (`Stowapp1`/`stowapp2`): Generator Data's Capex is also
  ~0 ($0.01M/MW) -- consistent with being the battery half of the WAPP solar PPAs, paid
  through the PV VOM. Left as-is; they start in 2032, after the 2018-2030 horizon.
- Not modeled (on in `Settings1`): fuel limits, 10% planning reserve margin, min-gen.
- **General fix -- zero ramp rates**: hydro, imports and Zagtouli/Ziga have raw
  `RampUpRate`/`RampDnRate` = 0 (EPM's blank -- `ramp_constraints=0` here anyway), which failed
  validation's (0, 1]. 0 is now treated like a missing value: fully flexible (1). Also affects
  Chad/Egypt/Georgia on regeneration.
- **General fix -- time weights in forecast mode**: forecast-demand regions previously got
  uniform `times.csv` weights; `Duration` is now read for them too (same `weight` column as
  detailed mode; year-invariant, first year used; skipped with a warning if it doesn't cover
  every profile hour). Burkina: each peak-day hour stands for 15 h (dry) / 14 h (wet), each
  offpeak-day hour 168 h, summing to 8760 -- peak days were overweighted ~11x. Every
  forecast-mode region picks this up on regeneration.

Regenerate any/all regions with:
```
python scripts/convert_epm_open.py [Lao] [Maldives] [Iraq] [Morocco] [Poland] [Georgia] [Egypt] [Sudan] [Bosnia] [Chad] [Djibouti] [Liberia] [Mauritania] [Namibia] [Nigeria] [Senegal] ["Sierra Leone"] [Botswana] ["Burkina Faso"] [Gambia] [Rwanda] ["Sao Tome and Principe"] [Ukraine]
```
- **Manual change -- `settings.csv` gets `voll,500`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $500/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
