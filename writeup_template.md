# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

I followed the steps as described in the course and included them the provided file under `transform_image()` method
1. Convert the image to grayscale
2. Blur the image to reduce image noise and reduce detail and simplify the image for further processing
    - The kernel size was chosen to be 5 since I taking more pixels into account (5x5) was better in smoothing the image
3. Apply canny edge detection on the image with 1:3 ratio of threshold. I chose 50 as per the tutorial and saw it giving decent returns
4. I then calculated the region of interest to form a polygon relative to image shape. 
    ```
    Point 1: (40,imshape[0])
    Point 2: (imshape[1]/2-30, imshape[0]/2+80)
    Point 3: (imshape[1]/2+30, imshape[0]/2+80)
    Point 4: (imshape[1]-40,imshape[0])
    ```
5. Followed up with drawing hough lines with  rho = 2, theta = np.pi/180, threshold = 20, min_line_length = 40,max_line_gap = 20
6. Hough lines were drawn using the following draw_lines method which had following high level operations
    ```
    1. Fetch the list of lines from the cv2 method for houghLines
    2. Iterate through the list of lines and use a dictionary to store top most point and bottom most point per lane. Left lane has negative slope and right lane as positive slope.
    3. Using the numpy method `polyFit`, construct a line between top most and bottom most point
    4. Feed the above line to `poly1d` method which will return the slope and intercept of that line
    5. Use the above parameters to get the point on the bottom edge of image where the above line will intesect. Call it bottom-edge point
    6. Feed the bottom-edge point and top most point per lane to the cv2 draw line method to draw a line on a blak image
    ```
7. Use the above hough lines and apply them on the image using `weighted_img` method
8. This results in following image
![alt text](/test_images_output/solidWhiteRight.png)


### 2. Identify potential shortcomings with your current pipeline
1. Inherent danger in any pipeline algo is that any flaw in initial steps can exponentially affect the subsequent steps unless proper error correct is built in the next steps. This pipeline has similar issues as described in next points
2. Edge detection can fail on extraneous markings on the road and can confound the subsequent steps in the pipeline.
3. Polygon calculation for region of interest is highly dependent on camera position and which can change drastically if the camera alignment is changes or when the car is in lane transition
4. draw lines uses only 2 points to draw a line which may cause the lanes to be widely off due to lesser data to fit a line. You can see that issue happening in the video too where there are some abberations due to this issue
5. A lane is never a straight line always, it's curved sometime so draw_lines may not fully cover the lane properly.
6. A faded lane lines can cause the top most point to not be discovered and thereby a complete lane will be missed out.
7. Finding the bottom point on the image for draw_lines is flawed due to camera position

### 3. Suggest possible improvements to your pipeline
1. Instead of draw_lines use draw curves which will use cv2.drawPolygon
2. Instead of only using 2 points to draw line, use all the points from the hough lines and fit a line to those points to get a smooth line covering most of the lane
3. Tweak parameters in edge detection to exclude certain features which are not lane specific
