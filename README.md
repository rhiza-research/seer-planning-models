# seer-planning-models

Capacity expansion planning models for power systems, in the input schema of
[seer](https://github.com/rhiza-research/seer). Each folder is a complete model: load it
with seer and solve it with the PyPSA or Pyomo backend.

## from_epm

Models converted from the World Bank's
[Electricity Planning Model (EPM)](https://github.com/ESMAP-World-Bank-Group/EPM) "EPM open"
input workbooks, using seer's EPM pipeline (`scripts/convert_epm_open.py` -> `epm_to_seer()`).

The conversion reproduces EPM's own assumptions. It does not correct them toward the real system.
Where a model needed a fix to solve or to match EPM's intent, the change is listed in
that model's `NOTES.md`, including any manual edits to the csvs.

| Model | Zones | Years | Notes |
|---|---|---|---|
| Bosnia | 1 | 2020-2040 | |
| Botswana | 1 | 2022-2040 | |
| Burkina Faso | 1 | 2018-2030 | |
| Chad | 1 | 2020-2035 | |
| Djibouti | 1 | 2018-2030 | |
| Egypt | 1 | 2018-2035 | |
| Georgia | 1 | 2021-2040 | |
| Iraq | 1 | 2018-2040 | |
| Lao | 4 | 2020-2030 | |
| Liberia | 1 | 2016-2042 | |
| Maldives | 92 | 2021-2040 | |
| Morocco | 1 | 2020-2050 | |
| Namibia | 3 | 2020-2040 | |
| Nigeria | 1 | 2018 | single-year dispatch backcast |
| Senegal | 1 | 2017-2030 | |
| Sierra Leone | 1 | 2019-2030 | |
| Sudan | 1 | 2019-2030 | |
| WestAfrica | 16 | 2018-2040 | full WAPP, from EPM's Mauritania workbook |

Each folder holds the seer schema csvs (`zones`, `years`, `times`, `settings`,
`demand_forecast`, `demand_profile`, `generator_constants`, `generator_costs`,
`storage_constants`, `storage_costs`, `renewable_profiles`, `network`) and a `NOTES.md` with
that model's processing notes.

## Provenance

Generated with seer at commit `5ae6d098a15824fe3f5b0364b604b394bea47377`. Paths mentioned in
`NOTES.md` (`data/...`, `scripts/...`) refer to the seer repo.
