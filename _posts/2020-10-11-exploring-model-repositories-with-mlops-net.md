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