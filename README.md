======================================================================
README — Pre-Processing Workflow for Large-Scale PTV
======================================================================
Author:     Dr.-Ing. José Aliaga-Villagrán
Affiliation: Leichtweiß-Institute for Hydraulic Engineering and Water Resources
Contact:     j.aliaga-villagran@tu-braunschweig.de / jose.aliaga.v@protonmail.com
Version:     1.0
Date:        2025-11-04

---------------------------------------------------------------------
OVERVIEW
---------------------------------------------------------------------
This documentation describes the complete workflow for pre-processing
videos in Large-Scale Particle Tracking Velocimetry (PTV) experiments.

The process is divided into two main blocks:
  1) Get All Calibration Parameters (camera and geometry)
  2) Batch Pre-Processing of Experiment Videos

======================================================================
BLOCK 1 — GET ALL CALIBRATION PARAMETERS
======================================================================
This block generates all geometric corrections and scaling information
needed for quantitative PTV analysis.

---------------------------------------------------------------------
STEP 1: LENS DISTORTION CALIBRATION
---------------------------------------------------------------------
Goal: Correct optical distortion (e.g., fisheye) introduced by the lens.

Input:
- A video of a checkerboard calibration target from various angles.

Workflow:
1. Extract calibration frames:
     -> Run: PreProc_Video2Frames.mlx
     -> Select the checkerboard calibration video.
     -> Output: Folder 'Frames_raw' with still images.

2. Estimate lens distortion parameters:
     -> Run: PreProc_Lensdist_Params.mlx (or use Camera Calibrator App).
     -> Set square size and units.
     -> Choose 'Fisheye' model for GoPro-type cameras.
     -> Calibrate and export parameters (cameraParams.mat).

3. Optionally validate batch correction:
     -> Run: PreProc_Lensdist_Batch.mlx
     -> Input: cameraParams.mat + calibration images.
     -> Output: Folder 'Frames_corrected' with distortion-free images.

---------------------------------------------------------------------
STEP 2: ORTHORECTIFICATION PARAMETER ESTIMATION
---------------------------------------------------------------------
Goal: Define planar projective transform to map image to world plane.

Input:
- Video showing at least 4 Ground Control Points (GCPs) on the water surface.
- Known pairwise distances between the 4 GCPs.

Workflow:
1. Extract frames showing GCP markers:
     -> Run: PreProc_Video2Frames.mlx
     -> Select the calibration video with markers.

2. Correct lens distortion on one useful frame:
     -> Run: PreProc_Lensdist_Batch.mlx
     -> Input: one selected frame + cameraParams.mat.
     -> Output: Distortion-corrected image of markers.

3. Estimate orthorectification parameters:
     -> Run: PreProc_Orthorec_Params.mlx
     -> Double-click 4 GCP centers (any order).
     -> Enter the 6 pairwise distances (same order).
     -> Script reconstructs geometry and saves *ortho_params.mat*.

Outputs of Block 1:
  - cameraParams.mat         (lens calibration)
  - GCP_ortho_params.mat     (orthorectification parameters)

======================================================================
BLOCK 2 — BATCH PRE-PROCESSING OF EXPERIMENT VIDEOS
======================================================================
This block processes all experiment videos using the calibration parameters.

---------------------------------------------------------------------
STEP 1: FRAME EXTRACTION
---------------------------------------------------------------------
Run: PreProc_Video2Frames.mlx
Input: Experiment video file.
Output: Folder 'Frames_raw' with extracted frames.

---------------------------------------------------------------------
STEP 2: LENS DISTORTION CORRECTION
---------------------------------------------------------------------
Run: PreProc_Lensdist_Batch.mlx
Input: cameraParams.mat + 'Frames_raw' folder.
Output: 'Frames_corrected' folder with distortion-free frames.

---------------------------------------------------------------------
STEP 3: 2D ORTHORECTIFICATION
---------------------------------------------------------------------
Run: PreProc_Orthorec_Batch.mlx
Input: GCP_ortho_params.mat + 'Frames_corrected' folder.
Select: Full-extent isotropic option (recommended).
Output: 'Frames_ortho_full' folder with rectified frames ready for PTV.

======================================================================
SUMMARY OF KEY SCRIPTS
======================================================================
PreProc_Video2Frames.mlx      -> Extract frames from video files.
PreProc_Lensdist_Params.mlx    -> Launch Camera Calibrator app.
PreProc_Lensdist_Batch.mlx    -> Batch lens distortion correction.
PreProc_Orthorec_Params.mlx   -> Estimate orthorectification parameters.
PreProc_Orthorec_Batch.mlx    -> Batch orthorectify full image set.

======================================================================
BEST PRACTICES
======================================================================
- Ensure GCPs lie on the same plane (water surface).
- Keep the same camera configuration between calibration and experiment.
- Save all *.mat parameter files with the corresponding experiments.
- For GoPro cameras, use 'Fisheye' mode in MATLAB calibration.
- Avoid mixing frames from different cameras or sessions.

======================================================================
OUTPUT FOLDER STRUCTURE EXAMPLE
======================================================================
PTV_PreProcessing/
 ├── Calibration_Checkboard/
 │     ├── Calibration.mp4
 │     ├── Frames_raw/
 │     ├── cameraParams.mat
 │     └── Frames_corrected/
 │
 ├── Calibration_GCPs/
 │     ├── Markers.mp4
 │     ├── Frames_raw/
 │     ├── frame_0010_corrected.png
 │     └── GCP_ortho_params.mat
 │
 ├── Experiment_Run_01/
 │     ├── Raw_Video.mp4
 │     ├── Frames_raw/
 │     ├── Frames_corrected/
 │     └── Frames_ortho_full/
 │
 └── README_PreProcessing.txt

======================================================================
END OF DOCUMENT
======================================================================
