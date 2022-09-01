#  Titanic Data Analysis 


## About
The Titanic dataset comes from the Kaggle website https://www.kaggle.com/competitions/titanic/data and explained with the following description:

"The sinking of the Titanic is one of the most infamous shipwrecks in history.On April 15, 1912, during her maiden voyage, the widely considered “unsinkable” RMS Titanic sank after colliding with an iceberg. Unfortunately, there weren’t enough lifeboats for everyone onboard, resulting in the death of 1502 out of 2224 passengers and crew.While there was some element of luck involved in surviving, it seems some groups of people were more likely to survive than others.In this challenge, we ask you to build a predictive model that answers the question: “what sorts of people were more likely to survive?” using passenger data (i.e. name, age, gender, socio-economic class, etc.)."


Kaggle competition usually provides competition data. There is a “Data” tab on any competition site. Click on the Data tab at the top of the competition page, you will find the raw data provided and most of time there are brief explanation of the data attributes too.

There are three files in the Titanic Challenge:

- train.csv,
- test.csv, and
- gender_submission.csv

The training set is supposedly used to build your models. For the training set, it provides the outcome (also known as the “ground truth”) for each passenger. 

The test set should be used to see how well our model performs on unseen data. For the test set, there is no ground truth for each passenger is provided. It is our job to predict these outcomes. For each passenger in the test set, we will use the model trained to predict whether or not they survived the sinking of the Titanic.

The data sets has also include gender_submission.csv, a set of predictions that assume all and only female passengers survive, as an example of what a submission file should look like.Titanic competition requires the results need be submitted in the file. The file structure is demonstrated in the “gender_submission.csv”. It is also provided as an example that shows how we should structure your results, which means predictions.

The example submission in “Gender_submission” predicts that all female passengers survived, and all male passengers died. It is clearly biased. Our hypotheses regarding survival will probably be different, which will lead to a different submission file.

### Basic understanding of the data



We start reading our training and test set:
```
titanic_train <-read.csv("train.csv")
titanic_test <- read.csv("test.csv")
```

We can check the structure of the data using str():

```{r, results='hide'}
str(titanic_train)
```
```{r, results='hide'}
str(titanic_test)
```

The training set has 891 observations and 12 variables and the testing set has 418 observations and 11 variables. The traning set has 1 extra varible. Check which which one we are missing. I know we could see that in a very small dataset like this, but if its larger we want two compare them.

```{r}
colnames_check <- colnames(titanic_train) %in% colnames(titanic_test)
colnames(titanic_train[colnames_check==FALSE])
```

As we can see we are missing the Survived in the test set. Which is correct because thats our challenge, we must predict this by creating a model.


```{r}
#Use sapply(#object, class) to check the class of every column.
sapply(titanic_train, class)
```

We can see that the Survived and Pclass column are integers and Sex character. But they are actually categorical variables. To convert them into categorical variables (or factors), use the factor() function.Survived is a nominal categorical variable, whereas Pclass is an ordinal categorical variable. For an ordinal variable, we provide the order=TRUE and levels argument in the ascending order of the values( Pclass 3 < Pclass 2 < Pclass 1).

```{r}
#change columns class
#Survived: from integer into factor
titanic_train$Survived = as.factor(titanic_train$Survived)
titanic_train$Sex = as.factor(titanic_train$Sex)
titanic_train$Pclass=factor(titanic_train$Pclass,order=TRUE, levels = c(3, 2, 1))
```

Let’s look deeper into the training set, and check how many passengers that survived vs did not make it.

```{r}
table(titanic_train$Survived)
```

Out of the 891 there are only 342 who survived it. Check also as proportions.

```{r}
prop.table(table(titanic_train$Survived))
```

A little more than one-third of the passengers survived the disaster. Now see if there is a difference between males and females that survived vs males that passed away.

```{r}
table(titanic_train$Sex, titanic_train$Survived)
```
```{r}
prop.table(table(titanic_train$Sex, titanic_train$Survived),margin = 1)
```


