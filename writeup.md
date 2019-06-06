
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

[image1]: ./output_images/undistorted_calibration.jpg "Undistorted"
[image2]: ./output_images/undistorted_test.jpg "Road Undistored"
[image3]: ./output_images/threshold_test.jpg "Threshold Example"
[image4]: ./output_images/warped_test.jpg "Warp Example"
[image5]: ./output_images/final_test.jpg "Output"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

We can see that not all images are processed since the nuber of corners are not the same on all images (9,6)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I made a `undistort_images(img, obj_points, img_points)` function witch takes the image, objpoints and imgpoints calculated in a step before to obtain camera calibration information and generated undistorted images above. It's clearly visible that positions of cars on a picture shift slightly when we undistort the images.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I  built `threshholds(image)` function witch takes our image and splits it in hsl channels and gray scale.
After that I calculated the sobel gradient along x-axis, and performed binary threshholding on both sobel, and s channel of image. After that I used cv2.morphologyEx finction on my sobel threshold to close gaps between lines.  Finally combining my sobel and s channel threshold into final image and returning the result:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

My function  `perspective_transform(img)`  takes image as a input caluclates the transform matrix using `cv2.getPerspectiveTransform` function and `cv2.warpPerspective` to wrap the image using hardcoded `src` and `dst` points :

`src = np.float32([(200, 700),(600, 450), (700,450),(1080, 700)])`
`dst=np.float32([[350, 700], [350, 0],[950, 0],[950, 700]])` 
    

After the transformation I use the `region_of_interest(img)` function that masks the area:

`vertices = np.array([[(200,700),(200,0),(1200,0),(1200, 700)]],dtype=np.int32)`
In thresholded image witch results in more smoother recognition of lane lines

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Using the techniqe in the lessons for finding the lane line pixels i defined the `find_lanes()` using sliding window technique to find the lane lines and fit a polynomial using 9 windows from the bottom to the top of the image.



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Finally using the function `find_curves()` witch Convert from pixels to metre's fits polynomials to the left and right lane line points again (this time in real-world space instead of pixel-space) calculate the curvature as per the equation for radius from a 2nd order polynomial and calculate the lane deviation from the center 



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

`draw_lanes_on_image()` function witch draws a filled polygon using cv2.fillPoly(). This function then un-warps the image using the inverse perspective transform parameters Minv to show the completed result

![alt text][image5]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Finaly building the pipeline() function that calls all other functions and generates the video

Here's a [link to my video result](output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline is very slow but performs quite well, minor jittering on some parts due to surface and color transition witch could be better handled with better thresholding techniqes and error correction. It fails on diferent surface areas and other vertical lines. I tried compensating using ROI but on the other videos that doesn't help. 

