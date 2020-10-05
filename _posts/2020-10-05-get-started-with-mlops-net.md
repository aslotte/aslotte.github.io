---
title: Get started with MLOps.NET
comments: true
tags:
  - mlops mlops.net
date: 2020-10-05 09:49 -0400
---
In this upcoming series I want to dive into the world of MLOps and [MLOps.NET](https://github.com/aslotte/MLOps.NET), and explore how MLOps.NET can help you set your organization up for success by managing and automating the machine learning life-cycle. 

![](/images/post-images/pexels-snapwire-618612-1-.jpg)

## What is MLOps

MLOps have over the last year quickly emerged as a vital instrument in managing and automating the flow of a given machine learning model. As machine learning become more and more mainstream,  more organizations venture into the world of intelligent applications. As the number and complexity of models grow, it can quickly become a nightmare knowing which model was deployed where and on what data and with which parameters it was trained on. This is where MLOps can help. MLOps as a practice is very broad and involves everything from tracking data sources (also see [DataOps](https://en.wikipedia.org/wiki/DataOps)), tracking experiments and runs, logging hyper-parameters, managing evaluation metrics, automating the deployment of models, measuring data drift in the wild and much more. Throughout this series, we'll dive into how MLOps.NET can support all of this when training machine learning models using [ML.NET](https://github.com/dotnet/machinelearning)

## Why do we need MLOps.NET?

As MLOps have become increasingly, the number of tools to use have quickly grown. One of the more popular MLOps tools out there today is Databrick's MLflow, but Azure Machine Learning, AWS Sagemaker and Kubeflow are not far behind. 

So if there're already a number of great tools out there, why do we need MLOps.NET? Well, all the tooling to date only supports models built in Python or R, using libraries such as Scikitlearn, PyTorch or Tensorflow. Although it's possible to use some existing tooling to track e.g. run metrics, none of them support ML.NET model deployments, and none of the integrates natively with an ML.NET model. 

## How to get started with MLOps.NET

To get started with [MLOps.NET](https://github.com/aslotte/MLOps.NET), there're two fundamental concepts one should know. The **storage provider** one should provision for the model's metadata and the provider for the **versioned model repository** in which we'll store our trained and deployed models. 

MLOps.NET to date currently supports using a SQLite, SQL Server or a Cosmos DB as our storage provider for metadata, and an Azure Blob Storage, AWS S3 or a local file share for the versioned model repository. 

The design of MLOps.NET mimics in large parts of that of ML.NET, but instead of being centered around a `MLContext`, we use a `MLOpsContext`. The `MLOpsContext` can be configured using an `MLOpsBuilder`.

If you for example would like to use a Cosmos Db for your metadata and an Azure Blob Storage for your model repository, building a `MLOpsContext` would look something like this

```csharp
  IMLOpsContext mlOpsContext = new MLOpsBuilder()
    .UseCosmosDb("accountEndPoint", "accountKey")
    .UseAzureBlobModelRepository("connectionString")
    .Build();
```

If you on the other hand wanted to use a SQLServer and a local file share then building a `MLOpsContext` would look like this. 

#### SQL Server with Local model repository

```csharp
  IMLOpsContext mlOpsContext = new MLOpsBuilder()
    .UseSQLServer("connectionString")
    .UseLocalFileModelRepository()
    .Build();
```

As you can image there are many more combinations, and you are free to configure the tool to your needs.

### Wrapping up

I have in this brief article introduced MLOps and MLOps.NET, and we've had a chance to look at how we can configure the `MLOpsContext`. In some upcoming posts we'll dive into experiment tracking, logging training runs and much more!