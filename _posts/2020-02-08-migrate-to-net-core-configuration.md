---
title: Migrate to .NET Core configuration
comments: true
tags:
  - dotnetcore
date: '2020-02-08 08:44 -0500'
---
## Problem

Migrating a large enterprise application written in .NET Framework to .NET Core can be  a lot of work and risky. But the benefits once complete are worth the effort. As I'm currently in the middle of such a migration, I wanted to take a moment to discuss a particular are of interest, **configuration**. Despite the best efforts to organize web.config and app.configs, a large application's configuration can quickly become complex. How do one go about migrating a legacy app's XML based configuration to leverage .NET Core's, among others, JSON based configuration style? Let's have a look. 



## Proposed solution
