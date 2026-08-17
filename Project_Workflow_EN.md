---
title: Counting Molecules Four-Part Project Workflow
aliases:
  - Counting Molecules Workflow English
tags:
  - ADS2002
  - CountingMolecules
  - STM
  - Workflow
language: en
---

# Counting Molecules Four-Part Project Workflow

## 1. Project Topic

**Automated molecule detection, counting, and morphology classification in STM images**

This project uses one manually annotated molecular image to establish the **ground truth**. It first compares molecule extraction methods and classification methods separately, then selects the best complete pipeline and applies it to other unlabelled STM images.

> [!important]
> The annotated image is an evaluation reference, not an algorithm input. The algorithm must operate on the raw STM data, height matrix, or uncoloured greyscale image.

## 2. Target Image and Research Scope

This project selects **`Helicene_Ag(111)008.sxm`** as the common target image and evaluation benchmark for Parts A, B, and C. The main reason for selecting this image is that the associated research paper provides a **manual count or manually annotated result** for the corresponding STM image. This enables the paper result to be used as ground truth for objective algorithm evaluation.

The image scope of each part is defined as follows:

- **Part A:** compare preprocessing, segmentation, and molecule extraction methods on `Helicene_Ag(111)008.sxm`;
- **Part B:** compare feature representations and classification algorithms using molecular objects from the same image;
- **Part C:** continue using the same image to compare the end-to-end performance of complete extraction and classification combinations;
- **Part D:** fix the best pipeline selected in Part C, apply it to other unlabelled `.p` and `.sxm` images, and analyse its generalisability.

The benchmark must separate two different sources of information:

- **Algorithm input:** the raw STM height matrix or uncoloured greyscale image read from `Helicene_Ag(111)008.sxm`;
	- ![[Pasted image 20260811101827.png]]
- **Evaluation reference:** the manual count, molecular centres, or category annotations reported for the image in the paper.
	- ![[Pasted image 20260811101850.png]]

Before algorithm comparison, the correspondence between the paper figure and the `.sxm` data must be verified by checking the molecular arrangement, image orientation, cropped region, scale, and spatial registration. If the paper provides only a manual total, only **Count Error** can be evaluated directly. TP, FP, FN, Precision, Recall, F1-score, and complete object-level evaluation require molecular centre locations or recoverable annotation positions.

> [!important]
> Parts A, B, and C use the same benchmark so that all methods are compared fairly using the same data and ground truth. Part D does not repeat algorithm selection; it tests whether the selected pipeline generalises to other STM images.

## 3. Part A: Preprocessing and Molecule Extraction

### 3.1 Objective

Compare image preprocessing, segmentation, and object extraction methods to determine which method detects individual molecules most accurately in an STM image.

Preprocessing is not simply intended to make the image look cleaner. It must remove background variation and noise while:

- preserving real molecules;
- preventing noise from being detected as molecules;
- separating touching or overlapping molecules;
- producing individual molecular objects for counting and classification.

### 3.2 Main Process

```text
Raw STM image or height matrix
→ Standardise image orientation, dimensions, and pixel scale
→ Crop the valid region and handle image boundaries
→ Apply background or plane correction
→ Apply denoising and intensity normalisation
→ Use thresholding to generate a binary mask
→ Clean the mask with morphological operations
→ Use a distance transform to identify centre candidates
→ Use watershed segmentation to separate touching molecules
→ Extract each molecular contour and centre point
→ Compare the result with the ground truth
```

### 3.3 Candidate Methods

- The preprocessing and extraction method reproduced from the paper;
- Gaussian filtering with Otsu thresholding;
- Background correction with adaptive thresholding;
- Morphological operations with connected-component analysis;
- Distance transform with watershed segmentation;
- An improved method constructed from the experimental findings.

### 3.4 Evaluation

Detected molecular centres or segmented regions are matched to the ground truth:

- **TP (True Positive):** a real molecule is correctly detected;
- **FP (False Positive):** background or noise is incorrectly detected as a molecule;
- **FN (False Negative):** a real molecule is missed;
- **Precision:** the proportion of detected objects that are real molecules;
- **Recall:** the proportion of ground-truth molecules that are detected;
- **F1-score:** the harmonic mean of precision and recall;
- **Count Error:** the difference between the automated and manual counts.

Overlay visualisations should also be inspected for:

- missed small molecules;
- noise-related false detections;
- touching molecules merged into one object;
- one molecule incorrectly split into several objects;
- false detections near image boundaries.