As we can see most of the female survived and most of the male did not make it.

## Data Preparation

Now we need to clean the dataset to create our models. Note that it is important to explore the data so that we understand what elements need to be cleaned.

```{r, results='hide'}
#missing data
is.na(titanic_train)
sum(is.na(titanic_train))
```

```{r}
#This function shows us exactly how much values are missing in each column.
apply(titanic_train, MARGIN = 2, FUN = function(x) {sum(is.na(x))})
```

```{r}
# Graphically check the missing data
library("mice")
missing_pattern <- md.pattern(titanic_train, rotate.names = TRUE)
```

```{r}
colSums(is.na(titanic_test))
```


As we can see we have missing values in Age in the training set and Age, Fare in the test set.

First we tackle the missing Fare, because this is only one value. Let see in wich row it’s missing.

```{r}
titanic_test[!complete.cases(titanic_test$Fare),]
```

As we can see the passenger on row 1044 has an NA Fare value. Now, we need to deal with the NA values in Age column. We will drop these rows using na.omit. Since the PassengerID is a unique identifier for the records, we will drop it. Intuitively the Name, Fare, Embarked and Ticket columns will not decide the survival, so we will drop them as well. So we will select the remaining columns using the select() function from dplyr library:

```{r}
#drop missing data
titanic_test_dropedna <- na.omit(titanic_test)
titanic_train_dropedna <- na.omit(titanic_train)
library(dplyr)
titanic_train_1 <-  select(titanic_train_dropedna, Survived, Pclass, Age, Sex, SibSp, Parch)
titanic_test_1 <- select(titanic_train_dropedna, Survived, Pclass, Age, Sex, SibSp, Parch)
```


```{r, results='hide'}
#Alternatively we can use dropping NAs by:
titanic_train[rowSums(is.na(titanic_train)) <= 0,]
# or
library(tidyr)
titanic_train %>% drop_na()
```

```{r}
#separate data
titanic_survivor = titanic_train_dropedna[titanic_train_dropedna$Survived == 1, ]
titanic_nonsurvivor = titanic_train_dropedna[titanic_train_dropedna$Survived == 0, ]
```

## Data Visualization

```{r}
#barchart
barplot(table(titanic_survivor$Sex))
barplot(table(titanic_nonsurvivor$Sex))
```

```{r}
## number of  survivals by Sex
library(ggplot2)
ggplot(titanic_survivor, aes(x = Sex)) +
  geom_bar(width=0.5, fill = "coral") +
  geom_text(stat='count', aes(label=stat(count)), vjust=-0.5) +
  theme_classic()
```
```{r}
## number of  Non survivals by Sex
library(ggplot2)
ggplot(titanic_nonsurvivor, aes(x = Sex)) +
  geom_bar(width=0.5, fill = "coral") +
  geom_text(stat='count', aes(label=stat(count)), vjust=-0.5) +
  theme_classic()
```
```{r}
ggplot(titanic_train_dropedna, aes(x = Survived, fill=Sex)) +
 geom_bar(position = position_dodge()) +
 geom_text(stat='count', 
           aes(label=stat(count)), 
           position = position_dodge(width=1), vjust=-0.5)+
 theme_classic()
```

```{r}
ggplot(titanic_train_dropedna, aes(x = Pclass, fill=Sex)) +
 geom_bar(position = position_dodge()) +
 geom_text(stat='count', 
           aes(label=stat(count)), 
           position = position_dodge(width=1), vjust=-0.5)+
 theme_classic()
```

Here we have created a temporary attribute called Discretized.age which groups the ages with a span of 10 years.
We discretize the age using the cut() function and specify the cuts in a vector.
The temporary attribute it discarded after plotting.
Most of the patients that died during hospitalization are in the age range from 70-80 years old.

