---
title: Experiment Tracking with MLOps.NET
comments: true
tags:
  - mlops mlops.net experiments runs
date: 2020-10-06 16:20 -0400
---
This is the second post in my MLOps.NET series, in which we explore MLOps as a discipline and MLOps.NET as the tool to support that process. In this post I want to explore experiments and runs.

## What are experiments and runs?

In the world of MLOps, experiments and runs are two fundamental concepts you'll see mentioned across all the tooling out there. You can view an experiment as the specific problem you are trying to solve with your model, while each run represents an attempt to train a machine learning model. To give an example, we may want to train a model to detect fraudulent transactions. The overarching goal solve this problem will be the experiment, while each attempt to train a model to solve it will be an individual run. As I am sure you can see, each run will have a number of characteristics associated with it, for example what data was used to train the model, what algorithm we picked, how we transformed the data and so forth. It is important to keep track of all of this information as it will more easily let you understand where a model originated from.  

This particular process is often referred to as experiment tracking. Let's take a look at how MLOps.NET can support this.

## Creating a Run

Once you've configured and built your `MLOpsContext` (see [Get Started with MLOps.NET](https://www.alexanderslotte.com/get-started-with-mlops-net/)) it's time to create an experiment and a run. The easiest way to do this is to create an experiment and a run in one go (if the experiment already exists it will just add a new run to it)

```
var run = await mlOpsContext.LifeCycle.CreateRunAsync(experimentName: "Titanic");
```

## Logging Training and Test data

Training a model always starts with the data. How the model performs will greatly depend on the data it has been trained on. 

In most cases you are interested in logging the name and data type of each column that was present during model training, and in certain cases you may also want to calculate a computed hash for your data so that you can determine if the data has changed in between runs. In certain cases you may also want to calculate and log the specific distribution of one or more columns, which is particularly important for classification scenarios.

All methods to log data can be found on the `Data` catalog.

```
//This will log all columns and their type in addition to calculating a hash for the dataset
await mlOpsContext.Data.LogDataAsync(run.RunId, data);

//This will log the data distribution of the Age column
await mlOpsContext.Data.LogDataDistribution<int>(run.RunId, data, nameof(ModelInput.Age));
```

## Logging the Algorithm and Hyper-Parameters

How did my Binary Decision Tree do against that time I tried to use Support Vector Machines, or wait, what did I actually use? 
Tracking algorithms and associated hyper parameters can quickly become cumbersome, especially for multiple runs and multiple models.  MLOps.NET makes this simple by integrating seamlessly with ML.NET. By passing the selected trainer to the library, MLOps.NET will automatically log all of this for you in one go.

You'll find this method available on the `Training` catalog

```
await mlOpsContext.Training.LogHyperParametersAsync(run.RunId, trainer);
```

If we peek into the configured storage provider for the metadata, after we have called `LogHyperParametersAsync` it should look something like this:

![](/images/post-images/hyperparameter.png)

## Tracking a Model's Evaluation Metrics

A model that doesn't perform well is a model not worth continuing with. Similarly, it's important to know if the model you're currently training is better than one that you have already deployed to production. As such, it's important to track evaluation metrics such as accuracy, F1 scores, recall, precision and so forth. Tracking metrics allows you to plot the performance of a model vs changes in e.g. hyper-parameters or training data. 

You can find the methods to log metrics in the `Evaluation` catalog.

```
//Log evaulation metrics
await mlOpsContext.Evaluation.LogMetricsAsync(run.RunId, metrics);

//Optionally log the result of a confusion matrix
await mlOpsContext.Evaluation.LogConfusionMatrixAsync(run.RunId, metrics.ConfusionMatrix);
```

If we again peek into the configured storage provider, it should look something like this:

![](/images/post-images/metrics.png)


## Wrapping up
As MLOps.NET continues to mature, we'll see additional features introduced to support the experiment tracking process. Keeping a detailed audit trail on how your model came to be is becoming increasingly more important and as federal regulations catch up with the AI/ML industry this will become a necessity. 