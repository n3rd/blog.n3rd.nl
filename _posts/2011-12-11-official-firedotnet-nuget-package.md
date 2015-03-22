---
layout: post
title: Official FireDotNet NuGet package
tags: [ASP.NET, FireDotNet, NLog, NuGet]
description:
---

FireDotNet is now available as an official NuGet package on [nuget.org](http://nuget.org/packages/NLog.Targets.FireDotNet):

![firedotnet-nuget-badge](https://cloud.githubusercontent.com/assets/3578344/6769736/8806f676-d0a7-11e4-984a-26545cdc005a.png)

[FireDotNet](http://code.google.com/p/firedotnet/) is a library I created to show ASP.NET server-side log messages in the Firebug console on the client-side. A few months ago I updated the library to a [NLog](http://nlog-project.org/) target so all NLog messages are send to Firebug.

![FireDotNetScreenShot](https://cloud.githubusercontent.com/assets/3578344/6651638/9d6c8272-ca4b-11e4-9a15-c34fc1813095.png)

I used this opportunity to make some changes too. First of all I automated the builds using TeamCity. I installed a NuGet plugin for TeamCity so dependencies are automatically resolved. This means that I no longer need to have the NLog.dll files under version control. Besides managing dependencies it can also automatically make a NuGet package and publish it to both my private NuGet server and the official NuGet server.

Another nice feature of [TeamCity](http://www.jetbrains.com/teamcity/) is that it can manage version numbers for you. There is a *build feature* named *AssemblyInfo patcher* and - as the name already implies - it modifies the version number of your AssemblyInfo.cs files. So from now on all FireDotNet builds will have a version number that consists of:

`MajorVersion.MinorVersion.BuildNumber.SvnRevisionNumber eg. 0.2.30.34`
**
Where I control the first two numbers and the last two are automatically set by TeamCity.

Last but not least I managed to add a new feature too! From now on FireDotNot by default only outputs to requests form localhost. This means that you, when deploying your project, no longer have to think of removing FireDotNet from it. Except of course if you changed the allowRemote property.
