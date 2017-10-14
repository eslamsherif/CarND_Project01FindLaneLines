# **CarND_Project01FindLaneLines**
My Solution for Udacity Self Driving Car Nano-degree first project: Finding Lane Lines on a Road

---

**Finding Lane Lines on the Road**

[//]: # (Image References)

[LCS]: https://i.pinimg.com/originals/63/c8/9a/63c89aba0ed994edcfce462b2a4b2b6b.jpg
[CF1]: ./Doc_Images/ColorFiltering/solidWhiteCurve.jpg
[CF2]: ./Doc_Images/ColorFiltering/solidYellowCurve2.jpg
[B1]: ./Doc_Images/Blurred/solidWhiteCurve.jpg "Grayscale"
[B2]: ./Doc_Images/Blurred/solidYellowCurve2.jpg "Grayscale"
[CED1]: ./Doc_Images/CannyEdge/solidWhiteCurve.jpg
[CED2]: ./Doc_Images/CannyEdge/solidYellowCurve2.jpg
[HL1]: ./Doc_Images/HoughLines/solidWhiteRight.jpg
[HL2]: ./Doc_Images/HoughLines/solidYellowCurve2.jpg
[LAE1]: ./Doc_Images/LineAveragingAndExtrapolation/solidWhiteCurve.jpg
[LAE2]: ./Doc_Images/LineAveragingAndExtrapolation/solidYellowCurve2.jpg
[OI1]: ./test_images_output/solidWhiteCurve.jpg
[OI2]: ./test_images_output/solidYellowCurve2.jpg

---

### Reflection

Udacity Self Driving Car Nanodegree first project is to find Lane lines on the road in series of videos provided by Udacity.

Before discussing the solution to achieve this goal, it is important to understand the problem and how to actually achieve the needed output.

Lane lines have some distinct features that can be used to identify them:
  * They have a unique color compared to the rest of the environment (usually white or yellow).
  * Their position is relatively constant in an image from the driver perspective.
  * They have sharp and clear edges.

Knowing this then it is clear that the tools that can be used are:
  * Color filtering and detection
  * Spatial filtering
  * Edge detection

I have assembled a 7 stage pipeline to achieve the project goal described below:

---

1) Color filtering
1a) Color space transformation
There exists multiple color spaces each with advantages/disadvantages the most common of the is the RGB color space used by most modern monitors and TVs.

I have chosen to convert to LAB color space, I think this color space is suitable for the application because:
  * It has a separate lighting axis, allowing for better handling for different lighting conditions.
  * High values in the B axis represent pure yellow color, allowing for more easier saperation of yellow components in an image.

below is a chart showing the color distribution in LAB color space.
![alt text][LCS]

P.S.: In python all values are re-mapped to be 0-255 range.

1b) Filtering Bright white portions in an image.
1c) Filtering yellow portions in an image.
1d) Merging of the two filtered portions

P.S.: The filtering values are chosen by Trial and error.
This approach is not optimal because it doesn't take lighting conditions a better approach would be to perform histogram equalization , e.g. CLAHE, prior to applying filtering.

The images below provides an example of what the output of this step looks like:
![alt text][CF1]
![alt text][CF2]

The lane lines are extracted successfully but it is clear that there exists a lot of unwanted objects in the image.

---

2) Converting image to grayscale color space:
Gray scale images have only shades of gray no other color can be found in such images, this reduces the amount of information we can obtain from an image but significantly reduce the computational bandwidth we need to operate on those images, having obtained the needed information from stage 1 we can downscale the resulting filtered image to gray scale to reduce needed computations in following steps

---

3) Image blurring:
Image blurring is a useful tool to reduce the amount of details in an image by changing pixel values based on their neighbors.
The number of pixels affecting the pixel value is known as a kernel size, the larger the kernel the more aggressive the blurring effect would be and more smooth the output image would look like, this is a good preprocessing for the next stage to get rid of unnecessary details as small edges that would affect accuracy of our calculations later.

---

4) Edge detection using Canny operation:
Lane Lines have very clear and sharp edges so a good edge detection algorithm as Canny would usually detect all present edges in the images.
OpenCV canny implementation provided a two way thresholding of edges where you define a weak and a strong edge thresholds.
* All edges less that the lower thresholds are not edges for sure.
* All edges above the upper threshold are strong edge for sure.
* edges that lie between the two thresholds must be connected to at least on strong edge, i.e. above upper threshold, to be considered an  edge.

Output of this stage is shown below
![alt text][CED1]
![alt text][CED2]

---

5) Hough Line detection:
Canny has identified the strong edges present in the image, however they lane edges are unconnected and not considered a single solid line. Hough line detection algorithm allows to connect similar lines to form a solid connected line.
I am using the probabilistic version defined by OpenCV due to it's lower computational needs.

Output of this stage is shown below:
![alt text][HL1]
![alt text][HL2]

---

6) Lines averaging and Extrapolation:
It is clear that the lane lines are connected and solid now, however there still exists a lot of edges that are not related to lane lines.
To help filter out those lines we need to understand their characteristics:
  * A lot of those lines are near horizontal lines, filtering based on line slope would filter those lines.
  * The lane lines seems to be represented by multiple small lines, averaging those lines would filter those out and leave a single line.

Output of this step is shown below:
![alt text][LAE1]
![alt text][LAE2]

---

7) Spatial Filtering:
Although the output lines seems very satisfying at the moment we can still use the remaining Lane line characteristic identified above to minimize the number of false positives identified in an image.
A polygon representing the observable lane was created and every thing outside this polygon was eliminated from the output image.

With The last stage output we can overlay the detected lines on top of the original image as shown below:
![alt text][OI1]
![alt text][OI2]

---

It is clear that the Lane Lines are detected correctly as requested, please check the Jupyter notebook for source code and the output videos in test_videos_output.
