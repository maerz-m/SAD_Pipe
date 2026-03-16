# Synthetic Assessment Data Pipeline Engine (SAD-Pipe)

A modular framework for simulating, scoring, and analyzing multiple‑choice exam data using both Classical Test Theory (CTT) and Item Response Theory (IRT).  
The toolkit supports realistic data generation, psychometric modeling, and publication‑quality diagnostics.

---

## Purpose
The dataset is designed to provide reproducible synthetic response data for prototyping and testing downstream analysis workflows, including scoring pipelines, item‑level diagnostics, distractor behavior analysis, and exploratory modeling.
The simulation integrates several components to approximate realistic test‑taking behavior:
- 2PL‑like knowledge model Item difficulty b and discrimination a parameters shape the probability of a correct response.
- Person‑level behavioral traits: Latent ability (theta_true), carelessness, guessiness, and testwiseness jointly influence response patterns.
- Diverse item types: Regular, regular_clean, diluted, and misleading items allow stress‑testing of analysis methods under varied conditions.
- Distractor attractiveness profiles

Each item includes structured distractor behavior to support fine‑grained distractor‑level diagnostics.
---

### Item Types

The simulation includes several item types designed to stress‑test scoring and diagnostic workflows:

- **regular**
  Standard 2PL items with well‑behaved distractors and no special structure.

- **regular_clean**  
  High‑quality items with minimal distractor noise and no misleading cues.  
  Useful as a baseline for comparison.

- **diluted**  
  Diluted items are useful for stress‑testing scoring models and evaluating how
  robustly they handle poorly functioning or weakly aligned items.

- **misleading**  
  Items containing distractors that appear attractive to certain students  
  (e.g., low testwiseness or high guessiness).  
  Designed to reveal vulnerabilities in scoring and estimation.

---
### Distractor Profiles

Each item includes a structured distractor profile that defines how attractive  
each distractor is to different types of students.

- **Base attractiveness**  
  Each distractor has a baseline weight reflecting how plausible it appears.

- **Ability modulation**  
  Low‑ability students are more likely to select high‑weight distractors.  
  High‑ability students suppress distractor weights more effectively.

- **Behavioral modulation**  
  - **Guessiness** triggers uniform random selection across all options.  
  - **Testwiseness** influences distractor elimination or miselimination.  
  - **Carelessness** introduces occasional random deviations.

Distractor weights are renormalized after any elimination step to preserve  
valid probability distributions.

---

### Test‑Taker Traits

Each simulated student is assigned a set of behavioral traits that interact  
with their latent ability (θ_true) to produce realistic response patterns.

- **theta_true**  
  The underlying ability parameter used in the generative 2PL model.

- **testwiseness**  
  Strategic behavior in using item cues and eliminating distractors.  
  Positive values indicate productive strategy use;  
  negative values indicate counterproductive behavior  
  (e.g., eliminating good distractors or over‑choosing the middle option).

- **guessiness**  
  Tendency to guess impulsively.  
  When triggered, the student selects uniformly among all options,  
  producing chance‑level correctness.

- **carelessness**  
  Random lapses in attention or execution.  
  Introduces noise without systematic bias.

- **slip**  
  Probability of an unintended incorrect response despite knowing the answer.  
  Contributes to underestimation bias in ability recovery.

---


### How the Response Model Works

Each response is generated through a combination of cognitive ability,  
behavioral traits, and item‑specific distractor structure.

1. **Ability‑based probability of correctness (2PL model)**  
   Each item has difficulty (b) and discrimination (a).  
   The probability of a correct response is:

$$
   P(\text{correct}) = \sigma(a(\theta_{\text{true}} - b))
$$

2. **Slip**  
   Even when the student “knows” the item, a slip may cause an incorrect response.

3. **Behavioral modulation**  
   Behavioral traits modify the response process:

   - **Testwiseness**  
     Influences distractor elimination or miselimination.  
     Negative values may cause the student to remove good distractors  
     or over‑select the middle option.

   - **Guessiness**  
     When triggered, the student selects uniformly among all options  
     (correct + distractors), producing chance‑level correctness.

   - **Carelessness**  
     Introduces occasional random deviations from the intended response.

4. **Distractor selection**  
   If the student does not answer correctly via the 2PL process,  
   distractor selection is governed by item‑specific distractor weights,  
   modified by the student's behavioral traits.  
This layered structure produces realistic, heterogeneous response patterns  
that challenge scoring models and support rich diagnostic analysis.

---

## Repository Overview

### Core Notebooks 

1. **Python: `notebooks/01_simulation.ipynb` or R: `rquarto/01_simulation.Rmd`** 

   Generates synthetic exam data with configurable parameters and realistic test‑taker behavior.  
   Produces:
   - Binary answer matrix  
   - Scored responses  
   - Test‑taker metadata (ability, behavioral traits)  
   - Item metadata (difficulty, discrimination, distractor structure)

2. **Python:`notebooks/02_analysis.ipynb` or R: `rquarto/02_analysis.Rmd`** 

   Performs a full psychometric analysis of the simulated exam.  
   Includes:
   - CTT item statistics  
   - IRT (2PL) parameter estimation  
   - Person‑fit diagnostics  
   - Item‑level and test‑level visualizations

---

## Data Flow

