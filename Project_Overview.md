# ADS2002 Project Core Overview: Counting Molecules

### Disclaimer
This document is intended to clarify the overall framework and background of the project, ensuring a unified understanding across our team. Please note that this is not a specific task delegation list.

---

### 1. Main Topic
The theme of this project is **"Counting Molecules."** Our ultimate goal is to utilize Python scientific computing and image processing libraries (such as `scikit-image`, `scikit-learn`) to build an automated image processing pipeline. This system must be capable of processing raw microscopy images, automatically detecting and accurately counting molecules, and classifying molecule species based on morphological features.

### 2. Background
The data used in this project originates from the forefront of physics research. All images were captured by researchers using Scanning Tunneling Microscopy (STM) at the nanoscale. In traditional research, molecule counting and classification usually rely on manual visual identification, a process that is time-consuming and prone to subjective errors. The core significance of this project is to introduce computer vision algorithms to automate the analysis workflow of nanoscale image data.

### 3. Data Characteristics and Methodology Shift
This project differs significantly from conventional data analysis tasks. In past practices, the subjects of analysis were mostly structured tabular data (e.g., CSV format), heavily relying on DataFrames for numerical cleaning and statistical analysis.

However, the dataset provided for this project consists of **14 raw microscopy images**. This implies a mandatory shift in research methodology: the essence of an image is an unstructured pixel matrix. We will not be able to use traditional numerical processing methods; instead, we must introduce "Digital Image Processing" and "Computer Vision" techniques. Examples include: background leveling of the image matrix, spatial domain noise filtering, and morphological segmentation based on pixel connectivity.

### 4. Existing Notebooks
To intuitively demonstrate the data structure and provide preliminary technical support, two Jupyter Notebook files have been configured in the project repository:
*   **Basic Data Reader Notebook:** Contains underlying code for parsing `.sxm` and `.p` raw microscopy data formats.
*   **Global Data Overview Notebook (`Read_All_ProjectData_Images.ipynb`):** This file successfully loads all 14 images and generates a panoramic visualization (Contact Sheet). It also includes a detailed English project summary at the end. It is highly recommended that all members run and read this file first to establish an intuitive understanding of the dataset.

### 5. Dataset Structure
Under the `ProjectData` directory, the dataset is strictly divided into three subfolders, corresponding to three target molecules with significant geometric differences:
*   **Hellerstedt APT (10 images):** Morphologically presents as small bright spots or three-lobe structures. Some samples exhibit highly dense, aggregated molecular clusters due to surface treatments (like UV irradiation or annealing).
*   **Castelli MgPc (1 image):** Magnesium phthalocyanine molecule, presenting a regular, large cross or four-lobe star shape.
*   **Stetsovych Helicene (3 images):** Helicene molecule, characterized by square or rhombic ring-like structures.

### 6. Progressive Analytical Steps
To achieve the final goal, we need to link various tasks to build a fully automated code system. The development of this system will strictly follow these four progressive steps:

*   **Step 1: Preprocessing and Standardization**
    STM microscopy images typically contain unavoidable flaws, such as slanted backgrounds (one side bright, the other dark) and horizontal scan line noise.
    **Specific Task:** Utilize Python code to perform 2D plane fitting and local filtering. This step aims to achieve "data standardization," converting all 14 images into uniform standard base images with flat backgrounds and minimal interference, laying a solid foundation for subsequent analysis.

*   **Step 2: Object Detection and Segmentation**
    After cleaning the background, the bright molecules need to be extracted from the dark background. In image processing, this is called "Thresholding."
    **Core Challenge:** Many molecules in the images (especially APT) tend to densely aggregate and stick together. Without proper handling, the system will miscount a clump of molecules as a single one. Therefore, we must introduce advanced segmentation techniques like the "Watershed algorithm" to enable the code to automatically locate overlapping areas and accurately split them apart.

*   **Step 3: Feature Extraction and Classification**
    After successfully isolating each independent molecule, the code still needs to determine which species they belong to.
    **Specific Task:** Have the system automatically measure the "geometric fingerprint" of each molecule, such as Area, Eccentricity (how flat it is), and Aspect Ratio. Based on these shape parameters, the model can accurately distinguish between the cross-shaped MgPc, the ring-shaped Helicene, and the smaller APT. Meanwhile, if the system detects an object with an abnormally huge area (like surface dust or step defects), it will identify it as an image artifact and automatically remove it, thereby filtering out useless data.

*   **Step 4: Counting and Validation**
    This is the final evaluation phase of the project. At this point, our system should be able to automatically output the total number of target molecules in each image.
    **Specific Task:** Select representative test images and conduct accurate manual visual counting (the Ground Truth). Then, compare the manual counting results with the system's automatically calculated results. If the two values are highly similar with minimal error, it proves that the automated counting and classification system we built is accurate and practically usable.
