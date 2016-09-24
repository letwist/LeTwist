##I downloaded all data sets onto my desktop before loading into R.
##Merges the training and the test sets to create one data set.

##Load packages
library(dplyr)
library(data.table)
library(tidyr)

##Load data tables
yTest <- read.table("~/Desktop/y_test.txt", header = FALSE)
yTrain <- read.table("~/Desktop/y_train.txt", header = FALSE)
xTrain <- read.table("~/Desktop/X_train.txt", header = FALSE)
xTest <- read.table("~/Desktop/X_test.txt", header = FALSE)
sTest <- read.table("~/Desktop/subject_test.txt", header = FALSE)
sTrain <- read.table("~/Desktop/subject_train.txt", header = FALSE)

##combine subject data - This is not a merge, stacks data?
allSubjects <- rbind(sTrain,sTest)
##change old (V1) to new name (subjects)
setnames(allSubjects, "V1","subjects")

##combine data
allData <- rbind(xTrain,xTest)

##combine activity data
allActivity <- rbind(yTrain,yTest)
##change old (V1) to new name (activity)
setnames(allActivity, "V1", "activity")

##load data containing variable names
features <- tbl_df(read.table("~/Desktop/features.txt", header = FALSE))
##setnames
setnames(features, names(features),c("featureNum", "featureName"))

##Name the allData variables
colnames(allData) <- features$featureName

##load data containing activity levels
actLevels <- read.table("~/Desktop/activity_labels.txt", header = FALSE)
##set names
setnames(actLevels, names(actLevels),c("activity", "actName"))

##Merge data tables using cbind
subAct <- cbind(allSubjects, allActivity)
totalData <- cbind(subAct, allData)

##Extracts only the measurements on the mean and standard deviation for each 
##measurement.
##select only those names that correspond to the mean and std
featuresMeanStd <- grep("mean\\(\\)|std\\(\\)",features$featureName, value=TRUE)
##Make sure that subject and activity number are still present after grep
featuresMeanStd <- union(c("subject", "activityNum"), featuresMeanStd)
##Subset the totalData set according to the grep variables
totalData <- subset(totalData, select = featuresMeanStd)

##Uses descriptive activity names to name the activities in the data set
##use the activity names in conjunction with the numeric activity levels
totalData <- merge(actLevels, totalData, by = "activity", all.x = TRUE)
##make sure actName is being treated as a character
totalData$actName <- as.character(totalData$actName)

##Appropriately labels the data set with descriptive variable names.
##std is understood to represent the standard deviation in R, no need to change
##to view the names in a concise manner
head(str(totalData),2)
##a leading t in the original represents time
names(totalData) <- gsub("^t", "time", names(totalData))
##a leading f in the original represents frequency
names(totalData) <- gsub("^f", "frequency", names(totalData))
##Acc present anywhere represents accelerometer
names(totalData)<-gsub("Acc", "Accelerometer", names(totalData))
##gyro present anywhere represents gyroscope
names(totalData)<-gsub("Gyro", "Gyroscope", names(totalData))
##mag present anywhere represents magnitude
names(totalData)<-gsub("Mag", "Magnitude", names(totalData))
##bodybody present anywhere represents body
names(totalData)<-gsub("BodyBody", "Body", names(totalData))

##From the data set in step 4, creates a second, independent tidy data set with the ##average of each variable for each activity and each subject.
write.csv(totalData, "cleannready.csv", row.names=FALSE)