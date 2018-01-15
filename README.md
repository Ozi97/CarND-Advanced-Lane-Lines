## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Test Image"
[image3]: ./output_images/undistorted.jpg "Undistorted"
[image4]: ./output_images/binary.jpg
[image5]: ./output_images/ROI.jpg
[image6]: ./output_images/Pers_lane.png
[image7]: ./output_images/l_l.jpg
[image8]: ./output_images/r_l.jpg
[image9]: ./output_images/lane_lines.jpg
[image10]: ./output_images/lane.jpg
[image11]: ./output_images/finall.jpg
[image12]: ./output_images/img_w.jpg
[video]: ./p4.mp4 "Video"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the Camera Calibration and Distortion Correction section of the IPython notebook located in "ALane.ipynb".  

Using `cv2.findChessboardCorners`, the corners points are stored in an array `imgpoints` for each calibration image where the chessboard could be found. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  

`objpoints` and `imgpoints` are used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. Then `cv2.undistort()` function is used to obtain the following result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Distorted image:

![alt text][image2]

Distortion-corrected image:

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.


A combination of color and gradient thresholds was used to generate a binary image. Color theresholding for yellow and white lanes was done in hsv space then HLS. S and L channel was used. Gardient was computed along x direction. The code for these filters is present in the heading `Binary Image`

![alt text][image4]

After Selecting Region:

![alt text][image6]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for  perspective transform is included in a function called `transformation_matrix()`. The function takes as inputs an image (`img`). After having poor results with lane coordinates, I decided to use predefined coordinates.

```python
 src = np.array([[585, 455],
                [705, 455],
                [1100, 680],
                [250 , 700]], np.float32)

        dst = np.array([[300 ,100],
                [1000, 100],
                [1000, 720],
                [300, 720]], np.float32)
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 585, 455      | 300, 100        |
| 705, 455      | 1000, 100      |
| 1100, 680     | 1000, 720      |
| 250, 700      | 300, 720        |

Example:

Undistorted image with source points drawn:

![alt text][image6]

Warped image:

![alt text][image12]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the `Identifying lane-line pixels` heading the `get_lane_px` searches for lane pixels using window size of 100px and base coordinates of a lane.

```
    window_size = 100
    
    right_base = right_base[0]
    
    if(right_base > window_size*2):
            window_low = right_base - window_size
    else:
            window_low = 0

    window_high = right_base + window_size
    
    # Define a region
    window_r = image[:, window_low:window_high]

    # Find the coordinates of the white pixels in this region
    #print(window_r[:100])
    right_x, right_y = np.where(window_r == 1)

    # original location
    right_y += (window_low) 
```

The output looks like this:

![alt text][image7]
![alt text][image8]


The `draw_curves` uses a second order polynomial to fit lane pixels.

Result:

![alt text][image9]

Then using the lane lines `draw_lanes` plots the lane.

Result:

![alt text][image10]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature and the position of the vehicle with respect to center is calculated upon calling the `get_curvature` in sixth code cell 

For a second order polynomial f(y)=A y^2 +B y + C the radius of curvature is given by R = [(1+(2 Ay +B)^2 )^3/2]/|2A|.

The distance from the center of the lane is calculated by measuring the distance to each lane and calculating the position assuming the lane has a given fixed width of 3.7m.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image11]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

* The video output has the right lane boundary is skewed in some frames. This might be due to missing lane markers after prespetive transform is calculated, which offsets the polymonial fit.

* The pipline need to be optimized as there are alot of repetitive calculations.

* Laplacian or other gradients could be used to improve line detection.

