FIGURE PLACEHOLDER
==================

File needed: figure1.png
Used in: paper.md (Figure 1)

What this figure should show:
  Panel A — Screenshot of the scPassport() Shiny popup with fields filled in
  Panel B — Screenshot of read_passport() console output

HOW TO MAKE THE FIGURE — Step by step
======================================

STEP 1 — Take the Shiny popup screenshot (Panel A)
---------------------------------------------------
1. Open RStudio
2. Run this in the console:

   library(scPassport)
   library(SummarizedExperiment)
   se <- SummarizedExperiment()
   se <- scPassport(se)

3. The popup will open. Fill in some example fields:
     Object ID:   WTHeme
     Animal ID:   M01
     Species:     Mus musculus
     Sex:         male
     Age:         P60
     Condition:   control
     Tissue:      lung
     Project:     PH_study
     Researcher:  Sedat Kacar

4. DO NOT click Done yet — take a screenshot of the popup window
   - Windows: Press Win + Shift + S, drag to select just the popup window
   - Save as: panel_A_popup.png

STEP 2 — Take the read_passport() screenshot (Panel B)
------------------------------------------------------
1. Click Done in the popup (saves the passport)
2. Also add some log steps:
   se <- log_step(se, "QC filter", params = list(min_cells = 3))
   se <- log_step(se, "NormalizeData")
   se <- log_step(se, "RunPCA", params = list(npcs = 30))
3. Run:
   read_passport(se)
4. The console output will show the full passport + processing log
5. Screenshot just the console output panel in RStudio
   - Save as: panel_B_console.png

STEP 3 — Combine into one figure
----------------------------------
Easiest option — PowerPoint:
1. Open PowerPoint, set slide size to 16x9 inches
2. Insert panel_A_popup.png on the left half
3. Insert panel_B_console.png on the right half
4. Add labels: bold "A" top-left of panel A, bold "B" top-left of panel B
5. File > Export > Save as PNG > set width to 2400px
6. Save as: figure1.png in this folder

Alternative — free online tool:
- Go to https://www.canva.com (free account)
- Create a blank document, drag both screenshots in side by side
- Download as PNG

STEP 4 — Check the figure
--------------------------
- File must be named: figure1.png
- Minimum width: 1200px (2400px recommended for print quality)
- Must be in this folder: scPassport/paper_figures/figure1.png

Once figure1.png is in this folder, the paper is ready to submit to JOSS.
