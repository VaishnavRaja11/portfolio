# Projects

A curated set of builds across **autonomy**, **robotics**, **embedded electronics**, and **mechanical systems**. Bullets emphasize what I owned, key technical decisions, and outcomes.
<div class="grid cards" markdown>

-   :material-factory: **Autonomous & Industrial Systems**

    ---

    AGVs, pick-to-light, inventory, IoT at the edge.  
    [:octicons-arrow-right-24: View](#autonomous-industrial)

-   :material-car: **Automotive & Mechanical Systems**

    ---

    BAJA ATV: CAD → CAE → fabrication → race.  
    [:octicons-arrow-right-24: View](#automotive-mechanical)

-   :material-toy-brick: **Hobby Projects**

    ---

    KUKA Drawing, Mitsubishi milling, Smart Room, Smart Locker.  
    [:octicons-arrow-right-24: View](#hobby-projects)

-   :material-dots-horizontal: **Miscellaneous Projects**

    ---

    Smart India Hackathon.  
    [:octicons-arrow-right-24: View](#misc-projects)

</div>

<hr/>

## Autonomous & Industrial Systems {: #autonomous-industrial }

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

<div class="project" markdown="1">

### Autonomously Guided Vehicle — YCH Logistics (Singapore)
**Summary.** Exploratory AGV with multiple guidance modalities for warehouse navigation.  
**My role.** System prototyping and method evaluation.      
- **Drive/control:** Three-wheel base; **two Arduinos (PID)** for steppers; **custom driver** hardware.     
- **Navigation trials:** **QR-code waypoints**, **line following**, and **metal strip** guidance (floor/wall).      
- **Evaluation:** Compared repeatability, latency, and maintenance demands across methods.      
- **Outcome:** Comparative basis for selecting a maintainable guidance scheme under real constraints.       

</div>

<hr/>

## Automotive & Mechanical Systems {:#automotive-mechanical}

<div class="project" markdown="1">

### All-Terrain Vehicle (ATV) — SAE BAJA (Team Ezhal)
**Summary.** Competition-grade ATV from concept to race; limited budget, high scrutiny.  
**My role.** **CAE lead** (Ansys), transmission support, and fabrication contributor.       
- **Design compliance:** Full **CAD** to **SAE BAJA rulebook**; **crash/impact** cases simulated and presented in first-round review.       
- **Fabrication:** **Machining, welding, assembly, testing** with rapid design changes in electrical and drivetrain.        
- **Innovation:** **Disengageable driveshaft** for tighter maneuverability and controlled drifting.     
- **Reliability fixes:** **Mud-blocking inlet cover** post-tests to protect intake.     
- **Result:** **State 2nd** and **9th overall** nationally.     

<div class="media-strip">
  <a href="../assets/img/baja/baja3.png"><img src="../assets/img/baja/baja3.png" alt="Fabrication and assembly"></a>
  <a href="../assets/img/baja/baja1.png"><img src="../assets/img/baja/baja1.png" alt="ATV frame CAD"></a>
  <a href="../assets/img/baja/baja2.png"><img src="../assets/img/baja/baja2.png" alt="Ansys crash/impact setup"></a>
  <a href="../assets/img/baja/baja4.png"><img src="../assets/img/baja/baja4.png" alt="Competition test runs"></a>
</div>
</div>

<hr/>

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
