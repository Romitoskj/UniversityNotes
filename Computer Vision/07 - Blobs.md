# Extracting features
>[!example]- For example we need features for panorama stitching:
![[Pasted image 20251014090940.png|250]]![[Pasted image 20251014091012.png|250]]
![[Pasted image 20251014091043.png|500]]
## Characteristics of good features
- **Repeatability:** same feature can be found in several images despite geometric and photometric transformation
- **Saliency:** each feature has a distinctive description
- **Compactness and efficiency:** many fewer features than image pixels (the most descriptive with few spatial dimension, like points)
- **Locality:** a feature occupies a relatively small area of the image; robust to clutter and occlusion
## Applications
- Motion tracking
- Image alignment
- 3D reconstruction
- Object recognition
- Indexing and database retrieval
- Robot navigation
## Finding corners
**Key property:** In the region around a corner, image gradient has two or more dominant directions. They are repeatable and distinctive.
### Basic idea
Shift a small window in *any direction* should give a *large change* in intensity.
![[Pasted image 20251014092515.png]]
### Harris Detector
Difference of intensity
%%Add formulas%% 
At the end of computation we obtain a $2 \times 2$ matrix computed from image derivates.

![[Pasted image 20251014093152.png]]
Dominant gradient directions align with x or y axis.
if either $\lambda$ is close to 0, then this is not a corner, so look for location where both are large.

Since M is symmetric we have ![[images/Pasted image 20251014094107.png|200]]
We can visualize M as an ellipse with axis lengths determined by the eigenvalues and orientation determined by R
![[images/Pasted image 20251014094228.png]]
#### Visualization of second moment matrices
![[images/Pasted image 20251014094645.png|300]]
#### Interpreting the eigenvalues
![[images/Pasted image 20251014094800.png|400]]
#### Corner response function
![[images/Pasted image 20251014094859.png]]
![[images/Pasted image 20251014094915.png|400]]
#### Harris detector steps
1. Gaussian derivatives at each pixel
2. Compute second moment matrix M in a gaussian window around each pixel
3. Compute corner response function R
4. Threshold R
5. Find local maxima of response function (non-maximum suppression)
## Invariance
We want features to be detected despite geometric or photometric changes in the image: if we have to transformed versions of the same image, features should be detected in corresponding locations.
### Models of image change
- Geometric
	- Rotation
	- Scale
	- Affine
- Photometric
	- Affine intensity change $(I \rightarrow aI+b)$ 
# Scale-invariant feature detection
