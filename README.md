# CAPSTONE PROJECT

# Differencial Diagnosis Between COVID, Viral Pneumonia, and Tuberculosis



The outbreak of Novel Coronavirus (SARS-COV-2) has taken over all our lives ever since the pandemic hit last year. The difficult part at the beginning of the outbreak was to detect the virus since the symptoms are shared among other common desieses and infections. One of these include pneumonia.

> Deep learning AI models have been used in the past to detect pneumonia, and have been reported to gain about 90% accuracy. [1]

While discussing about pneumonia, it is paramount to understand that pneumonia is often misdiagnosed as tuberculosis. [2]

This project explores building different models that would detect if a lung X-Ray correctly diagnoses COVID-19, Pneumonia, Tuberculosis, Viral Pneumonia, or if it is a normal Chest X-ray. It also explores the varios regions of the x-ray that the model is looking in for an accurate diagnosis.

> Accuracy, Precision, and Recall have been used as the deciding factor for the best model

Precision would be important when the model is used to distinguish between the different desieses whoes diagnosis are often confused with another, while recall becomes significant while determining if the desiese is present at all.


## Data

The Data for this project has been generated by combining the X-Ray Images from 2 sources. One dataset consisted of the X-ray Images differentiating COVID, Viral Pneumonia and Normal Chest X-rays, while the other data set had chest x-rays for TB and normal. The normal x-rays from each data set were not combined since there could be a possibility that the X-rays not showing pneumonia could have TB and vis-a-versa. 
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Data.png">
<div align="center"> Figure 1: Dataset </div>
<br />

The images were then renamed and combined for preprocessing using simple coding in Python. To load the data in google colab notebook, file path data frame was also created using python.

Figure 2 shows the class imbalance in the data set. To minimie the effect of class imbalance, the models were built with the class weights parameter.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Class_Imbalance.png">
<div align="center"> Figure 2: Class Imbalance </div>
<br />

The ImageDataGenerator object was created with rescalling of 1./255, after which the images were loaded in using keras's flow_from_dataframe function, which yeilded 4687 images classified into 5 classes. Since one of the images in the file was corrupted, that image is omited out of any further process.

The train-val-test split consist of 3795,422, and 469 images respectively.


## Methodology

- Data Pre-processing
- Explore and optimize various pre-trained models to use as base model to build on.
- Finalize  the best model based on performance measures (Accuracy, Precision, Recall) and model explanation


## Dummy Model

A dummy model is first created using scikit learn's DummyClassifier to compare to and improve upon the results. The DummyClassifier bore very poor results with only 18% accuracy and 25% precision.

Next, a Basic Keras CNN model was built to analyse the results.


## Basic Keras CNN

The code snippet below shows the model structure used to build the basic CNN Model. Adam optimizer was used to compile the model.

```
#Building the basic CNN model

basic_keras=Sequential()
basic_keras.add(tf.keras.Input(shape=X_train.shape[1:]))
basic_keras.add(Conv2D(32,(3,3),activation='relu'))
basic_keras.add(MaxPooling2D((2,2)))
basic_keras.add(Conv2D(64,(3,3),activation='relu'))
basic_keras.add(MaxPooling2D((2,2)))
basic_keras.add(Flatten())
basic_keras.add(layers.Dense(256,activation='relu'))
basic_keras.add(layers.Dense(5,activation='softmax'))

basic_keras.compile(loss='categorical_crossentropy',
              optimizer=optimizers.Adam(lr=0.001),
              metrics=['acc'])
```

While the results seemed promising for this model, with high precision (92%), recall (91%) and accuracy (94%), the training and validation curves are far apart and it does not perform as well with the validation data set. The model is overfitted and would not give consistent results with any unknown data.

> **3 Pre-trained models were then explored to use as a base model to build on. Various optimizers, number of layers, drop out, and LeakyReLu layers were also explored using different parameters for optimization**


## Best Pre-Trained Model Transfer VGG16 - Accuracy 97%

The parameters which resulted in the best model, out of 6, with pre-trained VGG16 base were:

> Additional layer on top of the base model: 1 dense layer, activation - 'relu'\
Optimizer: Adam
epochs: 50
batch size: 64
callbacks: Early stopping with patience 3
learning rate: 0.001
loss: categorical crossentropy
metric: accuracy
final layer: dense, activation 'softmax'

Table 1 shows the classifcation report and Figure 3 shows the confusion matrix for this model when run on the test data.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Classification_report_vgg16.png">
<div align="center"> Table 1: Classification Report-VGG16 </div>
<br />
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/cm_vgg16.png">
<div align="center"> Figure 3: Confusion Matrix - VGG16 </div>
<br />

Figure 4 shows the train and validation accuracy and loss plots while the model was being trained.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Acc_loss_vgg16.png">
<div align="center"> Figure 4: Train-Validation Accuracy and Loss - VGG16 </div>
<br />

The plot shows that the model is likely being overfitted and needs additional tweeking in the design and number of layers to stop the model from being over-trained.

