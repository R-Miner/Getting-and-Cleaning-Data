---
title: "README"
output: html_document
---
The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis.


### Downloading files
fileurl<- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
if(!file.exists("Dataset"))
  dir.create("Dataset")
download.file(fileurl,destfile ="UCI HDR Dataset.zip")
unzip("UCI HDR Dataset.zip")

workdir<- getwd()

### Reading Training Sets
subjecttrain<- read.table(file.path(workdir,"UCI HAR Dataset","train","subject_train.txt"),header = FALSE)
xtrain<- read.table(file.path(workdir,"UCI HAR Dataset","train","X_train.txt"), header = FALSE)
ytrain<- read.table(file.path(workdir,"UCI HAR Dataset","train","y_train.txt"), header = FALSE)

### Reading Testing Sets
subjecttest<- read.table(file.path(workdir,"UCI HAR Dataset","test","subject_test.txt"), header = FALSE)
xtest<- read.table(file.path(workdir,"UCI HAR Dataset","test","X_test.txt"), header = FALSE)
ytest<- read.table(file.path(workdir,"UCI HAR Dataset","test","y_test.txt"), header = FALSE)

### Reading Activity_labels set
activitylabels<- read.table(file.path(workdir,"UCI HAR Dataset", "activity_labels.txt"), header = FALSE)

### Reading Features Info set
featurestable<- read.table(file.path(workdir,"UCI HAR Dataset", "features.txt"), header = FALSE)


# STEP 1: Merge the training and test sets to create one data set.

### X is Features and Y are activities. Binding all 3 together.
subject<- rbind(subjecttrain,subjecttest)
features<- rbind(xtrain,xtest)
activity<- rbind(ytrain,ytest)

colnames(subject) <- "subject"
colnames(features) <-featurestable[,2]
colnames(activity) <-  "activity"

# STEP 2: Extracts only the measurements on the mean and standard deviation for each measurement. 
features <- features[ , grep("mean()|std()", colnames(features) )]

### Merging all into a single data set -STEP 1 completed
completeset<- cbind(subject, activity,features)
dim(completeset)

# STEP 3: Uses descriptive activity names to name the activities in the data set

### Building the lookup table
lut <- c("1" = "WALKING", "2" = "WALKING_UPSTAIRS", "3" = "WALKING_DOWNSTAIRS", "4" = "SITTING", "5" = "STANDING", "6" = "LAYING")

### Use lut to translate the Activity column of completeset
completeset$activity <- lut[completeset$activity]

# STEP 4: Appropriately labels the data set with descriptive variable names. Make feature names more human readble

names(completeset)<-gsub("Acc", "Accelerometer", names(completeset))
names(completeset)<-gsub("Gyro", "Gyroscope", names(completeset))
names(completeset)<-gsub("BodyBody", "Body", names(completeset))
names(completeset)<-gsub("Mag", "magnitude", names(completeset))
names(completeset)<-gsub("^t", "time", names(completeset))
names(completeset)<-gsub("^f", "freq", names(completeset))
names(completeset)<-gsub("tBody", "timeBody", names(completeset))

# STEP 5: From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
library(dplyr)

### Create the second tidy set
tidydata <- completeset %>%
  group_by(subject, activity) %>%
  summarise_each(funs(mean))

### Writing onto a file
 write.table(tidydata, file="tidydata.txt", row.names = FALSE)

