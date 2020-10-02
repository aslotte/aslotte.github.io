---
title: Announcing MLOps.NET v1.2
comments: true
tags:
  - mlnet mlops machinelearning
date: 2020-10-02 08:16 -0400
---
It's with great pleasure I announce the release v1.2 of MLOps.NET! The library has come a long way since the first code file was committed in May. We each submitted PR, we inch forward towards a complete end-to-end MLOps solution for ML.NET.

\## What is MLOps.NET?

MLOps.NET is a data science tool designed to manage and handle the machine learning lifecycle for a model trained in ML.NET. MLOps as a practice has over the last year quickly emerged as a vital instrument to successfully deploy machine learning models to production, while keeping track of how they were trained, what data they were trained on and much more.

As a response to the lack of MLOps tooling for ML.NET, the journey to develop MLOps.NET was embarked with the vision that it would seamlessly support managing, tracking and deploying ML.NET models.

Previous releases of the library have added support for a multitude of use cases such as experiment tracking, logging hyper parameters and data as well as deploying a model to a Uri endpoint so that e.g. an ASP.NET Core application may consume the trained and deployed model. 

MLOps.NET is highly configurable and allows a user of the library to store their metadata about a model either on a SQLite, SQL Server or Cosmos database. The tool furthermore implements a versioned model repository that can be backed by either Azure Blob Storage, AWS S3 or a local file share. 



\## What is new in v1.2

So what is new in v1.2? From the start, the goal of v1.2 was to support containerized model deployment of ML.NET models to a Kubernetes cluster. 



\### Containerized Model Deployment to Kubernetes