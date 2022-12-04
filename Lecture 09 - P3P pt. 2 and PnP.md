- Continuing P3P Problem: How do we solve for $R,T$ from frame $B$ to $A$, given $n$ point correspondences with known depths
	- $\frac{d_i}{||p_i||_2} = RP_i + T \rightarrow A_i = RB_i + T$
	- Becomes a minimization problem: $min_{R,T} \sum_i^N ||A_i - RB_i - T||^2$
	- Differentiate with respect to $T$ and we find: $T = \frac{1}{N} \sum_i^NA_i - R\frac{1}{n} \sum_i^NB_i = \bar{A} - R\bar{B}$
		- $\bar{A}$ and $\bar{B}$ are the centroids of matrix $A$ and $B$ respectively
		- This gives us 2 important observations
			- If we find the correct rotation matrix $R$, we can solve for $T$ afterwards
			- If the centroids are both 0, we can set $T=0$
	- Rotation matrix is unaffected by translating the point sets, so we can subtract the centroids from the point sets before finding the rotation
	- We now have $min_R ||\hat{A} - R\hat{B}||^2_F$
		- $\hat{A} = \begin{pmatrix} A_1 - \bar{A} & ... & A_N - \bar{A} \end{pmatrix}$
		- $\hat{B} = \begin{pmatrix} B_1 - \bar{B} & ... & B_N - \bar{B}\end{pmatrix}$
	- We can now reuse our solution to the Procrustes problem by slightly rewriting the minimization problem: $$argmin_{R \in SO(3)} ||\hat{A} - R\hat{B}||^2_F = argmin_{R \in SO(3)}||R - \hat{A}\hat{B}^T||^2$$
- Summary of P3P Step 2 (Already solved for depths)
	1. Compute centroids $\bar{A}$ and $\bar{B}$ of the 2 sets of 3D points
	2. Create matrices $\hat{A}_{3 \times n}$ and $\hat{B}_{3 \times n}$ after subtracting $\bar{A}$ and $\bar{B}$ from all points in the 2 sets (Note that each point is a column in $A$ and $B$ as opposed to the usual row)
	3. To find $R$ solve $argmin_{R \in SO(3)} ||\hat{A} - R\hat{B}||_F$
		1. Set $\hat{R} = (\hat{A}\hat{B}^T)_{3x3}$
		2. Decompose via SVD: $\hat{R} = U \Sigma V^T$
		3. Set $R = U \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & det(VU^T) \end{pmatrix} V^T$
		4. Set $T = \bar{A} - R \bar{B}$
	4. NOTE: There will most likely be multiple solutions due to the roots of the 4th degree polynomial when solving for depths. If a 4th point correspondence is known, the solved $R$ and $T$'s should be compared with it to select the most accurate solution
- PnP Problem
	- Gives unique solution by having at least 6 point correspondences and don't have to do ridiculous algebra like in P3P
	- Steps
		1. Convert to calibrated coordinates: $\begin{pmatrix} x_i \\ y_i \\ 1 \end{pmatrix} \sim K^{-1} \begin{pmatrix} u_i \\ v_i \\ 1 \end{pmatrix}$
		2. Need to solve: $\lambda_i \begin{pmatrix} x_i \\ y_i \\ 1 \end{pmatrix} = R \begin{pmatrix} X_i \\ Y_i \\ Z_i \end{pmatrix} + T$
			- Rewrite equation by dividing by lambda for $x_i$ and $y_i$ equations to make them linear in the elements of $R$ and $T$
			- Produces $A_{2n \times 12}x_{12 \times 1} = 0$ so need at least 11 equations or 6 point correspondences
		3. Kabsch Algorithm for Procrustes Again
			1. Solve $A_{2n \times 12}x_{12 \times 1} = 0$ from $n \geq 6$ correspondences
			2. Assemble rotation matrix $\hat{R}$ from the solution for $x$
			3. $\hat{R} = U \Sigma V^T$, set determinant to 1 by doing same diagonal matrix computation
			4. Adjust translation to be consistent with new $R$ by resolving $A_{2n \times 12}x_{12 \times 1} = 0$ for only $t_1, t_2, t_3$ holding $R$ fixed
- Calibrating Cameras Under more General Intrinsics
	- If pixels are not square, can have different focal lengths for $x$ and $y$ $$\begin{bmatrix} f & & u_0 \\ & f & v_0 \\ & & 1 \end{bmatrix} \rightarrow \begin{bmatrix} s_x & & u_0 \\ & s_y & v_0 \\ & & 1 \end{bmatrix}$$
	- If radial distortions are present, $u_0, v_0$ are no longer constants and are now functions of distance $r$ from camera center $$\begin{bmatrix} f & & u_0 \\ & f & v_0 \\ & & 1 \end{bmatrix} \rightarrow \begin{bmatrix} f & & u(r) \\ & f & v(r) \\ & & 1 \end{bmatrix}$$
		- Often parameterized as
			- $u(r) = u(1 + k_1r^2 + k_2r^4 + k_3r^6 + ...)$
			- $v(r) = v(1 + k_1r^2 + k_2r^4 + k_3r^6 + ...)$
			- Greater degree results in greater accuracy but requires more images to fit and is more computationally expensive