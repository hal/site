---
title: "Build & Run"
date: 2018-03-08T16:29:52+01:00
description: "Explains how to build and run the console. Talks about the prerequisites and what is necessary to debug the codebase."
icon: "/img/tools.png"
toc: true
weight: 10
---
This page explains how to build, run and debug the console. We recommend to use Maven and the command line. This will work reliably across different environments and IDEs. 

# Build

If not already done, clone the code from https://github.com/hal/console/ or [fork](https://github.com/hal/console/fork) the repository into your own personal GitHub account.

For a full build use 

```bash
mvn clean install
``` 

This includes the GWT compiler, which might take a while. If you just want to make sure that there are no compilation or test failures, you canskip the GWT compiler and use

```bash
mvn clean install -Dgwt.skipCompilation
``` 

# Run

HAL is a GWT application and as such it is served from a local Jetty server. As a one time prerequisite you need to add the URL of the local Jetty server as an allowed origin to your WildFly / JBoss EAP configuration: 

**Standalone Mode**

```bash
/core-service=management/management-interface=http-interface:list-add(name=allowed-origins,value=http://localhost:8888)
reload
```
**Domain Mode**

```bash
/host=master/core-service=management/management-interface=http-interface:list-add(name=allowed-origins,value=http://localhost:8888)
reload --host=master
``` 
 
The main GWT application is located in the `app` folder. To run the console use

```bash
cd app
mvn gwt:devmode
```

This will start the development mode. Wait until you see a message like 

```
00:00:15,703 [INFO] Code server started in 15.12 s ms
```

Then open http://localhost:8888/dev.html in your browser and connect to your WildFly / JBoss EAP instance as described in [independent mode]({{< relref "documentation/get-started.md#independent-mode" >}}). 

# Debug

{{< imgflow src="/img/development/debug.png" float="right" >}}
Start the console as described in the previous chapter. GWT uses the [SourceMaps standard](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?usp=sharing) to map the Java source code to the transpiled JavaScript code. This makes it possible to use the browser development tools for debugging.

In Chrome open the development tools and switch to the 'Sources' tab. Press <kbd>⌘ P</kbd> and type the name of the Java source file you want to open. 

Let's say we want to debug the enable / disable action in the data source column in configuration. Open the class `DataSourceColumn` and put a breakpoint on the first line of method `void setEnabled(ResourceAddress, boolean, SafeHtml)` (should be line 285). Now select a data source like the default 'ExampleDS' data source and press the enable / disable link in the preview. The browser should stop at the specified line and you can use the development tools to inspect and change variables. 
{{</ imgflow >}}

{{< callout warning >}}
##### Inspect Variables

If you're used to debug Java applications in your favorite IDE, the debugging experience in the browser development tools might feel strange at first. You can inspect simple types like boolean, numbers and strings. Support for native JavaScript types like arrays and objects is also very good. On the other hand Java types like lists or maps are not very well supported. In addition most variable names are suffixed with something like `_0_g$`. We recommend to inspect these variables using the console and call the `toString()` method on the respective object.    
{{</ callout >}}

# Develop

To apply changes made to Java code you just need to refresh the browser. GWT will detect the modifications and only transpile the changed sources. 

Changes to other resources require a little bit more effort. To make it easier, you can use the script `app/refresh.sh`. Change to the `app` folder and call `refresh.sh` with one of the following parameters, depending what kind of resource you've modified:

- `less`: Compile LESS stylesheets
- `html`: Update HTML snippets
- `i18n`: Process i18n resource bundles
- `mbui`: Regenerate MBUI resources

After calling the script, refresh the browser to see your changes. 
