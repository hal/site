---
title: "What is new in HAL 3.1.2"
date: 2019-02-14T08:30:25+02:00
description: "What are the new and noteworthy features of HAL 3.1.2 as part of Wildfly 16."
tags:
- runtime
- domain
- configuration
---

[HAL 3.1.2](http://hal.github.io/releases/) is released and packaged as part of [Wildfly 16](http://wildfly.org/downloads/) (which is in Candidate Release as of now and soon will release a Final).

HAL 3.1.2 brings many new features, bug fixes and enhancements, checkout the [release notes](http://hal.github.io/releases/) since HAL 3.0.6, which is the version packaged in Wildfly 14 and 15.

The new and noteworthy features are:

* Support socket log handler
* Show logs from logging-profile in log viewer
* Display microprofile health checks
* Expose CA management operations for key-store and certificate-authority-account
* Enhance the SSL Wizard to add support to LetsEncrypt CA
* Support runtime operations for messaging servers
* Add support to remote ActiveMQ server
* Display the the network open ports
* Show active management operation from hosts and servers in a central place
* Show unresponsive controller better
* Basic support to MicroProfile Metrics and Config

If you find any problem, bug or have a suggestion on how to improve HAL, please [open an issue or contact us](/documentation/contribute/).

## Support socket log handler

The socket handler allows to write the logs to a remote socket server, it is located in the `Handler` menu of the `Logging` subsystem.

{{< screenshot src="/img/blog/whats-new-3.1.2/socket-handler-nav.png" >}}

But, before add a new socket handler, you should create a socket binding, where the remote socket server is listening. In this sample, the remote server is identified by `socket_handler1` socket binding. 

{{< screenshot src="/img/blog/whats-new-3.1.2/socket-handler-add.png" >}}

Then to use this log handler, add your log category and set this handler for it.



## Show logs from logging-profile in log viewer

Wildfly logging allows to completely separate logging between deployed applications by using `Logging Profile`, if the application uses different logging categories, the logging profile catches all log statements.

You can configure a logging profile to write to a subdirectory of default logging directory `jboss.server.log.dir`. Then HAL should display log files from file handlers of logging profiles too.

In this sample, the `helloworld.log` is the log file from `helloworld` logging profile.

{{< screenshot src="/img/blog/whats-new-3.1.2/logging-profile-1.png" >}}

The preview pane.

{{< screenshot src="/img/blog/whats-new-3.1.2/logging-profile-2.png" >}}

## Display microprofile health checks

If your MicroProfile application makes use of health checks, then HAL displays them.

The preview pane shows the main outcome of the health check and the first 10 checks. Also, you can refresh the preview pane.

{{< screenshot src="/img/blog/whats-new-3.1.2/mp-health-preview.png" >}}

If there are more than 10 checks, you can open a more detailed view.

{{< screenshot src="/img/blog/whats-new-3.1.2/mp-health-view.png" >}}


## Expose CA management operations for key-store and certificate-authority-account

Recently Wildfly added support for management of certificates issued by [LetsEncrypt](https://letsencrypt.org/). This support allows Wildfly to enable SSL on an https-listener or the management interface, so you don't have to handle the manual process, let Wildfly do it for you.

In the Certificate Authority Account section you can manage the account details. At this moment the only provider is **LetsEncrypt**.

{{< screenshot src="/img/blog/whats-new-3.1.2/caa-ops.png" >}}

In the Key Store view, the list of available actions.

{{< screenshot src="/img/blog/whats-new-3.1.2/keystore-ops.png" >}}

There is an operation to `Obtain` a certificate from a `Certificate Authority Account`.

{{< screenshot src="/img/blog/whats-new-3.1.2/keystore-obtain.png" >}}

You can also, check the certificate details:

{{< screenshot src="/img/blog/whats-new-3.1.2/keystore-details.png" >}}



## Enhance the SSL Wizard to add support to LetsEncrypt CA

There is a wizard to help users to enable SSL on a https-listener or the management console.

The wizard was enhanced to allow an option to obtain the certificate from **LetsEncrypt** 

{{< screenshot src="/img/blog/whats-new-3.1.2/ssl-enable-letsenc.png" >}}


## Support runtime operations for messaging servers

For messaging server, there is two options to **Force Failover** and **Reset**

{{< screenshot src="/img/blog/whats-new-3.1.2/msg-monitor-server.png" >}}

A more detailed view of the runtime management of a running messaging server. There are options to:

* Monitor conneciont to an embedded ActiveMQ server
* The count of sessions from each connection
* The sessions and consumers of each connection
* Close connections for an IP address or user
* List of consumers, producers, connectors and roles
* List of ongoing transactions where the admin user can commit or rollback

{{< screenshot src="/img/blog/whats-new-3.1.2/msg-monitor-view.png" >}}



## Add support to remote ActiveMQ server

If there is a need to use a remote ActiveMQ server, it can be managed in the `Remote ActiveMQ Server` section.

{{< screenshot src="/img/blog/whats-new-3.1.2/msg-remote-mq.png" >}}

Allowed settings:

* Connectors
* Discovery Group
* Connection Factory
* Pooled Connection Factory
* External JMS Queue
* External JMS Topic

{{< screenshot src="/img/blog/whats-new-3.1.2/msg-external-jms.png" >}}


## Display the the network open ports

Sometimes it is useful to know which ports are open and listening. Navigate to the `Runtime -> Server` and the open ports are displayed in the preview pane.

It shows the service name and the correct bound port.

{{< screenshot src="/img/blog/whats-new-3.1.2/open-ports.png" >}}


## Show active management operation from hosts and servers in a central place

The [management operations](/blog/management-operations/) view in domain mode displayed information per host controller and the user should open each host controller and it didn't display the management operations from individual servers, a bit unproductive.

Then we enhanced this view, where the management operations from host controllers and servers are aggregated and displayed for ease of use on domain mode. Navigate to `Runtime` and click on `Management Operations` item.

{{< screenshot src="/img/blog/whats-new-3.1.2/mo-column-domain.png" >}}

A sample view. There is a specific blog post about the [management operations](/blog/management-operations/) feature.

{{< screenshot src="/img/blog/whats-new-3.1.2/mo-view.png" >}}


## Show unresponsive controller better

There is automatic notification in the header if there is an ongoing non progressig operation that may be slowing the server down.

{{< screenshot src="/img/blog/whats-new-3.1.2/mo-notification.png" >}}

If the user clicks on the top notification, it will open the management operations view.

The automatic notification is configurable in the settings dialog.

## Basic support to MicroProfile Metrics and Config

There is basic support to MicroProfile Config and Metrics specifications.

It is available in the `Configuration` menu.


{{< screenshot src="/img/blog/whats-new-3.1.2/mp-config-metrics.png" >}}

The MicroProfile Configuration view.

{{< screenshot src="/img/blog/whats-new-3.1.2/mp-config.png" >}}

The MicroProfile Metrics view.

{{< screenshot src="/img/blog/whats-new-3.1.2/mp-metrics.png" >}}

