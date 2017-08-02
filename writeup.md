
**Advanced Lane Finding Project**

The goal of this project is to detect lane using images taken from front-camera. To accomplish that goal, I used OpenCV and numpy to do the following tasks:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/chessboard.png "chessboard detection"
[image2]: ./examples/undistort_output.png "Undistorted"
[image3]: ./examples/binary_combo_example.png "Binary Example"
[image4]: ./examples/warped.png "Warp Example"
[image5]: ./examples/color_fit_lines.png "Fit Visual"
[image6]: ./examples/radcurve.png "Calculate Radius of Curvature"
[image7]: ./examples/example_output.png "Output"
[video1]: ./result.mp4 "Video 1"
[video2]: ./challenge_result.mp4 "Video 2"


---

### Camera Calibration
#### 1. Built the "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world.
#### 2. Used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result

![alt text][image1]

### Pipeline (single images)

#### 1. Undistort image
I used `cv2.calibrateCamera` to correct these types of distortion. It returned the camera matrix and distortion coefficients which were used in `cv2.undistort` to get the corrected image.
![alt text][image2]

#### 2. Create binary image
I extracted saturation channel from HSL image, and red channel from BGR image. Then, I use `cv2.GaussianBlur` and horizontal `cv2.Sobel` filter, which is more resistant to noise, to find the edges.  All the outputs were then combined to enhance the detection power. Here's an example of my output for this step.
![alt text][image3]

#### 3. Perform perspective transformation

For simplify the implementation I hardcode the source and destinations points, and pass it to the function to calculate the perspective

```python
src = np.float32([[560, 475],[750, 475],
                  [1230, 720],[150, 720]])
dst = np.float32([[offset, offset], [img_size[0]-offset, offset], 
                  [img_size[0]-offset, img_size[1]-offset], [offset, img_size[1]-offset]])
```

I plot some verification to check if the lines are parallel at the warp images

![alt text][image4]

#### 4. Identify lane-line pixels and fit their positions with a polynomial

I used small sliding windows to detect pixels for the right and left lane, then fit those pixels with a polynomial. The number of sliding windows was 18 and the window margin was 30. A lane class was constructed for each lane to perform sanity checks. The goal was to make sure that the polynomials are smooth and matches the lane line as much as possible.

![alt text][image5]

#### 5. Calculate the radius of curvature of the lane and the position of the vehicle with respect to center.

In function `fillLane()`, I calculated the bottom left point and the bottom right point, and took the midpoint as the center of the lane. From that I calculated the difference in distance between the lane center and the image center.
I calculated the radius of curvature of the lane based on the following formula
![alt text][image6]

#### 6. Provide an example image of the result plotted back down onto the road such that the lane area is identified clearly.

In function `process_vid()`, I drew the lane onto the warped blank image, warped the blank back to original image space using inverse perspective matrix (Minv), then combined the result with the original image.

![alt text][image7]

---

### Pipeline (video)

#### 1. Advanced lane detection videos:

Here's the result https://youtu.be/9tPfcH1N2D4

---

### Discussion

The basic test was fine, the challenge and the hard challenge, does not perform well, mainly because shadowns and sharp curves where the right or left lane disapear from the image. I think more cameras are necessary to achieve satisfatory result for the challenges.
