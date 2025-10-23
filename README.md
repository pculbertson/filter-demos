# CS 4750 Filter Demonstrations
Some filter demonstration video recordings for CS 4750.

## Problem setup
This demo shows how different state estimators we discussed in class perform for a difficult drone localization problem. We model the drone point mass in 2D, flying at a constant height, which gets height measurements from a noisy altimeter.

The drone's state is its position and velocity in 2D, $\mathbf{x} = [p_x, p_y, v_x, v_y]^T$. Its dynamics are a discrete-time version of $\mathbf{f} = m\mathbf{a}$ with a time step of $\Delta t$,
$$\mathbf{x}_{k+1} = \begin{bmatrix} 1 & 0 & \Delta t & 0 \\ 0 & 1 & 0 & \Delta t \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \mathbf{x}_k + \begin{bmatrix} 0 & 0 \\ 0 & 0 \\ 1 & 0 \\ 0 & 1 \end{bmatrix} \mathbf{u}_k + \epsilon_k,$$
where $\epsilon_k \sim \mathcal{N}(\mathbf{0}, \mathbf{Q})$ is process noise with covariance $\mathbf{Q} = 0.01 * \mathbf{I}$.

At each point $\mathbf{p} = [p_x, p_y]^T$, the drone receives a noisy height measurement from an altimeter:
$$z_k = h(\mathbf{p}_k) + \delta_k,$$
where $h$ is a nonlinear terrain function defined with sines and cosines, along with a flat section in to the southeast. The noise has a variance $Q = 0.05$. The terrain function is shown in the screenshot below.

This problem is highly ambiguous - many different positions correspond to the same height measurement, especially in the flat region of the terrain. 

[![Terrain function](terrain.png)](terrain.png)

The drone applies a control input $\mathbf{u}_k = -K_p * (\bar{\mathbf{p}}_k - \mathbf{p}_{des})$ tries to fly toward a desired position $\mathbf{p}_{des}$, where $\bar{\mathbf{p}}_k$ is the estimated position at time $k$ and $K_p = 0.2$ is a proportional gain.

### Visualization details
Each of the visualizations below will show an animation of the drone flying and show samples from the estimator's belief about the drone's position as a point cloud. The color of each point corresponds to how likely the current measurement is given that position (i.e., the likelihood of the measurement under the altimeter model at that position). The desired position is shown by a coordinate frame.

## Demonstrations - Extended Kalman Filter (EKF)
### Nominal case
At [this link](https://pculbertson.github.io/filter-demos/viser-client/?playbackPath=https://pculbertson.github.io/filter-demos/recordings/ekf_nominal.viser) you can find a 3D view of the EKF when the desired position is in a "bumpy" (identifiable) region of the terrain. We notice that the EKF can "hang on" for a few seconds, but quickly diverges - this is because the terrain is highly curved, so the linearizations used to update the belief are very inaccurate.

### Ambiguous case
At [this link](https://pculbertson.github.io/filter-demos/viser-client/?playbackPath=https://pculbertson.github.io/filter-demos/recordings/ekf_flat.viser) you can find a 3D view of the EKF when the goal is in the flat region of the terrain. Here, the EKF covariance quickly spreads out and diverges. This is because the linearization of the measurement model in this region provides no information about the drone's position.

## Demonstrations - Particle Filter (PF)
### Nominal case
At [this link](https://pculbertson.github.io/filter-demos/viser-client/?playbackPath=https://pculbertson.github.io/filter-demos/recordings/pf_nominal.viser) you can find a 3D view of the Particle Filter when the desired position is in a "bumpy" (identifiable) region of the terrain. The particle filter uses 10k particles.

We notice that the Particle Filter is able to maintain a highly multimodal belief about the drone's position, and is often able to recover from "bad" beliefs. Notice how the particles spread around "rings" of possible positions that correspond to the same height measurement.

### Ambiguous case
At [this link](https://pculbertson.github.io/filter-demos/viser-client/?playbackPath=https://pculbertson.github.io/filter-demos/recordings/pf_flat.viser) you can find a 3D view of the Particle Filter when the goal is in the flat region of the terrain. Here, the Particle Filter is still able to maintain a good belief about the drone's position, although the uncertainty grows larger (becoming almost uniform) in the flat region. Whenever the drone leaves the flat region, the filter is able to quickly re-localize, as the terrain outside this region is highly informative.

## Discussion
This example is adversarial for the EKF in two ways: the terrain is highly nonlinear, and the measurement model is highly ambiguous. The EKF's linearizations are not able to capture the true posterior distribution over the drone's position, leading to filter divergence. In contrast, the Particle Filter is able to maintain a highly multimodal belief about the drone's position, allowing it to successfully localize the drone even in this difficult scenario.

## Implementation details

All filters are implemented in [jax](https://github.com/google/jax), a package for high-performance numerical computing in Python. All Jacobians can be computed using the automatic differentiation features of jax, and particle filter updates can be efficiently vectorized over all particles. Visualizations are created using [viser](viser.studio), an excellent package for web-based 3D visualization.