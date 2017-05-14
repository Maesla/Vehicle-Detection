##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[car-notcar]: ./output_images/car-notcar.png
[car-notcar-hog]: ./output_images/car-notcar-hog.png
[features]: ./output_images/features.png
[windows]: ./output_images/image_with_windows.png
[classifier]: ./output_images/vehicle-classifier.png
[detection]: ./output_images/vehicle-detection.png
[heatmap]: ./output_images/heatmap.png
[pipeline]: ./output_images/detection-pipeline.png
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.
I started searching all the images in the vehicle and not vehicle folders.

![alt text][car-notcar]
The code I use to extract HOG features is implemented inside a class. The class name is Feature_Extractor. This class can be found at cell #3, under the title Extract Features.

This class implements everything necessary for feature extraction. It also implements the hog extraction.

![alt text][car-notcar-hog]


The code for this step is contained in the first code cell of the IPython notebook (or in lines # through # of the file called `some_file.py`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters. Not only for HOG, for all the feature extractions that can be used in order to train the classifier. Basically I have used trial and error.
Finally I have used this combination:

| Parameter       | Value |
|-----------------|-------|
| Color Space     | LUV   |
| HOG             | True  |
| Orient          | 12    |
| Pixels per cell | 8     |
| Cell per block  | 2     |
| Channels        | All   |
| Spatial         | True  |
| Size            | 32    |
| Histogram       | True  |
| Bins            | 32    |


####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I have decided to use a SVM classifier.

I apply these steps:

1. I load all the images. I load them in a random order, to avoid several frames together.
2. I extract all the features for each image, using the parameters explained in the previous point.
3. I normalize all the features
4. Then, I create the labels. 1 for vehicles and 0 for non vehicles.
5. I split the training set. 70% for training and 30% for testing.
6. I create the classifier and I train it.
7. I get an accuracy = 0.9803



The code for extracting the features and training the classifier can be found in cells #5, #6 and #7
![alt text][features]


###Sliding Window Search

####1. Vehicle Classifier
With the SVM classifier, I have implemented a class, **SimpleVehicleClassifier**. This class is injected by the SVM classifier and the scaler. It gets a 64x64 RGB at 255 image, extract its features, normalize them and predict if the image is a vehicle or not.

The code can be found at cell #8
![alt text][classifier]

####2. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?
I have implemented a full class in order to manage the window search. This class have several method to help with the window search.

This class is called **Window_Creator** and can be found in cell #10.

The main method is get_complex_windows. This method returns an window array. I have used several sizes. From 50 to 250 with several overlaps

The windows are calculated only in the road, so the skyline is ignored. For small windows, for example, only the horizon is scanned and without overlapping.


####3. Heatmap

For helping with detection, I have implemented a **HeatMap**. 


1. This class gets a positives windows (windows with a positive detection)
2. It generates a heatmap with all the votes.
3. It accepts a threshold. All the pixels with less votes are removed
4. Then, it calculates the labels, and with the labels creates new windows where the vehicles should be.

I have implemented also a FixedQueue. With this queue, I can cache the previous positive windows. It has a fixed size. If a new frame enters, the info from the oldest frame goes out.

Heatmap and FixedQueue can be found at cells #11 and #12

![alt text][heatmap]

####4. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Finally I have put all together in a single class. This class is called **VehicleDetector** and it is injected by **SimpleVehicleClassifier**, **Window_Creator** and **HeatMap**, all explained before. The code can be found at cell #14

This class implements the pipeline.

1. It gets an image
2. It iterates through the windows and caches the positives.
3. it feeds the heatmap with the positive. The heatmap filters with its own configuration.
4. It draws the positives returned by the heatmap 


![alt text][detection]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./out_project_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The pipeline is explained in the section above. The false positive are filter by a heatmap and a threshold value.

In the image, the frame count = XX and the threshold = YY. This means that if a positive is detected, it will be taken into account for the next XX frames. If in this XX frames, the zone has more than YY votes, a vehicle is detected.
![alt text][pipeline]

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

