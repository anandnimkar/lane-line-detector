# lane-line-detector
A machine vision pipeline that annotates images (frames of an incoming camera feed) with detected lane lines and their curvature.

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

[image1]: ./output_images/chessboard_corners.png "Chessboard Corners Identified"
[image2]: ./output_images/undistorted_chessboard.png "Chessboard Undistorted"
[image3]: ./output_images/undistorted_lane.png "Road Image Undistorted"
[image4]: ./output_images/gradient_and_thresholded.jpg "Gradient and Threshold Example"
[image5]: ./output_images/warped.png "Warp Example"
[image6]: ./output_images/polyfit_lanes.png "Fit Visual"
[image7]: ./output_images/pipeline_output.png "Output"
[video1]: ./output_videos/out_project_video.mp4 "Video"

---

###Note: The code for each step below is included within the equally named sections in pipeline.ipynb

###Camera Calibration

####1. Computing the camera matrix and distortion coefficients

Steps followed:
* read in all calibration images
* generate object points, which are points (x, y, z=0)  that will map to the image points detected for each corner
* generate image points by detecting corners using `cv2.findChessboardCorners`
* obtain the camera matrix `mtx` and distortion coefficients `dist` for the camera after calibration using `cv2.calibrateCamera`

Below is an example of the annotated output of detected chessboard corners (image points): 

![alt text][image1]


###Pipeline (single images)

####1. Examples of distortion-corrected images.

Below is an example of a chessboard image undistorted using `cv2.undistort` and the camera matrix and distortion coefficients calculated

![alt text][image2]

Below is an example of a road image undistorted using `cv2.undistort` and the camera matrix and distortion coefficients calculated

![alt text][image3]

####2. Using color transforms and gradients to create a thresholded binary image.

Steps followed:
* convert color space to HLS
* Sobel x gradient threshold on the l channel and return values between a threshold
* Threshold S color channel
* Combine the Sobel x and S color channels into a binary thresholded image
* The values finally chosen type of gradient and threshold values was determined using trial and error

Below is an example of a thresholded binary image following the approach above: 

![alt text][image4]

####3. Perspective transforms

Steps followed:
* Within function `transform` use `cv2.getPerspectiveTransform` and `cv2.warpPerspective` to generate the warped image and transformation matrix `M`
* source (`src`) and destination (`dst`) were identified for the images using trial and error to find the best values that generated parallel lines for road images of straight lane lines. These values are hard-coded.
* In the full Image Pipeline at the bottom of './pipeline.ipynb', a region of interest is also cropped after gradient thresholding and before warping.

Below is an example of a transformed image without thresholding to demonstrate the effects of warping.

![alt text][image5]

####4. Identification of lane-line pixels and fitting their positions with a polynomial

Steps followed:
* In the Image Pipline at the bottom of './pipeline.ipynb', lane pixels were identified using a sliding window histogram. The "sliding window histogram" technique involves dividing the input binary-warped image into two halves and the image height into `n` windows. For each half, starting from the bottom of the image, sum the individual pixel values (all 255) for each x value to determine the histogram. Then find the argmax of the histogram and we have the start of the lane. The next step would be to move one window vertically up and search within some margin around that window using the same technique.
* Instead of always using the sliding window histogram technique, when prior line detections existed from the previous frame, then we searched around a horizontal margin from the previous fitted polynomial line for each of the left and right lanes.
* Once the lane pixels were identified using one of the two techniques above, a polynomial line was fit to the resulting pixels.

Below is an example of the left and right pixels (blue and red respectively) and the poynomial line in yellow annotated to a binary-warped image.

![alt text][image6]

####5. Radius of curvature of the lane and the position of the vehicle with respect to center.

In the Image Pipline at the bottom of './pipeline.ipynb', curvature and vehicle position offset from centre were calculated with the `set_curvature` and `set_base_pos` methods of the `Line` class. Additionally, averaging and aggregating of the values for each `Line` instance was performed in the `Pipeline` class in the method `process_frame` as per the code comments.

Curvature and Vehicle Position are expressed in meters. 

####6. Example image of result plotted back down onto the road such that the lane area is identified clearly.

Refer to the Image Pipline at the bottom of './pipeline.ipynb' for the implemention of the pipeline and `process_image` method.

![alt text][image7]

---

###Pipeline (video)

####Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/out_project_video.mp4)

---

###Discussion

####Problems / issues faced in this implementation. Where this will pipeline likely fail. To-dos to make it more robust.
* This pipeline depends heavily on color and binary thresholding techniques to isolate lane lines in an input image. For a common set of road conditions like those specified in `./test_images` or `./input_videos/project_video.mp4`, a specific set of gradient, color space, and thresholds techniques may work, but these quickly fall apart on the harder videos in the `./input_videos` directory. This is because a different set of techniques would be better optimized for those videos. Furthermore, a lot of trial and error was required to identify and adjust the parameters and techniques used to one set of road conditions. Therefore, I think this approach does not generalize well and to the real world.
  * Instead, a deep learning approach should be used that would likely work better to identify what a lane is. After all, our eyes do not see in binary thresholded warped images of the lane ahead of us, and we are still able to understand the concept of a lane. A good brain in front of a normal camera should be enough to identify lane lines and any other road condition.
* Further to the point above, if another car or obstacle presents itself in front of the camera, the identified techniques would almost certainly stop working and cause lane lines to scramble or lose detection.

To-dos
* The image pipeline at at the bottom of ./pipeline.ipynb can benefit from some additional validation checks between the left and right lanes, for example checking that the lines are parallel, checking that the distance between them is consistent, and checking that the calculated curvature is within expected limits. If any of these validation checks fail, we can discard the detection by calling the `discard_last_detection` method of the `Line` instances representing the left and right lanes. This uses the average of the previous n valid detections as the project lane line.
* Optimization algorithms and genetic techniques may be able to create a set of thresholding parameters that generalize well to all the input videos.


