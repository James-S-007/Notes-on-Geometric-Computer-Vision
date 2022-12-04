### 3D Motion from Optical Flow
$$\dot{p} = \frac{1}{Z} \begin{bmatrix} xV_z - V_x \\ yV_z - V_y\end{bmatrix} + \begin{bmatrix} xy & -(1+x^2) & y \\ (1+y^2) & -xy & -x \end{bmatrix} \Omega$$

#### Translational Flow
$$\dot{p} = \begin{bmatrix} \dot{x} \\ \dot{y} \end{bmatrix} = \frac{1}{Z} \begin{bmatrix} xV_z - V_x \\ yV_z - V_y\end{bmatrix} = \frac{V_z}{Z} \begin{bmatrix} x - \frac{V_x}{V_z} \\ y - \frac{V_y}{V_z} \end{bmatrix}$$
- $\begin{bmatrix} \frac{V_x}{V_z} \\ \frac{V_y}{V_z} \end{bmatrix}$ is the **Focus of Expansion** (FOE)
	- Change in x or y coordinate is proportional to how far that coordinate is from the focus of expansion
	- FOE is the same as the epipole! (Image of the second camera from the first camera's frame)
		- If camera center moves from $(0,0,0)$ to $(V_x,V_y,V_z)$ at $t = 1$, the image of the $t = 1$ camera at $t = 0$ is $\frac{V_x}{V_z}, \frac{V_y}{V_z}$
		- FOE does not depend on the scene, just the motion
		- Can also be thought of as the intersection of the flow vectors (same as all epipolar lines connecting at epipole)
- $\frac{V_z}{Z}$ is the **Inverse Time-To-Collision**
	- Change in x or y coordinate is inversely proportional to the time to collision of the camera with the $Z$ plane of thfe object in the camera coordinate system

#### Rotational Flow
$$\dot{p} = \begin{bmatrix} xy & -(1+x^2) & y \\ (1+y^2) & -xy & -x \end{bmatrix} \Omega$$
- If we know angular velocity $\Omega$ (usually from IMU) can...
	- Compute optical flow $\dot{p}$ from the images with LK
	- Estimate rotational flow at each pixel $\dot{p}_{\text{rot}}$
	- Get translational flow $\dot{p}_{\text{trans}} = \dot{p} - \dot{p}_{\text{rot}}$

### Finding FOE and TTC
- Finding FOE  $\sim$ $V$ up to scale ambiguity
	- The FOE can be written as $V \sim [V_x,V_y,V_z]$
	- For a point with known translational flow, its flow line is $$p_1 \times (p_1 + \dot{p}_1) = p_1 \times \dot{p}_1$$
		- Recall: Equation of a line in projective space is $l = p_1 \times p_2$
		- Reduction comes from $p_1 \times p_1 = 0$
	- FOE is the intersection of all flow lines so $$(p_1 \times\dot{p}_1)^TV = 0$$
	- Given $n \geq 2$ points and flows $$\begin{pmatrix} (p_1 \times \dot{p_1})^T \\
	(p_2 \times \dot{p_2})^T \\
	...\\
	(p_n \times \dot{p_n})^T \\
	\end{pmatrix}V = 0$$
		- Solved equations like this many times before, take SVD, last right singular vector of $V$
		- NOTE: SVD is for solving the (left or right) null space of an over or underdetermined system $Ax = \vec{0}$ (or $xA = \vec{0}$ for left null space respectively) . Least squares/pseudo-inverse is for overdetermined systems $Ax = \vec{b}$
- Finding Time-To-Collision (TTC)
	- After computing FOE $$\text{Inverse TTC} = \frac{V_z}{Z} = \frac{||\dot{p}_{\text{trans}}||}{||p - FOE||}$$

### 3D Motion from Depth Cameras (Known $Z$)
- If $Z$ is known, $\dot{p}$ is linear in $V$ and $\Omega$
- We have 2 equations per point correspondence (the flow)
- If we have at least 3 optical flow vectors not on collinear points and their corresponding depths we can solve $$A_{2n \times 6} \begin{bmatrix} V_{3 \times 1} \\ \Omega_{3 \times 1} \end{bmatrix} = \dot{p}$$
	- Typically do RANSAC over many sets of 3 flow vectors since $\dot{p}$ can be very noisy
___
### Summary of Methods for Motion from Optical Flow
- Given optical flow and angular velocity $\Omega$, can solve for $FOE \sim V$ with 2 2D-2D correspondences
- Given optical flow and depths $Z$, we can solve for $V$ and $\Omega$ with 3 2D-2D correspondences
- Alone, we can solve for $V$ and $\Omega$ with 5 correspondences (SfM)
- With a known 3D scene, we can solve for single-frame camera pose with 3 2D-3D correspondences (PnP)