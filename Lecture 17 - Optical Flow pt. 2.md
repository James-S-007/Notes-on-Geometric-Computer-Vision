### Invertibility of $A^TA$
- Optical flow does not always reflect the physical motion field
	- I.e. barber pole! We'll see this example a lot coming up
- $\text{rank}(A^TA) = 0$
	- White wall problem
	- $\nabla I_{t+1}(x,y) = 0$
	- No gradient/approximately zero gradient, pixels are indistinguishable from one another
	- Results in both eigenvalues of $A^TA$ being small
	- I.e. uniform areas of barber pole (within red, blue, or white stripe)
- $\text{rank}(A^TA) = 1$
	- Constant/Aligned gradients
	- $\nabla I_{t+1}(x,y) = \vec{c} \text{, } \forall x,y \in N$
	- In barber pole these are the edges of the stripes. Gradient is indistinguishable along the direction of the stripe
- $\text{rank}(A^TA) = 2$
	- Best case scenario: Often times are corners in practice/highly patterned areas
	- In barber pole, these are the corner of the stripe
	- What part of the barber pole we focus on affects the optical flow we see!
	- In this case the optical flow we perceive is downward because the only rank 2 patches (corners) are on the vertical axis
![[Screen Shot 2022-11-17 at 11.01.26 AM.png]]

- Optical Flow Confidence
	- Check determinant of $A^TA$ for invertibility
	- Can rank various pixels according to determinant or to smallest singular value of $A$ (square root of smallest eigenvalue)
	- If $b$ in $Ax=b$ is not in the range of $A$, the MSE will be high
		- Occurs when other assumptions are violated like occlusion, light refraction, etc.

___
### Inferring 3D Motion from Optical Flow
- Problem Statement: Given optical flow, can we compute the 3D motion of the camera?
	- Haven't we already done this with SfM? Yes but with some additional reasonable assumptions we can compute it much more efficiently with this method
- Pixel Motion in terms of 3D Object Motion (1)
	- For $p = (x, y, [1]) \in \mathbb{P}^2$
		- $x = \frac{X}{Z}$
		- $y = \frac{Y}{Z}$
		- $p = \frac{1}{Z}P$
		- $$\dot{p} = \frac{dp}{dt} = \frac{\dot{P}}{Z} - \frac{\dot{Z}}{Z}p$$
		- Where...
			- $\frac{\dot{P}}{Z}$ is the projection of the 3D velocity
			- $\frac{\dot{Z}}{Z}$ is the inverse time to collision
- 3D Motion in terms of Camera Motion (2)
	- For a camera moving at velocity $V$ and spinning at angular velocity $\Omega$ $$\dot{P} = -\Omega \times P - V$$
- Combining 2 Key Equations
	- Plug $\dot{P}$ from equation (2) into equation (1)
	- $$\dot{p} = \frac{1}{Z} \begin{bmatrix} xV_z - V_x \\ yV_z - V_y\end{bmatrix} + \begin{bmatrix} xy & -(1+x^2) & y \\ (1+y^2) & -xy & -x \end{bmatrix} \Omega$$
		- First part of the equation is translational flow ($\dot{p}_{\text{trans}}$)
		- Second part is rotational flow ($\dot{p}_{\text{rot}}$) which is independent of depth