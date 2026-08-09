# ADS2002 Counting Molecules Project Workflow

## 1. Project Definition

### 1.1 Proposed Title

**Refining Molecular Fingerprints for Automated Counting and Unsupervised Categorisation of Molecules in STM Images**

### 1.2 Project Aim

This project uses the Counting Molecules method developed by Hellerstedt et al. as its baseline. It will first reproduce the three examples presented in the paper, then improve the algorithm through more robust background correction, segmentation of touching objects, and molecular fingerprint design. Independent manual annotations will be used to evaluate automated detection, counting, and morphological categorisation. The finalised pipeline will then be applied to all independent STM images, and the client will be advised about the image conditions under which the method is reliable and the situations that require manual review.

The main project sequence is:

```text
Reproduce the published method
-> Establish manual ground truth
-> Analyse failure modes
-> Improve preprocessing and segmentation
-> Develop molecular fingerprints and categorisation
-> Quantitatively compare the baseline and improved methods
-> Extend the selected method to the full dataset
-> Produce client recommendations
```

The APT condition analysis is an application of the algorithm. It does not replace the central tasks of molecule detection, counting, and categorisation.

## 2. Research Questions and Hypotheses

### 2.1 Research Questions

1. Can the published method produce similar counting and classification results on the three example files supplied locally?
2. Can improved preprocessing and watershed segmentation increase detection accuracy when images contain uneven backgrounds, scan-line artefacts, or touching molecules?
3. Which set of molecular fingerprints is most suitable for distinguishing molecular morphologies and, exploratorily, separating APT, MgPc, and Helicene?
4. Can the improved pipeline transfer to STM images that were not used during initial development and tuning?
5. Within the APT data, what descriptive differences in molecular density and morphology proportions are observed under different experimental conditions?

### 2.2 Testable Hypotheses

- **H1:** Compared with the published baseline, robust preprocessing and watershed segmentation will reduce counting error and improve detection F1.
- **H2:** Combining Zernike moments with geometric and intensity features will agree more closely with manual morphology labels and known molecular-system labels than using either Zernike or geometric features alone.
- **H3:** The improved pipeline will be more stable on images containing background tilt, scan-line artefacts, strong edge artefacts, and clustered molecules.

## 3. Data Scope

### 3.1 Primary Analysis Data

The primary analysis includes all `.p` and `.sxm` files: two `.p` files and twelve `.sxm` files, giving 14 files in total. However, the following files contain exactly the same image data:

- `Hellerstedt APT/111_test_data.p`
- `Hellerstedt APT/Ag111_APT_111.sxm`

The formal statistical analysis will therefore contain **13 independent images**. The duplicate files will only be used to verify that the two file formats produce identical readings.

`Christian Wackerlin/1705_6.txt` will be retained as an additional data-reading example, but it will not be included in the main model comparison because its experimental condition, physical scale, and relationship to the paper benchmarks are currently unclear.

### 3.2 Data Used for Reproducing the Paper

| Local file | Paper example | Published algorithm output | Role in this project |
| :--- | :--- | :--- | :--- |
| `Hellerstedt APT/Ag111_APT_CO_044.p` | Figure 2 | 145 contours, 6 categories | APT benchmark 1 |
| `Hellerstedt APT/111_test_data.p` | Figure 3 | 87 contours, 9 categories | APT benchmark 2 |
| `Stetsovych Helicene/Helicene_Ag(111)008.sxm` | Figure 4 | 252 contours, 7 categories after chirality processing | Helicene benchmark |

These values are outputs from the published algorithm, not manual ground truth. Differences caused by software versions, parameter interpretation, or data-reading behaviour are acceptable during reproduction only if their sources are identified and explained.

### 3.3 Data Roles

