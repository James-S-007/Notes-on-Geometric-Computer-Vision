- Aside: Orthographic Projections
	- Used in engineering drawings
	- Are images formed by rays orthogonal to the image plane, therefore lengths are preserved
- World to Camera Frame Euclidean Transformation
	- What do $R$ and $t$ mean in $\lambda \begin{bmatrix} u \\ v \\ 1  \end{bmatrix} = K \begin{bmatrix} R | t \end{bmatrix} X_w$
		- $R$ is the rotation of the world axes with respect to the camera axis: $R^c_w$
		- $T$ is performed *after* the perspective projection, so if $a$ is the direct translation from the camera origin to the world origin, then $T = Ra$
- Pose from 3 Point Correspondences (P3P)
	- Given 3 known world 3D coordinates $P^w_i \in \mathbb{R}^3$ and it's camera-centric 3D coordinates $P^c_i \in \mathbb{R}^3$...
	- The P3P problem is finding camera pose $R,T$ such that $$P^c_i = RP^w_i + T$$
	1. Convert pixels to calibrated coordinates: $p_i \sim K^{-1}  \begin{pmatrix} u_i & v_i & 1 \end{pmatrix}^T$
		- Essentially image plane coordinates with principal point as origin and focal length of 1
		- Euclidean interpretation: $p_i$ is the vector in camera coordinates pointing from the camera center in the direction of the 3D point we are imaging
		- Now just need to find depths to get camera-centric 3D coordinates
	2. Find $\lambda_i, R, T$ such that $$\lambda_ip_i = RP_i + T$$
		- $\lambda_i$'s are not guaranteed depths unless $p_i$'s are unit vectors
		- Therefore the depths are: $\frac{d_i}{||p_i||_2}p_i = RP_i + T$
	3. Use law of cosines to get a quadratic equation for each point $$d_i^2 + d_j^2 - 2d_id_j cos\delta_{ij} = d^2_{ij}$$
		- Law of Cosines: $c^2 = a^2 + b^2 - 2ab \cdot cos(C)$ for any triangle
		- ![[Screen Shot 2022-10-21 at 11.07.38 AM.png]]
	4. Now insane algebra (not all pictured here)
		- ![[Screen Shot 2022-10-21 at 11.09.45 AM.png]]
		- This will give us multiple valid solutions (Roots of a 4th degree polynomial)
		- To find best:
			- Only check solutions with positive, real roots
			- Use Kabsch algorithm to solve Procrustes problem for each valid solution and compare computed $R,T$ against 4th known world point (if known)