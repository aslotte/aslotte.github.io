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