| Data role | Images | Purpose |
| :--- | :--- | :--- |
| Baseline benchmark | The three paper example files | Reproduction, manual annotation, and baseline-versus-improved comparison |
| Easy extension | APT 007-010 and Helicene 020 | Assess transferability and basic stability |
| Hard cases | UV/annealed APT, MgPc, and Helicene 021 | Assess clustering, scan lines, and strong edge artefacts |
| Duplicate check | APT 111 in `.p` and `.sxm` formats | Verify reading consistency without double-counting |

### 3.4 The Three Label Types Must Remain Separate

- `species`: APT, MgPc, or Helicene.
- `condition`: an image-level experimental or imaging condition, such as Baseline, CO-functionalised tip, UV exposure, or annealing.
- `morphology`: a molecule-level observation, such as isolated, three-lobe, dimer, clustered, chiral form, or uncertain.

`condition` is not a class to be predicted from an individual molecule. A CO-functionalised tip is an imaging condition and must not be conflated with the effects of sample treatments such as UV exposure or annealing.

## 4. Staged Project Workflow

### Stage 0: Data Registration and Quality Assessment

**Input:** All `.p` and `.sxm` files, plus the additional `.txt` file.

**Processing:**

1. Develop a common reader that returns a two-dimensional height matrix and nm/pixel.
2. Record the filename, format, shape, scan range, species, condition, and data source.
3. Check for NaN values, extreme values, scan direction, background tilt, scan lines, and edge artefacts.
4. Identify duplicate images through direct array comparison or hashing.
5. Generate a raw-image contact sheet and an intensity summary for every image.

**Outputs:**

- `outputs/tables/image_manifest.csv`
- `outputs/figures/data_quality/raw_contact_sheet.png`
- `outputs/tables/duplicate_check.csv`
- A documented list of data-quality issues

**Completion criterion:** All 14 `.p`/`.sxm` files are readable; the identities of all 13 independent images are clear; duplicate files are not counted twice in later analysis.

### Stage 1: Reproduction of the Published Baseline

**Input:** The three example images used in the paper.

**Processing:**

1. Use the authors' reading logic to obtain the image and physical pixel scale.
2. Apply Gaussian low-frequency background subtraction, maximum-value normalisation, and two-dimensional plane fitting.
3. Use an Otsu-scaled local threshold to extract closed contours.
4. Filter edge contours and contours that are too small according to the authors' rules.
5. Generate centred molecular templates.
6. Calculate Zernike moments, normalised contour length, and normalised maximum height.
7. Reproduce BIRCH, Agglomerative Clustering, and hand-selected-exemplar Affinity Propagation.
8. Reproduce mirror-template chirality classification for the Helicene example.

**Outputs:**

- Baseline preprocessing figures, binary masks, numbered contour figures, and template grids
- Baseline classification figures and category histograms
- `outputs/tables/baseline_results.csv`
- A comparison between the published and locally reproduced outputs

**Completion criterion:** All three examples run successfully; interpretable contours and categories are produced; every parameter is recorded; similarity to the published counts is not treated as proof of accuracy.

### Stage 2: Manual Ground Truth

**Minimum annotation scope:** The three paper benchmarks.

**Recommended extension:** Add one clustered APT image, the MgPc image, and Helicene 021 so that manual validation covers the main failure modes.

**Annotation rules:**

1. Mark the visually identified molecular centre on the raw image or an image for which only the display range has been adjusted.
2. Do not display algorithm predictions during manual annotation, to avoid confirmation bias.
3. Mark touching molecules separately when their centres remain distinguishable.
4. Label clusters for which the number of molecules cannot be determined as `uncertain_cluster`.
5. Label steps, dark pits, and scan defects as artifacts rather than molecules.
6. Label incomplete objects on the image boundary as `partial_edge`.
7. Use a predefined morphology label guide; use `uncertain` when morphology cannot be determined.
8. Conduct a second independent review and record the reason for every revision.

**Annotation table fields:**

```text
image_id, object_id, x_pixel, y_pixel, morphology,
is_clustered, is_partial_edge, confidence, annotator, notes
```

