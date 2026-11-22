# GLGAN: Global-Local Graph Attention Network

GLGAN classifies subjects from paired fMRI and DTI connectivity. It uses fMRI connectivity profiles as node features and DTI connectivity to define the edges for graph propagation. The global branch combines structural PageRank scores with functional self-attention. The local branch combines fMRI-based neighbor attention with DTI self-attention. ROI-level contrastive alignment and attention fuse the two representations, and an MLP classifies the resulting subject embedding.

## Quick Start

From the repository root, set up the Python environment.

```bash
conda create -n glgan python=3.10
conda activate glgan
python -m pip install -r requirements.txt
```

Place the processed files `ADNI_fmri.mat` and `ADNI_DTI.mat` in `data/raw/`. The loader matches fMRI, DTI, and labels by subject ID. See the [data format](data/README.md) for array shapes, MATLAB keys, and label codes.

Run the default configuration on a binary cohort.

```bash
python main.py
```

For a multiclass input file, select the negative and positive groups with `--negative-classes` and `--positive-classes`. Use the original codes from the dataset.

## Experiments

The [experiment guide](experiments/README.md) covers the eight paired classification tasks, component ablations, dynamic connectivity, bidirectional coupling, class-conditional contrast, and the ABIDE experiment.

For paired data, the default is 10 stratified folds repeated 10 times with base seed `7`. Within each outer training partition, 20% of the samples are used for stratified validation. The checkpoint with the highest validation accuracy is evaluated once on that fold's test set. Training uses Adam for 300 epochs with batch size 20, learning rate `5e-5`, and weight decay `5e-6`. The ROI contrastive loss uses temperature `0.4` and weight `0.5`.

## Outputs and Analysis

The default output directory is `results/`. Use `--output-dir` to set a separate directory for each experiment.

- Each `run_XX/` stores the aligned cohort in `cohort.npz`, test metrics in `fold_metrics.csv`, and the train, validation, and test row indices in `fold_<n>_split.npz`. Model checkpoints are saved unless `--no-checkpoints` is set.
- `aggregate_metrics.csv` reports means and population standard deviations. Paired experiments aggregate run means by default, and ABIDE aggregates test folds. `training.log` records the configuration and seeds for each repetition.

The [analysis guide](analysis/README.md) lists the inputs and commands for ROC, t-SNE, ROI analysis, Elastic Net, and paired fold comparisons.

## Tests

```bash
python -m unittest discover -s tests -v
```

Please cite the corresponding manuscript when using this code.

## Reproducibility Scope

The public release provides the core GLGAN model and selected experiment and analysis workflows. Checking published scores requires the original processed cohorts and split assignments. The paper's baseline implementations, full ablation suite, and Table VI efficiency measurements are outside the scope of this release.
