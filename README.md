
'''R
library(dplyr)

#Load the data
xtrain = read.csv("UCI HAR Dataset/train/X_train.txt", sep="", header=FALSE)
ytrain = read.csv("UCI HAR Dataset/train/y_train.txt", sep="", header=FALSE)
subjectTrain = read.csv("UCI HAR Dataset/train/subject_train.txt", sep="", header=FALSE)
trainingData <- data.frame(xtrain, ytrain, subjectTrain)

xtest = read.csv("UCI HAR Dataset/test/X_test.txt", sep="", header=FALSE)
ytest = read.csv("UCI HAR Dataset/test/y_test.txt", sep="", header=FALSE)
subjectTest = read.csv("UCI HAR Dataset/test/subject_test.txt", sep="", header=FALSE)
testData <- data.frame(xtest, ytest, subjectTest)
'''

#Step 1
#Merges the training and the test sets to create one data set.
allData <- rbind(testData, trainingData)

#Add labels
features <- read.table("UCI HAR Dataset/features.txt")
x <- features[,2]
names(allData) <- c(as.character(x), "activity", "subject")
tableData <- tbl_df(allData)

#Remove duplicate cols
tableData <- tableData[ !duplicated(names(tableData)) ]

#Step 2
#Extracts only the measurements on the mean and standard deviation for each measurement.
tableData <- select(tableData,matches('std|mean|activity|subject'))

#Step 3
#Uses descriptive activity names to name the activities in the data set
tableData$activity[tableData$activity == "1"] <- "Walking"
tableData$activity[tableData$activity == "2"] <- "Walking_Upstairs"
tableData$activity[tableData$activity == "3"] <- "Walking_Downstairs"
tableData$activity[tableData$activity == "4"] <- "Sitting"
tableData$activity[tableData$activity == "5"] <- "Standing"
tableData$activity[tableData$activity == "6"] <- "Laying"

#Step 4
#Appropriately labels the data set with descriptive variable names.
names(tableData) <- gsub('Acc',"Acceleration",names(tableData))
names(tableData) <- gsub('tGravity|gravity',"Gravity",names(tableData))
names(tableData) <- gsub('mean',"Mean",names(tableData))
names(tableData) <- gsub('std',"StandardDev",names(tableData))
names(tableData) <- gsub('^t',"TimeDomain.",names(tableData))
names(tableData) <- gsub('^f',"FreqDomain.",names(tableData))
names(tableData) <- gsub('\\(|\\)',"",names(tableData), perl = TRUE)
names(tableData) <- gsub('-|,',"",names(tableData))

#Step 5
#From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
data_summary <- tableData %>%
  group_by(activity, subject) %>%
  summarise_each(funs(mean))

write.table(data_summary, "tidyDataset.txt", sep="\t", row.names=FALSE)
