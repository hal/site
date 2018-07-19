---
title: "Inspect HTTP Sessions"
date: 2018-07-19T14:03:25+02:00
description: "HAL supports many new operations to inspect the HTTP session of deployments."
tags:
- deployment
- runtime
---
The `undertow` subsystem in deployments has been enhanced and provides new operations to inspect HTTP sessions. 

## Session Statistics

To see session statistics please select a deployment under "Runtime / Server / Web / Deployment". Besides the main attributes like deployment name and context path, the preview shows statistics for the number of active, expired and invalid sessions. Besides that you can see statistics about the session time. 

{{< screenshot src="/img/blog/session/stats.png" >}}

## Session Attributes

If you select a deployment and press "View", you can see more details about the deployment. This falls into three categories: 

1. Sessions
1. Servlets
1. Web Sockets

In the section about sessions, you can see all active sessions. Select a session to see all session attributes. Finally you can invalidate a session by pressing "Invalidate Session".

{{< screenshot src="/img/blog/session/attributes.png" >}}
