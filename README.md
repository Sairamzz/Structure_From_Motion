# Structure From Motion (3D Reconstruction)

This project implements a complete Structure-from-Motion (SfM) pipeline from scratch. This was done as a part of Coursework for the EECE 7150 (Autonomous Field Robotics) course at Northeastern University.

The pipeline reconstructs 3D points and camera poses from unordered 2D images and is validated against COLMAP.

## OBJECTIVE

The goal of this project was to build a full SfM system without relying on existing SLAM frameworks (COLMAP, OpenMVG, ORB-SLAM).
The pipeline includes:

- Feature extraction and matching

- Epipolar geometry estimation (F, E matrices)

- Camera pose recovery

- Triangulation of 3D landmarks

- Perspective-n-Points (PnP)

- Bundle Adjustment using GTSAM

- Visualization of camera trajectory and sparse 3D reconstruction

* [Objective](#Objective)
* [Data_Collection](#Data_Collection)
* [Features](#Features)
* [Implementation](#Implementation)
* [Results](#Results)
  * [Actual_path_taken](#Actual_path_taken)
  * [GPS_Path_Trajectory](#GPS_Path_Trajectory)
  * [IMU_Path_Trajectory](#IMU_Path_Trajectory)
  * [Comparison_of_GPS+IMU](#Comparison_of_GPS+IMU)
* [How_to_run](#Howtorun)


## COLMAP
<img width="1318" height="1132" alt="Screenshot from 2025-11-16 16-09-43" src="https://github.com/user-attachments/assets/ce2e1c90-30a4-4fd0-84ff-152859ee867d" />

