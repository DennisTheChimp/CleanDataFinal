# Getting & Cleaning Data, Final Project

## Scriptfile: run_analysis.R

First the script loads the dplyr library
***
library(dplyr)
***

In the next section, all the input files are read and assigned to appropriate variables.  
* activity_labels holds the activity names.  
* measurement_labels holds the names of the measured variables (column names).  
* train_data and test_data hold the measured data for the train group and the test group (main data).  
* train_labels and test_labels hold the activity labels (numbers) for every observation of the respective groups.  
* train_subject and test_subject hold the subject labels (numbers) for every observation of the respective groups.  

***
activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")
measurement_labels <- read.table("./UCI HAR Dataset/features.txt")
train_data <- read.table("./UCI HAR Dataset/train/X_train.txt")
train_labels <- read.table("./UCI HAR Dataset/train/y_train.txt")
train_subjects <- read.table("./UCI HAR Dataset/train/subject_train.txt")
test_data <- read.table("./UCI HAR Dataset/test/X_test.txt")
test_labels <- read.table("./UCI HAR Dataset/test/y_test.txt")
test_subjects <- read.table("./UCI HAR Dataset/test/subject_test.txt")
***

The subject labels and activity labels are added to the train and test datasets by adding two new columns.  
The names of the columns are assigned "subject" and "activity"  
***
train_data <- mutate(train_data, subject = train_subjects[,1], activity = train_labels[,1])
test_data <- mutate(test_data, subject = test_subjects[,1], activity = test_labels[,1])
***

The train and test data are combined into a new dataframe all_data  
***
all_data <- rbind(train_data, test_data)
***

The activity labels in the activity column are replaced with the activity names (from activity_labels)  
***
all_data <- mutate(all_data, activity = activity_labels[activity,2])
***

The names of all columns with measurements are given the right name (from measurement_label)  
The columns with the subject and activity labels are assigned as factor variables so we can summarize  
***
names(all_data)[1:561] <- as.character(measurement_labels[,2])
for(i in 562:563) {all_data[,i] <- as.factor(all_data[,i])}
***

In order to select only measurements that hold a mean or standard deviation, we must make a subset  
First, only those columns are selected that have a variable name with the string "mean()" or "std"()" in it.  
These are combined with the subject and activity columns and assigned to a new dataframe tidy_data  
***
select_columns <- c(grep("mean\\(\\)|std\\(\\)",names(all_data)), 562:563)
tidy_data <- all_data[,select_columns]
***

This dataframe is grouped by activity and subject and then we calculate the colomn means of these groups  
***
tidy_data <- tidy_data %>% group_by(activity, subject) %>% summarize_each(funs(mean))
***

Lastly, the output is written to a flat textfile and the summary to the screen 
***
write.table(tidy_data, file = "tidydata.txt", row.names = FALSE)
summary(tidy_data)
***