# Webber - A Wheeled Bi-Pedal Robot (WBR): CAD, Structure, and the First Printed Build

**Date:** September 4, 2026.

The [previous post](webber_W0W1.md) ended with one word: CAD.

What happened in CAD was not what the sizing study prescribed. The link dimensions changed, the material changed, and the reach envelope turned out smaller than the range the optimizer had been sweeping over. None of that was an accident, and none of it was the optimizer being wrong. It was a set of decisions made against criteria the cost function never held.

This post is about those decisions. It is also about deliberately building a structure I already knew was provisional, and being specific about what that bought and what it cost.

---

## Correction To The Previous Post

The sizing table in the topology-study post listed the wheeled geometry as `L1 = 111 mm`, `L2 = 156 mm`, foot extension `76 mm`. Those numbers are retired. They came from a pre-CAD sizing run and they do not describe the robot that exists.

The as-built geometry is `L1 = 70 mm`, `L2 = L3 = 140 mm`, carrier extension `70 mm`.

I am leaving the original post up rather than silently editing it, because the gap between the two tables is the most useful thing in this write-up. But anyone quoting `111 / 156 / 76` is quoting a robot that was never built.

---

## What The Sizing Study Actually Handed To CAD

The handoff from MATLAB to SolidWorks was not a set of link lengths. It was a loads document, and the loads survived the CAD process even though the dimensions did not.

The study evaluated the parallel 4-bar across three load scenarios over the posture sweep: static support, an `8.6°` forward lean, and a `0.3 g` horizontal acceleration. Nine postures against three scenarios gave `27` load cases. Joint torques came from the Jacobian transpose, `τ = Jᵀ F`, with a genetic algorithm seeding an `fmincon` refinement, under the guard constraints described in the previous post.

What came out of it:

| Output | Value | Where it went in CAD |
| --- | --- | --- |
| Peak knee torque | `≈ 4.9–5.0 Nm` | binding actuator; drove the thermal concern |
| Peak hip torque | `≈ 3.9 Nm` | hip bracket sizing |
| Wheel axle resultant | `36.4–37.0 N` | axle and hub sizing |
| Bearing ODs | `20 / 15 / 12 mm` | hip pivot, linkage joints, wheel axle |
| Passive fold height | `167.8–169.0 mm` | inside the `150–170 mm` target band |
| Dynamic gate | passed, against a `1.20×` posture-matched static threshold | no geometry change required |

The knee number was the important one. The `GIM6010-8` is rated `5 Nm` continuous at the output with an `11 Nm` transient peak, and the practical continuous thermal limit I had been designing to was `4.0 Nm`. The study said the knee would sit above that at the tight end of the posture range. It was a flag, not a failure, and I carried it into CAD knowingly.

The handoff also carried a materials recommendation: Al6061, safety factor `1.5`, allowable stress `184 MPa`, topology optimization targeting `30–40%` mass removal, and a `6–10 mm` lateral offset on the parallel rods to clear interference.

I used the loads. I did not use the material. That is covered further down.

---

## The Geometry That Got Built

| Source | `L1` | `L2 = L3` | Carrier extension |
| --- | --- | --- | --- |
| Sizing study, run A | `122.43 mm` | `155.38 mm` | `66.70 mm` |
| Sizing study, run B | `135.21 mm` | `153.69 mm` | `56.72 mm` |
| Retired pre-CAD table | `111 mm` | `156 mm` | `76 mm` |
| **As-built** | **`70 mm`** | **`140 mm`** | **`70 mm`** |
| DIABLO, Table I | `90 mm` | `140 mm` | `140 mm` total |

The as-built set was not a rounded version of the optimizer output. It satisfies a relationship the optimizer's results do not:

```
L1 + L4_lower = L2
70 + 70 = 140
```

That identity makes the five-bar symmetric, and symmetry collapses the leg kinematics to closed form. With `θ1` and `θ2` as the two hip-side joint angles:

```
γ         = (θ1 + θ2) / 2          # carrier orientation
L         = 280 · cos((θ1 - θ2)/2) # hip-to-wheel distance
half_diff = acos(L / 280)          # inverse kinematics
```

No circle intersections. No two-branch assembly ambiguity to resolve at runtime. No iterative solver inside a `500 Hz` control loop, and no worst-case iteration count to budget for in the loop timing. Forward and inverse kinematics are a handful of trigonometric evaluations with deterministic cost.

Check the optimizer's numbers against the same identity. Run A gives `122.43 + 66.70 = 189.13`, against an `L2` of `155.38`. The optimum is not symmetric, and a non-symmetric five-bar needs a numeric position solve with explicit branch selection every cycle.