### 3.5 Outputs

- A binary mask from each method;
- Molecular contours and centre coordinates;
- Ground-truth and prediction overlay figures;
- A detection-metric comparison table;
- A shortlist of the best molecule extraction methods.

## 4. Part B: Molecule Classification

### 4.1 Objective

Compare feature representations and classification algorithms on a consistent set of individual molecular objects to determine which method best distinguishes molecular morphologies or categories.

For a fair comparison, the classification methods should use the same molecular objects, such as ground-truth molecular regions or objects generated by one fixed extraction method. This prevents detection errors from Part A from confounding the classification comparison.

### 4.2 Main Process

```text
Extracted individual molecules
→ Crop each local molecular image
→ Centre and resize each object, with rotation normalisation if required
→ Extract shape, intensity, height, contour, and texture features
→ Combine the features into a molecular fingerprint
→ Standardise the feature matrix
→ Apply different classification or clustering algorithms
→ Assign a category label to each molecule
→ Compare the result with the manual category labels
→ Calculate the total count and the count and proportion of each category
```

### 4.3 Candidate Features

- Area, perimeter, circularity, and aspect ratio;
- Mean intensity, maximum height, and height distribution;
- Contour and shape descriptors;
- Hu moments;
- Zernike moments;
- Texture features;
- A combined molecular fingerprint.

### 4.4 Candidate Algorithms

- Affinity Propagation;
- K-Means;
- Agglomerative Clustering;
- BIRCH;
- DBSCAN or HDBSCAN;
- Random Forest, SVM, or KNN if sufficient manual labels are available.

### 4.5 Evaluation

When manual category labels are available for individual molecules, the following metrics can be used:

- Classification Accuracy;
- Precision, Recall, and F1-score;
- Macro F1-score;
- Confusion Matrix;
- Count Error for each category.

For unsupervised clustering, the analysis can additionally use:

- Adjusted Rand Index (ARI);
- Normalised Mutual Information (NMI);
- The best mapping between clusters and manual categories;
- The interpretability and consistency of representative molecules in each cluster.

### 4.6 Outputs

- A feature table containing one row per molecule;
- A predicted category for each molecule;
- A classification-algorithm comparison table;
- A confusion matrix or clustering evaluation results;
- Representative molecular images for each category;
- A shortlist of the best classification methods.

## 5. Part C: Selecting the Best Complete Pipeline

### 5.1 Objective

Combine the strongest extraction methods from Part A with the strongest feature and classification methods from Part B, and compare the end-to-end performance of the resulting pipelines.

It is not necessary to test every possible combination. The methods can first be shortlisted separately, after which a manageable number of complete candidate pipelines can be constructed.

### 5.2 Classification Scope and Highest-Level Objective

The classification evaluation in this project **ignores handedness differences**. Handedness 1 and Handedness 2 examples with the same molecular structure or aggregation form are merged into one category. The model is therefore not required to determine left- and right-handed adsorption configurations.

The target categories in Part C are:

- **Tetramer of P1:** an assembly containing four P1 units;
	- ![[Pasted image 20260811101517.png]]
- **Trimer of P1:** an assembly containing three P1 units;
	- ![[Pasted image 20260811101528.png]]
- **Dimer of P1:** an assembly containing two P1 units;
	- ![[Pasted image 20260811101538.png]]
- **P2:** an individual P2 molecule;
	- ![[Pasted image 20260811101552.png]]
- **P3:** an individual P3 molecule.
	- ![[Pasted image 20260811101624.png]]

The classification objective has two levels of difficulty:

1. **Core objective:** distinguish the tetramer, trimer, and dimer of P1 using object area, contour, number of bright lobes, and spatial arrangement. These classes contain different numbers of P1 molecular units and therefore have relatively clear differences in overall STM size and morphology.
2. **Highest-level objective:** further distinguish P2 from P3. This is the most difficult classification task because both are planar individual molecules with similar apparent sizes and STM appearances.

According to the chemical structures reported in the paper, P2 is **dibenzo[a,m]indeno[1,2,3-e,f]coronene**, whereas P3 is **dibenzo[a,g]coronene**. In non-specialist terms, both are planar polycyclic molecules formed from fused carbon rings, but their molecular frameworks are not identical. Counting the fused rings shown in the structural formula, P2 contains an 11-ring fused carbon framework, whereas P3 contains 9 rings. P2 includes an additional **indeno-fused structural unit**, giving it a theoretically larger molecular framework and projected footprint, as well as a different structural symmetry from P3.
![[Pasted image 20260811101640.png]]![[Pasted image 20260811101733.png|233]]

