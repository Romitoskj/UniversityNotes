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
> An image has low contrast when the complete range  of possible value (*Display space*) is not used

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

> [!Info] Transformation:
> $$
> s_k=t(r_k)=(L-1)\sum_{j=0}^k\frac{n_j}{n}
> $$
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

>[!Info] Transformation:
> $$
> s_k=255\cdot B(k)
> $$

Meaning that each occupied gray level $k$ in the original image is mapped to a new gray level $s_k$ that is uniformly spaced between $0$ and $255$.

> [!Note] 
> Every used gray level gets an equal slice of display space

![[Pasted image 20251011013823.png]]
IR = Infra Red (other value scale)
## Plateau equalization (PE)
By clipping the histogram count at a saturation or *plateau value*, one can produce display allocations intermediate in character between in those of HP and HE.
### Problems with ordinary histogram equalization:
1. Over-enhancemente (too much contrast)
2. Noise amplification (especially in smooth regions)
3. Washed-out areas (loss of details in bright/dark zones)
That's because HE tries to make **all gray levels equally probable** even if some gray levels occur excessively due to noise or large uniform regions.

![[Pasted image 20251011204643.png]]

The PE algorithm computes the distribution not for the full image histogram but for the histogram clipped at a plateau value in the count.
- When that plateau value is set at 1, we generate $B(k)$ and so perform HP;
- When is set above the histogram peak, we generate $F(k)$ and so perform HE;
- At intermediate values, we generate an intermediate distribution denoted as $P(k)$

> [!Info] Transformation:
> $$
> s_k = 255 \cdot P(k)
> $$

- Compute a new histogram for all grey levels.
	- The plateau value $P$ is a threshold on the histogram height
	- If a gray level $k$ has $n_k > P$ pixels, we clip it down to $P$
	- if $n_k \leq P$, we keep it unchanged
- Perform an Histogram Equalization on the modified histogram

Doing so bright or dark regions that dominate the histogram no longer "squeeze" other tones and the resulting image has **better contrast** but **less noise** and smoother brightness transitions.
So, when you **clip** high peaks in the histogram and then equalize, you reduce their influence — this **softens** the equalization result, making it less extreme than HE but more adaptive than HP.

---
## Differences between transformations

| **Method** | **How display space is assigned**                                             | **Effect**                           |
| ---------- | ----------------------------------------------------------------------------- | ------------------------------------ |
| HE         | Space proportional to pixel count (based on Cumulative Distribution Function) | Strong contrast, sometimes too harsh |
| HP         | Equal space to each used gray level (ignores frequency)                       | Mild contrast stretch                |
| PE         | Space partly proportional to pixel count, but large counts are **clipped**    | Balanced contrast (intermediate)     |

# Contrast stretching
By stretching the histogram we attempt to use the available full grey level range. 
Do not help if all the display range is already used (better other techniques).

>[!Info] Transformation:
>$$
>s_k=255 \cdot (r_k-min)/(max-min)
>$$

![[Pasted image 20251011212813.png]]
min = 79, max = 136
# Application
- CT lung studies