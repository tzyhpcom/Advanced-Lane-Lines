## Writeup

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

[image1]: ./output_images/calibration.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Original test1"
[image22]: ./output_images/calibration2.jpg "Road Transformed"
[image3]: ./output_images/yellow_mask.jpg "Yellow mask"
[image31]: ./output_images/white_mask.jpg "White mask"
[image32]: ./output_images/dir_mask.jpg "grad direction threshold"
[image33]: ./output_images/mag_mask.jpg "mag mask"
[image34]: ./output_images/sobelx_mask.jpg "soble x mask"
[image35]: ./output_images/sobely_mask.jpg "soble y mask"
[image36]: ./output_images/all_mask.jpg "combined"
[image4]:  ./output_images/perspective.jpg "Warp Example"
[image5]: ./output_images/windows.jpg "Fit Visual"
[image6]: ./output_images/final.jpg "Output"
[video1]: ./test_videos_output/project_video_output.mp4 "Video1"
[video2]: ./test_videos_output/challenge_video_output.mp4 "Video2"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Advanced-Lane-Lines-Demo.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]  
And the distortion-corrected image is as follows.  
![alt text][image22]  

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
  
I used a combination of color and gradient thresholds to generate a binary image.  
1. Yellow color mask  
![alt text][image3]  
2. White color mask  
![alt text][image31]  
3. Gradient direction threshold  
![alt text][image32]  
4. Gradient magnitude threshold  
![alt text][image33]  
5. Soble x threshold  
![alt text][image34]  
6. Soble y threshold  
![alt text][image35]  
Then I use the following code to combine them.
```python
combined = np.zeros_like(dir_binary, dtype=np.uint8)
combined[((gradx == 1) & (grady == 1)) & ((mag_binary == 1) & (dir_binary == 1)) & ((yellow_b==1)| (white_b==1)) ] = 1
```  
![alt text][image36]  

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is as follows. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[543,490],[748,490],[255, 687],[1050, 687]])
dst = np.float32([[255,0],[1050,0],[255, 687],[1050, 687]])
M = cv2.getPerspectiveTransform(src, dst)
Minv = cv2.getPerspectiveTransform(dst, src)
```
 A test image is shown bellow.   
![alt text][image4]  

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
  
  I use histogram to find left and right lane base, then get all points belong to each of them with searching window, finally numpy.polyfit will calculate line coefficients.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `Advanced-Lane-Lines-Pipeline.ipynb`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I did this in `Advanced-Lane-Lines-Pipeline.ipynb`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

[project_video_output](./test_videos_output/project_video_output.mp4)  
[challenge_video_output](./test_videos_output/challenge_video_output.mp4)  

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?  

For project video, lane lines get messy when the road is dirty and blurry, add yellow and white mask can do much better.
For challenge video, it's easy to lose tracks under bridge, average 25 frames can solve it.
For harader challenge video, it's really easy to fail. I have twisted the number of averaged frames, but still no good result. Maybe I should change mask parameters to detect lane line better.
