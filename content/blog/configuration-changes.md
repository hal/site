---
title: "Log all configuration changes"
date: 2018-07-17T15:30:25+02:00
description: "How to log and display all configuration changes on a wildfly server"
tags:
- configuration
- runtime
---

There is a feature to record all configuration changes to an in-memory log per host or server, it records any change performed on Wildfly, for example: deploy an application, add a datasource, change any configuration, add  any resource. This blog post will show how it works.

Notes:

- For this blog post, domain mode will be used.
- After a domain or server restart all configuration changes are lost, because this is an in-memory logging.
- If you want a persistent audit logging, you must use the audit logging feature on `/host=master/core-service=management/access=audit` path.
- As in-meme, max-history memory impact.

## What is a configuration change

The `Configuration Change` is a JSON like structure:
```
{
    "operation-date" => "2018-07-17T20:42:27.728Z",
    "domain-uuid" => "65fe503f-64bb-4f7f-af30-9526b516f523",
    "access-mechanism" => "HTTP",
    "remote-address" => "localhost/127.0.0.1",
    "outcome" => "success",
    "operations" => [{
        "operation" => "composite",
        "address" => [],
        "steps" => [{
            "name" => "max-heap-size",
            "value" => "256m",
            "operation" => "write-attribute",
            "address" => [
                ("host" => "master"),
                ("server-config" => "server-two"),
                ("jvm" => "default")
            ]
        }],
        "operation-headers" => {
            "access-mechanism" => "HTTP",
            "caller-type" => "user"
        }
    }]
}
```

The change structure is:

- Access Mechanism: The source api that issued the command, **native** is a native api call (jboss-cli.sh) or **HTTP**.
- Remote Address: The source network address of the client that originated the change.
- Outcome: success or fail.
- Operations: Each change may be a single operation or a collection of operations. 
- Operation: A single operation on a single resource address, in the example above it is `write-attribute`
- Address: The path of the change, in the example above, the address is `/host=master/server-config=server-two/jvm=default`
- Name, Value pair: The attribute and value to change, in the example above it is: `max-heap-size=256m`

## Configuration changes in domain and standalone mode 

For domain mode the feature `Configuration Changes` may be per host and per profile. The `Configuration Changes` per host, will log all changes performed on the specific host and the servers under it. The `Configuration Changes` per profile, will log any changes performed on the specific profile and the associated server. For Wildfly 13, HAL handle only the `Configuration Changes` per host, but current HAL version (to be released with Wildfly 14) also features `Configuration Changes` per profile.

For standalone mode, the `Configuration Changes` will log any change.

## Loading configuration changes panel

Navigate to `Runtime` -> `Hosts` -> select a host, the click on the arrow to display de drop-down menu, then select **Configuration Changes** 

{{< screenshot src="/img/blog/configuration-changes/cc-01.png" >}}

The following screen is displayed, click the button to enable the recording of the changes.

{{< screenshot src="/img/blog/configuration-changes/cc-02.png" >}}

It will ask for the maximum number of entries to be recorded. Then, after confirmation, the following screen is displayed.

{{< screenshot src="/img/blog/configuration-changes/cc-03.png" >}}

It shows the last change on top.

## Show configuration changes

After a couple of changes there is the following entries, a detailed description of this view:

- Filter search results: On the top left, you can filter the results by:
 - outcome: valid values: failed, success
 - operation name
 - address
 - access mechanism: native or HTTP
- Sort by:
 - Outcome
 - Operation date
 - Remote address
 - Access Mechanism
 
The *Access Mechanism* means the *native* comes from jboss-cli.sh or API calls, the HTTP is self explanatory.

On the top right, there are two action buttons:

- Disable: Disable the *Configuration Changes*, it will remove the resource `/host=master/subsystem=core-management/service=configuration-changes`
- Reload: Reload the configuration changes list, by calling `/host=master/subsystem=core-management/service=configuration-changes:list-changes()`

After many changes are perfomed, there is a list of configuration changes.

{{< screenshot src="/img/blog/configuration-changes/cc-04.png" >}}

For each item on the list, there is a button `View` to show the full individual change.

If the change is large with many items, it will not be displayed right away, you must click on it to show, then the item will expand to show the change with more details.

{{< screenshot src="/img/blog/configuration-changes/cc-05.png" >}}

Then a large view of the change is displayed.

{{< screenshot src="/img/blog/configuration-changes/cc-06.png" >}}

For any message, you can click on the `View` button on the right to show the full raw message.

{{< screenshot src="/img/blog/configuration-changes/cc-07.png" >}}

## Pagination

On the bottom, there is a navigation bar to paginate the results

{{< screenshot src="/img/blog/configuration-changes/cc-08.png" >}}

## Disable the Configuration Changes

To disable, you must click on the `Disable` button.

{{< screenshot src="/img/blog/configuration-changes/cc-09.png" >}}


