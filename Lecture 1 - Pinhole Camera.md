- Can model a camera as pinhole camera
	- Good model as long as the aperture is small
	![[Screen Shot 2022-10-20 at 10.13.22 AM.png]]
	- Can move image plane in front of camera to remove negative sign
- Perspective Projective Equations
	$$x = f\frac{X}{Z}$$
 $$y = f\frac{Y}{Z}$$
- Properties preserved by Perspective Projections
	- Only straightness/collinearity
	- Not shape, parallelness, lengths, ratio of lengths
	- Perspective effects are strongest when the object is close to the camera
- Linear Algrebra Review
	-  Dot Product
$$x \cdot y = x^Ty = \sum_ix_iy_i = \lvert\lvert x \rvert\rvert * \lvert\lvert y \rvert\rvert cos\theta_{xy}$$
			- Output is scalar
			- Measures angles if applied to unit vectors
			- Measures projection of one vector onto another
		- Orthogonality: $x^Ty = 0$
		- Cross Product $a \times b$
			- Outputs vector with magnitude: $||a|| * ||b|| sin\theta_{xy}$ and direction perpendicular to both vectors
			- Computed as the determinant of $\begin{bmatrix} i & j & k \\ a_1 & a_2 & a_3 \\ b_1 & b_2 & b_3 \end{bmatrix}$
			- $a \times b = -(b \times a)$
