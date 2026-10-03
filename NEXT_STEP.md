# NEXT STEP

Stage E, Step 6 - Experiment manifest persistence.

Goal: a frozen ExperimentManifest model (Blueprint Section 35) capturing
every knob of a run, and a writer that persists it as the experiments row's
config_json. The engine accepts a manifest instead of a bare experiment_id.
After E6, an experiment can be reproduced from the journal alone.

Awaiting: instructor to issue the step contract.