However, an STM image represents apparent height and intensity arising from local electronic states rather than a direct photograph of molecular geometry. The apparent areas, intensities, and contours of P2 and P3 may therefore overlap considerably, and the assumption that "P2 is slightly larger" may not be sufficient for reliable classification. The model should combine area, contour, symmetry, intensity distribution, Zernike moments, and other relevant features.

The **ideal result** is reliable separation of P2 and P3. If the available image resolution or feature representation is insufficient, the two classes may instead be reported as **`P2/P3 unresolved monomer`**. This should not be presented as successful fine-grained classification. The limitation should be demonstrated using the confusion matrix, class-level F1-scores for P2 and P3, overlapping feature distributions, and the physical limitations of STM imaging. With transparent evaluation, an inability to separate these two classes remains a scientifically meaningful result.

### 5.3 Main Process

```text
Candidate molecule extraction method
+ Candidate feature set
+ Candidate classification algorithm
→ Construct several complete pipelines
→ Run every pipeline on the same annotated benchmark image
→ Match detected objects to the ground truth
→ Calculate detection, counting, and classification results
→ Calculate end-to-end performance
→ Rank the candidates and select the best pipeline
```

### 5.4 Core Metrics

The complete pipeline should be evaluated using:

- Detection Precision, Recall, and F1-score;
- Total Molecule Count Error;
- Classification Accuracy or Macro F1-score;
- Count Error for each molecular category;
- End-to-end Accuracy;
- Stability, interpretability, computational complexity, and runtime.

Before calculating classification metrics, Handedness 1 and Handedness 2 examples of the same structure should be merged into one label so that the evaluation matches the defined project objective. In addition to the overall Macro F1-score, Precision, Recall, F1-score, and mutual confusion counts should be reported separately for P2 and P3.

End-to-end accuracy can be defined as:

$$
\text{End-to-end Accuracy}
=
\frac{\text{Number of molecules correctly detected and classified}}
{\text{Total number of ground-truth molecules}}
$$

### 5.5 Outputs

- A complete pipeline ranking table;
- The selected extraction, feature, and classification combination;
- The final parameter settings;
- A justified explanation of the selection;
- Classification results for the dimer, trimer, and tetramer of P1;
- A dedicated P2-versus-P3 result or evidence that they cannot be separated reliably;
- Representative successful and failed cases.

## 6. Part D: Application to Other STM Images

### 6.1 Objective

Fix the best pipeline selected in Part C and apply it to the remaining unlabelled `.p` and `.sxm` images. The pipeline should produce molecular counts and classifications and allow its generalisability to be assessed.

### 6.2 Main Process

```text
Load another STM image
→ Apply the same preprocessing and segmentation procedure
→ Extract molecular contours and centre points
→ Calculate the total molecule count
→ Extract the same features
→ Apply the selected classifier or clustering rule
→ Assign a category to each molecule
→ Calculate the count and proportion of each category
→ Overlay the detection and classification results on the original image
→ Perform manual spot checks and analyse generalisability
```

### 6.3 Evaluation Without Complete Ground Truth

Because the remaining images do not have complete manual labels, their true accuracy should not be claimed directly. Instead, the project can:

- manually count randomly selected image regions;
- inspect samples for false positives, false negatives, and classification errors;
- compare molecular counts and category distributions across images;
- record the effects of background variation, noise, scan direction, and image quality;
- identify the conditions under which the pipeline is stable and the conditions under which it fails.

### 6.4 Outputs

For each image, the project should produce:

- The total molecule count;
- Molecular centre coordinates and contours;
- The predicted category of each molecule;
- The count and proportion of each category;
- A detection and classification overlay figure;
- Manual spot-check records;
- An analysis of generalisation and failure causes.

## 7. Overall Project Logic

```text
Part A: Use Helicene_Ag(111)008.sxm to identify the best method for finding molecules
→ Part B: Use the same image to identify the best method for classifying molecules
→ Part C: Use the same image to select the best complete algorithm combination
→ Part D: Test whether the selected pipeline generalises to other STM images
```

This structure evaluates detection and classification separately, making it possible to identify which stage causes a performance difference. The final end-to-end evaluation and application to additional images then provide a complete and interpretable project conclusion.
