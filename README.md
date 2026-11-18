# ⚡ UPredictor v1.0.0

![Python](https://img.shields.io/badge/Python-3.11-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Build](https://img.shields.io/badge/Build-Passing-brightgreen)
![Version](https://img.shields.io/badge/Version-1.0.0-orange)

**UPredictor** is a command-line tool that predicts **Hubbard U and J values** for custom materials using a pretrained ensemble model (Multilayer Perceptron + Random Forest).

It requires a `.cif` file and the target atomic species, providing a rapid screening approach before performing computationally expensive DFT+U calculations.

---

## 🌍 Overview

The **Hubbard U parameter** plays a crucial role in correcting electron-correlation effects in DFT+U calculations, especially for materials containing localized *d* and *f* orbitals.

However, obtaining reliable U values can be computationally expensive, often requiring complex linear-response or constrained DFT calculations.

UPredictor employs a **hybrid MLP–Random Forest ensemble model** to predict Hubbard U and J parameters for custom materials.

- **Multilayer Perceptron (MLP):** A feedforward neural network designed to capture nonlinear relationships between input descriptors.
- **Random Forest (RF):** An ensemble learning method that provides robust predictions through multiple decision trees.
- **Weighted ensemble:** Combines predictions from both models to generate final U and J estimates.

The models were developed using data obtained from the **Materials Project**.

---

## ⚙️ Workflow

### 1. Data Source 🧩

- Dataset obtained from the [Materials Project](https://materialsproject.org/).
- Includes structural and elemental descriptors alongside reference Hubbard U and J values.

### 2. Feature Engineering 🧱

- Extract structural and elemental properties using `pymatgen`.
- Calculate descriptors including:
  - Average nearest-neighbor distance.
  - Sum of atomic numbers of neighboring atoms.
  - Electronegativity.
  - Ionization energy.
  - Electron affinity.
- Apply polynomial feature expansion and feature scaling.
- Prepare feature matrices for both neural-network and tree-based models.

### 3. Model Architecture 🧬

**Hybrid MLP–Random Forest Ensemble**

**Multilayer Perceptron (MLP):**
- Multiple fully connected layers implemented using PyTorch.
- ReLU activation functions.
- Dropout regularization.
- Learns nonlinear relationships between input descriptors and Hubbard parameters.

**Random Forest (RF):**
- Implemented using Scikit-learn.
- Uses 100 decision trees.
- Captures nonlinear relationships and interactions between input features.

**Ensemble prediction:**

The final prediction is obtained through weighted blending:

\[
\text{Prediction}_{\text{ensemble}}
=0.7\,\text{Prediction}_{\text{RF}}
+0.3\,\text{Prediction}_{\text{MLP}}
\]

The same ensemble weighting is applied to both U and J predictions.

### 4. Training 🔥

- **PyTorch:** MLP implementation.
- **Scikit-learn:** Random Forest implementation.
- **MLP loss function:** Mean Squared Error (MSE).
- **MLP optimizer:** Adam with weight decay.
- **Random Forest:** 100 estimators, with out-of-bag scoring enabled.
- **MLP regularization:** Dropout and weight decay.
- **Early stopping:** Based on training loss.

### 5. Prediction 🚀

- Provide a material structure in CIF format.
- Specify the target chemical species.
- Extract the corresponding structural and elemental descriptors.
- Apply the trained preprocessing transformations.
- Generate MLP, Random Forest, and ensemble predictions for Hubbard U and J.

---

## 📊 Model Performance

The performance of UPredictor was evaluated using a separate benchmark training and testing experiment.

The benchmark used 603 samples, divided into training and testing subsets using an 80:20 split. Polynomial feature transformation and standardization were fitted exclusively on the training data to prevent test-set information leakage during preprocessing.

**Benchmark configuration**

| Parameter | Configuration |
|---|---|
| Dataset size | 603 samples |
| Training samples | 482 |
| Testing samples | 121 |
| Training/testing split | 80:20 |
| Random state | 42 |
| Input descriptors | 5 |
| Polynomial features | Degree 2, interaction-only |
| Feature scaling | StandardScaler |
| MLP architecture | 3 hidden layers, 128 neurons each |
| Random Forest | 100 estimators |
| Ensemble weighting | 70% RF + 30% MLP |

### 5.1 Hubbard U prediction

| Model | R² | MAE (eV) | MSE (eV²) | RMSE (eV) |
|---|---:|---:|---:|---:|
| MLP | 0.9415 | 0.3078 | 0.2043 | 0.4520 |
| Random Forest | 0.9524 | 0.2688 | 0.1660 | 0.4074 |
| **Ensemble** | **0.9548** | **0.2648** | **0.1579** | **0.3974** |

### 5.2 Hubbard J prediction

| Model | R² | MAE (eV) | MSE (eV²) | RMSE (eV) |
|---|---:|---:|---:|---:|
| MLP | 0.5403 | 0.1164 | 0.0373 | 0.1930 |
| Random Forest | 0.6410 | 0.1027 | 0.0291 | 0.1706 |
| **Ensemble** | **0.6462** | **0.1021** | **0.0287** | **0.1693** |

### 5.3 Performance summary

The ensemble model achieved the best overall performance among the three evaluated models for both Hubbard U and J prediction.

Key results:

- **Hubbard U:** R² = 0.9548 and RMSE = 0.3974 eV.
- **Hubbard J:** R² = 0.6462 and RMSE = 0.1693 eV.
- The Random Forest model outperformed the MLP when evaluated independently for both targets.
- Weighted blending of the two models provided modest additional improvements over Random Forest alone.

The results indicate that the current descriptor set captures variations in Hubbard U more effectively than variations in Hubbard J.

**Evaluation note:** These results represent a single reproducible 80:20 holdout benchmark with `random_state=42`. They should not be interpreted as 5-fold cross-validation results or as a statistical estimate of performance across all possible train/test splits.

The benchmark models and evaluation outputs are maintained separately from the original UPredictor v1.0.0 model artifacts.

---

## 🚀 Quick Start

### 🧩 1. Clone & Setup

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/Thejanonthehub/hub_upredictor_Tjn.git
cd hub_upredictor_Tjn
```

Create and activate a virtual environment.

**macOS / Linux**

```bash
python3 -m venv upd_env
source upd_env/bin/activate
```

**Windows (Command Prompt)**

```bash
py -m venv upd_env
upd_env\Scripts\activate
```

Install the required dependencies:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### ⚙️ 2. Install the CLI Tool

From inside the project directory, install UPredictor:

```bash
python -m pip install .
```

This installs UPredictor and registers the `upredictor` command-line interface, allowing you to launch the application from anywhere within the activated virtual environment.

### 🧠 3. Run the Predictor

Simply type:

```bash
upredictor
```

Follow the interactive prompts to provide the path to your CIF file and the target chemical element.

**Note:** Make sure your virtual environment is activated before running UPredictor.

**Startup banner:**

```text
   __  ______                ___      __            
      / / / / __ \________  ____/ (_)____/ /_____  _____
     / / / / /_/ / ___/ _ \/ __  / / ___/ __/ __ \/ ___/
    / /_/ / ____/ /  /  __/ /_/ / / /__/ /_/ /_/ / /    
    \____/_/   /_/   \___/\__,_/_/\___/\__/\____/_/     

   Welcome to the UPredictor ML Engine!
   ------------------------------------
```

**Interactive prompts:**

```text
Enter path to CIF file: /path/to/material.cif
Enter species to analyze (e.g., Ag, C, H): Fe
```

**Example prediction output:**

```text
Predicted Properties:
MLP Prediction (U, J): [4.32, 0.71]
RF Prediction (U, J): [4.28, 0.68]
Ensemble Prediction (U, J): [4.30, 0.70]
```

*The values above are illustrative examples, not measured benchmark results.*

### ⚠️ 4. Troubleshooting

**Error:**

```text
FileNotFoundError: './engine/hubbard_uj_poly.pkl'
```

**Solution:**

Ensure the `engine/` folder exists in the project directory and includes:

```text
engine/
├── hubbard_uj_poly.pkl
├── hubbard_uj_scaler.pkl
├── hubbard_uj_rf.pkl
└── hubbard_uj_mlp.pth
```

### 💡 5. Tip

UPredictor works offline after installation and supports CIF files compatible with `pymatgen`.

### 🧩 6. Example Workflow

1. Prepare your custom `.cif` file.
2. Run `upredictor` from the terminal.
3. Enter the path to your CIF file.
4. Specify the target atomic species (e.g., Fe, C, Ni).
5. Obtain predicted Hubbard U and J values from the MLP, Random Forest, and ensemble models.

---

## 📌 Future Development

Potential future improvements include:

- Expanded training datasets.
- Improved prediction accuracy for Hubbard J.
- Validation using repeated train/test splits and cross-validation.
- Additional structural and electronic descriptors.
- Integration with automated computational materials workflows.

---
