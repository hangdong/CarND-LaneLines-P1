#**Finding Lane Lines on the Road** 

##Hang's Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[yellow_img]: ./screen_cap/yellow.png "yellow color pic"
[1]: ./screen_cap/1_gray.png "gray scale coversion"
[2]: ./screen_cap/2_blur.png "gaussian blur image"
[3]: ./screen_cap/3_edge.png "canny edge detection"
[4]: ./screen_cap/4_region.png "apply region filter"
[5]: ./screen_cap/5_lines.png "line extraction"
[6]: ./screen_cap/6_avg_line.png "compute slope/intersect"
[7]: ./screen_cap/7_draw_over.png "draw over the original frame"

---

### Reflection

###1. Image processing pipeline for lane lines detection

My pipeline consisted of 7 steps, carried out on a per image frame basis. 

First, a given image is converted from color input to gray scale. 

![alt text][1]

Next a gaussian blur with kernel size 5 is applied. 

![alt text][2]

Followed by canny edge detection, 

![alt text][3]

A polygonal filtering region is applied to the detected edges to remove edge detect above the horizon and near left/right side of the image. 

![alt text][4]

Then the remaining edges are feed into hough transform to detect best-fit lines. It worth noting that the parameters are tuned to extract relative long lines (at least 150 pixels long) - under the assumption that lane marking from some of the longest lines on the image, and allowed for a large gap (up to 100 pixels) since there is large gap for lane divider markings.  

![alt text][5]

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by calculating the slope and intersect for each detected line from the two points. On inspection of the calculated values, it's clear that a relative consistent positive and negative slope represent the left and right lanes. So I grouped the resulting parameterized lines by positive and negative slope value, then computed average slope and intersect  from each of the two groups; given we want to draw and extrapolate line near bottom of the image to near horizon, two points are calculated for each of 2 average lines with fixed y1 and y2 that crosspond to bottom of image and about halfway up in image coordinates. 

![alt text][6]

Finally, the resulting averages lines are drawn over the original input image frame. 

![alt text][7]

###2. Identify potential shortcomings with your current pipeline

In the video I noticed minor jumps from frame to frame on lane detection location. 

The largest flaw int the current pipeline is how the lanes are estimated through averaging; the assumption that the entire image will only be consisted by 2 lanes marking separated by positive and negative slope breakdown under a few different scenarios:
1. multiple parallel lines from adjacent lanes are detected
2. when vehicle orientation with respect the lane changes (e.g instead parallel, intersect the lane or at an angle). 
3. when there is other strong lines features in image (e.g wall/edge from building)

It is also possible that sometimes there is or left or right lane marking (e.g due to deteriorated paint marking), in which case the average math breakdown due to dividing by zero error. The particular this case is accounted for by an explicit corner check in the code. Another divide by zero can show up in a rare corner case where the average slope happens to be zero as well. 

###3. Suggest possible improvements to your pipeline

For the lane detection frame to frame jump, a slide window filter could be used frame to frame to smooth out the lane location change. 

For the lane location detection corner cases, one possible improvement is to make use the color of the lane (yellow and white) information directly, which will likely improve detection accuracy. Current the canny edge detection technique largely ignores this info. 

Finally, for the potential zero slope corner case, I could switch the lane average math to polar coordinate rather than current Euclidean. 
