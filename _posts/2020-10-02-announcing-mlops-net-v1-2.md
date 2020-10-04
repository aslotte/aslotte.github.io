---
title: Announcing MLOps.NET v1.2.0
comments: true
image: /images/post-images/wheels.jpg
tags:
  - "#mlnet #mlops #machinelearning"
date: 2020-10-04 08:16 -0400
---
It is with great pleasure I announce the release of MLOps.NET v1.2.0! The library has come a long way since the first code file was committed in May. With each submitted PR, we inch forward towards a complete end-to-end MLOps solution for ML.NET

![](/images/post-images/wheels.jpg)

## What is MLOps.NET?

[MLOps.NET](https://github.com/aslotte/MLOps.NET) is a data science tool designed to manage and handle the machine learning lifecycle for a model trained in ML.NET. MLOps as a practice has over the last year quickly emerged as a vital instrument to successfully deploy machine learning models to production, while keeping track of how they were trained, what data they were trained on and much more.
In response to the lack of MLOps tooling for ML.NET, the journey to develop MLOps.NET was embarked with the vision that it would seamlessly support managing, tracking and deploying ML.NET models.

Previous releases of the library have added support for a multitude of use cases such as experiment tracking, logging hyper parameters and data, as well as deploying a model to a Uri endpoint so that, for example, an ASP.NET Core application may consume the trained and deployed model. 

MLOps.NET is highly configurable and allows the user of the library to store their metadata about a model either on an SQLite, SQL Server or Cosmos database. Furthermore, the tool implements a versioned model repository that can be backed by either Azure Blob Storage, AWS S3 or a local file share. 

To can download the library from [NuGet](https://www.nuget.org/packages/MLOps.NET/)

## What is new in v1.2.0

So, what is new in v1.2.0? From the start, the goal of v1.2.0 was to support containerized model deployment of ML.NET models to a Kubernetes cluster, ideally with one line of code. There were a lot of moving parts that needed to come together to make this happen. Let us dive into it! 

### Containerized Model Deployment to Kubernetes

Before we look at how we can utilize MLOps.NET to deploy a model to Kubernetes, let us take a step back and think about what actions we need to take, should we do this manually.

We need to:

* Auto-generate an ASP.NET Core Web App to serve the ML.NET model
* Decompile run-time instances of the models input and output schema and include that as part of the ASP.NET Core Web App
* Automatically detect any used package references so that they can be added to the ASP.NET Core Web App
* Download and include a registered model from a given model repository
* Build a complete Docker image
* Push the built Docker image to a private or public container registry either on-premise or in the cloud
* Connect to and create a Kubernetes namespace to deploy services to
* Create a secret in the Kubernetes cluster for any needed image pull secrets
* Create parameterized Kubernetes manifest files to deploy the built Docker image to a Pod in a replica set exposed to the world through an external IP address via a load balancer ingress 
* Apply the Kubernetes manifest files to the cluster in a namespace specific to an experiment and deployment target (e.g. stage vs prod)
* Persist and return the URL to which a user can access the deployed service

That is a lot of steps just to get a model deployed as a container, especially if you would have to do it manually. Let us explore how MLOps.NET solves this.

#### ASP.NET Core Web App Template

To auto-generate an ASP.NET Core Web App that is customized to serve ML.NET models via a RESTful endpoint, we can use a `dotnet new` template. There were currently no ML.NET specific templates when I set out to do this, so we went ahead and created a new repo containing an assortment of [templates](https://github.com/aslotte/ML.NET.Templates) that can be used either for MLOps.NET or anytime you need a template to train or deploy a mode. 

#### Decompiling run-time instances of the model's input and output

How can we ensure that the RESTful endpoint we will use to make predictions can accept and return the same model schema that the model was trained on?

```csharp
[HttpPost]
public ModelOutput Predict(ModelInput modelInput)
{
    return this.predictionEnginePool.Predict(modelInput);
}
```

As we can see, the JSON payload will be of type `ModelInput` and it will return a `ModelOutput`. Given that we need to ensure that the `ModelInput` and `ModelOutput` matches that of which the model has been trained on. If they do not, we will get run-time errors as we try to make predictions. To achieve this we can use [ILSpy](https://github.com/icsharpcode/ILSpy) to decompile a run-time instance and include that as a class in the Web App. It is possible to either register the schema beforehand during the run, or pass the run-time instances in to the deployment method and MLOps.NET will do this decompilation on the fly.

#### Detecting and installing package dependencies

In addition to the model schema, we will also need to copy over the physical model to make sure it is available for the application. However, in addition we also need to make sure that all required package dependencies are installed. ML.NET consists of a flora of packages ranging from `Microsoft.ML.FastTree` for decision trees to `Microsoft.ML.ImageAnalytics` for image support, and we need to ensure that the correct dependencies are included, or else we will see run-time exceptions. My first thought to achieve this was to scan the loaded AppDomain for all and any Microsoft.ML assemblies, but that quickly proved to be a rabbit hole given that not all dependencies are loaded until they are used, and the list would also include a lot of sub-dependencies not listed as NuGet packages. The way MLOps.NET solves this problem is by reading the `deps.json` file provided by the compiled application and based on that determining what the dependency graphs looks like.

#### Building and pushing a Docker image

With the final puzzle piece in place building a basic Docker image for the ASP.NET Core Web App is pretty straightforward. The image is then pushed to a configurable private or public image repository, which we will see examples of later.

#### Deploying the image to a Kubernetes cluster

Deploying a model to a Kubernetes cluster in the form of container image is a fantastic way to ensure performance, resilience and availability of a machine learning model. In v1.2.0 of MLOps.NET, the library will deploy the trained model as a replica set of one pod and expose the model to the world via an ingress load balancer. For simplicity’s sake it deploys the service and pod to a new namespace called `{experimentName-deploymentTargetName}`. This will allow models currently being tested and models in production to be separated, which allows for the possibility to apply different access and resource requirements as needed. Future releases of MLOps.NET will open up the possibility to configure various types of deployment settings, such as the type of load balancer, number of replicas and minimum amount of resources to be allocated.

## Show me the code

This all sounds amazing, so how do we make it happen? To deploy an ML.NET model to a Kubernetes cluster, two things need to happen. 

### 1. Configure the use of a Container Registry and a Kubernetes Cluster

We need to provide the name and credentials to a container registry and the content or path to a Kubernetes kubeconfig. If you want to push an image to a public image registry, you only need to provide the registry name. 

There are different ways of getting the credentials depending on if you host your container registry and Kubernetes cluster locally or in Azure or AWS. If you host them in Azure, you can get the Azure Container Registry credentials by running

```
az acr credential show --name {nameOfContainerRegistry}
```

Similarly, you can get the kubeconfig to your Azure Kubernetes Service by running

```
az aks get-credentials --resource-group {resourceGroupName} --name {serviceName} --file kubeconfig
```

To configure MLOps.NET, use the MLOpsBuilder

```csharp
IMLOpsContext mlOpsContext = new MLOpsBuilder()
    .UseLocalFileModelRepository()
    .UseSQLite()
    .UseContainerRegistry("RegistryName", "UserName", "Password")
    .UseKubernetes("kubeConfigPathOrContent")
    .Build();
```

### 2. Deploy a registered model to a Kubernetes Cluster

The method `DeployModelToKubernetesAsync` exists on the `Deployment` catalog and takes two generic types as parameters for the model input and output. The user will also need to provide a registered model and a deployment target, which you can read more about how to create on the libraries GitHub Readme page.

If you do not want to provide the schema at deployment, there is also an option to register the schema during the run and at deployment use a method overload without the generic arguments. 

```csharp
    var deployment = await sut.Deployment.DeployModelToKubernetesAsync<ModelInput,   ModelOutput>(deploymentTarget, registeredModel, "deployedBy");

    //e.g. http://20.62.210.236/api/Prediction
    var uri = deployment.DeploymentUri;
```

If you do not want to provide the schema at deployment time, there's also an option to register the schema during the run, and at deployment time use a method overload without the generic arguments 

```csharp
    var deployment = await sut.Deployment.DeployModelToKubernetesAsync(deploymentTarget, registeredModel, "deployedBy");

    //e.g. http://20.62.210.236/api/Prediction
    var uri = deployment.DeploymentUri;
```

## Wrapping up

There is a lot to uncover with MLOps.NET v1.2.0, and there are more features added than just the Kubernetes support. My hope is that you will find this tool useful in your journey towards more reliable deployment of machine learning models to production. Should you have any thoughts, ideas or feedback on the tool is always welcome, please reach out or open an issue on the GitHub repo.

Happy coding