My cost function scored torque uniformity across the workspace. It had no term for analytic tractability, and no term for whether the resulting geometry let me reuse a control formulation I had already decided to adopt. Those two considerations decided the build, and they were decided by hand with the optimizer's feasibility map in front of me rather than instead of it.

There is an honest gap here, and I would rather state it than have someone find it. **The as-built geometry has never been scored against the `27` load cases.** The pipeline exists and the design vector is a one-line change, so this is laziness rather than difficulty. Until that run happens, I know the built leg is kinematically clean and I do not know its torque profile in my own load model. That is the top item on the open list.

---

## Series Hips, And The Encoder Consequence

Both leg motors sit at the hip. The hip motor is flanged to the knee motor, giving a series connection rather than a parallel coaxial arrangement. DIABLO uses the same arrangement and gives the reason: for the same force applied at the foot, the series connection demands less torque from the motors than a parallel coaxial layout would.

The consequence shows up in firmware rather than in CAD. Because the knee motor's stator is carried on the hip motor's output, the knee encoder reads a relative angle, `θ2 - θ1`, not an absolute joint angle in the body frame. Every kinematic expression above wants absolute angles. That conversion has to happen once, early, in the state-estimation path, and it has to be the same convention everywhere. Getting it wrong produces a leg that tracks perfectly at the calibration pose and drifts as it moves, which is an unpleasant class of bug to chase.

There is a second consequence that matters more for the research side of this project. Webber's motors are geared `8:1`. DIABLO's are direct drive. Reflected rotor inertia scales with the square of the reduction, so the `8:1` gearbox multiplies each rotor's inertia by `64` as seen at the joint. On a robot this size that term is comparable to structural link inertia, which means DIABLO's equations of motion cannot be lifted across unmodified. That correction is the subject of a later post, but the mechanical arrangement is where it originates.

---

## Bent Links, And Why Pin-To-Pin Did Not Move

The links are bent. The kinematics are not.

With straight links, the fold pose is limited by the links themselves. The bend routes material out of the interference volume, and the result is that the hip, knee and wheel axes all come onto a single line when the leg is fully folded. Straight links cannot reach that pose at these proportions.

The discipline that mattered here was keeping pin-to-pin geometry identical. Every joint centre sits exactly where the straight-link version put it. The bend is a clearance feature and nothing else.

That constraint was not aesthetic. The symmetry identity above depends on the pin-to-pin lengths, the closed-form kinematics depend on the identity, and the LQR model depends on the kinematics. Moving a joint centre by a few millimetres to make a bend easier to print would have propagated into the control model and quietly invalidated it. The structural geometry and the kinematic geometry were treated as two separate things that happen to share a part file, and only one of them was allowed to change.

Link cross-sections came out at `10 mm` thickness on `L1` and `14 mm` on the carrier extension.

---

## Rollers Under The Head

This one is borrowed, and I want to be clear about that. DIABLO places four rollers beneath the head, and in crawling mode those rollers support the entire system while the two wheels provide power, which the paper describes as improving energy efficiency on horizontal terrain. Webber does the same thing for the same reason.

The stack-up on Webber:

- Crawl pose sits at `154 mm` hip height
- The legs fold a further `40 mm` clear of the ground in that pose
- Adding the rollers cost `36 mm` of standing ride height
- Rollers printed in TPU at `16 mm` radius, for floor grip

The `40 mm` clearance is the whole point, and it is why this claim does not need a measurement to stand up. In the crawl pose the legs are not lightly loaded, they are off the ground. The leg motors hold nothing. Body weight goes through the rollers, and the wheel motors do the driving. Whatever the efficiency number turns out to be, the load path is unambiguous.

What I have not done is measure it. The honest statement is that holding current at the leg joints goes to zero by geometry, and that the efficiency benefit is inherited from DIABLO's claim rather than established on my hardware. Logging driver current in crawl pose against a standing pose is a ten-minute test and it is on the list.

One thing this does **not** fix is the knee thermal issue. The knee is the binding actuator at low stance because the links approach perpendicular to the body there, and the rollers do not change that geometry. They provide a resting pose where nothing is loaded. They do not reduce torque during a low stance the robot is actively holding. Two separate problems.

---

## Structural Design: The Topology Study

The material removal was driven by a SolidWorks Simulation topology study on the head and the links, loaded with the peak joint loads from the sizing study applied at a `10 kg` body mass. The measured robot is `5.17 kg`, and the model in the sizing study assumed `6.0–6.1 kg`, so the structural case ran at roughly twice the real mass and about `1.6×` the analytical model.

That factor was deliberate. This is a first structure in plastic with an unquantified material allowable, so a nominal safety factor computed against a handbook yield number would have been a false precision. Overloading the case by `2×` and then removing material until the study stopped complaining is a cruder method, and it is honest about being crude.