**Outputs:**

- `annotations/manual_annotations.csv`
- `annotations/annotation_guide.md`
- Manual overlay figures and manual totals for each benchmark

**Completion criterion:** Every annotation is traceable to coordinates in the original image; uncertain objects and artifacts follow explicit rules; manual results are independent of algorithm-generated contours.

### Stage 3: Improved Preprocessing

**Candidate methods:**

- Row-median subtraction or low-order row-wise fitting for scan-line artefacts
- Robust two-dimensional plane fitting for global tilt
- Large-scale Gaussian or morphological background estimation for slowly varying backgrounds
- Percentile normalisation to prevent a single very bright pixel from controlling the intensity range
- An ROI mask for the strong bright edge in Helicene 021

These background-correction methods must not all be stacked unconditionally. Each step should only be retained after comparing raw and preprocessed images and confirming that molecular signals are preserved. Wherever possible, filtering scales should be converted from nanometres to pixels rather than fixed at 50 pixels for images of different resolutions.

**Outputs:**

- Raw, estimated-background, flattened, and normalised comparisons for each image
- `outputs/tables/preprocessing_parameters.csv`
- `outputs/masks/valid_roi/`

**Completion criterion:** Background variation and scan lines are reduced without substantially weakening molecular peaks, sizes, or morphologies; all parameters are reproducible.

### Stage 4: Improved Segmentation and Counting

**Processing:**

1. Compare global Otsu and adaptive/local thresholding.
2. Clean the mask using opening, closing, small-object removal, and border clearing.
3. Calculate a distance transform for touching objects.
4. Generate reliable markers and apply watershed segmentation.
5. Filter artifacts using physical area, dimensions, intensity, and ROI rules.
6. Export the centre, contour, label ID, and count for each detected object.

**Outputs:**

- Threshold masks, distance maps, watershed labels, and final detections
- `outputs/tables/detected_objects.csv`
- `outputs/tables/image_counts.csv`

**Completion criterion:** Every image has a complete diagnostic sequence:

```text
Raw -> Flattened -> Threshold -> Watershed -> Final detections
```

### Stage 5: Molecular Fingerprints and Categorisation

**Features extracted from each object:**

- Area, perimeter, and equivalent diameter
- Eccentricity, solidity, extent, and aspect ratio
- Mean, maximum, and integrated intensity
- Zernike moments
- Local texture or a radial profile where justified

Size-based features should retain physical units. The following feature sets must be compared separately:

1. Geometry and intensity only
2. Zernike only
3. Combined fingerprints

Unsupervised categorisation will primarily use BIRCH, Agglomerative Clustering, or Affinity Propagation. The analysis has two levels. The first examines morphology clusters within the same molecular system. The second explores whether the fingerprints separate the known APT, MgPc, and Helicene systems. Manual morphology labels will be used to evaluate clustering agreement, rather than assigning unsupported chemical identities to arbitrary cluster numbers.

Cross-species analysis must be validated by grouping observations at the image level. Molecules from the same image must not be randomly divided between training and test sets and then used to claim generalisation. Because MgPc is represented by only one image, its results can only demonstrate separability within the available image, not generalisation to unseen MgPc images. If the number and consistency of manual labels are sufficient, a simple supervised model may be added as an extension, but it must not replace the main unsupervised-categorisation objective.

**Outputs:**

- `outputs/tables/molecule_features.csv`
- Fingerprint correlations, PCA/UMAP, and cluster visualisations
- Exploratory separation results using known APT, MgPc, and Helicene labels
- Representative templates and manual interpretations for each cluster
- `outputs/tables/molecule_categories.csv`

### Stage 6: Quantitative Baseline-versus-Improved Evaluation

**Detection matching:** Use one-to-one matching within an allowed distance to connect automated centres with manually annotated centres.

