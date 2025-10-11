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

## Histogram equalization (HE)
Transform the intensity values so the histogram of the output image approximately matches the flat (uniform) histogram.

>[!Important]
> **Spreads pixels evenly across all brightness levels.**

$$
s_k=t(r_k)=(L-1)\sum_{j=0}^k\frac{n_j}{n}
$$
- $k=0,1,2,...,L-1$
- $L$: number of grey levels in an image (e.g. 255)
- $n_j$: number of times $j$-th grey level appears in an image
- $n$: total number of pixel in the image

This technique redistributes the pixel intensities so that the resulting image has more **uniform histogram** - meaning all intensity levels are used more evenly.
1. $\frac{n_j}{n}$: probability of gray level $j$ 
2. $\sum_{j=0}^k\frac{n_j}{n}$ is the **cumulative distribution function** up to level $k$ $\rightarrow$ It tells us the cumulative probability of intensity $\leq r_k$
3. Multiply by $(L-1)$ maps this cumulative probability into the range of gray levels $[0, L-1]$ 

![[Pasted image 20251011012021.png]]

## Histogram Projection (HP)
Assigns equal display space to every occupied raw signal level, regardless of how many pixels are at the same level. The raw signal histogram is "projected" into a similar-looking display histogram.

> [!Important]
> Spreads the *used gray levels* evenly across the display range, **ignoring how many pixels each level has**.

- *occupied grey level:* there is at least one pixel with that grey level
- *$B(k)$:* the fraction of occupied grey levels at or below grey level k, rises from $0$ to $1$ in discrete uniform steps of $1/n$, where $n$ is the total number of occupied levels.
$$
s_k=255\cdot B(k)
$$
Meaning that each occupied gray level $k$ in the original image is mapped to a new gray level $s_k$ that is uniformly spaced between $0$ and $255$.

> [!Note] 
> Every used gray level gets an equal slice of display space

![[Pasted image 20251011013823.png]]

# Contrast stretching
# Application