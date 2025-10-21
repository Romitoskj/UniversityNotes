Uses:
- Panorama stitching
- Object recognition
Two approaches:
- **Direct** (pixel-based) alignment: search for alignment where most pixels agree
- **Feature-based** alignment: search for alignment where extracted features agree

> [!info] Alignment as fitting
> Fitting a model to a transformation between pairs of features (matches) in two images
> ![](images/Pasted%20image%2020251021091305.png)

# Feature-based alignment
Steps:
- Extract features
- Compute *putative matches*
- Loop:
	- Hypothesize transformation T
	- Verify transformation
![](images/Pasted%20image%2020251021091524.png)
![](images/Pasted%20image%2020251021091536.png)

## 2D transformation models
- Similarity (translation, scale, rotation)
- Affine
- Projective

Fitting a model to a transformation between pairs of features (*matches*) in two images

>[!info] Basic 2D transformations
>![](images/Pasted%20image%2020251021092136.png)
>
>![](images/Pasted%20image%2020251021092750.png)

What if we don't know the correspondences? (we extracted features but we don't have matches)
We can compare each one, but is expensive

## Feature descriptors
Assuming the patches are already normalized, how do we compute their similarity?
Want invariance to intensity changes, noise, perceptually insignificant changes of the pixel pattern.

Descriptor: vector of raw intensity values
we compare them with sum of squared differences SSD
but small shifts can affect matching score a lot

### SIFT
Descriptor computation:
- divide patches into 4x4 sub-patches
- compute histogram of gradient orientations inside each sub-patch
- resulting description: $4\times4\times8=128$ dimension

Gradient is less sensitive to illumination change
"subdivide and disorder" strategy achieves robustness to small shifts, but still preserves spatial information