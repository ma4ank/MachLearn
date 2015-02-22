

The following Libraries were used for this project, which you should install - if not done yet - and load on your working environment.

library(caret)
library(rpart)
library(e1071)
library(randomForest)

##Code 

setwd("D:/My project/Machine learning")
if(!file.exists("./data/")){dir.create("./data/")}

fileUrl<-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
download.file(fileUrl,destfile="./data/inTrain.csv")
inTrain<-read.csv("./data/inTrain.csv")

fileUrl2<-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
download.file(fileUrl,destfile="./data/inTest.csv")
inTest<-read.csv("./data/inTest.csv")

Partioning Training data set into two data sets, 60% for myTraining, 40% for myTesting:


trainPart <- createDataPartition(y=inTrain$classe, p=0.6, list=FALSE)
subTrain <- inTrain[trainPart, ]
subTest <- inTrain[-trainPart, ]
dim(subTrain)
dim(subTest)

##Cleaning the data

The following transformations were used to clean the data:

Transformation 1: Cleaning NearZeroVariance Variables Run this code to view possible NZV Variables:

myDataNZV <- nearZeroVar(subTrain, saveMetrics=TRUE)

Run this code to create another subset without NZV variables:

myNZVvars <- names(subTrain) %in% c("new_window", "kurtosis_roll_belt", "kurtosis_picth_belt",
"kurtosis_yaw_belt", "skewness_roll_belt", "skewness_roll_belt.1", "skewness_yaw_belt",
"max_yaw_belt", "min_yaw_belt", "amplitude_yaw_belt", "avg_roll_arm", "stddev_roll_arm",
"var_roll_arm", "avg_pitch_arm", "stddev_pitch_arm", "var_pitch_arm", "avg_yaw_arm",
"stddev_yaw_arm", "var_yaw_arm", "kurtosis_roll_arm", "kurtosis_picth_arm",
"kurtosis_yaw_arm", "skewness_roll_arm", "skewness_pitch_arm", "skewness_yaw_arm",
"max_roll_arm", "min_roll_arm", "min_pitch_arm", "amplitude_roll_arm", "amplitude_pitch_arm",
"kurtosis_roll_dumbbell", "kurtosis_picth_dumbbell", "kurtosis_yaw_dumbbell", "skewness_roll_dumbbell",
"skewness_pitch_dumbbell", "skewness_yaw_dumbbell", "max_yaw_dumbbell", "min_yaw_dumbbell",
"amplitude_yaw_dumbbell", "kurtosis_roll_forearm", "kurtosis_picth_forearm", "kurtosis_yaw_forearm",
"skewness_roll_forearm", "skewness_pitch_forearm", "skewness_yaw_forearm", "max_roll_forearm",
"max_yaw_forearm", "min_roll_forearm", "min_yaw_forearm", "amplitude_roll_forearm",
"amplitude_yaw_forearm", "avg_roll_forearm", "stddev_roll_forearm", "var_roll_forearm",
"avg_pitch_forearm", "stddev_pitch_forearm", "var_pitch_forearm", "avg_yaw_forearm",
"stddev_yaw_forearm", "var_yaw_forearm")
subTrain <- subTrain[!myNZVvars]
dim(subTrain)

Transformation 2: Killing first column of Dataset - ID Removing first ID variable so that it does not interfer with ML Algorithms:

subTrain <- subTrain[c(-1)]

Transformation 3: Cleaning Variables with too many NAs. For Variables that have more than a 60% threshold of NA's I'm going to leave them out:

trainingV3 <- subTrain
for(i in 1:length(subTrain)) { 
        if( sum( is.na( subTrain[, i] ) ) /nrow(subTrain) >= .6 ) { 
        for(j in 1:length(trainingV3)) {
            if( length( grep(names(subTrain[i]), names(trainingV3)[j]) ) ==1)  { 
                trainingV3 <- trainingV3[ , -j]
            }   
        } 
    }
}

dim(trainingV3)


subTrain <- trainingV3
rm(trainingV3)
Same transformations on subTest and inTrain data sets.

clean1 <- colnames(subTrain)
clean2 <- colnames(subTrain[, -58])
subTest <- subTest[clean1]
inTest <- inTest[clean2]
dim(inTest)

##Coerce the data into the same type.

for (i in 1:length(inTest) ) {
        for(j in 1:length(subTrain)) {
        if( length( grep(names(subTrain[i]), names(inTest)[j]) ) ==1)  {
            class(inTest[j]) <- class(subTrain[i])
        }      
    }      
}

inTest <- rbind(subTrain[2, -58] , inTest)
inTest <- inTest[-1,]

##Random Forests

modFitRF <- randomForest(classe ~. , data=subTrain)

##Predicting:

predictionsB1 <- predict(modFitRF, subTest, type = "class")

confusionMatrix(predictionsRF, subTest$classe)
##Overall Statistics

 ##              Accuracy : 0.999          
 ##                95% CI : (0.998, 0.9996)
 ##   No Information Rate : 0.2845         
 ##   P-Value [Acc > NIR] : < 2.2e-16      

 ##                 Kappa : 0.9987         
 ##Mcnemar's Test P-Value : NA 

Testing on Real Testset
predictionsB2 <- predict(modFitRF, inTest, type = "class")


pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}

pml_write_files(predictionsB2)
