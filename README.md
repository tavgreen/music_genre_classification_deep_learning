# Music Genre Classification using Deep Convolutional Neural Network

## Description ##
In this tutorial, we will try to classify music genre using Deep Convolutional Neural Network(CNN) in music file. CNN is powerfull as image classification model, so why do we need CNN in music that literally sound? the answer is we can extract features from music sound like **spectogram** that can be 2D (time, frequency) like images. so lets get started.

## Method ##
not yet finished

## Program ##
not yet finished

## Result ##
## Model Performance ##
After performing training data to Sundanese and Minang song dataset with 10 epochs, validation accuracy reach 75% with loss function as follow:

we test model on 4 data: 2 Sundanese song and 2 Minang Song, the results are:

|           |precision|recall|f1-score|support|
|-----------|:-------:|:----:|:------:|:-----:|
|Sunda Song |   0.67  | 1.00 |  0.80  |   2   |
|Padang Song|   1.00  | 0.50 |  0.67  |   2   |
|avg / total|   0.83  | 0.75 |  0.73  |   4   |

## How to measure performance ##

- Precision can be calculated: TP / ( TP + FP )
- Recall can be calculated: TP / ( TP + FN )
- F1 Score can be Calculated: (2 * TP) / ( TP + FP ) + ( FP + FN )

TP: True Possitive (The num of Possitive Class that predicted true)
FP: False Possitive (The num of Possitive Class that predicted false)
TN: True Negative (The num of Negative Class that predicted true)
FN: False Negative (The num of Negative Class that predicted false)


Let say confusion matrix from 10 test case (5 sundanese and 5 minang):

|          |Sundanese|Minang|
|----------|:-------:|:----:|
|Sundanese |    5    |   0  |
|Minang    |    2    |   3  |

Above diagram mean: 5 Sundanese test case song when predicted by using model can produce 5 correct prediction. in the other side, 5 Minang test case song when predicted by using model can produce 3 correct prediction and 2 incorrect prediction.

- Precision(Sunda): TP / (TP + FP) = 5 / (5 + 0) = 1
- Precision(Minang): TP / (TP + FP) = 3 / (3 + 5) = 0.375
- Recall(Sunda): TP / (TP + FN) = 5 / (5 + 2) = 0.71
- Recall(Minang): TP / (TP + FN) = 3 / (3 + 0) = 1
- F1(Sunda): (2 * TP) / ( TP + FP ) + ( FP + FN ) = 10 / 12 = 0.83
- F1(Minang): (2 * TP) / ( TP + FP ) + ( FP + FN ) = 6 / 8 = 0.75
- F1: (1/2 * ((0.8) + (0.75))) = 0.775

## References ##
- [Music Information Retrieval](http://musicinformationretrieval.org)
- [Blogg Keras](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html)
- Choi et al. 2016. Automatic Tagging using Deep Convolutional Neural Network
