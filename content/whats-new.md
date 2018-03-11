---
title: "What's New"
date: 2018-03-08T16:28:57+01:00
toc: true
---
Starting with version 3.x we've rewritten the console from scratch. We still use a similar [technical stack]({{< relref "development/architecture.md" >}}), but we removed a lot of old and deprecated code, refactored the main business logic and rewrote pretty much all of the UI related code. We moved from [GWT widgets](http://www.gwtproject.org/doc/latest/RefWidgetGallery.html) to [Elemento](https://github.com/hal/elemento) and prepared the codebase for the upcoming GWT 3.0 release. We also fully adopt [PatternFly](https://www.patternfly.org/) now.  

Finally we took the opportunity to enhance the existing features and added support for many new subsystems and attributes. The following sections show some highlights of the latest version. Most screenshots show the new version on the left and the old on the right side. For all details see the [3.0.0.Final release notes]({{< relref "releases/3.0.0.Final.md" >}}). 

# Finder

The finder has been greatly improved. You can now use the cursor keys for navigation inside and across columns. To open an application press ↵ (enter) to go back press ⌫ (backspace). Items in one column are now ordered alphabetically by default. You can pin frequently used items to stay at the top. Most columns offer a filter which can be used to quickly find the items you're looking for. Finally the previews have been enriched and provide detailed documentation or the main attributes of the selected resource. 

{{< cocoen before="img/whatsnew/before/finder.png" after="img/whatsnew/after/finder.png" reverse="true" >}} 

# Applications

Applications provide a new breadcrumb at the top to quickly switch between items of the same kind. Furthermore more complex applications can include a vertical navigation. Finally most applications can be easily opened in an external window and provide an expert mode which uses the generic model browser.  

{{< cocoen before="/img/whatsnew/before/application.png" after="/img/whatsnew/after/application.png" reverse="true" >}} 

# Deployments

Many new features have been added to the deployment section: 

- Use drag and drop to deploy artifacts
- Content browser with preview for text and images
- Create exploded deployments
- CRUD support for exploded deployments:
  - Add empty files
  - Upload content
  - Modify content
  - Remove content
- Download complete deployments or deployment content
 
{{< cocoen before="/img/whatsnew/before/deployment.png" after="/img/whatsnew/after/deployment.png" reverse="true" caption="Deployments" >}} 
{{< cocoen before="/img/whatsnew/before/deployment-model.png" after="/img/whatsnew/after/deployment-model.png" reverse="true" caption="Deployment Model" >}} 
{{< screenshot src="/img/whatsnew/after/deployment-content.png" caption="The new content browser" >}}

# Topology

The topology view has been reintroduced to the HAL management console. It was removed in the last versions due to performance issues with large domains. But thanks to new management operations, we were able to add this useful tool again.

{{< screenshot src="/img/whatsnew/after/topology.png" >}}

# Runtime

The lifecycle operations for hosts, server groups and servers have been improved. New operations are available for hosts and disconnected hosts are now shown in the finder columns. For servers you can specify custom URLs which is extremely useful when running WildFly inside a docker container.

{{< cocoen before="/img/whatsnew/before/runtime.png" after="/img/whatsnew/after/runtime.png" reverse="true" >}} 

# Monitor

The existing screens have been improved and many new subsystems have been added to the monitoring section. Some of the new and enhanced subsystems are:

- Batch
- EJB
- IO
- JAX-RS
- Messaging
- Web (Undertow)

{{< cocoen before="/img/whatsnew/before/monitor-server.png" after="/img/whatsnew/after/monitor-server.png" reverse="true" >}} 
{{< screenshot src="/img/whatsnew/after/monitor-ejb.png" caption="EJB Subsystem" >}}
{{< screenshot src="/img/whatsnew/after/monitor-jaxrs.png" caption="JAX-RS Resources" >}}
{{< screenshot src="/img/whatsnew/after/monitor-undertow.png" caption="Undertow Listener Statistics" >}}
