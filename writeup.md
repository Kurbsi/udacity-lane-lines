# **Finding Lane Lines on the Road**

## Writeup Template

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.


1. Prepare for Canny edge detection: In order to properly use a Canny edge detection algorithm, we need to provide a feasible image. This means we need to remove as much image noise as possible.

Therefore we have to apply two steps: First, we convert our color coded iamge to gray scale. This is done with the OpenCv function
```cv2.cvtColor()```.
The function accepts the original image and the color code to which the image should be converted. It returns than a copy of image in the respective color space. Second, we apply Gaussian blur to get rid of the noise and smooth it therefore further. To do so a image convolution technique is applied with a Gaussian kernel. The kernel size needs to be an odd number (3x3, 5x5, 7x7,...). 5x5 proved to be most efficient kernel size without losing to much information. OpenCV offers therefore the method ```cv2.GaussianBlur```.

2. Next, we apply Canny edge detection. Canny edge detection uses the gray scaled and blurred image, as well as an lower and a upper threshold and returns an image where pixel which seem to be part of a edge are made white and the rest is made black. Internally the algorithm detects edges based on the color gradient between pixels.

The Canny edge detection algorithm works in a two step process. First, pixel with a gradient above the threshold are detected and those with a gradient lower then the lower threshold are rejected. Pixel with a gradient between lower and upper threshold are accepted, if they are connected to a strong edge.

A threshold ratio of 1:2 or 1:3 is recommended, I used `low_threshold = 50` and `high_threshold = 150`.

3. Since we are only interested in a specific area where we know that there are lanes, we mask the image. Our region of intereset excludes the sky which is the upper half of the image and the left and right side. I even go further and only mask a triangle to compensate image dilatation. This reflects the fact, that objects appear smaller the further they are away from where the image was taken.

4. Last, we need to detect real lines in our binary image. Therefore we use the OpenCV function `cv2.HoughLinesP()`. This function returns a array of lines which fullfill the criteria.

5. Finally, we need to decide which lines are actually lane lines. I make this decision based on the slope. Since we can view an image as part of a coordination system with its origin on the top left corner of the image, the x-axis is pointing to the right where as the y-axis is pointing to the bottom. With this information we know, that a positive slope indicates the right lane line and a negative slope indicates a left lane line.

In order to draw the lines on there appropriate place, we need to calculate the average over the respective lines. Two different approaches are presented. In ```extrapolate_lines_polyfit()``` we use a running average over the beginning and end of each line and polyfit the averge slope and intercept, in ```extrapolate_lines_fitline()``` we use ```cv2.fitLine()``` to calculate the respective fit over the lines found by Hough. For both methodologies we calcualte the resulting line by using a fixed y-value for the start and end pixel of our line and calculate the respective x-value with $y=mx+b$, where m = slope and b = intercept.

Both apporaches lead to pretty similar results.


### 2. Identify potential shortcomings with your current pipeline


In this rather crude approach of detecting lines are apparent:
- We restrict the region of interest pretty hard. If, for example, we would be in a curve and therefore the expected lane is not right in fron of us we would not detect lines this accurate anymore. The same holds true for driving up- or downhill where the horizon would appear on a completly different pixel row on the image.
- Also we have pretty strict values for Canny edge detection. For lines which have a color more similar to its sourroundings this might lead to undetected lines.


### 3. Suggest possible improvements to your pipeline

As stated in 2. a couple of issues are apparent in our algorithm. Possible improvements are to create a more flexible region of interest where for example the sky is detected based on specific colors and therefore excluded. Also the street could be detected in a color based way, and therefore a different tilt angle of the camera compensated.

Another improvement could be made in which lines are actually used for calculating the resulting line. After separating the left and right lines, these list should be filtered and outliers should be removed before calcualting the average.

