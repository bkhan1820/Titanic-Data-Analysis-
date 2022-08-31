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
