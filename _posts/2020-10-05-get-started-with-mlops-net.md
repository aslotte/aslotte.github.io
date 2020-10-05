---
title: Get started with MLOps.NET
comments: true
tags:
  - mlops mlops.net
date: 2020-10-05 09:49 -0400
---
In this upcoming series I want to dive into the world of MLOps and MLOps.NET, and explore how MLOps.NET can help you set your organization up for success by managing and automating the machine learning life-cycle. 

## What is MLOps
MLOps have over the last year quickly emerged as a vital instrument in managing and automating the flow of a given machine learning model. As machine learning become more and more mainstream,  more organizations venture into the world of intelligent applications. As the number and complexity of models grow, it can quickly become a nightmare knowing which model was deployed where and with what data and hyper-parameters it was trained on. This is where MLOps can help. MLOps as a practice is very broad and involves everything from tracking data sources (also see [DataOps](https://en.wikipedia.org/wiki/DataOps)), tracking experiments and runs, logging hyper-parameters, managing evaluation metrics, automating the deployment of models and measuring data drift in the wild and much more. Throughout this series, we'll dive into how MLOps.NET can support this when training machine learning models using [ML.NET](https://github.com/dotnet/machinelearning)

## Why do we need MLOps.NET?
As MLOps has become increasingly more and more popular, the number of tools to use has quickly grown. One of the more popular MLOps tools out there today is Databrick's MLflow, but Azure Machine Learning, AWS Sagemaker or Kubeflow are not far behind. 

So if there're already a number of great tools out there, why do we need MLOps.NET? All the tooling to date only support models built in Python or R, using libraries such as Scikitlearn, PyTorch or Tensorflow. Although it's possible to use some existing tooling to track e.g. run metrics, none of the support ML.NET model deployments, and none of the integrates natively with an ML.NET model. 

## How to get started with MLOps.NET
