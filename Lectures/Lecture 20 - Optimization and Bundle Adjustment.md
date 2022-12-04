- Reprojection Error: $$\text{argmin}{\{X_n\}^N_{n=1}, \{R_f,T_f\}^F_{f=1}} \sum_{n,f}d(x_{nf}, K_f[R_f|T_f]X_n)$$
	- $d(...)$ is a distance function and is usually just 2D mean squared error after normalizing homogeneous coordinate to 1. This produces $$\text{argmin}_{P,X}\sum_{n,f}||P_f(X_n)-x_{nf}||^2$$
		- Where $P_f(X_n)$ is the reprojection of point $n$ in frame $f$
		- $x_{nf}$ is the observed point $n$ in frame $f$
- Why can't we do bundle adjustment directly
	- Non-linear minimization: Requires good initializations and is very computationally expensive
- Framing the Problem
	- Set reference frame to be 1st keyframe
		- $R_1 = I$ and $T_1 = \vec{0}$
	- Number of unknowns $$6(F-1)+3N - 1$$
		- 3 DOF for rotation and 3 for translation in each frame $f$
		- $X,Y,Z$ for each point $n$
		- $-1$ due to scale invariance
	- If every point is visible in every frame (the best case scenario) we have $2NF$ equations which are:
		- $$x_p^f = \frac{R_{11}^fX_p + R_{12}^fY_p + R_{13}^fZ_p + T_x}{R_{31}^fX_p + R_{32}^fY_p + R_{33}^fZ_p + T_z}$$
		- $$y_p^f = \frac{R_{21}^fX_p + R_{22}^fY_p + R_{23}^fZ_p + T_y}{R_{31}^fX_p + R_{32}^fY_p + R_{33}^fZ_p + T_z}$$
		- If equations are independent we need $$2NF \geq 6F + 3N - 7$$
			- For 2 frames (SfM), we already know we need 5 correspondences
			- In practical cases, $N$ and $F$ are HUGE - on the order of thousands to hundreds of thousands for each
- Expanding Reprojection Error
	- $$\text{argmin}_{u=\{\{X_n\}^N_{n=1}, \{R_f,T_f\}^F_{f=1}\}}||\epsilon(u)||^2_2$$
	- $$\epsilon^T = (..., x_p^f - \frac{R_{11}^fX_p + R_{12}^fY_p + R_{13}^fZ_p + T_x}{R_{31}^fX_p + R_{32}^fY_p + R_{33}^fZ_p + T_z}, y_p^f - \frac{R_{21}^fX_p + R_{22}^fY_p + R_{23}^fZ_p + T_y}{R_{31}^fX_p + R_{32}^fY_p + R_{33}^fZ_p + T_z}, ...)$$
	- Non-linear least squares problem

___
### Optimization Crash Course
- First Order Optimization: Gradient Descent
	- To minimize a differentiable function $f(x)$ over parameters $x$
	1. Start from some initialization $x$
	2. Compute the gradient $g = \frac{df}{dx}(x)$
	3. Descent along that direction by some step-length $\alpha \rightarrow$ $\delta x = -\alpha g$
	4. Set $x \leftarrow x + \delta x$
- Second Order Optimization: Bird's Eye View
	- Minimize a twice-differentiable function $f(x)$ over parameters $x$
	1. Start with an initial estimate for $x$
	2. Locally approximate $f(x)$ using a second-order Taylor expansion $$f(x+\delta x) \approx f(x) + g^T\delta x + \frac{1}{2}\delta x^TH \delta x$$
		- $g = \frac{df}{dx}(x)$
		- $H = \frac{d^2f}{dx^2}(x)$ (the Hessian)
	3. Find a displacement that locally minimizes the quadratic local approximation of $f(x)$
- Newton/Newton-Raphson's Method
	- Same as above
	- Assume Hessian is positive, semi-definite (if not i.e. from poor initialization, quadratic function could be concave down and have an unbounded minimum). Then can find the minimum with $$\frac{df}{dx}(x+\delta x) \approx H \delta x + g = 0$$
	 $$\delta x = -H^{-1}g$$
	 - Iterating over the above update is called Newton's method
- Gauss-Newton Method for Least Squares
	- $g = \nabla_uf = 2\sum_i \epsilon_i(u)\nabla_u\epsilon_i(u) = 2J(u)^T \epsilon(u)$
		- $J_{ij} = \frac{\partial \epsilon_i}{\partial u_j}$
	- $H(u) = 2\sum_i \nabla_u \epsilon_i(u) \nabla_u \epsilon_i(u) + \epsilon_i(u) \frac{\partial^2\epsilon_i}{\partial^2u^2} \therefore$
	- $H(u) = 2J(u)^TJ(u) + 2\sum_i\epsilon_i(u)\frac{\partial^2\epsilon_i}{\partial u^2}$
	- We will just ignore the hard-to-compute quadratic terms so $$H(u) \approx 2J(u)^TJ(u)$$
		- Holds true when error $\epsilon_i$ is small or the function is approximately linear
	- Applying above to Newton's update $\delta u = -H^{-1}g \rightarrow$ Gauss-Newton update/Normal equation $$\delta u = -(J(u)^TJ(u))^{-1}J(u)^T \epsilon(u)$$
		- Basically pseudoinverse of jacobian with error! ($||Ju - \epsilon||^2_2$)
### Applying to Bundle Adjustment
- Problems right out the gate
	- $J_{i,j} = \frac{\partial \epsilon_i}{\partial u_j}$
	- Dimension of $\epsilon$ is $2NF$
	- Dimension of unknown parameters $u$ is $M=6F+3N-7$
	- Size of $J$ is $2NF$! Computing the inverse of that ($O(N^3)$ operation) is impossible at scale
- Workaround: Linear algebra implementation tricks
	- Block diagonal matrices can be inverted block by block
	- Try to set up $Ax=b$ with a triangular matrix $A$ (can solve element by element)
	- Avoid matrix multiplications of large general matrices, sparse are much better