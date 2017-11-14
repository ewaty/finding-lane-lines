# Advanced Lane Finding Project

---

![](lanes.gif)

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

[image1]: ./camera_cal/calibration18.jpg "Calibration Image"
[image2]: ./camera_cal/calibration18_dots.jpg "Chessboard Corners Detected"
[image3]: ./camera_cal/calibration18_cal.jpg "Undistorted Image"
[image4]: ./output_images/original.jpg "Original Image"
[image5]: ./output_images/undist.jpg "Undistorted Image"
[image6]: ./output_images/hue.jpg "Hue Filter"
[image7]: ./output_images/light.jpg "Light Filter"
[image8]: ./output_images/sat.jpg "Saturation Filter"
[image9]: ./output_images/sobel.jpg "Sobel Filter"
[image10]: ./output_images/sat2.jpg "Final Color Processing"
[image11]: ./output_images/mask.jpg "Region of Interest"
[image12]: ./output_images/persp.jpg "Perspective Transform"
[image13]: ./output_images/lines2.jpg "Sliding Window Search"
[image14]: ./output_images/lines3.jpg "Line Fitting"
[image15]: ./output_images/lines.jpg "Unwarping"
[image16]: ./output_images/full.jpg "Final Output"
[image17]: ./output_images/original1.jpg "Original Image"
[image18]: ./output_images/undist1.jpg "Undistorted Image"
[image19]: ./output_images/hue1.jpg "Hue Filter"
[image20]: ./output_images/light1.jpg "Light Filter"
[image21]: ./output_images/sat1.jpg "Saturation Filter"
[image22]: ./output_images/sobel1.jpg "Sobel Filter"
[image23]: ./output_images/sat21.jpg "Final Color Processing"
[image24]: ./output_images/mask1.jpg "Region of Interest"
[image25]: ./output_images/persp1.jpg "Perspective Transform"
[image26]: ./output_images/lines21.jpg "Sliding Window Search"
[image27]: ./output_images/lines31.jpg "Line Fitting"
[image28]: ./output_images/lines1.jpg "Unwarping"
[image29]: ./output_images/full1.jpg "Final Output"
[image30]: ./output_images/trapeze.jpg "Perspective Transform"
[image31]: ./output_images/warped.png "Warped Image"
[video1]: ./project_video_out.mp4 "Video Output"

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the fourth code cell of the IPython notebook "finding-lane-lines.ipnb".  

![alt text][image1]

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.
![alt text][image2]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image3]

### Pipeline (single images)

I will describe each step by comparing two frames. One of them is straightforward to process because the lane lines are clearly visible and the background in the region of interest (the asphalt) is uniform in color and light exposure. In the second frame it is more difficult to detect lane lines because they are obscured by shadows and the color of the asphalt is not uniform.

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to two of the test images. 

   |Easy frame                 |Difficult frame
:-::-------------------------:|:-------------------------:
Input|![alt text][image4]        |  ![alt text][image17]

I undistort the images using the undistortion matrix obtained in the previous step. The undistorted images can be seen below:

|Easy frame                 |Difficult frame
:-::-------------------------:|:-------------------------:
Undistorted|![alt text][image5]        |  ![alt text][image18]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cell 10 of "finding-lane-lines.ipnb"). I begin with converting the image into HLS color space. I then split the image into three binary images each representing a channel in the HLS image. Using thresholding I extract pixels containing information of ineterest to me in each image. The thresholded hue image highlights yellow lines. After thresholding the saturation image clearly shows the white lines. Finally, I apply a low threshold to the lightness image to extract areas with dark shadows, which have high saturation and will pass the threshold in the saturation image. I also apply the Sobel operator in the x direction to the greyscaled original image. This helps highlight lanes visible in the distance which are not well represented in the HLS thresholded images. I compose the final image according to the formula:

