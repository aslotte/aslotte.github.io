---
title: Exploring Model Repositories with MLOps.NET
comments: true
tags:
  - modelrepository mlops
date: 2020-10-11 17:31 -0400
---
What is a model repository in the context of MLOps, and why do we need one?

![](/images/post-images/storage.jpg)

## Model repositories

A model repository is source control for machine learning. Just as Git, or the source control system of your choice, is a vital piece of modern software development, a model repository is a centralized storage in which we can store and access machine learning models. 

We could in theory use Git to store our models in, and use release branches or tags on the trunk to symbolize the release of a new version. However, machine learning models can easily become big, especially deep learning models, and a source control system is not optimized for large individual files. Nor is it optimized for incrementing the version of a model, organizing models in an easily discoverable way or keeping a robust audit trail to a model's metadata. 

You can implement a model repository in several ways. The most straightforward approach would be to manually provision an Azure Storage Container or an AWS S3 Bucket and manually set up a model repository, keeping track of the version and the link to a model's metadata. As your organization grows and the number of models increases, this quickly becomes a difficult solution to scale. To solve this, all major MLOps tools out there, such as MLflow and Azure Machine Learning implement a versioned model repository to which you can upload  trained models for an experiment with the ability to register them so that they can be deployed to e.g. stage or production.

## Using a Model Repository in MLOps.NET

[MLOps.NET](https://github.com/aslotte/MLOps.NET) implements support for a versioned model repository and ensures that the version number is incremented automatically (upon registration of a run artifact) and the link to a model's metadata is persisted. The library currently supports using either Azure Blob Storage, AWS S3 or a local file share as the backing storage provider for the model repository, depending on where you want to keep your models.

Let's take a look at how you can configure a model repository in [MLOps.NET](https://github.com/aslotte/MLOps.NET), and how your models get organized internally.

### Configuring a Model Repository

You can configure the type of model repository to use while building an `IMLOpsContext`

### Azure Blob Storage

To use Azure Blob Storage, call the `UseAzureBlobModelRepository` builder extension method, supplying the connection string to your blob storage.

```
  IMLOpsContext mlOpsContext = new MLOpsBuilder()
    .UseSQLite()
    .UseAzureBlobModelRepository("connectionString")
    .Build();
```

### AWS S3

To use AWS S3, call the `UseAWSS3ModelRepository` builder extension method and supply the AWS access key, secret access key and the region name.

```
  IMLOpsContext mlOpsContext = new MLOpsBuilder()
    .UseSQLite()
    .UseAWSS3ModelRepository("awsAccessKey", "awsSecretAccessKey", "regionName")
    .Build();
```

### Local file share

Finally, to use a local file share, call the `UseLocalFileModelRepository` builder extension method. You can optionally provide a path to a root folder. If no path is supplied, models will be stored under `C:\Users\{user}\.mlops\model-repository`

```
  IMLOpsContext mlOpsContext = new MLOpsBuilder()
    .UseSQLite()
    .UseLocalFileModelRepository()
    .Build();
```

The internal structure of the model repository is flat, and each model is stored with the name of the unique GUID that represents the identifier of the run it originated from. However, there's no need to look into the internals of the model repository as MLOps.NET will manage this for you.

![](/images/post-images/model-repo.png)

To upload a model to a model repository, call the `UploadAsync` method on the `Model` catalog, passing in the run id and the path to the trained model. 

```
await mlOpsContext.Model.UploadAsync(run.RunId, "{pathToTheModel}");
```

This will upload the model to the model repository as a run artifact and can if you like be done for each model training run. At this point the model is just associated with a run, but it is not available for deployment.

To make a model available to be deployed, you need to register it. This is normally done for models you think are potential release candidates.

To register a model call the `RegisterModel` method on the `Model` catalog. 

```
await mlOpsContext.Model.RegisterModel(run.ExperimentId, runArtifact.RunArtifactId, "registered by");
```

This will automatically increment the version of the model for a given experiment. For example, you may have a model currently deployed to production for experiment A with version 3. If you call the `RegisterModel` method your newly registered model will be assigned version 4.

## Wrapping up

In this post we have explored the importance of a centralized versioned model repository where  can store the machine learning models we train. We have also taken a deep dive into the internals of MLOps.NET to understand how we can configure the library to use a repository of our choice.



Happy coding!