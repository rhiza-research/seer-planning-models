# seer-planning-models

This repository contains capacity planning models specified in the [seer](https://github.com/rhiza-research/seer) schema. seer tools can be used to run, modify, and visualize these models. 

Each model is contained in a single folder and fully defined by its [seer schema](https://github.com/rhiza-research/seer/tree/main/seer/schema) csv files. Top level folders indicate different model datasets and sources. 

Models included here come from different sources and have varying licenses; please check in each model folder for the appropriate licensing information. Models are made available under their source licenses.

### EPM Models

`from_epm/` 

A set of models converted into the seer schema from the [Electricity Planning Model (EPM)](https://github.com/ESMAP-World-Bank-Group/EPM) dataset. The original model data was programmatically and manually converted into the seer schema. Changes to the original model data including corrections, gap filling, or simplifications which are recorded for each model in its associated `NOTES.md` file.

See the [dataset readme](from_epm/README.md) for a summary of models. 


