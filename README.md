# Description



## Task 1 is done  in 5 steps

## $$$$$$$$$$ ------------START OF TASK 1 -------------- $$$$$



library(dplyr)



### 1 Load the 561 fields, separated by space

fields<- read.table("features.txt", header=F, sep=" "

                    , col.names=c("sn","feature"), stringsAsFactors = FALSE)

### 1.a  rename the fields names to all lower case and replace '-' and brackets

fields$feature<- tolower(  fields$feature<- gsub( "\\)" ,"", gsub("\\(" ,"", gsub("-","_", fields$feature)) ) )



### 2 Load the activity labels

act_labels<-read.table("activity_labels.txt", header=F, sep=" "

                       , col.names=c("number","label"), stringsAsFactors = TRUE)



### 3Load the X_train.txt and X_test.txt with the 561 field names

X_train<- read.fwf("train/X_train.txt",widths=rep(16,561), header=F, col.names=c(fields$feature) 

                   , stringsAsFactors = FALSE)



X_test<- read.fwf("test/X_test.txt",widths=rep(16,561), header=F, col.names=c(fields$feature) 

                   , stringsAsFactors = FALSE)

### 4 Column bind the above to main datasets with the activity labels(1-6) and User ID( 1-30)



### 4a subject_train and subject_test as factor

subject_train<-read.fwf("train/subject_train.txt",widths=2, header=F, col.names="subject")

subject_train$subject<- as.factor(subject_train$subject)

subject_test<-read.fwf("test/subject_test.txt",widths=2, header=F, col.names="subject")

subject_test$subject<- as.factor(subject_test$subject)

### 4b y_train and y_test

y_train<-read.fwf("train/y_train.txt",widths=1, header=F, col.names="act")

y_train$act<- as.factor(y_train$act)

y_test<-read.fwf("test/y_test.txt",widths=1, header=F, col.names="act")

y_test$act<- as.factor(y_test$act)

### 4c  Cbind

test_bind<- cbind(subject_test, y_test, X_test)

train_bind<- cbind(subject_train, y_train, X_train)

### 5Union of train_bind and test_bind into a single dataset 

final_bind<- rbind(train_bind, test_bind)



## $$$$$$$$$$ ------------END OF TASK 1 -------------- $$$$$





## $$$$$$$$$$ ------------START OF TASK 2 -------------- $$$$$





## Task 2 -- extract columns with just mean and std

mean_std<-select(final_bind, grep("mean|std", names(final_bind)), act, subject)



## $$$$$$$$$$ ------------START OF TASK 4 -------------- $$$$$



### Task 4- Give the column names descriptive names

###  Basically give longer names to columns

library(gsubfn)

names(mean_std)<- gsubfn("acc|mag",list("acc"="_acceleration","mag"="_magnitude") , names(mean_std) )

names(mean_std)<-gsub("^t","time_", colnames(mean_std) )

names(mean_std)<-gsub("^f","frequency_", colnames(mean_std) )

names(mean_std)<-gsub("bodybody","body", colnames(mean_std) )

names(mean_std)<-gsub("\\.","_", colnames(mean_std) )



## $$$$$$$$$$ ------------START OF TASK 3 -------------- $$$$$



### Task 3- rename the values in the act column and give them their actual label names

### join with the label table first, then delte the act column, rename label column to activity

mean_std_lab<- merge(mean_std, act_labels, by.x="act", by.y="number", all.x=TRUE)

mean_std_lab$act=NULL

names(mean_std_lab)[names(mean_std_lab)=="label"]<- "activity"

###  rarrange columns to bring subject and activity to the beginning of the dataframe

mean_std_lab<-mean_std_lab[, c(grep("subject|activity",names(mean_std_lab)), 1:(ncol(mean_std_lab)-2))]



## $$$$$$$$$$ ------------START OF TASK 5 -------------- $$$$$



### Task 5

### First convert columns 3:88 to numeric

mean_std_lab[, 3:88] <- sapply ( mean_std_lab[, 3:88]  , as.numeric )

###  Tried using Summarize, apply functions and others to get the mean of all

### eventually got tired and resorted to sqldf

library(readr)

library(sqldf)

sql_string<-read_file("sql_file.sql")

result<-sqldf(sql_string)

resthin<- gather(result, variable_measured, mean_for_subject_and_activity, -c("subject","activity"))



## $$$$$$$$$$ ------------THE  END -------------- $$$$$