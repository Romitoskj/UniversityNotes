Choose a parametric model to represent a set of features.
Membership criterion is not local: can't tell whether a point belongs to a given model just looking a that point.

- What model represents this set of features best?
- Which of several model instances gets which feature?
- How many model instances are there?

Issues:
- noise in the measured feature locations
- extraneous data: clutter, multiple lines
- missing data: occlusions

Let each feature vote for all the models that are compatible with, hopefully the noise features will not vote consistently for any single model.

# Hough space
## Polar representation
![](images/Pasted%20image%2020251016085705.png)
