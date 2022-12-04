- Recap: Strategy for 2-View SfM
	- Solve for $E = \hat{T}R$
	- Then from $E$, get $R,T$
	- Finally recover $\lambda_i, \mu_i$
	- Problem: How to ensure $E$ is valid
- Valid Essential Matrices
	- $E$ is essential if and only if $\sigma_1(E) = \sigma_2(E) \neq 0$ and $\sigma_3(E) = 0$
	- Proof:
		- $E$ is a singular matrix because $E = \hat{T}R$ and $\det (\hat{T}) = 0$
		- Can find the other 2 singular values of $E$ from the eigenvalues of $EE^T$
		- Skipping over the math, we get $\sigma_1(E) = \sigma_2(E) = ||T||$
- To get the closest valid essential matrix,
	1. $$E = SVD(E) = U \begin{bmatrix} \sigma_1 & & \\ & \sigma_2 & \\ & & \sigma_3 \end{bmatrix} V^T$$
	2. $$E_{new} = U \begin{bmatrix} (\sigma_1 + \sigma_2)/2 & & \\ & (\sigma_1 + \sigma_2)/2 & \\ & & 0 \end{bmatrix} V^T$$
	3. Alternatively can also just set $\Sigma = \begin{bmatrix} 1 & & \\ & 1 & \\ & & 0 \end{bmatrix}$ because $E$ is scale-invairant
- Construction of $E$ as $\hat{T}R$
	- Proof:
		1. 
			- Can consider the simplest possible valid essential matrix: $E = \begin{bmatrix} 1&0&0\\0&1&0\\0&0&0\end{bmatrix}$
			- This can be decomposed into $\begin{bmatrix} 0&1&0\\-1&0&0\\0&0&0 \end{bmatrix} \begin{bmatrix} 0&-1&0\\1&0&0\\0&0&1 \end{bmatrix} = \hat{T}R$ which gives us $T = T_{-z} = \begin{bmatrix} 0\\0\\-1 \end{bmatrix}$ and $R = R_{z,\pi/2}$, a 90$\degree$ rotation about the z axis
		2. 
			- If $Q$ is orthogonal, $Q^TQ = I$, then $\hat{Qa} = Q\hat{a}Q^T$
		- Putting $1$ and $2$ together
			- $E = U \begin{bmatrix} \sigma & 0 & 0 \\ 0 & \sigma & 0 \\ 0 & 0 & 0 \end{bmatrix} V^T = \sigma U \begin{bmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 0 \end{bmatrix} V^T$
			- $$E = \sigma U \hat{T_{-z}}R_{z,\pi/2}V^T = \sigma U \hat{T_{-z}}U^TUR_{z,\pi/2}V^T = \sigma(\hat{UT_{-z}})(UR_{z,\pi/2}V^T)$$
			- We can see $T$ is the last left singular vector of $E$ with a flipped sign (kinda, will revisit in a sec) and $R$ is orthogonal because $U$ and $V$ are orthogonal as well
		- This solution is not unique! There are 4 possible solutions (also can remove scale $\sigma$ technically, so we shall do so below). $E = ...$
			1. $(\hat{UT_{-z}})(UR_{z,+\pi/2}V^T)$
			2. $(\hat{UT_{+z}})(UR_{z,+\pi/2}V^T)$
			3. $(\hat{UT_{+z}})(UR_{z,-\pi/2}V^T)$
			4. $(\hat{UT_{-z}})(UR_{z,-\pi/2}V^T)$
		- Can disambiguate these solutions by enforcing all points to have positive depths (as they are in front of the camera)
		- Additionally, 2 of the rotation matrices will be in a left-hand coordinate system (having $\det(R) = -1$) so we can remove solutions that way as well
	- Computing depths $\lambda_i, \mu_i$ through triangulation
		- Can compute depths up to a scale factor after computing $R$ and $T$
		- Set $||T|| = 1$ since scale is irrelevant
		- $$\begin{pmatrix} q_i & -Rp_i \end{pmatrix}_{3 \times 2} \begin{pmatrix} \mu_i \\ \lambda_i \end{pmatrix}_{2 \times 1} = T_{3 \times 1}$$
		- Have 3 equations and 2 unknown, so can solve each point independently using the pseudoinverse