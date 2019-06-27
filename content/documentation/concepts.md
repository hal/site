---
title: "Basic Concepts"
date: 2018-03-08T16:29:20+01:00
description: "Describes the most important concepts in HAL from the user's point of view. Explains the basic UI components and how they are used."
icon: "/img/blueprint.png"
toc: true
weight: 20
---
HAL uses a set of UI components which follow the design guides from [PatternFly](https://www.patternfly.org). If necessary, these UI components are extended and adapted to the needs of HAL. The following chapters explain the most important concepts and their use.

# Header & Footer

The header and footer provide access to global features and give feedback about various states of WildFly. The color of the top line in the header shows the used <a name="theme">theme</a>: 

- <span style="background-color:#f0ab00">&nbsp;&nbsp;&nbsp;&nbsp;</span> HAL ([standalone mode]({{< relref "/documentation/get-started.md#standalone-mode" >}}))
- <span style="background-color:#39a5dc">&nbsp;&nbsp;&nbsp;&nbsp;</span> WildFly (community)
- <span style="background-color:#c82e2e">&nbsp;&nbsp;&nbsp;&nbsp;</span> EAP (product)

The header also contains the top level navigation. The navigation changes depending on whether the finder or an application is used. Please refer to the [manual]({{< relref "/documentation/manual.md" >}}) to find out about all elements in the header and footer. 

# Finder
{{< imgflow src="/img/documentation/finder.png" float="right" >}} 
The finder is used as a primary way to navigate in HAL. All top level categories (except the homepage) use the finder for its navigation. The finder contains a number of columns which stack from left to right and one preview area. A maximum of four columns are visible at once. If more columns are required, the column on the far left is hidden and the new column is inserted on the right.

Columns are used to show both logical levels and resource hierarchies from the management model. Each column has a title, an optional set of actions and an optional filter. Columns contain items which are all of the same kind. Each item has a title, an optional status icon, an optional subtitle and a list of actions. Items in one column are ordered alphabetically by default. If supported by the column, you can pin frequently used items to stay at the top.  

Selecting an item updates the preview pane. You can use the cursor keys for navigation inside and across columns. To execute the main action of an item press ↵ (enter). If more actions are available for an item use the dropdown to access them. Many items offer a different set of action depending on the state of the item (e.g. if a server is running, there won't be a 'Start' action).
{{</ imgflow >}}

# Applications
{{< imgflow src="/img/documentation/application.png" float="left" >}}
Applications are views which use the full browser width. In general they are opened by executing actions of finder items. Applications are used to view and modify resources of the WildFly management model. Other use cases include global features such as the model browser or macro recording. 

Applications normally modify the navigation in the header. Most applications show a breadcrumb in the header to easily navigate between items of the same kind. In addition an application can have its own navigation which is shown on the left hand side. This navigation is restricted to two levels. If an application needs more than two levels of navigation, an internal breadcrumb is used.   

Besides that applications can show icons in the right area of the header. This is used by many applications which manage resources from the management model:

- <i class="fas fa-sitemap"></i> opens the resource in expert mode. The expert mode uses the generic [model browser]({{< relref "/documentation/manual.md#management-model" >}}) which shows all resources and attributes managed by the application. This might be useful in case the application does not yet include attributes which have been recently added to the management model.  

- <i class="fas fa-external-link-alt"></i> opens the application in an external window. The external window will just show the application without the header and footer. This is especially useful for applications which monitor or trace resources (e.g. log files).
{{</ imgflow >}}
