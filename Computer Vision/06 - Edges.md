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
> Edges correspond to extremes of derivative

## Image gradient
The image is a function of two variables so we use a gradient:
$$
\nabla f = \bigg[ \frac{\partial f}{\partial x},\frac{\partial f}{\partial y}\bigg]
$$
The gradient points in the direction of most rapid increase in intensity:
- The gradient direction is given by $\theta=tan^{-1} \big(\frac{\partial f}{\partial y} \big/ \frac{\partial f}{\partial x} \big)$
- 