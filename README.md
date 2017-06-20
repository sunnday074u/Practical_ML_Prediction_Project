# Practical_ML_Prediction_Project
Machine Language Analysis
##Background

This data set was created by measuring several individuals performing a weight lifting exercise, in one of several ways: correctly, and while making one of several common mistakes.

Various physical measurements of their movement were made recording position, momentum, orientation, of several body parts. The goal was to be able to detect whether the exercise is performed correctly, or detect a specific mistake, based on these physical measurements.

To simplify the analysis, the raw digital signals were summarized by some summary statistics over various time-slices.
Analysis

For this course assignment we are provided with some subset of this data, with labeled outcomes (the file pml-training.csv), as well as 20 cases (the file pml-testing.csv), whose outcomes are unlabeled and we have to guess.

We describe below each step of the analysis performed:
Loading and cleaning the data

We find that the data contains mostly numerical features. However, many of them contain nonstandard coded missing values. In addition to the standard NA, there are also empty strings "", and error expressions "#DIV/0!".


#Data description

The outcome variable is classe, a factor variable with 5 levels. For this data set, participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in 5 different fashions:

    exactly according to the specification (Class A)
    throwing the elbows to the front (Class B)
    lifting the dumbbell only halfway (Class C)
    lowering the dumbbell only halfway (Class D)
    throwing the hips to the front (Class E)


```{r}
library(ggplot2)
library(scales)
library(rpart)
library(rpart.plot)
library(dplyr)
```

# Attaching package: 'dplyr'
 

```{r}
library(caret)
```

# Loading required package: lattice

```{r}
library(e1071)
library(randomForest)
```

# randomForest 4.6-10
# Type rfNews() to see new features/changes/bug fixes.

```{r}
library(lubridate)
```

Data processing

In this section the data is processed. Some basic transformations and cleanup will be performed, so that NA values are omitted. Irrelevant columns such as user_name, raw_timestamp_part_1, raw_timestamp_part_2, cvtd_timestamp, new_window, and num_window (columns 1 to 7) will be removed in the subset.

The pml-training.csv data is used to devise training and testing sets. The pml-test.csv data is used to predict and answer the 20 questions based on the trained model.

```{r}
training <- read.csv("C:/Users/hp-pc/Desktop/data/pml-training.csv", row.names = 1, stringsAsFactors = FALSE, na.strings = c("NA", "", "#DIV/0!")) %>%
  mutate(cvtd_timestamp = mdy_hm(cvtd_timestamp),
         user_name      = as.factor(user_name),
         new_window     = as.factor(new_window),
         classe         = as.factor(classe))

testing <- read.csv("C:/Users/hp-pc/Desktop/data/pml-testing.csv", row.names = 1, stringsAsFactors = FALSE, na.strings = c("NA", "", "#DIV/0!"))  %>%
  mutate(cvtd_timestamp = mdy_hm(cvtd_timestamp),
         user_name      = as.factor(user_name),
         new_window     = as.factor(new_window))
```

# Subset data
```{r}
training<-training[,colSums(is.na(training)) == 0]
testing <-testing[,colSums(is.na(testing)) == 0]
training   <-training[,-c(1:7)]
testing <-testing[,-c(1:7)]
```

#Cross-validation

In this section cross-validation will be performed by splitting the training data in training (75%) and testing (25%) data.
```{r}
subSamples <- createDataPartition(y=training$classe, p=0.75, list=FALSE)
subTraining <- training[subSamples, ] 
subTesting <- training[-subSamples, ]
```


#Expected out-of-sample error
The expected out-of-sample error will correspond to the quantity: 1-accuracy in the cross-validation data. Accuracy is the proportion of correct classified observation over the total sample in the subTesting data set. Expected accuracy is the expected accuracy in the out-of-sample data set (i.e. original testing data set). Thus, the expected value of the out-of-sample error will correspond to the expected number of missclassified observations/total observations in the Test data set, which is the quantity: 1-accuracy found from the cross-validation data set.

#Exploratory analysis

The variable classe contains 5 levels. The plot of the outcome variable shows the frequency of each levels in the subTraining data.

```{r}
plot(subTraining$classe, col="blue", main="Levels of the variable classe", xlab="classe levels", ylab="Frequency")
```
The plot above shows that Level A is the most frequent classe. D appears to be the least frequent one.

#Prediction models
In this section a decision tree and random forest will be applied to the data.

#Decision Tree
```{r}
# Fit model
modFitDT <- rpart(classe ~ ., data=subTraining, method="class")

# Perform prediction
predictDT <- predict(modFitDT, subTesting, type = "class")

# Plot result
rpart.plot(modFitDT, main="Classification Tree", extra=102, under=TRUE, faclen=0)
```

Following confusion matrix shows the errors of the prediction algorithm.
```{r}
confusionMatrix(predictDT, subTesting$classe)
```

#Random forest
```{r}
# Fit model
modFitRF <- randomForest(classe ~ ., data=subTraining, method="class")

# Perform prediction
predictRF <- predict(modFitRF, subTesting, type = "class")
```

Following confusion matrix shows the errors of the prediction algorithm.

```{r}
confusionMatrix(predictRF, subTesting$classe)
```

##Conclusion
#Result

The confusion matrices show, that the Random Forest algorithm performens better than decision trees. The accuracy for the Random Forest model was 0.995 (95% CI: (0.993, 0.997)) compared to 0.739 (95% CI: (0.727, 0.752)) for Decision Tree model. The random Forest model is choosen.

#Expected out-of-sample error

The expected out-of-sample error is estimated at 0.005, or 0.5%. The expected out-of-sample error is calculated as 1 - accuracy for predictions made against the cross-validation set. Our Test data set comprises 20 cases. With an accuracy above 99% on our cross-validation data, we can expect that very few, or none, of the test samples will be missclassified.

#Submission

In this section the files for the project submission are generated using the random forest algorithm on the testing data.

```{}
# Perform prediction
predictSubmission <- predict(modFitRF, testing, type="class")
predictSubmission

# Write files for submission
pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("./data/submission/problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}

pml_write_files(predictSubmission)
```

