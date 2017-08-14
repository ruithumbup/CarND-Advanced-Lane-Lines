## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/undist_check_calibration1.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/th_undist_test1.jpg "Binary Example"
[image4]: ./examples/warp_test1.jpg "Warp Example"
[image5]: ./examples/lane_output_test1.jpg "Lane-line Finding"
[image6]: ./examples/poly_output_test1.jpg "Fit Visual"
[image7]: ./examples/lane_to_world_test1.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced_Lane_finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at code cell `Thresholded Binary Images` section in the notebook).  Here's an example of my output from given test image `test1.jpg` for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_trans()`, which appears in the code cell `Perspective Transform` of the IPython notebook).  The `perspective_trans()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[200,700],[555,470],[720,470],[1080,700]])
dst = np.float32([[190,700],[190,100],[980,100],[980,700]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 700      | 190, 700        | 
| 555, 470      | 190, 100      |
| 720, 470     | 980, 100      |
| 1080, 700      | 980, 700        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used `convolve` method to find the lane-line pixels. The details can be found in function `find_window_centroids()` in code cell `Find Lane boundary` in the notebook.  

There are two cases. 

* When this function is called to detect lane-line pixels for a single image, it divides the image into 9 levels and will start from the bottom of image and search level by level for lane-line pixels

* When the function is called with continuous image frames within a video and once lane-line pixels were detected in previous frames, the function will rely on the detected lane-line pixels from previous frame on each layer to search new lane-line pixels. Note: This can contribute an error of shifting pixels from frame to frame due to noises. So when processing video's, I set a limit to allow this function to generate lane-line pixels this way for a maximum of 5 times.

Typical lane-line pixels findings are like this:

![alt text][image5]

Then I did some other stuff and fit my lane lines with a 2nd order polynomial. The details can be found in function `poly_fit_curves()` in code cell `Create Polynomial Fit of Curve Lane Lines`. The result kinda like this:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in function `poly_fit_curves()` in code cell `Create Polynomial Fit of Curve Lane Lines`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function `draw_lane_to_world()` in code cell `Draw Lane Lines Back to World Image`.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

#### 2. Provide a link to your challenge video output.

The model performs really well when it comes to turns or strong shades, except it cannot plot the lane lines back to video frames precisely. This is because the lane lines are in different region of this video frame then in previous video. Modify the coordinates chosen for perspective transform can easily solve this problem.

Here's a [link to my video result](./challenge_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

The lane-line pixel relys on maximum of 5 frames to detect lane-line pixels, if it goes beyong 5 frames, the lane-line pixel detector will start from the bottom of new frame and detect all over again. 

Every detected lane-line pixels will then used to generate poly-fitted coordinates. These coordinates will then go through a sanity checker based on the previous 5 reasonable findings. If the poly-fitted coordinates could not pass sanity check, they will be disgarded, and in the video it will show `NOISY` for detector status.

The video process pipeline then rely on the past 10 reasonable findings and use the average of them to plot the lane lines

The model seems robust to me except when there are a lot of pit holes on the road and makes the binary thresholded image blurry and thus the video process pipeline confused. I think putting weight on the sobelx threshold and adding sobel direction threshold may seem to improve this.  
