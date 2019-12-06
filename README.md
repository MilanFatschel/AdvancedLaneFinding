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

[image1]: ./output_images/calibration1.png "Calibration"
[image2]: ./output_images/original2.png "Original"
[image3]: ./output_images/undistorted3.png "Undistorted"
[image4]: ./output_images/combined_grad_color10.png "Combined Binary Filter"
[image5]: ./output_images/perspective11.png "Perspective Transform"
[image6]: ./output_images/histogram12.png "Histogram"
[image7]: ./output_images/sliding_window13.png "Sliding Window"
[image8]: ./output_images/margins14.png "Margin Search"
[image9]: ./output_images/metrics16.png "Pipeline Result"
[video1]: ./project_video.mp4 "Video Result"


---

### Camera Calibration

LaneDetection.ipynb (Under Calibration)

I start by preparing "object points", which will be the (x, y, z) coordinates of the 
chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) 
plane at z=0, such that the object points are the same for each calibration image. 
Thus,`objp` is just a replicated array of coordinates, and `objpoints` will be appended 
with a copy of it every time I successfully detect all chessboard corners in a test image. 
`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the 
image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and 
distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion 
correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion-corrected image.

LaneDetection.ipynb (Under Undistort)

Original Image:

![alt text][image2]

Undistorted:

![alt text][image3]

#### 2. Color transforms, gradients to create a thresholded binary image. 

LaneDetection.ipynb (Under Gradient Direction / Color Filter Combination)

I used a combination of color and gradient thresholds to generate a binary image. This included the 
combination of filters such as sobel, magnitude, gradient and color (s channel) to retrieve an
image that highlights both sets of lanes nicely. This was mostly a trial and error process 
of tuning thresholds to get the correct effect.

![alt text][image4]

#### 3. Perspective transform

LaneDetection.ipynb (Under Perspective Transform)

The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and 
destination (`dst`) points.  I hardcoded the source and destination points as parameters 
since the camera and image sizes were static. I verified that my perspective transform was 
working as expected by drawing the `src` and `dst` points onto a test image and its warped 
counterpart to verify that the lines appear parallel in the warped image. The OpenCV function
warpPerspective() was used which creates a translation matrix based on given source points and
destination points.  

![alt text][image5]

#### 4. Identifiy lane-line pixels and fit their positions with a polynomial

LaneDetection.ipynb (Under Histogram)

I then used a histogram to find where the greatest peaks of pixels were in the image. 
The two peaks would show where the two lane lines were, which gave starting points:

![alt text][image6]

After finding the peaks, the sliding window approach was used to find the mean
pixel locations within a specified area. The function fit_polynomial() was then used 
to fit a polynomial throught these windows:

![alt text][image7]

Once the location of the lane-lines are known, the program can be optimized 
by using this information. Instead of re-searching the whole image, the program 
can look in the same location + or - some margin width since lane-lines are
continuous throughout a video.

The green represents the area that was searched:

![alt text][image8]


#### 5. Calculate the radius of curvature of the lane and the position of the vehicle with respect to center.

LaneDetection.ipynb (Under Measuring Cuvature of Lines)

The radius of curvature for each line was then calculated with the radius of 
curvature formula which can be found in the code. It was then necessary to convert 
these values to real world space by using meters per pixels based on real lane
measurements in each direction. Both of the curvatures were then averaged into 
one curvature.


#### 6. Final result with lanes marked.


![alt text][image9]

---

### Final Result (video)

Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Futher Challenges

The main issue with this code at the moment is that it cannot handle changes in lane
colors very well. We see this in the more challenging videos where there is an abundant
amount of shadows and change in brightness.The filters are set to relatively tight lane 
color constraints using the s channel. In order to impove this other channels or filtering
methods would have to be added to account for these offsets. The pipeline also does not 
handle extremely tight turning and cornering very well. More research has to be done to 
find a solution for this.