```
01_simulation.ipynb/.Rmd → data/simulation_results → 02_analysis.ipynb/.Rmd

```

The simulation notebook saves all generated data to `data/simulation_results`, and the analysis notebook loads these files automatically.

---

## Project Structure

```
project/
│
│   CITATION.cff
│   LICENSE.txt
│   README.md
│   requirements.txt
│   
├───data
│   ├───analysis_results
│   │   │  item_analysis.pkl
│   │   │  item_analysis.Rds
│   │   │  item_analysis_py.csv
│   │   │  item_analysis_r.csv
│   │   │  testtakers_post_analysis.pkl
│   │   │  testtakers_post_analysis.Rds
│   │   │  testtakers_post_analysis_py.csv
│   │   │  testtakers_post_analysis_r.csv
│   │   │  
│   │   └───item_plots/
│   │         misleading/
│   │         diluted/
│   │         regular/
│   │         regular_clean/
│   │               
│   └───simulation_results
│          answer_matrix.pkl
│          answer_matrix.rds
│          answer_matrix_py.csv
│          answer_matrix_R.csv
│          item_metadata.pkl
│          item_metadata.rds
│          item_metadata_py.csv
│          item_metadata_R.csv
│          scored_responses.pkl
│          scored_responses.rds
│          scored_responses_py.csv
│          scored_responses_R.csv
│          testtakers.pkl
│          testtakers.rds
│          testtakers_py.csv
│          testtakers_R.csv
│           
├───notebooks
│      01_simulation.ipynb
│      02_analysis.ipynb
│             
└───rquarto
        01_simulation.html
        01_simulation.Rmd
        02_analysis.html
        02_analysis.Rmd     
```

## Getting Started - Option A:  Run the Full Simulation 


### In Python: Create and Activate an Environment

Using conda:

```bash
conda create -n exam_processing python=3.11 -y
conda activate exam_processing
```

#### Install Dependencies

```bash
# Core scientific stack
conda install numpy pandas matplotlib seaborn scipy

# Bayesian modeling (optional)
conda install pymc arviz

# Interactive visualizations (optional)
conda install plotly
```
### In R and RStudio: 
Optional: create a project

Install all needed packages. 

Simulation:
```R
install.packages("tidyverse")
install.packages("fs")
install.packages("rlang")
install.packages("patchwork")
install.packages("ggplot2")
```

Analysis:
```R
install.packages("tidyverse")
install.packages("fs")
install.packages("here")
install.packages("rstan")
install.packages("patchwork")
install.packages("ggplot2")
```

### Usage 
#### 0. Configure the Test and Student Population
Open `01_simulation`, run the cells until Step 0. This is the setuo of the test
- choose the number of items and the proportions of each item type,
- adjust the ability distribution (θ) for the student population,
- preview the resulting θ curve and summary statistics.
These configuration cells are optional but designed for experimentation.
You can keep the defaults or tune the parameters until the test and population match your scenario.
Once you are satisfied with the setup, continue with the simulation.

#### 1. Run the Simulation

Open `01_simulation` and execute all cells.  
The notebook generates:

- `answer_matrix` — raw responses (option labels)  
- `scored_responses` — correctness and total scores  
- `testtakers` — ability and behavioral traits  
- `item_metadata` — item parameters and distractor structure  
- `irt_2pl_idata.nc` — cached PyMC samples (InferenceData)

All files are saved to the `data/` directory.

#### 2. Analyze the Results

Open `02_analysis` and run the analysis pipeline:

- CTT difficulty and discrimination  
- IRT 2PL estimation  
- Item characteristic curves  
- Distractor analysis  
- Person‑fit and ability recovery  
- Item‑level diagnostic panels

---

### Key Features

#### Simulation
- Configurable test length and sample size  
- Realistic distractor behavior  
- Behavioral traits (carelessness, guessing, testwiseness)  
- Ground‑truth parameters for validation

#### Analysis
- CTT difficulty and discrimination  
- Corrected item–total correlations  
- IRT 2PL estimation via PyMC  
- ICCs and model‑fit diagnostics  
- Person‑fit and ability recovery plots

#### Visualization
- Item response distributions  
- IRT characteristic curves    
- Six‑panel item diagnostic figures  

---


### Configuration

Simulation settings are controlled via `SimulationConfig` in `01_simulation`.

The notebook provides dedicated cells where you can:
- define the test (number of items and item‑type proportions),
- construct and preview the ability distribution (θ),
- optionally pass a custom θ array into the simulation.
Once the configuration cells are set, the notebook builds the SimulationConfig object and runs the full simulation pipeline.

The simulation returns a dictionary with four DataFrames:

- `answer_matrix`  
- `scored_responses`  
- `item_metadata`  
- `testtakers`  

---

### Reproducibility

Running the notebooks with the same configuration and seed produces identical outputs.  
For consistent results, use:

```
Python: Kernel → Restart & Run All

R: Restart R and Run all Chunks
```

---
## Option B — Use the Provided CSV Files (No Python Setup Required)
If you don’t want to run the simulation, you can directly load the pre‑generated CSV files located in:
- data/simulation_results/
- data/analysis_results/


## License

This project is released under the MIT License.  
See `LICENSE` for details.

---

## Citation

If you use this toolkit in academic work, please cite it using the metadata in `CITATION.cff`.