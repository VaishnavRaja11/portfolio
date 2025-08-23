# Projects

A curated set of End-to-end robotics projects across **Perception & SLAM**, **Industrial systems**, **Controls/Mechanisms**, and **Learning-based control**.  
Each card lists **what I owned**, the **stack**, and **one outcome**—with deep dives where useful.

<div class="grid cards" markdown>

-   :material-star: **Perception & SLAM**
    ---
    6DoF Pose Estimation, LiDAR SLAM, ORB-SLAM3 + YOLO  
    [:octicons-arrow-right-24: View](#perception-slam)
    <!-- [Open Related Projects](#perception-slam){ .card-hit aria-label="Open Perception & SLAM" } -->

-   :material-robot-industrial-outline: **Industrial Systems**
    ---
    AGV + Pick-to-Light, Stock Mgmt (SAP)   
    [:octicons-arrow-right-24: View](#industrial-systems)
    <!-- [See Related Projects](#industrial-systems){ .card-hit aria-label="Open Industrial Systems" } -->

-   :material-axis: **Controls & Mech**
    ---
    UR5 IK, Base-actuated RCM   
    [:octicons-arrow-right-24: View](#controls-mechanisms)
    <!-- [See Related Projects](#controls-mechanisms){ .card-hit aria-label="Open Controls & Mechanisms" } -->

-   :material-brain: **Learning-based Control**
    ---
    Deep RL Navigation (DDPG/TD3)   
    [:octicons-arrow-right-24: View](#learning-control)
    <!-- [Open Project ](#learning-control){ .card-hit aria-label="Open Learning-based Control" } -->

</div>



<hr/>

## Perception & SLAM {: #perception-slam }

<div class="project" markdown="1">

### Perception Model for 6DoF Pose Estimation
**Summary.** Built a baseline multi-view pipeline for **6DoF pose** of industrial parts: **YOLOv11** detects, **ResNet50/SimplePoseNet** regresses **rotation per view**, and **epipolar matching + triangulation** recovers **translation**, evaluated on **IPD** (Industrial Plenoptic Dataset).     
**My role.** End-to-end pipeline, training, epipolar matching/triangulation, analysis. 

- **Design:** Decouple **R** (learned per-view) and **t** (multi-view geometry); Hungarian matching on **symmetric epipolar distance** before triangulation.
- **Software:** YOLOv11 (Ultralytics), PyTorch **ResNet50** PoseNet, calibrated 3-cam rig. 
- **Metrics/notes:** Report **ADD-S** and rotation error; detector mAP≈**0.97** (synth), **0.84/0.81** P/R on real; 3-view runtime ≈ **<50 ms** on RTX 4070.  
- **Outcome:** Clean geometric **t** with consistent multi-view matches; identified next steps (**ICP refinement**, direct **t** regression, correspondence-based upgrades).

<!-- <div class="media-strip">
  <a href="../assets/img/prcv-pose/pipeline.png"><img src="../assets/img/prcv-pose/pipeline.png" alt="Pipeline: detect → pose (R) → epipolar match → triangulate (t)"></a>
  <a href="../assets/img/prcv-pose/detect.png"><img src="../assets/img/prcv-pose/detect.png" alt="YOLOv11 detections"></a>
  <a href="../assets/img/prcv-pose/epi.png"><img src="../assets/img/prcv-pose/epi.png" alt="Epipolar matching"></a>
  <a href="../assets/img/prcv-pose/triang.png"><img src="../assets/img/prcv-pose/triang.png" alt="Triangulated translation"></a>
</div> -->
[Read more →](project_pages/prcv-pose.md){ .md-button }
</div>
<hr/>

<div class="project" markdown="1">

### GNSS-aided 3D LiDAR SLAM (Lego-LOAM + GPS)
**Summary.** Extended **Lego-LOAM** with an absolute **GPS unary factor** in a factor graph to control drift—especially in the **z-axis**—on long, real-world routes. Evaluated on **UTBM** and **Mulran** datasets with clear drift reduction versus baseline.
**My role.** GPS factor integration & tuning (noise/gating), factor-graph plumbing in **GTSAM**, dataset evaluation/plots, and write-up.    

- **Architecture / design:** Classic LOAM front-end (odometry + mapping threads) + loop-closure; **iSAM2** back-end with **GPSFactor** fused as an absolute prior (LLA → local Cartesian via **GeographicLib**); outlier handling when GPS “jumps”.    
- **Software:** Built on **Lego-LOAM**, influenced by **LIO-SAM** and **hdl_graph_slam**; back-end in **GTSAM** (no custom Jacobians needed vs Ceres).     
- **Methods:** Converted GPS to ENU/local frame; adaptive noise and covariance checks; delayed trust on re-initialization; factors added incrementally before iSAM2 updates.    
- **Outcome:** Lower global drift on long sequences; **notable z-drift reduction**; robust behavior in multi-environment runs.    

<!-- <div class="media-strip"> -->
  <!-- Replace with your real images (plots, architecture, trajectory comparisons) -->
  <!-- <a href="../assets/img/gnss-lidar-slam/arch.png"><img src="../assets/img/gnss-lidar-slam/arch.png" alt="System diagram: LOAM front-end + iSAM2 back-end with GPSFactor"></a> -->
  <!-- <a href="../assets/img/gnss-lidar-slam/utbm_comp.png"><img src="../assets/img/gnss-lidar-slam/utbm_comp.png" alt="UTBM: baseline vs GNSS-aided trajectory overlay"></a> -->
  <!-- <a href="../assets/img/gnss-lidar-slam/mulran_comp.png"><img src="../assets/img/gnss-lidar-slam/mulran_comp.png" alt="Mulran: loop alignment with reduced drift"></a> -->
<!-- </div> -->

<!-- Optional video (if you have a demo): place at docs/assets/video/gnss-lidar-slam/clip.mp4 -->
<!--
<video src="../assets/video/gnss-lidar-slam/clip.mp4" controls preload="metadata" style="width:100%; border-radius:10px; margin-top:.5rem;"></video>
-->

<!-- Optional deep dive page -->
[Read more →](project_pages/gnss-lidar-slam.md){ .md-button }

</div>

<hr/>

<div class="project" markdown="1">

### Learning-Based Feature Selection for ORB-SLAM3
**Summary.** Integrated **YOLOv5** with **ORB-SLAM3** to drop ORB features that fall inside **dynamic-object** detections, aiming to stabilize data association in real scenes. Evaluated on **EuRoC**, **KITTI 00**, and a custom **NUance** driving dataset.    
**My role.** Integrated the YOLO→feature filtering step, ran experiments, and analyzed trajectories/metrics.    

- **Architecture / design:** Keep the standard ORB-SLAM3 pipeline; after ORB keypoints are extracted, run **YOLOv5** and **remove features inside dynamic-class boxes** (person, car, truck, etc.). Tracking, mapping, and loop-closure proceed unchanged.    
- **Software:** ORB-SLAM3 (C++), YOLOv5 for detections; scripts for plotting **trajectory overlays** and **L1 error** against GT/GPS.   
- **Methods:** Compare baseline vs. filtered variant across datasets; qualitative overlays + **L1 absolute error**.   
- **Outcome:** **EuRoC:** near-perfect overlays. **KITTI 00:** parity with vanilla ORB-SLAM3 (~0.5% drift). **NUance (~2 FPS):** cleaner straight-line tracking; sharp turns still fail due to too few frames. Also compared with **DROID-SLAM** as a learning baseline (strong accuracy, high compute).    

<div class="media-strip">
  <!-- Replace with your images; keep 2–4 -->
  <!-- <a href="../assets/img/rsn-orb-slam/euroc.png"><img src="../assets/img/rsn-orb-slam/euroc.png" ></a>
  <a href="../assets/img/rsn-orb-slam/kitti.png"><img src="../assets/img/rsn-orb-slam/kitti.png" ></a>
  <a href="../assets/img/rsn-orb-slam/nuance-uturn.png"><img src="../assets/img/rsn-orb-slam/nuance-uturn.png" ></a> -->
  <a href="../assets/img/rsn-orb-slam/droidslamresult.png"><img src="../assets/img/rsn-orb-slam/droidslamresult.png"></a>
  <a href="../assets/img/rsn-orb-slam/groundtruth.png"><img src="../assets/img/rsn-orb-slam/groundtruth.png" ></a>
</div>

[Read more →](project_pages/rsn-orb-slam3.md){ .md-button }

</div>

<hr/>

## Learning-based Control {: #learning-control }

<div class="project" markdown="1">

### Autonomous Robot Navigation with Deep RL

**Summary.** Trained a **differential-drive robot** in a custom 2D “EscapeRoom” simulator to **reach a goal without collisions** using **DDPG** and **TD3**. Continuous actions are left/right wheel speeds; a **shaped reward** balances progress, heading, time, collisions, and success bonus.   
**My role.** Co-author: environment + reward shaping, PyTorch implementation, training/eval, analysis. 

- **Design:**     
    State: \[x, y, heading, linear & angular speed, distance to goal, angle to goal\]  
    Action: \[left wheel speed, right wheel speed\] ∈ \[-1, 1\]
- **Algorithms:** **DDPG** (actor–critic with replay + targets) and **TD3** (double critics, delayed policy, target smoothing). 
- **Outcome:** DDPG reduced collisions but showed critic instability; **TD3** stabilized learning and improved episode reward trends, though occasional regressions remained. 
<!-- Optional images -->
<!-- <div class="media-strip">
  <a href="../assets/img/rl-escape-room/env.png"><img src="../assets/img/rl-escape-room/env.png" alt="EscapeRoomEnv render"></a>
  <a href="../assets/img/rl-escape-room/rewards.png"><img src="../assets/img/rl-escape-room/rewards.png" alt="Episode rewards trend"></a>
</div> -->
[Read more →](project_pages/rl-escape-room.md){ .md-button }
</div>
<hr/>

## Controls & Mechanisms {: #controls-mechanisms }

<div class="project" markdown="1">

### UR5 — Kinematic Control for a Square Trajectory
**Summary.** Programmed a **UR5** to trace a **square path in 3D** while keeping a fixed tool orientation. Built the **forward kinematics** from a DH model and used a **Newton/iterative IK** (Jacobian-based, body frame) to solve \( \theta \) for each waypoint; added a **3D animation** to verify the motion.   
**My role.** End-to-end: DH modeling, FK, **IK solver**, trajectory planner, and animation in **MATLAB**.   

- **Robot / model:** UR5, 6R manipulator; **standard DH table** and link lengths per spec. 
- **FK:** chained transforms to obtain the end-effector pose \(T^0_6\).
- **Trajectory:** corner waypoints define a task-space square; interpolate line segments.
- **IK:** **Newton’s method** with the Jacobian; iterates from the previous pose for smooth convergence.
- **Validation:** animated stick-figure UR5 following the path; orientation held constant.
<div class="video-embed" style="--maxh:560px;"> <iframe
    src="https://www.youtube-nocookie.com/embed/fRtvTPKPAiI?rel=0&modestbranding=1"
    title="RCM demo" loading="lazy"
    allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen></iframe>
</div>
</div>
<hr/>

<div class="project" markdown="1">

### Base-Actuated 3-Rhombus RCM Mechanism
**Summary.** Designed a **planar 2-DOF Remote Center of Motion (RCM)** mechanism with **all actuators on the base** to avoid proximal-actuation issues (RCM offset, added inertia, blocked field of view). Work covers **concept → kinematics → singularities → SolidWorks simulation**; intended for MIS tasks (e.g., lumbar puncture).    
**My role.** Solo project: literature review, mechanism synthesis, analysis, CAD and motion verification. 

- **Architecture / design:** Three-rhombus linkage; two **active links** at the base drive the end-effector. **Decoupled motions:** same-direction drive → rotation about RCM; opposite-direction drive → translation. 
- **Analysis:** Planar DOF check (Grübler) confirms **2 DOF**.  
  Forward kinematics: θ = ½(θ₁ + θ₂) , d = L₅ − L₃·cos(α) − L₁·cos((θ₁ − θ₂)/2).  
  Singularity when α = (θ₁ − θ₂)/2. 
- **Implementation:** SolidWorks model with base motors + linear guides; needle as end-effector; verified home/extended and ±rotation configs.
- **Outcome:** Validated RCM behavior and workspace sector; promising for MIS; next step: spatial extension. 
<div class="video-embed" style="aspect-ratio: 4/3; --maxw:560px" > <iframe
    src="https://www.youtube-nocookie.com/embed/YCBZ6UfwL7Y?rel=0&modestbranding=1"
    title="RCM demo" loading="lazy"
    allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen></iframe>
</div>
</div>
<hr/>

<div class="project" markdown="1">

### All-Terrain Vehicle (ATV) — SAE BAJA (Team Ezhal)
**Summary.** Competition-grade ATV from concept to race; limited budget, high scrutiny.  
**My role.** **CAE lead** (Ansys), transmission support, and fabrication contributor.       
- **Design compliance:** Full **CAD** to **SAE BAJA rulebook**; **crash/impact** cases simulated and presented in first-round review.       
- **Fabrication:** **Machining, welding, assembly, testing** with rapid design changes in electrical and drivetrain.        
- **Innovation:** **Disengageable driveshaft** for tighter maneuverability and controlled drifting.     
- **Reliability fixes:** **Mud-blocking inlet cover** post-tests to protect intake.     
- **Result:** **State Runner-ups** .     

<div class="media-strip">
  <a href="../assets/img/baja/baja3.png"><img src="../assets/img/baja/baja3.png" alt="Fabrication and assembly"></a>
  <a href="../assets/img/baja/baja1.png"><img src="../assets/img/baja/baja1.png" alt="ATV frame CAD"></a>
  <a href="../assets/img/baja/baja2.png"><img src="../assets/img/baja/baja2.png" alt="Ansys crash/impact setup"></a>
  <a href="../assets/img/baja/baja4.png"><img src="../assets/img/baja/baja4.png" alt="Competition test runs"></a>
</div>
</div>

<hr/>

## Industrial Systems {: #industrial-systems }

<div class="project" markdown="1">

### Autonomously Guided Vehicle (AGV) — Accenture (Bengaluru)
**Summary.** Three-wheeled AGV (two powered steppers + caster) integrated into a warehouse flow; edge control with cloud coordination.  
**My role.** End-to-end prototyping (mechanical, electronics, firmware) and system integration.     
- **Architecture:** Raspberry Pi as the **central controller**; **two Arduinos** running **PID** loops for each stepper; **custom motor driver** for current control.   
- **Software:** **MongoDB + Node.js + React**; a **local server on the Pi** syncs with a central server and inventory DB.      
- **Navigation:** Pre-fed location matrix and odometry; tuned for smooth starts/stops and accurate docking.      
- **Outcome:** Predictable task execution on the shop floor with live status and hand-offs. 

<div class="media-strip">
  <a href="../assets/img/agv/agv1.png"><img src="../assets/img/agv/agv1.png" alt="AGV prototype – chassis and electronics bay"></a>
  <a href="../assets/img/agv/agv2.png"><img src="../assets/img/agv/agv2.png" alt="AGV – dual stepper drive with custom driver"></a>
</div>
</div>

<hr/>

<div class="project" markdown="1">

### iWarehouse — AGV + Pick-to-Light Integration (Accenture)
**Summary.** Merged **Pick-to-Light** with the AGV for autonomous putaway/replenishment.  
**My role.** Hardware–software integration and workflow design.   
- **Pick-to-Light:** **Weight-based bin sensing** triggers light cues; **count displays** show current inventory and pick/put confirmations.    
- **Flow:** Orders → central server → Pi (edge) → AGV motion + light cues; confirmations sync back to inventory.    
- **Tech:** Same AGV control stack; expanded React UI for operators.        
- **Outcome:** Faster material movement and fewer pick errors; compelling demo for stakeholders/customers.

<div class="media-strip">
  <a href="../assets/img/iwarehouse/iw2.png"><img src="../assets/img/iwarehouse/iw2.png" alt="iWarehouse – bin weights and count display"></a>
  <a href="../assets/img/iwarehouse/iw1.png"><img src="../assets/img/iwarehouse/iw1.png" alt="iWarehouse – pick-to-light lanes with AGV"></a>
</div>
</div>

<hr/>

<div class="project" markdown="1">

### Intelligent Stock Management System — Accenture (Chennai)
**Summary.** IoT stock monitoring with **load-cell bins** that auto-initiate refill orders; demo integrated with **SAP**.  
**My role.** Hardware design (sensor + power), firmware, and backend integration.  
- **Sensing:** **ESP8266** nodes read **load cells**; thresholding/smoothing to reject noise and detect real consumption.  
- **Edge → server:** Local Pi broker relays updates → central server → **SAP** for order creation in the demo environment.  
- **Electronics:** Linear regulation, grounding/decoupling for stable ADC readings.  
- **Recognition:** **Best Innovator Award (Accenture, Oct 2019)**.  
- **Outcome:** Reliable refill triggers and bin-level visibility; reduced manual checks and out-of-stock risk.

<div class="media-strip">
  <a href="../assets/img/istockmanagementsystem/isms1.png"><img src="../assets/img/istockmanagementsystem/isms1.png" alt="Load-cell bin node with ESP8266"></a>
  <a href="../assets/img/istockmanagementsystem/isms2.png"><img src="../assets/img/istockmanagementsystem/isms2.png" alt="Stock dashboard and refill trigger flow"></a>
</div>
</div>
<hr/>

<!-- <div class="project" markdown="1">

### Autonomously Guided Vehicle — YCH Logistics (Singapore)
**Summary.** Exploratory AGV with multiple guidance modalities for warehouse navigation.  
**My role.** System prototyping and method evaluation.      
- **Drive/control:** Three-wheel base; **two Arduinos (PID)** for steppers; **custom driver** hardware.     
- **Navigation trials:** **QR-code waypoints**, **line following**, and **metal strip** guidance (floor/wall).      
- **Evaluation:** Compared repeatability, latency, and maintenance demands across methods.      
- **Outcome:** Comparative basis for selecting a maintainable guidance scheme under real constraints.       

</div>
<hr/> -->

## Hobby/Fun Projects {: #hobby-projects}

### Robotics & Automation

<div class="project" markdown="1">

#### KUKA Robotic Arm Drawing
**Summary.** Parametric motion pipelines for KUKA using **Grasshopper** integrated with **KRC**.  
**My role.** Toolchain integration and workflow setup.      
- Built **Grasshopper** definitions to generate toolpaths and poses; exported to KRC programs.      
- Reduced iteration time from CAD geometry to robot motion with reusable templates.     

<div class="media-strip">
  <a href="../assets/img/hobby/krc-draw.png"><img src="../assets/img/hobby/krc-draw.png" alt="Grasshopper-driven KUKA path planning"></a>
</div>
</div>

<hr/>

<div class="project" markdown="1">

#### Mitsubishi Robotic Arm Milling
**Summary.** Added a **high-torque spindle** to a Mitsubishi arm for light milling operations.  
**My role.** Integration and controls.      
- **Robot control:** **Mitsubishi Melfa** for arm actuation; **GX Works** for PLC I/O/safety.       
- **Mechatronics:** Fixture design, stiffness checks, conservative feeds/speeds for proof-of-concept.       
- **Outcome:** Successful pilot milling; defined requirements for rigidity upgrades.

<div class="media-strip">
  <a href="../assets/img/hobby/melfa-milling.png"><img src="../assets/img/hobby/melfa-milling.png" alt="Mitsubishi arm with spindle for milling"></a>
</div>
</div>

<hr/>

### IoT

<div class="project" markdown="1">

#### Smart Room
**Summary.** Custom PCB + firmware to retrofit a standard room with IoT controls.  
- **Electronics:** **KiCad** board around an **ESP** module; relay drivers, power conditioning, isolation.  
- **Firmware:** Wi-Fi provisioning, topic-based control, simple REST/MQTT endpoints.  
- **Outcome:** Reliable switching/telemetry with safe mains isolation.

<div class="media-strip">
  <a href="../assets/img/hobby/smartroom.png"><img src="../assets/img/hobby/smartroom.png" alt="Smart Room PCB and control panel"></a>
</div>
</div>

<hr/>

<div class="project" markdown="1">

#### Smart Locker
**Summary.** Office-space locker system for guided deliveries and pickups.  
- **Compute:** **Raspberry Pi** with camera and simple UI.  
- **Mechanics:** Solenoid/servo latch control; status LEDs; QR-based pickup flow.  
- **Outcome:** Reduced hand-offs and better audit trail for small-parcel movement.

<div class="media-strip">
  <a href="../assets/img/hobby/smartlocker1.png"><img src="../assets/img/hobby/smartlocker1.png" alt="Smart Locker – external view"></a>
  <a href="../assets/img/hobby/smartlocker2.png"><img src="../assets/img/hobby/smartlocker2.png" alt="Smart Locker – internal mechanism"></a>
</div>
</div>

<hr/>

## Miscellaneous projects {: #misc-projects}

### Smart India Hackathon(2019)
**Summary.** **Knowledge Management portal** for DPSUs to share practices and avoid repeat mistakes.  
**My role.** Front-end implementation and UX refinement.  
- **Stack:** **React**, **Node/Express**, **PostgreSQL**.  
- **Features:** Discussion threads, resource repositories, searchable case histories.  
- **Outcome:** **'Won the 2019 edition'**; demonstrated tangible value in cross-org knowledge reuse.

<hr/>

<div class="backbar" markdown>
[↑ Back to top](#projects){ .md-button }
</div>
