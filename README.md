**Advanced Lane Finding Project**

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

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"


## Camera calibration
Images of cameras are distorted. In order to undistort the images I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/p.png)

## Undistortion
With the image and object points I was able to calculate the distortion coefficients and calibrate the camera.
Then I undistorted all the other images.
Here is a great example where the difference is obvious:

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/u.png)

Next, I undistorted the image which we will work with.

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/u2.png)

## Thresholding
Our next step is to Manipulate the image to see the lines. I used a combination of color and gradient thresholds to generate a binary image. Here I converted the image from BRG colorspace to HSL. The S Channel of the HSL color space showed the best results within a threshold between 200 and 255. However when the distance to the car increased the S Channel of HSL lost track of the lines. This is why I also used the xGradient of the image. Here I used a threshold with a minimum of 35 and a maximum of 255. Then I combined both.
On the left image below the green pixels are from the gradient and the blue pixels are from the S channel of HSV.
On the right image you can see the binary combined image.

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/c.png)

## Perspective transformation

First I defined calibration box coordinates in the original that are shaped like a Trapezoid.
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/4.png)

Then I computed the perspective transform, M, given source and destination points.
My next step was tp warp an image using the perspective transform, M:

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/t.png)

## Lane detection without information from previous images
Peaks in a histogram of the lower half of the image show were lane lines are.

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/h.png)

Find the peak of the left and right halves of the histogram.
These will be the starting point for the left and right lines.
Then, I defined Windows which slide through the image horizontally and meassure the white pixel density. The area with the greatest white pixel density on the left half and right half should describe the left and right lane line and will be colored blue and red.

Then I fit two polynoms of order two through the red and blue colored pixels.

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/o.png)

## Lane detection with information from previous images
Since we know where the lane lines of the previous images were the position of the lane line in the next frame should be relatively close. That's why I will search for the new line close to the the polynom of the last image frame. When the new line is found I will fit two polynoms of the order two again and highlight the white pixels with a maximum distance of 60 pixels next to them.

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/w.png)

## Projecting the found lane lines space between the lines onto the original image
Now the lane lines in the transformed birds eye view are found and we need to reverse the transformation 
to draw to found lines onto the original image.
I used the inversetransformation. Then I combined The original image with the newly found area and lines:

![](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/output_images/l.png)

## Curvature
I'll want to measure the radius of the curvature at the very bottom of the image: y-value= 720.
Because the measurement of the curvature should be in real world meters rather than pixels I defined conversions in x and y from pixels space to meters.
X converstation: 4*12/720 where 4 is the number of dashed lines in the selected trapezoid and 12 is the number of meters between dashed lines start. Consequently 4*12 is the distance of real world meters and 720 is the distance of the lines in pixels. 4*12/720 is the conversation factor from pixelspace to meterspace.

## Vehicles distance from road center
To calculate the vehicles distance to the road center I added the position of the two lane lines and took the half of the sum. Then I subtracted the position of the camera which is mounted in the center of the car. 

## writing the curvature and offset from center onto the image
my last step in for imageprocessing. I used the `cv2.putText()` function.

# Video
Here's a [link to my video result](https://github.com/christianreiser/P4-Advanced-Lane-Lines/blob/master/project_output.mp4)


# Discussion
## Problems and further ideas for improvement
On the straight highway without shadows my pipeline works pretty well. However, if the curvature of the road is too strong, the camera is not able to capture the whole road. Therefore, my program is not able to detect the lane lines in this case. To make it more robust I would have to use a wide angle camera or use multiple cameras to capute the road on the very left and right. 
Another problem are shadows. Maybe I could destinguish lane lines from shadows by looking at positive gradients which are longer, have a certain near vertical angle right next to a negative gradient with a similar angle.
The probably most compelling implementation would be to take the average of more frames to reduce misconceptions in specific frames.
Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 
