---
title: Autonomous Robot Navigation with Deep RL (DDPG / TD3)
tags: [reinforcement-learning, navigation, ddpg, td3, actor-critic, pygame]
---

# Autonomous Robot Navigation using Deep RL

## Info "At a glance"
- **Goal:** Learn a policy for a differential-drive robot to **reach a goal** in a 2D room **without collisions** using continuous control.
- **Env & interface:** Custom **EscapeRoomEnv** (OpenAI Gym-style) with walls/obstacles; state includes pose, velocities, **distance/angle to goal**; actions are **wheel speeds** \[(u_L,u_R)∈[-1,1]\]. 
- **Methods:** **DDPG** baseline + **TD3** (double critics, delayed policy, target smoothing); experience replay + soft targets. 
- **Reward shaping:** step penalty, distance change, heading alignment, **collision penalty**, and **success bonus** with time component. 
- **Results:** DDPG improved navigation but suffered **critic/Q-instability**; **TD3** reduced over-estimation and produced **more stable rewards**, with occasional drops late-training. 
- **Next:** Curriculum, **HER**, refined shaping, and exploring **SAC** for robustness. 

If you’d like the full details, the complete report is embedded below.
---

## Report (PDF)

<div class="pdf-embed">
  <iframe src="../../assets/pdfs/RL_Project.pdf#page=2&view=FitH&toolbar=1&navpanes=0" title="RL Project Report"></iframe>
</div>

### Credits
Course project presented along with Rahul Sha . 

<div class="backbar" markdown>
[← Back to Projects](../projects.md){ .md-button }
</div>
