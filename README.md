# Project-05-Vehicle-Detection-and-Tracking


## Extracting Features

A histogram of oriented gradients (HOG) is used to extract features from an image so that it can be used in a support vector classifier. All of the individual color channels in the HLS, HSV, YUV, and RGB color spaces are used to create a support vector classifier which is then tested on the same set of training data and test data. The color channels which yielded the best accuracy were the HSV value channel and the YUV luminance channels. Combining the features of this channel proved to yield the highest test accuracy of 96.7%. 

Color histogram was not used as it was found to decrease the test accuracy even when scaled with the HOG features. Scaling was also not used on the features since scaling was found to slightly decrease the test accuracy. Since the features only comprised of HOG data it was found that scaling was not needed to achieve maximum test accuracy from the classifier.

The final HOG parameters used are 9 orientation bins, 8x8 cell pixels, and 2x2 cell boxes. Just like in deciding what HOG features to use I picked this number based on the resultant test accuracy it would achieve when used to train a support vector classifier. These values are the standard values that were used in the lecture quizes because although I could achieve a higher test accuracy by changing these values, usually the marginal increase in test accuracy wasn't worth the additional computation time from the greatly increased number of features.


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

For the final classifier decision_function() is used over predict() so that a threshold can be set to limit the amount of false positives. This only filters out the predictions that are weak and leaves the predictions that the SVC is more confident about. The threshold of 0.3 is used for the final video pipeline as it eliminated false positives but didn't have significant impact on detection.


## Sliding Window Search

A sliding window search is used to find windows and then resize them so that portions of the entire image frame can be fed into the classifier in an attempt to detect any cars. Two separate sliding window creation functions are used by the final algorithm, one which is a general search over the whole image and one which is a focused search only in the location where a car was detected in the previous frame.

Pictures of the general sliding window search can be seen below. The image on the left shows the sliding window with 50% of overlap and on the right the windows with no overlap. The window size was determined by first fitting the smallest windows to a car in test images from the project video and then creating an exponential growth rate which was tuned to fit the largest car seen in the test images. 50% overlap was used for the final algorithm on the largest and smallest window size as it allowed a great fit over the entire video screen without greatly increasing the window size and thus not slow down the entire pipeline. The medium window size however used a 70% overlap as I found the pipeline was having significant trouble detecting the car at that range and increasing the overlap greatly helped that.

![General Sliding Window - Small](/readme_images/sw1.png)

![General Sliding Window - Medium](/readme_images/sw2.png)

![General Sliding Window - Large](/readme_images/sw3.png)

The focused window search can be seen below. It contains one large window and three sets of smaller windows with overlap in order to increase detection probability once a car has reasonable been detected.

![Focused Sliding Window - All](/readme_images/sw4.png)

A function called combine_pred is used to combine multiple prediction windows (search windows that have been classified as a car) which are close to each other. This function is the main filter for false positives as in order for a group of windows to be combined and classified as a car, there must be atleast 3 windows which detect a car in close proximity. This is classified in the pipeline as a blue box. If there are only two windows in close proximity, the combined result is placed in a pink box which signifies a detection but higher possibility of it being a false positive. This function decides where the final prediction boxes will be drawn on the output video. These boxes are also used as the location for the focus window search for the following frame.

An example of the input and output of the combine_pred function can be seen below.

![Combine Predictions - Input](/readme_images/combinein.png)

![Combine Predictions - Output](/readme_images/combineout.png)


The only difference between the video implementation and the implementation on individual frames is use of a class to carry the information output from the combine_pred function over to the next frame so that the focus window search will be implemented at the location where previously a vehicle was detected.

My final video output is by no means perfect but I did make most decisions with optimization in mind which resulted in an algorithm that can process around 4 frames per second.

## Discussion

To improve this pipeline I believe I would need to achieve a higher accuracy on the classifier by using a powerful but slower machine learning tool. Without the focused window search there are roughly 350 search windows. With an accuracy of 96.7% that means that 12 windows are misclassified. This is not only a large generation of false positives each frame but also has the classifier missing out on what could be a few key windows in order to classify a car. Because of this high rate of false positives, the aggressive setting of the combine_pred function and the decision function on the SVC need to be used. This is why in the output video the cars are not detected. It is not because there are no windows detecting the car but rather that in order to filter out the false positives these predictions need to be filtered out.

One thing that could greatly help with this would be to take cropped images from the project video that are known to not be a car and included them in the classifier training data. However I didn't do this as its kind of like training the SVC on the test data.

Another thing I could improve would be some sort of averaging scheme on prediction boxes as they are quite wobbly and rapidly changing.
