# Practical-Machine-Learning
Practical Machine Learning

## Summary

A lot people nowadays use technical devices to track pattern of physical activity, however, rarely people track the quality of moving. This paper describes an approach using a machine learning method to predict the quality of certain exercises involving a weightlifting barbell by using supervised machine learning methods. Data has been made available from moving accelorators with an outcome classification of correct and incorrect lifts. 

## Approach 

First, data needs to be loaded into R and cleaned. 
```{r getting data, message=FALSE, warning=FALSE}
# load packages
library(caret)
library(rpart)
library(randomForest)

# load data from files
if(!file.exists("pml-training.csv")){
	download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", 
		destfile = "pml-training.csv")
}
if(!file.exists("pml-testing.csv")){
	download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", 
		destfile = "pml-testing.csv")
}
# load data
train <- read.csv("pml-training.csv", na.strings=c("NA","NaN","#DIV/0!", ""))
test<- read.csv("pml-testing.csv", na.strings=c("NA","NaN","#DIV/0!", ""))

```
Only complete variables will be used for building a model thus reducing the number of variables. Moreover, some irrelevant variables will be removed as they do not contain information on barbell lifting and thus would violate the prediction model.

```{r prepare}
#removing unusable columns
training <- train[,-nearZeroVar(train)]
training <- train[,-c(1:7)]

# removing columns with 60%+ NAs
insert<-colSums(is.na(training)) <= 0.6*nrow(training)
training<-training[, insert]

# setting up test set 
testing <- test[, names(test) %in% names(training)]

# convert class
training$classe <- as.factor(training$classe)

for(i in 1:(length(training)-1)){
  training[,i] <- as.numeric(training[,i])
  testing[,i] <- as.numeric(testing[,i])
}

```

# Cross validation
Cross validation is one of the high-standard methods to test accuracy of the predition model. Therefore, the training set needs to be split in another two parts, one for training the model and one for validation purpose before testing prediction accuary on test sample. 

```{r validation}

inTrain <- createDataPartition(y=training$classe, p=0.6, list=FALSE)
subsetTraining <- training[inTrain,]
subsetValidation <- training[-inTrain,]

#dim(testing)
#dim(validation)
#dim(trainset)
```
# Prediction models
To compare prediction models, firstly, random forest method is used, afterwards a generalized boosted regression model.

```{r prediction}
set.seed(123)
modelFit <- train(classe ~ ., method="rf", data=subsetTraining)
gbmmodelFit <- train(classe~., method = "gbm", data=subsetTraining, verbose=F)
```

Investigating final models by: 
```{r}
modelFit$finalModel
gbmmodelFit$finalModel
```
Next, the validation set is used to predict classe of lifts based on both models to test and compare accuarcy of both models.
```{r, message=FALSE}
predictedData_rf <- predict(modelFit, subsetValidation)
confusionMatrix(subsetValidation$classe, predictedData_rf)

predictedData_gbm <- predict(gbmmodelFit, subsetValidation)
confusionMatrix(subsetValidation$classe, predictedData_gbm)

```
Since the prediction model using random forest (accuracy levels: rf = 99,46% vs. gbm= 96.28%) seems to fit better, we use this model to predict values on the test data set.


```{r}
# final prediction
predictTest <- predict(modelFit, testing)
predictTest
```


## Conclusion
Random forest seem to be a better choice in this data set. Therefore the 20 cases in the test data set can be predicted with high accuracy. 
