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