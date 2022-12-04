- Calibrating Camera Coordinates in Practice
	- Use OpenCV and multiple views of a checkerboard
**End of Single View Geometry**
___
- 3D Motion from 2 Views: Structure from Motion Problem (SfM)
	- Problem Overview
		- Input: 2 calibrated views of the same rigid 3D scene
		- Output: Find $R,T$ between the two views and depths for point correspondences from each view
		- Mathematical Formation
			1. $X^{C1} \rightarrow [R^{C2}_{C1}[X]^{C1} + T^{C2}_{C1}]^{C2}$
			2. $[\lambda p]^{C1} \rightarrow [R(\lambda p) + T]^{C2} = [\mu q]^{C2}$
		- ![[Screen Shot 2022-10-22 at 11.14.01 AM.png]]
	- Epipolar Constraints between 2 Views of a Scene
		- $$q^T(T \times Rp) = 0$$
		- Allow us to eliminate depths from the equation
		- How?
			- In our previous equation: $\lambda Rp + T = \mu q$; $q,Rp,T$ are the edges of a triangle, which is by definition planar
			- Volume of a parallelpiped: ![[Screen Shot 2022-10-22 at 11.32.31 AM.png]]
			- The vectors $q$, $T$, and $Rp$ form a triangle which has volume 0 (because it's planar)
			- Don't overthink it, very weird problem reformation but it's actually pretty simple