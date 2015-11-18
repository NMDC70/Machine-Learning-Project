---
output: html_document
---
# Machine Learning Project
Coursera Project Machine Learning 
---
title: "Machine Learning Project"
author: "NMDC"
date: "November 17, 2015"

---

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:


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
