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

[image1]: ./camera_cal_results/corners_calibration3.jpg "Corners"
[image2]: ./camera_cal_results/res_calibration3.jpg "Undistorted"
[image3]: ./test_images_results_v2/Undistorted/res_test4.jpg "Undistorted 2"
[image4]: ./test_images_results_v2/Binary_colored/res_straight_lines2.jpg "Binary Example"
[image5]: ./test_images_results_v2/Warped/res_straight_lines2.jpg "Warp Example"
[image6]: ./test_images_results_v2/Lanes_rectangle/res_test3.jpg "Rectangle Example"
[image7]: ./test_images_results_v2/Lanes_rectangle/polynomials.png "polynomials Example"
[image8]: ./test_images_results_v2/Lane_final/res_test4.jpg "final result example"
[image9]: ./test_images_results_v2/Frame_by_frame/res_frame588.jpg "result example"

[image8]: ./examples/color_fit_lines.jpg "Fit Visual"
[image9]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

The complete code for the project is included in the jupyter notebook P2_Advanced_Lane_Finding_BW.ipynb.
---

### Camera Calibration

The code for this step is contained in the second code cell of the jupyter notebook P2_Advanced_Lane_Finding_BW.ipynb.  

The code for this step is based on the provided repository in the lecture Camera Calibration - Correcting for Distortion.

First I define the number of corners the chess board has (9 and 6). Then I define where the object points (corners of chessboard) shoud be in the real world. Then I loop over all the provided chessboard pictures with the following steps:

1. read in picture
2. convert to grayscale
3. identify corners
4. if corners were identified correctly the code adds them to the list imgpoints and also appends the previously defined object points to the list objpoints.
5. lastly the corners are drawn upon he picture and if necessary saved

After completing the loop I calculate the camera matrix and distortion coefficients necessary for undistorting images using cv2.calibrateCamera.

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I created a function that you find in the third cell lines 2-4.

Please note that color of the output image is not correct. I should have used cv2.cvtColor(final_image, cv2.COLOR_BGR2RGB) before saving the image. But I saved the images while creating the pipeline and focussed on the correct implementation more than having the color channels correct. Now I just don't want to rerun the code step by step and save all the intermediate results again.

![alt text][image3]

More examples can be found in the folder: test_images_results_v2/Binary_colored/

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines 7 through 31). First I used the sobel in the x direction, since lane lines should be primarely vertically. Additionally I used a color thresholds based on the black-white picture. Compared to the lectures I used a quite low threshold to since some part of the lanes appeared quite dark.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

To identify the 4 source points I used the test image straight_lines1.jpg. As low points I used the transition from car hood to the right and left lane line. The high points I eye balled based on the dotted lane line trying to take lane line points that are about 30m ahead of the car.

For the destination points I used the recommendation from the udacity Q&A platform to have the x position at 280 and 1000 pixels.

Based on the identified source and destination points I calculate the transformation matrix using cv2.getPerspectiveTransform (fifth code cell lanes 3 - 17). The matrix I pass to a function (second code cell line 34 - 36).

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To identify the lane line pixels I used the code we worked on in the lecture Advanced Computer Vision and put it in the function lane_finding (third cell).

When there is no previous polynomial I start with a calculation of the pixel density in lower half of the picture and calculate the two x position (left and right) with the highest pixel density. Those x positions are the starting point for a loop in which a stack rectangles on top of each other and extracting the lane line pixels within the rectangles. In each iteration I change the x coordinates of the rectangle to the mean of the identified pixels if the pixel threshold is reached.

![alt text][image6]

If there are already previously calculated polynomials I don't need the slow loop but search around the polynomials with a defined margin. All pixels that are within that margin are identified as new lane line pixels.

![alt text][image7]

Based on the identified lane line pixels from the rectangle or around poly search I fit the new second degree polynomials.

Last step is to plot the lane on picture and rewarp to the original image. This is done in the function plot lanes. The final result of this step looks like this:

![alt text][image8]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To translate my polynomial from pixels to meters I used the formula a fellow student proposed (mentioned in lesson Advanced Computer Vision). This is already done in the function fit_polynomial (lines 226 and 227).

The actual calculation of the radius and deviation is done within the function (plot_lanes). Here I also use the formula from the lecture to calculate radius from a second degree polynomial for a certain y value. As y value I use the bottom of the picture where the current position of the car is.

The calculation of the deviation from the line center is fairly simple. Based on the x values from the calculated polynomials at for the max y value of the image I can calculate where the middle of the lane is. This is compared to the middle of the picture.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

To be honest I had a few problems with using classes in python. I wasn't to familiar with the concept and especially the init function and class variables. This took a bit of time reading up on it. I am not to sure if my implementation is elegant but at least I made it work.

Regarding the pipeline I found that I spent most time in debugging the code and trimming thresholds. Since much of the code I used was from the lectures, the project and all the little errors helped me a lot in understanding what actually happens. For me it helped quite a bit extracting frame by frame from the video and focus on the more difficult parts with shadows and more curvature.

Even though the code works quite well on the project video there is still a lot of room for improvement. If I had more time I would like to integrate a confidence score for the polyfitting and use this confidence score as a weight when averaging the last frames or also rejecting some fits. Also comparing left and right polynomials and there distance would be necessary to make it more robust. Those shortcomings become obvious when trying the code on the challenge videos.

### Additional Ressources used

- Binary https://www.codespeedy.com/convert-rgb-to-binary-image-in-python/
- Draw polygon https://docs.opencv.org/master/dc/da5/tutorial_py_drawing_functions.html
- Udacity lectures
