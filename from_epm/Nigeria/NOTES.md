# Nigeria -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Nigeria`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Nigeria.xlsx` (29 generators, single zone `NGA`; storage and capex
both disabled). `ZoneData`'s own "MAP ZONES TO COUNTRIES" block is shifted a column over here
(zone column comes back entirely NaN) -- exposed a **general robustness gap** in the zcmap
fallback: it previously only triggered on an empty block, not a present-but-garbage one; now
checks whether the zone column itself is actually populated. `GT` renamed to `OCGT`.
**`Demand` is a genuine full 8760-hour single calendar year (2018)** -- 365 real days x 24 real
hours, `year` constant throughout -- not a representative-days sample like every other region,
and only one modeled year results (no multi-year capex/storage data exists either, consistent
with a single-snapshot dispatch study rather than a capacity-expansion one).

**Finalized** (2026-10-26) -- seer dataset in `data/seer/final/epm_open/Nigeria` (results in
`data/seer/results/Nigeria`, 2018; records in `data/reports/country_report.pdf`). Re-checked
2026-10-26 before finalizing: a fresh regeneration matches apart from the four hand edits, and
the re-solve with the current models gives the same results. A
2018 dispatch backcast: one zone, 8760 real hours (weight 1, 8760 h), existing plants only (no
build options), no lines, no storage (`includeStorage=0`; `Sto_NGA` has no Generator Data row
and is excluded). Changes in `REGIONS["Nigeria"]` plus **four seer-folder hand edits (below) --
regenerating overwrites them, re-apply all four**.
- **Regenerated 2026-10-22** with the current pipeline; validates.
- **Manual change -- `include_vom`**: gas is $0/MMBtu in `Fuel Prices` (and the computed
  `FuelPrices` sheet); every gas plant's cost, fuel included, is in `VOM` ($28.5-56.4/MWh, ~3.4 x
  HeatRate). Without it every marginal cost came out $0. Hydro VOM $1.12/MWh.
- **Manual change -- `Dummy_RE` dropped** (`drop_generators`): a 0 MW PV placeholder with an
  all-zero profile and no build data.
- **Manual change -- gas plants derated** (seer hand edit of `generator_constants.csv`):
  `GenAvailability` gives each plant its observed 2018 monthly availability. Capacity x the
  day-weighted mean of the 12 months, where that is below 0.8 (Iraq's rule): 20 of 25 plants,
  gas 10,972 -> 5,995 MW. AESEBUTEBARGE and ASCO (0 all year) go to 0 MW; Sapele 720 -> 38 MW
  (0.053), Egbin 1,320 -> 706 MW (0.534), Alaoji 500 -> 121 MW (0.242). Left as-is (>= 0.8):
  Afam VI, Geregu gas, Olorunsogo gas, Omotosho gas, Paras. Not a per-month cap: an hourly
  availability profile would need `isre=True` on thermal plants, which we don't want. The
  month-to-month variation is lost (e.g. December is tighter in EPM than here).
- **Manual change -- `zones.csv` country `Nigeria`** (seer hand edit; `MapCountry` gave `NGA`).
- **Manual change -- `settings.csv` gets `voll,500`** (seer hand edit): `Settings1` VOLL, $500/MWh.
- **Manual change -- three negative demand hours set to 0** (seer hand edit of
  `demand_profile.csv`: 06-23 10:00, 06-28 14:00, 09-16 18:00, down to -69 MW). EPM absorbs them
  as surplus at `costSurplus` $0, the same as 0 load here.
- Demand is the `Demand` sheet's real 2018 MW, kept as-is: ~100 hours below 0.3 of peak are grid
  collapses and their recovery (e.g. 09-16 18:00 to 09-17 02:00). Peak x profile is 30.6 TWh,
  11% below `Demand_Forecast`'s `Energy` (34.4 TWh). EPM runs on the hourly MW, so not reshaped.
- `ST` + `Gas` (Egbin, Sapele: gas steam plants) map to `CCGT` through the `ST-Gas` synonym --
  label only, costs come from VOM.
- Not modeled: per-plant monthly gas `Fuel Limits` (bcf, `fuel_constraints=1`) -- in the
  solution, the cheapest plants run 7.9 TWh above their own limits (gas use 20.0 TWh vs 26.0 TWh
  allowed for the plants with limits, so EPM shifts to dearer plants rather than shedding);
  `MinLimitShare` 0.05 min gen; 15% planning reserve (no effect without build options);
  `MinPlantOutput`'s observed output (reference only).
- **Results (2018, VOLL $500)**: pyomo and PyPSA both $758.84M, no lost load. Generation 30.59
  TWh = demand: CCGT 12.0 TWh, OCGT 10.9, hydro 7.7 (hydro at its cap throughout). No annualized
  run: there are no build options, so it would be identical.