```{r}
#Discretize age to plot survival
titanic_survivor$Discretized.age = cut(titanic_survivor$Age, c(0,10,20,30,40,50,60,70,80,100))
# Plot discretized age
ggplot(titanic_survivor, aes(x = Discretized.age, fill=Sex)) +
  geom_bar(position = position_dodge()) +
  geom_text(stat='count', aes(label=stat(count)), position = position_dodge(width=1), vjust=-0.5)+
  theme_classic()
#data.frame$Discretized.age = NULL
```

```{r}
ggplot(titanic_survivor, aes(x = Pclass, fill=Sex)) +
 geom_bar(position = position_dodge()) +
 geom_text(stat='count', 
           aes(label=stat(count)), 
           position = position_dodge(width=1), vjust=-0.5)+
 theme_classic()
```

```{r}
# Boxplot
titanic_train_dropedna %>% 
  ggplot(aes(x = Survived, y = Age)) +
  geom_boxplot() +
  theme_classic() +
  labs(title = "Survival rates by Age", x = NULL)
```
Passengers who survived seems to have a lower median age.

## Decision Tree Model

Now I am going to train a model to predict survivability and then test the model. The model will be saved and submitted to Kaggle. The file I send to Kaggle needs to have the Passenger ID and the prediction of whether or not that passenger survived. We obtained Kaggle submission score of 0.7751 which represents our submission's accuracy. A score of 0.7751 in this competition indicates we predicted Titanic survival correctly for 77.51% of people.


```{r}
library(tidyverse)
library(caret)
library(rpart)
set.seed(123)  # for reproducibility
model1 <- rpart(Survived ~ Pclass + Sex + Age + SibSp + Parch + Fare + Embarked, data=titanic_train,method="class")
```

```{r}
library(caret)
options(digits=4)
# assess the model's accuracy with train dataset by make a prediction on the train data. 
Predict_model1_train <- predict(model1, titanic_train, type = "class")
#build a confusion matrix to make comparison
conMat <- confusionMatrix(as.factor(Predict_model1_train), as.factor(titanic_train$Survived))
#show confusion matrix 
conMat$table
```

A brief assessment shows our model1’s accuracy is 83.28%. It is not bad! Let us use this model to make a prediction on test dataset.

```{r}
#show percentage of same values - accuracy
predict_train_accuracy <- conMat$overall["Accuracy"]
predict_train_accuracy
```

```{r}
# The firs prediction produced by the first decision tree which only used one predictor Sex
Prediction1 <- predict(model1, titanic_test, type = "class")
```


```{r}
# plot our full house classifier 
library(rpart.plot)
prp(model1, type = 0, extra = 1, under = TRUE)
# plot our full house classifier 
rpart.plot(model1)
```
Our prediction is produced. Let us submit to Kaggle for an evaluation. We need to convert our prediction into Kaggle’s required format and save it into a file and name it as “Tree_Model1.CSV”. Here, the importance is knowing the procedure.


```{r}
# produce a submit with Kaggle required format that is only two attributes: PassengerId and Survived
submit1 <- data.frame(PassengerId = titanic_test$PassengerId, Survived = Prediction1)
# Write it into a file "Tree_Model1.CSV"
write.csv(submit1, file = "/Users/bkhan/Documents/Projects/titanic/Tree_Model1.csv", row.names = FALSE)
```


We check our prediction model’s performance. We check our prediction’s death and survive ratio on the test dataset and compare with the same ratio on the train dataset.

```{r}
# Inspect prediction
summary(submit1$Survived)
```

```{r}
prop.table(table(submit1$Survived, dnn="Test survive percentage"))
```

```{r}
#train survive ratio
prop.table(table(as.factor(titanic_train$Survived), dnn="Train survive percentage"))
```

The result shows that among a total of 418 passengers in the test dataset, 272 passengers predicted non survived (with survived value 0), which counts as 65%, and 146 passengers predicted to be survived (with survived value 1) and which count as 35%. This is not too far from the ratio on the training dataset, which was 62% non survived and 38% survived.

```{r}
print.data.frame(submit1)
```
