# ML Experiment Workflow

## Rules
1. Identify the baseline before changing anything.
2. Do not overwrite artifacts.
3. Isolate config, feature, model, evaluation, and reporting changes so results
   stay attributable.
4. Run cheap sanity checks before full runs.
5. Record data version, config, seed, and code state where applicable.

## Output
- baseline used
- isolated change introduced
- sanity checks run
- full run approval status
- artifact paths
