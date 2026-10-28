# Rwanda -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Rwanda`). Paths below refer to the seer repo.

Source: `data/raw_epm_open/Rwanda.xlsx` (32 existing generators, no build options, 1 zone;
detailed demand, 12 months x every day -- 8760 h -- for 2021 only, so a one-year dispatch
model). **Finalized** (2026-10-27) -- seer dataset in `data/seer/final/epm_open/Rwanda`
(results in `data/seer/results/Rwanda`, 2021; records in `data/reports/country_report.pdf`).
One zone, 8760 real hours (weight 1 each; 365 distinct daily shapes, peak 29 Sep 19:00). 300 MW existing: 133 MW ROR (18 plants; `Rusizi I`/`Rusizi II`
are 0 MW in the workbook -- shared with DRC/Burundi, kept), 78.6 MW typed `OCGT`/`Gas`
(Kivuwatt 27 MW methane, plus SES Masoro, SO Energy Birembo/Mukungwa and SOPP -- really
diesel/HFO rentals, but kept as EPM has them, at gas $5.7/MMBtu x 12.4 = $70.7/MWh), 28.8 MW
oil (Jabana 1/2, $138/MWh), 16 MW peat (Gishoma), 32 MW imports (Kenya 30, Uganda 2; $80/MWh,
HeatRate 3.4), 11.8 MW PV. No build options, no storage, no lines.
- `GT` -> `OCGT`; `INT`/`Import` -> Import.
- `BuildLimitperYear` is the text `NO` on 30 plants -- the converter now blanks non-numeric
  param values with a warning (general fix).
- Demand: peak 167.4 MW; peak x profile = 1,017.1 GWh, EPM's `Energy` exactly. Three hours look
  like outage/metering dips (12 Nov 08:00 at 0.03; 27 Dec 13-14:00 at 0.23-0.28, vs ~0.7
  around them; nothing else below ~0.43) -- kept as EPM has them (<0.2 GWh, peak unaffected).
- **Manual change -- ROR uses `GenAvailability`** (`availability_profile_types: ["STO HY",
  "ROR"]`, as Egypt/Morocco/Poland, in `REGIONS`): `REProfile`'s `Water` rows are a flat 1.0
  every hour, so with the earlier `Water` -> `ROR` rename the 133 MW of ROR ran free at full
  capacity against a 167 MW peak. EPM also caps each plant's energy at its GenAvailability
  (0.24-0.73, flat over the months), so each plant now gets that as a flat profile (the
  rename is dropped and the 1.0 rows don't match). ROR averages 61.7 MW, 540 GWh/yr.
- **Manual change -- `marginal_cost_from_tariff` `{"Gishoma": "Peat"}`** (in `REGIONS`): Peat
  is $280/MWh in Fuel Prices, but Gishoma's HeatRate (9.08) took it to $745/MWh (the Botswana
  `Moz_imports` case). It doesn't run either way.
- EPM has `includeVOM` 0 and nothing is priced only through VOM -- `include_vom` not set.
- Thermal GenAvailability is 0.95-1 -- no derate. Nasho Solar's 0.35 isn't applied (PV uses
  its `REProfile` profile).
- **Manual change -- `settings.csv` gets `voll,2500`** (Settings1 VOLL). **Re-apply after regenerating.**
- Not modeled: 15% planning reserve margin, 50% max import share, fuel limits, min-gen and
  system spinning reserve (`Settings1`).
- **Results (2021, VOLL $2,500)**: pyomo and PyPSA both $32.59M (marginal cost only; there's
  nothing to build, so the annualized run is identical). Generation 1,018 GWh: ROR 540, "OCGT"
  455, PV 17, imports 6. Oil and peat don't run, and there's no lost load. Imports
  ($79.8/MWh) sit just above the gas units ($70.7), so they only cover the evening peak once
  all 78.6 MW of gas is running. The full-year dispatch plot makes imports look bigger than
  they are: thin spikes merge at that scale.

