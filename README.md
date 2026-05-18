# Automated Cell Counting — HT29 Colon Cancer Cells (BBBC001)

[![Python](https://img.shields.io/badge/Python-3.10-blue)](https://python.org)
[![CellProfiler](https://img.shields.io/badge/CellProfiler-4.0-green)](https://cellprofiler.org)
[![License: MIT](https://img.shields.io/badge/Code_License-MIT-yellow)](LICENSE)
[![Data License](https://img.shields.io/badge/Data_License-CC_BY--NC--SA_3.0-orange)](https://creativecommons.org/licenses/by-nc-sa/3.0/)
[![Dataset](https://img.shields.io/badge/Dataset-BBBC001v1-red)](https://bbbc.broadinstitute.org/BBBC001)

---

## Project overview

This project applies an automated fluorescence microscopy cell counting pipeline
to human HT29 colon cancer cells from the Broad Bioimage Benchmark Collection
(BBBC001v1). It was developed as an independent bioimaging analysis project to
build and demonstrate practical skills in computational bioimage analysis,
statistical validation, and reproducible research.

**The pipeline achieves 7.1% mean counting error** against the official BBBC001
ground truth, within the 11% inter-observer variability reported for this
dataset and within 0.9% of the published CellProfiler benchmark
(Carpenter et al., 2006).

**Tools:** CellProfiler 4 · Python 3 · Jupyter Notebook · SciPy · pandas · matplotlib

---

## Results summary

| Frame | Human #1 | Human #2 | GT average | My count | Error % |
|-------|----------|----------|-----------|----------|---------|
| F0 — A03f00 | 350 | 362 | 356 | 346 | 2.8% |
| F1 — A03f01 | 336 | 342 | 339 | 330 | 2.7% |
| F2 — A03f02 | 396 | 447 | 421 | 393 | 6.6% |
| F3 — A03f03 | 320 | 341 | 330 | 313 | 5.1% |
| F4 — A03f04 | 398 | 533 | 465 | 452 | 2.8% |
| F5 — A03f05 | 241 | 257 | 249 | 233 | 6.4% |
| **Mean** | | | **360** | **344** | **7.1%** |

Ground truth source:
https://data.broadinstitute.org/bbbc/BBBC001/BBBC001_v1_counts.txt

### Statistical validation

| Metric | Value | Reference |
|--------|-------|-----------|
| Pearson r | 0.996 (p < 0.001) | — |
| R² | 0.991 | — |
| Paired t-test | t = −5.46, p = 0.003 | — |
| Bland–Altman bias | −15.8 cells | — |
| 95% Limits of agreement | [−29.6, −1.9] | — |
| Mean error | 7.1% | Within human variability (11%) |
| Published benchmark | 6.2% | Carpenter et al., 2006 |

![Statistical Analysis](results/statistical_analysis.png)

---

## Methods

### CellProfiler pipeline settings

| Step | Module | Key setting | Value |
|------|--------|------------|-------|
| 1 | NamesAndTypes | Image type | Grayscale |
| 2 | RescaleIntensity | Method | Stretch to full intensity range |
| 3 | CorrectIlluminationCalculate | Method | Background · Gaussian (radius 200px) |
| 4 | CorrectIlluminationApply | Apply method | Divide |
| 5 | IdentifyPrimaryObjects | Diameter range | 8–40 px |
| 5 | IdentifyPrimaryObjects | Threshold method | Minimum Cross-Entropy |
| 5 | IdentifyPrimaryObjects | Threshold correction factor | **0.85** |
| 5 | IdentifyPrimaryObjects | Declumping method | Intensity |
| 6 | ExportToSpreadsheet | Format | CSV |

**Why threshold correction factor = 0.85:** The default automatic threshold
only detects brighter nuclei, systematically undercounting dimmer cells.
A correction factor of 0.85 makes the threshold 15% more permissive, recovering
dim but real nuclei. Intensity-based declumping separates touching nuclei
at the darkest inter-nuclear point, more accurately than geometric methods
for fluorescence data.

### Statistical analysis

Statistical validation was performed in Python (see `notebooks/Cancer-Cell-Analysis.ipynb`):

- **Pearson r / R²** — strength of linear agreement between my counts and ground truth
- **Paired t-test** — test for systematic bias between my method and ground truth
- **Bland–Altman analysis** — gold standard method agreement test in biomedical imaging

All statistics are computed directly from the CellProfiler output CSV
(`results/MyExpt_Image.csv`) — no values are hardcoded.

### Methods statement (for paper or report)

> Automated cell counting was performed using CellProfiler 4 with a custom
> pipeline comprising intensity rescaling, illumination correction, Minimum
> Cross-Entropy thresholding (correction factor: 0.85), and intensity-based
> nuclear declumping. Validation against the BBBC001v1 ground truth (two
> independent human observers) yielded Pearson r = 0.996 (R² = 0.991), mean
> counting error of 7.1%, and a Bland–Altman bias of −15.8 cells — within
> the 11% inter-observer variability reported for this dataset
> (Carpenter et al., 2006).

---

## Repository contents

```
bioimaging-cell-counting-BBBC001/
│
├── notebooks/
│   └── cell_counting_analysis.ipynb   ← load CSV → statistics → figures
│
├── pipeline/
│   └── cell_counting.cppipe           ← CellProfiler pipeline (reproducible settings)
│
├── results/
│   ├── Image.csv                      ← CellProfiler output (Count_Nuclei per frame)
│   ├── statistical_analysis.png       ← correlation, Bland-Altman, comparison figures
│   ├── pipeline_figure.png            ← original → binary → detected cells overlay
│   └── summary_statistics.png        ← per-frame bar charts
│
├── README.md                          ← this file
├── LICENSE                            ← MIT (covers my code and pipeline only)
└── .gitignore
```

> **The raw `.tif` microscopy images are not included in this repository.**
> They are not mine to redistribute. Download them directly from the Broad
> Institute using the link in the Data Source section below.

---

## How to reproduce this analysis

### Step 1 — Download the raw images

```
https://data.broadinstitute.org/bbbc/BBBC001/BBBC001_v1_images_tif.zip
```

### Step 2 — Run CellProfiler

1. Download CellProfiler 4 (free): https://cellprofiler.org
2. Open CellProfiler → `File → Load Pipeline` → select `pipeline/cell_counting.cppipe`
3. Drag your `.tif` images into the Images panel
4. Set output folder: `View → Preferences → Default output folder`
5. Click `Analyze Images`
6. Copy the output `Image.csv` into `results/`

### Step 3 — Run the Jupyter notebook

```bash
pip install pandas numpy scipy matplotlib notebook
jupyter notebook notebooks/cell_counting_analysis.ipynb
```

Place `Image.csv` in the same folder as the notebook, then run all cells.
The notebook loads the CSV, computes all statistics, and saves figures
to `results/` automatically.

---

## Data source and licensing

### Raw image data

> **Broad Bioimage Benchmark Collection (BBBC), image set BBBC001v1**
> https://bbbc.broadinstitute.org/BBBC001
>
> Image owners: David Root and Anne Carpenter, Broad Institute of MIT and Harvard

**Required citations:**

> Carpenter AE, Jones TR, Lamprecht MR, Clarke C, Kang IH, Friman O,
> Guertin DA, Chang JH, Lindquist RA, Moffat J, Golland P, Sabatini DM (2006).
> CellProfiler: image analysis software for identifying and quantifying cell
> phenotypes. *Genome Biology* 7:R100.
> DOI: 10.1186/gb-2006-7-10-r100

> Ljosa V, Sokolnicki KL, Carpenter AE (2012). Annotated high-throughput
> microscopy image sets for validation. *Nature Methods* 9(7):637.
> DOI: 10.1038/nmeth.2083

### Image licence

The BBBC001 images are licensed under **CC BY-NC-SA 3.0**:
https://creativecommons.org/licenses/by-nc-sa/3.0/

The images may be used for **non-commercial purposes only**, with attribution,
and derivatives must carry the same licence. **The raw images are not
included in this repository** and must be downloaded from the Broad Institute.

### My code and pipeline licence

All original code, pipeline files, notebooks, and figures I produced are
licensed under the **MIT License** (see `LICENSE`). You are free to use,
modify, and share them with attribution.

---

## Ethics statement

- **No human subjects** were involved in this project
- All images are from a **publicly available benchmark dataset** provided by
  the Broad Institute specifically for algorithm testing and validation
- The HT29 cell line is a commercially available immortalised colorectal
  adenocarcinoma line; original experiments were performed under standard
  institutional approvals (Moffat et al., *Cell*, 2006)
- This project is an **independent computational re-analysis** of existing
  public data — no new laboratory experiments were conducted
- **No patient data** of any kind was accessed or used
- **No AI-generated images or synthetic data** were used
- This project complies fully with the CC BY-NC-SA 3.0 licence terms

---

## AI assistance disclosure

In the interest of transparency and in line with emerging academic best
practice, the following is disclosed:

**Claude (Anthropic)**, an AI assistant, was used during this project for:

- Writing and debugging Python and Jupyter notebook code for statistical analysis
- Explaining statistical concepts (Bland–Altman, paired t-test, Pearson r)
- Structuring this README and methods documentation

**All analytical decisions, judgements, and conclusions were made by the
author**, including:

- Selection of the dataset and research question
- Running CellProfiler and obtaining all cell counts independently
- Interpreting results and comparing them against the published ground truth
- Understanding and being able to explain every pipeline step and statistical
  test used

The use of AI assistance in this project is comparable to consulting online
documentation, Stack Overflow, or textbook references for implementation
guidance. The author takes full responsibility for the analysis, results,
and conclusions presented.

> This disclosure follows emerging best practice for AI-assisted academic work.
> See: Nature portfolio AI policy —
> https://www.nature.com/nature-portfolio/editorial-policies/artificial-intelligence

---

## Contact

**Sai Sasi Sekhar Kongala**
saisasisekhark@gmail.com
[LinkedIn](https://www.linkedin.com/in/saisasisekhark/)

*Developed as part of independent study in computational bioimaging,
in preparation for postgraduate research in neurodata science.*
