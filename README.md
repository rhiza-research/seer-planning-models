# seer-planning-models

This repository contains capacity planning models for regions and countries, specified in the
[seer](https://github.com/rhiza-research/seer) schema. Each model is contained in a single folder
and specified by a set of csv files.

seer tools can be used to run, modify, and visualize these models. 

Models can come from different sources and have varying licenses; please check in each model folder
for the appropriate licensing information.

Top level folders indicate different model datasets and sources. 

### from_epm

A set of models converted into the seer schema from the
[Electricity Planning Model (EPM)](https://github.com/ESMAP-World-Bank-Group/EPM) dataset.
These models were programmatically and manually converted into the seer schema.
Changes to the original model data including corrections, gap filling, or simplifications are
recorded in the `NOTES.md` file associated with each model.

The set of models are summarized below:

| Model | Zones | Years | Notes |
|---|---|---|---|
| Bosnia | 1 | 2020-2040 | |
| Botswana | 1 | 2022-2040 | |
| Burkina Faso | 1 | 2018-2030 | |
| Chad | 1 | 2020-2035 | |
| Djibouti | 1 | 2018-2030 | |
| Egypt | 1 | 2018-2035 | |
| Gambia | 3 | 2019-2035 | |
| Georgia | 1 | 2021-2040 | |
| Iraq | 1 | 2018-2040 | |
| Lao | 4 | 2020-2030 | |
| Liberia | 1 | 2016-2042 | |
| Maldives | 92 | 2021-2040 | |
| Morocco | 1 | 2020-2050 | |
| Namibia | 3 | 2020-2040 | |
| Nigeria | 1 | 2018 | single-year dispatch backcast |
| Poland | 1 | 2020-2050 | |
| Rwanda | 1 | 2021 | single-year dispatch backcast, 8760 h, no build options |
| Sao Tome and Principe | 2 | 2020-2035 | two unconnected islands; nothing buildable before 2022 |
| Senegal | 1 | 2017-2030 | |
| Sierra Leone | 1 | 2019-2030 | |
| Sudan | 1 | 2019-2030 | |
| Ukraine | 8 | 2020-2050 | |
| WestAfrica | 16 | 2018-2040 | full WAPP, from EPM's Mauritania workbook |
