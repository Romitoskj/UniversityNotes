# Edge Detection
**Goal:** Identify sudden changes (discontinuities) in an image, because most semantic and shape information from the image can be encoded in the edges.
**Ideal:** artist's line drawing
## Origin of edges
![[Pasted image 20251012204902.png]]

> [!Info] Edge
> An edge is a place of rapid change in the image intensity function
> 
> ![[Pasted image 20251012213908.png]]
> 
> *Edges correspond to extremes of derivative*

## Image gradient
The image is a function of two variables so we must use gradient:
$$
\nabla f = \bigg[ \frac{\partial f}{\partial x},\frac{\partial f}{\partial y}\bigg]
$$
The gradient points in the direction of most rapid increase in intensity:
- The gradient *direction* is given by $\theta=\tan ^{-1} \big(\frac{\partial f}{\partial y} \big/ \frac{\partial f}{\partial x} \big)$ which is perpendicular to the edge direction
- The edge *strength* is given by the gradient magnitude $\lVert \nabla f \rVert = \sqrt{\big(\frac{\partial f}{\partial y} \big)^2 + \big( \frac{\partial f}{\partial x} \big)^2}$ (norm)

> [!Note] Finite filterse
> It translates to finite difference filters (one for x axis one for y axis):
> 
>![[Pasted image 20251012222754.png]]
>![[Pasted image 20251012222859.png]]

## Effect of noise

> [!fail] Where is the edge?
> Consider a single row or column of the image and plotting the intensity as a function of position given signal
> 
![[Pasted image 20251012223944.png]]
>
> **Finite difference filters respond strongly to noise.**

> [!check] Solution: Smooth First
> ![[Pasted image 20251012224256.png]]
> 
> To find edges, look for peaks in the differentiation after applying a smooth filter.

> [!note]
> To save one operation, we can combine the smoothing filter by the differentiation filter because convolution is associative: $\frac{d}{dx}(f*g)=f*\frac{d}{dx}g$ 
> 
> ![[Pasted image 20251012224950.png]]
> 
> Example: the derivative of the **Gaussian filter**
> 
> ![[Pasted image 20251013123756.png]]
> ![[Pasted image 20251013123846.png]]
> 
> **Tradeoff between smoothing and localization** because smoothed derivative removes noise but also blurs edges
> 
> ![[Pasted image 20251013124909.png]]

## Implementation issues
