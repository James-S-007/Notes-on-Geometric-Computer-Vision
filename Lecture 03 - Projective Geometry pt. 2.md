- Projective Geometry $\leftarrow \rightarrow$ Euclidean Interpretation
	- In the Euclidean interpretation, we treat $w$ as the third spatial coordinate
	- $w$ is a scaled version of the principal axis $Z$
	- Image plane is $w = 1$
	- $w = 0$ is the same as $Z = 0$ and is parallel to the image plane, passing through the camera center
- The Line at Infinity
	- $$l_\infty = (0,0,1)^T$$
	- Rewritten as $\begin{bmatrix} 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x_1 \\ x_2 \\ 0 \end{bmatrix} = 0$
	- Line passing through all ideal points (points at infinity)
- Camera Coordinate System with Principal Point Offset
	- Recall: $\begin{bmatrix} X \\ Y \\ Z \\ 1 \end{bmatrix} \rightarrow \begin{bmatrix} fX \\ fY \\ Z \end{bmatrix}$
	- With an offset from the image center:
	- $$u = f \frac{X_c}{Z_c} + u_0$$
	- $$v = f \frac{Y_c}{Z_c} + v_0$$
	- This becomes... $$\lambda \begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = \begin{bmatrix} f & & u_0 \\ & f & v_0 \\ & & 1 \end{bmatrix} \begin{bmatrix} X_c \\ Y_c \\ Z_c \end{bmatrix}$$
- Camera to World: Euclidean Transformation
	- $$\begin{bmatrix} X_c \\ Y_c \\ Z_c \\ 1 \end{bmatrix} = R_{3 \times 3} \begin{bmatrix} X_w \\ Y_w \\ Z_w \\ 1\end{bmatrix} + t = \begin{bmatrix} R_{3 \times 3} & t \\ 0 & 1 \end{bmatrix} X_w$$
	- Putting it all together...$$\lambda \begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = \begin{bmatrix} f & & u_0 \\ & f & v_0 \\ & & 1 \end{bmatrix} \begin{bmatrix} R_{3 \times 3} & t \end{bmatrix} \begin{bmatrix} X_w \\ Y_w \\ Z_w \\ 1 \end{bmatrix} = K \begin{bmatrix} R | t\end{bmatrix}X_w$$
	- In summary
		- Rotation and translation convert from world coordinates to camera-centric coordinates
		- $K$ projects and converts to pixel coordinates
		- $$x \sim K[R|t]X_w$$
		- Camera Projection Matrix: $P_{3 \times 4} = K_{3 \times 3}[R_{3 \times 3}|t_{3 \times 1}]$
			- $K$ represents camera *intrinsics* and $R$ and $t$ represent *extrinsics*
			- $P$ has 9 DOF (3 for rotation + 3 for translation + 3 for intrinsics) 