The goal of image alignment is to find a transformation T that minimises the residual between pairs of features, xi​ and xi′​, in the two images. This transformation T minimises ∑i​residual(xi​,xi′​,T)

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

then generate putative matches: for each patch in one image, find a short list of patches in the other image that could match it based solely on appearance

### Dealing with outliers
Strategies:
1. RANSAC
2. Incremental alignment
3. Hough transform
4. Hashing
# Application
- Panorama stitching
- Recognizing panoramas
# Issues
- choosing the geometric alignment model
- choosing the descriptor
- strategy for finding putative matches
