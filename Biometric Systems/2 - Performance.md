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
	- non-invertable, impossible to get raw data from it for privacy reason
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
- **Equal Error Rate ERR** FAR = FRR
- **Detection Error Trade-off DET**
- **Receiving Operating Curve ROC**

> [!info]
> All measures depend on the adopted **acceptance threshold**

> [!fail] Accuracy $\neq$ FAR
> accuracy = number of correct responses / total responses
> high accuracy systems can have high accuracy but also high FAR or FRR

### Error measures
Let us assume to have:
- a ground truth function $id(template)$ that, given a template, returns its **true** identity. 
- $topMatch(p_j, identity)$ returns the best match between $p_j$ and the templates associated to the claimed identity in the gallery.
- $s(t1,t2)$ returns the similarity between template $t1$ and template (t2)

- $P_G=$ set of probes ***belonging to the subjects*** in the gallery
- $P_N=$ set of probes **belonging to the subjects*** not in the gallery (but in the dataset, so id function works)
---

![[images/Pasted image 20251014132419.png|300]]![[images/Pasted image 20251014132914.png|300]]

> [!info] Definition
> A score is said **genuine** (authentic) if the results from matching two samples of the biometric trait of a same enrolled individual; it is said **impostor** if it results from matching the sample of a non-enrolled individual.

![[images/Pasted image 20251014133657.png]]

*We chose the threshold at EER or we chose to minimize on type of error.*
## Identification
Individual **does not make** an identity claim. The steps for identification:
1. calculate the distances/similarities
2. order them
3. check if the first one is below/above the threshold
![](images/Pasted%20image%2020251014142938.png)
### Open set
In the open set identification task (e.g. watchlist, as black and whitelists), the biometric system  determines if the individual's biometric signature matches a biometric signature of someone in the gallery (or not - never enrolled).

Question:
- is the probe subject in the database? (**Detection**)
- who is the probe subject?
#### Possible errors
There are different cases in comparison with verification:

- ***correct detect and identify***
	- two individuals **above** the threshold = **correct detect**
	- the first individual is the **right** one = **correct identification**
- ***no correct detect and identify***
	- **No** individual above the threshold = **no detect**
		- we do not care about looking at the top individual = **no correct identification**
- **no correct detect and identify**
	- two individuals **above** the threshold = **correct detect**
	- the first individual **is not** the right one = **no correct identification**
	- 
### Closed set
We can assume for sure that every single probe image has a corresponding match in the database. Who is the person?


---

...

--- 
# All vs All
All the template are compared with any other template once and for all.
Diagonal values are not considered (comparison with itself)