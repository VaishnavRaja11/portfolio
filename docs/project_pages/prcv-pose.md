---
title: Bin-Picking 6DoF Pose Estimation (PRCV)
tags: [pose-estimation, multiview, yolo, triangulation, bop, ipd]
---

# Bin-Picking Model for 6DoF Pose Estimation

## Info "At a glance"        

- **Goal:** Estimate **6-DoF pose** of known parts for **bin-picking**, so a robot can place a grasp in SE(3) despite clutter and occlusion.
- **Approach:** Detect/segment the target, align the CAD model to observations (keypoints/contours → **PnP**), then **refine with depth/ICP** and reject outliers by score/visibility.
- **Inputs/Outputs:** RGB-D frames → object pose **T<sub>world</sub><sup>obj</sup>** (+ grasp pose) with confidence.
- **What worked:** CAD-guided refinement and depth-based filtering improved stability under partial occlusion; symmetric-object handling reduced pose flips.
- **Failure modes:** Heavy occlusion, specular/texture-poor parts, and extreme bin clutter; runtime depends on detector & ICP settings.
- **Next:** Multi-view fusion, uncertainty-aware grasp sampling, and on-robot validation.       

If you'd like to do a deep dive further on the project, Please feel free to go through the complete write-up below.

---

## Report

> Full report(PDF) is embedded below. Download links are provided if your browser blocks inline PDFs.


<div class="pdf-embed">
  <iframe src="../../assets/pdfs/PRCV_Project.pdf#view=FitH" title="PRCV Project Report"></iframe>
</div>

### **Credits:** Course Project presented along with Rahul Sha and Reza Farrokhi.
---

<div class="backbar" markdown>
[← Back to Projects](../projects.md){ .md-button }
</div>
