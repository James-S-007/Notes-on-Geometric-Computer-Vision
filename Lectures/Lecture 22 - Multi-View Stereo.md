### Recap
- With fronttoparallel cameras, can triangulate with...
	- $Z = f \frac{B}{d}$
	- $B$ is the baseline between the two cameras
	- $d$ is the disparity between the point correspondences ($u_l - u_r$)
- Point correspondences for fronttoparallel cameras displaced along the x-axis lie on the same row
### Point Correspondences
- Setting Search Range of Disparity
	- Assume some minimum "depth", $Z_{\text{min}}$
	- Set $d_{\text{max}} = \frac{1}{Z_{\text{min}}}$
	- Quantize the interval $[-d_\text{max}, d_\text{max}]$
- Components of Stereo Correspondence Matching
	- **Matching Criterion** (error function)
		- Quantify similarity of a pixel pair
		- Options
			- Direct RGB intensity difference
			- Correlation
	- **Aggregation Method**
		- How the error function is accumulated
		- Options
			- Pixelwise
			- Edgewise
			- Window-wise
	- **Optimization and Winner Selection**
		- How the final correspondences are determined
		- Options
			- Winner-take-all
			- Dynamic Programming
			- Graph Cuts
- A Simple Process
	- Matching Windows: For a given left window, we will select from various right windows along the scanline as candidate correspondences
	- Matching Windows with SSD Over the Window
		- $w_L, w_R$ are corresponding $m \times m$ windows of pixels
			- Define the window function $$W_m(x,y) = \{u,v | x - \frac{m}{2} \leq u \leq x + \frac{m}{2}, y - \frac{m}{2} \leq v \leq y + \frac{m}{2} \}$$
			- SSD Cost is then $$C_r(x,y,d) = \sum_{(u,v) \in W_m(x,y)}[I_L(u,v) - I_R(u-d,v)]^2$$
	- Matching Windows with SSD + Winner-Take-All
		- Assign lowest SSD as match for each window
___
### Stereo Rectification
- What to do if cameras aren't frontoparallel to start with $\rightarrow$ "rectify" them!
- Works best when cameras are roughly aligned
- Key Idea: Reproject image planes onto a common plane parallel to the line between camera centers. We need two homographies (which produce a single homography)
- Simplified Process
	1. Estimate $E$ using 8-point algorithm
	2. Decompose $E$ into $R$ and $T \approx e$
	3. Build $R_\text{rect}$ from $e$
	4. Set $R_1 = R_\text{rect}$ and $R_2 = RR_\text{rect}$
	5. On left and right camera pixel planes, apply the homographies $KR_1$ and $KR_2$ respectively
- Calculating $R_\text{rect}$
	- Done by setting the epipole to $\infty$
	- Can use the idea of $r_1 = e_1 = \frac{T}{||T||}$ and $r_2, r_3$ are orthogonal to produce the homography
	- Proof
		- Let $$R_\text{rect} = \begin{bmatrix} r_1^T \\ r_2^T \\ r_3^T\end{bmatrix}$$
		- $r_1 = e_1 = \frac{T}{||T||}$
		- $r_2 = - \frac{1}{\sqrt{T_x^2 + T_y^2}}[-T_y, T_x, 0]$
		- $r_3 = r_1 \times r_2$
		- This satisfies the property that the homography of the epipole is the point at positive x infinity and minimizes the necessary rotation (there are infinite solutions for this)
### Multi-View Stereo
- Problem Formulation: Given several images of the same object or scene, compute a representation of its 3D shape
	- Input: Calibrated images from several viewpoints with known intrinsics, extrinsics, and projection matrices
	- Output: 3D object model
- Exploiting Multiple Neighbors
	- Can match windows using more than 1 neighboring view to get a stronger match signal
	- Possible Options for Picking Neighbors
		- Pick neighbors based on baseline
			- Problem: Too small of a baseline results in large depth error and too large of a baseline is a difficult search problem
		- Best: Combine All
			- Add all graphs for pixel matching score vs depth and pick the minimum
			- ![[Screen Shot 2022-11-27 at 5.22.16 PM.png]]
- Plane-Sweep Stereo
	- An efficient way to compute multi-view stereo
	- Algorithm
		- Sweep family of planes parallel to reference camera image plane
		- Reproject neighbors onto each plane (via homography) and compare reprojections
		- At each iteration, we pretend that each camera's entire image was of a single plane at depth $z$ from the reference camera and backproject onto the plane from each camera and see how much the neighbors agree for each pixel
		- When neighbors agree at a pixel, that pixel is likely to have depth $z_0$. The pixel's cost for depth $z$ is the variance over the neighbor backprojections
		- $z$ is then incremented and the next iteration begins
		- At the end of the "sweep" over $z$, the min-cost $z$ is selected for each pixel, to form the full depth map
	- How to backproject onto plane $Z = z_0$
		- At each iteration, pretend that each camera's entire image was of a single plane at depth $z_0$ from the reference camera, and backproject onto that plane from each camera
		- Imaging the world plane $Z = 0$ corresponds to the homography from world to image $H = K[r_1, r_2, t]$
		- How does this homography change when imaging the plane $Z = z_0$? Idk yet tbd
