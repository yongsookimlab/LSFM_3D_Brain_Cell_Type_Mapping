# 3D Mouse Brain Cell Type Mapping & Quantification Pipeline for Light Sheet Fluorescence Microscopy (LSFM) 
- Created by the [Yongsoo Kim Lab](https://kimlab.io/)
- README written and updated on 20250415 by J. Liwang

## Overview
This code package is designed for comprehensive 3D cell counting using whole mouse brain images. The pipeline includes:
1. Implementation of a trained cell segmentation model using ilastik (pixel and/or object classification via supervised machine learning) or Cellpose 2.0 (pretrained/human-in-the-loop deep neural network),
2. 3D image registration of light sheet fluorescence microscopy (LSFM)-acquired whole mouse brain data to an age-matched reference mouse brain template,
3. Registration of assigned and segmented voxels/"cells" to the reference space based on the common coordinate framework (CCF) system, and
4. Transformation of anatomical annotations (from the reference brain atlas, ie. Allen CCFv3) to the sample image registered to the reference space.


## System Requirements
> Please make sure the following requirements (ie. operating system, software, and tools) are downloaded and installed on your machine prior to code use.

### Hardware
Ideally, a high-performance computer with a 32- or 64-core processor to perform parallel computing in MATLAB.
- Microsoft Windows 10, 64-bit (operating system that the code was built and tested with) 

### Software
- MATLAB (MathWorks): [download](https://www.mathworks.com/products/matlab.html?s_tid=hp_products_matlab)
  - Tested using MATLAB versions R2020a and R2021b
- Fiji (ImageJ2): [download](https://imagej.net/software/fiji/)
  - Tested using ImageJ (1.53t) with Java 1.8.0_172 (64-bit)
- ilastik: [download](https://www.ilastik.org/documentation/basics/installation) or [ilastik github](https://github.com/ilastik/ilastik)
  - Tested using ilastik version 1.4.0
- Cellpose 2.0: [github] (https://github.com/MouseLand/cellpose)
- Advanced Normalization Tools (ANTs) in Python (ANTsPy): [github] (https://github.com/ANTsX/ANTsPy)
- Python: [download](https://www.python.org/downloads/)
  - Tested using Python versions 3.8.3 and 3.9.7
 
### Data and Tools
- Full-resolution, stitched, LSFM imaging data acquired with SmartSPIM (LifeCanvas Technologies)
  - A dataset consisting of one LSFM-imaged brain with expected data output has been made available for testing/demo purposes: [download here](https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/yuk17_psu_edu/EkTTKApE7aFLs7xzEMAnKloBq24jZ_rrKDmVWUt4mql93A?e=4DCPtz) 
  - Raw LSFM images acquired elsewhere can be fed through our custom stitching code (available [here](https://github.com/yongsookimlab/TracibleTissueCyteStitching)) for file structure and metadata compatibility.
  - Additionally, the Brain Image Library (BIL, RRID:SCR_017272) is a public [database](https://www.brainimagelibrary.org/index.html) of brain imaging data that has LSFM datasets for download which can be used as input for the counting code.
    
- Mouse brain reference atlas consisting of averaged templates and anatomical labels
  - Early Postnatal Developmental Mouse Brain Atlas (epDevAtlas, RRID:SCR_024725) can be viewed [here](https://kimlab.io/brain-map/epDevAtlas/) and downloaded [here](https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/yuk17_psu_edu/EkS4MIAfRgdKp93QHphJmfoBwOPt4fr2IFERVUMlcR3Rvg?e=tEUQVx).
  - Allen Mouse Brain Reference Atlas (Allen CCFv3, RRID:SCR_002978) can be viewed and downloaded [here](https://mouse.brain-map.org/static/atlas).


## How To Use

### Cell Segmentation with ilastik
> The Pixel Classification workflow categorizes pixels by utilizing both pixel features and user annotations. This workflow provides flexibility in choosing from a range of generic pixel features, including smoothed pixel intensity, edge filters, and texture descriptors. After selecting the desired features, a Random Forest classifier is trained interactively using user annotations.

- See ilastik's [tutorial](https://www.ilastik.org/documentation/pixelclassification/pixelclassification) on pixel classification for cell segmentation and all related documentation.

In brief:
1. Open the ilastik software for machine learning-based pixel classification.
2. Select new project. Project type: Pixel classification.
3. Input data for ML training.
    - Select and upload stitched STPT data pertaining to specific cell type (and/or age).
    - The number of single TIFs uploaded can vary, but it is good to have at least 5 images representing different brain regions in anterior-posterior axis.
4. Select features.
    - It is recommended to start off with a wider range (10 sigma) of features.
5. Train your classifier.
   - Under the "Group Visibility" section, right-click on **Input Data** to adjust the brightness threshold.
      - Threshold value should remain consistent during across all images during training.
    - Examples of labels (minimum of three for counting code, with Label 3 segmenting your cells of interest):
      - Label 1: empty background
      - Label 2: brain tissue background
      - Label 3: signal of interest for segmentation
      - Label 4: (optional) signal #2, extraneous fibers, etc
6. Click on the **Live Update** button to let the ML training of drawn labels update on the imaged TIFs.
     - Note: Unclick **Live Update** when navigating around image and drawing additional labels because the program can lag.
7. Toggle between **Prediction** and **Segmentation** buttons to view how the trained ML is performing based on user input thus far.
8. **SAVE PROJECT** continually during ML training.
   - Save this .ilp file on a local computer where the counting code will be executed or on a shared network drive.
   - Remember file pathname for input into counting code.
  
  
### Cell Segmentation with Cellpose 2.0



### Cell Counting and Atlas Registration
**Available in this Github repository are the necessary MATLAB scripts designed for 3D cell counting in LSFM-imaged whole mouse brains. However, to execute the main script ***RUN_THIS_002_batch_counting3d.m***, all downloaded scripts from this repository, installed software, and reference atlas files must be gathered in one parent directory. The code is written with a specific folder structure, which can be found by viewing/downloading the entire code package including test sample data [here](https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/yuk17_psu_edu/ElSwPmP7iJ5MgHRibS-t2UoBStedo5zEuEMjOwElt5RBxA?e=OiGXtY).**

> Note: This README provides an overview for executing the following MATLAB scripts: 1) Editing parameter settings with **EDIT_THIS_001_param_setting.m**, and 2) Running the main script **RUN_THIS_002_batch_counting3d.m** that calls on a collective of scripts in the **private** folder. It is crucial to refer to the comments (preceded by % and %%%) in the main script for detailed information on each section and parameter.

I. Edit ‘sess2process.csv’
This CSV file contains the location/path of the sample images and the functional switches. By adding rows, you can sequentially process multiple datasets. In each column, 1 = switch on, and 0 = switch off, to individually run different parts of the counting code. 
  - Column A – path_sess: path to your data (session) to process. This path must be directed to the data folder.
  - Column B – switch_registration: use ANTs for image registration.
  - Column C – switch_preprocess: performs image preprocessing (remove background, make intensity even, etc). 
  - Column D – use_preprocessed_img_for_ML: if you want to use preprocessed images for the ilastik model and cell counting, you should set use_preprocessed_img_for_ML = 1. Otherwise, set columns C and D to 0 (zero).
  -	Column E – switch_machine_learning: applies trained ilastik classification model to entire stitched dataset of indicated signal channel.
  -	Column F – switch_counting3d: takes ML classification results, applies size and gaussian filters to find the local maxima and centroid (of cells), then, using xyz coordinates of counted cells, performs 3D correction to remove multiple counted cells in a specific location, thus preventing overcounting.
  -	Column G – switch_postprocess: applies ANTs transformation of counted cell coordinates to the sample data space and the reference brain-registered sample image. Additionally, it generates a quantitative CSV output of counted cells, brain region volumes, and cell densities based on your brain region ontology of choice (ie. CCFv3). 
  -	Column H – switch_qc3d: provides visualization to enable quality check of 3D counted cells .
  -	Column I & J – size_filter_pxl_thr1 & size_filter_pxl_thr2: after cell identification by ML (ilastik), you can further filter out cells that are either too small or too large by setting size thresholds. Thr1 is the lower bound and thr2 is the upper bound. 

II.	Edit ‘EDIT_THIS_001_param_setting.m’
Here, you can specify parameters for your data. For instance, you can specify image resolution for downscaling, iDISCO vs. LifeCanvas sample (different brain orientation), reference brain, annotation file, etc.
Within the script, only edit what is written in blue, if necessary.

1)	Basic
  •	params.signal_ch = 1 – identify signal channel from LSFM output; 0 = stitched_00, 1 = stitched_01, 2 = stitched_02
  •	params.is_LifeCanvas = 1 – this setting has to do with the preferred brain orientation for LSFM imaging in the Kim Lab; 1 = LifeCanvas; 0 = iDISCO; check Advanced Settings at the bottom of the script for editing orientation
  •	params.xyz_resolution = [1.8 1.8 5.0] – for 4x objective LSFM imaging with 5um z-step intervals, the xyz resolution should be set to [1.80, 1.80, 5]; change if otherwise.
  •	params.target_resolution = 20 – indicate the target resolution for registration. The downsampled image resolution is usually 20um isotropic, so set this equal to 20.

2)	ML – location of your ilastik model
  •	params.path_ml_project - set path location to ilastik trained ML trained model, including file name

3)	ANTs registration
  •	params.path_ref = [pwd filesep 'ref_brains']; (Do not change)
  •	params.path_ANTs_tmp = 'D:\ANTs_tmp'; if ~exist(params.path_ANTs_tmp, 'dir'), mkdir(params.path_ANTs_tmp); end (Do not change)
  •	params.fixed = [params.path_ANTs_tmp filesep 'rotated_chx.nii.gz']; (Do not change)
  •	params.moving = [params.path_ref filesep 'T_P04_LSFM_Symmetric20um_template0_u16_n10_clean_PA.nii'];
    o	Reference brain template for ANTs registration must be saved as a nifti (.nii) file in the ref_brains folder within the parent working directory (pwd). 
    o	Check the Properties of the file in Fiji/ImageJ and make sure the pixel width, height, and depth are 1 per pixel.
  •	params.anno = [params.path_ref filesep 'P04_CCFv3_annotations_16b_v3_iso20um_u16.nii'];
    o	Reference brain annotations for ANTs registration must be saved as a nifti (.nii) file in the ref_brains folder within the parent working directory (pwd). 
    o	Check the Properties of the file in Fiji/ImageJ and make sure the pixel width, height, and depth are 1 per pixel.
  •	params.path2downsample = [path_sess filesep 'stitched_00'];
    o	Indicate which imaged data folder will be used for down sampling. This is typically a background, autofluorescence channel. 
  •	params.thr_blurs = 17000 – Set the threshold for background blurs in LSFM data; working range: 12000 ~ 20000

4)	Cell counting 3d
  •	params.size_filter_pxl_thr1 = size_filter_pxl_thr1; (Do not change) – if need to change, edit sess2process.csv
  •	params.size_filter_pxl_thr2 = size_filter_pxl_thr2; (Do not change) – if need to change, edit sess2process.csv

5)	QC 3D
  •	params.z_step = 200 – indicate the Z-step size interval for QC; e.g. skip every 200 z-steps
  •	params.z_block_depth = 9 – validate 10 z-sections (1+9); e.g.) 1-10, 200-210, ... etc
  •	params.z_padding = 2 – set the z-step padding outside of ROI block depth
  •	params.increase_contrast = 0 – change only when image contrast is not good
  •	params.img_ceiling = 7000 – set the max intensity value of imgs; to increase contrast

III.	Run ‘RUN_THIS_002_batch_counting3d.m’
Open this file on MATLAB and click ‘Run’ button.


     - Expected code runtime for a single early postnatal brain can range from approximately 3 to 6 hours using a 64-core computer (if no other tasks are running in the background).
     - If running on a normal home desktop computer (average 8 cores), the runtime may last or exceed 24 to 48 hours.
10. Output in sample directory:
    - Spreadsheet (counted_3d_cells.csv) with columns for brain regions (listed in hierarchical order based on CCFv3 ontology), cell counts, cell densities, and volumes per region for an individual brain sample.
      -  This output file is ready for analysis. By implementing this code for multiple,  imaged brains, you can calculate and generated averaged datasets with appropriate statistical measures.
      -  All quantitative results in the related [manuscript](https://www.biorxiv.org/content/10.1101/2023.11.24.568585v1.full) were analyzed using Prism (GraphPad) and Excel (Microsoft).
    - Registration output (elastix folder)
       - All processes and errors during registration are logged (elastix.log) and these files are generated automatically. 
    - Reverse registration output with mapped cells in 3D reference space (cell_counted_refspace.tif)
      - Open this file in Fiji, change image type to 32-bit (Image > Type > 32-bit), and apply a Gaussian filter with 2.0 sigma (Process > Filters > Gaussian blur 3D) for better visualization of the 3D counted cells in the entire TIF stack.
      - Save this edited TIF stack with a new file name. It can now be utilized as input for our isocortical flatmap visualization code, found [here](https://github.com/yongsookimlab/CorticalFlatMap). 
    - **Examples of test data output can be found in the shared folder [here](https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/yuk17_psu_edu/EkTTKApE7aFLs7xzEMAnKloBq24jZ_rrKDmVWUt4mql93A?e=4DCPtz)**


### Quality Check for ML Cell Counting and Image Registration
> The purpose of this section is to utilize the scripts within the **quality_control** folder for quickly checking the segmentation accuracy of the trained ilastik ML for signal-containing voxels/cells and the image registration accuracy. It is helpful to use this tool during ML training and optimization. 

1. Download **quality_control** folder and ensure all components are inside:
     - **Run_QC1_QC2.m**
     - **quality_check_counting_setting_pack.m**
     - **quality_check_background.m**
     - **quality_check_counting.m**
2. Open the script **quality_check_counting_setting_pack.m** in MATLAB and copy settings from **RUN_THIS_FILE.m**. Additionally:
     - Set the stack_thickness = 0 to avoid z-overlapping.
     - Set "radii_draw" to a value (in micrometers) for the program to draw a circle around for the counted cells.
     - The "drawing_range" set to 3000 for the maximum contrast has previously worked for this purpose, but this number can be changed if desired.
3. Open **quality_check_counting.m** in MATLAB for counting quality check (QC1).
     - See commented code for details on input settings.
     - When run, this code will check the full resolution stitched images at the specified z-intervals and draw red circles around each counted cell based on the ilastik ML results. With this QC, it is easy to check whether the trained ML needs improvement. You can also use this to calculate an F-scpre.
4. Open **quality_check_background.m** in MATLAB for registration quality check (QC2).
     - See commented code for details on input settings.
     - When run, this code will use a rotated, downsized image TIF stack of the brain and elastix registration results to align the counted 3D voxels to the chosen 3D reference space. The result is a RGB image (NII and/or PNG file) with the aligned 3D counts in reference space. The warmer colors denote higher counts/density. With this QC, you can quickly check registration quality for specified z-intervals without performing elastix on the full dataset.
5. Save when finished with the settings for all three scripts.
6. Open **Run_QC1_QC2.m** and execute this MATLAB script to perform quality checks for both counting and registration.

   
> Contact: For questions or assistance, please contact the lab's principal investigator, Yongsoo Kim (yuk17@psu.edu).


## Limitations
This code was developed for 3D cell quantification utilizing whole brain imaging with the TissueCyte (TissueVision) STPT system in mind, which has specific parameters that may not apply to other imaging modalities. It is possible to use this code with 3D whole brain images acquired via light sheet fluorescence microscopy, but this is currently under optimization by the Yongsoo Kim Lab.


## License
- The epDevAtlas and associated code is licensed under a Creative Commons Attribution 4.0 International License as of December 1, 2023, but it is free and openly accessible for academic use.
- If you use this code anywhere, we would appreciate if you cite the following [preprint](https://www.biorxiv.org/content/10.1101/2023.11.24.568585v1):
  - **epDevAtlas: Mapping GABAergic cells and microglia in postnatal mouse brains**
    - Josephine K. Liwang, Fae A. Kronman, Jennifer A. Minteer, Yuan-Ting Wu, Daniel J. Vanselow, Yoav Ben-Simon, Michael Taormina, Steffy B. Manjila, Deniz Parmaksiz, Sharon W. Way, Hongkui Zeng, Bosiljka Tasic, Lydia Ng, Yongsoo Kim. bioRxiv 2023.11.24.568585; doi: https://doi.org/10.1101/2023.11.24.568585
