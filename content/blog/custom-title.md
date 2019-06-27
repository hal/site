---
title: "Custom Title"
date: 2019-06-27T10:30:25+02:00
description: "Customize the title of the management console"
tags:
- settings
---
[HAL 3.2.0](https://hal.github.io/releases/) which is part of [Wildfly 17](https://wildfly.org/downloads/) comes with a new option to customize the title of the browser window / tab. This is especially useful if you have multiple HAL windows open and need to know which console manages which WildFly instance. 

The new features makes use of two attributes in the root resource of the management model:

1. `name`: The name of this server. If not set, defaults to the runtime value of `InetAddress.getLocalHost().getHostName()`.
1. `organization`: Identification of the current organization running this server.

You can use these attributes as part of your title. Use `%n` to refer to the name and `%o` to refer to the organization. By default the title is `%n | Management Console`. To change the title select "Settings" in the footer: 

{{< screenshot src="/img/blog/custom-title/settings.png" >}}
