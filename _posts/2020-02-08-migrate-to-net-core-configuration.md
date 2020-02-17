---
title: Migrating configuration in .NET Framework to .NET Core
comments: true
image: /images/post-images/rima-kruciene-gpKe3hmIawg-unsplash.jpg
tags:
  - 'dotnetcore configuration'
date: '2020-02-16 08:44 -0500'
---
## The challenge

Migrating an enterprise application from .NET Framework to .NET Core can both be a lot of work and risky. But the benefits once complete are well worth the effort. One of the many things that have been improved is configuration management. Configuration management in .NET Core is awesome. Not only is it primarily JSON based with cleanly divided sections, but it's also possible to have your application read from multiple sources such as environmental variables, Azure Key Vault and so forth. The hot reload options and the ability to out of the box bind sections to strongly typed classes makes the migration worthwhile. But how do one go about safely migrating a legacy app's XML based configuration to .NET Core's JSON based style? 

With emphasis on safely. Let's have a look. 

![](/images/post-images/rima-kruciene-gpKe3hmIawg-unsplash.jpg)

## A proposed solution

It's possible to migrate our application's configuration while still targeting .NET Framework. To do so, we'll need to install and use the `Microsoft.Extensions.Configuration` package, and refactor our application to not depend on the `System.Configuration.ConfigurationManager.` This may sound easy, but it can in fact be very complex depending on the size of the application.

To safely perform this migration, I propose a 4-step approach:

1. Install `Microsoft.Extensions.Configuration` and create a custom configuration provider 
2. Remove any static reference to the `System.Configuration.ConfigurationManager`
3. Convert `web.config` and `app.config` to `appsettings.json`
4. Leverage strongly typed configuration classes

Let's have a look at each step individually.

### 1. Install `Microsoft.Extensions.Configuration` and create a custom configuration provider

We can continue to read configuration values from the `web.config` while migrating to use `Microsoft.Extensions.Configuration`. [Ben Foster](https://benfoster.io/blog/net-core-configuration-legacy-projects) has written an excellent blog post on how to do this, in which he creates a custom configuration provider that reads and parses values from the `web.config`. Before doing anything else, I would recommend you:

1. Install the following NuGet packages:

   * `Microsoft.Extensions.Configuration`
   * `Microsoft.Extensions.Abstractions`
   * `Microsoft.Extensions.Primitives`
2. Create a custom configuration provider by following the steps in [Ben Foster's](https://benfoster.io/blog/net-core-configuration-legacy-projects) post
3. Add the following to your `Global.asax` or `Startup.cs` file

   ```
   IConfiguration configuration = new ConfigurationBuilder()
       .Add(new LegacyConfigurationProvider())
       .Build();
       
     //Register IConfiguration in your DI framework here
   ```

### 2. Remove any static reference to the `System.Configuration.ConfigurationManager`

Many legacy .NET Framework apps either directly use the static `System.Configuration.ConfigurationManager` instance to fetch their configuration values, or have their own abstraction of it so that it's not directly exposed across the application. Regardless of how your application handles it, we will need to replace the usage of the `System.Configuration.ConfigurationManager` across the application with `IConfiguration` registered above. I would recommend doing step 1 and 2 together in the same PR. This change would then be able to safely go to production once all automated tests passes.  

### 3.  Convert `web.config` and `app.config` to `appsettings.json`

Alright, so we have the piping set up, and our application now depends on `IConfiguration` instead of on the `System.Configuration.ConfigurationManager.` The next step is to convert our `web.config` to `appsettings.json`

There are a couple of things to keep in mind while doing this:

#### 3.1 Organize our configuration values into sections

Take full advantage of sections.

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

As we are now using sections, we'll have to make sure to update any reference to a config value so that it is prefixed with the section name. This can be easy to forget, especially if you're using a tool such as Octopus to transform your configuration before deployment.

### 4. Leverage strongly typed configuration classes

The final step in the transformation is to refactor our application to depend on strongly typed configuration classes instead of on `IConfiguration`. For each section in the `appsettings.json` file, create a corresponding settings class. Such a settings class could look like below:

```
using System; 

public class FeatureToggles 
{
    public bool Feature1Enabled { get; set; }
  
    public bool Feature2Enabled { get; set; }
}
```

We can then bind a class to each section and register the instance in our DI framework, which will allow us to inject specific settings classes only to instances that depend on them. A great separation of concern.

```
IConfiguration configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json.config", optional: true)
    .Build();
    
var featureToggles = configuration.GetSection(nameof(FeatureToggles));
//Register the instance in your DI framework here
```

### Summary

I hope this post has given you some ideas on how you can get started migrating the configuration in your .NET Framework app to .NET Core. Changing the way we configure our application can be risky, but by doing so in small increments, each tested with automated tests and released to production, can make it less of a heavy lift. As always feel free to reach out on Twitter or LinkedIn, should you have any questions. Happy coding!
