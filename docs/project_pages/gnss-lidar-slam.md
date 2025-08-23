---
title: GNSS-aided 3D LiDAR SLAM (Lego-LOAM + GPS factor)
tags: [slam, lidar, gps, factor-graph, gtsam, perception]
---

# GNSS-aided 3D LiDAR SLAM (Lego-LOAM + GPS)

**Date:** December 5, 2023.

## Summary
The **Lego-LOAM** was extended with an absolute **GPS unary factor** in a factor-graph back end to curb long-term drift—especially in the **z-axis**—across real-world routes. The approach builds on ideas from **Lego-LOAM**, **LIO-SAM**, and **hdl_graph_slam** and shows reduced drift in evaluation sequences.

---

## Motivation: why fuse GPS with LiDAR?
Different sensors fail differently; leveraging their **complementarity** improves state estimation. GPS tends not to accumulate drift but can **jump** due to interference/obstruction; LiDAR odometry accumulates drift but avoids abrupt jumps and prefers dense environments, struggling in open spaces.

---

## System overview
A modern LiDAR SLAM stack separates a **front end** (LiDAR odometry) and a **back end** (graph optimization with solvers like g2o, Ceres, or **GTSAM**). Sensor data flows through odometry to produce a SLAM estimate, with the back end refining poses. 

Work is based on **Lego-LOAM** and incorporate lessons from **LIO-SAM** and **hdl_graph_slam**. 
---

## GPS integration design

### Factor type and coordinate frame
- Treat each GPS measurement as a **prior / absolute / unary observation** (a **GPSFactor**). 
- Convert **LLA → local Cartesian** (e.g., ENU) using **GeographicLib** before adding to the graph. 

### Optimization engine
- Using **GTSAM/iSAM2** avoids hand-deriving residual Jacobians (in contrast to defining them directly in Ceres). 
### Practical workflow
1. Compare measurement covariance; **set the GPS factor noise**.  
2. **Convert** GPS to the local Cartesian frame.  
3. **Add factor** to the graph.  
4. Run **iSAM2 update** after adding LiDAR odometry factor(s) and the GPS measurement.

### Robustness detail
- If the **GPS “jumps”**, give time for re-initialization — be **distrustful of initial measurements** before resuming normal fusion. 

---

## GPS modalities referenced
- **PPP** and **RTK** are noted; **RTK** can reach ~**10 cm** accuracy but requires base-station correction.

---

## Threads in Lego-LOAM
- **LiDAR odometry** (main thread), **loop closure**, and **visualization** threads are part of the baseline system architecture we extend.

---

## Datasets & evaluation

<div class="side-by-side" markdown="1" style="grid-template-columns: 1fr 3fr;">
<div markdown>

### UTBM (EU long-term autonomous driving dataset)
- **Heterogeneous sensors** and **long-term** sequences (~5 km, road loop) captured in **downtown** and **suburban** Montbéliard (France).
- Our comparisons show **reduced drift in the z-axis** versus baseline Lego-LOAM. 
</div>
<figure class="side-figure">
  <img src="../../assets/img/gnss-lidar-slam/utbm_comparison.png" >
</figure>
</div>

<div class="side-by-side" markdown="1" style="grid-template-columns: 1fr 3fr;">
<div markdown>

### Mulran (multimodal range dataset)
- **Radar + LiDAR**, multiple cities/environments, **month-level temporal gaps**, multiple loop candidates; sequences include **Riverside, 6.8 km road loop**. 
- Comparative overlays vs. baseline Lego-LOAM are presented.
</div>
<figure class="side-figure" >
  <img src="../../assets/img/gnss-lidar-slam/mulran_comparison.png" >
</figure>
</div>
---

## Credits & References
- Project presented along with *Zhexin (Jason) Xu*. 
- **Lego-LOAM** — Shan et al., IROS 2018. 
- **LIO-SAM** — Shan et al., IROS 2020. 
- **hdl_graph_slam** — Koide et al., IJARS 2019. 
- **GeographicLib** — coordinate conversion utility.  
- **UTBM** — Yan et al., IROS 2020 (EU long-term dataset).   
- **Mulran** — Kim et al., ICRA 2020 (multimodal range dataset). 

---
<div class="backbar" markdown>
[:material-arrow-left: Back to Projects](../projects.md){ .md-button }
</div>