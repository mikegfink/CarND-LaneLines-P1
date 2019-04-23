# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[image1]: ./test_images_output/filter_white_and_yellow.jpg "Filter white and yellow"
[image2]: ./test_images_output/canny.jpg "After applying Canny filter"
[image3]: ./test_images_output/hough.jpg "After applying Hough transform, slope filtering and smoothing"
[image4]: ./test_images_output/final.jpg "Final image drawn on original"
---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 9 steps. The initial pipeline was straightforward and quite effective on the simple videos. All of the modifications resulted from handling the challenge video.

1) Find colors.

    This stage applies an inrange mask to find a range of yellow colors. Then another inrange mask to find white colors. The stage then combines the masks and removes all other colors from the original image with a bitwise_and operation.
    
![Filter Colors][image1] 
2) Grayscale.

    This stage applies a simple grayscale function to remove the color channels.
3) Gaussian blur.

    This stage applies a gaussian blur function with a kernel size of 3. I found that this was a relatively uninfluential step.
4) Mask area of interest.

    This stage applies a polygonal area filter to reduce the data processing on the image.
5) Canny filter.

    This stage applies a Canny filter to the image. The Canny filter uses permissive settings to try to pick up as many occluded and passing lane lines as possible. Extraneous lines are handled by the filtering in other stages. The settings were a low threshold of 30 and a high threshold of 100.
    
![Canny Filter][image2]
6) Hough lines.

    This stage applies a Hough filter to the Canny image. The Hough filter uses relatively permissive settings as well for the same reason as the Canny filter. The rho and theta values remained the common values of 2 and 1 degree. The threshold value is 10, the minimum line length is 40 and the maximum line gap is 20 pixels. The permissive settings are mostly to pick up broken lane lines.
    
7) Filter slopes of found lines.

    This stage applies a slope filter. First, we split the lines into two piles, one for the left and one for the right lane line. This is accomplished by calculating the slope of every line that was found from the Hough lines stage. If the slope is negative (moving from lower left to upper right) and is on the left half of the area of interest, then it can contribute to the left lane line. Similarly, if the slope is positive and the line on the right half of the area of interest, then it can contribute to the right lane line. This stage was introduced to handle the low contrast portions of the challenge video. The lines contributing to each lane line are then averaged and extended to the base of the image and up to an appropriate portion of the image. The extension function is simple manipulation of m = dy/dx to find the correct x values.
    
8) Smooth the lines.

    This stage smooths lines from frame to frame. An average line for the left and right lane are kept, if the detected lines stray from the average by too much, then they are rejected. Lines can be rejected if the lower x value has changed by an experimentally selected percentage from the average, in this case 4%. If the line passes the filter then a weighted average of the detected line and the average lane line is computed and output. This prevents the output from jumping around too much in imperfect videos.
    
![Hough Transform and Friends][image3]
9) Draw on final image.

    This stage simply adds the lane lines to the original video.
    
![Final Drawing][image4]


### 2. Identify potential shortcomings with your current pipeline

There are many shortcomings with the current pipeline. First, it is probably overfit to the challenge video. The color selection and filter values especially might not be as effective in other conditions. For example, an older road in bright sunshine, even if lined, might catch the yellow color filter. 

Second, the filter slopes will prevent detection of aggressive turns that fall outside of the expected slope range. Additional processing would be needed for these and if perpendicular lines need to be detected.

Third, the averaging system in the pipeline is very simple and every line detected in the whole video contributes to the average. Preferably, this would be a moving average that only tracks a set of the most recent frames (e.g. 10). I didn't want to maintain a circular buffer for this project, so I used a simple mechanism. This means that if, for example the lane width changed, our current lane lines would be affected by the previous lane width.

Fourth, higher speed corners are likely to cause issues with line detection. The challenge video already forced me to reduce the draw height in order to not stray from the lines in the video on turns. At higher car speeds and/or sharper turns the curved lines will take up even more of the screen. A Hough transform should be able to detect curves, but this obsoletes any value there may be in finding the average slope of detected lines.

Finally, it is much too slow. While Python isn't ideal for performance, this pipeline takes munch longer to process a video than the runtime of the video, which means it cannot be used in real time (even ignoring the cost of drawing back the image and creating the video file, the filtering appeared to take longer than the runtime).

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to segment the lane lines to allow for variations in slope. For example if the road is curving we could detect this and draw the curve or at least line segments with varying slopes. Alternatively, we could fit to a curve instead of just straight lines.

Another potential improvement could be to use more advanced line filtering. As mentioned in the shortcomings section, the averaging was simplistic. With a history of detected lines, you could better ensure smooth changes from frame to frame. Further, with a history of images you could potentially determine a confidence level of the line detected in the current frame. If we have a lot of found lines (long line segments as a proxy perhaps), then we could be more confident in the result. Then you could weight data from frames in the average based on the confidence for more accurate lanes. Lanes aren't likely to change quickly, and even if they did you could probably detect that in frames with high confidence lines that differ from one another.

The found lane lines could also be transformed to a vertical view such that the lane lines are perpendicular. This way, you would only need to be confident in one lane line and the width of the lane in order to determine both lane lines.

Of couse, one could just research the current state of the art in lane finding and apply those techniques instead.
