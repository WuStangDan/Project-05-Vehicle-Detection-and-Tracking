# Project-05-Vehicle-Detection-and-Tracking


## Extracting Features

A histogram of oriented gradients (HOG) is used to extract features from an image so that it can be used in a support vector classifier. All of the indivdual color channels in the HLS, HSV, YUV, and RGB color spaces are used to create a support vector classifier which is then tested on the same set of training data and test data. The color channels which yielded the betst accuracy were the HSV value channel and the YUV luminance channels. Combining the features of this channel proved to yield the highest test accuracy of 96.7%. 

Color histogram was not used as it was found to decrease the test accuracy even when scaled with the HOG features. Scaling was also not used on the features since scaling was found to slightly decrease the test accuracy. Since the features only comprised of HOG data it was found that scaling was not needed to achieve maximum test accuracy from the classifier.


![HOG Features](/readme_images/hog.png)

## Training Classifier

A linear support vector classifier (SVC) is found to produce satisfactory accuracy results while remaining extremely fast to enable close to real time prediction. An SVC using the rbf kernel is found to produce similar test accuracy to the linear kernel but takes more than 10 times as long to not only train but also classify a feature set. Training a decision tree classifier produced very low accuracy results compared to a linear SVC. None of the hyper parameters were changed from the default LinearSVC values although the max iterations, tolerance, and loss function were all experimented with.

The HOG classifiers test accuracy which is used to select the color channels to be used for the final HOG can be seen below.
    HOG Color Space [ Channel, Test Accuracy]
	-----------------------------------------
    HLS [0, .933], [1, .959], [2, .905]
    HSV [0, .933], [1, .910], [2, .954]
    YUV [0, .963], [1, .947], [2, .xxx]
    RGB [0, .950], [1, .960], [2, .956]


## Sliding Window Search

A sliding window search is used to find windows and then resize them so that portions of the entire image frame can be fed into the classifier in an attempt to detect any cars. Two separate sliding window creation functions are used by the final algorithm, one which is a general search over the whole image and one which is a focused search only in the location where a car was detected in the previous frame.

Pictures of the general sliding window search can be seen below. The image on the left shows the sliding window with 50% of overlap and on the right the windows with no overlap. The window size was determined by first fitting the smallest windows to a car in test images from the project video and then creating an exponential growth rate which was tuned to fit the largest car seen in the test images. 50% overlap was used for the final algorithm as it allowed a great fit over the entire video screen without greatly increasing the window size and thus not slow down the entire pipeline.

![General Sliding Window - Small](/readme_images/sw1.png)

![General Sliding Window - Medium](/readme_images/sw2.png)

![General Sliding Window - Large](/readme_images/sw3.png)

The focused window search can be seen below. It contains one large window and three sets of smaller windows with overlap inorder to inrease detection probability once a car has reasonable been detected.

![Focused Sliding Window - All](/readme_images/sw4.png)

A function called combine_pred is used to combine multiple prediction windows (search windows that have been classified as a car) which are close to each other. This function is the main filter for false positives as inorder for a group of windows to be combined and classified as a car, there must be atleast 3 windows which detect a car in close proximity. This function decides where the final prediction boxes will be drawn on the output video. These boxes are also used as the location for the focus window search for the following frame.

An example of the input and output of the combine_pred functino can be seen below.

![Combine Predictions - Input](/readme_images/combinein.png)

![Combine Predictions - Output](/readme_images/combineout.png)



