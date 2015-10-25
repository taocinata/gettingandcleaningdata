##Assignment
The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected. </br>

One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained: </br>

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones 

Here are the data for the project: </br>

https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

 You should create one R script called run_analysis.R that does the following. </br>

   1. Merges the training and the test sets to create one data set.</br>
   2. Extracts only the measurements on the mean and standard deviation for each measurement. </br>
   3. Uses descriptive activity names to name the activities in the data set</br>
   4. Appropriately labels the data set with descriptive variable names. </br>
   5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject. </br>

Good luck! </br>
## Code with comments what I did
```{r}
# Install and load all library that we need for this task
#############################################################

install.packages("data.table")
library("data.table")
library(dplyr)
library(reshape2)

# Download zip file from source and uzip it
###############################################################

url<- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
temp <- tempfile()
download.file(url,temp, mode="wb", method = "curl")
a<- unzip(temp)

# See where is the path of our working project
getwd()

# Read tables from txt files to R
# We need to open and merege those files, step 1
###############################################################

trainS <- fread("./UCI HAR Dataset//train/subject_train.txt")
# 2947 rows and 1 column
trainA <- fread("./UCI HAR Dataset//train/y_train.txt") 
# 2947 rows and 1 column
trainread <- read.table("./UCI HAR Dataset//train/X_train.txt")
trainwrite <- data.table(trainread)
# 2947 rows and 561 columns
testS <- fread("./UCI HAR Dataset//test/subject_test.txt")
# 7352 rows and 1 column
testA <- fread("./UCI HAR Dataset//test/y_test.txt")
# 7352 rows and 1 column
testread <- read.table("./UCI HAR Dataset//test/X_test.txt")
testwrite <- data.table(testread)
# 7352 rows and 561 columns
features <- fread("./UCI HAR Dataset/features.txt")
setnames(features,c("V1","V2"),c("id","name"))
# 561 rows and 2 columns

# MERGE (Step 1)
# First we merge train Subject and test Subject, then we merge train Activity and test Activity, 10299rows and 1 column for both tables
####################################################################

submerge <- rbind(trainS, testS)
activemegere <- rbind(trainA,testA)

# We should merge now train and test, subject and activities, 10299 row and 2 columns
SAmerged <- cbind(submerge,activemegere)

# (Step 4) : Now, we merged train resulsts and test results, we got 10299 rows and 561 column, and change column names to features names
merged <- rbind(trainwrite,testwrite)
names(merged) <- features$name
names(merged) <- gsub("\\.|-","",names(merged))
names(merged) <- gsub("^t{1}","time",names(merged))
names(merged) <- gsub("^f{1}","frequency",names(merged))
names(merged) <- gsub("\\()","",names(merged))

# We should merge now all train and test results with subject and activities, and also change column names to Subject for subjects and Activity for activities
all <- cbind(merged, SAmerged)
colnames(all)[562] <- "Subject"
colnames(all)[563] <- "Activity"

# (Step 2) : Extract only measurments with std and mean, I put Subject and Activite because I already merger those tabels here
all_new <- select(all, matches("mean|std|Subject|Activity"))

# Load activites label with names of Activity
namedofactivity <- fread("./UCI HAR Dataset/activity_labels.txt")
setnames(namedofactivity,c("V1","V2"),c("Activity","ActivityName"))

# (Step 3) : Merge and make "all_fat" data with activity names instead of id/number who is linked with activity
all_fat<- merge(all_new,namedofactivity, by=("Activity"))
# We got one column more with activity id/number so we should deleted it
all_fat$Activity <- NULL
# change column name from ActivityName to Activity
colnames(all_fat)[88] <- "Activity"

# (Step 5) : make tidy data, group all data by subject and activity, summarise it and calculate average.
groupbyActSub <- group_by(all_fat, Subject, Activity)
tidy <- summarise_each(groupbyActSub, funs(mean))
# Make it tidy, narrow form, where we have for every subject every activity, and every mesurament and mean of value.
melted <- melt(tidy, id=c("Subject", "Activity"))

# Write txt file, and it can be attached to course project 
write.table(melted, "tidy.txt", sep="\t", row.names = FALSE)


```
