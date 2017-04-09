# GettingAndCleaningData
## Find if folder exists if not created data folder

if(!file.exists("./data")){dir.create("./data")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl,destfile="./data/Dataset.zip",method="curl")
unzip(zipfile="./data/Dataset.zip",exdir="./data")
path_rf <- file.path("./data" , "UCI HAR Dataset")
files<-list.files(path_rf, recursive=TRUE)
files

## Read Activity Files 

ActivityTestData  <- read.table(file.path(path_rf, "test" , "Y_test.txt" ),header = FALSE)
ActivityTrainData <- read.table(file.path(path_rf, "train", "Y_train.txt"),header = FALSE)

## Read Subject Files

SubjectTrainData <- read.table(file.path(path_rf, "train", "subject_train.txt"),header = FALSE)
SubjectTestData  <- read.table(file.path(path_rf, "test" , "subject_test.txt"),header = FALSE)

## Read Features Files 

FeaturesTestData  <- read.table(file.path(path_rf, "test" , "X_test.txt" ),header = FALSE)
FeaturesTrainData <- read.table(file.path(path_rf, "train", "X_train.txt"),header = FALSE)

str(ActivityTestData)
str(ActivityTrainData)
str(SubjectTestData)
str(SubjectTrainData)
str(FeaturesTestData)
str(FeaturesTrainData)

## Merge training and test data

SubjectData <- rbind(SubjectTrainData, SubjectTestData)
ActivityData<- rbind(ActivityTrainData, ActivityTestData)
FeaturesData<- rbind(FeaturesTrainData, FeaturesTestData)

names(SubjectData) <- c("Subject")
names(ActivityData) <-c("Activity")
FeatureNamesData <- read.table(file.path(path_rf, "features.txt"), head=FALSE)
names(FeaturesData) <-FeatureNamesData$V2

## Data frame for all data

CombinedData <- cbind(SubjectData, ActivityData)
Data1 <- cbind(FeaturesData, CombinedData)

SubFeaturesNamesData<-FeatureNamesData$V2[grep("mean\\(\\)|std\\(\\)", FeatureNamesData$V2)]

## Data frame by selected Features 

SelectedNames<-c(as.character(SubFeaturesNamesData), "subject", "activity" )
Data1<-subset(Data1,select=SelectedNames)

## Struture Check 

str(Data1)

ActivityLabels <- read.table(file.path(path_rf, "activity_labels.txt"),header = FALSE)

head(Data1$Activity,15)

names(Data1)<-gsub("^t", "time", names(Data1))
names(Data1)<-gsub("^f", "frequency", names(Data1))
names(Data1)<-gsub("Acc", "Accelerometer", names(Data1))
names(Data1)<-gsub("Gyro", "Gyroscope", names(Data1))
names(Data1)<-gsub("Mag", "Magnitude", names(Data1))
names(Data1)<-gsub("BodyBody", "Body", names(Data1))

names(Data1)


library(plyr);
Data2<-aggregate(. ~Subject + Activity, Data1, mean)
Data2<-Data2[order(Data2$Subject,Data2$Activity),]
write.table(Data2, file = "tidydata.txt",row.name=FALSE)
