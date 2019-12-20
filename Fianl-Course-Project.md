Background
==========

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement – a group of enthusiasts who take measurements
about themselves regularly to improve their health, to find patterns in
their behavior, or because they are tech geeks. One thing that people
regularly do is quantify how much of a particular activity they do, but
they rarely quantify how well they do it. In this project, your goal
will be to use data from accelerometers on the belt, forearm, arm, and
dumbell of 6 participants. They were asked to perform barbell lifts
correctly and incorrectly in 5 different ways. More information is
available from the website here:
<http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har>
(see the section on the Weight Lifting Exercise Dataset).

Loading Required packages for Predictions
-----------------------------------------

Data Loading and Pre-Processing
===============================

Loading Training and Test data sets
-----------------------------------

    training <-read.csv('pml-training.csv')
    Final_Quiz <- read.csv('pml-testing.csv')

Clean data sets
---------------

### remove NA vaules and unneeded variables and limiting my training set to the same variables avaiable in Final\_Quiz set

    Final_Quiz <- Final_Quiz %>% dplyr::select_if(~ !any(is.na(.))) %>% dplyr::mutate(classe = "")
    training <- training %>% dplyr::select_if(colnames(.) %in% colnames(Final_Quiz))

    training <- training[,-c(1:7)] ##the first 7 columns contains reference variables to individual with no relatiopn to our prediciton
    Final_Quiz <- Final_Quiz[,-c(1:7)]

Prediction Models
-----------------

### Creating a training and tests data set within my training df (75% / 25%)

    set.seed(1776)
    inTrain <- with(training, createDataPartition(classe, p = 0.75)[[1]])

    train <- training[inTrain, ]
    test  <- training[-inTrain, ]

### Initally I used Random Forest due to its high accuracy rates.

    control <- trainControl(method = 'cv', number = 3)
    model_rf <- train(classe ~ ., data = train, method = "rf", metric = 'Accuracy', trControl = control)
    pred_rf <- predict(model_rf, test)
    confusionMatrix(test$classe, pred_rf)

![Alt text](RF-Output.jpg)

### I then ran a Gradient Boosting Machines also due to its ability to get high accuracy rates.

    fitControl <- trainControl(method = "cv", number = 3)

    model_gdm <- train(classe ~ ., method = "gbm", data = train, trControl = fitControl, verbose = FALSE)
    pred_gdm <- predict(model_gdm, test)
    confusionMatrix(test$classe, pred_gdm)

![Alt text](GMB%20-%20Output.jpg)

### Lastly, I uesed a Liner Discriminant Analysis. This method proved to process faster but with lower accuracy

    model_lda <- train(classe ~ ., method = "lda", data = train)
    pred_lda <- predict(model_lda, test)
    confusionMatrix(test$classe, pred_lda)

![Alt text](LDA%20-%20Output.jpg)

Random Forest has the highest Predition Accuracy. Hence, we will use this method for predicting our testing data base
=====================================================================================================================

    Final_pred <- predict(model_rf, newdata = testing)
    Final_pred

![Alt text](Final-Output.jpg)
