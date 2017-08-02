##Advanced Lane Finding Project
The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # "Image References"

[image1]: ./output_images/cali.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/binary.png "Binary Example"
[image4]: ./output_images/src.png
[image5]: ./output_images/warp.png
[image6]: ./output_images/hist1.png
[image7]: ./output_images/fit.png
[image8]: ./output_images/eq.jpg
[image9]: ./output_images/lane.png
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd and 4th code cell of the IPython notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image. All thresholds are determined based on experiments with the testing images.

- The code for color thresholds is in the 8th cell of the IPython notebook. `HLS` color space is used. Pixels with s channel value in range [180, 255] or l channel value [205,253] are selected.


- The code for gradient thresholds is in the 10th and 11th cells of the IPython notebook. Pixels with absolute x direction gradient value in range [30, 255] and y direction gradient value in range [20, 150] are selected. In addition, pixels with gradient magnitude in range [40,200] and gradient direction in range [0.3,1.3] are also selected (a lightness filter is also applied here to filter out the too dark and too bright area, which means a selected pixel also has a l channel value in range [15,253]).


- The 13th cell of the Ipython notebook shows a function combining all thresholds to generate a binary image.

Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 6th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32(
    [[273,680],
         [1046,680],
         [585,455],
         [697,455]])
dst = np.float32(
         [[300,720],
         [980,720],
         [300,0],
         [980,0]])
```
The `src` coordinates are determined by drawing points on a straight lane which is show below.

![alt text][image4]

The `dst` points are arbitrarily selected to make sure the warped image is large enough for further processing.

This resulted in the following source and destination points:

|  Source   | Destination |
| :-------: | :---------: |
| 273, 680  |  300, 720   |
| 1046, 680 |  980, 720   |
| 585, 455  |   300, 0    |
| 697, 455  |   980, 0    |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I did this in the 17th cell of the IPython notebook by following these steps:

1. Take a histogram of the bottom half of the binary image (generated by applying thresholds). The position of two peaks are considered as the starting points for two lane lines. One example of warped binary image and the corresponding histogram is shown below.

   ![alt text][image6]

   2. From each starting point, sliding a window to locate area with lane pixels. There are 9 windows from the bottom to the top of the whole image. The center of upper window is decided by the lane pixels in the window right below it (specifically, it's the mean value of the lane points' x coordinates). All pixels in the 9 windows are considered as lane pixels.

   3. Fit the points from each starting point to a 2nd order polynomial, which represents the lane line.

      One example is shown below, where red points are the left lane pixels, blue points are the right lane pixels, green squares are the windows and yellow lines are the fitted 2nd order polynomial.

      ![alt text][image7]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The calculation of radius of curvature is shown in the 18th cell of the IPython notebook.

If the fitted 2nd order polynomial is `f(y)=A*y^2 + B*y +C`, the equation for radius of curvature is ![alt text][image8] as discussed in the class. To calculate the radius of curvature closest to the vehicle, the y value at the bottom of image (720) is used.

In addition, to convert the pixel to real distance in real-life, it is assumed one pixel in y dimension is 30/720 meters and one pixel in x dimension is 3.7/700 from the estimation based on `US governments specifications for highway curvature` as described in the class.



The calculation of the position of the vehicle is shown in the 20th cell of the IPython notebook.

There is no particular function for this.  `sanity_check()`  calculates the distance between detected lane and the vehicle (`self.line_base_pos`). The distance is just the distance between bottom of fitted polynomial and the center of image. Then `draw_measurement()` calculates the difference of distances to left lane and right lane (`offset`) and draw it on the original image.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 19th cell of the IPython notebook. It basically unwarp the detected lanes and draw them on the original image. The calculated radius of curvature and offset are also drawn on the top of the original image.

Here is an example of my result on a test image:

![alt text][image9]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Shadow Areas

It was much more difficult to detect lanes in the shadow area. Currently I am just using hard-coded threshold in HLS color space to eliminate the effects of shadows in the binary images. However, they are only guaranteed to work for this particular video. In general, the current pipeline should have a high possibility to fail when there is a sudden change in lighting conditions. A preprocessing that handles the shadows and super bright areas from images should be very helpful in my opinion.

Assumption of the plane ground

It was obvious that the detection of lane has relatively large error at the end of the video. I think the main reason for this is that we assumed a plane ground and it was not true at the end of the video (there is a bridge or something ahead...). In fact, plane ground is not really a realistic assumption even for highways, so some considerations about this should also be included in the lane detection to make the pipeline more robust.