- TP: an automated detection successfully matched to a manual object
- FP: an automated detection with no matching manual object
- FN: a manual object with no matching automated detection

**Metrics:**

```text
Precision = TP / (TP + FP)
Recall = TP / (TP + FN)
F1 = 2 * Precision * Recall / (Precision + Recall)
Absolute count error = |predicted count - manual count|
Relative count error = absolute count error / manual count
```

When reliable manual class labels are available, classification will be evaluated using a confusion matrix and macro-F1. Unsupervised clustering will be evaluated using silhouette score, adjusted Rand index, or normalised mutual information, together with manual interpretation of representative templates.

**Comparison experiments:**

| Experiment | Purpose |
| :--- | :--- |
| Paper baseline | Establish the performance of the original method |
| Improved preprocessing only | Isolate the contribution of background correction |
| Improved preprocessing plus watershed | Isolate the contribution of touching-object separation |
| Geometry only vs Zernike only vs combined | Isolate the contribution of fingerprint design |

**Outputs:**

- `outputs/tables/evaluation_metrics.csv`
- Baseline-versus-improved comparison tables and figures
- FP, FN, touching-object, and artifact failure-case figures
- A confusion matrix or clustering-agreement results

**Target reference:** Aim for a benchmark detection F1 of at least 0.80 and a relative count error no greater than 15%. These are project targets, not guaranteed outcomes. If they are not achieved, the causes of failure and the applicable operating range must be analysed.

### Stage 7: Full-Dataset Extension and Generalisation Assessment

After method selection is completed on the benchmarks, freeze the main pipeline and apply it to the remaining independent images. A small number of predefined configurations may be used for clearly defined image types, but results must not be described as automated if parameters are adjusted arbitrarily for every image.

**Outputs:**

- Final detection figures for all 13 independent images
- Count, valid area, density, dominant morphology, and quality flag for each image
- Lists of images that were processed automatically, require manual review, or are unsuitable for the method

### Stage 8: Descriptive Analysis of APT Conditions

Analyse image-level conditions only within the APT data: Baseline, CO-tip, UV, and annealing. Because scan areas differ across images, use:

```text
molecular density = molecule count / valid scan area (nm^2)
```

Also compare morphology proportions, clustered fraction, and uncertainty. The image, rather than each molecule, is the independent experimental unit for condition comparisons. Hundreds of molecules within one image must not be treated as hundreds of independent experimental replicates. If a condition is represented by only one image, report descriptive comparisons only and do not claim a causal effect.

**Outputs:**

- APT condition count-density comparisons
- Morphology-proportion plots
- A limitations statement covering sample size, imaging conditions, and potential confounding

### Stage 9: Client Recommendations

The final conclusions must answer:

1. Which pipeline is most suitable for routine molecule counting in STM images?
2. Under which image-quality conditions can processing be automated?
3. Which situations require manual inspection or parameter adjustment?
4. Which fingerprint set is most effective: Zernike, geometric, or combined?
5. Are the available data sufficient to support statistical conclusions about condition effects?

## 5. Notebook and Output Structure

Recommended notebook sequence:

| Notebook | Primary responsibility |
| :--- | :--- |
| `01_Data_Inspection.ipynb` | Data registration, quality assessment, and duplicate detection |
| `02_Paper_Baseline_Reproduction.ipynb` | Reproduce the three examples from the paper |
| `03_Manual_Annotation.ipynb` | Annotate manual centres and morphologies |
| `04_Improved_Preprocessing_Segmentation.ipynb` | Improve preprocessing, ROI handling, thresholding, and watershed segmentation |
| `05_Features_Classification_Evaluation.ipynb` | Evaluate fingerprints, categorisation, and model performance |
| `06_Full_Dataset_Condition_Analysis.ipynb` | Produce full-dataset outputs and analyse APT conditions |

Recommended directory structure:

