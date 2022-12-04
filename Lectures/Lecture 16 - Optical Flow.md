- What Methods for Each Type of Correspondence?

|  Correspondence | Method |
| ---- | ---- |
| 3D -> 3D | Procrustes to solve $R,T$ | 
| 3D -> 2D | PnP/P3P |
| 2D -> 2D | SfM/Image Stitching |


### Optical Flow
- The pattern of apparent motion of objects, surfaces, and edges in a visual scene caused by the relative motion between an observer and a scene
- Problem:
	- Given two images $I_t(x,y)$ and $I_{t+1}(x,y)$, can we compute the "optical flow" to tell us which pixel in the second image matches with the first
	- Solution: If video frames are close enough, we can look in the neighborhood of $I_t(x,y) \approx I_{t+1}(x+\delta x, y + \delta y)$
- Local Search Objective Function
	- Parameters:
		- $(x_0, y_0)$ Pixel we are computing the optical flow from
		- $N$ Neighborhood patch around a pixel
		- $(\delta x, \delta y)$ Offset we are optimizing for - the optical flow
	- We want to optimize (in discrete form) $$\text{argmin}_{\delta x, \delta y} \sum_{(x,y) \in N(x_0,y_0)}(I_t(x,y) - I_{t+1}(x+\delta x, y + \delta y))^2$$
	- To make this tractable, we can make the assumption that the change is small and use a first order taylor expansion: $$I_t(x,y) - I_{t+1}(x + \delta x, y + \delta y) \approx \Delta I_t(x,y) - \nabla I_{t+1}(x,y)^T \begin{pmatrix} \delta x \\ \delta y \end{pmatrix} $$
	- Difference $\Delta I_t(x,y)$: $I_t(x_i,y_i) - I_{t+1}(x_i,y_i)$ for each respective pixel
	- Spatial Gradient $\nabla I_{t+1}(x,y)^T$: Compute through convolution in x and y direction
- Lukas-Kanade Optical Flow
	- Solution to above overdetermined linear systems of equations
		- Why overdetermined: Cannot use patch size of 1, would result in 1 equation with 2 unknowns
	- $$\begin{bmatrix}
		\nabla_x I|_{p_0} & \nabla_y I|_{p_0} \\
		... & ... \\
		\nabla_x I|_{p_i} & \nabla_y I|_{p_i} \\
		... & ... \\
		\nabla_x I|_{p_n} & \nabla_y I|_{p_n}
	\end{bmatrix}
	\begin{bmatrix}
		\delta x \\ \delta y
	\end{bmatrix} = 
	\begin{bmatrix}
		\Delta I|_{p_0}\\
		...\\
		\Delta I|_{p_i}\\
		...\\
		\Delta I|_{p_n}
	\end{bmatrix}
	$$
	- Solve with pseudo-inverse $Ax = b \rightarrow x = (A^TA)^{-1}A^Tb$
- Aside: Horn-Schunck Optical Flow
	- Instead of assuming shared flow over a local pixel neighborhood, Horn-Schunck assumes smoothness over the global flow field rather than local consistency
	- No closed-form solution, use iterative gradient descent updates
- Assumptions We've Made
	- Brightness Constancy: Color doesn't change between different viewpoints
	- No Occlusions: Things stay in sight
	- Small Motions: Minimal patch displacement (because of Taylor expansion)
	- Locally Uniform Motion: Constancy of pixel motion within a small neighborhood (ex: rotation/scaling breaks this assumption)
	- **Invertibility of $(A^TA)^{-1}$** 