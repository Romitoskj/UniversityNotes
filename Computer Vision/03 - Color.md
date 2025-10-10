Color is a psychological property of our visual experiences when we look at objects and lights, *not a physical property of objects or light*.
It is the result of interaction between physical light in the environment and our visual system.

> Observed color is the result of interaction of light source with surface reflectance

# Trichromacy
Three numbers are sufficient for encoding colors: most people can match any given light with three primaries (exception: color blindness)
# Linear color spaces
- Defined by choice of three primaries
- The coordinates of a color are given by the weifhts of the primaries used to match it
- *Matching functions:* weights required to match single-wavelength light sources
- Mixing two lights produces colors that lie along a straight line along them in color space
- Mixing three lights produces colors that lie within the triangle defined in color space
## RGB
Primaries are monochromatic lights (for monitors, they correspond to the three types of phosphors)
# Color perception
- Color/lightness constancy
	- the ability of the human visual system to perceive the intrinsic reflectance properties of the surfaces despite changes in illumination condition
	- ![[Pasted image 20251010211510.png]]
- Instantaneous effects
	- Simultaneous contrast: background color affects perceived color of the target
	- Match bands
- Gradual effects
	- light/dark adaptation
	- chromatic adaptation
		- the visual system changes its sensitivity depending on the luminances prevailing in the visual field
		- e.g. walking into a building from full sunshine
	- afterimages
- High-level effects
	- color naming
		- our perception of color is different form the computer
# Uses of color in computer vision
- Color histograms for indexing and retrieval
- Skin detection
- Image segmentation and retrieval
- Building appearance models for tracking
- Judging visual realism