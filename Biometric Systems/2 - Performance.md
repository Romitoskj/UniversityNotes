## Problems
- Intra-class variations (expressions, poses, occlusion)
- Inter-class variations (twins or relatives)
- noisy or distorted acquisitions
- non-universality of a trait
- spoofing attacks

## Comparisons
### What to compare
- **Sample:** the raw captured data
- **Hand-crafted features:** features that are manually engineered by data scientist and *extracted from samples*
- **Template:** collection of feature extracted from raw data
	- non-invertible, impossible to get raw data from it for privacy reason
### How to compare templates
Templates are...
- vectors:
	- **Euclidean distance**
	- **Cosine similarity** (it take the angles into account)
- Histograms or sets of points:
	- **Pearson Correlation**
	- **Bhattacharyya distance**
- Time series:
	- **Dynamic Time warping**
	- 