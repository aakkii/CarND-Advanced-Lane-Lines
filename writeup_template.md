## Writeup Template

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.    

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./advance_lanelines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained undistorted result. This result is displayed in output of 5th code cell. 



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image:
Chessboard camera calibration results was saved. And I used it to undistort the test image. Example of undistorted image is displayed as output of 6th code cell. 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I have tried Gradient Threshold, Magnitude of Threshold, Direction of Gradient and combination of all three. Output of each of this function is avialable in 8th code cell, 10th code cell, 12th code cell and 14th code cell respectively. 

For color transform I have extracted S-Channel from HLS. Output of S-Channel is displayed in 16th code cell. 

In my final piple line I used combined binrary of L-Channel, S-Channel and Gradient Threshold.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `corners_unwarp()`, which appears in 17th code cell of the IPython notebook).  The `corners_unwarp()` function takes as inputs an image (`img`). I chose to hardcode the source and destination points in the following manner:

```python
src = np.float32([[590,450],[700,450],[1135,680],[225,680]])
offset = 100 # offset for dst points
dst = np.float32([[offset, 0], [img_size[0]-offset, 0], 
                      [img_size[0]-offset, img_size[1]], 
                      [offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 590, 450      | 100, 0        | 
| 700, 450      | 1180, 0       |
| 1135, 680     | 1180, 720     |
| 225, 680      | 100, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. Output of this function is available in 18th code cell.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First take a histogram along all the columns in the lower half of the image to find the peaks like below.

histogram = np.sum(binary_warped[binary_warped.shape[0]/2:,:], axis=0)

Then use peak as starting position and use sliding window search to find lane-line pixels and fit with polynomial. Output of this is available in output_images/sliding_window folder.

Once you know where the lines are skip the sliding window step. Instead just search in a margin around the previous line position. Output of this function is avialable in output_images/prev_sliding_window folder.

For processing the video, Line class is defined to hold interesting paramaters. Everytime, good fit is found it is added to the bestfit. Processing of the image happens using avarage of 5 best fits. If good fit is not found then do the sliding window search again. 


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 27. First it defines conversions in x and y from pixels space to meters. For this project, the lane is assumed to be about 30 meters long and 3.7 meters wide. After this it finds left and right lane curvature using forumla given for the Radius of curvature. 

http://www.intmath.com/applications-differentiation/8-radius-curvature.php


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 28,29 and 30 in function 'draw_lane' and 'draw_curvature'. Output of this is displayed in code cell 30 and code cell 33.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I have experimented with different thresholds and color transform to form my final piple line as described in point 2 above. Finally, I used combined binrary of L-Channel, S-Channel and Gradient Threshold.

Biggest problem I faced in this project while using sliding window search. Also, I am comparatively still new to Python. So this creates problem while using complex python functions. 

pipleline is failing when there is a line (not actual lane but some color difference) between the lane as given in challenge_video.mp4. This is where pipleline struggles to find actual lane since it is probably evaluting lane lines mistakely because of color difference within lane. To correct this I believe I should play around more with different channels and thresolds to form more robust pipeline. 