The model can now be explained using the lime package. It explains the model by highlighting the area of the image that was used to classify. This helps in making better judgements and confidence that the model will accurately classify a new image that was not used in training by looking at the right areas. Figure 5 shows the lime explaination of all the bad predictions that were made using this model. The total number bad predictions were 15 out of the 422 test images fed to the model.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/bad_pred_vgg16.png">
<div align="center"> Figure 5: Bad Prediction Explanation - VGG16 </div>
<br />

Figure 6 shows the lime explaination of randomn 5 images from the test set that were classified using this model.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/lime_vgg16.png">
<div align="center"> Figure 6: Model Explanation - VGG16 </div>
<br />

Based on these explainations, we may find that most of the bad predictions are not looking in the right areas. however, the the probability of true negative is looking inthe right area almost every time.

To improve these explainations, it thus becomes important to utilize a mask to help the model look into the correct area. While the model acheived a 96% accuracy, it would be more trustable with an addition of masks to further guide the model


## Best Pre-trained Model Tranfer Xception - Accuracy 94%

The parameters which resulted in the best model with pre-trained Xception base were:

> Additional layer on top of the base model: 4 dense layer, activation - 'relu', LeakyReLu, Dropout\
Optimizer: Adam
epochs: 50
batch size: 64
callbacks: Early stopping with patience 3
learning rate: 0.001
loss: categorical crossentropy
metric: accuracy
final layer: dense, activation 'softmax'

Table 2 shows the classifcation report and Figure 7 shows the confusion matrix for this model when run on the test data.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Classification_report_xc.png">
<div align="center"> Table 2: Classification REport - Xception </div>
<br />
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/cm_xc.png">
<div align="center"> Figure 7: Confusion Matrix Xception </div>
<br />

Figure 8 shows the train and validation accuracy and loss plots while the model was being trained.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Acc_loss_xc.png">
<div align="center"> Figure 8: Train-Validation Accuracy and Loss - Xception </div>
<br />

Based on the precision and recall values, the model would need to be optimized to make Viral pneumonia classification more trustable.


## Best Pre-trained Model Tranfer DenseNet121 - Accuracy 95%

The parameters which resulted in the best model with pre-trained DenseNet121 base were:

> Additional layer on top of the base model: 4 dense layer, activation - 'relu', LeakyReLu, Dropout\
Optimizer: Adam
epochs: 50
batch size: 64
callbacks: Early stopping with patience 3
learning rate: 0.001
loss: categorical crossentropy
metric: accuracy
final layer: dense, activation 'softmax'

Table 3 shows the classifcation report and Figure 9 shows the confusion matrix for this model when run on the test data.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Classification_report_dnet.png">
<div align="center"> Table 3: Classification Report - DenseNet121 </div>
<br />
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/cm_dnet.png">
<div align="center"> Figure 9: Confusion Matrix - DenseNet121 </div>
<br />

Figure 10 shows the train and validation accuracy and loss plots while the model was being trained.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/Acc_loss_dnet.png">
<div align="center"> Figure 10: Train-Validation Accuracy and Loss - DenseNet121 </div>
<br />

Based on these plots, and the performance measures, we can say that out model is doing a good job in accurately classifying the diagnosis, and with high precission and recall for each class.

Although the accuracy is not as high as the VGG16 model, this model is more trustable since it is not overfit, and has high precision and recall values for all classes.

To further strenthen our case, lime explanation was plotted for bad-predictions (Figure 11) and 5 random images (Figure 12) from the test dataset.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/bad_pred_dnet.png">
<div align="center"> Figure 11: Bad Predictions - DenseNet121 </div>
<br />
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-capstone-project-v2-onl01-dtsc-pt-041320/master/exp_dnet.png">
<div align="center"> Figure 12: Lime Explanation - DenseNet121 </div>
<br />

Comparing these explanations with VGG16 model, we have a clear winner here since the model generated with DenseNet121 looks into more relevant areas for all predictions compared to VGG16, which makes it more trustable.


## Recommendations

- With acceptabley high accuracy of more than 95%, and high precision and recall for covid and pneumonia patients, it is recommended to consider using an AI based model for accurate diagnosis.

- Looking at comparatively lower accuracy, precision and recall with TB patients, it is recommended to invest in further research in gathering more data and building further on the model to be able to use the model to detect TB.

- Although the model shows high accuracy, it is important and recommended to use a combination of other symptoms either by knowledge or integrating it with the model for better diagnosis and patient care.


## Future Work

- Building a Dashboard/Application that would accept any format of the Xray, preprocess and use the model for diagnosis, and include the area of the X-ray used for the result (explaination)

- Explore other models by building by scratch, and including image augmentation, and creating masks to improve the predictability of the model

- Include other symptoms in the model to aid in better diagnosis


## References
[1] https://towardsdatascience.com/detecting-covid-19-induced-pneumonia-from-chest-x-rays-with-transfer-learning-an-implementation-311484e6afc1

[2] Aiyesha Sadiya, Anusha V Illur, Aekhata Nanda, Eshwar Rao, Vidyashree K P, Mansoor Ahmed. *Differential Diagnosis of Tuberculosis and
Pneumonia using Machine Learning*