---

output: html_document
---
# Machine Learning 
Coursera Project  
---
"
author: "Nagesh Madhwal"
date: "November 22, 2015"

---

## Background to the Project

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about ersonal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).  


## Goal For the Project

Use the Training data to train a Machine Learning Model to correctly predict the manner in which the test subjects did the activity. Twenty cases are provided in the Test data for which prediction has to be done. A report has to be created on the approach & methodology including an estimation of the out of sample error.

## Our Approach 

Quick analysis of the training data shows that there are 160 variables and 19622 observations. As such Big Data Analysis would be focused on using the richess of the entire data set but given the humble capability of my laptop cleaning up the data & feature selection would be a key first step to make the this a manageable project. Following are the actiivities that will be carried out in this project:

1) Feature Selection
2) Initial Evaluation of multiple models & shortlist to one
3) Tune the chosen Model on the full Data Set
4) Submit Predictions

## Feature Selection

```{r}
Fileurl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
Traindata <- read.csv(Fileurl)
## Dimensions of the Training data
dim(Traindata)
```


Initial Exploratory analysis tells us that there are multiple columns with NA values, empty columns & columns with undefined values (DIV/0!). So we will remove these from the Training Data.

```{r echo = FALSE}
projtrain <- read.csv("pml-training.csv")
projtrain2 <- projtrain
projtrain2[projtrain2 == "#DIV/0!" | projtrain2 == ""] <- NA
inttrainset <- projtrain2[, !(colMeans(is.na(projtrain2)) > 0.5)]
inttrainset <- inttrainset[complete.cases(inttrainset), ]

library(mlbench)
library(caret)
correlationMatrix <- cor(inttrainset[,8:59])
highlyCorrelated <- findCorrelation(correlationMatrix, cutoff=0.75)
filteredDescr <- inttrainset[,-(highlyCorrelated + 7)]
filteredDescr1 <- filteredDescr[, -c(1,3,4,5)]
filteredDescr2 <- filteredDescr1[, -2]
intrain <- createDataPartition(y = filteredDescr2$classe, p = 0.7, list = FALSE)
mytrain <- filteredDescr2[intrain, ]
mytest <- filteredDescr2[-intrain, ]
testset <- read.csv("pml-testing.csv")
library(foreach)
library(doParallel)
registerDoParallel(4)
##output <- train(classe ~ ., data = dataSet, trControl = trainControl(method = "cv", number = 3))
##importance <- varImp(output, scale=FALSE)
##print(importance)
##plot(importance)
```



```{r eval=FALSE}
set.seed(1001)
modelrf <- train(classe ~ ., data = mytrain, trControl = trainControl(method = "cv", number = 10))
importance <- varImp(output, scale=FALSE)
print(importance)
plot(importance)
confusionMatrix(modelrf)
output$finalModel
predrf <- predict(modelrf, newdata = mytest)
compmytest <- mytest
compmytest$predright <- predrf == compmytest$classe
sum(compmytest$predright)/ nrow(compmytest)
```


```{r eval=FALSE}
## for future research
##control <- trainControl(method="repeatedcv", number=10, repeats=3)
##model <- train(diabetes~., data=PimaIndiansDiabetes, method="lvq", preProcess="scale", trControl=control)
##
## RFE - Recursive feature elimination
control <- rfeControl(functions=rfFuncs, method="cv", number=10)
results <- rfe(filteredDescr[,1:37], filteredDescr[,38], sizes=c(1:37), rfeControl=control)
plot(results, type=c("g", "o"))
```


```{r eval=FALSE}
set.seed(1001)
control <- trainControl(method="repeatedcv", number=10, repeats=3)
modellvq1 <- train(classe ~ ., data = mytrain, method="lvq", trControl=control, tuneLength= 5)
predlvq1 <- predict(modellvq1, newdata = mytest)
compmytestlvq <- mytest
compmytestlvq$predright <- predlvq1 == compmytestlvq$classe
sum(compmytestlvq$predright)/ nrow(compmytestlvq)
```


```{r eval=FALSE}
## with Preprocess
set.seed(1001)
control <- trainControl(method="repeatedcv", number=10, repeats=3)
modellvq2 <- train(classe ~ ., data = mytrain, method="lvq", preProcess="scale", trControl=control, tuneLength = 5)
predlvq2 <- predict(modellvq2, newdata = mytest)
compmytestlvq2 <- mytest
compmytestlvq2$predright <- predlvq2 == compmytestlvq2$classe
sum(compmytestlvq2$predright)/ nrow(compmytestlvq2)

## LVQ is not the right approach for this data set
```


```{r eval=FALSE}
set.seed(1001)

fitControl <- trainControl(method = "repeatedcv", number = 10, repeats = 10)
modelgbm <- train(classe ~., data = mytrain, methos = "gbm", trControl = fitControl, verbose = FALSE)
```

You can also embed plots, for example:

```{r, echo=FALSE}
plot(cars)
```

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
