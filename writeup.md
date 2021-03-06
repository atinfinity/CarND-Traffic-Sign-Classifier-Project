# **Traffic Sign Recognition** 

## Writeup

---

**Build a Traffic Sign Recognition Project**

The goals / steps of this project are the following:

* Load the data set (see below for links to the project data set)
* Explore, summarize and visualize the data set
* Design, train and test a model architecture
* Use the model to make predictions on new images
* Analyze the softmax probabilities of the new images
* Summarize the results with a written report

[//]: # (Image References)

[visualize_dataset]: ./images/visualize_dataset.png "Visualization"
[feature_map]: ./images/feature_map.png "Feature Map"
[curve_of_accuracy]: ./images/curve.png "Curve of Accuracy"

## Rubric Points
### Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/481/view) individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf. You can use this template as a guide for writing the report. The submission includes the project code.

You're reading it! and here is a link to my [project code](https://github.com/atinfinity/CarND-Traffic-Sign-Classifier-Project/blob/writeup/Traffic_Sign_Classifier.ipynb)

### Data Set Summary & Exploration

#### 1. Provide a basic summary of the data set. In the code, the analysis should be done using python, numpy and/or pandas methods rather than hardcoding results manually.

I used the pandas library to calculate summary statistics of the traffic
signs data set:

* The size of training set is
  * 34799(original)
  * 215000(with augumented images)
* The size of the validation set is 4410
* The size of test set is 12630
* The shape of a traffic sign image is (32, 32, 3)
* The number of unique classes/labels in the data set is 43

#### 2. Include an exploratory visualization of the dataset.

Here is an exploratory visualization of the data set.  

![visualize_dataset][visualize_dataset]

### Design and Test a Model Architecture

#### 1. Describe how you preprocessed the image data. What techniques were chosen and why did you choose these techniques? Consider including images showing the output of each preprocessing technique. Pre-processing refers to techniques such as converting to grayscale, normalization, etc. (OPTIONAL: As described in the "Stand Out Suggestions" part of the rubric, if you generated additional data for training, describe why you decided to generate additional data, how you generated the data, and provide example images of the additional data. Then describe the characteristics of the augmented training set like number of images in the set, number of images for each class, etc.)

I used the following techniques as preprocess.

- Grayscale
  - I think that shape and edge are more important than colors to recognize traffic sign
- Equalization of histogram
  - This process is effective to reduce the influence of contrast
- Normalization
  - This process is effective to reduce the influence of brightness

My preprocess code is as follows.

```python
# convert to grayscale
def grayscale(image):
    return cv2.cvtColor(image, cv2.COLOR_RGB2GRAY)

def equalize(image):
    return cv2.equalizeHist(image)

def normalize(image):
    min, max = np.min(image), np.max(image)
    return (image - min) / (max - min) * 2 - 1

def preprocess_image(image):
    return np.expand_dims(normalize(equalize(grayscale(image))), axis=2)
```

Here is an example of an original image and an preprocessed image:

| ![](images/preprocess/01_image.png) | ![](images/preprocess/02_gray.png) | ![](images/preprocess/03_equalized.png) | ![](images/preprocess/04_normalized.png) |
|:------:|:------:|:------:|:------:|
| Original Image | Grayscale | Histogram <br>Equalization | Normalization |

And, I decided to generate additional data because this process is effective to increase variations of data. As a result, generalization ability can be improved.

To add more data to the the data set, I used the following techniques. 

- Change of brighness
  - This process is effective to increase variations of light environment.
- Rotation
  - This process is effective to increase variations of angle.  
- Translation
  - This process is effective to increase variations of translation. 

My data augumentation code is as follows.

```python
def random_bright(image):
    eff = 0.5 + np.random.random()
    return image * eff

def rotate(image, angle=15):
    angle = np.random.randint(-angle, angle)
    M = cv2.getRotationMatrix2D((16, 16), angle, 1)
    return cv2.warpAffine(src=image, M=M, dsize=(32, 32))

def translate(image, pixels=2):
    tx = np.random.choice(range(-pixels, pixels))
    ty = np.random.choice(range(-pixels, pixels))
    M = np.float32([[1, 0, tx], [0, 1, ty]])
    return cv2.warpAffine(src=image, M=M, dsize=(32, 32))

def generate(images, count):
    generated = []
    while True:
        for image in images:
            if len(generated) == count:
                return generated
            image = random_bright(image)
            image = rotate(image)
            image = translate(image)
            image = normalize(image)
            generated.append(np.expand_dims(image, axis=2))
```

Here is an example of an original image and an augmented image:

| ![](images/data_augumentation/01_image.png) | ![](images/data_augumentation/02_brighted.png) | ![](images/data_augumentation/03_rotated.png) | ![](images/data_augumentation/04_translated.png) |
|:------:|:------:|:------:|:------:|
| Original Image | Change of <br>brighness | Rotation | Translation |

#### 2. Describe what your final model architecture looks like including model type, layers, layer sizes, connectivity, etc.) Consider including a diagram and/or table describing the final model.

My final model consisted of the following layers:

| Layer					| Description									| 
|:-------------:|:---------------------------------------------:| 
| Input					| 32x32x1 RGB image								| 
| Convolution 3x3		| 1x1 stride, `VALID` padding, outputs 28x28x6 	|
| RELU					|												|
| Max pooling			| 2x2 stride, outputs 14x14x6 					|
| Convolution 3x3		| 1x1 stride, `VALID` padding, outputs 10x10x16 |
| RELU					|	Activation											|
| Max pooling			| 2x2 stride, outputs 5x5x16 					|
| Fully connected 1		| input 400, output 120											|
| RELU					|	Activation											|
| Dropout				|	keep_prob=0.5											|
| Fully connected 2		| input 120, output 84											|
| RELU					|	Activation											|
| Dropout				|	keep_prob=0.5											|
| Fully connected 3 | input 84, output 43 |
| Softmax | |

#### 3. Describe how you trained your model. The discussion can include the type of optimizer, the batch size, number of epochs and any hyperparameters such as learning rate.

To train the model, I used:

* Optimizer: Adam optimizer
* batch size = 128
* learning rate = 0.001
* epochs = 50

#### 4. Describe the approach taken for finding a solution and getting the validation set accuracy to be at least 0.93. Include in the discussion the results on the training, validation and test sets and where in the code these were calculated. Your approach may have been an iterative process, in which case, outline the steps you took to get to the final solution and why you chose those steps. Perhaps your solution involved an already well known implementation or architecture. In this case, discuss why you think the architecture is suitable for the current problem.

My final model results were:

* training set accuracy of 0.99
* validation set accuracy of 0.98
* test set accuracy of 0.95

If an iterative approach was chosen:

* What was the first architecture that was tried and why was it chosen?
  * At first, I tried LeNet-5[1]. Because, this architecture works really fast and really well, allowing us to train our model in a short period of time. 
* What were some problems with the initial architecture?
  * accuracy of the validation did not improve(=over-fitting)
* Which parameters were tuned? How were they adjusted and why?
  * I increased batch size to speed up learning
  * I increased epochs to improve accuracy
* What are some of the important design choices and why were they chosen? For example, why might a convolution layer work well with this problem? How might a dropout layer help with creating a successful model?
  * I think that convolution layer can extract features(edge, change of contrast, etc...) to recognize traffic sign.
  * I used dropout to avoid over-fitting.
  * I observed the curve of accuracy(training, validation, test) to confirm the progress of learning.

![curve_of_accuracy][curve_of_accuracy]

### Test a Model on New Images

#### 1. Choose five German traffic signs found on the web and provide them in the report. For each image, discuss what quality or qualities might be difficult to classify.

Here are five German traffic signs that I found on the web:

| <img src="./test_images/general_caution.jpg" width="120px"> | <img src="./test_images/no_entry.jpg" width="120px"> | <img src="./test_images/road_work.jpg" width="120px"> | <img src="./test_images/speedlimit_30km.jpg " width="120px"> | <img src="./test_images/stop.jpg" width="120px"> |
|:------:|:------:|:------:|:------:|:------:|
| General caution | No entry | Road work | Speed limit (30km/h) | Stop |

- The 1st image might be difficult to classify because this image has rotatation.
- The 4th image might be difficult to classify because a part of the sign is missing.

#### 2. Discuss the model's predictions on these new traffic signs and compare the results to predicting on the test set. At a minimum, discuss what the predictions were, the accuracy on these new predictions, and compare the accuracy to the accuracy on the test set (OPTIONAL: Discuss the results in more detail as described in the "Stand Out Suggestions" part of the rubric).

Here are the results of the prediction:

| Image	label				| Prediction									| Result |
|:---------------------:|:---------------------------------------------:|:------:|
| General caution		| General caution   							| PASSED |
| No entry				| No entry 										| PASSED |
| Road work				| Road work										| PASSED |
| Speed limit (30km/h)	| Speed limit (30km/h)							| PASSED |
| Stop					| Stop											| PASSED |

The model was able to correctly guess 5 of the 5 traffic signs, which gives an accuracy of 1.0.  
This compares favorably to the accuracy on the test set accuracy of 0.95.

#### 3. Describe how certain the model is when predicting on each of the five new images by looking at the softmax probabilities for each prediction. Provide the top 5 softmax probabilities for each image along with the sign type of each probability. (OPTIONAL: as described in the "Stand Out Suggestions" part of the rubric, visualizations can also be provided such as bar charts)

The code for making predictions on my final model is located in the 11th cell of the Ipython notebook.

```python
with tf.Session() as sess:
        saver.restore(sess, tf.train.latest_checkpoint('.'))
        predictions = sess.run(tf.argmax(logits, 1), feed_dict={x: test_images_n, keep_prob: 1.0})
```

For the first image, the model is relatively sure that this is a `General caution`(probability of 1.00), and the image does contain a `General caution`. The top five soft max probabilities were as follows.

| Probability			| Prediction									| 
|:---------------------:|:---------------------------------------------:| 
| 1.00000000e+00  | General caution								| 
| 7.01177874e-12  | Traffic signals								|
| 3.46768413e-25  | Bumpy road									|
| 2.88710889e-25  | Road narrows on the right						|
| 2.91583627e-29  | No vehicles      								|

For the second image, the model is relatively sure that this is a `No entry`(probability of 0.99), and the image does contain a `No entry`. The top five soft max probabilities were as follows.

| Probability         	|     Prediction	        					| 
|:---------------------:|:---------------------------------------------:| 
| 9.99989629e-01  | No entry										| 
| 7.29284375e-06  | Keep right									|
| 2.32902880e-06  | Stop											|
| 6.80126675e-07  | Yield											|
| 4.36696617e-08  | Vehicles over 3.5 metric tons prohibited |

### (Optional) Visualizing the Neural Network (See Step 4 of the Ipython notebook for more details)
#### 1. Discuss the visual output of your trained network's feature maps. What characteristics did the neural network use to make classifications?

Here are the visualization of features for one of the German traffic signs(Image 1) :

![Feature Map][feature_map]

I think that the first hidden layer of the trained neural network gets activated by edges on the image or changes in the contrast.

### Reference
- [1] [Y. LeCun, L. Bottou, Y. Bengio, and P. Haffner. Gradient-based learning applied to document recognition. Proceedings of the IEEE, november 1998.](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf)
- [2] [Road signs in Germany](https://en.wikipedia.org/wiki/Road_signs_in_Germany)
