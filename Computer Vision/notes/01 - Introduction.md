# What is computer vision?
Given an image or more, extract properties of the 3D world

# Challenges
- illumination
- object pose
- clutter
- occlusion
- intra-class appearance
- viewpoint 
- scale
- deformation
- linear and aerial perspective
- shading
# Grouping cues
- similarity
- "common fate"
# Course overview
1. Early vision: Image formation and processing
	- How images are formed - physics
		- Cameras
			- what a camera does
			- how to tell where the camera was
		- Light
			- how to measure light
			- what light does at surfaces
			- how the brightness values
		- Color
			- the underlying mechanisms of color
			- how to describe and measure it
	- Representing small patches of image
		- establish correspondence between points in different images , so we need to describe the neighbourhood points
		- sharp changes are important in practise (**edges**)
		- representing texture by giving some statistic of the different kinds of small patch present in the texture.
	- Filter outputs
		- form a dot product between a pattern and an image, while shifting the pattern across the image
	- Texture
		- object are distinguished by their texture
		- texture are represented with statistics of the filter outputs
		- object with different textures can be segmented
		- variation in texture is a cue in shape
2. Mid-level vision: Grouping and fitting
	- Finding coherent structure so as to break the image or movie into big units
		- segmentation:
			- breaking images and videos into useful pieces
		- Tracking:
			- Keeping track of a moving object through a long sequence of views
3. Multi-view geometry
4. Recognition
5. Advanced topics
