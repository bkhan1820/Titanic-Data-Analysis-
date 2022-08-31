#  Titanic Data Analysis 

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
