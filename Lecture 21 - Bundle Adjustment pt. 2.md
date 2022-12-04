### Recap: Minimization Problem
$$\text{argmin}_{u=\{\{X_n\}^N_{n=1}, \{R_f,T_f\}^F_{f=1}\}}||\epsilon(u)||^2_2
$$
$$\epsilon^T = (..., x_p^f - \frac{R_{11}^fX_p + R_{12}^fY_p + R_{13}^fZ_p + T_x}{R_{31}^fX_p + R_{32}^fY_p + R_{33}^fZ_p + T_z}, y_p^f - \frac{R_{21}^fX_p + R_{22}^fY_p + R_{23}^fZ_p + T_y}{R_{31}^fX_p + R_{32}^fY_p + R_{33}^fZ_p + T_z}, ...)$$

Use Newton's method  for least squares can be implemented as the Gauss-Newton method $$\delta u = -(J^TJ)^{-1}J^T\epsilon$$
### Creating Structure
- Separate the parameters $u$ into
	- Structure parameters $a_{3n \times 1}$
	- Camera parameters $b_{6f \times 1}$
	- So $u = \begin{bmatrix} a \\ b \end{bmatrix}$
- Structure of the Jacobian
	- Recall $J_{ij} = \frac{\partial \epsilon_i \text{(errors)}}{\partial u_j \text{(parameters)}}$
	- The Jacobian looks like the following
		- ![[Screen Shot 2022-11-20 at 2.04.52 PM.png]] where
		- $J_a$ is in yellow
			- $A1,A$ corresponds to the reprojection error of pt $A$ in camera $1$
			- All entries are zero except the reprojection of a pt visible in a camera view to that camera's view
			- NOTE: These are block matrices, each pt $A,B,...$ has 3 DOF and camera $1,2,...$ has 6
			- NOTE: If every point were visible in every camera, there would be no non-zero rows (zero-rows omitted in image above)
		- $J_b$ is in green
			- $A_1,1$ corresponds to ???
- $J^TJ$
	- $J^TJ$ can be written as a block matrix of the components defined above $$J^TJ = \begin{bmatrix} J^T_aJ_a & J_a^TJ_b \\ J^T_bJ_a & J^T_bJ_b \end{bmatrix} = \begin{bmatrix} U & W \\ W^T & V\end{bmatrix}$$
	- ![[Screen Shot 2022-11-20 at 2.11.40 PM.png]]
	- $J^TJ_a$ and $J^TJ_b$ are block diagonal matrices
	- $J_a^TJ_b = (J_b^TJ_a)^T$
		- This matrix would be full if every point were visible in every camera
	- Adding this into our least squares update equation, we get $$\begin{bmatrix} \delta a \\ \delta b \end{bmatrix} = -\begin{bmatrix} J^T_aJ_a & J_a^TJ_b \\ J^T_bJ_a & J^T_bJ_b \end{bmatrix}^{-1}J^T \epsilon$$
	- $$\begin{bmatrix} U & W \\ W^T & V \end{bmatrix} \begin{bmatrix} \delta a \\ \delta b \end{bmatrix} = -J^T \epsilon$$
- Continuing to exploit structure
	- We can multiple by a carefully chosen matrix to make the problem easier
	- $$\begin{bmatrix} U & W \\ W^T & V \end{bmatrix} \begin{bmatrix} \delta a \\ \delta b \end{bmatrix} = -J^T \epsilon = \begin{bmatrix} \epsilon_a' \\ \epsilon_b' \end{bmatrix}$$
	- $$\begin{bmatrix} I & -WV^{-1} \\ 0 & I \end{bmatrix} \begin{bmatrix} U & W \\ W^T & V \end{bmatrix} \begin{bmatrix} \delta a \\ \delta b \end{bmatrix} = \begin{bmatrix} I & -WV^{-1} \\ 0 & I \end{bmatrix} \begin{bmatrix} \epsilon_a' \\ \epsilon_b' \end{bmatrix}$$
	- $$\begin{bmatrix} U - WV^{-1}W^T & 0 \\ W^T & V \end{bmatrix} \begin{bmatrix} \delta a \\ \delta b \end{bmatrix} = \begin{bmatrix} \epsilon_a' - WV^{-1}\epsilon_b' \\ \epsilon_b' \end{bmatrix}$$
	- This allows us to first solve for $\delta a$ by $$(U-WV^{-1}W^T) \delta a = \epsilon_a' - WV^{-1} \epsilon_b'$$
	- And then solve for $\delta b$ with our calculation of $\delta a$ $$V \delta b = \epsilon_b' - W^T \delta a$$
	- The only inverse we have to calculate is $V^{-1}$ (remember $V$ is block diagonal so computing its inverse is relatively trivial)
___
### Multi-View Stereo
- Goal: Produce dense 3D reconstruction of the scene (not just a sparse set of points in it)
- Given:
	- Images of the same scene from multiple cameras
	- $K,R,T$ for all cameras
- Frontoparallel Cameras
	- ![[Screen Shot 2022-11-20 at 2.27.16 PM.png]]
	- $(u_l,v_l) = (f\frac{X}{Z}, f\frac{Y}{Z})$
	- $(u_r,v_r) = (f\frac{X-B}{Z}, f\frac{Y}{Z})$
	- Disparity $d$: Difference in projections between the two cameras
		- $d = u_l - u_r = f\frac{B}{Z}$
		- $Z = f\frac{B}{d}$
	- We can use this to find correspondences! Even when the pixel movements are large (does not permit optical flow due to first order taylor expansion)
		- Correspondences lie on the same horizontal line (epipole is horizontal point at $\infty$, point correspondence must lie on epipolar line)