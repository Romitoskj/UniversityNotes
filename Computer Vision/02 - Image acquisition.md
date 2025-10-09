## Surface properties
- Absorption
- Diffusion
- Reflex
- Transparency
- Refraction

![[Pasted image 20250925084549.png]]
![[Pasted image 20250925084812.png]]
![[Pasted image 20250925084856.png]]

#### Image formation
Natural images has:
- light source that generate incident light on a point
- reflectivity function
- light reflects from a point and the reflected light is captured by an imaging device

#### Colors 
- cameras have three sensor with sensitivity functions tuned to the colors red, green and blue, outputting three image functions

## Digitalization
**Digital image** = multidimensional array of number (intensity image) or vectors (color image)
Each component in the image called **pixel** associates with value (single value in intensity images or vector in color images)

**pixel** = single value on a sampled grid, does not correspond to a but but to the smallest treatable area

**Image** = multidimensional function of spatial coordinates *f(x,y)*
**Spatial coordinate** = (x,y) with origin on the top left
***f(x,y)*** may represent the intensity or color

#### Digital image types
- Intensity or monochrome image: each pixel represent light intensity (grayscale)
- RGB: each pixel vector with red, green and blue
- Binary image: black and white , each pixel 1 white and 0 black
- Index image: pixel contains index number pointing to a color in a color table

**Spatial resolution:** the smallest distinguishable detail in an image:
- ppi: pixel per inch
- dpi: dot per inch
- M\*N does not express the relationship between the number of pixel and the detail level of the original scene

## Relationship of pixels
### Neighbours
Neighbourhood relation is used to tell adjacent pixels, useful for analyzing regions
- 4-neighbours (vertical and horizontal)
- 8-neighbours (all neighbour pixels)
- diagonal 
### Connectivity
Two pixels are connected if they are in the same class (e.g. same color) and they are neighbour
### Adjancency
A pixel is adjacent to another pixel if they are connected
Two image subsets are adjacent if some pixel in the first subset is adjacent to some pixel in the second subset
### Path
a path from pixel p at (x,y) to a pixel q at (s,t) is a sequence of distinct and adjacent pixels that connects p to q
### Distance
For pixel p, q and z with coordinates (x,y), (s,t) and (u,v), D is. a distance function or metric if:
- $D(p,q) \geq 0$ 
- $D(p,q) = D(q,p)$
- $D(p,z) \leq D(p,q) + D(q,z)$
