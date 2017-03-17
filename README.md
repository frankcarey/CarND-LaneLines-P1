## Finding Lane Lines on the Road

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

Code and examples can be found in the [python notebook](P1.ipynb) 

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale and applied a small gaussian blur to reduce noise before finding edges.

<img src="./examples/p1_grayscale.png" width="300px">

Then, I used a canny filter to find points where the gradient (light/dark) was the highest, showing where the edges were.

<img src="./examples/p1_canny.png" width="300px">

Since, we're only interested in edges on the road, I removed all edges outside of where I expect the road to be and then find all Hough lines from that region. I settled on a `threshold` of 40 , `max_line_gap` of 100, and `min_line_len` of 100.

<img src="./examples/p1_region.png" width="300px">
<img src="./examples/p1_hough_lines.png" width="300px">

In order to draw a single line on the left and right lanes, I created a new function called `fit_line(lines, approx_angle, angle_threshold)`, which allows you to pass in a set of lines, in this case the ones from the Hough Transform function, an expected angle you think your lines will be in, and "fudge" factor in degrees that you're willing to consider. All the eligible lines are then averaged together or `None` is returned if none are found.

Since we want to maximize the line down to the bottom of the frame, I created an `extend_line(line, box)` function that allows you to set a maximum top and bottom that you want to extend the line out to. This worked well, but I felt that it didn't reflect a sense of confidence based on the line length.

<img src="./examples/p1_line_selection_and_scaling.png" width="300px">

In order to get things working the best with the challenge video, I decided to add some smoothing. This allows the left and right lines to be averaged out over a series of frames (5 by default). In addition to limiting the noise, it will also just return the last line if one isn't found in the current frame, adding to the stability. Instead of setting the top of the lines to a fixed y value, I instead used the top of the found line, but still extended the line all the way to the bottom of the image. The result worked pretty well in all the videos, even the challenge one.

<img src="./examples/p1_final_output.png" width="300px">


### 2. Identify potential shortcomings with your current pipeline


Because of the smoothing, even if there is no lane line found in the image/frame, the pipeline will still output the last one it found. That may make the system overly confident that a line line exists, even if it doesn't. For example, perhaps all the lines go missing in a stretch or road like when a new road is being paved and they haven't drawn the lines yet.

There are also a lot of hand-coded values like for the expected angles of the lane lines, thresholds, number of lines, etc. I'm not sure how well it would work on something more complex like a lane change, an on-ramp, or a crosswalk. The shadows in the challenge video show how fragile it can be.


### 3. Suggest possible improvements to your pipeline

I think it might be helpful to find the vanishing point in the image and use that to predict what lines are the most appropriate to use instead of hard-coding values.

Another potential improvement could be to remember the line positions between frames of the video and use that as a way to select better regions when finding the next set of lines.
