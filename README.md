#####Description of run_analysis.R

run_analysis.R is a script that processes UCI's Human Activity Recognition Using Smartphones Dataset.  

Data source:  
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip  

Data description:  
http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones 

run_analysis.R does the following:  
1. Merges the training and the test sets to create one data set.  
2. Extracts only the measurements on the mean and standard deviation for each measurement.  
3. Uses descriptive activity names to name the activities in the data set  
4. Appropriately labels the data set with descriptive variable names.  
5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.  

run_analysis.R
#####1. Merges the training and the test sets to create one data set.
######List all files
path<-"C:/Users/Lorelie/Documents/Coursera/Data Science/03_Cleaning Data/Course_Project/UCI HAR Dataset"  
files<-list.files(path,recursive=TRUE)  

library(data.table)  

######Assign pertinent files to variables  
######Data set  
testdata<-read.table(file.path(path, "test" , "X_test.txt" ),header = FALSE)  
trainingdata<-read.table(file.path(path, "train" , "X_train.txt" ),header = FALSE)  

######Data labels  
testlabels<-read.table(file.path(path, "test" , "y_test.txt" ),header=FALSE)  
traininglabels<-read.table(file.path(path, "train" , "y_train.txt" ),header=FALSE)  

######Data subjects  
testsubject<-read.table(file.path(path,"test","subject_test.txt"),header=FALSE)  
trainingsubject<-read.table(file.path(path,"train","subject_train.txt"),header=FALSE)  

######Merge rows
dataset<-rbind(testdata,trainingdata)  
datalabels<-rbind(testlabels,traininglabels)  
datasubjects<-rbind(testsubject,trainingsubject)  

######Assign variable names
names(datalabels)<-c("Activity_Code")  
names(datasubjects)<-c("Subjects")  
datafeatures<-read.table(file.path(path,"features.txt"),head=FALSE)  
names(dataset)<- datafeatures$V2  

######Merge columns  
data<-cbind(dataset,datasubjects,datalabels)  

#####2. Extracts only the measurements on the mean and standard deviation for each measurement.
datamean<-sapply(data,mean,na.rm=TRUE)  
datastddev<-sapply(data,sd,na.rm=TRUE)  

#####3. Uses descriptive activity names to name the activities in the data set.
activityname<-read.table(file.path(path,"activity_labels.txt"),head=FALSE,colClasses="character")  
data$Activity<-factor(data$Activity,levels=activityname$V1,labels=activityname$V2)  

#####4. Appropriately labels the data set with descriptive variable names.
names(data)<-gsub("^t", "time", names(data))  
names(data)<-gsub("^f", "frequency", names(data))  
names(data)<-gsub("Acc", "Accelerometer", names(data))  
names(data)<-gsub("Gyro", "Gyroscope", names(data))  
names(data)<-gsub("Mag", "Magnitude", names(data))  
names(data)<-gsub("BodyBody", "Body", names(data))  

#####5. Creates a second, independent tidy data set with the average of each variable for each activity and each subject.
library(plyr)  
tidydata<-aggregate(.~Subjects+Activity,data,mean)  
tidydata<-tidydata[order(tidydata$Subjects,tidydata$Activity),]  
write.table(tidydata,file="tidydata.txt",row.name=FALSE)  
