- SVD Primer to the Primer
	- Any matrix $A_{m \times n} = U_{m \times m} \Sigma_{m \times n} V^T_{n \times n}$ 
	- Columns $u_i$ of $U$ are an orthonormal basis for the column space (left singular vectors)
	- Columns $v_i$ of $V$ (not $V^T$!) are an orthonormal basis for the row space (right singular vectors)
	- $\Sigma$ is a diagional matrix containing square roots $\sigma_i = \sqrt{\lambda_i}$  of eigenvectors of $A^TA$ (the eigenvalues)
	- In relation to the homography
		- $AV = U\Sigma$ or $Av_i = \sigma_iu_i$ so $\sigma_i$ is the length of $Av_i$
		- The closest approximation to the null vector $h$ for finding homographies is therefore the smallest right singular vector, corresponding to the smallest singular value
- What is preserved under a Homography?
	- Not lengths, not length ratios
	- Ratio of ratios of distances of 2 points from 2 other collinear points is however
- Cross Ratio
	- $\frac{AC}{AD}:\frac{BC}{BD}$ is the cross ratio of the 4 collinear pts $A,B,C,D$
	- Can also be written $\frac{AC \cdot BD}{AD \cdot BC}$
	- Remains invariant under projective transformations
	- Different ordering of $CR(A,B,C,D)$ will produce different values but are still preserved under a projective transformation
- Cross Ratio of a Pt at $\infty$
	- Given pt $D$ is at $\infty$, the cross ratio becomes $\frac{AC}{\infty} : \frac{BC}{\infty} = \frac{AC}{BC} \rightarrow$ just a ratio!
	- Vanishing pts allow us to leverage this, only need measurement of 1 distance in the world
	- Can also be used the other way: given 2 known lengths in the world plane and the $A', B', C'$ in pixels we can compute the vanishing point
- Computing Distances *from* a World Plane
	1. Find the horizon (from $H$ or by vanishing points)
	2. Connect horizon with bottom of known world length $b_1$ and unknown world length $b_2$, denote this $a$
	3. Connect $a$ with top of known length $t_1$ and continue to connect to object of unknown length, denote this $m$. $mb_2$ is the same length as $b_1t_1$
	4. Compute the unknown length via cross ratios and the vanishing point where $t_1$ and $t_2$ converge (could be at $\infty$ or not)
	![[Screen Shot 2022-10-21 at 10.36.22 AM.png]]
- Camera 6DOF Pose Estimation with Respect to World Plane
	- Compute extrinsics given homography $H$ and camera intrinsics $K$
	- Naive Solution:
		- Given all points in the world lie on the ground plane $Z=0$ $$\begin{pmatrix} u \\ v \\ w \end{pmatrix} \sim
		K \begin{pmatrix} r_1 & r_2 & T \end{pmatrix} \begin{pmatrix} X \\ Y \\ W \end{pmatrix}$$
		- We then should be able to recover: $K^{-1}H = (r_1,r_2,T)$ and $r_3 = r_1 \times r_2$ BUT we are not guaranteed to recover a valid rotation matrix in this  manner; there is no constraint guaranteeing $r_1$ and $r_2$ will be orthonormal
	- Procrustes
		- Goal: Given camera intrinsics and homography, let us find the closest valid rotation matrix
		- Given $K^{-1}H = \begin{pmatrix} h_1' & h_2' & h_3' \end{pmatrix}$
		- We find the orthogonal matrix $R$ that is the closest to $\begin{pmatrix} h_1' & h_2' & h_1' \times h_2' \end{pmatrix}$
		- $$arg min_{R \in SO(3)} || R - \begin{pmatrix} h_1' & h_2' & h_1' \times h2' \end{pmatrix}||^2_F$$
	- Kabsch Algorithm for Procrustes
		- If the SVD of $$\begin{pmatrix} h_1' & h_2' & h_1' \times h_2' \end{pmatrix} = U \Sigma V^T$$
		- Then $$R = U \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & det(UV^T)\end{pmatrix} V^T$$
		- $$T = h_3' / ||h_1'||$$
		- Diagonal matrix is inserted to guarantee $det(R) = 1$