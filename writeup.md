
**Advanced Lane Finding Project**

Note: this is Udacity Nano Degree project, please refer to [Udacity Repository](https://github.com/udacity/CarND-Advanced-Lane-Lines.git) for the project information and requirements.

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position. 

---

### Camera Calibration

#### 1. The calibrate_camera() function uses the `cv2.findChessboardCorners()` and `cv2.calibrateCamera()` funtion to implement the camera calibration.


In the function calibriate(), the process as below:

* Prepare the object points for chessboards points
* Create lists to hold the 3D object points and 2D corner pioints.
* Search corners for all calibration images.
* Calibrate camera and get calibration parameters.
* Write the calibration paramter to a pickle file *camera_cal.p*.

Here is an example of how the camera calibration parameters can be used to undistort a raw image of a chessboard:

![undistort_image](./output_images/undistort_example.png)

---

### Create a lande detection pipeline

All the steps described bellow will be used in a Pipeline class that will be used to detect lanes. The purpose of each step will be demonstrated via examples and the processed results will be displayed as images. Sometimes different implementations will be discussed to achieve better results in different scenarios.


#### 1. Apply undistortion on a raw image captured by the same camera

Camera calibration parameters should have been calculated by `calibrate_camera()`, and here is how it looks like to apply undistortion with the parameters on raw images collected by the same camera:

![undistort_raw_image](./output_images/undistort_raw_image.png)

#### 2. Threshold the image.

There are many ways to threshold an undistorted image, either in color space or using gradient decent. I have included two ways in this project: HLS space + Gradient thresholding and yellow and white thresholding.

##### 1. HLS + Sobel gradient thresholding

This is a combination of color and gradient thresholds to generate a binary image.
in color: it use a HLS's s channel, for gradient, use x direction gradient on l channel.

![HLS_Sobel_thresholding](./output_images/HLS_Sobel_thresholding.png)

##### 2. Yellow and white thresholding

This is a combination of yellow-thresholding and white-thresholding to generate a binary image. Yellow high/low and white high/low need to be manually adjusted for best performance.

![Yellow_white_thresholding](./output_images/Yellow_white_thresholding.png)


#### 3. Perspective transform

To implement perspective tranform, first need get the tranform parameter `M` and `Minv`.
To get the thransform parameters the `cv2.getPerspectiveTransform` function uses manully adjusted 4 source pionts and 4 destination points. I tested the parameters on `test_images/straight_lines1.jpg` using `perspective_transform()` function. The wraped binary image using the transform parameters is as following:

![warped_example](./output_images/warped_example.png)


#### 4. Find left and right lanes and apply polynomial fit

The function `find_lane_sliding_window` uses a sliding window approach to find the lane-line pixels. 

![test6_lane_detected](./output_images/test6_lane_detected.jpg)

After getting the lanes, use `np.polyfit()` to get polynomial paratmeters, this is done in `get_polynomial` in the example. To visualize the search result and fit polynomial, use `draw_lane_fit()` function to fill the gap between the two polynomial lines. Here is an example:

![test6_lane_painted](./output_images/test6_lane_painted.jpg)

#### 5. Calculate the radius of curvature of the lane and the position of the vehicle with respect to center.

The cacualtion is done in the `calculate_curvature()` function and `calculate_offset()` function.

* To get curvature after getting the polynomial parameter, use the function R = (1+(2Ay+B)^2)^3/2 / (|2A|)

* For the offset, it is similar, tranfer pixel to meter, compare the lane center with picture center to get offse.


#### 6. Plot lane area and display the radius and offset.

Use `draw_lane_info()` to draw the curvature and offset on top of the undistorted image.

![processed_test4](./output_images/processed_test4.jpg)

### Apply the pipeline to project video

#### Sanity check

Sometimes lanes are not detected correctly, I got wrong polynomial fittings like bellow:

![test6_lane_info_displayed](./output_images/test6_lane_info_displayed.jpg)

In order to solve such problems, I checked the distance between the left and right polinomial lines at the top, middle and bottom of the image. If they fall out of a hand picked range, I ignore such fitting and just retrieve most recent valid fitting.

#### Smoothing

To stablize the detection, I made use of the `smooth_number` to just get the average of the most recent 23 valid detections. The bellow is the processed video for the complete project video.

<a href="http://www.youtube.com/watch?feature=player_embedded&v=EOUp2sg4StQ
" target="_blank"><img src="http://img.youtube.com/vi/EOUp2sg4StQ/0.jpg" 
alt="project video demo" width="240" height="180" border="10" /></a>

---

### Discussion

#### Generalize the algorithm

I can modify the Pipeline a bit to work reasonably well on the challenge video:

<a href="http://www.youtube.com/watch?feature=player_embedded&v=WoGBw4sw1e4
" target="_blank"><img src="http://img.youtube.com/vi/WoGBw4sw1e4/0.jpg" 
alt="challenge video demo" width="240" height="180" border="10" /></a>

However, the algorithm struggles to detect lanes in a more challenging lighting/road surface condition even after trying a more dynamic yellow-white thresholding and a smarter sliding window lane detection approach:

<a href="http://www.youtube.com/watch?feature=player_embedded&v=AVEcr9qhisc
" target="_blank"><img src="http://img.youtube.com/vi/AVEcr9qhisc/0.jpg" 
alt="more challenge video demo" width="240" height="180" border="10" /></a>

Like the thresholding method, the sanity check is also hand picked, which will fail in general purpose. Especially when the we have a very sharp turn. A more robust model should be developed. I have started exporing some automatic thresholding for the **more challenge video**. Maybe a nueral network based approach is worth a try. But to apply the algorithm in the real world, where efficiency is as important as correctness, we should measure the computation requirement as well as the power consumption. A really good solution should be a combination of pure computer vision approach maybe plus a really light weight neural network, which does not require a heavy duty GPU.

#### Extreme weather condition

So far I have only tested different lighting, road conditions. Other parameters like raining or snowing may terribly affect the performance of the system. I could find more dataset under different weather condition to get a more robust model.

