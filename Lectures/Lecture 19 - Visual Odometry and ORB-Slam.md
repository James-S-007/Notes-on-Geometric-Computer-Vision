### Visual Odometry
- Types
	- Odometry: Measuring how far you go by counting wheel rotations or steps (also known as path integration)
	- Inertial Odometry: Integration of velocity or acceleration measurements
	- Visual Odometry: Process of incrementally estimating your position and orientation with respect to an initial reference frame by tracking visual features in an unknown environment
- Why Visual Odometry?
	- Error from inertial approaches continues to grow larger as time goes on (double integration of acceleration), known as drift
- Why not just adapt SfM to more than 2 frames
	- Would also lead to drift
	- Need a new system that maintains some kind of consistency over longer durations
### Visual Odometry Basic Pipeline
1. Do 2-View SfM between first 2 frames from video
	- Outputs:
		1. $R_2,T_2$ of camera 2 with respect to camera 1
		2. A 3D map with the 3D positions $X_i$ of points seen in both frames
2. For next frame $k$, identify 2D-2D correspondences with the previous frame
	- These produce 2D-3D correspondences between this frame and the current map
	- Compute $R_k,T_k$ with PnP
3. Improve triangulated 3D map $X_i$ and expand it using correspondences that aren't yet in the map
	- Go back to Step 2
- NOTE: Larger baselines ($T_k \rightarrow T_{k+1}$) result in lower triangulation error. Pixels are discrete and the true point could lie anywhere in the discretization of the pixel. A wider baseline reduces this as seen below
![[Screen Shot 2022-11-19 at 11.48.40 AM.png]]

### ORB-SLAM
![[Screen Shot 2022-11-19 at 11.49.36 AM.png]]
- Definitions
	- **Map Points**: A list of 3D points that represent the map of the environment reconstructed from the key frames
	- **Key Frames**: A subset of video frames that contain cues for localization and tracking. The current key frame should have some overlap with the previous key frame but also be sufficiently different to gain additional information
#### Map Initialization
1. Wait for a good keyframe
	1. Keep comparing between 1st frame and $k^{th}$ until none of the following happens
		- Homographic frames (rank deficient system for estimation of essential matrix $E$)
			- Means scene is planar or there is no translation
		- Insufficient inliers for an estimate $E$ after performing RANSAC
	2. Declare this as the next keyframe
2. Store keyframe features and poses computed with 2-view SfM
3. Triangulate and initialize a 3D map with these localized points
#### Tracking
1. Extract ORB features with previous keyframe that are already in the 3D map
2. Use PnP with RANSAC to estimate the camera pose with respect to the last keyframe
3. Given the camera pose, project the unmatched map points from the previous key frame into the current frame and search for more features correspondences
4. Refine PnP with "pose-only bundle adjustment" (out of scope)
5. Define this frame as a key frame if both:
	1. At least 20 frames have passed since the last key frame or the current frame tracks fewer than 100 map points
	2. Fewer than 90% of the map points tracked by the current frame are also tracked by the reference key frame
#### Local Mapping
1. Project local map points visible in several neighboring keyframes into the current frame to search for more features correspondences
	1. Refine for camera pose further
	2. Triangulate new correspondences to grow the map further
2. Local bundle adjustment to refine these and achieve optimal reconstruction in the neighborhood of the current camera pose
- Overall: Lots of quality checking to avoid redundant keyframes and low-quality correspondences
### Loop Closure
1. How to know whether I've returned to a place I've visited?
	- Deep learning or bag of visual words to find past keyframes that look similar
	- If visual similarity candidate is found, run a geometric consistency check: Align 3D points from current frame and this keyframe and check that they follow a similarity transformation
2. If loop closure declared, run bundle adjustment to update all poses in essential graph

### Bundle Adjustment
- Used to refine camera poses and 3D points starting from some pretty good initialization
- Minimize reprojection error: The sum of errors between 2D observations and the reprojected 2D points $$\text{argmin}_{\{X_n\}^N_{n=1}, \{R_f,T_f\}^F_{f=1}} \sum_{n,f}d(x_{nf}, K_f[R_f|T_f]X_n)$$
	- $x_{nf}$ is the observed $n^{th}$ point in frame $f$
	- $R_f, T_f$ are the extrinsics for frame $f$
	- $X_n$ is the 3D estimation of the observed point
	- We are solving for $R_f,T_f,X_n$ for all frames $f$ and points $n$