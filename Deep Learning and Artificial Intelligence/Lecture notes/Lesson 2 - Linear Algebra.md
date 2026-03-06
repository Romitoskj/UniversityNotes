*Feb. 25 2026*

### 1. Philosophy of Linear Algebra

- **Reframing the Subject:** Linear algebra is not merely the study of matrices and lists of numbers. It is formally the **study of linear maps on finite-dimensional vector spaces**.
- **The Role of Matrices:** Matrices are tools used to *represent* linear maps. As the lecturer notes, "Linear algebra is about matrices as much as astronomy is about telescopes".
- **Mental Shift:** Think in terms of maps and spaces first, and grids of numbers (matrices) second.

### 2. Vector Spaces

A vector space $V$ is a set of objects (vectors) equipped with two operations: **addition** and **scalar multiplication**, satisfying specific axioms.

- **Core Intuition:**
    - **Closure:** The most critical concept is "What happens in Vegas, stays in Vegas." If you sum two vectors or scale a vector, the result must remain within the same vector space.
    - **Axioms:** The operations must satisfy properties like commutativity, associativity, additive identity (zero vector), additive inverse, multiplicative identity and distributivity.
    - **Non-Requirements:** A vector space does not strictly require a definition of length (norm), distance (metric), or multiplication between vectors. Adding those creates specific types of spaces (e.g., Hilbert spaces, Metric spaces).
- **Examples & Non-Examples:**
    - **Lists of Numbers ($\mathbb{R}^n$):** The standard example. $\mathbb{R}^n$ is the set of sequences of n numbers. Addition and scalar multiplication are element-wise.
    - **Function Spaces:** The set of all functions $f:D\rightarrow R$ forms a vector space.
        - Sum: (f+g)(x)=f(x)+g(x).
        - Scaling: (λf)(x)=λ⋅f(x).
        - The zero vector is the function that is 0 everywhere.
        - _Note:_ The domain D does not affect the vector space property. D can be an interval, a set of pixels, or a 3D surface.
    - **The "Rabbit" (Surface) Example:**
        - The set of points making up a 3D surface (like a rabbit mesh) is **not** a vector space. Summing two points on the surface might result in a point inside or outside the rabbit (closure is violated).
        - However, the set of **functions defined on the rabbit** (e.g., a heatmap or texture coloring) **is** a vector space.

### 3. Basis and Dimension

A **basis** is a collection of vectors that are linearly independent and span the vector space.

- **Spanning:** Any vector in the space can be expressed as a **linear combination** of the basis vectors ($v=\sum \alpha_i​v_i$​).
- **Uniqueness:** There is only **one** unique way to express a specific vector using a given basis. If there are multiple ways, the set is "over-complete" and not a basis (basis vector must be linearly independent).
- **The Standard Basis (Indicator Functions):**
    - In $\mathbb{R}^n$, these are vectors like (1,0,…).
    - In function spaces (like images), these are "one-hot" functions—pixels that are 1 at a specific location and 0 everywhere else. Any image can be reconstructed by scaling and summing these pixel-functions.
- **Dimension:** The dimension of the space is the number of vectors in its basis.
    - **Finite Dimensional:** Digital images (discrete pixels), polynomials of degree n.
    - **Infinite Dimensional:** Continuous functions on a continuous domain (e.g., functions on a real line interval). This moves from Linear Algebra into **Functional Analysis**.

### 4. Linear Maps

A linear map T is a function between two vector spaces ($T:V→W$) that preserves the vector space structure.

- **Two Essential Conditions:** To prove a map is linear, one must check:
    1. **Additivity:** $T(u+v)=T(u)+T(v)$.
    2. **Homogeneity:** $T(λv)=λT(v)$.
- **Examples of Linear Maps:**
    - **Differentiation:** $T(f)=f'$. The derivative of a sum is the sum of derivatives.
    - **Integration:** $T(f)=∫f(x)dx$.
    - **Reflection:** Flipping a coordinate is linear.
    - **Rotation:** Rotating a vector is linear.
- **Non-Examples (Important):**
    - **Translation:** $T(x)=ax+b$. This is **not** linear (it fails additivity). It is called an **Affine** map.
    - **Squaring:** $T(x)=x^2$ is not linear.
    - **Confusion Warning:** A map can be linear with respect to one variable but not another. For example, $y=x^2f(x)$ is linear _with respect to the function_ f (rescaling), even though it involves an $xˆ2$ term.
- **Vector Space of Maps:** The set of all linear maps between two vector spaces is, itself, a vector space (you can sum maps $S+T$ and scale them).

### 5. Matrices

A matrix is a **representation** of a linear map relative to specific bases.

- **Construction of a Matrix:**
    1. Take a basis vector from the input space ($v_j$​).
    2. Apply the linear map ($T(v_j​)$).
    3. Express the result as a linear combination of the output space's basis.
    4. These coefficients form the j-th **column** of the matrix.
- **Dependency on Basis:** A single linear map can be represented by infinitely many different matrices, depending on the choice of basis for the domain and codomain. In Deep Learning, we usually assume the standard basis unless specified otherwise.
- **Matrix Operations as Map Operations:**
    - **Matrix-Vector Product ($Ax$):** Represents applying the linear map to a vector. It corresponds to a linear combination of the matrix columns weighted by the vector entries.
    - **Matrix-Matrix Product ($AB$):** Represents the **composition of functions** ($S∘T$).
    - **Non-Commutativity:** Matrix multiplication is generally not commutative ($AB\neq BA$) because function composition is generally not commutative.

**6. Matrix Meta-Mechanics (Advanced Properties)**

- **Symmetry:** A matrix is symmetric if $A=A^T$.
- **Orthogonal Matrix:** A matrix is orthogonal if $A^TA=I$ (or $A^{−1}=AT$). Its columns are orthonormal vectors.
- **Trace:** The sum of diagonal elements. The trace is a linear map from the space of square matrices to R.
- **Rank-1 Matrices:** An outer product of two vectors (xyT) results in a rank-1 matrix. Its columns are all collinear (scaled versions of the same vector), so they span a 1D subspace.
- **Inner vs. Outer Product:**
    - **Inner (Dot) Product ($x^Ty$):** Results in a scalar.
    - **Outer Product ($xy^T$):** Results in a matrix (column × row).