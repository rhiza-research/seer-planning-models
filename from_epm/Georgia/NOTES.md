# Georgia -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Georgia`). Paths below refer to the seer repo.


Source: `data/raw_epm_open/Georgia.xlsx` (165 generators, single zone `GE` + one external
interconnection zone `EX`; storage and capex trajectories both disabled/empty).

- No `MapCountry`/"MAP ZONES TO COUNTRIES" block in `ZoneData` at all (some single-zone
  workbooks omit it) -- `zcmap.csv` falls back to the plain zone list with `zone == country`,
  making the country-broadcast a no-op (correctly, since fuel prices are already keyed by `GE`
  directly). No `REgenProfile` sheet exists either -- handled as empty rather than erroring.
- `Type="GT"`/`"INT"` renamed the usual way (`"OCGT"`/`"Import"`).
- **`Type="WIND1"/"WIND2"/"WIND3"` and `"SOLAR1"/"SOLAR2"/"SOLAR3"` are real, distinct
  resource-quality categories** (High/Average/Low resource wind/solar), each with its own
  `REProfile` capacity-factor data, shared by multiple generators (e.g. `Wind-1`/`Wind-2`/
  `Wind-3`/`High Resource Wind` all share `Type="WIND1"`). Folding these to plain `Wind`/`PV`
  would collide multiple genuinely different profiles into one (zone, tech) key -- the same
  "conflicting duplicate profile" problem Iraq's Rooftop fold caused. Left unmapped (no
  PyPSA-aligned equivalent exists for "wind, low resource site"), only re-cased so the
  generator's `Type` ("WIND1") matches `REProfile`'s own tech-column spelling exactly ("Wind1")
  -- unmapped tech values are matched by exact string, not case-insensitively.
- Only 1 of 156 Existing/Committed plants has the `BuildLimitperYear`/`MaxTotalBuild`-
  duplicates-`Capacity` artifact -- negligible here, unlike Lao/Iraq/Morocco.

**Finalized** -- seer dataset in `data/seer/final/epm_open/Georgia` (results in
`data/seer/results/Georgia`, 2022-2026: pyomo $252.7M = PyPSA $252.7M, identical dispatch; annualized
capex in `.../annualized`, same result -- nothing built). No hand edits -- regenerating reproduces it.
- **Regenerated 2026-10-19** with the current pipeline. Fuel prices converted from $/tcm gas
  ($143 -> $4.28/MMBtu 2018-21, $230 -> $6.89 from 2022) and $/ton coal ($46 -> $1.79) -- match the
  `FuelPrices` sheet. Marginal costs: CCGT ~$45-48, old gas steam (`Mtkvari`/`Tbilsresi`/`Gpower`
  TPP, typed CCGT by the tech mapping) ~$69-77/MWh (2022+). Time grid: 12 months x 1 day type x 5
  load blocks (`t1..t5`, 60 steps; the 4h48 spacing in `times.csv` is a label only), `Duration`
  weights sum to 8760 h. Ramp rates all raw 0 -> 1.
- **General fix found here -- blank demand years** (`_write_demand_detailed`): GE has no `Demand`
  rows for 2018-2020 (EPM's demand starts 2021), and the profile was taken from the first year, so
  `demand_profile.csv` came out entirely blank -- and still passed validation. Now only years every
  zone has demand for are modeled (Georgia: 2021-2040) and each zone's profile comes from its first
  year with data. peak x profile is within ~2% of EPM's energy (2021 exact, 2026 -2.1%).
- **Manual change -- dropped zone `EX`** (`drop_zones`, `drop_generators: Generator`): EX is EPM's
  export market, included in the model (`interconMode=1`): 4,000 MW flat demand served by its own
  4,000 MW `Generator` (ROR, 0.98 availability, VOM $40/MWh), with a 4,000 MW GE->EX line from 2020.
  It was only falling out through a casing mismatch (`ex` in Demand/TransferLimit/Topology, `EX` in
  ZoneData), and `Generator` was landing in GE as 4 GW of free existing hydro (15.3 GW "existing"
  instead of ~5.7 GW). Left out on purpose: no exports, so summer hydro surplus is just curtailed.
- **Manual change -- zone renamed `GE` -> `Georgia`** (`zone_renames`, now applied to every
  zone column the converter writes, not just Generator Data). EPM's sheets say `GE` throughout.
- **Manual change -- ROR availability** (`availability_profile_types: ["STO HY", "ROR"]`): all 130
  ROR plants have monthly `GenAvailability` factors (mean 0.57) and no REProfile -- before this
  they were unlimited free generation.
- **Manual change -- VOM in marginal cost** (`include_vom`): the two 750 MW import rows are priced
  only through VOM ($60/MWh) and came out free. Also adds thermal VOM ($0.06-5.6/MWh).
- **Manual change -- dropped `Tkibuli TPP`** (`drop_generators`): 0 MW existing coal, inert.
- Existing system (2021): 5.9 GW -- 2.0 GW reservoir hydro, 1.6 GW ROR, 1.1 GW gas, 310 MW wind,
  126 MW PV, 750 MW imports; ~5.6 GW more is committed through 2030 (110 small ROR plants,
  Namakhvani/Nenskra/Khudoni, Wind-1..3 1.16 GW, Solar PV-1..3 505 MW -- EPM's assumptions).
- Left as-is: `GenAvailability` derates for thermal are ignored (EPM: old gas steam units ~0.14
  mean, CCGT 0.95, Qartli wind 0.46 on top of its profile -- don't bind in 2022-2026: the steam units
  never run at $69-77/MWh); RICE (`ICE`) maps to tech CCGT; FOM not
  modeled. Not modeled (in `Settings1`): 15% system reserve, `MaxImports` 50%, `ImportShare` 20%.
- Results (2022-2026): ~78% hydro/ROR, ~9% wind, ~10% gas (winter only, falling 1.5 -> 0.5 TWh as
  committed hydro arrives), ~2% imports (winter peaks). No lost load. Reservoir hydro is curtailed
  Apr-Jun (flat seasonal cap, no energy budget) and so is most PV. Nothing built even with
  annualized capex: gas only runs in winter, when PV output is low.
- **Manual change -- `settings.csv` gets `voll,10000000`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $10,000,000/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
