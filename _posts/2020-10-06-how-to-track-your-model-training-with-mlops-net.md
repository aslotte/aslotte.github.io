---
title: Experiment Tracking with MLOps.NET
comments: true
tags:
  - mlops mlops.net experiments runs
date: 2020-10-06 16:20 -0400
---
In this post I want to dive further into what experiments and runs are, and how we can use MLOps to track various aspects of the model training.


## What are an experiment and a run?

In the world of MLOps, experiments and runs are two fundamental concepts you'll see mentioned across all the tooling out there. You can view an experiment as the problem you are trying to solve with your model, while each run for an experiment represents each individual attempt to train a machine learning model to solve that specific problem. To give an example, we may want to train a model to detect fraudulent transactions. The attempt to train a model to solve this problem will be the experiment, while each attempt to train a model to solve it will be an individual run. As I am sure you can see, each run will have a number of characteristics associated with it, for example what data was used to train the model, what algorithm we picked, how we transformed the data and so forth. It is important to keep track of all of this information as it will more easily let you understand where a model came from and why it may make a certain prediction.

The process of doing this is called experiment tracking. Let's take a look at what capabilities we currently have in MLOps.NET to do this.

## Creating a Run
Once you've configured and built your `MLOpsContext` (see my previous post) it's time to create an experiment and a run. The easiest way to do this is to create an experiment and a run in one go (if the experiment already exists it will just add a new run to it)

```
var run = await mlOpsContext.LifeCycle.CreateRunAsync(experimentName: "Titanic");
```

## Logging Training and Test data
Training a model always start with the data. How the model performs will greatly depend on the data it has been trained on. 

In the majority of cases you want to log data attributes such as what columns are present and their type, as well as a calculated hash to know if the data has changed in between training runs. In certain cases you may also want to calculate and log the specific distribution of a column which is particularly important for classification scenarios.

All operations to log data is present on the `DataCatalog`.

```
//This will log all columns and their type as well as calculating a hash for the dataset
await mlOpsContext.Data.LogDataAsync(run.RunId, data);

//This will log the data distribution of the Age column
await mlOpsContext.Data.LogDataDistribution<int>(run.RunId, data, nameof(ModelInput.Age));
```

## Logging Algorithm and Hyper-Parameters
How did my binary decision tree do against that time I tried using support vector machines, or what did I actually use? Tracking what algorithm was used for which model, and to some extent what types of hyper-parameters the algorithm was fine-tuned with can be a lot. MLOps.NET makes this simple by integrating seamlessly with ML.NET. By passing the selected trainer to the library, MLOps.NET will automatically log all of this for you.

```
await mlOpsContext.Training.LogHyperParametersAsync(run.RunId, trainer);
```

## Tracking a model's evaluation metrics

- Metrics

- Confusion matrix

## Wrapping up