# Histogram
The (intensity/brightness) histograms shows how many times a particular grey level (intensity) appears in an image.

![[Pasted image 20251010213316.png]]

$$
\text{x-axis}=\text{intensity}
$$
$$
\text{y-axis}=\text{number of pixels}
$$

> [!Info] Low contrast
> An image has low contrast when the complete range of possible value is not used

## Histograms of color images
For color images there three approaches:
- RGB color can be converted to a grey scale value by $Y=0.299R+0.587G+0.114B$ 
- or an individual histogram of red, green and blue
- or a 3-D histogram with three axes representing the red, blue and green channels, and brightness at each point representing the pixel count
# Histogram transformation
Method used in image processing to enhance contrast or distribution of pixel values (grey levels) in an image.

It is a *point operation* (do not consider neighbours ) $T(r_k)=s_k$, hence each pixel value $r_k$ is transformed to a new value $s_k$ using a transformation function $T$. 

![[Pasted image 20251011005542.png]]
# Histogram equalization (HE)
>[!Important]
Transform the intensity values so the histogram of the output image approximately matches the flat (uniform) histogram
# Contrast stretching
# Application