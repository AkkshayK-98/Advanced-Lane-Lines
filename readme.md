# **Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points 

---

## Camera Calibration

### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here, I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function.

#### Before
[Distorted](test_images/straight_lines1.jpg)
#### After
[Undistorted](output_images/undist.jpg)


## Pipeline (single images)

### ROI

After undistorting, the next step is to find the region of interest. I believe that it is easier to process a little part of undistorted image. The coordinate of corner points of ROI is as follows:

    Point             Coordinates
    left top           580, 450
    right top          730, 450
    right bottom      1200, 672
    left bottom        150, 672

[ROI](output_images/roi.jpg)

### Perspective Transform

The code for my perspective transform includes a function called warper(). The warper() function takes as inputs an image (img), as well as source (src) and destination (dst) points. I chose the hardcode the source and destination points in the following manner:

First, I use function getPerspectiveTransform() to calculte the transform matrix. src is the region to be warped, and of course, this region is ROI. dst is the position to put the warped image, I choose an image with 1280 * 720 just as the origin image.

    M = cv2.getPerspectiveTransform(src, dst)

Then, I'll use the cv2.warpPerspective() function to transform the ROI

    warped = cv2.warpPerspective(img, M, (1280, 720), flags=cv2.INTER_LINEAR)
    
[Perspective Transform](output_images/perspective_transform.jpg)
    
### Binary

In this step, I'll generate a binary image of the warped ROI. The lane lines are almost vertical, thus, I use the gradient in the x direction (sobel x) to detect them. However, when something unexpected appears in the image, like lighting change, it may be recognized as lane lines by mistake, like in the challenge video for the first project. To solve this problem, I used the hue and lightness channels in HLS color space to perform and operation. This is done in the function xhl_thresh().

[Binary](output_images/binary.jpg)

### Histogram

Before finding the pixels of lane lines, it will be easier to find the region where the lane lines are most likely to appear. Calculate the histogram is a way achieve this. The two peaks are x coordinate of most pixels of lane lines. 


### Sliding Windows

This step slides a window to scan the most likely region for lane lines, as suggested by the histograms.

[Undistorted](output_images/sliding_windows.jpg)

### Fit Polynomial

Then I fit a polynomial through the sliding windows to detect the lane lines.


### Curvature

The radius of curvature is calculated by following equation:
    
    R = [1 + (2Ay + B)^2]^3/2 / |2A|


### Rewarp

The final step is to rewarp the above result to the undistorted image. 

[Lane Warp](output_images/lane_warp.jpg)

[Result](output_images/result.jpg)

---

## Pipeline (video)

### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](output_images/project_video_xhl_output.mp4)

---

## Discussion

### Thresh
I combine sobel-x with hue and lightness to transform the warped image to binary. Each paremeter should given the min_thresh and max_thresh. It is not easy to find the best combination of channels to use, and the min and max thresholds for each channel. Also, due to the lack of data, we do not know whether this combination would work for every scenario. 

