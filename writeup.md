## Writeup Template

Vefak Murat AKman

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

[image1]: ./img/calibrate_drawing.png "Found Corners"
[image2]: ./img/calibratedimg.png "Undistorted Image"
[image3]: ./img/undistored_original.png "Undistorted Original Images"
[image4]: ./img/warped.png "Warp Images"
[image5]: ./img/sobelxy.png "Sobel X Y Absolute"
[image6]: ./img/sobelmagdir.png "Sobel Magnitude & Direction"
[image7]: ./img/colortransforms.png "Color Extractions"
[image8]: ./img/birdview.png "Birdview Process"
[image9]: ./img/histogram.png "Histogram"
[image10]: ./img/sliding.png "Sliding Windows"
[image11]: ./img/prior.png "Search From Prior"
[image12]: ./img/allproc.png "Pipeline"


[image13]: ./img/colortransforms.png "Sobel X Y Absolute"
[image14]: ./img/colortransforms.png "Sobel X Y Absolute"



[video1]: ./project_video.mp4 "Video"



### Camera Calibration
---

First, all chessboard images are converted to grayscale. Then, using OpenCV's `findChessboardCorners()` function to find corners of chess board. The corners are in below. Function found 17 of 20 images' corners
Corner and object points are saved in `objpoints` and `imgpoints` list variables.
![alt text][image1]

```python
Detect Corners:  17
Failed:  3```

`cv2.undistort` function is used to correct distortion on images. In example below, the distortion can be seen top of chessboard. The distortion corrected in second image
![alt text][image2]



### Pipeline (Test Images)
---

##### An example of a distortion-corrected image

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]


#### Perspective Transform
---
In this step, I first defined a Region of Interest (ROI):

Left Bottom Corner defined as "Left"
Right Bottom Corner defined as "Right"
Left Top Corner defined as "Left Apex"
Right Top Corner defined as "Right Apex"


The code for my perspective transform includes a function called `warper()`. The source and destination points are set. Then `getPerspectiveTransform()` function is used for perspective transform. Also, a function called `ROI()` is used for visualizing purposes. 



This resulted in the following source and destination points:

|Point      | Source     | Destination | 
|:---------:|:----------:|:-----------:| 
|Left       | 100, 720   | 200, 720    | 
|Right      | 1180, 720  | 1080, 720   |
|Left Apex  | 590, 450   | 200, 0      |
|Right Apex | 690, 450   | 1050, 0     |

![alt text][image4]

#### Sobel 
---
In this section, I tried several sobel methods to extract lines. The best given method is Sobel Absolute Threshold with X orient.
Also, Sobel Magtinude gave good result but when I combined it with color transformation images, the result were not satisfied

* Sobel Absolute Threshold Orient X
* Sobel Absolute Threshold Orient Y
* Sobel Magnitude Threshold
* Sobel Direction Threshold

![alt text][image5]
![alt text][image6]

#### Color Extraction 
---
Here, I tried several colorspace extraction method to identify lanes

* **HLS Colorspace: L & S Channel Extraction:**
     L channel threshold limits (100,255)-
     S channel threshold limits (125,255)
  
* **HSV Colorspace: S Channel Extraction**
     S channel threshold limits (100,255)
     
* **LAB Colorspace: B Channel Extraction**
     B channel threshold limits (190,255)
    
* ** White Extracting with `inRange` function**
	Differently from others, I tried the extracting white colors to find white lanes. Then, I found optimum values to extract both yellow and white lanes. First, I converted image into HSV colorspace and determined upper and lower limits. Finally I used `inRange` function to extract lanes. The given image was not in binary format. So, I used basic binary threshold method to convert image in binary format
     Lower limits (0,0,230)
     Upper limits (120,160,255)![alt text][image11]
    

![alt text][image7]
#### Birdview Processing
---
I created a function called `birdview()`  to process undistortion, warping, sobel and color transformation methods. After several tests, combination of Sobel Absolute X, HLS: S Channel, HSL: L Channel, LAB: B Channel and inRange method gave best results.
![alt text][image8]


#### Identify Lane Pixels
---
Sample histogram of warped image. The peaks show the right and left lanes. It is obvious that left lane more dominant.
![alt text][image9]

#### Sliding Window Method 
---

This method is used for calculating coefficients of polynomial lines. I will use these values in next step. Basically, we found peaks in histogram and matched it with images. Then, I calculated non zero x and y pixels. After, iteration is started and find new non zero x&y point for new windows. Finally, I used `polyfit()` function to calculates coefficients of right and left lanes

![alt text][image10]

#### Search From Prior 
---
The previous lane coefficients are used to calculate lanes. Also, the way of visualize this time is little bit different from sliding window method. 


![alt text][image11]

#### Radius of Curvature and Distance from Center Calculation
---
First, I defined a values for converting pixels to meters. Then calculate the curvature from left and right lanes. Then,return mean of these values. 
For Distance-: The center of image is the center of the car. To calculate the deviation from the center, the pixel positions in the left lane and the right lane were observer. The mean of the left bottom most point of the left lane and right bottom most point of the right lane were calculated. Then subtract it from the center of the car to get the deviation from the center.
```python
Left Curve: 459.62002618542544 | Right Curve: 270.37523011106555
Distance:  -1.19898590173
Curve:  364.997628148
```
#### Draw the Detected Area on Original Image
---
The warped section we also calculate reverse of process, Minv. 
Draw polynomial lanes and ROI on image. Also, the information about distance and curvature are put on image.
To have better visualizing, I also put sliding window method right-top of image.
Finally, I created a function called `process_image()` to process all pipeline. Then I tested process with all test images

![alt text][image12]

### Pipeline (video)
---
#### Here's a YouTube link of my project video. 

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/hzATw-_4V3U/0.jpg)](https://www.youtube.com/watch?v=hzATw-_4V3U)

---

### Discussion
---


The first problem is to find the correct source and destination points. It is a hit and trial approach and even 10 pixels up and down can make a big impact.

The second problem is lighting. When I tested challenge, I observed that the my method can not find lanes properly. So, tracking lanes are failed.

Image processing methods are good and enough for most of situation. But It may not be sufficient for changeable enviroment conditions. Learning-based approach would be used to overcome problems

The another problem is when the curvature of the road is high, the lanes are not properly finding.