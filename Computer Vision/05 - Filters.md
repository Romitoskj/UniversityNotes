## Neighbourhood operations
**Neighborhood operations** operate on a rectangle around a central pixel, for each pixel the outcome is written on the same position at the target image.

![[Pasted image 20251012160448.png]]
- Input = rectangle
- Output = pixel (central)

Simple neighbourhood operations:
- **Min:** set pixel value to the minimum in the neighbourhood
- **Max:** set pixel value to the maximum in the neighbourhood
# Spatial Filtering
![[Pasted image 20251007091225.png]]

### Spatial filtering equation form
$$
g(x,y) = \sum_{s=-a}^a\sum_{t=-b}^bw(s,t)f(x+s, y+t)
$$
- $f$ is the original image function
- $g$ is the output image function
- $w$ is the filter function or *kernel*
## Smoothing
Average all of the pixel in a neighbourhood around a central value $\rightarrow$ Blurring

>[!Warning]
>The greater is the kernel (filter) the more the output is blurred

![[Pasted image 20251007092126.png]]
#### Weighted smoothing
pixel closer to the centre is more important

#### Median Filter
used to remove noise from images

#### What happen at the edge
- omit missing pixel
- pad the image
- replicate border pixels
- truncate the image

> The filtering we have been talking about so far is referred to as *correlation* with the filter itself referred to as the *correlation kernel*

## Convolution
Convolution is similar but mirrored
![[Pasted image 20251007094716.png]]

#### Sharpening
seek to highlight fine detail (remove blurring and highlight edges)
based on spatial differentiation

##### Spatial differentiation
measures the *rate of change* of a function based on second derivative to construct *Laplacian filter*, getting a laplacian image. To sharpened image subtract the laplacian filter from the original one.
![[Pasted image 20251007101827.png]]
![[Pasted image 20251007101900.png]]
![[Pasted image 20251007101923.png]]

Simplified filter that do not need subtraction

#### Highboost filtering
Using sequence of linear spatial filters in order to get sharpening effect:
- Blur
- subtract from the original image
- add resulting mask to original image
#### $1^{st}$ Derivative Filtering
