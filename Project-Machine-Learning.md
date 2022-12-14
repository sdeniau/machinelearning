## Project objective

In this analysis we try to predict the manner in which people exercise
based on accelerometers on the belt, forearm, arm, and dumbell of 6
participants from the Weight Lifting Exercise Dataset using different
machine learning algorithms.

Six participants were asked to perform barbell lifts correcty and
incorrectly in five different manners wearing fitness trackers like
Jawbone Up, Nike FuelBand, and Fitbit in this dataset. The data gained
from this devices is used to train the models.

## Data downloading

``` r
#Download the files
train_url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
test_url  <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
train_file <- "pml-training.csv"
test_file <- "pml-testing.csv"
download.file(train_url,train_file)
download.file(test_url,test_file)

#Extract the datasets
training <-read.csv(train_file, na.strings=c("NA","#DIV/0!", ""))
testing <-read.csv(test_file , na.strings=c("NA", "#DIV/0!", ""))
```

## Data cleaning

We do three different cleaning of the training and testing datasets:

1- we remove near zero variance variables

2- we remove variables with more than 95% NAs

3- we remove non pertinent data i.e. the seven firsts columns of the
datasets

``` r
#Clean datasets - near zero variance 
non_zero_var <- nearZeroVar(training)
training_clean <- training[,-non_zero_var]
testing_clean <- testing[,-non_zero_var]

#Clean datasets - more than 95% NA
non_na <- colSums(is.na(training_clean))/nrow(training_clean) < 0.95
training_clean <- training_clean[,non_na]
testing_clean <- testing_clean[,non_na]

#Clean datasets - useless variables
training_final   <-training_clean[,-c(1:7)]
testing_final <-testing_clean[,-c(1:7)]
```

## Model partition and cross validation

We decide to split training dataset (70% training, 30% testing).

``` r
#Partitioning
set.seed(666)
sample <- createDataPartition(y=training_final$classe, p=0.7, list=FALSE)
traindata <- training_final[sample, ]
testdata <- training_final[-sample, ]
```

Regarding cross validation we choose K=5.

``` r
#Cross validation with K = 5
fit_control <- trainControl(method="cv", number = 5)
```

## Models building

As it is supervised learning for classification xxxx with labeling, we
decide to choose 4 different types of model:

1- Decision tree `fit_dt`

2- Support Vector Machine `fit_smv`

3- Gradient boosting `fit_gbm`

4- Random Forest `fit_rf`

``` r
fit_dt <- train(
        classe ~ ., 
        data=traindata,
        trControl=fit_control,
        method="rpart")

fit_gbm <- train(
        classe ~ ., 
        data=traindata,
        trControl=fit_control,
        method="gbm",
        verbose=FALSE)

fit_rf <- train(
        classe ~ ., 
        data=traindata,
        trControl=fit_control,
        method="rf",
        ntree=100)

fit_svm <- train(
      classe ~ ., 
      data=traindata,
      trControl=fit_control,
      method="svmLinear")
```

## Models evaluation

We evaluate model accuracy thanks to `testdata` :

-   We first predict `classe` for each case

-   And then calculate the confusion matrix in order to compare
    accuracies

``` r
pred_dt <- predict(fit_dt, newdata=testdata)
confmatr_dt <- confusionMatrix(pred_dt, as.factor(testdata$classe))

pred_gbm <- predict(fit_gbm, newdata=testdata)
confmatr_gbm <- confusionMatrix(pred_gbm, as.factor(testdata$classe))

pred_rf <- predict(fit_rf, newdata=testdata)
confmatr_rf <- confusionMatrix(pred_rf, as.factor(testdata$classe))

pred_svm <- predict(fit_svm, newdata=testdata)
confmatr_svm <- confusionMatrix(pred_svm, as.factor(testdata$classe))


assessment <- data.frame(
        Model = c("Decision Tree","Support Vector Machine", "Gradient Boosting", "Random Forest"),
        Accuracy = rbind(confmatr_dt$overall[1], confmatr_svm$overall[1], confmatr_gbm$overall[1], confmatr_rf$overall[1])
)
```

       N° Model                   Accuracy
       -  ----------------------  ---------
       1  Decision Tree           0.5760408
       2  Support Vector Machine  0.7692438
       3  Gradient Boosting       0.9605777
       4  Random Forest           0.9933730
       -- ----------------------  ---------

With 99.3% of accuracy, Random Forest is the best model among the four.

    Confusion Matrix of Random Forest model:
                       Reference
                    A    B    C    D    E
               - ---- ---- ---- ---- ----
    Prediction A 1674   11    0    0    0
               B    0 1125    5    0    0
               C    0    2 1017   12    3
               D    0    0    3  951    0
               E    0    1    1    1 1079
               - ---- ---- ---- ---- ----

We will retain it and apply it for the prediction on the testing dataset

## Prediction

When we apply the model to the 20 case of the testing dataset we get:

``` r
pred_testing <- predict(fit_rf, newdata=testing_final)
```

    [1] B A B A A E D B A A B C B A E E A B B B
    Levels: A B C D E
