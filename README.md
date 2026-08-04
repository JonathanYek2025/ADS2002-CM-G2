# ADS2002-CM-G2-Ziyue
## Weekly Progress
### Week 2 Progress

**Objective:** Project initialization, data exploration, and team alignment.

**What I did this week:**
- **Data Exploration & Profiling:** Scanned all 14 STM raw images (`.p` and `.sxm` files) in `ProjectData`. Identified the 3 distinct molecular species (Hellerstedt APT, Castelli MgPc, Stetsovych Helicene) and handled a duplicated test file.
- **Difficulty Classification:** Categorized the dataset into 4 processing difficulty levels (Level 1 to Level 4) based on background tilt, scan line noise, and molecule aggregation, which will guide our future segmentation strategies.
- **Code Refactoring:** Cleaned up and restructured the initial exploratory code into a presentation-ready Jupyter Notebook (`Read_All_ProjectData_Images.ipynb`). It now automatically generates a global contact sheet grouped by species.
- **Literature Review:** Conducted a background study on the origin of the dataset, connecting our tasks to the core 2022 "Counting Molecules" paper and relevant deep learning/STM object detection literature.
- **Project Documentation:** Drafted a comprehensive Project Overview document (in both CN & EN) to clarify the Computer Vision (CV) pipeline methodology (Preprocessing -> Segmentation -> Feature Extraction -> Classification) for the entire team.
