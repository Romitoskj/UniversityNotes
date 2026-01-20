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
> ![](images/Pasted%20image%2020251023091101.png)

>[!Info] Color histograms
>![](images/Pasted%20image%2020251023091425.png)
### Limitations
- can work on relatively simple patterns
- not robust to clutter, occlusion, lighting changes

## Sliding window
A sliding window approach involves checking a classifier across multiple scales and orientations of an image. The core idea is to move a window (or patch) across the entire image space and, at each location, classify the contents of that window.

Key issues associated with this method include:
- The **scale/orientation range** that needs to be searched over.
- The **speed** of the processing.
- The role of **context**.
### Bag-of-words
The Bag of Words model for object recognition originated from two main areas:

- **Texture Recognition (Origin 1)**: This approach treats objects like **texture**. For stochastic textures, recognition depends on the **identity of basic elements** or "textons," rather than their spatial arrangement. A universal texton dictionary is used, and the texture is represented by a **histogram** of these textons.
- **Bag-of-Words Models in Text (Origin 2)**: In text processing, a document is represented as an **orderless document representation** based on the **frequencies of words** drawn from a dictionary
### Bag-of-features
the Bag-of-features model represents an image (or an object) as a collection of local image features, analogous to words in a document, ignoring their relative spatial locations within the image.

The general procedure for implementing the Bag of features model involves four main steps
1. **Extract Features**: Local features are extracted from the image. This can be done using methods like interest point detectors (e.g., SIFT descriptors) or by sampling patches on a regular grid or using random sampling.

2. **Learn "Visual Vocabulary"**: This step involves **clustering** the extracted features from a training dataset to create a "visual vocabulary". The centers of these clusters are known as "visual words" (or "codewords"). A common method for clustering is K-means clustering, which aims to minimize the sum of squared Euclidean distances between points and their nearest cluster centers. Issues arise in choosing the vocabulary size: too small, and words are unrepresentative; too large, and it may lead to overfitting.

3. **Quantize Features**: Each extracted feature from a new image is assigned to the nearest visual word in the vocabulary.

4. **Represent Images by Frequencies**: The image is ultimately represented by a **histogram** that records the **frequencies of these "visual words"** (or features) found within it

The Bag-of-features model is considered part of the history of object representation methods, developed after techniques like geometric models, global appearance-based models (e.g., Eigenfaces), and sliding window approaches. It is used extensively for tasks like **image-level classification** and **scene categorization**.