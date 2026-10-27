# Botswana -- processing notes

Extracted from seer's `data/processed_epm_open/README.md` (section `Botswana`). Paths below refer to the seer repo.


**Finalized** -- seer dataset in `data/seer/final/epm_open/Botswana` (results in
`data/seer/results/Botswana`, records in `data/reports/country_report.pdf`). Contains a hand
edit (the restored `New_Battery_Bot`, below) -- regenerating overwrites it.

Source: `data/raw_epm_open/Botswana.xlsx` (25 generators + 1 real storage plant, 2 zones:
`Botswana` itself, `Namibia` interconnection-only). Forecast demand (`q1..q4`, lowercase).
`GT` renamed to `OCGT`. `Moz_imports` (Mozambique's EDM utility) has `fuel1="EDMImport"`, a
region-specific import-source name like Namibia's `ESKOMimport`/`ZESCOImport`/`ZPCImport`.
`fuel1="GasECBM"` (enhanced coal-bed methane) is deliberately left raw, not folded to `"Gas"` --
`Fuel Prices` genuinely prices it separately (same reasoning as Sudan's Coke / Bosnia's
mine-specific coal); its generator (`Type="CCGT"`) already resolves tech cleanly regardless.
One orphaned storage pair (`New_Battery_Bot`/`New_Storage_wPV_Bot`) has no matching
`Generator Data` row, no real linked generator, and absurd placeholder capacities
(1,000,000 / 10,000,000 MWh) -- confirmed junk, dropped (not rescued). `New_Battery_Bot` also
has a `Generator Data` row (Type `STORAGE`), through which it survived as storage with the
placeholder MWh (20,000 h duration) -- dropped there too (`drop_generators`).
- **Manual change -- `New_Battery_Bot` restored by hand** (hand-edited `storage_constants.csv` /
  `storage_costs.csv` in the seer folder -- **regenerating overwrites this, re-apply it**): only
  its MWh was junk; its `Generator Data` row is a plausible candidate (500 MW project, 100 MW
  units, 50 MW/yr). Re-added as a battery build option with an **assumed 4 h duration**: units
  400 MWh, 200 MWh/yr, 2,000 MWh total; efficiency 0.85, capex $250k/MWh (Storage sheet's
  `CapexMWh` 250), life 25 (Generator Data; the Storage sheet says 8), 2022-2050. `New_Storage_wPV_Bot` stays dropped.
- **Manual change -- dropped zone `Namibia`** (`drop_zones`): `ZoneData` "Include in model:
  NO", no generators or demand, `interconMode=0` -- an external trade zone at the far end of the
  90/250 MW `TransferLimit` line. No trade with Namibia is modeled.
- **Manual change -- dropped `Coal_Greenfield_Bot`** (`drop_generators`): a disabled candidate
  (Capacity/BuildLimitperYear/MaxTotalBuild all 0) that came out expandable with no limit.
- **Manual change -- no expansion options for committed plants** (`no_expansion`): `Coal_Bot`
  (260 MW), `Wind_2027` (50 MW) and `New_CBM_Bot_gas` (10 MW) carry build limits, which split off
  a `<name>_expansion` build option -- duplicating real candidates (`New_Coal_Bot`/
  `MorupuleB5_6_Bot`, `Wind_2030`, `New_CBM_Bot_gas_2031`/`New_ECBM_Bot`), with looser limits
  (Coal_Bot's and Wind_2027's had a per-year limit and no total cap, though their own
  MaxTotalBuild equals their capacity). Their build limits are cleared; Botswana only.
- **Manual change -- assumed gas price** (`fuel_price_overrides`): `Gas` and `GasECBM` are
  $0.01/MMBtu in `Fuel Prices` (EPM ran with this too), which made every gas plant ~free
  ($0.06/MWh) baseload, undercutting coal. No other number in the workbook (the `ExportPrice`
  scenario is empty). Replaced with an **assumed** flat $6/MMBtu (~$37/MWh at heat rate 6.2) --
  a placeholder, not sourced; revisit with a Botswana CBM price if one turns up.
- **Manual change -- `Moz_imports` priced at its tariff** (`marginal_cost_from_tariff`): its
  `HeatRate` is the 9.42 placeholder (also on wind/solar), so heat rate x price turned the
  `EDMImport` tariff ($100-123/MWh) into ~$285-340/MWh (EPM did the same). Its marginal cost is
  now the tariff itself, from `Fuel Prices` (written to `pMarginalCostOverride.csv`, read by
  `inputs/epm2seer/generator_params.py`). Only matters for 2022-2024 -- it retires in 2024.
- **General fix -- fuel price units**: coal is priced in $/ton and `EDMImport` in $/MWh, and
  were used as $/MMBtu (coal ~$600/MWh). Now converted with `FuelTechnologies`' table (1 ton =
  25.63 MMBtu, 1 MWh = 3.41), matching the workbook's own `FuelPrices` sheet: coal ~$23/MWh.
- **Manual change -- `settings.csv` gets `voll,1000`** (hand edit in the final seer
  folder, 2026-10-20): EPM's `Settings1` VOLL ("Cost of unserved energy per MWh, $"),
  $1,000/MWh, replacing seer's $1M/MWh default. Re-solved 2026-10-20 (both models read it now).
  **Re-apply after regenerating.**
