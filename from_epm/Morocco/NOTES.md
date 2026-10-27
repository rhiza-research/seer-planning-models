# Morocco -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Morocco`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Morocco.xlsx` (119 generators + 5 real storage plants, 3 zones:
MOROCCO itself, plus SPAIN/ALGERIA as pure interconnection zones with no generators of their
own -- topology/transfer-limit endpoints only, same pattern as Lao's Thai/Cambodian zones but
without an explicit DUMMY import generator here).

- `Type="GT"`/`"WindOff"` have no tech synonym at all (only `"OCGT"`/`"Offshore"` are
  recognized) -- renamed.
- `fuel1="CoalCCS"` resolves fine as a plain fuel on its own (`pypsa_fuel_names.csv` already has
  it), but the *tech*-side compound match (`"ST-Coal"` -> `coal`) needs the exact raw fuel
  string, not the standardized one -- `("ST", "CoalCCS")` was unmapped until `"CoalCCS"` was
  folded to `"Coal"` at the source, matching how gams-region CCS variants are already handled.
- Same `BuildLimitperYear`/`MaxTotalBuild`-duplicates-`Capacity` artifact as Lao/Iraq: 103 of
  103 Existing/Committed plants have it.
- Demand uses the "detailed" formulation (`Settings1.altDemand=0`, like Lao), with `Q1..Q4`
  seasons (Lao used `Dry`/`Wet`) -- same `Demand`+`Duration` derivation.
- Fuel prices are keyed by Country (`MOROCCO`/`SPAIN`/`ALGERIA`) here too, but since every real
  generator is in the `MOROCCO` zone/country, the broadcast is a no-op in practice.
- Capex trajectories disabled (`Captraj=0`).

**Finalized** (2026-10-21) -- seer dataset in `data/seer/final/epm_open/Morocco` (results in
`data/seer/results/Morocco`, 2026-2030, and `.../annualized`; records in
`data/reports/country_report.pdf`). One zone, no lines,
no trade. Everything is in `REGIONS["Morocco"]` except one hand edit (below) -- **re-apply it
after regenerating**.
- **Manual change -- `settings.csv` gets `voll,10000`** (hand edit in the seer folder,
  2026-10-21): EPM's `Settings1` VOLL, $10,000/MWh.
- **Regenerated 2026-10-21** with the current pipeline; validates. Years 2020-2050 (every year
  has demand), 4 quarters x 9 days, `Duration` weights -> 8760 h. `Q1..Q4` map to `m1..m4`, so
  plots label the quarters Jan-Apr (cosmetic). Load factor of peak x profile 0.725: matches
  EPM's `Energy` within 0.4% through 2038, then drifts up to +25% by 2050 (EPM's peak grows
  faster than its energy) -- left as-is.
- **Manual change -- dropped SPAIN and ALGERIA** (`drop_zones`, 2026-10-21): external zones
  (`ZoneData` "Include in model" = NO), no generators, demand or `TransferLimit` rows
  (`interconMode=0`). EPM's price-based trade with Spain (`TradePrices`, `allowExports=1`) is
  not modeled.
- **Manual change -- ROR uses `GenAvailability`** (`availability_profile_types: ["STO HY",
  "ROR"]`, as Egypt): the 16 ROR plants (246 MW existing, refurbishments from 2030) have
  seasonal availability 0.15/0.23/0.25/0.16 but no `REProfile` rows, and came out as always-on
  free capacity. Reservoir hydro is 0.17/0.13/0.07/0.07 (~1 TWh/yr from 1,086 MW).
- **Manual change -- dropped `WI_NN_WNDoff_21_MOR`** (`drop_generators`): offshore wind option
  (130 GW, $2.3M/MW), but `REProfile` has only Wind/PV/CSP rows, so EPM gives it zero output --
  without a profile it would be free firm capacity. Also dropped `Generic_Coal_CCS_MOR`
  (`BuildLimitperYear` and `MaxTotalBuild` both 0 -- disabled).
- **Manual change -- `PSH_Menzel_25` is fixed capacity** (`no_expansion`): committed (Status 2)
  350 MW / 4.2 GWh pumped hydro from 2025, but its `BuildLimitperYear` (300) differs from
  `Capacity`, so it came out as an uncapped build option.
- Fuel prices match the `FuelPrices` sheet ($/MMBtu; coal $3.3/MMBtu in 2026 -> ~$28/MWh,
  LNG $7.3 -> ~$52/MWh CCGT, fuel oil/diesel GTs $120-135/MWh). `includeVOM=0` in EPM, so VOM is
  left out (CCS options' $35-48/MWh VOM too). Thermal `GenAvailability` is 0.9 everywhere -- left
  as-is (>= 0.8).
- Existing/committed in 2026: 11.4 GW -- coal 3.8 GW, onshore wind 2.9 GW, hydro 1.1 GW + ROR
  0.25 GW, OCGT 1.1 GW, CCGT 0.8 GW, PV 0.8 GW, CSP 0.4 GW, oil 0.26 GW; plus 1,050 MW / 12.6 GWh
  PHS (Afourer, Abdelmoumen, Menzel). Build options: coal/CCGT/OCGT/HFO (2 GW/yr each, many
  without a total cap -- `MaxTotalBuild` 0 is no cap in EPM), PV ($0.8M/MW), onshore wind
  ($1.2M/MW, 1 GW/yr, 25 GW), CSP ($2.69M/MW incl. Noor Midelt I 400 MW), PHS ($73k/MWh, 12 h),
  battery ($289k/MWh, 8 h); CCS options from 2030.
- **Results (2026-2030, VOLL $10,000)**: upfront pyomo $8,583.4M, PyPSA $8,876.2M (+3.4%);
  annualized pyomo $6,007.1M, PyPSA $6,267.7M (+4.3%). The gap is all PHS round-trip losses
  (0.75, which pyomo ignores; PyPSA loses 2.2 TWh): PyPSA re-solved with storage efficiency 1
  gives $8,583.6M upfront. Upfront builds ~1.7-1.9 GW CCGT + 0.75-0.85 GW OCGT; lost load only
  at the 2029-2030 evening peaks (pyomo 23 GWh / 11 hours, PyPSA 9 GWh / 4 hours) -- the
  end-of-horizon effect. Annualized has no lost load and builds wind (~1 GW/yr), PV (pyomo
  ~3.9 GW, PyPSA ~0.8 GW), PHS, some CCGT and coal. Dispatch (upfront, per year): coal ~33 TWh
  (~55%), onshore wind 11.4 TWh, CCGT ~11 TWh, PV 1.5 TWh, CSP 1.1 TWh, hydro + ROR 1.5 TWh.
