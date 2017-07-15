
**Advanced Lane Finding Project**

The goals of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[camera_dist]: ./camera_cal/calibration2.jpg
[camera_undist]: ./output_images/calibration2_undistorted.jpg

[test2]:./test_images/test2.jpg
[test2_undist]:./output_images/test2_undist.jpg

[v1_grayscale]:./output_images/v1_test2_grayscale.jpg
[v1_lchannel]:./output_images/v1_test2_lchannel.jpg
[v1_schannel]:./output_images/v1_test2_schannel.jpg
[v1_sobelx]:./output_images/v1_test2_sobelx.jpg
[v1_threshed]:./output_images/v1_test2_threshed.jpg
[v1_all]:./output_images/v1_test2_all.jpg

[test2_bird]:./output_images/test2_bird.jpg

[v1_test3_warp]:./output_images/v1_test3_warp.jpg

[test3]:./test_images/test3.jpg
[test3_lanes]:./output_images/test3_lanes_rad.jpg

[hard1]:./test_images/hard1.jpg
[hard1_all]:./output_images/v2_hard1_all.jpg
[hard1_lanes]:./output_images/v2_hard1_lanes.jpg
[hard1_warp]:./output_images/v2_hard1_warp.jpg

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

---

### Camera Calibration

The code for this step is contained in `calibrate_camera()` in the [IPython notebook](./lane_finder.ipynb).

The steps are straightforward: preparing 'object points' - (x, y, z) coordinates of the chessboard corners in the world, and 'image points' - (x, y) pixel positions of each of the corners in the image plane. Using OpenCV `cv2.calibrateCamera()` function the camera calibration and distortion coefficients are calculated.

Example of distorted and undistorted images are provided below:

![camera_dist]
![camera_undist]

### Pipeline (single images)

#### 1. Distortion correction

Each image is undistorted at the first step, as described above.

Original image:
![test2]

Undistorted image:
![test2_undist]

#### 2. Binary image

The following techniques were used (function `process_image_v1()`): applying x-oriented Sobel for S channel of HLS image color space, x-oriented, magnitude and oriented Sobel for grayscaled image, thresholding L-channel of HLS image color space (for discarding gradients arising because of dark shadows).

Some samples are below:

Original image:
![test2]

X-oriented Sobel applied to S-channel:
![v1_schannel]

Grayscaled image:
![v1_grayscale]

X-oriented Sobel applied to grayscaled image:
![v1_sobelx]

Magnitude and direction Sobel combined with x-oriented Sobel above, applied to grayscaled image:
![v1_threshed]

L-channel thresholded:
![v1_lchannel]

All thresholds combined:
![v1_all]

#### 3. Perspective transform

Source and destination points for perspective transform were hardcoded as

| Source        | Destination   |
|:-------------:|:-------------:|
| 568, 470      | 272, 0        |
| 721, 470      | 1037, 0      |
| 1048, 642     | 1037, 720      |
| 275, 642      | 272, 0        |

Correctness verification was made by applying perspective for image where lines appear to be parralel and checking the result.

Examples of applying the perspective transform are below:

Original image:
![test2]

Birds-eye view:
![test2_bird]

#### 4. Identifying lane-line pixels

Lane lines were identified by sliding window search (function `sliding_window_search_v1`) with some restrictions applied. For example, I check if we can't find the pixels near the image edge for few cycles, the search process is stopped early for this line.

![v1_test3_warp]

#### 5. Radius of curvature and position of vehicle

This is made in `calculate_curvature_radius()` and `calculate_offset()` functions in the code.

#### 6. Resulted image

The whole process is described in `find_lane_v1()` function in the [notebook](./lane_finder.ipynb). Here are the original and resulted images:

![test3]
![test3_lanes]

---

### Pipeline (video)

#### 1. Search lanes

If we have lanes identified on previous steps, we try to search lanes on the current frame near these old lanes (funcion `search_points_knowing_polys_v1`). If we fail (lanes appear to be intersecting or too narrow), the sliding window search is applied.

#### 2. Final video.

Here's a [first video](./output_videos/project_video_v1.mp4) and [second video](./output_videos/challenge_video_output_v1.mp4) results. The model runs decent for [third video](./output_videos/harder_challenge_video_output_v1.mp4) except the last turn.

---

### Model v2

For handling turns like in third video, I've tried to create a better model. I used combination of blurring image, applying x- and y- oriented Sobel for grayscaled image and V-channel of CIE Luv image colorspace (it turns out it is recognizes yellow lines just great even in bad conditions!) and thresholding L-channel of HLS color space. The sample is provided below:

Original image:
![hard1]

Applying thresholds:
![hard1_all]

Also, the perspective transformation was changed a little to prevent cutting off the pixels near the image edges and more restrictions were applied for identifying if lanes are 'good' in `sliding_window_search_v2()` and `search_points_knowing_polys_v2()` functions.

![hard1_warp]

Result image:
![hard1_lanes]

The result is better for the turn, however model still needs a lot of tuning on 'noisy' frames, especially for white lanes: [resulted video](./output_videos/harder_challenge_video_output_v2.mp4)

### Discussion

After many tries it seems to me that best model can be built considering the following points:
* Applying strict conditions for thresholding and lanes themselves (curvature, spacing, etc.) and trying to detect lane lines;
* If we fail to detect one lane, we can lower restrictions and try to find the missing lane again, checking if it doesn't mess with the first one;
* If we fail again, we can apply additional restrictions for sliding window search (early stopping technique, rising min pixels count etc.) trying to avoid 'noisy' pixels;
* Smart smoothing of lanes over the video frames is important (weights, checking that line doesn't 'jump' a lot etc.);
* Perspective really matters! Choosing the right one could help a lot, especially for hard frames.