```text
CountingMolecules/
├── ProjectData/                  # Raw data; read only
├── Materials/                    # Papers and background materials
├── annotations/                  # Manual annotations and annotation rules
├── notebooks/                    # Formal analysis notebooks
├── src/                          # Reusable reading, preprocessing, segmentation, and evaluation functions
├── outputs/
│   ├── figures/
│   ├── masks/
│   ├── tables/
│   └── models/
├── README.md
└── Project_Workflow_EN.md
```

## 6. GitHub and Reproducibility

- Whether raw large files are included in GitHub should be determined by the course repository limits; code and small result tables must be version controlled.
- Every notebook should use relative paths or a common configuration rather than absolute paths tied to one computer.
- Parameters should be stored in tables or configuration objects, not only in temporary notebook variables.
- Baseline and improved results must be saved separately to prevent accidental overwriting.
- Commit messages should identify the completed data or modelling stage.
- The README should record environment dependencies, execution order, data sources, and known limitations.

## 7. Final Report Structure and Alignment with the Marking Criteria

The submission should be written as a client report. The main text must not exceed 20 pages; code and detailed parameters should be placed in the appendix.

| Section | Suggested length | Project content |
| :--- | :---: | :--- |
| Executive Summary | 2 pages | Problem, key results, and client recommendations |
| Introduction | 2 pages | STM context, data sources, research questions, hypotheses, and literature |
| Data Quality | 2 pages | File structure, duplicate image, field of view, noise, scan lines, and artifacts |
| Model Development | 4 pages | Baseline and improved methods, fingerprints, assumptions, and limitations |
| Results | 6 pages | Manual validation, model comparison, metrics, failure cases, and full-dataset performance |
| Conclusions | 2 pages | Conclusions, applicable operating range, recommendations, and future work |
| Appendix | Excluded from main-text limit | Code, complete parameters, additional images, and detailed result tables |

Evidence supporting a High Distinction includes:

- Multiple credible and relevant sources on STM, image processing, and molecular recognition
- Analysis of the published method's assumptions, implementation, and limitations, rather than merely describing what it did
- Conclusions supported by manual ground truth and quantitative metrics
- Explanation of failure cases and the range over which conclusions apply
- Translation of analytical findings into actionable client recommendations
- Figures with clear titles whose significance is explained in the text
- A consistent citation style and complete reference list

The component percentages listed in the Final Report Guidelines sum to 110%, which may be a formatting error. The actual weighting should be confirmed using the Moodle rubric or the teaching team's latest advice.

## 8. Project Priorities

### Must Complete

1. Assess data quality and handle duplicate files.
2. Reproduce the three examples from the paper.
3. Manually annotate molecular centres in the three benchmark images.
4. Implement at least one improved preprocessing method and watershed segmentation.
5. Calculate baseline-versus-improved detection and counting metrics.
6. Compare at least two molecular-fingerprint sets.
7. Present clear failure modes, limitations, and client recommendations.

### Complete If Time Permits

1. Extend manual annotations to three hard cases.
2. Process all 13 independent images automatically.
3. Conduct a descriptive comparison of APT conditions.
4. Develop a supervised morphology classifier.
5. Develop an interactive manual-review tool.

Priority must be given to the research questions, baseline, ground truth, model comparison, and defensible conclusions. Adding many features must not substitute for analytical depth.

## 9. Definition of Done

The project is complete only when all of the following conditions are met:

- All primary-analysis files are readable and duplicate samples are not counted twice.
- The three examples from the paper have baseline reproduction results.
- The benchmark images have independent manual annotations.
- The baseline and improved methods are compared against the same ground truth.
- Every evaluated image has TP, FP, FN, and count-error results.
- Fingerprint selection is supported by experimental evidence.
- The results clearly distinguish species, condition, and morphology.
- All parameters, figures, and outputs in the pipeline are reproducible.
- Report conclusions do not extend beyond what the data support.
- The final report provides specific client recommendations rather than only presenting code and images.
