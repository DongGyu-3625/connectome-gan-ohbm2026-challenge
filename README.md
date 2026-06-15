# Connectome-GAN Reproducibility

This repository provides Jupyter notebooks and documentation for reproducing the OHBM 2026 abstract:

**Connectome-GAN: A Generative Model for Rest-to-Task Functional Connectome Transformation**

This project investigates whether resting-state functional connectivity can be transformed into task-specific functional connectivity using a generative deep learning framework. The repository was prepared for the OHBM 2026–2027 OSSIG Reproducibility Challenge.

## Overview

Connectome-GAN is a CNN-based conditional generative adversarial network designed to generate task-specific functional connectomes from resting-state functional connectomes. The model operates directly on whole-brain functional connectivity matrices derived from the 268-node Shen atlas.

The input is a resting-state functional connectivity matrix, and the target output is the corresponding task-state functional connectivity matrix from the same subject.

## Data

The model uses functional connectivity matrices with shape:

```text
Input:  resting-state FC  -> 268 × 268 × 1
Target: task-state FC     -> 268 × 268 × 1
```

The original dataset consists of 316 subjects from the Human Connectome Project. Due to data-sharing restrictions, raw individual-level HCP data are not redistributed in this repository. This repository provides code and documentation to support reproducibility.

## Model architecture

Connectome-GAN follows a pix2pix-style conditional GAN framework consisting of a generator and a discriminator.

### Generator

The generator is a compact U-Net-like CNN encoder-decoder that maps resting-state FC to task-state FC.

Architecture:

```text
Input: 268 × 268 × 1 resting-state FC

Encoder:
- Conv2D, 128 filters, kernel size 4, stride 2 -> 134 × 134
- Conv2D, 256 filters, kernel size 4, stride 2 -> 67 × 67

Bottleneck:
- Conv2D, 256 filters, kernel size 4, stride 1 -> 67 × 67

Decoder:
- Conv2DTranspose, 128 filters, kernel size 4, stride 2 -> 134 × 134
- Skip connection from the first encoder layer
- Conv2DTranspose, 1 filter, kernel size 4, stride 2 -> 268 × 268

Output:
- Symmetrized task-state FC: 0.5 × (A + Aᵀ)
```

The final symmetrization layer ensures that the generated functional connectivity matrix remains symmetric, matching the structure of empirical connectomes.

### Discriminator

The discriminator is a conditional PatchGAN discriminator. It receives the resting-state FC and either the empirical or generated task-state FC as a two-channel input.

Architecture:

```text
Input pair: [resting-state FC, task-state FC] -> 268 × 268 × 2

- Conv2D, 128 filters, kernel size 4, stride 2 -> 134 × 134
- Conv2D, 256 filters, kernel size 4, stride 2 -> 67 × 67
- Conv2D, 512 filters, kernel size 4, stride 1 -> 67 × 67
- Conv2D, 1 filter, kernel size 4, stride 1 -> 67 × 67 patch logits
```

The discriminator outputs a patch-level real/fake prediction map rather than a single scalar value.

## Loss function

The generator is trained using a combination of adversarial loss, L1 reconstruction loss, and a structure-preserving connectivity loss.

```text
L_G = L_GAN + λ × L_recon

L_recon = L1 + 0.5 × L_structure
```

The structure-preserving loss is based on row-wise and column-wise Pearson correlation between the generated and empirical task functional connectivity matrices. This encourages the generated connectome to preserve node-wise connectivity profiles and network-level structure.

The discriminator is trained with binary cross-entropy loss using real and generated rest-task FC pairs.

## Training strategy

The notebook implements leave-one-out cross-validation. For each run, one subject is held out as the test subject, while the remaining subjects are used for training.

```text
Number of subjects: 316
Training subjects per fold: 315
Held-out test subject per fold: 1
Training steps: 20,000
Batch size: 1
Optimizer: Adam
Initial learning rate: 2e-4
```

During training, the generated task FC is evaluated against the empirical task FC of the held-out subject using:

* spatial correlation
* mean squared error

## Repository contents

```text
notebooks/
    Connectome_GAN_268.ipynb

README.md
LICENSE
```

## Requirements

The notebook was implemented using Python and TensorFlow.

Main dependencies:

```text
tensorflow
numpy
h5py
matplotlib
```

## Usage

Open the notebook:

```text
notebooks/Connectome_GAN_268.ipynb
```

Then run the notebook sections in order:

```text
1. Data loading
2. Loss functions
3. Generator and discriminator definition
4. Training step
5. Leave-one-out training loop
6. Evaluation
```

Before running the notebook, update the dataset path and MATLAB variable names according to your local environment.

## Data availability

The original functional connectivity matrices are derived from the Human Connectome Project. Raw data and individual-level derivatives are not included in this repository because of data-use restrictions. Researchers who wish to reproduce the analysis should obtain access to the HCP dataset through the appropriate data-use agreement and generate the required connectivity matrices following the documented preprocessing procedure.

## License

This repository is released under the MIT License. The license applies only to the code and documentation in this repository, not to the original HCP data.

## Contact

Donggyu Kim
Sungkyunkwan University
