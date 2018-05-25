``` r
library(plyr)
library(dataMaid)
```

Experiment description
======================

The experiments have been carried out with a group of 30 volunteers within an age bracket of 19-48 years. Each person performed six activities (WALKING, WALKING\_UPSTAIRS, WALKING\_DOWNSTAIRS, SITTING, STANDING, LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist. Using its embedded accelerometer and gyroscope, we captured 3-axial linear acceleration and 3-axial angular velocity at a constant rate of 50Hz. The experiments have been video-recorded to label the data manually. The obtained dataset has been randomly partitioned into two sets, where 70% of the volunteers was selected for generating the training data and 30% the test data. The sensor signals (accelerometer and gyroscope) were pre-processed by applying noise filters and then sampled in fixed-width sliding windows of 2.56 sec and 50% overlap (128 readings/window). The sensor acceleration signal, which has gravitational and body motion components, was separated using a Butterworth low-pass filter into body acceleration and gravity. The gravitational force is assumed to have only low frequency components, therefore a filter with 0.3 Hz cutoff frequency was used. From each window, a vector of features was obtained by calculating variables from the time and frequency domain. See "features\_info.txt" for more details. For each record it is provided:

-   Triaxial acceleration from the accelerometer (total acceleration) and the estimated body acceleration.
-   Triaxial Angular velocity from the gyroscope.
-   A 561-feature vector with time and frequency domain variables.
-   Its activity label.
-   An identifier of the subject who carried out the experiment.

Available descriptive files
---------------------------

-   "features\_info.txt": Shows information about the variables used on the feature vector.
-   "features.txt": List of all features.
-   "activity\_labels.txt": Links the class labels with their activity name.
-   "train/X\_train.txt": Training set.
-   "train/y\_train.txt": Training labels.
-   "test/X\_test.txt": Test set.
-   "test/y\_test.txt": Test labels.

Available data files
--------------------

The following files are available for the train and test data. Their descriptions are equivalent.

-   "train/subject\_train.txt": Each row identifies the subject who performed the activity for each window sample. Its range is from 1 to 30.
-   "train/Inertial Signals/total\_acc\_x\_train.txt": The acceleration signal from the smartphone accelerometer X axis in standard gravity units "g". Every row shows a 128 element vector. The same description applies for the "total\_acc\_x\_train.txt" and "total\_acc\_z\_train.txt" files for the Y and Z axis.
-   "train/Inertial Signals/body\_acc\_x\_train.txt": The body acceleration signal obtained by subtracting the gravity from the total acceleration.
-   "train/Inertial Signals/body\_gyro\_x\_train.txt": The angular velocity vector measured by the gyroscope for each window sample. The units are radians/second.

Text files in “Inertial Signals” folder will be discarded in this Project.

Source code
===========

The following is the source code for this project and a description of each step.

0. Read files
-------------

Set data sets directory.

``` r
dir.data <- file.path(getwd(),"UCI HAR Dataset")
```

Load the list of features names

``` r
cFeatures <- read.table(file.path(dir.data,"features.txt"), stringsAsFactors = FALSE)
cFeatures <- cFeatures[,2]
```

Parenthesis are not valid characters for column names.

``` r
cFeatures <- gsub("[()]", "", cFeatures)
```

Read train data and name columns using the above features list. Then, read activities and subjects and merge train data.

``` r
dtTrainX <- read.table(file.path(dir.data,"train","X_train.txt"), col.names=cFeatures)
dtTrainAct <- read.table(file.path(dir.data,"train","y_train.txt"), col.names=c("IdActivity"))
dtTrainSub <- read.table(file.path(dir.data,"train","subject_train.txt"), col.names=c("IdSubject"))
dtTrain <- cbind(dtTrainSub,dtTrainAct,dtTrainX)
```

Do the same for the test set.

``` r
dtTestX <- read.table(file.path(dir.data,"test","X_test.txt"), col.names=cFeatures)
dtTestAct <- read.table(file.path(dir.data,"test","y_test.txt"), col.names=c("IdActivity"))
dtTestSub <- read.table(file.path(dir.data,"test","subject_test.txt"), col.names=c("IdSubject"))
dtTest <- cbind(dtTestSub,dtTestAct,dtTestX)
```

1. Merge data sets
------------------

Merge the training and the test sets to create one data set.

``` r
dtFinal <- rbind(dtTrain,dtTest)
```

2. Extract measurements
-----------------------

Some column names were invalid and they have been transformed into valid names, so we must get the "new" colum names.

``` r
cNewFeatures <- names(dtFinal)
```

Extract only the measurements on the mean and standard deviation for each measurement.

``` r
cNewFeatures <- cNewFeatures[grepl("mean|std",cNewFeatures)]
dtFinalFilt <- dtFinal[,c("IdSubject","IdActivity",cNewFeatures)]
```

3. Name variables
-----------------

Use descriptive activity names to name the activities in the data set.

``` r
dtActDesc <- read.table(file.path(dir.data, "activity_labels.txt"),col.names = c("IdActivity","Activity"))
dtFinalFilt <- merge(dtFinalFilt, dtActDesc, by="IdActivity")
```

4. Label data
-------------

Label the data set with descriptive variable names.

``` r
cFeaturesFilt <- names(dtFinalFilt)
```

Prefix "t" means that data are in "Time" domain.

``` r
cFeaturesFilt <- gsub("^t", "Time", cFeaturesFilt)
```

Prefix "f" means that data are in "frequency" domain.

``` r
cFeaturesFilt <- gsub("^f", "Frequency", cFeaturesFilt)
```

"Acc" stands for "Accelerometer".

``` r
cFeaturesFilt <- gsub("Acc", "Accelerometer", cFeaturesFilt)
```

"Gyro" means "Gyroscope".

``` r
cFeaturesFilt <- gsub("Gyro", "Gyroscope", cFeaturesFilt)
```

"Mag" means "Magnitude".

``` r
cFeaturesFilt <- gsub("Mag", "Magnitude", cFeaturesFilt)
```

Some more clean up ...

``` r
cFeaturesFilt <- gsub("\\.mean", "Mean", cFeaturesFilt)
cFeaturesFilt <- gsub("\\.std", "StandardDeviation", cFeaturesFilt)
cFeaturesFilt <- gsub("BodyBody", "Body", cFeaturesFilt)
```

Assign new column names

``` r
names(dtFinalFilt) <- cFeaturesFilt
```

5. Final dataset
----------------

From the data set in step 4, create a second, independent tidy data set with the average of each variable for each activity and each subject.

``` r
dtTidy <- ddply(dtFinalFilt,.(IdSubject,IdActivity),numcolwise(mean))
```

Make code book of tidy data.

``` r
makeCodebook(dtTidy, file="codebook.Rmd", replace = TRUE)
```
