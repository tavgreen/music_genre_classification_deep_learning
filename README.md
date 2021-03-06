# Music Genre Classification using Deep Convolutional Neural Network

## Description ##
In this tutorial, we will try to classify music genre using Deep Convolutional Neural Network(CNN) in music file. CNN is powerfull as image classification model, so why do we need CNN in music that literally sound? the answer is we can extract features from music sound like **spectogram** that can be 2D (time, frequency) like images. so lets get started.

## Method ##
There are 5 steps to do in performing music genre classification:

**1. Obtain music data**

you can obtain music data from [Million Song Dataset](https://labrosa.ee.columbia.edu/millionsong/) or other sources, but be carefull of music licensed. In this project, i used music from Indonesian traditional music that consists of two classes: Sundanese music and Minang music. both of music have 100 music file for training, 10 music file for validation and 2 music file for testing.

**2. Extract features from music data**

After obtaining music file, extract *Spectogram* feature that will be used in this tutorial. in the other case, you can choose MFCC. here example Spectogram extracted from [Librosa](https://librosa.github.io). This data will be entered into Convolutional Neural Network (CNN) model.
![Fig.1](https://librosa.github.io/librosa/_images/librosa-feature-melspectrogram-1.png)

Librosa can be used to extract features like MFCC or spectogram 
```python
x_, fs = librosa.load(music, duration=120) #load from music data. set duration to 120 second
s = librosa.feature.melspectrogram(x_, fs) #change into melspectogram
```
After extracting feature, save into Numpy Npz file 
```python
np.savez('song.npz',x_train= np.array(x_train), y_train= np.array(y_train),x_val= np.array(x_val), y_val= np.array(y_val),x_test= np.array(x_test), y_test= np.array(y_test))
```
Change into normalized format as Keras can read:
```python
#Assumption: you have already music data stored in npz
x_train, y_train, x_val, y_val, x_test, y_test = readDataset(location) #load from your dataset
x_train = x_train.reshape(x_train.shape[0], 128, 5168, 1) #reshape x_train into: (num of data, 120,180,20)
x_val = x_val.reshape(x_val.shape[0], 128, 5168, 1) #reshape x_train into: (num of data, 120,180,20)
x_train = x_train.astype('float32') / 255. #normalization of x_train
x_val = x_val.astype('float32') / 255. #normalization of x_train		
y_train = np_utils.to_categorical(y_train, 3)
y_val = np_utils.to_categorical(y_val, 3)
```

**3. Train Model**

Train your music data in Spectogram form with Convolutional Neural Network (CNN). The architecture of CNN can be seen below:
![Fig.1](https://raw.github.com/tavgreen/landuse_classification/master/file/cnn.png?raw=true "Auto Encoder") 

we can see from above architecture, there are several layer consist in CNN like Input layer, Convolutional Layer, Subsampling/Pooling Layer, Fully Connected Layer and so on. We did some modification in typical CNN like:
- Input data are 100 spectograms for Sundanese and Minang Music respectively. The size of each Spectograms was modified into 128x5168.
- We use Sequential model based on Cho et al references. below is modification from original one(data will be flow through Sequential model from top to bottom):

|   Layer  |     Shape     |
|----------|:-------------:|
|  Conv2D  |    3x3x64     |
|BatchNorm |               |
|Activation|     RELU      |
|MaxPooling|      2x4      |
| Dropout  |     0.25      |
|  Conv2D  |    3x3x128    |
|BatchNorm |               |
|Activation|     RELU      |
|MaxPooling|      2x4      |
| Dropout  |     0.25      |
|  Conv2D  |    3x3x64     |
|BatchNorm |               |
|Activation|     RELU      |
|MaxPooling|      3x5      |
| Dropout  |     0.25      |
|  Conv2D  |    3x3x32     |
|BatchNorm |               |
|Activation|     RELU      |
|MaxPooling|      2x4      |
| Dropout  |     0.25      |
|FullyConn |               |
| Softmax  |               |

```python
x = Conv2D(64, (3, 3), padding='same')(x)
x = BatchNormalization(axis=3, name='bn1')(x)
x = Activation('relu')(x)
x = MaxPooling2D(pool_size=(2, 4))(x) #64x1296
x = Dropout(0.25)(x)
x = Conv2D(128, (3, 3), padding='same')(x)
x = BatchNormalization(axis=3, name='bn2')(x)
x = Activation('relu')(x)
x = MaxPooling2D(pool_size=(2, 4))(x) #32x324
x = Dropout(0.25)(x)
x = Conv2D(128, (3, 3), padding='same')(x)
x = BatchNormalization(axis=3, name='bn3')(x)
x = Activation('relu')(x)
x = MaxPooling2D(pool_size=(3, 5))(x)#16x81
x = Dropout(0.25)(x)		
x = Conv2D(64, (3, 3), padding='same')(x)
x = BatchNormalization(axis=3, name='bn4')(x)
x = ELU()(x)#x = (Activation('relu'))
x = MaxPooling2D(pool_size=(2, 4))(x) #8x27
x = Dropout(0.25)(x)
x = Conv2D(64, (3, 3))(x)
x = BatchNormalization(axis=3, name='bn5')(x)
x = Activation('relu')(x)#x = (Activation('relu'))
x = MaxPooling2D(pool_size=(2, 4))(x) #4x9
x = Dropout(0.25)(x)
x = Flatten()(x)
x = Dense(3, activation='softmax')(x)
model = Model(spectogram_input,x)
opt = keras.optimizers.rmsprop(lr=0.0001, decay=1e-6)
model.compile(loss='categorical_crossentropy',optimizer=opt,metrics=['accuracy'])
model.fit(self.x_train, self.y_train,batch_size=self.batch_size,epochs=self.epochs,validation_data=(self.x_val, self.y_val),shuffle=True)
				
```

**4. Test Model**

After obtaining a model from train process, test model that have not seen in training data will be performed. If the model can predict testing data based on Ground Truth, the result is good otherwise, tuning your model.
```python
y_pred = model.predict(x_test)
print('\n', sklearn.metrics.classification_report(np.where(y_test > 0)[1],np.argmax(y_pred, axis=1),target_names=list(maps.values())), sep='')
```

**5. Measure performance**

Perform measurement performance by using Accuracy, Precision, Recall and F1 score. Classification can be measured by utilizing 
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

## Result ##
### Model Performance ###
After performing training data to Sundanese and Minang song dataset with 10 epochs, validation accuracy reach 75% with loss function as follow:

![Fig.1](https://raw.github.com/tavgreen/music_genre_classification_deep_learning/master/file/loss.png?raw=true "Loss Function") 

we test model on 4 data: 2 Sundanese song and 2 Minang Song, the results are:

|           |precision|recall|f1-score|support|
|-----------|:-------:|:----:|:------:|:-----:|
|Sunda Song |   0.67  | 1.00 |  0.80  |   2   |
|Padang Song|   1.00  | 0.50 |  0.67  |   2   |
|avg / total|   0.83  | 0.75 |  0.73  |   4   |

### Improvement ###
The result is still not good because overfitting happened. Fine tuning of model, increasing train and validation data size can be a solution for improvement.

## References ##
- [Music Information Retrieval](http://musicinformationretrieval.org)
- [Blogg Keras](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html)
- Choi et al. 2016. Automatic Tagging using Deep Convolutional Neural Network
- [Million Song Dataset](https://labrosa.ee.columbia.edu/millionsong/)
