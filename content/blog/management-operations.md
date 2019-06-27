---
title: "Monitor active management operations and detect non-progressing operations"
date: 2019-02-13T08:30:25+02:00
description: "How to monitor active management operations and detect non-progressing operations."
tags:
- runtime
---
A management operation is when any api client that connects to the Wildfly kernel and run operations to read from a running server or write a configuration. Examples of a management operation:

* Add a datasource.
* Deploy an application.
* Change the undertow buffer pool.
* A domain controller updates a slave server configuration.
* Host/Server synchronization.

The following tasks are not management operations:

* A HTTP request from a browser to a deployed application.
* An application sending a jms message.
* An application writing to a database.

Wildfly modular kernel allows extensions (also know as subsystem) to read/write from/to management model, it can be configuration or runtime data (performance metrics, read-only, etc.). Those requests needs strict control over to not read or write bad data. These api requests are exposed as management operations for the administrator to inspect it and cancel it. 

If for some reason, a management operation from an unknown client IP is performing an operation, the administrator may cancel it.

Since Wildfly 13, HAL (Web Management Console) shows the running management operations in an user friendly way. A sample screen is:

{{< screenshot src="/img/blog/management-operations/mo-view.png" >}}

## Open the management operations view

In domain mode, navigate to `Runtime -> Management Operations` and click `View`. 
It shows the management operations of all host controllers and servers.

In standalone mode, navigate to `Runtime -> Server -> Monitor -> Management Operations` and click `View`.

{{< screenshot src="/img/blog/management-operations/mo-column-domain.png" >}}


## API Client

The API client is any program to remotely connect to the Wildfly management endpoint.

* jboss-cli.sh
* HAL
* A subsystem

## How to list the active management operations

HAL displays the currently running operations.

{{< screenshot src="/img/blog/management-operations/mo-view.png" >}}


The example above shows the following information:

* The green icon on the left, it is an ongoing management operation.
* If it is a red icon, the operation is non progressing.
* Unique id of the management operation.
* The status of the operation can be: executing, awaiting-other-operation, awaiting-stability, completing, rolling-back.
* The address where the operation is running.
* The operation name.
* Access mechanism can be HTTP for a the web management console, "NATIVE" for jboss-cli.sh, JMX and IN_VM_USER.
* Running time: Amount of time the operation has been executing.
* Exclusive running time: Amount of time the operation has been executing with the exclusive operation execution lock held.

If running in domain mode, there are two specific fields:

* Host: Name of the host controller running the operation.
* Server: Name of the server under a host controller running the operation.

You can filter and sort by selecting different fields.

## Canceling a management operation

The administrator can cancel any operation, by clicking on the button "Cancel" on the right side of the operation line.

{{< screenshot src="/img/blog/management-operations/mo-view-cancel-op.png" >}}


## Non Progressing Operations

There are operations that requires an exclusive lock, for example:

* Write data to the configuration xml file (host.xml, domain.xml, etc.).
* A subsystem explicitly require an exclusive lock.
* Deployments

But most operations are not exclusive.

If there is an operation that acquired the exclusive lock and is running for more than 15 seconds, it is flagged as non progressing.
A non progressing operation, may cause problems to Wildfly, such as: slow performance, unable to read or write configuration or runtime data.

A example of a non progressing operation.

{{< screenshot src="/img/blog/management-operations/mo-view-non-prog.png" >}}


The administrator can cancel any non progressing operation, by clicking on the "Cancel Non Progressing Operation", on the top, right side. 

If a non progressing operation was canceled, the notification shows the operation id that was canceled.

{{< screenshot src="/img/blog/management-operations/mo-view-cancel-non-prog.png" >}}


## Automatic Notification of Non Progressing Operation

Starting in Wildfly 16, there is automatic notification in the header if there is an ongoing non progressig operation.

{{< screenshot src="/img/blog/management-operations/mo-notification.png" >}}

If the user clicks on the top notification, it will open the management operations view.

The automatic notification is configurable in the settings dialog.

The user can configure the following fields:

* Poll time: The number of seconds to elapse for HAL to perform a remote network call to Wildfly management interface.
* Poll: Enables or disables the polling mechanism. 

{{< screenshot src="/img/blog/management-operations/mo-settings.png" >}}

