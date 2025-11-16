# Structure From Motion (3D Reconstruction)

* [Objective](#Objective)
* [SFM_Pipeline](#SFM_Pipeline)
* [Results](#Results)

This project implements a complete Structure-from-Motion (SfM) pipeline from scratch. This was done as a part of Coursework for the EECE 7150 (Autonomous Field Robotics) course at Northeastern University.

The pipeline reconstructs 3D points and camera poses from unordered 2D images and is validated against COLMAP.

## Objective

The goal of this project was to build a full SfM system without relying on existing SLAM frameworks (COLMAP, OpenMVG, ORB-SLAM).
The pipeline includes:

- Feature extraction and matching
- Epipolar geometry estimation (F, E matrices)
- Camera pose recovery
- Triangulation of 3D landmarks
- Perspective-n-Points (PnP)
- Bundle Adjustment using GTSAM
- Visualization of camera trajectory and sparse 3D reconstruction

## SFM_Pipeline

### Camera Intrinsics (from COLMAP)

The camera intrinsic matrix (K) and radial distortion coefficient (k₁) are extracted by parsing the cameras.txt file produced by COLMAP. I also visualized the sparse reconstruction result of COLMAP for comparison.

COLMAP repository:
https://github.com/colmap/colmap

### Helper Functions Overview

1. Correcting Radial Distortion:
- ``` undistort_points_simple_radial() ```
Removes radial lens distortion for all detected keypoints using the distortion coefficient k1 obtained from COLMAP.

2. Fundamental Matrix Estimation:
- ``` normalize_points() ```
Applies Hartley normalization to improve numerical stability before running the 8-point SVD algorithm.
- ``` eight_point_F() ```
Implements the normalized 8-point algorithm to compute the Fundamental matrix F between two images.
- ``` sampson_error() ```
Computes the Sampson approximation of the geometric reprojection error — used as the scoring metric inside RANSAC.
- ``` F_Matrix() ```
Full RANSAC-based estimator: repeatedly samples 8 correspondences → computes F → selects the model with the most inliers (lowest Sampson error).

3. Essential Matrix:

- *E=KTFK*
  - K - Camera Intrinsic Matrix
  - F - Fundamental Matrix

4. Pose Recovery from the Essential Matrix:
- ``` decompose_E() ```
Decomposes the Essential matrix into four possible (R,t) pairs representing the relative camera motions.
- ``` triangulate_linear() ```
Performs DLT triangulation to compute 3D points from two camera views.
- ``` baseline_parallax() ```
Computes the median angular parallax between rays — used to ensure that triangulation is stable (sufficient baseline).
- ``` correct_pose() ```
Evaluates all four (R,t) decompositions by triangulating points and selecting the one that:
  - gives positive depth (points in front of both cameras)
  - has adequate parallax
This yields the correct relative camera pose.

### Feature Extraction & Pairwise Epipolar Matching

For each image:

- For each image, SIFT keypoints + descriptors are extracted
- Pairwise matching is performed with neighboring frames
- RANSAC filtering is applied to retain only epipolar-consistent inliers

This produces a reliable set of image correspondences across the dataset.

### Seed Pair Selection & Initial 3D Reconstruction

Among all matched image pairs, the pair with the:

   - highest number of inliers
   - strongest baseline
   - lowest reprojection error

is chosen as the seed pair.
This pair bootstraps the entire SfM process by providing the initial:
   - two camera poses
   - initial set of triangulated 3D landmarks

### Feature Tracks via Union–Find

Multi-view feature tracks are built using the Union–Find based track fusion method.

This merges all pairwise correspondences into globally consistent feature tracks, enabling incremental reconstruction.

### Camera Registration via PnP

Each new image is registered into the existing reconstruction by:

1. Matching its 2D keypoints to already-triangulated 3D points
2. Solving PnP to estimate its pose relative to the existing map

This step grows the SfM solution one view at a time.

### PnP + Triangulation Loop

For each new camera view:
- PnP recovers the camera pose
- Triangulation adds new 3D points
- Tracks are updated
- Reconstruction grows until all 24 images are registered

### Global Bundle Adjustment (GTSAM)

A full non-linear optimization using Levenberg–Marquardt (LM) is performed to jointly refine:

- all camera poses

- all 3D landmarks

GTSAM minimizes the total reprojection error and produces the final optimized reconstruction.

## Results:

### COLAMAP OUTPUT:

``` Plotly plot ```
<img width="1318" height="1132" alt="Screenshot from 2025-11-16 16-09-43" src="https://github.com/user-attachments/assets/ce2e1c90-30a4-4fd0-84ff-152859ee867d" />

### MY SFM OUTPUT (Before GTSAM optimization):

``` Matplotlib plot ```
<img width="577" height="590" alt="image" src="https://github.com/user-attachments/assets/cdf909f3-6d39-46d8-bfb0-0884192fe7b0" />

``` Plotly plot ```
<img width="1720" height="1265" alt="3D_Image_Reconstruction" src="https://github.com/user-attachments/assets/98a0d8f5-8629-455d-8226-90d29b9f5b9d" />

### MY SFM OUTPUT (After GTSAM optimization):

``` Plotly plot ```
<img width="1720" height="1265" alt="3D_Image_Reconstruction(GTSAM)" src="https://github.com/user-attachments/assets/7b324462-41ff-40bb-9b2f-5e7e4793eb90" />

### Quantitaive Results:

- Initial error: 3533887.474
- Final error:   1700053.542
- Improvement:   51.89%

RMS reprojection error: 20.076px → 13.925px