- Use saturation image as base
- If a pixel is 1 in the hue image, make it also 1 in the saturation image to add yellow lines
- If a pixel is 0 in the light image, make it also 0 in the saturation image to remove shadows
- If a pixel is 1 in the sobel image, make it also 1 in the saturation image to add faraway lines
- Apply mask to narrow down the area to region of interest

Each step and the final combined images can be seen in the following table:

   |Easy frame                 |Difficult frame
:-:|:-------------------------:|:-------------------------:
H    |![alt text][image6]        |  ![alt text][image19]
L    |![alt text][image7]        |  ![alt text][image20]
S    |![alt text][image8]        |  ![alt text][image21]
Sobel|![alt text][image9]        |  ![alt text][image22]
Combined|![alt text][image10]        |  ![alt text][image23]
Mask|![alt text][image11]        |  ![alt text][image24]
 

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective()`, which appears in the 7th code cell of the IPython notebook.  The `perspective()` function takes an image (`img`) as an input. The source (`src`) and destination (`dst`) points are defined outside the function.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[702, 460],[580, 460],[250, 705],[1050, 705]])
dst = np.float32([[900, 100],[450, 100],[450, 705],[900, 705]])
```

I obtained these points by finding a set of point on an undistorted image that roughly represented a rectangle, as shown on the image below:

![alt text][image30] 

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image31]

Perspective transform applied to the example images looks as follows:

   |Easy frame                 |Difficult frame
:-:|:-------------------------:|:-------------------------:
Perspective    |![alt text][image12]        |  ![alt text][image25]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To identify lane line pixels I implemented a sliding window search in cell 11 of `finding-lane-lines.ipnb`. This algorithm analyzes the bottom half of the image and finds the locations of two histogram peaks to determine the starting point for lane pixel search. Two windows of specific size are applied starting at the bottom of the image, centered on the previously found histogram peaks. Pixels inside the window are counted as lane pixels. Then the window moves up the image by its length. It slides left and right in a predefined range to find a region with most pixels. The center of that region is the new window center. The process is repeated through the whole width of the image.

Results of the sliding window search are shown below:

   |Easy frame                 |Difficult frame
:-:|:-------------------------:|:-------------------------:
Sliding window search   |![alt text][image13]        |  ![alt text][image26]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature and the position of the car with respect to the middle of the road are calculated inside the sliding window search function as shown in images below:

   |Easy frame                 |Difficult frame
:-:|:-------------------------:|:-------------------------:
Final result   |![alt text][image14]        |  ![alt text][image27]

To calculate the radius of curvature I fit a second order polynomial to the pixels identified in the sliding window search.

Then I calculate the radius using the following equation:
$R_{curvature}=\frac{[1+(\frac{dx}{dy})^2]^{3/2}}{|\frac{d^2x}{dy^2}|}$

To calculate the position with respect to the center of the road I calculate the points at which the line crosses the bottom horizontal axis of the image. Then I take an average of their x-cooridinates and compare it to the center of the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 11 of `finding-lane-lines.ipnb` inside the function `sliding_window_search`. To plot the results of previous steps onto the original image I used a function `inv_perspective` which warps images like `perspective` function but with parameters reversed. Here is an example of my result on test images:

   |Easy frame                 |Difficult frame
:-:|:-------------------------:|:-------------------------:
Final result   |![alt text][image15]        |  ![alt text][image28]

I then overlayed the original image with the image containing detected lane lines:

   |Easy frame                 |Difficult frame
:-:|:-------------------------:|:-------------------------:
Final result   |![alt text][image16]        |  ![alt text][image29]

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)


### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline mostly identifies lane lines correctly throughout the project video, but results from challenge videos are not as promising. Another problem is that despite correct identification of lane pixels the polynomial fit sometimes appears visibly incorrect.
I fixed this by rejecting frames where the distance between closest and furthest pixels is too little and ones where there is very little pixels through which to fit the line. In those cases I use lines from previous frames.
Occasionally though, the lines can wobble slightly. This could be fixed by implementing a procedure that remembers the curvature of the last five correctly detected lines and if the new line deviates significantly from the average of the past lines then it should be rejected.

