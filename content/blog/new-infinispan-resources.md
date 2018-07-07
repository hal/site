---
title: "New Infinispan Resources"
date: 2018-07-01T15:48:25+02:00
description: "The latest HAL build comes with support for many new Infinispan resources such as remote cache container and new store implementations."
draft: true
tags:
- configuration
---
In WildFly 13 the Infinispan subsystem introduces several new resources: 

- Remote Cache Container
- Scattered Cache
- HotRod Store 

All of these resources are supported in the latest HAL version (3.0.3 and above). 

## Remote Cache Container

Remote cache containers are listed together with regular cache containers in the 'Cache Container' column. You can use the button drop down to add both remote and regular cache containers.

{{< screenshot src="/img/blog/infinispan/remote-cache-container.png" >}}

## Scattered Cache

All caches of the selected cache container are now displayed in their own column. To add a new cache select the cache type from the button drop down. Each cache provides a rich preview with the most important attributes. The list of caches includes the new scattered cache. 

{{< screenshot src="/img/blog/infinispan/scattered-cache.png" >}}

## HotRod

In the application view of a cache you can choose the new HotRod store implementation.  

{{< screenshot src="/img/blog/infinispan/hotrod.png" >}}
