---
title: "End User"
date: 2018-03-08T16:29:20+01:00
icon: "/img/book.png"
toc: true
weight: 20
---
The HAL management console features a rich user interface to configure, analyse and monitor the complete WildFly management model. The console uses six main sections: homepage, deployments, configuration, runtime, patching and access control.
 
<!--more-->

This page gives a quick introduction to the main sections. If you want to learn more about the most common use cases, visit the [YouTube channel](https://www.youtube.com/channel/UCcXaCVPvmzgLz74FnzlWoSg/) and watch the screencasts.

# Homepage

{{< imgflow src="/img/documentation/homepage.png" float="right" >}}
The homepage provides an overview of the main sections in the console. For each section there's a short introduction followed by a step-by-step guide of the main use case. Links allow you to jump directly into the topic. Finally the homepage provides a list of related resources and additional help.

The homepage is the page which is opened by default and you can always go back to the homepage by clicking on the logo in the header.  
{{</ imgflow >}}

# Deployments

{{< imgflow src="/img/documentation/deployments.png" float="left" >}}
The deployment section is all about adding and managing deployments. You can upload existing deployments, add unmanaged deployments or create new empty deployments for quick prototyping. 

In a managed domain, deployments are associated with a `server-group`. Any server within the server group will then be provided with that deployment. The domain and host controller components manage the distribution of binaries across network boundaries.

Deployments can have different states:

- enabled / disabled
- managed / unmanaged
- archived / exploded

**Enabled / Disabled**

Once a deployment has been enabled it is active and can be accessed by the user. If the deployment defines a context root, it's visible in the deployment preview.  

**Managed / Unmanaged**

WildFly supports two mechanisms for dealing with deployment content – managed and unmanaged deployments.

With a managed deployment the server takes the deployment content and copies it into an internal content repository and thereafter uses that copy of the content, not the original user-provided content. The server is thereafter responsible for the content it uses.

With an unmanaged deployment the user provides the local filesystem path of deployment content, and the server directly uses that content. However the user is responsible for ensuring that content, e.g. for making sure that no changes are made to it that will negatively impact the functioning of the deployed application.

Managed deployments have a number of benefits over unmanaged:

- They can be manipulated by remote management clients, not requiring access to the server filesystem.
- In a managed domain, WildFly/EAP will take responsibility for replicating a copy of the deployment to all hosts/servers in the domain where it is needed. With an unmanaged deployment, it is the user's responsibility to have the deployment available on the local filesystem on all relevant hosts, at a consistent path.
- The deployment content actually used is stored on the filesystem in the internal content repository, which should help shelter it from unintended changes.

**Archived / Exploded**

Managed and unmanaged deployments can be 'exploded', i.e. on the filesystem in the form of a directory structure whose structure corresponds to an unzipped version of the archive. An exploded deployment can be convenient to administer if your administrative processes involve inserting or replacing files from a base version in order to create a version tailored for a particular use (for example, copy in a base deployment and then copy in a jboss-web.xml file to tailor a deployment for use in WildFly.) Exploded deployments are also nice in some development scenarios, as you can replace static content (e.g. .html, .css) files in the deployment and have the new content visible immediately without requiring a redeploy.

Please note that you have to disable a deployment, before you can explode it. Once exploded you can use the console to browse the deployment's content, add, replace or remove files. 
{{</ imgflow >}}

# Configuration

{{< imgflow src="/img/documentation/configuration.png" float="right" >}}
The configuration section provides access to all important resources used to configure the server. This includes

- subsystems
- interfaces
- socket bindings
- paths
- system properties

The majority of the configuration section are the subsystems. By default the console lists all subsystems in alphabetical order. Depending on the profile in domain mode and the used configuration file in standalone mode, this list includes different entries. 

More complex subsystems like Infinispan or messaging add additional columns to the finder. Other simpler subsystems are managed by a single application. 
{{</ imgflow >}}

# Runtime

{{< imgflow src="/img/documentation/runtime.png" float="left" >}}
The runtime section...Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.
{{</ imgflow >}}

# Patching

The patching section...Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. 

Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.

# Access Control

WildFly introduces a Role Based Access Control scheme that allows different administrative users to have different sets of permissions to read and update parts of the management tree. This replaces the simple permission scheme used in JBoss AS 7, where anyone who could successfully authenticate to the management security realm would have all permissions.

WildFly ships with two access control "providers", the "simple" provider, and the "rbac" provider. The "simple" provider is the default, and provides a permission scheme equivalent to the JBoss AS 7 behavior where any authenticated administrator has all permissions. The "rbac" provider gives the finer grained permission scheme that is the focus of this section.

The access control scheme implemented by the "rbac" provider is based on seven standard roles. A role is a named set of permissions to perform one of the actions: addressing (i.e. looking up) a management resource, reading it, or modifying it. The different roles have constraints applied to their permissions that are used to determine whether the permission is granted.

## RBAC Roles

The seven standard roles are divided into two broad categories, based on whether the role can deal with items that are considered to be "security sensitive". Resources, attributes and operations that may affect administrative security (e.g. security realm resources and attributes that contain passwords) are "security sensitive".

Four roles are not given permissions for "security sensitive" items:

- Monitor – a read-only role. Cannot modify any resource.

- Operator – Monitor permissions, plus can modify runtime state, but cannot modify anything that ends up in the persistent configuration. Could, for example, restart a server.

- Maintainer – Operator permissions, plus can modify the persistent configuration.

- Deployer – like a Maintainer, but with permission to modify persistent configuration constrained to resources that are considered to be "application resources". A deployment is an application resource. The messaging server is not. Items like datasources and JMS destinations are not considered to be application resources by default.

Three roles are granted permissions for security sensitive items:

- SuperUser – has all permissions. Equivalent to a JBoss AS 7 administrator.

- Administrator – has all permissions except cannot read or write resources related to the administrative audit logging system.

- Auditor – can read anything. Can only modify the resources related to the administrative audit logging system.

## Switching to the "rbac" provider

Before changing the provider to "rbac", be sure your configuration has a user who will be mapped to one of the RBAC roles, preferably with at least one in the Administrator or SuperUser role. Otherwise your installation will not be manageable except by shutting it down and editing the xml configuration. If you have started with one of the standard xml configurations shipped with WildFly, the "$local" user will be mapped to the "SuperUser" role and the "local" authentication scheme will be enabled. This will allow a user running the CLI on the same system as the WildFly process to have full administrative permissions. Remote CLI users and web-based admin console users will have no permissions.
