Given 2 files, we are asked to develop a machine learning paradigm to predict the type of exercise performed based an a variety of measurements. The first file is the training file, used to develop our algorithm, and the second is used to make our final predictions.

The training data is: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The testing data is: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

Once downloaded to our working directory, we read in the file to perform some basic exploratory data analysis. We notice there are many blanks and NA values (I do not show here in the interest of space), so I shall re-read the file so that all non-valid entries (blanks, DIV/0, NA), are read in as NA in R. I continue to examine and remove columns which contain NA’s, as well as remove columns which I do not believe have any outcome on the class.

a=read.csv('pml-training.csv',na.strings=c('','NA'))
b=a[,!apply(a,2,function(x) any(is.na(x)) )]
c=b[,-c(1:7)]
This leave us with 19622 observations and 53 predictors (one of which is the response variable)

To continue with the analysis we download the necessary packages

install.packages('randomForest')
library('randomForest')
install.packages('caret')
library('caret')
install.packages('e1071')
library('e1071')
For cross validation, We split our testing data into sub groups, 60:40

subGrps=createDataPartition(y=c$classe, p=0.6, list=FALSE)
subTraining=c[subGrps,]
subTesting=c[-subGrps, ]
dim(subTraining);dim(subTesting)
We see there are 11776 in the subTraining group, and 7846 in the subTesting group.

I then continue to make a predictive model based on the random forest paradigm, as it is one of the best performing, using the subTraining group. Once the model is made, we predict the outcome of the other group, subTesting, and examine the confusion matrix to see how well the predictive model performed

model=randomForest(classe~., data=subTraining, method='class')
pred=predict(model,subTesting, type='class')
z=confusionMatrix(pred,subTesting$classe)
save(z,file='test.RData')
setwd('C:/users/a/Desktop')
load('test.RData')
z$table
##           Reference
## Prediction    A    B    C    D    E
##          A 2229    7    0    0    0
##          B    1 1508   10    0    0
##          C    0    3 1358   17    1
##          D    1    0    0 1267   11
##          E    1    0    0    2 1430
z$overall[1]
##  Accuracy 
## 0.9931175
Based on this, The accuracy is 99.31%. The out of sample error, that is the error rate on a new (subTesting) data set, here is going to be 0.69%, with a 95% confidence interval of 0.52% to .9%.

Final Data Set Analysis and Predictions

This is very good, so I continue with the final testing data set. I read it in and preporcess it the same way as the training set previously.

d=read.csv('pml-testing.csv',na.strings=c('','NA'))
e=d[,!apply(d,2,function(x) any(is.na(x)) )]
f=e[,-c(1:7)]
Once the dataset it processed, I continue to analyse it using the model developed above

predicted=predict(model,f,type='class')
save(predicted,file='predicted.RData')
The final prediction for the 20 ends up as:


load('predicted.RData')
predicted

##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
