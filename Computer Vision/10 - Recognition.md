Object recognition involve:
- Verification
- Detection
- Identification
- Object categorization
- Scene and context categorization

Model variability (Intra-class variation):
- Camera position
- Illumination
- Internal parameters
Alignment to manage those problems

>[!info] Recognition by components
>Recognition can be performed by components, recognizing the shapes present in the object
>
>![](images/Pasted%20image%2020251023090738.png)
## Appearance-based techniques
> [!info] Eigenfaces
> ![](../Pasted%20image%2020251023091101.png)

>[!Info] Color histograms
>![](../Pasted%20image%2020251023091425.png)
### Limitations
- can work on relatively simple patterns
- not robust to clutter, occlusion, lighting changes

## Sliding window
A sliding window approach involves checking a classifier across multiple scales and orientations of an image. The core idea is to move a window (or patch) across the entire image space and, at each location, classify the contents of that window.

Key issues associated with this method include:
- The **scale/orientation range** that needs to be searched over.
- The **speed** of the processing.
- The role of **context**.