Uses:
- Panorama stitching
- Object recognition
Two approaches:
- **Direct** (pixel-based) alignment: search for alignment where most pixels agree
- **Feature-based** alignment: search for alignment where extracted features agree

> [!info] Alignment as fitting
> Fitting a model to a transformation between pairs of features (matches) in two images
> ![](../Pasted%20image%2020251021091305.png)

# Feature-based alignment
Steps:
- Extract features
- Compute *putative matches*
- Loop:
	- Hypothesize transformation T
	- Verify transformation
![](../Pasted%20image%2020251021091524.png)
![](../Pasted%20image%2020251021091536.png)

## 2D transformation models
- Similarity (translation, scale, rotation)
- Affine
- Projective

Fitting a model to a transformation between pairs of features (*matches*) in two images

>[!info] Basic 2D transformations
>![](../Pasted%20image%2020251021092136.png)
>
>![](../Pasted%20image%2020251021092750.png)

