---
title: "002 - VAD: Vectorized Scene Representation for Efficient Autonomous Driving"
created: 2024-03-26
tags:
  - Paper Reading
  - Imitation Learning
  - End-to-End planning
---

!!! note "Paper Card"

    * Title: VAD: Vectorized Scene Representation for Efficient Autonomous Driving
	* First author: Bo Jiang (HUST)
	* Correspoding author: Xinggang Wang (HUST)
	* Venue: ICCV 2023
	* Open source: https://github.com/hustvl/VAD
	* Key Words: Imitation Learning, End-to-End planning


## Model
<figure markdown="span">
  ![VAD architecture](Pasted%20image%2020240408113259.png){ width="800" }
  <figcaption>VAD architecture.</figcaption>
</figure>

### Map Representation
Initialize some map queries, and then predict vectorized map elements in a manner similar to MapTR. The predicted vector representations are $V_m [M, P, 2]$, which means M elements, each element has P sampling points, and 2 corresponds to x, y coordinates. In VAD, only three types of map elements are considered: lane divider (boundary), road boundary, pedestrian crossing (crosswalk). I also believe these three are the most useful map elements in imitation/prediction.

### Agent Representation
Initialize some agent queries, and then use deformable attention for sparse interactions on the BEV feature. Then use typical estimated prediction interactions, namely a2a, a2m, to predict the future state of the agent, including location, class score, orientation, etc. The predicted vector representations are $V_a [A, K, F, 2]$, where A is the number of agents, K is the number of modalities, and F is the number of future time steps.

### Ego Representation and Planning
Initialize an ego query, interact it with the agent queries first, and then with the map queries. Note that the interactions here are with the implicit query features of the previous module, not the explicit states of the agent and map decoded. However, the explicit state decoded by the previous module also plays a role in the form of positional encoding:

<figure markdown="span">
  ![Ego representation](Pasted%20image%2020240408114632.png){ width="600" }
</figure>

After the ego query completes the interactions, it is inputted into a PlanHead along with a high-level command (similar to intention, including three types: turn left, turn right, go straight), and finally outputs the vectorized planning trajectory $V_{ego}$.

### Planning Constraint
Thanks to the explicit vector representations constructed by VAD, these dynamic obstacles and map elements can be directly used to further constrain the planning trajectory of the ego vehicle. Moreover, this constraint can be differentiable, thus not breaking the end-to-end training process. These constraints are divided into the following three types: Ego-Agent Collision Constraint, Ego-Boundary Overstepping Constraint, Ego-Lane Directional Constraint.

<figure markdown="span">
  ![Constraint](Pasted%20image%2020240408114639.png){ width="800" }
</figure>

### Ego-Agent Collision Constraint
First, filter out agent prediction results with low confidence. Then, for each agent, select the modality with the highest probability for loss calculation. For each future time step, filter the nearest neighbor agent through different thresholds in the lateral and longitudinal directions, and then calculate the following L1 distance loss in x, y directions:

<figure markdown="span">
  ![Collision](Pasted%20image%2020240408114648.png){ width="400" }
</figure>

### Ego-Boundary Overstepping Constraint
Similar to the above process, first filter out map element prediction results with low confidence, then calculate the distance between the trajectory point and the nearest road boundary for each future time step.

<figure markdown="span">
  ![Overstepping](Pasted%20image%2020240408114656.png){ width="400" }
</figure>

### Ego-Lane Directional Constraint
Similar to the above process, first filter out map element prediction results with low confidence, then calculate the difference between the ego's heading and the nearest lane divider's heading for each future time step.

<figure markdown="span">
  ![Directional](Pasted%20image%2020240408114712.png){ width="400" }
</figure>

### Loss
The final loss consists of 3 main parts, perception + constraint + Imitation

<figure markdown="span">
  ![Loss](Pasted%20image%2020240408114721.png){ width="500" }
</figure>

* $L_{map}$: Vectorized map element loss, sampling point regression + instance classification
* $L_{mot}:$ Trajectory prediction loss, trajectory regression + modality classification
* $L_{col}:$ Collision constraint loss, stepwise L1 in x, y directions
* $L_{bd}:$ Boundary constraint loss, isotropic stepwise L1
* $L_{dir}:$ Directional constraint loss, L1 of yaw angle
* $L_{imi}:$ Ego trajectory regression loss, L1

## Experiments
* Training: 2s of historical information (4 frames), predict future 3s trajectory (6 frames). Longitudinal perception range is 60m, lateral perception range is 30m.
* VAD-Base: 200 x 200 (0.3m x 0.15m resolution)
* VAD-Tiny: 100 x 100 (0.6m x 0.3m resolution)

### Open Loop
Open loop training on 8x3090, batch_size_per_gpu=1, epoch=60

<figure markdown="span">
  ![Open loop exp](Pasted%20image%2020240408114732.png){ width="500" }
</figure>
<figure markdown="span">
  ![Open loop viz](Pasted%20image%2020240408114739.png){ width="500" }
</figure>

#### Ablation
The interaction with map elements has the biggest impact on ADE (0.72 -> 0.86) and ACR (0.22 -> 0.29), followed by the interaction with agent elements.
After adding constraint-based loss, the average collision rate decreased by 6%.

### Closed Loop
Closed-loop simulator: CARLA simulator
<figure markdown="span">
  ![CARLA exp](Pasted%20image%2020240408114825.png){ width="500" }
</figure>

* DS: CARLA driving score
* RC: Route completion rate

## Thoughts
* VAD explicitly models agents and maps in vectorized form, but the ego does not interact directly with BEV features, which does not fully utilize information and does not well demonstrate the advantages of E2E.
* Explicit vectorized modeling of agents and maps provides great convenience for adding interpretable constraints, which can be a key point of reference.
* How to balance various constraints and multi-task loss is still a challenging problem.