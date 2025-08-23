---
title: Learning-Based Feature Selection for ORB-SLAM3
tags: [visual-slam, orb-slam3, yolo, feature-selection, perception]
---

# Learning-Based Feature Selection for ORB-SLAM3

**Date:** Dec 14, 2023  

## Summary

Integrated the **YOLOv5** with **ORB-SLAM3** to **filter features on dynamic objects** and improve data association in visual SLAM. After ORB keypoints are extracted, detections from YOLOv5 identify dynamic classes; any ORB features **inside those bounding boxes are dropped**, and tracking proceeds. We evaluate on **EuRoC**, **KITTI**, and a custom **NUance** driving dataset; results show near-perfect EuRoC trajectories, **no degradation** on KITTI 00 (≈**0.5%** translational drift with vanilla ORB-SLAM3), and **improved straight-line tracking** on the low-FPS NUance run. We also compare with **DROID-SLAM** as a learning baseline.

---

## Motivation

Dynamic objects corrupt epipolar geometry and feature correspondences, degrading VO/SLAM. Our objective was to **reduce the impact of moving objects** in real driving scenes (occlusions, motion, calibration & distortion issues) and produce better camera trajectories—especially on **NUance**, a challenging real-world dataset often used at Northeastern. 

---

## Datasets

- **EuRoC** — indoor visual-inertial MAV sequences with GT; few dynamic objects; good for sanity checks.  
- **KITTI** — outdoor stereo/LiDAR/IMU (2012); relatively controlled scenes with mainly **static parked** vehicles.  
- **NUance (Boston)** — custom ROS bag from the NUance self-driving car (cameras + IMU + GPS); **many dynamic objects** (pedestrians, cyclists, traffic). *Captured at ~2 FPS*, which stresses SLAM during turns. 

---

## System & Method

### Baseline and insertion point
We keep the standard **ORB-SLAM3** pipeline (front-end tracking, back-end map/loop). After ORB features are extracted on each frame, we run **YOLOv5** to obtain bounding boxes and **remove ORB features that fall inside those boxes** (assumed dynamic). The rest of ORB-SLAM3 proceeds unchanged. 

### Detector and classes
**YOLOv5** is used for its practical C++ integration and speed. We flag common dynamic classes: “person”, “car”, “motorbike”, “bus”, “train”, “truck”, “boat”, “bird”, “cat”, “dog”, “horse”, “sheep”, “crow”, “bear”—features within these boxes are discarded.  

---

## Experiments

### Setup
We tested in order of complexity: **EuRoC → KITTI → NUance**. Evaluation was both **qualitative** (trajectory & map overlays) and **quantitative** via **L1 error** against ground-truth (or GPS).  
### Metrics
- **Qualitative:** visual alignment and map continuity.  
- **Quantitative:** **L1 absolute error** vs ground truth/GPS trajectory. 

---
## Results

<div class="side-by-side" markdown="1">
<div markdown>

### EuRoC
- Trajectory closely follows ground truth (2D overlay).  
- **L1 error:** **X = 0.0226**, **Y = 0.0257** (very small residuals).

</div>
<figure class="side-figure">
  <img src="../../assets/img/rsn-orb-slam/euroc.png">
</figure>
</div>

<div class="side-by-side" markdown="1">
<div markdown>

### KITTI
- Vanilla **ORB-SLAM3** achieves **~0.5% translational drift**; the YOLO-filtered variant **matches** this, confirming no regression and similar compute behavior.

</div>
<figure class="side-figure">
  <img src="../../assets/img/rsn-orb-slam/kitti.png">
</figure>
</div>

<div class="side-by-side" markdown="1">
<div markdown>

### NUance (Boston)
- **Low FPS (~2 Hz)** is the primary failure driver during **sharp turns** (only ~4 frames across a 1–2 s maneuver).  
- With YOLO filtering, **straight segments** track better than vanilla, but **90° turns** still fail due to too few frames—map breaks around the U-turn.
</div>
<figure class="side-figure">
  <img src="../../assets/img/rsn-orb-slam/nuance-uturn.png" >
</figure>
</div>

### DROID-SLAM baseline
- On a **cropped NUance route** (avoiding the U-turn), **DROID-SLAM** produces a trajectory **close to GPS ground truth**, underscoring the value of learned feature selection and dynamic-object handling—albeit with **heavy compute** (NVIDIA Tesla V100 on Discovery cluster).
<div class="media-strip">
<img src="../../assets/img/rsn-orb-slam/droidslamresult.png">
<img src="../../assets/img/rsn-orb-slam/groundtruth.png">
</div>

---

## Limitations & Lessons

1) **Frame rate matters:** For car-scale motion, **20–30 FPS** is a practical floor; **2 FPS** is insufficient at intersections.  
2) **Turns vs straights:** Filtering helped straight-line tracking but couldn’t compensate for too few frames during rapid turns.  
3) **Proofs of concept:** On EuRoC and KITTI, the method **doesn’t break** ORB-SLAM3’s performance, supporting feasibility.  
4) **Deep SLAM trade-offs:** DROID-SLAM is promising but **computationally heavy** for onboard self-driving deployments.

---

<!-- ## Future Work

- **Real-time integration in ROS:** connect **ORB-SLAM3 + YOLOv5 + stereo** with minimal per-frame latency; compare multiple SLAM systems under the same stack.  
- **Better dynamics modeling:** upgrade to **segmentation masks** (e.g., newer YOLO with seg) instead of boxes; move beyond class-assumptions to **actual motion-based dynamics**.  
- **Data collection hygiene:** ensure higher FPS and more consistent turning behavior to improve sequence viability.  
--- -->

## Credits & Assets

- Presented the project along with Tarun Srinivasan and Thomas Rowan 
- **Demo:** DROID-SLAM on NUance.
<div class="video-embed">
  <iframe
    src="https://www.youtube-nocookie.com/embed/xo5Cw8TEuJM?rel=0&modestbranding=1"
    title="DROID-SLAM on NUance"
    loading="lazy"
    allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen
  ></iframe>
</div>

<div class="backbar" markdown>
[:material-arrow-left: Back to Projects](../projects.md){ .md-button }
</div>
