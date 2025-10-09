# Problems
- Intra-class variations (expressions, poses, occlusion)
- Inter-class variations (twins or relatives)
- noisy or distorted acquisitions
- non-universality of a trait
- spoofing attacks

# Comparisons
## What to compare
- **Sample:** the raw captured data
- **Hand-crafted features:** features that are manually engineered by data scientist and *extracted from samples*
- **Template:** collection of feature extracted from raw data
	- non-invertible, impossible to get raw data from it for privacy reason
## How to compare templates
Templates are...
- vectors:
	- **Euclidean distance**
	- **Cosine similarity** (it take the angles into account)
- Histograms or sets of points:
	- **Pearson Correlation**
	- **Bhattacharyya distance**
- Time series:
	- **Dynamic Time warping**

> [!Info]
> Once similarity or distance is computed, it is compared with an acceptance threshold if in verification or identification open set

# Possible errors
## Verification
A subject is accepted if the similarity achieved from the comparison is greater than or equal to acceptance threshold, or distance is less than or equal to threshold.

4 possible cases:
- ✅ **Genuine Acceptance (GA)**: The claim of identity is true and the subject is accepted
- ❌ **False Rejection: (FR)** The claimed identity is true but the subject is rejected
- ✅ **Genuine Rejection (GR):** An impostor is rejected
- ❌ **False Acceptance (FA):** An impostor subject is accepted

It is necessary to *measure* and *compare* performance (counting errors is not suited):
- **False Acceptance Rate - FAR:** percentage of recognition operations with an impostor claim in which false acceptance occurs. *False acceptance rate + genuine acceptance rate GAR = 1*
- **False Rejection Rate - FRR:** percentage of recognition operations with a genuine claim in which false rejections occurs. *False rejection rate + genuine rejection rate  GRR= 1*
- **Equal Error Rate ERR**
- **Detection Error Trade-off DET**
- **Receiving Operating Curve ROC**

> [!Danger] I
> All measures depend on the adopted **acceptance threshold**

### Problem of accuracy:
accuracy = number of correct responses / total responses
high accuracy systems can have high accuracy but also high FAR or FRR

