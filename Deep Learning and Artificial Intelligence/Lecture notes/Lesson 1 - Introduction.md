*February 24 2026*

### The Importance of Data Analysis

- **"Look at the Data":** The first step in any project is to visualize and inspect the data, rather than blindly trusting cleaning algorithms or summary statistics.

	- **Examples of Failure:**
	- **MNIST:** A past project failed on the MNIST dataset because the student did not look at the data to realize the images were rotated.
	- **Audio Generation:** A generative model for electronic sounds failed to interpolate correctly because the training data was too sparse, leaving "empty space" between data points. Visualizing the distribution revealed the gap.

-  **The Trap of Summary Statistics:**
	-  **Anscombe's Quartet / Datasaurus:** Different datasets (e.g., a dinosaur shape, a star, a circle, or random noise) can share identical summary statistics (mean, variance, correlation) up to two decimal places.
		![](Images/Pasted%20image%2020260302161506.png)

-  **Conclusion:** You cannot trust summary statistics alone; visualization is required. If data is too high-dimensional to visualize directly (e.g., earthquake signals), one must use dimensionality reduction techniques or consult domain experts.

### Defining Machine Learning & Deep Learning

-  **The Inverse Problem:**

	- **High School Math:** Given a function (y=ax+b) and parameters (a,b), find the unknown (x).
	 -  **Machine Learning:** Given the data (x,y), find the unknown parameters (a,b) that explain the data.
	 - **Deep Learning:** The same concept applies, but instead of two parameters (a,b), models have millions or billions of parameters.

- **Priors vs. Learning:**

	- There is a debate on how much "prior knowledge" to force into a model.
	- **Example:** If modeling a bicycle wheel, one could force the model to use a sine wave (periodic prior). This constrains the solution space but might prevent the model from learning unexpected phenomena (e.g., measurement errors).
	- **Warning:** Imposing a view on the model can prevent breakthroughs; the model should ideally learn the structure itself.

### Data Representation

- **Key assumption:** Data as an underlying structure (not always true)
- **Representation Matters:** The choice of how to represent data fundamentally changes the difficulty of the problem.
- **Spiral Example:** A spiral shape cannot be represented as a function y=f(x) because it is not one-to-one. However, it can be easily represented using:
	- **Parametric Curves:** x(t) and y(t) as functions of time.
	- **Polar Coordinates:** Expressing the radius as a function of the angle makes the spiral a linear problem.
- **Takeaway:** Deep learning involves finding learned representations (embeddings) where complex data becomes easier to manipulate.

### The Curse of Dimensionality

- Happens when *features >> observations*
- **Concept:** Increasing the resolution (dimension) of data requires an exponential increase in the amount of data to maintain the same density.
	- **1D (1 pixel):** 100 photos might show a clear distribution on a line.
	- **2D/3D (2-3 pixels):** The same 100 photos become sparse scattered points.
	- **1 Megapixel:** The data points are effectively isolated in a million-dimensional space.
- **Implication:** Better quality data "curses" us by requiring exponentially more samples to extract meaningful structure.

### The Manifold Hypothesis

- **Definition:** While high-dimensional data (like photos) technically lives in a massive space (e.g., $\mathbb{R} ^{1,000,000}$), the "meaningful" data (e.g., realistic photos of volcanoes) lies on a lower-dimensional surface, or **manifold**, embedded within that space.
- **Structure:** This hypothesis implies that despite the curse of dimensionality, there is a structured subspace where data is dense and meaningful. Random noise fills the ambient space, but valid data adheres to the manifold.


>[!quote] **Occam’s Razor:**
>When multiple explanations fit the data, the simplest one is usually correct. Deep learning optimization algorithms act similarly: they try to find the simplest solution that fits the data. If the data is biased, the model learns the "cheat" rather than the complex truth.