Material came off by hand, guided by the study output rather than exported from it. That is normal practice and worth explaining for anyone who has not run one: a topology study returns an organic density field, not a manufacturable body. The output tells you where load is not flowing. Turning that into pockets, ribs and fillets that a printer can produce without bridging failures is an interpretation step, and it is where the engineering judgment actually sits.

Results were asymmetric on purpose:

- **Head: `30%` mass reduction.** The largest single mass in the structure, and the biggest lever.
- **Links: much less.** The legs are the part of the design I expect to keep. The head is the part I expect to change once the electronics layout settles. Effort went where the geometry was still moving, not where it was already converged.

The head also did not save the print time I expected, because the pocketing added support material. Mass came down and print time did not. Worth knowing before optimizing a part for filament cost.

### The Material Decision

The handoff recommended Al6061 at a `1.5` safety factor. The robot is FDM plastic: PLA links, a PETG head, TPU rollers.

The reasoning was scope. The purpose of this structure is to unblock the electronics and the control software. Software will tell me what the hardware actually needs, in ways that a structural study cannot predict, and committing to machined aluminium before that conversation happens would have been an expensive guess. A provisional structure that gets the control loop running should cost fewer total iterations than a good structure designed against assumptions.

What that decision costs, stated plainly:

- No continuous-stress margin worth quoting. PLA creeps under sustained load at room temperature, and the leg joints hold static load whenever the robot stands.
- Layer adhesion becomes the governing failure mode rather than bulk yield, which means the useful strength depends on print orientation and cannot be read off a datasheet.
- The topology study ran against isotropic properties. Printed parts are not isotropic. The study's stress field is directionally correct and quantitatively unreliable.

None of that is an argument for having done it differently at this stage. It is an argument for not quoting a safety factor.

---

## Print And Assembly

- **Links: PLA.** Stiffer than PETG per unit mass, and stiffness is what the linkage needs.
- **Head: PETG.** Tougher and more temperature-tolerant, which matters because the electronics and the battery live inside it.
- **Rollers: TPU.** Purely for floor grip.
- **Layer orientation:** links printed with the layer lines kept off the tension axis, so the load path runs along the extrusions rather than across layer boundaries.
- **Joints: shoulder bolts.** The unthreaded shoulder gives a proper bearing surface at each revolute joint. Running a threaded fastener through a rotating joint chews the bore out and introduces backlash that shows up as position error in the leg.
- **Head-to-leg interface: screws directly into plastic.** No heat-set inserts yet. This is known debt. Threads formed in plastic lose preload every time the joint is broken down, and this assembly gets broken down often during bring-up. Heat-set inserts are the retrofit.

### Wheels

The wheels are off-the-shelf RC crawler tyres, `120 mm` OD and `45 mm` wide, on a printed hub adapter that bolts to the wheel motor output. Nominal rolling radius `60 mm`.

Buying them was a scope decision of the same kind as the plastic structure. Developing a TPU tyre with acceptable grip and acceptable durability is its own project, and it is not on the critical path to a balancing robot. The tyres cost very little and grip better than anything I would have printed on a first attempt.

There is one open item attached to this. A pneumatic-style crawler tyre has a loaded radius smaller than its free radius, and the wheel radius enters the balance dynamics directly. The LQR model wants the loaded value under the actual robot weight, not the catalogue number. Measuring it is trivial and has not been done.

---

## What The Hardware Measured

This is the part where the CAD model and the physical robot are allowed to disagree.

**Mass.** CAD body `4.5756 kg`. Measured robot `5.17 kg`. The delta is harness, fasteners, breadboard and the things that never get modelled. Worth noting that electronics account for roughly `36%` of total mass, at a fixed and comparatively high location in the head, so suppressing them in a simulation model does not produce noise, it produces a systematically wrong COM height.

**Inertia about the COM,** for the balance model: `I_pitch = 0.040165 kg·m²`, `I_yaw = 0.084724 kg·m²`, `I_roll = 0.110701 kg·m²`, with the COM `137.28 mm` from the reference datum.

**Reach envelope.** Measured on the built leg:

| Quantity | Value |
| --- | --- |
| Mechanical range | `95.5 – 330.4 mm` |
| Kinematic singularity | `340 mm`, links collinear |
| Validated commandable band | `150 – 280 mm`, later extended to `100 – 300 mm` |
| Crawl pose | `154 mm` |

Compare that against the sizing study, which swept `200 mm` to `400 mm`. The top `60 mm` of that sweep does not physically exist on this robot, and the region above `330 mm` is unreachable at any torque. The study's torque predictions at high stance describe a leg that was not built. This does not invalidate the mid-range results, which is where the robot actually operates, but it is a reminder that an optimizer will happily report numbers for postures you cannot reach.

