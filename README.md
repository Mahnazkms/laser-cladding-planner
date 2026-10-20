# Laser Cladding Experiment Planner (prototype)

A small Python project that takes a proposed coating composition and a table of past laser cladding experiments, checks that the proposal is valid, finds the most similar past experiments, and logs the outcome of each experiment (successful, partial or failed) in a structured, machine-readable file.

It is a small prototype of an experiment-planning workflow, not a predictive model. The dataset is small (20 rows), so results should not be read as accurate predictions.

## Status

The dataset, the four core functions (`load_data`, `validate`, `powder_mass`, `suggest_params`), the main loop and the result-logging function are implemented in `agent_workflow.ipynb`. Running the notebook validates a set of example proposals (including two invalid ones), calculates powder masses, suggests scan speeds from the thesis data and saves the experiment plans to `experiment_plans.json`.

`log_result()` shows the structure for recording outcomes. No real experiment results have been logged with it yet, so the repository contains no `results_log.csv`.
A first LLM tool-calling demo is in `agent_demo.ipynb`.

## Dataset

File: `laser_cladding_data.csv` (20 experiments, one row per clad track).

All data come from my own MSc thesis work: FeCoCrNiNbx high-entropy alloy layers deposited by laser cladding on AISI 420 martensitic stainless steel. No data from other papers are included.

| Column | Meaning |
|---|---|
| `id` | Row number |
| `source` | Origin of the data (`own_thesis`) |
| `substrate` | Substrate material (`AISI420`) |
| `composition` | Coating composition, e.g. `FeCoCrNiNb0.15` |
| `nb_content` | Nb molar ratio in the coating (numeric) |
| `laser_power_W` | Laser power in watts |
| `scan_speed_mm_s` | Scan speed in mm/s (rounded to 2 decimals) |
| `porosity` | 1 if porosity was observed, 0 otherwise |
| `crack` | 1 if cracks were observed, 0 otherwise |
| `outcome` | Overall result: `good`, `partial` or `failed` |

### Outcome definition

- `good`: no porosity and no cracks.
- `partial`: minor porosity, no cracks.
- `failed`: cracks, or severe porosity.

Outcomes were assigned by me from the observed defects. Row 6 (porosity only) is labelled `failed` because the porosity was severe; the other porosity-only rows are `partial`.

### Notes and limitations

- Laser power is constant (280 W) in all experiments. Only Nb content and scan speed vary, so the dataset cannot show any effect of laser power.
- Rows 9 and 10 are repeats of the same experiment (same composition and scan speed, same result). They are kept on purpose as replicates.
- Hardness and wear data exist for part of the thesis work but are not included in this dataset.
- Other process parameters (powder feed rate, beam diameter, overlap, shielding gas) are not included as columns.
- The dataset is small and covers only one substrate and one alloy family, so it is meant to demonstrate the workflow, not to support general conclusions.

## Workflow

1. `load_data()`: read the CSV with pandas.
2. `validate(proposal)`: check that the proposed composition is valid (fractions sum to 1, only allowed elements).
3. `powder_mass(proposal, total_g)`: calculate the mass of each element powder for a given batch size.
4. `suggest_params(proposal, data)`: find the most similar past experiments and suggest starting scan speeds.
5. Main loop: run several proposals through steps 2 to 4 and write an experiment plan for each.
6. Results log: save each experiment and its outcome (`good`, `partial`, `failed`) in `results_log.csv`.

## LLM agent demo

`agent_demo.ipynb` wraps the functions above as a tool (`plan_for_composition`) and lets a small local language model (llama3.2, run with Ollama on CPU) decide when to call it.

In a first version the tool took the Nb ratio as a number, and the model passed a wrong value (x = 0) on every attempt. The tool rejected it each time, because the validation is done in code and not by the model. In the current version the tool takes the alloy name (for example `FeCoCrNiNb0.3`) and the code extracts the ratio. The model then called the tool correctly, and its summary matched the tool output.

This is an early experiment, not a reliable agent: the model is small, it was tested on a single request, and its answers should always be checked against the tool output. The success rate in the output refers to the nearest composition in the data (Nb = 0.25), not to the requested one.

To run it, install Ollama, pull the model with `ollama pull llama3.2` and run the notebook. If requests fail with a 503 error while a proxy or VPN is active, create the client with `trust_env=False`, as done in the notebook.

## Requirements

Python 3, pandas, numpy, Jupyter Notebook. Ollama and the ollama Python package (only for the agent demo).

## Author

Mahnaz Keymasi
