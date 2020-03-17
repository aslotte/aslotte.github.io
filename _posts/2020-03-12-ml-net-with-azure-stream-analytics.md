---
title: ML.NET with Azure Stream Analytics
comments: true
tags:
  - mlnet azurestreamanalytics
date: '2020-03-12 19:53 -0400'
---
In this post I will explain how we can set up a real-time data streaming pipeline to achieve real-time inference with ML.NET using Azure Stream Analytics and C# user-defined functions (UDF).

Integrating ML.NET with Azure Stream Analytics is something I've been wanting to do for almost a year, and it's kind of funny story how all of this came about. I'd previously worked extensively with Azure Stream Analytics and ML.NET separately, but never together. When ProgNET's CFP in London went live in April/May of 2019, I thought to myself, why not submit a half-day workshop on how to build a model from scratch and deploying it to a real-time data pipeline in Azure. Little did I know that what I was envisioning was not yet possible, and something I probably should have confirmed before getting accepted to speak. Fortunately for me I was able to work around it using Azure Functions for my workshop, but today I'm thrilled to see that Azure Stream Analytics finally offers native support for ML.NET.









\- Explain C# UDF with Azure Stream Analytics

\- Why is this cool?

\- Show examples