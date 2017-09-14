# **Finding Lane Lines on the Road** 


---

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./debug_images/debug-1.png "Colour Mask"
[image2]: ./debug_images/debug-2.png "Grayscale"
[image3]: ./debug_images/debug-3.png "Gaussian Blur"
[image4]: ./debug_images/debug-4.png "Canny Edge Detection"
[image5]: ./debug_images/debug-5.png "Region Masking"
[image6]: ./debug_images/distcon-lines.png "Discontinuous Lines"
[image7]: ./debug_images/solidWhiteRightLines.png "Region Masking error"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

At a high-level the pipeline I created consists of the following steps. 
* Apply a colour mask and convert the image to grayscale
* Apply a Gaussian blur 
* Apply the Canny edge detection
* Mask the image to focus on the relevant parts
* Apply a Hough Transform to extract the lane lines
* Clean the lane line data
* Identify the lines for the left lane marker and the right lane marker
* Apply a moving average
* Render the image

I have included example images below to show how the image data is modified during the stages from converting to grayscale through to the Hough Transform. 

The colour mask application

![alt text][image1]

The conversion to grayscale. Strictly speaking the colour mask step could be omitted here as the extra value that is brought because of this step is minimal. 

![alt text][image2]

The Gaussian Blur. I used a kernel size of 7 to generate this. 

![alt text][image3]

The Canny Edge Detection

![alt text][image4]

Region Masking. This is a big help as it can remove so much of the noise. We need to be careful that we do not cut into the valid area where lane lines could be observed. 

![alt text][image5]



In order to improve the way lines were drawn I needed to modify the draw_lines() function. There were two primary objectives, the first was to draw a single line that represented the entire lane line as opposed to during individual lines for dashed line markings. The second modification was the need to introduce an averaging behaviour to smooth the rendering of the lane lines. I decided to move the for loop code that iterated through the lane lines into my process_image function()

My approach for drawing single lines to represent the discontinuous lines was to calculate the average slope and intercept for the left lane lines and repeat for the right lane lines. I found that there was noise in the data that meant the exact slope did not always line up exactly with the lane lines in the image. To address this I introduced an additional filter where I removed slope/intercept values based on standard deviation. Whilst this offered some improvement it did not completely solve this issue. I then added a rolling average based on the last n calculations for slope and intercept. This helped but still does not produce a solution as precise as I would have liked. This was most pronounced on the right hand lane line so I added a solution (read as dirty hack) where I slightly adjusted the intercept.

I tried an alternate approach to fit a line to the points using the numpy.polyfit function. This seemed to produce a result that was worse than the intercept/slope calculation method but I suspect this would work well and that the problem lies with the cleaning of the data as opposed to the line fit function. 

To get a solution working for the optional challenge video I had to use a weighted average to cover the period where the car drives over the different coloured tarmac. In my code it would not detect lines during this period (well not consistently) so I smoothed this via the averaging function. This worked but feels like cheating as it would be more robust if the code could still detect the lines consistently irrespective of whether the tarmac colour changes. 

I needed to make sure that I calculated the region mask using percentage values as opposed to fixed points. This was because the third challenge video was a different size from the others and this initially caused a problem. 

I also encountered strange behaviour at the start of my videos with the initial position of the lines being off centre. I realised this was because I was not resetting the rolling average between each video. Once I updated this the issue was resolved. 

Overall I found getting an exact match on the angle of the lines to be much tougher than I had expected. I believe this is due to the noise in the line detection process and I feel I would benefit from spending more time cleaning up the detected lines as opposed to trying to find a fix to manipulate the calculated average slope and intercept. 

### 2. Identify potential shortcomings with your current pipeline


There are a number of shortcomings with the current approach, I describe them below and offer suggestions for addressing them in the next section. 

* The lines are straight. In the videos we use the lines are of largely straight roads and slight curves. I render a linear line using a slope and intercept calculation. This is not going to be suitable for tighter bends.
* Lane changes. At the moment I am only looking for a left land marker and a right lane marker. When moving in and out of lanes on a multi-lane road (motorway/highway) would cause confusion during the transition time (lane change).
* Weather and night time. I am not convinced that the proposed method would work as is during the night time or perhaps during heavy rain. The impact of headlights and heavy rain on the lines would need to be evaluated. 
* Inefficient. I suspect that the approaches I have used are not as efficient as they would need to be in order to run this process in a continual  manner on practical and real world hard ware. 
* Cars moving across lines. As other cars in front of the vehicle change lanes they will potentially obscure the lane markings and may have an impact of the line detection process. 
* Hills. I am not sure how well this would work with rapid elevation changes.

### 3. Suggest possible improvements to your pipeline

In order to address the shortcomings highlighted in the previous section we could do the following.

* As opposed to drawing straight lines we could fit a curve that better represented the lane lines, e.g. polynomial regression. 
* Lane line mirrors. Given that lanes are highly likely to be of even width we could mirror the intercept of a well detected lane line against one that we were struggling to detected (e.g. dotted lane-markings). We could use this to sanity check the slope or include it as part of the slope calculation. 
* To address the weather and impact of night time driving we would first need to collect additional data from these environments. I suspect that night vision camera modes may help or we may find that the lights from the car are sufficient.
* Colour filter. We could determine what colours to remove in a mask by selecting random samples from in front of the vehicle where we believe the road is likely to be. This would allow different colour tarmac and road surfaces to be removed. 

