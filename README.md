# Machine Learning for Weight Lifting Exercises
Yaakov Miller  
July 9, 2017  



## Overview

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 

In this project, the goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E).

Class A corresponds to the specified execution of the exercise, while the other 4 classes correspond to common mistakes.

More information is available from the website here: <http://groupware.les.inf.puc-rio.br/har> (see the section on the Weight Lifting Exercise Dataset).

## The Data


```r
# Requirments
library(caret)
library(rattle)
library(rpart)
library(rpart.plot)
library(randomForest)
library(repmis)

training <- read.csv("pml-training.csv", na.strings = c("NA", ""))
testing <- read.csv("pml-testing.csv", na.strings = c("NA", ""))


# Eliminating NA's
training <- training[, colSums(is.na(training)) == 0]
testing <- testing[, colSums(is.na(testing)) == 0]

# Trimming non-useful columns
training <- training[, -c(1:7)]
testing <- testing[, -c(1:7)]

# Training dataset size (Observations - Variables)
dim(training)
```

```
## [1] 19622    53
```

```r
# Testing dataset size (Observations - Variables)
dim(testing)
```

```
## [1] 20 53
```

## Splitting the data


The original trainig data set will be splitted 60/40 for training and validation.



```r
set.seed(10000) 
inTraining <- createDataPartition(training$classe, p = 0.6, list = FALSE)
training0 <- training[inTraining, ]
validation0 <- training[-inTraining, ]
# Training dataset size (Observations - Variables)
dim(training0)
```

```
## [1] 11776    53
```

```r
# Validation dataset size (Observations - Variables)
dim(validation0)
```

```
## [1] 7846   53
```

## Predictions models

The following algorithms will be studied:

* Classfiication Trees &
* Random Forest
        
To speed-up calculations the k-fold cross validation was set to 5-fold cross validation.


```r
control <- trainControl(method = "cv", number = 5)
```


### Classification Trees model


```r
fit_rpart <- train(classe ~ ., data = training0, method = "rpart", 
                   trControl = control)

print(fit_rpart, digits = 4)
```

```
## CART 
## 
## 11776 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 9420, 9420, 9421, 9420, 9423 
## Resampling results across tuning parameters:
## 
##   cp       Accuracy  Kappa  
##   0.03571  0.5104    0.36059
##   0.06087  0.4426    0.25279
##   0.11640  0.3330    0.07406
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was cp = 0.03571.
```

```r
fancyRpartPlot(fit_rpart$finalModel)
```

![](project_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
# Model Validation 
# predict outcomes using validation set
predict_rpart <- predict(fit_rpart, validation0)
# Show prediction result
conf_rpart <- confusionMatrix(validation0$classe, predict_rpart)
```

### Random Forest model


```r
fit_rf <- train(classe ~ ., data = training0, method = "rf", 
                trControl = control)
print(fit_rf, digits = 4)
```

```
## Random Forest 
## 
## 11776 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 9420, 9421, 9421, 9421, 9421 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy  Kappa 
##    2    0.9893    0.9865
##   27    0.9883    0.9852
##   52    0.9802    0.9750
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 2.
```

```r
# predict outcomes using validation set
predict_rf <- predict(fit_rf, validation0)
# Model Validation 
# Show prediction result
(conf_rf <- confusionMatrix(validation0$classe, predict_rf))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2230    1    0    0    1
##          B   16 1498    4    0    0
##          C    0   20 1346    2    0
##          D    0    0   24 1260    2
##          E    0    0    3    4 1435
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9902          
##                  95% CI : (0.9877, 0.9922)
##     No Information Rate : 0.2863          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9876          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9929   0.9862   0.9775   0.9953   0.9979
## Specificity            0.9996   0.9968   0.9966   0.9960   0.9989
## Pos Pred Value         0.9991   0.9868   0.9839   0.9798   0.9951
## Neg Pred Value         0.9971   0.9967   0.9952   0.9991   0.9995
## Prevalence             0.2863   0.1936   0.1755   0.1614   0.1833
## Detection Rate         0.2842   0.1909   0.1716   0.1606   0.1829
## Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
## Balanced Accuracy      0.9963   0.9915   0.9870   0.9957   0.9984
```

## Results

### Classification Trees model

The correct Classe for the testing data are given by the result of the quiz 
with 100% score. 



```r
Correct_Classe = LETTERS[c(2,1,2,1,1,5,4,2,1,1,2,3,1,5,5,1,2,2,2)]

(accuracy_rpart <- conf_rpart$overall[1])
```

```
##  Accuracy 
## 0.4887841
```

```r
(predict(fit_rpart, testing))
```

```
##  [1] C A C A A C C A A A C C C A C A A A A C
## Levels: A B C D E
```

```r
Correct_Classe
```

```
##  [1] "B" "A" "B" "A" "A" "E" "D" "B" "A" "A" "B" "C" "A" "E" "E" "A" "B"
## [18] "B" "B"
```

### Random Forest model


```r
(accuracy_rf <- conf_rf$overall[1])
```

```
##  Accuracy 
## 0.9901861
```

```r
(predict(fit_rf, testing))
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

```r
Correct_Classe
```

```
##  [1] "B" "A" "B" "A" "A" "E" "D" "B" "A" "A" "B" "C" "A" "E" "E" "A" "B"
## [18] "B" "B"
```

Random forest provides a better solution to our problem with a medium cost in 
computation resources.
