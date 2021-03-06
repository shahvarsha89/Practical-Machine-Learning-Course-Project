# Practical-Machine-Learning-Course-Project

---
title: "Practical Machine Learning Project : Prediction Assignment Writeup"
author: "Varsha Shah"
date: "Monday, January 09, 2017"
output: html_document
---

Step 1 : Environment Preparation
================================

```{r, echo=TRUE,include=FALSE,comment=NA,warning=FALSE,results='asis'}
rm(list=ls())# free up memory for the download of the data sets
setwd("C:/Users/CICAdmin/Desktop/varsha")
library(knitr)
library(caret)
library(rpart)
library(rpart.plot)
library(rattle)
library(randomForest)
library(corrplot)
set.seed(12345)
```

Step 2 : Data Loading and Cleaning
==================================

The next step is loading the dataset from the URL provided above. The training dataset is then partinioned in 2 to create a Training set (70\% of the data) for the modeling process and a Test set (with the remaining 30\%) for the validations. The testing dataset is not changed and will only be used for the quiz results generation.

```{r, echo=TRUE}
# download the datasets
training <- read.csv("C:/Users/CICAdmin/Desktop/varsha/pml-training.csv")
testing  <- read.csv("C:/Users/CICAdmin/Desktop/varsha/pml-testing.csv")

# create a partition with the training dataset 
inTrain  <- createDataPartition(training$classe, p=0.7, list=FALSE)
TrainSet <- training[inTrain, ]
TestSet  <- training[-inTrain, ]
dim(TrainSet)
```

```{r, echo=TRUE}
dim(TestSet)
```

Both created datasets have 160 variables. Those variables have plenty of NA, that can be removed with the cleaning procedures below. The Near Zero variance (NZV) variables are also removed and the ID variables as well.

```{r, echo=TRUE}
NZV <- nearZeroVar(TrainSet)
TrainSet <- TrainSet[, -NZV]
TestSet  <- TestSet[, -NZV]
dim(TrainSet)
```

```{r, echo=TRUE}
dim(TestSet)
```

Step 3 : Remove Variables that are almost NA
============================================

```{r, echo=TRUE}
na.data<- sapply(TrainSet , function(x)  mean(is.na(x)) > 0.95 )
TrainSet<-TrainSet[,na.data==FALSE]
TestSet<-TestSet[,na.data==FALSE]
dim(TrainSet)
```

```{r, echo=TRUE}
dim(TestSet)
```

Remove Identification VAribale 1 to 5

```{r, echo=TRUE}
TrainSet<-TrainSet[,-c(1:5)]
TestSet<-TestSet[,-c(1:5)]
dim(TrainSet)
```

```{r, echo=TRUE}
dim(TestSet)
```

With the cleaning process above, the number of variables for the analysis has been reduced to 54 only.

Step 4 : Correlation Analysis
=============================

A correlation among variables is analysed before proceeding to the modeling procedures.

```{r, echo=TRUE,fig.height=10,fig.width=10}
corMatrix<- cor (TrainSet[,-54])
corrplot(corMatrix,order="FPC",method="color",type="lower",tl.cex=0.8,tl.col=rgb(0,0,0))
graphics.off()
```

The highly correlated variables are shown in dark colors in the graph above. To make an evem more compact analysis, a PCA (Principal Components Analysis) could be performed as pre-processing step to the datasets. Nevertheless, as the correlations are quite few, this step will not be applied for this assignment.

Step : 5 Prediction Model Building
==================================

Three methods will be applied to model the regressions (in the Train dataset) and the best one (with higher accuracy when applied to the Test dataset) will be used for the quiz predictions. The methods are: Random Forests, Decision Tree and Generalized Boosted Model, as described below.

A Confusion Matrix is plotted at the end of each analysis to better visualize the accuracy of the models.

Method: Random Forest
---------------------

```{r, echo=TRUE}
set.seed(12345)
controlRF <- trainControl(method="cv", number=3, verboseIter=FALSE)
modFitRandForest <- train(classe ~ ., data=TrainSet, method="rf",
                          trControl=controlRF)
modFitRandForest$finalModel
```


```{r, echo=TRUE}
# prediction on Test dataset
predictRandForest <- predict(modFitRandForest, newdata=TestSet)
confMatRandForest <- confusionMatrix(predictRandForest, TestSet$classe)
confMatRandForest
```

```{r, echo=TRUE}
# plot matrix results
plot(confMatRandForest$table, col = confMatRandForest$byClass, 
     main = paste("Random Forest - Accuracy =",
                  round(confMatRandForest$overall['Accuracy'], 4)))
graphics.off()
```

Method: Decision Trees
----------------------

```{r, echo=TRUE}
set.seed(12345)
modFitDecTree <- rpart(classe ~ ., data=TrainSet, method="class")
fancyRpartPlot(modFitDecTree)
```

```{r, echo=TRUE}
predictDecTree <- predict(modFitDecTree, newdata=TestSet, type="class")
confMatDecTree <- confusionMatrix(predictDecTree, TestSet$classe)
confMatDecTree
```

```{r, echo=TRUE}
plot(confMatDecTree$table, col = confMatDecTree$byClass, 
     main = paste("Decision Tree - Accuracy =",
                  round(confMatDecTree$overall['Accuracy'], 4)))
graphics.off()
```

Method: Generalized Boosted Model
---------------------------------

```{r, echo=TRUE}
set.seed(12345)
controlGBM <- trainControl(method = "repeatedcv", number = 5, repeats = 1)
modFitGBM  <- train(classe ~ ., data=TrainSet, method = "gbm",
                    trControl = controlGBM, verbose = FALSE)
modFitGBM$finalModel
```

```{r, echo=TRUE}
predictGBM <- predict(modFitGBM, newdata=TestSet)
confMatGBM <- confusionMatrix(predictGBM, TestSet$classe)
confMatGBM
```


```{r, echo=TRUE}
plot(confMatGBM$table, col = confMatGBM$byClass, 
     main = paste("GBM - Accuracy =", round(confMatGBM$overall['Accuracy'], 4)))
graphics.off()
```

Applying the Selected Model to the Test Data
============================================

The accuracy of the 3 regression modeling methods above are:

   * Random Forest : 0.9963
   * Decision Tree : 0.7368
   * GBM : 0.9839
   
In that case, the Random Forest model will be applied to predict the 20 quiz results (testing dataset) as shown below.

```{r, echo=TRUE}
predictTEST <- predict(modFitRandForest, newdata=testing)
predictTEST
```
