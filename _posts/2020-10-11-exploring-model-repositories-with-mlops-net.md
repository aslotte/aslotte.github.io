---
title: Exploring Model Repositories with MLOps.NET
comments: true
tags:
  - modelrepository mlops
date: 2020-10-11 17:31 -0400
---
What is a model repository in the context of MLOps, and why do we need one?

## Model repositories
A model repository is source control for machine learning. Just as Git, or the source control system of your choice, is a vital piece of modern software development, a model repository is a centralized storage in which we can store and access **versioned models** machine learning models. 

We could in theory use Git to store our models in as well, and use release branches or tags on the trunk to symbolize the release of a new version. However, machine learning models can easily become big, especially deep learning models, and a source control system is not optimized for large individual files. Nor is it optimized for incrementing the version of a model, organizing models in an easily discoverable way or keeping a robust audit trail to a model's metadata. 

You can implement a model repository in several ways. The most straightforward approach would be to manually provision an Azure Storage Container or an AWS S3 Bucket and manually set up a model repository, keeping track of the version and the link to a model's metadata. As your organization grows and the number of models increases, this quickly becomes a difficult solution to scale. To solve this, all major MLOps tools out there, such as MLflow and Azure Machine Learning implements a versioned model repository to which you can upload  trained models for an experiment you may want to proceed with, e.g. deploying to a staging environment or production. 

## Should all models be stored in a model repository? 
The short answer is no. Data science is highly experimental and you will throughout your journey in finding a good model discover many bad ones. Each model you train as part of a run is commonly referred to a run artifact. By logging metadata about your previous runs, you can determine if a newly trained model is better than the one you currently have uploaded to a model repository, or that may be deployed to production use. If the model you just trained doesn't meet the expectations, then there is no need to store it with a new version in a model repository.

## Using a Model Repository in MLOps.NET
MLOps.NET offers support for a version model repository and makes sure to manage everything for you from automatically incrementing the version number of registered models to keeping a link from a model to its related metadata. The library currently supports using either Azure Blob Storage, AWS S3 or a local file share as the backing storage provider for the model repository, depending on where you want to keep your models.

Let's take a look at how you can configure a model repository and how your models get organized.
