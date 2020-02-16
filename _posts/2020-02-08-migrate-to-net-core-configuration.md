---
title: Migrating your .NET Framework configuration to .NET Core
comments: true
tags:
  - dotnetcore
date: '2020-02-08 08:44 -0500'
---
## Problem

Migrating a large enterprise application written in .NET Framework to .NET Core can be  a lot of work and risky. But the benefits once complete are worth the effort. Configuration management in .NET Core is awesome. Not only is it primarily JSON based with cleanly divided sections, but you can also easily have your application read from multiple sources such as environmental variables, Azure Key Vault and so forth. The hot reload options and the ability to out of the box bind sections to strongly typed classes makes the migration worthwhile. But how do one go about safely migrating a legacy app's XML based configuration to .NET Core's JSON based style? With emphasis on safely. Let's have a look. 

## Proposed solution

It's possible to migrate your configuration while still targeting .NET Framework. To do so, you'll need install and use the `Microsoft.Extensions.Configuration` package, and refactor your application to not depend on `System.Configuration.ConfigurationManager.` This can sound very easy, but it can in fact be very complex depending on the size of the application.

To safely perform this migration, I propose a 4-step approach:

1. Install `Microsoft.Extensions.Configuration` and create a custom configuration provider 
2. Remove any static reference to `System.Configuration.ConfigurationManager`
3. Convert `web.config`/`app.config` to `appsettings.json`
4. Leverage strongly typed configuration classes

### 1. Install `Microsoft.Extensions.Configuration` and create a custom configuration provider

We can continue to read configuration values from the `web.config` while migrating to `Microsoft.Extensions.Configuration`. [Ben Foster](https://benfoster.io/blog/net-core-configuration-legacy-projects) has written an excellent blog post on how to do this, in which he creates a custom configuration provider that reads and parses values from the `web.config`. Before doing anything else, I would recommend you:

1. Installing the following NuGet packages:

   * `Microsoft.Extensions.Configuration`
   * `Microsoft.Extensions.Abstractions`
   * `Microsoft.Extensions.Primitives`
2. Creating a custom configuration provider by following the steps in [Ben Foster's](https://benfoster.io/blog/net-core-configuration-legacy-projects) post
3. Adding the following to your Global.asax or Startup.cs file

   ```
   IConfiguration configuration = new ConfigurationBuilder()
       .Add(new LegacyConfigurationProvider())
       .Build();
       
     //Register IConfiguration in your DI framework
   ```

### 2. Remove any static reference to `System.Configuration.ConfigurationManager`

Many legacy .NET Framework apps either directly use the static `System.Configuration.ConfigurationManager` instance to fetch their configuration values, or have their own abstraction of it so that it's not directly exposed across the application. Regardless of how your application does it, we will need to replace the usage of `System.Configuration.ConfigurationManager` across the application with `IConfiguration` registered in step 1. I would recommend doing step 1 and 2 together in the same PR, as it's a great way to set a solid foundation for our migration work. This change would then be able to safely go to production once all tests passes.  

### 3.  Convert `web.config` and `app.config` to `appsettings.json`

Alright, so we have the piping setup and our application now depending on `IConfiguration` instead of `System.Configuration.ConfigurationManager.` The next step is to convert our `web.config`'s to `appsettings.json`

There are a couple of things to keep in mind while doing so:

#### 3.1 Organize your configuration values in sections

```
{
  "FeatureToggles: {
    "Feature1Enabled": "true:,
    "Featuer2Enabled": "false:
  },
  "ConnectionStrings: {
    "Database": "ConnectionStringValue"
  }
}
```

#### 3.2 Configuration values are now retrieved by `Section:Key`

As we are now using sections, we'll have to make sure to update any reference to a configuration value so that it is prefixed with the section name. This can be easy to forget, especially if you're using a tool such as Octopus to transform your configuration before deployment.

### 4. Leverage strongly typed configuration classes

The final step in the transformation is to refactor our application to depend on strongly typed configuration classes. For each section in the `appsettings.json` file, create a corresponding settings class. Such a settings class could look like below:

```
using System; 

public class FeatureToggles 
{
  public bool Feature1Enabled { get; set; }
  
  public bool Feature2Enabled { get; set; }
}
```

We can then bind and register each specific configuration class in our DI framework, which will allow us to inject specific settings classes only to instances that depend on them. A great separation of concern.

```
IConfiguration configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json.config", optional: true)
    .Build();
    
var featureToggles = configuration.GetSection(nameof(FeatureToggles));
////Register your instance in your DI framework
```

### Summary

I hope this post has given you some ideas on how you can safely get started migrating the configuration in your .NET Framework app to .NET Core. As always feel free to reach out on Twitter or LinkedIn, should you have any questions. Happy coding!
