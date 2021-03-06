# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
This dataset contains data about individuals who have been interesting in the banks services. The opportunity at hand is to see if we can identify patterns in individuals that subscribed to the bank's services. 

When building a model there were two approaches used:
1) A logistic regression model using hyperdrive to optimize C and max iteration
2) An AutoML approach that tries multiple different normalization techniques and models to find the best model

The best performaning model came from the AutoML approach when it used a VotingEnsemble model resulting in an accuracy of 91.71%

## Scikit-learn Pipeline
**Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**

The sklearn model is a Logistic Regression and the two parameters available to tune are 
- C: The regularization
- Max_iter: The maximum number of iterations used to fit the parameter

The model architecture uses hyperdrive to optimize the logistic regression parameters by using a random choice sample from the specifice list.

For C, the regularization it will choice between 0.5 and 1 and for max iters it randomly draws from 50, 100 and 150 for each run.

The hyperdrive architecture will randomly draw parameter options from the specifies list and continue until it runs out of options or hits its early stopping policy. The early stopping policy is a bandit policy which uses two parameters an evaulation interval and a slack factor.
- Evaulation interval: The frequency the bandit policy should be applied. Everytime the hyperdrive runs an iteration it counts as one interval
- Slack factor: The ratio the bandit policy calcualtes the allowed distance from the best performing model

A benefit of using the bandit policy is that it will automatically terminate runs that are not producing good results and only keep the best performing runs that are withing the allowed slack factor.

The data consist of continuous and categorical variables. In the train preprocessing step, the categorical variables are converted to:
- The months variables is decode to their respective monthly value;, i.e., Jan: 1
- The week gets its corresponding week values; i.e., mon: 1
- The diferent job, contact and education values are converted to binary dummy variables
- Martial, default, house and loan are converted to binary values

## AutoML
The autoMl arcitechture used was:

```python
automl_config = AutoMLConfig(
    experiment_timeout_minutes=30,
    task='classification',
    primary_metric='accuracy',
    training_data=ds,
    label_column_name='y',
    n_cross_validations=3)
```

The configuation parameters mean that experiement will time out after 30 minutes if it has not already terminated. It will only evaluate classification models using accuracy as its elvuation criteria. The training data is a tabular data set called ds. In the data set the variable the model will predict is called 'y' and the data used from training is split into the cross validation data sets. 

The best performing autoML model is a VotingEnsemble model with a standard scaller as a preprocessing step. The cool feature about the autoML ensemble models is that it does not retrain the models. It uses a soft-voting which uses weighted averages from the models that have already been trained. This means the ensemble model uses a hybrid approach by combining the results from XGBoostClassifier, ExtremeRandomTrees and RandomForest. It builds a tree using 25 estimators on the predicted Y values.

## Pipeline comparison

The table below has the metrics from the best model for each approach. The accuracy between the two models is minimal.
	
  ![image](https://user-images.githubusercontent.com/6833720/109431338-e94dcc80-79ba-11eb-9c23-2c6f1089a14e.png)
  
 The achitecture is compeltely different between the two. The hyperdrive is running a random search to find the best parameters for the logistic regression model. The automl is trying multiple different models and normalization techniques to find the best performing model. If you let the autoML go long enough it usually will always beat the parameter optimization approach of a single model. The autoML eventually starts to explore essemble models to find the best perfmiing architecture and not rely on a single model.

## Future work
**Data Balance Issues**

The data is extermely imblanced which is evident by some of the metric the autoML returned with the best performing model. The accuracy of the model was 0.91 but when you calcualte the balanced accuracy it drops to 0.749. This occurs because the model has gotten pretty good at predicting the majority class and does not do so well on the minority class. 

Class imbalance is a very common issue in the business world were the business usually wants to know when the unusall thing occurs and not the majority class. There are a few solutions to get around this issue.

In the autoML config, we could change the metric used from a simple 'accuracy' to something that attempts to adjust the results using class weights. A few examples of this are AUC, AUC_weighted, precision_score_weighted, balanced_accuracy and weighted_accuracy.

Besides just changinging the metric used to find the best performing model you would also over sample the minority class or down sample the majority class so the data used to train the model is more balanced  and one class does not dominate the other.

**Feature Creation**

I did not explore creating new features with the data set. After running the benchmark model, you could explore the feature importance to each class and try to create features that help explain the variance for the minority class.

## Proof of cluster clean up

![image](https://user-images.githubusercontent.com/6833720/109432447-1fda1600-79c0-11eb-8856-6b1736c85463.png)

