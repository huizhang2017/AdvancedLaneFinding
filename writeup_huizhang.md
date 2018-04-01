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

[image1]: ./my_p4_examples/undistort_image.jpg  "Undistorted"
[image2]: ./my_p4_test_images/test3.jpg "Road Transformed"
[image3]: ./my_p4_examples/binary_combo_example.jpg "Binary Example"
[image4]: ./my_p4_examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./my_p4_examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./my_p4_examples/example_output.jpg "Output"
[video1]: ./project_video_submit.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./submit_P4_AdvanceLanes_HuiZhang.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (from cell 4 to cell 8).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()` in the 3rd cell of the IPython notebook).  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
    src = np.float32([[600,465],
                      [380,600],
                      [930,590],
                      [730,465]])
    
    # four desired coordinates
    dst = np.float32([[300,200],
                      [300,650],
                      [900,650],
                      [900,200]])
```


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I plot the histogram of the thresholded binary image. The histogram was separated with left and right part, I constrained the positions of the peaks for both the right and left part.
According to these two peak positions, I set left and right windows, I totally used four windows for each part, the height of the windows is equal to the total rows of the images 
divided by four. And the width of the window is equal to 200 pixels, which is centered around the two peak positions. Then I searched the good pixels belong to the lane lines in 
each windows, at last I fitted the all the concatenated good pixels belong to the left or right lanes lines with a 2nd order polynomial separately. The result is like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I first convert the pixel distance to the meter. I notice that the width of the lane is about 700 and the height of the image is about 720 in my case, so the conversion between 
the real world space and pixel space followed the following relation: xm_per_pix = 3.7/700 (x-dimension) and ym_per_pix = 30/720 (y-dimension), unit in meter.
Then the curvature is calculated with the following formular: R_curvature = ((1 + (2*a*y_eval*ym_per_pix + b)**2)**1.5) / np.absolute(2*a), a and b are the coefficient of the polynomial,
and I calculate the radius of the curvature at the bottom of the image (that is y_eval). The details are shown in cell 11 in the fuction called 'radius_of_curvature'.

Because I have fitted the lane lines, so I can constrain the left/right x coordinate at the bottom of the image, then I can calculate the midpoint the lane. If I assume the camera 
is mounted at the center of the car, then the offset of the lane center from the center of the image (converted from pixels to meters) is the distance that the vehicle deviate
to the center of the lane. The offset is calculated in cell 12 with the function name 'offset_lane'.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane ar
usea is identified clearly.

I implemented this step in cell 17.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_submit.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
In this project, I first calibrate the camera, to get rid of the distortion effect caused by the optical lens in the camera or the position of the camera. Then I do the perspective
transform to the images and get a bird-eye view. This will help me to get a parallel lane in the image and will help me to fit the lane lines will 2nd polynomial. Before the fitting, 
I convert the RGB color to HLS color space, this will help me to get a binary image. Then I calculated the gradient in a gray image which will also help me to get a binary image.
Then combined with the result from the color selection and the gradient, I got a final binary image, which is supposed to have good pixel with lane lines. I used a sliding window to
concatnate the good pixels of the lane lines and used the 2nd polynomial to fit. I also calculate the radius of the curvature and the offset of the vehicle to the center of the lane
to double check the result of the lane finding. At last I use a pipeline to apply all these methods/procedure to a video.


The method I used strongly depends on the threshold I selected. If the enviroment is changed, such as there are many shadows which is almost parallel to the lane line. Because the gradient
will also appear around these shadows, it will hard to distinguish the gradient from the lane or from the shadows. One possible way is to use a masked region, which only contain 
the lane lines and just ahead of the vehicals.

Another problems is the calculation of the radius of the curvature. I find that in a few frames the radius of the curvature of the left lane line and right lane line is very much 
different, the difference is more than one orders of magnitude. I try to figure it out by adjust the perspective transform, because the parallel of the lane line is the key to solve
this problem. I also accept the suggestion from someone else in the 'Slack', that to include as more area at the horizon of the road as possible. It seems to be better, however, such
larger difference is still exist. I think this is caused by the number of good data points on the lane line. More data points will give more robust results. In fact I averge the fitting
result from the previous 10 frames as shown in cell 16 of 'pipeline' function, and I do not search the good lane points with sliding the window along the lane line for each frame, I just
use the result of previous frame to fit the lane lines for other frames (please see the cell 10, function name is 'look_for_lane_line2').
