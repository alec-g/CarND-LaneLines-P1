# **Finding Lane Lines on the Road** 

## Writeup Template

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/gray.jpg "Grayscale"
[image2]: ./test_images_output/edges.jpg "Edges"
[image3]: ./test_images_output/masked_edges.jpg "ROI Edges"
[image4]: ./test_images_output/final.jpg "Detected lane lines"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline for detecting lane lines in the provided image and video data consisted of the following steps. Firstly, the image is pre-processed, this includes converting it to gray scale and blurring it in order to improve the results for the next part of the pipleine. 

![alt text][image1]

After this step, i extract the edges from the input image, this is done using the cv2 canny function. I define some threshold parameter (min, max) that specifiy the range of intensities which define the detected edges. Next, i extract a region of interest (ROI) from the edge image, this is done using four points of a polygon to mask the an area of the image. 

![alt text][image2] ![alt text][image3]

Now that i have an image with edges of only the lane lines, i ran hough lines on the edge image with some pre defined tuning parameters to correctly detect the lines that corrospond to the lane lines, and to filter out any abnormal detections that staryed from the lane lines. Within the hough lines function outputs an image with lines drawn on it, however there is an extra step taken to process the output of hough lines.

The draw lines function performs averaging and interpolation on the output of hough lines to draw lane lines which span from the bottom of the image to the top of the ROI. This function works by first iterating over every line provided by the hough lines function, each line consists of four points specifiying its start and end location. For each line, we determine if it is a left or right lane based of it's slop, essentially if the slope is less than 0, it is a negative slop indiciating a right lane, and vise versa for the left lane. After segmenting the lane line, we calculate the slope and intercept and append this to an array specific to the lane (right or left), we also find the minimum y value which will be used for our "average lane line" which is simpl the detected line with the minimum y value. After we have this information for the left and right lane, we average the slope and intercept and use this to calculate the minimum and maximum x values for our averaged lane line. Given that the maxmium y value for our averaged lane line will be the bottom of the image, we now have four points to draw a line which corrosponds to the average of the detected hough lines. I take a step further and interpolate the end of the line (towards the top of the image) so that it can extend to the top of the ROI. I do this by using the equation of slopes and rearrange to find x given that we want the interpolated y value to be the top of the ROI. From here we simply input out values and the results is x_min_interp and y_min_interp which are the extended points for our line that reach the top of the ROI. A final feature of the draw lines function is that a history is kept of the lane lines, so if in one iteration the incoming frame did not detect any lane lines, the previous lane lines will be drawn, and this history is only updated the next time a lane line is detected.

![alt text][image4]


### 2. Identify potential shortcomings with your current pipeline

I have identified a few possible shortcomings for my implemented pipeline and listed them as follows:

1. The tuned parameters are specific to the provided images / video, so given new images, different lighting conditions, etc. the current parameters are unlikely to work.
2. When there is a line running perpendicular to the lane line, these are also detected, which can throw off the averaging.


### 3. Suggest possible improvements to your pipeline

Some improvements:

1. One improvement could be to clean the code for this pipeline, ideally the pipeline should be all contained within a class, i defined a class FindLanes to wrap the helper function, this was in order to keep a history state for the left and right lanes, but the entire pipeline should be wrapped in this class.
2. Based on the quality of the detected lane lines, an improvment could be to auto adjust the tuning parameters to minimise error, something like a machine vision solution.

