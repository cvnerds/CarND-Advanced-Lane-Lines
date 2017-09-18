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

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here is an example image:

![alt text][image2]

For convenience I created a class Camera, which stores the calibration data and which also offers a method called undistort. Once the calibration is done offline, the camera class can be reused to undistort an image.


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.
The thresholding is done in the function `binarize_rgb`. The individual thresholding steps are currently as follows:

* Use HLS colorspace and apply blur and gradient magnitude operators to the S channel.
* The gradient magnitude is thresholded (see function `filter_gradient()`).
* Separately the color in HLS space is thresholded. In HLS space S channel is chosen to be rather large while at the same time L (luminance) is set to a minimum of 30. This removes shadows.
* Two further color filters in RGB space are defined: `filter_yellow()` and `filter_white()`. These help to identify the lines.

The contributions of the thresholds can be seen in the following image (R=yellow/white, G=gradient, B=Saturation/Luminance)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I used my Camera class to store the perspective transform. This way the camera can also provide a convenience method called `warpPerspective()`, which is used to warp the regular image into a birds-eye view.

The src and destination points mentioned in the markup were used. I adjusted them slightly to achieve a subjectively slightly better result (note the +- adjustments):

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585-7, 460      | 320, 0      | 
| 203-3, 720      | 320, 720    |
| 1127, 720       | 960, 720    |
| 695+10, 460     | 960, 0      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

This is obviously the code from the Udacity courses. I tested out the convolutional approach, but found it more problematic in simple scenario. Maybe I didn't understand the purpose of the convolutional approach. I can imagine it forms a much better basis to do more advanced analysis useful for the challenging videos.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I'm doing this in function `line_metrics()`. For all y coordinates from y to image height, the corresponding x coordinates are computed. Then the x coordinates that are closes to the car indicate the distance of the lines to each other. Assuming the camera is in the center of the car, the center of the car corresponds to half the camera width. The distance of the camera image center to the lang center gives the offset.

To arrive at a metric value of the curvature, the constants from the lectures are used. However, the currently measured lane width in pixels is used to update the scale factor in x-direction.

For the curvature I typically get values in the thousands, especially for straight lines where the curvature converges infinity. This was a bit confusing at the beginning. The offset to the center is typically within 0.10cm, ball-park figure.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function `drawFinalImage()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

For starters, the whole pipeline builds on top of initially thresholding the image. This is always sensitive. To improve on robustness multiple modalities are used, which complement each other. Edges and colors are two such complementary approaches. It would be beneficial to research more such filters.

In terms of thresholding color, this is not too robust. We are assuming the lines look white or yellow. While white remains rather bright usually, it needs to be highlighted in bad lighting conditions. The challenging video shows how some sort of darker line exists, but it hides the white lines. As for the yellow lines, I think their appearance varies even more, which will lead to the color thresholds to fail.

In the really challenging video we can see extreme lighting conditions and reflections. This is hard. I would propose to normalise the grayscale image in the first place to maximise the contrast. This does not solve all the problems though.

Another problem is the line fitting, especially on dashed lines. If there are not enough samples then a dashed line curvature is often much fit badly. Filtering over time adds robustness, but it would be good if we could detect that we are in a scene with dashed lines and adjust our expectations. For example a dashed line does not have samples in every scanline due to the gap. Furthermore if we have a prior curve, we might specifically try to sample along the curve in a dashed fashion.

I think the line fitting could be more robust with RANSAC to reduce the effect of outliers.

The algorithm does not behave well in very steep curves, such as seen in the very challenging video. A problem is that the line search algorithm has a rather large vertical window. I can see an example frame (frame 60 of video harder_challenge_video.mp4), where the curve has almost 90° and the trees in the background are part of the binarised image. The convolutional approach seems to be working better here.

The other question is whether the perspective transform is stil working when you drive downhill. But I assume it does mostly. Compared to the thresholding and search problems the small change in street angle and road surface height, seems negligible.

There is also a scene where a motor driver comes from the side, which could screw up the algorithm. Here it would be beneficial to have object detection to track moving objects and filter them out.

I am left a little bit confused as to how to make it more robust using the hints and how to use some values of the suggested Line data structure. My future ideas are the following:

To intialise the tracking I need a good initial estimate. Hence I would require that a raw fit is established that has two almost parallel lines and similar curve coefficients. Going forward, the tracker does aready compute the mean over the last N=10 frames. However, I have not yet defined good methods to identify problematic scenario. I would like to be able to reset the tracking if no "good" pair of lines are found over some time. Furthermore, since we are assuming that lines are parallel, in theory it should be enough if either the left or the right line are found with high confidence. If such is the case, we should be able to shift the line with the same coefficients to the other side using a consant or previously learned distance of the lines or the previously filtered average base position.

Coming to think of it, we are also using a quadratic formula for the road curvature. Clearly, when we have serpentine type of roads a quadratic function is not expressive enough anymore and we would have to use a cubic model instead.