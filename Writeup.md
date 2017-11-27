## **Advance Lane Line Detection** 
---
[image01]: ./images/Chess_Image.PNG
[image02]: ./images/Correct_Distortion.PNG
[image03]: ./images/Birds_Eye.PNG
[image04]: ./images/Gradient_Thresholding.PNG
[image05]: ./images/S_Thresholding.PNG
[image06]: ./images/Combined.PNG
[image07]: ./images/Sliding_Window.PNG
[image08]: ./images/Sliding_Window_1.PNG
[image09]: ./images/Chess_Corrected.PNG
[image10]: ./images/Mapping_Lanes.PNG
[image11]: ./images/Radius.PNG
[image12]: ./images/Histogram.PNG



### 1. Loading Chessboard Data

- Below cell reads the chessboard images from the given path
- Stores the chessboard images in a list and return the list
- This chessboard images is then used for camera calibration
![ScreenShot][image01]


### 2. Camera Calibration


#### Camera Calibration: To compute the transformation between 3D object points in the world and 2D image points

#### Below cell performs the camera calibration

- I started by preparing **"object points"**, which will be the (x, y, z) coordinates of the chessboard corners in the world.
- Here I am assuming chessboard is fixed on (x, y) plane at z=0, such that object points are same for each calibration image.
- Thus, **`objp`** is just a replicated array of coordinates.
- **`objpoints`** will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.
- **`imgpoints`** will be appended with (x, y) pixel position of each of corners in image plane with each successful chessboard detection.
- I then used the output **`objpoints`** and **`imgpoints`** to compute the **`camera calibration`** and **`distortion coefficients`** using the **`cv2.calibrateCamera()`** function.
- The **`cv2.calibrateCamera()`** function is used in Cell number 4, Line Number 40

### 3. Distortion Correction

- To ensure that the geometrical shape of object is represented consistently, no matter where they appear in an image I perform 
- Image distortion occurs when a camera looks at 3D objects in the real world and transforms them into a 2D image **Distortion Correction**.
- This transformation by camers isn’t perfect
- Distortion actually changes what the shape and size of these 3D objects appear to be
- So, I need to undo this distortion
- I will use **Camera calibration parameters** for undistorting a distorted image
- Below is example of distorted images at left 
- Right image is undistorted image of the left image which is caluclated by **camera calibration matrix** and **distortion coefficients**
- To compute the undistorted image  **`cv2.undistort()`** function is used
![ScreenShot][image09]


### 4. Distortion-corrected image Example
- Below is another example of distorted images at left of lane image 
- Right image is undistorted image of the left image
![ScreenShot][image02]

### 5. Defining the Source and Destination points for perspective transformation


| Source         | Destination   | 
|:-------------: |:-------------:| 
| 575,  464      | 450, 0        | 
| 707,  464      | 450, 720      |
| 258,  682      | 830, 720      |
| 1049, 682      | 830, 0        |




### 6. Perspective Transform

- A perspective transform maps the points in a given image to different, desired, image points with a new perspective
- The perspective transform to get a bird’s-eye view transform that let’s us view a lane from above
- The **`src`** points in the below code are mapped to **`dst`** points in a different perspective image 
- First I will find the perspective transformation matrix from the **`src`** and **`dst`** points
- I used the **`cv2.getPerspectiveTransform()`** function for getting the transformation matrix Cell 9, Line number 4.
- After calculating transformation matrix **`M`** we will perform transformation using **`cv2.warpPerspective`** function in Cell 9, Line number 7 
![ScreenShot][image03]



### 7. Gradient Thresholding 

- We want to find vertical lines because lane lines are vertical
- I applied Sobel Operator in x direction
- Applying Sobel in x direction gives us vertical lines
- Applied thresholding so that we get only lane lines and discard other lines
- 65 is the minimum threshold value and 255 is maximum threshold value
- All the pixels below 65 are discarded
- You can see the results below
![ScreenShot][image04]

 
### 8. Color Thresholding

- By applying gradient thresholding we loose color information
- This is because we convert the image to grayscale
- We will now explore HLS space
- In HLS space the S channel helps in finding the vertical lane lines
- S channel does a fairly robust job of picking up the lines under very different color and contrast conditions
- 125 is the minimum threshold value and 255 is maximum threshold value
- All the pixels below 125 are discarded
- You can see the results below
![ScreenShot][image05]


### 9. Color and Gradient

- S channel does a fairly robust job of picking up the lines under very different color and contrast conditions
- But S chennel does not detect lanes that are far away
- The far away lane lines are detected by applying gradient threshold in x -direction
- I combine color and gradient thresholding to get the best of both worlds
- Here's an example of how that might look
- Red pixels are results of applying gradient thresholding
- Blue pixels are results of applying color thresholding to S channel
![ScreenShot][image06]


### 10. Locate the Lane Lines and Fit a Polynomial


- I  need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line
- I used sliding window method to fit a second order polynomial
- I first take a histogram along all the columns in the lower half of the image 
- In my thresholded binary image, pixels are either 0 or 1
- So two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines.
- I can use that as a starting point for where to search for the lines
-  From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.
- After finding the lane line pixels, I simply use np.polyfit to fit the lane line pixels to a second order polynomial
![ScreenShot][image07]
![ScreenShot][image08]

### 11. Visualizing the histogram of grayscale Combined Image

- Below is example of histogram along all the columns in the lower half of the image 

![ScreenShot][image12]






### 12. Radius of curvature of the fit

- we'll compute the radius of curvature of the fit.
- we have located lane line pixel
- used their x and y pixel positions to fit a second order polynomial curve:
   $$f(y) = {Ay^2 + By + C}.$$
- Radius of curvature at any point x of the function x = f(y) is:
   $$R_ {curve} = {[1 + (\frac{dy}{dx})^2]^{3/2} \over |\frac{d^2y}{dx^2}|}.$$
- In the case of the second order polynomial above, the first and second derivatives are: 
  $$f'(y) = {\frac{dy}{dx}} = {2Ay + B }.$$
  $$f''(y) = {\frac{d^2y}{dx^2}} = {2A}.$$
- So, our equation for radius of curvature becomes:
  $$R_ {curve} = {[1 + ({2Ay + B })^2]^{3/2} \over |{2A}|}.$$

- we actually need to perform this calculation after converting our x and y values to real world space.
-  x and y values are converted to real world space in Cell 24, Line number 3 and 4
- The above equation is calculated in Cell 24, Line number 27 and 28

![ScreenShot][image10]
![ScreenShot][image11]

### 13. Pipeline (video)

Here's a [link to my video result](./project_video_output.mp4)

### 14. Discussion

My Pipeline fails on challenge videos