---
title: Migrate to .NET Core configuration
comments: true
tags:
  - dotnetcore
date: '2020-02-08 08:44 -0500'
---
## Problem

Migrating a large enterprise application written in .NET Framework to .NET Core can be  a lot of work and risky. But the benefits once complete are worth the effort. Configuration management in .NET Core is awesome. Not only is it primarily JSON based with cleanly divided sections, but you can also easily have your application read from multiple sources such as environmental variables, Azure Key Vault and so forth. The hot reload options and the ability to out of the box bind sections to strongly typed classes makes the migration worthwhile. But how do one go about safely migrating a legacy app's XML based configuration to .NET Core's JSON based style? With emphasis on safely. Let's have a look. 

## Proposed solution

The first thing to keep in mind, is that it's completely possible to do this conversion before you fully migrate your projects to .NET Core. The following NuGet packages needs to get installed:

* `Microsoft.Extensions.Configuration`
* `Microsoft.Extensions.Abstractions`
* `Microsoft.Extensions.Primitives`

By installing these packages, you will be able to build a configuration using the `ConfigurationBuilder `in your `Global.asax` or `Startup.cs` file. This `IConfiguration `can be registered in your dependency injection framework. At the end, the code will look something like this, which will register your appsetting.json file.

```
IConfiguration configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json.config", optional: true)
    .Build();
```



However, to get there safely, I propose a 4-step approach:

1. Create a custom configuration provider and inject `IConfiguration`
2. Remove any static reference to `System.Configuration.ConfigurationManager `
3. Convert `web.config`/`app.config` to `appsettings.json`
4. Leverage strongly typed configuration classes



### 1. Create a custom configuration provider

We can continue to read our configuration values from a web- or app.config despite transitioning to the package `Microsoft.Extensions.Configuration`. [Ben Foster](https://benfoster.io/blog/net-core-configuration-legacy-projects) has written an excellent blog post on how to do this, in which he creates a custom configuration provider that reads the appsettings and connection string values from our web.config. 



### 1. Remove any static reference to System.Configuration.ConfigurationManager

Many legacy .NET Framework applications either use the static System.Configuration.ConfigurationManager to fetch configuration settings, or have their own abstraction of the ConfigurationManager, such that it's not directly exposed across the application. Before we even consider migrating our configuration we would need to remove any of these static references. If you 



#### 1.1 Create a custom ConfigurationBuilder

Install
