---
title: Exploring Model Repositories with MLOps.NET
comments: true
tags:
  - modelrepository mlops
date: 2020-10-11 17:31 -0400
---
What is a model repository in the context of MLOps, and why do we need one?

A model repository is source control for machine learning. Just as Git/GitHub/Bit Bucket/TFS or whatever source control system you may use for your software development, a model repository is a centralized storage in which we can store and access **versioned models**. 

We could in theory use GitHub to store our models and use release branches or tags on the trunk to symbolize the release of a new version. However, machine learning models can easily become fairly big, especially deep learning models, and a source control system is not optimized for large individual files. Nor is it optimized for automatically incrementing the version of a model, organizing them in an easily discoverable way or keeping a robust audit trail to a model's metadata. 