**Position tracking under load,** at `250 mm` stance: `0.14°` error at the hip, `0.01°` at the knee. Both legs matched within `0.01 mm` at commanded heights of `220`, `250` and `280 mm`. The as-shipped driver position gains were sufficient. No tuning was required at this stage, which was a better outcome than I expected from plastic links and printed joints.

**Thermal.** The knee is the binding actuator at low stance, exactly as the sizing study predicted, and it binds for the geometric reason the study identified: the links approach perpendicular to the body, so the moment arm works against the knee. On hardware, sustained low stance drives the knee drivers into thermal limiting, with one node reaching `105.8 °C` at the FETs against a `95 °C` limit.

That is the single most useful result in this whole sequence. The dimensions the optimizer produced were not used. The actuator it identified as the constraint, and the reason it gave, held up against hardware months later. The sizing study earned its keep as a diagnostic even though it lost the argument about geometry.

---

## Standard Practice, And What Is Actually Mine

This project has a research bar, so it is worth separating the two.

**Standard practice, competently applied:** the parallelogram leg architecture, the series hip arrangement, rollers under the head for a passive resting pose, topology optimization for mass removal, and FDM prototyping to de-risk a structure before committing to metal. All of it is established, and DIABLO in particular is the direct source for three of those five.

**Provisional by choice, not contributions:** the plastic structure, screws into plastic without inserts, and the absence of anisotropic structural verification. These are known debt with known retrofits.

**Arguably mine:**

- The bent-link geometry that brings all three joint axes collinear in the fold pose, while holding pin-to-pin lengths invariant so the control model is untouched.
- The decision to trade optimizer cost for a symmetric geometry that yields closed-form kinematics, and the discipline of treating structural and kinematic geometry as separately governed.
- The reflected-inertia correction required to apply a direct-drive control formulation to a geared parallel-leg platform, which is where this project departs from its reference and is the subject of the next post.

---

## Open Work

In rough order of how much it bothers me:

1. Score the as-built `70 / 140 / 140` geometry against the `27` load cases. The pipeline exists; this is a config change and one run.
2. Log driver current in crawl pose against standing pose, to replace an inherited efficiency claim with a measured one.
3. Measure loaded wheel radius under robot weight, for the balance model.
4. Heat-set inserts at the head-to-leg interface.
5. Structural verification against anisotropic printed properties, or an empirical link test to failure, so the structure has a number attached to it rather than a `2×` load case.
6. Chassis redesign for modularity, with a removable battery section, swappable motor mounts and a separable electronics bay. Retrofit first on the existing core.

---

## What Came Next

Electronics and firmware.

The structure exists, the legs track to a hundredth of a degree, the envelope is characterised and the thermal constraint is understood. That was the entire purpose of building it this way. The next post is the CAN bus, the dual-IMU state estimation, and the balance loop, which is where the reflected inertia term stops being a footnote.

---

## References

- Liu, D., F. Yang, X. Liao, and X. Lyu. "DIABLO: A 6-DoF Wheeled Bipedal Robot Composed Entirely of Direct-Drive Joints." arXiv:2407.21500, accepted to IROS 2024. [arXiv](https://arxiv.org/abs/2407.21500). Source for the roller arrangement, the series hip connection, and the baseline control formulation.
- Klemm, V., A. Morra, C. Salzmann, F. Tschopp, K. Bodie, L. Gulich, N. Kung, D. Mannhart, C. Pfister, M. Vierneisel, F. Weber, R. Deuber, and R. Siegwart. "Ascento: A Two-Wheeled Jumping Robot." ICRA 2019, 7515-7521. [arXiv](https://arxiv.org/abs/2005.11435).
- Zhang, J., S. Wang, H. Wang, J. Lai, Z. Bing, Y. Jiang, Y. Zheng, and Z. Zhang. "An Adaptive Approach to Whole-Body Balance Control of Wheel-Bipedal Robot Ollie." IROS 2022, 12835-12842. [IEEE Xplore](https://ieeexplore.ieee.org/abstract/document/9981985).
- Bjelonic, M., V. Klemm, J. Lee, and M. Hutter. "A Survey of Wheeled-Legged Robots." CLAWAR 2022, pp. 83-94. [DOI](https://doi.org/10.1007/978-3-031-15226-9_11).
- Wensing, P. M., A. Wang, S. Seok, D. Otten, J. Lang, and S. Kim. "Proprioceptive actuator design in the MIT cheetah." IEEE Transactions on Robotics, 33(3), 509-522, 2017. [PDF](https://fab.cba.mit.edu/classes/865.18/motion/papers/mit-cheetah-actuator.pdf). Relevant here for the reflected-inertia discussion.

<div class="backbar" markdown>
[:material-arrow-left: Back to Blogs](../blogs.md){ .md-button }
</div>
