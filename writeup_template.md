# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of several steps, generally contained in the main() function:
1. Convert the image to grayscale using the function grayscale()
2. Apply a gaussian blur to the grayscale image using function gaussian_blur()
   * *Kernel size: 7*
3. Apply a canny image filter to the output of the gaussian_blur function
   * *low_threshold: 60*
   * *high_threshold: 150*
4. Mask the output of the canny image filter to trapezoid region that covers the lane of the road (manually determined):
   * *(100,539)*
   * *(900,539)*
   * *(625,325)*
   * *(375,325)*
5. Take Hough transform of the masked canny image filter to get most probable lines from canny image transform points using parameters:
   * *rho: 1*
   * *theta: pi/180*
   * *threshold: 25*
   * *min_line_len: 1*0
   * *max_line_gap: 3*
6. Using the *lines* output from the hough_lines() function (I modified the given function to achieve this), the **getLineStats()** function is called to calculate the slopes and y-intercepts of all discovered hough lines, and sort them into a rightLine dictionary or leftLine dictionary based on positive vs. negative slope:
7. Outlier lines are then discarded from the leftLine and rightLine dictionaries using **discardOutlierLines()** by masking out any slope and intercept pairs if the slope is outside a one standard-deviation band from the mean.
8. The average y-intercept and slope of the remaining lines are calculated using the **getAverageLine()** function and converted to a line structure for use by **draw_lines()**. If no lines are detected by the function, an integer value of 0 is returned
9. The draw_lines() function is unmodified from what was provided, but is only called if there are lines returned by **getAverageLine()**

Refer to *"./test_images_output/testing_output.pdf"* to see a sample output for all 6 test images provided in this project

### 2. Identify potential shortcomings with your current pipeline

There are several shortcomings with this solution:
* Shock & vibration causing the camera angle to change
  * The masked region may no longer be accurate, may detect the wrong lanes. Current solution has no error detection to handle this case, and mask region is fixed rather than determined via an algorithm.
* Detection of curves
  * Current solution detects lane-lines in straight-travel over a fairly large mask region. Fitting lines to these curves will present some error to the algorithm, potentially causing the car to think the center of the lane is in the wrong spot.
* Width of lanes
  * The width of lanes may vary from road-to-road, causing the selected mask range of interest to be incorrect, or detect extra lanes which add error to the lane detection algorithm.
* Integrity of the painted lines, changes in environment
  * The parameters fed to the canny image filter appear to work well for the input images and test videos provided (excepting the challenge), but damaged painted lines may not be detected well using these parameters. Furthermore, changes in the environment such as fog or lighting can impact the ability of the algorithm to detect lanes.


### 3. Suggest possible improvements to your pipeline

Possible improvements to this pipeline could include:
* Adding error handling to support downstream autonomous vehicle control functions
  * i.e. if no lanes detected for a configurable time interval, throw error triggering manual control & driver notification
* Add capability to detect width of lanes (i.e. determine masking range algorithmically)
  * Adds fault tolerance for a camera that is loosened from its mounting by shock and vibration
* Have input from additional sensors into the algorithm to mitigate inaccuracies due to poor lighting, damaged painted lines, or other changes in environment
